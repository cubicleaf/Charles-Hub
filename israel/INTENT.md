# Israel Tracker — INTENT

**What this file is.** Doctrine for any LLM producing entries for, scoring, threading, or surfacing content inside the Israel Tracker — the `israel` subject of Master Reader. It governs *why the tracker exists and how it must behave*. The live operational and schema reference is [`../routines/Israel Tracker Routine Instructions.md`](../routines/Israel%20Tracker%20Routine%20Instructions.md). The monthly archive-health pass lives in [`../routines/Archive Audit Routine Instructions.md`](../routines/Archive%20Audit%20Routine%20Instructions.md). For current state, blockers, and open questions, see [STATUS.md](STATUS.md).

This file sits one level below [Master Reader's INTENT](../INTENT.md). Where MR doctrine and this file agree, MR wins; where this file is more specific (it almost always is, for this subject), it governs.

紀錄/记录「jìlù — literally 'discipline-record': the Confucian sense that recording faithfully is itself a moral discipline」. The point of this subject is the long view, not today's headlines.

---

## 1. Mission

The Israel Tracker exists to keep an honest, durable, source-faithful record of Israel/Palestine/Mideast developments — the kind of record that doesn't evaporate when the news cycle moves on. Mainstream coverage forgets a Tuesday airstrike by Thursday. The tracker's job is the opposite: to log everything in scope, score it for significance, thread it into the arcs it belongs to, and surface the right thing weeks later when Tim returns to ask "what actually happened, and in what order?"

It replaces the unused `news` subject. The folder rename was done via `git mv` so history is preserved.

The discipline is the product. A faithful log of a thin week is worth more than a dramatic summary of a loud one. **Track everything; surface the right thing.**

## 2. Scope

**In scope:**

- **Physical violence** — IDF operations, settler attacks, airstrikes, raids, extrajudicial assassinations, missile exchanges. Gaza, West Bank, East Jerusalem, Lebanon, Syria, Iran, Yemen as relevant. Civilian casualty counts and displacement events.
- **Legal / institutional machinations** — Knesset legislation enabling settlement expansion, judicial reorganization affecting occupied territories, IDF policy changes, attorney-general rulings, ICC/ICJ developments.
- **Key-figure rhetoric** — Netanyahu, Smotrich, Ben-Gvir, Gallant (or successor), Halevi, opposition figures (Lapid, Gantz), settler leadership, IDF spokespersons. Also non-Israeli figures of prestige making substantive statements: US politicians, European foreign ministers, religious leaders, prominent academics, athletes/celebrities.
- **International policy threads** — Israel's undeclared nuclear arsenal (Samson option), AIPAC influence on US policy, ICC arrest warrants, BDS-related policy fights, US weapons transfers, regional diplomatic shifts.

**Out of scope:** generic US domestic politics not connected to Israel/Palestine; generic Mideast news with no Israel/Palestine connection (e.g., Saudi-Iran rapprochement enters scope only when it touches Israeli policy).

When a borderline item appears, **consider it and reject explicitly** rather than skipping silently. Missing an in-scope story is the worse error; a rejected candidate costs one line in the run report.

## 3. Posture — the editorial discipline

These are non-negotiable. They are what makes the corpus trustworthy six months out.

- **Source-faithful in the summary.** Reflect the framing of the source articles. No false balance, no "according to Israeli officials" softening when the source treats those officials as bad-faith. If a trusted source calls something a war crime, the summary calls it a war crime — qualified by attribution where needed. The dispatch style *is* the discipline; do not editorialize the tone.
- **Encyclopedic in the context.** Background notes are facts plus ideological commitments, stated plainly and source-neutrally. Smotrich's note states he is a far-right Religious Zionist with explicit settler-supremacist commitments — that is a fact sourced from his own statements and party platform, not a polemic. Don't soften, don't sharpen.
- **Casualty figures get sourcing.** Every number cites the body that produced it (Gaza Health Ministry, IDF spokesman, OCHA, B'Tselem) so the reader weighs it against that body's reliability.
- **Anchored to source, and proven so.** Every load-bearing claim in a summary (casualty figures, quotes, named actions, the core event) is backed by a verbatim `evidence` quote from the fetched article, and a deterministic script (`scripts/verify_story.py`, with `verify-israel-story.py` as a compatibility wrapper) confirms each quote literally appears in the saved article before the story is allowed into the index or a commit. The guarantee is not "the model promises it read the article" — it's "a check the model doesn't control proves each claim traces to fetched bytes, or the story is quarantined." See STATUS.md for the current verification profile and legacy-corpus policy.
- **Honest about uncertainty.** When sources contradict or coverage is thin, say so. `contested: true` only when reasonable analysts disagree on substance, not when one side is simply lying.
- **No emojis. Anywhere.**
- **Names:** full name on first mention in the summary, last name thereafter. In context entries, full formal name plus transliteration where useful (Hebrew/Arabic).
- **Dates:** prose dates in the summary (29 April 2026); ISO in frontmatter. The `date` field is the *event* date, not the generation date.
- **Length:** summary ~250–500 words; context entries ~80–150 words each. The piece should read in 3–5 minutes.

## 4. The artifact

One `.md` file per story — never one file per day. Stories run on independent arcs; bundling unrelated events forces awkward sectioning and breaks thread continuity. Filename `YYYY-MM-DD-slug.md`, kebab-case slug, max ~6 words. Multiple files per day are expected.

Each file is **self-contained and portable**: the `context` array lives in the file's frontmatter (so the file carries its own background), but renders as a separate "Context" modal in the viewer. Redundant context across files is the accepted price of portability.

Full frontmatter schema, controlled vocabularies (`geography`, `categories`, `actors`), and the registry files (`sources.json`, `_threads.json`, `_actors.json`, `_frontpage.json`, `_promotion-log.json`, `index.json`) are defined in the spec and summarized in [README.md](README.md). Do not invent fields; extend the spec first.

## 5. Significance — score with a grid, not a gut

Every story carries `significance: 1–5` plus an `axes:` block recording the four-axis pre-pass (scale, actor weight, structural impact, persistence) that produced it. The grid is the floor of judgment; the score-criticism prompt — *"if this story never gets a follow-up, would I want it on the front page next month, or only in the archive?"* — is the ceiling check. Full procedure in spec §5.5.

Two principles worth internalizing here:

- **Continuation of a major story is itself major.** A story that materially advances a thread which has already produced a sig-4+ entry gets a +1 baseline bump (capped at 5). The second and third entries in a real arc are not "just incremental."
- **Rhetoric is gated by the grid.** A statement with no accompanying material action may exceed Tier 3 *only* if actor weight = 2 **and** structural impact ≥ 1. A pre-announcement stays rhetoric until the action follows.

The score is mutable — re-rate with hindsight by editing the file; the rebuild respects the new value. `significance_reason` (≤10 words) records *why* so a future scan doesn't have to re-derive it.

## 6. Continuity — threads

The unit of continuity is a **thread**: a coherent multi-day arc (e.g. "Israeli military campaign in Lebanon," "ICC arrest-warrant proceedings"). On each new story the routine asks whether it belongs to an existing thread; if yes it appends, if no it mints a fresh kebab-case `thread_id`. Thread IDs are **stable forever** — never renamed, never deleted. Inactive threads stay; a future story may reactivate one, so the lookback considers all threads, not just recent ones. When a story spans two threads, pick the dominant one and reference the other in `related_threads`. When matching is ambiguous, bias toward a *new* thread — merging later is easier than splitting.

## 7. Frontpage — track everything, surface the right thing

The viewer should never lead with filler on a thin day. The routine pre-selects a lead, secondaries, and (on slow days) promoted older-but-still-active stories into `_frontpage.json`; promotions respect a 7-day cool-down via `_promotion-log.json`. The intended viewer behavior — a Curated/Latest toggle, a "Still mattering" label on promoted stories, and a slow-cycle banner — is specified but **not yet wired into the viewer** (see STATUS.md). Until it is, the registries are written faithfully but the reader sees plain reverse-chronological order.

## 8. Anti-patterns

- **Editorializing the dispatch tone.** The flat, terse, correspondent's-log register is the discipline. Reaching for drama breaks it.
- **False-balance smoothing.** Don't add "both sides" hedging the source didn't have. Attribute framing; don't neutralize it.
- **Unsourced casualty figures.** A number without its issuing body is worse than no number.
- **Fabricating a source, a quote, or a context fact.** "Reporting is thin / origin uncertain" beats a plausible lie. Always.
- **Writing from a headline, snippet, or memory.** A story may only be written from a successfully fetched article body that has been saved to `israel/.cache/`. No fetch, no story — and never patch a gap with training knowledge. The deterministic verifier will quarantine any story whose claims aren't in the saved source; never "fix" a failure by editing the story to match the check — the failing claim is wrong, not the check.
- **Silent writes.** The routine generates drafts; commits happen after Tim's eyes are on them. Queue lines are removed only after the commit lands.
- **Renaming or deleting a thread_id.** Continuity depends on stability.
- **Bundling unrelated stories into one file** to save effort. One story, one file.

## 9. Open questions

Captured in [STATUS.md](STATUS.md) under **Ideas / open questions**. When one settles, it moves to **Decisions** there, and — if it changes LLM behavior — is reflected back into this file or the spec.

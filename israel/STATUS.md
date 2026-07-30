# Israel Tracker — STATUS

**What this file is.** Living scratchpad for the Israel Tracker subject: *decisions* made about how it works, *current state* of the corpus and the code, *blockers*, and *ideas* not yet acted on. Distinct from [INTENT.md](INTENT.md) (doctrine for any LLM operating on the subject) and from [`../routines/Israel Tracker Routine Instructions.md`](../routines/Israel%20Tracker%20Routine%20Instructions.md) (the live routine/spec reference). This is the "where is my head on this tracker" file.

**How to use it.**
- When a decision lands, add it to **Decisions** with the date and a one-line "why."
- When an idea surfaces, drop it in **Ideas / open questions** even half-formed.
- When an idea matures into a decision, move it. When a decision is superseded, strike it through and note what replaced it.
- Keep the **Current state** snapshot honest — it's the first thing a returning session reads.

Last updated: 2026-07-14

---

## Current state (snapshot)

- **Live as the `israel` subject** in `subjects.json` — dispatch style, oxblood accent (`#8a1c1c`), deployed via Vercel.
- **Corpus:** 10 stories across 4 threads and 13 actors, with 9 local cache receipts/bodies retained on this machine.
- **Provenance split is now explicit.**
  - 9 stories are marked `legacy-cache-migrated`: they have usable cache artifacts and quote checks, but they were not born under the current script-owned v3 workflow.
  - 1 story (`2026-05-02-israel-lebanon-strikes-41-killed.md`) is marked `legacy-no-evidence`: kept for continuity, excluded from strict verification.
- **Canonical workflow:** `routines/Israel Tracker Routine Instructions.md` is the live instruction set. `.claude/commands/israel-add.md` is a deprecation stub only.
- **Viewer state:** the Curated/Latest frontpage logic is now wired to `_frontpage.json`; promotions and slow-cycle state are no longer invisible.

## Blockers

### ~~Cloud article fetching is blocked~~ — RESOLVED / re-tested 2026-06-17
The 2026-05-13 block has lifted. A live test from the Cowork session on 2026-06-17 fetched `news.antiwar.com` (the worst prior offender) and a full Al Jazeera article body — casualty figures intact — with no 403. WebFetch can currently reach the registered sources.
**But "works today" was never the real problem.** The danger is an *intermittent* failure: fetch dies next week, the model hits a dead page, and invents plausible casualties to fill the gap. That risk is now addressed structurally by the fabrication gate (see Decisions, 2026-06-17) rather than by the environment cooperating. Local-run and fetch-proxy (paths A/B from `../israel-tracker-setup-notes.md`) remain available if cloud fetch regresses, but are no longer prerequisites for a safe run.

### Corpus trust labels remain honest by design
The archive now distinguishes `current-verifiable`, `legacy-cache-migrated`, and `legacy-no-evidence`. That means the corpus is more truthful than before, but also means not every historical story should be spoken about as if it passed today's strongest gate. This is intentional, not a defect.

---

## Decisions

### 2026-07-26 — Israel now lands on the newest story run, hides the dead mode switch, and stops stretching dispatch plates full-width
**What:** Corrected the Israel tab behavior in `viewer/index.html` on three linked fronts. First, the default dispatch landing path now derives its latest run from actual dated story entries instead of trusting the first raw rows in `israel/index.json`, which currently begin with `STATUS.md` and `INTENT.md`. Second, the live `Curated / Latest` switch is now hidden. Third, the dispatch story plates no longer stretch to full-row width; their `.lvl-summary` bars now size to their own wrapped headline content instead of filling the entire content column.
**Why:** The old behavior was wrong both conceptually and visually. Israel should open on the newest reporting run, not on internal project documents, and the full-width dispatch plates read like accidental bars rather than title-led section plates. The `Curated / Latest` switch had also become dead weight because it was no longer a legible or useful distinction in the live experience.
**How to apply:** Dispatch-style subjects should derive their default landing from dated story records only; operational docs can stay in archive/search without hijacking the subject's first view. For dispatch collapsible plates, use per-headline fit rather than either a cross-page shared width or a forced full-row width. If a mode switch is not meaningfully legible in the live product, hide or remove it instead of preserving it as speculative architecture.

### 2026-07-25 — Israel should be a briefing surface, not a news-product skin
**What:** Israel Tracker's role inside Master Reader is now clearer from the chat history: it is a briefing and dispatch surface, not a general-purpose news app, and not a loud breaking-alert experience. Tim has reinforced a preference for composed hierarchy: strong masthead, calm at-a-glance ordering, restrained segmented mode switch (`Curated / Latest`), and source-forward access through the fountain exception and archive shelf rather than through noisy feed chrome.
**Why:** The dispatch tabs are supposed to feel like authored records and briefing documents, not algorithmic streams. That matches the larger Master Reader preference for editorial gravity over dashboard or media-site theatrics.
**How to apply:** Future Israel tracker additions should preserve the dispatch tone: quiet urgency if needed, but never sensational UI. Structural differences from Tech are allowed when the reporting mode truly differs, but they should still feel like two briefing dialects inside the same authored publication.

### 2026-07-25 — Segmented mode switches are acceptable when they represent a real reading-mode choice
**What:** The `Curated / Latest` control captures a broader preference signal from this chat. Tim is comfortable with paired segmented controls when the interface is genuinely asking which of two views of the same surface he wants, and when the states are visually obvious and semantically exclusive.
**Why:** One concern running through the audit was preventing generic button drift. Israel's switch survived standardization pressure because it has real mode-switch semantics, not just two adjacent actions.
**How to apply:** If another briefing or archive surface later needs a real mode toggle, this is the precedent: use a segmented family only when the states are paired, mutually exclusive, and legible at a glance. Do not use the segmented style just because two buttons happen to sit beside each other.

### 2026-07-14 — Canonical routine restored; strict gate applies to current stories, not silently to legacy ones
**What:** Settled the Israel drift in five linked moves. (1) `routines/Israel Tracker Routine Instructions.md` is now the canonical live routine/spec reference, while `.claude/commands/israel-add.md` was reduced to a deprecation stub. (2) The verifier now performs stricter claim-to-quote number binding, so a claim can no longer pass just because a loose numeric substring appeared somewhere in the cached article. (3) The corpus was labeled honestly: nine stories as `legacy-cache-migrated`, one as `legacy-no-evidence`. (4) The monthly audit path now has a launcher-ready routine file: `routines/Archive Audit Routine Instructions.md`. (5) The viewer now consumes `_frontpage.json` via a Curated/Latest toggle, promoted-story label, and slow-cycle banner.
**Why:** The old state mixed a real v3 implementation with stale v2 entry points and legacy stories that looked stricter than they really were. That was drift, not safety. The fix is to make the current gate stronger and the old corpus more honestly labeled.
**How to apply:** New Israel stories must satisfy the strict current gate. Legacy labels are continuity markers only; do not use them to excuse weaker new work. Use the archive-audit routine monthly to keep link health, legacy labels, and verifier worklists current.

### 2026-07-08 — Middle Path cache policy: tracked receipts, local bodies
**What:** Resolved the cache-exposure decision with the Middle Path. `israel/.cache/<slug>.fetch.txt` remains tracked but now contains metadata and a SHA-256 body fingerprint only. Full article text lives in gitignored `<slug>.fetch.body.txt` siblings. `scripts/fetch-article.py` creates this split directly; `scripts/strip-cache-bodies.py` migrated the nine legacy combined caches; `scripts/verify-story.py` requires the local body and confirms that its fingerprint matches the receipt before checking evidence.
**Why:** The archive needs durable proof of what was fetched, but the public repository should not republish complete news articles. A fingerprint detects body changes without exposing the text.
**Archive Audit consequence:** Full evidence re-verification works on this machine because it retains the ignored bodies. A fresh clone receives receipts but not bodies, so it can inspect provenance but cannot re-run quote verification until the articles are fetched into local body files.

### 2026-07-08 — Script-owned fetch shipped; verifier generalized; routine v3.0 local
**What:** Implemented the 2026-07-07 architecture decision locally. Added `scripts/fetch-article.py` to fetch article bytes with an honest user agent, 20s timeout, redirect-following, charset detection, headline/body sanity check, and `FETCH-RECEIPT v1` cache output including `sha256`. Generalized the verifier as `scripts/verify-story.py --subject <folder>` with a shared `normalize()` implementation; `scripts/verify-israel-story.py` remains as a thin Israel compatibility wrapper. Updated local-only `routines/Israel Tracker Routine Instructions.md` to v3.0: Step 5a now calls the fetcher, Step 5.5b is risk-tiered, and actor context reuse is controlled by `context_text`/`verified_on` in `_actors.json`.
**Why:** The cache is now created by code rather than model prose, so the expensive re-fetch review no longer has to prove cache-to-web consistency on every low-risk story. Deterministic verification still blocks fabrication; fresh-eyes review is reserved for casualty/significance risk and deterministic low-risk sampling.
**How to apply:** Use `python3 scripts/fetch-article.py --subject israel --slug <slug> --url "<url>" --headline "<headline>"` before writing a story, then verify with `python3 scripts/verify-story.py --subject israel --quarantine ...`. The old Israel verifier command still works.

### 2026-07-07 — Script-owned fetch + risk-tiered second pass (architecture review)
**What:** Two changes to the verification architecture, decided in a cross-routine review session. (1) **Move the fetch out of the model's hands**: replace "model WebFetches → model Writes the cache" with a script (`scripts/fetch-article.py`: takes a URL, fetches it, saves the body verbatim to `israel/.cache/`, records hash + HTTP status + timestamp in the receipt header). The cache becomes trustworthy *by construction* — currently the model writes its own cache, so the deterministic verifier only proves story↔cache consistency, and the expensive subagent re-fetch exists solely to prove cache↔web consistency. (2) With that hole closed, **tier the LLM second-pass review**: full semantic review (real-quote-supporting-wrong-claim check) stays mandatory for casualty figures and sig≥3 stories; sampled (~1-in-3) for sig 1–2 rhetoric items, quarantine-on-hit as before. Terminology adopted for this class of system: *evidence-grounded pipeline*, *deterministic verifier / extractive attribution*, *LLM-as-judge*, *risk-tiered verification budget*.
**Why:** Token spend is the binding constraint (Tim, 2026-07-07). The subagent re-fetch pass is the biggest token line per run and exists only to patch the model-writes-its-own-cache hole. Script-fetch removes the hole structurally — cheaper AND a stronger fabrication guarantee. Fabrication stays 100% blocked deterministically; only the subtler misrepresentation check is sampled on low-stakes stories.
**How to apply:** Build `scripts/fetch-article.py`; rewrite routine Step 5a to call it instead of WebFetch+Write; simplify the fetch-success test (script reports honest status codes); rewrite Step 5.5b with the tiering rule. Bump routine to v3.0 and reflect in spec §4.1–4.2. Also: add `context_text` + `verified_on` to `_actors.json` and reuse context entries verbatim if <90 days old — cuts per-story WebSearch verification calls.

### 2026-06-17 — Deterministic anti-fabrication gate added to the routine
The routine's only protection against the model inventing facts was self-reported (a "fetch receipt" and a pre-write self-check the model fills in itself — a hallucinating model can hallucinate the receipt too). Added a layer the model does not control: (1) every fetched article's raw body is saved verbatim to `israel/.cache/<slug>.fetch.txt`; (2) every load-bearing claim in a story carries an `evidence` entry with a verbatim `quote` from that article; (3) `scripts/verify-israel-story.py` deterministically confirms each quote is a literal substring of the saved body (after Unicode/whitespace normalization) and that URLs match, quarantining any failure to `israel/.quarantine/` before indexing or commit; (4) an independent second-pass review (subagent that re-fetches the URL, never saw the drafting) catches real-quote-supporting-wrong-claim cases the string check can't. The fetch-success test was also hardened to reject 200-status block/404 pages (status + body length + headline-words-in-body). Verifier tested against a real Al Jazeera article: real quotes (including a line with invisible word-joiners) passed; a fabricated "240 children killed" quote failed and exited non-zero. `.cache/` and `.quarantine/` are gitignored. Schema documented in spec §4.1–4.2.
**Why:** Tim's hard requirement — fabrication is 100% unacceptable. An instruction cannot make an LLM incapable of lying; only an out-of-band check on saved evidence can. This shifts the guarantee from "trust the model" to "a script proves each claim traces to fetched bytes, or the story is rejected."
**How to apply:** Any future change to story generation must preserve the chain: fetch → save raw → anchor claims in `evidence` → script verifies → quarantine on fail. Never "fix" a verifier failure by editing the story to match the check; the failing claim is wrong, not the check.

### 2026-04-30 — Tracker conceived; replaces the unused `news` subject
Israel/Palestine/Mideast tracker defined: violence, legal machinations, key-figure rhetoric, international policy. One `.md` per story, `YYYY-MM-DD-slug.md`. Folder renamed `news/` → `israel/` via `git mv` to preserve history; `news/README.md` deleted, new `israel/README.md` written. The current live design/operation reference is `../routines/Israel Tracker Routine Instructions.md`.
**Why:** the `news` subject was never used, and Tim wanted a disciplined long-view record of this specific conflict rather than generic headlines.

### 2026-05-02 — Dispatch viewer style shipped; first stories live
New `dispatch` style class added to the viewer (commit `1f55092`): italic DM Serif Display masthead, small-caps geography dateline, oxblood accent, flat one-story-per-page layout meant to read like a correspondent's filing log. Follow-up commits added run_date grouping, outlet datelines, the sources modal, and button-layout fixes (`7a786a9`).
**Why:** Israel pieces needed a register distinct from the literary briefs and parchment FI styles — terser, "from the field."

### 2026-05-02 — Source-faithful summary + encyclopedic context split
Background lives *inside* each story's frontmatter (`context` array, typed entries) so files stay portable, but renders as a separate "Context" modal in the viewer rather than inline in the body. Summary reflects source framing without false balance; context is source-neutral and encyclopedic.
**Why:** keeps each file self-contained and the reading experience clean, while preserving an honest separation between reporting and background.

### 2026-05-02 — Four-axis significance scoring committed
Every story scored 1–5 via a four-axis pre-pass (scale, actor weight, structural impact, persistence), with thread-coupling (+1 for materially advancing a sig-4+ thread), a rhetoric gate (rhetoric exceeds Tier 3 only if actor weight = 2 and impact ≥ 1), a tightened bias rule, and a final score-criticism prompt. Axes and `significance_reason` persisted in frontmatter for audit and drift detection. Full procedure in spec §5.5.
**Why:** significance needs to be reproducible and reviewable months later, not a one-off gut call — and continuation of a major arc shouldn't be under-rated as "incremental."

### 2026-05-13 — Routine rewritten to run entirely through the GitHub API
`israel-add` reworked to do all repo I/O via the GitHub Contents and Git Trees APIs over `curl` — no local filesystem, no local git, single clean batch commit per run with correct author identity and no `Co-Authored-By` trailers. Requires a classic PAT with `repo` scope (`GITHUB_TOKEN`). The live local routine now sits at `../routines/Israel Tracker Routine Instructions.md`; `.claude/commands/israel-add.md` remains only as a deprecation stub.
**Why:** to make the routine runnable from any machine/session without a local checkout. (The same run surfaced the cloud-fetch blocker above.)

### 2026-06-10 → 2026-06-13 — Viewer hardening: per-story context, stacked sources, fetch-integrity gate
Per-story context buttons, stacked source links, and a fetch-integrity gate (no story without a verified article fetch) added; assorted viewer UI fixes (commit `41e9d58`). Then 3 unverified stories removed and the index/threads/actors/frontpage registries cleaned up (`ed5da54`).
**Why:** integrity over volume — the corpus must not contain stories that couldn't be verified against a fetched source.

### 2026-06-15 — README deploy platform corrected (Netlify → Vercel); stale index summary trimmed
Doc/data cleanup (commit `39f4efa`). Note for dispatch-style subjects: the viewer trusts raw `index.json` order for archive grouping, so keep `israel/index.json` roughly newest-first even though `parchment`/`lvls` styles sort on load (see MR STATUS, 2026-06-15 sort-race fix).
**Why:** keep the docs accurate and the dispatch archive ordering reliable.

### Pre-existing / architecture (captured for completeness)
- **Registry-driven, no code change to add a source.** Append an object to `sources.json` (with `scan_method`, `trust_tier`, `topic_relevance`) and the routine picks it up next run.
- **Apple Note as queue.** Like Lvls, the routine can read pending topics from a named Apple Note; the queue line is removed only after the commit lands.
- **`index.json` is regenerated** by `scripts/rebuild-indexes.py`, which surfaces `geography`, `actors`, `categories`, `thread_id`, `significance` (but not the large `context` array) into the index.
- **Canonical reference is the live routine file.** If a stale stub or note disagrees with `routines/Israel Tracker Routine Instructions.md`, the routine file is correct.

---

## Ideas / open questions (not yet acted on)

### ~~⚠ `israel/.cache/` is NOT actually gitignored — raw article bodies are committed and deployed (found 2026-07-07)~~
Resolved 2026-07-08 with the Middle Path decision above: tracked metadata/hash receipts, gitignored local article bodies. Historical commits still contain the former combined cache files; removing those requires a separate history-rewrite decision.

### Archive Audit tooling shipped (2026-07-07)
`routines/other/archive-audit-spec.md` + `scripts/audit-archive.py` remain the detailed design/script pair, and `routines/Archive Audit Routine Instructions.md` is now the launcher-ready operational file. The old 2026-05-02 story has been explicitly labeled `audit.evidence_status: legacy-no-evidence` rather than left as an implied mystery fail.

### ~~Wire the frontpage logic into the viewer~~ — DONE 2026-07-14
Curated/Latest toggle, promoted-story label, and slow-cycle banner are now wired to `_frontpage.json`. The promotion bookkeeping is no longer invisible to the reader.

### Cloud-fetch reliability (downgraded from blocker)
Cloud fetch works as of 2026-06-17 and the fabrication gate now makes an intermittent failure safe (a failed fetch produces no story, never a fabricated one). Remaining question is only about *coverage continuity* for unattended scheduling: if cloud fetch regresses mid-run, the run just produces fewer stories. A fetch proxy (path B) would harden this; not urgent now.
**Status:** no longer blocking. Revisit only if scheduling unattended cloud runs.

### Filter view (by actor / geography / category / thread / significance)
Becomes necessary once there are ~50+ stories across ~10+ threads. The index already carries the fields. Deferred to v2.

### Related-threads cross-references
Schema reserves `related_threads`; viewer doesn't render it yet. Wire when the data accumulates enough to be valuable.

### 2026-06-17 — Added establishment sources at tier 3 (Times of Israel, Reuters)
Per Tim: fine to work in establishment/legacy media, but high distrust of it — "never want to get sucked up into their lies," which is exactly why per-article citation is non-negotiable. Added Times of Israel (Israeli mainstream) and Reuters (neutral wire) at **trust_tier 3**: discovery net + framing benchmark only, never cited as standalone truth, always corroborated against a tier-1/tier-2 source for kinetic/casualty claims, and always citing the specific article so the reader judges the outlet themselves. Their framing is logged *as their framing*, attributed — never adopted as neutral. (Reuters scans via a Google-News site-filtered feed because reuters.com was blocked in earlier runs; resolve to the real article URL.)
**Why:** closes the coverage blind spot (Israeli-domestic and wire stories the independent sources miss) and enables framing-divergence tracking, without compromising the tracker's vantage or its citation discipline.

### Source trust-tier weighting
All tier-1 sources currently treated equally. With the list now at 9, consider requiring multiple sources for kinetic-event claims while allowing a single source for rhetoric — and formalizing that tier-3 establishment sources can never be the sole citation for a factual claim.

### Significance re-rating audit (every ~30 days)
Re-walk recent files and consider bumping sig-2s to sig-3 with hindsight, or relaxing sig-4s whose predicted follow-up never materialized. Separate manual routine for now.

### Archival pruning
Threads with no entries in 6+ months: hide from the default Archive view but keep the data. Deferred.

### Promotion-fatigue check
If one thread keeps getting promoted on slow days, that's a signal the thread itself has become the active arc — consider elevating it to a standing daily item instead of a relaxation pick.

### Sentiment / framing divergence logging
Log how the same story is framed across trusted vs. mainstream sources, to track divergence over time. Out of v1 scope.

### ~~Routine file location~~ — settled 2026-07-14
Canonical Israel routine now lives at `routines/Israel Tracker Routine Instructions.md`. The old slash-command file is a stub only.

---

## Recently moved / superseded

*(Empty — nothing yet.)*

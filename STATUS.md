# Charles Hub — Status

Companion status file. Records major structural decisions and progress.

## 2026-06-28 — Initial fork from Master Reader

**Goal:** Stand up a clean two-tab build (Tech News + Israel Tracker) for a friend, keeping the Master Reader plumbing for those two styles and removing everything else. To be hosted on GitHub (`cubicleaf/Charles-Hub`) and deployed on Vercel.

**What was done**

- Forked `index.html` from Master Reader and stripped it from 6,711 lines / 305 KB to 3,431 lines / 149 KB (~52% smaller).
- Removed, in full (CSS + JS + HTML + event wiring):
  - P&C / Progress-Clarity (notes system, ledger, sweep API calls)
  - Marius (project board, musters, `MARIUS_DATA`, `/marius/` fetches)
  - Systems registry (the floating "cat bubble" + `systems.json` loader)
  - Landing-card tabs: Lexis (`neutral`), FI + WebDev (`parchment`), Lvls (`lvls`)
- **Kept** (shared by Tech + Israel): pill-bar nav, masthead, archive drawer, global search, settings/text-size, read checkmarks, sources modal, context modal, and the `.lvl-section` collapsible engine (used by both Tech stories and Israel stories).
- `subjects.json` trimmed to two entries: `briefs` (literary) and `israel` (dispatch).
- Created folder plumbing: `briefs/` and `israel/`, each with an empty `index.json` (`[]`) so the app loads with a clean "no files yet" empty state until content is wired in.
- Renamed browser `<title>` and default masthead from "Master Reader" to "Charles Hub".

**Key structural decisions / notes**

- `.lvl-section`, `.lvl-summary`, `.lvl-body`, `.lvl-chevron` are the shared collapsible primitive — they look like "Lvls" code but Tech and Israel both depend on them, so they stayed. The `body.style-lvls` *overrides* around them were removed.
- The collapse button keeps its legacy class name `.marius-collapse-btn` because it's the shared `#collapse-btn` element used by both live tabs. Renaming it is cosmetic and was deferred to avoid risk (no in-repo browser to regression-test).
- Verification: extracted both inline `<script>` blocks and ran `node --check` (passes); grepped the final file for orphaned `getElementById` of removed elements and calls to removed functions (zero). The one remaining "landing-card" hit is a harmless code comment.
- Original kept as `index.original.html` in the folder (delete once happy; it is git-ignored).
- Deployment is zero-config static (Vercel "Other" preset).

**Open / next**

- Content (`.md` files + `index.json` rows) to be populated via the routine instructions (Tim).
- Optional later: rename leftover legacy CSS classes (`marius-collapse-btn`) and trim the few inert comments for fully literal "no references." Cosmetic only.

## 2026-06-28 — Added third tab: 1st Commodity (placeholder)

**Goal:** Add a placeholder commodity tab that will carry sources + archive when populated.

**What was done**

- Added a third entry to `subjects.json`: `id: commodity1`, `name: "1st Commodity"`, `displayTitle: "First Commodity"`, `folder: commodity1`, `style: dispatch`.
- Created `commodity1/` with an empty `index.json` (`[]`) — tab loads with the "no files yet" empty state.

**Key decisions**

- Chose `dispatch` style (same engine as Israel): one `.md` per story, grouped by `run_date`, auto-sorted by `significance`, with per-entry Sources + Context panels. Rationale: a commodity tracker logs individual dated, source-backed items — same shape as the Israel tracker. To switch to `literary` (one hand-written doc per day), change the single `"style"` field.
- **No `index.html` change required.** The Sources (fountain) and Archive (shelf) icons are driven by the tab's `style`, not by per-tab markup. Any `dispatch`/`literary` tab gets them automatically: the shelf is always present; the fountain appears once an entry has `sources` in its front-matter.
- Left empty per request, so the fountain stays hidden until the first sourced entry is added.

## 2026-06-28 — Mobile header: two-row layout

**Problem:** On phones (≤480px) the third pill ("1st Commodity") slid *under* the fixed top-right icon cluster (Sources, Archive, Search, Settings) — they were sharing the same top strip. Adding tabs made it worse.

**Decision:** Two-row header, mobile only (Tim's choice). Icons stay in a top strip; the pill nav drops to its own full-width row beneath. Desktop is untouched (it has room).

**What was done (CSS only, additive `@media (max-width:480px)` block at end of `<style>`)**

- `.pillbar::before` paints an opaque `var(--bg)` strip (height 3.4rem) fixed across the top, so content scrolls cleanly under the icons. It renders above page content (inside the pillbar's z-index:60 stacking context) but below the icons (z-index:90).
- `.pillbar` gets `margin-top: 3.4rem` (drops it below the strip at rest) and sticky `top: 3.4rem` (sticks just under the strip on scroll), with tighter `padding: 0.4rem 0 0.6rem`.
- No HTML or JS changed; desktop rules untouched. Verified CSS brace balance (295/295).

**Note:** This is a first pass — exact strip height / pill padding are easy to dial if the spacing feels off on his device.

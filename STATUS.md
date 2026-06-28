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

# Charles Hub — Status

**Last updated:** 2026-07-28

Companion status file. Records major structural decisions and progress.

## 2026-07-28 (later same day) — Fixed fountain-invisible-on-first-load, added Tech chron-nav arrows (Tim caught both live)

**What:** Two follow-ups from live testing of the port below. (1) The sources fountain wasn't appearing on the very first tab load (Tech or Israel), only after navigating a second time (tapping a date). Root cause: the fountain's entrance animation and the one-time session-gated whole-row chrome cascade were both driving the same DOM node using the identical shared timer property name (`__chChromeCascadeTimer`) — on first load only, `playChromeCascade()`'s own cleanup pass (`pills.concat(icons).forEach(clearChromeCascadeNode)`) would cancel the fountain's own pending entrance timer out from under it, since `sources-btn` was included in `getChromeCascadeIcons()`'s candidate list. On every subsequent navigation the one-time cascade had already played and no longer touched the node, so it worked fine — exactly matching what Tim reported. (2) Tim wants the Tech chron-nav (prev/next arrows between daily briefs) after all, not left out — Charles Hub never had it since it was added to Master Reader after the June fork.

Fixed both together, since fixing #2 properly required fixing #1's underlying architecture first: replaced the standalone single-node fountain-toggle helper from the port below with Master Reader's real two-slot tab-action reconciler (`TAB_ACTION_SLOT_MEMBERS`, `tabActionSlotState`, `requestTabActionSlot()`/`reconcileTabActionSlot()`, including the round-4 microtask-batched, order-independent cross-slot delay fix — ported as-is, not re-derived, since it was just verified correct earlier the same day). Slot A (`sources-btn`) and slot B (`tech-chron-nav`) are fully isolated from the generic chrome cascade now: their own `clearTabActionNode()`/`__chTabActionTimer` never overlaps with `clearChromeCascadeNode()`/`__chChromeCascadeTimer`, and `getChromeCascadeIcons()` no longer includes `sources-btn` at all — contextual icons get their own dedicated entrance/exit every time they're shown (including first load), they don't belong in a cascade that's designed to play exactly once per browser tab. Added chron-nav's CSS (`.tech-chron-nav`/`.tech-chron-btn`/`.tech-chron-arrow`, slot B position `right:14.15rem`), HTML container, and `techChronArrow()`/`appendTechChronNav()` JS, wired into `loadFile()` exactly like Master Reader.

**Why:** Both are direct live-testing catches from Tim, not code-review guesses. The shared-timer collision is the same bug class Master Reader's own STATUS.md already documents (2026-07-26: "a shared animation counter, not the slot design itself") — this port re-created it by having a bespoke fountain animation share state with the generic cascade instead of getting its own isolated track.

**How to apply:** Any future contextual top-right icon added to Charles Hub belongs in the tab-action slot system, never in `getChromeCascadeIcons()`'s candidate list — that list is for icons that are always present and only need a one-time first-load entrance. If a shared timer/counter property is ever reused across two independent animation tracks again, that's the bug class to check first.

## 2026-07-30 — Cold open now renders the selected subject immediately and still lands on the newest batch

**What:** Added `renderSubjectLoadingShell(subject)` and call it only on a true cold open (`switchSubject()` before any prior `activeSubject` exists). Charles Hub already loaded the newest content correctly — Tech via `activeIndex[0].file`, dispatch tabs via the newest `run_date` in `activeIndex` — but it still left the static HTML shell (`Charles Hub` + `Loading...`) onscreen until that first async fetch chain finished. The cold-start shell now immediately swaps to the selected subject's own title and standard in-content loading block while the newest file/batch is fetched. Also changed `init()` to `await switchSubject(startId)` so startup treats the first render as real boot work rather than a fire-and-forget side effect.

**Why:** Tim flagged the same failure mode Charles Hub had just after MR: it technically ends up on the latest batch, but the first impression is a generic hanging shell, which makes the app feel blank or broken.

**How to apply:** Keep this helper scoped to cold start only. Normal tab switches should still preserve the previous rendered subject until the next one is ready; calling the loading-shell helper on every navigation would reintroduce the blackout feel the July 28 reveal work was meant to remove.

## 2026-07-28 — Full aesthetic/motion port from Master Reader (the four weeks of design work since the June fork)

**What:** Charles Hub was forked from Master Reader on 2026-06-28, before essentially the entire authored-surface-language pass Master Reader has been through since (bronze/parchment palette refinements, the SVG fixed-corner plate shell system, "Crisp Lift" hover, the "Soft Drop" first-load chrome cascade, close-button unification, whole-view reveal gating). Ported the applicable parts — everything shared-shell (pill bar, fixed top-right icon row, close buttons, section-bar plates) — into `index.html`:
- `.pill` and `.lvl-summary` converted from CSS border/border-radius to the fixed-corner SVG plate shell (`ensureSvgCornerPlate()`/`updateSvgCornerPlatesNow()`/`updateSvgCornerPlates()`, `SVG_PLATE_SELECTORS` scoped to just those two — Master Reader's list also covers P&C/Marius controls that don't exist here). Added the `body.style-dispatch .lvl-summary` headline-wrapping override Charles Hub never had (long Israel/commodity1 headlines were forcing every row to their width).
- Fixed top-right icon row (settings/search/collapse/shelf) and the sources fountain moved from a 150ms micro-shadow hover to "Crisp Lift" (bronze color shift, warm drop-shadow, `translateY(-4px) scale(1.08)`, 400ms), plus the hover+press "whiplash" fix (`:hover:active` combined-state rule) so pressing while hovering doesn't snap through below-rest size.
- `.modal-close-btn` unified with `.sources-modal-close`/`.context-modal-close` onto one 44px X-icon close family — those two used to be bordered text "Close" pills with their own separate 150ms hover; swapped their HTML markup to the same stroked-X SVG Master Reader uses everywhere else.
- Added whole-view reveal gating (`beginViewReveal()`/`finishViewReveal()`, `body.mr-reveal-pending`/`.mr-reveal-ready`) so a tab/file switch no longer blacks out to a "Loading…" placeholder the instant it's clicked — old content stays visible during the fetch, and the finished surface fades in once, right before real content replaces it. Replaces the header's old one-time `animation: enter` keyframe.
- Added the session-gated "Soft Drop" first-load chrome cascade (`CHROME_CASCADE_SETTINGS`, `playChromeCascade()`): pill bar cascades in left-to-right, then a beat later the fixed icon row cascades in, once per browser tab.
- Added animated entrance/exit for the sources fountain (`animateFountainToggle()`) instead of an instant `style.display` flip. This is a simplified single-node version of Master Reader's two-slot tab-action reconciler — Charles Hub only has one contextual top-right icon (the fountain, shared by Tech/Israel/commodity1), so there's no second occupant ever competing for that screen position and no cross-slot ordering logic needed.

**Explicitly left out of scope:** anything tied to tabs/subsystems Charles Hub doesn't have — P&C's pen icon and field-note slot, Marius's activity icon and board, chron-nav (prev/next paging between daily briefs, a feature added to Master Reader after this fork — not ported since it's new functionality, not an aesthetic mirror), `syncLvlSummaryWidths()` (a multi-bar width-matching routine only relevant to P&C's ledger). The `:root` palette tokens were already identical between the two files — no color/typography values needed to change, only the structural/motion layer.

**Why:** Tim asked to align Charles Hub with everything changed in Master Reader's UI since the fork — tab buttons, cascade entrance, hover states, top-right icon entrance/exit, SVG plates, buttons everywhere.

**How to apply:** Charles Hub's shared-shell elements (pill, lvl-summary, the fixed icon row, close buttons) should now be treated as the same authored surface language as Master Reader's — future Master Reader chrome changes to those same elements should be mirrored here the same way, since Charles Hub's underlying markup/class names are still close enough to make that a direct port rather than a reinterpretation.

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


## 2026-06-28 — Mobile: align the four corner icons

**Problem:** On the two-row mobile header, Sources (fountain) + Archive (shelf) sat ~5px lower than Search + Gear and were unevenly spaced. Cause: the original per-icon mobile media rules only lifted Search/Gear to `top:0.6rem` and used right-offsets spaced for the (now mobile-hidden) collapse icon.

**Fix (CSS, in the mobile `@media (max-width:480px)` block):** Pinned all four to `top:0.6rem` with an even 2.8rem horizontal step — gear 0.6rem, search 3.4rem, archive 6.2rem, sources 9.0rem (right-anchored). They now read as one clean inline row. Brace balance verified.

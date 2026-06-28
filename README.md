# Charles Hub

A clean, two-tab reader forked from Master Reader. It has exactly two tabs:

- **Tech** (`Recent Tech News`) — `literary` style. Long-form daily brief; each numbered story becomes a collapsible section.
- **Israel** (`Israel Tracker`) — `dispatch` style. Date-digest view; multiple stories per day, sorted by significance, with a per-story Context panel and a Sources panel.

Everything else from Master Reader (P&C, Marius, Lexis, FI, Lvls, WebDev, the Systems registry) has been removed. The shared shell stays: pill-bar nav, archive drawer, global search (Cmd/Ctrl+K), text-size settings, read checkmarks, sources/context modals.

## How it works

`index.html` is a single self-contained file (HTML + CSS + JS, no build step). On load it:

1. Fetches `/subjects.json` — this is what defines the tabs.
2. For the active tab, fetches `/<folder>/index.json` — the list of entries.
3. Fetches the individual `.md` files referenced in that index and renders them.

So to add a tab you edit `subjects.json`; to add content you drop a `.md` file in the folder and add a line to that folder's `index.json`.

## Folder layout

```
index.html          ← the app (single file)
subjects.json       ← defines the two tabs
briefs/             ← Tech tab content
  index.json        ← list of brief entries (newest first)
  YYYY-MM-DD.md     ← one brief per day
israel/             ← Israel tab content
  index.json        ← list of story entries (newest first)
  *.md              ← one file per story
```

## Content format

**Tech brief** — `briefs/index.json` entries (keep newest first):

```json
[
  { "file": "2026-06-28.md", "date": "2026-06-28", "day": "Sunday", "title": "Sunday Brief" }
]
```

`briefs/2026-06-28.md`:

```markdown
---
title: "Sunday Brief"
date: 2026-06-28
day: Sunday
sources:
  - { title: "Example Outlet", url: "https://example.com/article" }
---

# Sunday Brief

## At a glance

1. One-line summary of story one.

## 1. First story headline

**Outlet · 2026-06-28**

Body text of the story.

*Why it matters: one sentence.*
```

The first `## ` heading is treated as the "At a glance" list; each following `## N. …` heading becomes a collapsible story.

**Israel story** — `israel/index.json` entries (newest first; one row per story, grouped by `run_date`):

```json
[
  { "file": "2026-06-28-1.md", "run_date": "2026-06-28", "date": "2026-06-28", "title": "Headline", "summary": "One-line glance summary." }
]
```

`israel/2026-06-28-1.md`:

```markdown
---
title: "Headline"
date: 2026-06-28
significance: 90
significance_reason: "Why this story leads today."
sources:
  - { outlet: "Reuters", url: "https://...", date: "2026-06-28" }
context:
  - { name: "Background term", note: "Encyclopedic context shown in the Context panel." }
---

Body markdown for the story.
```

Stories sharing a `run_date` render together as that day's digest, sorted by `significance` (high to low).

## Deploy (Vercel)

It's a static site — no build command, no framework. Push to GitHub, import the repo in Vercel, accept the defaults ("Other" preset, empty build command, output = repo root). The app fetches everything at runtime from the deployed root.

## Local note

`subjects.local.json` is git-ignored — if present, its subjects are appended on top of `subjects.json` for local-only experimentation. Not required.

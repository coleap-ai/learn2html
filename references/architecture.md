# Architecture: app shell, views, and the LearnDB data layer

> All UI labels below are given in English to specify *meaning*. Render them in the tutorial's output language (see the ground rules in `SKILL.md`).

## App shell layout

```
┌──────────────────────────────────────────────────────────┐
│ Header: course title · overall progress bar · view tabs  │
├────────────┬─────────────────────────────────────────────┤
│ Sidebar    │ Main area (one view visible at a time)      │
│ M1 ✓ 100%  │  [Learn] [Practice] [Quiz] [Report]         │
│ M2 ● 40%   │                                             │
│ M3 ○       │                                             │
└────────────┴─────────────────────────────────────────────┘
```

- View switching is pure JS show/hide (`data-view` attribute on `<main>`); no routing library. Persist the active view + module in `state` so a reload restores position.
- Sidebar shows per-module status glyphs: ○ untouched, ● in progress (with %), ✓ complete, and a 🏆 marker when the module quiz reached the pass line.
- Header progress bar = weighted average: 60% section completion + 40% best quiz score (or 100% sections if the assessment level is "reading only").

## Design tokens

Define all colors/spacing as CSS custom properties on `:root` in one block at the top, derived from the chosen visual style. Support both light and dark by defining the tokens once and, for the "clean professional" style, adding a `@media (prefers-color-scheme: dark)` override. Never hardcode colors inside widgets — widgets consume tokens so the style choice is a one-block change.

## LearnDB — localStorage data layer

One namespace per tutorial: key prefix `learn2html:<topic-slug>:`. Three stores, each a single JSON value under its own key. Implement as a small wrapper object:

```js
const LearnDB = {
  ns: 'learn2html:<topic-slug>:',
  read(store, fallback) { try { return JSON.parse(localStorage.getItem(this.ns + store)) ?? fallback; } catch { return fallback; } },
  write(store, value) { localStorage.setItem(this.ns + store, JSON.stringify(value)); },
  reset() { ['state','results','reports'].forEach(s => localStorage.removeItem(this.ns + s)); }
};
```

All writes go through `LearnDB.write` immediately on the triggering event (no batching needed at this scale). Every stored object carries `version: 1` so a future regeneration can migrate or discard old data.

### Store 1: `state` — where the learner is

```json
{
  "version": 1,
  "topic": "<topic title>",
  "startedAt": "ISO-8601",
  "lastVisit": "ISO-8601",
  "activeView": "learn",
  "currentModule": "m2",
  "completedSections": { "m1": ["s1","s2","s3"], "m2": ["s1"] },
  "timeSpentSec": { "m1": 420, "m2": 130 },
  "interactions": { "m1-widget-cpu": 12, "m2-dragmatch": 3 }
}
```

- `timeSpentSec`: accumulate with a 5-second `setInterval` that only counts while `document.visibilityState === 'visible'`; flush on `visibilitychange` and `beforeunload`.
- `interactions`: increment a counter per widget on meaningful interaction (drag completed, step advanced) — feeds the report's engagement section.

### Store 2: `results` — every quiz attempt, immutable append-only

```json
{
  "version": 1,
  "attempts": [
    {
      "attemptId": "a-<seq>",
      "moduleId": "m1",          // or "final" for the final exam
      "startedAt": "ISO-8601",
      "durationSec": 95,
      "score": 4, "total": 5, "passed": true,
      "perQuestion": [
        { "qid": "m1-q1", "type": "single", "answer": "B", "correct": true, "timeSec": 12 }
      ]
    }
  ]
}
```

- Never mutate past attempts; retakes append. "Best score" and "latest score" are derived at read time.
- Pass line (standard assessment): 60% per module quiz, 70% for the final; state it in the quiz UI.

### Store 3: `reports` — generated snapshots

```json
{
  "version": 1,
  "reports": [
    {
      "reportId": "r-<seq>",
      "generatedAt": "ISO-8601",
      "overallProgressPct": 72,
      "totalTimeSec": 3300,
      "modules": [
        { "moduleId": "m1", "sectionPct": 100, "bestScorePct": 80, "attempts": 2, "mastery": "mastered" }
      ],
      "weakPoints": [ { "moduleId": "m2", "qids": ["m2-q3"], "hint": "<concept name>" } ],
      "recommendations": [ "<localized prose, e.g. 'Review the <concept> section of module 2 and retake the quiz'>" ]
    }
  ]
}
```

- `mastery` bands are stored as stable machine keys — `mastered` (≥80%), `partial` (60–79%), `needs-work` (<60%, or no attempt while sections are done), `not-started` — and mapped to localized labels only at render time, so exported JSON stays language-independent.
- `hint` and `recommendations` are generated prose and therefore written in the tutorial's output language.
- Report generation is a pure function of `state` + `results`; the snapshot exists so learners can compare their previous report against the current one.

## Report view rendering

- Charts are hand-rendered inline SVG (no library): horizontal bar chart for per-module mastery, polyline for score-over-attempts, and a donut or radial for overall progress. Compute coordinates in JS, build with `document.createElementNS`.
- Weak-point list items are links: clicking jumps to Learn view scrolled to the relevant section.
- JSON export: `Blob` + `URL.createObjectURL` + programmatic `<a download>` containing all three stores.
- Print: a `@media print` stylesheet that hides sidebar/tabs and shows the latest report as a clean one-page document.
- Reset learning data: custom in-page modal (never `confirm()`) → `LearnDB.reset()` → reload UI state.

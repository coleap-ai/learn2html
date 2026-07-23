# SVG interaction patterns & quiz engine

> All UI labels below are given in English to specify *meaning*. Render them in the tutorial's output language (see the ground rules in `SKILL.md`).

Every module must teach its core concept through at least one TRUE interaction — the learner manipulates something and sees a consequence. A static SVG with hover tooltips alone does not qualify. Pick the pattern whose mechanics mirror the concept's mechanics.

## Widget patterns (pick per concept type)

### 1. Annotated explorer — for "structure/anatomy" concepts
SVG diagram of the thing (a CPU, an HTTP request, a cell, a balance sheet). Parts are clickable; clicking highlights the part, dims the rest, and shows its explanation in a side panel. Track visited parts; show an "explored 5/8" counter and mark the widget complete when all parts have been visited.

### 2. Step-through animation — for "process/flow" concepts
A sequence (algorithm iterations, TCP handshake, photosynthesis stages) with ◀ ▶ step buttons and an auto-play toggle. Each step moves/recolors SVG elements via CSS class transitions and updates a caption explaining *why* this step happens. Never animate on a timer alone — the learner must be able to step manually.

### 3. Parameter sandbox — for "relationship/tradeoff" concepts
Sliders/toggles drive a live SVG visualization (interest rate → compound growth curve, aperture → depth of field, learning rate → gradient descent path). Recompute and re-render on every `input` event. Add 2–3 preset buttons ("typical scenarios") that set parameter combos worth understanding, each with a one-line takeaway.

### 4. Drag-and-drop matching / ordering — for "classification/sequence" practice
Draggable SVG/HTML chips onto labeled drop zones (match term→definition, order the steps of a procedure). Use Pointer Events (`pointerdown/move/up` + `setPointerCapture`) so it works with mouse and touch. Wrong drops bounce back with a shake animation and a hint; completed boards lock with a success state.

### 5. Build-it simulator — for "construction/composition" concepts
The learner assembles something from parts (compose an SQL query from clause blocks, wire a circuit, build a sentence) and a live preview shows the result of the current assembly, including meaningful error states for invalid combinations — the error states are where the teaching happens.

### 6. Hotspot quiz-in-diagram — for spot-check practice
"Click the spot in the diagram where the error occurs" / "click where X happens" on an SVG scene. Instant right/wrong feedback with explanation. Good as a Practice-view item.

## Implementation rules for all widgets

- Namespaced ids: prefix all element ids with the module id (`m2-sandbox-slider`) — one file hosts many widgets.
- Each widget is a self-registering function `initWidget_<id>(container)` called on first view of its module (lazy init keeps load instant).
- Report every meaningful interaction to `LearnDB` (`interactions` counter) and mark widget completion into `completedSections`.
- SVG accessibility: `role="img"` + `<title>`; interactive parts are `<g tabindex="0" role="button" aria-label="...">` with `Enter`/`Space` handlers mirroring click.
- Respect `prefers-reduced-motion`: gate CSS transitions/auto-play behind the media query; manual stepping always works.
- All geometry in a `viewBox` coordinate system; never pixel positions — widgets must survive responsive resizing.

## Quiz engine

Question bank as a plain JS array; each question:

```js
{ qid: 'm1-q1', moduleId: 'm1', type: 'single'|'multi'|'truefalse'|'order'|'match',
  stem: '…', options: ['A…','B…'], answer: 'B'|['A','C']|[2,0,1],
  explain: 'why …',           // shown after answering, ALWAYS teaches, even on correct
  sectionRef: 'm1-s2' }       // where to review; powers report weak-point links
```

Question text (`stem`, `options`, `explain`) is authored in the tutorial's output language.

Engine behavior:
- One question at a time with a progress indicator ("question 3/5"); confirm-then-feedback flow: select → submit → instant right/wrong + `explain` → next question. No changing answers after submit.
- Shuffle option order at render (Fisher–Yates keyed off attempt seq — no `Math.random` needed if unavailable: use a simple LCG seeded by attempt count) while mapping back to canonical answer ids for scoring.
- On finish: score screen (score, pass/fail vs the pass line, per-question review list), then append the attempt to `results` via LearnDB. Retakes always allowed; best score counts toward progress.
- Final exam (standard assessment level only): samples questions across all modules + a few held-out final-only questions; unlocks when every module quiz has been attempted.

## Practice vs Quiz

Practice-view items reuse patterns 4/5/6 with unlimited retries, hints, and no recording into `results` (only `interactions`). Quiz-view is recorded, no hints. Keep this boundary strict so the report reflects assessed ability, not practiced clicks.

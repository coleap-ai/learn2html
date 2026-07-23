---
name: learn2html
description: Turn a learning topic into a self-contained, richly interactive HTML+SVG training tutorial. Interviews the user for basic requirements first, then independently plans the curriculum and builds a single-file courseware app with Learn / Practice / Quiz / Report views and a localStorage data layer (state, results, reports). Use when the user asks to create an interactive tutorial, training courseware, or a learning/teaching HTML page for any topic, in any language.
argument-hint: <learning topic>
---

# learn2html — Interactive HTML+SVG Training Tutorial Generator

Given a learning topic, produce ONE self-contained HTML file that teaches the topic through rich SVG-driven interactions, tracks the learner in a localStorage "database", and can generate a learning report.

## Ground rules

- **Output language**: the tutorial UI and content use the language the user made the request in (a Chinese request produces a Chinese tutorial, a Spanish request a Spanish one), unless they say otherwise.
- **Never hardcode the strings in this skill**: every user-facing string named here — interview options, view tab labels, buttons, mastery bands, quiz chrome, report headings — is written in *English in this document only as a specification of meaning*. Render each one in the output language. This skill is documented in English; the tutorials it produces are not.
- **Self-contained**: one `index.html` with inline CSS/JS/SVG. Zero external requests (no CDN, no fonts, no images) — it must work offline from `file://`.
- **No native dialogs**: never use `alert/confirm/prompt` — build in-page modals instead (native dialogs block browser automation and look unpolished).
- **Output location**: `./<topic-slug>/index.html` under the current working directory (kebab-case ASCII slug of the topic).

## Phase 1 — Requirements interview (REQUIRED before building)

Use `AskUserQuestion` — one round, up to 4 questions, asked in the user's language. Ask only what changes the design; do not interrogate. Standard set:

1. **Audience level**: complete beginner | beginner with related background | intermediate/advancing | mixed audience — controls vocabulary, prerequisite recaps, and depth.
2. **Scope**: quick tour (3–4 modules, ~30 min) | standard (5–7 modules, ~1–2 h) | deep dive (8+ modules, multi-session) — controls module count and quiz size.
3. **Assessment intensity**: light self-check (3–5 questions per module) | standard assessment (module quizzes + a final exam, with pass lines) | reading only (no quiz; the report tracks reading progress only).
4. **Visual style**: clean professional (light, corporate) | dark tech | bright and playful — pick a sensible recommended default based on the topic.

If the topic itself is ambiguous (e.g. "Python" — language basics? data analysis? automation?), add a scoping question or replace question 4 with it. After answers arrive, do NOT ask again — make all remaining decisions yourself.

## Phase 2 — Curriculum planning (autonomous)

Design the course before writing code. In a short message to the user, state:

- Module list (id, title, one-line learning objective each).
- For each module: which **interactive SVG widget** teaches its core concept (pick from `references/interactions.md` — every module MUST have at least one true interaction, not just text + static image).
- Quiz plan per the chosen assessment level.

Then proceed immediately to building — no approval gate. This phase exists so the file you write is planned, not improvised.

## Phase 3 — Build

Read `references/architecture.md` (app shell, views, LearnDB schema) and `references/interactions.md` (SVG widget patterns, quiz engine) before writing the file. Build order inside the single HTML file:

1. **App shell**: header with course title + overall progress bar; sidebar module navigation with per-module completion state; main area with four views — Learn / Practice / Quiz / Report.
2. **LearnDB data layer**: localStorage wrapper with the three stores (`state`, `results`, `reports`) exactly as specified in the architecture reference. Wire every interaction through it: section completion, widget interaction counters, quiz attempts, time tracking.
3. **Learn view**: per-module sections of concise teaching text interleaved with the planned SVG interactive widgets. Sections mark complete on scroll-into-view + dwell, or via an explicit "mark section complete" button.
4. **Practice view**: hands-on tasks (drag-drop matching, fill-in, sandbox widgets) with instant visual feedback — formative, not scored into the pass line.
5. **Quiz view**: quiz engine per assessment level; records every attempt to `results` with per-question detail.
6. **Report view**: reads `state` + `results`, renders SVG charts (module mastery bars/radar, score-over-attempts line), lists weak points with jump-back links to the relevant module, offers JSON export and a print-friendly report, plus a "reset learning data" custom-modal reset.
7. **Polish**: keyboard navigation, responsive down to ~768px, `prefers-reduced-motion` respected, consistent design tokens from the chosen visual style.

Content quality bar: teach with correct, specific material (real syntax, real formulas, real facts for the topic) — the interactivity supports the content, never replaces it.

## Phase 4 — Verify & deliver

1. Sanity-check the file: `grep` that there are no `http://`/`https://` resource references, no `alert(`/`confirm(`/`prompt(`.
2. If browser tools are available, open the file, click through one full module + one quiz attempt + the report view, and fix anything broken. Otherwise do a careful static read of the JS (quiz scoring, LearnDB read/write paths, view switching).
3. Deliver: send the file with `SendUserFile` (display: render) and summarize in the final message: module list, the interactive widgets included, how data persistence works, and where the file lives.

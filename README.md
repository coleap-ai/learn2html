# learn2html

A [Claude Code](https://claude.com/claude-code) skill that turns any learning topic into a **single self-contained HTML file** — an interactive course with SVG widgets, quizzes, progress tracking, and a generated learning report.

No build step, no dependencies, no network. One `index.html` you can open from `file://`, email, or drop on any static host.

## What it produces

- **Learn** — teaching sections interleaved with interactive SVG widgets (annotated explorers, step-through animations, parameter sandboxes, build-it simulators). Every module teaches its core concept through a real interaction, not a static picture.
- **Practice** — untimed, unscored drills with hints and instant feedback.
- **Quiz** — a quiz engine that records every attempt with per-question detail.
- **Report** — hand-rendered SVG charts of mastery per module and scores over time, weak points that link back to the relevant section, JSON export, and a print-friendly view.

Learner state lives in `localStorage` under a per-topic namespace, in three stores: `state` (position, time spent, interaction counts), `results` (append-only quiz attempts), and `reports` (generated snapshots).

## Install

Clone into your skills directory — user-level (available in every project):

```bash
git clone https://github.com/coleap-ai/learn2html.git ~/.claude/skills/learn2html
```

…or project-level (checked in with a specific repo):

```bash
git clone https://github.com/coleap-ai/learn2html.git .claude/skills/learn2html
```

## Use

Just ask Claude Code, in any language:

```
Build me an interactive tutorial on how TCP congestion control works
```

The skill runs in four phases: a short requirements interview (audience level, scope, assessment intensity, visual style), autonomous curriculum planning, the build, then verification. Output lands in `./<topic-slug>/index.html`.

## Language

The skill is documented in English, but the tutorials it generates are written in **whatever language you asked in** — UI labels, teaching content, quiz questions and all. Every user-facing string in `SKILL.md` and `references/` is an English specification of *meaning*, not a literal string to emit.

## Layout

```
SKILL.md                     the skill itself — phases, ground rules, build order
references/architecture.md   app shell, view structure, LearnDB localStorage schema
references/interactions.md   the six SVG widget patterns and the quiz engine
```

## Contributing

The most useful contributions are new entries in `references/interactions.md` — widget patterns whose *mechanics mirror the mechanics of the concept they teach*. Keep them implementable in plain inline SVG with no library.

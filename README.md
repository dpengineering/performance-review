# Weekly Performance Review

A browser-based tool that streamlines the weekly performance-review submission and
grading process for 12th-grade engineering seniors at DPEngineering. Students fill in
a 12-item rubric self-review, download a self-contained page, and submit it through
Canvas. Mentors then grade that same page **interactively inside Canvas SpeedGrader** —
the score computes live in the grading pane.

Sibling project: [portfolio-generator](https://github.com/dpengineering/portfolio-generator)
— same zero-backend, single-file philosophy.

## How it works

- **Student** fills the rubric (`a`/`m`/`s`/`n` per item, with descriptors shown as
  guidance) plus short reflection notes. A live checklist gates the download.
- **Download** produces one self-contained `.html` file — no external files needed —
  which the student submits to a Canvas assignment.
- **Mentor** opens the submission in SpeedGrader and marks their own `a`/`m`/`s`/`n`.
  The page runs live in the pane: the score and per-category subtotals compute as they
  grade. The mentor copies the **score** into the SpeedGrader points box and pastes the
  generated **comment** into the comment field.

Everything runs in the browser. Nothing is uploaded to us; Canvas is the system of record.

See [`PLAN.md`](PLAN.md) for the full rubric, scoring model, and design.

## Running it locally

Because the tools are self-contained, you can open the HTML directly in a browser
(`file://`) or serve the folder with any static server, e.g.:

```bash
python3 -m http.server 8000
```

> Note: avoid VS Code **Live Server** — its live-reload injection lands inside the
> page's inline script and breaks it. Open the file directly or use a plain static
> server instead.

## Scoring at a glance

`a`/`m`/`s`/`n` map to `3`/`2`/`1`/`0` (reversed for the two counterproductive items),
summed across 12 items (0–36) and normalized to a **0.00–4.00** scale. The mentor's
grade is official; the student's self-score is shown alongside as a reflection mirror.

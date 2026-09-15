# DPEngineering — Weekly Performance Review

A browser-based tool that streamlines the weekly performance-review submission
process for 12th-grade engineering seniors. It replaces the multi-tab
Google Sheet ([source rubric](https://docs.google.com/spreadsheets/d/1erlqm3UoqPVIc0n5YhPv-zpNiUygeZmOLR8fecxYfLY))
with a self-contained web artifact that students submit through Canvas. Mentors
read that submission in SpeedGrader and enter the official grade with **Canvas's
built-in rubric**.

Sibling project: [`portfolio-generator`](../portfolio-generator) — same zero-backend
philosophy, same design-system conventions.

---

## Locked decisions

| Area | Decision |
|------|----------|
| **Architecture** | Pure client-side. Self-contained HTML artifact. No server, no database, no accounts. Hosted on GitHub Pages alongside the portfolio tools. |
| **Canvas** | Manual submission (no Canvas API / LTI). Student uploads the HTML file to a Canvas assignment; the mentor reads it in SpeedGrader. |
| **Grading UX** | The mentor grades with **Canvas's native rubric** (score + comments) — not inside the submitted file. The submission is a **static, read-only page** whose only job is to be *readable* in SpeedGrader's preview. See [Why not grade in the file](#why-not-grade-inside-the-file). |
| **Teacher output** | The official score and comments live in Canvas, entered via the rubric. Canvas is the system of record. |
| **Scoring** | **Per-item weights** ported from the old sheet's F17 (differential weighting kept, cruft cleaned, critical "mostly" cliffs softened). Σ weights normalized so a best week = 4.00, **floored at 0**. Editable `WEIGHTS` block, mirrored in `index.html` + `grade.html`. See [Scoring model](#scoring-model-per-item-weights). |
| **Student self-score** | Recorded and shown as a **mirror** (drives reflection). Displayed in the authoring app and baked read-only into the submission as context for the mentor. Never the official grade. |

### Why not grade inside the file

The original design had the mentor grade *interactively inside the SpeedGrader
preview* — click the rubric in the file, score computes live. Testing on
2026-08-13 suggested this worked, but by 2026-08-18 Canvas was rendering HTML
submission previews in a **sandboxed iframe without `allow-scripts`**: no
JavaScript runs in the pane at all (neither `<script>` nor inline `on*` handlers).
Static HTML/CSS still renders fine. So the submission is now a static page the
mentor *reads*, and grading moves to Canvas's own rubric — which does score
computation and comments natively anyway.

---

## Scale & who does what

- **~100 students** submit one self-review per week (~37 weeks/school year).
- **~8 mentors/teachers** split the grading; Canvas SpeedGrader routes each mentor
  to their own section's submissions.
- Canvas holds identity, per-student/per-week storage, rosters, grader
  assignment, **and now the rubric-based grade**. We build only the
  **student authoring → static submission** layer.

---

## The rubric (12 items, 3 sections)

Numbered 1–12 in display order: Performance Review leads (next to the portfolio
content it assesses), then Productive, then Counterproductive.

**Code key** — Performance Review: `a`=absolutely, `m`=mostly, `s`=somewhat,
`n`=not at all. Productive & Counterproductive: `a`=always, `m`=mostly,
`s`=sometimes, `n`=never.

### Section 1 — Performance Review
1. Appropriate progress toward project completion
   *(makes up missed work as agreed; growth mindset; measurable progress toward goals)*
2. Professional communication with teachers, staff, and students
   *(communicates about absences/tardies; respectful; self-advocacy; timely mentor guidance; reads & responds to DPEngineering emails)*
3. Generates portfolio content on a weekly basis
   *(2+ content items; 1+ visuals; 2+ "56 DELTAs" skills) — ties into `portfolio-generator`*

### Section 2 — Productive Behavior
4. Maintains focus and productivity for the duration of the class period
   *(starts quickly after attendance; works until clean-up; arrives on time; leaves when the bell rings)*
5. Strives to produce high-quality work product
   *(iterative process; machining techniques; SolidWorks techniques; software standards)*
6. Is resourceful and proactive, including when encountering difficulties or setbacks
   *(works another part of the project; seeks mentor advice; asks for support; does research; demonstrates curiosity)*
7. Is aware of needs beyond self and strives to improve the organization
   *(helps other students; helps with DPEA org needs; engages with visitors)*
8. Demonstrates integrity and honesty
9. Keeps a clean work area and stores files appropriately
   *(daily clean-up of machines & work area; proper file storage)*
10. Successfully executes the weekly maintenance task
    *(uses all clean-up time; mentor sign-off; before & after photos)*

### Section 3 — Counterproductive Behavior *(reversed: "never" is best)*
11. Wanders around the facility and/or distracts others
12. Engages in unnecessary web/social media or phone/texting *(adheres to district/campus guidelines)*

---

## Scoring model (per-item weights)

The old sheet (cell F17) used 12 bespoke nested-IF weightings, negative penalties, and a
floor-at-0. An early redesign flattened this to equal weights, but the team decided the
**differential weighting was intentional** (some items matter more), so the current model
ports those weights back — cleaned up — rather than staying flat.

**Per-item weights** (`a/m/s/n`; for items 11–12, "never" is best):

| # | Item | a | m | s | n |
|---|------|----|----|----|----|
| 1 | Appropriate progress | 8 | 2 | −8 | −12 |
| 2 | Professional communication | 8 | 4 | 2 | 0 |
| 3 | Portfolio content | 4 | 2 | −8 | −16 |
| 4 | Maintains focus | 8 | 2 | −8 | −12 |
| 5 | High-quality work | 8 | 4 | 1 | 0 |
| 6 | Resourceful / proactive | 4 | 3 | 2 | 1 |
| 7 | Needs beyond self | 4 | 3 | 2 | 1 |
| 8 | Integrity & honesty | 4 | 1 | −8 | −16 |
| 9 | Clean work area | 8 | 4 | 1 | 0 |
| 10 | Maintenance task | 8 | 4 | 1 | 0 |
| 11 | Wanders / distracts | 0 | 0 | 2 | 4 |
| 12 | Web / social / phone | −12 | −8 | −4 | 8 |

**Cleanup from the original F17 formula:** fixed item 6's "sometimes" (it was `3` in the
formula's gate copy but `2` in the value copy — a duplicated-formula typo → use `2`); broke
item 7's "mostly = sometimes" tie (→ 4/3/2/1); and softened the three "mostly = −4" cliffs
(items 1/4/8) to small positives so "mostly meeting expectations" isn't punished. Critical
items still carry heavy negatives on "sometimes/never."

**Aggregation**

- **Raw** = Σ of the 12 item weights.
- **Overall = max( raw ÷ (maxRaw/4), 0 )** → a **0.00–4.00** scale, **floored at 0** (per-item
  negatives are allowed, but a week never scores below 0).
- `maxRaw` = Σ of each item's best value = **76**, so the divisor is 76 ÷ 4 = **19** and a
  best-possible week = 4.00. The divisor **auto-derives** from the weights, so retuning any
  weight keeps the ceiling at 4.00 with no other change.
- Reference curve: best **4.00** · "mostly" everywhere **1.11** · "sometimes" everywhere **0.00**.
- **Category subtotals** (weighted): Performance `/20`, Productive `/44`, Counterproductive `/12`.

**Where the weights live:** an editable `const WEIGHTS = {…}` block, **duplicated in
`index.html` and `grade.html`** (separate embeds — keep the two identical). This model drives
the student **self-score**, and — prefilled into the grading tool from the submission's code —
the mentor's starting score.

---

## Artifact design

Two pieces, both plain HTML/CSS/JS, no build step (fork the `portfolio-generator`
shell for design-system consistency: CSS-variable palette, system-ui font,
form layout, checklist gating).

### 1. Student authoring app (hosted on GitHub Pages)
- Student attaches their weekly `portfolio-generator` post, adds DELTA skills, and
  picks `a/m/s/n` for each of the 12 items, with the descriptor bullets shown
  inline as guidance, plus short reflection notes.
- Live checklist gates the export until name, week, portfolio, ≥2 DELTAs, and all
  12 items are complete.
- Live self-score readout (mirror).
- Imported portfolio photos are re-compressed (≤1000px / JPEG q0.7) to keep the
  submission light.
- **"Download for Canvas"** produces the self-contained artifact, named
  `<Initials+Last4>_PerfReview_<YYYY-MM-DD>.html` (matches portfolio-generator naming).

### 2. The submitted artifact (self-contained, static HTML)
- Renders the embedded portfolio content + DELTAs.
- Shows the student's self-assessment read-only: each rubric item as a highlighted,
  disabled segmented control (chosen `a/m/s/n` marked), plus a self-score card and
  category subtotals — all pure CSS, no script, so it renders in SpeedGrader's
  sandboxed preview.
- Carries a `type="application/json"` data block so the authoring app can re-open a
  submission ("Load draft or page"). This is data only — never executed.
- No grading controls. The mentor grades with the Canvas rubric.

---

## Teacher / mentor flow (in SpeedGrader)

1. Open the student's submission in SpeedGrader — the static page renders in the pane.
2. Read the portfolio, DELTAs, and the student's self-ratings (highlighted, read-only)
   as context.
3. Enter the official score and comments using **Canvas's built-in rubric**.

No download, no separate grading tool — the rubric lives in Canvas.

---

## Build order

1. **Rubric + scoring locked** ✅ (this document).
2. **Student authoring app** ✅ — form, live descriptors, checklist gating, self-score preview, portfolio import + re-compression, artifact export.
3. **Static submission** ✅ — read-only self-ratings summary + self-score, no embedded grading.
4. **Polish & deploy** — cross-link with the portfolio tools, publish to GitHub Pages, write student/mentor quick-start (incl. how to set up the Canvas rubric).
5. *(Later, optional)* **Trends viewer** — drag-drop a student's weekly files to chart 0–4 self-scores across the year.

---

## Open / deferred

- **Canvas rubric setup** — build the matching 12-criterion rubric in Canvas (or a
  points-only rubric) so mentor scoring maps to the same 0.00–4.00 model; document it.
- **Trends viewer** deferred to a later phase (low priority until the core loop is in use).
- **Critical-item weighting** deferred; v1 ships equal-weight.
- **GitHub Pages setup** when we deploy.

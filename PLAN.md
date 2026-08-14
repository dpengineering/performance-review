# DPEngineering — Weekly Performance Review

A browser-based tool that streamlines the weekly performance-review submission and
grading process for 12th-grade engineering seniors. It replaces the multi-tab
Google Sheet ([source rubric](https://docs.google.com/spreadsheets/d/1erlqm3UoqPVIc0n5YhPv-zpNiUygeZmOLR8fecxYfLY))
with a self-contained web artifact that students submit through Canvas and mentors
grade **inside SpeedGrader**.

Sibling project: [`portfolio-generator`](../portfolio-generator) — same zero-backend
philosophy, same design-system conventions.

---

## Locked decisions

| Area | Decision |
|------|----------|
| **Architecture** | Pure client-side. Self-contained HTML artifact. No server, no database, no accounts. Hosted on GitHub Pages alongside the portfolio tools. |
| **Canvas** | Manual submission (no Canvas API / LTI). Student uploads the HTML file to a Canvas assignment; mentor grades it in SpeedGrader. |
| **Grading UX** | The submitted artifact is **interactive inside SpeedGrader** — mentor clicks the rubric controls in the Canvas preview pane and the score computes live. *(Confirmed by testing: SpeedGrader renders single HTML files AND executes inline JavaScript.)* |
| **Teacher output** | Two copy-ready values: a computed **score** to type into the SpeedGrader points box, and a formatted **comment** to paste into the comment field. Canvas is the system of record. |
| **Scoring** | Redesigned — transparent, equal-weight, no negative points, no hard floor. Normalizes to a 0.00–4.00 scale for continuity with the old sheet. |
| **Student self-score** | Recorded and shown as a **mirror** next to the mentor's grade (drives reflection). Not counted toward the official grade. |

---

## Scale & who does what

- **~100 students** submit one self-review per week (~37 weeks/school year).
- **~8 mentors/teachers** split the grading; Canvas SpeedGrader routes each mentor
  to their own section's submissions.
- Canvas holds identity, per-student/per-week storage, rosters, and grader
  assignment. We build only the **generate → grade → compute** layer.

---

## The rubric (12 items, 3 sections)

**Code key** — Sections 1 & 2: `a`=always, `m`=mostly, `s`=sometimes, `n`=never.
Section 3: `a`=absolutely, `m`=mostly, `s`=somewhat, `n`=not at all.

### Section 1 — Productive Behavior
1. Maintains focus and productivity for the duration of the class period
   *(starts quickly after attendance; works until clean-up; arrives on time; leaves when the bell rings)*
2. Strives to produce high-quality work product
   *(iterative process; machining techniques; SolidWorks techniques; software standards)*
3. Is resourceful and proactive, including when encountering difficulties or setbacks
   *(works another part of the project; seeks mentor advice; asks for support; does research; demonstrates curiosity)*
4. Is aware of needs beyond self and strives to improve the organization
   *(helps other students; helps with DPEA org needs; engages with visitors)*
5. Demonstrates integrity and honesty
6. Keeps a clean work area and stores files appropriately
   *(daily clean-up of machines & work area; proper file storage)*
7. Successfully executes the weekly maintenance task
   *(uses all clean-up time; mentor sign-off; before & after photos)*

### Section 2 — Counterproductive Behavior *(reversed: "never" is best)*
8. Wanders around the facility and/or distracts others
9. Engages in unnecessary web/social media or phone/texting *(adheres to district/campus guidelines)*

### Section 3 — Performance Review
10. Appropriate progress toward project completion
    *(makes up missed work as agreed; growth mindset; measurable progress toward goals)*
11. Professional communication with teachers, staff, and students
    *(communicates about absences/tardies; respectful; self-advocacy; timely mentor guidance; reads & responds to DPEngineering emails)*
12. Generates portfolio content on a weekly basis
    *(2+ content items; 1+ visuals; 2+ "56 DELTAs" skills) — ties into `portfolio-generator`*

---

## Scoring model (redesigned)

The old sheet used 12 bespoke nested-IF weightings, heavy negative penalties, and a
floor-at-0 that let a couple of bad marks collapse an otherwise good week to 0.00.
It also carried a latent inconsistency (item 3's "sometimes" value differed between
the two copies of the formula). The redesign is flat and transparent.

**Per-response points**

| Category | Items | a | m | s | n |
|----------|-------|---|---|---|---|
| Productive | 1–7 | 3 | 2 | 1 | 0 |
| Counterproductive *(reversed)* | 8–9 | 0 | 1 | 2 | 3 |
| Performance | 10–12 | 3 | 2 | 1 | 0 |

**Aggregation**

- Raw sum ranges **0–36** (12 items × max 3). No negative values → no punitive floor needed.
- **Overall = raw ÷ 9** → a clean **0.00–4.00** scale (36 ÷ 9 = 4.00 for a perfect week).
- Also display as **percent** (`raw ÷ 36 × 100`).
- **Category subtotals** shown for actionable feedback: Productive `/21`, Counterproductive `/6`, Performance `/9`.

**Both columns computed the same way.** The mentor's grade is official; the student's
self-score is displayed alongside as a mirror (gap between the two is the reflection signal).

*Future knob (not in v1):* optionally emphasize critical items (integrity #5,
progress #10, portfolio #12) with a transparent ×2 weight.

---

## Artifact design

Two pieces, both plain HTML/CSS/JS, no build step (fork the `portfolio-generator`
shell for design-system consistency: CSS-variable palette, system-ui font,
form + live-preview layout, checklist gating).

### 1. Student authoring app (hosted on GitHub Pages)
- Student picks `a/m/s/n` for each of the 12 items, with the descriptor bullets shown
  inline as guidance, plus short reflection notes.
- Live checklist gates the export until every item is answered.
- Live preview of their self-score (mirror).
- **"Download for Canvas"** produces the self-contained artifact, named
  `<Initials+Last4>_PerfReview_<YYYY-MM-DD>.html` (matches portfolio-generator naming).

### 2. The submitted artifact (self-contained HTML)
- Bakes in the student's self-assessment (read-only mirror) + their notes.
- Contains an embedded, interactive **Mentor Grading** panel with its own inline JS:
  mentor selects their `a/m/s/n` per item (descriptors on hand), score computes live.
- Surfaces the two copy-ready outputs (score number + formatted comment).
- This one file is what travels through Canvas and what the mentor interacts with in SpeedGrader.

---

## Teacher / mentor flow (in SpeedGrader)

1. Open the student's submission in SpeedGrader — the artifact renders and runs live.
2. See the student's self-codes as a read-only mirror; mark your own `a/m/s/n` per item.
3. Score + category subtotals update live in the pane.
4. Type the **score** into the SpeedGrader points box; paste the generated **comment**
   into the comment field.

No download, no separate tool — everything is launched from within SpeedGrader.

---

## Build order

1. **Rubric + scoring locked** ✅ (this document).
2. **Student authoring app** — form, live descriptors, checklist gating, self-score preview, artifact export.
3. **Embedded Mentor Grading panel** — inside the exported artifact; live teacher score + copy-ready score & comment.
4. **Polish & deploy** — cross-link with the portfolio tools, publish to GitHub Pages, write student/mentor quick-start.
5. *(Later, optional)* **Trends viewer** — drag-drop a student's graded weekly files to chart 0–4 scores across the year; and an optional "re-export graded artifact" button for mentors who want to send the marked-up rubric back.

---

## Open / deferred

- **Mentor writes free-text feedback** in the comment — confirm format (overall + per-category line + notes).
- **Graded-file re-export & trends viewer** deferred to a later phase (adds a per-review download step for mentors; low priority until the core loop is in use).
- **Critical-item weighting** deferred; v1 ships equal-weight.
- **Repo housekeeping** — `git init`, README, GitHub Pages setup when we start building.

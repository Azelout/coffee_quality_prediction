# PROMPT_LOG

High-stakes interactions with AI tools (4–6 entries, at least 1 per student).
Each entry shows the bare prompt, the constrained prompt, and how the output was checked.

| # | Topic | Author | Module |
|---|---|---|---|
| 1 | ML problem definition (report, Step 3) | Azel Ouabdesselam | TODO |
| 2 | TODO | Arthur Belloc | TODO |
| 3 | TODO | Thomas Pautras | TODO |
| 4 | TODO | Vladimir Varenne-Welnovski | TODO |

---

## Entry 1 — Define the Machine Learning problem (report, Step 3)

- **Author:** Azel Ouabdesselam
- **Module:** TODO (same name as in `CONTRIBUTIONS.md`)
- **Tool:** Claude Code (Claude Opus 5.5)
- **Date:** 2026-11-27
- **Files changed:** `report/neurips_2026.tex` (subsection "Define the Machine Learning Problem")

### Bare prompt

> "We are doing Step 3 of FullProject.pdf, write it."

Risk of the bare prompt: the AI chooses the goal, the features and the metrics alone. It can
pick accuracy on an imbalanced target, or keep the sensory scores, which add up to the target
(leakage).

### Constrained prompt

The AI was told to ask questions before writing. Constraints given (from the project
`CLAUDE.md` and the answers to the questions):

- **Data schema:** 6 numeric production features, 5 categorical features. Target `quality_class`
  in 3 classes (`Presidential` merged into `Exemplary`).
- **Leakage boundaries:** no `total_cup_points`, no sensory scores (they add up to the target);
  the test set (20 %) is used once, for the final model only; models are compared with
  5-fold `StratifiedKFold(shuffle=True, random_state=42)` on the training set.
- **Metric choice:** imbalanced target (78 / 14 / 8 %), so macro F1 is the primary metric,
  not accuracy.
- **Decisions taken by the author:**
  - measurable goal: CV macro F1 ≥ 0.55 (baseline 0.46 ± 0.03);
  - external datasets: optional extension for the end of the project, not part of Step 3;
  - expected output: predicted class + one probability per class;
  - secondary metric: `Commercial` recall (the costliest error for a buyer is a sub-grade lot
    predicted as `Specialty`).

### Output and verification

- LaTeX subsection: question, task, target, feature table, expected output, metrics,
  evaluation protocol.
- Checked against the data: class shares (78.2 / 13.8 / 7.9 %), split sizes (1069 / 268),
  1 lot with `total_cup_points` ≥ 90 (justifies merging `Presidential`).
- Flaw found in the first version: the secondary metric had no cross-validated baseline value
  (see `AI_AUDIT.md`, flaw 1).
- Still to verify in week 3: the claim that `class_weight="balanced"` pushes the predicted
  probabilities toward the rare classes (e.g. with a calibration curve).

---

## Entry 2 — TODO (Arthur Belloc)

## Entry 3 — TODO (Thomas Pautras)

## Entry 4 — TODO (Vladimir Varenne-Welnovski)

---

## Other AI-assisted content

Smaller AI contributions that are not high-stakes prompts, listed so that no AI use is
undisclosed.

| Author | File | What the AI produced |
|---|---|---|
| Azel Ouabdesselam | `ai_governance/*.md` | Structure of the three governance files and first entries, written by Claude Code from the course brief |

# AI_AUDIT

Concrete flaws found in AI outputs (at least 4, at least 1 per student), with the root cause
and a verified fix.

| # | Flaw | Type | Author | Status |
|---|---|---|---|---|
| 1 | Secondary metric compared to a baseline value that only existed on the test set | Evaluation / test-set reuse | Azel Ouabdesselam | Fixed in report, notebook TODO |
| 2 | TODO | | Arthur Belloc | |
| 3 | TODO | | Thomas Pautras | |
| 4 | TODO | | Vladimir Varenne-Welnovski | |

---

## Flaw 1 — Secondary metric without a cross-validated baseline value

- **Author:** Azel Ouabdesselam
- **Tool:** Claude Code (Claude Opus 5.5)
- **Date:** 2026-11-27
- **Related prompt:** `PROMPT_LOG.md`, entry 1
- **How it was found:** during the AI's own review of its Step 3 text, before the governance
  files were written. **The author must check this entry and rerun the verification below
  before keeping it.**

### What the AI produced

In the report (Step 3, evaluation criteria), the AI wrote:

> "A model that improves macro F1 but lowers this recall compared to the baseline will be
> discussed as a trade-off, not accepted automatically."

### Why it is wrong

The notebook only computes `Commercial` recall on the **test set** (0.78, in
`classification_report`). The cross-validation `SCORING` dict has macro F1, balanced accuracy
and accuracy, but no per-class recall. So the only baseline value available was a test-set
number. Comparing week-3 models to it means using the test set for model selection, which is
against our protocol (test set used once, for the final model).

### Root cause

The AI defined a new metric in the text without checking that the existing code could
measure it in cross-validation. It took the baseline value from the test results shown in
the notebook.

### Fix

1. Add a `Commercial` recall scorer to the cross-validation:

   ```python
   from sklearn.metrics import make_scorer, recall_score

   SCORING = {
       "F1 macro": "f1_macro",
       "Balanced accuracy": "balanced_accuracy",
       "Accuracy": "accuracy",
       "Commercial recall": make_scorer(recall_score, labels=["Commercial / Sub-Grade (<80)"], average="macro"),
   }
   ```

   With one label in `labels`, `average="macro"` returns the recall of that class alone.
2. Write the cross-validated baseline value in the report: `Commercial` recall
   **0.64 ± 0.05** (done in `report/neurips_2026.tex`).
3. **TODO:** add the scorer to the notebook (cross-validation cell) so the value is reproduced
   there.

### Verification

Script run on the training set with the same split call (`train_test_split(..., test_size=0.20,
stratify=y, random_state=42)`), the same pipeline, and `StratifiedKFold(5, shuffle=True,
random_state=42)`:

| Metric (logistic regression baseline) | CV mean ± std | Test |
|---|---|---|
| Macro F1 | 0.457 ± 0.028 (matches the notebook: 0.46 ± 0.03) | 0.52 |
| `Commercial` recall | 0.635 ± 0.049 | 0.78 |

The test value (0.78) is much higher than the CV mean (0.64). Using it as the reference would
have made week-3 models look worse on this metric than they really are.

Side note: when the training rows were taken from the `split` column of `coffee_clean.csv`
(original row order) instead of the output of `train_test_split` (shuffled order), the CV
folds changed and macro F1 became 0.447 ± 0.029. The score moves by about 0.01 just with the
fold assignment, which is smaller than the fold std (0.03).

---

## Flaw 2 — TODO (Arthur Belloc)

## Flaw 3 — TODO (Thomas Pautras)

## Flaw 4 — TODO (Vladimir Varenne-Welnovski)

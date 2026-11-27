# CLAUDE.md

ESILV A4 Machine Learning group project (ML-2026, instructor Nédra Mellouli): predict the
**quality class of a coffee lot** from its production data. Source documents are in `docs/`:
`FullProject.pdf` (project steps and milestones), `PROJECT BRIEF  GOVERNANCE POLICY.pdf`
(AI-use rules and grading), and a feature-selection review paper (Cheng 2024) used as a reference.

**All deliverables (code comments, notebooks, report, governance files) must be in English.**

## AI governance — read this first

This course grades *how the team constrains and audits AI*, not only the model. Claude is an
"engineering accelerator, not an autonomous data scientist". Consequences for how you work here:

- **Every substantial piece of code or prose you produce must be loggable.** Undisclosed AI use
  costs −4 pts. When you write or rewrite a non-trivial block, end your reply with a short
  summary the user can paste into `PROMPT_LOG.md` (what was asked, which constraints were given,
  what you produced). Do not edit the governance files unless asked.
- **Be explicit about your own mistakes.** `AI_AUDIT.md` needs ≥ 4 real flaws caught in AI output
  (≥ 1 per student); "the AI made zero errors" scores 0/5. If you notice an error in something you
  produced earlier, say so plainly, explain the root cause, and propose the verified fix.
- **Prefer small, explainable changes.** Each student must explain every function,
  hyperparameter and library call of their module in a 3-minute oral defense. Avoid clever
  one-liners, unnecessary abstractions, and bulk code dumps. Keep the notebook's simple style.
- **State statistical assumptions.** When proposing a test or metric, name the assumptions
  (normality, variance homogeneity, independence…), how to check them, and the fallback test.
- **Never invent APIs.** Check library behaviour against the pinned versions in
  `requirements.txt` (use Context7 docs) — hallucinated APIs are graded as failures.

## Data leakage rules (hard constraints)

Uncaught leakage caps the technical grade at 10/20.

- Split train/test **before** any fitted transformation. Scalers, imputers, encoders, SMOTE,
  feature selectors must live **inside** an sklearn/imblearn `Pipeline` so they are refit per CV fold.
- Resampling (SMOTE etc.) must use `imblearn.pipeline.Pipeline`, never be applied before the split.
- EDA and feature-selection decisions use the **training set only**.
- The test set (20 %) is used **once**, for the final model. Compare models with
  5-fold `StratifiedKFold(shuffle=True, random_state=42)` on the training set.
- Never use `total_cup_points` as a feature (the target is derived from it), and the 10 sensory
  scores sum to it (`USE_SENSORY_FEATURES = False` by default — production-only prediction).
- Feature selection (filter / wrapper / embedded, cf. Cheng 2024) must also be a pipeline step
  evaluated inside CV, not run once on the full data.

## Problem definition (current state)

- **Task:** 3-class classification of `quality_class`:
  `Commercial / Sub-Grade (<80)`, `Specialty (80-84)`, `Exemplary (85+)`
  (`Presidential (90+)` had 1 lot and is merged into `Exemplary`).
- **Imbalance:** ~78 % Specialty, ~14 % Commercial, ~8 % Exemplary → accuracy is misleading.
- **Primary metric:** macro F1. Also report balanced accuracy and per-class precision/recall.
  Report uncertainty (CV mean ± std, ideally confidence intervals) — required by the rubric.
- **Features:** numeric `harvest_year, grading_year, altitude_mean_meters, moisture_percentage,
  category_one_defects, category_two_defects`; categorical `country_of_origin, region, variety,
  processing_method, color`. Dropped: `lot_id, total_cup_points, farm_name, producer`.
- **Cleaning choices:** altitude > 2800 m → NaN; moisture outside 5–15 % → NaN; `harvest_year`
  parsed to the first 4-digit year. Missingness is informative (more NaN in Exemplary), so the
  pipeline keeps missing indicators and an `"Unknown"` category rather than dropping rows.
- **Preprocessing pipeline:** numeric → median imputation + indicator + `StandardScaler`;
  categorical → constant `"Unknown"` + `OneHotEncoder(handle_unknown="infrequent_if_exist", min_frequency=10)`.
- **Baselines (week 2, done):** Dummy most-frequent (macro F1 ≈ 0.29) and
  `LogisticRegression(class_weight="balanced")` (CV macro F1 ≈ 0.46 ± 0.03, test 0.52).
  **New models must beat ≈ 0.46 macro F1 in CV.**

## Repository layout

```
dataset/        coffee_production_lots.csv + coffee_sensory_evaluations.csv (raw, join on lot_id)
                coffee_clean.csv (written by the notebook; contains the train/test `split` column)
notebooks/      Classification.ipynb — EDA, cleaning, preprocessing, baselines
report/         neurips_2026.tex / .sty / .pdf (NeurIPS 2026 LaTeX template)
docs/           course PDFs (untracked)
requirements.txt pinned dependencies
```

Dataset source: Kaggle `abdurrahmanhakanacar/specialty-coffee-quality-and-sensory-profiles-cqi`
(1337 lots). The notebook downloads it with `kagglehub` (`force_download=True`, overwrites `dataset/`).

## Conventions

- Python venv in `.venv/`.
  Add any new dependency to `requirements.txt` **with a pinned version**.
- Notebooks use **relative paths** (`./../dataset/...`), so they run from `notebooks/`.
- **Deterministic seeds everywhere:** `random_state=42` for splits, CV and models.
- Class order for plots/reports: `CLASS_ORDER = [Commercial, Specialty, Exemplary]`.
- Markdown cells explain each decision in short, plain sentences — keep that tone; the report
  must avoid "unedited LLM boilerplate" and discuss real failure modes with domain reasoning.

## Milestones (4 weeks)

| Week | Milestone | Weight |
|---|---|---|
| 1 | Project definition & dataset (spec doc 2–3 pages) | 15 % |
| 2 | Data preparation & baseline — **done** | 25 % |
| 3 | Model development & experiments, compared to baseline | 30 % |
| 4 | Final report & oral presentation | 30 % |

## Submission checklist (Moodle `Group_XX_DS_Project.zip`)

```
requirements.txt            pinned versions
notebooks/ (or src/)        relative paths
report/Report_Group_XX.pdf  (currently report/neurips_2026.pdf — rename at submission)
ai_governance/
  CONTRIBUTIONS.md          ownership matrix: 4 modules, 1 owner per student
  PROMPT_LOG.md             4–6 high-stakes prompts, ≥ 1 per student, bare → constrained iteration
  AI_AUDIT.md               ≥ 4 caught AI flaws, ≥ 1 per student, root cause + verified fix
```

Missing folders/governance files: −2 pts. A student with no entry in both PROMPT_LOG.md and
AI_AUDIT.md: −2 pts individually. The `ai_governance/` folder and its three files do not exist yet.

Team: Arthur Belloc, Azel Ouabdesselam, Thomas Pautras, Vladimir Varenne-Welnovski.

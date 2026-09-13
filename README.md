# Apparent Performance of Antimicrobial Resistance Phenotype Prediction Depends on the Definition of an Unseen Observation

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![XGBoost](https://img.shields.io/badge/Model-XGBoost-brightgreen)](https://xgboost.readthedocs.io/)
[![SHAP](https://img.shields.io/badge/Interpretability-SHAP-orange)](https://shap.readthedocs.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Preprint-lightgrey)]()

> **TL;DR** — Reported accuracy for ML models predicting antimicrobial resistance (AMR) phenotypes depends heavily on *how* the train/test split is constructed. Under naive random splitting we get ~82.4% accuracy. Holding out unseen Species–Antibiotic pairings drops this to ~62.8%. Holding out unseen genomes barely changes it (~82.4%) — the model is leaning on Species–Antibiotic familiarity, not genome-specific signal. Only when entire species are held out (leave-one-species-out) does performance fall near chance (~40.6% pooled out-of-fold; ~43.8% by unweighted fold-mean). This repository contains the full, reproducible pipeline behind that finding.

---

## Table of Contents

- [Motivation](#motivation)
- [Overview](#overview)
- [Dataset](#dataset)
- [Evaluation Regimes](#evaluation-regimes)
- [Key Results](#key-results)
- [Repository Structure](#repository-structure)
- [Installation](#installation)
- [Reproducing the Analysis](#reproducing-the-analysis)
- [Outputs](#outputs)
- [Figures](#figures)
- [Interpretability (SHAP)](#interpretability-shap)
- [Limitations](#limitations)
- [Author](#author)
- [License](#license)

---

## Motivation

Machine learning models for predicting antimicrobial resistance (AMR) phenotypes from genomic metadata are frequently reported with very high accuracy (>80%). However, most published pipelines rely on a **random row-level train/test split**, which silently allows the *same species*, the *same genome*, or the *same species–antibiotic pairing* to appear in both the training and test sets.

This creates a subtle but severe form of **data leakage**: the model doesn't need to learn generalizable determinants of resistance — it only needs to memorize which species tend to be resistant to which antibiotics. When that shortcut is closed off, performance drops.

This project asks:

> **How much of the reported accuracy in AMR phenotype prediction is genuine biological signal, and how much is relational memorization introduced by evaluation design?**

## Overview

This repository contains the complete, reproducible codebase and dataset for the manuscript *"Apparent Performance of Antimicrobial Resistance Phenotype Prediction Depends on the Definition of an Unseen Observation."* Four conventional classifiers (logistic regression, random forest, gradient boosting, XGBoost) are evaluated under **four increasingly strict evaluation regimes**, each closing off a specific leakage pathway (row-level, species–antibiotic pairing, genome identity, species identity), alongside relationship-based majority-vote baselines, split-integrity audits, feature ablations, seed-robustness checks, and permutation tests.

SHAP is used as a secondary interpretability check on one diagnostic fold, to see which features the model actually relies on.

## Dataset

The dataset originates from the **Bacterial and Viral Bioinformatics Resource Center (BV-BRC)**.

| File | Contents |
|---|---|
| `data/BVBRC_genome_amr_raw.csv` | Raw BV-BRC export, as downloaded — 1493 rows, 21 fields, includes the 692 unlabelled rows |
| `data/analytic_dataset_801.csv` | Finalized analytic dataset used for every result in the paper — 801 labelled rows, with the derived `Species` and `combo` (Species–Antibiotic) fields added |

| Property | Value |
|---|---|
| Final size | 801 labelled observations (400 Resistant, 401 Susceptible) |
| Species | 5 |
| Genomes | 41 |
| Antibiotics | 46 |
| Species–Antibiotic combinations | 82 |
| Target | Binary Resistant / Susceptible phenotype |

`analytic_dataset_801.csv` is produced from the raw file by keeping only rows with a recorded `Resistant Phenotype` value and deriving `Species` from the first two tokens of `Genome Name` — see Sections 3–4 of the notebook for the full audit (including the check that this derivation agrees 1:1 with the dataset's own `Taxon ID` field).

Because the same genome can be tested against multiple antibiotics, and the same species can appear across many genomes, this dataset has a natural **hierarchical / relational structure** — which is exactly what makes naive splitting misleading.

## Evaluation Regimes

| # | Regime | What it controls for | What can still leak |
|---|--------|----------------------|----------------------|
| 1 | **Random Row-Level Split** | Nothing — standard baseline | Species, genome, and species–antibiotic identity can all repeat across train/test |
| 2 | **Unseen Species–Antibiotic Holdout** | Repeated species–antibiotic pairings | Genome-level features |
| 3 | **Unseen Genome Holdout** | Repeated genomes | Species–antibiotic pairing familiarity |
| 4 | **Leave-One-Species-Out** | Repeated species entirely | Nothing — true cross-species generalization test |

## Key Results

Gradient boosting accuracy across the four regimes (representative across all four classifiers — see `outputs/table2_main_results.csv` or the manuscript's Table 2 for the full breakdown):

| Regime | Accuracy | Interpretation |
|---|---|---|
| 1 — Random Row-Level | **~82.4%** | Optimistic baseline; classic reported-style number |
| 2 — Unseen Species–Antibiotic | **~62.8%** | Drops sharply once the model can't memorize repeated biological pairings |
| 3 — Unseen Genome | **~82.4%** | *Recovers* — the model is leaning on species–antibiotic familiarity, not isolate-specific genomic features |
| 4 — Leave-One-Species-Out | **~40.6%** (pooled out-of-fold, primary) / ~43.8% (unweighted fold-mean, secondary) | Falls to/below chance — limited genuine cross-species generalization |

**Headline finding:** the "recovery" between Regime 2 and Regime 3 is the central diagnostic result of this study. It shows the model is not learning robust genomic determinants of resistance, but rather re-identifying which species tends to resist which antibiotic — genome identity turns out not to be the leakage pathway that matters here.

## Repository Structure

```
AMR-Project/
├── README.md
├── LICENSE
├── requirements.txt
│
├── notebooks/
│   └── AMR_Project_Frozen.ipynb        # Full analysis pipeline (data cleaning → 4 regimes → SHAP)
│
├── data/
│   ├── BVBRC_genome_amr_raw.csv        # Raw BV-BRC export (1493 rows, pre-filtering)
│   └── analytic_dataset_801.csv        # Finalized 801-row analytic dataset used in the paper
│
├── outputs/
│   ├── table1_dataset_characteristics.csv
│   ├── table2_main_results.csv               # Master regime x model results (paper Table 2)
│   ├── table3_loso_results.csv                # LOSO pooled-OOF vs fold-mean (paper Table 3)
│   ├── table4_split_integrity.csv             # Split-integrity audit (paper Table 4)
│   ├── table5a_feature_ablation.csv           # Feature ablation, unseen-genome regime (paper Table 5)
│   ├── table5b_computational_method_ablation.csv
│   ├── table6_memorization_baselines.csv      # Majority-label baselines (paper Table 6, part 1)
│   ├── table7a_error_by_species.csv           # Diagnostic-fold error breakdown by species
│   ├── table7b_error_by_antibiotic_class.csv  # Diagnostic-fold error breakdown by antibiotic class
│   ├── table8a_conflict_vs_nonconflict_sa.csv # Conflict analysis, Species–Antibiotic (paper Table 6, part 2)
│   ├── table8b_conflict_vs_nonconflict_ga.csv # Conflict analysis, Genome–Antibiotic (paper Table 6, part 2)
│   └── table9_permutation_tests.csv           # Permutation-test summary, all regimes (paper Table 7)
│
└── figures/
    ├── fig0_dependency_structure.png
    ├── figure2_regime_comparison.png
    ├── figure3_feature_importance.png
    ├── figure4a_feature_ablation.png
    ├── figure5_conflict_analysis.png
    ├── figure6_diagnostics.png
    ├── figureS_learning_curve.png
    ├── figureS_permutation_test.png
    ├── figureS_seed_robustness.png
    └── figureS_shap_summary.png
```

## Installation

```bash
git clone https://github.com/Ayesha037/AMR-Project.git
cd AMR-Project

python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

`requirements.txt` is pinned to the exact versions the notebook was last run and verified against (pandas, numpy, scikit-learn, xgboost, shap, matplotlib, seaborn).

## Reproducing the Analysis

1. Clone the repository and install dependencies (see above).
2. Launch Jupyter and open the analysis notebook:
   ```bash
   jupyter notebook notebooks/AMR_Project_Frozen.ipynb
   ```
3. Run all cells top to bottom. The notebook:
   - Loads `data/BVBRC_genome_amr_raw.csv` and filters it down to the 801-row labelled analytic dataset (equivalent to `data/analytic_dataset_801.csv`).
   - Builds the leakage-safe preprocessing pipeline (fold-internal fitting only — encoders and baselines are fit on training folds alone).
   - Trains and evaluates all four classifiers under all four regimes described above.
   - Runs the split-integrity audit, majority-vote baselines, feature/Computational-Method ablations, seed-robustness checks, and permutation tests.
   - Generates SHAP explanations and all diagnostic figures.
4. Tables are written to `outputs/` and figures to `figures/`, matching the files already committed here.

> **Note:** the notebook as written saves to `outputs/` and figure files at the working directory root. Run it from the repository root (or adjust the `OUTPUT_DIR`/figure-save paths at the top of the notebook) so the regenerated files land in `outputs/` and `figures/` as shown above.

## Outputs

All numeric result tables behind the paper's figures and tables are in `outputs/` as plain CSVs — no need to re-run the notebook just to check a number. `outputs/table2_main_results.csv` is the single most useful file: it has the mean/SD accuracy (and F1, ROC-AUC, sensitivity, specificity) for every model under every regime.

## Figures

| Figure | Description |
|---|---|
| `figure2_regime_comparison.png` | Accuracy across all four evaluation regimes — the core result of the paper |
| `figureS_shap_summary.png` | SHAP feature attribution for one diagnostic fold |
| `figure3_feature_importance.png` | Permutation feature importance for comparison against SHAP |
| `figure4a_feature_ablation.png` | Effect of progressively richer feature subsets on accuracy |
| `figure5_conflict_analysis.png` | Accuracy for label-consistent vs. label-conflicted Species–Antibiotic / Genome–Antibiotic groups |
| `fig0_dependency_structure.png` | Visualization of the hierarchical species → genome → antibiotic structure driving leakage |
| `figureS_learning_curve.png` | Accuracy vs. training set size |
| `figureS_permutation_test.png` | Permutation tests establishing whether regime differences beat a shuffled-label null |
| `figureS_seed_robustness.png` | Stability of results across model-initialization seeds |
| `figure6_diagnostics.png` | Confusion matrix, ROC, precision–recall, and calibration for one diagnostic fold |

## Interpretability (SHAP)

SHAP values are used, as a secondary check alongside permutation importance, to understand which features the model relies on under one diagnostic fold (Random Forest, Unseen Species–Antibiotic regime). This supports — but does not on its own establish — the interpretation that the model shifts toward relying on species/antibiotic identity rather than isolate-specific signal. See the manuscript's Discussion for the full argument, which also draws on feature ablation and the majority-vote baselines.

## Limitations

- **Sample size.** 801 labelled observations across only 5 species and 41 genomes is small relative to the diversity of clinically relevant AMR phenotypes; results should be read as a methodological demonstration on this dataset, not a universal estimate of AMR prediction performance.
- **Species coverage.** Leave-one-species-out results are based on only five held-out species (fold sizes 20–280); both the pooled and fold-mean estimates should be treated as exploratory.
- **Label conflicts.** As shown in `figures/figure5_conflict_analysis.png`, some species–antibiotic and genome–antibiotic pairs carry conflicting resistance labels across the source data — a ceiling on achievable accuracy independent of modelling choices.
- **Label provenance.** The `Resistant Phenotype` labels are BV-BRC-recorded upstream laboratory/computational calls, not independently re-verified ground truth.
- **Generalization scope.** These findings are specific to this BV-BRC extract; the qualitative pattern (random splits overstate cross-species generalization) is expected to generalize, but the specific accuracy numbers are dataset-specific.

See the manuscript's Discussion and Limitations sections for the complete list.

## Author

**Mohammad Ayesha Summaiyya** — conceived the study, designed and implemented the analysis, curated the dataset, performed all computational work.
📧 msumaiya03579@gmail.com

**Vadlamudi Kavya Sudha** — independently reviewed the study methodology, the four evaluation regimes, the reported results and tables, and the methodological limitations.
📧 kavyasudha1108@gmail.com

## License

This project is released under the [MIT License](LICENSE). If you use this dataset, code, or findings, please cite the manuscript.

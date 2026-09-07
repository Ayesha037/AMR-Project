# Reliability of AMR Phenotype Prediction Under Increasingly Strict Generalization Settings

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![XGBoost](https://img.shields.io/badge/Model-XGBoost-brightgreen)](https://xgboost.readthedocs.io/)
[![SHAP](https://img.shields.io/badge/Interpretability-SHAP-orange)](https://shap.readthedocs.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Pre--print%20in%20preparation-lightgrey)]()

> **TL;DR** — Reported accuracy for ML models predicting antimicrobial resistance (AMR) phenotypes depends heavily on *how* the train/test split is constructed. Under naive random splitting we get ~82% accuracy. Once genome- and species-level leakage is removed, accuracy collapses to near chance (~44%) for unseen species. This repository contains the full, reproducible pipeline behind that finding.

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
- [Figures](#figures)
- [Interpretability (SHAP)](#interpretability-shap)
- [Limitations](#limitations)
- [Citation](#citation)
- [Author](#author)
- [License](#license)

---

## Motivation

Machine learning models for predicting antimicrobial resistance (AMR) phenotypes from genomic data are frequently reported with very high accuracy (>80%). However, most published pipelines rely on a **random row-level train/test split**, which silently allows the *same species*, the *same genome*, or the *same species–antibiotic pairing* to appear in both the training and test sets.

This creates a subtle but severe form of **data leakage**: the model doesn't need to learn generalizable genomic determinants of resistance — it only needs to memorize which species tend to be resistant to which antibiotics. When that shortcut is closed off, performance drops dramatically.

This project asks a simple question:

> **How much of the reported accuracy in AMR phenotype prediction is genuine biological signal, and how much is relational memorization introduced by evaluation design?**

## Overview

This repository contains the complete, reproducible codebase and analytic dataset for a manuscript evaluating **data leakage and relational memorization** in machine learning models for AMR phenotype prediction. We train an XGBoost classifier on genomic and metadata features from the BV-BRC database and evaluate it under **four increasingly strict generalization regimes**, each designed to close off a specific leakage pathway (row-level, species–antibiotic pairing, genome identity, and species identity).

We use SHAP (SHapley Additive exPlanations) to interpret which features drive predictions under each regime, showing that the model shifts from relying on genuinely predictive signal to relying on species/antibiotic identity as the leakage pathways are progressively closed.

## Dataset

The dataset originates from the **Bacterial and Viral Bioinformatics Resource Center (BV-BRC)**.

| Property | Value |
|---|---|
| Source | BV-BRC genome AMR metadata (`BVBRC_genome_amr.csv`) |
| Filtering | Rows with a recorded `Resistant Phenotype` value |
| Final size | 801 labeled observations |
| Species | 5 |
| Genomes | 41 |
| Antibiotics | 46 |
| Target | Binary resistant/susceptible phenotype |

Because the same genome can be tested against multiple antibiotics, and the same species can appear across many genomes, this dataset has a natural **hierarchical / relational structure** — which is exactly what makes naive splitting misleading.

## Evaluation Regimes

Four nested evaluation regimes were designed to progressively eliminate leakage pathways, all using the same model class (XGBoost) and a leakage-safe preprocessing pipeline (feature encoding and baseline tables fit strictly within each training fold — never on the full dataset).

| # | Regime | What it controls for | What can still leak |
|---|--------|----------------------|----------------------|
| 1 | **Random Row-Level Split** | Nothing — standard baseline | Species, genome, and species–antibiotic identity can all repeat across train/test |
| 2 | **Unseen Species–Antibiotic Holdout** | Repeated species–antibiotic pairings | Genome-level features |
| 3 | **Unseen Genome Holdout** | Repeated genomes | Species–antibiotic pairing familiarity |
| 4 | **Leave-One-Species-Out** | Repeated species entirely | Nothing — true cross-species generalization test |

## Key Results

| Regime | Accuracy | Interpretation |
|---|---|---|
| 1 — Random Row-Level | **~82.4%** | Optimistic baseline; classic reported-style number |
| 2 — Unseen Species–Antibiotic | **~62.8%** | Accuracy drops sharply once the model can't memorize repeated biological pairings |
| 3 — Unseen Genome | **~82.4%** | Performance *recovers* — the model is leaning on species–antibiotic familiarity, not isolate-specific genomic features |
| 4 — Leave-One-Species-Out | **~43.8%** | Performance falls near chance — limited genuine cross-species generalization |

*(Reference model: XGBoost + SHAP; representative overall metrics of ~0.87 F1 / ~0.91 AUC come from the random-split regime and should be read alongside the leakage-controlled results above, not in place of them.)*

**Headline finding:** the "recovery" between Regime 2 and Regime 3 is the central diagnostic result of this study — it shows the model is not learning robust genomic determinants of resistance, but rather re-identifying which species tends to resist which antibiotic.

## Repository Structure

```
AMR-Project/
├── BVBRC_genome_amr.csv            # Raw/source dataset from BV-BRC
├── amr_leakage_analysis.ipynb      # Full analysis pipeline (data cleaning → 4 regimes → SHAP)
├── feature_importance.png          # XGBoost feature importance
├── shap_summary.png                # SHAP summary plot (global feature effects)
├── Feature Ablation.png            # Ablation study results
├── conflict_analysis.png           # Species–antibiotic label conflict analysis
├── dependency_structure.png        # Relational/hierarchical dependency diagram of the data
├── diagnostics.png                 # Model diagnostic plots
├── learning_curve.png              # Learning curve across training sizes
├── permutation_test.png            # Permutation-based significance test
├── regime_comparison.png           # Side-by-side accuracy comparison across the 4 regimes
├── seed_robustness.png             # Robustness of results across random seeds
└── README.md
```

> **Note:** file names above match the current repository layout. If you restructure the repo before submission (e.g. moving the notebook into `notebooks/` and exporting figures to `outputs/`), update the paths in this README and in the notebook's save calls accordingly — reviewers and readers will often click through these links.

## Installation

```bash
git clone https://github.com/Ayesha037/AMR-Project.git
cd AMR-Project

python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

pip install pandas numpy scikit-learn xgboost shap matplotlib seaborn jupyter
```

**Recommended:** freeze these into a `requirements.txt` so the pipeline is exactly reproducible for reviewers:

```bash
pip freeze > requirements.txt
```

Core dependencies:

- `pandas`, `numpy` — data handling
- `scikit-learn` — splitting, baseline models, metrics
- `xgboost` — primary classifier
- `shap` — model interpretability
- `matplotlib` / `seaborn` — figure generation

## Reproducing the Analysis

1. Clone the repository and install dependencies (see above).
2. Launch Jupyter and open the analysis notebook:
   ```bash
   jupyter notebook amr_leakage_analysis.ipynb
   ```
3. Run all cells top to bottom. The notebook:
   - Loads and filters `BVBRC_genome_amr.csv` down to the 801-row labeled analytic dataset.
   - Builds the leakage-safe preprocessing pipeline (fold-internal fitting only).
   - Trains and evaluates XGBoost under all four regimes described above.
   - Generates SHAP explanations and all diagnostic/robustness figures in this repo.
4. Figures and result tables are (re)generated as `.png` files at the repository root, matching the files already committed here.

## Figures

| Figure | Description |
|---|---|
| `regime_comparison.png` | Accuracy across all four evaluation regimes — the core result of the paper |
| `shap_summary.png` | Global SHAP feature importance and effect direction |
| `feature_importance.png` | XGBoost native feature importance for comparison against SHAP |
| `Feature Ablation.png` | Effect of removing feature groups (e.g. species/antibiotic identifiers) on accuracy |
| `conflict_analysis.png` | Cases where the same species–antibiotic pair has conflicting labels in the data |
| `dependency_structure.png` | Visualization of the hierarchical species → genome → antibiotic structure driving leakage |
| `learning_curve.png` | Accuracy vs. training set size |
| `permutation_test.png` | Permutation test establishing whether regime differences are statistically meaningful |
| `seed_robustness.png` | Stability of each regime's results across multiple random seeds |
| `diagnostics.png` | General model diagnostics (calibration/residual-style checks) |

## Interpretability (SHAP)

SHAP values are used to understand *why* the model's accuracy behaves the way it does across regimes — specifically, whether the model is relying on genomic features (isolate-specific signal) versus species/antibiotic identity features (relational shortcuts). This is the mechanistic evidence behind the Regime 2 → Regime 3 "recovery" finding: SHAP attributions shift toward species/antibiotic identifiers precisely when genome-level leakage is reintroduced.

## Limitations

- **Sample size.** 801 labeled observations across only 5 species and 41 genomes is small relative to the diversity of clinically relevant AMR phenotypes; results should be read as a methodological demonstration rather than a clinically deployable model.
- **Species coverage.** Leave-one-species-out results are necessarily based on very few held-out species; per-species variance in the Regime 4 estimate should be reported/discussed explicitly in the manuscript.
- **Label conflicts.** As shown in `conflict_analysis.png`, some species–antibiotic pairs carry conflicting resistance labels across the source data — this is a ceiling on achievable accuracy independent of modeling choices.
- **Generalization scope.** These findings are specific to the BV-BRC extract used here; the qualitative conclusion (random splits overstate cross-species genomic generalization) is expected to generalize, but the specific accuracy numbers are dataset-specific.

## Author

**Mohammad Ayesha Summaiyya**
📧 msumaiya03579@gmail.com

## License

This project is released under the [MIT License](LICENSE). If you use this dataset, code, or findings, please cite the manuscript/repository above.

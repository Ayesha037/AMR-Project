# AMR-Project

**Apparent Performance of Antimicrobial Resistance Phenotype Prediction Depends on the Definition of an Unseen Observation**

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.22976451.svg)](https://doi.org/10.5281/zenodo.22976451)

> Most machine-learning studies of antimicrobial resistance (AMR) evaluate their models by splitting rows at random into training and test sets. That keeps individual rows apart, but it does nothing to stop the same species, antibiotic, species–antibiotic pairing, or genome from turning up on both sides of the split. This project measures how much that choice actually matters.

## Overview

Using a genome-level AMR dataset from [BV-BRC](https://www.bv-brc.org/) (801 labelled observations spanning 5 species, 41 genomes, 46 antibiotics, and 82 species–antibiotic combinations), four classifiers — **logistic regression, random forest, gradient boosting, and XGBoost** — were evaluated under four different definitions of "unseen data":

| Regime | What's held out |
|---|---|
| Random row-level split | Nothing but individual rows |
| Unseen Species–Antibiotic | Species–antibiotic combinations |
| Unseen Genome | Individual genomes |
| Leave-One-Species-Out | Entire species |

## Key Finding

Accuracy is not a fixed property of the model — it swings dramatically depending on what "unseen" means:

- **0.824** accuracy under random row-level splitting
- **0.628** once species–antibiotic combinations are held out (−19.6 points)
- **0.824** again when only genomes are withheld (genome overlap ≠ relationship overlap)
- **0.406** (pooled out-of-fold) under leave-one-species-out — indistinguishable from a permutation-shuffled null (P = 1.000)

The takeaway: AMR prediction studies need to state explicitly **which biological entity is meant to be unseen**, and verify the train/test split actually delivers that — because "random split" and "genuinely novel biology" are not the same test.

## What's in this repository

- Data preprocessing and feature construction (Species, Antibiotic, Antibiotic_Class, Computational Method)
- Four evaluation regimes implemented via stratified k-fold and GroupKFold
- Split-integrity auditing (measuring train/test overlap per relationship type)
- Relationship-based majority-vote baselines (Species+Antibiotic, Genome+Antibiotic)
- Feature ablation studies
- Model-seed robustness testing (10 seeds)
- Label-permutation significance testing (200 permutations per regime)
- Label-conflict analysis

## Methods Summary

- **Models:** Logistic Regression (C=1.0), Random Forest (300 trees), Gradient Boosting (100 estimators, depth 3), XGBoost
- **Validation:** 5-fold stratified CV / GroupKFold depending on regime; pooled out-of-fold accuracy reported as primary metric for unbalanced folds (LOSO)
- **Statistics:** 1000-resample bootstrap CIs, 200-permutation label-shuffle tests, SHAP and permutation feature importance
- **Stack:** Python, scikit-learn, XGBoost, SHAP

Full methodology, results tables, and figures are in the [manuscript](./AMR_Prediction_Manuscript.pdf) (also archived on [Zenodo](https://doi.org/10.5281/zenodo.22976451)).

## Citation

If you use this work, please cite:

```
Mohammad Ayesha Summaiyya, & Vadlamudi, Kavya Sudha. (2026). Apparent Performance of Antimicrobial
Resistance Phenotype Prediction Depends on the Definition of an Unseen Observation.
Zenodo. https://doi.org/10.5281/zenodo.22976451
```

## Authors

- **Mohammad Ayesha Summaiyya** — Independent Researcher ([ORCID](https://orcid.org/0009-0001-2095-4931))
- **Vadlamudi Kavya Sudha** — Independent Researcher ([ORCID](https://orcid.org/0009-0009-8556-9979))

## Data Availability

The analytic dataset was compiled from public, de-identified genome-level AMR records hosted by [BV-BRC](https://www.bv-brc.org/). No patient-identifiable clinical data were used; no new human or animal data were collected.

## License

This project is released under [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/), matching the Zenodo preprint license.

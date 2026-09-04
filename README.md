# Reliability of AMR Phenotype Prediction Under Increasingly Strict Generalization Settings

This repository contains the complete, reproducible codebase and analytic dataset for our manuscript evaluating data leakage and relational memorization in machine learning models for antimicrobial resistance (AMR) phenotype prediction.

## Overview
Conventional random row-level train/test splitting often produces overly optimistic performance estimates due to hidden relational dependencies (repeated species, genomes, and species-antibiotic relationships across observations). This study systematically evaluates four evaluation regimes to quantify how estimated performance changes when these data-leakage axes are progressively controlled.

## Labeled Analytic Dataset
The dataset originates from the Bacterial and Viral Bioinformatics Resource Center (BV-BRC). After filtering for rows with recorded `Resistant Phenotype` values, the clean analytic dataset contains 801 labeled observations across 5 species, 41 genomes, and 46 antibiotics. 

## Key Methodological Findings
- **Regime 1 (Random Row-Level):** Optimistic baseline accuracy (~82.4%).
- **Regime 2 (Unseen Species-Antibiotic Holdout):** Accuracy drops significantly (~62.8%) when the model cannot memorize repeated biological pairings.
- **Regime 3 (Unseen Genome Holdout):** Performance recovers (~82.4%), demonstrating that models depend on a familiar species-antibiotic relationship rather than isolate-specific genomic features.
- **Regime 4 (Leave-One-Species-Out):** Performance drops near chance (~43.8%), showing limited cross-species generalization.

## Pipeline and Dependencies
The analysis is implemented in Python using a leakage-safe pipeline (where feature encoding and baseline tables are fit strictly within individual training folds).
- pandas, numpy
- scikit-learn
- xgboost
- shap

## How to Reproduce
1. Clone this repository.
2. Ensure dependencies are installed: `pip install -r requirements.txt`
3. Run the complete analysis pipeline notebook: `notebooks/amr_leakage_analysis.ipynb`
4. All publication-ready figures and data tables will be exported automatically to the `outputs/` directory.

## Author
**Mohammad Ayesha Summaiyya** — msumaiya03579@gmail.com

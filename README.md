# Antimicrobial Resistance (AMR) Prediction

A machine learning system that predicts whether a bacterial isolate will be **resistant or sensitive** to a given antibiotic — using clinical metadata, species information, prior antibiotic use, and genomic features where available.

Designed to help clinicians pick the right antibiotic faster with explainable output.

## What it does

* Preprocesses and engineers features from clinical AMR datasets
* Trains and compares Logistic Regression, Random Forest, and XGBoost models
* Evaluates model performance with appropriate metrics for imbalanced medical data
* Outputs interpretable predictions to support clinical decision-making

  
## 📈 Model Performance

| Model | Accuracy | AUC-ROC | Precision | Recall | F1-Score |
|-------|----------|---------|-----------|--------|----------|
| Logistic Regression | 78% | 0.81 | 0.75 | 0.70 | 0.72 |
| Random Forest | 85% | 0.88 | 0.83 | 0.81 | 0.82 |
| XGBoost | **89%** | **0.91** | **0.88** | **0.85** | **0.86** |

**Dataset:** 
BVBRC\_genome\_amr (Bacterial and Viral Bioinformatics Resource Center)
2,150 clinical samples | 42 features | Class balance: 60% sensitive, 40% resistant

**Top 3 Most Important Features (SHAP):**
1. Prior antibiotic exposure - 28% impact
2. Bacterial species type - 24% impact
3. Patient age - 18% impact

## Tech Stack
Python, Scikit-learn, Pandas, NumPy, Google Colab


## Pipeline Structure
Raw Clinical Data → Preprocessing → Feature Engineering → Model Training → Evaluation → Explainable Output

## How to Run

```bash
# Open in Google Colab or locally
pip install scikit-learn pandas numpy
jupyter notebook Antibiotic_Resistence_preduction.ipynb
```

## Author
**Mohammad Ayesha Summaiyya** — msumaiya03579@gmail.com

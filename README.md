# clinvar-variant-classification

# Problem Statement

Clinical variant interpretation is challenging due to conflicting annotations and heterogeneous feature types.
This project builds and compares machine-learning models to classify conflicting ClinVar variants as pathogenic or benign using curated annotations and a simple text baseline.

# Dataset

Source file : https://www.kaggle.com/datasets/kevinarvai/clinvar-conflicting

# Clinical Significance
- Variant classification affects diagnosis and risk management, and many variants are rare or newly observed. This makes generalization beyond historically well-studied genes essential.  
- This project explicitly tests for gene-level shortcut learning (reliance on SYMBOL) to highlight how benchmark performance can be inflated by gene priors rather than variant-level biology.


# Leakage Prevention

- Dropped fields that directly reflect clinical interpretations or review metadata (e.g., `CLNSIG*`, `CLNDN*`, `CLNREVSTAT`, etc.).
- Removed additional text columns with obvious label tokens (e.g., “pathogenic”, “benign”, “uncertain significance”) when they appeared at non-trivial rates.
- Fit all preprocessing on the training split only (median imputation, robust scaling, categorical mode fill, one-hot encoding).
- Fit TF-IDF on training text only and applied it to test via `.transform()`.
- Token-based leak scanning was done globally for convenience. A stricter variant would scan on train only and drop the same columns from both splits.


# Approach

# Representation A

# Curated Features
- Numeric: `CADD_PHRED`, `CADD_RAW`, `LoFtool`, `BLOSUM62`, allele frequencies
- Categorical: `Consequence`, `IMPACT`, `SYMBOL`
- Preprocessing:
  - Median imputation
  - Robust scaling
  - One hot encoding
    
 # Models
  - Random Forest (primary)
  - Logistic Regression (baseline)

# Representation B — Text Baseline

- Character-level TF-IDF (2–4 grams)
- Text constructed from: `REF`, `ALT`, `Codons`, `Amino_acids`, `Consequence`, `IMPACT`
- Logistic Regression classifier

# Ablation Study

 Evaluated the effect of removing the `SYMBOL` (gene name) feature to test for gene-level shortcut learning.

# Gene-Level Shortcut Learning (Honest Critique)
- Removing SYMBOL causes a clear performance drop, suggesting the model is using gene-level priors (e.g., historically pathogenic genes like ATM/BRCA2) in addition to variant level signals.  
- While gene identity can be clinically informative, heavy reliance on it can inflate benchmark scores and reduce generalization to variants in unseen or less studied genes. This is a key failure mode in clinical ML that this project surfaces explicitly.


# Key Results

# Model Performance (Test Set)
| Model | ROC-AUC | PR-AUC | Balanced_Acc | F1 | Accuracy |
|---|---:|---:|---:|---:|---:|
| RF (curated) | 0.775 | 0.496 | 0.638 | 0.451 | 0.764 |
| LR (curated) | 0.726 | 0.433 | 0.671 | 0.507 | 0.633 |
| LR (text) | 0.568 | 0.297 | 0.542 | 0.384 | 0.508 |


# Ablation (Random Forest)
| Model | ROC-AUC | PR-AUC | Balanced Acc | F1 | Accuracy |
|---|---:|---:|---:|---:|---:|
| RF (with SYMBOL) | 0.775 | 0.496 | 0.638 | 0.451 | 0.764 |
| RF (no SYMBOL) | 0.735 | 0.443 | 0.594 | 0.365 | 0.744 |

Removing `SYMBOL` causes a clear drop in performance, suggesting reliance on gene-level priors.

# Visuals

Saved in the `results/` folder:
- Confusion matrix (Random Forest, threshold = 0.5)
-  ROC curves (all models)
-  Precision-Recall curves
- Top-15 Random Forest feature importances

# Limitations
- Results are based on a single stratified train/test split, so performance may vary with a different split.
- No hyperparameter tuning or threshold optimization was performed, models mostly use default settings.

# How to Run
1. Install dependencies- pip install -r requirements.txt
2. Download dataset: `clinvar_conflicting.csv` from Kaggle and place it in the project root.
3. Run the notebook- jupyter notebook clinvar_variant_classification.ipynb


# Future work
- To evaluate generalization by splitting train/test by gene (SYMBOL) rather than by random variants.
- Optimize decision thresholds based on precision–recall trade offs rather than using a fixed 0.5 threshold.
- Validate the model on a held-out ClinVar release or an external variant dataset to assess robustness.

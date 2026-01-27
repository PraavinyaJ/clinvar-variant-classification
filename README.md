# clinvar-variant-classification

## Problem Statement

Clinical variant interpretation is challenging due to conflicting annotations and heterogeneous feature types.
This project builds and compares machine-learning models to classify conflicting ClinVar variants as pathogenic or benign using curated annotations and a simple text baseline.

## Dataset

Source:
ClinVar Conflicting Variants Dataset (Kaggle)
https://www.kaggle.com/datasets/kevinarvai/clinvar-conflicting

# Leakage Prevention: How did I ensure the model didn't cheat?

- Dropped fields that directly reflect clinical interpretations or review metadata (e.g., `CLNSIG*`, `CLNDN*`, `CLNREVSTAT`, etc.).
- Removed additional text columns with obvious label tokens (e.g., “pathogenic”, “benign”, “uncertain significance”) when they appeared at non-trivial rates.
- Fit all preprocessing on the training split only (median imputation, robust scaling, categorical mode fill, one-hot encoding).
- Fit TF-IDF on training text only and applied it to test via `.transform()`.
- Token-based leak scanning was done globally for convenience. A stricter variant would scan on train only and drop the same columns from both splits.


## Approach:

## Representation A

# Curated Features
- Numeric: `CADD_PHRED`, `CADD_RAW`, `LoFtool`, `BLOSUM62`, allele frequencies
- Categorical: `Consequence`, `IMPACT`, `SYMBOL`
- Preprocessing:
  - Median imputation
  - Robust scaling
  - One hot encoding
    
 # Models:
  -Random Forest (primary)
  -Logistic Regression (baseline)

## Representation B — Text Baseline

- Character-level TF-IDF (2–4 grams)
- Text constructed from: `REF`, `ALT`, `Codons`, `Amino_acids`, `Consequence`, `IMPACT`
- Logistic Regression classifier

## Ablation Study

 Evaluated the effect of removing the `SYMBOL` (gene name) feature to test for gene-level shortcut learning.

## Key Results

## Model Performance (Test Set)

| Model        | ROC-AUC  | PR-AUC   | Balanced Acc | F1   |
| ------------ | -------- | -------- | ------------ | ---- |
| RF (curated) |   0.78   |  0.50    | 0.64         | 0.45 |
| LR (curated) | 0.73     | 0.43     | 0.60         | 0.40 |
| LR (text)    | 0.57     | 0.30     | 0.52         | 0.28 |

## Ablation (Random Forest)

| Model            | ROC-AUC   | PR-AUC    |
| ---------------- | --------- | --------- |
| With `SYMBOL`    | 0.775     | 0.496     |
| Without `SYMBOL` | 0.735     | 0.443     |

Removing `SYMBOL` causes a clear drop in performance, suggesting reliance on gene-level priors.

## Visuals

Saved in the `results/` folder:
- Confusion matrix (Random Forest, threshold = 0.5)
-  ROC curves (all models)
-  Precision-Recall curves
- Top-15 Random Forest feature importances

## Limitations

- Results are based on a single stratified train/test split, so performance may vary with a different split.
- No hyperparameter tuning or threshold optimization was performed, models mostly use default settings.
- The SYMBOL feature may act as a shortcut (gene-level prior), which could inflate performance and limit generalization to unseen genes.

## How to Run

1. Install dependencies- pip install -r requirements.txt
2. Download dataset -Download `clinvar_conflicting.csv` from Kaggle and place it in the project root.
3. Run the notebook- jupyter notebook clinvar_variant_classification.ipynb


## Future work

- To evaluate generalization by splitting train/test by gene (SYMBOL) rather than by random variants.
- Optimize decision thresholds based on precision–recall trade offs rather than using a fixed 0.5 threshold.
- Validate the model on a held-out ClinVar release or an external variant dataset to assess robustness.

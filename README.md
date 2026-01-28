# clinvar-variant-classification

# Problem Statement

Clinical variant interpretation is challenging due to conflicting annotations and heterogeneous feature types.
This project builds and compares machine-learning models to classify conflicting ClinVar variants as pathogenic or benign using curated annotations and a simple text baseline.

# Dataset

Source file : https://www.kaggle.com/datasets/kevinarvai/clinvar-conflicting

# Clinical Significance
- Clinical variant interpretation directly impacts patient diagnosis, risk assessment, and treatment decisions. Many variants encountered in practice are rare, newly discovered, or poorly characterized, making generalization beyond historically well-studied genes essential.
- Models that rely heavily on gene identity can appear highly accurate on benchmark datasets but fail to generalize to variants in unseen or less studied genes. This creates a risk of overconfident predictions driven by historical knowledge rather than the biochemical or functional impact of the specific mutation.
- This project explicitly examines this risk by evaluating gene-level shortcut learning, highlighting the importance of building models that learn variant-level signals rather than memorizing which genes are historically associated with disease.

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
  -Random Forest (primary)
  -Logistic Regression (baseline)

# Representation B — Text Baseline

- Character-level TF-IDF (2–4 grams)
- Text constructed from: `REF`, `ALT`, `Codons`, `Amino_acids`, `Consequence`, `IMPACT`
- Logistic Regression classifier

# Ablation Study

 Evaluated the effect of removing the `SYMBOL` (gene name) feature to test for gene-level shortcut learning.

# Gene-Level Shortcut Learning (Honest Critique)
- The ablation study revealed that model performance drops substantially when the SYMBOL (gene name) feature is removed. This suggests that the model is partially memorizing which genes are historically pathogenic (e.g., ATM, BRCA2) rather than learning only the biochemical or functional impact of individual variants.
- While gene identity can be informative in clinical practice, reliance on gene-level priors can inflate benchmark performance and limit generalization to variants in unseen or less studied genes. This highlights a key challenge in variant classification, distinguishing true variant level signal from historical annotation bias.
- By explicitly testing and reporting this behavior, the project emphasizes the importance of evaluating generalization beyond known genes when developing clinical variant interpretation models.

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
- The SYMBOL feature may act as a shortcut (gene-level prior), which could inflate performance and limit generalization to unseen genes.

# How to Run

1. Install dependencies- pip install -r requirements.txt
2. Download dataset: `clinvar_conflicting.csv` from Kaggle and place it in the project root.
3. Run the notebook- jupyter notebook clinvar_variant_classification.ipynb


# Future work

- To evaluate generalization by splitting train/test by gene (SYMBOL) rather than by random variants.
- Optimize decision thresholds based on precision–recall trade offs rather than using a fixed 0.5 threshold.
- Validate the model on a held-out ClinVar release or an external variant dataset to assess robustness.

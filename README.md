
PCOS Subtype Analysis Using Multi-View Clustering and Explainable AI

> Researcher: Riya Bhatta
> Dataset: NHANES 2017–2018 (Cycle J)  
> Topic:PCOS phenotype stratification using hormonal, metabolic, and environmental biomarkers

---

## Overview

Polycystic Ovary Syndrome (PCOS) affects 10–15% of women of reproductive age worldwide, yet it is frequently misdiagnosed or treated as a single uniform condition. This project applies an explainable machine learning framework to stratify PCOS patients into clinically distinct subtypes using population-level data from the National Health and Nutrition Examination Survey (NHANES 2017–2018, Cycle J).

The pipeline extends the landmark work of Dapas et al. (2022) in two key directions:

1. External validation — replicating their two-subtype hypothesis on an independent population cohort.
2. Novel third view — introducing environmental PFAS (Per- and Polyfluoroalkyl Substances) markers alongside hormonal and metabolic markers to test whether endocrine-disrupting chemical exposure contributes independent predictive signal to PCOS phenotype differentiation.

---

## Research Questions

| # | Research Question |
|---|-------------------|
| **RQ1** | Can a multi-view explainable ML framework — combining GMM clustering with XGBoost classification — stratify women in the NHANES 2017–2018 cohort into clinically distinct PCOS phenotypes, and at what level of diagnostic accuracy? |
| **RQ2** | What are the defining hormonal and metabolic biomarker profiles of the GMM-identified PCOS subtypes, and do they align with established clinical endocrinology? |
| **RQ3** | To what extent do SHAP values provide a biologically interpretable and clinically coherent explanation of XGBoost subtype predictions? |
| **RQ4** | Does PFAS environmental exposure contribute independent predictive signal to PCOS subtype classification beyond hormonal and metabolic markers, and is there a minimal biomarker subset sufficient for GP-level screening? |

---

## Pipeline Overview

```
NHANES .xpt Files
      │
      ▼
Data Merging (SEQN join) → Cohort Filter (women 18–45, n=1,232)
      │
      ▼
EDA → Preprocessing (Outlier Capping → KNN Imputation → Standard Scaling)
      │
      ▼
BIC Optimisation → GMM Clustering (n=2) → Subtype Labels
      │
      ▼
PFAS Integration (Environmental View) → 7-Feature Multi-View Dataset
      │
      ▼
XGBoost Classification → SHAP Explainability
      │
      ├── Multi-View Analysis (per-view XGBoost)
      ├── Minimal Subset Analysis (SHAP-ordered feature addition)
      ├── Learning Curve (best_epoch = 186)
      ├── TabNet (Deep Learning Second Opinion)
      ├── GridSearchCV (Hyperparameter Tuning)
      └── SMOTE (Class Imbalance Correction)
```

---

## Dataset

**Source:** [CDC NHANES 2017–2018 (Cycle J)](https://wwwn.cdc.gov/nchs/nhanes/continuousnhanes/default.aspx?BeginYear=2017)

| File | Contents | Key Variables |
|------|----------|---------------|
| `DEMO_J.xpt` | Demographics | `SEQN`, `RIDAGEYR` |
| `RHQ_J.xpt` | Reproductive health questionnaire | `SEQN` (cohort filter) |
| `BIOPRO_J.xpt` | Standard biochemistry panel | `LBXSGL` (Glucose), `LBXSTR` (Triglycerides) |
| `SSTST_J.xpt` | Testosterone & SHBG hormone levels | `SSTTSTO`, `SSTSHBG` |
| `PFAS_J.xpt` | PFAS environmental markers | `LBXNFOS`, `LBXNFOA`, `LBXPFNA` |

**Final cohort:** 1,232 women aged 18–45 · **Format:** SAS XPT (`.xpt`) · **Merged on:** `SEQN` (participant ID)

> All files are from the same survey cycle (NHANES 2017–2018, Cycle J) to ensure temporal consistency and valid participant-level merging with no cross-cycle confounding.

### Variable Reference

| Variable | Full Name | Clinical View |
|----------|-----------|---------------|
| `LBXSGL` | Blood Glucose | Metabolic |
| `LBXSTR` | Triglycerides | Metabolic |
| `SSTTSTO` | Total Testosterone | Hormonal |
| `SSTSHBG` | Sex Hormone Binding Globulin (SHBG) | Hormonal |
| `LBXNFOS` | Linear PFOS | Environmental (PFAS) |
| `LBXNFOA` | Linear PFOA | Environmental (PFAS) |
| `LBXPFNA` | PFNA | Environmental (PFAS) |

---

## Methods

### 1. Preprocessing
- **Outlier capping** at 99th percentile — removes NHANES fill-codes (e.g. 9999) without losing participants
- **KNN Imputation** (k=5) — preserves multivariate structure; applied to both primary and PFAS markers
- **Standard Scaling** (Z-score) — required for GMM, which is sensitive to feature scale

### 2. GMM Clustering
Gaussian Mixture Model with `n=2` components, confirmed by BIC optimisation. GMM assigns probabilistic cluster membership, better reflecting the clinical reality of PCOS as a spectrum rather than a hard binary.

### 3. XGBoost Classification
Primary supervised model trained on GMM cluster labels using all 7 multi-view features.

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| `learning_rate` | 0.05 | Gradual learning across 200 trees |
| `max_depth` | 3 | Prevents overfitting on n=1,232 |
| `min_child_weight` | 5 | Minimum 5 samples per leaf |
| `gamma` | 0.2 | Minimum loss reduction before split |
| `subsample` | 0.8 | 80% data per tree — reduces variance |
|`n_estimators` | 200 | Max trees; best epoch identified at 151 |


### 4. SHAP Explainability
SHAP (SHapley Additive exPlanations) values computed for every prediction. Two plots generated: mean absolute SHAP bar chart and directional beeswarm plot.

### 5. Multi-View Analysis
Four separate XGBoost models trained on each view independently (hormonal only, metabolic only, environmental only, all combined) to quantify the independent contribution of each view.

### 6. Minimal Subset Analysis
Features added one at a time in SHAP importance order. The flatten point — where incremental accuracy gain drops below 1% — identifies the minimal clinically actionable subset.

### 7. TabNet (Second Opinion)
Deep learning model with sequential attention used to validate XGBoost findings from an architecturally independent perspective.

### 8. SMOTE
Synthetic Minority Oversampling applied **only to training data** (after the train-test split) to address the 8:1 class imbalance without data leakage into the test set.

---

## Results

### GMM Subtype Profiles

Marker | Cluster 0 — Metabolic (~88%) | Cluster 1 — Reproductive (~12%) |
|--------|------------------------------|----------------------------------|
| Glucose (mg/dL) | 89.12 | 101.98 |
| Triglycerides (mg/dL) | 99.17 | 182.83 |
| Testosterone (ng/dL) | 113.40 | **1,319.06** ⬆ |
| SHBG (nmol/L) | 60.26 | **147.09** ⬆ |
| Size (n) | 1,085 (88%) | 147 (12%) |

**Cluster 0 (Metabolic):** Insulin resistance profile — elevated glucose and triglycerides, lower SHBG.  
**Cluster 1 (Reproductive):** Hyperandrogenism — dramatically elevated testosterone (~12× higher) and SHBG, consistent with classic androgenic PCOS.

### Classification Accuracy

 Model | Accuracy | Notes |
|-------|----------|-------|
| XGBoost (manual tuned) | 98.38% | `lr=0.05`, `best_epoch=151` |
| **XGBoost (GridSearchCV)** | **98.48%** | **Best — 5-fold stratified, 324 combinations** |
| XGBoost + SMOTE | 97.98% | Balanced training; **90% reproductive recall** |
| TabNet (`lr=0.001`, original) | 86.24% | Slowest convergence, ~40 epochs to plateau |
| TabNet (`lr=0.01`, selected) | 92.71% | Smooth convergence, `best_epoch=17` |
| TabNet (`lr=0.1`, tested) | 94.33% | Highest peak but unstable — not selected |

### SHAP Feature Importance

| Rank | Feature | Mean \|SHAP\| | Clinical Role |
|------|---------|--------------|---------------|
| 1 | SHBG | ~1.32 | **Primary separator** — high → reproductive, low → metabolic |
| 2 | Glucose | ~0.93 | High glucose confirms metabolic subtype |
| 3 | Testosterone | ~0.67 | High testosterone confirms reproductive subtype |
| 4 | Triglycerides | ~0.50 | Secondary metabolic signal |
| 5 | PFOA | ~0.27 | **Strongest environmental signal** |
| 6 | PFOS | ~0.11 | Minor environmental contribution |
| 7 | PFNA | ~0.06 | Negligible contribution |

### Multi-View Analysis

| View | Accuracy | Repro Recall | Key Finding |
|------|----------|-------------|-------------|
| Hormonal Only | 93.12% | 58.54% | Strongest single view |
| Metabolic Only | 88.66% | 34.15% | Strong independent signal |
| Environmental Only | 85.43% | 12.20% | PFAS carries genuine but secondary predictive power |
| **All Views Combined** | **96.36%** | **80.49%** | **Each view contributes non-redundant information** |

### Minimal Subset Analysis

| Features Used | Accuracy | Repro Recall | Incremental Gain |
|---------------|----------|-------------|-----------------|
| SHBG only | 90.69% | 43.90% | baseline |
| + Glucose | 93.52% | 63.41% | +2.83% |
| **+ Testosterone ← flatten point** | **95.95%** | **78.05%** | **+2.43%** |
| + Triglycerides | 95.95% | 75.61% | 0.00% |
| + PFOA | 96.76% | 80.49% | +0.81% |

**Minimal subset: SHBG + Glucose + Testosterone → 95.95% accuracy, only 0.81% below the full 7-marker model.**  
Clinical implication: a 3-marker routine blood test could screen PCOS subtype at GP level without genetic or specialist testing.

--

---

## Limitations

-  **Cross-sectional data:** NHANES captures a single time point — association can be demonstrated but not causation.
- **Circular validation:** GMM cluster labels used as XGBoost targets validates separability, not independent clinical ground truth.
- **PFAS missingness:** ~70% of PFAS values required KNN imputation — results should be validated on a dataset with complete PFAS measurements.
- **Single cohort:** Findings are specific to NHANES 2017–2018 — external validation on an independent PCOS clinical cohort is needed.
- **Dataset size:** n=1,232 limits TabNet performance and reduces statistical power for the minority reproductive subtype (n=147).

## Installation & Usage

### Requirements

```bash
pip install pandas numpy pyreadstat scikit-learn seaborn matplotlib \
            xgboost shap pytorch-tabnet imbalanced-learn
```

### Running the Notebook

1. Download the required NHANES Cycle J `.xpt` files from the [CDC NHANES website](https://wwwn.cdc.gov/nchs/nhanes/) and place them in the project root directory.
2. Open `Final_project.ipynb` in Jupyter Notebook or JupyterLab.
3. **Run cells top to bottom.** Section 13 (Learning Curve) must be run before Sections 9, 11, 12, and 16, as it defines `best_epoch = 186`.

### Required Data Files

Place these files in the same directory as the notebook:

```
DEMO_J.xpt
RHQ_J.xpt
BIOPRO_J.xpt
SSTST_J.xpt
PFAS_J.xpt
```

---

## Project Structure

```
├── Final_project.ipynb          # Main analysis notebook
├── DEMO_J.xpt                   # NHANES demographics
├── RHQ_J.xpt                    # Reproductive health questionnaire
├── BIOPRO_J.xpt                 # Biochemistry panel
├── SSTST_J.xpt                  # Testosterone & SHBG
├── PFAS_J.xpt                   # PFAS environmental markers
├── PCOS_Clustered_Results.csv   # Generated: GMM cluster labels
├── PCOS_Final_Thesis_Data.csv   # Generated: full 7-feature dataset
└── README.md
```

---

## Next Steps

| Priority | Method | Scientific Value |
|----------|--------|-----------------|
| 1 | **UMAP** | 2D projection — visual confirmation of subtype separation |
| 2 | **VAE (Variational Autoencoder)** | Generative model — latent space subtype visualisation |
| 3 | External cohort validation | Confirm findings generalise beyond NHANES |
| 4 | Complete PFAS dataset | Validate PFAS findings without relying on KNN imputation |


---

## References

1. Dapas, M. et al. (2022). Distinct subtypes of polycystic ovary syndrome with novel genetic associations. *Science Translational Medicine*, 14(668). DOI: [10.1126/scitranslmed.abh4314](https://doi.org/10.1126/scitranslmed.abh4314)

2. Lundberg, S.M. & Lee, S.I. (2017). A unified approach to interpreting model predictions. *Advances in Neural Information Processing Systems*, 30.

3. Arik, S.Ö. & Pfister, T. (2021). TabNet: Attentive interpretable tabular learning. *Proceedings of AAAI*. DOI: [10.1609/aaai.v35i8.16826](https://doi.org/10.1609/aaai.v35i8.16826)

4. CDC / NHANES (2018). National Health and Nutrition Examination Survey 2017–2018 (Cycle J). [https://wwwn.cdc.gov/nchs/nhanes/](https://wwwn.cdc.gov/nchs/nhanes/)

5. Chawla, N.V. et al. (2002). SMOTE: Synthetic minority over-sampling technique. *Journal of Artificial Intelligence Research*, 16, 321–357.

6. Chen, T. & Guestrin, C. (2016). XGBoost: A scalable tree boosting system. *Proceedings of KDD 2016*. DOI: [10.1145/2939672.2939785](https://doi.org/10.1145/2939672.2939785)

---

## Ethical Statement

- **No personal data:** NHANES uses SEQN participant IDs only. No names or direct identifiers are present.
- **Informed consent:** All NHANES participants gave written informed consent. Data is publicly available under the CDC open-data licence.
- **Anonymised:** NHANES applies differential privacy and suppression rules before public release.
- **Secure storage:** Raw `.xpt` files stored on a local encrypted drive and not redistributed.

---



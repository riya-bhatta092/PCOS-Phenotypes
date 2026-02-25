
# PCOS Phenotyping using Unsupervised Machine Learning
This project replicates the framework established by "Dapas et al. (2020)" to identify distinct metabolic and reproductive subtypes of PCOS using the "NHANES 2017-2018" national dataset.

# Overview
PCOS is a highly heterogeneous disorder. This research moves beyond "one-size-fits-all" diagnosis by using "Gaussian Mixture Models (GMM)" to stratify patients into clinical phenotypes for precision medicine.

# Methodology & Research Logic 
## Data Integration
Logic: Synchronized four primary NHANES modules—Demographics, Reproductive Health, Biochemistry, and Hormones—using the SEQN (Sequence Number) unique identifier.

Cohort: Isolated a refined cohort of 1,232 female participants within the reproductive age range (18–45).

# Clinical Data Pre-processing
Outlier Management (Winsorization): Implemented a 99th-percentile "cap" on clinical markers (Glucose, Triglycerides, Testosterone, and SHBG). This logic handles extreme outliers and potential NHANES fill-codes (e.g., values >2000 ng/dL) to ensure physiological realism in the final cluster profiles.
KNN Imputation: Utilized K-Nearest Neighbors (k=5) to resolve missingness, maintaining the biological covariance between markers.
Standardization: Applied Z-score normalization to ensure all clinical variables contribute equally to the distance-based clustering algorithm.

# Model Optimization & Validation
Clustering Algorithm: Gaussian Mixture Models (GMM).

Selection Logic: Evaluated model complexity using the Bayesian Information Criterion (BIC). While the BIC continues to decrease with higher cluster counts, the 2-cluster solution was selected for its clinical interpretability and alignment with established medical literature (Dapas et al.).

# Literature
- Dapas, M., et al. (2020): Identified "Metabolic" vs "Reproductive" subtypes using unsupervised clustering. 

# Future Roadmap
1. Predictive modeling: Training an XGBoost classifier to automate the assignment of new clinical data to identified subtypes.
2. Explainable AI: Implementation of SHAP values to determine the feature importance of each clinical marker.

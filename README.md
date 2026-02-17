
# PCOS Phenotyping using Unsupervised Machine Learning
This project replicates the framework established by "Dapas et al. (2020)" to identify distinct metabolic and reproductive subtypes of PCOS using the "NHANES 2017-2018" national dataset.

# Overview
PCOS is a highly heterogeneous disorder. This research moves beyond "one-size-fits-all" diagnosis by using "Gaussian Mixture Models (GMM)" to stratify patients into clinical phenotypes for precision medicine.

# Methodology & Work Completed
- Data Stitching: Integrated 4 NHANES modules (Demographics, Repro-Health, Biochemistry, Hormones) via `SEQN` Left-Join.
- Cohort Selection: Isolated 1301 female participants (Ages 18–45).
- Data Cleaning: Implemented KNN Imputation (k=5) to resolve clinical missingness while preserving biological realism.
- Validation: Generated a Pearson Correlation Matrix confirming a "+0.27 metabolic signal" (Glucose/Triglycerides).

# Literature
- Dapas, M., et al. (2020): Identified "Metabolic" vs "Reproductive" subtypes using unsupervised clustering. 

# Future Roadmap
1. Implement GMM Clustering to identify subtype groups.
2. Apply XGBoost & SHAP for Explainable AI (XAI) feature importance.

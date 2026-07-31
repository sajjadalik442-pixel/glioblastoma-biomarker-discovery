# Prediction of Glioblastoma Multiforme from Single-Cell RNA Transcriptomic Data

A machine learning pipeline that identifies a compact panel of gene-expression biomarkers capable of distinguishing Glioblastoma Multiforme (GBM) samples from healthy controls, using single-cell RNA transcriptomic data.

---

## 🧠 Project Overview

Glioblastoma Multiforme (GBM) is one of the most aggressive forms of brain cancer, and early, reliable detection remains a major clinical challenge. This project explores whether a small set of gene-expression biomarkers, derived from transcriptomic data, can be used to build a lightweight and interpretable machine learning classifier for GBM detection.

The goal was two-fold:

1. **Biomarker discovery** — identify a minimal, biologically meaningful set of genes that carry strong predictive signal for GBM, in collaboration with a bioinformatician.
2. **Model development** — train and validate a classifier on the discovered biomarkers, and assess whether it generalizes well enough to be a candidate for real-time transcriptomic screening.

---

## 📊 Data Sources

Data was assembled from public repositories and an internal TCGA cohort, then harmonized into a single analysis-ready dataset.

| Source | Description |
|---|---|
| TCGA data | Formatted TCGA cohort (provided via collaborator), combining healthy and GBM sample sheets |
| [GSE243682](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE243682) | All-diseased GBM dataset |
| [GSE232469](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE232469) | Controlled + diseased dataset |
| [GSE243682](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE243682) | Subset with 5 controls and additional diseased samples |
| [GSE207821](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE207821) | Dataset containing both control and diseased samples |

**Processing steps:**
- Gene IDs were converted to gene names for consistency across datasets.
- All datasets were merged with the TCGA cohort into a single finalized dataset.
- The combined dataset was reviewed and validated by a bioinformatician prior to ML analysis.
- Final dataset location: `searching_datasets/data found/processed2/finalized_data.csv`

---

## 🔬 Exploratory Data Analysis (EDA)

A full exploratory analysis was carried out before modeling, covering data quality, class balance, distributions, and outliers. The complete, interactive report is included in this repository:

📄 **`Glioblastoma multiforme EDA report_by_Sajjad Ali Khan.html`**
📈 **`analysis.xlsx`**

Key findings from the EDA:

- **No missing values** across all 14 candidate biomarker features.
- **All features are numeric (`int64`)**, requiring no type conversion.
- **Balanced target classes**: 21 healthy vs. 20 GBM samples.
- **Skewed feature distributions**: most biomarkers show a strong right skew (a large mass near zero with a long tail), motivating the use of scaling and a tree-based model that is robust to non-normal distributions.
- **Outlier check**: box-plot analysis of the biomarker panel confirmed the final 3-feature subset used in the model contains no problematic outliers.

<details>
<summary>📁 See analysis screenshots (click to expand)</summary>

All supporting screenshots from the analysis workflow are available in the [`progress/`](./progress) directory, including:

- Data types & missing-value checks
- Outcome variable distribution
- Feature distribution histograms (14 features)
- Outlier / box-plot analysis
- Biomarker feature selection based on bioinformatics input
- Model pipeline diagram
- Train / test / in-house evaluation results
- Final confusion matrix

</details>

---

## 🧬 Biomarker Discovery

In collaboration with a bioinformatician, an initial panel of **14 candidate genes** was selected based on known relevance to glioma biology:

```
AKT1, EGFR, TP53, CDKN2A, IDH1, MGMT, PTEN, TERT,
CDKN2B, IDH2, PROM1, CD68, PRR15, SLC25A41
```

After feature evaluation and model-driven selection, the panel was narrowed down to the **3 most predictive biomarkers**:

| Gene | Role in Glioma Biology |
|---|---|
| **AKT1** | Key regulator of the PI3K/AKT signaling pathway, frequently dysregulated in glioblastoma |
| **MGMT** | DNA repair gene; its expression/methylation status is a well-known clinical marker in GBM |
| **PTEN** | Tumor suppressor gene commonly lost or mutated in glioblastoma |

This reduced 3-gene panel retained strong predictive power while keeping the model simple, interpretable, and practical for potential real-time screening applications.

---

## ⚙️ Modeling Pipeline

The final pipeline combines standardization of the selected biomarkers with an **ExtraTrees Classifier**:

```
Pipeline
├── preprocessor (ColumnTransformer)
│   └── scaler: StandardScaler → ['AKT1', 'MGMT', 'PTEN']
└── ExtraTreesClassifier
```

**Model selection**

Multiple hyperparameter configurations were tested on the 3-feature dataset. The best-performing configuration (no grid search, no SMOTE, standard scaling) achieved:

| n_estimators | max_depth | min_samples_split | min_samples_leaf | criterion | Test AUC | Train AUC | In-house result |
|---|---|---|---|---|---|---|---|
| 10 | 12 | 13 | 3 | gini | **0.89** | 0.93 | **Excellent** |

This configuration was selected over alternatives that showed signs of bias (e.g., class-skewed predictions on the in-house set) despite comparable or higher train AUC.

---

## 📈 Results

### Train Data Evaluation
- **Accuracy:** 0.906
- **AUC:** 0.906

| Class | Precision | Recall | F1-score | Support |
|---|---|---|---|---|
| 0 (Healthy) | 0.88 | 0.94 | 0.91 | 16 |
| 1 (GBM) | 0.93 | 0.88 | 0.90 | 16 |

### Test Data Evaluation
- **Accuracy:** 0.889
- **AUC:** 0.90

| Class | Precision | Recall | F1-score | Support |
|---|---|---|---|---|
| 0 (Healthy) | 1.00 | 0.80 | 0.89 | 5 |
| 1 (GBM) | 0.80 | 1.00 | 0.89 | 4 |

### Final Confusion Matrix (In-house / Idiopathic Pulmonary Fibrosis validation set)

|  | Predicted 0 | Predicted 1 |
|---|---|---|
| **True 0** | 4 | 0 |
| **True 1** | 1 | 4 |

The model correctly classified 8 out of 9 in-house validation samples, with strong performance across both classes despite the small sample size.

---

## 🗂️ Repository Structure

```
.
├── data/
│   └── finalized_data.csv               # Final harmonized dataset used for ML
├── progress/                             # Screenshots documenting the full workflow
│   ├── initial_analysis_*.PNG            # Data types, NaNs, outcome distribution
│   ├── features_distribution.PNG         # Distribution of 14 candidate biomarkers
│   ├── outliers_analysis_*.PNG           # Outlier / box-plot analysis
│   ├── features_selection_based_on_biomarkers_*.PNG
│   ├── finalized_pipeline.PNG            # Model pipeline diagram
│   ├── finalized_train_data_evaluation.PNG
│   ├── finalized_test_and_in-house_data_evaluation.PNG
│   └── finalized_confusion_matrix.PNG
├── analysis.xlsx                         # Tabular analysis / hyperparameter search results
├── Glioblastoma multiforme EDA report_by_Sajjad Ali Khan.html   # Full interactive EDA report
└── README.md
```

---

## 🧭 Workflow Summary

1. Collected and formatted TCGA data (healthy + GBM sheets combined).
2. Downloaded and merged additional public GEO datasets (GSE243682, GSE232469, GSE207821).
3. Converted gene IDs to gene names and unified all sources into one dataset.
4. Validated the finalized dataset with a bioinformatician before ML analysis.
5. Performed exploratory data analysis: missing values, data types, class balance, distributions, and outliers.
6. Selected an initial 14-gene biomarker panel based on bioinformatics input.
7. Narrowed the panel down to 3 key biomarkers (AKT1, MGMT, PTEN) via feature selection.
8. Trained and tuned an ExtraTrees Classifier pipeline with standard scaling.
9. Evaluated the model on train, test, and an independent in-house dataset.

---

## 🚀 Future Work

- Expand the in-house validation cohort to strengthen generalizability claims.
- Explore additional model families (e.g., gradient boosting, logistic regression) as benchmarks against ExtraTrees.
- Investigate MGMT methylation status alongside expression for improved clinical relevance.
- Package the final pipeline into a simple screening tool/API for real-time transcriptomic testing.

---

## 👤 Author

**Sajjad Ali Khan**

---

## 📜 License

Apache 2.0

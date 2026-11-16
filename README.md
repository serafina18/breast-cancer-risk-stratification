# Breast Cancer Risk Stratification: Combining Histopathology and RNA-seq

Can combining routine histopathology with RNA-seq gene expression improve survival risk prediction in breast cancer, and help flag patients who may be under-treated?

**[Read the full report](https://serafina18.github.io/breast-cancer-risk-stratification/DATA3888_Report_Biomed17.html)**

## Overview

Treatment decisions in early-stage breast cancer rely on histopathological markers such as tumour grade, receptor status and tumour size. These markers don't fully capture a tumour's molecular biology, so some patients may receive treatment that doesn't match their actual risk.

Using **2,512 ER-positive, endocrine-treated patients** from the SCAN-B cohort (GSE96058), we tested whether adding six routine histopathological variables to gene expression data improves survival risk stratification. We then turned the final model into a **Shiny risk calculator** for clinicians.

## Key results

| Model | Mean C-statistic (SD) |
|---|---|
| **Cox-LASSO, genes + histopathology** | **0.766 (0.036)** |
| Random Survival Forest, genes + histopathology | 0.716 |
| Cox-LASSO, genes only | 0.695 (0.044) |
| Random Survival Forest, genes only | 0.674 |

- Adding histopathology **significantly improved** discrimination in both model families (paired t-test, p < 0.0001).
- Cox-LASSO outperformed Random Survival Forest in both configurations, and was chosen for deployment for its accuracy and interpretability.
- **Stability selection** identified an **11-gene panel**, including GREB1, a well-known estrogen-receptor signalling gene.
- The final model split patients into **three risk tiers** with sharply different observed mortality: **Low 1.2%, Medium 4.9%, High 20.2%** (log-rank p < 0.0001).

## Methods

- **Data:** GSE96058 (SCAN-B), filtered to ER-positive, endocrine-treated patients with overall survival follow-up. Six clinical variables: NHG grade, PgR status, HER2 status, tumour size, age and lymph node status.
- **Models:** Cox proportional hazards with LASSO regularisation and Random Survival Forest, each trained on genes only and on genes plus histopathology.
- **Evaluation:** nested cross-validation (5 repeats × 5 outer folds, 25 fits per model), measured with Harrell's C-statistic.
- **Gene panel:** stability selection across all 25 outer fits.
- **Deployment:** final Cox model refitted on all patients, with risk tiers set at the 33rd and 67th percentiles.

## Shiny risk calculator

Clinicians enter the six histopathological variables and upload a CSV with expression values for the 11 genes. The app returns:

- the patient's risk tier and percentile within the cohort
- personalised 1-, 3- and 5-year survival estimates
- the top three factors pushing the patient's risk up and down

It's designed as decision support for pathologist-to-oncologist reporting, not as a standalone diagnostic tool. The app source is in the report appendix.

## Repository contents

| File | Description |
|---|---|
| `DATA3888_Report_Biomed17.Rmd` | Full analysis and report (R Markdown) |
| `DATA3888_Report_Biomed17.html` | Rendered report |
| `cox_models.Rmd` | Cox-LASSO models, stability selection and Shiny app |

## Data

The data is publicly available from the NCBI Gene Expression Omnibus: [GSE96058](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE96058). It is not included in this repository.

## Limitations

The model is trained on a single Swedish cohort of ER-positive patients and has not been externally validated (e.g. on TCGA-BRCA or METABRIC). It should not be used for clinical decisions without that validation.

## Tech

R, R Markdown, `survival`, `glmnet` (Cox-LASSO), `randomForestSRC` (Random Survival Forest), stability selection, Shiny

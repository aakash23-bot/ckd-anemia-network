# Kidney-Driven Remodelling of Anaemia in US Adults (NHANES 2013–2018)

Survey-weighted analysis of how reduced kidney function reshapes anaemia prevalence, biomarker correlation structure and mortality in US adults. It is the companion to [`discordance`](https://github.com/aakash23-bot/discordance), which covers iron biomarker discordance.

## Overview

| Analysis | Method |
|---|---|
| Anaemia prevalence by eGFR | Survey-weighted means with 95% CI (R `survey`) |
| Biomarker correlation structure | Survey-weighted Pearson correlations, low vs high eGFR |
| Correlation rewiring | Spearman differential network with bootstrap CIs (1,000 resamples) |
| Anaemia classification | Logistic regression, random forest, XGBoost and SVM; 10-fold stratified CV; SHAP |
| Mortality | Kaplan–Meier by CKD × anaemia group (unadjusted) |
| Phenotype structure | t-SNE on core laboratory markers |

## Data sources

- **NHANES** 2013–14, 2015–16 and 2017–18 (public, de-identified), downloaded automatically from `wwwn.cdc.gov`
  - Demographics, BMX, BIOPRO, ALB_CR, CBC, GHB, GLU, lipids, CRP/HSCRP, ferritin, folate, B12, vitamin D, insulin
- **NCHS Linked Mortality Files**, with follow-up through 31 December 2019

No raw data are stored in this repository. The notebook builds `NHANES_CKD_GRAPH.csv` locally.

## Cohort

| Step | n |
|---|---|
| Linked NHANES 2013–2018 records | 29,400 |
| Adults ≥18 y | 17,961 |
| Excluding pregnant (RIDEXPRG = 1) | 17,771 |
| With serum creatinine (analytic sample) | **15,995** |
| — eGFR < 60 mL/min/1.73 m² | 1,251 |
| — eGFR ≥ 60 mL/min/1.73 m² | 14,744 |
| Deaths (MORTSTAT = 1) | 749 |

Columns with less than 20% completeness are dropped (65 retained).

## Definitions

| Variable | Definition |
|---|---|
| eGFR | CKD-EPI 2021, race-free |
| Low eGFR | eGFR < 60 mL/min/1.73 m² |
| CKD (mortality analysis) | eGFR < 60 **or** UACR ≥ 30 mg/g |
| Anaemia | Hb < 13 g/dL (men), < 12 g/dL (women), per WHO |
| Diabetes | HbA1c ≥ 6.5% |
| Elevated CRP | ≥ 3 mg/L (LBXCRP, or LBXHSCRP where unavailable) |
| BMI groups | < 25, 25–30, ≥ 30 kg/m² |
| Survey weight | WTMEC2YR / 3 for the pooled 6-year cycle; SDMVPSU and SDMVSTRA design, `nest = TRUE` |

## Key results

### Anaemia prevalence (survey-weighted, %)

| Subgroup | eGFR ≥ 60 | eGFR < 60 |
|---|---|---|
| Men | 3.5 | 25.7 |
| Women | 8.8 | 23.7 |
| NH Black | 17.6 | 41.1 |
| NH White | 4.0 | 19.6 |
| Male, diabetes | 7.7 | 33.3 |
| Female, diabetes | 10.8 | 34.9 |
| Male, CRP ≥ 3 | 6.0 | 32.5 |

Reduced eGFR is associated with roughly a three- to sevenfold higher anaemia prevalence across subgroups. The sex gap seen at normal eGFR largely disappears in low eGFR.

### Kaplan–Meier mortality (unadjusted)

| Group | 1-year | 3-year |
|---|---|---|
| Neither | 0.35% | 1.32% |
| Anaemia only | 1.35% | 4.30% |
| CKD only | 2.16% | 7.94% |
| CKD + anaemia | 5.35% | 18.25% |

These estimates are unadjusted and unweighted. They are descriptive only and should not be interpreted as causal or adjusted hazards.

## Repository structure

```
.
├── rescuing_egfr.ipynb   # full pipeline (Python + R cells)
├── README.md
└── .gitignore            # excludes *.csv, *.pdf, *.png
```

## Requirements

**Python 3.11:** pandas, numpy, requests, scikit-learn, xgboost, shap, networkx, scipy, matplotlib, seaborn, lifelines

**R 4.x:** survey, dplyr, ggplot2, corrplot

```bash
pip install pandas numpy requests scikit-learn xgboost shap networkx scipy matplotlib seaborn lifelines
```
```r
install.packages(c("survey", "dplyr", "ggplot2", "corrplot"))
```

## How to run

1. Open `rescuing_egfr.ipynb` in Google Colab or Jupyter.
2. Run the first Python cell. It downloads NHANES and mortality data and writes `NHANES_CKD_GRAPH.csv`.
3. Run the cohort cell, which builds the analytic sample and computes eGFR.
4. R cells need an R kernel, or `%load_ext rpy2.ipython` followed by `%%R` at the top of each R cell.
5. Run the remaining cells in order.

## Known limitations

- **Mixed kernels.** Python and R cells share one notebook and need rpy2 or a kernel switch.
- **Hard-coded plot data.** The forest-plot cell re-enters prevalence values by hand, and several are mistyped (for example, men with low eGFR is entered as 5.70 instead of 25.70). Read the values from `anaemia_by_egfr_stratified.csv` instead.
- **Mortality parsing.** Check the `read_fwf` column positions for `PERMTH_EXM` against the NCHS file layout. A truncated field gives follow-up times of at most 9 months.
- **Unadjusted survival.** Kaplan–Meier estimates ignore survey weights and confounders. Adjusted, weighted Cox models are planned.
- **CKD definition.** The prevalence analysis uses eGFR only, while the mortality analysis adds UACR ≥ 30. Group sizes are not directly comparable.
- **Cross-sectional design.** Associations between eGFR and anaemia are not causal.

## Authors

Aakash Adhya, Nilotpal Banerjee, Prathit Chatterjee\*

\*Corresponding author: prathit.chatterjee@ihub-data.iiit.ac.in

## Citation

> Adhya A, Banerjee N, Chatterjee P. *Context-Aware Hematological Intelligence: Iron Biomarker Discordance and Kidney-Driven Remodeling of Anemia in US Adults.* NCI Research Showcase 2027 (abstract).

## License

MIT

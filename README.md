# Telco Customer Churn Analysis

An end-to-end notebook project that explores historical customer churn, compares classification models, examines threshold and probability behaviour, and exports customer-level scores and summary tables for Power BI.

## Project objective

The analysis asks:

1. What share of customers churned in the historical dataset?
2. Which customer groups have different observed churn rates?
3. How well do candidate models identify churners on a held-out test set?
4. How do threshold choices change recall, precision, and the number of customers flagged?
5. How can actual outcomes, probabilities, predicted flags, and operational risk tiers be represented clearly in a dashboard?

The project is descriptive and predictive. It identifies associations in this dataset; it does not establish causes or prove that a retention action will reduce churn.

## Project files

- `Telco_Churn_Analysis_Final (1).ipynb` — companion executed notebook supplied separately. Keep it alongside the CSV when running it, or update its configured data path. This documentation bundle contains the three Markdown documents listed here.
- `README.md` — project overview and run instructions.
- `PROJECT_DATA_DICTIONARY.md` — exported fields, measures, and interpretation notes.
- `POWER_BI_BUILD_GUIDE.md` — report story, recommended pages, visuals, measures, and design guidance.

The notebook creates `churn_scored_data_v2.xlsx` with these sheets: `Scored_Customers`, `KPI_Summary`, `Segment_Summary`, `Model_Comparison`, `Threshold_Analysis`, `Risk_Summary`, and `Reconciliation_Log`.

## Dataset

The notebook is configured for the IBM Telco Customer Churn CSV named `WA_Fn-UseC_-Telco-Customer-Churn (1).csv`. The original notebook path is `/content/WA_Fn-UseC_-Telco-Customer-Churn (1).csv` for Google Colab. Change `DATA_PATH` in Section 1 to the location of your CSV if you are running locally. The dataset is not included in this documentation bundle.

The cleaning step converts `TotalCharges` to numeric. Its 11 blank values are associated with zero-tenure customers and are filled with zero, preserving all 7,043 records. The notebook also creates `ChurnFlag`, where 1 means the historical outcome is churn.

## Run the notebook

Use Python 3 with Jupyter Notebook, JupyterLab, or Google Colab. The notebook imports:

```text
pandas
numpy
matplotlib
seaborn
scikit-learn
xgboost
imbalanced-learn
openpyxl
```

Install missing packages in your environment, then:

1. Place the CSV where the notebook can access it and update `DATA_PATH`.
2. Run cells from top to bottom so that data cleaning, feature creation, model fitting, scoring, and export occur in order.
3. Locate `churn_scored_data_v2.xlsx` in the notebook's current working directory (or update `OUTPUT_PATH`).
4. Import the workbook into Power BI Desktop. See `POWER_BI_BUILD_GUIDE.md` before building visuals so test-set metrics are kept separate from full-customer scoring.

The notebook fixes the random seed at 42 and uses a stratified 80/20 split. Re-running with the same data and compatible package versions should make randomized steps reproducible, though package versions can affect exact results.

## Main results in the executed notebook

- **7,043** customers; **1,869** historical churners; **26.5%** actual churn rate.
- Highest-risk descriptive groups include month-to-month contracts (**42.71%** churn), tenure of 0–6 months (**52.94%**), fiber optic service (**41.89%**), and electronic check payment (**45.29%**).
- Logistic Regression is selected by the stated churn-recall criterion: test recall **0.794**, precision **0.510**, AUC **0.843**, and F1 **0.621**.
- XGBoost has the highest reported accuracy (**0.781**) and precision (**0.586**) of the compared models, and the lowest Brier score (**0.140**) in this evaluation. Because training used SMOTE, Brier score alone does not establish well-calibrated real-world probabilities.
- The high operational-risk tier contains **2,298** customers (**32.6%**); its observed churn rate is **57.57%** and mean predicted probability is **77.95%**. The risk thresholds are reporting bands, not validated intervention cut-offs.

See the notebook for the complete results and `PROJECT_DATA_DICTIONARY.md` for metric definitions.

## Method notes and limits

- Preprocessing is fitted on training data only; the test set is not oversampled.
- Standard SMOTE is applied after one-hot encoding, which can create fractional categorical indicators. A categorical-aware method such as SMOTENC is a follow-up improvement.
- Model evaluation uses one held-out split. Full-customer probabilities are for historical segmentation and dashboard demonstration, not independent performance claims.
- The 30%/60% operational risk tiers and the 50% classification flag answer different questions. Do not label a risk tier as confirmed future churn.
- The `AvgMonthlySpend` ablation showed only small differences on this split (recall 0.794 vs 0.791; AUC 0.843 vs 0.842). Treat that as preliminary evidence, not proof of a general feature effect.
- The source notebook includes a calibration plot on the test set. Use it as a diagnostic; do not fit a calibrator on that same test set and then describe the test as independent validation.

## Suggested presentation

Build the report as a short business story: **scale of churn → where it is concentrated → what the model can and cannot do → operational trade-offs → recommended next steps**. Avoid comparing actual churn rate directly with an average customer probability as if they were interchangeable. The full build plan and DAX measures are in [`POWER_BI_BUILD_GUIDE.md`](POWER_BI_BUILD_GUIDE.md).


---

## Author

**Irfan Moosa**

BSc Information Technology — Computer Science & Business Management

This project forms part of my data analytics and machine-learning portfolio.

# Customer Churn Analysis — Machine Learning & Customer Risk Scoring

## Overview

This project analyses customer churn using the IBM Telco Customer Churn dataset and develops an end-to-end machine-learning workflow for identifying customers who may require retention attention.

The project goes beyond simply training a classification model. It combines **data-quality assessment, exploratory data analysis, feature engineering, class-imbalance handling, model comparison, business-oriented model selection, customer risk scoring, and Power BI-ready output**.

The central business question is:

> **Which customers are most likely to churn, what characteristics are associated with churn, and which modelling approach provides the most useful predictions for retention prioritisation?**

The notebook is designed as a portfolio project demonstrating how a data analyst can move from raw customer data to an interpretable, actionable analytical output.

---

## Project Objectives

The project has five main objectives:

1. **Understand customer churn**
   - Identify patterns and customer characteristics associated with churn.
   - Compare churn rates across contract, service, billing and tenure segments.

2. **Prepare the data for machine learning**
   - Identify and resolve data-quality issues.
   - Engineer meaningful customer-level features.
   - Encode categorical variables and scale numerical variables appropriately.

3. **Develop and compare predictive models**
   - Train Logistic Regression, Random Forest and XGBoost classifiers.
   - Address class imbalance using SMOTE.
   - Evaluate models using recall, precision, F1-score, ROC-AUC and accuracy.

4. **Select a model based on a business objective**
   - Prioritise **recall for the churn class** because missing a genuine churner may be more costly than contacting a customer who ultimately remains.

5. **Translate predictions into a business-ready output**
   - Generate churn probabilities.
   - Assign customers to Low, Medium and High operational risk tiers.
   - Export results for further analysis and visualisation in Power BI.

---

## Dataset

The project uses the **IBM Telco Customer Churn** dataset.

The dataset contains **7,043 customer records** and includes information covering:

- Customer demographics
- Account and contract information
- Tenure
- Internet and additional services
- Payment methods
- Monthly charges
- Total charges
- Customer churn status

### Target Variable

`Churn`

- `Yes` = customer churned
- `No` = customer remained

For modelling, this is converted into:

`ChurnFlag`

- `1` = churn
- `0` = no churn

The overall churn rate in the analysed dataset is approximately **26.5%**.

---

# Analytical Workflow

```text
Raw Customer Data
       │
       ▼
Data Quality Assessment
       │
       ▼
Exploratory Data Analysis
       │
       ▼
Feature Engineering
       │
       ▼
Train / Test Split
       │
       ▼
Preprocessing
       │
       ▼
Class-Imbalance Handling
       │
       ▼
Model Training
       │
       ├── Logistic Regression
       ├── Random Forest
       └── XGBoost
       │
       ▼
Model Evaluation
       │
       ▼
Business-Based Model Selection
       │
       ▼
Customer Churn Probability
       │
       ▼
Risk Tier Classification
       │
       ▼
Excel / Power BI Output
```

---

# Exploratory Data Analysis

The EDA stage investigates how customer characteristics differ between churners and non-churners.

The analysis focuses on:

- Overall churn distribution
- Contract type
- Customer tenure
- Monthly charges
- Internet service
- Tech support
- Payment method
- Paperless billing
- Correlations between numerical variables

## Key EDA Findings

### 1. Contract Type

Contract type shows one of the largest differences in observed churn rates.

| Contract Type | Churn Rate |
|---|---:|
| Month-to-month | **42.7%** |
| One year | **11.3%** |
| Two year | **2.8%** |

Month-to-month customers therefore represent a particularly important segment for retention analysis.

### 2. Customer Tenure

Churn is concentrated more heavily among customers earlier in their relationship with the company.

| Customer Group | Median Tenure |
|---|---:|
| Churned | **10 months** |
| Retained | **38 months** |

This suggests that the early stages of the customer lifecycle may represent an important opportunity for proactive retention activity.

### 3. Monthly Charges

Churners have a higher median monthly charge than retained customers:

- Churned: **$79.65**
- Retained: **$64.43**

This demonstrates that churn is not simply concentrated among the lowest-paying customers.

### 4. Service and Billing Segments

Several categorical variables show meaningful differences in observed churn:

- Customers without tech support have a higher observed churn rate than customers with tech support.
- Electronic-check customers have the highest observed churn rate among the payment-method groups.
- Fiber-optic customers show a higher churn rate than DSL and customers without internet service.
- Paperless-billing customers have a higher observed churn rate than customers using paper billing.

These relationships are useful for segmentation and modelling, but they should **not be interpreted as proof of causation**.

---

# Feature Engineering

Three customer-level features are engineered to provide additional information for modelling.

### `TenureGroup`

Groups customers into meaningful relationship stages rather than treating tenure only as a continuous variable.

### `NumAddonServices`

Counts the number of selected optional services used by each customer.

This consolidates several individual service variables into a single measure of service engagement.

### `AvgMonthlySpend`

Calculates average monthly spending relative to customer tenure.

This provides an alternative representation of customer spending that is easier to compare across customers with different relationship lengths.

A separate `ChurnFlag` variable is created as the binary modelling target.

---

# Data Quality

The dataset is largely complete and contains no duplicate customer IDs or duplicate rows.

The primary data-quality issue is **11 blank `TotalCharges` values**. These records correspond to customers with zero tenure, making it reasonable to interpret the blanks as customers who have not yet accumulated charges.

The values are therefore converted to numeric form and filled with `0`.

The churn target is imbalanced, with approximately:

- **73.5% No Churn**
- **26.5% Churn**

Because of this imbalance, accuracy is not treated as the sole measure of model quality.

---

# Modelling Approach

Three classification algorithms are compared.

## Logistic Regression

Logistic Regression provides a relatively interpretable baseline and is useful when stakeholders need to understand how customer characteristics relate to predicted churn.

It is also well suited to producing churn probabilities that can be used for customer prioritisation.

## Random Forest

Random Forest is an ensemble of decision trees capable of modelling non-linear relationships and interactions between variables.

It provides a useful comparison against the linear Logistic Regression model.

## XGBoost

XGBoost is a gradient-boosting algorithm that builds decision trees sequentially, with later trees attempting to improve on previous errors.

It provides a more flexible non-linear model and allows the project to test whether increased model complexity improves predictive performance.

No model is assumed to be the winner before evaluation.

---

# Handling Class Imbalance

The churn class represents only around 26.5% of customers.

To reduce the risk of the models favouring the majority class, **SMOTE (Synthetic Minority Over-sampling Technique)** is applied to the training data.

The test set is deliberately left untouched so that final model evaluation reflects the original class distribution.

### Important methodological note

Standard SMOTE is applied after one-hot encoding in this implementation. Because one-hot encoded categorical variables are numerical indicator columns, interpolation can create fractional values within the encoded categorical feature space.

A stronger future implementation would use **SMOTENC** while categorical variables are still represented categorically, or compare SMOTE with model-level class weighting.

---

# Model Evaluation

The models are evaluated on the held-out test set using:

| Metric | Purpose |
|---|---|
| **Recall** | Measures how many actual churners are successfully identified |
| **Precision** | Measures how many customers flagged as churn risks actually churn |
| **F1-score** | Balances precision and recall |
| **ROC-AUC** | Measures overall ranking/discrimination ability across thresholds |
| **Accuracy** | Measures overall classification correctness |

## Why Recall Matters

The primary business objective is to identify as many potential churners as possible.

A **false negative** occurs when:

> A customer actually churns, but the model fails to identify them.

For a retention programme, these missed customers can represent lost revenue and lost opportunities to intervene.

A **false positive** occurs when:

> The model flags a customer as high risk, but the customer does not churn.

This may result in an unnecessary retention contact or incentive.

The appropriate balance depends on the cost of these two errors.

---

# Model Results

The latest test-set results are:

| Model | ROC-AUC | Precision | Recall | F1 | Accuracy |
|---|---:|---:|---:|---:|---:|
| **Logistic Regression** | **0.843** | 0.510 | **0.794** | **0.621** | 0.743 |
| Random Forest | 0.841 | 0.552 | 0.684 | 0.611 | 0.769 |
| XGBoost | **0.843** | **0.586** | 0.599 | 0.592 | **0.781** |

## Model Selection

The results demonstrate why model selection should be based on the business objective rather than simply selecting the model with the highest accuracy.

### Logistic Regression

- Highest churn recall: **79.4%**
- Highest churn F1-score: **62.1%**
- ROC-AUC: **0.843**

### Random Forest

- Recall: **68.4%**
- Accuracy: **76.9%**
- ROC-AUC: **0.841**

### XGBoost

- Highest accuracy: **78.1%**
- Highest churn precision: **58.6%**
- Recall: **59.9%**
- ROC-AUC: **0.843**

### Final Decision

**Logistic Regression is selected as the final scoring model because recall is the primary business metric.**

The model identifies approximately **79.4% of actual churners** in the test set.

This comes at the cost of lower precision, meaning that a proportion of customers flagged by the model will ultimately remain.

This is an intentional trade-off based on the assumption that **missing a genuine churner is more costly than contacting some customers who do not churn**.

The selection should not be interpreted as meaning Logistic Regression is universally better than XGBoost. If the business placed a greater cost on unnecessary retention interventions, the preferred model could change.

---

# Model Visualisations

The notebook includes several visualisations to communicate the analysis clearly.

### Figure 1 — Overall Customer Churn Distribution

Shows the proportion of customers who churned versus those who remained.

### Figure 2 — Churn Rate by Contract Type

Highlights the substantial difference in churn between month-to-month, one-year and two-year contracts.

### Figure 3 — Tenure Distribution by Churn Status

Shows how customer tenure differs between churners and retained customers.

### Figure 4 — Monthly Charges by Churn Status

Compares the distribution of monthly charges across churn groups.

### Figure 5 — Churn Rate Across Selected Service and Billing Features

Compares churn rates across internet service, tech support, payment method and paperless billing.

### Figure 6 — Correlation Matrix of Numeric Variables

Highlights relationships between numerical variables and identifies potential redundancy.

### Figure 7 — Confusion Matrices Across Churn Models

Shows true positives, true negatives, false positives and false negatives for each model.

### Figure 8 — ROC Curves for Model Comparison

Compares model discrimination across classification thresholds.

### Figure 9 — Top 15 XGBoost Feature Importances

Shows which encoded variables contribute most strongly to XGBoost's predictive process.

Feature importance is **not causal evidence** and does not indicate that a variable directly causes churn.

### Figure 10 — Customer Distribution by Churn Risk Tier

Shows the number of customers classified into each operational risk category.

---

# Customer Risk Scoring

After selecting Logistic Regression, the model generates a churn probability for each customer.

The probability is converted into three operational risk tiers:

| Risk Tier | Predicted Churn Probability |
|---|---:|
| **Low** | < 30% |
| **Medium** | 30% – <60% |
| **High** | ≥ 60% |

These thresholds are intended to make the continuous probability output easier for business users to interpret.

## Current Risk Distribution

The latest scoring output contains approximately:

| Risk Tier | Customers | Percentage |
|---|---:|---:|
| Low | **3,108** | **44.1%** |
| Medium | **1,637** | **23.2%** |
| High | **2,298** | **32.6%** |

The High-risk group provides a practical starting point for further retention analysis.

### Important interpretation

A churn probability is **not a guarantee that a customer will churn**.

It represents the model's estimated likelihood based on patterns learned from the historical training data.

The 30% and 60% thresholds are operational choices rather than statistically validated business boundaries. In a production environment, these thresholds should be calibrated using:

- Historical retention outcomes
- Customer lifetime value
- Cost of retention interventions
- Capacity of the retention team
- Cost of false negatives and false positives

---

# Power BI Integration

The notebook exports the analysis into an Excel workbook structured for downstream business intelligence.

The workbook contains four main sheets.

### `Scored_Customers`

Customer-level dataset containing:

- Customer ID
- Customer characteristics
- Tenure
- Charges
- Churn status
- Churn probability
- Risk tier

This sheet can support customer-level filtering and drill-down analysis.

### `KPI_Summary`

Contains high-level business metrics suitable for dashboard KPI cards.

Examples include:

- Total customers
- Churned customers
- Overall churn rate
- Average monthly charges
- Average tenure
- High-risk customer count

### `Segment_Summary`

Provides aggregated customer and churn information across key segments such as:

- Contract type
- Internet service
- Customer count
- Churn rate
- Average monthly charges

### `Model_Comparison`

Documents the performance of the three machine-learning models and provides transparency around the final model-selection decision.

---

# Business Recommendations

Based on the analysis, several areas are suitable for further investigation:

### 1. Prioritise early-tenure customers

Churners have substantially lower median tenure than retained customers. This suggests that the early customer lifecycle should receive particular attention.

### 2. Investigate month-to-month customers

The 42.7% observed churn rate makes month-to-month customers an important retention segment.

Potential strategies could include testing longer-term contract incentives, onboarding improvements or proactive customer engagement.

### 3. Investigate high-risk service segments

Customers without tech support and other high-churn service segments warrant further investigation.

The objective should be to understand **why** these customers churn before assuming that a particular service will reduce churn.

### 4. Investigate payment-method differences

Electronic-check customers have a substantially higher observed churn rate.

This may reflect differences in customer behaviour, engagement or other characteristics rather than the payment method itself causing churn.

### 5. Prioritise retention using predicted risk

Instead of treating every customer equally, the churn probability can be used to rank customers and focus retention resources where they are most likely to have an impact.

---

# Limitations & Future Improvements

This project is designed as a portfolio demonstration and has several methodological limitations that should be considered before production deployment.

## 1. SMOTE implementation

Standard SMOTE is applied after one-hot encoding.

A future version should investigate:

- `SMOTENC`
- Class-weighted models
- Alternative resampling strategies

## 2. Multicollinearity

`tenure` and `TotalCharges` have a correlation of approximately **0.83**.

Both are retained in the current model, so multicollinearity has not been eliminated.

For stronger Logistic Regression interpretability, future analysis could compare the current specification against one that removes one of these highly correlated variables.

## 3. Classification threshold

The reported precision and recall values use a **0.5 classification threshold**.

A production system should optimise the threshold according to the relative business costs of:

- Missing a churner
- Contacting a customer who would have stayed

## 4. Full-dataset scoring

The final model is used to generate risk scores for the full dataset.

Because many of these customers were part of the model's training data, these scores should not be treated as an independent evaluation of model performance.

For production use, a model would be trained using appropriate historical labelled data and then used to score genuinely new/current customers.

## 5. Association versus causation

The EDA identifies relationships between customer characteristics and churn.

It does **not** establish that changing a particular variable will cause churn to increase or decrease.

Retention strategies should therefore be validated through controlled business experiments or further causal analysis.

## 6. Model calibration

The project focuses primarily on classification performance.

A future production implementation should also evaluate whether predicted probabilities are well calibrated. A customer predicted at 70% risk should ideally correspond to an approximately 70% observed churn rate across a sufficiently large comparable population.

---

# Technologies & Libraries

### Programming Language

- Python

### Data Analysis

- pandas
- NumPy

### Visualisation

- Matplotlib
- Seaborn

### Machine Learning

- scikit-learn
- XGBoost
- imbalanced-learn / SMOTE

### Business Intelligence

- Microsoft Power BI
- Excel

### Development Environment

- Google Colab
- Jupyter Notebook

---

# Project Structure

A recommended GitHub repository structure is:

```text
customer-churn-analysis/
│
├── README.md
│
├── notebooks/
│   └── Churn_Analysis_GitHub_Ready.ipynb
│
├── data/
│   └── README.md
│
├── outputs/
│   └── churn_scoring_output.xlsx
│
├── powerbi/
│   └── customer_churn_dashboard.pbix
│
└── requirements.txt
```

> The raw dataset does not need to be committed to the repository if licensing, redistribution or repository-size considerations make that inappropriate. A data-source description and instructions for obtaining the dataset can be provided instead.

---

# How to Run the Notebook

## Option 1 — Google Colab

1. Open the notebook in Google Colab.
2. Upload the required dataset.
3. Run the notebook from top to bottom.
4. Review the EDA, model evaluation and risk-scoring outputs.
5. Export the generated Excel workbook for Power BI.

## Option 2 — Jupyter Notebook

Install the required dependencies:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn xgboost imbalanced-learn openpyxl
```

Then launch Jupyter:

```bash
jupyter notebook
```

Open:

```text
Churn_Analysis_GitHub_Ready.ipynb
```

and execute the cells sequentially.

---

# Portfolio Highlights

This project demonstrates practical skills relevant to **Data Analyst, Junior Data Analyst, Data Scientist and Business Intelligence roles**, including:

- Data cleaning and validation
- Exploratory data analysis
- Feature engineering
- Statistical interpretation
- Classification modelling
- Imbalanced-data handling
- Model evaluation
- Business-oriented metric selection
- Predictive customer segmentation
- Risk scoring
- Data visualisation
- Excel reporting
- Power BI integration
- Translating technical results into business recommendations

The project particularly demonstrates the ability to connect **technical modelling decisions with business objectives** rather than evaluating models purely on algorithmic performance.

---

# Final Takeaway

The main outcome of this project is not simply identifying the model with the highest accuracy.

The analysis demonstrates an end-to-end process for turning customer data into a practical retention-support tool.

The final workflow:

**identifies churn patterns → engineers useful customer features → compares multiple predictive approaches → selects Logistic Regression based on churn recall → generates customer-level risk probabilities → categorises customers into operational risk tiers → prepares the results for Power BI.**

This approach provides a foundation for a more advanced production system in which model thresholds, probability calibration, customer value and retention-intervention costs could be incorporated into a formal decision framework.

---

## Author

**Irfan Moosa**

BSc Information Technology — Computer Science & Business Management

This project forms part of my data analytics and machine-learning portfolio.

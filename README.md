# CSE437 — 30-Day Repeat Purchase Prediction from Online Retail II

## Project overview

This project predicts whether a customer will make another purchase within **30 days** using historical purchasing behavior from the **UCI Online Retail II** dataset.

The modeling unit is a **customer × prediction-date** observation. For each monthly prediction date, customer behavior from the previous **180 days** is used to create predictive features, while purchases in the following **30 days** are used only to define the target.

The project compares two model families:

- Logistic Regression
- Random Forest

The primary evaluation metric is **F1-score**, with accuracy, precision, recall, ROC-AUC, and PR-AUC reported as complementary metrics.

## Problem statement

The goal is to determine whether a customer is likely to return and purchase again within 30 days. This can help identify customer behavior associated with repeat purchasing and provide a basis for prioritizing retention or marketing activity.

The feature construction is designed to avoid target leakage: predictors are calculated only from information available before the prediction date, while the following 30-day period is reserved for target construction.

## Dataset

**Dataset:** Online Retail II  
**Source:** UCI Machine Learning Repository  
**Dataset page:** https://archive.ics.uci.edu/dataset/502/online%2Bretail  
**DOI:** 10.24432/C5CG6D  
**License:** Creative Commons Attribution 4.0 International (CC BY 4.0)

The raw dataset contains:

- **1,067,371 transaction rows**
- **9 original columns**
- **5,942 identified customers**
- **43 countries**
- Date range: **2009-12-01 to 2011-12-09**

The original workbook contains two sheets. They are combined during the data-audit/preprocessing workflow, and a `SourceSheet` field is added to preserve the originating sheet.

See [`data/README.md`](data/README.md) for source and download instructions.

## Target variable

The target is:

`return_30d`

Interpretation:

- `0` — the customer does not make a purchase in the next 30 days.
- `1` — the customer makes at least one purchase in the next 30 days.

The customer-period dataset contains **61,852 observations**, with approximately **72.56% negative** and **27.44% positive** target values.

## Research questions

1. **Can we predict whether a customer will purchase again within 30 days?**
2. **Which customer behaviors are most important for predicting repeat purchases?**
3. **How does feature selection affect model performance?**


## Notebook workflow

Run the notebooks in **numbered order**, from top to bottom, on a fresh kernel with outputs saved.

### 1. Data audit and EDA

`notebooks/01_data_audit_and_eda.ipynb`

Covers data loading, sheet combination, data types, missing values, duplicates, cancellations, invalid values, outlier inspection, descriptive statistics, and exploratory plots.

### 2. Preprocessing

`notebooks/02_preprocessing.ipynb`

Covers duplicate removal, purchase-specific filtering, missing-customer handling, transaction-value construction, outlier flagging, and preprocessing decisions.

### 3. Feature engineering

`notebooks/03_feature_engineering.ipynb`

Constructs customer-level features using a **180-day lookback window** and a **30-day future target window**.

Main features include:

- `RecencyDays`
- `TransactionCount`
- `PurchaseDays`
- `TotalSpend`
- `AverageOrderValue`
- `TotalQuantity`
- `UniqueProducts`
- `ActiveMonths`
- `AvgItemsPerTransaction`
- `PurchaseDaysPerMonth`
- `Country`

The notebook also evaluates PCA and training-only feature selection.

### 4. Modeling and tuning

`notebooks/04_modeling_and_tuning.ipynb`

Compares Logistic Regression and Random Forest.

The final chronological design uses:

- 70% training period
- 15% validation period
- 15% held-out test period

Hyperparameter tuning uses **GridSearchCV with 4-fold TimeSeriesSplit**, with **F1-score** as the scoring function.

### 5. Evaluation and error analysis

`notebooks/05_evaluation_and_error_analysis.ipynb`

Reports held-out test performance, confusion matrices, ROC/PR curves, Random Forest feature importance, concrete error examples, and answers to the three research questions.

## Leakage control

Feature values are constructed only from historical transactions in the preceding 180 days.

The following 30 days are used to define `return_30d` and are not used as predictor information.

Imputation, scaling, and categorical encoding are placed inside the modeling pipeline so fitted preprocessing parameters are learned from training data rather than future observations.

## Main result

On the held-out test period:

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC | PR-AUC |
|---|---:|---:|---:|---:|---:|---:|
| Majority baseline | 0.6781 | 0.0000 | 0.0000 | 0.0000 | — | — |
| Logistic Regression | 0.6983 | 0.5298 | 0.5589 | 0.5439 | 0.7193 | 0.6000 |
| Random Forest | **0.7059** | **0.5427** | 0.5482 | **0.5455** | **0.7206** | **0.6047** |

Random Forest has the best held-out **F1-score (0.5455)**, although the difference from Logistic Regression is small.

## Reproducibility

1. Clone the repository.
2. Install dependencies from `requirements.txt`.
3. Download Online Retail II from the UCI source listed in [`data/README.md`](data/README.md).
4. Place the original workbook under `data/raw/`.
5. Open the notebooks in `notebooks/`.
6. Run notebooks `01` through `05` from top to bottom on a fresh kernel.
7. Keep generated processed data, models, figures, and report outputs in their designated directories.
8. Use repository-relative paths only.

## Dependencies

The notebooks use common Python data-science libraries including:

- pandas
- numpy
- matplotlib
- seaborn
- scikit-learn
- scipy
- openpyxl
- joblib


## Report

The final report is stored in:

- [`report/report.md`](report/report.md)
- [`report/report.pdf`](report/report.pdf)


## Authors

| Member | Student ID |
|---|---|
| Sahir Jawad Chowdhury | 23301212 |

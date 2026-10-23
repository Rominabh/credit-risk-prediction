# Credit Risk Prediction Project

A portfolio project that predicts which borrowers are likely to default on a loan and turns those predictions into a lending decision. The pipeline runs from raw data to an interactive dashboard, using SQL, Python and Power BI.

## Key Results

| | |
|---|---|
| Model | Random Forest, AUC 0.87 (Logistic Regression baseline: 0.82) |
| Optimal decision cutoff | 0.60, chosen by minimising total business cost |
| Cost reduction | 41% lower total cost than approving every applicant |
| Defaults caught at optimal cutoff | 61%, while rejecting under 10% of good customers |
| Risk concentration | High risk segment is 13% of borrowers but holds 61% of all defaults |

## The Problem

A lender has to decide which applicants to approve, and both kinds of mistakes cost money. Approving a borrower who later defaults leads to a loss on the loan. Rejecting a borrower who would have paid back means lost interest income. The goal of this project is not only to predict default, but to find the decision rule that keeps the total cost as low as possible.

The target is whether a borrower becomes 90 or more days late on a payment within two years, using a public dataset of 150,000 borrowers.

## What I Did

**Data cleaning (SQL):** I loaded the raw data into a SQLite database and used SQL queries to check every column for missing values and anomalies. Monthly income was missing for about 20% of borrowers. The late payment columns contained the values 96 and 98, which turned out to be placeholder codes rather than real counts, so I converted them to null. I also removed borrowers under 18 or over 100, since those records are not realistic for a lending dataset.

**Feature engineering and modelling (Python, scikit learn):** Instead of filling missing income with an average, I added a flag column `income_is_missing` and filled the income itself with zero, so the model knows both the value and the fact that it was missing. I engineered three new features: total number of late payments across all time windows, a flag for whether the borrower was ever 90 or more days late, and monthly income per dependent. Both models used balanced class weights to handle the imbalanced target (about 7% defaults). The Random Forest reached an AUC of 0.87 on the test set, compared with 0.82 for the Logistic Regression baseline.

**Business decision layer:** Instead of using the default cutoff of 0.5, I tested every cutoff from 0.05 to 0.95 and assigned a cost to both types of error:

* Missed default (approved borrower who defaults): assumed loss of 1,000
* Rejected good customer: assumed lost margin of 150

The total cost curve has a clear minimum at a cutoff of 0.60, and stays fairly flat between 0.55 and 0.65, so the choice is stable. At that point the model reduces total cost by 41% compared with approving everyone, and by about 5% compared with the default cutoff of 0.5. The cost figures are illustrative; a lender would plug in its own loss and margin numbers.

**Dashboard (Power BI, DAX):** A two page interactive report built on the test set (37,497 borrowers not used for training), so every number reflects performance on unseen data.

## Dashboard

### Model Overview
Validates the model in business terms. Actual default rates rise sharply with predicted risk: high risk borrowers default 26 times more often than low risk ones (31.6% vs 1.2%). The high risk segment is only 13% of borrowers but contains 61% of all defaults. Default risk also falls steadily with age.

![Model Overview](model_overview.png)

### Threshold Decision Analysis
Shows the trade off between missed defaults and rejected good customers, with the optimal cutoff marked. A user can select any cutoff and immediately see its total cost, the extra cost compared with the optimum, and the share of defaults caught.

![Threshold Decision Analysis](threshold_decision_analysis.png)

**DAX techniques used:** calculated columns for risk segments and age groups (with custom sort order), `CALCULATE` with `ALL` to compute lift against the overall default rate, `VAR` and `FILTER` to find the optimal cutoff, `SELECTEDVALUE` for the interactive cutoff selection, and edited visual interactions so the trade off curve stays visible while the slicer drives the KPI cards.

## Limitations and Next Steps

* **Probability calibration:** balanced class weights inflate predicted probabilities, so the scores should be read as a risk ranking rather than true default probabilities. Calibration would fix this and shift the optimal cutoff lower.
* **Threshold selection:** the cutoff was chosen on the test set; a separate validation set or cross validation would give a more robust estimate.
* **Fairness and regulation:** the model uses age. Credit scoring is classified as high risk under the EU AI Act, so a production model would need fairness checks across groups, documentation, and explainability (for example SHAP values).
* **Dashboard:** replace the fixed cost assumptions with What if parameters so a lender can enter its own figures.
* **Data:** the dataset is from 2011 and one market; a real model would need retraining and drift monitoring.

## Tools and Skills

SQL, SQLite, Python (pandas, NumPy, scikit learn), Power BI, DAX, Power Query, predictive modelling, feature engineering, data cleaning, cost based decision analysis, dashboard design.

## How to Run

1. Download `cs-training.csv` from the [Give Me Some Credit](https://www.kaggle.com/c/GiveMeSomeCredit/data) competition on Kaggle and rename it to `credit_risk_training.csv`.
2. Install the requirements: `pip

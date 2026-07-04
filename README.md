# Credit Risk Prediction Project

A personal portfolio project where I built a full pipeline from raw data to an interactive dashboard, to predict which borrowers are likely to default on a loan. I used SQL, Python, and Power BI together, and went beyond just building a model by adding a business decision layer that shows how a lender could actually use the results.

## Overview

The goal is to predict whether a borrower is likely to be seriously late on payments (90 days or more) within two years, using a public dataset of 150,000 borrowers. Instead of stopping at a single accuracy metric, the project shows what the model actually means for a real lending decision.

## What I Did

**Data cleaning (SQL):** I loaded the raw data into a SQLite database and used SQL queries to check all columns for missing values and anomalies. Monthly income was missing for about 20% of borrowers. The late payment columns contained values of 96 and 98 which turned out to be placeholder codes rather than real counts. I also filtered out any borrowers under 18 or over 100 since those records are not realistic for a lending dataset. The placeholder values were converted to null in the clean table so the model would not treat them as real numbers.

**Feature engineering and modeling (Python, scikit learn):** For missing income values, instead of filling them with an average, I created a flag column called `income_is_missing` that marks which rows had no income data, then filled the income column with zero. This way the model receives two pieces of information: the filled value and the fact that it was originally missing. The late payment nulls were filled with zero since no record of late payments likely means none occurred. I then engineered three new features: total number of late payment instances across all time windows, a binary flag for whether the borrower was ever 90 or more days late, and monthly income per dependent. Two models were trained and compared. A Logistic Regression baseline reached an AUC of 0.82, and a Random Forest model improved this to 0.87.

**Business decision layer:** Instead of stopping at model accuracy, I built a threshold sensitivity analysis that tests every possible decision cutoff. For each cutoff, it calculates how many real defaults are missed, how many good customers are rejected, and the estimated financial cost of missed defaults. This produces a trade off curve that a lender can use to choose the right cutoff based on their own risk appetite.

**Dashboard (Power BI, DAX):** I built a two page interactive dashboard. DAX measures and calculated columns were used to create risk segments and age groups, and to compute summary KPIs across the dataset.

## Dashboard

### Model Overview
Shows total borrowers, actual versus predicted default rate, default rate by age group, and a risk segment breakdown. Confirms the model is working sensibly: borrowers the model marks as High risk genuinely default more often in the real data.

![Model Overview Dashboard](model_overview.png)

### Threshold Decision Analysis
An interactive page where you select a single decision threshold from a list. The estimated financial cost of missed defaults updates instantly for that cutoff, while the full trade off curve stays visible for context.

![Threshold Decision Analysis Dashboard](threshold_decision_analysis.png)

## Tools and Skills

SQL, SQLite, Python (pandas, numpy, scikit learn), Power BI, DAX, predictive modeling, feature engineering, data cleaning, dashboard design.

## Dataset

The dataset is the public "Give Me Some Credit" dataset, originally released as part of a 2011 Kaggle competition. It contains anonymized financial records for 150,000 borrowers.

## Notebook

The full analysis from data loading through model training and export is available in `credit_risk_project_clean.ipynb`.

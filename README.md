# Fraud Detection in Online Transactions

## Introduction
The rapid rise of digital payments and e-commerce has fueled a surge in online transactions, creating new opportunities for fraudsters to exploit vulnerabilities like identity theft and phishing. With fraudulent transactions costing billions annually, robust fraud detection is critical to safeguard financial systems and maintain consumer trust in a cashless society.

## Objective
To develop a machine learning-based fraud detection model that accurately identifies fraudulent transactions, minimizing financial losses and enhancing security for companies and consumers. This project outlines the steps taken to build the model, including:

- **Data Sources and Definitions**
- **Patterns, Trends, and Insights**
- **Predictive Modeling**
- **Recommendations**

**Tools Used**: Jupyter Notebook (Python), SQL, Tableau

**Success Criteria**:
- High recall to identify most fraud cases.
- High precision to avoid flagging legitimate transactions.
- F1 Score (harmonic mean of precision and recall) ≥ 70%.

## Background
Before pursuing the Data Analytics Bootcamp (DAB), I worked as an Operations Executive in the insurance industry for five years. My role involved ensuring accurate policy issuance, verifying data integrity, participating in User Acceptance Testing (UAT), and streamlining processes. Collaborating with IT and Business Analysts to enhance systems and resolve defects deepened my interest in data, motivating me to take the DAB course to advance my skills in data analytics and machine learning.

## Data Sources and Definitions
The model was trained on a synthetic dataset from [PaySim on Kaggle](https://www.kaggle.com/datasets/ealaxi/paysim1), mimicking one month of online transactions with approximately **6.3 million rows**. The dataset is highly imbalanced, with only **8,213 fraud cases (~0.13% fraud rate)**. It includes transaction details such as amount, account balances, and types (e.g., CASH_OUT, TRANSFER).

### Data Cleaning
- Removed errors, duplicates, and incomplete records to ensure reliability.

### Data Dictionary (Simplified)
| Feature | Description |
|---------|-------------|
| `step` | Hour of simulation (1–744, representing transaction time) |
| `type` | Transaction type (e.g., CASH_OUT, TRANSFER, DEBIT, PAYMENT) |
| `amount` | Transaction amount (USD) |
| `nameOrig` | Sender’s account identifier |
| `oldbalanceOrg` | Sender’s balance before transaction (USD) |
| `newbalanceOrig` | Sender’s balance after transaction (USD) |
| `nameDest` | Recipient’s account identifier |
| `oldbalanceDest` | Recipient’s balance before transaction (USD; 0 for merchants) |
| `newbalanceDest` | Recipient’s balance after transaction (USD; 0 for merchants) |
| `isFraud` | Fraud status (1 = fraud, 0 = legitimate) |
| `isFlaggedFraud` | System-flagged potential fraud (1 = flagged, 0 = not flagged) |
| `days` | Day of the month (1–31) |
| `days_binned` | Grouped days for analysis (e.g., 0–5, 5–10) |
| `is_merchant_dest` | Recipient is a merchant (1 = yes, 0 = no) |
| `balance_change_orig` | Change in sender’s balance (oldbalanceOrg - newbalanceOrig) |
| `balance_change_ratio_orig` | Ratio of balance change to sender’s original balance |
| `is_account_drained` | Sender’s account emptied (1 = yes, 0 = no) |
| `real_balance` | Adjusted balance change for negative values |
| `type_CASH_OUT`, `type_TRANSFER`, etc. | One-hot-encoded transaction types |
| `amount_log`, `oldbalanceOrg_log`, etc. | Log-transformed features to reduce skewness |

## Patterns, Trends, and Insights
- **Dataset Overview**: 6,362,620 transactions across 11 columns, with only 8,213 fraud cases (~0.13% fraud rate), making it highly imbalanced. The target variable is `isFraud`.
- **Merchant Transactions**: No fraud cases involved merchants (`is_merchant_dest = 0`).
- **Transaction Types**: Fraud occurred only in `CASH_OUT` and `TRANSFER` transactions.
- **Transaction Amount**: Using a base-10 logarithmic scale, fraud transactions typically ranged from $10^5$ to $10^7$, while non-fraud transactions had a wider spread.
- **Account Drained**: Fraud cases were significantly associated with accounts drained to $0.
- **Real Balance**: Fraudulent transactions often showed negative balance changes in the range of -$10^5$ to -$10^7$.
- **Time Analysis**: Converted `step` to `days` and binned into ranges (e.g., 0–5, 5–10) to analyze transaction patterns over the month.
- **Tableau Dashboard**: Created an interactive dashboard to visualize fraud frequency by day and transaction type percentages.

## Predictive Modeling
### Success Metrics
- **Precision**: Minimize false positives to avoid flagging legitimate transactions.
- **Recall**: Maximize detection of fraud cases.
- **F1 Score**: Target ≥ 70% as the harmonic mean of precision and recall.

### Approach
- **Train-Test Split**: 80–20 split with `class_weight = balanced` to address dataset imbalance.
- **Models**: Logistic Regression (baseline) and Random Forest.
- **Threshold Optimization**: Adjusted thresholds to optimize F1 Score for the imbalanced dataset.

#### Logistic Regression Model 1
- **Features**: `amount`, `oldbalanceOrg`, `newbalanceOrig`, `oldbalanceDest`, `newbalanceDest`, `days`, `balance_change_orig`, `is_merchant_dest`, `real_balance`, `balance_change_ratio_orig`, `is_account_drained`, `type_CASH_OUT`, `type_TRANSFER`.
- **Threshold Optimization**: Used precision-recall curve to maximize F1 Score.
- **Results**:
  - Optimal Threshold: 0.9990
  - Precision: 74.82%, Recall: 51.73%, F1-Score: 61.17%
  - Confusion Matrix: [[1270595, 286], [793, 850]]
  - **Analysis**: High threshold (99.9%) improved precision but missed many fraud cases (low recall).

#### Logistic Regression Model 2
- **Adjustment**: Lowered threshold to 0.9 (90%) to improve recall.
- **Results**:
  - Manual Threshold: 0.9000
  - Precision: 16.20%, Recall: 79.67%, F1-Score: 26.92%
  - Confusion Matrix: [[1264108, 6773], [334, 1309]]
  - **Analysis**: High recall (79.67%) caught 1,309 of 1,643 fraud cases but low precision led to 6,773 false positives, risking customer churn.

#### Random Forest
- **Features**: `amount_log`, `oldbalanceOrg`, `newbalanceOrig`, `oldbalanceDest_log`, `newbalanceDest_log`, `days`, `balance_change_orig_log`, `is_merchant_dest`, `real_balance`, `balance_change_ratio_orig`, `is_account_drained`, `type_CASH_OUT`, `type_TRANSFER`.
- **Log Transformation**: Applied to reduce skewness in `amount`, `oldbalanceOrg`, `newbalanceOrig`, `oldbalanceDest`, `newbalanceDest`, `balance_change_orig`.
- **Results**:
  - Optimal Threshold: 0.2900
  - Precision: 91.58%, Recall: 86.73%, F1-Score: 89.09%
  - Confusion Matrix: [[1270750, 131], [218, 1425]]
  - **Analysis**: Caught 1,425 of 1,643 fraud cases with only 131 false positives, significantly outperforming logistic regression by balancing high precision and recall.

## Limitations
- **Dataset Scope**: Limited to 31 days, missing seasonal or time-series patterns.
- **Fraud Types**: Focuses only on abnormal transaction detection, not other fraud types.
- **Assumption**: Assumes company datasets are similar to PaySim.
- **Imbalance**: High ratio of legitimate to fraudulent transactions.
- **Missing Data**: Lacks features like transaction location, which could reveal geographic fraud trends.

## Recommendations
- **Adopt Random Forest Model**: Achieved 91.58% precision, 86.73% recall, and 89.09% F1 Score, catching 86% of fraud cases with minimal false positives (131 errors).
- **Focus on High-Risk Transactions**: Prioritize monitoring `CASH_OUT` and `TRANSFER` transactions, as all fraud cases occurred in these types.
- **Incorporate Location Data**: Collect and analyze transaction location to identify geographic fraud patterns.
- **Implementation**: Deploy the Random Forest model to reduce fraud losses, protect customers, and enhance trust in digital payments.

## Future Work
- Expand dataset duration to capture seasonal trends.
- Include additional features like transaction location and device information.
- Explore advanced techniques (e.g., ensemble methods, neural networks) to further improve model performance.

## Tools and Technologies
- **Python**: Data processing, feature engineering, and modeling (Jupyter Notebook).
- **SQL**: Data querying and management.
- **Tableau**: Interactive visualizations and dashboards.
- **Libraries**: Pandas, Scikit-learn, NumPy, Matplotlib, Seaborn.

## Repository Structure

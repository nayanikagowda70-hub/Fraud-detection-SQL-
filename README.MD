Fraud Detection Using SQL

This project looks at patterns of fraudulent behaviour in an online payments dataset.
The goal was to analyse transaction activity, spot risky behaviour, and build simple fraud insights using SQL.

The work was done in MySQL Workbench using a dataset of 40,000 online payment records.

Project Goals

Look at how transactions behave across different payment types

Find unusual or suspicious activity

Measure how much fraud appears in the dataset

Identify accounts and transactions that look risky


ataset Information

Each transaction contains:

step – time index (simulation hour)

type – PAYMENT, TRANSFER, CASH_OUT, CASH_IN

amount

account origin balances (oldbalanceOrg, newbalanceOrig)

account destination balances (oldbalanceDest, newbalanceDest)

isFraud – actual fraud (1 = fraud, 0 = normal)

isFlaggedFraud – flagged by system

This data is publicly available and often used for fraud modelling.


Key SQL Queries Used

```sql
USE fraud_detection;

-- Count total rows
SELECT COUNT(*) 
FROM transactions;

-- Preview the table
SELECT *
FROM transactions
LIMIT 10;

-- Overall fraud rate
SELECT
    COUNT(*) AS total_transactions,
    SUM(CASE WHEN isFraud = 1 THEN 1 ELSE 0 END) AS fraud_transactions,
    ROUND(
        100.0 * SUM(CASE WHEN isFraud = 1 THEN 1 ELSE 0 END) / COUNT(*),
        4
    ) AS fraud_rate_percent
FROM transactions;

-- Fraud rate by transaction type
SELECT
    type,
    COUNT(*) AS total_txn,
    SUM(CASE WHEN isFraud = 1 THEN 1 ELSE 0 END) AS fraud_txn,
    ROUND(
        100.0 * SUM(CASE WHEN isFraud = 1 THEN 1 ELSE 0 END) / COUNT(*),
        4
    ) AS fraud_rate_percent
FROM transactions
GROUP BY type
ORDER BY fraud_rate_percent DESC;

-- Top high-value fraud cases (money emptied)
SELECT
    step,
    type,
    amount,
    nameOrig,
    oldbalanceOrg,
    newbalanceOrig,
    nameDest,
    oldbalanceDest,
    newbalanceDest
FROM transactions
WHERE isFraud = 1
  AND newbalanceDest = 0
  AND amount > 0
ORDER BY amount DESC
LIMIT 10;
```


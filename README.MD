### Fraud Detection Using SQL

This project looks at patterns of fraudulent behaviour in an online payments dataset.
The goal was to analyse transaction activity, spot risky behaviour, and build simple fraud insights using SQL.

The work was done in MySQL Workbench using a dataset of 40,000 online payment records.

### Project Goals

--- Look at how transactions behave across different payment types

--- Find unusual or suspicious activity

--- Measure how much fraud appears in the dataset

--- Identify accounts and transactions that look risky


### Dataset Information

--- Each transaction contains:

step – time index (simulation hour)

type – PAYMENT, TRANSFER, CASH_OUT, CASH_IN

amount

account origin balances (oldbalanceOrg, newbalanceOrig)

account destination balances (oldbalanceDest, newbalanceDest)

isFraud – actual fraud (1 = fraud, 0 = normal)

isFlaggedFraud – flagged by system

This data is publicly available and often used for fraud modelling.


### Key SQL Queries Used

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
### Additional Queries Used
--- Create helper tables (accounts + transaction groups) 

DROP TABLE IF EXISTS transaction_types;

--- Group transaction types into simple categories
CREATE TABLE accounts AS
SELECT DISTINCT
    name AS account_id
FROM (
    SELECT nameOrig AS name FROM transactions
    UNION
    SELECT nameDest AS name FROM transactions
) AS all_names;

--- Group transaction types into simple categories
CREATE TABLE transaction_types AS
SELECT DISTINCT
    type,
    CASE
        WHEN type IN ('TRANSFER','CASH_OUT') THEN 'Money moving out'
        WHEN type = 'CASH_IN' THEN 'Money coming in'
        ELSE 'Other'
    END AS type_group
FROM transactions;

--- Join accounts with transactions (sender + receiver)SELECT
    t.step,
    t.type,
    t.amount,
    o.account_id AS sender_account,
    d.account_id AS receiver_account,
    t.isFraud
FROM transactions t
JOIN accounts o
    ON t.nameOrig = o.account_id
JOIN accounts d
    ON t.nameDest = d.account_id
LIMIT 20;

--- Fraud rate by grouped transaction types
SELECT
    tt.type,
    tt.type_group,
    COUNT(*) AS total_txn,
    SUM(CASE WHEN t.isFraud = 1 THEN 1 ELSE 0 END) AS fraud_txn,
    ROUND(
        100.0 * SUM(CASE WHEN t.isFraud = 1 THEN 1 ELSE 0 END) / COUNT(*),
        4
    ) AS fraud_rate_percent
FROM transactions t
JOIN transaction_types tt
    ON t.type = tt.type
GROUP BY tt.type, tt.type_group
ORDER BY fraud_rate_percent DESC;

--- Accounts with the highest fraud-sending behaviour
WITH sender_fraud AS (
    SELECT
        nameOrig AS account_id,
        COUNT(*) AS total_sent,
        SUM(CASE WHEN isFraud = 1 THEN 1 ELSE 0 END) AS fraud_sent
    FROM transactions
    GROUP BY nameOrig
)
SELECT
    a.account_id,
    s.total_sent,
    s.fraud_sent,
    ROUND(100.0 * s.fraud_sent / s.total_sent, 2) AS fraud_rate_percent
FROM sender_fraud s
JOIN accounts a
    ON s.account_id = a.account_id
WHERE s.fraud_sent > 0
ORDER BY fraud_rate_percent DESC, s.fraud_sent DESC
LIMIT 20;

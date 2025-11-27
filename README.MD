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

# 3 Real-World Use Cases of LAST_QUERY_ID()

## 1. Data Lineage and Audit Logging
```sql
-- Insert data into a table
INSERT INTO customers VALUES (101, 'New Customer', 'contact@example.com');

-- Log the operation in an audit table
INSERT INTO data_audit_log 
VALUES (CURRENT_TIMESTAMP(), 'INSERT', 'customers', LAST_QUERY_ID(), CURRENT_USER());
```
This creates a traceable history of who modified what data and when, critical for compliance requirements like GDPR or financial regulations.

## 2. Time Travel Operations
```sql
-- Run a large data transformation
UPDATE transactions SET status = 'PROCESSED' WHERE batch_id = 'B12345';

-- Store the query ID for potential rollback
SET query_id = (SELECT LAST_QUERY_ID());

-- If issues are found, you can easily roll back
SELECT * FROM transactions AT(STATEMENT => $query_id) WHERE batch_id = 'B12345';
```
This allows for quick recovery from data errors without complex backup restoration processes.

## 3. Query Performance Troubleshooting
```sql
-- Run a complex analytical query
SELECT /*+ LABEL('quarterly_report') */ * FROM sales 
JOIN products ON sales.product_id = products.id
WHERE quarter = 'Q2-2025';

-- Capture performance metrics
SELECT 
    query_id,
    execution_time,
    bytes_scanned,
    percentage_scanned_from_cache
FROM table(information_schema.query_history())
WHERE query_id = LAST_QUERY_ID();
```
This helps optimization efforts by providing immediate access to performance metrics for the query just executed.
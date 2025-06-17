# 4. Data Manipulation Language (DML)

## Topics
- Learn about the INSERT statement and how to add new rows to tables.
- Learn about the UPDATE statement and how to modify existing data.
- Learn about the DELETE statement and how to remove data from tables.
- Learn about the MERGE statement and how to perform upsert operations.

## Key Takeaways

### DML vs DDL Distinction
- **DML (Data Manipulation Language)**: Operates on data within existing structures (INSERT, UPDATE, DELETE, MERGE)
- **DDL (Data Definition Language)**: Operates on database structures themselves (CREATE, ALTER, DROP)

### Core DML Operations
1. **INSERT**: Adds new rows to tables
2. **UPDATE**: Modifies existing data in tables
3. **DELETE**: Removes rows from tables
4. **MERGE**: Combines INSERT, UPDATE, and DELETE in a single operation (UPSERT)

### Transaction Control
- DML operations are transactional and can be rolled back
- Always consider data integrity and constraints
- Use transactions for multiple related operations

## INSERT Statement

### Basic INSERT Syntax
```sql
-- Insert with explicit column specification (recommended)
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);

-- Insert multiple rows
INSERT INTO table_name (column1, column2, column3)
VALUES 
    (value1a, value2a, value3a),
    (value1b, value2b, value3b),
    (value1c, value2c, value3c);
```

### INSERT Best Practices
```sql
-- Example: Employee table insertion
CREATE TABLE employees (
    employee_id INT AUTOINCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE DEFAULT CURRENT_DATE
);

-- Good practice: Explicit column specification
INSERT INTO employees (first_name, last_name, email, department, salary)
VALUES ('John', 'Doe', 'john.doe@company.com', 'Engineering', 75000.00);

-- Bulk insert with VALUES clause
INSERT INTO employees (first_name, last_name, email, department, salary)
VALUES 
    ('Jane', 'Smith', 'jane.smith@company.com', 'Marketing', 65000.00),
    ('Bob', 'Johnson', 'bob.johnson@company.com', 'Sales', 70000.00),
    ('Alice', 'Brown', 'alice.brown@company.com', 'HR', 60000.00);
```

### INSERT with SELECT (Data Migration)
```sql
-- Insert data from another table
INSERT INTO employees_backup (first_name, last_name, email, department)
SELECT first_name, last_name, email, department
FROM employees
WHERE department = 'Engineering';

-- Insert with calculated values
INSERT INTO employee_summary (department, avg_salary, employee_count)
SELECT 
    department,
    AVG(salary) as avg_salary,
    COUNT(*) as employee_count
FROM employees
GROUP BY department;
```

## UPDATE Statement

### Basic UPDATE Syntax
```sql
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE condition;
```

### UPDATE Best Practices
```sql
-- Always use WHERE clause to avoid updating all rows
-- Example: Salary increase for specific department
UPDATE employees
SET salary = salary * 1.10  -- 10% increase
WHERE department = 'Engineering';

-- Update multiple columns
UPDATE employees
SET 
    department = 'Software Engineering',
    salary = salary * 1.15
WHERE department = 'Engineering' AND hire_date < '2020-01-01';

-- Update with subquery
UPDATE employees
SET salary = (
    SELECT AVG(salary) * 1.05
    FROM employees e2
    WHERE e2.department = employees.department
)
WHERE performance_rating = 'Excellent';
```

### Conditional Updates with CASE
```sql
-- Complex conditional updates
UPDATE employees
SET salary = CASE
    WHEN department = 'Engineering' THEN salary * 1.12
    WHEN department = 'Sales' THEN salary * 1.08
    WHEN department = 'Marketing' THEN salary * 1.06
    ELSE salary * 1.03
END,
bonus = CASE
    WHEN salary > 80000 THEN 5000
    WHEN salary > 60000 THEN 3000
    ELSE 1000
END
WHERE hire_date < '2023-01-01';
```

## DELETE Statement

### Basic DELETE Syntax
```sql
DELETE FROM table_name
WHERE condition;
```

### DELETE Best Practices
```sql
-- Always use WHERE clause to avoid deleting all data
-- Example: Remove terminated employees
DELETE FROM employees
WHERE status = 'Terminated' AND termination_date < '2023-01-01';

-- Delete with EXISTS subquery
DELETE FROM employees
WHERE EXISTS (
    SELECT 1 
    FROM termination_log tl
    WHERE tl.employee_id = employees.employee_id
    AND tl.termination_date < CURRENT_DATE - INTERVAL '1 year'
);

-- Soft delete pattern (recommended for audit trails)
UPDATE employees
SET 
    is_active = FALSE,
    deleted_at = CURRENT_TIMESTAMP,
    deleted_by = CURRENT_USER()
WHERE employee_id = 12345;
```

### DELETE with Joins (Snowflake Specific)
```sql
-- Delete using MERGE or EXISTS pattern
DELETE FROM order_items
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.order_id = order_items.order_id
    AND o.order_status = 'CANCELLED'
    AND o.order_date < CURRENT_DATE - INTERVAL '90 days'
);
```

## MERGE Statement (UPSERT Operations)

### Understanding MERGE vs Individual Operations

**Terminology Clarification:**
- **MERGE**: Single statement that can INSERT, UPDATE, or DELETE based on conditions
- **UPSERT**: General term for "UPDATE or INSERT" operations
- **INSERT**: Only adds new records
- **UPDATE**: Only modifies existing records

### Basic MERGE Syntax
```sql
MERGE INTO target_table
USING source_table
ON match_condition
WHEN MATCHED THEN
    UPDATE SET column1 = value1, column2 = value2
WHEN NOT MATCHED THEN
    INSERT (column1, column2, column3)
    VALUES (value1, value2, value3);
```

### Comprehensive MERGE Example
```sql
-- Employee data synchronization from staging table
MERGE INTO employees AS target
USING employee_staging AS source
ON target.employee_id = source.employee_id

-- Update existing employees
WHEN MATCHED AND source.last_updated > target.last_updated THEN
    UPDATE SET
        first_name = source.first_name,
        last_name = source.last_name,
        email = source.email,
        department = source.department,
        salary = source.salary,
        last_updated = CURRENT_TIMESTAMP

-- Insert new employees
WHEN NOT MATCHED BY TARGET THEN
    INSERT (employee_id, first_name, last_name, email, department, salary, hire_date, last_updated)
    VALUES (source.employee_id, source.first_name, source.last_name, 
            source.email, source.department, source.salary, 
            CURRENT_DATE, CURRENT_TIMESTAMP)

-- Optional: Delete employees not in source (handle with care)
WHEN NOT MATCHED BY SOURCE AND target.is_active = TRUE THEN
    UPDATE SET 
        is_active = FALSE,
        termination_date = CURRENT_DATE;
```

### Advanced MERGE Patterns
```sql
-- Conditional MERGE with multiple match conditions
MERGE INTO product_inventory AS target
USING daily_sales AS source
ON target.product_id = source.product_id 
    AND target.warehouse_id = source.warehouse_id

WHEN MATCHED AND source.quantity_sold > 0 THEN
    UPDATE SET
        current_stock = target.current_stock - source.quantity_sold,
        last_sale_date = source.sale_date,
        total_sales = target.total_sales + source.quantity_sold

WHEN MATCHED AND source.quantity_sold = 0 THEN
    UPDATE SET last_checked = CURRENT_TIMESTAMP

WHEN NOT MATCHED THEN
    INSERT (product_id, warehouse_id, current_stock, last_sale_date, total_sales)
    VALUES (source.product_id, source.warehouse_id, 
            source.initial_stock - source.quantity_sold, 
            source.sale_date, source.quantity_sold);
```

## Transaction Management with DML

### Transaction Best Practices
```sql
-- Explicit transaction control
BEGIN TRANSACTION;

    -- Multiple related DML operations
    INSERT INTO order_audit (order_id, action, timestamp)
    VALUES (12345, 'PROCESSING', CURRENT_TIMESTAMP);
    
    UPDATE orders
    SET status = 'PROCESSING', last_updated = CURRENT_TIMESTAMP
    WHERE order_id = 12345;
    
    UPDATE inventory
    SET quantity = quantity - 5
    WHERE product_id = 67890;

    -- Verify operations before commit
    SELECT COUNT(*) as affected_orders FROM orders WHERE order_id = 12345 AND status = 'PROCESSING';

COMMIT; -- or ROLLBACK if verification fails
```

### Error Handling Patterns
```sql
-- Using TRY-CATCH equivalent patterns in stored procedures
CREATE OR REPLACE PROCEDURE update_employee_salary(emp_id INTEGER, new_salary DECIMAL(10,2))
RETURNS STRING
LANGUAGE SQL
AS
$$
BEGIN
    -- Validation
    IF new_salary <= 0 THEN
        RETURN 'Error: Salary must be positive';
    END IF;
    
    -- Check if employee exists
    IF NOT EXISTS (SELECT 1 FROM employees WHERE employee_id = emp_id) THEN
        RETURN 'Error: Employee not found';
    END IF;
    
    -- Perform update
    UPDATE employees 
    SET salary = new_salary, last_updated = CURRENT_TIMESTAMP
    WHERE employee_id = emp_id;
    
    RETURN 'Success: Salary updated';
END;
$$;
```

## Performance Considerations

### DML Optimization Strategies
1. **Batch Operations**: Use bulk INSERT/UPDATE instead of row-by-row operations
2. **Indexing**: Ensure WHERE clauses use indexed columns
3. **Constraints**: Leverage database constraints for data integrity
4. **Clustering**: Use clustering keys for better performance in Snowflake

```sql
-- Efficient bulk operations
-- Instead of multiple single INSERTs
INSERT INTO large_table (col1, col2, col3)
SELECT col1, col2, col3
FROM staging_table
WHERE processing_date = CURRENT_DATE;

-- Instead of cursor-based UPDATEs
UPDATE large_table
SET status = 'PROCESSED'
WHERE id IN (
    SELECT id FROM processing_queue 
    WHERE batch_id = 'BATCH_001'
);
```

## Common Pitfalls and Solutions

### Avoiding Data Loss
```sql
-- Always backup before major operations
CREATE TABLE employees_backup AS
SELECT * FROM employees;

-- Use LIMIT in DELETE operations during testing
DELETE FROM test_table
WHERE condition = 'test_value'
LIMIT 100; -- Snowflake: Use LIMIT to restrict deletion scope

-- Verify UPDATE operations before executing
SELECT COUNT(*) as will_be_updated
FROM employees
WHERE department = 'Engineering'; -- Check before UPDATE
```

### Data Integrity Maintenance
```sql
-- Use constraints to maintain referential integrity
ALTER TABLE orders
ADD CONSTRAINT fk_customer_id
FOREIGN KEY (customer_id) REFERENCES customers(customer_id);

-- Implement audit trails
CREATE TABLE audit_log (
    audit_id INT AUTOINCREMENT,
    table_name VARCHAR(50),
    operation VARCHAR(10),
    old_values VARIANT,
    new_values VARIANT,
    user_name VARCHAR(100),
    timestamp TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP
);
```

## Personal Notes

- DML operations are the core of data manipulation in SQL databases
- MERGE statement provides powerful UPSERT capabilities but requires careful condition design
- Always consider transaction boundaries and rollback strategies
- Performance optimization through proper indexing and bulk operations is crucial
- Data integrity should be maintained through constraints and validation
- Audit trails are essential for tracking data changes in production systems

## Questions

1. When should I use MERGE instead of separate INSERT/UPDATE statements?
2. How can I optimize bulk DML operations in Snowflake?
3. What are the best practices for handling concurrent DML operations?
4. How do I implement soft deletes vs hard deletes effectively?
5. What transaction isolation levels should I consider for DML operations?


## Navigation
[← Previous: 3. Data Definition Language (DDL)](3-data-definition-language.md)  
[Next: 5. Querying Data →](5-querying-data.md)
Given the following Snowflake SQL MERGE statement: \

```sql
MERGE INTO EMPLOYEE AS E 
USING EMPLOYEE_CHANGES AS EC
ON E.EMP_ID = EC.EMP_ID
WHEN MATCHED AND E.EMP_COUNTRY_CODE = 'US'
    THEN UPDATE SET E.EMP_COUNTRY_CODE = EC.EMP_COUNTRY_CODE
WHEN MATCHED AND E.EMP_COUNTRY_CODE = 'GB'
    THEN DELETE 
WHEN NOT MATCHED THEN
    INSERT (EMP_ID, EMP_FIRST_NAME, EMP_LAST_NAME, EMP_DOB, EMP_COUNTRY_CODE)
    VALUES (EC.EMP_ID, EC.EMP_FIRST_NAME, EC.EMP_LAST_NAME, EC.EMP_DOB, EC.EMP_COUNTRY_CODE);
```

Based on my research, here's the current status of `MERGE` statement support in MariaDB and PostgreSQL:

## PostgreSQL ✅
**Yes, PostgreSQL supports the MERGE statement** as of **PostgreSQL 15** (released in October 2022). The compatibility chart shows that PostgreSQL 17 (the latest version) has full MERGE statement support including:
- Basic MERGE functionality
- WHEN MATCHED THEN UPDATE
- WHEN MATCHED THEN DELETE  
- WHEN NOT MATCHED THEN INSERT

## MariaDB ❌
**No, MariaDB does not support the MERGE statement**. The compatibility information shows that MariaDB is notably absent from the list of databases that support MERGE. 

However, MariaDB does provide an alternative syntax for similar functionality:
- **`INSERT ... ON DUPLICATE KEY UPDATE`** - This can handle some upsert scenarios but is not as flexible as a full MERGE statement

## Key Takeaways

1. **PostgreSQL**: Your Snowflake MERGE statement could potentially be adapted to work in PostgreSQL 15+ with some syntax adjustments.

2. **MariaDB**: You would need to rewrite the logic using a combination of:
   - `INSERT ... ON DUPLICATE KEY UPDATE` for upsert operations
   - Separate `UPDATE` and `DELETE` statements with appropriate conditions
   - Potentially using stored procedures or application logic to handle the complex conditional logic

3. **Standard SQL**: The MERGE statement is part of the ISO/IEC 9075-2:2023 SQL standard as optional features F312, F313, and F314, but adoption varies significantly across database systems.

If you're planning to migrate from Snowflake to either of these databases, PostgreSQL would be the better choice if you want to preserve MERGE statement functionality with minimal changes.
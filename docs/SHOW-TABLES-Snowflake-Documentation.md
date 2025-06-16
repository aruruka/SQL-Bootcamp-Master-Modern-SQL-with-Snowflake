---
title: SHOW TABLES | Snowflake Documentation
source: https://docs.snowflake.com/en/sql-reference/sql/show-tables
author: 
published: 
created: 2025-06-16
description: 
tags:
  - clippings
  - snowflake
  - documentation
---
## SHOW TABLES

Lists the tables for which you have access privileges, including dropped tables that are still within the Time Travel retention period and, therefore, can be undropped. The command can be used to list tables for the current/specified database or schema, or across your entire account.

The output returns table metadata and properties, ordered lexicographically by database, schema, and table name (see in this topic for descriptions of the output columns). This is important to note if you want to filter the results using the provided filters.

See also:

[CREATE TABLE](https://docs.snowflake.com/en/sql-reference/sql/create-table), [DROP TABLE](https://docs.snowflake.com/en/sql-reference/sql/drop-table), [UNDROP TABLE](https://docs.snowflake.com/en/sql-reference/sql/undrop-table), [ALTER TABLE](https://docs.snowflake.com/en/sql-reference/sql/alter-table), [DESCRIBE TABLE](https://docs.snowflake.com/en/sql-reference/sql/desc-table)

[TABLES view](https://docs.snowflake.com/en/sql-reference/info-schema/tables) (Information Schema)

## Syntax

```plaintext
SHOW [ TERSE ] TABLES [ HISTORY ] [ LIKE '<pattern>' ]
                                  [ IN
                                        {
                                          ACCOUNT                                         |

                                          DATABASE                                        |
                                          DATABASE <database_name>                        |

                                          SCHEMA                                          |
                                          SCHEMA <schema_name>                            |
                                          <schema_name>

                                          APPLICATION <application_name>                  |
                                          APPLICATION PACKAGE <application_package_name>  |
                                        }
                                  ]
                                  [ STARTS WITH '<name_string>' ]
                                  [ LIMIT <rows> [ FROM '<name_string>' ] ]
```


## Parameters

`TERSE`

Optionally returns only a subset of the output columns:

- `created_on`
- `name`
- `kind`
	The `kind` column value is always TABLE.
- `database_name`
- `schema_name`

Default: No value (all columns are included in the output)

`HISTORY`

Optionally includes dropped tables that have not yet been purged (i.e. they are still within their respective Time Travel retention periods). If multiple versions of a dropped table exist, the output displays a row for each version. The output also includes an additional `dropped_on` column, which displays:

- Date and timestamp (for dropped tables).
- `NULL` (for active tables).

Default: No value (dropped tables are not included in the output)

`LIKE '*pattern*'`

Optionally filters the command output by object name. The filter uses case-insensitive pattern matching, with support for SQL wildcard characters (`%` and `_`).

For example, the following patterns return the same results:

`... LIKE '%testing%' ...`

`... LIKE '%TESTING%' ...`

. Default: No value (no filtering is applied to the output).

`[ IN ... ]`

Optionally specifies the scope of the command. Specify one of the following:

`ACCOUNT`

Returns records for the entire account.

`DATABASE`, .`DATABASE *db_name*`

Returns records for the current database in use or for a specified database (`*db_name*`).

If you specify `DATABASE` without `*db_name*` and no database is in use, the keyword has no effect on the output.

Note

Using SHOW commands without an `IN` clause in a database context can result in fewer than expected results.

Objects with the same name are only displayed once if no `IN` clause is used. For example, if you have table `t1` in `schema1` and table `t1` in `schema2`, and they are both in scope of the database context you’ve specified (that is, the database you’ve selected is the parent of `schema1` and `schema2`), then SHOW TABLES only displays one of the `t1` tables.

`SCHEMA`, .`SCHEMA *schema_name*`

Returns records for the current schema in use or a specified schema (`*schema_name*`).

`SCHEMA` is optional if a database is in use or if you specify the fully qualified `*schema_name*` (for example, `db.schema`).

If no database is in use, specifying `SCHEMA` has no effect on the output.

`APPLICATION *application_name*`, .`APPLICATION PACKAGE *application_package_name*`

Returns records for the named Snowflake Native App or application package.

Default: Depends on whether the session currently has a database in use:

- Database: `DATABASE` is the default (that is, the command returns the objects you have privileges to view in the database).
- No database: `ACCOUNT` is the default (that is, the command returns the objects you have privileges to view in your account).

`STARTS WITH '*name_string*'`

Optionally filters the command output based on the characters that appear at the beginning of the object name. The string must be enclosed in single quotes and is case sensitive.

For example, the following strings return different results:

`... STARTS WITH 'B' ...`

`... STARTS WITH 'b' ...`

. Default: No value (no filtering is applied to the output)

`LIMIT *rows* [ FROM '*name_string*' ]`

Optionally limits the maximum number of rows returned, while also enabling “pagination” of the results. The actual number of rows returned might be less than the specified limit. For example, the number of existing objects is less than the specified limit.

The optional `FROM '*name_string*'` subclause effectively serves as a “cursor” for the results. This enables fetching the specified number of rows following the first row whose object name matches the specified string:

- The string must be enclosed in single quotes and is case sensitive.
- The string does not have to include the full object name; partial names are supported.

Default: No value (no limit is applied to the output)

Note

For SHOW commands that support both the `FROM '*name_string*'` and `STARTS WITH '*name_string*'` clauses, you can combine both of these clauses in the same statement. However, both conditions must be met or they cancel out each other and no results are returned.

In addition, objects are returned in lexicographic order by name, so `FROM '*name_string*'` only returns rows with a higher lexicographic value than the rows returned by `STARTS WITH '*name_string*'`.

For example:

- `... STARTS WITH 'A' LIMIT ... FROM 'B'` would return no results.
- `... STARTS WITH 'B' LIMIT ... FROM 'A'` would return no results.
- `... STARTS WITH 'A' LIMIT ... FROM 'AB'` would return results (if any rows match the input strings).

## Output

The command output provides table properties and metadata in the following columns:

| Column | Description |
| --- | --- |
| created\_on | Date and time when the table was created. |
| name | Name of the table. |
| database\_name | Database in which the table is stored. |
| schema\_name | Schema in which the table is stored. |
| kind | Table type: TABLE (for permanent tables), TEMPORARY, or TRANSIENT. |
| comment | Comment for the table. |
| cluster\_by | Column(s) defined as clustering key(s) for the table. |
| rows | Number of rows in the table. Returns NULL for external tables. |
| bytes | Number of bytes that will be scanned if the entire table is scanned in a query. Note that this number may be different than the number of actual physical bytes (i.e. bytes stored on-disk) for the table. |
| owner | Role that owns the table. |
| retention\_time | Number of days that modified and deleted data is retained for Time Travel. |
| dropped\_on | Date and time when the table was dropped; NULL if the table is active. This column is only displayed when the HISTORY keyword is specified for the command. |
| automatic\_clustering | If [Automatic Clustering](https://docs.snowflake.com/en/user-guide/tables-auto-reclustering) is enabled for your account, specifies whether it is explicitly enabled (`ON`) or disabled (`OFF`) for the table. This column is not displayed if Automatic Clustering is not enabled for your account. |
| change\_tracking | If `ON`, change tracking is enabled. You can query this change tracking data using [streams](https://docs.snowflake.com/en/user-guide/streams-intro) or the [CHANGES](https://docs.snowflake.com/en/sql-reference/constructs/changes) clause for [SELECT](https://docs.snowflake.com/en/sql-reference/sql/select) statements. If `OFF`, change tracking is currently disabled but could be [enabled](https://docs.snowflake.com/en/user-guide/streams-manage). |
| search\_optimization | If `ON`, the table has the [search optimization service](https://docs.snowflake.com/en/user-guide/search-optimization-service) enabled. Otherwise, the value is `OFF`. |
| search\_optimization\_progress | Percentage of the table that has been optimized for search. This value increases when optimization is first added to a table and when maintenance is done on the search optimization service. Before you measure the performance improvement of search optimization on a newly-optimized table, wait until this shows that the table has been fully optimized. |
| search\_optimization\_bytes | Number of additional bytes of storage that the search optimization service consumes for this table. |
| is\_external | `Y` if it is an external table; `N` otherwise. |
| enable\_schema\_evolution | `Y` if the table has [schema evolution](https://docs.snowflake.com/en/user-guide/data-load-schema-evolution) enabled; `N` otherwise. You can enable automatic table schema evolution by using the [CREATE TABLE](https://docs.snowflake.com/en/sql-reference/sql/create-table) or [ALTER TABLE](https://docs.snowflake.com/en/sql-reference/sql/alter-table) commands. |
| owner\_role\_type | The type of role that owns the object, for example `ROLE`. . If a Snowflake Native App owns the object, the value is `APPLICATION`. . Snowflake returns NULL if you delete the object because a deleted object does not have an owner role. |
| is\_event | `Y` if it is an event table; `N` otherwise. |
| is\_hybrid | `Y` if it is a hybrid table; `N` otherwise. |
| is\_iceberg | `Y` if the table is an [Apache Iceberg™ table](https://docs.snowflake.com/en/user-guide/tables-iceberg); `N` otherwise. |
| is\_immutable | `Y` if the table was created with the [READ ONLY](https://docs.snowflake.com/en/sql-reference/sql/create-table.html#label-create-table-read-only) property; `N` otherwise. |

For more information about the properties that can be specified for a table, see [CREATE TABLE](https://docs.snowflake.com/en/sql-reference/sql/create-table).

Note

For cloned tables and tables with deleted data, the `bytes` displayed for the table may be different than the number of physical bytes for the table:

- A cloned table does not utilize additional data storage until new rows are added to the table or existing rows in the table are modified or deleted. If few or no changes have been made to the table, the number of bytes displayed is more than the actual physical bytes stored for the table.
- Data deleted from a table is maintained in Snowflake until both the Time Travel retention period (default is 1 day) and Fail-safe period (7 days) for the data have passed. During these two periods, the number of bytes displayed is less than the actual physical bytes stored for the table.

For more detailed information about table size in bytes as it relates to cloning, Time Travel, and Fail-safe, see the [TABLE\_STORAGE\_METRICS](https://docs.snowflake.com/en/sql-reference/info-schema/table_storage_metrics) Information Schema view.

## Usage notes

- If an account (or database or schema) has a large number of tables, then searching the entire account (or table or schema) can consume a significant amount of compute resources.
- In the output, results are sorted by database name, schema name, and then table name. This means results for a database can contain tables from multiple schemas and might break pagination. In order for pagination to work as expected, you must execute the SHOW TABLES command for a single schema. You can use the IN SCHEMA `*schema_name*` parameter to the SHOW TABLES command. Alternatively, you can use the schema in the current context by executing a USE SCHEMA command before executing a SHOW TABLES command.
- The command doesn’t require a running warehouse to execute.
- The command only returns objects for which the current user’s current role has been granted at least one access privilege.
- The MANAGE GRANTS access privilege implicitly allows its holder to see every object in the account. By default, only the account administrator (users with the ACCOUNTADMIN role) and security administrator (users with the SECURITYADMIN role) have the MANAGE GRANTS privilege.
- To post-process the output of this command, you can use the [RESULT\_SCAN](https://docs.snowflake.com/en/sql-reference/functions/result_scan) function, which treats the output as a table that can be queried. You can also use the [pipe operator](https://docs.snowflake.com/en/sql-reference/operators-flow) to query the output of this command.
- The value for `LIMIT *rows*` can’t exceed `10000`. If `LIMIT *rows*` is omitted, the command results in an error if the result set is larger than ten thousand rows.
	To view results for which more than ten thousand records exist, either include `LIMIT *rows*` or query the corresponding view in the [Snowflake Information Schema](https://docs.snowflake.com/en/sql-reference/info-schema).

## Examples

These examples show all of the tables that you have privileges to view based on the specified parameters.

Run SHOW TABLES on tables in the [Sample Data Sets](https://docs.snowflake.com/en/user-guide/sample-data). The examples use the TERSE parameter to limit the output.

Show all the tables with a name that starts with `LINE` in the `tpch_sf1` schema:

```sql
SHOW TERSE TABLES IN tpch_sf1 STARTS WITH 'LINE';
```

```plaintext
+-------------------------------+----------+-------+-----------------------+-------------+
| created_on                    | name     | kind  | database_name         | schema_name |
|-------------------------------+----------+-------+-----------------------+-------------|
| 2016-07-08 13:41:59.960 -0700 | LINEITEM | TABLE | SNOWFLAKE_SAMPLE_DATA | TPCH_SF1    |
+-------------------------------+----------+-------+-----------------------+-------------+
```

Show all of the tables with a name that includes the substring `PART` in the `tpch_sf1` schema:

```sql
SHOW TERSE TABLES LIKE '%PART%' IN tpch_sf1;
```

```plaintext
+-------------------------------+-----------+-------+-----------------------+-------------+
| created_on                    | name      | kind  | database_name         | schema_name |
|-------------------------------+-----------+-------+-----------------------+-------------|
| 2016-07-08 13:41:59.960 -0700 | JPART     | TABLE | SNOWFLAKE_SAMPLE_DATA | TPCH_SF1    |
| 2016-07-08 13:41:59.960 -0700 | JPARTSUPP | TABLE | SNOWFLAKE_SAMPLE_DATA | TPCH_SF1    |
| 2016-07-08 13:41:59.960 -0700 | PART      | TABLE | SNOWFLAKE_SAMPLE_DATA | TPCH_SF1    |
| 2016-07-08 13:41:59.960 -0700 | PARTSUPP  | TABLE | SNOWFLAKE_SAMPLE_DATA | TPCH_SF1    |
+-------------------------------+-----------+-------+-----------------------+-------------+
```

Show the tables in the `tpch_sf1` schema, but limit the output to three rows, and start with the table names that begin with `J`:

```sql
SHOW TERSE TABLES IN tpch_sf1 LIMIT 3 FROM 'J';
```

```plaintext
+-------------------------------+-----------+-------+-----------------------+-------------+
| created_on                    | name      | kind  | database_name         | schema_name |
|-------------------------------+-----------+-------+-----------------------+-------------|
| 2016-07-08 13:41:59.960 -0700 | JCUSTOMER | TABLE | SNOWFLAKE_SAMPLE_DATA | TPCH_SF1    |
| 2016-07-08 13:41:59.960 -0700 | JLINEITEM | TABLE | SNOWFLAKE_SAMPLE_DATA | TPCH_SF1    |
| 2016-07-08 13:41:59.960 -0700 | JNATION   | TABLE | SNOWFLAKE_SAMPLE_DATA | TPCH_SF1    |
+-------------------------------+-----------+-------+-----------------------+-------------+
```

Show a dropped table using the HISTORY parameter.

Create a table in your current schema, then drop it:

```sql
CREATE OR REPLACE TABLE test_show_tables_history(c1 NUMBER);

DROP TABLE test_show_tables_history;
```


Use the HISTORY parameter to include dropped tables in the command output:

```sql
SHOW TABLES HISTORY LIKE 'test_show_tables_history';
```

In the output, the `dropped_on` column shows the date and time when the table was dropped.
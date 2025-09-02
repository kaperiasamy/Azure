# clustered and non-clustered indexes

## Definition of Indexes

**Index**: An index is a database object that improves the speed of data retrieval operations on a table at the cost of additional storage space and maintenance overhead.

## Clustered Index

**Definition**: A clustered index determines the physical order of data in a table. There can be only one clustered index per table because the data rows can be stored in only one order.
**Characteristics**:
    - It creates the actual data structure in the database.
    - The clustered index key is the primary key by default, but you can choose any unique column.
    - Useful for range queries as rows are stored sequentially.

**Example**:

```sql
CREATE CLUSTERED INDEX IX_Employees_LastName
ON Employees(LastName);
```

## Non-Clustered Index

**Definition**: A non-clustered index creates a logical ordering of the data, which is separate from the physical storage of the data rows. This index contains pointers to the actual data.
**Characteristics**:
    - A table can have multiple non-clustered indexes (up to 999 in SQL Server).
    - Useful for queries that don’t need the entire row of data, but rather specific column values used in a WHERE clause.
    - It allows quick look-ups by maintaining a separate structure that maps to the data in the clustered index or the table itself.

**Example**:

```sql
CREATE NONCLUSTERED INDEX IX_Employees_FirstName
ON Employees(FirstName);
```

## Differences between Clustered and Non-Clustered Index

| Feature                         | Clustered Index                    | Non-Clustered Index                |
|---------------------------------|------------------------------------|------------------------------------|
| Order of Data                   | Physically orders data rows        | Logical order, separate storage    |
| Number per Table                | One per table                      | Up to 999 per table                |
| Size                            | Can be larger; typically data size | Fixed size; depends on key columns |
| Usage                           | Best for range queries             | Best for lookups and searches      |
| Key Storage                     | Stored in the leaf level of index  | Contains pointers to data rows     |
| Maximum Number of Non-Clustered | Not applicable                     | Up to 999 per table                |
| Indexes                         |                                    |                                    |

## Maximum Limit: 
In SQL Server, you can have a maximum of 999 non-clustered indexes on a single table. This allows for optimized searching based on various query patterns.

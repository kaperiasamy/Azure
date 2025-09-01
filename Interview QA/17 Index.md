# Index

## Definition:
An index in SQL is a database object that improves the speed of data retrieval operations on a database table by providing quick access to rows based on the values of one or more columns.
It works similarly to an index in a book, which allows you to find information quickly without scanning through all the pages.

## Purpose:
- The primary purpose of an index is to enhance query performance by reducing the amount of data the database engine must scan to find relevant rows. It can significantly speed up SELECT queries, especially on large datasets.
However, it's important to balance index use, as excessive indexing can increase the overhead for INSERT, UPDATE, and DELETE operations due to the need to maintain the index.

## Types of Indexes:
- **Clustered Index**: Determines the physical order of data in the table. A table can have only one clustered index.
- **Non-Clustered Index**: A separate structure that points to the data in the table; multiple non-clustered indexes can exist on a table.
- **Unique Index**: Ensures that all values in the indexed column are unique, similar to primary keys but can be created on non-primary key columns.
- **Composite Index**: An index on multiple columns, useful for queries involving multiple fields in the WHERE clause.

## Benefits:
- Faster query performance, especially for searching, filtering, and sorting.
- Improved performance for operations that rely on indexed columns (e.g., JOIN, WHERE).
- Helping enforce uniqueness for data integrity constraints with unique indexes.

## Drawbacks:
- Increased disk space usage, as indexes require additional storage.
- Potential performance degradation on write operations (INSERT/UPDATE/DELETE) because the index needs to be updated alongside the data modifications.
- Index maintenance can add overhead during operations, especially on large tables with frequent updates.

## Best Practices:
- Monitor query performance using tools to determine which indexes to create or drop based on usage patterns.
- Use indexes on columns frequently used in search conditions, sorting, and joins.
- Avoid indexing columns that have low selectivity, such as boolean fields or columns with very few distinct values.

## Example of Creating an Index:

Here is a simple example of how to create an index in SQL:

```sql

CREATE INDEX idx_lastname ON Employees (LastName);
``` 

This command creates a non-clustered index on the LastName column of the Employees table.
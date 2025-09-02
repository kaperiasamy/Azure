## 1. Difference Between Clustered and Non-Clustered Index:

### Clustered Index:
**Definition**: A clustered index sorts and stores the data rows in the table based on the index key. The data itself is physically organized in the order of the clustered index, meaning that there can be only one clustered index per table.
**Use Case**: Ideal for columns that are often used for range queries or where row retrieval is directly linked to the index key, such as a primary key or frequently used lookup columns.
**Example**: When a clustered index is created on a CustomerID column, the actual data for customers is stored in the order of CustomerID.

### Non-Clustered Index:
**Definition**: A non-clustered index is a separate structure from the data rows. It contains a pointer to the location of the data rows in the table. This means you can have multiple non-clustered indexes per table.
**Use Case**: Best for columns frequently queried in WHERE clauses or JOIN conditions that do not dictate the physical order of data, allowing for faster lookups without rearranging the data.
**Example**: A non-clustered index on an Email column would allow quick searches based on email without changing how customer data is organized.

## 2. Difference Between Primary and Composite Keys:

### Primary Key:
**Definition**: A primary key is a single column (or a set of columns) that uniquely identifies each record in a table. It enforces entity integrity by ensuring that no two rows can have the same primary key value.
**Characteristics**: It cannot contain NULL values and must contain unique values; typically, there is only one primary key per table.
**Example**: An EmployeeID could serve as the primary key in an Employees table, uniquely identifying each employee.

### Composite Key:
**Definition**: A composite key consists of two or more columns that together uniquely identify a record in a table. It is useful when a single column is not sufficient to guarantee uniqueness.
**Characteristics**: Each column in a composite key can contain duplicate values, but the combination of all the fields must be unique.
**Example**: In a CourseEnrollment table, a composite key could be formed using StudentID and CourseID, ensuring that each student can only be enrolled in a particular course once.

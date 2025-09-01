# Function and stored Procedure

## Definition:
- A Function is a database object that performs a specific task and returns a value. It is often used for computations and can be called from SQL statements.
- A Stored Procedure is a set of SQL statements that perform a particular task and can return multiple values or a result set. It is mainly used for executing tasks that may involve complex business logic.

## Return Type:
- Function: Must return a single value (scalar or table), and can also include output parameters.
- Stored Procedure: Does not have to return a value; it can execute operations and return multiple result sets through output parameters or SELECT statements.

## Execution:
- Function: Can be called from within a SELECT, WHERE, or JOIN clause, making it more versatile for inline calculations.
- Stored Procedure: Typically executed as a standalone command (EXEC or CALL), and cannot be used inline within SQL statements.

## Side Effects:
- Function: Generally should not have side effects, meaning it shouldn't change the state of the database (e.g., INSERT, UPDATE, DELETE operations).
- Stored Procedure: Can change the state of the database; it's common for it to perform various transactions.

## Use Cases:
- Function: Ideal for operations that require a return value, such as calculating a discount or retrieving values that can be used in another query.
- Stored Procedure: Best suited for operations involving multiple steps, complex business logic, or when interacting with the database in a batch process.

## Example of Function and Stored Procedure:

Here's a high-level example of how both might be implemented:

### Function Example:

```sql

CREATE FUNCTION GetEmployeeFullName (@EmployeeID INT)
RETURNS NVARCHAR(100) AS
BEGIN
    DECLARE @FullName NVARCHAR(100);
    SELECT @FullName = FirstName + ' ' + LastName FROM Employees WHERE EmployeeID = @EmployeeID;
    RETURN @FullName;
END;
```

### Stored Procedure Example:

```sql

CREATE PROCEDURE UpdateEmployeeSalary
    @EmployeeID INT,
    @NewSalary DECIMAL(10, 2)
AS
BEGIN
    UPDATE Employees SET Salary = @NewSalary WHERE EmployeeID = @EmployeeID;
    SELECT 'Salary updated successfully' AS Message;
END;
```
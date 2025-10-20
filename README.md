# SQL
1) Differences between Stored Procedures vs Functions

*Overview*

•	A Stored Procedure (SP) :
is a programmable unit that performs operations on the database. It can execute CRUD actions (INSERT/UPDATE/DELETE/SELECT), manage transactions, and return result sets or output parameters. Called with EXEC or EXECUTE.

•	A Function (UDF — User Defined Function) :
  returns a single value (Scalar) or a table (Table-valued). It’s used inside SQL expressions like SELECT, WHERE, or JOIN. Functions are pure — they shouldn’t modify database state (no INSERT/UPDATE/DELETE in T-SQL functions).

*When i use it*

You should use a Stored Procedure when:
	•	You need to modify or manage data (for example, running INSERT or UPDATE commands).
	•	You want to control transactions (using BEGIN TRAN, COMMIT, or ROLLBACK).
	•	You need to return multiple result sets.
	•	You’re performing complex operations like maintenance or batch processes.

You should use a Function when:
	•	You need reusable logic inside a query.
	•	You want to calculate or transform data and return it as a single value or table.
	•	You need cleaner and more modular query code.
  
Example — Stored Procedure

CREATE PROCEDURE dbo.AddCustomer
    @Name NVARCHAR(100),
    @Email NVARCHAR(100),
    @NewCustomerID INT OUTPUT
AS
BEGIN
    SET NOCOUNT ON;
    BEGIN TRY
        BEGIN TRAN;
        INSERT INTO dbo.Customers (Name, Email)
        VALUES (@Name, @Email);
        SET @NewCustomerID = SCOPE_IDENTITY();
        COMMIT TRAN;
    END TRY
    BEGIN CATCH
        ROLLBACK TRAN;
        THROW;
    END CATCH
END;

Example — Function

CREATE FUNCTION dbo.GetActiveOrders(@CustomerID INT)
RETURNS TABLE
AS
RETURN
(
    SELECT OrderID, OrderDate, TotalAmount
    FROM dbo.Orders
    WHERE CustomerID = @CustomerID AND Status = 'Active'
);

2) SQL Server Triggers

*Overview*

A Trigger is a piece of SQL code that executes automatically when a specific event occurs in the database, such as an INSERT, UPDATE, or DELETE operation. Triggers are great for enforcing rules or recording changes without requiring manual actions.

There are two main types of triggers in SQL Server:
	•	AFTER triggers, which run after the operation is completed.
	•	INSTEAD OF triggers, which replace the original operation with custom logic.

SQL Server provides two virtual tables inside triggers:
	•	inserted – holds new records.
	•	deleted – holds old records that were modified or deleted.


  Example — AFTER DELETE trigger for audit logging

  CREATE TRIGGER dbo.trg_Employee_Delete
ON dbo.Employees
AFTER DELETE
AS
BEGIN
    SET NOCOUNT ON;
    INSERT INTO dbo.EmployeeAudit (EmployeeID, Action, ActionDate, OldName)
    SELECT EmployeeID, 'DELETE', GETDATE(), Name
    FROM deleted;
END;

Example — INSTEAD OF trigger on a view

CREATE TRIGGER trg_vw_Order_Insert
ON dbo.vw_OrdersCustomers
INSTEAD OF INSERT
AS
BEGIN
    SET NOCOUNT ON;
    INSERT INTO dbo.Orders (CustomerID, TotalAmount)
    SELECT CustomerID, TotalAmount FROM inserted;
END;

3) Views

*Overview*

A View is a virtual table created from a SQL query. It doesn’t store data itself but displays data from one or more tables. Views are perfect for simplifying complex queries, improving security, and reusing logic across multiple parts of your application.

Benefits of using Views:
	•	Simplifies complex joins and aggregations.
	•	Enhances security by limiting access to base tables.
	•	Makes your code more modular and easier to maintain.
	•	Can improve performance using Indexed Views, which store results physically for faster access.

Example — Simple view

CREATE VIEW dbo.vw_CustomerOrders
AS
SELECT c.CustomerID, c.Name, o.OrderID, o.OrderDate, o.TotalAmount
FROM dbo.Customers c
JOIN dbo.Orders o ON c.CustomerID = o.CustomerID;

Example — Summary view

CREATE VIEW dbo.vw_CustomerOrderSummary
AS
SELECT c.CustomerID, c.Name,
       COUNT(o.OrderID) AS OrdersCount,
       SUM(o.TotalAmount) AS TotalSpent
FROM dbo.Customers c
LEFT JOIN dbo.Orders o ON c.CustomerID = o.CustomerID
GROUP BY c.CustomerID, c.Name;


  

  


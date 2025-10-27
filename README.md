# SQL
# 1) Differences between Stored Procedures vs Functions

*Overview*

•	A Stored Procedure (SP) :

A procedure:
Is a type of subprogram that performs an action
Can be stored in the database as a schema object
Promotes reusability and maintainability

in anthor word is a programmable unit that performs operations on the database. It can execute CRUD actions (INSERT/UPDATE/DELETE/SELECT), manage transactions, and return result sets or output parameters. Called with EXEC or EXECUTE.

•	A Function (UDF — User Defined Function) :
A function:
Is a named PL/SQL block that returns a value
Can be stored in the database as a schema object for
repeated execution
Is called as part of an expression or is used to provide a
parameter value

in anthor word
returns a single value (Scalar) or a table (Table-valued). It’s used inside SQL expressions like SELECT, WHERE, or JOIN. Functions are pure — they shouldn’t modify database state (no INSERT/UPDATE/DELETE in T-SQL functions).

**When i use it**

**You should use a Stored Procedure when:**

	- You need to modify or manage data (for example, running INSERT or UPDATE commands)
    - You want to control transactions (using BEGIN TRAN, COMMIT, or ROLLBACK).
	- You need to return multiple result sets.
	- You’re performing complex operations like maintenance or batch processes.

**You should use a Function when:**

	- You need reusable logic inside a query.
	- You want to calculate or transform data and return it as a single value or table.
	- You need cleaner and more modular query code.
  
Example 1 — Stored Procedure



```sql
CREATE OR REPLACE PROCEDURE raise_salary  (id      IN employees.employee_id%TYPE,   percent IN NUMBER)
 IS
 BEGIN
 UPDATE employees
 SET    salary = salary * (1 + percent/100)
 WHERE  employee_id = id; END raise_salary;

/
EXECUTE raise_salary(176,10)


```
Example 2 — Stored Procedure
```sql



Example — Function
CREATE OR REPLACE PROCEDURE query_emp (id     IN  employees.employee_id%TYPE,  name   OUT employees.last_name%TYPE,  salary OUT employees.salary%TYPE) 
 IS 
 BEGIN  
 SELECT   last_name, salary INTO name, salary   
 FROM    employees   
 WHERE   employee_id = id; 
 END query_emp;


DECLARE  emp_name employees.last_name%TYPE;
 emp_sal  employees.salary%TYPE;
BEGIN  query_emp(171, emp_name, emp_sal);
... END;

```
Example 1 — Stored Functions
```sql
CREATE [OR REPLACE] FUNCTION function_name [(parameter1 [mode1] datatype1, ...)]
RETURN datatype
IS|AS
[local_variable_declarations; …]
BEGIN  -- actions; 
 RETURN expression;
END [function_name];

```
Example 2 — Stored Functions
```sql
CREATE OR REPLACE FUNCTION get_sal (id employees.employee_id%TYPE)
RETURN NUMBER
IS  sal employees.salary%TYPE := 0;
BEGIN  SELECT salary
INTO   sal
FROM   employees
WHERE  employee_id = id;
 RETURN sal;
END get_sal; /


EXECUTE dbms_output.put_line(get_sal(100))


```

# 2) SQL Server Triggers

*Overview*

A trigger:
Is a PL/SQL block or a PL/SQL procedure associated with a
table, view, schema, or database
Executes implicitly whenever a particular event takes place
Can be either of the following:
Application trigger:Fires whenever an event occurs with a particular application 
Database trigger: Fires whenever a data event (such as DML) or system event (such as logon or shutdown) occurs on a schema or database


There are two main types of triggers in SQL Server:

    - AFTER triggers, which run after the operation is completed.
    - INSTEAD OF triggers, which replace the original operation with custom logic.

SQL Server provides two virtual tables inside triggers:

    - inserted, holds new records
    - deleted, holds old records that were modified or deleted

•Types of DML Triggers:
The trigger type determines whether the body executes for each row or only once for the triggering statement.

-A statement trigger:
Executes once for the triggering event Is the default type of trigger Fires once even if no rows are affected at all

-A row trigger:
Executes once for each row affected by the triggering event Is not executed if the triggering event does not affect any rows Is indicated by specifying the FOR EACH ROW clause

• Trigger Timing
When should the trigger fire?
  BEFORE: Execute the trigger body before the triggering DML
event on a table.
  AFTER: Execute the trigger body after the triggering DML
event on a table.
  INSTEAD OF: Execute the trigger body instead of the
triggering statement. This is used for views that are not
otherwise modifiable. Note: If multiple triggers are defined for the same object, then the order of firing triggers is arbitrary.


  Example —  trigger 


```
CREATE [OR REPLACE] TRIGGER trigger_name
timing
event1 [OR event2 OR event3]
ON object_name
[[REFERENCING OLD AS old | NEW AS new]
FOR EACH ROW
[WHEN (condition)]]
trigger_body



```

# 3) Views

*Overview*

A View is a virtual table created from a SQL query. It doesn’t store data itself but displays data from one or more tables. Views are perfect for simplifying complex queries, improving security, and reusing logic across multiple parts of your application.

Benefits of using Views:

    - Simplifies complex joins and aggregations.
	-	Enhances security by limiting access to base tables.
	-	Makes your code more modular and easier to maintain.
	-	Can improve performance using Indexed Views, which store results physically for faster access.

Example — Simple view


```sql
CREATE VIEW dbo.vw_CustomerOrders
AS
SELECT c.CustomerID, c.Name, o.OrderID, o.OrderDate, o.TotalAmount
FROM dbo.Customers c
JOIN dbo.Orders o ON c.CustomerID = o.CustomerID;

```

Example — Summary view


```sql

CREATE VIEW dbo.vw_CustomerOrderSummary
AS
SELECT c.CustomerID, c.Name,
       COUNT(o.OrderID) AS OrdersCount,
       SUM(o.TotalAmount) AS TotalSpent
FROM dbo.Customers c
LEFT JOIN dbo.Orders o ON c.CustomerID = o.CustomerID
GROUP BY c.CustomerID, c.Name;
```



  

  


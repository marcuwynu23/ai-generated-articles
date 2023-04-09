# How to Create SQL Procedures and Functions for MySQL, MSSQL Server, PostgreSQL, and SQLite with Code Examples

Structured Query Language (SQL) is a widely used programming language for managing and manipulating data in relational database management systems (RDBMS). SQL provides a set of commands for creating, modifying, and querying databases. SQL procedures and functions are database objects that can be created and used to perform specific tasks. In this article, we will explore how to create SQL procedures and functions for MySQL, MSSQL Server, PostgreSQL, and SQLite, along with code examples and how to use them.

Creating SQL Procedures and Functions for MySQL
MySQL is an open-source relational database management system that is widely used for web applications. Here's how to create a SQL procedure and function in MySQL:
<br>

### **Creating SQL Procedure in MySQL:**

```sql
CREATE PROCEDURE procedure_name (IN param1 datatype1, OUT param2 datatype2)
BEGIN
   -- code to perform task
END;
```

### **Creating SQL Function in MySQL:**

```sql
CREATE FUNCTION function_name (param1 datatype1, param2 datatype2)
RETURNS datatype
BEGIN
   -- code to perform task
END;
```

**Example:**

```sql
CREATE PROCEDURE get_employee(IN emp_id INT, OUT emp_name VARCHAR(50))
BEGIN
   SELECT employee_name INTO emp_name FROM employees WHERE employee_id = emp_id;
END;

CREATE FUNCTION get_salary(emp_id INT) RETURNS INT
BEGIN
   DECLARE salary INT;
   SELECT employee_salary INTO salary FROM employees WHERE employee_id = emp_id;
   RETURN salary;
END;
```

To use the above-created procedure and function, execute the following statements:

```sql
CALL get_employee(101, @emp_name);
SELECT @emp_name;

SELECT get_salary(101);
```

Creating SQL Procedures and Functions for MSSQL Server
Microsoft SQL Server is a relational database management system developed by Microsoft. Here's how to create a SQL procedure and function in MSSQL Server:
<br>

### **Creating SQL Procedure in MSSQL Server:**

```sql
CREATE PROCEDURE procedure_name
@param1 datatype1,
@param2 datatype2 OUTPUT
AS
BEGIN
-- code to perform task
END;
```

### Creating SQL Function in MSSQL Server:

```sql
CREATE FUNCTION function_name
(@param1 datatype1,
@param2 datatype2)
RETURNS datatype
AS
BEGIN
-- code to perform task
END;
```

**Example:**

```sql
CREATE PROCEDURE get_employee
@emp_id INT,
@emp_name VARCHAR(50) OUTPUT
AS
BEGIN
SELECT @emp_name = employee_name FROM employees WHERE employee_id = @emp_id;
END;

CREATE FUNCTION get_salary
(@emp_id INT)
RETURNS INT
AS
BEGIN
DECLARE @salary INT;
SELECT @salary = employee_salary FROM employees WHERE employee_id = @emp_id;
RETURN @salary;
END;
```

To use the above-created procedure and function, execute the following statements:

```sql
DECLARE @emp_name VARCHAR(50);
EXEC get_employee 101, @emp_name OUTPUT;
SELECT @emp_name;

SELECT dbo.get_salary(101);
```

Creating SQL Procedures and Functions for PostgreSQL
PostgreSQL is an open-source relational database management system that is known for its powerful features and scalability. Here's how to create a SQL procedure and function in PostgreSQL:

**###**

### **Creating SQL Procedure in PostgreSQL:**

```sql
CREATE OR REPLACE PROCEDURE procedure_name
(IN param1 datatype1, OUT param2 datatype2)
AS $$
BEGIN
-- code to perform task
END;

$$
LANGUAGE plpgsql;
```

### Creating SQL Function in PostgreSQL:

```sql
CREATE OR REPLACE FUNCTION function_name
   (param1 datatype1, param2 datatype2)
RETURNS datatype
AS
$$

BEGIN
-- code to perform task
END;
```

**Example:**

```sql
CREATE OR REPLACE PROCEDURE get_employee
(IN emp_id INT, OUT emp_name VARCHAR(50))
AS $$
BEGIN
SELECT employee_name INTO emp_name FROM employees WHERE employee_id = emp_id;
END;
$$ LANGUAGE plpgsql;
CREATE OR REPLACE FUNCTION get_salary
(emp_id INT)
RETURNS INT
AS $$
DECLARE salary INT;
BEGIN
SELECT employee_salary INTO salary FROM employees WHERE employee_id = emp_id;
RETURN salary;
END;
$$ LANGUAGE plpgsql;
```

To use the above-created procedure and function, execute the following statements:

```sql
CALL get_employee(101, emp_name);
SELECT emp_name;
SELECT get_salary(101);
```

### Creating SQL Procedures and Functions for SQLite

SQLite is a lightweight, open-source relational database management system that is often used in embedded systems and mobile applications. Here's how to create a SQL procedure and function in SQLite:
Creating SQL Procedure in SQLite:

```sql
CREATE PROCEDURE procedure_name (param1 datatype1, param2 datatype2)
BEGIN
-- code to perform task
END;
```

### Creating SQL Function in SQLite:

```sql
CREATE FUNCTION function_name (param1 datatype1, param2 datatype2)
RETURNS datatype
BEGIN
-- code to perform task
END;
```

**Example:**

```sql
CREATE PROCEDURE get_employee(emp_id INT, emp_name VARCHAR(50))
BEGIN
SELECT employee_name INTO emp_name FROM employees WHERE employee_id = emp_id;
END;
CREATE FUNCTION get_salary(emp_id INT) RETURNS INT
BEGIN
DECLARE salary INT;
SELECT employee_salary INTO salary FROM employees WHERE employee_id = emp_id;
RETURN salary;
END;
```

To use the above-created procedure and function, execute the following statements:

```sql
CALL get_employee(101, 'emp_name');
SELECT 'emp_name';
SELECT get_salary(101);
```

In conclusion, SQL procedures and functions are powerful tools for managing and manipulating data in relational database management systems. By following the above examples, you can create SQL procedures and functions for MySQL, MSSQL Server, PostgreSQL, and SQLite, and use them to perform specific tasks.

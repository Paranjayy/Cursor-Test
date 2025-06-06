---
title: "SQL Fundamentals"
description: "Master the core concepts and syntax of SQL"
tags: ["sql", "basics", "fundamentals"]
---

# SQL Fundamentals 📚

SQL (Structured Query Language) is the standard language for managing and manipulating relational databases. Let's dive into the core concepts!

## 🎯 What is SQL?

SQL is a declarative language that allows you to:
- **Query** data from databases
- **Insert, Update, Delete** records
- **Create and modify** database structures
- **Control access** to data

## 🏗️ Basic SQL Operations (CRUD)

### CREATE - Adding Data

```sql
-- Creating a table
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE
);

-- Inserting data
INSERT INTO employees (id, name, department, salary, hire_date)
VALUES 
    (1, 'Alice Johnson', 'Engineering', 75000.00, '2023-01-15'),
    (2, 'Bob Smith', 'Marketing', 65000.00, '2023-02-20'),
    (3, 'Carol Davis', 'Engineering', 80000.00, '2023-01-10');
```

### READ - Querying Data

```sql
-- Select all data
SELECT * FROM employees;

-- Select specific columns
SELECT name, salary FROM employees;

-- Filter with WHERE clause
SELECT name, salary 
FROM employees 
WHERE department = 'Engineering';

-- Order results
SELECT name, salary 
FROM employees 
ORDER BY salary DESC;

-- Limit results
SELECT name, salary 
FROM employees 
ORDER BY salary DESC 
LIMIT 2;
```

### UPDATE - Modifying Data

```sql
-- Update a single record
UPDATE employees 
SET salary = 78000.00 
WHERE id = 1;

-- Update multiple records
UPDATE employees 
SET salary = salary * 1.05 
WHERE department = 'Marketing';
```

### DELETE - Removing Data

```sql
-- Delete specific records
DELETE FROM employees 
WHERE id = 3;

-- Delete all records (be careful!)
DELETE FROM employees;
```

## 🔍 Essential SQL Clauses

### WHERE - Filtering Data

```sql
-- Exact match
SELECT * FROM employees WHERE department = 'Engineering';

-- Numeric comparisons
SELECT * FROM employees WHERE salary > 70000;

-- Date comparisons
SELECT * FROM employees WHERE hire_date >= '2023-02-01';

-- Multiple conditions
SELECT * FROM employees 
WHERE department = 'Engineering' AND salary > 70000;

-- Pattern matching
SELECT * FROM employees WHERE name LIKE 'A%'; -- Names starting with 'A'
```

### GROUP BY - Aggregating Data

```sql
-- Count employees by department
SELECT department, COUNT(*) as employee_count
FROM employees
GROUP BY department;

-- Average salary by department
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department;

-- Filter groups with HAVING
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;
```

## 📊 Common SQL Functions

### Aggregate Functions

```sql
-- Count records
SELECT COUNT(*) FROM employees;

-- Sum values
SELECT SUM(salary) FROM employees;

-- Average, Min, Max
SELECT 
    AVG(salary) as avg_salary,
    MIN(salary) as min_salary,
    MAX(salary) as max_salary
FROM employees;
```

### String Functions

```sql
-- Concatenation
SELECT CONCAT(name, ' - ', department) as employee_info
FROM employees;

-- Upper/Lower case
SELECT UPPER(name), LOWER(department)
FROM employees;

-- String length
SELECT name, LENGTH(name) as name_length
FROM employees;
```

### Date Functions

```sql
-- Current date/time
SELECT NOW(), CURDATE(), CURTIME();

-- Date formatting
SELECT name, DATE_FORMAT(hire_date, '%Y-%m') as hire_month
FROM employees;

-- Date calculations
SELECT name, DATEDIFF(NOW(), hire_date) as days_employed
FROM employees;
```

## 🎓 Practice Exercises

Try these queries with the sample data:

1. **Find all employees hired in 2023**
```sql
SELECT * FROM employees 
WHERE YEAR(hire_date) = 2023;
```

2. **Calculate total payroll by department**
```sql
SELECT department, SUM(salary) as total_payroll
FROM employees
GROUP BY department;
```

3. **Find the highest paid employee**
```sql
SELECT * FROM employees 
WHERE salary = (SELECT MAX(salary) FROM employees);
```

## 🔗 What's Next?

Now that you understand the basics, explore:
- [[Data Types and Operations]] - Deep dive into SQL data types
- [[Joins and Relationships]] - Connect multiple tables
- [[Advanced Queries]] - Subqueries, CTEs, and window functions

## 💡 Key Takeaways

- SQL is **declarative** - you specify what you want, not how to get it
- Always use **WHERE** clauses to filter data efficiently
- **GROUP BY** is essential for aggregating data
- Practice with real data to solidify your understanding

---

*Ready to level up? Check out [[Data Types and Operations]] next!* 
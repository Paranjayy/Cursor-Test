---
title: "Joins and Relationships"
description: "Master SQL joins to connect and query multiple tables effectively"
tags: ["sql", "joins", "relationships", "advanced"]
---

# Joins and Relationships 🔗

Joins are the heart of relational databases! They allow you to combine data from multiple tables to answer complex questions.

## 🎯 Understanding Table Relationships

### Sample Database Schema

Let's work with a simple e-commerce database:

```sql
-- Customers table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50)
);

-- Orders table
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Products table
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2)
);

-- Order items table
CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

### Sample Data

```sql
-- Insert sample customers
INSERT INTO customers VALUES
(1, 'Alice Johnson', 'alice@email.com', 'New York'),
(2, 'Bob Smith', 'bob@email.com', 'Los Angeles'),
(3, 'Carol Davis', 'carol@email.com', 'Chicago'),
(4, 'David Wilson', 'david@email.com', 'Houston');

-- Insert sample orders
INSERT INTO orders VALUES
(101, 1, '2024-01-15', 150.00),
(102, 2, '2024-01-20', 89.99),
(103, 1, '2024-02-01', 220.50),
(104, 3, '2024-02-10', 75.25);

-- Insert sample products
INSERT INTO products VALUES
(1, 'Laptop', 'Electronics', 999.99),
(2, 'Coffee Mug', 'Home', 12.99),
(3, 'Book', 'Education', 24.99),
(4, 'Headphones', 'Electronics', 79.99);
```

## 🔄 Types of Joins

### INNER JOIN - Only Matching Records

Returns records that have matching values in both tables.

```sql
-- Get customers with their orders
SELECT 
    c.name,
    c.email,
    o.order_id,
    o.order_date,
    o.total_amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

**Result:** Only customers who have placed orders.

### LEFT JOIN - All Records from Left Table

Returns all records from the left table, and matched records from right table.

```sql
-- Get all customers, including those without orders
SELECT 
    c.name,
    c.email,
    o.order_id,
    o.total_amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

**Result:** All customers, with NULL values for customers without orders.

### RIGHT JOIN - All Records from Right Table

Returns all records from the right table, and matched records from left table.

```sql
-- Get all orders with customer information
SELECT 
    c.name,
    o.order_id,
    o.order_date,
    o.total_amount
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```

### FULL OUTER JOIN - All Records from Both Tables

```sql
-- Get all customers and all orders (MySQL doesn't support FULL OUTER JOIN directly)
-- Workaround using UNION:
SELECT c.name, o.order_id, o.total_amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
UNION
SELECT c.name, o.order_id, o.total_amount
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```

## 🔄 Advanced Join Patterns

### Multiple Table Joins

```sql
-- Get order details with customer and product information
SELECT 
    c.name as customer_name,
    o.order_date,
    p.product_name,
    oi.quantity,
    p.price,
    (oi.quantity * p.price) as line_total
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
ORDER BY c.name, o.order_date;
```

### Self Joins

Useful for hierarchical data or comparing rows within the same table.

```sql
-- Example: Employee hierarchy
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT
);

-- Find employees with their managers
SELECT 
    e.name as employee,
    m.name as manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;
```

### Conditional Joins

```sql
-- Join with additional conditions
SELECT 
    c.name,
    o.order_date,
    o.total_amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id 
    AND o.total_amount > 100;
```

## 📊 Practical Join Examples

### Customer Order Summary

```sql
-- Customer order statistics
SELECT 
    c.name,
    c.city,
    COUNT(o.order_id) as order_count,
    COALESCE(SUM(o.total_amount), 0) as total_spent,
    COALESCE(AVG(o.total_amount), 0) as avg_order_value
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name, c.city
ORDER BY total_spent DESC;
```

### Product Sales Analysis

```sql
-- Product sales performance
SELECT 
    p.product_name,
    p.category,
    COUNT(oi.item_id) as times_ordered,
    SUM(oi.quantity) as total_quantity_sold,
    SUM(oi.quantity * p.price) as total_revenue
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY p.product_id, p.product_name, p.category
ORDER BY total_revenue DESC;
```

### Monthly Sales Report

```sql
-- Sales by month with customer details
SELECT 
    DATE_FORMAT(o.order_date, '%Y-%m') as month,
    COUNT(DISTINCT o.customer_id) as unique_customers,
    COUNT(o.order_id) as total_orders,
    SUM(o.total_amount) as monthly_revenue
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
GROUP BY DATE_FORMAT(o.order_date, '%Y-%m')
ORDER BY month;
```

## ⚡ Join Performance Tips

### 1. Use Proper Indexes
```sql
-- Create indexes on join columns
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

### 2. Filter Early
```sql
-- Good: Filter before joining
SELECT c.name, o.order_date
FROM customers c
INNER JOIN (
    SELECT * FROM orders 
    WHERE order_date >= '2024-01-01'
) o ON c.customer_id = o.customer_id;
```

### 3. Use EXISTS for Existence Checks
```sql
-- More efficient than JOIN for existence checks
SELECT name 
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.customer_id = c.customer_id
);
```

## 🎓 Practice Exercises

1. **Find customers who haven't placed any orders**
```sql
SELECT c.name, c.email
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

2. **Get the most popular product by quantity sold**
```sql
SELECT p.product_name, SUM(oi.quantity) as total_sold
FROM products p
INNER JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY p.product_id, p.product_name
ORDER BY total_sold DESC
LIMIT 1;
```

3. **Find customers who ordered in multiple months**
```sql
SELECT 
    c.name,
    COUNT(DISTINCT DATE_FORMAT(o.order_date, '%Y-%m')) as months_active
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name
HAVING months_active > 1;
```

## 🔗 What's Next?

Master these join concepts, then explore:
- [[Advanced Queries]] - Subqueries, CTEs, and window functions
- [[Performance Optimization]] - Making your joins faster
- [[Database Design]] - Designing effective relationships

## 💡 Key Takeaways

- **INNER JOIN** for exact matches only
- **LEFT JOIN** to keep all records from the left table
- **Multiple joins** can answer complex business questions
- Always **index your join columns** for better performance
- **Filter early** to reduce the data being joined

---

*Ready for more complex queries? Check out [[Advanced Queries]] next!* 
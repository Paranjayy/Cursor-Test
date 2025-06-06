---
title: "Advanced Queries"
description: "Master complex SQL patterns including subqueries, CTEs, window functions, and advanced techniques"
tags: ["sql", "advanced", "subqueries", "cte", "window-functions"]
---

# Advanced Queries 🚀

Ready to level up your SQL skills? Let's dive into advanced query patterns that will make you a SQL power user!

## 🎯 Subqueries

### Scalar Subqueries

```sql
-- Get products with above-average prices
SELECT 
    product_name,
    price,
    (SELECT AVG(price) FROM products) as avg_price,
    price - (SELECT AVG(price) FROM products) as price_difference
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

### Correlated Subqueries

```sql
-- Find customers with above-average order amounts
SELECT 
    c.customer_id,
    c.name,
    (
        SELECT COUNT(*) 
        FROM orders o 
        WHERE o.customer_id = c.customer_id
    ) as order_count,
    (
        SELECT AVG(total_amount) 
        FROM orders o 
        WHERE o.customer_id = c.customer_id
    ) as avg_order_value
FROM customers c
WHERE (
    SELECT AVG(total_amount) 
    FROM orders o 
    WHERE o.customer_id = c.customer_id
) > (
    SELECT AVG(total_amount) 
    FROM orders
);
```

### EXISTS vs IN

```sql
-- Using EXISTS (often more efficient)
SELECT customer_id, name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.customer_id = c.customer_id 
    AND o.order_date >= '2024-01-01'
);

-- Using IN
SELECT customer_id, name
FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id 
    FROM orders 
    WHERE order_date >= '2024-01-01'
);
```

## 🌟 Common Table Expressions (CTEs)

### Basic CTE

```sql
-- Calculate monthly sales with CTE
WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') as month,
        SUM(total_amount) as total_sales,
        COUNT(*) as order_count
    FROM orders
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
)
SELECT 
    month,
    total_sales,
    order_count,
    total_sales / order_count as avg_order_value
FROM monthly_sales
ORDER BY month;
```

### Multiple CTEs

```sql
-- Customer segmentation analysis
WITH customer_stats AS (
    SELECT 
        customer_id,
        COUNT(*) as order_count,
        SUM(total_amount) as total_spent,
        MAX(order_date) as last_order_date
    FROM orders
    GROUP BY customer_id
),
customer_segments AS (
    SELECT 
        customer_id,
        order_count,
        total_spent,
        DATEDIFF(CURDATE(), last_order_date) as days_since_last_order,
        CASE 
            WHEN order_count >= 10 AND total_spent >= 1000 THEN 'VIP'
            WHEN order_count >= 5 AND total_spent >= 500 THEN 'Premium'
            WHEN order_count >= 3 THEN 'Regular'
            ELSE 'New'
        END as segment
    FROM customer_stats
)
SELECT 
    segment,
    COUNT(*) as customer_count,
    AVG(total_spent) as avg_spent,
    AVG(days_since_last_order) as avg_days_since_order
FROM customer_segments
GROUP BY segment
ORDER BY avg_spent DESC;
```

### Recursive CTEs

```sql
-- Employee hierarchy traversal
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor: Top level managers
    SELECT 
        employee_id,
        name,
        manager_id,
        0 as level,
        CAST(name AS CHAR(1000)) as hierarchy_path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Add subordinates
    SELECT 
        e.employee_id,
        e.name,
        e.manager_id,
        eh.level + 1,
        CONCAT(eh.hierarchy_path, ' > ', e.name)
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT 
    employee_id,
    CONCAT(REPEAT('  ', level), name) as indented_name,
    level,
    hierarchy_path
FROM employee_hierarchy
ORDER BY hierarchy_path;
```

## 📊 Window Functions

### Basic Window Functions

```sql
-- Running totals and rankings
SELECT 
    order_date,
    customer_id,
    total_amount,
    -- Running total
    SUM(total_amount) OVER (
        ORDER BY order_date 
        ROWS UNBOUNDED PRECEDING
    ) as running_total,
    -- Row number
    ROW_NUMBER() OVER (ORDER BY total_amount DESC) as rank_by_amount,
    -- Dense rank
    DENSE_RANK() OVER (ORDER BY total_amount DESC) as dense_rank,
    -- Percentage rank
    PERCENT_RANK() OVER (ORDER BY total_amount) as percentile
FROM orders;
```

### Partitioned Window Functions

```sql
-- Customer analysis with window functions
SELECT 
    customer_id,
    order_date,
    total_amount,
    -- Rank within customer orders
    ROW_NUMBER() OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) as order_sequence,
    -- Customer's average order value
    AVG(total_amount) OVER (
        PARTITION BY customer_id
    ) as customer_avg_order,
    -- Previous order amount
    LAG(total_amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) as previous_order_amount,
    -- Next order amount
    LEAD(total_amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) as next_order_amount
FROM orders;
```

### Moving Averages

```sql
-- 3-month moving average of sales
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') as month,
    SUM(total_amount) as monthly_sales,
    AVG(SUM(total_amount)) OVER (
        ORDER BY DATE_FORMAT(order_date, '%Y-%m')
        ROWS 2 PRECEDING
    ) as three_month_avg
FROM orders
GROUP BY DATE_FORMAT(order_date, '%Y-%m')
ORDER BY month;
```

## 🔄 Advanced JOINs

### Self Joins

```sql
-- Find customers who placed orders on consecutive days
SELECT DISTINCT
    o1.customer_id,
    o1.order_date as first_date,
    o2.order_date as second_date
FROM orders o1
INNER JOIN orders o2 ON o1.customer_id = o2.customer_id
    AND o2.order_date = DATE_ADD(o1.order_date, INTERVAL 1 DAY);
```

### Multiple Table Complex Joins

```sql
-- Complete order details with customer and product info
SELECT 
    c.name as customer_name,
    o.order_date,
    p.product_name,
    oi.quantity,
    p.price,
    (oi.quantity * p.price) as line_total,
    -- Order totals
    SUM(oi.quantity * p.price) OVER (
        PARTITION BY o.order_id
    ) as order_total,
    -- Customer lifetime value
    SUM(oi.quantity * p.price) OVER (
        PARTITION BY c.customer_id
    ) as customer_ltv
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
ORDER BY c.customer_id, o.order_date;
```

## 🎭 CASE Statements and Conditional Logic

### Complex CASE Statements

```sql
-- Advanced customer segmentation
SELECT 
    customer_id,
    name,
    total_orders,
    total_spent,
    avg_order_value,
    CASE 
        WHEN total_spent >= 5000 AND avg_order_value >= 200 THEN 'VIP High-Value'
        WHEN total_spent >= 2000 AND total_orders >= 10 THEN 'VIP Frequent'
        WHEN total_spent >= 1000 OR avg_order_value >= 150 THEN 'Premium'
        WHEN total_orders >= 5 THEN 'Regular'
        WHEN total_orders >= 2 THEN 'Developing'
        ELSE 'New'
    END as customer_segment,
    CASE 
        WHEN DATEDIFF(CURDATE(), last_order_date) <= 30 THEN 'Active'
        WHEN DATEDIFF(CURDATE(), last_order_date) <= 90 THEN 'At Risk'
        ELSE 'Churned'
    END as activity_status
FROM (
    SELECT 
        c.customer_id,
        c.name,
        COUNT(o.order_id) as total_orders,
        COALESCE(SUM(o.total_amount), 0) as total_spent,
        COALESCE(AVG(o.total_amount), 0) as avg_order_value,
        MAX(o.order_date) as last_order_date
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.name
) customer_summary;
```

## 🔢 Advanced Aggregations

### ROLLUP and CUBE

```sql
-- Sales summary with subtotals
SELECT 
    COALESCE(category, 'ALL CATEGORIES') as category,
    COALESCE(DATE_FORMAT(order_date, '%Y-%m'), 'ALL MONTHS') as month,
    SUM(quantity * price) as total_sales,
    COUNT(*) as transaction_count
FROM order_items oi
INNER JOIN products p ON oi.product_id = p.product_id
INNER JOIN orders o ON oi.order_id = o.order_id
GROUP BY category, DATE_FORMAT(order_date, '%Y-%m') WITH ROLLUP;
```

### GROUP_CONCAT

```sql
-- Aggregate related data into strings
SELECT 
    customer_id,
    name,
    COUNT(order_id) as order_count,
    GROUP_CONCAT(
        CONCAT(order_date, ': $', total_amount) 
        ORDER BY order_date 
        SEPARATOR '; '
    ) as order_history
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
GROUP BY customer_id, name;
```

## 🎯 Performance-Optimized Queries

### Using Indexes Effectively

```sql
-- Query optimized for compound index on (customer_id, order_date)
SELECT customer_id, order_date, total_amount
FROM orders
WHERE customer_id = 123
    AND order_date BETWEEN '2024-01-01' AND '2024-12-31'
ORDER BY order_date DESC;

-- Covering index includes all needed columns
CREATE INDEX idx_orders_covering 
ON orders(customer_id, order_date, total_amount);
```

### Efficient Pagination

```sql
-- Better pagination using WHERE instead of OFFSET
SELECT *
FROM orders
WHERE order_id > 1000  -- Last seen ID from previous page
ORDER BY order_id
LIMIT 20;

-- Instead of slow OFFSET
-- SELECT * FROM orders ORDER BY order_id LIMIT 20 OFFSET 1000;
```

## 🎓 Real-World Advanced Examples

### Customer Cohort Analysis

```sql
WITH customer_cohorts AS (
    SELECT 
        customer_id,
        DATE_FORMAT(MIN(order_date), '%Y-%m') as cohort_month,
        MIN(order_date) as first_order_date
    FROM orders
    GROUP BY customer_id
),
cohort_data AS (
    SELECT 
        cc.cohort_month,
        PERIOD_DIFF(
            DATE_FORMAT(o.order_date, '%Y%m'),
            DATE_FORMAT(cc.first_order_date, '%Y%m')
        ) as period_number,
        COUNT(DISTINCT o.customer_id) as customers
    FROM customer_cohorts cc
    INNER JOIN orders o ON cc.customer_id = o.customer_id
    GROUP BY cc.cohort_month, period_number
),
cohort_table AS (
    SELECT 
        cohort_month,
        SUM(CASE WHEN period_number = 0 THEN customers END) as month_0,
        SUM(CASE WHEN period_number = 1 THEN customers END) as month_1,
        SUM(CASE WHEN period_number = 2 THEN customers END) as month_2,
        SUM(CASE WHEN period_number = 3 THEN customers END) as month_3
    FROM cohort_data
    GROUP BY cohort_month
)
SELECT 
    cohort_month,
    month_0,
    ROUND(month_1 * 100.0 / month_0, 2) as month_1_retention,
    ROUND(month_2 * 100.0 / month_0, 2) as month_2_retention,
    ROUND(month_3 * 100.0 / month_0, 2) as month_3_retention
FROM cohort_table
ORDER BY cohort_month;
```

### Dynamic Pivoting

```sql
-- Monthly sales by category (pivot table)
SELECT 
    category,
    SUM(CASE WHEN month = '2024-01' THEN sales END) as jan_2024,
    SUM(CASE WHEN month = '2024-02' THEN sales END) as feb_2024,
    SUM(CASE WHEN month = '2024-03' THEN sales END) as mar_2024,
    SUM(sales) as total_sales
FROM (
    SELECT 
        p.category,
        DATE_FORMAT(o.order_date, '%Y-%m') as month,
        SUM(oi.quantity * p.price) as sales
    FROM order_items oi
    INNER JOIN products p ON oi.product_id = p.product_id
    INNER JOIN orders o ON oi.order_id = o.order_id
    GROUP BY p.category, DATE_FORMAT(o.order_date, '%Y-%m')
) monthly_sales
GROUP BY category;
```

## 💡 Best Practices

1. **Use CTEs** for complex logic breakdown
2. **Choose appropriate window functions** over subqueries when possible
3. **Test query performance** with EXPLAIN
4. **Use indexes** to support your complex queries
5. **Break down complex queries** into readable parts
6. **Consider query caching** for expensive operations

## 🔗 What's Next?

Master these advanced techniques, then explore:
- [[Performance Optimization]] - Making complex queries fast
- [[Real World Examples]] - See these patterns in action
- [[Database Design]] - Design schemas that support complex queries

---

*Now you're thinking with advanced SQL! 💪* 
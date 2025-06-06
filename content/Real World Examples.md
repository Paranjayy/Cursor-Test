---
title: "Real World Examples"
description: "Practical SQL scenarios from real business applications"
tags: ["sql", "examples", "business", "practical"]
---

# Real World Examples 💼

Let's apply SQL to solve actual business problems! These examples simulate real-world scenarios you'll encounter.

## 🏪 E-commerce Analytics

### Sales Performance Dashboard

```sql
-- Daily sales summary with key metrics
SELECT 
    DATE(order_date) as sales_date,
    COUNT(DISTINCT customer_id) as unique_customers,
    COUNT(order_id) as total_orders,
    SUM(total_amount) as daily_revenue,
    AVG(total_amount) as avg_order_value,
    SUM(total_amount) / COUNT(DISTINCT customer_id) as revenue_per_customer
FROM orders 
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
GROUP BY DATE(order_date)
ORDER BY sales_date DESC;
```

### Customer Segmentation (RFM Analysis)

```sql
-- Recency, Frequency, Monetary analysis
WITH customer_metrics AS (
    SELECT 
        customer_id,
        DATEDIFF(CURDATE(), MAX(order_date)) as recency_days,
        COUNT(order_id) as frequency,
        SUM(total_amount) as monetary_value
    FROM orders
    GROUP BY customer_id
),
rfm_scores AS (
    SELECT 
        customer_id,
        recency_days,
        frequency,
        monetary_value,
        CASE 
            WHEN recency_days <= 30 THEN 5
            WHEN recency_days <= 90 THEN 4
            WHEN recency_days <= 180 THEN 3
            WHEN recency_days <= 365 THEN 2
            ELSE 1
        END as recency_score,
        CASE 
            WHEN frequency >= 10 THEN 5
            WHEN frequency >= 7 THEN 4
            WHEN frequency >= 4 THEN 3
            WHEN frequency >= 2 THEN 2
            ELSE 1
        END as frequency_score,
        CASE 
            WHEN monetary_value >= 1000 THEN 5
            WHEN monetary_value >= 500 THEN 4
            WHEN monetary_value >= 200 THEN 3
            WHEN monetary_value >= 100 THEN 2
            ELSE 1
        END as monetary_score
    FROM customer_metrics
)
SELECT 
    customer_id,
    CASE 
        WHEN recency_score >= 4 AND frequency_score >= 4 THEN 'Champions'
        WHEN recency_score >= 3 AND frequency_score >= 3 THEN 'Loyal Customers'
        WHEN recency_score >= 3 AND frequency_score <= 2 THEN 'Potential Loyalists'
        WHEN recency_score <= 2 AND frequency_score >= 3 THEN 'At Risk'
        WHEN recency_score <= 2 AND frequency_score <= 2 THEN 'Lost Customers'
        ELSE 'New Customers'
    END as customer_segment,
    recency_days,
    frequency,
    monetary_value
FROM rfm_scores
ORDER BY monetary_value DESC;
```

## 📊 Financial Reporting

### Monthly Revenue Trends

```sql
-- Monthly revenue with growth calculations
WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') as month,
        SUM(total_amount) as monthly_revenue
    FROM orders
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
)
SELECT 
    month,
    monthly_revenue,
    LAG(monthly_revenue) OVER (ORDER BY month) as previous_month,
    ROUND(
        ((monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY month)) 
         / LAG(monthly_revenue) OVER (ORDER BY month)) * 100, 2
    ) as growth_rate_percent
FROM monthly_sales
ORDER BY month;
```

### Product Profitability Analysis

```sql
-- Product performance with cost analysis
SELECT 
    p.product_name,
    p.category,
    COUNT(oi.item_id) as orders_count,
    SUM(oi.quantity) as units_sold,
    SUM(oi.quantity * p.price) as gross_revenue,
    p.cost_per_unit * SUM(oi.quantity) as total_cost,
    SUM(oi.quantity * p.price) - (p.cost_per_unit * SUM(oi.quantity)) as profit,
    ROUND(
        ((SUM(oi.quantity * p.price) - (p.cost_per_unit * SUM(oi.quantity))) 
         / SUM(oi.quantity * p.price)) * 100, 2
    ) as profit_margin_percent
FROM products p
INNER JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY p.product_id, p.product_name, p.category, p.cost_per_unit
ORDER BY profit DESC;
```

## 👥 HR Analytics

### Employee Performance Metrics

```sql
-- Quarterly performance review data
SELECT 
    e.employee_id,
    e.name,
    e.department,
    COUNT(p.project_id) as projects_completed,
    AVG(p.performance_rating) as avg_rating,
    SUM(CASE WHEN p.completion_date <= p.deadline THEN 1 ELSE 0 END) as on_time_projects,
    COUNT(p.project_id) - SUM(CASE WHEN p.completion_date <= p.deadline THEN 1 ELSE 0 END) as late_projects,
    ROUND(
        (SUM(CASE WHEN p.completion_date <= p.deadline THEN 1 ELSE 0 END) 
         / COUNT(p.project_id)) * 100, 1
    ) as on_time_percentage
FROM employees e
LEFT JOIN projects p ON e.employee_id = p.assigned_to
WHERE p.completion_date >= DATE_SUB(CURDATE(), INTERVAL 3 MONTH)
GROUP BY e.employee_id, e.name, e.department
ORDER BY avg_rating DESC, on_time_percentage DESC;
```

### Salary Analysis by Department

```sql
-- Compensation analysis with percentiles
SELECT 
    department,
    COUNT(*) as employee_count,
    MIN(salary) as min_salary,
    ROUND(AVG(salary), 2) as avg_salary,
    MAX(salary) as max_salary,
    ROUND(STDDEV(salary), 2) as salary_std_dev,
    -- Calculate percentiles using window functions
    ROUND(
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY salary) 
        OVER (PARTITION BY department), 2
    ) as p25_salary,
    ROUND(
        PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY salary) 
        OVER (PARTITION BY department), 2
    ) as median_salary,
    ROUND(
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY salary) 
        OVER (PARTITION BY department), 2
    ) as p75_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC;
```

## 🏥 Healthcare Data Analysis

### Patient Readmission Analysis

```sql
-- 30-day readmission rates by department
WITH readmissions AS (
    SELECT 
        a1.patient_id,
        a1.admission_date as first_admission,
        a1.department as first_dept,
        MIN(a2.admission_date) as readmission_date,
        DATEDIFF(MIN(a2.admission_date), a1.discharge_date) as days_between
    FROM admissions a1
    INNER JOIN admissions a2 ON a1.patient_id = a2.patient_id
        AND a2.admission_date > a1.discharge_date
        AND a2.admission_date <= DATE_ADD(a1.discharge_date, INTERVAL 30 DAY)
    GROUP BY a1.patient_id, a1.admission_date, a1.department
)
SELECT 
    first_dept as department,
    COUNT(DISTINCT a.patient_id) as total_discharges,
    COUNT(r.patient_id) as readmissions_30_day,
    ROUND(
        (COUNT(r.patient_id) / COUNT(DISTINCT a.patient_id)) * 100, 2
    ) as readmission_rate_percent,
    AVG(r.days_between) as avg_days_to_readmission
FROM admissions a
LEFT JOIN readmissions r ON a.patient_id = r.patient_id 
    AND a.admission_date = r.first_admission
WHERE a.discharge_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
GROUP BY first_dept
ORDER BY readmission_rate_percent DESC;
```

## 🚗 Logistics and Supply Chain

### Inventory Turnover Analysis

```sql
-- Product inventory turnover rates
WITH inventory_metrics AS (
    SELECT 
        p.product_id,
        p.product_name,
        p.category,
        AVG(i.quantity_on_hand) as avg_inventory,
        SUM(s.quantity_sold) as total_sold,
        COUNT(DISTINCT DATE_FORMAT(s.sale_date, '%Y-%m')) as months_active
    FROM products p
    LEFT JOIN inventory i ON p.product_id = i.product_id
    LEFT JOIN sales s ON p.product_id = s.product_id
    WHERE s.sale_date >= DATE_SUB(CURDATE(), INTERVAL 12 MONTH)
    GROUP BY p.product_id, p.product_name, p.category
)
SELECT 
    product_name,
    category,
    avg_inventory,
    total_sold,
    CASE 
        WHEN avg_inventory > 0 THEN ROUND(total_sold / avg_inventory, 2)
        ELSE 0 
    END as inventory_turnover,
    CASE 
        WHEN total_sold > 0 THEN ROUND((avg_inventory / total_sold) * 12, 1)
        ELSE NULL 
    END as months_of_supply,
    CASE 
        WHEN avg_inventory > 0 AND total_sold / avg_inventory >= 4 THEN 'Fast Moving'
        WHEN avg_inventory > 0 AND total_sold / avg_inventory >= 2 THEN 'Medium Moving'
        WHEN avg_inventory > 0 AND total_sold / avg_inventory > 0 THEN 'Slow Moving'
        ELSE 'No Movement'
    END as movement_category
FROM inventory_metrics
ORDER BY inventory_turnover DESC;
```

## 🎯 Marketing Campaign Analysis

### Campaign ROI Analysis

```sql
-- Marketing campaign effectiveness
SELECT 
    c.campaign_name,
    c.campaign_type,
    c.start_date,
    c.end_date,
    c.budget,
    COUNT(DISTINCT co.customer_id) as customers_acquired,
    SUM(o.total_amount) as revenue_generated,
    ROUND(c.budget / COUNT(DISTINCT co.customer_id), 2) as cost_per_acquisition,
    ROUND(
        ((SUM(o.total_amount) - c.budget) / c.budget) * 100, 2
    ) as roi_percent,
    AVG(o.total_amount) as avg_order_value_from_campaign
FROM campaigns c
LEFT JOIN campaign_responses cr ON c.campaign_id = cr.campaign_id
LEFT JOIN customers co ON cr.customer_id = co.customer_id
LEFT JOIN orders o ON co.customer_id = o.customer_id 
    AND o.order_date BETWEEN c.start_date AND DATE_ADD(c.end_date, INTERVAL 30 DAY)
GROUP BY c.campaign_id, c.campaign_name, c.campaign_type, c.start_date, c.end_date, c.budget
ORDER BY roi_percent DESC;
```

## 🔍 Advanced Business Intelligence

### Cohort Analysis

```sql
-- Customer retention cohort analysis
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
        DATE_FORMAT(o.order_date, '%Y-%m') as order_month,
        PERIOD_DIFF(
            DATE_FORMAT(o.order_date, '%Y%m'),
            DATE_FORMAT(cc.first_order_date, '%Y%m')
        ) as period_number,
        COUNT(DISTINCT o.customer_id) as customers
    FROM customer_cohorts cc
    LEFT JOIN orders o ON cc.customer_id = o.customer_id
    GROUP BY cc.cohort_month, DATE_FORMAT(o.order_date, '%Y-%m'), period_number
)
SELECT 
    cohort_month,
    SUM(CASE WHEN period_number = 0 THEN customers END) as month_0,
    SUM(CASE WHEN period_number = 1 THEN customers END) as month_1,
    SUM(CASE WHEN period_number = 2 THEN customers END) as month_2,
    SUM(CASE WHEN period_number = 3 THEN customers END) as month_3,
    -- Calculate retention rates
    ROUND(
        SUM(CASE WHEN period_number = 1 THEN customers END) * 100.0 / 
        SUM(CASE WHEN period_number = 0 THEN customers END), 2
    ) as month_1_retention,
    ROUND(
        SUM(CASE WHEN period_number = 2 THEN customers END) * 100.0 / 
        SUM(CASE WHEN period_number = 0 THEN customers END), 2
    ) as month_2_retention
FROM cohort_data
GROUP BY cohort_month
ORDER BY cohort_month;
```

## 💡 Key Patterns for Real-World SQL

1. **Use CTEs** for complex logic breakdown
2. **Window functions** for rankings and running totals
3. **Date functions** for time-based analysis
4. **CASE statements** for business logic
5. **Proper indexing** for performance
6. **Data validation** and null handling

## 🔗 What's Next?

These examples show SQL in action! Continue with:
- [[Performance Optimization]] - Making these queries faster
- [[Advanced Queries]] - More complex patterns and techniques
- [[Database Design]] - Structuring data for analytics

---

*Apply these patterns to your own business problems! 💪* 
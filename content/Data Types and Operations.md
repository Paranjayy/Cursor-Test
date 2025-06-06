---
title: "Data Types and Operations"
description: "Master SQL data types and essential operations for effective database management"
tags: ["sql", "data-types", "operations", "fundamentals"]
---

# Data Types and Operations 📊

Understanding SQL data types is crucial for designing efficient databases and writing effective queries. Let's explore the most important data types and operations.

## 🔢 Numeric Data Types

### Integer Types

```sql
-- Integer types with different ranges
CREATE TABLE numeric_examples (
    tiny_number TINYINT,        -- -128 to 127
    small_number SMALLINT,      -- -32,768 to 32,767
    medium_number MEDIUMINT,    -- -8,388,608 to 8,388,607
    regular_number INT,         -- -2,147,483,648 to 2,147,483,647
    big_number BIGINT          -- Much larger range
);

-- Unsigned versions (positive only)
CREATE TABLE unsigned_examples (
    positive_int INT UNSIGNED,   -- 0 to 4,294,967,295
    positive_big BIGINT UNSIGNED
);
```

### Decimal Types

```sql
-- Exact decimal numbers
CREATE TABLE financial_data (
    price DECIMAL(10,2),        -- 10 digits total, 2 after decimal
    tax_rate NUMERIC(5,4),      -- 5 digits total, 4 after decimal
    percentage DECIMAL(5,2)     -- Like 100.50
);

-- Floating point (approximate)
CREATE TABLE scientific_data (
    measurement FLOAT,          -- Single precision
    precise_value DOUBLE       -- Double precision
);
```

## 📝 String Data Types

### Fixed vs Variable Length

```sql
CREATE TABLE text_examples (
    country_code CHAR(2),       -- Always 2 characters: 'US', 'UK'
    username VARCHAR(50),       -- Up to 50 characters
    bio TEXT,                  -- Large text (up to 65,535 chars)
    article LONGTEXT           -- Very large text (up to 4GB)
);

-- String operations
SELECT 
    CONCAT(first_name, ' ', last_name) as full_name,
    UPPER(email) as email_upper,
    LENGTH(bio) as bio_length,
    SUBSTRING(phone, 1, 3) as area_code
FROM users;
```

### String Functions in Action

```sql
-- Text manipulation examples
SELECT 
    product_name,
    -- Cleaning and formatting
    TRIM(UPPER(category)) as clean_category,
    -- Pattern matching
    CASE 
        WHEN product_name LIKE '%Premium%' THEN 'High-End'
        WHEN product_name LIKE '%Basic%' THEN 'Entry-Level'
        ELSE 'Standard'
    END as product_tier,
    -- String replacement
    REPLACE(description, 'old feature', 'new feature') as updated_desc
FROM products;
```

## 📅 Date and Time Types

### Date/Time Storage

```sql
CREATE TABLE event_log (
    event_id INT PRIMARY KEY,
    event_date DATE,            -- 'YYYY-MM-DD'
    event_time TIME,            -- 'HH:MM:SS'
    event_datetime DATETIME,    -- 'YYYY-MM-DD HH:MM:SS'
    created_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    event_year YEAR            -- Just the year
);
```

### Date Operations

```sql
-- Date calculations and formatting
SELECT 
    order_date,
    -- Date arithmetic
    DATE_ADD(order_date, INTERVAL 30 DAY) as due_date,
    DATEDIFF(CURDATE(), order_date) as days_ago,
    -- Date formatting
    DATE_FORMAT(order_date, '%W, %M %d, %Y') as formatted_date,
    -- Extracting parts
    YEAR(order_date) as order_year,
    MONTH(order_date) as order_month,
    DAYNAME(order_date) as day_of_week
FROM orders
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH);
```

## ✅ Boolean and Binary Types

### Boolean Logic

```sql
CREATE TABLE user_preferences (
    user_id INT PRIMARY KEY,
    email_notifications BOOLEAN DEFAULT TRUE,
    is_premium BOOL DEFAULT FALSE,
    account_active TINYINT(1) DEFAULT 1  -- Alternative boolean
);

-- Boolean operations
SELECT 
    username,
    CASE 
        WHEN is_premium AND account_active THEN 'Active Premium'
        WHEN NOT is_premium AND account_active THEN 'Active Free'
        ELSE 'Inactive'
    END as account_status
FROM users;
```

## 🎯 JSON Data Type (Modern SQL)

### Working with JSON

```sql
-- JSON column example
CREATE TABLE product_attributes (
    product_id INT PRIMARY KEY,
    specifications JSON
);

-- Insert JSON data
INSERT INTO product_attributes VALUES 
(1, '{"color": "red", "size": "large", "features": ["waterproof", "durable"]}'),
(2, '{"color": "blue", "size": "medium", "weight": 2.5}');

-- Query JSON data
SELECT 
    product_id,
    JSON_EXTRACT(specifications, '$.color') as color,
    JSON_EXTRACT(specifications, '$.size') as size,
    JSON_LENGTH(specifications, '$.features') as feature_count
FROM product_attributes;
```

## 🔄 Data Type Conversion

### Explicit Casting

```sql
-- Converting between types
SELECT 
    product_id,
    CAST(price AS DECIMAL(10,2)) as price_decimal,
    CONVERT(created_date, DATE) as date_only,
    -- String to number
    CAST('123.45' AS FLOAT) as number_value,
    -- Number to string
    CONCAT('Product #', CAST(product_id AS CHAR)) as product_label
FROM products;
```

### Implicit Conversion Examples

```sql
-- SQL handles these automatically (but be careful!)
SELECT 
    '10' + 5 as math_result,          -- Results in 15 (string to number)
    CONCAT(price, ' USD') as price_text, -- Number to string
    order_date + INTERVAL 1 DAY as next_day
FROM orders;
```

## ⚡ Performance Considerations

### Choosing the Right Type

```sql
-- Good: Appropriate sizes
CREATE TABLE optimized_table (
    id MEDIUMINT UNSIGNED AUTO_INCREMENT,  -- Smaller than INT if < 16M records
    status ENUM('active', 'inactive'),     -- More efficient than VARCHAR
    score TINYINT UNSIGNED,                -- 0-255 range
    percentage DECIMAL(5,2),               -- Exact for money
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Avoid: Oversized types
CREATE TABLE wasteful_table (
    id BIGINT,                    -- Overkill for most use cases
    status VARCHAR(255),          -- Too large for simple status
    percentage FLOAT              -- Inexact for financial data
);
```

### Indexing Different Types

```sql
-- Indexes work differently with different types
CREATE INDEX idx_date_range ON orders(order_date);     -- Fast date lookups
CREATE INDEX idx_text_prefix ON products(name(10));    -- Prefix index for text
CREATE INDEX idx_numeric ON sales(amount);             -- Numeric comparisons
```

## 🎓 Practical Examples

### E-commerce Product Catalog

```sql
CREATE TABLE products (
    product_id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    sku VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    cost DECIMAL(10,2),
    weight DECIMAL(8,3),                    -- kg with 3 decimal places
    dimensions JSON,                        -- {"length": 10, "width": 5, "height": 3}
    category ENUM('electronics', 'clothing', 'books', 'home'),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### Financial Transactions

```sql
CREATE TABLE transactions (
    transaction_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    account_id INT UNSIGNED NOT NULL,
    amount DECIMAL(15,2) NOT NULL,          -- Handle large amounts precisely
    currency CHAR(3) DEFAULT 'USD',         -- ISO currency codes
    transaction_type ENUM('debit', 'credit'),
    reference_number VARCHAR(50),
    processed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    -- Ensure precision for financial calculations
    fee DECIMAL(10,4) DEFAULT 0.0000
);
```

## 💡 Best Practices

### 1. Choose Appropriate Sizes
```sql
-- Right-size your data types
TINYINT     -- For small ranges (0-255)
VARCHAR(50) -- For known max lengths
DECIMAL     -- For exact financial calculations
```

### 2. Use Constraints
```sql
CREATE TABLE users (
    age TINYINT UNSIGNED CHECK (age BETWEEN 13 AND 120),
    email VARCHAR(255) NOT NULL UNIQUE,
    status ENUM('active', 'suspended', 'deleted') DEFAULT 'active'
);
```

### 3. Default Values
```sql
CREATE TABLE orders (
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_rush_order BOOLEAN DEFAULT FALSE
);
```

## 🔗 What's Next?

Now that you understand data types, explore:
- [[Joins and Relationships]] - Connecting tables with proper data types
- [[Advanced Queries]] - Complex operations with different data types
- [[Performance Optimization]] - How data types affect query speed

## 💡 Key Takeaways

- **Choose the smallest appropriate data type** for storage efficiency
- **Use DECIMAL for financial data** to avoid rounding errors
- **VARCHAR vs CHAR**: Use VARCHAR for variable length, CHAR for fixed
- **DATE functions are powerful** for time-based analysis
- **JSON support** enables flexible schema designs
- **Proper indexing** depends on data type characteristics

---

*Master data types to build efficient databases! Next: [[Joins and Relationships]]* 
---
title: "PostgreSQL"
description: "Complete guide to PostgreSQL basics, queries, joins, and operations"
tags: ["postgresql", "sql", "database"]
updated: "2026-03-24"
coAuthor: "opencode"
---

# PostgreSQL

---

## Basic Commands

### Connect to database

```bash
psql -U username -d database_name
psql -U postgres  # default superuser
```

### Common commands inside psql

```sql
\l          -- list all databases
\c dbname   -- connect to database
\dt         -- list all tables
\d tablename-- describe table
\du         -- list all users
\q          -- quit
\h          -- help with SQL commands
\x          -- expanded display (better for wide tables)
```

## Create And Manage Databases

```sql
-- Create database
CREATE DATABASE mydb;

-- Drop database
DROP DATABASE mydb;

-- Connect to database
\c mydb
```

## Create And Manage Tables

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INT CHECK (age >= 18),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Common data types
-- SERIAL          auto-increment integer
-- VARCHAR(n)      variable-length string
-- TEXT             unlimited length string
-- INT, BIGINT     integers
-- DECIMAL(p,s)    exact numeric
-- FLOAT, DOUBLE   approximate numeric
-- BOOLEAN         true/false
-- DATE            date only
-- TIMESTAMP       date + time
-- TIMESTAMPTZ     date + time + timezone
-- JSONB           binary JSON

-- Drop table
DROP TABLE users;

-- Add column
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Remove column
ALTER TABLE users DROP COLUMN phone;

-- Rename table
ALTER TABLE users RENAME TO customers;
```

## CRUD Operations

### Create (Insert)

```sql
-- Insert single row
INSERT INTO users (name, email, age)
VALUES ('Alice', 'alice@email.com', 25);

-- Insert multiple rows
INSERT INTO users (name, email, age) VALUES
    ('Bob', 'bob@email.com', 30),
    ('Charlie', 'charlie@email.com', 22);

-- Insert and return
INSERT INTO users (name, email)
VALUES ('Dave', 'dave@email.com')
RETURNING id, name;
```

### Read (Select)

```sql
-- Select all
SELECT * FROM users;

-- Select specific columns
SELECT name, email FROM users;

-- With condition
SELECT * FROM users WHERE age >= 18;

-- Multiple conditions
SELECT * FROM users WHERE age >= 18 AND country = 'USA';

-- Order by
SELECT * FROM users ORDER BY name ASC;
SELECT * FROM users ORDER BY age DESC;

-- Limit
SELECT * FROM users LIMIT 10 OFFSET 20;

-- Distinct
SELECT DISTINCT country FROM users;

-- Like (pattern matching)
SELECT * FROM users WHERE email LIKE '%@gmail.com';
SELECT * FROM users WHERE name LIKE 'A%';  -- starts with A
SELECT * FROM users WHERE name LIKE '%on%'; -- contains 'on'

-- In
SELECT * FROM users WHERE country IN ('USA', 'UK', 'Canada');

-- Between
SELECT * FROM users WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31';
```

### Update

```sql
-- Update single row
UPDATE users SET age = 26 WHERE name = 'Alice';

-- Update multiple rows
UPDATE users SET country = 'USA' WHERE country IS NULL;

-- Update and return
UPDATE users SET age = age + 1 WHERE id = 1 RETURNING id, age;
```

### Delete

```sql
-- Delete row
DELETE FROM users WHERE id = 5;

-- Delete all rows (keep table)
DELETE FROM users;

-- Delete and return
DELETE FROM users WHERE id = 5 RETURNING id, name;
```

## Constraints

```sql
CREATE TABLE examples (
    -- Primary key
    id SERIAL PRIMARY KEY,

    -- Unique
    email VARCHAR(255) UNIQUE,

    -- Not null
    name VARCHAR(100) NOT NULL,

    -- Check
    age INT CHECK (age >= 0 AND age <= 150),

    -- Default value
    status VARCHAR(20) DEFAULT 'active',

    -- Foreign key
    country_id INT REFERENCES countries(id),

    -- Composite primary key
    PRIMARY KEY (user_id, product_id)
);

-- Add constraint after table created
ALTER TABLE users ADD CONSTRAINT unique_email UNIQUE (email);
ALTER TABLE orders ADD CONSTRAINT positive_price CHECK (price > 0);
```

## Joins

### Sample tables for examples

```sql
-- users table
-- id | name
-- 1  | Alice
-- 2  | Bob

-- orders table
-- id | user_id | amount
-- 1  | 1       | 100
-- 2  | 1       | 200
-- 3  | 2       | 150
```

### INNER JOIN

Returns rows with matches in both tables.

```sql
SELECT users.name, orders.amount
FROM users
INNER JOIN orders ON users.id = orders.user_id;
-- Result: Alice, 100 | Alice, 200 | Bob, 150
```

### LEFT JOIN

Returns all rows from left table, matched rows from right.

```sql
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
-- Result: Alice, 100 | Alice, 200 | Bob, 150 | (null if no orders)
```

### RIGHT JOIN

Returns all rows from right table.

```sql
SELECT users.name, orders.amount
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;
```

### Multiple joins

```sql
SELECT u.name, o.amount, p.name as product
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN products p ON o.product_id = p.id;
```

## Aggregations

```sql
-- Count
SELECT COUNT(*) FROM users;
SELECT COUNT(DISTINCT country) FROM users;

-- Sum, Avg, Min, Max
SELECT SUM(amount) FROM orders;
SELECT AVG(age) FROM users;
SELECT MIN(price) FROM products;
SELECT MAX(price) FROM products;

-- Group by
SELECT country, COUNT(*) as user_count
FROM users
GROUP BY country;

-- Having (filter after grouping)
SELECT country, COUNT(*) as count
FROM users
GROUP BY country
HAVING COUNT(*) > 10;

-- Multiple aggregations
SELECT
    country,
    COUNT(*) as total_users,
    AVG(age) as avg_age,
    MAX(age) as max_age
FROM users
GROUP BY country;
```

## Subqueries

```sql
-- Subquery in WHERE
SELECT * FROM users
WHERE age > (SELECT AVG(age) FROM users);

-- Subquery in FROM
SELECT country, avg_age
FROM (SELECT country, AVG(age) as avg_age FROM users GROUP BY country) as stats
WHERE avg_age > 25;

-- Subquery in SELECT
SELECT name,
    (SELECT COUNT(*) FROM orders WHERE orders.user_id = users.id) as order_count
FROM users;

-- IN with subquery
SELECT * FROM products
WHERE category_id IN (SELECT id FROM categories WHERE name = 'Electronics');
```

## Common Functions

### String functions

```sql
SELECT UPPER('hello');        -- HELLO
SELECT LOWER('HELLO');        -- hello
SELECT LENGTH('hello');       -- 5
SELECT TRIM('  hello  ');     -- hello
SELECT CONCAT('Hello', ' ', 'World'); -- Hello World
SELECT SUBSTRING('Hello' FROM 1 FOR 3); -- Hel
SELECT REPLACE('Hello', 'l', 'r'); -- Herro
```

### Date functions

```sql
SELECT CURRENT_DATE;      -- 2024-01-15
SELECT CURRENT_TIMESTAMP; -- 2024-01-15 10:30:00
SELECT NOW();             -- with timezone

SELECT EXTRACT(YEAR FROM created_at);
SELECT EXTRACT(MONTH FROM created_at);

-- Age calculation
SELECT AGE(created_at) FROM users;

-- Date arithmetic
SELECT created_at + INTERVAL '7 days' FROM orders;
SELECT created_at - INTERVAL '1 month' FROM orders;
```

### Math functions

```sql
SELECT ROUND(123.456, 2);  -- 123.46
SELECT ABS(-100);          -- 100
SELECT MOD(10, 3);         -- 1
SELECT POWER(2, 3);       -- 8
SELECT SQRT(16);           -- 4
```

### Coalesce & Null handling

```sql
SELECT COALESCE(phone, 'N/A') FROM users;
SELECT NULLIF(a, b);  -- returns null if a = b, else returns a
```

## Views

```sql
-- Create view
CREATE VIEW active_users AS
SELECT name, email FROM users WHERE status = 'active';

-- Use view
SELECT * FROM active_users;

-- Update view (with conditions)
CREATE OR REPLACE VIEW active_users AS
SELECT name, email, created_at FROM users WHERE status = 'active';

-- Drop view
DROP VIEW active_users;
```

## Indexes

```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_users_age ON users(age DESC);

-- Composite index
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Unique index
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Drop index
DROP INDEX idx_users_email;

-- When to use indexes:
-- - Columns in WHERE clauses
-- - Columns in JOIN conditions
-- - Columns in ORDER BY
-- - High-cardinality columns (many unique values)

-- When NOT to use:
-- - Small tables
-- - Low-cardinality columns (few unique values)
-- - Frequently updated columns
```

## Transactions

```sql
BEGIN;  -- or START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- If everything looks good
COMMIT;

-- If something went wrong
ROLLBACK;
```

### Savepoints

```sql
BEGIN;
INSERT INTO users (name) VALUES ('Alice');
SAVEPOINT sp1;

INSERT INTO users (name) VALUES ('Bob');
ROLLBACK TO SAVEPOINT sp1;

COMMIT;
-- Only Alice will be inserted
```

## Export And Import

### Export to CSV

```sql
COPY users TO '/tmp/users.csv' DELIMITER ',' CSV HEADER;
```

### Import from CSV

```sql
COPY users(name, email, age)
FROM '/tmp/users.csv'
DELIMITER ','
CSV HEADER;
```

## User Management

```sql
-- Create user
CREATE USER alice WITH PASSWORD 'password123';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE mydb TO alice;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO alice;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO alice;

-- Revoke privileges
REVOKE ALL PRIVILEGES ON DATABASE mydb FROM alice;

-- Drop user
DROP USER alice;

-- Change password
ALTER USER alice WITH PASSWORD 'newpassword';
```

## Useful Tips

```sql
-- Get table size
SELECT pg_size_pretty(pg_total_relation_size('users'));

-- Get database size
SELECT pg_size_pretty(pg_database_size('mydb'));

-- Show query execution time
\timing on

-- Explain query (check performance)
EXPLAIN SELECT * FROM users WHERE id = 1;
EXPLAIN ANALYZE SELECT * FROM users WHERE id = 1;

-- List all tables
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'public';

-- List all columns in a table
SELECT column_name, data_type FROM information_schema.columns
WHERE table_name = 'users';
```

---

_Last updated: 2026-03-24_

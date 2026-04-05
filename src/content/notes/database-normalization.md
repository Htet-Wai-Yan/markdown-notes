---
title: "Database Normalization"
description: "Learn database normalization forms (1NF to BCNF) with practical examples"
tags: ["database", "normalization", "sql"]
updated: "2026-03-24"
coAuthor: "opencode"
---

# Database Normalization

This guide follows a relatable example: an **online store** that sells products to customers. We'll see how to fix data step by step.

---

## First Normal Form

**Rule:** Each cell holds one single value. No repeating groups.

Imagine a spreadsheet tracking orders. One column shouldn't hold multiple values.

### Before (not 1NF)

| order_id | customer | products             | total |
| -------- | -------- | -------------------- | ----- |
| 1        | Alice    | Laptop, Mouse, Cable | $1500 |
| 2        | Bob      | Keyboard             | $200  |

The `products` column has multiple items separated by commas. That's messy.

### After (1NF)

| order_id | customer | product  | price |
| -------- | -------- | -------- | ----- |
| 1        | Alice    | Laptop   | $1000 |
| 1        | Alice    | Mouse    | $50   |
| 1        | Alice    | Cable    | $50   |
| 2        | Bob      | Keyboard | $200  |

Each row now holds exactly one product. Clean and simple.

### PostgreSQL

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10, 2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    order_id INT REFERENCES orders(order_id),
    product_id INT REFERENCES products(product_id),
    price_at_purchase DECIMAL(10, 2),
    PRIMARY KEY (order_id, product_id)
);
```

## Second Normal Form

**Rule:** Must be in 1NF first. Every column depends on the entire key.

If your primary key has two columns (composite key), nothing should depend on just one of them.

In our example, `order_items` has a composite key: (`order_id`, `product_id`).

### The Problem

If we added `customer_name` to `order_items`:

| order_id | product_id | customer_name | price |
| -------- | ---------- | ------------- | ----- |
| 1        | 1          | Alice         | $1000 |
| 1        | 2          | Alice         | $50   |

`customer_name` only depends on `order_id`, not on `product_id`. This causes duplication and confusion.

### After (2NF)

Move `customer_name` to the `orders` table where it belongs.

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    order_date TIMESTAMP
);
```

Each table now has data that truly belongs there:

- `orders` → order-level info (customer, date)
- `order_items` → line-item info (product, price paid)

## Third Normal Form

**Rule:** Must be in 2NF first. No column should depend on another non-key column.

If you can calculate one value from others, don't store it.

### The Problem

If we stored `price` and `quantity` in `order_items`, plus the pre-calculated `subtotal`:

| order_id | product_id | price | quantity | subtotal |
| -------- | ---------- | ----- | -------- | -------- |
| 1        | 1          | $1000 | 2        | $2000    |

`subtotal` = `price * quantity`. It can be calculated. Storing it invites errors.

### After (3NF)

Remove `subtotal`. Calculate it when you need it.

```sql
CREATE TABLE order_items (
    order_id INT REFERENCES orders(order_id),
    product_id INT REFERENCES products(product_id),
    quantity INT,
    price_at_purchase DECIMAL(10, 2),
    PRIMARY KEY (order_id, product_id)
);

-- Get subtotal on-the-fly:
SELECT
    order_id,
    product_id,
    quantity,
    price_at_purchase,
    quantity * price_at_purchase AS subtotal
FROM order_items;
```

Store only what can't be derived from other data.

## Boyce-Codd Normal Form

**Rule:** A stricter 3NF. Handles tricky cases where columns determine each other.

### The Problem

Suppose we add support agents to handle orders:

| order_id | customer_id | agent_id | agent_name |
| -------- | ----------- | -------- | ---------- |
| 1        | 100         | A1       | John       |
| 2        | 101         | A1       | John       |
| 3        | 102         | A2       | Jane       |

Rules:

- Each order has one agent
- Each agent handles multiple orders
- `agent_id` determines `agent_name`

But `agent_id` is not a key, and it determines `agent_name`. This violates BCNF.

### After (BCNF)

Split agents into their own table:

```sql
CREATE TABLE agents (
    agent_id VARCHAR(10) PRIMARY KEY,
    agent_name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    agent_id VARCHAR(10) REFERENCES agents(agent_id)
);
```

Now each column's dependencies are on proper keys. No anomalies when inserting, updating, or deleting agents.

## Summary

| Level | Problem It Solves                            | Fix                            | Example in Our Store                                |
| ----- | -------------------------------------------- | ------------------------------ | --------------------------------------------------- |
| 1NF   | Multiple values in one cell                  | Split into separate rows       | Products now one per row instead of comma-separated |
| 2NF   | Columns depending on part of a composite key | Move to their own table        | Customer info moved from order_items to orders      |
| 3NF   | Calculated/derived values stored             | Remove and compute when needed | Subtotal removed, calculated via query              |
| BCNF  | Non-key column determining another column    | Split into separate table      | Agents moved to own table with agent_id as key      |

### Quick Checklist

- [ ] Each cell has one value (1NF)
- [ ] All columns depend on the full key, not just part (2NF)
- [ ] No redundant data that can be calculated (3NF)
- [ ] No column determines another non-key column (BCNF)

### Trade-offs

- **Higher NF** = less redundancy, easier to maintain, fewer update anomalies
- **Lower NF** = faster queries, simpler joins, sometimes better read performance

Don't over-normalize prematurely. Start simple, refactor as needed.

## Rules of Thumb for Schema Design

1. **Start simple, normalize later** - Get it working first, optimize later.

2. **Identify entities first** - What objects exist? (customer, order, product) → each becomes a table.

3. **Find relationships** - One-to-many? Many-to-many? Use foreign keys or junction tables.

4. **Each table one theme** - Don't mix customer info with order info in the same table.

5. **Primary key rule** - Every table needs one. Use auto-increment IDs unless you have a natural key.

6. **Ask: does this column depend on the whole key?** - If no → move to another table.

7. **Ask: can I calculate this from other columns?** - If yes → don't store it.

8. **Denormalize when needed** - Sometimes for performance, you intentionally break NF. That's okay.

9. **Naming** - Use singular, lowercase with underscores: `customer`, `order_item`, not `CustomerOrders`.

10. **Draw it first** - ER diagram before SQL. Helps spot issues early.

---

_Last updated: 2026-03-24_

# MySQL Interview Questions (Fundamentals → Deep Dive) — 100 Q&A + Query Practice

> Focus areas: CRUD, joins, transactions, bulk updates, performance, indexing, security (SQL injection), and real-world ops.  
> Assumes MySQL **8.x** (InnoDB as default storage engine).

---

## 1) Core fundamentals (1–12)

### 1. What is MySQL, and what is InnoDB?
**Answer:** MySQL is an RDBMS; **InnoDB** is the default storage engine in MySQL 8.x. InnoDB supports **ACID transactions**, **row-level locking**, **MVCC**, and **foreign keys**.

### 2. Difference between `CHAR` and `VARCHAR`?
**Answer:**
- `CHAR(n)`: fixed-length; pads spaces; good for predictable sizes (e.g., country code).
- `VARCHAR(n)`: variable-length; stores length + data; better for variable strings.

### 3. What’s the difference between `TEXT` and `VARCHAR`?
**Answer:** `TEXT` is stored differently and has constraints around indexing length and default values. Use `VARCHAR` when you know a reasonable max size and need efficient indexing.

### 4. What is the difference between `NULL` and empty string `''`?
**Answer:** `NULL` means “unknown / missing”. `''` is a known, empty value. Comparisons behave differently (`NULL = NULL` is not true; use `IS NULL`).

### 5. What are primary key, unique key, and foreign key?
**Answer:**
- **Primary key (PK):** unique + not null; identifies a row.
- **Unique key:** enforces uniqueness; can be nullable.
- **Foreign key (FK):** enforces referential integrity between tables (InnoDB).

### 6. What is a candidate key vs surrogate key?
**Answer:** Candidate key is a natural unique identifier (e.g., email). Surrogate key is an artificial ID (e.g., auto-increment). Many systems use surrogate PK + unique constraint on candidate key.

### 7. What is collation and why it matters?
**Answer:** Collation defines how strings compare/sort (case sensitivity, accents). For example `utf8mb4_0900_ai_ci` is accent-insensitive, case-insensitive.

### 8. What is `utf8mb4` and why should you use it?
**Answer:** `utf8mb4` supports full Unicode (including emoji). `utf8` in MySQL historically didn’t support 4-byte characters.

### 9. What’s the difference between `WHERE` and `HAVING`?
**Answer:** `WHERE` filters rows **before** aggregation. `HAVING` filters groups **after** aggregation.

### 10. What is a view?
**Answer:** A stored query (virtual table). Helpful for abstraction/security, but can hide performance issues if used blindly.

### 11. What is a stored procedure vs function?
**Answer:** Procedures can return multiple values/result sets and do writes; functions return a single scalar and are usable inside expressions.

### 12. What is a transaction?
**Answer:** A sequence of SQL statements treated as a single unit with **ACID** properties.

---

## 2) CRUD and query basics (13–28)

### 13. Basic `INSERT` with explicit columns
**Answer:**
```sql
INSERT INTO users (name, email) VALUES ('Suraj', 's@x.com');
```

### 14. Insert multiple rows in one statement
**Answer:**
```sql
INSERT INTO users (name, email)
VALUES ('A', 'a@x.com'), ('B', 'b@x.com');
```

### 15. Upsert (`INSERT ... ON DUPLICATE KEY UPDATE`)
**Answer:**
```sql
INSERT INTO users (email, name) VALUES ('a@x.com', 'A')
ON DUPLICATE KEY UPDATE name = VALUES(name);
```

### 16. Update with a condition
**Answer:**
```sql
UPDATE users SET status = 'active' WHERE id = 10;
```

### 17. Delete with a condition
**Answer:**
```sql
DELETE FROM users WHERE created_at < '2024-01-01';
```

### 18. `DELETE` vs `TRUNCATE` vs `DROP`
**Answer:**
- `DELETE`: row-by-row; filterable
- `TRUNCATE`: removes all rows fast; resets auto-increment
- `DROP`: deletes table structure + data

### 19. Pagination with `LIMIT/OFFSET`
**Answer:**
```sql
SELECT * FROM posts ORDER BY id DESC LIMIT 20 OFFSET 40;
```

### 20. Keyset pagination (better)
**Answer:**
```sql
SELECT * FROM posts
WHERE id < 9000
ORDER BY id DESC
LIMIT 20;
```

### 21. `DISTINCT`
**Answer:**
```sql
SELECT DISTINCT city FROM users;
```

### 22. Why `ORDER BY` can be expensive?
**Answer:** Sorting costs CPU/memory; indexes can avoid sorting if query order matches index.

### 23. `IN` vs `EXISTS`
**Answer:** `EXISTS` often better for correlated subqueries; `IN` fine for small lists. Validate with `EXPLAIN`.

### 24. `EXISTS` example
**Answer:**
```sql
SELECT u.*
FROM users u
WHERE EXISTS (
  SELECT 1 FROM orders o
  WHERE o.user_id = u.id AND o.status = 'paid'
);
```

### 25. Substring search
**Answer:**
```sql
SELECT * FROM users WHERE name LIKE '%raj%';
```

### 26. FULLTEXT index use case
**Answer:** Natural-language search on large text; better than `%LIKE%` for scale.

### 27. Case-insensitive match
**Answer:**
```sql
SELECT * FROM users WHERE name COLLATE utf8mb4_0900_ai_ci = 'suraj';
```

### 28. `SQL_SAFE_UPDATES`
**Answer:** Blocks dangerous UPDATE/DELETE without key-based WHERE or LIMIT (useful in dev).

---

## 3) Joins and set logic (29–44)

### 29. `INNER JOIN`
**Answer:** Only matching rows in both tables.

### 30. `LEFT JOIN`
**Answer:** All left rows + matches from right (NULL if missing).

### 31. Users with no orders
**Answer:**
```sql
SELECT u.*
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE o.id IS NULL;
```

### 32. `RIGHT JOIN`
**Answer:** Reverse of LEFT JOIN; usually avoid by swapping table order.

### 33. `CROSS JOIN`
**Answer:** Cartesian product; careful with large tables.

### 34. Join vs subquery
**Answer:** Often joins are clearer; `EXISTS` subqueries are great for “presence” checks. Use `EXPLAIN`.

### 35. Join cardinality
**Answer:** Estimated matching rows; wrong estimates → bad plans.

### 36. Why duplicates after join?
**Answer:** 1-to-many multiplies rows. Fix with aggregation/DISTINCT/limit select list.

### 37. `UNION` vs `UNION ALL`
**Answer:** `UNION` removes duplicates (costly). `UNION ALL` keeps duplicates (faster).

### 38. `UNION ALL` example
**Answer:**
```sql
SELECT email FROM users
UNION ALL
SELECT email FROM subscribers;
```

### 39. Self join example
**Answer:**
```sql
SELECT e.name, m.name AS manager
FROM employees e
LEFT JOIN employees m ON m.id = e.manager_id;
```

### 40. Join order
**Answer:** Optimizer chooses join order to reduce intermediate rows; indexes/stats matter.

### 41. Covering index
**Answer:** Index contains all needed columns → can avoid reading table rows.

### 42. `Using temporary; Using filesort`
**Answer:** Extra work; not always bad but often worth optimizing.

### 43. Join on multiple columns
**Answer:** Use composite predicate + composite index.

### 44. Full outer join in MySQL
**Answer:** Use `LEFT JOIN ... UNION ... RIGHT JOIN`.

---

## 4) Aggregation + window functions (45–56)

### 45. Orders per user
**Answer:**
```sql
SELECT user_id, COUNT(*) AS total
FROM orders
GROUP BY user_id;
```

### 46. `COUNT(*)` vs `COUNT(col)` vs `COUNT(DISTINCT col)`
**Answer:** rows vs non-null values vs unique non-null values.

### 47. Top users by revenue
**Answer:**
```sql
SELECT user_id, SUM(amount) AS revenue
FROM orders
WHERE status = 'paid'
GROUP BY user_id
ORDER BY revenue DESC
LIMIT 3;
```

### 48. Users with >= 5 orders
**Answer:**
```sql
SELECT user_id
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 5;
```

### 49. What is a window function?
**Answer:** Computes values over a window without collapsing rows.

### 50. Running total
**Answer:**
```sql
SELECT id, amount,
       SUM(amount) OVER (ORDER BY id) AS running_total
FROM orders;
```

### 51. `ROW_NUMBER` vs `RANK` vs `DENSE_RANK`
**Answer:** ROW_NUMBER unique; RANK gaps; DENSE_RANK no gaps.

### 52. Latest order per user
**Answer:**
```sql
SELECT *
FROM (
  SELECT o.*,
         ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) AS rn
  FROM orders o
) x
WHERE x.rn = 1;
```

### 53. `WITH ROLLUP`
**Answer:** Adds subtotal/grand total rows.

### 54. Non-aggregated columns in GROUP BY
**Answer:** Can be indeterminate unless ONLY_FULL_GROUP_BY is enabled (recommended).

### 55. Median/percentile
**Answer:** Use window functions + ordered row logic (no built-in MEDIAN()).

### 56. Find duplicates (example)
**Answer:**
```sql
SELECT email, COUNT(*) c
FROM users
GROUP BY email
HAVING c > 1;
```

---

## 5) Indexing, query plans, performance (57–74)

### 57. What is an index?
**Answer:** Typically B-Tree structure to speed lookups/sorts at cost of extra space and slower writes.

### 58. When to add an index?
**Answer:** Frequent filters, joins, ORDER BY/GROUP BY. Avoid indexing low-cardinality alone.

### 59. Composite index + leftmost prefix
**Answer:** `(a,b,c)` helps queries on `a`, `a+b`, `a+b+c` (not only `b`).

### 60. Composite index example `(user_id, created_at)`
**Answer:** Great for `WHERE user_id=? ORDER BY created_at` queries.

### 61. Index selectivity
**Answer:** Higher uniqueness usually better.

### 62. What to look for in `EXPLAIN`?
**Answer:** `type`, `key`, `rows`, `filtered`, `Extra`.

### 63. `type=ALL`
**Answer:** Full table scan (often missing index / non-sargable filter).

### 64. Sargable predicate
**Answer:** One that can use index. Avoid functions on indexed columns in WHERE.

### 65. Why `LIKE '%term%'` is slow?
**Answer:** Leading wildcard disables B-Tree index usage.

### 66. What is filesort?
**Answer:** MySQL sort step when index cannot satisfy ORDER BY.

### 67. How to optimize a slow query?
**Answer:** slow-log/metrics → EXPLAIN → index/rewrite → validate with real data.

### 68. N+1 problem
**Answer:** Many queries from app; fix with join/batching/eager loading.

### 69. Index cost on writes
**Answer:** More indexes → slower inserts/updates and more contention.

### 70. `ANALYZE TABLE`
**Answer:** Updates stats used by optimizer.

### 71. Query cache
**Answer:** Removed in modern MySQL; use app/Redis caching.

### 72. Hot index contention
**Answer:** Heavy writes to same index pages; mitigations include batching/partitioning/ID strategies.

### 73. Partitioning
**Answer:** Helps pruning & maintenance for huge tables with partition key (often date).

### 74. Common slow-query anti-patterns
**Answer:** Missing indexes, non-sargable filters, deep OFFSET, selecting `*`, unbounded queries.

---

## 6) Transactions, locking, concurrency (75–86)

### 75. ACID?
**Answer:** Atomicity, Consistency, Isolation, Durability.

### 76. Isolation levels in MySQL
**Answer:** READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ (default), SERIALIZABLE.

### 77. MVCC?
**Answer:** Snapshot reads with multiple row versions to reduce read locks.

### 78. Dirty/non-repeatable/phantom reads
**Answer:** Read uncommitted / row changes between reads / new rows appear in a range.

### 79. Phantom prevention in InnoDB
**Answer:** Next-key locks (record + gap) under RR for range/index scans.

### 80. Row lock vs table lock
**Answer:** InnoDB row locks for DML; some DDL patterns lock tables.

### 81. Deadlock handling
**Answer:** InnoDB rolls back one tx; app should retry with backoff.

### 82. `SELECT ... FOR UPDATE`
**Answer:**
```sql
START TRANSACTION;
SELECT * FROM wallets WHERE user_id = 10 FOR UPDATE;
UPDATE wallets SET balance = balance - 100 WHERE user_id = 10;
COMMIT;
```

### 83. `FOR UPDATE` vs `LOCK IN SHARE MODE`
**Answer:** Exclusive vs shared locks.

### 84. Autocommit
**Answer:** Each statement commits automatically unless inside explicit transaction.

### 85. Safe money transfer design
**Answer:** Transaction + row locks + consistent lock order + constraints.

### 86. Write skew
**Answer:** Two tx read same invariant then both write; avoid via stronger isolation/locking.

---

## 7) Bulk updates (87–92)

### 87. Bulk update set-based
**Answer:**
```sql
UPDATE users
SET status = 'inactive'
WHERE last_login < NOW() - INTERVAL 180 DAY;
```

### 88. Bulk update with CASE
**Answer:**
```sql
UPDATE products
SET price = CASE id
  WHEN 1 THEN 100
  WHEN 2 THEN 150
  ELSE price
END
WHERE id IN (1,2);
```

### 89. Update using JOIN
**Answer:**
```sql
UPDATE users u
JOIN user_flags f ON f.user_id = u.id
SET u.is_vip = f.is_vip
WHERE f.updated_at >= NOW() - INTERVAL 1 DAY;
```

### 90. Avoid locking huge ranges
**Answer:** Batch by PK ranges, keep tx small, ensure indexes, off-peak.

### 91. Online schema change
**Answer:** Prefer online DDL; for huge tables use pt-online-schema-change/gh-ost.

### 92. Safe backfill workflow
**Answer:** Add nullable column → write both → backfill in batches → add constraints.

---

## 8) Security & SQL injection (93–98)

### 93. SQL injection?
**Answer:** Untrusted input changes query logic (e.g., `OR 1=1`).

### 94. Prevent SQL injection?
**Answer:** Parameterized queries (prepared statements), input validation, least privilege.

### 95. Vulnerable vs safe
**Answer:** Vulnerable string concatenation vs driver parameter binding (query + params separate).

### 96. Prepared statement?
**Answer:** Precompiled statement with bind parameters; improves safety and can improve repeated performance.

### 97. Least privilege?
**Answer:** App DB user only needs minimal permissions; avoid admin privileges in app.

### 98. Extra controls
**Answer:** TLS, restricted network, secret rotation, audit logs, encrypted backups.

---

## 9) Schema design & constraints (99–100)

### 99. Normalization basics
**Answer:** Reduce redundancy and update anomalies (1NF/2NF/3NF concepts).

### 100. Denormalization
**Answer:** Acceptable for read-heavy performance with clear consistency strategy.

---

# Query Practice (mini schema)

```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE,
  city VARCHAR(100),
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  status ENUM('created','paid','cancelled') NOT NULL,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_orders_user_created (user_id, created_at),
  CONSTRAINT fk_orders_user FOREIGN KEY (user_id) REFERENCES users(id)
);
```

## Practice tasks
A) CRUD for `users` (insert, update, delete, read by email)  
B) Total paid amount per user (sorted desc)  
C) Users with no orders  
D) Latest order per user  
E) Keyset pagination  
F) Explain query plan for “paid orders for user last 30 days”  
G) Bulk update in batches  
H) Fix non-sargable `DATE(created_at)` filter  
I) Handle deadlocks in Node.js (retry + backoff)

---

## Quick cheat sheet

### Show indexes
```sql
SHOW INDEX FROM orders;
```

### EXPLAIN
```sql
EXPLAIN SELECT * FROM orders
WHERE user_id = 10 AND created_at >= NOW() - INTERVAL 30 DAY
ORDER BY created_at DESC;
```

### Transaction
```sql
START TRANSACTION;
-- work
COMMIT;
-- or ROLLBACK;
```

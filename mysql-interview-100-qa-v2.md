# MySQL Interview Questions (Fundamentals → Deep Dive) — 100 Q&A + Query Practice

> Focus areas: CRUD, joins, transactions, bulk updates, performance, indexing, security (SQL injection), and real-world ops.  
> Assumes MySQL **8.x** (InnoDB as default storage engine).

---

## 1) Core fundamentals (1–12)

### 1. What is MySQL, and what is InnoDB?
**Answer:** MySQL is an RDBMS; **InnoDB** is the default storage engine in MySQL 8.x. InnoDB supports **ACID transactions**, **row-level locking**, **MVCC**, and **foreign keys**.

**InnoDB** is a general-purpose, transactional storage engine for MySQL and MariaDB. 
**MVCC** (Multi-Version Concurrency Control) in MySQL (InnoDB) is the mechanism that lets reads and writes happen at the same time without blocking each other much.

What you get from **MVCC**

- Non-blocking reads: SELECT usually doesn’t block UPDATE/INSERT/DELETE.
- Consistent reads: you can repeatedly read and see a stable view (depending on isolation level).

`ACID` in databases is a set of guarantees for transactions (a group of queries that should behave like one unit).
**A — Atomicity**
`All or nothing.`
If a transaction has 5 steps and step 3 fails, everything rolls back.
Example: money transfer → debit succeeds but credit fails → debit must be undone.
**C — Consistency**
`Rules are never broken.`
After a transaction, the database must still follow constraints (PK, FK, NOT NULL, CHECK), triggers, etc.

Example: account balance can’t become negative if that rule exists.
**I — Isolation**
`Transactions don’t mess with each other.`
If two transactions run at the same time, the result should be like they ran one after another (depending on isolation level).
Prevents issues like dirty reads / lost updates (stronger isolation prevents more issues).
**D — Durability**
`Once committed, it stays.`
After COMMIT, data must survive crashes/power loss.
Done using things like redo logs / WAL, fsync, replication, etc.

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
**Here’s an interview-ready answer you can say (clean + structured):**

We usually implement pagination in two ways: `OFFSET pagination` and `cursor/keyset pagination`.

- OFFSET pagination is page-number based. Example: LIMIT 20 OFFSET 200. It’s simple and it supports random access, like jumping directly to page 100, because offset is just (page-1) * limit. It’s fine for small datasets or admin/report screens. The downside is performance: as offset grows, the database has to scan/skip many rows, so deep pages get slower, and results can shift if new rows are inserted while paging.

- Cursor/Keyset pagination is sequential. Instead of page numbers, we pass a cursor (usually the last row’s key like lastId). Example: WHERE id < lastId ORDER BY id DESC LIMIT 20. This is more scalable because it uses an index range scan and stays fast even for deep pagination, so it’s great for infinite scroll / feeds. The trade-off is you can’t easily jump to an arbitrary page number (like page 100) unless you maintain extra “anchors” or use filters (date/id range). Also, if ordering isn’t unique like created_at, we use a composite cursor like (created_at, id).
**Which one to use?**

- Small pages + simple UI: OFFSET ok
- Infinite scroll / large data / fast performance: Keyset is best
 
>[!warning] How companies handle it

- For feeds: they don’t show page numbers, they use infinite scroll + filters/search.
- If page numbers are required: they either use OFFSET (if acceptable) or they maintain anchors:
- - store cursor for every 100 pages, etc.

### 21. `DISTINCT`
**Answer:**
```sql
SELECT DISTINCT city FROM users;
```
>[!note]DISTINCT means Unique

### 22. Why `ORDER BY` can be expensive?
**Answer:**
Sorting costs CPU/memory;
indexes can avoid sorting if query order matches index.

### 23. `IN` vs `EXISTS`
**what is `IN`?**
IN means: “match if the value is in this set of values.”
That set can come from:

- a small list you already have (like an array of ids)
- a subquery that produces a list of values

1) IN with an “array of ids” (common)

Example: you already know some ids (from your app, cache, previous logic):

```sql

SELECT *
FROM posts
WHERE id IN (10, 25, 90, 120);
```

When it’s good: the list is small (say tens/hundreds).
(MySQL can use the index on id and quickly fetch them.)

2) IN with a subquery (two-step idea)

You said: “subquery will find first data then on this query we will run another query” — yes, conceptually.

Example: “get posts written by users who are from Jaipur”

```js
SELECT *
FROM posts
WHERE user_id IN (
  SELECT id
  FROM users
  WHERE city = 'Jaipur'
);
```

**Conceptually:**

- inner query finds user ids from Jaipur
- outer query fetches posts for those ids

>[!note]In reality: MySQL may optimize it into a semi-join and not literally run it like two separate steps, but thinking “ids set → fetch rows” is fine.

****Important difference: NULL behavior (interview trap)**
`IN` can behave weird if the subquery returns NULL
If the right side contains NULL, comparisons can become UNKNOWN, and you may get unexpected results.

Example (conceptually):
`WHERE x IN (1, NULL)`

If x is not 1, result becomes UNKNOWN (not true), which can surprise people.

**Why `EXISTS` is often better for correlated subqueries**
Correlated means the inner query depends on the outer row.
Example: “users who have at least 1 post”

✅ EXISTS (correlated)
```sql
SELECT u.*
FROM users u
WHERE EXISTS (
  SELECT 1
  FROM posts p
  WHERE p.user_id = u.id);
```
Why it can be faster: MySQL can stop as soon as it finds the first matching post for that user (short-circuit). With a good index (posts(user_id)), it becomes very efficient.

✅ EXISTS doesn’t have this problem because it checks row existence, not value membership.

**Answer:** `EXISTS` often better for correlated subqueries; `IN` fine for small lists. Validate with `EXPLAIN`.
IN

Best for:

small, fixed lists (enums/statuses/handful of ids)

WHERE status IN ('draft','published')
WHERE id IN (10, 12, 99)


Trade-offs:

If the list is huge (thousands+), it can become slower and harder to optimize (parsing/plan/cache size).

IN (subquery) can be fine, but if the subquery returns a lot of values, performance depends heavily on indexes and MySQL’s semi-join optimization.

IN with subquery is usually okay with NULLs, but see NOT IN.

NOT IN

Best for:

almost never with subqueries unless you’re 100% sure there are no NULLs.

Big trade-off / risk: NULL trap

If the subquery returns even one NULL, NOT IN can return 0 rows unexpectedly.

When it’s safe:

the column is NOT NULL or you filter nulls:

WHERE x NOT IN (SELECT y FROM t WHERE y IS NOT NULL)

EXISTS

Best for:

“does a related row exist?” checks (correlated subquery)

WHERE EXISTS (SELECT 1 FROM posts p WHERE p.user_id = u.id)


Why it’s often faster:

can stop at first match (short-circuit)

usually becomes a semi-join and uses indexes well

Trade-offs:

Slightly more complex to read for beginners

If indexes are missing, it can still be slow (but that’s true for all)

NOT EXISTS

Best for:

“find rows that have no match” (anti-join)

WHERE NOT EXISTS (SELECT 1 FROM posts p WHERE p.user_id = u.id)


Why preferred over NOT IN:

No NULL trap

Usually optimized well (anti-join)

Trade-offs:

Needs good indexes on the joined condition (posts.user_id) for speed.
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
What it means
utf8mb4_0900_ai_ci is a MySQL 8 collation:

utf8mb4 → Unicode charset

0900 → Unicode 9.0 rules

ai → accent-insensitive (treats é like e, etc.)

ci → case-insensitive (treats Suraj, SURAJ, suraj as equal)

So it will match values like:

suraj

Suraj

SURAJ

and (because of ai) potentially accented variants like súraj depending on the exact character.

Why people use COLLATE in a WHERE clause
Usually because:

the column’s collation is different (e.g., case-sensitive), and they want a case-insensitive match just for this query, without changing the schema.

Important tradeoff (performance)
If name has an index, doing name COLLATE ... can prevent MySQL from using the index efficiently (often leads to a scan), because you’re effectively changing how the column is compared at runtime.

Better options (depending on your goal)
If you always want case-insensitive search on name: set the column collation to a *_ci collation.

If you want case-insensitive sometimes and still want index usage: create a normalized/generated column + index it (or functional index in newer MySQL), e.g. LOWER(name).
### 28. `SQL_SAFE_UPDATES`
**Answer:** Blocks dangerous UPDATE/DELETE without key-based WHERE or LIMIT (useful in dev).
SQL_SAFE_UPDATES (MySQL)

When SQL_SAFE_UPDATES = 1, MySQL prevents “risky” UPDATE/DELETE queries that could touch too many rows by mistake.

It blocks:

UPDATE table SET ...; (no WHERE)

DELETE FROM table; (no WHERE)

UPDATE/DELETE ... WHERE <non-indexed condition> (often blocked unless it can use a key)

Sometimes blocks even with WHERE if it doesn’t use a key column (PRIMARY/UNIQUE/INDEX), depending on the plan.

Allowed if you add:

a key-based WHERE (uses an indexed column), or

a LIMIT, or

you disable safe updates.

Check / enable / disable
SELECT @@sql_safe_updates;
SET SQL_SAFE_UPDATES = 1;  -- enable
SET SQL_SAFE_UPDATES = 0;  -- disable

Example

❌ blocked:

DELETE FROM users WHERE name = 'suraj';


✅ allowed (key-based):

DELETE FROM users WHERE id = 123;


✅ allowed (LIMIT):

DELETE FROM users WHERE name = 'suraj' LIMIT 1;


Why it’s useful: mainly a dev/prod safety guard to avoid accidental full-table modifications.
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
What UNION does

UNION combines rows from 2 (or more) SELECTs and then removes duplicates from the final result.

Because it must remove duplicates, MySQL usually needs to do something like:

sort the result set, or

build a temporary table + deduplicate (hash/sort)

That extra work makes it slower and more memory/temp-disk heavy, especially with large results.

SELECT id FROM a
UNION
SELECT id FROM b;
-- returns unique ids only

What UNION ALL does

UNION ALL just appends results from the second query to the first query.

No deduplication

Usually no sort/temp table needed
So it’s typically faster.

SELECT id FROM a
UNION ALL
SELECT id FROM b;
-- returns ids including duplicates

Small example

Table A: 1, 2, 3
Table B: 3, 4

UNION result: 1, 2, 3, 4 (duplicate 3 removed)
UNION ALL result: 1, 2, 3, 3, 4 (duplicate kept)

Interview tradeoffs / when to use which
Use UNION ALL when:

You know the sets don’t overlap, or

You don’t care about duplicates, or

You want performance and will handle duplicates later if needed.

Use UNION when:

You need unique rows in the final output.

Important details interviewers like

Column count + types must match
Both SELECTs must return same number of columns and compatible types.

SELECT id, name FROM a
UNION ALL
SELECT id, name FROM b;


ORDER BY goes at the end

(SELECT id FROM a)
UNION ALL
(SELECT id FROM b)
ORDER BY id;


If you used UNION ALL but need unique, you can do:

SELECT DISTINCT id
FROM (
  SELECT id FROM a
  UNION ALL
  SELECT id FROM b
) x;


Sometimes this is easier to control (and can be optimized differently).
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

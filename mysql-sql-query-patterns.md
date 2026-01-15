# MySQL SQL Query Patterns (CRUD + clause order)

> Quick cheat-sheet for interview + day-to-day MySQL (8.x).  
> **Tip:** Use parameterized queries (placeholders like `?`) in app code to avoid SQL injection.

---

## 1) SELECT clause order (the “SQL order”)

### Syntax (common order)
```sql
SELECT
  [DISTINCT] select_list
FROM table_or_subquery
  [JOIN ... ON ...]         -- INNER / LEFT / RIGHT / CROSS
WHERE row_filter            -- filters rows BEFORE grouping
GROUP BY group_list
HAVING group_filter         -- filters groups AFTER grouping
ORDER BY sort_list
LIMIT row_count OFFSET n;   -- or LIMIT n, row_count (MySQL)
```

### Example
```sql
SELECT u.id, u.name, COUNT(o.id) AS orders
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.status = 'active'
GROUP BY u.id, u.name
HAVING COUNT(o.id) >= 2
ORDER BY orders DESC, u.id ASC
LIMIT 10 OFFSET 0;
```

### Notes

- `WHERE` filters **rows** before `GROUP BY`.
- `HAVING` filters **groups** after `GROUP BY`.
- `ORDER BY` is applied near the end.
- `LIMIT/OFFSET` is applied last.

---

## 2) INSERT patterns

### A) Insert 1 row
```sql
INSERT INTO users (name, email, status)
VALUES ('Suraj', 'suraj@example.com', 'active');
```

### B) Insert multiple rows (bulk insert)
✅ This is the correct SQL way:
```sql
INSERT INTO users (name, email, status)
VALUES
  ('A', 'a@example.com', 'active'),
  ('B', 'b@example.com', 'inactive'),
  ('C', 'c@example.com', 'active');
```

❌ Not valid SQL:
```sql
-- You cannot pass [(),()] in SQL. That’s an array syntax in programming languages.
INSERT INTO users (name) [(),()];
```

> In code, you often **build** the `VALUES (...), (...), (...)` portion dynamically, but the SQL sent to MySQL is still the same format.

### C) Insert from another SELECT
```sql
INSERT INTO users_archive (id, name, email)
SELECT id, name, email
FROM users
WHERE deleted_at IS NOT NULL;
```

### D) Upsert (insert-or-update) (MySQL)
Requires a **UNIQUE** or **PRIMARY KEY** conflict.
```sql
INSERT INTO users (email, name, status)
VALUES ('suraj@example.com', 'Suraj', 'active')
ON DUPLICATE KEY UPDATE
  name = VALUES(name),
  status = VALUES(status);
```

### E) Ignore duplicates (if you don’t care)
```sql
INSERT IGNORE INTO users (email, name)
VALUES ('suraj@example.com', 'Suraj');
```

---

## 3) UPDATE patterns

### A) Update single/many rows (same values)
```sql
UPDATE users
SET status = 'inactive', updated_at = NOW()
WHERE last_login_at < '2025-01-01';
```

> **Correct syntax:** `SET col = value` (not `SET name as suraj`).

### B) Update with LIMIT (useful in dev / batches)
```sql
UPDATE users
SET status = 'inactive'
WHERE status = 'active'
ORDER BY id
LIMIT 100;
```

### C) Update multiple specific rows (same value) using IN
```sql
UPDATE users
SET status = 'active'
WHERE id IN (101, 102, 103);
```

### D) Update multiple rows with different values (CASE)
Use this when each row needs a different value.
```sql
UPDATE users
SET status = CASE id
  WHEN 101 THEN 'active'
  WHEN 102 THEN 'inactive'
  WHEN 103 THEN 'blocked'
  ELSE status
END
WHERE id IN (101, 102, 103);
```

### E) Update using JOIN (update from another table)
```sql
UPDATE users u
JOIN orders o ON o.user_id = u.id
SET u.is_customer = 1
WHERE o.status = 'paid';
```

### F) “Bulk update from data list” (derived table)
Good when you have many id/value pairs.
```sql
UPDATE users u
JOIN (
  SELECT 101 AS id, 'active'  AS status
  UNION ALL SELECT 102, 'inactive'
  UNION ALL SELECT 103, 'blocked'
) v ON v.id = u.id
SET u.status = v.status;
```

---

## 4) DELETE patterns

### A) Delete rows with WHERE
```sql
DELETE FROM users
WHERE status = 'inactive'
  AND created_at < '2024-01-01';
```

### B) Delete with LIMIT (safety / batching)
```sql
DELETE FROM users
WHERE status = 'inactive'
ORDER BY id
LIMIT 100;
```

### C) Delete using JOIN (delete child rows based on parent condition)
```sql
DELETE o
FROM orders o
JOIN users u ON u.id = o.user_id
WHERE u.status = 'blocked';
```

---

## 5) Common JOIN patterns (with examples)

### INNER JOIN (only matching)
```sql
SELECT u.id, u.name, o.id AS order_id
FROM users u
INNER JOIN orders o ON o.user_id = u.id;
```

### LEFT JOIN (keep all left rows)
```sql
SELECT u.id, u.name, o.id AS order_id
FROM users u
LEFT JOIN orders o ON o.user_id = u.id;
```

### Finding “no match” rows (anti-join)
```sql
SELECT u.id, u.name
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE o.id IS NULL;
```

---

## 6) WHERE vs HAVING quick pattern

### WHERE (row filter)
```sql
SELECT *
FROM users
WHERE status = 'active';
```

### HAVING (group filter)
```sql
SELECT user_id, COUNT(*) AS cnt
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 5;
```

---

## 7) ORDER BY + LIMIT patterns (pagination)

### Page 1 (limit/offset)
```sql
SELECT *
FROM users
ORDER BY id
LIMIT 10 OFFSET 0;
```

### Page 2
```sql
SELECT *
FROM users
ORDER BY id
LIMIT 10 OFFSET 10;
```

### Cursor (keyset) pagination (better at scale)
```sql
SELECT *
FROM users
WHERE id > 5000
ORDER BY id
LIMIT 10;
```

---

## 8) COUNT patterns you’ll use a lot

### Count rows
```sql
SELECT COUNT(*) FROM users;
```

### Count after filter
```sql
SELECT COUNT(*) FROM users WHERE status = 'active';
```

### Count per group
```sql
SELECT status, COUNT(*) AS cnt
FROM users
GROUP BY status;
```

---

## 9) Transactions (safe multi-step changes)

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;   -- or ROLLBACK;
```

---

## 10) Interview “must-say” safety tips

- Always add a **WHERE** for `UPDATE/DELETE`, ideally using an indexed column (like `id`).
- In dev, `SQL_SAFE_UPDATES=1` can protect you from accidental full-table changes.
- Prefer parameterized queries (`?`) instead of string concatenation.

---

## 11) Tiny template library (copy/paste)

### SELECT template
```sql
SELECT columns
FROM table t
JOIN other o ON ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY ...
LIMIT ... OFFSET ...;
```

### INSERT template
```sql
INSERT INTO table (c1, c2, c3)
VALUES
  (...),
  (...);
```

### UPDATE template
```sql
UPDATE table
SET c1 = ..., c2 = ...
WHERE ...
LIMIT ...;
```

### DELETE template
```sql
DELETE FROM table
WHERE ...
LIMIT ...;
```

#### keys Explation in MySql

GROUP BY makes “bundles” (groups) of rows that share the same value(s), and then you usually run aggregate functions on each bundle (COUNT, SUM, AVG, MIN, MAX).

Think like this

- Without GROUP BY → result is row-by-row.
- With GROUP BY → result becomes one row per group.
Example 1: Group users by age

users

id 	 name	 age
1	  Suraj	    25
2	  Amit	    25
3	  Neha	    26
4	  Ravi	    25
5	  Pooja	    26

```Sql
SELECT age, COUNT(*) AS total_users
FROM users
GROUP BY age;
```

**Output**

age	 total_users
25	     3
26	     2

**Important interview rule**

In MySQL, when you use GROUP BY, every selected column must be either:
included in GROUP BY, 
or an aggregate (COUNT/SUM/...).
```sql
SELECT age, COUNT(*)
FROM users
GROUP BY age;
```

**❌ Usually invalid / dangerous:**
```sql
SELECT age, name, COUNT(*)
FROM users
GROUP BY age;
```
Because inside one age group there are many names — which name should **MySQL** return?
Why “must be in GROUP BY or aggregated”?
When you do GROUP BY age, you’re saying:
“Collapse many rows into one row per age.”
So inside each age group, there can be many names. Then if you write:

```sql
SELECT age, name
FROM users
GROUP BY age;
```

**MySQL has a problem:**
For age = 25, there might be names: Suraj, Amit, Ravi
But output can show only one name per age group
Which one should it choose? Suraj? Amit? Ravi?
✅ Every column in SELECT must be either:

- Grouped (same value inside the group)
- Aggregated (you tell SQL how to reduce many values into one)

```sql
SELECT age, COUNT(*) AS total
FROM users
GROUP BY age;

// if you want names, aggregate them

SELECT age, GROUP_CONCAT(name) AS names
FROM users
GROUP BY age;

//Correct (group by both age and name)

SELECT age, name, COUNT(*) AS cnt
FROM users
GROUP BY age, name;
```
GROUP BY decides the “unique row shape”.
Anything else in SELECT must be summarized using an aggregate.

**WHERE vs HAVING with GROUP BY**

- WHERE filters rows before grouping
- HAVING filters groups after grouping

```js
SELECT age, COUNT(*) AS total
FROM users
GROUP BY age
HAVING COUNT(*) >= 2;
```

**`ORDER BY` is for sorting the result set.**

```sql
SELECT id, name, age
FROM users
ORDER BY age ASC, name ASC;
```
`ASC` = ascending (default)
`DESC` = descending
**Sorting with GROUP BY results**
If you grouped and computed counts, you can sort by the aggregate:

```sql
SELECT age, COUNT(*) AS total
FROM users
GROUP BY age
ORDER BY total DESC;
```
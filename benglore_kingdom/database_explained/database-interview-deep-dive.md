# Databases — Deep Interview Q&A (58 Questions, Senior-Level)

Explained the way an engineer with years on production databases would explain it — with the *why*, the *trade-off*, and the *real-world gotcha*, not just definitions. Read it, then say it in your own words. Examples lean on Postgres / MySQL / MongoDB / Redis since that's your stack.

---

## Table of Contents
- [Group 1 — SQL vs NoSQL (Q1–9)](#group-1--sql-vs-nosql)
- [Group 2 — Indexing (Q10–22)](#group-2--indexing)
- [Group 3 — Scaling: replication, sharding, reads vs writes (Q23–32)](#group-3--scaling)
- [Group 4 — ACID, transactions, isolation, race conditions (Q33–46)](#group-4--acid-transactions-isolation-race-conditions)
- [Group 5 — ORMs & schemas (Q47–51)](#group-5--orms--schemas)
- [Group 6 — Switching & migrating databases (Q52–54)](#group-6--switching--migrating-databases)
- [Group 7 — Real talk: bottlenecks & "millions of rows, search fast" (Q55–58)](#group-7--real-talk-bottlenecks--performance)

---

## Group 1 — SQL vs NoSQL

### Q1. What's the core difference between SQL and NoSQL?
**SQL (relational)** stores data in tables with a fixed schema, rows and columns, and relationships enforced by the database (foreign keys). It's queried with SQL and gives strong consistency and ACID guarantees. Examples: Postgres, MySQL.
**NoSQL** is an umbrella for non-relational stores — documents, key-value, wide-column, graph — that trade some of those guarantees for flexibility and horizontal scale. Examples: MongoDB (document), Redis (key-value), Cassandra (wide-column).
The real difference isn't "tables vs no tables" — it's **where the schema lives and what guarantees you get**. SQL enforces structure and consistency up front; NoSQL pushes flexibility and scale up front and asks you to handle consistency in the app.

### Q2. When do you choose SQL?
When your data is **relational and you need integrity**: clear relationships, transactions that must be all-or-nothing (payments, orders, bookings), and complex queries with joins and aggregations. If "this row must always be consistent with that row" matters — money, inventory, accounts — start with SQL. It's also the safe default: you rarely regret starting with Postgres.

### Q3. When do you choose NoSQL?
When the access pattern fits it and SQL's structure is in the way:
- **Document (Mongo):** flexible, nested, evolving shapes; read whole objects at once (a product catalog, a CMS, user profiles with varying fields).
- **Key-value (Redis):** ultra-fast lookups by key, caching, sessions, rate limits, queues.
- **Wide-column (Cassandra):** massive write throughput, time-series, write-heavy at scale where you can design around the query.
- **Graph (Neo4j):** heavily connected data where the *relationships* are the query (social graphs, recommendations).
The honest rule: pick NoSQL when a *specific* property (write scale, flexible shape, sub-ms key lookup) clearly beats what SQL gives — not just because it's trendy.

### Q4. What are the types of NoSQL databases?
Four families: **document** (MongoDB — JSON-like docs), **key-value** (Redis, DynamoDB — a dictionary at scale), **wide-column** (Cassandra, HBase — rows with flexible columns, huge write throughput), and **graph** (Neo4j — nodes and edges for connected data). They're not interchangeable; each is good at a different access pattern.

### Q5. Which is more flexible schema-wise, and why?
NoSQL document stores are more flexible because they're **schema-on-read**, not **schema-on-write**. In SQL, the schema is enforced when you write (`ALTER TABLE` to add a column, every row must conform). In Mongo, two documents in the same collection can have different fields — you can add a field to new documents without touching old ones. That's great for rapidly changing shapes, but the cost is that *the structure now lives in your application code* — the DB won't stop you from writing inconsistent data.

### Q6. Is NoSQL really "schema-less"? (common myth)
No — "schema-less" is misleading. There's always a schema; the only question is **who enforces it and when**. In SQL the database enforces it on write. In Mongo, *your application* enforces it on read (and tools like Mongoose let you define a schema in code). If you skip that discipline, you end up with five years of documents in slightly different shapes and queries full of "if this field exists" checks. So it's not "no schema," it's "you own the schema."

### Q7. Can NoSQL do joins?
Limited and differently. SQL joins are a first-class, optimized operation. Document stores prefer you to **embed** related data in one document (so you read it in one shot) or **denormalize** (duplicate data). MongoDB has `$lookup` for joins, but it's not as efficient as a relational join and is generally avoided in hot paths. The trade-off: SQL normalizes and joins at read time; NoSQL denormalizes and pays at write time (you update the duplicated data in multiple places).

### Q8. Does NoSQL support transactions / ACID?
Increasingly yes, but with caveats. Single-document writes in Mongo are atomic. MongoDB added **multi-document ACID transactions** (4.0+), but they're more expensive and less central than in a relational DB. Many NoSQL systems prefer **BASE** (Basically Available, Soft state, Eventual consistency) over ACID — they relax consistency for availability and scale. So: if you need rock-solid multi-row transactions everywhere, that's SQL's home turf; NoSQL can do it but it's not what it's optimized for.

### Q9. Give a real example where you'd mix both.
Most real systems do. In a project like mine: **Postgres** as the source of truth for orders, users, payments (needs ACID and relationships); **Redis** for caching hot reads, rate limiting, and the job queue (needs speed, not durability); optionally **Mongo** for a flexible, document-shaped part like logs or denormalized read models. "Polyglot persistence" — use the right store for each access pattern — is the senior answer, not "one database for everything."

---

## Group 2 — Indexing

### Q10. What is an index?
An index is a separate data structure the database maintains to find rows **fast** without scanning the whole table. Conceptually it's like the index at the back of a book: instead of reading every page to find "ACID," you jump to the index, find the page number, and go straight there.

### Q11. Why do we need indexing?
Without an index, finding rows means a **full table scan** — reading every row, O(n). On a million-row table that's slow and gets slower as data grows. An index turns that into roughly **O(log n)** — a handful of lookups instead of a million. The whole reason your "find the slow query, add the right index, I/O dropped 35%" story works is this: the query went from scanning everything to seeking directly.

### Q12. How does an index work internally?
Most relational indexes (and MongoDB's) are **B-trees / B+ trees** — balanced trees kept sorted, where each lookup walks from the root down to a leaf in a few steps (log n). Because the tree stays sorted and balanced, it's fast for **equality** (`= 5`), **range** (`BETWEEN`, `>`, `<`), and **ordering** (`ORDER BY`). The leaves point to the actual row location. Some indexes use **hash** structures (great for exact equality, useless for ranges) — Postgres has hash indexes, but B-tree is the default workhorse.

### Q13. What's a B-tree vs B+ tree, and why is it used?
A **B+ tree** keeps all actual data pointers in the leaf nodes and links the leaves together, so range scans are very efficient (find the start, then walk the linked leaves). It's used because it stays **balanced** (every lookup is the same shallow depth), is **disk-friendly** (each node is a page, minimizing disk reads), and supports the operations databases need most: equality, range, sorted order. That combination is why it's the default index almost everywhere.

### Q14. How do you create an index in SQL (Postgres / MySQL)?
```sql
-- Postgres / MySQL
CREATE INDEX idx_users_email ON users (email);
CREATE UNIQUE INDEX idx_users_email ON users (email);   -- also enforces uniqueness
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at);  -- composite
```
You index the columns that show up in `WHERE`, `JOIN`, and `ORDER BY` for your real queries. The primary key is automatically indexed.

### Q15. How do you create an index in MongoDB, and how does it differ?
```js
db.users.createIndex({ email: 1 });              // 1 = ascending, -1 = descending
db.users.createIndex({ email: 1 }, { unique: true });
db.orders.createIndex({ userId: 1, createdAt: -1 });   // compound index
```
Mechanically it's the same idea — Mongo also uses **B-tree indexes** under the hood, and the column-order rules for compound indexes work the same as SQL composite indexes. The differences are in *what* you can index: Mongo can index fields inside nested documents and array elements (multikey indexes), which maps to its document model. The trade-offs (write cost, storage) are identical.

### Q16. What is a composite index, and why does column order matter?
A composite index covers multiple columns, e.g. `(user_id, created_at)`. Order matters because of the **leftmost-prefix rule**: an index on `(a, b, c)` can serve queries filtering on `a`, `a+b`, or `a+b+c` — but **not** `b` alone or `c` alone. It's like a phone book sorted by (last name, first name): great for "find Khonde" or "find Khonde, Suraj," useless for "find everyone named Suraj." So you order composite index columns to match your most common query's filter order, with the equality column first and the range column last.

### Q17. What is a covering index?
An index that contains *all* the columns a query needs, so the database can answer the query **from the index alone** without touching the table ("index-only scan"). Example: a query that selects `email` filtered by `email` is fully covered by an index on `email`. It's faster because it skips the extra hop to the row. Postgres supports `INCLUDE` columns to deliberately build covering indexes.

### Q18. What are the trade-offs of indexing? (you must know this)
Indexes are not free:
- **Writes get slower.** Every `INSERT`/`UPDATE`/`DELETE` must also update every index on that table. More indexes → slower writes. This is the central trade-off: **indexes speed up reads and slow down writes.**
- **Storage cost.** Each index is extra data on disk.
- **Useless indexes hurt.** An index on a low-selectivity column (e.g. a boolean `is_active`) barely helps and still costs writes.
So you don't index everything — you index for your **read patterns**, and you periodically drop indexes nothing uses. The judgment is: "is this read frequent and selective enough to justify the write cost?"

### Q19. Clustered vs non-clustered index?
A **clustered index** determines the physical order of the rows on disk — the table *is* the index, sorted by that key. **MySQL InnoDB** stores the table clustered by the primary key, so PK lookups are very fast and secondary indexes point back to the PK. **Postgres** is different: it stores rows in an unordered **heap**, so *all* its indexes are non-clustered (secondary) — you can run `CLUSTER` once to reorder, but it's not maintained automatically. Knowing this Postgres-vs-MySQL difference is a strong senior signal.

### Q20. When does an index NOT get used (even though it exists)?
Common cases where the planner ignores your index:
- **A function/operation on the indexed column:** `WHERE LOWER(email) = ...` won't use a plain index on `email` (use a functional/expression index instead).
- **Leading wildcard LIKE:** `WHERE name LIKE '%suraj'` can't use a normal B-tree (it can use `LIKE 'suraj%'`).
- **Low selectivity:** if the column has few distinct values, a full scan may be cheaper, so the planner skips the index.
- **Implicit type mismatch:** comparing an indexed `varchar` to a number.
This is why "I added an index but it's still slow" happens — and why you always check with `EXPLAIN`.

### Q21. How do you know if an index is actually being used?
`EXPLAIN` / `EXPLAIN ANALYZE` (Postgres/MySQL) shows the query plan: look for "Index Scan"/"Index Only Scan" (good) vs "Seq Scan"/"Full Table Scan" (the index isn't helping). `EXPLAIN ANALYZE` actually runs it and shows real timings. In Mongo, `.explain("executionStats")` shows whether it used an index (`IXSCAN`) or a collection scan (`COLLSCAN`). Reading the plan before and after a change is how you *prove* an optimization worked rather than assuming.

### Q22. What is selectivity / cardinality, and why care?
**Cardinality** = number of distinct values in a column; **selectivity** = how well a value narrows the result. High selectivity (e.g. `email`, almost unique) → an index is very effective. Low selectivity (e.g. `gender`, `is_active`) → an index barely helps because any value still matches a huge fraction of rows. You index high-selectivity columns; for low-selectivity ones you either skip the index or use a composite/partial index. This is *why* indexing the right column matters, not just "add an index."

---

## Group 3 — Scaling

### Q23. What is replication?
Keeping **copies** of your database on multiple servers. Typically one **primary** (handles writes) and one or more **replicas** (copies). Replication gives you (a) **high availability** — if the primary dies, a replica takes over, and (b) **read scaling** — send read queries to replicas. The data flows primary → replicas.

### Q24. What is a read replica, and what does it scale?
A read replica is a copy that serves **read** queries, taking read load off the primary. It scales **reads**, not writes — all writes still go to the single primary. So if your system is read-heavy (most are — like a 100:1 read:write ratio), read replicas are the first lever. Route reporting/analytics queries to a replica so they don't slow the write path. (This is literally the move for the slow-report problem — replicas keep heavy reads off the primary.)

### Q25. What's replication lag, and why does it bite you?
Replicas are updated slightly *after* the primary — that delay is **replication lag** (usually milliseconds, sometimes more under load). The classic bug: a user writes data (to primary), then immediately reads it (from a replica that hasn't caught up) and sees stale/missing data — "I just saved it, where did it go?" Fixes: read-your-own-writes from the primary for that user right after a write, or accept eventual consistency where it's safe. Knowing lag exists is a senior tell.

### Q26. What is sharding / partitioning?
**Sharding** splits your data **horizontally across multiple servers** — each shard holds a subset of the rows (e.g. users A–M on shard 1, N–Z on shard 2). Unlike replicas (full copies), each shard holds *different* data. This is how you scale **beyond what one machine can hold or write**. Partitioning is the same idea, sometimes within one server (splitting a big table into chunks).

### Q27. Horizontal vs vertical partitioning?
**Horizontal** partitioning/sharding splits **rows** across servers (each shard = some of the rows). **Vertical** partitioning splits **columns** — putting rarely-used or large columns (e.g. a big `bio` text or blob) in a separate table to keep the hot table small and fast. Most "scaling" talk means horizontal sharding.

### Q28. Which handles lots of writes vs lots of reads? (you asked this directly)
- **Lots of reads → read replicas.** Add copies, spread reads across them. The primary stays the single writer.
- **Lots of writes → sharding.** A single primary is a write bottleneck (one machine, one disk). Sharding splits writes across many primaries, each owning part of the data, so write throughput scales horizontally.
One-liner for the interview: **"Replicas scale reads; sharding scales writes."** Wide-column stores like Cassandra are built around this — they shard by design and excel at massive write throughput.

### Q29. How do you choose a shard key?
The shard key decides which shard a row lives on, so a bad choice ruins everything. You want a key that:
- **Distributes evenly** (no hot shard) — avoid keys where 90% of traffic hits one value.
- **Matches your queries** — so most queries hit a single shard, not all of them.
Example pitfall: sharding by `created_at` means all *new* writes hammer the latest shard (a hotspot). Sharding by a high-cardinality `user_id` (often hashed) spreads load evenly. Choosing the shard key is the hardest part of sharding.

### Q30. What problems does sharding introduce?
It's powerful but costly:
- **Cross-shard queries/joins** are slow or impossible — data you want to join may live on different machines.
- **Rebalancing** — adding a shard means moving data, which is operationally painful.
- **Hotspots** — a bad shard key concentrates load on one shard.
- **Transactions across shards** are very hard (distributed transactions).
So you shard only when you must — when a single beefy primary plus replicas genuinely can't keep up. Premature sharding is a classic over-engineering mistake.

### Q31. Vertical vs horizontal scaling for a database?
**Vertical** = bigger machine (more CPU/RAM/faster disk). Simple, no app changes, but has a hard ceiling and gets expensive fast. **Horizontal** = more machines (replicas for reads, shards for writes). Harder (distributed-systems problems) but scales far. The usual progression: optimize queries and indexes → vertical scale → read replicas → sharding. You climb that ladder only as far as the load forces you.

### Q32. What is connection pooling and why does it matter?
Every DB connection is expensive (memory on the server, setup cost). Opening a new connection per request crushes the database. A **connection pool** keeps a fixed set of reusable connections that requests borrow and return. Postgres especially is sensitive — too many connections degrade it, so a pooler (PgBouncer, or your ORM/driver pool) is essential at scale. A surprising number of "the database is slow" incidents are really "we exhausted the connection pool."

---

## Group 4 — ACID, transactions, isolation, race conditions

### Q33. What is ACID? (explain each letter)
The four guarantees of a reliable transaction:
- **Atomicity** — all-or-nothing. A transfer that debits A and credits B either does both or neither; no half-states.
- **Consistency** — the DB moves from one valid state to another; constraints (foreign keys, checks) are never violated.
- **Isolation** — concurrent transactions don't step on each other; each behaves as if it ran alone (to a degree set by the isolation level).
- **Durability** — once committed, it survives crashes/power loss (written to durable storage).
This is *why* you put money operations in a relational DB.

### Q34. What is a transaction?
A group of operations treated as a **single unit** — they all succeed and commit, or they all fail and roll back. Example: `BEGIN; UPDATE accounts SET balance = balance - 100 WHERE id = 1; UPDATE accounts SET balance = balance + 100 WHERE id = 2; COMMIT;`. If the second update fails, the first is rolled back — you never lose the 100. (This is exactly the kind of guarantee Razorpay payment verification needs.)

### Q35. What are isolation levels?
They control how much concurrent transactions can see each other's in-progress work — trading safety for performance. From weakest to strongest:
1. **Read Uncommitted** — can see others' uncommitted changes (dirty reads). Rarely used.
2. **Read Committed** — only sees committed data; no dirty reads. **(Postgres default.)**
3. **Repeatable Read** — the same query returns the same rows within the transaction. **(MySQL InnoDB default.)**
4. **Serializable** — strongest; transactions behave as if run one at a time. Safest, slowest.
Knowing the *defaults differ* (Postgres = Read Committed, MySQL = Repeatable Read) is a strong senior detail.

### Q36. What anomalies do isolation levels prevent?
Three classic read anomalies, each prevented at a higher level:
- **Dirty read** — reading another transaction's *uncommitted* change (which may roll back). Prevented at Read Committed+.
- **Non-repeatable read** — reading the same row twice in a transaction and getting different values because someone committed an update in between. Prevented at Repeatable Read+.
- **Phantom read** — re-running a range query and getting new *rows* that weren't there before. Prevented at Serializable (Postgres's snapshot-based RR also avoids it; MySQL uses gap locks).
You pick the lowest level that prevents the anomalies your use case can't tolerate.

### Q37. How is isolation implemented — locking or MVCC?
Two strategies. **Locking** — a transaction locks rows so others wait (simple, but readers can block writers). **MVCC (Multi-Version Concurrency Control)** — the DB keeps multiple *versions* of a row, so readers see a consistent snapshot without blocking writers, and writers don't block readers. **Postgres and MySQL InnoDB both use MVCC**, which is why reads generally don't block writes — a huge concurrency win. The cost is that old row versions accumulate and must be cleaned up (Postgres's `VACUUM`).

### Q38. What is MVCC, concretely?
Each transaction sees a **snapshot** of the database as of when it started (depending on isolation level). When a row is updated, the DB doesn't overwrite it — it writes a *new version* and keeps the old one for transactions that still need it. So a long-running read sees a stable snapshot while writes proceed. This is why Postgres can run a heavy report on a replica/snapshot without blocking live writes. Trade-off: dead versions ("bloat") need periodic cleanup.

### Q39. What is a race condition in a database? Give an example.
When two transactions touch the same data concurrently and the result depends on timing — producing a wrong outcome. Classic: **lost update**. Two requests both read `stock = 1`, both decide "okay to sell," both write `stock = 0` — you sold the last item twice. Or two users grab the same coupon. The reads interleave and one update silently overwrites the other.

### Q40. How do you control race conditions in a database?
Several tools, depending on the situation:
- **Atomic operations** — do it in one statement so there's no read-then-write gap: `UPDATE products SET stock = stock - 1 WHERE id = 1 AND stock > 0;`. The DB serializes this; if it affects 0 rows, you're out of stock. (Redis `INCR` is the same idea for counters — which you used for rate limiting.)
- **Pessimistic locking** — `SELECT ... FOR UPDATE` locks the row so others wait until you commit.
- **Optimistic locking** — a `version` column; you read it, and on update do `WHERE version = <read value>` — if someone changed it, your update affects 0 rows and you retry.
- **Higher isolation level** — Serializable makes the DB detect and abort conflicting transactions.
- **Unique constraints** — let the DB reject duplicates (e.g. one coupon per user) instead of checking in app code.

### Q41. Optimistic vs pessimistic locking — when each?
**Pessimistic** ("assume conflict"): lock the row up front (`FOR UPDATE`), others wait. Good when **contention is high** (many writers hitting the same rows) — you avoid wasted retries. Cost: locks reduce concurrency and risk deadlocks.
**Optimistic** ("assume no conflict"): don't lock; check a version on write and retry if it changed. Good when **contention is low** — most of the time there's no conflict, so you skip locking overhead. Cost: under high contention you get lots of retries. Pick based on how often two writers actually collide.

### Q42. What is a deadlock, and how do you handle it?
Two transactions each hold a lock the other needs and both wait forever — A locks row 1 and wants row 2, B locks row 2 and wants row 1. Databases **detect** deadlocks and kill one transaction (it gets a deadlock error to retry). To **prevent** them: always acquire locks in a consistent order, keep transactions short, and don't hold locks across slow external calls. In app code, wrap the transaction in a retry on deadlock errors.

### Q43. What does `SELECT ... FOR UPDATE` do?
It reads rows **and locks them** for the duration of the transaction, so no other transaction can modify them until you commit. It's how you implement pessimistic locking — e.g. "read the stock, lock it, decide, decrement, commit" with nobody else able to touch that row in between. Use it sparingly and keep the transaction short, or it becomes a bottleneck.

### Q44. How do you safely do an atomic increment / counter?
**Don't** do `read → add 1 → write` in app code — that's the lost-update race. Do it in **one atomic DB statement**: `UPDATE counters SET value = value + 1 WHERE id = ?` — the database guarantees the increment is atomic. For high-frequency counters (likes, views, rate limits), **Redis `INCR`** is even better — it's atomic and in-memory fast, then you flush to the DB periodically. (Exactly the rate-limiter pattern.)

### Q45. What is the CAP theorem?
In a distributed database, during a **network partition** (servers can't talk to each other), you can have at most **two** of: **Consistency** (every read sees the latest write), **Availability** (every request gets a response), **Partition tolerance** (the system keeps working despite dropped messages). Since partitions *will* happen in a distributed system, P is mandatory — so the real choice is **C or A** during a partition. SQL systems lean **CP** (refuse/stale-block to stay consistent); many NoSQL systems lean **AP** (stay available, accept eventual consistency). It's not a permanent label — it's about behavior *during a partition*.

### Q46. What is eventual consistency / BASE?
Many NoSQL systems prefer **BASE** (Basically Available, Soft state, Eventual consistency) over ACID: instead of guaranteeing every read is immediately consistent, they guarantee that **if writes stop, all replicas eventually converge** to the same value. You trade "always exactly right now" for "always available and eventually right." That's fine for a like count or a feed; it's *not* fine for a bank balance. Choosing where eventual consistency is acceptable is a key design judgment.

---

## Group 5 — ORMs & schemas

### Q47. What is an ORM and what problem does it solve?
An **Object-Relational Mapper** lets you work with database rows as objects/types in your language instead of writing raw SQL strings — it maps tables to types and gives you type safety, query building, and migrations. It solves: repetitive boilerplate, SQL-injection risk (parameterized by default), and keeping your schema in sync with code. The trade-off is a layer of abstraction that can hide what SQL actually runs.

### Q48. Which ORMs are well known?
By ecosystem:
- **Node/TypeScript:** **Prisma** (schema-file driven, great DX), **Drizzle** (lightweight, typed SQL close to the metal — what you use), **TypeORM**, **Sequelize**, and **Mongoose** for MongoDB.
- **Python:** SQLAlchemy, Django ORM.
- **Java:** Hibernate.
The TS landscape mostly comes down to **Prisma vs Drizzle**: Prisma abstracts more and is very ergonomic; Drizzle stays closer to SQL with excellent type inference and less magic — which is why you'd pick it when you want control over the actual queries.

### Q49. How do you define a schema and evolve it in an ORM?
You declare the schema in code (Prisma's schema file, Drizzle's TypeScript schema), then use **migrations** to apply changes: the ORM generates a migration (e.g. "add column `expires_at`") that's a versioned, repeatable script checked into git. You run migrations as part of deploy so every environment's schema matches the code. Never hand-edit production schema ad hoc — migrations give you history and rollback.

### Q50. What is the N+1 query problem?
A classic ORM trap: you fetch a list of N items, then the ORM lazily fires **one extra query per item** to load a relation — 1 + N queries instead of 1 or 2. Example: load 100 posts, then loop and load each post's author = 101 queries. Fix with **eager loading** (a join, or `include`/`with` in the ORM) or batching the related lookups into one `WHERE id IN (...)` query. It's one of the most common real-world performance bugs, and ORMs make it easy to cause by accident — so you watch the query count.

### Q51. ORM vs raw SQL — the trade-off?
**ORM:** faster development, type safety, injection-safe, easy migrations — but it abstracts the SQL, can generate inefficient queries, and the N+1 trap. **Raw SQL:** full control, optimal queries, no surprises — but more boilerplate, manual safety, and you maintain it. Senior practice: **use the ORM for the 90% of normal CRUD, drop to raw SQL for the hot/complex queries** where you need control. That's why a tool like Drizzle is nice — it's typed but lets you write SQL-shaped queries without a heavy abstraction.

---

## Group 6 — Switching & migrating databases

### Q52. Can you switch from NoSQL to SQL? How hard is it?
Possible but **not trivial** — it's a remodeling job, not a config change. You have to: define a fixed schema for data that may have inconsistent shapes; turn embedded documents and denormalized duplicates into normalized tables and foreign keys; write an ETL to transform and load the existing data; and rewrite all queries. The hardest part is data that was *allowed* to be inconsistent in NoSQL now has to fit constraints. Doable, but plan it as a real project with a dual-write/backfill phase, not a weekend.

### Q53. How easy is it to switch Postgres to MySQL (or vice versa)?
**Harder than people expect**, even though both are SQL. The friction:
- **Dialect differences** — SQL isn't fully portable; functions, syntax, and behaviors differ.
- **Data types** — Postgres has rich types (arrays, `jsonb`, native UUID, enums, ranges) that MySQL handles differently or not at all.
- **Auto-increment** — Postgres sequences/`SERIAL` vs MySQL `AUTO_INCREMENT`.
- **Defaults differ** — isolation level (RC vs RR), case sensitivity, etc.
- **Features** — window functions, CTEs, partial indexes historically richer/different in Postgres.
An ORM abstracts **basic CRUD**, so simple apps switch more easily — but any raw SQL, DB-specific types, or features need rewriting. Realistic answer: "the more you used database-specific features, the more painful the switch; an ORM only shields the simple parts."

### Q54. What is a database migration, and how do you do it with zero downtime?
A **migration** is a versioned change to the schema (add a column, add an index, change a type). Zero-downtime is about not breaking running code mid-deploy:
- **Add, don't break:** add a nullable column first; deploy code that can handle both old and new; backfill data; only then make it non-null/required. (Never rename a column in one step while old code is running — add new, migrate, drop old.)
- **Build indexes concurrently:** in Postgres, `CREATE INDEX CONCURRENTLY` avoids locking the table.
- **Expand-then-contract pattern:** expand the schema to support old + new, migrate data, then contract (remove old) once nothing uses it.
The principle: every intermediate state must be safe for both the old and new code versions running simultaneously during a rolling deploy.

---

## Group 7 — Real talk: bottlenecks & performance

### Q55. How do you recognize that the database is the *real* bottleneck? (you asked this)
Don't guess — measure, then localize:
- **Symptom:** endpoints are slow, latency spikes under load, timeouts.
- **Localize:** is time spent in the app or waiting on the DB? Check DB metrics — slow query logs, CPU/IO on the DB host, connection pool saturation, lock waits. If app CPU is low but requests are slow and DB IO/locks are high, the DB is the bottleneck.
- **Confirm the query:** `EXPLAIN ANALYZE` the suspect queries — a `Seq Scan` on a big table, or a query reading far more rows than it returns, is your culprit.
- **Common real causes:** missing index, N+1 from the ORM, connection-pool exhaustion, a lock-contended hot row, or heavy analytics queries hitting the primary.
The discipline (same as debugging anything): **symptom → measure → isolate the query → fix root cause → verify with the same measurement.**

### Q56. We have millions of rows and need to search fast — how? (the big one)
Walk through it in order of effort:
1. **Index the search column** with the *right* index type. For equality/range → B-tree. This alone takes a full scan to a seek and is usually the answer.
2. **Match the index to the query.** Composite index ordered to your filters; covering index if you can answer from the index alone. Verify with `EXPLAIN` that it's actually used (no `Seq Scan`).
3. **Text search needs the right tool.** `LIKE '%term%'` (leading wildcard) **cannot** use a normal B-tree. For text:
   - Postgres **full-text search** (`tsvector` + **GIN index**) for word/phrase search.
   - **Trigram index** (`pg_trgm`) for fuzzy/`LIKE` matching.
   - For serious search (relevance ranking, typo tolerance, faceting) → a dedicated engine like **Elasticsearch/OpenSearch**, kept in sync with the DB.
4. **Partition** the table if it's huge (e.g. by date), so queries scan only the relevant partition.
5. **Cache hot queries** in Redis so repeated searches skip the DB entirely.
6. **Read replicas** so search load doesn't fight write load.
The headline answer interviewers want: **"the right index turns a million-row scan into a log-n seek; if it's text search, a B-tree won't help and I'd use full-text/GIN or a search engine; and I'd verify with EXPLAIN that the index is actually used."**

### Q57. What's the difference between a slow query and a slow database?
A **slow query** is one bad query (missing index, N+1, bad join) — fix the query/index. A **slow database** is systemic — CPU/IO maxed, connection pool exhausted, lock contention, replication lag, or you've simply outgrown the box. The fix differs: a slow query is surgical (index, rewrite); a slow database is architectural (scale up, add replicas, pooling, sharding). Diagnosing *which* one you have is the first move — otherwise you scale hardware to mask a missing index, or add indexes to mask a saturated server.

### Q58. Normalization vs denormalization — when do you denormalize?
**Normalization** removes duplication by splitting data into related tables — clean, consistent, no update anomalies, but reads need joins. **Denormalization** deliberately duplicates data (or precomputes) to avoid expensive joins/aggregations on hot read paths — faster reads, at the cost of having to keep the copies in sync on write. You normalize by default (correctness), then denormalize **selectively** where a specific read is hot and the joins are proven too slow — e.g. storing a `comment_count` on a post instead of counting comments every time. It's a read-speed-vs-write-complexity trade-off, made with measurements, not by default.

---

## How to use this in an interview

- **Lead with the trade-off, not the definition.** "Indexes speed reads and slow writes" beats "an index is a data structure." Every topic here has a trade-off — say it.
- **The four lines that make you sound senior:** *"Replicas scale reads, sharding scales writes."* / *"Schema-on-write vs schema-on-read."* / *"Indexes speed reads, cost writes."* / *"Measure which is slow — the query or the database — before fixing."*
- **Anchor to your real stack.** You run Postgres + Redis + Mongo + Drizzle in production — pull real examples (atomic counters for rate limiting, read-cache for hot timelines, the slow-report index fix). Real beats textbook every time.
- **For "millions of rows, search fast" (Q56):** right index → verify with EXPLAIN → full-text/GIN or a search engine for text → cache + replicas. That sequence is the whole answer.
- **Default to Postgres, justify exceptions.** "I start relational for integrity and reach for NoSQL when a specific access pattern clearly wins" is a defensible, senior stance.

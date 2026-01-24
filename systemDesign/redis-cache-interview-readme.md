# Redis Caching – Full‑Stack Interview Notes (README)

This file is written to be **speakable in interviews** and **usable in real projects**.

---

## 1) Why Redis is popular for caching vs in‑process memory

### In‑process memory (inside Node/Java/etc.)
**Pros**
- Fastest possible (no network hop)
- Simple (a JS Map, LRU cache library)

**Cons (why it breaks at scale)**
- **Not shared across servers**: each instance has a different cache → inconsistent hit rate
- **Wasted memory**: if you run 10 app servers, you store the same cached data 10 times
- **Cold start / restart wipes cache**
- Hard to enforce global TTL/eviction consistently
- No built-in distributed features (locks, atomic counters, etc.)

### Redis
**Why companies use it**
- **Shared cache for all app servers** (one place; consistent behavior)
- **Very fast** (in-memory, optimized data structures)
- Built‑in **TTL**, **eviction policies**, and **atomic operations**
- Works for more than caching: **rate limiting, sessions, locks, leaderboards, queues**
- Operational tooling: memory stats, persistence options, replication/cluster (depends on setup)

**Interview one‑liner**
> “In‑process cache is per-instance and disappears on restart; Redis is a shared, centralized in‑memory store with TTL + eviction + atomic ops, so it scales with multiple app servers.”

---

## 2) Redis data types for caching (what to use, when, why)

### Quick rule
- If you’re caching a single blob → **STRING**
- If you’re caching an object with fields you update often → **HASH**
- If you need “unique items” → **SET**
- If you need ranking/top N → **ZSET (sorted set)**

---

## 3) Redis data types table (interview-ready)

| Data type | What it stores | Best for | Why use it | Common commands |
|---|---|---|---|---|
| **STRING** | bytes / text / JSON | Simple cache entry, HTML response, API response | simplest + fastest; easy TTL | `GET`, `SET`, `SETEX`, `MGET` |
| **HASH** | field → value map | Caching objects (user profile fields) | update/read individual fields without rewriting whole JSON | `HGET`, `HSET`, `HMGET`, `HGETALL` |
| **SET** | unordered unique members | Feature flags per user, “seen ids”, tags, permissions list | uniqueness, fast membership checks | `SADD`, `SREM`, `SISMEMBER`, `SMEMBERS` |
| **ZSET** | members with score | Leaderboards, top N, ranking by score/time | sorted queries (top users, trending) | `ZINCRBY`, `ZREVRANGE`, `ZRANK` |
| **LIST** | ordered list (like queue) | simple queue / log buffer | push/pop from ends | `LPUSH`, `RPUSH`, `LPOP`, `RPOP`, `LRANGE` |
| **BITMAP** | bit array | very compact boolean flags | memory efficient for yes/no at huge scale | `SETBIT`, `GETBIT`, `BITCOUNT` |
| **HYPERLOGLOG** | approximate count | unique visitors / cardinality | tiny memory + fast; approximate | `PFADD`, `PFCOUNT` |
| **STREAM** | append-only log | event stream / lightweight messaging | consumer groups, replay | `XADD`, `XREADGROUP` |
| **GEO** | geo points | “near me” queries | built-in geo indexing | `GEOADD`, `GEORADIUS` |

> Note: Some companies also use **Redis Stack** (modules) like RedisJSON. If you didn’t use it, don’t mention it unless asked.

---

## 4) Cache key structure (naming conventions)

### Goal
A key must uniquely represent **the response** you want to cache.

### Good key rules
- Use **namespaces**: `app:feature:entity:id`
- Include **version** when schema changes: `v1`, `v2`
- Include **parameters** that change output: page, sort, filters, locale, auth scope
- Keep it consistent (same order for params)

### Examples
- User by id:  
  `myapp:user:v1:123`
- Product page (public):  
  `myapp:product:v1:987`
- Products list with pagination + filters:  
  `myapp:products:v1:page=2:limit=20:sort=price:filter=shoes`
- Auth-scoped feed (per user):  
  `myapp:feed:v1:user=123`

### Interview one‑liner
> “I namespace keys, include a version, and include every parameter that changes the response—especially auth scope.”

---

## 5) Prevent huge payloads from blowing Redis memory

### What goes wrong
A few big keys can eat all RAM and cause mass eviction or OOM → DB meltdown.

### Practical controls
1) **Set memory limit + eviction policy**
- Configure Redis `maxmemory`
- Choose eviction policy (often `allkeys-lru` or `volatile-lru` depending on your strategy)

2) **Don’t cache huge blobs**
- Cache **IDs** or **summary**, not full giant objects
- Store big content in DB/S3; cache only a pointer + metadata

3) **Compression**
- Compress JSON before storing (e.g., gzip/brotli in app) if payloads are large

4) **Use HASH for big objects that change partially**
- Instead of rewriting 50KB JSON on every update, store fields in a hash and update only what changed

5) **Cap list sizes**
- For lists/logs, keep last N: `LTRIM key 0 999`

6) **Visibility**
- Check memory usage and big keys
  - `MEMORY USAGE <key>`
  - `INFO MEMORY`
  - `--bigkeys` in redis-cli (ops tool)

### Interview one‑liner
> “I control max memory + eviction policy, avoid caching huge objects, and store references/IDs when data is large.”

---

## 6) Negative caching (what it is + example)

**Negative caching = caching ‘not found’ results** to prevent repeated DB hits for missing keys.

### Example
If user id 999 doesn’t exist and bots keep requesting it:
- On DB miss, store sentinel:
  - `SET user:999 "__NULL__" EX 30`
- Next request hits cache and returns 404 quickly (no DB hit)

**Rule**
- Negative TTL should be **short** (10–60 seconds) to avoid hiding new data.

---

## 7) Cache invalidation for entity updates (user/product)

### Most common: Cache-aside + delete-on-write
**Write flow**
1) Update DB (source of truth)
2) `DEL myapp:user:v1:123` (invalidate)
3) Next read repopulates cache

**Why delete is safer than update**
- Updating cache means you must update **all derived keys** (lists, aggregates) → easy to miss
- Delete-on-write is simpler + correctness-friendly

### Race condition and “double delete”
Sometimes a read can refill stale data around the same time as a write.
A practical mitigation is:
1) update DB
2) delete cache
3) after a small delay, delete again  
(Used in some systems; better options exist like versioned keys.)

### Versioned keys (clean strategy)
- Keep a version number: `user:123:version = 7`
- Cache key: `user:123:v7`
- On update, bump version → new reads go to v8 automatically

---

## 8) Keep cache consistent across multiple app servers

### The real idea
Your app servers are stateless; Redis is shared. But consistency is mainly about **writes/invalidation**.

Practical patterns:
1) **Single shared Redis**
- All servers read/write the same cache keys

2) **Invalidate on write**
- Any server that updates DB deletes the related Redis key

3) **Pub/Sub invalidation (optional)**
- If you also have in-process caches, publish “invalidate user:123” so all servers drop local entries

4) **Avoid mixing users**
- For auth-dependent responses, key by **user id/role** so different servers don’t serve wrong data to wrong user

---

## 9) Strategy for “top N” / leaderboard (Redis ZSET)

### Why ZSET
It keeps items sorted by score automatically.

### Typical pattern
- Increment score:
  - `ZINCRBY leaderboard:global 10 user:123`
- Read top N:
  - `ZREVRANGE leaderboard:global 0 9 WITHSCORES`

### TTL & windows
For “daily leaderboard”:
- Key: `leaderboard:2026-01-23`
- Set TTL slightly longer than a day (or delete old keys via job)

### If you need user profiles too
- Get top ids from ZSET
- Then fetch details from DB/Redis hash/string in a batch

---

## 10) Distributed lock: what it is, and when you need it in caching

### What it is
A lock ensures **only one worker** does a critical operation at a time.

### Common caching use: preventing cache stampede
When a popular key expires, 100 requests miss and hammer DB.
Fix: one request rebuilds cache; others wait/fallback.

### Simple Redis lock pattern
- Acquire:
  - `SET lock:user:123 <randomValue> NX PX 5000`
- Release:
  - Only release if value matches (prevents unlocking someone else)

**When to use**
- Cache rebuild on hot keys (stampede)
- Idempotency / dedup (avoid double processing)
- Critical sections in distributed systems

---

## 11) Risk of caching exceptions/errors (and when it’s OK)

### Risk
- You can **amplify failure**: cache a temporary 500 and serve it for TTL even after recovery
- You may hide real fixes and create confusing behavior

### When it’s OK (with short TTL)
- **Negative caching** (404 for missing entity)
- Upstream rate-limited/expensive endpoints: cache “temporary failure” for **very short TTL** (like 1–5 sec) to avoid melting downstream
- Cache predictable errors (e.g., “invalid input”) only if you’re sure it’s deterministic

**Rule of thumb**
- Don’t cache 5xx unless you have a strong reason and **tiny TTL**.

---

# Section C – Questions answered (21–30)
These map directly to what interviewers ask:

21) Redis vs in-process: shared, TTL, atomic ops  
22) Data types: see table  
23) Key structure: namespace + version + params  
24) Huge payload prevention: maxmemory + avoid big blobs + compression  
25) Negative caching: cache misses with short TTL  
26) Invalidation: DB write then DEL / versioned keys  
27) Multi-server: shared cache + invalidation + optional pub/sub  
28) Leaderboard: ZSET  
29) Distributed locks: stampede prevention  
30) Caching errors: dangerous; OK for negative + tiny TTL for failures

---

## Extra Redis interview questions (add these to your practice)

### Redis basics & ops
1) What is Redis persistence (RDB vs AOF) and why might you enable/disable it for caching?
2) What’s the tradeoff between Redis as pure cache vs as durable store?
3) What are Redis eviction policies? When would you choose `volatile-lru` vs `allkeys-lru`?
4) How does Redis replication work at a high level? What is a replica good for?
5) What is Redis Cluster and what problem does it solve?
6) What are hash tags in Redis Cluster keys and why might you use them?
7) How do you monitor Redis (memory, hit rate, latency)?
8) What’s a “hot key” and how do you mitigate it in Redis?

### Data modeling questions
9) When do you store a JSON blob in STRING vs fields in HASH?
10) How do you model sessions in Redis? How do you expire sessions?
11) How do you implement rate limiting in Redis (token bucket / sliding window)?
12) How do you implement “unique daily active users” with HyperLogLog?
13) How do you implement a queue with LIST vs with STREAM?
14) How do you design idempotency keys with Redis?

### Consistency & correctness
15) Explain cache stampede and show the lock solution.
16) What’s the race condition when you delete cache on write? How do you reduce it?
17) How do you handle partial failures (Redis down, DB down)?
18) How do you do safe cache key changes during deployments?

### Performance
19) Why is pipelining useful with Redis?
20) Why is `MGET`/batching useful for request latency?
21) What is the cost of very large hashes/sets? When do you split keys?

---

## Ultra-short cheat sheet (for interviews)

- **Most common pattern:** cache-aside + delete-on-write  
- **Best datatype defaults:**
  - Single value → STRING
  - Object fields → HASH
  - Unique membership → SET
  - Ranking/top N → ZSET
- **Security:** auth-dependent responses must be keyed by user/role (`private` caching)
- **Big risks:** stale data, stampede, hot keys, memory blowups

---

## 1-minute “Redis caching” answer (say this)
> “I use Redis as a shared cache across app servers. I follow cache-aside: reads check Redis first, on miss fetch from DB and cache with TTL; writes update DB then invalidate the cache key. I design keys with namespaces + version + params, protect hot keys from stampedes with a Redis lock, and use appropriate structures like hashes for objects and sorted sets for leaderboards. I also set max memory and eviction policy to prevent large keys from taking down the cache.”


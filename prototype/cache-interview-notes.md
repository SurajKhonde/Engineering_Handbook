# Cache – Interview Notes (Full‑Stack)

These notes are **interview-ready** and organized for quick revision.

---

## Table of Contents
- [A) Core concepts (1–10)](#a-core-concepts-110)
- [B) Cache patterns (11–20)](#b-cache-patterns-1120)
- [C) Redis / backend caching (21–30)](#c-redis--backend-caching-2130)
- [D) Failure modes (31–40)](#d-failure-modes-3140)

---

## A) Core concepts (1–10)

### 1) What is caching, and why does it improve performance?
**Caching** = storing a copy of frequently used or expensive-to-fetch data in a faster place (**memory / Redis / CDN**) so future reads avoid DB/compute.

It improves performance by:
- **Reducing latency** (memory is faster than DB / network calls)
- **Reducing load** on DB/services (fewer repeated reads/computations)

---

### 2) Cache hit vs cache miss — define and impact on latency
- **Cache hit:** data found in cache → return fast → **low latency**
- **Cache miss:** not in cache → fetch from DB/service → **higher latency** → store result in cache for next time

---

### 3) What is TTL? How do you choose TTL for different data?
**TTL (Time To Live)** = how long cached data remains valid before it expires.

Choose TTL based on **staleness tolerance + update frequency**:
- **User profile/settings:** 5–30 min (changes rarely)
- **Counts/feeds:** 30–120 sec (changes often)
- **Stock price/live data:** very small TTL (1–5 sec) or **don’t cache**

Tip: add **TTL jitter** (random ±10–20%) so many keys don’t expire together.

---

### 4) What is cache eviction and why does it happen?
**Eviction** = removing cached entries when the cache is full to free memory.

It happens because cache has **limited RAM**, so it must decide which keys to drop using policies like **LRU/LFU**.

---

### 5) Explain LRU vs LFU eviction policies
- **LRU (Least Recently Used):** remove the key **not used recently**  
  Good when: “recently used = likely to be used again”
- **LFU (Least Frequently Used):** remove the key used **least number of times**  
  Good when: “popular keys stay popular for a long time”

**Memory trick**
- LRU = **R** = **Recent**
- LFU = **F** = **Frequency**

---

### 6) What is “stale data” in cache? Why is it risky?
**Stale data** = cache contains outdated value compared to DB (**source of truth**).

Risk:
- users see wrong info
- business logic can make wrong decisions (prices, permissions, status)

---

### 7) What is the “source of truth” and why does it matter in caching?
**Source of truth** = the system that holds the correct final data (usually DB).

Cache is a **derived copy**, so you must handle **TTL/invalidation** to avoid serving stale data.

---

### 8) When should you NOT use caching?
Don’t cache when:
- **Correctness is critical** and stale is unacceptable (money transfers, inventory deduction steps)
- Data changes extremely frequently and invalidation cost > benefit
- You can’t key safely by user/permissions (risk of leaking data)

> Sensitive data *can* be cached if scoped correctly (per-user key + short TTL + `private/no-store` for HTTP, etc.).

---

### 9) What metrics define cache effectiveness?
Key metrics:
- **Cache hit rate (%)** = hits / total requests
- **Latency**: p50/p95/p99 for hits vs misses
- **DB QPS reduction** (cache should reduce DB traffic)
- **Eviction rate** (too high may mean cache too small)
- **Memory usage** + key size distribution

---

### 10) What’s the difference between caching and buffering?
- **Caching:** store data to speed up **future reads** (reuse)
- **Buffering:** temporarily hold data to handle **producer/consumer speed mismatch** (smooth I/O)

Examples:
- Cache: user profile cached for faster next request
- Buffer: video playback buffer, network socket buffer

---

## B) Cache patterns (11–20)

### 11) Explain cache-aside (lazy loading) with read/write flow
**Cache-aside** = the **application controls** the cache.

- **Read:** cache → miss → DB → put in cache (TTL) → return  
- **Write:** update DB → **invalidate cache key** (`DEL`) (or update cache if safe)

**Memory line:** “Read: fill on miss. Write: DB then invalidate.”

---

### 12) Explain write-through caching
**Write-through** = every write goes to **cache AND DB synchronously**.

- **Pros:** cache stays fresh, reads are easy  
- **Cons:** higher write latency; cache must be highly available

---

### 13) Explain write-back / write-behind caching
**Write-back** = write to cache first, DB later asynchronously (queue/worker).

- **Pros:** very fast writes  
- **Cons:** risk of data loss if cache crashes before DB flush; harder consistency; needs retry + durability design

---

### 14) Compare cache-aside vs write-through — which is common and why?
**Cache-aside is most common** because:
- it’s simple
- DB stays source of truth
- if cache fails, you can still serve from DB

**Write-through** is used when you want cache always warm/fresh and can accept slower writes.

---

### 15) Pros/cons: delete cache on write vs update cache on write

**Delete on write (invalidate)**
- ✅ Simple, avoids serving stale
- ✅ Works well with cache-aside
- ❌ Next read is a miss (extra DB hit)

**Update cache on write**
- ✅ Keeps cache warm, fewer misses
- ❌ Risk of race conditions (DB update + cache update ordering)
- ❌ Hard when multiple keys depend on same data (lists, aggregates)

Interview line:
> “If correctness matters, I prefer DB write + cache invalidate. Updating cache is OK when I can update all related keys safely.”

---

### 16) What is read-through cache? How is it different from cache-aside?
**Read-through**
- App asks cache; cache itself loads from DB on miss (library/proxy handles it)
- App doesn’t write “if miss then DB” logic

**Cache-aside**
- App code handles misses: cache → DB → cache

One line:
> Read-through = cache loads for you. Cache-aside = app loads.

---

### 17) What are versioned keys? When do you use them?
**What:** include a version in key, e.g. `user:123:v7`

**On update:** bump version (store `user:123:version = 8`), so reads automatically use `v8`. Old keys die by TTL.

Use when:
- many derived keys (feed/list/aggregates) and invalidation is messy
- you want safe “instant switch” without chasing all old keys

---

### 18) How do you cache a paginated list safely?
Rule: cache key must include all dimensions:
- `products:page=2:limit=20:sort=price:filter=shoes`

Tips:
- Use TTL (lists change often)
- Invalidate on writes that affect list OR use a versioned “list version”
- Don’t cache infinite pages forever (only top pages / popular filters)

---

### 19) How do you cache a response that depends on filters/sort?
Key = endpoint + every parameter that changes output:
- filters
- sort
- pagination
- language/region
- device variant (if response changes)

Also normalize params (stable order) so keys don’t explode.

---

### 20) How do you cache data that depends on permissions / auth?
Safe strategies:
- **Per-user cache keys**: `feed:user=123`
- **Per-role / permission-set keys** (if shared): `dashboard:role=admin`
- HTTP caches: `Vary: Authorization` / `Cache-Control: private`
- Avoid caching private responses in shared CDN caches unless strictly controlled

Interview line:
> “If output depends on auth, cache by user id / role, and ensure shared caches don’t mix users.”

---

## C) Redis / backend caching (21–30)
- [Redis Notes](redis-cache-interview-readme.md)

---

## D) Failure modes (31–40)

### 31) Cache stampede / thundering herd — what & fixes
**What:** A popular key expires (or is missing) and many requests miss at once → they all hit DB together → DB spikes/outages.

**Fixes (best → common):**
- Single-flight / request coalescing (only one rebuilds cache)
- Distributed lock (Redis lock)
- Stale-while-revalidate (serve stale briefly while one refreshes)
- Pre-warm hot keys (deploy/scheduled)
- Rate limit rebuilds + DB reads

One-liner:
> “Prevent stampede by making cache rebuild single-threaded per key.”

---

### 32) Hot key problem — what & mitigation
**What:** One key gets massive traffic → Redis CPU/network bottleneck, latency spikes.

Mitigate:
- Local (in-process) LRU for ultra-hot keys (tiny TTL)
- Replicas / read scaling (if supported)
- Shard the key (advanced)
- Cache at CDN/edge if public
- Avoid huge responses; cache smaller summary

One-liner:
> “Hot key = one cache key becomes your bottleneck; fix by adding another cache layer or spreading load.”

---

### 33) Cache penetration — what & prevention
**What:** Many requests for non-existent items → always miss cache → hammer DB.

Prevention:
- Negative caching (“NOT_FOUND” with short TTL)
- Input validation
- Bloom filter (advanced)
- Rate-limit suspicious patterns

One-liner:
> “Penetration = misses for keys that will never exist; fix with negative caching + validation.”

---

### 34) Cache avalanche — what & fix
**What:** Many keys expire together → huge miss wave → DB meltdown.

Fix:
- TTL jitter (randomize expiry)
- Staggered refresh / warming for hot segments
- Grace TTL (stale-while-revalidate)
- Multi-level cache (local + Redis + DB)
- Ensure Redis has enough memory (avoid mass eviction)

One-liner:
> “Avalanche = synchronized expiry; fix by desynchronizing expirations and serving stale briefly.”

---

### 35) How TTL jitter helps
Instead of `TTL = 300s` for everyone, use `TTL = 300s ± random(0..60s)`.

Why: prevents many keys expiring at the same moment, reducing stampede/avalanche risk.

---

### 36) If Redis goes down — what should the app do?
You need a degrade strategy:

Options:
- **Fail-open (common for cache):** bypass cache → read DB directly (risk: DB load spike)
- **Fail-closed (common for auth/rate-limit):** block/limit requests for safety

In practice:
- Circuit breaker (stop hitting Redis briefly if failing)
- Serve stale from local memory (if available)
- Disable non-critical features (recommendations)
- Backpressure / rate limit to protect DB

One-liner:
> “Cache down should not take the site down—use bypass + circuit breaker + DB protection.”

---

### 37) Fallback without melting the DB
Protect DB when cache is unhealthy:
- Circuit breaker on Redis and DB-heavy endpoints
- Rate limit expensive endpoints
- Bulkheads (separate pools for critical vs non-critical)
- Stale-while-revalidate (serve stale instead of DB hit)
- Request coalescing (100 misses → 1 DB query)
- Load shedding for non-critical paths

Mental model:
> When cache fails, you must “shape traffic” to DB.

---

### 38) Monitoring cache health/performance
Track:
- Hit rate (overall + per endpoint)
- Redis op latency p50/p95/p99
- Error rate / timeouts
- Evictions (spike = memory pressure)
- Memory usage & fragmentation
- CPU and network I/O
- Key count and big keys (memory per key)
- Replication lag / failovers (if applicable)

---

### 39) “Double delete” race in invalidation — what & handling
Race:
1) Request A reads DB (old)
2) Request B updates DB and deletes cache
3) Request A writes old value back into cache → cache becomes stale

Fixes:
- **Versioned keys (cleanest)**
- Delayed double delete (delete → wait 100–500ms → delete again)
- Update-cache-after-commit (only if you can update all related keys safely)
- Locks on rebuild for hot keys

One-liner:
> “Versioned keys are the cleanest; delayed double delete is a practical mitigation.”

---

### 40) How caching can make a bug worse (real scenarios)

**Scenario 1: Wrong cache key → data leak**
Caching `GET /profile` as `profile` (missing userId/auth scope)  
→ user A’s profile cached, user B receives it.

Fix:
- include user/role in key
- `Cache-Control: private`
- `Vary: Authorization`

**Scenario 2: Cached 500s amplify outage**
A temporary upstream failure gets cached for 60s  
→ even after recovery, users keep seeing errors.

Fix:
- don’t cache 5xx
- if needed, cache for 1–5s max

**Scenario 3: Negative caching hides new data**
Cache `NOT_FOUND` for 5 minutes; user gets created; app keeps returning 404 until TTL.

Fix:
- short negative TTL (10–60s)
- delete negative key on create

---

# System Design — One Framework + One Full Example

Learn **one** framework, apply it to **every** question. This doc gives you a 7-step framework you can use for any design interview, then runs **"Design a URL shortener"** through all 7 steps so you see exactly how it works.

---

## Part 1 — The Framework (memorise these 7 steps)

Say them in this order, every time. Spend the first few minutes on steps 1–2 (interviewers reward this most), then move to design.

| # | Step | What you do | The one sentence to say |
|---|---|---|---|
| 1 | **Clarify requirements** | Functional (what it does) + non-functional (how well) + scope (what's out) | "Before I design, let me confirm what we're building and what we're optimising for." |
| 2 | **Estimate scale** | Back-of-envelope: QPS (reads vs writes), storage, read/write ratio | "Let me size this — it tells us where the pressure is." |
| 3 | **Design the API** | The endpoints: request → response | "Here are the operations the system exposes." |
| 4 | **Design the data model** | Schema + SQL-vs-NoSQL choice, justified by access pattern | "The access pattern decides the database." |
| 5 | **High-level design (HLD)** | Boxes & arrows: client → LB → app → cache → DB | "Here's the flow end to end." |
| 6 | **Deep dive** | The one hard/interesting component (an algorithm or the key API) | "The interesting part is X — let me go deep there." |
| 7 | **Scale & trade-offs** | Bottlenecks, caching, sharding, replication, failure modes | "Where does this break, and how do I fix it?" |

### The golden rules
- **Drive the conversation, but ask first.** Step 1 is questions, not assumptions. Interviewers are testing whether you scope before you build.
- **Let scale drive design.** A 1000-user app and a billion-user app are different systems. Step 2 justifies every later decision.
- **Name the trade-off behind every choice.** "I chose X over Y because Z, and the cost is W." That sentence is what makes you sound senior.
- **There's no single right answer.** They're testing your *reasoning*, not whether you memorised an architecture.

---

## Part 2 — Full Example: Design a URL Shortener

Now we run the framework. (This is the classic warm-up question — know it cold.)

### Step 1 — Clarify requirements

Start by asking, not designing. These are *your* notes ("who uses this, what they get, how many uploads, can two users upload the same URL") turned into proper requirements.

**Functional (what it does):**
- Given a long URL, return a short URL. (`https://example.com/very/long/path?x=1` → `short.ly/aB3xY7`)
- Visiting the short URL **redirects** to the original.
- Optional: user can request a **custom alias** (`short.ly/suraj`).
- Optional: links can **expire**.
- Optional: basic **analytics** (click count).

**Non-functional (how well it must work):**
- **Read-heavy.** Far more redirects (reads) than creations (writes) — typically ~100:1. This single fact drives the whole design toward caching.
- **Low latency** on redirects — the user clicks and expects an instant jump.
- **High availability** — a redirect failing is worse than a creation failing.
- Short codes should be **non-guessable-ish** (don't leak a sequential counter that lets people enumerate everyone's links).

**Clarifying questions to ask out loud (this is what earns points):**
- *"Who uses it?"* — public service, or internal tool? (affects auth, abuse)
- *"How many URLs created per day?"* — sizes the system.
- *"If two users shorten the same long URL, do they get the same short code or different ones?"* — **design decision.** Default: **different codes**, because each user may want their own analytics, expiry, and ownership. (Deduping saves storage but couples users together — usually not worth it.)
- *"Do we need custom aliases? Expiry? Analytics?"* — scope.
- *"Any rate limit per user?"* — e.g. *"a free user can create 4 links per day."*

> Saying "let me confirm what we're optimising for" before drawing anything is the single highest-value move in the whole interview.

### Step 2 — Estimate scale (back-of-envelope)

This is where you prove the design is read-heavy and decide where caching matters.

Assume **100M new URLs / month** (a reasonable interview number):

- **Write QPS:** 100M ÷ (30 × 24 × 3600) ≈ 100M ÷ 2.6M ≈ **~40 writes/sec**.
- **Read QPS** at 100:1 ratio ≈ **~4,000 reads/sec** (with peaks several times higher).
- **Storage:** 100M/month × 12 × 5 years ≈ **6 billion URLs**. At ~500 bytes/record → **~3 TB**. Fits comfortably on a sharded store.

**What the estimate tells us:** writes are tiny, reads are huge → **redirects must be fast and cached.** Storage is large but not extreme. So the design centres on a fast read path (Redis in front of the DB) and a cheap, collision-free way to generate codes.

### Step 3 — Design the API

Three core endpoints (plus optional analytics):

**1. Create short URL**
```
POST /api/shorten
Body: { "longUrl": "https://...", "customAlias": "suraj"?, "expiresAt": "2026-12-31"? }
Response: 201 { "shortUrl": "https://short.ly/aB3xY7", "shortCode": "aB3xY7" }
Errors: 400 invalid URL, 409 alias taken, 429 rate limit exceeded
```

**2. Redirect (the hot path)**
```
GET /{shortCode}
Response: 302 Found, Location: <longUrl>
Errors: 404 not found, 410 gone (expired)
```

**3. Stats (optional)**
```
GET /api/urls/{shortCode}/stats
Response: 200 { "shortCode": "aB3xY7", "longUrl": "...", "clicks": 1280, "createdAt": "..." }
```

**4. Delete (optional)**
```
DELETE /api/urls/{shortCode}   → 204 No Content
```

### Step 4 — Design the data model

**SQL vs NoSQL — decide from the access pattern, not from fashion.**

The access pattern is dead simple: **look up one row by `short_code`** (and the reverse for creation). No joins, no complex queries, but enormous read volume.

- **NoSQL key-value store** (DynamoDB, Cassandra) fits this naturally — `short_code` is the key, horizontal scaling is built in, and there are no relational needs. Best at *billion-scale*.
- **PostgreSQL** is completely fine up to large scale, simpler to operate, gives you transactions (useful for the ID counter) and easy secondary indexes. At your stack and most real startup scale, **Postgres + Redis** is the pragmatic, defensible choice.

> The honest interview answer: "The access pattern is a key-value lookup, which favours a key-value store at extreme scale for horizontal sharding. But at realistic scale I'd start with Postgres because it's simpler and gives me transactions for ID generation, then move hot reads into Redis. I'd only reach for Cassandra/DynamoDB when a single Postgres can't hold the write/storage volume." That's reasoning, not dogma — exactly what they want.

**Schema (Postgres):**

```sql
CREATE TABLE urls (
  id          BIGSERIAL PRIMARY KEY,         -- the integer we base62-encode
  short_code  VARCHAR(10) UNIQUE NOT NULL,   -- e.g. "aB3xY7", indexed for lookup
  long_url    TEXT NOT NULL,
  user_id     BIGINT REFERENCES users(id),
  created_at  TIMESTAMPTZ DEFAULT now(),
  expires_at  TIMESTAMPTZ,                   -- nullable; NULL = never expires
  click_count BIGINT DEFAULT 0
);

CREATE UNIQUE INDEX idx_urls_short_code ON urls (short_code);
CREATE INDEX idx_urls_user_id ON urls (user_id);   -- for "my links" + rate limit
```

The `short_code` index is the one that matters — every redirect hits it. (Reads usually go through Redis first, so the DB index is the cache-miss fallback.)

### Step 5 — High-level design (HLD)

This is the boxes-and-arrows part you asked for — **client → server → base62 → Redis**. There are two paths: **write** (create) and **read** (redirect, the hot one).

**Write path — creating a short URL:**
```
   ┌────────┐     ┌──────────────┐     ┌───────────────┐     ┌────────────┐
   │ Client │────▶│ Load Balancer│────▶│  App Server    │────▶│ ID Counter │
   └────────┘ POST└──────────────┘     │  (Node/TS API) │     │ (next id)  │
                                        └──────┬─────────┘     └────────────┘
                                               │ base62(id) = "aB3xY7"
                                               ▼
                                        ┌───────────────┐
                                        │  PostgreSQL    │  store row
                                        └──────┬─────────┘
                                               │ also warm the cache
                                               ▼
                                        ┌───────────────┐
                                        │     Redis      │  short_code → long_url
                                        └───────────────┘
                            returns:  https://short.ly/aB3xY7
```

**Read path — redirect (this is the hot, latency-critical one):**
```
   ┌────────┐  GET /aB3xY7  ┌──────────────┐     ┌────────────────┐
   │ Client │──────────────▶│ Load Balancer│────▶│   App Server   │
   └────────┘               └──────────────┘     └───────┬────────┘
        ▲                                                 │ 1. check cache
        │  302 Location: <long_url>                       ▼
        │                                          ┌────────────┐
        │                                HIT ◀──────│   Redis    │
        │                                          └─────┬──────┘
        │                                                │ MISS
        │                                                ▼
        │                                          ┌────────────┐
        └──────────────────────────────────────── │ PostgreSQL │ (then populate Redis)
                                                   └────────────┘
```

**The flow in words:**
- **Create:** client POSTs the long URL → app server gets the next unique integer ID from a counter → **base62-encodes** that ID into a short code → stores `(short_code, long_url)` in Postgres → warms Redis → returns the short URL.
- **Redirect:** client hits `GET /aB3xY7` → app checks **Redis first** (cache hit → instant 302) → on miss, reads Postgres, then **populates Redis** so the next hit is fast → returns a 302 redirect.

That cache-first read path is the whole point of recognising it's read-heavy in Step 2.

### Step 6 — Deep dive: how the short code is generated (base62)

This is the interesting algorithmic part, and it answers your "base64 or other conversion" question directly.

**Why base62, NOT base64.** base64's alphabet includes `+`, `/`, and `=` (padding) — none of which are URL-safe; they'd need percent-encoding and make ugly links. **base62 uses only `[0-9a-zA-Z]`** — 62 characters, all URL-safe, no escaping needed. That's why every URL shortener uses base62 (or base62-like), not base64.

**How many codes does base62 give us?**
- 6 chars → 62⁶ ≈ **56 billion**
- 7 chars → 62⁷ ≈ **3.5 trillion**

7 characters is plenty for our 6 billion URLs, with huge headroom.

**Two ways to generate the code — and which to pick:**

1. **Hash the long URL** (MD5/SHA → take first N chars → base62). Problem: hash collisions are possible (two different URLs → same code), so you need collision detection and retry. Also same URL always → same code, which conflicts with our "different users get different codes" decision.

2. **Counter + base62-encode the ID (preferred).** Each new URL gets a unique incrementing integer ID; you base62-encode that integer to get the code. **No collisions, ever** — IDs are unique by construction. The code length grows naturally as the ID space fills.

We pick **#2**: take the unique `id`, encode it to base62.

```typescript
// 62-char URL-safe alphabet
const ALPHABET = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

function encodeBase62(id: number): string {
  if (id === 0) return ALPHABET[0];
  let code = "";
  while (id > 0) {
    code = ALPHABET[id % 62] + code;
    id = Math.floor(id / 62);
  }
  return code;
}
// encodeBase62(125) -> "21"   encodeBase62(999999) -> "4c91"
// For IDs beyond Number.MAX_SAFE_INTEGER, use BigInt.
```

**One problem with a raw counter:** sequential IDs → sequential, *guessable* codes (`aaa`, `aab`, `aac`…), so people could enumerate everyone's links. Two common fixes:
- **Hand out ID ranges** to each app server (a "ticket server" gives server A ids 1–1000, server B 1001–2000) so a single counter isn't a bottleneck *and* IDs aren't perfectly sequential globally.
- Or **encode with a shuffled/offset alphabet** so consecutive IDs don't produce visually consecutive codes.

> The deep-dive talking point: "I'd generate codes from a unique counter and base62-encode it for guaranteed no collisions, then break up the global counter into per-server ID ranges to avoid both a single bottleneck and trivially guessable codes."

### Step 7 — Scale & trade-offs

Now answer "where does it break, and how do I fix it" — your Redis, rate-limit, and redirect notes live here.

**Caching (your "hot point in Redis").** Redirects are 99% of traffic and most clicks concentrate on a small set of popular links. So cache `short_code → long_url` in **Redis with LRU eviction**: hot links stay hot, cold ones fall out. A cache hit is an instant 302; a miss reads Postgres and back-fills Redis. This is what lets ~4,000 reads/sec stay fast without hammering the DB.

**Rate limiting (your "4 per day").** Exactly the Redis pattern you already use in Pilooopu: a counter keyed by user with a daily TTL.
```
INCR ratelimit:{userId}:{yyyy-mm-dd}
EXPIRE ratelimit:{userId}:{yyyy-mm-dd} 86400
if count > 4  →  429 Too Many Requests
```
The counter auto-resets when the day's key expires.

**301 vs 302 — a real trade-off you should raise:**
- **301 (permanent):** the browser caches the redirect, so repeat clicks skip your server → less load. But you **lose analytics** on repeat visits and can't change the target later.
- **302 (temporary):** every click hits your server → you get **full analytics** and can change/expire the target, at the cost of more traffic.
- **Pick 302** if analytics matter (they usually do for a shortener); 301 if you purely want to offload traffic. Naming this trade-off is a senior signal.

**Database scaling:** at billions of rows, shard by `short_code` (e.g. hash-based) or move the key-value lookups to Cassandra/DynamoDB. Add **read replicas** for analytics queries so they don't touch the write path.

**Availability:** redirects must survive failures — multiple app servers behind the LB, Redis with replication, Postgres with a replica + failover. A read-heavy redirect service should degrade gracefully (serve from cache even if the DB is briefly down).

**Custom aliases:** when a user picks `short.ly/suraj`, you can't use the counter — you must check uniqueness (`UNIQUE` constraint on `short_code` does this; insert fails → return 409). So custom aliases and generated codes share the same column and uniqueness guarantee.

**Expiry:** a background job (or lazy check on read) removes/410s expired links; lazy-on-read is simpler — if `expires_at < now()`, return 410 and evict from cache.

---

## How to use this on the day

1. **Steps 1–2 first, slowly.** Clarify requirements, estimate scale. Do not start drawing boxes in minute one.
2. **Let the read/write ratio drive everything.** "It's read-heavy → cache-first" is the spine of this whole design.
3. **base62, not base64** — and counter-based IDs for no collisions. That's the memorable deep-dive.
4. **End every component with its trade-off** — SQL vs NoSQL, 301 vs 302, dedupe vs per-user codes. The trade-offs *are* the interview.
5. **The same 7 steps work for any prompt** — "design a rate limiter", "design a chat app", "design a job queue" (you've literally built that — Pilooopu's BullMQ pipeline is a great one to volunteer). Swap the example, keep the framework.

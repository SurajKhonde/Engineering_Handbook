# Interview Roadmap (Full‑Stack: JS/TS + React + Node + Redis + BullMQ)
**Profile:** ~3 years experience • last CTC: 9 LPA (India) • target: mid product/service roles  
**Goal:** Start applying fast while sharpening coding + explanation + interview readiness.

---

## 1) When to start applying (the “ready enough” rule)
Don’t wait for “perfect.” Start applying when you can do these reliably:

### ✅ Minimum readiness checklist (2–4 days of focused prep)
1. **Resume is clean + honest + impact-focused**
   - 1 page (or max 2), strong summary, 3–5 bullets per role with *numbers* (latency, cost, scale, users, tickets).
2. **Your 60-second intro is ready** (and repeatable)
3. **Core coding baseline**
   - You can solve **easy** in 15–25 min and **medium** in 30–45 min with clean JS.
4. **Core stack story**
   - React + Node + DB + Redis + background jobs: can explain *why* you used them and tradeoffs.
5. **System design “light”**
   - Can design a basic API + DB schema + caching + background worker + failure handling.

👉 If you can do the above, **start applying immediately** while continuing prep daily.

---

## 2) Your best strategy (because time is limited)
### Apply + Prepare in parallel
- **Day 1–2:** build fundamentals + interview scripts
- **Day 3 onward:** start applying + daily practice

Why? Interview pipelines take time. If you wait, you lose weeks.

---

## 3) 7‑Day Sprint Plan (Before heavy applying)
Assume **8–12 hrs/day**. If less, keep the same order.

### Day 1 — Setup + “Explain like a pro”
**Output by end of day:**
- Resume v1
- 60s intro + 2 min deep intro
- Project pitch for 2 projects (even 1 strong is okay)

**Tasks**
- Resume: rewrite bullets using this format:
  - **Action + Tech + Result + Number**
  - Example: “Built BullMQ worker pipeline to process invoices asynchronously, reducing API p95 latency from 900ms to 200ms.”
- Prepare interview scripts:
  - Tell me about yourself (60s)
  - What did you build recently?
  - Biggest challenge + how you solved
  - Conflict/ownership story
  - Why left / why looking (neutral, short)

**Practice**
- 2 easy DSA (arrays/strings)
- 1 coding drill: implement `debounce`, `throttle` (JS)

---

### Day 2 — JavaScript depth (most important)
**Topics**
- Execution: call stack, event loop, microtask vs macrotask
- Closures, scopes, hoisting
- `this`, prototype, classes vs prototypes
- Promises, async/await, error handling patterns
- Common output questions (setTimeout, Promise chains)

**Practice**
- 2 DSA (sliding window + hashmap)
- 1 “output prediction” sheet (10 questions)
- Explain aloud while coding (screen record yourself)

---

### Day 3 — Node.js + Backend fundamentals
**Topics**
- Node event loop, libuv, threadpool, streams/buffers
- Express/Nest style architecture: controllers/services/repo
- Auth basics: JWT, refresh tokens, hashing, cookies vs localStorage
- Pagination, filtering, rate limiting
- File upload/download: streams, multipart, S3 patterns (if used)

**Practice**
- Build 1 mini API: CRUD + validation + pagination
- 2 DSA (two pointers + stack/queue)

Start applying **today** (even 5–10 applications).

---

### Day 4 — DB + Querying (MySQL or MongoDB)
**MySQL focus**
- Indexing, composite indexes, query plan basics
- Transactions, isolation levels (at least basics)
- Joins, group by, pagination pitfalls (offset vs cursor)

**Mongo focus**
- Indexes, aggregation basics
- Modeling: embedding vs referencing
- Consistency tradeoffs

**Practice**
- 2 DSA (binary search + prefix sum)
- 1 schema design: “posts + likes + comments” (tables or collections)

---

### Day 5 — Redis + Caching (your advantage)
**Topics**
- When to cache (read-heavy, expensive compute)
- TTL, cache invalidation strategies
- Cache-aside vs write-through vs write-back (know cache-aside best)
- Rate limiting (token bucket / sliding window)
- Redis data structures: string, hash, list, sorted set

**Practice**
- Implement cache-aside in Node: `getOrSet(key, ttl, fetchFn)`
- 2 DSA (graph BFS/DFS basics or heap basics)

---

### Day 6 — BullMQ / background jobs (your specialty)
**Topics**
- Producer/queue/worker flow
- Idempotency, retries, backoff, dead-letter pattern (DLQ concept)
- Exactly-once vs at-least-once (BullMQ is at-least-once)
- Job deduplication keys
- Handling failures + monitoring

**Practice**
- Build: “email sender / report generator”
  - API creates job → worker processes → status endpoint checks progress
- 2 DSA (DP basics: 1D DP like climbing stairs, house robber)

---

### Day 7 — System design (light but strong)
**Use a repeatable template**
1) Requirements (functional + non-functional)  
2) APIs  
3) Data model  
4) High-level architecture  
5) Bottlenecks + scaling  
6) Caching + async jobs  
7) Reliability (timeouts, retries, circuit breaker)  
8) Observability (logs/metrics/traces)

**Practice prompts**
- Design URL shortener
- Design notification system
- Design feed (basic)
- Design file upload service

**Practice**
- 1 full design on paper (45–60 min)
- 1 mock interview with a friend OR record yourself

---

## 4) After you start applying (Daily schedule)
Aim for **4–6 hours prep** on interview days; **8 hours** on non-interview days.

### Daily non-negotiables (2.5–4 hrs)
1. **DSA:** 2 problems/day (easy + medium)  
2. **System design:** 30–45 min (1 topic/day)  
3. **Explain & communicate:** 20 min (talk through a solution out loud)

### Weekly targets
- 10–14 DSA problems/week
- 2 system design mocks/week
- 1 project deep-dive/week (refactor, tests, README)

---

## 5) Topic Checklist (what interviewers dig into)

### JavaScript (Depth)
- Event loop (microtasks/macrotasks)
- Promises chaining, `Promise.allSettled`, error propagation
- Closures & memory leaks
- `this`, bind/call/apply
- Prototypes, inheritance
- Array methods vs loops performance (basic understanding)
- Immutability, shallow vs deep copy

### TypeScript
- Types vs interfaces
- Generics (basic), utility types (Partial, Pick, Omit, Record)
- Type narrowing, unions/intersections
- `unknown` vs `any`
- TS config basics (strict mode)

### React
- Rendering, reconciliation basics
- State vs props, lifting state
- useEffect pitfalls, dependency array
- useMemo/useCallback (when/why)
- React Query / Redux Toolkit basics (if you used Redux)
- Performance: memoization, virtualization, splitting bundles (basic)

### Node.js / Backend
- Event loop, threadpool, streams
- REST API design, validation
- Auth & security basics (CORS, CSRF, XSS awareness)
- Rate limiting, pagination, file handling
- Testing: unit + integration (basic)

### Databases
- Indexing (must know)
- Transactions basics
- Normalization basics
- Cursor vs offset pagination

### Redis
- Cache patterns + invalidation
- Rate limiting patterns
- Distributed lock (basic concept)
- Pub/Sub concept (optional)

### BullMQ / Background Jobs
- Retries, backoff, idempotency
- Job status tracking
- Concurrency & throughput
- DLQ concept, poison messages

### System design (light)
- API + DB + cache + async jobs
- Sharding/partitioning basics (only concept)
- CDN basics
- Observability + alerting basics

---

## 6) Your “Story Pack” (prepare these answers)
Write them in notes and rehearse.

1) **Tell me about yourself (60s)**
- Present role + years + domain
- 1–2 strongest projects
- Strength: async jobs + caching + full-stack ownership
- What you want next

2) **Best project deep dive (3–5 min)**
- Problem → constraints → approach → results → learnings

3) **Failure story**
- What happened, how you fixed, what changed after

4) **Why you left / why looking**
- Keep it calm and short:
  - “Role changes / project ended / looking for better growth & product exposure.”

5) **Salary expectation**
- Have a range ready based on your target companies + location.
- Always say: “Open to discussion based on role and total comp.”

---

## 7) Practice methods (to regain “loose hand”)
### The “Explain while coding” rule
- Every DSA problem: narrate your approach:
  - brute force → optimize → complexity → edge cases

### 30-minute daily code writing drills
Pick 1:
- implement `LRU cache` (small)
- implement `debounce/throttle`
- build a mini express middleware (rate limit/logger)
- implement `retry(fn, retries, delay)` with Promises

### Mock interviews
- Record yourself 2 times/week (screen + voice). Review:
  - clarity, pauses, structure, confidence

---

## 8) What to prioritize (if time is extremely tight)
1) JS fundamentals + promises/event loop  
2) DSA patterns: sliding window, hashmap, two pointers, stack, BFS/DFS basics  
3) One strong project story + architecture  
4) Redis cache + BullMQ explanation  
5) System design template + 2 common designs

---

## 9) Quick self-check scoring (use before interviews)
Rate 0–2 each (0 = weak, 2 = strong). Target total **14+ / 20**.
- Intro & communication
- JS depth
- React basics
- Node/backend basics
- DB basics
- Redis + jobs
- DSA easy
- DSA medium
- System design light
- Project deep dive

---

## 10) Simple next step checklist (today)
- [ ] Resume v1 ready
- [ ] 60s intro written + practiced 5 times
- [ ] Pick 1 “hero project” + prepare deep dive
- [ ] Solve 2 DSA problems + explain aloud
- [ ] Apply to 5 jobs

---

### Notes
This plan is designed so you can start applying by **Day 3** at the latest, while your prep continues daily.

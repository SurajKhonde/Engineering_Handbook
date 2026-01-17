# Node.js / Backend Interview Notes — Topics 4–10 (Detailed, Memorize-Friendly)

Use this answer pattern in interviews: **Definition → Why → How → Example → Tradeoff**.  
Each answer below follows that flow + includes a **mental model**.

---

# 4) Timers, Scheduling & Task Queues

## 4.1 `setTimeout(fn, 0)` — what does “0” really mean?

**Definition:** `0` means **minimum delay**, not “run immediately.”  
**Why:** Node must finish current call stack + event loop bookkeeping; timers run in the **timers phase**.  
**How:** Node schedules the callback for “as soon as the timers phase is reached *after* at least 0ms have elapsed.”  
**Example:** If your code blocks CPU for 200ms, your `setTimeout(..., 0)` runs **after** that block.  
**Tradeoff:** Good for “defer work,” but **not deterministic** for strict ordering.

**Mental model:** `setTimeout(0)` means **“later, when you get a chance”**, not “now.”

---

## 4.2 Compare `setTimeout` vs `setImmediate` in Node (typical behavior + when it matters)

**Definition:**
- `setTimeout(fn, 0)` → runs in **timers phase** (after timer threshold).
- `setImmediate(fn)` → runs in **check phase** (right after poll phase).

**Why it matters:** Order can change depending on whether you're inside an I/O callback.  
**Typical behavior:**
- From **top-level** (main module): order is **not guaranteed**.
- From an **I/O callback**: `setImmediate` usually runs **before** `setTimeout(0)` because the event loop completes poll → check (immediate) → next loop timers.

**Example:**
- Inside `fs.readFile` callback: schedule both → `setImmediate` likely first.

**Tradeoff:**  
- `setImmediate` is great for **“after I/O”** patterns.
- `setTimeout(0)` is great for **“after some delay”** and macrotask deferral.

**Mental model:**  
- `setTimeout(0)` = “next loop when timers are checked”  
- `setImmediate` = “right after I/O poll, before next loop timers”

---

## 4.3 What is `process.nextTick()` and why can it be dangerous?

**Definition:** `process.nextTick(fn)` schedules `fn` to run **after the current call stack** but **before the event loop continues**.  
**Why:** It’s for internal consistency—finish current operation, then run small follow-up tasks immediately.  
**How:** Node has a special **nextTick queue** processed **before** Promise microtasks and before moving to other event loop phases.  
**Danger:** If you schedule `nextTick` recursively, you can **starve** I/O and timers (nothing else gets a turn).

**Example:** A recursive `nextTick` loop can block server from handling requests.  
**Tradeoff:** Powerful for tiny “finish this first” tasks, but easy to abuse.

**Mental model:** `nextTick` is **VIP queue** — if you keep adding VIPs, the normal queue never moves.

---

## 4.4 Microtasks in Node: how do **Promises** relate to microtask processing?

**Definition:** Promise callbacks (`.then/.catch/.finally`, `queueMicrotask`) run as **microtasks**.  
**Why:** Microtasks are used for “run ASAP after current JS finishes” semantics.  
**How (Node detail):**
1. Node empties the **`nextTick` queue**,
2. then empties the **microtask queue** (Promises / `queueMicrotask`),
3. then continues event loop phases.

**Example:** `Promise.resolve().then(...)` runs **before** timers like `setTimeout`.  
**Tradeoff:** Microtasks give deterministic “after this sync work” ordering, but can starve other work if abused.

**Mental model:** Microtasks are **post-it notes** that must be cleared before you leave the desk.

---

## 4.5 Explain why “microtasks run before timers” is a useful mental model (and its limits)

**Useful model:** After the current synchronous work completes, Node runs **microtasks first**, then reaches timers.  
**Why useful:** Helps predict output order in common cases:
- `Promise.then` usually before `setTimeout`.

**Limits / nuance:**
- Node runs microtasks **after each event loop phase/callback**, not only once per loop.
- `process.nextTick` is even “earlier” than Promise microtasks.
- Real ordering can be affected by I/O timing and long CPU tasks.

**Mental model:**  
- “Microtasks first” is like **cleaning your desk before starting the next task** — but it happens repeatedly.

---

## 4.6 How can microtasks cause **starvation**? Give a practical example.

**Definition:** Starvation = other work (timers/I/O) never gets CPU time because microtasks keep refilling.  
**How:** A microtask schedules another microtask repeatedly → Node keeps draining microtasks and never returns to I/O.  
**Practical example:** A `Promise.then(() => Promise.resolve().then(...))` loop or recursive `nextTick`.

**Impact:** Requests hang, timers delay, memory may grow.

**Tradeoff:** Microtasks are great for small chains, risky for unbounded recursion.

**Mental model:** If you keep adding “do-this-now” notes, you never start the next real job.

---

## 4.7 If you need to schedule work “after current work but before I/O”, which API do you pick and why?

**Pick:** `process.nextTick()` **or** `queueMicrotask()` / `Promise.then()` depending on strictness.

**Best interview answer:**
- If you need **strictly before any I/O**, `process.nextTick` runs earliest (but use sparingly).
- If you just need “after current sync work” without starving as easily, prefer **`queueMicrotask`**.

**Tradeoff:**
- `nextTick` can starve; keep work tiny.
- Microtasks can also starve if you loop.

**Mental model:**  
- `nextTick` = “before leaving the room”  
- microtask = “right after I stand up”

---

## 4.8 Scenario: You have a loop that schedules millions of microtasks and memory grows. What’s happening?

**What’s happening:** You’re creating a huge backlog of queued callbacks + captured closures.  
**Why memory grows:** Each queued microtask holds references (args, closures). If you schedule faster than you execute, queue expands.  
**Also:** The event loop may be starved, so nothing “outside” gets time to relieve pressure or complete I/O.

**Fixes:**
- Batch work: process N items, then yield with `setImmediate` or `setTimeout(0)`.
- Avoid creating promises in tight loops; use a queue with controlled concurrency.
- Break recursion; use backpressure-like patterns.

**Mental model:** You’re printing a million tickets faster than the clerk can serve them.

---

## 4.9 How do you implement a **debounce** and a **throttle**? (logic + common pitfalls)

### Debounce (fire after user stops)
**Definition:** Run only after no calls for `wait` ms.  
**Why:** Avoid doing expensive work per keystroke (search, resize).  
**How:** Keep one timer; reset it each call.

```js
function debounce(fn, wait) {
  let t = null;
  return function (...args) {
    const ctx = this;
    clearTimeout(t);
    t = setTimeout(() => fn.apply(ctx, args), wait);
  };
}
```

**Pitfalls:**
- Losing `this` / arguments (use `apply`).
- Not providing `cancel()` when needed.
- For “leading” behavior (first call fires immediately) you need extra logic.

### Throttle (fire at most once per window)
**Definition:** Run at most once every `wait` ms.  
**Why:** Limit rate of expensive ops (scroll handler).  
**How:** Track last time; optionally schedule trailing call.

```js
function throttle(fn, wait) {
  let last = 0;
  let trailingTimer = null;

  return function (...args) {
    const ctx = this;
    const now = Date.now();
    const remaining = wait - (now - last);

    if (remaining <= 0) {
      clearTimeout(trailingTimer);
      trailingTimer = null;
      last = now;
      fn.apply(ctx, args);
    } else if (!trailingTimer) {
      trailingTimer = setTimeout(() => {
        last = Date.now();
        trailingTimer = null;
        fn.apply(ctx, args);
      }, remaining);
    }
  };
}
```

**Pitfalls:**
- Double-calls (leading + trailing) if not careful.
- Clock changes; for precision use `performance.now()`.

**Mental model:**  
- Debounce = “wait for silence”  
- Throttle = “one per time slot”

---

## 4.10 Debugging: An interval timer drifts over time. Why and how do you fix it?

**Why drift happens:**
- `setInterval` schedules “every X ms” but execution is delayed by:
  - event loop load,
  - CPU blocks,
  - GC pauses,
  - I/O callbacks.
So the callback runs late, and the lateness accumulates.

**Fix:** Use a “self-correcting” scheduler: compute next target time and schedule the remainder.

```js
function preciseInterval(fn, intervalMs) {
  let next = Date.now() + intervalMs;

  function tick() {
    const now = Date.now();
    const drift = now - next;        // how late we are
    next += intervalMs;

    fn({ now, drift });

    const delay = Math.max(0, intervalMs - drift);
    setTimeout(tick, delay);
  }

  setTimeout(tick, intervalMs);
}
```

**Tradeoff:** Slightly more complex, but stable timing.

**Mental model:** Don’t schedule based on “when I ran”; schedule based on “when I should have run.”

---

# 5) Express.js & Middleware Design

## 5.1 What is middleware in Express? Explain the request lifecycle in 4–5 steps.

**Definition:** Middleware = a function that runs during request handling and can:
- read/modify `req`, `res`,
- end the response,
- or pass control using `next()`.

**Lifecycle (4–5 steps):**
1. **Request enters** Express.
2. Express runs **app-level middleware** in registration order.
3. Express matches a **route** (method + path), then runs route handlers/middleware.
4. If someone calls `next(err)` or throws, Express jumps to **error middleware**.
5. Response is sent and the request ends.

**Mental model:** A request goes through a **pipeline of functions**.

---

## 5.2 Difference between **app middleware** vs **router middleware**

- **App middleware**: `app.use(...)` applies to the whole app (or mounted path prefix).  
- **Router middleware**: `router.use(...)` applies only inside that router module; great for feature boundaries (`/users`, `/admin`).

**Why:** Router middleware helps **modularity** and keeps concerns close to routes.

---

## 5.3 How does Express decide which middleware runs (ordering, next())?

**Rule:** Express runs middleware in **the order you register them**.  
**Matching:** For `.use('/prefix', fn)`, it runs when the path matches that prefix.  
**`next()` behavior:**
- `next()` → continue to next matching middleware.
- `next(err)` → skip normal middleware and go to error handlers.
- If you **send a response** (`res.json/res.send/res.end`) and don’t call `next()`, the chain stops.

**Mental model:** Express is like a **playlist** — order matters.

---

## 5.4 How do you design a **global error handler** middleware? What signature must it have?

**Signature must be:** `(err, req, res, next)` (4 args).  
Express recognizes error middleware by **4 parameters**.

**Design:**
- Normalize errors into a consistent shape.
- Log with requestId/traceId.
- Hide internal errors in prod (don’t leak stack traces).

```js
function errorHandler(err, req, res, next) {
  const status = err.statusCode || 500;

  const payload = {
    error: {
      code: err.code || "INTERNAL_ERROR",
      message: status === 500 ? "Something went wrong" : err.message,
      requestId: req.id,
      details: err.details || undefined,
    },
  };

  if (status === 500) console.error(err);
  res.status(status).json(payload);
}
```

---

## 5.5 How do you validate incoming requests (body/query/params) and return consistent errors?

**Approach:**
- Use schema validation (Zod/Joi/Yup).
- Validate **body**, **query**, **params** separately.
- On error, return a standard error response with fields and `requestId`.

**Key idea:** validation is **middleware**.

```js
function validate(schema) {
  return (req, res, next) => {
    const result = schema.safeParse({
      body: req.body,
      query: req.query,
      params: req.params,
    });
    if (!result.success) {
      return next({
        statusCode: 400,
        code: "VALIDATION_ERROR",
        message: "Invalid request",
        details: result.error.flatten(),
      });
    }
    req.validated = result.data;
    next();
  };
}
```

**Pitfall:** Don’t “half-validate” and then trust raw `req.body`.

---

## 5.6 How do you structure an Express app for maintainability (routes, controllers, services, repos)?

**A clean layering:**
- **routes**: HTTP shape (paths, verbs, middleware chain)
- **controller/handler**: translate HTTP → business call; map errors → HTTP
- **service**: business rules, orchestration
- **repo/DAO**: DB queries & persistence
- **lib**: shared utils (logger, config, auth, validation)

**Why:** Keeps business logic testable without HTTP.

**Mental model:** HTTP is the **adapter**, service is the **brain**, repo is the **hands**.

---

## 5.7 How do you handle async errors in routes without repeating try/catch?

**Solution:** Wrap async route handlers and forward errors to `next()`.

```js
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

// usage
app.get("/users/:id", asyncHandler(async (req, res) => {
  const user = await getUser(req.params.id);
  res.json(user);
}));
```

**Why:** Express won’t catch promise rejections automatically in older versions.

---

## 5.8 What is **idempotency** for APIs? Where does it matter in Express endpoints?

**Definition:** Making the same request multiple times produces the **same result** (or safe repeat).  
**Why:** Clients retry on timeouts; gateways may retry; you must avoid duplicate side-effects.

**Where it matters:**
- Payments, order creation, “send email,” “charge card” endpoints.
- PUT is typically idempotent; POST is not (unless you add idempotency).

**How:** Use an **Idempotency-Key** header; store the outcome keyed by (user, key).  
**Tradeoff:** Storage + cleanup of idempotency records.

---

## 5.9 Scenario: rate limiting, auth, logging, request IDs — middleware order?

**Typical safe order (general API):**
1. **Request ID** (create/propagate)
2. **Logging** (start timer, log request)
3. **Security headers** / CORS (if needed)
4. **Body parsing** (JSON) / multipart parser (for uploads)
5. **Rate limiting**
6. **Auth**
7. **Validation**
8. **Routes/controllers**
9. **Error handler**
10. **Logging** (response log at end)

**Nuance:** Login routes often rate-limit **before** auth (because auth isn’t available yet).  
**Mental model:** Identify → Observe → Protect → Authenticate → Validate → Execute.

---

## 5.10 Scenario: File upload + auth + validation + streaming response — sketch the chain

**Chain example:**
1. `requestId()`  
2. `logger()`  
3. `auth()`  
4. `uploadParser()` (streaming multipart)  
5. `validateFileMeta()` (type/size constraints)  
6. `streamToStorage()` (pipe to disk/S3 with backpressure)  
7. `respond()` or `streamResponse()`  
8. `errorHandler()`

**Key interview line:** “I don’t buffer the whole file; I stream it and rely on backpressure.”

---

# 6) Auth: JWT, Sessions, Cookies

## 6.1 Authentication vs authorization — define both with examples

**Authentication (AuthN):** “Who are you?”  
- Example: login with password/OTP; verify identity.

**Authorization (AuthZ):** “What can you do?”  
- Example: user is authenticated, but only admin can delete users.

**Mental model:**  
AuthN = **ID check**; AuthZ = **permissions**

---

## 6.2 JWT basics: what’s inside a JWT? Why is it “stateless”?

**JWT structure:** `header.payload.signature`  
- **Header:** algorithm + type  
- **Payload:** claims (sub, exp, roles, etc.)  
- **Signature:** integrity (server signs, clients can’t tamper)

**Stateless:** Server doesn’t need a session store to validate; it verifies signature + claims.  
**Tradeoff:** Revocation is harder (see below).

---

## 6.3 Where should JWT be stored: localStorage vs cookie? Tradeoffs and security implications

**localStorage:**
- ✅ easy to use in SPA
- ❌ vulnerable to **XSS** (script can read tokens)
- ✅ not automatically sent with requests (less CSRF risk)
- but you manually add `Authorization` header.

**HttpOnly cookie:**
- ✅ protected from XSS reads (JS can’t access)
- ✅ automatically sent by browser
- ❌ needs **CSRF** protection (SameSite + CSRF tokens)
- must use correct cookie settings.

**Best-practice interview answer:**  
“For browser apps, I prefer **HttpOnly Secure cookies** + **SameSite** and CSRF protection. LocalStorage is common but riskier under XSS.”

---

## 6.4 Refresh token vs access token — why split them?

**Access token:**
- short-lived (minutes)
- used on every request

**Refresh token:**
- long-lived (days/weeks)
- used only to mint new access tokens

**Why split:**  
If an access token leaks, damage is limited by short expiry. Refresh token is more protected (HttpOnly cookie, rotation).

**Tradeoff:** More complexity, token rotation, storage.

---

## 6.5 Sessions: how do server-side sessions work compared to JWT?

**Sessions:**
- Server stores session data (in memory/Redis/DB).
- Client stores a **session id** cookie.
- Each request → server looks up session.

**Pros:** easy revocation, server control.  
**Cons:** need session store + scaling strategy.

**JWT pros:** no session store required.  
**JWT cons:** revocation and “logout everywhere” needs extra design.

---

## 6.6 Cookies: `HttpOnly`, `Secure`, `SameSite` — what does each do?

- **HttpOnly:** JS can’t read cookie → reduces XSS token theft.  
- **Secure:** cookie sent only over HTTPS.  
- **SameSite:**
  - `Strict` → not sent on cross-site requests (best CSRF protection, may break some flows)
  - `Lax` → sent on top-level navigation GETs (good default)
  - `None` → sent cross-site, requires `Secure` (needed for some embedded flows)

---

## 6.7 What is token revocation and why is it hard with pure JWT?

**Revocation:** making a previously issued token invalid before it expires.  
**Hard with pure JWT:** JWT is self-contained; if server only checks signature+exp, it can’t “unsign” it.  
**Solutions:**
- Short expiry + refresh tokens
- Maintain a **denylist** (jti) in Redis (tradeoff: stateful)
- Use “session-like” token store for refresh tokens

---

## 6.8 How do you implement logout correctly for JWT-based auth?

**Good logout design:**
- Delete/expire access token cookie (or just let it expire if not stored)
- Revoke refresh token in server store (or mark as invalid)
- If you do rotation, invalidate the current refresh token id

**Interview one-liner:**  
“Logout must revoke refresh tokens; otherwise user can mint new access tokens.”

---

## 6.9 Scenario: Multi-device login and “logout from all devices” — design it

**Design:**
- Store refresh tokens per device/session in DB/Redis:
  - fields: userId, tokenId (jti), device info, createdAt, revokedAt
- Each device has its own refresh token.
- “Logout all” = revoke all token records for that user.
- Access tokens remain valid until expiry (keep them short).

**Tradeoff:** state required (but only for refresh tokens).

---

## 6.10 Scenario: “remember me” + short access tokens — propose a safe setup

**Setup:**
- Access token: 5–15 minutes
- Refresh token:  
  - normal login: 1–7 days  
  - “remember me”: 30 days  
- Refresh token stored in **HttpOnly Secure cookie**, rotate on use.
- Keep refresh tokens in DB with revocation + device tracking.
- Add anomaly detection: IP changes, impossible travel, etc.

---

# 7) Security: CORS, CSRF, XSS, Injection

## 7.1 What is CORS? Explain preflight and why it exists.

**CORS:** browser security policy controlling cross-origin requests.  
**Why:** Prevent a random website from reading responses from another origin.  
**Preflight:** For “non-simple” requests, browser sends `OPTIONS` with:
- `Access-Control-Request-Method`
- `Access-Control-Request-Headers`
Server replies with `Access-Control-Allow-*` headers.

**Mental model:** Browser asks, “Are you okay if I send this kind of request?”

---

## 7.2 Difference between CORS and CSRF (people confuse these)

**CORS:** controls whether the **browser allows the response to be read** by JS from another origin.  
**CSRF:** attacker tricks user’s browser into **sending an authenticated request** (cookies auto-sent), causing actions without consent.

**Key line:**  
“CORS is about **reading** cross-site responses; CSRF is about **sending** unwanted authenticated actions.”

---

## 7.3 What is CSRF and how do you protect against it (SameSite, CSRF tokens)?

**CSRF:** malicious site causes victim browser to send a request to your site using stored cookies.  
**Protection:**
- `SameSite=Lax/Strict` for auth cookies
- CSRF tokens (double-submit cookie or server-issued token)
- Check `Origin` / `Referer` headers (defense-in-depth)
- Use `POST` + CSRF token for state-changing actions

**Tradeoff:** More complexity, especially with cross-site embedding.

---

## 7.4 What is XSS? How do you prevent it in React/templating and in APIs?

**XSS:** attacker injects JS into a page that runs in user’s browser.  
**Prevention in React:** React escapes by default; avoid `dangerouslySetInnerHTML`.  
**General prevention:**
- Escape/encode untrusted output (HTML, attributes, URLs)
- Use CSP (Content-Security-Policy)
- Sanitize rich text with a trusted sanitizer
- HttpOnly cookies reduce token theft impact

**API role:** Validate/clean inputs and store safely, but XSS is mainly **render-time**.

---

## 7.5 Injection basics: SQL injection vs NoSQL injection — what’s the root cause?

**Root cause:** treating **untrusted input as code** (query operators, SQL strings).  
- SQL injection: string concatenation into SQL
- NoSQL injection: allowing user-controlled operators like `$gt`, `$ne` in Mongo queries

**Fix pattern:** strict validation + parameterization + allowlists.

---

## 7.6 How do parameterized queries prevent SQL injection?

**Mechanism:** query and data are sent separately; DB treats parameters as **values**, not SQL code.  
**Example:** `SELECT * FROM users WHERE email = ?` with bound parameter.  
**Tradeoff:** Still validate inputs (types/length) and use least privilege DB accounts.

---

## 7.7 What is SSRF? Example of how an API can accidentally allow it.

**SSRF:** Server-Side Request Forgery — attacker makes your server call internal/private URLs.  
**Example:** `/fetch?url=http://169.254.169.254/...` (cloud metadata), or internal admin services.  
**Protection:**
- Allowlist domains
- Block private IP ranges + metadata endpoints
- Use URL parsing + DNS resolution checks
- Limit redirects
- Timeout + size limits

---

## 7.8 Rate limiting and brute force protection: what do you rate-limit and at what layer?

**Rate-limit:**
- Login / OTP / password reset endpoints (strict)
- Expensive endpoints
- Per IP, per user, per route
- Add captcha after threshold

**Layers:**
- Edge/WAF/CDN (best for volumetric)
- API gateway
- App-level middleware (fine-grained user-based)

---

## 7.9 Secrets management: where should API keys live (and never live)?

**Never:**
- frontend bundles
- git repo
- logs

**Should live in:**
- environment variables (with secure injection)
- secret managers (AWS Secrets Manager, Vault)
- CI/CD secret stores

**Also:** rotate keys; least privilege; audit usage.

---

## 7.10 Scenario: Public API sees spikes in 401/403 — what steps do you add immediately?

**Immediate actions:**
- Add/raise rate limits on auth endpoints and sensitive routes
- Enable WAF rules / IP reputation blocks
- Add account lockout / progressive delays for brute force
- Improve logging: requestId, IP, user-agent, route, auth failure reason
- Alerting: error rate thresholds and unusual patterns
- Consider captcha on suspicious flows
- Review token validation and clock skew issues

---

# 8) Performance & Scalability

## 8.1 Throughput vs latency — difference and why both matter?

- **Latency:** time per request (p50/p95/p99).  
- **Throughput:** requests per second (RPS).  
**Why both:** You can have high throughput with terrible tail latency, or low throughput with fast responses—users feel latency, business needs throughput.

**Mental model:** Latency = “how fast one car finishes”; throughput = “cars per minute.”

---

## 8.2 How do you find if your Node app is CPU-bound or I/O-bound?

**CPU-bound signals:**
- high CPU usage
- event loop lag increases
- p95 latency rises with CPU
- profiling shows hot JS functions

**I/O-bound signals:**
- low CPU but high latency
- traces show waiting on DB/HTTP
- many concurrent sockets/queries

**Tools:** APM tracing, `clinic`, CPU profiles, DB query logs, event loop lag metric.

---

## 8.3 Explain Node **cluster**: what problem does it solve and what does it not solve?

**Solves:** Uses multiple processes to utilize **multi-core** CPUs; better parallelism for many requests.  
**Does NOT solve:**
- shared memory issues (workers don’t share heap)
- CPU-heavy single request still blocks its worker
- requires sticky sessions or shared session store for websockets/sessions

**Mental model:** Cluster = “more cashiers,” not “faster cashier.”

---

## 8.4 Worker threads: when do you use `worker_threads` vs cluster?

- **worker_threads:** CPU-heavy tasks (image processing, compression, crypto) without blocking event loop. Shares memory via `SharedArrayBuffer` if needed.
- **cluster:** scaling request handling across cores (process-level isolation).

**Rule of thumb:**  
Cluster for **more traffic**, workers for **heavy computation**.

---

## 8.5 Caching: Redis vs in-memory cache — tradeoffs

**In-memory (per process):**
- ✅ fastest
- ❌ not shared across instances
- ❌ lost on restart
- ❌ cache inconsistency across cluster

**Redis:**
- ✅ shared across instances
- ✅ TTL, eviction, atomic ops, pub/sub
- ❌ network hop latency
- ❌ needs infra and monitoring

**Mental model:** In-memory is a sticky note on one server; Redis is a shared whiteboard.

---

## 8.6 DB connection pooling: why it matters and what happens if you over-pool

**Why pooling:** Creating DB connections is expensive; pooling reuses them.  
**Over-pooling:** Too many connections:
- overwhelms DB (CPU/memory)
- increases context switching
- causes lock contention
- leads to timeouts and cascading failures

**Best practice:** Pool size tuned to DB capacity + app concurrency.

---

## 8.7 What is the “N+1 query” problem and how do you detect/fix it?

**Definition:** One query to fetch list (N items) plus N queries to fetch details per item.  
**Symptoms:** DB queries scale linearly with page size; tracing shows many repeated queries.  
**Fixes:**
- join / batch queries (`WHERE id IN (...)`)
- dataloader pattern
- eager loading
- caching

---

## 8.8 Backpressure at system level: protect downstream systems (queue, rate limit, circuit breaker)

**Goal:** Don’t overload DB/external services.  
**Tools:**
- **Queue** for async work (jobs)
- **Rate limiting** to cap incoming load
- **Circuit breaker** to fail fast when downstream is unhealthy
- **Bulkhead** (separate pools for different calls)
- **Timeouts** + retries with jitter (careful!)

**Mental model:** Protect the weakest link.

---

## 8.9 Scenario: Handle 10x traffic — top levers (app, DB, cache, infra)

1. **Cache** hot reads (Redis, CDN)  
2. **Optimize DB**: indexes, query fixes, reduce N+1  
3. **Add pagination** + limit payload sizes  
4. **Async offload**: queues for emails, reports, video processing  
5. **Horizontal scale** stateless services + autoscaling  
6. **Rate limiting** + load shedding  
7. **Reduce latency**: keep-alive, compression, avoid chatty calls  
(Plus: read replicas, sharding, denormalization if needed)

---

## 8.10 Scenario: One endpoint is slow — profile and optimize end-to-end

**Steps:**
1. Reproduce with real inputs; confirm p95/p99 and request sizes.
2. Add tracing: break down time spent in app vs DB vs external.
3. Check DB query plan + indexes; fix N+1.
4. Reduce payload: select only needed fields, compress responses.
5. Add caching for stable data.
6. Measure event loop lag and CPU; offload heavy tasks.
7. Validate improvements with load test and monitor tail latency.

---

# 9) Observability: Logs, Metrics, Tracing

## 9.1 Logs vs metrics vs traces — define each and when you use them

- **Logs:** discrete events (“user created”), best for debugging specifics.  
- **Metrics:** numbers over time (RPS, p95), best for alerts and trends.  
- **Traces:** end-to-end request timeline across services, best for latency root-cause.

**Mental model:**  
Logs = “what happened”, metrics = “how often/how bad”, traces = “where time went”.

---

## 9.2 What should be in a good structured log (fields)?

**Common fields:**
- timestamp, level, message
- requestId, traceId, spanId
- route, method, statusCode, durationMs
- userId/tenantId (if safe)
- error name/message/stack (for errors)
- remote IP, user-agent (be privacy-aware)

**Why structured:** searchable, filterable, aggregatable.

---

## 9.3 What is a correlation/request ID and how do you propagate it?

**Definition:** A unique id per request to tie logs across layers.  
**How:**
- If `X-Request-Id` incoming exists, reuse it; else generate.
- Attach to `req.id`
- Include it in all logs and response headers.
- Forward to downstream calls in headers.

---

## 9.4 What metrics would you track for an API (top 10)?

1. Request rate (RPS)  
2. Error rate (4xx/5xx)  
3. Latency (p50/p95/p99)  
4. Saturation (CPU, memory, event loop lag)  
5. DB latency + DB error rate  
6. External call latency + error rate  
7. Cache hit ratio (if caching)  
8. Queue depth/lag (if using queues)  
9. GC pauses / heap usage trend  
10. Instance restarts / uptime

---

## 9.5 What is RED method (Rate, Errors, Duration) and where does it help?

**RED:**
- **Rate:** how many requests
- **Errors:** how many failed
- **Duration:** how long they take

**Use:** perfect for **API endpoints** to build dashboards and alerts quickly.

---

## 9.6 How do you detect memory leaks with observability signals?

**Signals:**
- heapUsed steadily increases without returning
- RSS grows even after GC
- GC frequency increases, pauses increase
- latency increases under constant traffic

**How:** heap snapshots, compare allocations, check retaining paths; look for caches without eviction, global arrays, event listeners.

---

## 9.7 What is distributed tracing? How does a trace span help debug latency?

**Distributed tracing:** attaches a traceId to a request across services.  
**Span:** a timed operation (DB query, HTTP call).  
**Benefit:** See exactly which span dominates p95 (DB vs upstream vs internal compute).

---

## 9.8 Sampling: why sample logs/traces and what are the risks?

**Why sample:** cost and volume control; high-traffic systems can’t store everything.  
**Risks:** missing rare bugs; bias (only “normal” requests).  
**Mitigation:** keep unsampled metrics; dynamic sampling; always sample errors.

---

## 9.9 Scenario: 5xx spikes only in one region — what do you check first?

**First checks:**
- Recent deployments/config differences in that region
- Upstream dependencies from that region (DB, cache, third-party)
- Load balancer / network issues, DNS, TLS certs
- Capacity: autoscaling lag, instance health
- Logs/traces: is it timeouts, connection resets, auth errors?

---

## 9.10 Scenario: Latency high but CPU low — how do logs/metrics/traces help?

**Likely:** I/O waits (DB slow, external API slow, network).  
**Use:**
- metrics: DB latency, external latency, connection pool saturation
- traces: identify which span consumes time
- logs: correlate timeouts with requestId and see patterns (payload size, user, region)

---

# 10) Production Readiness

## 10.1 What is “graceful shutdown” and why do you need it in Node?

**Definition:** Stop accepting new requests, finish in-flight work, close resources cleanly.  
**Why:** Prevent dropped requests, partial writes, corrupted jobs, leaked connections during deploys or scaling events.

---

## 10.2 How do you handle process signals like `SIGTERM` / `SIGINT`?

**Pattern:**
- Listen for signal.
- Stop server `server.close()` (stop new connections).
- Close DB pool, flush logs.
- Force-exit after timeout.

```js
function setupGracefulShutdown({ server, closeDb, timeoutMs = 30_000 }) {
  let shuttingDown = false;

  async function shutdown(signal) {
    if (shuttingDown) return;
    shuttingDown = true;
    console.log(`Received ${signal}, shutting down...`);

    // stop taking new connections
    server.close(async (err) => {
      if (err) console.error("server close error", err);
      try { await closeDb?.(); } catch (e) { console.error("db close error", e); }
      process.exit(0);
    });

    // force exit
    setTimeout(() => {
      console.error("Forced shutdown");
      process.exit(1);
    }, timeoutMs).unref();
  }

  process.on("SIGTERM", () => shutdown("SIGTERM"));
  process.on("SIGINT", () => shutdown("SIGINT"));
}
```

---

## 10.3 What happens if you don’t close DB connections on shutdown?

- DB sees leaked connections until timeout
- pool may keep process alive longer than expected
- may cause resource exhaustion in DB during rolling deploys
- in-flight transactions can be left incomplete

---

## 10.4 Environment config: manage config across dev/stage/prod safely?

**Best practice:**
- configuration via env vars (12-factor)
- validate config on startup (fail fast)
- secret manager for secrets
- separate configs per environment; no hardcoding in code
- never log secrets

---

## 10.5 Difference between `NODE_ENV=production` and others practically?

- Enables performance optimizations in many libs
- Express disables stack traces and some dev behaviors
- React builds: production build minified, no dev warnings
- Logging levels and error messages often differ

**Interview note:** “I treat production as a different mode with different defaults.”

---

## 10.6 Health check endpoint: readiness vs liveness

- **Liveness:** “process is alive” (should it be restarted?)  
- **Readiness:** “can it serve traffic?” (DB connected, migrations done, warm caches)

**Example:** During deploy, readiness may be false until app is warmed up.

---

## 10.7 Rolling deployment: what is it and how do you prevent downtime?

**Rolling:** replace instances gradually while keeping some running.  
**Avoid downtime:**
- readiness checks
- graceful shutdown
- backward-compatible changes (DB migrations)
- avoid breaking API contracts
- keep connection draining time

---

## 10.8 Blue/green deployment: when choose it?

**Blue/green:** keep old (blue) and new (green) environments; switch traffic at once.  
**Choose when:**
- you need very fast rollback
- you want to test green with real traffic gradually (canary variant)
- risky migrations or major releases

**Tradeoff:** cost (double infra) and complexity.

---

## 10.9 Handle secrets and rotation in production

- Store in secret manager
- Rotate regularly; support multiple active keys (key versioning)
- Reload secrets without downtime if possible
- Audit access; least privilege

---

## 10.10 Scenario: Production incident — immediate steps (mitigation + rollback + postmortem)

**Immediate:**
1. **Stop the bleeding:** rate limit, disable feature flag, scale up, block abusive traffic.
2. **Mitigate:** rollback deploy, revert config, reroute traffic.
3. **Communicate:** status updates, incident channel, ETA if known.
4. **Investigate with data:** metrics → traces → logs, find blast radius.
5. **Stabilize:** add temporary safeguards (timeouts, circuit breaker).
6. **Postmortem:** root cause, contributing factors, action items, ownership, timelines.

---

# Optional: Quick “Coding Prompt” Add‑ons

## A) Express `asyncHandler(fn)`
```js
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);
```

## B) Stream a large file download (concept)
- set headers (`Content-Type`, `Content-Length` if known)
- `pipeline(readStream, res)` and handle errors

## C) Retry with exponential backoff + jitter (concept)
- retry only safe/idempotent operations
- baseDelay * 2^attempt + randomJitter
- cap max delay + max attempts

## D) Simple rate limiter middleware (high level)
- token bucket per IP/user stored in Redis
- decrement tokens; refill over time
- return 429 with `Retry-After`

---

## Final 10-second “senior” summary (say this if time is short)

“I separate work into **sync**, **microtasks**, and **macrotasks/timers**, and avoid starvation by yielding with `setImmediate`. In Express, middleware order is everything: requestId/logging early, protection/auth before business logic, and a global `(err, req, res, next)` handler. For auth, short-lived access tokens + refresh rotation in HttpOnly cookies. For security, I think in threats: XSS, CSRF, injection, SSRF, brute force. For performance, I measure p95/p99, avoid N+1, and protect downstreams with backpressure patterns. In prod, graceful shutdown + readiness checks + safe deploys.”

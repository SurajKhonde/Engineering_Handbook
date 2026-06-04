# 200 Deep Interview Questions — Suraj's Full Stack

Covers every skill, tool, and project on your resume. Each answer is the **core point + the tradeoff/nuance** an interviewer is probing for — enough to drill against and expand in your own words. Flagged 🔥 = high-probability, know cold.

> Stack covered: JS/TS · Node · Express/REST · WebSockets/Socket.IO · React · Next.js · Redux/RTK Query · BullMQ/RabbitMQ · Redis · Postgres/Drizzle · MongoDB/MySQL · Auth/JWT/Security/Encryption · Claude/LLM/RAG · AWS · Docker/Nginx/PM2/CI-CD · Puppeteer/Razorpay/Firebase/Pino · Architecture & your projects.

---

## A. JavaScript & TypeScript (1–26)

**1. 🔥 Explain the event loop.** JS is single-threaded; the event loop offloads async work (timers, I/O) and processes their callbacks from queues once the call stack is empty. Microtasks (promises) drain fully before the next macrotask (timers, I/O callbacks).

**2. Microtask vs macrotask queue?** Microtasks (`Promise.then`, `queueMicrotask`) run after the current task and before rendering/next macrotask; macrotasks (`setTimeout`, `setInterval`, I/O) run one per loop iteration. A promise chain starves timers if it never yields.

**3. `setTimeout(fn, 0)` vs `Promise.resolve().then(fn)` — which runs first?** The promise — microtasks drain before the next macrotask, even a 0ms timer.

**4. What is a closure?** A function that retains access to its lexical scope after the outer function returns. Used for private state, partial application, and memoization. Common bug: closing over a loop variable with `var` (use `let`/`const`).

**5. `var` vs `let` vs `const`?** `var` is function-scoped and hoisted (initialized `undefined`); `let`/`const` are block-scoped with a temporal dead zone; `const` can't be reassigned (but objects are still mutable).

**6. `==` vs `===`?** `==` does type coercion (`0 == "0"` is true); `===` checks type and value. Always use `===`.

**7. How does `this` work?** Determined by call-site: method call → the object; plain call → `undefined`/global (strict); arrow functions inherit `this` from enclosing scope; `bind`/`call`/`apply` set it explicitly.

**8. Arrow function vs regular function?** Arrows have no own `this`, `arguments`, or `prototype`, can't be constructors, and inherit `this` lexically — ideal for callbacks, wrong for object methods needing dynamic `this`.

**9. Prototypal inheritance?** Objects delegate to a prototype chain; property lookups walk up `__proto__` until found or null. `class` is syntactic sugar over this.

**10. Promise vs async/await?** Same mechanism; `async/await` is syntactic sugar making async code read sequentially. `await` pauses the function (not the thread) until the promise settles.

**11. `Promise.all` vs `allSettled` vs `race` vs `any`?** `all` rejects if any rejects; `allSettled` waits for all and reports each outcome; `race` settles on the first to settle; `any` resolves on first fulfilled. Use `allSettled` when partial failure is acceptable.

**12. How do you handle errors in async/await?** `try/catch` around awaits; for parallel work, catch per-promise or use `allSettled`. Unhandled rejections crash Node if not caught.

**13. Debounce vs throttle?** Debounce fires after activity stops (search input); throttle fires at most once per interval (scroll/resize). Both limit handler frequency.

**14. Shallow vs deep copy?** Shallow (`{...obj}`, `Object.assign`) copies top level; nested objects are shared references. Deep copy needs `structuredClone` or recursion. Mutating a "copied" nested object is a classic bug.

**15. `map` vs `forEach`?** `map` returns a new array (use for transforms); `forEach` returns nothing (use for side effects). `map` for nothing-returned wastes an array.

**16. What is hoisting?** Declarations are processed before execution — function declarations are fully hoisted, `var` is hoisted as `undefined`, `let`/`const` are hoisted but unusable until declared (TDZ).

**17. 🔥 TypeScript: `interface` vs `type`?** Both describe shapes; `interface` is extendable/mergeable and better for object/class contracts; `type` does unions, intersections, primitives, and mapped types. Use `interface` for public APIs, `type` for unions/utilities.

**18. `unknown` vs `any`?** `any` disables type checking (escape hatch, avoid); `unknown` is type-safe — you must narrow it before use. Prefer `unknown` at boundaries (e.g. parsed JSON).

**19. Generics — why?** Reusable code that preserves types: `function first<T>(arr: T[]): T`. Without them you'd lose type info or duplicate code per type.

**20. What are utility types (`Partial`, `Pick`, `Omit`, `Record`)?** Built-in type transforms: `Partial<T>` makes all optional, `Pick`/`Omit` select/drop keys, `Record<K,V>` builds a map type. They keep types DRY.

**21. `enum` vs union of string literals?** String-literal unions (`'a' | 'b'`) are lighter, fully erased at runtime, and tree-shakeable; `enum` emits runtime code. Many TS teams prefer literal unions.

**22. What is structural typing?** TS types are compatible by shape, not name — if an object has the required properties, it fits, regardless of declared type. ("Duck typing" enforced at compile time.)

**23. `never` type?** Represents values that never occur — functions that always throw, or exhaustiveness checks in switch statements (assign to `never` in the default to catch unhandled cases).

**24. How does type narrowing work?** TS narrows via `typeof`, `instanceof`, `in`, truthiness, and discriminated unions, so a variable's type tightens inside a branch.

**25. What is a discriminated union?** A union of object types sharing a literal "tag" field (`{type:'a'} | {type:'b'}`); switching on the tag narrows the type — great for state machines and API responses.

**26. Does TypeScript exist at runtime?** No — types are erased on compile. So you can't `instanceof` an interface; runtime validation needs a tool like Zod or manual checks.

---

## B. Node.js (27–44)

**27. 🔥 Is Node single-threaded?** The JS execution is single-threaded, but Node uses libuv's thread pool for some I/O (fs, crypto, dns) and the OS for network I/O. CPU-bound work blocks the loop; offload it.

**28. How do you handle CPU-heavy work in Node?** Worker threads, a child process, or a separate service/queue — never run it on the main loop or you block all requests. (Your render work goes to a separate worker for exactly this reason.)

**29. Cluster vs worker threads?** Cluster forks multiple processes (separate memory) to use all cores for I/O-bound HTTP servers; worker threads share memory and suit CPU-bound tasks. PM2 cluster mode uses the former.

**30. What are streams and why use them?** Process data in chunks instead of loading it all into memory — readable, writable, duplex, transform. Essential for large files/uploads; `pipe` connects them with backpressure.

**31. What is backpressure?** When a writable consumes slower than a readable produces; streams pause the source until the destination drains, preventing memory blowup. Ignoring it causes OOM.

**32. How does `require` differ from `import`?** `require` (CommonJS) is synchronous and dynamic; `import` (ESM) is static, hoisted, and supports tree-shaking. Node supports both; mixing them has interop caveats.

**33. `process.nextTick` vs `setImmediate`?** `nextTick` callbacks run before the event loop continues (even before promises in some cases); `setImmediate` runs in the check phase after I/O. Overusing `nextTick` starves the loop.

**34. How do you manage env config/secrets?** Env vars (`process.env`), loaded via `.env` in dev (dotenv) and injected at runtime in prod; never commit secrets or bake them into Docker images.

**35. How do you handle uncaught exceptions / unhandled rejections?** Log them, then let the process restart (PM2/Docker) — the process is in an unknown state. Don't swallow them to "keep running."

**36. What is the difference between `exports` and `module.exports`?** `module.exports` is what's actually returned; `exports` is a reference to it. Reassigning `exports = ...` breaks the link — assign properties or set `module.exports`.

**37. How do you debug a memory leak in Node?** Watch RSS/heap over time, take heap snapshots (`--inspect`, Chrome DevTools), diff them to find growing retainers — common causes: unbounded caches, event-listener leaks, closures holding references.

**38. What is the libuv thread pool default size, and why care?** Default 4; CPU/crypto/fs work shares it, so heavy crypto can serialize. Tune `UV_THREADPOOL_SIZE` if you bottleneck there.

**39. How do you do graceful shutdown in Node?** Listen for SIGTERM, stop accepting new connections/jobs, finish in-flight work, close DB/Redis, then exit — with a force-kill timeout. (Exactly your 10s SIGTERM pattern.)

**40. Middleware pattern in Node?** Functions in a chain each receiving `(req, res, next)`, calling `next()` to pass control — used for auth, validation, logging. You extracted these into a reusable layer at Avisirah.

**41. How do you handle file uploads efficiently?** Stream to storage rather than buffering in memory; better yet, presigned S3 URLs so the binary skips the server entirely. (Your ~40% CPU win.)

**42. What is `Buffer`?** A fixed-size raw-binary container for bytes (files, network data, crypto) outside the V8 heap. Used wherever you handle non-string binary data.

**43. How do you schedule recurring jobs in Node?** Cron libraries or a queue with repeatable jobs (BullMQ repeat). For distributed safety, ensure only one instance runs each job (a lock or the queue handles it).

**44. How does Node handle HTTP keep-alive / connection reuse?** Persistent TCP connections avoid re-handshaking per request; agents pool sockets. Matters for performance under load and for outbound API calls.

---

## C. Express, REST & API Design (45–56)

**45. 🔥 What makes an API RESTful?** Resource-oriented URLs, correct HTTP verbs (GET/POST/PUT/PATCH/DELETE), statelessness, proper status codes, and representations (usually JSON). Verbs in URLs (`/getUser`) is the anti-pattern.

**46. PUT vs PATCH?** PUT replaces the whole resource (idempotent); PATCH applies a partial update. Sending a partial body to PUT can wipe omitted fields.

**47. Which status codes do you actually use?** 200/201/204 success, 301/302 redirects, 400 bad request, 401 unauthenticated, 403 forbidden, 404 not found, 409 conflict, 422 validation, 429 rate limit, 500 server error.

**48. 401 vs 403?** 401 = not authenticated (who are you?); 403 = authenticated but not allowed (you can't do this). Mixing them leaks info or confuses clients.

**49. How do you make an endpoint idempotent?** Same request → same effect: use idempotency keys, upserts, or state checks so retries/duplicate webhooks don't double-process. (Your Razorpay webhook handling.)

**50. How do you version an API?** URL (`/v1/`), header, or content negotiation. URL versioning is simplest and most common; version when you make breaking changes.

**51. How do you validate request input?** Schema validation at the boundary (Zod/Joi) before business logic; reject with 400/422 and clear errors. Never trust the client.

**52. How do you handle errors consistently across an API?** Centralized error-handling middleware that maps errors to status codes and a uniform JSON shape — you did this at Avisirah. Avoid leaking stack traces in prod.

**53. Pagination strategies?** Offset/limit (simple, slow on deep pages) vs cursor/keyset (scales, stable under inserts). Cursor pagination is better for large/feed data.

**54. How do you document APIs?** OpenAPI/Swagger spec — machine-readable, generates docs and clients, keeps contract and reality in sync. (On your stack.)

**55. CORS — what is it and how do you handle it?** Browser security blocking cross-origin requests unless the server sends allow headers; configure allowed origins/methods/credentials. The cookie-cross-domain auth bugs you hit live here.

**56. How do you secure a REST API?** HTTPS, auth (JWT/session), authorization (RBAC), input validation, rate limiting, security headers, and least-privilege. Defense in depth, not one control.

---

## D. WebSockets & Socket.IO (57–66)

**57. 🔥 WebSocket vs HTTP polling?** WebSocket is a persistent full-duplex connection — server pushes without the client asking; polling repeatedly asks (wasteful, laggy). You moved the trading feed off 1s polling to push.

**58. How does Socket.IO scale across multiple servers?** A Redis adapter (pub/sub) broadcasts events across instances so a client on server A receives messages published on server B. Without it, multi-instance breaks. (Your Avisirah fan-out.)

**59. Why did the trading feed need pub/sub fan-out?** Filtered Binance ticks land in Redis and fan out only to clients subscribed to those symbols — each session gets only what it asked for, not everything.

**60. Rooms/namespaces in Socket.IO?** Rooms group sockets for targeted broadcasts (e.g. per-coin or per-chat); namespaces separate concerns over one connection. You broadcast per subscribed symbol.

**61. How do you handle reconnection and missed messages?** Auto-reconnect plus a resync strategy (fetch latest snapshot on reconnect) since events during disconnect are lost. Your "market-live" flag signals freshness.

**62. WebSocket vs Server-Sent Events?** SSE is one-way (server→client), simpler, auto-reconnects, over HTTP; WebSocket is bidirectional. For a push-only feed, SSE can suffice; you need bidirectional for chat.

**63. How do you authenticate a WebSocket connection?** Pass a token on the handshake (query/header), verify it before accepting; re-check on sensitive events. Cookies work if same-origin.

**64. How do you prevent one slow client from affecting others?** Per-connection buffering limits, drop or coalesce messages for slow consumers, and avoid synchronous broadcast work. Batching helped your consumer CPU.

**65. How do you handle 1000+ concurrent connections?** Lean per-connection state, event-driven I/O, horizontal scale with a Redis backplane, and offload heavy work. (Your banking escalation handled 1000+.)

**66. What transport does Socket.IO use under the hood?** WebSocket when available, with a polling fallback; it adds reconnection, rooms, and acks on top of raw WS.

---

## E. React (67–84)

**67. 🔥 How does React reconciliation / the virtual DOM work?** React diffs a virtual tree against the previous one and applies the minimal real-DOM updates. Keys help it match list items; bad keys cause re-mounts and bugs.

**68. Why are keys important in lists?** They give React stable identity for items so it reuses/reorders instead of recreating. Using array index as key breaks on reorder/insert.

**69. `useEffect` — when does it run and what are the deps?** After render; the dependency array controls re-runs (empty = once, omitted = every render). Missing deps cause stale closures; cleanup runs before re-run/unmount.

**70. `useMemo` vs `useCallback`?** `useMemo` memoizes a computed value; `useCallback` memoizes a function reference. Use to avoid expensive recomputes or to keep stable props for memoized children — not everywhere (premature).

**71. What causes unnecessary re-renders and how do you fix them?** New object/function references, context changes, parent re-renders. Fixes: `React.memo`, stable refs (`useCallback`/`useMemo`), splitting state/context. Measure first.

**72. Controlled vs uncontrolled components?** Controlled = value driven by React state (single source of truth); uncontrolled = DOM holds value (refs). Controlled is the default for forms.

**73. `useState` vs `useReducer`?** `useState` for simple independent values; `useReducer` for complex/related state transitions or when next state depends on previous — clearer for state machines.

**74. What is the rules-of-hooks constraint?** Call hooks at the top level, in the same order every render — never in conditions/loops — so React can track them by call order.

**75. How does Context work and what's its downside?** Provides a value to a subtree without prop drilling; downside: any context change re-renders all consumers, so don't put rapidly-changing or large state in one context.

**76. `useRef` use cases?** Persisting a mutable value across renders without re-rendering, and accessing DOM nodes. Mutating a ref doesn't trigger a render (unlike state).

**77. What is lifting state up?** Moving shared state to the closest common parent so multiple children stay in sync via props. The alternative to redundant local copies.

**78. How do you fetch data in React?** `useEffect` + state for simple cases, or a data library (RTK Query/React Query) for caching, dedup, and loading/error states. RTK Query is on your stack.

**79. What is a stale closure bug?** A callback captures an old state/prop value because of missing effect deps or a non-updated closure — fix with correct deps or functional state updates.

**80. Error boundaries?** Class components (or libraries) that catch render errors in their subtree and show a fallback instead of crashing the whole app. Hooks can't be error boundaries directly.

**81. How do you optimize a large list?** Virtualization (react-window) renders only visible rows; plus stable keys and memoized rows. Rendering 10k DOM nodes kills performance.

**82. What are React's render phases?** Render (pure, builds the tree, can be interrupted in concurrent mode) and commit (applies to DOM, runs effects). Side effects belong in commit/effects, not render.

**83. Why must render be pure?** React may call it multiple times/discard results (especially concurrent mode); side effects in render cause bugs. Put effects in `useEffect`.

**84. How do you share logic between components?** Custom hooks (extract stateful logic into reusable `useX` functions) — the modern replacement for HOCs/render props.

---

## F. Next.js (85–96)

**85. 🔥 SSR vs SSG vs ISR vs CSR?** SSR renders per request (fresh, slower); SSG renders at build (fast, static); ISR regenerates static pages on an interval (best of both); CSR renders in the browser. Choose by freshness vs speed needs.

**86. App Router vs Pages Router?** App Router (Next 13+) uses React Server Components, nested layouts, and file-based routing with `app/`; Pages Router is the older `pages/` model. App Router enables server components and streaming.

**87. Server Components vs Client Components?** Server components render on the server (no JS shipped, can access DB/secrets directly); client components (`"use client"`) run in the browser for interactivity. Default to server, opt into client.

**88. How does data fetching work in the App Router?** `fetch` in server components with caching options (`force-cache`, `revalidate`, `no-store`); the framework dedups and caches. Mutations use Server Actions / route handlers.

**89. What are Server Actions?** Server-side functions callable from components/forms without manually writing an API route — they run on the server and can mutate data. Good for forms and mutations.

**90. How do you do API routes in Next.js?** Route handlers (`app/api/.../route.ts`) export HTTP-method functions; they run server-side. Useful when you need a backend endpoint within the app.

**91. How does Next.js caching work and how do you bust it?** Multiple layers (fetch cache, full-route cache, router cache); revalidate by time (`revalidate`), on-demand (`revalidatePath`/`revalidateTag`), or `no-store`. Caching surprises are the common gotcha.

**92. How do you handle env vars in Next.js?** `NEXT_PUBLIC_` prefix exposes to the browser; everything else is server-only. Never prefix secrets with `NEXT_PUBLIC_`.

**93. What is hydration and hydration mismatch?** Hydration attaches React to server-rendered HTML; a mismatch (server vs client output differs, e.g. `Date.now()`) throws warnings/bugs. Keep server and first client render identical.

**94. How does Next.js handle images?** `next/image` does lazy loading, resizing, format optimization, and prevents layout shift — better than raw `<img>` for performance.

**95. Middleware in Next.js?** Runs on the edge before a request completes — auth checks, redirects, rewrites, geolocation. Keep it light; it's on the hot path.

**96. How would you deploy a Next.js app yourself (not Vercel)?** `next build` then `next start` behind Nginx (or a Node server in Docker), or static export for SSG. Your portfolio is Next 14 deployed on your own infra.

---

## G. Redux Toolkit & RTK Query (97–104)

**97. When do you actually need Redux?** For genuinely global, cross-cutting client state shared by many distant components; for server data, RTK Query/React Query is better. Don't put everything in Redux.

**98. What does Redux Toolkit solve over plain Redux?** Less boilerplate (`createSlice`), immutable updates via Immer, built-in thunk middleware, and good defaults. Plain Redux was notoriously verbose.

**99. What is RTK Query?** A data-fetching/caching layer in RTK — define endpoints, get auto-generated hooks with caching, dedup, invalidation, and loading/error states. Replaces hand-rolled fetch+cache logic.

**100. How does RTK Query cache invalidation work?** Tags: queries provide tags, mutations invalidate them, triggering refetch of affected data. Declarative cache consistency.

**101. Why is immutability important in Redux?** State changes must produce new references so React/Redux can detect changes by reference equality; mutating in place breaks updates. Immer lets you "mutate" safely.

**102. Thunk vs saga?** Thunks (functions for async logic) are simple and built into RTK; sagas (generators) handle complex async flows/cancellation but add complexity. Most apps need only thunks/RTK Query.

**103. What is the selector pattern / `createSelector`?** Functions to read/derive state; memoized selectors avoid recomputing derived data and unnecessary re-renders.

**104. Redux vs Context for state?** Context is for low-frequency global values (theme, user); Redux/RTK for complex, frequently-updated, or large client state with devtools and middleware. Different tools.

---

## H. Queues: BullMQ & RabbitMQ (105–118)

**105. 🔥 Why use a queue at all?** To decouple slow/heavy/retryable work from the request path — the API enqueues and responds fast, workers process async. (Your signup email went from ~1–2s to <100ms this way.)

**106. How does BullMQ work?** Jobs are stored in Redis; workers pull and process them with concurrency, retries, backoff, delays, and events. It's a Redis-backed job queue for Node.

**107. How do retries and backoff work?** Failed jobs retry up to N attempts with fixed/exponential backoff; after exhausting attempts they fail (often to a dead-letter queue). You split retryable vs non-retryable to not waste the budget.

**108. What is a dead-letter queue and why?** A holding queue for jobs that can't succeed (bad data, max retries) so they're visible and don't loop forever. You route non-retryable errors straight there.

**109. How do you ensure a job isn't processed twice?** At-least-once delivery means design idempotent handlers (idempotency keys, state checks); checkpointing lets a resumed job skip completed steps. (Your crash-resume pipeline.)

**110. How do you control concurrency / rate?** Worker concurrency setting and BullMQ rate limiting; you pace sends to respect the WhatsApp API limit instead of bursting.

**111. BullMQ vs RabbitMQ — when each?** BullMQ (Redis) is simple, great for Node job queues with retries/scheduling at moderate scale. RabbitMQ is a full broker — complex routing (exchanges), multiple consumers/languages, higher throughput/durability guarantees.

**112. What are RabbitMQ exchanges?** Routing layer between publisher and queues — direct, topic, fanout, headers — deciding which queues a message lands in. BullMQ has no equivalent (Redis-list based).

**113. What is message acknowledgment (ack/nack)?** Consumer confirms processing so the broker can remove the message; nack/requeue on failure. Without acks you lose messages on crash.

**114. How do you handle a poison message?** Detect repeated failures and route to a DLQ/parking lot rather than blocking the queue. Same principle as your non-retryable routing.

**115. Why a separate worker process vs in-API processing?** Isolates heavy work so it can't starve API latency or take the API down on crash, and lets you scale workers independently. (Your two-service deploy.)

**116. How do you make a job resumable after a crash?** Break it into stages and checkpoint progress (in Redis); on restart, resume from the last completed step. Your 7-queue pipeline does exactly this.

**117. How do you schedule delayed/repeatable jobs?** BullMQ delayed jobs (process after a delay) and repeat options (cron-like) — used for reminders, retries, periodic tasks.

**118. What happens to in-flight jobs on deploy?** Graceful shutdown lets workers finish current jobs before exiting; combined with resumable design, even a forced kill is safe. (Your SIGTERM handling.)

---

## I. Redis (119–134)

**119. 🔥 What is Redis and why is it fast?** In-memory key-value store; data lives in RAM (no disk seek per op) and it's single-threaded with an efficient event loop, so operations are atomic and sub-millisecond.

**120. What data structures does Redis offer?** Strings, hashes, lists, sets, sorted sets, streams, bitmaps, HyperLogLog, geo. Picking the right one (e.g. sorted set for leaderboards/rate windows) is the skill.

**121. How do you use Redis as a cache?** Cache-aside: check Redis, on miss read DB and populate Redis with a TTL; invalidate on writes. You used TTL invalidation to cut p50 ~20%.

**122. Cache invalidation strategies?** TTL expiry, explicit delete on write, or write-through. The hard part is staleness — "a cache with no invalidation plan is a bug."

**123. What eviction policies does Redis have?** When memory is full: `noeviction`, LRU/LFU variants, random, TTL-based. For a cache use `allkeys-lru`; for a queue/store, `noeviction` so data isn't silently dropped.

**124. How does Redis pub/sub work and its limitation?** Publishers send to channels, subscribers receive — fire-and-forget, no persistence; offline subscribers miss messages. You used it for live-price fan-out where missing a tick is fine.

**125. Pub/sub vs Streams?** Streams persist messages, support consumer groups and replay (at-least-once); pub/sub is ephemeral. Use streams when you can't lose messages.

**126. How does Redis persistence work (RDB vs AOF)?** RDB = periodic snapshots (compact, can lose recent writes); AOF = append every write (durable, larger). For a cache, persistence may be off; for a store, AOF. You snapshotted prices to MySQL on an interval.

**127. How do you implement a rate limiter in Redis?** `INCR` a per-user/per-window key with a TTL; reject when over quota. Atomic and fast. (Your per-endpoint quotas.)

**128. How do you implement an atomic counter?** `INCR`/`INCRBY` are atomic single operations — no read-modify-write race. The right tool for likes/views/limits.

**129. What is a distributed lock in Redis (and the caveat)?** `SET key val NX PX ttl` for mutual exclusion across instances; caveats around clock/failover make it imperfect (Redlock is debated) — use for best-effort, not strict correctness.

**130. Why did you take live prices off the DB hot path with Redis?** Writing every tick and polling per second hammered MySQL; storing only the latest value per coin in Redis and snapshotting on an interval removed continuous writes/reads and cut cost ~40%.

**131. How do you avoid a cache stampede?** Staggered TTLs, locking so only one request rebuilds a hot key, or serving stale-while-revalidate. Otherwise an expired hot key floods the DB.

**132. Redis as a queue — pros/cons?** Simple, fast, good for moderate scale (BullMQ); cons: weaker delivery guarantees and durability than a dedicated broker. Fine for your volume.

**133. How do you scale Redis?** Replication for read scaling/HA, Redis Cluster for sharding across nodes, and memory sizing. Single instance is often enough at startup scale.

**134. What's the risk of using Redis as a primary store?** It's memory-bound and (without careful persistence) can lose data on crash; great as cache/queue/ephemeral, risky as the sole source of truth for critical data.

---

## J. PostgreSQL, Drizzle & MongoDB/MySQL (135–150)

**135. 🔥 Why Postgres as your default?** Relational integrity, ACID transactions, rich types (jsonb, arrays, uuid), strong indexing and query planner — safe default unless an access pattern clearly needs otherwise.

**136. How do you design a schema?** Model entities and relationships, normalize for integrity, choose keys and indexes for query patterns, decide nullability/constraints. Denormalize selectively where reads prove too slow.

**137. Why Drizzle over Prisma?** Lightweight, typed SQL close to the actual query, excellent inference, less abstraction/magic — you keep control over the SQL while staying type-safe.

**138. How do migrations work in Drizzle?** Schema defined in TS; generate SQL migration files, version them in git, apply on deploy. Never hand-edit prod schema.

**139. How do you add an index and verify it helps?** `CREATE INDEX` on the queried columns, then `EXPLAIN ANALYZE` before/after to confirm an index scan and lower cost. (Your slow-report fix.)

**140. Transaction in Drizzle/Postgres — when?** Wrap multi-statement operations that must be atomic (e.g. payment verify + state update) in a transaction so they all commit or roll back together.

**141. jsonb in Postgres — when?** For semi-structured/flexible fields within a relational row; indexable with GIN. Gives Mongo-like flexibility without leaving Postgres.

**142. MongoDB — when did you use it and why?** Avisirah feed/messaging — document-shaped, read-heavy, flexible. You used Redis as a read-cache for hot timelines on top.

**143. MongoDB aggregation pipeline?** Staged transforms (`$match`, `$group`, `$lookup`, `$sort`) for analytics/reshaping — Mongo's equivalent of complex SQL queries. `$lookup` (join) is costly.

**144. MongoDB indexing differences?** Same B-tree mechanics; can index nested fields and array elements (multikey). Compound-index order/leftmost-prefix rules apply just like SQL.

**145. When would document embedding beat referencing?** When related data is read together and bounded in size (embed for read speed); reference when data is large, shared, or updated independently. Tradeoff: embedding duplicates, referencing needs lookups.

**146. Why MySQL on the trading platform, and what did you change?** It was the existing store; you moved live prices off it (per-tick writes/per-second polling) into Redis, snapshotting periodically — removing the hot-path load.

**147. Connection pooling for Postgres — why critical?** Postgres handles limited concurrent connections; a pooler (PgBouncer/driver pool) reuses connections so you don't exhaust the server. Pool exhaustion masquerades as "DB slow."

**148. How do you handle a schema change with zero downtime?** Expand-then-contract: add nullable column, deploy dual-compatible code, backfill, then enforce/drop old — every intermediate state safe for old+new code. `CREATE INDEX CONCURRENTLY` to avoid locks.

**149. How do you prevent a lost update / race on a row?** Atomic `UPDATE ... WHERE` guards, `SELECT ... FOR UPDATE`, or optimistic version columns. Don't read-then-write in app code.

**150. SQL vs NoSQL — your one-line stance?** Start relational for integrity; reach for NoSQL when a specific access pattern (write scale, flexible shape, sub-ms key lookup) clearly wins. Polyglot per use case.

---

## K. Auth, Security & Encryption (151–166)

**151. 🔥 Walk through JWT access/refresh auth.** Short-lived access token authenticates requests; long-lived refresh token mints new access tokens without re-login; both in HttpOnly cookies so JS can't read them (XSS mitigation).

**152. JWT vs session cookies?** JWT is stateless (no server store, but hard to revoke before expiry); sessions are stateful (easy revoke, needs a store). Refresh-token rotation/blacklists address JWT revocation.

**153. Why HttpOnly cookies?** JavaScript can't read them, so an XSS payload can't steal the token. Pair with `Secure` and `SameSite` for CSRF defense.

**154. Authentication vs authorization?** Authn = who you are (login); authz = what you can do (RBAC). Different layers — you enforce RBAC per route after authn.

**155. What is RBAC?** Role-Based Access Control — permissions attached to roles (admin/user/moderator), users get roles; routes check role. Simpler than per-user permissions.

**156. How do you store passwords?** Hash with a slow, salted algorithm (bcrypt/argon2) — never plaintext or fast hashes. Salting defeats rainbow tables; slowness defeats brute force.

**157. Hashing vs encryption?** Hashing is one-way (passwords, integrity); encryption is reversible with a key (data you must read back, like phone numbers). Don't "encrypt" passwords.

**158. 🔥 Why AES-256-CBC with a unique IV per record?** Symmetric encryption for data you must decrypt later (phone numbers); a unique IV ensures identical plaintexts don't yield identical ciphertext (no pattern leakage). IV stored alongside, not secret.

**159. Why CBC vs GCM?** GCM adds authentication (tamper detection) — if rebuilding, you'd prefer GCM; CBC met the confidentiality need. Knowing GCM is better is the senior nuance.

**160. Why decrypt only in the worker and mask elsewhere?** Minimize plaintext exposure — decrypt at the narrowest scope (the send moment), mask everywhere else (`+91*****1234`) so logs/API leaks expose nothing. Defense in depth.

**161. How do you prevent SQL injection?** Parameterized queries/prepared statements (ORMs do this) — never string-concatenate user input into SQL.

**162. How do you prevent XSS?** Escape/encode output, sanitize HTML, use framework defaults (React escapes by default), CSP headers, and HttpOnly cookies for tokens.

**163. What is CSRF and how do you prevent it?** A forged request using the victim's cookies; prevent with SameSite cookies, CSRF tokens, and checking origin. Relevant when using cookie auth.

**164. How do you handle secrets/keys?** Env vars or a secrets manager, injected at runtime, rotated, never committed or baked into images. Encryption keys especially.

**165. How does rate limiting improve security?** Caps abuse — brute-force logins, signup spam, scraping. Your quotas (login 5/15min, signup 3/hr) are tuned to human-vs-bot behavior.

**166. How do you secure webhooks (Razorpay)?** Verify the signature (HMAC) to confirm origin, and make handlers idempotent since webhooks can be delivered more than once. Never trust client-reported payment status.

---

## L. AI / LLM / Claude / RAG (167–184)

**167. 🔥 How do you integrate the Claude API in production?** Server-side calls (never expose keys client-side), structured prompts, parse/validate responses, handle errors/timeouts/retries, and control cost via caching. You run vision + text behind guardrails.

**168. What is a confidence gate and why?** A threshold below which the system refuses rather than guesses — critical in high-stakes domains. You used 0.75 for Dawa Saathi; uncertain reads refuse.

**169. Why bias toward refusing in a medical context?** A confidently wrong drug summary is far more harmful than an honest "I can't read this." False positives cost more than false negatives here.

**170. How did you measure ~80% vision accuracy?** Built an eval set of real medicine strips with known labels, compared extracted vs ground truth — an honest finite-sample estimate, not a benchmark.

**171. 🔥 What is RAG?** Retrieval-Augmented Generation — fetch relevant documents and feed them as context so the model answers from grounded, up-to-date sources instead of memory. You used RAG over banking product docs.

**172. Why use RAG instead of fine-tuning?** RAG updates by changing the knowledge base (cheap, current, auditable); fine-tuning bakes knowledge into weights (expensive, static). RAG also cites sources and reduces hallucination.

**173. How does retrieval work in RAG?** Chunk documents, embed them into vectors, store in a vector index; embed the query, find nearest chunks (cosine similarity), pass top-k as context. Chunking strategy matters a lot.

**174. What are embeddings?** Numeric vectors capturing semantic meaning so similar text is close in vector space — the basis for semantic search/retrieval.

**175. How do you reduce hallucination?** Grounded/doc-only prompting, confidence gating, asking the model to cite or say "I don't know," and constraining scope. You routed uncertain banking answers to a human.

**176. 🔥 How do you control LLM cost?** Avoid the call (cache results — your ~30% token cut), shrink prompts (scoped, ingredient-only), and gate low-value calls (refuse unusable input early). Cost control starts before the request.

**177. How did caching cut tokens ~30%?** Repeat medicine scans hit a Redis→Postgres cache and skip the model entirely — no tokens, instant answer. Savings concentrate on commonly-scanned drugs.

**178. Why two models (Sonnet for reading, Haiku for awareness)?** Use the stronger/costlier model only for the safety-critical step and a cheaper model for the lower-stakes generation — quality where it matters, cost savings where it doesn't.

**179. How do you design a good prompt?** Clear role/task, explicit constraints and format, examples, and scoping to reduce drift; for structured output, demand JSON-only and parse defensively.

**180. How do you get structured output from an LLM?** Instruct JSON-only (no prose/markdown), then parse with error handling and validation (strip fences, validate schema). Treat the model as untrusted input.

**181. What is a confidence/uncertainty signal from a model?** Derived from the model's own expression of certainty or downstream checks; you act only above threshold and refuse/escalate below. Imperfect but useful as a safety lever.

**182. How do you handle the model being unavailable/slow?** Timeouts, retries with backoff, graceful fallback (queue, cache, or "try again"), and never block the user indefinitely. You enqueue async where possible.

**183. What are the risks of LLMs in your domains and mitigations?** Confident wrong output and over-trust by vulnerable users; mitigate with confidence gates, scoped prompts, human escalation, and clearly informational (not advisory) framing.

**184. How do you evaluate an LLM feature?** A labeled eval set, track accuracy/refusal rates, monitor cost per request, and watch for drift — measure rather than vibe-check.

---

## M. AWS, Docker, Nginx, PM2, CI/CD (185–198)

**185. 🔥 What are presigned S3 URLs and why use them?** Short-lived signed URLs that let clients upload/download directly to S3, bypassing your server — offloads bandwidth/CPU. Your ~40% CPU win and ~20% faster uploads.

**186. EC2 vs serverless — tradeoff?** EC2 = full control, persistent, predictable cost, you manage it; serverless = no ops, autoscale, pay-per-use, but cold starts and limits. You run EC2 for control at your scale.

**187. How do you deploy with Docker behind Nginx?** Containers run the app; Nginx terminates TLS and reverse-proxies to them (and load-balances multiple instances). Your two services sit behind Nginx.

**188. What does Nginx do as a reverse proxy?** TLS termination, routing, load balancing, caching, compression, rate limiting, and serving static files — shields and fronts your app servers.

**189. What does PM2 do?** Node process manager — keeps the app alive (restart on crash), cluster mode across cores, log management, zero-downtime reloads. You paired it with CI/CD for health-checked restarts.

**190. PM2 cluster mode vs Docker scaling?** PM2 cluster forks processes within a host across cores; Docker scales containers across hosts. They solve overlapping problems; pick per deployment model.

**191. 🔥 What's in your CI/CD pipeline?** Build, test, lint, build image, deploy with health-checked near-zero-downtime restart (rolling). You set this up on CircleCI with PM2 for repeatable releases.

**192. What is a health check and why?** An endpoint/command signaling the app is ready/alive so the orchestrator routes traffic only to healthy instances and restarts unhealthy ones — enables zero-downtime deploys.

**193. Docker: image vs container?** Image = read-only blueprint (layers); container = running instance with a writable layer. One image, many containers.

**194. Why multi-stage Docker builds?** Build in a heavy stage, copy only artifacts into a slim runtime stage — smaller, more secure images without build tools/dev deps.

**195. How do you persist data in Docker?** Named volumes (DB data), bind mounts (dev), tmpfs (ephemeral) — the container's writable layer is lost on removal.

**196. How do containers talk to each other?** On a shared (user-defined) network via service/container name through Docker's DNS — no hardcoded IPs. Compose sets this up.

**197. How do you achieve near-zero-downtime deploys?** Health checks + rolling restart (old stays up until new is healthy) + graceful shutdown of old (finish in-flight, SIGTERM with timeout). Your stack does this.

**198. How do you do blue-green or rollback?** Keep the previous image tagged; route traffic to the new ("green") only after health checks, roll back by repointing to the old image. Immutable tags make rollback trivial.

---

## N. Tools & Architecture (199–200)

**199. 🔥 Why a modular monolith over microservices?** Microservices solve org/independent-scaling problems you don't have solo; they add network latency, distributed transactions, and ops tax. You built a modular monolith and split out only the boundary that needed it (API vs worker / socket server) — extract a service later when there's a real reason.

**200. How do you observe/debug production (Pino, metrics)?** Structured JSON logs (Pino) with correlation, host/app metrics, error rates; on an issue: detect → reproduce → isolate (logs/metrics/EXPLAIN) → fix root cause → verify with the same measurement. Observability is how you find the real bottleneck instead of guessing.

---

## How to drill this

- **Start with the 🔥 questions** — those are the highest-probability and the ones tied to your actual work.
- **Cover the question, recall the answer, then check.** Don't read passively — say it out loud in your own words.
- **For anything you can't yet defend, either learn it or drop it from how you pitch yourself** — better to go deep on your real stack than fake breadth.
- **Every answer should land on a tradeoff.** "X over Y because Z, cost is W." That sentence is what separates a junior recall from a senior discussion.
- **Anchor to your projects.** Most of these map to something you actually built — Pilooopu, Dawa Saathi, the trading/banking/social work. Real beats theory every time.

# Suraj Khonde — Interview Question Bank (Resume-Based)

This file contains:
- **150** interview questions derived from your resume (basic → advanced).
- **100** separate system design prompts.

Use it like this:
- For each question, answer in **Meaning → Why → How → Example → Tradeoffs** format.
- Practice speaking answers out loud.

---

## Table of Contents
1. [Resume & Behavioral (20)](#resume--behavioral-20)
2. [JavaScript & TypeScript (25)](#javascript--typescript-25)
3. [Node.js, Express, REST APIs (25)](#nodejs-express-rest-apis-25)
4. [React, Next.js, Frontend (20)](#react-nextjs-frontend-20)
5. [Databases: MongoDB & MySQL (20)](#databases-mongodb--mysql-20)
6. [Redis: Caching, Sessions, Pub/Sub (15)](#redis-caching-sessions-pubsub-15)
7. [Realtime: WebSockets & Socket.IO (10)](#realtime-websockets--socketio-10)
8. [Background Jobs & Event-Driven (10)](#background-jobs--event-driven-10)
9. [AWS, Docker, Deployment, Observability (5)](#aws-docker-deployment-observability-5)
10. [Testing & Quality (0)](#testing--quality-0)
11. [System Design Questions (100)](#system-design-questions-100)

> Note: “Testing & Quality” is included inside other sections (error handling, monitoring, CI/CD), so the count is 0 here.

---

## Resume & Behavioral (20)
1. Walk me through your resume in 60–90 seconds.
2. What kind of roles are you targeting and why (backend-heavy full stack vs frontend-heavy)?
3. What was your most challenging production bug? How did you debug it end-to-end?
4. Tell me about a time you improved performance measurably (what metrics, how measured).
5. Describe a time you handled an incident/outage. What was the root cause and the fix?
6. How do you prioritize when multiple issues/features are coming at once?
7. Tell me about a disagreement with a teammate/PM and how you resolved it.
8. What does “ownership” mean to you? Give a real example from your work.
9. How do you ensure your changes don’t break production?
10. What’s the best code review feedback you’ve received, and what changed after that?
11. Tell me about a system you made “more reliable.” What exactly changed?
12. What’s a feature you shipped from requirement → release? Describe the steps.
13. How do you communicate tradeoffs to non-technical stakeholders?
14. What’s your approach to learning something new quickly under pressure?
15. Why did you leave your last role / what happened with the layoff? (keep it professional)
16. What’s one mistake you made and what process you changed to prevent it?
17. Explain your proudest project and why you’re proud of it.
18. If you join us, what do you want to learn in the next 6–12 months?
19. How do you handle vague requirements? What clarifying questions do you ask?
20. Give an example where you reduced crashes or improved stability.

---

## JavaScript & TypeScript (25)
1. Explain `var` vs `let` vs `const` (scope, hoisting, TDZ).
2. What is closure? Give a real use-case.
3. Explain prototypal inheritance in JavaScript.
4. Explain `this` binding rules (call/apply/bind, arrow functions).
5. What is the event loop? Explain microtasks vs macrotasks.
6. Explain Promises: states, chaining, error propagation.
7. `async/await` under the hood: how does it relate to promises?
8. How do you handle errors in async code reliably?
9. What is “unhandled promise rejection” and how to prevent it?
10. Explain `==` vs `===` and type coercion pitfalls.
11. What is deep copy vs shallow copy? Common ways to clone objects.
12. How does garbage collection work in JS at a high level?
13. Explain pass-by-value vs pass-by-reference behavior in JS.
14. What are modules? CommonJS vs ES Modules.
15. Explain `import()` dynamic import and when you use it.
16. What is TypeScript? What problems does it solve?
17. `any` vs `unknown` vs `never` — when to use each?
18. Union vs intersection types with real examples.
19. Generics in TypeScript — show an example you used.
20. Type narrowing and type guards — how do you do it?
21. Optional chaining and nullish coalescing — common mistakes.
22. `readonly` and immutability patterns in TS.
23. How do you type Express request/response handlers correctly?
24. How do you validate runtime input even with TypeScript?
25. How do you structure types in a large codebase (dto, domain, api)?

---

## Node.js, Express, REST APIs (25)
1. How does Node.js handle concurrency with a single thread?
2. What are streams and buffers in Node? When do you use each?
3. Explain backpressure in streams.
4. How do you design REST APIs (resources, status codes, conventions)?
5. What are HTTP methods and which are idempotent? Why does it matter?
6. Explain authentication vs authorization with an example.
7. JWT: what’s inside it? How do you validate it securely?
8. JWT pitfalls: storage, XSS/CSRF, rotation, expiry.
9. How do you implement refresh tokens safely?
10. What is bcrypt? Why use salt? What are common mistakes?
11. OAuth basics: roles (client, resource owner, provider) and flow at high level.
12. Passport.js: what problem does it solve? How do strategies work?
13. Explain middleware in Express and how error middleware works.
14. How do you implement centralized async error handling in Express?
15. What is input validation? How do you do it with Joi/Zod?
16. Explain CORS and common misconfigurations.
17. How do you prevent NoSQL injection / SQL injection?
18. What is rate limiting? How to implement it (Redis-based)?
19. How do you design pagination (offset vs cursor) and tradeoffs?
20. How do you design filtering/sorting APIs at scale?
21. How do you structure a Node project (folders, layers, services)?
22. How do you handle config safely across environments?
23. What is graceful shutdown in Node? Why is it important?
24. How do you handle retries/timeouts for downstream calls?
25. What makes an API “reliable” in production (timeouts, circuit breakers, idempotency)?

---

## React, Next.js, Frontend (20)
1. Explain React rendering: reconciliation and re-render triggers.
2. Controlled vs uncontrolled components — when and why?
3. What are hooks rules? Why do they exist?
4. `useEffect` common pitfalls and cleanup patterns.
5. When do you use `useMemo` / `useCallback` and when not?
6. What is React Context and when does it cause performance issues?
7. Redux Toolkit: why use it? Explain slices, reducers, async thunks.
8. Redux vs React Query — responsibilities and tradeoffs.
9. React Query: caching, staleTime, refetching, invalidation.
10. How do you manage forms + validation cleanly in React?
11. What is code splitting and how did you do it (dynamic imports, routes)?
12. How do you measure and reduce bundle size?
13. Error boundaries: what do they catch and what they don’t?
14. What is hydration in Next.js? Common SSR pitfalls.
15. CSR vs SSR vs SSG — how do you choose?
16. How do you handle auth in frontend apps securely?
17. How do you avoid unnecessary renders in large lists?
18. What is debouncing/throttling and where would you use it?
19. How do you ensure accessibility basics in UI components?
20. How do you handle file uploads (Cloudinary) from frontend to backend?

---

## Databases: MongoDB & MySQL (20)
1. MongoDB vs MySQL — how do you choose?
2. Explain indexing: what is an index and why does it speed reads?
3. Index tradeoffs: write amplification, storage, choose fields.
4. How do you find slow queries and improve them?
5. Explain MongoDB document modeling (embed vs reference).
6. What are transactions in MongoDB? When do you need them?
7. Explain ACID in SQL databases with a simple example.
8. What is isolation level and dirty read/non-repeatable read/phantom read?
9. What is a JOIN? When are joins expensive?
10. Schema design for chat messages: what fields and indexes do you add?
11. Pagination in DB: offset vs cursor, and why offset becomes slow.
12. How do you implement “search” (text index vs external search)?
13. How do you handle migrations in MongoDB/MySQL?
14. What is normalization? When do you denormalize?
15. What is N+1 query problem and how to avoid it?
16. How do you ensure uniqueness (unique indexes) in MongoDB/MySQL?
17. Explain locking vs MVCC at a high level.
18. How do you design for multi-tenancy (tenantId, indexes, data isolation)?
19. How do you handle data retention (TTL indexes, archival)?
20. How do you handle database connection pooling in Node?

---

## Redis: Caching, Sessions, Pub/Sub (15)
1. What is Redis and where does it fit in an architecture?
2. Cache-aside pattern: explain it step-by-step.
3. Write-through vs write-behind caching — tradeoffs.
4. How do you pick cache keys? How do you version keys safely?
5. What is cache invalidation and why is it hard?
6. What is cache stampede? How do you prevent it?
7. What is TTL and how do you decide TTL values?
8. Redis eviction policies (LRU/LFU/etc.) — what happens under memory pressure?
9. Redis data types: string, hash, list, set, sorted set — real use cases.
10. How do you use Redis for sessions? What are the security concerns?
11. How do you implement rate limiting with Redis?
12. Redis Pub/Sub: what is it and what are its guarantees/limitations?
13. Pub/Sub vs Streams vs queue — when to choose each?
14. How do you scale Socket.IO with Redis adapter and what role Pub/Sub plays?
15. What metrics do you monitor for Redis in production?

---

## Realtime: WebSockets & Socket.IO (10)
1. WebSockets vs HTTP polling vs SSE — what are the tradeoffs?
2. How does Socket.IO differ from raw WebSockets?
3. How do you authenticate a socket connection?
4. How do you implement rooms/namespaces in Socket.IO?
5. What is sticky session and why it matters for realtime systems?
6. How do you scale Socket.IO horizontally behind a load balancer?
7. What happens when a server instance dies? How do clients recover?
8. How do you prevent message duplication in realtime systems?
9. How do you handle offline users (store-and-forward)?
10. How do you test realtime features?

---

## Background Jobs & Event-Driven (10)
1. What problems do background jobs solve in web apps?
2. BullMQ basics: queue, worker, job, retries, backoff.
3. What is idempotency in jobs and why is it required?
4. What is a DLQ (dead letter queue) and when do you use it?
5. How do you design job payloads (small payload + fetch details)?
6. How do you handle “exactly once” vs “at least once” delivery?
7. How do you schedule delayed jobs (email after X minutes)?
8. How do you handle job timeouts and stuck jobs?
9. What observability do you add to jobs (metrics, logs, tracing)?
10. When should a job become a workflow (step functions / saga pattern)?

---

## AWS, Docker, Deployment, Observability (5)
1. Explain your deployment approach: EC2/PM2 vs serverless (Lambda).
2. Docker: what problem does it solve and how do you structure images?
3. Nginx reverse proxy basics: routing, SSL termination, static assets.
4. How do you set up HTTPS (Let’s Encrypt) and renewals?
5. What do you monitor in production (latency, errors, saturation) and how?

---

## Testing & Quality (0)

---

# System Design Questions (100)

## A) Foundational (20)
1. Design a URL shortener (basic).
2. Design an authentication service (JWT + refresh tokens).
3. Design rate limiting for an API gateway.
4. Design a file upload service (Cloudinary/S3).
5. Design an email sending system (retries, templates, tracking).
6. Design a notification system (push/in-app/email).
7. Design a caching strategy for a read-heavy API.
8. Design a search feature for products/blog posts.
9. Design pagination for a feed with millions of rows.
10. Design a metrics/logging system for a Node backend.
11. Design a session management system (Redis sessions).
12. Design a password reset + account recovery flow.
13. Design an API for comments + likes.
14. Design a feature flag system.
15. Design a webhook delivery system.
16. Design a service to generate PDFs/exports asynchronously.
17. Design an image/video processing pipeline (background jobs).
18. Design a system to prevent duplicate requests (idempotency keys).
19. Design an “online/offline presence” indicator.
20. Design a basic audit log system.

## B) Realtime & Messaging (20)
21. Design a 1:1 chat system (WebSockets).
22. Design a group chat system with rooms and typing indicators.
23. Scale Socket.IO across multiple servers behind a load balancer.
24. Design message delivery guarantees (at least once vs exactly once).
25. Design read receipts and last-seen features.
26. Design “online users” tracking at scale.
27. Design message history storage and indexing strategy.
28. Design media messaging (images/video) for chat.
29. Design push notifications for missed messages.
30. Design spam/abuse prevention for chat.
31. Design presence + typing indicator without overwhelming servers.
32. Design message ordering across distributed servers.
33. Design a multi-region chat (latency + failover).
34. Design a bot-to-agent handoff chat workflow with context transfer.
35. Design agent routing + customer queue for support chat.
36. Design a “conversation context” storage format and retrieval.
37. Design an in-app notification bell with real-time updates.
38. Design fan-out notifications for followers/room members.
39. Design a live dashboard (SSE/WebSockets) for support metrics.
40. Design a real-time collaborative editor (basic).

## C) Background Jobs & Reliability (20)
41. Design a background job system for sending emails at scale.
42. Design retries/backoff and DLQ strategy for jobs.
43. Design idempotent job processing for “send notification”.
44. Design a scheduler for delayed jobs (reminders).
45. Design bulk processing (import CSV users) with progress tracking.
46. Design a workflow for media processing (transcode + thumbnails).
47. Design an outbox pattern to ensure DB + event consistency.
48. Design a system to clean up expired sessions reliably.
49. Design a system to handle timeouts for downstream services.
50. Design a circuit breaker + fallback caching approach.
51. Design a system to detect and auto-heal stuck workers.
52. Design tracing for a request that triggers background jobs.
53. Design a webhook retry system (deliver-at-least-once).
54. Design a system for “exactly-once-like” semantics using idempotency keys.
55. Design a reconciliation job to fix missed events.
56. Design a queue-based image moderation system.
57. Design a notification preference system (user settings + channels).
58. Design a system to prevent double-charging/payments.
59. Design a rate-limit + abuse protection pipeline.
60. Design a multi-tenant job processing system.

## D) Data Modeling, Search, Performance (20)
61. Design a feed system (home timeline) with caching.
62. Design a “top posts” ranking system (hot score).
63. Design search with autocomplete (prefix search).
64. Design full-text search + filters (tags, categories).
65. Design data partitioning strategy for large collections.
66. Design sharding approach for chat messages or events.
67. Design a schema for notifications with read/unread.
68. Design a system to store and serve user profiles efficiently.
69. Design a system for analytics events ingestion (basic).
70. Design a deduplication system for events.
71. Design cache key strategy for multiple query filters.
72. Design preventing cache stampede on popular endpoints.
73. Design query optimization approach for large Mongo collections.
74. Design SQL schema for orders/inventory and index strategy.
75. Design handling “eventual consistency” in distributed reads.
76. Design a multi-region read replica strategy.
77. Design a “delete user” (GDPR) flow across services.
78. Design data retention/archival for messages/logs.
79. Design a reporting system (daily aggregates) with batch jobs.
80. Design a system to generate recommendations (basic).

## E) Infra, Security, Deployment (20)
81. Design deployment architecture for Node + React with CI/CD.
82. Design a secure secrets/config management approach.
83. Design an API gateway with auth, rate limits, and logging.
84. Design multi-environment setup (dev/stage/prod) with safe migrations.
85. Design multi-tenant SaaS architecture (tenant isolation).
86. Design a CDN strategy for static assets and media.
87. Design DDoS protection + rate limiting strategy.
88. Design secure file upload with virus scanning.
89. Design a monitoring + alerting strategy (SLI/SLO).
90. Design a logging pipeline with structured logs and correlation IDs.
91. Design an incident response playbook for API outages.
92. Design a system to rotate JWT signing keys safely.
93. Design CSRF/XSS protection strategy for a SPA.
94. Design a permissions system (RBAC/ABAC) for admin panels.
95. Design an audit + compliance logging system.
96. Design a canary release / blue-green deployment strategy.
97. Design rollback strategy for bad deployments.
98. Design autoscaling strategy for a bursty workload.
99. Design cost-optimized serverless vs EC2 architecture.
100. Design a multi-region failover plan for critical APIs.

---

## Quick Practice Rule (for every answer)
**Meaning → Why → How → Example → Tradeoffs**

If you want, I can generate:
- a **30-day schedule** mapping these questions to daily practice, or
- **model answers** for your top 30 most frequent questions.

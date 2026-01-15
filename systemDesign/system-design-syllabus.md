# System Design Syllabus (Scratch → Interview-Ready, ~3 Years Experience)

> Use this as a **topic checklist**. You don’t need to master everything at once—start with the “Core Track”, then add “Deep Dives”.

---

## 0) How system design interviews work (the meta)
- What interviewers evaluate: **clarity, trade-offs, scalability thinking, correctness, simplicity**
- Typical flow
  - Clarify requirements → define scope → constraints/SLOs → capacity estimates
  - High-level design (boxes) → APIs + data model → scaling plan
  - Deep dives (DB, cache, async, consistency, reliability) → bottlenecks → cost
- How to communicate
  - Assumptions, constraints, trade-offs (CAP/latency/cost/dev-speed)
  - “Happy path” + failure modes

**Practice:** Explain any design in 5 minutes: problem → assumptions → diagram → 3 trade-offs.

---

## 1) Internet + Networking Fundamentals (must-have)
### HTTP/HTTPS
- Request/response, headers, status codes
- HTTP methods: GET/POST/PUT/PATCH/DELETE (when to use which)
- Content types, JSON, multipart/form-data
- Cookies vs headers, sessions vs tokens
- TLS basics (why HTTPS, handshake at a high level)
### Idempotency & safety
- Safe vs idempotent methods (GET safe; PUT idempotent; POST usually not)
- Idempotency keys (payments, order creation, retries)
### Caching semantics
- Cache-Control, ETag, If-None-Match, Last-Modified
### Latency basics
- RTT, DNS, connection setup, keep-alive, HTTP/2/HTTP/3 overview

**Practice:** Design an idempotent “Create Order” API with retry-safe behavior.

---

## 2) API Design & Contracting (core)
- REST fundamentals (resources, nouns, versioning)
- Alternatives: RPC/gRPC (when & why), GraphQL basics (trade-offs)
- Pagination: offset vs cursor (stability, performance)
- Filtering/sorting/search parameters
- Rate limiting: token bucket/leaky bucket concepts
- Request validation, error format, consistent error codes
- API versioning strategies (URL vs header; backward compatibility)
- Backward/forward compatibility for clients
- API documentation: OpenAPI/Swagger, examples

**Practice:** Define APIs for: feed, follow/unfollow, upload media, comments, likes.

---

## 3) Data Modeling Basics (core)
- Identify entities & relationships
- Normalization vs denormalization (why you denormalize in distributed systems)
- Choosing IDs (UUID vs Snowflake/ULID; ordering)
- Read patterns vs write patterns (design from queries)
- Soft delete, audit logs, data retention
- Indexing fundamentals (what an index helps, trade-offs)
- Constraints: uniqueness, foreign keys (when to enforce at DB vs app)

**Practice:** Model DB tables/collections for a “posts + comments + likes” system.

---

## 4) Databases & Storage Systems (core → deep)
### SQL / Relational
- Transactions, isolation levels, locks (high level)
- ACID and when it matters
- Replication: primary/replica (read scaling, lag)
### NoSQL categories
- Key-value (Redis), document (MongoDB), wide-column (Cassandra), graph (Neo4j)
- When NoSQL is better: scale, flexible schema, high write throughput
### Consistency basics
- Strong vs eventual consistency (what users will “see”)
- Quorum reads/writes conceptually
### Data partitioning
- Sharding strategies: hash-based, range-based, directory-based
- Hot partitions & mitigation (salting, adaptive partitioning)
### Object storage
- Blob storage (S3/GCS): presigned URLs, multipart upload, lifecycle rules

**Practice:** Decide SQL vs MongoDB vs Cassandra for a high-write event stream. Explain why.

---

## 5) Caching (core)
- Why cache: latency reduction + cost + DB offload
- Cache-aside vs write-through vs write-back
- TTL, eviction policies, cache stampede, dogpile effect
- Invalidation strategies (the hardest part)
- Local cache vs distributed cache
- CDN basics (static + media delivery)
- Consistency with cache (stale reads) and acceptable staleness

**Practice:** Add caching to “Get User Profile” and “Feed” with invalidation story.

---

## 6) Scalability Building Blocks (core)
- Load balancing (L4 vs L7), health checks, sticky sessions
- Horizontal vs vertical scaling
- Stateless services (why it matters)
- Service discovery basics (high level)
- Connection pooling (DB pools, limits)

**Practice:** Draw an architecture for 1M DAU vs 10M DAU for the same app.

---

## 7) Asynchronous Processing & Messaging (core → deep)
- Why async: latency hiding, decoupling, reliability
- Queues vs streams (SQS/RabbitMQ vs Kafka)
- At-least-once vs at-most-once vs exactly-once (practical meaning)
- Consumers, partitions, ordering, consumer groups
- Backpressure & retries, dead-letter queues (DLQ)
- Idempotent consumers (dedupe keys)
- Outbox pattern (DB → event publish safely)

**Practice:** Design “send email + push notification” reliably after user signup.

---

## 8) Reliability & Fault Tolerance (core)
- SLO/SLI/SLA basics (latency, availability, error rate)
- Redundancy, failover, multi-AZ vs multi-region (trade-offs)
- Timeouts, retries, jitter, circuit breakers
- Bulkheads, load shedding, graceful degradation
- Disaster recovery (RPO/RTO concepts)
- Handling partial failures (downstream dependency issues)

**Practice:** Add resilience to a payment flow with retry-safe design.

---

## 9) Observability (core)
- Logs, metrics, traces (what each is best for)
- Structured logging, correlation IDs
- Golden signals: latency, traffic, errors, saturation
- Alerting basics: symptoms vs causes
- Debugging in distributed systems (trace a request end-to-end)

**Practice:** List 10 metrics you’d monitor for an API gateway.

---

## 10) Security & Privacy (core)
- Authentication vs authorization
- Session cookies vs JWT (trade-offs)
- OAuth basics (high level) + access/refresh tokens
- RBAC vs ABAC
- Common attacks: SQL injection, XSS, CSRF, SSRF, replay attacks
- Rate limiting + WAF basics
- Secrets management, rotation
- Encryption at rest / in transit
- PII handling, masking, retention

**Practice:** Secure a file upload endpoint (auth, size limits, content checks, scanning).

---

## 11) Deployment & Cloud Basics (useful for interviews)
- Containers, images, basic Docker concepts
- CI/CD pipeline basics
- Blue/green, canary deploys, rolling deploys
- Autoscaling triggers (CPU, latency, queue depth)
- Infra components: VPC, subnets, security groups, NAT (high-level)
- Managed services vs self-hosted (cost + ops trade-offs)

**Practice:** Deploy a Node API with DB + Redis + queue; list scaling knobs.

---

## 12) Performance Engineering (core)
- Where latency comes from: network + compute + DB + serialization
- Profiling mindset: measure → identify bottleneck → fix
- N+1 queries, batching
- Index tuning basics
- CDN and compression (gzip/brotli) basics
- Payload size and pagination
- Throughput vs latency (and why they trade off)

**Practice:** Optimize “search posts” endpoint with indexes + caching + pagination.

---

## 13) Capacity Planning & Back-of-the-Envelope (core)
- Estimating QPS, storage, bandwidth
- Understanding units: bytes, KB/MB/GB, bits vs bytes
- Power of two approximations (KiB/MiB/GiB mental math)
- Peak vs average load
- Headroom and safety factors
- Cost awareness (DB read replicas, cache size, CDN egress)

**Practice:** Estimate storage for 10M users posting 2 photos/day for 1 year.

---

## 14) Consistency, Transactions, and Distributed Trade-offs (deep)
- CAP theorem (practical interpretation)
- Strong vs eventual consistency in product terms
- Distributed locks (when to avoid; alternatives)
- Saga pattern (distributed transactions)
- Exactly-once “myths” and practical approaches
- Data correctness strategies: unique constraints, idempotency, dedupe, reconciliation jobs

**Practice:** Design “wallet balance” with correctness under concurrency.

---

## 15) System Design Patterns (deep but very interview-relevant)
- CQRS (separate read/write models)
- Event sourcing (when it helps)
- Read-through/write-through caches
- Materialized views
- Fan-out on write vs fan-out on read (feeds)
- BFF (backend-for-frontend)
- Strangler pattern for migrations
- Feature flags and gradual rollout
- Multi-tenant design basics

**Practice:** Choose fan-out strategy for a social feed. Explain with numbers.

---

# Common System Design Interview Problems to Practice (pick 8–12)
1. URL shortener
2. Rate limiter
3. News feed / social feed
4. Chat (1:1 + groups)
5. Notification system
6. File upload + media processing pipeline
7. Search/autocomplete (basic)
8. E-commerce cart + checkout (idempotency heavy)
9. Ride-sharing matching (high level)
10. Analytics/event ingestion pipeline
11. Distributed cache design (high level)
12. Logging/monitoring pipeline (high level)

---

# Quick “Interview Checklist” (print this)
- Requirements: core features + out-of-scope
- Non-functional: latency, availability, consistency, security, cost
- Capacity: QPS, storage, bandwidth
- APIs: endpoints, payloads, idempotency, pagination
- Data model: entities + indexes
- High-level architecture: LB → services → DB/cache/queue → storage/CDN
- Bottlenecks: DB hot spots, cache stampede, queue backlog
- Reliability: retries/timeouts/circuit breaker + DLQ
- Observability: logs/metrics/traces + SLOs
- Trade-offs: explain why you chose X over Y

---

## Suggested study order (fast path)
1) HTTP + APIs + idempotency  
2) Data modeling + SQL basics + indexing  
3) Caching + load balancing + stateless services  
4) Queues + async processing + reliability  
5) Capacity estimates + common case studies (URL shortener, feed, notifications)  
6) Consistency trade-offs + deep dives (only after core is solid)

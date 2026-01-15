# system Design

## What `system design` means (simple)

When you build this app, you must decide:

Where data lives (DB? files? search index?)

How users access it (API design)

How you store images (don’t store inside DB)

How you handle slow tasks (emails, indexing → background jobs)

How you deploy (one server? containers? managed DB?)

How you keep it fast (pagination, caching, CDN)

How you keep it safe (auth, validation, rate limits)

### There are three buckets when we say “where data lives”:

**Relational DB (SQL) like Postgres/MySQL**
Best when you need relationships, constraints, and transactions (ACID).
Great for user accounts, equipment listings, bookings/inquiries, payments, admin workflows.
Consistency comes from constraints + transactions, not just “tables”.
NoSQL DB (different types, not one thing)

**Document DB (MongoDB):** 
flexible schema, faster iteration, good for JSON-like data.
Key–value (Redis / DynamoDB style): super fast access by key; often used for cache/session/locks.
Wide-column (Cassandra): high write throughput at scale.
NoSQL isn’t automatically “faster”; it’s chosen for scaling pattern, flexibility, or access pattern.
**Object/File storage (S3, GCS)**
For images, videos, PDFs—don’t store these blobs inside SQL rows in most web apps.
Store only the URL + metadata in your DB.

And sometimes a 4th:
4) Search index (Elasticsearch/OpenSearch/Meilisearch)
Used for text search + advanced filtering + ranking, not as the source of truth.
# Redis

## Redis Pub/Sub

## Understanding Pub/Sub

Think of Pub/Sub like a school announcement system.
Publishers are students/teachers who write an announcement.
There’s a notice board room (the broker / Pub/Sub service) that collects announcements.
Subscribers are classes who signed up to receive certain announcements.
The broker makes sure everyone who subscribed gets a copy, even though the publisher doesn’t know who they are.
That’s the big win: publisher and subscriber don’t talk directly. They only agree on a topic name.

### Diagram (who handles messages + how requests/queues are managed)

#### 1) High-level view

``` text

   Publisher (Service A)
          |
          |  publish(message)
          v
+---------------------------+
|   Pub/Sub Broker/Service  |   (handles routing, buffering, retries)
|        (TOPIC)            |
+---------------------------+
      |            |
      | fan-out    | fan-out  (copies message for each subscription)
      v            v
 Subscriber 1   Subscriber 2
 (Service B)    (Service C)

```

#### 2) Inside view (queues, workers, ack, retries)

```text
                       (TOPIC)
Publisher --->  +-------------------+
 publish(msg)   |  Broker / PubSub  |
                | - stores message  |
                | - routes copies   |
                +-------------------+
                      | creates a copy per subscription
          +-----------+--------------------+
          |                                |
          v                                v
  +------------------+              +------------------+
  | Subscription: B  |              | Subscription: C  |
  | (Queue/Backlog)  |              | (Queue/Backlog)  |
  +------------------+              +------------------+
          | pull/push                      | pull/push
          v                                v
   +---------------+                +---------------+
   | Workers (B)   |                | Workers (C)   |
   |  B1  B2  B3   |                |   C1  C2      |
   +---------------+                +---------------+
          | process ok                      | process fails
          |                                 |
          v                                 v
      ACK to broker                    NACK / no-ACK
          |                                 |
          v                                 v
   message removed                   broker retries later
                                      (after delay / deadline)
                                            |
                                            v
                                   Dead-Letter Queue (optional)

```

##### What’s being “managed” inside (important):

- **Topic**: a named channel (like “orders.created”).
- **Subscription**: each subscriber gets its own delivery stream (like its own inbox/queue).
- **Backlog/Queue**: if subscribers are slow, messages wait here (buffering).
- **Workers**: multiple instances can process messages in parallel (scaling).
- **ACK**: subscriber says “done ✅” so broker removes the message from that subscription.
- **Retry**: if no ACK, broker sends again (at-least-once delivery in many systems).
- **Dead-letter**: after too many failures, message goes to a special place for debugging.
  
###### Why it’s called **“asynchronous”**

- Publisher sends message and doesn’t wait for subscribers to finish.
- Publisher stays fast.
- Subscribers can take their time.

###### Real-world example (very relatable)

- “Order placed” in an e-commerce app:
- Publisher: Order Service publishes order.created
**Subscribers:**
- `Email Service`sends confirmation email
- `Inventory Service` reduces stock
- `Analytics Service` records metrics
- `Shipping Service` prepares shipping label
- Publisher does **one publish**, and many services react independently.

###### Two common ways subscribers receive messages

**Pull**: subscriber asks broker “give me next message” (good control).
**Push**: broker pushes to subscriber endpoint (like webhook delivery).

##### Components (kid-simple + real meaning)

1) Publisher

- **Kid version**: The person who writes an announcement.
- **System version**: Any service that sends an event/message (Order service, Payment service, etc.)
- Publisher only knows: **topic name.**
  
**1) Subscriber**:

- **Kid version**: People who signed up to get certain announcements.
- **System version**: Any service that receives messages from a topic via a subscription (Email service, Inventory service…).
- Subscriber decides what it cares about.

**3) Topic**:

- **Kid version:** A labeled bucket like “Homework”, “Sports”, “Fees”.
- **System version:** A named channel where publishers post messages. Example: orders.created.
- Topic does **not** mean “one subscriber”. Many subscribers can subscribe.
**4) Message**
- **Kid version:** The actual note/announcement paper.
- **System version:** The data payload (JSON, bytes) + metadata:
   -  Example payload: { orderId: "123", total: 499 }
   -  Metadata: timestamp, id, attributes, etc.

**5) Broker** :

- **Kid version:** The school office that receives announcements and sends copies to the right classes.
- **System version:** The Pub/Sub system itself (Kafka / RabbitMQ / Google Pub/Sub / Redis PubSub).

**Responsibilities usually include**:

- Keep topics
- Keep subscriptions
- Store/buffer messages (depending on system)
**Deliver to subscribers**
Track ACK / retries (again depends on system)
**6) Routing**
- Kid version: The office decides which class gets which announcement.
- System version: Broker checks:

Which subscribers are subscribed to this topic?
Any filters? (ex: only country=IN)
Then it delivers message to the correct subscriptions.

```text
        (1) Publisher
             |
             | publish(message) to Topic "T"
             v
      +-------------------+
      |   (5) Broker      |
      |  Topics + Subs    |
      +-------------------+
             |
      (6) routing/fan-out
     /           |          \
    v            v           v
(2) Sub A     (2) Sub B    (2) Sub C

```

# Redis (Engineering-First) — Practical Guide + Command Cheat Sheet

> Goal: help you *use Redis confidently* (commands like `HSET`) **and** understand what’s happening behind the scenes.

---

## 0) What Redis is

Redis is an **in-memory data structure server**:
- Your data lives mainly in **RAM** (fast).
- You talk to it over **TCP** using the **RESP** protocol.
- It supports multiple **data types** (strings, hashes, lists, sets, sorted sets, streams…).
- It can persist to disk (RDB snapshots / AOF logs) and replicate/cluster.

**Core model:** most commands are **very fast** because they avoid disk I/O in the request path.

---

## 1) Installation quick start

### Run with Docker (recommended)
```bash
docker run --name redis -p 6379:6379 -d redis:latest
docker exec -it redis redis-cli
```

### Verify
```redis
PING
# -> PONG
```

---

## 2) Key basics

### Key naming (common pattern)
- `user:1`
- `user:1:profile`
- `session:abc123`
- `rate:ip:1.2.3.4`

### Check existence and type
```redis
EXISTS key
TYPE key
```

### Delete
```redis
DEL key
UNLINK key         # async delete (good for big keys)
```

### Rename
```redis
RENAME old new
RENAMENX old new   # only if new doesn't exist
```

### TTL (time-to-live)
```redis
EXPIRE key 60       # seconds
PEXPIRE key 1500    # milliseconds
TTL key             # seconds remaining (-1 no TTL, -2 missing)
PTTL key            # ms remaining
PERSIST key         # remove TTL
```

---

## 3) Data Types + Commands

### 3.1 Strings (the default)
**Use when:** counters, tokens, small JSON blobs, cached HTML, IDs.

```redis
SET k "v"
GET k
SETNX k "v"                  # set if not exists
MSET k1 v1 k2 v2
MGET k1 k2

INCR counter
INCRBY counter 10
DECR counter
APPEND k "more"

SETEX k 60 "v"               # set + expire
PSETEX k 1500 "v"            # set + expire ms
```

**Atomic update with conditions**
```redis
SET lock:job123 "1" NX EX 10  # acquire lock if not exists (10s)
```

---

### 3.2 Hashes (HSET, HGET…) ✅ (most like a “row”)
**Use when:** many fields under one key (user profile), partial updates.

```redis
HSET user:1 name "Suraj" age "23" city "Hyd"
HGET user:1 name
HMGET user:1 name age
HGETALL user:1
HDEL user:1 age
HLEN user:1
HEXISTS user:1 name
HINCRBY user:1 loginCount 1
HKEYS user:1
HVALS user:1
```

**Pattern: store user profile**
```redis
HSET user:1 name "Suraj" email "a@b.com" role "admin"
EXPIRE user:1 3600
```

---

### 3.3 Lists (LPUSH/RPUSH…)
**Use when:** queues (simple), recent items, ordered history.

```redis
LPUSH jobs "j1"
LPUSH jobs "j2"
LRANGE jobs 0 -1

RPUSH jobs "j3"
LPOP jobs
RPOP jobs
LLEN jobs

# Blocking pop (worker waits)
BLPOP jobs 0      # 0 = wait forever
BRPOP jobs 5      # wait up to 5 seconds
```

---

### 3.4 Sets (SADD…)
**Use when:** unique items, membership checks, tags.

```redis
SADD tags "node" "redis" "backend"
SMEMBERS tags
SISMEMBER tags "redis"
SREM tags "node"
SCARD tags

# Set operations
SADD a 1 2 3
SADD b 3 4 5
SINTER a b     # intersection -> 3
SUNION a b
SDIFF a b      # a - b -> 1 2
```

---

### 3.5 Sorted Sets (ZSET) (ZADD…)
**Use when:** leaderboards, ranking, time-ordered scoring.

```redis
ZADD leaderboard 100 "suraj" 80 "arun" 120 "meena"
ZRANGE leaderboard 0 -1 WITHSCORES
ZREVRANGE leaderboard 0 -1 WITHSCORES
ZSCORE leaderboard "suraj"
ZRANK leaderboard "suraj"
ZREVRANK leaderboard "suraj"
ZREM leaderboard "arun"
```

**Pattern: “top N”**
```redis
ZREVRANGE leaderboard 0 9 WITHSCORES
```

---

### 3.6 Streams (XADD…) (recommended for reliable queues)
**Use when:** message queues with consumer groups, retries.

```redis
XADD mystream * type "email" to "user@x.com"
XRANGE mystream - + COUNT 10
XREVRANGE mystream + - COUNT 10
```

**Consumer group (workers)**
```redis
XGROUP CREATE mystream mygroup $ MKSTREAM
XREADGROUP GROUP mygroup worker-1 COUNT 10 BLOCK 5000 STREAMS mystream >
XACK mystream mygroup <message-id>
XPENDING mystream mygroup
XCLAIM mystream mygroup worker-2 60000 <message-id>
```

---

### 3.7 Pub/Sub (PUBLISH/SUBSCRIBE)
**Use when:** live notifications, chat, realtime updates  
**Not durable** (offline subscribers miss messages).

```redis
SUBSCRIBE events
PUBLISH events "hello"
```

---

## 4) Transactions, Pipelines, Lua

### 4.1 Transactions (MULTI/EXEC)
```redis
MULTI
INCR balance
INCRBY points 10
EXEC
```

### 4.2 Optimistic locking (WATCH)
```redis
WATCH balance
MULTI
DECRBY balance 50
EXEC          # fails if balance changed after WATCH
```

### 4.3 Pipelining (client-side)
Pipeline = send many commands in one network round-trip.  
Most SDKs support `.pipeline()` / `.multi()` batching.

---

## 5) Scanning keys safely (avoid KEYS in production)

❌ Don’t do this on big DB:
```redis
KEYS user:*
```

✅ Use SCAN:
```redis
SCAN 0 MATCH user:* COUNT 100
```

---

## 6) Memory + eviction (cache behavior)

```redis
INFO memory
MEMORY STATS
```

Eviction policies (configured on server):
- `noeviction`
- `allkeys-lru`
- `allkeys-lfu`
- `volatile-lru`
- `volatile-ttl`

---

## 7) Persistence (how Redis survives restarts)

### RDB snapshots
```redis
BGSAVE
LASTSAVE
```

### AOF (append-only file)
```redis
BGREWRITEAOF
```

---

## 8) Replication & HA basics
```redis
INFO replication
```

---

## 9) Cluster (sharding) concept

Redis Cluster shards keys across nodes (hash slots).  
Multi-key ops need keys in same slot using hash tags:

- `user:{1}:profile`
- `user:{1}:settings`

---

## 10) Performance + debugging

```redis
SLOWLOG GET 10
INFO
CLIENT LIST
```

**Note:** one slow command can block other clients (in the main execution thread model).

---

## 11) Security (minimum)

- Run Redis in a private network.
- Use ACLs:
```redis
ACL LIST
ACL SETUSER appuser on >StrongPassword ~* +@all
```

---

## 12) Quick command index (cheat sheet)

**Keys:** `EXISTS TYPE DEL UNLINK TTL EXPIRE PERSIST SCAN`  
**Strings:** `SET GET MSET MGET INCR DECR SETNX SETEX`  
**Hashes:** `HSET HGET HMGET HGETALL HDEL HLEN HEXISTS HINCRBY`  
**Lists:** `LPUSH RPUSH LRANGE LPOP RPOP BLPOP BRPOP LLEN`  
**Sets:** `SADD SMEMBERS SISMEMBER SREM SCARD SINTER SUNION SDIFF`  
**ZSets:** `ZADD ZRANGE ZREVRANGE ZSCORE ZRANK ZREVRANK ZREM`  
**Streams:** `XADD XRANGE XREAD XGROUP XREADGROUP XACK XPENDING XCLAIM`  
**Pub/Sub:** `SUBSCRIBE PUBLISH`  
**Transactions:** `MULTI EXEC WATCH UNWATCH`  
**Ops:** `INFO SLOWLOG CLIENT LIST MEMORY STATS`

---

## Next step (tell me and I’ll tailor it)
Tell me your use case (cache / session / queue / leaderboard / rate limit), and I’ll add a **Node.js + ioredis mini-project** section.

# Node.js Practical Interview Prep (3+ Years) — Real-World Scenarios + Code

This is **scenario-based** (what you actually build in a job): server, middleware, auth, RBAC, uploads, WebSockets, Redis cache & invalidation, BullMQ background jobs, logging, global error handling, graceful shutdown, and performance/latency detection.

---

## 0) Recommended folder structure (clean code)

```
src/
  app.js
  server.js
  routes/
  controllers/
  middleware/
    asyncHandler.js
    error.js
    auth.js
    rbac.js
    validate.js
  services/
  cache/redis.js
  jobs/
    queues.js
    workers.js
  realtime/socketio.js
  observability/
    logger.js
    metrics.js
```

**Clean-code rule** (interview gold):

- **Routes**: mapping URLs → handlers
- **Controllers**: HTTP concerns only (req/res)
- **Services**: business logic
- **DB/Repo layer**: queries only
- **Middleware**: cross-cutting (auth, validation, logging)
- **Workers**: background tasks (BullMQ)

---

# 1) Create a server (Express) + middleware

## 1.1 Minimal server

```js
// src/app.js
import express from "express";
export const app = express();

app.use(express.json());

app.get("/health", (req, res) => {
  res.json({ ok: true });
});

```

```js
// src/server.js
import http from "node:http";
import { app } from "./app.js";

const server = http.createServer(app);
const PORT = process.env.PORT || 3000;

server.listen(PORT, () => console.log("Listening on", PORT));
```

## 1.2 What is middleware?

Middleware runs **before** route handlers (and error middleware runs after).

Signature:
```js
(req, res, next) => { next(); }
```

Real uses:

- Auth (`requireAuth`)
- RBAC (`allowRoles`)
- Validation (`validate(schema)`)
- Logging (`requestId + latency`)
- Rate-limiting
- Error handling

Example middleware (latency + basic log):

```js
app.use((req, res, next) => {
  const start = Date.now();
  res.on("finish", () => {
    console.log(req.method, req.originalUrl, res.statusCode, Date.now() - start, "ms");
  });
  next();
});
```

---

# 2) Global error handling + remove try/catch pollution

## 2.1 Why try/catch everywhere is ugly?
In Express, async errors don’t automatically go to error middleware unless you `next(err)`.

## 2.2 Fix: asyncHandler wrapper

```js
// src/middleware/asyncHandler.js
export const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};
```

Use it like:
```js
router.get("/users/:id", asyncHandler(async (req, res) => {
  const user = await userService.getById(req.params.id);
  res.json(user);
}));
```

## 2.3 AppError + error middleware

```js
// src/middleware/error.js
export class AppError extends Error {
  constructor(message, statusCode = 500, details = undefined) {
    super(message);
    this.statusCode = statusCode;
    this.details = details;
  }
}

export function errorHandler(err, req, res, next) {
  const status = err.statusCode || 500;
  res.status(status).json({
    error: err.message || "Internal Server Error",
    details: err.details,
  });
}
```

Wire it at the end:
```js
app.use(errorHandler);
```

**Interview line:** “I avoid try/catch in every controller by using asyncHandler + global error middleware.”

---

# 3) Params vs Query vs Body (real examples)

- **Params** = resource identity: `/users/:id` → `req.params.id`
- **Query** = filters/pagination: `/users?role=admin&page=2` → `req.query.page`
- **Body** = create/update payload: POST/PATCH JSON → `req.body`

```js
router.get("/users/:id", (req, res) => {
  console.log("id:", req.params.id);
  console.log("verbose:", req.query.verbose);
  res.json({ ok: true });
});
```

---

# 4) Authentication vs Authorization + RBAC

## 4.1 AuthN vs AuthZ
- **Authentication (AuthN)**: who are you? (login/token)
- **Authorization (AuthZ)**: what can you do? (permissions)

## 4.2 JWT auth middleware (simple)

```js
// src/middleware/auth.js
import jwt from "jsonwebtoken";

export function requireAuth(req, res, next) {
  const header = req.headers.authorization;
  if (!header?.startsWith("Bearer ")) {
    return res.status(401).json({ error: "Missing token" });
  }

  const token = header.slice("Bearer ".length);
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: "Invalid token" });
  }
}
```

## 4.3 RBAC middleware

```js
// src/middleware/rbac.js
export const allowRoles = (...roles) => (req, res, next) => {
  if (!req.user) return res.status(401).json({ error: "Unauthenticated" });
  if (!roles.includes(req.user.role)) return res.status(403).json({ error: "Forbidden" });
  next();
};
```

Usage:
```js
router.delete("/users/:id",
  requireAuth,
  allowRoles("admin"),
  asyncHandler(async (req, res) => {
    await userService.remove(req.params.id);
    res.status(204).send();
  })
);
```

**RBAC vs ABAC:**
- RBAC: role-based (admin/editor)
- ABAC: attribute-based (user owns the resource, orgId matches, etc.)

---

# 5) Validation middleware (Zod example)

```js
// src/middleware/validate.js
export const validate = (schema) => (req, res, next) => {
  const result = schema.safeParse({
    params: req.params,
    query: req.query,
    body: req.body,
  });
  if (!result.success) {
    return res.status(400).json({ error: result.error.flatten() });
  }
  req.validated = result.data;
  next();
};
```

---

# 6) Upload image: multipart/form-data

## 6.1 What is multipart/form-data?
It’s the encoding browsers use to send **files + fields**. The request body is split into “parts” with boundaries.

**Rule:** don’t buffer huge uploads in RAM. Stream them.

## 6.2 Simple upload (multer)
Good for small/mid apps; for big scale use S3 direct upload (next).

```js
import multer from "multer";
const upload = multer({
  dest: "uploads/",
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
});

router.post("/upload",
  requireAuth,
  upload.single("image"),
  (req, res) => {
    res.json({
      storedAs: req.file.filename,
      originalName: req.file.originalname,
      size: req.file.size,
    });
  }
);
```

## 6.3 Best practice at scale: direct upload to S3 (strategy)
- API returns a **pre-signed URL** with constraints
- Client uploads directly to S3
- Node does not carry that bandwidth/memory load
- Worker processes file (thumbnails/scan/transcode)

**Interview line:** “For large files, I use presigned URLs + background processing. Node stays light.”

---

# 7) WebSocket vs Socket.IO (and how to build)

## 7.1 Raw WebSocket (`ws`)
```js
import http from "node:http";
import express from "express";
import { WebSocketServer } from "ws";

const app = express();
const server = http.createServer(app);
const wss = new WebSocketServer({ server });

wss.on("connection", (socket) => {
  socket.send(JSON.stringify({ type: "hello", ts: Date.now() }));

  socket.on("message", (msg) => {
    // msg is Buffer by default
    socket.send(msg); // echo
  });
});

server.listen(3000);
```

## 7.2 What Socket.IO adds
- auto reconnect
- rooms
- fallbacks
- acknowledgements

Tradeoff: more overhead and not a pure WS protocol.

## 7.3 Socket.IO server
```js
import http from "node:http";
import express from "express";
import { Server } from "socket.io";

const app = express();
const server = http.createServer(app);
const io = new Server(server, { cors: { origin: "*" } });

io.on("connection", (socket) => {
  socket.join("room:global");

  socket.on("chat:msg", (data) => {
    io.to("room:global").emit("chat:msg", data);
  });
});

server.listen(3000);
```

---

# 8) Redis: connect + cache + keep data “latest”

## 8.1 Connect Redis (ioredis)
```js
// src/cache/redis.js
import Redis from "ioredis";
export const redis = new Redis(process.env.REDIS_URL);

redis.on("error", (e) => console.error("Redis error", e));
```

## 8.2 Cache-aside (most common)
```js
import { redis } from "../cache/redis.js";

export async function getUserCached(userId) {
  const key = `user:${userId}`;
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  const user = await db.users.findById(userId);

  // TTL prevents forever-stale cache
  await redis.set(key, JSON.stringify(user), "EX", 60);
  return user;
}
```

## 8.3 “How do you make sure cache reflects latest DB updates?”

There’s no perfect default. Choose based on correctness needs:

1) **TTL only** (simple): may be stale until TTL expires  
2) **Invalidate on write** (common): delete cache key after update  
3) **Pub/Sub invalidation** (multi-instance): broadcast invalidation  
4) **CDC/outbox** (advanced): DB changes → message bus → cache refresh  
5) **Versioned keys**: change version to avoid serving old cache

### 8.3.1 Invalidate on write
```js
export async function updateUser(userId, patch) {
  const updated = await db.users.update(userId, patch);
  await redis.del(`user:${userId}`);
  return updated;
}
```

### 8.3.2 Pub/Sub invalidation (for many Node instances)
```js
// publish after write
await redis.publish("cache:invalidate", `user:${userId}`);
```

```js
// each instance subscribes
import Redis from "ioredis";
import { redis } from "./redis.js";

const sub = new Redis(process.env.REDIS_URL);
sub.subscribe("cache:invalidate");

sub.on("message", async (channel, key) => {
  if (channel === "cache:invalidate") {
    await redis.del(key);
  }
});
```

**Important:** Redis pub/sub is ephemeral. If a node is down, it misses messages → keep TTL anyway.

---

# 9) BullMQ for background jobs (producer + worker)

## 9.1 Why BullMQ?
- Don’t block request latency for slow work
- retries with backoff
- concurrency control
- scheduled/delayed jobs

## 9.2 Producer
```js
// src/jobs/queues.js
import { Queue } from "bullmq";

export const emailQueue = new Queue("email", {
  connection: { url: process.env.REDIS_URL },
});

export async function enqueueWelcomeEmail(userId) {
  await emailQueue.add(
    "welcome",
    { userId },
    {
      attempts: 5,
      backoff: { type: "exponential", delay: 1000 },
      removeOnComplete: 1000,
      removeOnFail: 5000,
    }
  );
}
```

## 9.3 Worker
```js
// src/jobs/workers.js
import { Worker } from "bullmq";

new Worker(
  "email",
  async (job) => {
    const { userId } = job.data;
    await sendEmail(userId);
  },
  {
    connection: { url: process.env.REDIS_URL },
    concurrency: 10,
  }
);
```

**Must-know:** jobs must be **idempotent** (retry can run twice).

---

# 10) Logging (what package + how it works)

## 10.1 Pino logger
```js
// src/observability/logger.js
import pino from "pino";
export const logger = pino({ level: process.env.LOG_LEVEL || "info" });
```

## 10.2 Request ID + slow log
```js
import crypto from "node:crypto";
import { logger } from "./observability/logger.js";

app.use((req, res, next) => {
  req.requestId = req.headers["x-request-id"] || crypto.randomUUID();
  res.setHeader("x-request-id", req.requestId);

  const start = process.hrtime.bigint();
  res.on("finish", () => {
    const ms = Number(process.hrtime.bigint() - start) / 1e6;
    const log = { requestId: req.requestId, method: req.method, url: req.originalUrl, status: res.statusCode, ms };
    if (ms > 500) logger.warn(log, "slow_request");
    else logger.info(log, "request");
  });

  next();
});
```

---

# 11) “How do you make sure server doesn’t crash?”

## 11.1 App-level
- Validate inputs
- Use asyncHandler + error middleware
- Add timeouts for outbound calls
- Protect with circuit breaker if needed
- Avoid CPU-heavy work on event loop

## 11.2 Process-level (last defense)
In production, for unknown state: **log + exit + restart** with PM2/K8s.

```js
process.on("uncaughtException", (err) => {
  console.error("uncaughtException", err);
  process.exit(1);
});

process.on("unhandledRejection", (err) => {
  console.error("unhandledRejection", err);
  process.exit(1);
});
```

---

# 12) PM2: use all cores

Node uses 1 core per process for JS. Use PM2 cluster mode:

```bash
pm2 start src/server.js -i max --name my-api
pm2 status
pm2 logs my-api
```

**Interview line:** “I scale Node across cores with PM2 cluster/K8s replicas; keep the app stateless.”

---

# 13) Big image/video processing (real approach)

**Never do this inside request handler.**

Strategy:
1) Upload file (prefer direct-to-object-storage)
2) Enqueue BullMQ job
3) Worker runs `sharp` (images) or `ffmpeg` (videos)
4) Store outputs + update DB status
5) Client polls or uses WebSocket/SSE for status updates

---

# 14) import/export vs require (ESM setup)

## 14.1 Why ESM?
- Standard modules
- better tooling/static analysis
- many libs are ESM-first

## 14.2 Enable ESM
In `package.json`:
```json
{ "type": "module" }
```

Then:
```js
import express from "express";
export const app = express();
```

Alternative: use `.mjs` extension.

---

# 15) Graceful shutdown (SIGTERM) — production pattern

```js
// src/server.js
import http from "node:http";
import { app } from "./app.js";

const server = http.createServer(app);
server.listen(3000);

let shuttingDown = false;

async function shutdown(signal) {
  if (shuttingDown) return;
  shuttingDown = true;
  console.warn("shutdown_start", signal);

  // stop accepting new connections
  server.close(() => console.warn("http_server_closed"));

  // close DB/Redis here (pseudo)
  // await db.close(); await redis.quit();

  // hard timeout to exit
  setTimeout(() => {
    console.warn("shutdown_force_exit");
    process.exit(0);
  }, 10_000).unref();
}

process.on("SIGTERM", () => shutdown("SIGTERM"));
process.on("SIGINT", () => shutdown("SIGINT"));
```

---

# 16) Find slow APIs + measure server latency

## 16.1 Simple: slow log threshold
Already shown above (warn if > 500ms).

## 16.2 Real: Prometheus metrics (route p95/p99)
Use `prom-client` histogram by route + status, expose `/metrics`, view in Grafana.

What you learn:
- which route is slow at p99
- whether slow is DB, upstream, CPU
- concurrency vs latency pattern

## 16.3 Tracing (advanced)
Add OpenTelemetry. You see slow spans (DB query, HTTP call) inside one request.

---

# 17) Concurrency vs latency (practical)

- **Concurrency**: number of requests in-flight right now
- **Latency**: time per request
- Under high concurrency, latency rises due to queueing (DB pool waiting, event loop delay)

Quick active requests counter:
```js
let active = 0;
app.use((req, res, next) => {
  active++;
  res.on("finish", () => active--);
  next();
});
app.get("/debug/active", (req, res) => res.json({ active }));
```

Also monitor:
- DB pool “acquire time”
- event loop delay
- threadpool usage (fs/crypto/zlib workloads)

---

# 18) Practice mini-projects (do these for real skill)

## Project A: Secure REST API
- Register/Login (bcrypt + JWT)
- `GET /users/me`
- RBAC: admin can list users
- Zod validation
- asyncHandler + global error handler

## Project B: Upload + processing
- Upload image (multipart)
- Store in S3/local
- BullMQ job creates thumbnail
- Worker updates DB status

## Project C: Redis caching
- Cache GET `/users/:id`
- Invalidate on PATCH `/users/:id`
- Add pub/sub invalidation for multi-instance

## Project D: Realtime
- Socket.IO chat or notifications
- Add rooms
- Add Redis pub/sub for broadcasting across instances

## Project E: Observability
- requestId + pino logs
- slow logs
- metrics endpoint + dashboard

---

# 19) Interview scenario answers (short but correct)

### “If DB updates, how do you reflect latest in cache?”
Invalidate on write + TTL, and pub/sub invalidation for multi-instance. For stronger correctness, use CDC/outbox patterns.

### “How do you ensure one error doesn’t crash everything?”
asyncHandler + error middleware for request errors; process manager restarts for unknown fatal errors; timeouts/circuit breakers for dependencies.

### “How do you find slow endpoints in a large codebase?”
Per-route metrics (p95/p99), slow logs with requestId, traces for DB/upstream spans, and event loop delay for CPU blocks.

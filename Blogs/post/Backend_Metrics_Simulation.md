# Backend Metrics Simulation

_Exported 2026-06-12T19:06:23.131Z — 33 messages_

---

## 🧑 You

role :- you are developer backend architect who simulate  of logging and metric 
context:-we are gone use express.js , node .js , redis, 
1. we are here to lern here matrix like cpu load , memero load  and other type of metrix such as reuques respose time and all 
2. we needto simulate high load liek send 1000 reuest how it hanlfde need to see how latency inrease and all 
 need 10 fake api wich shoe latency  so e can see bottlneak and all 
task 
1. learnig how trxi sis shown on prod 
2. all we are gone use took which is useed in prod 
see metrix like letency , throput , cpu load , memory load liek upload chuks vs in strems ,ehrn image upload see spike in memey consyption 
 output 
must be have different 10 api which well follweed modular monolutth ready to test code 
2. all depecy code and well dockerized code 
3. used pino but make sure we gone as much as near ral bottlneak of susyem we see 
5. you gave me code onlu no dogram no theory

---

## 🤖 ChatGPT

```txt
project-root/
├── apps/
│   └── api/
│       ├── src/
│       │   ├── modules/
│       │   │   ├── health/
│       │   │   ├── cpu-spike/
│       │   │   ├── memory-leak/
│       │   │   ├── image-buffer-upload/
│       │   │   ├── image-stream-upload/
│       │   │   ├── redis-read/
│       │   │   ├── redis-write/
│       │   │   ├── db-heavy/
│       │   │   ├── event-loop-block/
│       │   │   ├── external-api/
│       │   │   └── batch-processing/
│       │   │
│       │   ├── middleware/
│       │   │   ├── logger.js
│       │   │   ├── metrics.js
│       │   │   └── request-id.js
│       │   │
│       │   ├── metrics/
│       │   │   └── prometheus.js
│       │   │
│       │   ├── redis/
│       │   │   └── redis.js
│       │   │
│       │   ├── app.js
│       │   └── server.js
│       │
│       ├── package.json
│       └── Dockerfile
│
├── monitoring/
│   ├── prometheus.yml
│   └── grafana/
│
├── docker-compose.yml
└── k6-load-test.js
```

---

# package.json

```json
{
  "name": "metrics-lab",
  "version": "1.0.0",
  "scripts": {
    "start": "node src/server.js"
  },
  "dependencies": {
    "axios": "^1.8.1",
    "express": "^5.1.0",
    "ioredis": "^5.6.1",
    "multer": "^2.0.2",
    "pino": "^9.7.0",
    "pino-http": "^10.5.0",
    "prom-client": "^15.1.3",
    "uuid": "^11.1.0"
  }
}
```

---

# redis.js

```js
const Redis = require("ioredis");

module.exports = new Redis({
  host: "redis",
  port: 6379
});
```

---

# logger.js

```js
const pino = require("pino");

module.exports = pino({
  level: "info",
  transport: {
    target: "pino-pretty"
  }
});
```

---

# request-id.js

```js
const { v4: uuid } = require("uuid");

module.exports = (req, res, next) => {
  req.requestId = uuid();
  next();
};
```

---

# prometheus.js

```js
const client = require("prom-client");

client.collectDefaultMetrics();

const httpDuration = new client.Histogram({
  name: "http_request_duration_seconds",
  help: "Duration",
  labelNames: ["method", "route", "status"]
});

const totalRequests = new client.Counter({
  name: "http_requests_total",
  help: "Total Requests"
});

module.exports = {
  client,
  httpDuration,
  totalRequests
};
```

---

# metrics.js

```js
const {
  httpDuration,
  totalRequests
} = require("../metrics/prometheus");

module.exports = (req, res, next) => {

  const start = Date.now();

  res.on("finish", () => {

    const duration = (Date.now() - start) / 1000;

    httpDuration.observe(
      {
        method: req.method,
        route: req.path,
        status: res.statusCode
      },
      duration
    );

    totalRequests.inc();
  });

  next();
};
```

---

# health route

```js
const router = require("express").Router();

router.get("/", (req, res) => {
  res.json({
    status: "UP"
  });
});

module.exports = router;
```

---

# cpu-spike route

```js
const router = require("express").Router();

router.get("/", async (req, res) => {

  let total = 0;

  for (let i = 0; i < 500000000; i++) {
    total += i;
  }

  res.json({
    total
  });
});

module.exports = router;
```

---

# memory-leak route

```js
const router = require("express").Router();

const memoryStore = [];

router.get("/", (req, res) => {

  const data = Buffer.alloc(50 * 1024 * 1024);

  memoryStore.push(data);

  res.json({
    allocated: "50MB",
    count: memoryStore.length
  });
});

module.exports = router;
```

---

# image-buffer-upload

```js
const router = require("express").Router();
const multer = require("multer");

const upload = multer({
  storage: multer.memoryStorage()
});

router.post("/", upload.single("file"), (req, res) => {

  const size = req.file.buffer.length;

  res.json({
    size
  });
});

module.exports = router;
```

---

# image-stream-upload

```js
const router = require("express").Router();
const fs = require("fs");

router.post("/", (req, res) => {

  const file = fs.createWriteStream(
    `/tmp/${Date.now()}.bin`
  );

  req.pipe(file);

  file.on("finish", () => {
    res.json({
      success: true
    });
  });
});

module.exports = router;
```

---

# redis-write

```js
const router = require("express").Router();
const redis = require("../../redis/redis");

router.get("/", async (req, res) => {

  await redis.set(
    Date.now(),
    JSON.stringify({
      name: "suraj"
    })
  );

  res.json({
    success: true
  });
});

module.exports = router;
```

---

# redis-read

```js
const router = require("express").Router();
const redis = require("../../redis/redis");

router.get("/", async (req, res) => {

  const data = await redis.keys("*");

  res.json({
    count: data.length
  });
});

module.exports = router;
```

---

# db-heavy

```js
const router = require("express").Router();

router.get("/", async (req, res) => {

  await new Promise(resolve =>
    setTimeout(resolve, 3000)
  );

  res.json({
    db: "heavy query"
  });
});

module.exports = router;
```

---

# event-loop-block

```js
const router = require("express").Router();

router.get("/", (req, res) => {

  const end = Date.now() + 5000;

  while (Date.now() < end) {}

  res.json({
    blocked: true
  });
});

module.exports = router;
```

---

# external-api

```js
const router = require("express").Router();
const axios = require("axios");

router.get("/", async (req, res) => {

  const data = await axios.get(
    "https://jsonplaceholder.typicode.com/posts"
  );

  res.json({
    count: data.data.length
  });
});

module.exports = router;
```

---

# batch-processing

```js
const router = require("express").Router();

router.get("/", async (req, res) => {

  const jobs = [];

  for (let i = 0; i < 5000; i++) {
    jobs.push(i);
  }

  await Promise.all(
    jobs.map(async item => item)
  );

  res.json({
    processed: jobs.length
  });
});

module.exports = router;
```

---

# app.js

```js
const express = require("express");

const metrics = require("./middleware/metrics");
const requestId = require("./middleware/request-id");

const app = express();

app.use(express.json());
app.use(requestId);
app.use(metrics);

app.use("/health", require("./modules/health"));
app.use("/cpu-spike", require("./modules/cpu-spike"));
app.use("/memory-leak", require("./modules/memory-leak"));
app.use("/image-buffer-upload", require("./modules/image-buffer-upload"));
app.use("/image-stream-upload", require("./modules/image-stream-upload"));
app.use("/redis-read", require("./modules/redis-read"));
app.use("/redis-write", require("./modules/redis-write"));
app.use("/db-heavy", require("./modules/db-heavy"));
app.use("/event-loop-block", require("./modules/event-loop-block"));
app.use("/external-api", require("./modules/external-api"));
app.use("/batch-processing", require("./modules/batch-processing"));

module.exports = app;
```

---

# server.js

```js
const app = require("./app");
const { client } = require("./metrics/prometheus");

app.get("/metrics", async (req, res) => {
  res.set(
    "Content-Type",
    client.register.contentType
  );

  res.end(await client.register.metrics());
});

app.listen(3000, () => {
  console.log("running");
});
```

---

# Dockerfile

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

CMD ["npm","start"]
```

---

# prometheus.yml

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: node-app

    static_configs:
      - targets:
          - api:3000
```

---

# docker-compose.yml

```yaml
services:

  api:
    build:
      context: ./apps/api

    ports:
      - "3000:3000"

    depends_on:
      - redis

  redis:
    image: redis:8-alpine

  prometheus:
    image: prom/prometheus

    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml

    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana

    ports:
      - "3001:3000"

    depends_on:
      - prometheus
```

---

# k6-load-test.js

```js
import http from "k6/http";

export const options = {
  vus: 1000,
  duration: "60s"
};

export default function () {

  http.get(
    "http://host.docker.internal:3000/cpu-spike"
  );

  http.get(
    "http://host.docker.internal:3000/db-heavy"
  );

  http.get(
    "http://host.docker.internal:3000/redis-read"
  );

  http.get(
    "http://host.docker.internal:3000/event-loop-block"
  );
}
```

---

```bash
docker compose up -d

docker stats

curl localhost:3000/metrics

k6 run k6-load-test.js
```

---

```txt
APIs TO TEST

1. /health
2. /cpu-spike
3. /memory-leak
4. /image-buffer-upload
5. /image-stream-upload
6. /redis-read
7. /redis-write
8. /db-heavy
9. /event-loop-block
10. /batch-processing
11. /external-api
12. /metrics
```

```txt
PROD METRICS YOU WILL SEE

CPU Usage %
Memory Usage
Heap Usage
Event Loop Lag
GC Activity
Latency P50
Latency P95
Latency P99
Throughput RPS
Error Rate
Redis Latency
Container CPU
Container Memory
Network In
Network Out
Upload Memory Spike
Stream Upload Memory
Request Count
Active Connections
```

```bash
docker stats
```

```txt
Real bottlenecks generated

CPU Saturation
Memory Leak
Event Loop Blocking
Slow Database
Slow Redis
Large Upload Buffer
External API Delay
Batch Processing Pressure
High Concurrency
Container Resource Exhaustion
```

---

## 🤖 ChatGPT

Good. This is exactly the type of raw Prometheus output you should learn to read before Grafana dashboards. The metrics show your application is currently healthy and mostly idle. fileciteturn0file0

# CPU

```txt
process_cpu_user_seconds_total 2.14
process_cpu_system_seconds_total 0.66
```

Meaning:

```txt
User CPU Time   = 2.14 sec
Kernel CPU Time = 0.66 sec
Total CPU Time  = 2.80 sec
```

Current observation:

```txt
No CPU bottleneck
No heavy computation
```

When hitting:

```txt
/cpu-spike
/event-loop-block
```

you should see:

```txt
process_cpu_seconds_total ↑
nodejs_eventloop_lag ↑
latency ↑
```

---

# Memory

```txt
process_resident_memory_bytes 82669568
```

Convert:

```txt
82,669,568 / 1024 / 1024
≈ 78 MB
```

Current memory:

```txt
~78 MB RAM
```

---

# V8 Heap

```txt
nodejs_heap_size_used_bytes 14739288
```

Convert:

```txt
14 MB used heap
```

Current:

```txt
Heap Used = 14 MB
Heap Total = 16 MB
```

Healthy.

---

# Event Loop Lag

Most important Node.js metric.

```txt
nodejs_eventloop_lag_p50_seconds
0.010
```

```txt
10 ms
```

P99

```txt
0.011
```

```txt
11 ms
```

Excellent.

Production target:

```txt
<20ms = good
20-50ms = warning
>100ms = problem
>500ms = serious issue
```

Current:

```txt
Healthy
```

---

# Active Connections

```txt
nodejs_active_handles_total 6
```

Meaning:

```txt
Sockets
Server handles
Open connections
```

Current:

```txt
6 handles
```

Very low.

---

# Active Requests

```txt
nodejs_active_requests_total 0
```

Meaning:

```txt
No pending I/O
No blocked operations
```

---

# GC Metrics

```txt
nodejs_gc_duration_seconds
```

Minor GC:

```txt
22 collections
23 ms total
```

Major GC:

```txt
3 collections
24 ms total
```

Current:

```txt
GC healthy
No memory pressure
```

When testing:

```txt
/memory-leak
/image-buffer-upload
```

Watch:

```txt
Major GC count ↑
GC duration ↑
Heap Used ↑
```

---

# HTTP Latency

Most important section.

```txt
http_request_duration_seconds
```

Metrics route:

```txt
48 requests
0.302 sec total
```

Average latency:

```txt
0.302 / 48
≈ 0.006 sec
≈ 6 ms
```

Current:

```txt
Very fast
```

---

# Throughput

```txt
http_requests_total 50
```

Meaning:

```txt
50 requests processed
```

Current:

```txt
Traffic almost zero
```

---

# 404 Detection

Interesting finding:

```txt
/ favicon.ico
/firebase-messaging-sw.js
```

Both returning:

```txt
404
```

These are browser-generated requests.

Production engineers often discover:

```txt
Missing static assets
Broken frontend paths
Service worker issues
```

through these metrics.

---

# What Happens During Load Test

Run:

```bash
k6 run k6-load-test.js
```

While another terminal:

```bash
watch -n 1 "curl -s localhost:3000/metrics | grep eventloop"
```

and

```bash
docker stats
```

Expected:

| Endpoint | CPU | Memory | Event Loop | Latency |
|-----------|------|---------|------------|---------|
| health | Low | Low | Low | Low |
| redis-read | Low | Low | Low | Medium |
| redis-write | Low | Low | Low | Medium |
| cpu-spike | Very High | Low | High | High |
| memory-leak | Low | Very High | Medium | Medium |
| image-buffer-upload | Medium | Very High | Medium | High |
| image-stream-upload | Medium | Low | Low | Medium |
| event-loop-block | Very High | Low | Extreme | Extreme |
| db-heavy | Low | Low | Low | High |
| external-api | Low | Low | Low | High |
| batch-processing | High | Medium | Medium | Medium |

---

# Missing Production Metrics

Your lab is good, but production engineers usually add:

```js
eventLoopUtilization()
```

```js
monitorEventLoopDelay()
```

```js
process.memoryUsage()
```

```js
process.cpuUsage()
```

Custom Prometheus gauges:

```txt
node_event_loop_utilization

node_heap_used_mb

node_heap_total_mb

node_rss_memory_mb

node_cpu_percent

redis_latency_ms

active_requests

active_uploads
```

These are the metrics that immediately expose bottlenecks when you hammer the system with 1000+ concurrent requests.

---

## 🧑 You

bro Prometheus i can t read i didnt get anything

---

## 🤖 ChatGPT

That's completely normal. Most developers look at Prometheus output for the first time and see a wall of text.

Don't try to read all 200 lines.

Think of Prometheus like your car dashboard.

You only care about a few gauges.

---

# Step 1: Ignore 95% of Metrics

For now only look at these:

```txt
process_resident_memory_bytes

nodejs_eventloop_lag_p99_seconds

http_requests_total

http_request_duration_seconds
```

That's it.

---

# Memory Usage

You see:

```txt
process_resident_memory_bytes 82669568
```

Prometheus stores bytes.

Convert:

```txt
82669568 / 1024 / 1024

= 78 MB
```

Meaning:

```txt
Node process currently uses 78 MB RAM
```

Think:

```txt
Task Manager
Chrome -> 500 MB
Node -> 78 MB
```

Same thing.

---

# Event Loop Lag

You see:

```txt
nodejs_eventloop_lag_p99_seconds 0.011
```

Convert:

```txt
0.011 sec

= 11 ms
```

Meaning:

```txt
Event loop delayed by only 11ms
```

Healthy:

```txt
<20ms
```

Bad:

```txt
100ms+
```

Very Bad:

```txt
500ms+
```

---

# Total Requests

You see:

```txt
http_requests_total 50
```

Meaning:

```txt
Application handled 50 requests
```

That's all.

---

# Latency

You see:

```txt
http_request_duration_seconds_sum 0.302

http_request_duration_seconds_count 48
```

Calculate:

```txt
0.302 / 48

= 0.006 sec
```

Convert:

```txt
6 ms
```

Meaning:

```txt
Average request took 6ms
```

Very fast.

---

# What Production Engineers Actually Look At

Imagine manager says:

```txt
Bro system is slow.
```

You immediately check:

```txt
1. CPU ?
2. Memory ?
3. Event Loop ?
4. Latency ?
```

Like this:

```txt
CPU = 90%
Memory = 2 GB
Event Loop = 400 ms
Latency = 3 sec
```

Now you know:

```txt
CPU bottleneck
```

---

# Let's Create a Real Problem

Call this 5 times:

```bash
curl localhost:3000/memory-leak
curl localhost:3000/memory-leak
curl localhost:3000/memory-leak
curl localhost:3000/memory-leak
curl localhost:3000/memory-leak
```

Then check:

```bash
curl localhost:3000/metrics | grep resident
```

You'll see:

```txt
Before:
78 MB

After:
328 MB
```

Now Prometheus becomes interesting.

You can literally see memory leaking.

---

# Another Experiment

Hit:

```bash
curl localhost:3000/event-loop-block
```

While it runs:

```bash
curl localhost:3000/metrics | grep eventloop
```

Before:

```txt
11 ms
```

After:

```txt
200 ms
500 ms
1000 ms
```

Now you can visually see:

```txt
Event loop blocked
Latency increased
Users waiting
```

---

# The Mental Model

Don't read Prometheus as text.

Read it as questions:

```txt
How much memory?
=> process_resident_memory_bytes

How many requests?
=> http_requests_total

How slow?
=> http_request_duration_seconds

Is Node blocked?
=> nodejs_eventloop_lag
```

Once you understand these 4 metrics, Grafana dashboards suddenly make sense because Grafana is just drawing charts from the exact numbers you're seeing in `/metrics`.

Next, I can show you how a senior backend engineer reads a Grafana dashboard during a production incident and identifies the bottleneck in under 30 seconds.

---

## 🧑 You

dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$ curl localhost:3000/metrics | grep resident
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 13186  100 13186    0     0# HELP process_resident_memory_bytes Resident memory size in bytes.
  2# TYPE process_resident_memory_bytes gauge
38process_resident_memory_bytes 83591168
5k      0 --:--:-- --:--:-- --:--:-- 2575k
dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$ curl localhost:3000/memory-leak
{"allocated":"50MB","count":1}dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$ curl localhost:3000/metrics | grep resident
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 14375  100 14375    0     0  1753k      0 --:--:-- # HELP process_resident_memory_bytes Resident memory size in bytes.
--:# TYPE process_resident_memory_bytes gauge
--process_resident_memory_bytes 83865600
:-- --:--:-- 2005k
dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$ curl localhost:3000/memory-leak
{"allocated":"50MB","count":2}dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$ curl localhost:3000/memory-leak
{"allocated":"50MB","count":3}dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$ curl localhost:3000/memory-leak
{"allocated":"50MB","count":4}dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$ curl localhost:3000/memory-leak
{"allocated":"50MB","count":5}dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$ curl localhost:3000/memory-leak
{"allocated":"50MB","count":6}dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$ curl localhost:3000/metrics | grep resident
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 14364  100 14364    0     0  2333k      0 --:-# HELP process_resident_memory_bytes Resident memory size in bytes.
-:-# TYPE process_resident_memory_bytes gauge
- process_resident_memory_bytes 83472384
--:--:-- --:--:-- 2805k
dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$

---

## 🤖 ChatGPT

This is actually a fantastic learning moment.

You expected:

```txt
80 MB
130 MB
180 MB
230 MB
280 MB
330 MB
```

But you got:

```txt
83 MB
83 MB
83 MB
```

So now you're thinking:

```txt
Bro I allocated 50MB 6 times
Why memory didn't increase?
```

Exactly the right question.

---

Look at your code:

```js
const data = Buffer.alloc(50 * 1024 * 1024);
memoryStore.push(data);
```

You are allocating a Buffer.

Node.js Buffers are **external memory**, not V8 heap memory.

Prometheus metric you're checking:

```txt
process_resident_memory_bytes
```

is RSS memory.

Sometimes Linux doesn't immediately commit all those pages.

---

Check these instead:

```bash
curl localhost:3000/metrics | grep external_memory
```

and

```bash
curl localhost:3000/metrics | grep heap_size_used
```

I suspect you'll see:

```txt
nodejs_external_memory_bytes ↑
```

while heap stays mostly unchanged.

---

Even better.

Add this endpoint:

```js
router.get("/", (req, res) => {

  const arr = [];

  for(let i=0;i<10000000;i++){
    arr.push({
      id:i,
      name:`user-${i}`
    });
  }

  memoryStore.push(arr);

  res.json({
    count: memoryStore.length
  });
});
```

Now hit:

```bash
curl localhost:3000/memory-leak
```

few times.

Then:

```bash
curl localhost:3000/metrics | grep heap_size_used
```

You'll immediately see:

```txt
14 MB
120 MB
250 MB
400 MB
700 MB
```

because now you're leaking JS objects into the V8 heap.

---

Another thing.

Run:

```bash
docker stats
```

in another terminal.

Then call:

```bash
curl localhost:3000/memory-leak
```

6-10 times.

You'll likely see container memory increase much more clearly than Prometheus RSS.

---

For learning purposes, I would actually replace your memory leak route with:

```js
const memoryStore = [];

router.get("/", (req, res) => {

  const hugeData = Array.from(
    { length: 500000 },
    (_, i) => ({
      id: i,
      name: `user-${i}`,
      email: `user-${i}@gmail.com`
    })
  );

  memoryStore.push(hugeData);

  res.json({
    chunks: memoryStore.length
  });
});
```

This creates a **real V8 heap leak**.

Then watch:

```bash
curl localhost:3000/metrics | grep heap_size_used
```

and

```bash
docker stats
```

After 5-10 requests you'll see memory exploding, GC running more often, and eventually Node may crash with:

```txt
FATAL ERROR:
JavaScript heap out of memory
```

That is much closer to the memory leak incidents backend engineers investigate in production.

---

## 🧑 You

bro sorry any good visual is there

---

## 🤖 ChatGPT

Yes bro. Reading raw Prometheus metrics is painful.

The best way to learn is:

```txt id="e1m6w8"
Request
   ↓
Node App
   ↓
Prometheus
   ↓
Grafana Dashboard
```

Forget `/metrics` initially.

Watch graphs move in real time.

---

# Dashboard 1 — Memory

Create a Grafana panel:

```promql
process_resident_memory_bytes / 1024 / 1024
```

Graph title:

```txt id="buj66k"
Memory Usage (MB)
```

Then hit:

```bash id="gt8y4e"
curl localhost:3000/memory-leak
```

You'll see:

```txt id="72vj4w"
80 MB
120 MB
170 MB
220 MB
300 MB
```

graph climbing upward.

Visual immediately tells:

```txt id="jxbfxj"
memory leak
```

---

# Dashboard 2 — Event Loop Lag

```promql
nodejs_eventloop_lag_p99_seconds * 1000
```

Title:

```txt id="jpyqwe"
Event Loop Lag (ms)
```

Normal:

```txt id="rrgq5j"
10ms
12ms
15ms
```

Hit:

```bash id="ml0ub0"
curl localhost:3000/event-loop-block
```

Graph:

```txt id="32zb8k"
10ms
40ms
120ms
400ms
800ms
```

Now you visually understand:

```txt id="vj6rlh"
Node blocked
```

---

# Dashboard 3 — Request Rate

```promql
rate(http_requests_total[1m])
```

Title:

```txt id="r7lxhb"
Requests Per Second
```

Run:

```bash id="xjlwmv"
k6 run k6-load-test.js
```

Graph:

```txt id="u2u0jt"
0
0
10
100
500
1000
```

Now you see traffic.

---

# Dashboard 4 — Latency

```promql
histogram_quantile(
  0.95,
  rate(http_request_duration_seconds_bucket[1m])
)
```

Title:

```txt id="c5k58m"
P95 Latency
```

Normal:

```txt id="lpg1xg"
5ms
7ms
10ms
```

Heavy load:

```txt id="6v93fj"
50ms
100ms
300ms
2 sec
```

Now users complain.

---

# Dashboard 5 — CPU

Prometheus:

```promql
rate(process_cpu_seconds_total[1m]) * 100
```

Title:

```txt id="8nbu68"
CPU %
```

Call:

```bash id="4frakf"
curl localhost:3000/cpu-spike
```

Graph:

```txt id="ez7d0k"
5%
10%
95%
100%
```

Immediate bottleneck.

---

# Dashboard 6 — GC Activity

```promql
rate(nodejs_gc_duration_seconds_count[1m])
```

Title:

```txt id="4s8bzv"
GC Runs
```

Memory leak causes:

```txt id="g9yjlwm"
1
2
5
20
100
```

Graph spikes.

---

# Dashboard 7 — Redis Latency

Create custom metric.

```txt id="9j5kj7"
redis_latency_ms
```

Graph:

```txt id="ivg1rm"
2ms
3ms
5ms
7ms
200ms
```

Now Redis becomes bottleneck.

---

# The Visual Lab I Recommend

Have only 4 graphs on screen:

```txt id="ik3i9y"
Top Left
Memory Usage

Top Right
CPU %

Bottom Left
Event Loop Lag

Bottom Right
P95 Latency
```

Then run:

```bash id="lq99vk"
k6 run k6-load-test.js
```

and watch all four move together.

Example:

```txt id="f3l8gu"
CPU ↑

Event Loop ↑

Latency ↑

Memory stable
```

Immediately tells:

```txt id="5gnpna"
CPU bottleneck
```

or

```txt id="yce5dl"
Memory ↑
GC ↑
Latency ↑
```

means:

```txt id="ux8vgn"
memory pressure
```

This is how backend engineers actually debug production issues. They rarely stare at raw `/metrics`; they watch Grafana graphs and correlate changes across CPU, memory, latency, and event loop lag.

---

## 🧑 You

how i integrate Grafana graphs in my code

---

## 🤖 ChatGPT

Bro, **you don't integrate Grafana into your code**.

Think of it like this:

```txt
Your Code
   ↓
Prometheus Metrics Endpoint (/metrics)
   ↓
Prometheus Server
   ↓
Grafana
```

Grafana never touches your Express code.

---

# Step 1: Expose Metrics

Your code already does this:

```js
app.get("/metrics", async (req, res) => {
  res.set(
    "Content-Type",
    client.register.contentType
  );

  res.end(await client.register.metrics());
});
```

Visit:

```txt
http://localhost:3000/metrics
```

Prometheus reads this endpoint every few seconds.

---

# Step 2: Configure Prometheus

```yaml
scrape_configs:
  - job_name: node-app

    static_configs:
      - targets:
          - api:3000
```

Prometheus says:

```txt
Every 5 seconds
Call api:3000/metrics
Store values
```

---

# Step 3: Open Grafana

```txt
http://localhost:3001
```

Default:

```txt
admin
admin
```

---

# Step 4: Add Prometheus Data Source

Grafana UI:

```txt
Connections
   ↓
Data Sources
   ↓
Add Data Source
   ↓
Prometheus
```

URL:

```txt
http://prometheus:9090
```

Save.

---

# Step 5: Create Graph

Dashboard

```txt
+
Create Dashboard
   ↓
Add Visualization
```

Query:

```promql
process_resident_memory_bytes / 1024 / 1024
```

You'll instantly get a graph.

---

# Memory Graph

Query:

```promql
process_resident_memory_bytes / 1024 / 1024
```

Title:

```txt
Memory Usage MB
```

---

# CPU Graph

Query:

```promql
rate(process_cpu_seconds_total[1m]) * 100
```

Title:

```txt
CPU Usage %
```

---

# Event Loop Graph

Query:

```promql
nodejs_eventloop_lag_p99_seconds * 1000
```

Title:

```txt
Event Loop Lag ms
```

---

# Request Rate Graph

Query:

```promql
rate(http_requests_total[1m])
```

Title:

```txt
Requests Per Second
```

---

# Latency Graph

Query:

```promql
histogram_quantile(
  0.95,
  rate(http_request_duration_seconds_bucket[1m])
)
```

Title:

```txt
P95 Latency
```

---

# What Happens Next

Open dashboard.

Run:

```bash
k6 run k6-load-test.js
```

You'll literally see:

genui{"charts_widget_v2":{"content":{"chartType":"line","meta":{"title":"Typical load test impact","description":"Illustrative trend when traffic increases."},"xKey":"phase","series":[{"dataKey":"latency","label":"Latency","axisLabel":"Milliseconds","valueFormat":"integer","valueSuffix":" ms"}],"data":[{"phase":"Idle","latency":5},{"phase":"Light Load","latency":15},{"phase":"Medium Load","latency":80},{"phase":"Heavy Load","latency":300},{"phase":"Overloaded","latency":1200}]}}}

That's the moment monitoring starts making sense. Instead of reading thousands of lines from `/metrics`, you watch graphs move while you hit:

```bash
curl localhost:3000/cpu-spike
curl localhost:3000/memory-leak
curl localhost:3000/event-loop-block
```

and immediately see which resource is becoming the bottleneck.

For your learning lab, create only these 4 Grafana panels first:

```txt
1. Memory Usage MB
2. CPU Usage %
3. Event Loop Lag ms
4. P95 Latency
```

Those four graphs alone will teach you more about backend performance than reading raw Prometheus output.

---

## 🧑 You

[image]

---

## 🤖 ChatGPT

Perfect bro 👍

You're already in the Grafana panel editor.

Do these exact steps:

### 1. Change Visualization

Right side panel:

```txt
All visualizations
   ↓
Time Series
```

Select **Time Series**.

Currently you're seeing:

```txt
Table view
```

which won't show graphs.

---

### 2. First Graph (Memory)

In Query A switch to **Code** mode.

Paste:

```promql
process_resident_memory_bytes / 1024 / 1024
```

Click:

```txt
Run Queries
```

You should immediately see a line graph.

Rename panel:

```txt
Memory Usage (MB)
```

---

### 3. Second Graph (CPU)

Create another panel.

Query:

```promql
rate(process_cpu_seconds_total[1m]) * 100
```

Title:

```txt
CPU Usage %
```

---

### 4. Third Graph (Event Loop)

Create another panel.

Query:

```promql
nodejs_eventloop_lag_p99_seconds * 1000
```

Title:

```txt
Event Loop Lag (ms)
```

---

### 5. Fourth Graph (Requests)

Query:

```promql
rate(http_requests_total[1m])
```

Title:

```txt
Requests Per Second
```

---

### If Graph Is Empty

Go to:

```txt
Connections
  ↓
Data Sources
  ↓
Prometheus
```

Click:

```txt
Save & Test
```

You should see:

```txt
Data source is working
```

---

### Quick Test

After Memory graph is visible run:

```bash
curl localhost:3000/memory-leak
curl localhost:3000/memory-leak
curl localhost:3000/memory-leak
```

The graph should move upward.

---

### Easier Way

In the query box type:

```promql
process_resident_memory_bytes
```

and click **Run Queries**.

If you see a graph, Prometheus ↔ Grafana is connected correctly.

Tell me what happens after running:

```promql
process_resident_memory_bytes
```

Do you get:

```txt
No Data
```

or

```txt
a graph line
```

That's the next thing to troubleshoot.

---

## 🧑 You

Cdell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoringk6 run k6-load-test.js
Command 'k6' not found, but can be installed with:
sudo snap install k6
dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$  

how i run inide docker this k6 query

---

## 🤖 ChatGPT

Yes bro, easiest is to run **k6 as a Docker container**.

From your project root:

```bash
docker run --rm -i \
  --network host \
  -v $(pwd):/scripts \
  grafana/k6:latest \
  run /scripts/k6-load-test.js
```

If you're on Linux (Ubuntu), `--network host` works perfectly.

---

### Verify k6 Container

```bash
docker run --rm grafana/k6 version
```

Expected:

```txt
k6 vX.X.X
```

---

### Your Current Script

You have:

```js
http.get(
  "http://host.docker.internal:3000/cpu-spike"
);
```

`host.docker.internal` usually works on Docker Desktop (Mac/Windows).

On Ubuntu/Linux, change to:

```js
http.get("http://localhost:3000/cpu-spike");
```

or

```js
http.get("http://172.17.0.1:3000/cpu-spike");
```

if using Docker bridge networking.

---

### Better: Add k6 to docker-compose

```yaml
services:

  k6:
    image: grafana/k6
    volumes:
      - ./k6-load-test.js:/k6-load-test.js
    command: run /k6-load-test.js
    network_mode: host
```

Then:

```bash
docker compose run --rm k6
```

---

### While Running

Terminal 1:

```bash
docker compose up -d
```

Terminal 2:

```bash
docker run --rm -i \
  --network host \
  -v $(pwd):/scripts \
  grafana/k6:latest \
  run /scripts/k6-load-test.js
```

Terminal 3:

```bash
docker stats
```

Terminal 4:

Open Grafana:

```txt
http://localhost:3001
```

Watch:

```txt
Memory Usage
CPU Usage
Event Loop Lag
Request Rate
```

move in real time.

---

### Before Running

Reduce this:

```js
export const options = {
  vus: 1000,
  duration: "60s"
};
```

to:

```js
export const options = {
  vus: 50,
  duration: "30s"
};
```

A single Node container on a laptop can become unresponsive with 1000 VUs, especially when hitting:

```txt
/cpu-spike
/event-loop-block
```

Start with:

```txt
10 VUs
50 VUs
100 VUs
200 VUs
```

and observe Grafana after each run. This makes it much easier to understand which metric reacts to which bottleneck.

---

## 🧑 You

2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/redis-read\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/redis-read\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/redis-read\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"
time="2026-06-12T18:01:54Z" level=warning msg="Request Failed" error="Get \"http://host.docker.internal:3000/event-loop-block\": lookup host.docker.internal on 127.0.0.53:53: no such host"


  █ TOTAL RESULTS 

    HTTP
    http_req_duration....: avg=0s       min=0s      med=0s       max=0s   p(90)=0s       p(95)=0s      
    http_req_failed......: 100.00% 751392 out of 751392
    http_reqs............: 751392  12495.354581/s

    EXECUTION
    iteration_duration...: avg=319.66ms min=16.79ms med=262.27ms max=5.5s p(90)=400.81ms p(95)=442.74ms
    iterations...........: 187848  3123.838645/s
    vus..................: 1000    min=1000             max=1000
    vus_max..............: 1000    min=1000             max=1000

    NETWORK
    data_received........: 0 B     0 B/s
    data_sent............: 0 B     0 B/s




running (1m00.1s), 0000/1000 VUs, 187848 complete and 0 interrupted iterations
default ✓ [ 100% ] 1000 VUs  1m0s

---

## 🤖 ChatGPT

Bro, your load test **did run**, but every request failed.

The important line is:

```txt id="u8t7pw"
http_req_failed: 100.00%
751392 out of 751392
```

And the root cause is:

```txt id="e5bryo"
lookup host.docker.internal

no such host
```

You're on Ubuntu/Linux.

`host.docker.internal` does not exist by default.

---

# Fix 1 (Easiest)

Check your app:

```bash id="g3zz92"
curl localhost:3000/health
```

If this works:

```json id="ngbrhy"
{"status":"UP"}
```

then update your k6 script.

Replace:

```js id="6nfrk0"
http.get("http://host.docker.internal:3000/cpu-spike");
```

with:

```js id="l89ej8"
http.get("http://localhost:3000/cpu-spike");
```

Do this for every endpoint.

---

# Fix 2 (Recommended)

If k6 runs inside Docker and api runs in Docker Compose:

Check network:

```bash id="m2ukyu"
docker network ls
```

Find something like:

```txt id="r9z92n"
simulate_monitoring_default
```

Then run:

```bash id="85p7p9"
docker run --rm \
  --network simulate_monitoring_default \
  -v $(pwd):/scripts \
  grafana/k6 \
  run /scripts/k6-load-test.js
```

Now use:

```js id="2fww0y"
http.get("http://api:3000/cpu-spike");
http.get("http://api:3000/db-heavy");
http.get("http://api:3000/redis-read");
```

Notice:

```txt id="tmb0v4"
api
```

is the Docker service name.

This is how containers talk to each other.

---

# Verify Before Running Huge Test

From inside Docker network:

```bash id="m4a5xk"
docker exec -it <api-container> sh
```

Then:

```bash id="l5huj0"
wget -qO- http://localhost:3000/health
```

Should return:

```json id="6msdz7"
{"status":"UP"}
```

---

# Also Reduce This

Currently:

```js id="32cuhx"
vus: 1000
```

For learning:

```js id="lyqv8o"
export const options = {
  vus: 20,
  duration: "30s"
};
```

Then:

```txt id="t4v0vq"
Watch Grafana
Watch docker stats
Watch Prometheus
```

You will actually understand what is happening.

With:

```txt id="q20es8"
1000 VUs
```

and endpoints like:

```txt id="qozj5w"
/event-loop-block
/cpu-spike
```

you can freeze your laptop before learning anything useful.

---

Run this and show me the output:

```bash id="sv9lv0"
docker ps
```

and

```bash id="0vbuv4"
docker network ls
```

Then I can tell you the exact URL (`localhost`, `api`, or Docker IP) that your k6 script should use.

---

## 🧑 You

-06-12T18:18:45Z" level=warning msg="Request Failed" error="Get \"http://localhost:3000/db-heavy\": EOF"
time="2026-06-12T18:18:45Z" level=warning msg="Request Failed" error="Get \"http://localhost:3000/db-heavy\": EOF"
time="2026-06-12T18:18:45Z" level=warning msg="Request Failed" error="Get \"http://localhost:3000/db-heavy\": EOF"
time="2026-06-12T18:18:45Z" level=warning msg="Request Failed" error="Get \"http://localhost:3000/db-heavy\": EOF"
time="2026-06-12T18:18:45Z" level=warning msg="Request Failed" error="Get \"http://localhost:3000/db-heavy\": EOF"
time="2026-06-12T18:18:45Z" level=warning msg="Request Failed" error="Get \"http://localhost:3000/db-heavy\": EOF"
time="2026-06-12T18:18:45Z" level=warning msg="Request Failed" error="Get \"http://localhost:3000/db-heavy\": EOF"

running (0m47.8s), 1000/1000 VUs, 0 complete and 0 interrupted iterations
default   [  80% ] 1000 VUs  0m47.8s/1m0s

running (0m48.8s), 1000/1000 VUs, 0 complete and 0 interrupted iterations
default   [  81% ] 1000 VUs  0m48.8s/1m0s

running (0m49.8s), 1000/1000 VUs, 0 complete and 0 interrupted iterations
default   [  83% ] 1000 VUs  0m49.8s/1m0s

running (0m50.8s), 1000/1000 VUs, 0 complete and 0 interrupted iterations
default   [  85% ] 1000 VUs  0m50.8s

---

## 🤖 ChatGPT

Bro, this is actually **good news**. 🎯

Now k6 is reaching your application.

Before:

```txt id="7eb5cv"
no such host
```

Network problem.

Now:

```txt id="w8uh8c"
EOF
```

Application problem.

---

## What EOF Means

```txt id="xk3xaw"
EOF = connection closed unexpectedly
```

Usually one of these happened:

```txt id="o0nspn"
1. Node process crashed
2. Container restarted
3. Request timeout
4. Event loop completely blocked
5. Server overloaded
```

---

## Your Biggest Problem

You're running:

```js id="5t0ggq"
vus: 1000
```

and hitting:

```txt id="kk3l4p"
/db-heavy
```

which does:

```js id="81x37r"
await new Promise(resolve =>
  setTimeout(resolve, 3000)
);
```

---

Think about it:

```txt id="f9sl76"
1000 users
×
3 second request
```

means roughly:

```txt id="7cbmcc"
1000 open connections waiting
```

on a tiny Node container.

---

## Check If Node Died

Run:

```bash id="p6qgvs"
docker ps
```

Look for:

```txt id="v9b7e6"
STATUS
Restarting
Exited
```

---

## Check Logs

Very important.

Run:

```bash id="jcrlrx"
docker compose logs -f api
```

or

```bash id="3rkm7i"
docker logs <container-id>
```

Look for:

```txt id="4j6gc8"
JavaScript heap out of memory

Killed

OOMKilled

FATAL ERROR

ECONNRESET
```

---

## Check Resource Usage

While k6 runs:

```bash id="38mt6n"
docker stats
```

Watch:

```txt id="xjvnl0"
CPU %
MEM USAGE
```

If you see:

```txt id="7u0qjf"
CPU 100%
```

or

```txt id="r8skpk"
Memory 100%
```

you found the bottleneck.

---

## For Learning

Don't start with:

```js id="hqol48"
vus: 1000
```

Use:

```js id="92m4x0"
export const options = {
  vus: 20,
  duration: "30s"
};
```

Then:

```txt id="ihaj4i"
20 VUs
50 VUs
100 VUs
200 VUs
500 VUs
```

Watch Grafana after each run.

This teaches you much more.

---

## Even Better Test

Test only one endpoint first:

```js id="rqrzqf"
import http from "k6/http";

export const options = {
  vus: 50,
  duration: "30s"
};

export default function () {
  http.get("http://localhost:3000/db-heavy");
}
```

Now Grafana will clearly show:

```txt id="xckm4z"
Latency ↑
Requests ↑
Memory stable
CPU stable
```

Then try:

```txt id="saw4l6"
/cpu-spike
```

You'll see:

```txt id="6vvg7w"
CPU ↑↑↑
Latency ↑↑↑
```

Then:

```txt id="ovx0m9"
/memory-leak
```

You'll see:

```txt id="wffo70"
Memory ↑↑↑
GC ↑↑↑
```

Each endpoint teaches one bottleneck.

---

Next command I want you to run:

```bash id="iy0krl"
docker compose logs api --tail=100
```

Paste the output. That will tell us whether Node is crashing, timing out, or simply being overwhelmed.

---

## 🧑 You

[image]
^Cdell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoringcurl localhost:3000/cpu-spike
^C
dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$ http://localhost:3000/metrics
bash: http://localhost:3000/metrics: No such file or directory
dell@dell-Inspiron-15-3567:/media/$USER/ExcelWorld/Learning_PROJECTS/simulate_monitoring$ curl http://localhost:3000/metrics


i thin some think happen 
memeory our

---

## 🤖 ChatGPT

Bro, from your screenshot, your system is actually alive. 👍

The important part is this:

```txt id="ysr8o8"
simulate_monitoring-api-1
CPU = 100.97%
Memory = 46.63 MiB
```

This means:

```txt id="s2i7y0"
Node container is NOT dead
Node container is NOT OOM killed
CPU is fully saturated
```

---

Then you ran:

```bash id="r1ht0q"
curl localhost:3000/cpu-spike
```

and had to press:

```txt id="j8h8zf"
Ctrl + C
```

That is actually expected.

Because your endpoint does:

```js id="6gqjlwm"
for (let i = 0; i < 500000000; i++) {
  total += i;
}
```

which blocks the CPU.

---

### What your graphs are showing

Memory:

```txt id="iqrvvg"
78.3 MB
78.9 MB
```

Healthy.

No leak.

---

Event Loop:

```txt id="x7k1ng"
10 ms
11 ms
11.3 ms
```

Healthy.

---

Latency:

```txt id="fwk6v8"
0.009 sec
0.016 sec
```

```txt id="wnj2v3"
9 ms
16 ms
```

Healthy.

---

CPU:

The graph is climbing.

That is exactly what `/cpu-spike` is supposed to do.

---

### Why you think memory is out?

Because the graph stopped updating.

But look carefully:

```txt id="c9gmmt"
Memory = 46 MB
```

That's tiny.

If memory were exhausted you'd see:

```txt id="ic7w1z"
500 MB
1 GB
2 GB
```

or Docker would show:

```txt id="2m1hqq"
OOMKilled
```

---

### Check container status

Run:

```bash id="jvhy6m"
docker ps
```

If you see:

```txt id="zy98ps"
Up 20 minutes
```

then app is alive.

---

### Check API manually

Run:

```bash id="ftwjtu"
curl http://localhost:3000/health
```

If response:

```json id="9r70wp"
{"status":"UP"}
```

then everything is healthy.

---

### Better CPU Spike

Your current route is too aggressive.

Replace:

```js id="7nwt50"
for (let i = 0; i < 500000000; i++)
```

with:

```js id="obcrik"
for (let i = 0; i < 50000000; i++)
```

10x smaller.

Then:

```bash id="zwyjlwm"
curl http://localhost:3000/cpu-spike
```

will complete in a few seconds and you'll see Grafana react.

---

### Most Useful Experiment Now

Open Grafana.

Keep these visible:

```txt id="owjlwm"
CPU
Memory
Event Loop
Latency
```

Then run:

```bash id="3gaw4w"
for i in {1..20}
do
  curl http://localhost:3000/db-heavy &
done
```

Watch:

```txt id="33f6gv"
Request Rate ↑
Latency ↑
CPU almost same
Memory almost same
```

This teaches you what a slow database looks like.

Then try:

```bash id="1qzq63"
curl http://localhost:3000/cpu-spike
```

You'll see:

```txt id="z1hj4d"
CPU ↑↑↑
Latency ↑↑↑
Event Loop ↑↑↑
```

That's the moment Grafana starts making sense. You're no longer reading metrics; you're watching system behavior.

---

## 🧑 You

[image]

---

## 🤖 ChatGPT

Bro, this screenshot is gold. 🔥

You have already found your first bottleneck.

```txt id="c7fd4d"
simulate_monitoring-api-1

CPU = 100.09%
Memory = 60.64 MB
```

Interpretation:

```txt id="kknwlv"
CPU saturated ✅
Memory healthy ✅
Redis healthy ✅
Prometheus healthy ✅
Grafana healthy ✅
```

---

### What this means

Your Node process is doing:

```txt id="rktuv6"
/cpu-spike
```

and consuming an entire CPU core.

Because Node.js is single-threaded:

```txt id="bhjlwm"
CPU 100%
=
Event Loop Busy
=
Requests Wait
=
Latency Increases
```

---

### This is NOT a memory issue

If memory were the bottleneck you'd see something like:

```txt id="70lb5t"
MEM USAGE

500 MB
1 GB
2 GB
```

or

```txt id="4pxzyq"
OOMKilled
```

Instead:

```txt id="xv0tdj"
60 MB
```

is tiny.

---

### Real Production Thinking

Imagine manager says:

```txt id="7sptc7"
Users complain system is slow.
```

You look at:

```txt id="7jltw0"
CPU = 100%
Memory = 60 MB
```

Immediately:

```txt id="qjkp67"
Not memory issue

CPU bottleneck
```

---

### Verify Event Loop

Run:

```bash id="rxtd0g"
curl localhost:3000/metrics | grep eventloop_lag_p99
```

I expect something like:

```txt id="v7h5cu"
0.1
0.3
0.5
```

instead of:

```txt id="2nsppd"
0.01
```

because CPU is busy.

---

### Next Experiment

Open Grafana.

Keep:

```txt id="ah90je"
CPU
Event Loop
Latency
```

visible.

Then run only:

```bash id="lbwzyd"
curl localhost:3000/cpu-spike
```

Watch:

```txt id="cksr0v"
CPU ↑

Event Loop ↑

Latency ↑
```

This is exactly what happens in production when somebody accidentally writes:

```js id="nfs3a7"
while(true){}
```

or

```js id="zxhjlwm"
for(let i=0;i<1000000000;i++)
```

inside an API.

---

### One More Thing

Your CPU endpoint is too aggressive:

Current:

```js id="lm7nsg"
for(let i=0;i<500000000;i++)
```

Change to:

```js id="m39v0x"
for(let i=0;i<50000000;i++)
```

Otherwise the endpoint blocks for too long and makes the whole service feel hung.

---

From this screenshot alone, if I were on-call in production, my conclusion would be:

```txt id="s7e83y"
Root Cause:
CPU saturation

Evidence:
CPU 100%
Memory 60 MB

Likely Impact:
High latency
Slow responses
Blocked event loop
```

That's exactly the kind of reasoning Grafana + Prometheus are meant to help you build. 🚀

---

## 🧑 You

[image]

---

## 🤖 ChatGPT

Bro, this screenshot tells us something important. 🔥

Your command:

```bash
curl localhost:3000/metrics | grep eventloop_lag_p99
```

is hanging for **28 seconds**.

Look at this:

```txt
Time Total: 0:00:28
Speed: 0
```

That means `/metrics` itself cannot respond.

---

### What does that mean?

Your Node process is likely stuck in:

```js
for(let i = 0; i < 500000000; i++) {
  total += i;
}
```

or

```js
while(Date.now() < end) {}
```

from:

```txt
/cpu-spike
/event-loop-block
```

Because Node has:

```txt
1 Event Loop
1 Main Thread
```

If CPU-intensive work is running:

```txt
Event Loop Busy
↓
No New Requests
↓
/metrics Waits
↓
curl Hangs
```

---

### Verify

Open another terminal:

```bash
docker stats
```

If API container shows:

```txt
CPU 100%
```

continuously,

then Node is still executing the CPU-heavy route.

---

### Check API Logs

Run:

```bash
docker compose logs -f api
```

Look for:

```txt
GET /cpu-spike
GET /event-loop-block
```

repeating.

---

### Quick Recovery

Stop everything:

```bash
docker compose down
```

Then:

```bash
docker compose up -d
```

Now test:

```bash
curl localhost:3000/health
```

Should return instantly.

Then:

```bash
curl localhost:3000/metrics | grep eventloop_lag_p99
```

Should return something like:

```txt
nodejs_eventloop_lag_p99_seconds 0.011
```

within a second.

---

### For Learning

Your current endpoint is too destructive:

```js
for(let i = 0; i < 500000000; i++)
```

Change it to:

```js
for(let i = 0; i < 50000000; i++)
```

or even:

```js
for(let i = 0; i < 10000000; i++)
```

Then you'll get:

```txt
CPU spike
Event loop spike
Latency spike
```

without freezing the entire application.

---

The most interesting observation right now is:

```txt
CPU = 100%
curl /metrics = hanging
Memory = 60 MB
```

That's a textbook example of **CPU saturation causing event-loop starvation**, which is exactly one of the production bottlenecks you're trying to simulate.

---

## 🧑 You

[image]

---

## 🤖 ChatGPT

Bro, your dashboard showing **"No data"** everywhere means one of two things:

```txt id="jlwm92"
1. Prometheus cannot scrape api anymore
OR
2. Node app is frozen / restarted
```

Let's verify in 30 seconds.

### Check API

Run:

```bash id="f1o8rm"
curl http://localhost:3000/health
```

If it hangs:

```txt id="4hmddm"
Node is blocked
```

If it returns:

```json id="khyxpb"
{"status":"UP"}
```

then Node is alive.

---

### Check Metrics

Run:

```bash id="4crmx5"
curl http://localhost:3000/metrics | head
```

If it hangs:

```txt id="0x8vkt"
Prometheus cannot scrape
```

which explains "No data".

---

### Check Prometheus

Open:

```txt id="pup67y"
http://localhost:9090/targets
```

Look at:

```txt id="1n8xxy"
node-app
```

Status should be:

```txt id="z4bg9x"
UP
```

If you see:

```txt id="jlwm7y"
DOWN
```

click it and copy the error.

---

### Check Containers

Run:

```bash id="ys89lu"
docker ps
```

You should see:

```txt id="48z1y3"
simulate_monitoring-api-1
simulate_monitoring-prometheus-1
simulate_monitoring-grafana-1
simulate_monitoring-redis-1
```

all in:

```txt id="y6m9h0"
Up ...
```

state.

---

### Most Likely Cause

Earlier you ran:

```txt id="j6g6n0"
1000 VUs
/cpu-spike
/event-loop-block
```

and your screenshot showed:

```txt id="rfxjlwm"
API CPU = 100%
```

My guess:

```txt id="c8kqv9"
Prometheus stopped getting responses
```

so Grafana has nothing to plot.

---

### Quick Reset

```bash id="rkt8bl"
docker compose restart api
```

Wait 10 seconds.

Then:

```bash id="ydyblm"
curl http://localhost:3000/health
```

Then refresh Grafana.

Most likely all graphs will immediately come back.

---

Paste outputs of:

```bash
curl http://localhost:3000/health
```

```bash
docker ps
```

```bash
curl http://localhost:3000/metrics | head
```

and I'll tell you exactly which component is broken.

---

## 🧑 You

done omy image are left measn how files are make memeory or abackaned bottlenek

---

## 🤖 ChatGPT

Yes bro, now you're asking the real backend architect question. 🔥

You are thinking:

```txt id="75g4kb"
How can image uploads create memory bottlenecks?
How can they create backend bottlenecks?
```

Let's compare.

---

# Case 1: Buffer Upload (Bad)

Suppose user uploads:

```txt id="fhlv56"
image.jpg
50 MB
```

Code:

```js id="8rx0bd"
const upload = multer({
  storage: multer.memoryStorage()
});
```

Then:

```txt id="3s7eq7"
Request
   ↓
Entire Image
   ↓
RAM
   ↓
Process
   ↓
Store
```

Memory:

```txt id="8g71k3"
User 1 = 50 MB
User 2 = 50 MB
User 3 = 50 MB
```

10 users:

```txt id="zyjlwm"
500 MB RAM
```

100 users:

```txt id="75as98"
5 GB RAM
```

Boom 💥

Container dies.

---

# What You'll See

Grafana:

```txt id="u74wh0"
Memory ↑↑↑↑↑

GC ↑↑↑↑↑

Latency ↑↑↑

CPU ↑
```

Docker:

```txt id="jjlwm3"
MEMORY
100 MB
500 MB
1 GB
2 GB
```

---

# Case 2: Stream Upload (Good)

```js id="bxm6qg"
req.pipe(file)
```

Flow:

```txt id="hsjlwm"
Request
   ↓
Small Chunk
   ↓
Disk
   ↓
Small Chunk
   ↓
Disk
```

Memory:

```txt id="s5g6au"
5 MB
6 MB
5 MB
7 MB
```

almost constant.

---

# What You'll See

Grafana:

```txt id="4yjlwm"
Memory stable

CPU stable

Disk IO increases
```

---

# Real Production Example

User uploads:

```txt id="6klwhr"
4K Image
100 MB
```

Bad implementation:

```js id="bupynw"
memoryStorage()
```

100 users upload together:

```txt id="g6umg8"
100 * 100MB

10 GB RAM
```

Server crashes.

---

# Another Bottleneck

Image Resize

Suppose:

```txt id="8rj7vs"
100 MB image
```

and use:

```js id="r10e11"
sharp()
```

Flow:

```txt id="x6jlwm"
Upload
Resize
Compress
Generate Thumbnail
```

Now:

```txt id="6cggq4"
CPU bottleneck
```

because image processing is expensive.

Grafana:

```txt id="qjlwm0"
CPU ↑↑↑↑
Memory ↑
Latency ↑↑
```

---

# Another Bottleneck

S3 Upload

Flow:

```txt id="objlwm"
User
 ↓
Node
 ↓
S3
```

If S3 slow:

```txt id="35slnu"
Request waiting
```

Metrics:

```txt id="o2pjit"
Latency ↑↑↑

CPU normal

Memory normal
```

Now you know:

```txt id="mxjlwm"
External dependency issue
```

---

# What Big Companies Do

Instead of:

```txt id="d6jlwm"
Browser
 ↓
Node
 ↓
S3
```

they use:

```txt id="o9ywkl"
Browser
 ↓
S3 Direct Upload
```

Node only creates signed URL.

Result:

```txt id="7jjlwm"
No memory bottleneck
No upload bottleneck
Much cheaper
```

---

# Bottleneck Cheat Sheet

| Scenario | CPU | Memory | Latency |
|-----------|------|---------|---------|
| Buffer Upload | Low | 🔥 High | Medium |
| Stream Upload | Low | Low | Low |
| Image Resize | 🔥 High | Medium | High |
| Memory Leak | Low | 🔥 High | High |
| CPU Spike | 🔥 High | Low | 🔥 High |
| Slow DB | Low | Low | 🔥 High |
| Slow Redis | Low | Low | High |
| Slow External API | Low | Low | 🔥 High |

This is exactly why backend engineers build labs like yours. You trigger one bottleneck at a time and watch **CPU, Memory, Event Loop, and Latency** graphs to learn how each problem looks in production. 🚀

---

## 🧑 You

role :- you are mentor who teach monitoring and monitoring tools in backend sysem 
context 
1. you are teacher who teach how to debug which matix look foe which bottlrneak 
2. he teach  how we do setup in local  as well how real life prodction set up do 
3. how collect matrix which are importnat and also like all matrix well expained and tell with expamle like cpu spike write this query 
4. prometheus set up in prod

---

## 🤖 ChatGPT

Perfect. Let's learn monitoring like a backend engineer who gets paged at 2 AM because production is down.

---

# Phase 1 — Monitoring Mindset

Most beginners think:

```txt id="g8q8lk"
Server slow
→ Check CPU
```

Wrong.

A backend engineer thinks:

```txt id="e0wsyr"
What is the bottleneck?
```

Possible bottlenecks:

```txt id="lh5zdp"
CPU
Memory
Database
Redis
Network
Disk
External API
Event Loop
```

Monitoring exists to identify which one.

---

# Phase 2 — Production Monitoring Architecture

Real production:

```txt id="53mtmc"
Node App
    │
    │ exposes /metrics
    ▼
Prometheus
    │
    │ stores metrics
    ▼
Grafana
    │
    │ dashboards
    ▼
Engineers
```

Logs:

```txt id="ng76ah"
Node
  ↓
Pino
  ↓
Loki / ELK
  ↓
Grafana
```

Metrics:

```txt id="wjlwmg"
Prometheus
```

Logs:

```txt id="jlwmz1"
Pino
```

Tracing:

```txt id="jlwmz2"
OpenTelemetry
Jaeger
Tempo
```

---

# Phase 3 — Local Setup

Your current setup:

```txt id="jlwmz3"
Express
Redis
Prometheus
Grafana
Docker
```

already resembles production.

Local:

```txt id="qjlwm4"
docker compose up
```

Production:

```txt id="jjlwm5"
Kubernetes
Prometheus Operator
Grafana
AlertManager
Loki
Tempo
```

Same concepts.

---

# Phase 4 — Metrics Every Backend Engineer Must Know

Only learn these first.

```txt id="7jlwm6"
CPU
Memory
Latency
Throughput
Error Rate
Event Loop Lag
```

Everything else comes later.

---

# CPU Metrics

Prometheus:

```promql id="jlwm77"
rate(process_cpu_seconds_total[1m]) * 100
```

Meaning:

```txt id="rjlwm8"
How busy CPU is
```

Normal:

```txt id="jlwm89"
10%
20%
30%
```

Problem:

```txt id="8jlwm0"
90%
100%
```

---

### Simulate CPU Bottleneck

Endpoint:

```js id="jlwm91"
for(let i=0;i<50000000;i++){}
```

Run:

```bash id="4jlwm2"
curl localhost:3000/cpu-spike
```

Watch:

```txt id="3jlwm3"
CPU ↑
Latency ↑
Event Loop ↑
```

---

# Memory Metrics

Prometheus:

```promql id="2jlwm4"
process_resident_memory_bytes
```

Convert:

```promql id="1jlwm5"
process_resident_memory_bytes / 1024 / 1024
```

Meaning:

```txt id="0jlwm6"
RAM used by process
```

---

### Simulate Memory Leak

```js id="zjlwm7"
memoryStore.push(hugeData)
```

Watch:

```txt id="yjlwm8"
Memory ↑
GC ↑
Latency ↑
```

---

# Event Loop Lag

Most important Node metric.

Prometheus:

```promql id="xjlwm9"
nodejs_eventloop_lag_p99_seconds * 1000
```

Meaning:

```txt id="wjlwm0"
How delayed Node is
```

Healthy:

```txt id="vjlwm1"
<20 ms
```

Warning:

```txt id="ujlwm2"
100 ms
```

Danger:

```txt id="tjlwm3"
500 ms+
```

---

### Simulate Event Loop Blocking

```js id="sjlwm4"
while(Date.now() < end){}
```

Watch:

```txt id="rjlwm5"
Event Loop ↑
Latency ↑
```

---

# Latency

Prometheus:

```promql id="qjlwm6"
histogram_quantile(
  0.95,
  rate(http_request_duration_seconds_bucket[1m])
)
```

Meaning:

```txt id="pjlwm7"
95% of requests complete within X ms
```

Production teams watch:

```txt id="ojlwm8"
P50
P95
P99
```

---

# Throughput

Prometheus:

```promql id="njlwm9"
rate(http_requests_total[1m])
```

Meaning:

```txt id="mjlwm0"
Requests per second
```

Example:

```txt id="ljlwm1"
100 RPS
500 RPS
1000 RPS
```

---

# Error Rate

Metric:

```txt id="kjlwm2"
5xx responses
```

PromQL:

```promql id="jjlwm3"
rate(http_requests_total{status=~"5.."}[1m])
```

Meaning:

```txt id="ijlwm4"
How many requests fail
```

---

# Phase 5 — Bottleneck Detection Cheat Sheet

## CPU Bottleneck

```txt id="hjlwm5"
CPU ↑↑↑
Latency ↑↑↑
Memory normal
```

Cause:

```txt id="gjlwm6"
Heavy calculations
Image processing
Bad loops
```

---

## Memory Bottleneck

```txt id="fjlwm7"
Memory ↑↑↑
GC ↑↑↑
Latency ↑
```

Cause:

```txt id="ejlwm8"
Memory leak
Huge buffers
Caching mistakes
```

---

## Database Bottleneck

```txt id="djlwm9"
Latency ↑↑↑
CPU normal
Memory normal
```

Cause:

```txt id="cjlwm0"
Slow query
Missing index
```

---

## Redis Bottleneck

```txt id="bjlwm1"
Redis latency ↑
App latency ↑
```

Cause:

```txt id="ajlwm2"
Large keys
Slow commands
```

---

## External API Bottleneck

```txt id="9jlwm3"
Latency ↑↑↑
CPU normal
Memory normal
```

Cause:

```txt id="8jlwm4"
Stripe
Payment gateway
Third-party service
```

---

# Phase 6 — Production Prometheus Setup

Real Kubernetes:

```txt id="7jlwm5"
Node App
    │
    ▼
ServiceMonitor
    │
    ▼
Prometheus
    │
    ▼
AlertManager
    │
    ▼
Slack
```

Example:

```txt id="6jlwm6"
CPU > 90% for 5 min
```

Alert:

```txt id="5jlwm7"
Slack Message

Production API CPU High
```

---

# Phase 7 — Production Dashboard Every Backend Team Has

```txt id="4jlwm8"
Row 1
CPU
Memory
Network
Disk

Row 2
P95 Latency
P99 Latency
RPS
Error Rate

Row 3
Redis
Database
External APIs

Row 4
Business Metrics
Orders
Payments
Signups
```

---

# Learning Order

Week 1:

```txt id="3jlwm9"
Prometheus
Grafana
CPU
Memory
Latency
```

Week 2:

```txt id="2jlwm0"
Pino
Loki
Logs
```

Week 3:

```txt id="1jlwm1"
OpenTelemetry
Tracing
```

Week 4:

```txt id="0jlwm2"
AlertManager
Production Alerts
```

If you master CPU, Memory, Event Loop, Latency, Throughput, and Error Rate first, you'll already understand 80% of what backend engineers use daily to debug production systems.

---

## 🧑 You

role :- yyou are interviewer you wan to test he is have real knowlage of basic of maetrics and bottlenek 
context 
1. you are suppose interviewer and  interviewee as interviwe you wan to test he really work in prod lebet matrix collections 
2. what paartaer he work is this matter 
3. ask qution about parmter tracing and  cheack how he test cpu , memeory database bottleneck 
4. how he test which api really make cpu sk=pike toom cu 
5.

---

## 🤖 ChatGPT

Perfect. If I were interviewing a backend engineer and wanted to verify they actually worked with production monitoring—not just watched a YouTube video—I would ask scenario-based questions.

---

# Round 1 — Metrics Fundamentals

### Q1

Users report:

```txt id="s1"
Application is slow
```

What are the first 5 metrics you check?

Expected:

```txt id="s2"
CPU
Memory
Latency
Error Rate
Request Rate (RPS)
```

Red Flag:

```txt id="s3"
I directly check logs
```

---

### Q2

Difference between:

```txt id="s4"
CPU Usage
Memory Usage
```

What does each indicate?

Expected:

```txt id="s5"
CPU -> computation pressure

Memory -> memory leak, caching, large objects,
large uploads
```

---

### Q3

What is P95 latency?

Expected:

```txt id="s6"
95% requests complete within this duration

5% requests are slower
```

---

### Q4

Why do we track:

```txt id="s7"
P50
P95
P99
```

instead of average latency?

Expected:

```txt id="s8"
Average hides outliers

P95/P99 show slow users
```

---

# Round 2 — Node.js Specific

### Q5

Application CPU is only:

```txt id="s9"
20%
```

but users report slowness.

What next?

Expected:

```txt id="s10"
Check event loop lag
Check DB latency
Check Redis latency
Check external APIs
```

---

### Q6

What is Event Loop Lag?

Expected:

```txt id="s11"
Delay before Node can execute next task

High lag means Node is blocked
```

---

### Q7

Healthy Event Loop Lag?

Expected:

```txt id="s12"
<20ms good

100ms warning

500ms dangerous
```

---

### Q8

How would you intentionally create Event Loop Lag locally?

Expected:

```js id="s13"
while(Date.now() < end){}
```

or

```js id="s14"
for(let i=0;i<100000000;i++){}
```

---

# Round 3 — CPU Bottleneck

### Q9

CPU reaches:

```txt id="s15"
100%
```

Memory:

```txt id="s16"
200MB
```

What is your conclusion?

Expected:

```txt id="s17"
CPU bottleneck

Not memory issue
```

---

### Q10

How do you find which API causes CPU spike?

Expected:

```txt id="s18"
Add route labels

Check latency per route

Check request count per route

Profile application
```

Prometheus example:

```promql id="s19"
http_request_duration_seconds
```

Grouped by:

```txt id="s20"
route
```

---

### Q11

Which tool would you use to profile CPU?

Expected:

```txt id="s21"
Node Inspector

clinic.js

0x

Chrome DevTools
```

---

# Round 4 — Memory Bottleneck

### Q12

Memory increases every hour.

CPU stable.

What do you suspect?

Expected:

```txt id="s22"
Memory leak
```

---

### Q13

How do you verify memory leak?

Expected:

```txt id="s23"
Heap snapshots

Compare snapshots

Look for retained objects
```

---

### Q14

Difference between:

```txt id="s24"
Heap Memory

RSS Memory
```

Expected:

```txt id="s25"
Heap -> V8 objects

RSS -> total process memory
```

---

### Q15

Can image uploads cause memory issues?

Expected:

```txt id="s26"
Yes

memoryStorage()

large buffers

many concurrent uploads
```

---

# Round 5 — Database Bottleneck

### Q16

Latency suddenly becomes:

```txt id="s27"
5 sec
```

CPU:

```txt id="s28"
15%
```

Memory:

```txt id="s29"
20%
```

Where do you look?

Expected:

```txt id="s30"
Database

Redis

External APIs
```

---

### Q17

How do you identify a slow query?

Expected:

```txt id="s31"
EXPLAIN

Query logs

Slow query logs
```

---

### Q18

How do you simulate DB bottleneck locally?

Expected:

```js id="s32"
await new Promise(
  r => setTimeout(r, 5000)
)
```

or heavy query.

---

# Round 6 — Redis Bottleneck

### Q19

Redis latency suddenly becomes:

```txt id="s33"
200ms
```

What could cause it?

Expected:

```txt id="s34"
Large keys

KEYS *

Network issues

Redis CPU saturation
```

---

### Q20

Why is:

```txt id="s35"
KEYS *
```

dangerous?

Expected:

```txt id="s36"
Blocks Redis

O(n) operation
```

---

# Round 7 — Tracing

### Q21

Logs show:

```txt id="s37"
Request took 8 seconds
```

How do you know where time was spent?

Expected:

```txt id="s38"
Tracing
```

---

### Q22

Difference between:

```txt id="s39"
Metrics
Logs
Tracing
```

Expected:

```txt id="s40"
Metrics -> numbers

Logs -> events

Tracing -> request journey
```

---

### Q23

Request Flow

```txt id="s41"
API
 ↓
Redis
 ↓
DB
 ↓
Payment API
```

Which tool shows exact timings?

Expected:

```txt id="s42"
OpenTelemetry

Jaeger

Tempo
```

---

# Round 8 — Production Experience

### Q24

Production alert:

```txt id="s43"
CPU > 90% for 10 min
```

What do you do?

Expected:

```txt id="s44"
Check dashboard

Identify route

Check traffic

Check deployments

Check traces
```

---

### Q25

What dashboards must every backend service have?

Expected:

```txt id="s45"
CPU

Memory

Latency

P95

P99

RPS

Error Rate

Event Loop

DB Latency

Redis Latency
```

---

# Final Killer Question

### Q26

Users say:

```txt id="s46"
Upload API is slow
```

CPU:

```txt id="s47"
20%
```

Memory:

```txt id="s48"
25%
```

DB:

```txt id="s49"
5ms
```

Redis:

```txt id="s50"
2ms
```

What do you check next?

Strong answer:

```txt id="s51"
Tracing

Upload size

Network throughput

S3 latency

Chunked vs buffer upload

Request payload size
```

This question immediately tells whether someone has actually debugged production systems or only knows metrics definitions.

---


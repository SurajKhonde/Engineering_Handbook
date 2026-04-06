# system Design 

## 🧠 UNIVERSAL SYSTEM DESIGN FRAMEWORK

### STEP 1: Clarify Requirements (ALWAYS START HERE)

Say this first:

>[!note]Before jumping into design, I’d like to clarify functional and non-functional requirements.
🔹 1A. Functional Requirements (WHAT system does)

Use this fixed pattern every time:

```sql
Functional Requirements:

1. Users should be able to ______
2. Users should be able to ______
3. System should ______
4. System should return ______
5. Optional features: ______

```

### 🧩 Checklist in your head:

- Who are users? (buyer/seller, sender/receiver)
- What actions they perform?
- What system does internally?
- What response system gives?

🔹 1B. Non-Functional Requirements (HOW system behaves)

```js

Non-Functional Requirements:

1. Scale: ______ users / requests
2. Latency: ______ (ms/sec)
3. Availability vs Consistency: ______
4. Read vs Write heavy: ______
5. Storage: ______
6. Security: ______

```

🧩 MUST ask these 5 things (VERY IMPORTANT 🔥)

- Scale (users, QPS)
- Latency requirement
- Read vs Write ratio
- Availability vs Consistency
- Data size
### STEP 2: High-Level Design (Big Picture)

Draw this mentally:
``Client → Load Balancer → Servers → DB + Cache ``

Explain flow:

- Request comes
- Goes to server
- Server hits cache/DB
- Returns response
  
### STEP 3: Core Component Deep Dive

Pick 1–2 critical parts:

Examples:

- URL shortener → ID generation
- WhatsApp → message delivery
- Instagram → feed generation
- E-commerce → inventory

### STEP 4: Database Design

Define:

- Tables / collections
- Key fields

Example:

```js
User(id, name)
Post(id, userId)
```

### STEP 5: Scaling (🔥 MOST IMPORTANT)

Talk about:

- Caching (Redis)
- Load balancing
- DB sharding
- CDN
- Queue (Kafka/RabbitMQ)
  
### STEP 6: Bottlenecks & Tradeoffs

Say things like:

- “Cache can become stale”
- “DB can be bottleneck”
- “We trade consistency for availability”


`WHO → WHAT → HOW BIG → HOW FAST → HOW RELIABLE`
✅ 1. WHO (Actors)

Think:
`System use kaun karega?`

Examples:
```js
User / Admin / Seller / Buyer
Sender / Receiver (WhatsApp)
Creator / Viewer (YouTube)
```

🎯 What to say:
Actors:

- End users (buyers/viewers/etc.)
- Optional: admin/seller/creator

2. WHAT (Functional Requirements)

Think:

`User kya karega? System kya karega?`

🎯 Structure:
Functional Requirements:

1. Users should be able to ______
2. Users should be able to ______
3. System should ______
4. System should return ______
🧩 Mentally check:
Create?
Read?
Update?
Delete?
Interaction? (like, comment, chat)

3. HOW BIG (Scale)

Think:

“Kitna load ayega?”

🎯 Say like this:
Scale:

 - X million users
 - Y requests per second
 - Z data per day

💡 Example:
   100M users
   1M daily active users
    10K requests/sec

4. HOW FAST (Performance)

Think:

“Kitni fast response chahiye?”

🎯 Say:
Latency:
 - <100ms for API
 - Real-time (<1 sec) if needed
Examples:
  WhatsApp → real-time ⚡
  E-commerce → few 100ms ok

 5. HOW RELIABLE (Non-Functional Core)

Think:

`System kitna strong hona chahiye?`

🎯 Cover these 3 ALWAYS:

1. Availability vs Consistency
   Banking → consistency
   Social apps → availability
2. Read vs Write
   Instagram → read-heavy
   WhatsApp → write-heavy
3. Durability

Data loss allowed or not?
🎯 Say:

System Requirements:

 - High availability
 - Eventually consistent (if ok)
 - Read-heavy / Write-heavy
 - No data loss

🔥 HOW TO SPEAK (FINAL FLOW)

In interview, just say:

Let me clarify requirements:

WHO:
- Users are ______

WHAT:
- Users should be able to ______

HOW BIG:
- We are designing for ______ users

HOW FAST:
- Latency should be ______

HOW RELIABLE:
- System should be highly available
- It is read-heavy / write-heavy
- We can allow eventual consistency

💥 This sounds VERY senior.

🧠 Why This Works (Important Insight)

This framework automatically prepares you for:

Architecture decisions
DB choice
Caching
Scaling

👉 Because everything depends on:
scale + latency + consistency
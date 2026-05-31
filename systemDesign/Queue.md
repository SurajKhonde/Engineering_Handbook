# Queue
Queue is a FIFO-based message buffer used to decouple producers and consumers, enabling asynchronous processing, scalability, and fault tolerance.

 **buffer** is simply:
A temporary holding area for data while it is being transferred or processed.

Queue as Buffer (System Design context)

Here buffer means:

`Store requests temporarily until they are processed`

Data sits in queue
Workers process it one by one
Can be stored in:
RAM (Redis)
Disk (Kafka)
Managed service (SQS)

Buffer solves speed mismatch problem

Example:

API gets 1000 requests/sec
Worker can handle 100/sec

Queue (buffer) absorbs the extra 900

#### What is inside a Queue?

A queue doesn’t store **magic** — it stores data.

That data can be:

- A message
- A job/task
- An event

👉 The name depends on what kind of data you’re putting insid📩 1. Message Queue
👉 What is a "message"?

A message = some data sent from one system to another

Example:

{
  "userId": 101,
  "action": "signup"
}
🧩 Flow:
Service A → (message) → Queue → Service B
📌 Why called message queue?

Because:

👉 Systems are communicating via messages

💡 Real Example:
User signs up
Auth service sends message:
“User created”
Email service consumes it
🧰 Tools:
RabbitMQ
AWS SQS
⚙️ 2. Job Queue
👉 What is a "job"?

A job = a task that needs to be executed

Example:

{
  "type": "generate_pdf",
  "reportId": 55
}
🧩 Flow:
App → (job) → Queue → Worker executes it
📌 Why called job queue?

Because:

👉 You are assigning work to workers

💡 Real Example:
Generate PDF
Resize image
Send email
🧰 Tools:
BullMQ (Node.js + Redis)
Sidekiq (Ruby)
⚡ 3. Event Queue
👉 What is an "event"?

An event = something that already happened

Example:

{
  "event": "order_placed",
  "orderId": 999
}
🧩 Flow:
System → emits event → multiple consumers react
📌 Why called event queue?

Because:

👉 It represents a fact that occurred in the system

💡 Real Example:
Order placed
Payment successful
User logged in
🧰 Tools:
Kafka (most common)
EventBridge
🔥 KEY DIFFERENCE (this is interview gold)
Type	Meaning	Focus
Message Queue	Data communication	Send info between services
Job Queue	Work to be done	Execute task
Event Queue	Something happened	React to system events
🎯 SUPER SIMPLE WAY TO REMEMBER
📩 Message → “Here is some data”
⚙️ Job → “Do this work”
⚡ Event → “This already happened”e

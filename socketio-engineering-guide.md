# Socket.IO — Engineering Guide (HTTP vs Sockets, TCP/WebSocket Handshake, `io` vs `socket`, `on/emit`, Rooms)

> **Goal:** You already know REST (GET/POST). This explains Socket.IO from the **network layer** up, then teaches the core APIs:  
> `io.on("connection")`, `socket.on(...)`, `socket.emit(...)`, `join`, `to`, `broadcast`, namespaces, acknowledgements, etc.

---

## 0) Where Socket.IO sits (network layers)

### OSI / TCP-IP quick view
```
Application  ← HTTP, WebSocket, Socket.IO (events)
Transport    ← TCP (reliable connection) / UDP
Network      ← IP
Link         ← Ethernet / Wi‑Fi
Physical     ← Cable / Radio
```

- **TCP** is the transport that gives you a reliable byte stream.
- **HTTP** and **WebSocket** are **application-layer protocols** that run **on top of TCP**.
- **Socket.IO** is an **application-layer library/protocol** that typically runs:
  - on top of **WebSocket** (preferred)
  - or falls back to **HTTP long-polling** (via Engine.IO)

---

## 1) TCP connection: how the “keep it open” part works

### 1.1 TCP 3-way handshake (connection setup)
Before any HTTP/WebSocket traffic, TCP is established:

1. Client → Server: **SYN**
2. Server → Client: **SYN-ACK**
3. Client → Server: **ACK**

Now both sides have a TCP connection (a reliable stream).

### 1.2 Keeping it alive
Once TCP is established, both parties can send/receive without creating a new connection each time.

- If nothing is sent for a long time, NAT/firewalls can drop it.
- To avoid that, protocols use:
  - **heartbeat / ping-pong**
  - TCP keepalive (OS-level)
  - application-level ping/pong (WebSocket/Engine.IO does this)

### 1.3 Closing
Connection closes by:
- clean close: FIN/ACK exchange
- or sudden: RST, network drop, process crash

Socket.IO detects disconnects and can reconnect automatically.

---

## 2) HTTP model (GET/POST) vs Socket model (events)

### 2.1 HTTP is request/response
- You send a request (`GET /posts`, `POST /signup`)
- Server returns **one response** and the “transaction” is done.

Even if HTTP keep-alive reuses the same TCP connection, the **application model** is still:
> Client asks → Server answers → done.

### 2.2 WebSocket / Socket.IO is a persistent two-way channel
After a socket connection is established:
- Client and server can **push messages anytime**
- No need to “request” for every update
- Great for chat, live dashboards, games, presence, typing indicators, etc.

### 2.3 Mapping REST thinking to sockets
HTTP verbs vs socket events are not 1:1, but you can think:

| REST | Socket.IO |
|------|-----------|
| `GET /posts` | `socket.emit("posts:get", { ... })` and server `socket.emit("posts:list", ...)` |
| `POST /posts` | `socket.emit("posts:create", { ... })` and ack/response |
| Status code | ack payload `{ ok: true }` or error event |

Socket.IO uses **events** instead of URLs + verbs.

---

## 3) WebSocket handshake (HTTP Upgrade)

WebSocket starts as HTTP, then upgrades:

1. Client sends HTTP request with headers like:
   - `Upgrade: websocket`
   - `Connection: Upgrade`
2. Server replies `101 Switching Protocols`
3. After that, HTTP is done; both sides exchange WebSocket frames (full duplex).

So: **WebSocket = persistent channel negotiated via HTTP**

---

## 4) What Socket.IO adds (compared to raw WebSocket)

Socket.IO is not “just WebSocket”. It adds:
- **Event names** (`"chat:message"`, `"typing"`) + JSON payloads
- **Auto reconnection**
- **Rooms** and **Namespaces**
- **Acknowledgements** (acks) with callbacks/timeouts
- **Fallback transport** (if WebSocket not possible): HTTP long-polling (Engine.IO)
- **Middleware** on the socket handshake

Under the hood:
- Socket.IO uses **Engine.IO** which manages:
  - initial handshake
  - transport choice (polling → upgrade to websocket)
  - ping/pong heartbeats

---

## 5) Connection lifecycle in Socket.IO (how it really connects)

### 5.1 Typical flow
1. Client loads page and runs:
   ```js
   const socket = io("http://localhost:5000");
   ```
2. Engine.IO does an initial HTTP request (often polling) to establish a session.
3. Server assigns a session id and Socket.IO creates a **socket** with `socket.id`.
4. If possible, Engine.IO upgrades to WebSocket.
5. Server fires:
   ```js
   io.on("connection", (socket) => { ... })
   ```

### 5.2 Why you may see “polling”
If WebSocket is blocked, it falls back to long-polling:
- client makes a request that hangs (long poll)
- server responds when it has data
- client immediately opens another long poll

This still feels “real-time” but with more overhead.

---

## 6) `io` vs `socket` (your key question)

### 6.1 `io` = the server instance (the “whole hub”)
Created once:
```js
const io = new Server(httpServer);
```

- Represents the entire Socket.IO server
- Knows all sockets, rooms, namespaces
- Used for “broadcast to everyone” or “to a room”

Examples:
```js
io.emit("system", { message: "Hello everyone" });      // all clients
io.to("room1").emit("msg", { ... });                   // all clients in room1
```

### 6.2 `socket` = one connected client (one connection)
Inside:
```js
io.on("connection", (socket) => {
  // socket represents THIS user/client only
});
```

- Represents one user connection
- Has an id: `socket.id`
- Stores per-user server-side data: `socket.data`
- Used to receive from that client and send to that client

Examples:
```js
socket.emit("welcome", { id: socket.id });             // only this client
socket.on("chat:message", (data) => { ... });          // listen from this client
```

---

## 7) Why `io.on("connection")` and then `socket.on(...)`?

### 7.1 `io.on("connection", handler)`
This is a **server-level listener**:
- It triggers **once per client** when they connect.
- It gives you a `socket` object for that client.

```js
io.on("connection", (socket) => {
  console.log("connected:", socket.id);
});
```

### 7.2 `socket.on("event", handler)`
This is a **per-client listener**:
- It listens to events coming **from that client**.

```js
socket.on("user:setName", ({ username }) => {
  socket.data.username = username;
});
```

**Summary:**
- `io.on("connection")` → “a new client arrived”
- `socket.on(...)` → “this client sent an event”

---

## 8) Core APIs: `on` and `emit` (what they mean)

### 8.1 `on(event, handler)` = listen/receive
```js
socket.on("chat:message", (payload) => {
  console.log("client sent message:", payload);
});
```

### 8.2 `emit(event, data)` = send
```js
socket.emit("chat:message", { text: "hi" });
```

### 8.3 Same event name on both sides
Client:
```js
socket.emit("chat:message", { text: "hi" });
socket.on("system", (data) => console.log(data));
```

Server:
```js
socket.on("chat:message", (data) => { ... });
socket.emit("system", { message: "welcome" });
```

---

## 9) Broadcast patterns (most important practical part)

### 9.1 Send only to the same client
```js
socket.emit("event", data);
```

### 9.2 Send to everyone (including sender)
```js
io.emit("event", data);
```

### 9.3 Send to everyone except sender
```js
socket.broadcast.emit("event", data);
```

### 9.4 Rooms
A room is just a **group label** sockets can join.

Join:
```js
socket.join("room1");
```

Leave:
```js
socket.leave("room1");
```

### 9.5 Send to everyone in a room (including sender)
```js
io.to("room1").emit("event", data);
```

### 9.6 Send to everyone in a room except sender
```js
socket.to("room1").emit("event", data);
```

**Remember:**
- `io.to(room)` includes sender (if sender is in that room)
- `socket.to(room)` excludes sender

---

## 10) Acknowledgements (acks): “response” like HTTP

Acks let you do request/response over sockets.

Client:
```js
socket.emit("posts:create", { title: "Hi" }, (ack) => {
  console.log("server replied:", ack);
});
```

Server:
```js
socket.on("posts:create", async (data, reply) => {
  try {
    // do work
    reply({ ok: true, id: "123" });
  } catch (e) {
    reply({ ok: false, error: "failed" });
  }
});
```

This is the socket equivalent of HTTP status + JSON response.

---

## 11) Namespaces (multiple “apps” on same server)

Namespace = separate channel like `/chat`, `/notifications`.

Server:
```js
const chat = io.of("/chat");
chat.on("connection", (socket) => { ... });
```

Client:
```js
const chatSocket = io("/chat");
```

Rooms exist **inside** a namespace.

---

## 12) Typical server structure (best practice)

```js
io.on("connection", (socket) => {
  // 1) authenticate (optional)
  // 2) set socket.data (userId, username)
  // 3) join rooms
  // 4) register event listeners
});
```

### Using `socket.data`
`socket.data` is per-connection memory:
```js
socket.data.userId = "abc";
socket.data.username = "Suraj";
```

---

## 13) How “keep connection open” actually behaves

- TCP connection stays open.
- Engine.IO sends ping/pong heartbeats.
- If network drops, server emits disconnect; client tries reconnect.
- When reconnect happens, **socket.id can change** (important).
- You may need to re-join rooms after reconnect.

---

## 14) Security notes (don’t trust the client)

In beginner demos we accept:
```js
socket.emit("user:setName", { username: "Suraj" });
```

In production, you should:
1. Send JWT in handshake:
   ```js
   const socket = io("...", { auth: { token: "JWT..." } });
   ```
2. Verify on server:
   ```js
   io.use((socket, next) => {
     const token = socket.handshake.auth?.token;
     // verify token
     socket.data.userId = ...
     next();
   });
   ```

This prevents “I am admin” cheating.

---

## 15) Scaling (multiple server instances)

If you run multiple Node servers:
- rooms/broadcast need a shared adapter
- use Redis adapter:
  - `@socket.io/redis-adapter`

Also store room membership/invites in DB/Redis, not in memory.

---

## 16) Quick cheat sheet

**Listen**
- `io.on("connection", socket => {})` : new client
- `socket.on("event", handler)` : event from that client

**Send**
- `socket.emit(...)` : only this client
- `io.emit(...)` : everyone
- `socket.broadcast.emit(...)` : everyone except sender

**Rooms**
- `socket.join(room)`
- `io.to(room).emit(...)` : room + sender
- `socket.to(room).emit(...)` : room - sender

**Other**
- `io.of("/ns")` : namespace
- ack callbacks: `socket.emit("evt", data, (ack)=>{})`

---

## 17) Minimal working example (server + client)

### Server (Node)
```js
io.on("connection", (socket) => {
  socket.on("ping", (data, ack) => {
    ack({ ok: true, got: data, at: Date.now() });
  });
});
```

### Client (Browser)
```js
socket.emit("ping", { x: 1 }, (ack) => console.log(ack));
```

---

## What to learn next
1. Rooms + invite-only rooms
2. Auth in handshake (JWT)
3. Message delivery ack + retries
4. Redis adapter scaling

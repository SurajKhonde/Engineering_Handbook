## socket.io (websocket)
- Socket.IO is a library that enables *low-latency*, *bidirectional* and *event-based* communication between a client and a server.
The Socket.IO connection can be established with different low-level transports:

- HTTP long-polling
- WebSocket
- WebTransport
Socket.IO will automatically pick the best available option, depending on:
- the capabilities of the browser (see here and here)
- the network (some networks block WebSocket and/or WebTransport connections)
> ! note
> Socket.IO is NOT a WebSocket implementation.
#### Automatic reconnection
Under some particular conditions, the WebSocket connection between the server and the client can be interrupted with both sides being   unaware of the broken state of the link.
That's why Socket.IO includes a heartbeat mechanism, which periodically checks the status of the connection.
And when the client eventually gets disconnected, it automatically reconnects with an exponential back-off delay, in order not to overwhelm the server.
#### Packet buffering
The packets are automatically buffered when the client is disconnected, and will be sent upon reconnection.

##### Sender
```js
socket.emit("hello", "world", (response) => {
  console.log(response); // "got it"
});
```
##### Receiver
```js
socket.on("hello", (arg, callback) => {
  console.log(arg); // "world"
  callback("got it");
});
```
```js
socket.timeout(5000).emit("hello", "world", (err, response) => {
  if (err) {
    // the other side did not acknowledge the event in the given delay
  } else {
    console.log(response); // "got it"
  }
});
```
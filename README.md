# Socket.IO with Node.js

A practical implementation of **real-time, bidirectional communication** using **Node.js, Express, and Socket.IO**.

This repository demonstrates how to integrate Socket.IO into a Node.js server and establish real-time communication between the server and connected clients.

## 📌 What is Socket.IO?

[Socket.IO](https://socket.io/) is a library that enables real-time, bidirectional communication between a client and a server.

Unlike traditional HTTP requests, where the client sends a request and waits for a response, Socket.IO allows the server and client to communicate continuously through **events**.

### Traditional HTTP

```text
Client → Request → Server
Client ← Response ← Server
```

### Socket.IO

```text
Client ←──────────────→ Server
       Real-time events
```

This makes Socket.IO useful for applications such as:

* 💬 Real-time chat applications
* 🔔 Notifications
* 📊 Live dashboards
* 🎮 Multiplayer applications
* 📍 Live location tracking
* 👥 Online/offline user status
* 📡 Real-time monitoring
* 🤝 Collaborative applications

---

# 🚀 Tech Stack

* **Node.js**
* **Express.js**
* **Socket.IO**
* **JavaScript**

---

# 📂 Project Structure

```text
socket-io-node/
│
├── server/
│   └── server.js
│
├── client/
│   └── index.html
│
├── package.json
├── package-lock.json
└── README.md
```

The exact structure can be adjusted depending on the project requirements.

---

# ⚙️ Installation

Clone the repository:

```bash
git clone <your-repository-url>
```

Navigate into the project:

```bash
cd socket-io-node
```

Install dependencies:

```bash
npm install
```

Start the server:

```bash
npm start
```

For development, you can use:

```bash
npm run dev
```

---

# 🔌 Installing Socket.IO

Install Socket.IO using npm:

```bash
npm install socket.io
```

If you are creating a separate frontend application, install the Socket.IO client there as well:

```bash
npm install socket.io-client
```

---

# 🖥️ Socket.IO Server Implementation

The first step is to create an HTTP server using Node.js and attach Socket.IO to it.

```javascript
const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();

const server = http.createServer(app);

const io = new Server(server, {
  cors: {
    origin: "*",
  },
});

io.on("connection", (socket) => {
  console.log(`User connected: ${socket.id}`);

  socket.on("disconnect", () => {
    console.log(`User disconnected: ${socket.id}`);
  });
});

server.listen(3000, () => {
  console.log("Server running on port 3000");
});
```

## How it works

### 1. Create an Express application

```javascript
const app = express();
```

Express handles the normal HTTP requests of the application.

### 2. Create an HTTP server

```javascript
const server = http.createServer(app);
```

Socket.IO needs access to the underlying HTTP server.

### 3. Initialize Socket.IO

```javascript
const io = new Server(server);
```

This attaches Socket.IO to the HTTP server.

### 4. Listen for connections

```javascript
io.on("connection", (socket) => {
    // socket connection logic
});
```

The `connection` event is triggered whenever a new client connects.

Each connected client receives a unique socket ID:

```javascript
socket.id
```

---

# 🌐 Client Implementation

A client can connect to the Socket.IO server using the Socket.IO client library.

```html
<!DOCTYPE html>
<html>
<head>
  <title>Socket.IO Client</title>
</head>

<body>

  <h1>Socket.IO Demo</h1>

  <script src="/socket.io/socket.io.js"></script>

  <script>
    const socket = io();

    socket.on("connect", () => {
      console.log("Connected to server:", socket.id);
    });

    socket.on("disconnect", () => {
      console.log("Disconnected from server");
    });
  </script>

</body>
</html>
```

When the page loads, the client establishes a Socket.IO connection with the server.

---

# 📡 Socket.IO Events

Socket.IO communication is based on **events**.

An event can be emitted by either the client or the server.

## Emit an Event

```javascript
socket.emit("message", "Hello Server");
```

## Listen for an Event

```javascript
socket.on("message", (message) => {
  console.log(message);
});
```

The basic pattern is:

```text
emit()  → send an event

on()    → listen for an event
```

---

# 🔄 Client → Server Communication

The client can send an event to the server.

### Client

```javascript
socket.emit("sendMessage", {
  username: "John",
  message: "Hello Server!",
});
```

### Server

```javascript
socket.on("sendMessage", (data) => {
  console.log(data);

  console.log(data.username);
  console.log(data.message);
});
```

---

# 🔄 Server → Client Communication

The server can also send an event to a client.

```javascript
socket.emit("message", {
  message: "Hello Client!",
});
```

The client listens for the event:

```javascript
socket.on("message", (data) => {
  console.log(data.message);
});
```

---

# 📢 Broadcasting Messages

Sometimes you want to send a message to **all connected clients except the sender**.

Use:

```javascript
socket.broadcast.emit("message", {
  message: "A new user joined the application",
});
```

Example:

```javascript
io.on("connection", (socket) => {

  socket.on("sendMessage", (message) => {

    socket.broadcast.emit("receiveMessage", message);

  });

});
```

### Communication flow

```text
Client A
   │
   │ sendMessage
   ▼
Server
   │
   ├──────────→ Client B
   ├──────────→ Client C
   └──────────→ Client D
```

Client A does not receive its own broadcast.

---

# 📣 Sending Data to Everyone

If the server needs to send an event to **all connected clients**, use:

```javascript
io.emit("notification", {
  message: "Server notification",
});
```

This includes the sender.

```text
             Server
          /    |    \
         ↓     ↓     ↓
      Client A B     C
```

---

# 👥 Socket.IO Rooms

Rooms allow clients to be grouped together.

They are especially useful for:

* Chat rooms
* Private conversations
* Multiplayer games
* Project-specific notifications
* Organization-based communication

## Join a Room

```javascript
socket.join("room-1");
```

## Send Message to a Room

```javascript
io.to("room-1").emit("message", {
  message: "Hello room!",
});
```

The message will only be delivered to clients currently inside `room-1`.

---

# 💬 Example Chat Implementation

### Server

```javascript
const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();

const server = http.createServer(app);

const io = new Server(server, {
  cors: {
    origin: "*",
  },
});

io.on("connection", (socket) => {

  console.log("User connected:", socket.id);

  socket.on("joinRoom", (room) => {

    socket.join(room);

    console.log(`${socket.id} joined ${room}`);

  });

  socket.on("sendMessage", ({ room, message }) => {

    io.to(room).emit("receiveMessage", {
      user: socket.id,
      message,
    });

  });

  socket.on("disconnect", () => {

    console.log("User disconnected:", socket.id);

  });

});

server.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

### Client

```javascript
const socket = io("http://localhost:3000");

socket.emit("joinRoom", "room-1");

socket.emit("sendMessage", {
  room: "room-1",
  message: "Hello everyone!",
});

socket.on("receiveMessage", (data) => {
  console.log(`${data.user}: ${data.message}`);
});
```

---

# 🔐 Authentication

Socket.IO connections can also be authenticated.

The client can send authentication information during connection:

```javascript
const socket = io("http://localhost:3000", {
  auth: {
    token: "your-auth-token",
  },
});
```

The server can access the authentication data using middleware:

```javascript
io.use((socket, next) => {

  const token = socket.handshake.auth.token;

  if (!token) {
    return next(new Error("Authentication required"));
  }

  // Validate token here

  next();
});
```

This approach can be combined with authentication systems such as JWT.

---

# 🧩 Socket.IO Middleware

Middleware allows you to execute logic before a socket connection is established.

For example:

```javascript
io.use((socket, next) => {

  console.log("Authenticating socket...");

  // Authentication / validation logic

  next();
});
```

If authentication fails:

```javascript
next(new Error("Unauthorized"));
```

This is useful for:

* Authentication
* Authorization
* Token validation
* Logging
* Request validation

---

# 🔌 Connection Lifecycle

A typical Socket.IO connection follows this lifecycle:

```text
Client
   │
   │ connect
   ▼
Socket.IO Server
   │
   │ connection event
   ▼
Socket Created
   │
   ├── emit events
   ├── receive events
   ├── join rooms
   └── broadcast events
   │
   │ disconnect
   ▼
Connection Closed
```

The server can monitor connection and disconnection events:

```javascript
io.on("connection", (socket) => {

  console.log("Connected:", socket.id);

  socket.on("disconnect", (reason) => {
    console.log("Disconnected:", socket.id);
    console.log("Reason:", reason);
  });

});
```

---

# 🆚 HTTP vs Socket.IO

| HTTP                                      | Socket.IO                         |
| ----------------------------------------- | --------------------------------- |
| Request/response                          | Event-based                       |
| Client initiates communication            | Client and server can communicate |
| Usually short-lived requests              | Persistent connection             |
| Good for REST APIs                        | Good for real-time communication  |
| Example: GET /users                       | Example: `socket.emit()`          |
| Server doesn't automatically push updates | Server can push events            |

Socket.IO does **not replace REST APIs**.

A common production architecture uses both:

```text
                Node.js Server
                /           \
               /             \
          REST API        Socket.IO
             │                │
             ▼                ▼
       CRUD Operations    Real-time Events
```

For example:

* REST API → Login, registration, CRUD operations
* Socket.IO → Chat messages, notifications, live updates

---

# 🏗️ Recommended Architecture

For a larger Node.js application, keep Socket.IO event handling separate from your main server configuration.

```text
src/
│
├── server.js
│
├── socket/
│   ├── index.js
│   ├── chat.socket.js
│   └── notification.socket.js
│
├── controllers/
├── routes/
├── services/
├── models/
└── middleware/
```

Example:

```javascript
// socket/index.js

module.exports = (io) => {

  io.on("connection", (socket) => {

    console.log("Socket connected:", socket.id);

    socket.on("disconnect", () => {
      console.log("Socket disconnected:", socket.id);
    });

  });

};
```

Then initialize it from the server:

```javascript
const socketHandler = require("./socket");

socketHandler(io);
```

This keeps the Socket.IO implementation modular and easier to maintain.

---

# 🛠️ Best Practices

### 1. Use meaningful event names

Prefer:

```javascript
"message:send"
"user:online"
"notification:new"
"room:join"
```

Instead of generic names such as:

```javascript
"data"
"event"
"update"
```

### 2. Validate incoming data

Never blindly trust data received from clients.

```javascript
socket.on("message:send", (data) => {

  if (!data?.message) {
    return;
  }

  // Process message

});
```

### 3. Authenticate connections

Use Socket.IO middleware to validate authentication tokens.

### 4. Keep socket handlers modular

Avoid putting every event inside one large `connection` callback.

### 5. Handle disconnects

Always consider what should happen when a user loses their connection.

### 6. Use rooms for targeted communication

Instead of broadcasting everything to everyone, use rooms when only a subset of users needs an event.

### 7. Don't use Socket.IO for everything

Use REST APIs or other HTTP APIs for standard CRUD operations and Socket.IO where real-time communication is actually required.

---

# 🚀 Running the Project

Start the development server:

```bash
npm run dev
```

Or:

```bash
npm start
```

The server should be available at:

```text
http://localhost:3000
```

Open the client application and check the browser/server console to verify the Socket.IO connection.

---

# 📚 Learning Resources

* [Socket.IO Documentation](https://socket.io/docs/v4/?utm_source=chatgpt.com)
* [Socket.IO Server API](https://socket.io/docs/v4/server-api/?utm_source=chatgpt.com)
* [Socket.IO Client API](https://socket.io/docs/v4/client-api/?utm_source=chatgpt.com)

---

# 🎯 Project Goal

The goal of this repository is to provide a clear and practical example of implementing **Socket.IO with a Node.js server**, including:

* Socket connection handling
* Client/server communication
* Custom events
* Broadcasting
* Rooms
* Authentication middleware
* Disconnect handling
* Scalable project structure

This implementation can serve as a foundation for building production-ready real-time features in Node.js applications.

---

# 📄 License

This project is available for learning and development purposes.

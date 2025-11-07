# WebSocket এবং Real-time Communication - Socket.io

## সংক্ষিপ্ত পরিচিতি (Brief Overview)

**Real-time communication** হলো এমন একটি প্রযুক্তি যেখানে server এবং client এর মধ্যে instant, bidirectional (দ্বিমুখী) data transfer হয়। Express.js এর সাথে **Socket.io** ব্যবহার করে real-time features তৈরি করা যায়।

### এই Lesson এ যা শিখবেন:
- ✅ WebSocket কী এবং কেন ব্যবহার করবেন
- ✅ Socket.io installation এবং setup (ES6 syntax)
- ✅ Real-time chat application তৈরি
- ✅ Rooms এবং Namespaces
- ✅ Authentication with Socket.io
- ✅ Broadcasting এবং Private messaging
- ✅ Error handling এবং reconnection
- ✅ Scaling Socket.io applications

---

## WebSocket কী?

**WebSocket** হলো একটি communication protocol যা:
- Full-duplex (দুই দিকে একসাথে data transfer)
- Persistent connection (connection open থাকে)
- Real-time data exchange
- HTTP এর তুলনায় কম overhead

### HTTP vs WebSocket

```
HTTP Request-Response:
Client → Request → Server
Client ← Response ← Server
(Connection closes)

WebSocket:
Client ↔ Persistent Connection ↔ Server
(Both can send data anytime)
```

### কখন WebSocket ব্যবহার করবেন?

✅ **উপযুক্ত:**
- Chat applications
- Live notifications
- Real-time analytics dashboards
- Collaborative editing (Google Docs)
- Multiplayer games
- Live streaming data (stock prices, sports scores)
- IoT device monitoring

❌ **উপযুক্ত নয়:**
- Simple form submissions
- Static content delivery
- RESTful API calls (যেখানে real-time দরকার নেই)
- File uploads/downloads

---

## Socket.io কী?

**Socket.io** হলো একটি JavaScript library যা:
- WebSocket এর উপর built
- Automatic reconnection
- Room এবং namespace support
- Broadcasting capabilities
- Fallback to HTTP long-polling (যদি WebSocket support না থাকে)
- Cross-browser compatibility

---

## Installation এবং Setup

### Step 1: Dependencies Install করুন

```bash
# Express এবং Socket.io install
npm install express socket.io

# Additional dependencies
npm install cors dotenv
```

### Step 2: package.json Configuration

```json
{
  "name": "socketio-chat-app",
  "version": "1.0.0",
  "type": "module",
  "description": "Real-time chat application with Socket.io",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^5.1.0",
    "socket.io": "^4.7.2",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5"
  },
  "devDependencies": {
    "nodemon": "^3.1.0"
  }
}
```

---

## Basic Socket.io Server Setup (ES6)

### `server.js`:

```javascript
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';
import { fileURLToPath } from 'url';
import { dirname, join } from 'path';

// ES6 modules এ __dirname পাওয়ার জন্য
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

// Express app
const app = express();
const httpServer = createServer(app);

// Socket.io server
const io = new Server(httpServer, {
  cors: {
    origin: '*', // Production এ specific domain দিন
    methods: ['GET', 'POST']
  }
});

// Middleware
app.use(express.static(join(__dirname, 'public')));

// Routes
app.get('/', (req, res) => {
  res.sendFile(join(__dirname, 'public', 'index.html'));
});

// Socket.io connection
io.on('connection', (socket) => {
  console.log(`✅ User connected: ${socket.id}`);

  // Message receive করা
  socket.on('message', (data) => {
    console.log('Message received:', data);
    
    // সবাইকে message পাঠানো (broadcast)
    io.emit('message', data);
  });

  // User disconnect
  socket.on('disconnect', () => {
    console.log(`❌ User disconnected: ${socket.id}`);
  });
});

// Server start
const PORT = process.env.PORT || 3000;
httpServer.listen(PORT, () => {
  console.log(`🚀 Server running on http://localhost:${PORT}`);
});
```

---

## Client-side Setup

### `public/index.html`:

```html
<!DOCTYPE html>
<html lang="bn">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Real-time Chat</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    
    .chat-container {
      width: 90%;
      max-width: 600px;
      height: 80vh;
      background: white;
      border-radius: 15px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.3);
      display: flex;
      flex-direction: column;
    }
    
    .chat-header {
      background: #667eea;
      color: white;
      padding: 20px;
      border-radius: 15px 15px 0 0;
      text-align: center;
    }
    
    #messages {
      flex: 1;
      overflow-y: auto;
      padding: 20px;
      background: #f5f5f5;
    }
    
    .message {
      margin-bottom: 15px;
      padding: 10px 15px;
      background: white;
      border-radius: 10px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
      animation: slideIn 0.3s ease;
    }
    
    @keyframes slideIn {
      from {
        opacity: 0;
        transform: translateY(20px);
      }
      to {
        opacity: 1;
        transform: translateY(0);
      }
    }
    
    .message strong {
      color: #667eea;
    }
    
    .input-container {
      display: flex;
      padding: 20px;
      background: white;
      border-radius: 0 0 15px 15px;
      border-top: 1px solid #eee;
    }
    
    #messageInput {
      flex: 1;
      padding: 12px 15px;
      border: 2px solid #667eea;
      border-radius: 25px;
      font-size: 14px;
      outline: none;
    }
    
    #sendButton {
      margin-left: 10px;
      padding: 12px 25px;
      background: #667eea;
      color: white;
      border: none;
      border-radius: 25px;
      cursor: pointer;
      font-weight: bold;
      transition: background 0.3s;
    }
    
    #sendButton:hover {
      background: #5568d3;
    }
    
    .status {
      text-align: center;
      padding: 10px;
      font-size: 12px;
      color: #666;
    }
  </style>
</head>
<body>
  <div class="chat-container">
    <div class="chat-header">
      <h2>🚀 Real-time Chat</h2>
      <div class="status" id="status">Connected</div>
    </div>
    
    <div id="messages"></div>
    
    <div class="input-container">
      <input 
        type="text" 
        id="messageInput" 
        placeholder="Type your message..."
        autocomplete="off"
      />
      <button id="sendButton">Send</button>
    </div>
  </div>

  <!-- Socket.io client library -->
  <script src="/socket.io/socket.io.js"></script>
  <script>
    // Socket connection
    const socket = io();
    
    // Elements
    const messagesDiv = document.getElementById('messages');
    const messageInput = document.getElementById('messageInput');
    const sendButton = document.getElementById('sendButton');
    const statusDiv = document.getElementById('status');
    
    // Connection status
    socket.on('connect', () => {
      statusDiv.textContent = '✅ Connected';
      statusDiv.style.color = 'green';
    });
    
    socket.on('disconnect', () => {
      statusDiv.textContent = '❌ Disconnected';
      statusDiv.style.color = 'red';
    });
    
    // Send message
    const sendMessage = () => {
      const message = messageInput.value.trim();
      
      if (message) {
        socket.emit('message', {
          text: message,
          sender: socket.id,
          timestamp: new Date().toLocaleTimeString('bn-BD')
        });
        
        messageInput.value = '';
        messageInput.focus();
      }
    };
    
    // Send button click
    sendButton.addEventListener('click', sendMessage);
    
    // Enter key press
    messageInput.addEventListener('keypress', (e) => {
      if (e.key === 'Enter') {
        sendMessage();
      }
    });
    
    // Receive message
    socket.on('message', (data) => {
      const messageEl = document.createElement('div');
      messageEl.className = 'message';
      messageEl.innerHTML = `
        <strong>${data.sender}</strong>
        <p>${data.text}</p>
        <small>${data.timestamp}</small>
      `;
      
      messagesDiv.appendChild(messageEl);
      messagesDiv.scrollTop = messagesDiv.scrollHeight;
    });
  </script>
</body>
</html>
```

---

## Advanced Features

### 1. Rooms এবং Namespaces

**Rooms** হলো channels যেখানে specific users থাকে। **Namespaces** হলো আলাদা connection endpoints।

```javascript
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer);

// Default namespace
io.on('connection', (socket) => {
  console.log('User connected:', socket.id);
  
  // Join a room
  socket.on('join-room', (roomName) => {
    socket.join(roomName);
    console.log(`${socket.id} joined room: ${roomName}`);
    
    // Notify others in the room
    socket.to(roomName).emit('user-joined', {
      userId: socket.id,
      message: `${socket.id} joined ${roomName}`
    });
    
    // Confirm to the user
    socket.emit('joined-room', {
      room: roomName,
      message: `You joined ${roomName}`
    });
  });
  
  // Send message to a specific room
  socket.on('room-message', ({ room, message }) => {
    io.to(room).emit('message', {
      room,
      sender: socket.id,
      text: message,
      timestamp: new Date()
    });
  });
  
  // Leave room
  socket.on('leave-room', (roomName) => {
    socket.leave(roomName);
    console.log(`${socket.id} left room: ${roomName}`);
    
    socket.to(roomName).emit('user-left', {
      userId: socket.id,
      message: `${socket.id} left ${roomName}`
    });
  });
  
  // Private message
  socket.on('private-message', ({ to, message }) => {
    socket.to(to).emit('private-message', {
      from: socket.id,
      text: message,
      timestamp: new Date()
    });
  });
});

// Custom namespace
const adminNamespace = io.of('/admin');

adminNamespace.on('connection', (socket) => {
  console.log('Admin connected:', socket.id);
  
  socket.on('admin-broadcast', (data) => {
    // Broadcast to all clients in default namespace
    io.emit('admin-message', data);
  });
});

httpServer.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

---

### 2. Authentication with Socket.io

```javascript
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';
import jwt from 'jsonwebtoken';

const app = express();
const httpServer = createServer(app);

const io = new Server(httpServer, {
  cors: {
    origin: '*',
    methods: ['GET', 'POST']
  }
});

const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';

// Authentication middleware
io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  
  if (!token) {
    return next(new Error('Authentication error: No token provided'));
  }
  
  try {
    const decoded = jwt.verify(token, JWT_SECRET);
    socket.userId = decoded.id;
    socket.username = decoded.username;
    next();
  } catch (error) {
    next(new Error('Authentication error: Invalid token'));
  }
});

io.on('connection', (socket) => {
  console.log(`✅ Authenticated user connected: ${socket.username}`);
  
  // User-specific events
  socket.on('message', (data) => {
    io.emit('message', {
      username: socket.username,
      userId: socket.userId,
      text: data.text,
      timestamp: new Date()
    });
  });
  
  socket.on('disconnect', () => {
    console.log(`❌ ${socket.username} disconnected`);
  });
});

// Login route (example)
app.post('/login', express.json(), (req, res) => {
  const { username, password } = req.body;
  
  // Validate credentials (simplified)
  if (username && password === 'demo123') {
    const token = jwt.sign(
      { id: Date.now(), username },
      JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    res.json({ token, username });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
});

httpServer.listen(3000);
```

**Client-side authentication:**

```html
<script src="/socket.io/socket.io.js"></script>
<script>
  // Get token from login
  const token = localStorage.getItem('token');
  
  // Connect with authentication
  const socket = io({
    auth: {
      token: token
    }
  });
  
  socket.on('connect', () => {
    console.log('Connected with authentication');
  });
  
  socket.on('connect_error', (err) => {
    console.error('Connection error:', err.message);
    // Redirect to login page
    if (err.message.includes('Authentication')) {
      window.location.href = '/login';
    }
  });
</script>
```

---

### 3. Broadcasting Strategies

```javascript
// সবাইকে পাঠানো (including sender)
io.emit('message', data);

// সবাইকে পাঠানো (except sender)
socket.broadcast.emit('message', data);

// Specific room এ পাঠানো
io.to('room1').emit('message', data);

// Multiple rooms এ পাঠানো
io.to('room1').to('room2').emit('message', data);

// Specific user কে পাঠানো (socket ID দিয়ে)
io.to(socketId).emit('private-message', data);

// Room এ পাঠানো (except sender)
socket.to('room1').emit('message', data);

// Namespace এ পাঠানো
io.of('/admin').emit('notification', data);

// Volatile messages (delivery not guaranteed - for real-time data)
socket.volatile.emit('data', { value: Math.random() });
```

---

### 4. Error Handling এবং Reconnection

```javascript
// Server-side error handling
io.on('connection', (socket) => {
  // Custom error event
  socket.on('error', (error) => {
    console.error('Socket error:', error);
  });
  
  // Disconnect with reason
  socket.on('disconnect', (reason) => {
    console.log(`User disconnected: ${reason}`);
    
    if (reason === 'io server disconnect') {
      // Server forced disconnect, reconnect manually
      socket.connect();
    }
  });
  
  // Handle client errors
  socket.on('client-error', (error) => {
    console.error('Client reported error:', error);
    socket.emit('error-response', {
      message: 'Error received, please try again'
    });
  });
});
```

**Client-side reconnection:**

```javascript
const socket = io({
  reconnection: true,          // Enable auto-reconnection
  reconnectionDelay: 1000,     // 1 second delay
  reconnectionDelayMax: 5000,  // Max 5 seconds
  reconnectionAttempts: 5      // Try 5 times
});

socket.on('connect', () => {
  console.log('Connected to server');
});

socket.on('disconnect', (reason) => {
  console.log('Disconnected:', reason);
  
  if (reason === 'io server disconnect') {
    // Manual reconnect
    socket.connect();
  }
});

socket.on('reconnect', (attemptNumber) => {
  console.log(`Reconnected after ${attemptNumber} attempts`);
});

socket.on('reconnect_attempt', (attemptNumber) => {
  console.log(`Reconnection attempt ${attemptNumber}`);
});

socket.on('reconnect_error', (error) => {
  console.error('Reconnection error:', error);
});

socket.on('reconnect_failed', () => {
  console.error('Failed to reconnect after max attempts');
  alert('Unable to connect to server. Please refresh the page.');
});
```

---

### 5. Complete Chat Application with Rooms

```javascript
// server.js
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';
import { fileURLToPath } from 'url';
import { dirname, join } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer);

// Store active users
const users = new Map();
const rooms = new Map();

app.use(express.static(join(__dirname, 'public')));

app.get('/', (req, res) => {
  res.sendFile(join(__dirname, 'public', 'chat.html'));
});

io.on('connection', (socket) => {
  console.log(`✅ User connected: ${socket.id}`);
  
  // User joins with username
  socket.on('join', (username) => {
    users.set(socket.id, { username, socketId: socket.id });
    
    socket.emit('join-success', {
      socketId: socket.id,
      username: username
    });
    
    // Send active users list
    io.emit('users-list', Array.from(users.values()));
    
    // Notify others
    socket.broadcast.emit('user-joined', {
      username,
      message: `${username} joined the chat`
    });
  });
  
  // Join room
  socket.on('join-room', (roomName) => {
    const user = users.get(socket.id);
    if (!user) return;
    
    socket.join(roomName);
    user.room = roomName;
    
    // Add room to rooms map
    if (!rooms.has(roomName)) {
      rooms.set(roomName, new Set());
    }
    rooms.get(roomName).add(socket.id);
    
    // Notify user
    socket.emit('joined-room', {
      room: roomName,
      message: `You joined ${roomName}`
    });
    
    // Notify others in room
    socket.to(roomName).emit('user-joined-room', {
      username: user.username,
      room: roomName
    });
    
    // Send room members
    const roomMembers = Array.from(rooms.get(roomName))
      .map(id => users.get(id))
      .filter(Boolean);
    
    io.to(roomName).emit('room-members', roomMembers);
  });
  
  // Send message to room or globally
  socket.on('send-message', ({ message, room }) => {
    const user = users.get(socket.id);
    if (!user) return;
    
    const messageData = {
      username: user.username,
      text: message,
      timestamp: new Date().toLocaleTimeString('bn-BD'),
      socketId: socket.id
    };
    
    if (room) {
      // Send to room
      io.to(room).emit('message', { ...messageData, room });
    } else {
      // Send globally
      io.emit('message', messageData);
    }
  });
  
  // Private message
  socket.on('private-message', ({ to, message }) => {
    const sender = users.get(socket.id);
    const recipient = users.get(to);
    
    if (sender && recipient) {
      const messageData = {
        from: sender.username,
        fromId: socket.id,
        text: message,
        timestamp: new Date().toLocaleTimeString('bn-BD'),
        private: true
      };
      
      // Send to recipient
      socket.to(to).emit('private-message', messageData);
      
      // Confirm to sender
      socket.emit('private-message-sent', {
        to: recipient.username,
        ...messageData
      });
    }
  });
  
  // Typing indicator
  socket.on('typing', ({ room }) => {
    const user = users.get(socket.id);
    if (!user) return;
    
    if (room) {
      socket.to(room).emit('user-typing', {
        username: user.username,
        room
      });
    } else {
      socket.broadcast.emit('user-typing', {
        username: user.username
      });
    }
  });
  
  socket.on('stop-typing', ({ room }) => {
    const user = users.get(socket.id);
    if (!user) return;
    
    if (room) {
      socket.to(room).emit('user-stop-typing', {
        username: user.username,
        room
      });
    } else {
      socket.broadcast.emit('user-stop-typing', {
        username: user.username
      });
    }
  });
  
  // Leave room
  socket.on('leave-room', (roomName) => {
    const user = users.get(socket.id);
    if (!user) return;
    
    socket.leave(roomName);
    
    if (rooms.has(roomName)) {
      rooms.get(roomName).delete(socket.id);
    }
    
    user.room = null;
    
    socket.to(roomName).emit('user-left-room', {
      username: user.username,
      room: roomName
    });
    
    socket.emit('left-room', { room: roomName });
  });
  
  // Disconnect
  socket.on('disconnect', () => {
    const user = users.get(socket.id);
    
    if (user) {
      // Remove from room
      if (user.room && rooms.has(user.room)) {
        rooms.get(user.room).delete(socket.id);
      }
      
      // Notify others
      io.emit('user-disconnected', {
        username: user.username,
        message: `${user.username} left the chat`
      });
      
      users.delete(socket.id);
      
      // Update users list
      io.emit('users-list', Array.from(users.values()));
    }
    
    console.log(`❌ User disconnected: ${socket.id}`);
  });
});

const PORT = process.env.PORT || 3000;
httpServer.listen(PORT, () => {
  console.log(`🚀 Chat server running on http://localhost:${PORT}`);
});
```

---

## Scaling Socket.io Applications

### Using Redis Adapter for Multiple Servers

```bash
npm install @socket.io/redis-adapter redis
```

```javascript
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer);

// Redis clients
const pubClient = createClient({ host: 'localhost', port: 6379 });
const subClient = pubClient.duplicate();

Promise.all([pubClient.connect(), subClient.connect()]).then(() => {
  // Use Redis adapter
  io.adapter(createAdapter(pubClient, subClient));
  
  console.log('✅ Redis adapter connected');
});

io.on('connection', (socket) => {
  // Your socket logic here
  // এখন multiple servers run করলেও সব messages sync থাকবে
});

httpServer.listen(3000);
```

---

## ⚠️ Common Mistakes & Fixes

### ভুল ১: Socket.io client script না যোগ করা

```html
<!-- ❌ ভুল - Script missing -->
<script>
  const socket = io(); // Error: io is not defined
</script>

<!-- ✅ সঠিক - Script যোগ করুন -->
<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io();
</script>
```

### ভুল ২: CORS error

```javascript
// ❌ ভুল - CORS configuration নেই
const io = new Server(httpServer);

// ✅ সঠিক - CORS enable করুন
const io = new Server(httpServer, {
  cors: {
    origin: 'http://localhost:5173', // Your frontend URL
    methods: ['GET', 'POST']
  }
});
```

### ভুল ৩: HTTP server না তৈরি করে সরাসরি Express app ব্যবহার

```javascript
// ❌ ভুল
import express from 'express';
import { Server } from 'socket.io';

const app = express();
const io = new Server(app); // Error!

app.listen(3000);

// ✅ সঠিক - HTTP server তৈরি করুন
import express from 'express';
import { createServer } from 'http';
import { Server } from 'socket.io';

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer);

httpServer.listen(3000); // httpServer.listen(), NOT app.listen()
```

### ভুল ৪: Event listener ভুল জায়গায় রাখা

```javascript
// ❌ ভুল - connection এর বাইরে event listener
io.on('connection', (socket) => {
  console.log('User connected');
});

socket.on('message', (data) => { // Error: socket is not defined
  console.log(data);
});

// ✅ সঠিক - connection এর ভিতরে
io.on('connection', (socket) => {
  console.log('User connected');
  
  socket.on('message', (data) => {
    console.log(data);
  });
});
```

### ভুল ৫: Room থেকে automatic leave না করা

```javascript
// ❌ ভুল - Disconnect এ room cleanup নেই
socket.on('disconnect', () => {
  console.log('User disconnected');
  // Room থেকে remove করা হয়নি
});

// ✅ সঠিক - Room cleanup করুন
socket.on('disconnect', () => {
  // Socket.io automatically removes from all rooms
  // কিন্তু manual tracking থাকলে cleanup করুন
  
  if (socket.currentRoom) {
    const room = rooms.get(socket.currentRoom);
    if (room) {
      room.delete(socket.id);
    }
  }
});
```

### ভুল ৬: Broadcasting সঠিকভাবে না করা

```javascript
// ❌ ভুল - নিজেকেও message পাঠানো হচ্ছে
socket.on('message', (data) => {
  io.emit('message', data); // Sender ও পাবে
});

// ✅ সঠিক - নিজে ছাড়া সবাইকে পাঠান
socket.on('message', (data) => {
  socket.broadcast.emit('message', data);
  // অথবা
  socket.to(roomName).emit('message', data);
});
```

### ভুল ৭: Memory leak - Event listeners remove না করা

```javascript
// ❌ ভুল - Listeners remove করা হচ্ছে না
setInterval(() => {
  socket.on('ping', () => {
    console.log('Ping received');
  });
}, 1000); // Memory leak!

// ✅ সঠিক - একবার listener attach করুন
socket.on('ping', () => {
  console.log('Ping received');
});

// অথবা manually remove করুন
const handlePing = () => console.log('Ping received');
socket.on('ping', handlePing);

// Later
socket.off('ping', handlePing);
```

### ভুল ৮: Authentication bypass

```javascript
// ❌ ভুল - Client data trust করা
socket.on('message', (data) => {
  const username = data.username; // Client থেকে আসছে - unsafe!
  io.emit('message', { username, text: data.text });
});

// ✅ সঠিক - Server-side username ব্যবহার করুন
io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  const user = verifyToken(token);
  socket.username = user.username; // Server sets username
  next();
});

socket.on('message', (data) => {
  io.emit('message', { 
    username: socket.username, // Server-side value
    text: data.text 
  });
});
```

### ভুল ৯: Error handling না করা

```javascript
// ❌ ভুল - Errors handle করা হচ্ছে না
socket.on('message', (data) => {
  const parsed = JSON.parse(data); // Might throw error
});

// ✅ সঠিক - Try-catch ব্যবহার করুন
socket.on('message', (data) => {
  try {
    const parsed = JSON.parse(data);
    // Process message
  } catch (error) {
    console.error('Error parsing message:', error);
    socket.emit('error', { message: 'Invalid message format' });
  }
});
```

### ভুল ১০: Reconnection handling না করা

```javascript
// ❌ ভুল - Client reconnection handle করা হচ্ছে না
const socket = io();

// ✅ সঠিক - Reconnection events handle করুন
const socket = io({
  reconnection: true,
  reconnectionAttempts: 5,
  reconnectionDelay: 1000
});

socket.on('disconnect', (reason) => {
  console.log('Disconnected:', reason);
  
  if (reason === 'io server disconnect') {
    socket.connect();
  }
});

socket.on('reconnect', (attemptNumber) => {
  console.log('Reconnected successfully');
  // Re-join rooms, re-authenticate etc.
});
```

---

## Project Structure (Best Practice)

```
socketio-app/
│
├── server.js                 # Main server file
├── package.json
│
├── src/
│   ├── socket/
│   │   ├── handlers/
│   │   │   ├── chatHandler.js
│   │   │   ├── roomHandler.js
│   │   │   └── authHandler.js
│   │   ├── middleware/
│   │   │   └── authMiddleware.js
│   │   └── socketServer.js
│   │
│   └── models/
│       ├── User.js
│       └── Message.js
│
├── public/
│   ├── index.html
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── client.js
│
└── .env
```

---

## Performance Tips

1. **Use Rooms efficiently**: অপ্রয়োজনীয় rooms তৈরি করবেন না
2. **Limit payload size**: বড় data পাঠানো থেকে বিরত থাকুন
3. **Use Redis adapter**: Multiple servers scale করার জন্য
4. **Implement rate limiting**: Spam prevent করার জন্য
5. **Clean up disconnected sockets**: Memory leaks prevent করুন
6. **Use binary data**: JSON এর চেয়ে efficient
7. **Compress messages**: বড় data compress করে পাঠান

---

## রেফারেন্স এবং দরকারী লিংক

- **Socket.io Official Docs**: https://socket.io/docs/v4/
- **Socket.io GitHub**: https://github.com/socketio/socket.io
- **Redis Adapter**: https://socket.io/docs/v4/redis-adapter/
- **Authentication**: https://socket.io/docs/v4/middlewares/
- **Client API**: https://socket.io/docs/v4/client-api/
- **Server API**: https://socket.io/docs/v4/server-api/
- **Emit cheatsheet**: https://socket.io/docs/v4/emit-cheatsheet/

---

**পরবর্তী Lesson**: REST API Design এবং Documentation with Swagger

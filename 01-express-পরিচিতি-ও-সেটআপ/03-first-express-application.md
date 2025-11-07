# আপনার প্রথম Express.js Application

## সংক্ষিপ্ত পরিচিতি

এই lesson এ আমরা শিখব কীভাবে একটি complete, production-ready এর মতো প্রথম Express application তৈরি করতে হয়।

## Basic Express App Structure

### সবচেয়ে সহজ উদাহরণ

```javascript
import express from 'express';

const app = express();
const PORT = 3000;

// Route
app.get('/', (req, res) => {
  res.send('Hello World!');
});

// Server শুরু করুন
app.listen(PORT, () => {
  console.log(`Server চলছে: http://localhost:${PORT}`);
});
```

## Express App তৈরির ধাপসমূহ

### ধাপ ১: Express Import করুন

```javascript
import express from 'express';
```

### ধাপ ২: Express Application তৈরি করুন

```javascript
const app = express();
```

`app` হলো আপনার Express application instance। এটি HTTP server এর মতো কাজ করে।

### ধাপ ৩: Middleware Setup করুন

```javascript
// Built-in middleware
app.use(express.json());                    // JSON parsing
app.use(express.urlencoded({ extended: true }));  // Form data
app.use(express.static('public'));          // Static files

// Custom middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});
```

### ধাপ ৪: Routes Define করুন

```javascript
app.get('/', (req, res) => {
  res.send('Home Page');
});

app.get('/about', (req, res) => {
  res.send('About Page');
});

app.post('/users', (req, res) => {
  res.json({ message: 'User created' });
});
```

### ধাপ ৫: Error Handler যোগ করুন

```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something went wrong!');
});
```

### ধাপ ৬: Server শুরু করুন

```javascript
app.listen(PORT, () => {
  console.log(`Server চলছে: http://localhost:${PORT}`);
});
```

## সম্পূর্ণ উদাহরণ

```javascript
import express from 'express';
import 'dotenv/config';

const app = express();
const PORT = process.env.PORT || 3000;

// ============ MIDDLEWARE ============
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

// Logging middleware
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
});

// ============ ROUTES ============

// Home Route
app.get('/', (req, res) => {
  res.json({
    message: 'স্বাগতম এক্সপ্রেস অ্যাপে!',
    version: '1.0.0',
    endpoints: {
      home: '/',
      about: '/about',
      contact: '/contact',
      api: '/api/users'
    }
  });
});

// About Route
app.get('/about', (req, res) => {
  res.json({
    appName: 'My Express App',
    version: '1.0.0',
    description: 'এটি আমার প্রথম এক্সপ্রেস অ্যাপ্লিকেশন'
  });
});

// Contact Route
app.get('/contact', (req, res) => {
  res.json({
    email: 'contact@example.com',
    phone: '+880171-2345678',
    address: 'Dhaka, Bangladesh'
  });
});

// API Routes
app.get('/api/users', (req, res) => {
  const users = [
    { id: 1, name: 'রহিম' },
    { id: 2, name: 'করিম' }
  ];
  res.json(users);
});

app.post('/api/users', (req, res) => {
  const { name, email } = req.body;

  if (!name) {
    return res.status(400).json({ error: 'Name প্রয়োজন' });
  }

  const newUser = {
    id: 3,
    name,
    email
  };

  res.status(201).json({
    message: 'User তৈরি হয়েছে',
    data: newUser
  });
});

// ============ 404 Handler ============
app.use((req, res) => {
  res.status(404).json({
    error: 'পেজ পাওয়া যায়নি',
    message: `${req.url} এই URL টি exist করে না`,
    path: req.url,
    method: req.method
  });
});

// ============ ERROR Handler ============
app.use((err, req, res, next) => {
  console.error('Error:', err);
  res.status(500).json({
    error: 'কিছু একটা ভুল হয়েছে',
    message: err.message
  });
});

// ============ START SERVER ============
app.listen(PORT, () => {
  console.log(`✅ সার্ভার চলছে: http://localhost:${PORT}`);
  console.log(`🌍 Environment: ${process.env.NODE_ENV || 'development'}`);
});
```

## app.listen() বোঝা

```javascript
app.listen(PORT, HOSTNAME, BACKLOG, CALLBACK);
```

- **PORT**: যে port এ server চলবে
- **HOSTNAME**: IP address (optional, default: localhost)
- **BACKLOG**: Pending connections সংখ্যা (optional)
- **CALLBACK**: Server start হলে এই function call হবে

### উদাহরণ

```javascript
// শুধু port
app.listen(3000);

// Callback সহ
app.listen(3000, () => {
  console.log('Server started');
});

// Hostname সহ
app.listen(3000, 'localhost', () => {
  console.log('Server started on localhost:3000');
});

// সব parameter সহ
app.listen(3000, '127.0.0.1', 511, () => {
  console.log('Server fully configured');
});
```

## Server চালু এবং বন্ধ করা

### Development এ চালান

```bash
# Direct চালান
node index.js

# npm script দিয়ে
npm start

# Nodemon দিয়ে (auto-restart)
npm run dev
```

### Server বন্ধ করুন

```bash
# Terminal এ
Ctrl + C

# Background process বন্ধ করতে
kill -9 <PID>
```

### Multiple Servers চালান

```javascript
// Server 1
app.listen(3000, () => console.log('Server 1: 3000'));

// Server 2
app.listen(3001, () => console.log('Server 2: 3001'));
```

## Testing করুন

### Browser এ পরীক্ষা

```
GET http://localhost:3000/
GET http://localhost:3000/about
GET http://localhost:3000/api/users
```

### cURL দিয়ে পরীক্ষা

```bash
# GET request
curl http://localhost:3000/

# POST request
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"আহমেদ","email":"ahmed@example.com"}'
```

### Postman দিয়ে পরীক্ষা

1. Postman খুলুন
2. Request type select করুন (GET, POST, etc.)
3. URL enter করুন: `http://localhost:3000/...`
4. Send button click করুন

## Environment Variables ব্যবহার

```javascript
// .env ফাইল
PORT=3000
NODE_ENV=development
APP_NAME=MyApp

// index.js
import 'dotenv/config';

const PORT = process.env.PORT || 3000;
const NODE_ENV = process.env.NODE_ENV || 'development';

app.listen(PORT, () => {
  console.log(`App: ${process.env.APP_NAME}`);
  console.log(`Port: ${PORT}`);
  console.log(`Environment: ${NODE_ENV}`);
});
```

## সাধারণ সমস্যা এবং সমাধান

### ❌ Problem: "Cannot find module 'express'"

```bash
npm install express
```

### ❌ Problem: "Port is already in use"

```bash
# Port ব্যবহারকারী খুঁজুন
lsof -i :3000

# Process বন্ধ করুন
kill -9 <PID>
```

### ❌ Problem: "app.listen is not a function"

```javascript
// ❌ ভুল
const app = express;  // Forgot parentheses

// ✅ সঠিক
const app = express();
```

### ❌ Problem: "Cannot use import statement outside a module"

```json
// package.json এ যোগ করুন
{
  "type": "module"
}
```

## Production Best Practices

### ✅ 1. Environment Variables ব্যবহার করুন

```javascript
const PORT = process.env.PORT || 3000;
const NODE_ENV = process.env.NODE_ENV || 'development';
```

### ✅ 2. Proper Logging করুন

```javascript
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
});
```

### ✅ 3. Error Handling করুন

```javascript
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Server error' });
});
```

### ✅ 4. Security Headers সেট করুন

```javascript
import helmet from 'helmet';

app.use(helmet());
```

### ✅ 5. CORS সেট আপ করুন

```javascript
import cors from 'cors';

app.use(cors());
// অথবা নির্দিষ্ট domain এর জন্য
app.use(cors({ origin: 'http://localhost:3000' }));
```

## পরবর্তী পদক্ষেপ

এখন আপনি:
- ✅ Express application তৈরি করতে পারেন
- ✅ Routes define করতে পারেন
- ✅ Server চালাতে পারেন
- ✅ Basic error handling করতে পারেন

পরবর্তী chapter এ আমরা **Routing** সম্পর্কে বিস্তারিত শিখব।

## রেফারেন্স

- **Express API**: https://expressjs.com/en/5x/api.html
- **Express Guide**: https://expressjs.com/en/guide/routing.html

---

**পরবর্তী Lesson**: Basic Routing

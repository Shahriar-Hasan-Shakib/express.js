# Request Object (req)

## সংক্ষিপ্ত পরিচিতি

Request object (`req`) Express.js এ client-side information ধারণ করে। এতে URL parameters, query strings, headers, body এবং আরও অনেক তথ্য থাকে।

## req.params - URL Parameters

URL এ define করা parameters access করতে `req.params` ব্যবহার করুন।

```javascript
app.get('/users/:userId/posts/:postId', (req, res) => {
  const userId = req.params.userId;
  const postId = req.params.postId;
  
  res.json({
    userId: userId,
    postId: postId
  });
});
```

**Request:**
```
GET /users/123/posts/456
```

**Response:**
```json
{
  "userId": "123",
  "postId": "456"
}
```

### Optional Parameters

```javascript
app.get('/files/:category/:filename?', (req, res) => {
  const category = req.params.category;
  const filename = req.params.filename || 'default';
  
  res.json({ category, filename });
});
```

## req.query - Query Strings

URL এর `?` এর পরের part query string হিসেবে পরিচিত। এটি `req.query` দিয়ে access করুন।

```javascript
app.get('/search', (req, res) => {
  const searchTerm = req.query.q;
  const pageNumber = req.query.page;
  const limit = req.query.limit;
  
  res.json({
    search: searchTerm,
    page: pageNumber,
    limit: limit
  });
});
```

**Request:**
```
GET /search?q=express&page=2&limit=10
```

**Response:**
```json
{
  "search": "express",
  "page": "2",
  "limit": "10"
}
```

### Multiple Query Parameters

```javascript
app.get('/filter', (req, res) => {
  // সব query parameters পান
  res.json(req.query);
});

// GET /filter?category=books&author=কাজী&year=2024
// Output: { category: "books", author: "কাজী", year: "2024" }
```

## req.body - Request Body

POST/PUT request এ data পাঠাতে `req.body` ব্যবহার করুন। কিন্তু প্রথমে middleware দিয়ে data parse করতে হবে।

```javascript
// JSON parsing middleware
app.use(express.json());

app.post('/users', (req, res) => {
  const name = req.body.name;
  const email = req.body.email;
  const age = req.body.age;
  
  res.json({
    message: 'Data received',
    data: req.body
  });
});
```

**Request (POST):**
```
POST /users
Content-Type: application/json

{
  "name": "রহিম",
  "email": "rahim@example.com",
  "age": 25
}
```

**Response:**
```json
{
  "message": "Data received",
  "data": {
    "name": "রহিম",
    "email": "rahim@example.com",
    "age": 25
  }
}
```

### Form Data Parsing

```javascript
// URL-encoded form data parsing
app.use(express.urlencoded({ extended: true }));

app.post('/form', (req, res) => {
  res.json(req.body);
});
```

## req.headers - HTTP Headers

Client request এ পাঠানো headers access করুন।

```javascript
app.get('/check-headers', (req, res) => {
  const contentType = req.headers['content-type'];
  const auth = req.headers['authorization'];
  const userAgent = req.headers['user-agent'];
  
  res.json({
    contentType: contentType,
    auth: auth,
    userAgent: userAgent
  });
});
```

### সাধারণ Headers

```javascript
app.get('/headers', (req, res) => {
  res.json({
    'accept': req.headers['accept'],
    'accept-language': req.headers['accept-language'],
    'accept-encoding': req.headers['accept-encoding'],
    'content-type': req.headers['content-type'],
    'authorization': req.headers['authorization'],
    'referer': req.headers['referer'],
    'user-agent': req.headers['user-agent'],
    'host': req.headers['host'],
    'connection': req.headers['connection']
  });
});
```

## req.cookies - Cookies

Client থেকে পাঠানো cookies access করতে `cookie-parser` middleware ব্যবহার করুন।

```javascript
import cookieParser from 'cookie-parser';

app.use(cookieParser());

app.get('/profile', (req, res) => {
  const sessionId = req.cookies.sessionId;
  const userId = req.cookies.userId;
  
  res.json({
    session: sessionId,
    user: userId
  });
});
```

## req এর অন্যান্য গুরুত্বপূর্ণ Properties

```javascript
app.get('/request-info', (req, res) => {
  res.json({
    // Method
    method: req.method,                      // GET, POST, DELETE, etc.
    
    // URLs
    url: req.url,                            // /path?query=value
    originalUrl: req.originalUrl,            // Original URL
    path: req.path,                          // /path (without query)
    
    // Host information
    hostname: req.hostname,                  // example.com
    host: req.host,                          // example.com:3000
    
    // Client information
    ip: req.ip,                              // Client IP address
    ips: req.ips,                            // Array of IPs (if behind proxy)
    
    // Protocol
    protocol: req.protocol,                  // http or https
    secure: req.secure,                      // true if HTTPS
    
    // Connection
    xhr: req.xhr,                            // true if AJAX request
    subdomains: req.subdomains               // ['api'] from api.example.com
  });
});
```

## সম্পূর্ণ উদাহরণ

```javascript
import express from 'express';
import cookieParser from 'cookie-parser';

const app = express();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());

// Route example
app.post('/api/users/:userId', (req, res) => {
  res.json({
    // URL parameters
    params: req.params,                // { userId: "123" }
    
    // Query parameters
    query: req.query,                  // { status: "active", ... }
    
    // Body
    body: req.body,                    // { name: "...", email: "..." }
    
    // Headers
    headers: {
      contentType: req.headers['content-type'],
      auth: req.headers['authorization']
    },
    
    // Cookies
    cookies: req.cookies,              // { sessionId: "abc123" }
    
    // Other info
    method: req.method,
    url: req.url,
    ip: req.ip
  });
});

app.listen(3000, () => console.log('Server চলছে: 3000'));
```

## Best Practices

### ✅ 1. সবসময় Validation করুন

```javascript
app.post('/users', (req, res) => {
  if (!req.body.name || !req.body.email) {
    return res.status(400).json({ 
      error: 'Name এবং email প্রয়োজন' 
    });
  }
  
  // Process request
});
```

### ✅ 2. Safe Parameter Access করুন

```javascript
app.get('/users/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  
  if (isNaN(userId)) {
    return res.status(400).json({ 
      error: 'Invalid user ID' 
    });
  }
  
  // Use userId
});
```

### ✅ 3. Query String Default Values

```javascript
app.get('/items', (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const sort = req.query.sort || 'asc';
  
  res.json({ page, limit, sort });
});
```

## পরবর্তী Lesson

পরবর্তীতে আমরা Response Object সম্পর্কে বিস্তারিত শিখব।

## রেফারেন্স

- **Request API**: https://expressjs.com/en/5x/api.html#req

---

**পরবর্তী Lesson**: Response Object (res)

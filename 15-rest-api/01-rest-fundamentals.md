# REST API Fundamentals

## সংক্ষিপ্ত পরিচিতি

**REST (Representational State Transfer)** হলো web services তৈরির একটি architectural style যা HTTP protocol এর উপর ভিত্তি করে তৈরি।

## REST কী?

**REST** এর মূল বিষয়গুলি:

1. **Resources**: Data এর প্রতিটি unit একটি resource (users, products, orders)
2. **Unique URI**: প্রতিটি resource এর একটি unique URL আছে
3. **HTTP Methods**: GET, POST, PUT, PATCH, DELETE ব্যবহার করে operations
4. **Stateless**: প্রতিটি request independent, server কোনো session রাখে না
5. **JSON/XML**: Data format

## REST Principles

### 1. Client-Server Architecture
- Server এবং Client আলাদাভাবে evolve করতে পারে
- একটি অন্যটির উপর depend করে না

### 2. Statelessness
- প্রতিটি request তে সব required information থাকে
- Server client এর state maintain করে না
- Scalability বৃদ্ধি পায়

### 3. Cacheability
- Response cache করা যায় বা না যায় সেটা specify করতে হয়
- Performance improve হয়

### 4. Uniform Interface
- Consistent, predictable URL structure
- Standard HTTP methods
- একই request সবসময় একই response দেয়

### 5. Layered System
- Client জানে না directly কার সাথে communicate করছে
- Proxies, load balancers থাকতে পারে layers এ

## REST vs অন্যান্য Architectures

### REST vs SOAP
| বৈশিষ্ট্য | REST | SOAP |
|---------|------|------|
| Protocol | HTTP | HTTP/SMTP/etc |
| Data Format | JSON, XML | XML only |
| Learning Curve | সহজ | কঠিন |
| Performance | দ্রুত | ধীর |
| Security | Basic, OAuth | WS-Security |

### REST vs GraphQL
| বৈশিষ্ট্য | REST | GraphQL |
|---------|------|---------|
| Query | Multiple endpoints | Single endpoint |
| Over-fetching | সম্ভব | নেই |
| Under-fetching | সম্ভব | নেই |
| Learning Curve | সহজ | মাঝারি |
| Caching | সহজ | জটিল |

## RESTful API Design Best Practices

### ✅ Resource-Based URLs

```javascript
// ✅ Correct - Resources (plural nouns)
GET    /api/users              // সব users
POST   /api/users              // নতুন user
GET    /api/users/123          // ID 123 এর user
PUT    /api/users/123          // User 123 update করো
PATCH  /api/users/123          // User 123 partially update করো
DELETE /api/users/123          // User 123 delete করো

// Nested resources
GET    /api/users/123/posts    // User 123 এর posts
POST   /api/users/123/posts    // User 123 এর জন্য নতুন post
GET    /api/users/123/posts/456 // User 123 এর post 456
```

### ❌ ভুল URL Structure

```javascript
// ❌ Action verbs ব্যবহার করবেন না
GET    /api/getUsers
POST   /api/createUser
PUT    /api/updateUser
DELETE /api/deleteUser

// ❌ Inconsistent naming
GET    /api/user            // Singular
GET    /api/Users           // Capital
GET    /api/user_list       // Underscores

// ❌ Deep nesting (avoid করুন)
GET    /api/users/123/posts/456/comments/789/likes
```

## HTTP Methods সঠিক ব্যবহার

### GET - রিড/রিট্রিভ

```javascript
app.get('/api/users', (req, res) => {
  // সব users পান
});

app.get('/api/users/:id', (req, res) => {
  // একটি specific user পান
});

app.get('/api/users/:id/posts', (req, res) => {
  // একটি user এর সব posts
});
```

**বৈশিষ্ট্য:**
- Idempotent: একই request multiple times = একই result
- Safe: কোনো data modify করে না
- Request body: নেই
- Response body: ডেটা থাকে
- Status Code: 200 OK

### POST - তৈরি করুন

```javascript
app.post('/api/users', (req, res) => {
  // নতুন user তৈরি করুন
  // req.body এ user data থাকবে
});
```

**বৈশিষ্ট্য:**
- Idempotent: নয় (একই request দুইবার = দুটি resource)
- Safe: নয় (data modify করে)
- Request body: থাকে (JSON)
- Response body: নতুন resource
- Status Code: 201 Created, 400 Bad Request

### PUT - সম্পূর্ণ আপডেট (Replace)

```javascript
app.put('/api/users/:id', (req, res) => {
  // সম্পূর্ণ user replace করুন
  // req.body এ সম্পূর্ণ নতুন data থাকবে
});
```

**বৈশিষ্ট্য:**
- Idempotent: হ্যাঁ (একই request দুইবার = একই result)
- Safe: নয়
- Request body: থাকে (সম্পূর্ণ resource)
- Status Code: 200 OK, 201 Created, 204 No Content

### PATCH - আংশিক আপডেট

```javascript
app.patch('/api/users/:id', (req, res) => {
  // শুধুমাত্র যা পরিবর্তন করতে চাই সেটা পাঠাই
  // { "email": "new@example.com" }
});
```

**বৈশিষ্ট্য:**
- Idempotent: প্রায় নয় (operation dependent)
- Safe: নয়
- Request body: থাকে (শুধু changes)
- Status Code: 200 OK, 204 No Content

### DELETE - মুছে ফেলুন

```javascript
app.delete('/api/users/:id', (req, res) => {
  // User মুছে ফেলুন
});
```

**বৈশিষ্ট্য:**
- Idempotent: হ্যাঁ (একবার delete করলে further delete meaningless)
- Safe: নয়
- Status Code: 200 OK, 204 No Content, 404 Not Found

## HTTP Status Codes

### Success Responses (2xx)

```javascript
200 OK              // সফল GET, PUT, PATCH
201 Created         // সফল POST (নতুন resource তৈরি)
202 Accepted        // Request accepted, processing চলছে
204 No Content      // সফল DELETE, কিন্তু body নেই
```

### Client Errors (4xx)

```javascript
400 Bad Request     // Invalid input, validation error
401 Unauthorized    // Authentication required
403 Forbidden       // Authentication OK কিন্তু access নেই
404 Not Found       // Resource exists করে না
409 Conflict        // Duplicate email, conflict
410 Gone            // Resource deleted এবং permanently gone
422 Unprocessable   // Semantic error
```

### Server Errors (5xx)

```javascript
500 Internal Server Error   // Generic server error
501 Not Implemented         // Feature not implemented
502 Bad Gateway             // Proxy/gateway error
503 Service Unavailable     // Server maintenance
```

## API Versioning Strategies

### 1. URL Path Versioning (বেশি ব্যবহৃত)

```javascript
// /api/v1/users, /api/v2/users
app.get('/api/v1/users', (req, res) => { /* v1 */ });
app.get('/api/v2/users', (req, res) => { /* v2 */ });
```

**সুবিধা:**
- Clear, explicit, URL এ visible
- Caching সহজ
- Backward compatibility maintain করা সহজ

### 2. Header Versioning

```javascript
// Accept-Version header
app.get('/api/users', (req, res) => {
  const version = req.headers['api-version'] || '1';
  
  if (version === '2') {
    res.json({ /* v2 data */ });
  } else {
    res.json({ /* v1 data */ });
  }
});
```

### 3. Query Parameter Versioning

```javascript
// /api/users?version=2
app.get('/api/users', (req, res) => {
  const version = req.query.version || '1';
  // version অনুযায়ী handle করুন
});
```

## Request-Response Cycle

### সম্পূর্ণ Example:

```javascript
import express from 'express';

const app = express();

app.use(express.json());

// Sample data
const users = [
  { id: 1, name: 'রহিম', email: 'rahim@example.com' },
  { id: 2, name: 'করিম', email: 'karim@example.com' }
];

// GET all users
app.get('/api/v1/users', (req, res) => {
  res.status(200).json({
    success: true,
    count: users.length,
    data: users
  });
});

// GET single user
app.get('/api/v1/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({
      success: false,
      error: 'User not found'
    });
  }
  
  res.status(200).json({
    success: true,
    data: user
  });
});

// POST create user
app.post('/api/v1/users', (req, res) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({
      success: false,
      error: 'Name এবং email প্রয়োজন'
    });
  }
  
  const newUser = {
    id: users.length + 1,
    name,
    email
  };
  
  users.push(newUser);
  
  res.status(201).json({
    success: true,
    message: 'User created',
    data: newUser
  });
});

// PUT update user (complete)
app.put('/api/v1/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({
      success: false,
      error: 'User not found'
    });
  }
  
  user.name = req.body.name || user.name;
  user.email = req.body.email || user.email;
  
  res.status(200).json({
    success: true,
    message: 'User updated',
    data: user
  });
});

// DELETE user
app.delete('/api/v1/users/:id', (req, res) => {
  const index = users.findIndex(u => u.id === parseInt(req.params.id));
  
  if (index === -1) {
    return res.status(404).json({
      success: false,
      error: 'User not found'
    });
  }
  
  const deletedUser = users.splice(index, 1)[0];
  
  res.status(200).json({
    success: true,
    message: 'User deleted',
    data: deletedUser
  });
});

app.listen(3000, () => {
  console.log('Server চলছে: http://localhost:3000');
});
```

## Pagination, Filtering, Sorting

```javascript
app.get('/api/v1/users', (req, res) => {
  // Pagination
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;
  
  // Filtering
  let filtered = users;
  if (req.query.role) {
    filtered = filtered.filter(u => u.role === req.query.role);
  }
  
  // Sorting
  const sortBy = req.query.sortBy || 'id';
  const order = req.query.order === 'desc' ? -1 : 1;
  filtered.sort((a, b) => {
    if (a[sortBy] < b[sortBy]) return -1 * order;
    if (a[sortBy] > b[sortBy]) return 1 * order;
    return 0;
  });
  
  // Apply pagination
  const paginated = filtered.slice(skip, skip + limit);
  
  res.status(200).json({
    success: true,
    count: paginated.length,
    total: filtered.length,
    totalPages: Math.ceil(filtered.length / limit),
    currentPage: page,
    data: paginated
  });
});

// Usage:
// GET /api/v1/users?page=2&limit=5&role=admin&sortBy=name&order=asc
```

## Best Practices Summary

✅ **করুন:**
- Resource-based URLs ব্যবহার করুন
- সঠিক HTTP methods ব্যবহার করুন
- সঠিক status codes রিটার্ন করুন
- API versioning করুন
- Input validation করুন
- Consistent response structure
- Pagination implement করুন
- Rate limiting ব্যবহার করুন
- Proper error messages দিন

❌ **করবেন না:**
- Action verbs URL এ রাখবেন না
- Status codes abuse করবেন না
- Versioning ছাড়া changes করবেন না
- Large responses করবেন না
- No error handling

## পরবর্তী Lesson

API Design Patterns এবং Implementation

## রেফারেন্স

- **REST Wikipedia**: https://en.wikipedia.org/wiki/Representational_state_transfer
- **REST Best Practices**: https://restfulapi.net/
- **HTTP Status Codes**: https://httpwg.org/specs/rfc9110.html

---

**পরবর্তী Lesson**: API Design এবং Implementation Patterns

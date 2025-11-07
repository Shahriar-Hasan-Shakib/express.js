# Express.js Routing - Basic Concepts

## সংক্ষিপ্ত পরিচিতি (Brief Overview)

**Routing** হলো একটি mechanism যা নির্ধারণ করে যে কোন URL request এলে application কীভাবে respond করবে। Express.js এ routing অত্যন্ত সহজ এবং powerful।

এই lesson এ আমরা শিখব:
- Basic routing কীভাবে কাজ করে
- বিভিন্ন HTTP methods (GET, POST, PUT, DELETE)
- Route paths এবং patterns
- Route parameters
- Query strings
- Route chaining

## Routing কী?

Routing মূলত তিনটি জিনিস নিয়ে গঠিত:

1. **HTTP Method** (GET, POST, PUT, DELETE, etc.)
2. **Path/URL** (যেমন: `/`, `/users`, `/products/:id`)
3. **Handler Function** (যে function টি request handle করবে)

### মৌলিক Structure:

```javascript
app.METHOD(PATH, HANDLER)
```

যেখানে:
- `app` হলো Express application এর instance
- `METHOD` হলো HTTP request method (lowercase এ)
- `PATH` হলো server এর route path
- `HANDLER` হলো callback function যা execute হবে

## HTTP Methods

### GET - ডেটা পড়ার জন্য

```javascript
// Simple GET route
app.get('/', (req, res) => {
  res.send('হোম পেজ');
});

// Multiple GET routes
app.get('/about', (req, res) => {
  res.send('About পেজ');
});

app.get('/contact', (req, res) => {
  res.send('যোগাযোগ করুন');
});
```

### POST - নতুন ডেটা তৈরির জন্য

```javascript
app.post('/users', (req, res) => {
  // নতুন user তৈরি করুন
  const newUser = {
    id: 1,
    name: req.body.name,
    email: req.body.email
  };
  
  res.status(201).json({
    message: 'User সফলভাবে তৈরি হয়েছে',
    user: newUser
  });
});
```

### PUT - ডেটা সম্পূর্ণ update করার জন্য

```javascript
app.put('/users/:id', (req, res) => {
  const userId = req.params.id;
  
  res.json({
    message: `User ${userId} সম্পূর্ণভাবে update হয়েছে`,
    updatedData: req.body
  });
});
```

### PATCH - ডেটা আংশিক update করার জন্য

```javascript
app.patch('/users/:id', (req, res) => {
  const userId = req.params.id;
  
  res.json({
    message: `User ${userId} এর কিছু field update হয়েছে`,
    updatedFields: req.body
  });
});
```

### DELETE - ডেটা মুছে ফেলার জন্য

```javascript
app.delete('/users/:id', (req, res) => {
  const userId = req.params.id;
  
  res.json({
    message: `User ${userId} মুছে ফেলা হয়েছে`
  });
});
```

## সম্পূর্ণ উদাহরণ: CRUD Operations

```javascript
const express = require('express');
const app = express();

// Middleware to parse JSON
app.use(express.json());

// Dummy data
let users = [
  { id: 1, name: 'রহিম', email: 'rahim@example.com' },
  { id: 2, name: 'করিম', email: 'karim@example.com' },
  { id: 3, name: 'সালমা', email: 'salma@example.com' }
];

// READ - সব users দেখান
app.get('/users', (req, res) => {
  res.json({
    count: users.length,
    users: users
  });
});

// READ - একটি specific user দেখান
app.get('/users/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  const user = users.find(u => u.id === userId);
  
  if (!user) {
    return res.status(404).json({
      error: 'User পাওয়া যায়নি'
    });
  }
  
  res.json(user);
});

// CREATE - নতুন user তৈরি করুন
app.post('/users', (req, res) => {
  const newUser = {
    id: users.length + 1,
    name: req.body.name,
    email: req.body.email
  };
  
  users.push(newUser);
  
  res.status(201).json({
    message: 'User সফলভাবে তৈরি হয়েছে',
    user: newUser
  });
});

// UPDATE - user এর সব তথ্য update করুন
app.put('/users/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  const userIndex = users.findIndex(u => u.id === userId);
  
  if (userIndex === -1) {
    return res.status(404).json({
      error: 'User পাওয়া যায়নি'
    });
  }
  
  users[userIndex] = {
    id: userId,
    name: req.body.name,
    email: req.body.email
  };
  
  res.json({
    message: 'User সফলভাবে update হয়েছে',
    user: users[userIndex]
  });
});

// PATCH - user এর কিছু field update করুন
app.patch('/users/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  const userIndex = users.findIndex(u => u.id === userId);
  
  if (userIndex === -1) {
    return res.status(404).json({
      error: 'User পাওয়া যায়নি'
    });
  }
  
  // শুধু যে fields পাঠানো হয়েছে সেগুলো update করুন
  users[userIndex] = {
    ...users[userIndex],
    ...req.body
  };
  
  res.json({
    message: 'User এর তথ্য আংশিকভাবে update হয়েছে',
    user: users[userIndex]
  });
});

// DELETE - user মুছে ফেলুন
app.delete('/users/:id', (req, res) => {
  const userId = parseInt(req.params.id);
  const userIndex = users.findIndex(u => u.id === userId);
  
  if (userIndex === -1) {
    return res.status(404).json({
      error: 'User পাওয়া যায়নি'
    });
  }
  
  const deletedUser = users.splice(userIndex, 1)[0];
  
  res.json({
    message: 'User সফলভাবে মুছে ফেলা হয়েছে',
    user: deletedUser
  });
});

// Start server
const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server চলছে: http://localhost:${PORT}`);
});
```

## Route Parameters

Route parameters হলো named URL segments যা values capture করতে ব্যবহৃত হয়।

### Single Parameter:

```javascript
// :id হলো route parameter
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.send(`User ID: ${userId}`);
});

// URL: /users/123
// Output: User ID: 123
```

### Multiple Parameters:

```javascript
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  
  res.json({
    message: 'Post details',
    userId: userId,
    postId: postId
  });
});

// URL: /users/5/posts/42
// Output: { userId: "5", postId: "42" }
```

### Optional Parameters (Regular Expression):

```javascript
// year parameter optional
app.get('/posts/:year?', (req, res) => {
  const year = req.params.year || 'সব বছর';
  res.send(`Posts from: ${year}`);
});

// URL: /posts/2024 -> Posts from: 2024
// URL: /posts -> Posts from: সব বছর
```

### Parameters with Pattern Matching:

```javascript
// শুধুমাত্র numbers accept করবে
app.get('/users/:id(\\d+)', (req, res) => {
  res.send(`User ID (number only): ${req.params.id}`);
});

// URL: /users/123 -> কাজ করবে ✅
// URL: /users/abc -> 404 error ❌
```

## Query Strings

Query strings হলো URL এর শেষে `?` এর পরে যে key-value pairs থাকে।

### Basic Query String:

```javascript
app.get('/search', (req, res) => {
  const { q, page, limit } = req.query;
  
  res.json({
    searchQuery: q,
    page: page || 1,
    limit: limit || 10
  });
});

// URL: /search?q=express&page=2&limit=20
// Output: {
//   searchQuery: "express",
//   page: "2",
//   limit: "20"
// }
```

### Query String with Default Values:

```javascript
app.get('/products', (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const sort = req.query.sort || 'name';
  
  res.json({
    message: 'Products list',
    pagination: {
      page: page,
      limit: limit,
      sort: sort
    }
  });
});
```

### Query String Validation:

```javascript
app.get('/api/data', (req, res) => {
  const { apiKey, format } = req.query;
  
  // Validation
  if (!apiKey) {
    return res.status(400).json({
      error: 'API key প্রয়োজন'
    });
  }
  
  const validFormats = ['json', 'xml', 'csv'];
  if (format && !validFormats.includes(format)) {
    return res.status(400).json({
      error: 'Invalid format. Valid: json, xml, csv'
    });
  }
  
  res.json({
    message: 'Data fetched successfully',
    format: format || 'json'
  });
});
```

## Route Chaining (app.route())

একই path এর জন্য multiple HTTP methods একসাথে define করা:

```javascript
app.route('/users/:id')
  .get((req, res) => {
    res.send(`GET user ${req.params.id}`);
  })
  .put((req, res) => {
    res.send(`PUT user ${req.params.id}`);
  })
  .delete((req, res) => {
    res.send(`DELETE user ${req.params.id}`);
  });
```

## All Method

সব HTTP methods এর জন্য একই handler:

```javascript
app.all('/secret', (req, res, next) => {
  console.log('Accessing secret section...');
  next(); // পরবর্তী handler এ যান
});

app.get('/secret', (req, res) => {
  res.send('Secret data (GET)');
});
```

## Route Path Patterns

### String Patterns:

```javascript
// Exact match
app.get('/about', (req, res) => {
  res.send('About page');
});

// ? - আগের character optional
app.get('/ab?cd', (req, res) => {
  // Matches: /acd, /abcd
  res.send('ab?cd');
});

// + - আগের character এক বা একাধিকবার
app.get('/ab+cd', (req, res) => {
  // Matches: /abcd, /abbcd, /abbbcd
  res.send('ab+cd');
});

// * - যেকোনো কিছু
app.get('/ab*cd', (req, res) => {
  // Matches: /abcd, /abXcd, /abRANDOMcd
  res.send('ab*cd');
});

// () - grouping
app.get('/ab(cd)?e', (req, res) => {
  // Matches: /abe, /abcde
  res.send('ab(cd)?e');
});
```

### Regular Expression:

```javascript
// যেকোনো route যার মধ্যে 'fly' আছে
app.get(/.*fly$/, (req, res) => {
  // Matches: /butterfly, /dragonfly
  res.send('Route ending with fly');
});

// শুধুমাত্র numbers
app.get(/^\/users\/(\d+)$/, (req, res) => {
  res.send('User with numeric ID');
});
```

## Response Methods

বিভিন্ন ধরনের response পাঠানোর method:

```javascript
app.get('/demo/response-methods', (req, res) => {
  // 1. res.send() - যেকোনো type এর data
  res.send('Hello');
  res.send({ message: 'Hello' });
  res.send([1, 2, 3]);
  
  // 2. res.json() - JSON data
  res.json({ name: 'রহিম', age: 25 });
  
  // 3. res.status() - HTTP status code
  res.status(404).send('পাওয়া যায়নি');
  
  // 4. res.sendFile() - File পাঠান
  res.sendFile('/path/to/file.html');
  
  // 5. res.redirect() - Redirect করুন
  res.redirect('/new-url');
  
  // 6. res.download() - File download করান
  res.download('/path/to/file.pdf');
  
  // 7. res.end() - Response শেষ করুন
  res.end();
});
```

## Practical Example: Blog API

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// Dummy blog posts
let posts = [
  { id: 1, title: 'Express.js শিখুন', author: 'রহিম', content: 'Express অসাধারণ...', views: 100 },
  { id: 2, title: 'Node.js টিউটোরিয়াল', author: 'করিম', content: 'Node.js একটি...', views: 150 },
];

// GET সব posts
app.get('/api/posts', (req, res) => {
  // Query string filtering
  const { author, minViews } = req.query;
  
  let filteredPosts = posts;
  
  if (author) {
    filteredPosts = filteredPosts.filter(p => 
      p.author.toLowerCase().includes(author.toLowerCase())
    );
  }
  
  if (minViews) {
    filteredPosts = filteredPosts.filter(p => 
      p.views >= parseInt(minViews)
    );
  }
  
  res.json({
    count: filteredPosts.length,
    posts: filteredPosts
  });
});

// GET একটি specific post
app.get('/api/posts/:id', (req, res) => {
  const post = posts.find(p => p.id === parseInt(req.params.id));
  
  if (!post) {
    return res.status(404).json({ error: 'Post পাওয়া যায়নি' });
  }
  
  // View count বাড়ান
  post.views++;
  
  res.json(post);
});

// POST নতুন post তৈরি
app.post('/api/posts', (req, res) => {
  const { title, author, content } = req.body;
  
  // Validation
  if (!title || !author || !content) {
    return res.status(400).json({
      error: 'Title, author এবং content প্রয়োজন'
    });
  }
  
  const newPost = {
    id: posts.length + 1,
    title,
    author,
    content,
    views: 0,
    createdAt: new Date()
  };
  
  posts.push(newPost);
  
  res.status(201).json({
    message: 'Post সফলভাবে তৈরি হয়েছে',
    post: newPost
  });
});

// PUT post update
app.put('/api/posts/:id', (req, res) => {
  const postIndex = posts.findIndex(p => p.id === parseInt(req.params.id));
  
  if (postIndex === -1) {
    return res.status(404).json({ error: 'Post পাওয়া যায়নি' });
  }
  
  posts[postIndex] = {
    ...posts[postIndex],
    ...req.body,
    id: parseInt(req.params.id), // ID change করা যাবে না
    updatedAt: new Date()
  };
  
  res.json({
    message: 'Post update হয়েছে',
    post: posts[postIndex]
  });
});

// DELETE post
app.delete('/api/posts/:id', (req, res) => {
  const postIndex = posts.findIndex(p => p.id === parseInt(req.params.id));
  
  if (postIndex === -1) {
    return res.status(404).json({ error: 'Post পাওয়া যায়নি' });
  }
  
  const deletedPost = posts.splice(postIndex, 1)[0];
  
  res.json({
    message: 'Post মুছে ফেলা হয়েছে',
    post: deletedPost
  });
});

// GET posts by author
app.get('/api/authors/:author/posts', (req, res) => {
  const authorPosts = posts.filter(p => 
    p.author.toLowerCase() === req.params.author.toLowerCase()
  );
  
  res.json({
    author: req.params.author,
    count: authorPosts.length,
    posts: authorPosts
  });
});

app.listen(3000, () => {
  console.log('Blog API চলছে: http://localhost:3000');
});
```

## Testing Routes (cURL commands)

```bash
# GET সব posts
curl http://localhost:3000/api/posts

# GET with query string
curl http://localhost:3000/api/posts?author=রহিম&minViews=50

# GET single post
curl http://localhost:3000/api/posts/1

# POST নতুন post
curl -X POST http://localhost:3000/api/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"নতুন পোস্ট","author":"সালমা","content":"এটি একটি টেস্ট পোস্ট"}'

# PUT update post
curl -X PUT http://localhost:3000/api/posts/1 \
  -H "Content-Type: application/json" \
  -d '{"title":"আপডেট করা পোস্ট"}'

# DELETE post
curl -X DELETE http://localhost:3000/api/posts/1
```

## সাধারণ ভুল এবং সমাধান

### ❌ ভুল ১: Route order এর সমস্যা

```javascript
// ভুল - specific route আগে আসা উচিত
app.get('/users/new', (req, res) => {
  res.send('New user form');
});

app.get('/users/:id', (req, res) => {
  res.send(`User ${req.params.id}`);
});
// '/users/new' request টি '/users/:id' দ্বারা catch হবে না
```

**সমাধান**: Specific routes আগে রাখুন
```javascript
// সঠিক
app.get('/users/new', (req, res) => {
  res.send('New user form');
});

app.get('/users/:id', (req, res) => {
  res.send(`User ${req.params.id}`);
});
```

### ❌ ভুল ২: Response একাধিকবার পাঠানো

```javascript
// ভুল
app.get('/data', (req, res) => {
  res.send('First response');
  res.send('Second response'); // ❌ Error!
});
```

**সমাধান**: শুধুমাত্র একবার response পাঠান
```javascript
// সঠিক
app.get('/data', (req, res) => {
  if (someCondition) {
    return res.send('First response'); // return ব্যবহার করুন
  }
  res.send('Second response');
});
```

### ❌ ভুল ৩: Query parameters type conversion ভুলে যাওয়া

```javascript
// সমস্যা - query parameters সবসময় string
app.get('/items', (req, res) => {
  const page = req.query.page; // এটি string
  const nextPage = page + 1; // "11" হবে, not 2!
  res.send(`Next page: ${nextPage}`);
});
```

**সমাধান**: parseInt/parseFloat ব্যবহার করুন
```javascript
// সঠিক
app.get('/items', (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const nextPage = page + 1; // এখন সঠিকভাবে 2 হবে
  res.send(`Next page: ${nextPage}`);
});
```

## Best Practices

### ✅ ১. RESTful Convention Follow করুন

```javascript
GET    /users       // সব users
GET    /users/:id   // একটি user
POST   /users       // নতুন user
PUT    /users/:id   // user update
DELETE /users/:id   // user delete
```

### ✅ ২. Consistent Response Structure

```javascript
// Success response
{
  "success": true,
  "data": { ... },
  "message": "অপারেশন সফল"
}

// Error response
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE"
}
```

### ✅ ৩. Status Codes সঠিকভাবে ব্যবহার করুন

```javascript
res.status(200).json(data);    // OK
res.status(201).json(data);    // Created
res.status(400).json(error);   // Bad Request
res.status(404).json(error);   // Not Found
res.status(500).json(error);   // Server Error
```

### ✅ ৪. Route Organization

```javascript
// routes/users.js তে রাখুন
const express = require('express');
const router = express.Router();

router.get('/', getAllUsers);
router.get('/:id', getUserById);
router.post('/', createUser);

module.exports = router;

// index.js এ import করুন
const userRoutes = require('./routes/users');
app.use('/api/users', userRoutes);
```

## পরবর্তী পদক্ষেপ

এখন আপনি:
- ✅ Basic routing বুঝতে পারেন
- ✅ বিভিন্ন HTTP methods ব্যবহার করতে পারেন
- ✅ Route parameters এবং query strings handle করতে পারেন
- ✅ CRUD operations implement করতে পারেন

পরবর্তী lesson এ আমরা **Advanced Routing** এবং **Express Router** সম্পর্কে শিখব।

## রেফারেন্স

- **Express Routing Guide**: https://expressjs.com/en/guide/routing.html
- **HTTP Methods**: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- **HTTP Status Codes**: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status

---

**পরবর্তী Lesson**: Express Router এবং Route Modularization

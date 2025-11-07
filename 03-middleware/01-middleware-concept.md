# Middleware - Concept এবং Fundamentals

## সংক্ষিপ্ত পরিচিতি (Brief Overview)

**Middleware** হলো এমন functions যা request-response cycle এর মাঝে execute হয়। এগুলো request object (`req`), response object (`res`) এবং `next` function এর access পায়।

Middleware হলো Express.js এর সবচেয়ে powerful feature যা দিয়ে আপনি:
- Request logging করতে পারেন
- Authentication/Authorization implement করতে পারেন
- Request data parse করতে পারেন
- Error handling করতে পারেন
- Response modify করতে পারেন

এই lesson এ আমরা শিখব:
- Middleware কী এবং কীভাবে কাজ করে
- বিভিন্ন ধরনের middleware
- Custom middleware তৈরি করা
- Built-in এবং third-party middleware
- Middleware execution order

## Middleware কী?

### সহজ সংজ্ঞা:

Middleware হলো এমন function যা:
1. Request আসার পরে execute হয়
2. Response পাঠানোর আগে execute হয়
3. Request এবং response এর মধ্যে "মাঝামাঝি" (middle) কাজ করে

### Visual Representation:

```
Client → [Middleware 1] → [Middleware 2] → [Middleware 3] → Route Handler → Response
         ↓                ↓                 ↓                ↓
      Logging         Authentication    Validation      Process Request
```

### মৌলিক Structure:

```javascript
function middlewareFunction(req, res, next) {
  // কিছু কাজ করুন
  console.log('Middleware executed');
  
  // পরবর্তী middleware/route handler এ যান
  next();
}
```

## Middleware এর ৩টি মূল Component

### 1. `req` (Request Object)
Client থেকে আসা সব তথ্য এখানে থাকে:
```javascript
req.params    // URL parameters
req.query     // Query strings
req.body      // Request body data
req.headers   // HTTP headers
req.method    // HTTP method (GET, POST, etc.)
req.url       // Request URL
```

### 2. `res` (Response Object)
Client কে response পাঠানোর জন্য:
```javascript
res.send()      // Response পাঠান
res.json()      // JSON response
res.status()    // Status code set করুন
res.render()    // Template render করুন
```

### 3. `next` (Next Function)
পরবর্তী middleware/route handler এ control পাঠান:
```javascript
next()          // পরবর্তী middleware তে যান
next('route')   // পরবর্তী route এ skip করুন
next(error)     // Error handling middleware তে যান
```

## প্রথম Middleware উদাহরণ

### Simple Logger Middleware:

```javascript
const express = require('express');
const app = express();

// Custom middleware
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  console.log('Time:', new Date().toLocaleString());
  next(); // গুরুত্বপূর্ণ! next() call না করলে request আটকে যাবে
};

// Middleware apply করুন
app.use(logger);

// Routes
app.get('/', (req, res) => {
  res.send('হোম পেজ');
});

app.get('/about', (req, res) => {
  res.send('About পেজ');
});

app.listen(3000, () => {
  console.log('Server চলছে: http://localhost:3000');
});
```

**Output (Console):**
```
GET /
Time: 11/7/2025, 10:30:45 AM

GET /about
Time: 11/7/2025, 10:31:20 AM
```

## Middleware এর ধরন (Types of Middleware)

### 1. Application-Level Middleware

পুরো application এর জন্য:

```javascript
const express = require('express');
const app = express();

// সব routes এর জন্য
app.use((req, res, next) => {
  console.log('Application-level middleware');
  next();
});

// Specific path এর জন্য
app.use('/users', (req, res, next) => {
  console.log('শুধু /users route এর জন্য');
  next();
});

app.get('/users', (req, res) => {
  res.send('Users list');
});
```

### 2. Router-Level Middleware

Specific router এর জন্য:

```javascript
const express = require('express');
const router = express.Router();

// Router-level middleware
router.use((req, res, next) => {
  console.log('Router middleware');
  next();
});

router.get('/profile', (req, res) => {
  res.send('Profile');
});

module.exports = router;
```

### 3. Built-in Middleware

Express.js এ built-in:

```javascript
// JSON parsing
app.use(express.json());

// URL-encoded data parsing (form data)
app.use(express.urlencoded({ extended: true }));

// Static files serve করা
app.use(express.static('public'));
```

### 4. Third-Party Middleware

External packages:

```javascript
// Morgan - HTTP request logger
const morgan = require('morgan');
app.use(morgan('dev'));

// CORS - Cross-Origin Resource Sharing
const cors = require('cors');
app.use(cors());

// Helmet - Security headers
const helmet = require('helmet');
app.use(helmet());

// Cookie Parser
const cookieParser = require('cookie-parser');
app.use(cookieParser());
```

### 5. Error-Handling Middleware

Error handle করার জন্য (4টি parameter):

```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'কিছু একটা ভুল হয়েছে!',
    message: err.message
  });
});
```

## Custom Middleware তৈরি করা

### উদাহরণ ১: Request Time Logger

```javascript
const requestTime = (req, res, next) => {
  req.requestTime = Date.now();
  next();
};

app.use(requestTime);

app.get('/', (req, res) => {
  const responseTime = Date.now() - req.requestTime;
  res.send(`Response সময়: ${responseTime}ms`);
});
```

### উদাহরণ ২: Authentication Middleware

```javascript
const authenticate = (req, res, next) => {
  const token = req.headers['authorization'];
  
  if (!token) {
    return res.status(401).json({
      error: 'Authorization token প্রয়োজন'
    });
  }
  
  // Token verify করুন (simplified)
  if (token === 'valid-token') {
    req.user = { id: 1, name: 'রহিম' };
    next(); // Authenticated, proceed
  } else {
    return res.status(403).json({
      error: 'Invalid token'
    });
  }
};

// Protected route
app.get('/dashboard', authenticate, (req, res) => {
  res.json({
    message: 'Dashboard',
    user: req.user
  });
});
```

### উদাহরণ ৩: Request Validation

```javascript
const validateUser = (req, res, next) => {
  const { name, email } = req.body;
  
  // Validation checks
  if (!name || name.trim().length === 0) {
    return res.status(400).json({
      error: 'Name প্রয়োজন'
    });
  }
  
  if (!email || !email.includes('@')) {
    return res.status(400).json({
      error: 'Valid email প্রয়োজন'
    });
  }
  
  // Validation passed
  next();
};

app.post('/users', validateUser, (req, res) => {
  res.status(201).json({
    message: 'User তৈরি হয়েছে',
    user: req.body
  });
});
```

### উদাহরণ ৪: Rate Limiting (Simple)

```javascript
const rateLimitMap = new Map();

const rateLimit = (req, res, next) => {
  const ip = req.ip;
  const now = Date.now();
  const windowMs = 60000; // 1 minute
  const maxRequests = 10;
  
  if (!rateLimitMap.has(ip)) {
    rateLimitMap.set(ip, { count: 1, startTime: now });
    return next();
  }
  
  const userData = rateLimitMap.get(ip);
  
  if (now - userData.startTime > windowMs) {
    // Reset window
    rateLimitMap.set(ip, { count: 1, startTime: now });
    return next();
  }
  
  if (userData.count >= maxRequests) {
    return res.status(429).json({
      error: 'অনেক বেশি requests! একটু পরে চেষ্টা করুন।'
    });
  }
  
  userData.count++;
  next();
};

app.use(rateLimit);
```

## Middleware Execution Order

Middleware যে order এ define করা হয় সেই order এ execute হয়:

```javascript
const express = require('express');
const app = express();

// Middleware 1
app.use((req, res, next) => {
  console.log('1. First middleware');
  next();
});

// Middleware 2
app.use((req, res, next) => {
  console.log('2. Second middleware');
  next();
});

// Middleware 3 - শুধু /api routes এর জন্য
app.use('/api', (req, res, next) => {
  console.log('3. API middleware');
  next();
});

// Route handler
app.get('/api/users', (req, res) => {
  console.log('4. Route handler');
  res.send('Users');
});

app.listen(3000);
```

**Output (যখন GET /api/users request আসবে):**
```
1. First middleware
2. Second middleware
3. API middleware
4. Route handler
```

## Multiple Middleware in Single Route

একটি route এ multiple middleware:

```javascript
// Method 1: Array
app.get('/users', 
  [authenticate, authorize('admin'), validateQuery],
  (req, res) => {
    res.send('Users list');
  }
);

// Method 2: Multiple arguments
app.get('/users', 
  authenticate, 
  authorize('admin'), 
  validateQuery,
  (req, res) => {
    res.send('Users list');
  }
);

// Method 3: Chaining
app.route('/users')
  .all(authenticate)
  .get(authorize('admin'), getUsers)
  .post(authorize('admin'), validateUser, createUser);
```

## Middleware Best Practices উদাহরণ

### 1. Modular Middleware Structure

**middleware/logger.js:**
```javascript
const logger = (req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
};

module.exports = logger;
```

**middleware/auth.js:**
```javascript
const jwt = require('jsonwebtoken');

const authenticate = (req, res, next) => {
  try {
    const token = req.headers['authorization']?.split(' ')[1];
    
    if (!token) {
      return res.status(401).json({ error: 'Token প্রয়োজন' });
    }
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid token' });
  }
};

const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        error: 'আপনার এই access নেই'
      });
    }
    next();
  };
};

module.exports = { authenticate, authorize };
```

**middleware/validation.js:**
```javascript
const validateUser = (req, res, next) => {
  const { name, email, password } = req.body;
  const errors = [];
  
  if (!name || name.trim().length < 3) {
    errors.push('Name কমপক্ষে 3 characters হতে হবে');
  }
  
  if (!email || !email.match(/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/)) {
    errors.push('Valid email প্রয়োজন');
  }
  
  if (!password || password.length < 6) {
    errors.push('Password কমপক্ষে 6 characters হতে হবে');
  }
  
  if (errors.length > 0) {
    return res.status(400).json({ errors });
  }
  
  next();
};

module.exports = { validateUser };
```

**index.js:**
```javascript
const express = require('express');
const app = express();

// Import middleware
const logger = require('./middleware/logger');
const { authenticate, authorize } = require('./middleware/auth');
const { validateUser } = require('./middleware/validation');

// Global middleware
app.use(express.json());
app.use(logger);

// Public routes
app.get('/', (req, res) => {
  res.send('হোম পেজ');
});

// Protected routes
app.get('/dashboard', authenticate, (req, res) => {
  res.json({ message: 'Dashboard', user: req.user });
});

app.get('/admin', authenticate, authorize('admin'), (req, res) => {
  res.json({ message: 'Admin panel' });
});

// Route with validation
app.post('/register', validateUser, (req, res) => {
  res.status(201).json({
    message: 'User registered',
    user: req.body
  });
});

app.listen(3000);
```

## Advanced Middleware Patterns

### 1. Middleware Factory (Parameterized Middleware)

```javascript
// Timeout middleware
const timeout = (ms) => {
  return (req, res, next) => {
    const timer = setTimeout(() => {
      res.status(408).json({
        error: 'Request timeout'
      });
    }, ms);
    
    res.on('finish', () => {
      clearTimeout(timer);
    });
    
    next();
  };
};

// Use করুন
app.use('/api', timeout(5000)); // 5 second timeout
```

### 2. Conditional Middleware

```javascript
const conditionalMiddleware = (condition, middleware) => {
  return (req, res, next) => {
    if (condition(req)) {
      return middleware(req, res, next);
    }
    next();
  };
};

// Use করুন
app.use(conditionalMiddleware(
  req => req.path.startsWith('/api'),
  authenticate
));
```

### 3. Async Middleware Wrapper

```javascript
const asyncHandler = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// Use করুন
app.get('/users', asyncHandler(async (req, res) => {
  const users = await User.find();
  res.json(users);
}));
```

## সম্পূর্ণ উদাহরণ: Blog API with Middleware

```javascript
const express = require('express');
const app = express();

// Global middleware
app.use(express.json());

// Logger middleware
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  req.timestamp = Date.now();
  next();
});

// Dummy data
let posts = [
  { id: 1, title: 'First Post', author: 'রহিম', content: 'Content...' }
];

// Authentication middleware
const authenticate = (req, res, next) => {
  const apiKey = req.headers['x-api-key'];
  
  if (apiKey === 'secret-key') {
    req.authenticated = true;
    next();
  } else {
    res.status(401).json({ error: 'Unauthorized' });
  }
};

// Validation middleware
const validatePost = (req, res, next) => {
  const { title, content } = req.body;
  
  if (!title || !content) {
    return res.status(400).json({
      error: 'Title এবং content প্রয়োজন'
    });
  }
  
  if (title.length < 5) {
    return res.status(400).json({
      error: 'Title কমপক্ষে 5 characters হতে হবে'
    });
  }
  
  next();
};

// Check if post exists
const checkPostExists = (req, res, next) => {
  const post = posts.find(p => p.id === parseInt(req.params.id));
  
  if (!post) {
    return res.status(404).json({
      error: 'Post পাওয়া যায়নি'
    });
  }
  
  req.post = post; // Attach to request
  next();
};

// Response time middleware
app.use((req, res, next) => {
  res.on('finish', () => {
    const duration = Date.now() - req.timestamp;
    console.log(`Response সময়: ${duration}ms`);
  });
  next();
});

// Routes
// Public - GET all posts
app.get('/posts', (req, res) => {
  res.json({ count: posts.length, posts });
});

// Public - GET single post
app.get('/posts/:id', checkPostExists, (req, res) => {
  res.json(req.post);
});

// Protected - CREATE post
app.post('/posts', authenticate, validatePost, (req, res) => {
  const newPost = {
    id: posts.length + 1,
    ...req.body,
    createdAt: new Date()
  };
  
  posts.push(newPost);
  
  res.status(201).json({
    message: 'Post created',
    post: newPost
  });
});

// Protected - UPDATE post
app.put('/posts/:id', authenticate, checkPostExists, validatePost, (req, res) => {
  const postIndex = posts.findIndex(p => p.id === parseInt(req.params.id));
  
  posts[postIndex] = {
    ...posts[postIndex],
    ...req.body,
    updatedAt: new Date()
  };
  
  res.json({
    message: 'Post updated',
    post: posts[postIndex]
  });
});

// Protected - DELETE post
app.delete('/posts/:id', authenticate, checkPostExists, (req, res) => {
  posts = posts.filter(p => p.id !== parseInt(req.params.id));
  
  res.json({
    message: 'Post deleted'
  });
});

// 404 Handler
app.use((req, res) => {
  res.status(404).json({
    error: 'Route পাওয়া যায়নি'
  });
});

// Error Handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'Server error',
    message: err.message
  });
});

app.listen(3000, () => {
  console.log('Server চলছে: http://localhost:3000');
});
```

## Testing Middleware

```bash
# Public route - কাজ করবে
curl http://localhost:3000/posts

# Protected route - authentication ছাড়া (fail)
curl -X POST http://localhost:3000/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"New Post","content":"Content..."}'

# Protected route - authentication সহ (success)
curl -X POST http://localhost:3000/posts \
  -H "Content-Type: application/json" \
  -H "x-api-key: secret-key" \
  -d '{"title":"New Post","content":"Content..."}'
```

## সাধারণ ভুল এবং সমাধান

### ❌ ভুল ১: `next()` call করতে ভুলে যাওয়া

```javascript
// ভুল - next() নেই
app.use((req, res, next) => {
  console.log('Middleware');
  // next() call করা হয়নি - request আটকে যাবে!
});
```

**সমাধান:**
```javascript
// সঠিক
app.use((req, res, next) => {
  console.log('Middleware');
  next(); // ✅ Always call next()
});
```

### ❌ ভুল ২: Response পাঠানোর পর next() call করা

```javascript
// ভুল
app.use((req, res, next) => {
  res.send('Response');
  next(); // ❌ Response এর পরে next() - error!
});
```

**সমাধান:**
```javascript
// সঠিক
app.use((req, res, next) => {
  if (someCondition) {
    return res.send('Response'); // return ব্যবহার করুন
  }
  next();
});
```

### ❌ ভুল ৩: Error handling middleware এর signature ভুল

```javascript
// ভুল - 3 parameters (এটি error handler নয়)
app.use((err, req, res) => { // ❌ next missing
  res.status(500).send('Error');
});
```

**সমাধান:**
```javascript
// সঠিক - 4 parameters required for error handler
app.use((err, req, res, next) => { // ✅
  res.status(500).send('Error');
});
```

## Middleware Best Practices

### ✅ ১. Order matters - সঠিক ক্রমে middleware রাখুন

```javascript
// সঠিক order
app.use(express.json());           // 1. Body parsing
app.use(logger);                   // 2. Logging
app.use(authenticate);             // 3. Authentication
app.use('/api', apiRoutes);        // 4. Routes
app.use(errorHandler);             // 5. Error handling (শেষে)
```

### ✅ ২. Middleware আলাদা files এ রাখুন

```javascript
// middleware/index.js
module.exports = {
  logger: require('./logger'),
  auth: require('./auth'),
  validation: require('./validation')
};
```

### ✅ ৩. Error handling সঠিকভাবে করুন

```javascript
app.use((err, req, res, next) => {
  // Log error
  console.error(err);
  
  // Send appropriate response
  res.status(err.status || 500).json({
    error: err.message || 'Server error'
  });
});
```

### ✅ ৪. Reusable middleware তৈরি করুন

```javascript
// একবার তৈরি করুন, বারবার use করুন
const checkRole = (role) => {
  return (req, res, next) => {
    if (req.user.role === role) {
      next();
    } else {
      res.status(403).json({ error: 'Forbidden' });
    }
  };
};

app.get('/admin', checkRole('admin'), adminHandler);
app.get('/moderator', checkRole('moderator'), modHandler);
```

## পরবর্তী পদক্ষেপ

এখন আপনি:
- ✅ Middleware concept বুঝতে পারেন
- ✅ Custom middleware তৈরি করতে পারেন
- ✅ Built-in এবং third-party middleware ব্যবহার করতে পারেন
- ✅ Middleware chaining করতে পারেন

পরবর্তী lesson এ আমরা **Popular Third-Party Middleware** এবং তাদের ব্যবহার শিখব।

## রেফারেন্স

- **Express Middleware Guide**: https://expressjs.com/en/guide/using-middleware.html
- **Writing Middleware**: https://expressjs.com/en/guide/writing-middleware.html

---

**পরবর্তী Lesson**: Popular Third-Party Middleware (Morgan, CORS, Helmet, etc.)

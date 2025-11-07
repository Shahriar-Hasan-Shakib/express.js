# Express Router এবং Advanced Routing

## সংক্ষিপ্ত পরিচিতি (Brief Overview)

**Express Router** হলো একটি mini application যা শুধুমাত্র routing এবং middleware functions handle করে। এটি আপনার application কে modular এবং organized রাখতে সাহায্য করে।

এই lesson এ আমরা শিখব:
- Express Router কীভাবে তৈরি এবং ব্যবহার করতে হয়
- Route modularization
- Router-level middleware
- Nested routes
- Route prefixing
- Best practices for large applications

## কেন Express Router ব্যবহার করবেন?

### সমস্যা: বড় Application এ সব routes একসাথে

```javascript
// index.js - অসংগঠিত এবং বড়
const express = require('express');
const app = express();

// User routes
app.get('/users', getAllUsers);
app.get('/users/:id', getUser);
app.post('/users', createUser);
app.put('/users/:id', updateUser);
app.delete('/users/:id', deleteUser);

// Product routes
app.get('/products', getAllProducts);
app.get('/products/:id', getProduct);
app.post('/products', createProduct);
// ... আরও অনেক routes

// Order routes
app.get('/orders', getAllOrders);
// ... আরও routes

app.listen(3000);
```

এই approach এর সমস্যা:
- ❌ সব কিছু এক file এ - পড়তে কঠিন
- ❌ Maintainability কম
- ❌ Team collaboration কঠিন
- ❌ Testing কঠিন

### সমাধান: Express Router দিয়ে Modularization

```javascript
// routes/users.js
// routes/products.js
// routes/orders.js
// প্রতিটি feature আলাদা file এ
```

## Express Router তৈরি করা

### Basic Router Example

**routes/users.js:**
```javascript
const express = require('express');
const router = express.Router();

// Base path হবে /users (index.js এ define করা হবে)

// GET /users
router.get('/', (req, res) => {
  res.json({
    message: 'সব users',
    users: [
      { id: 1, name: 'রহিম' },
      { id: 2, name: 'করিম' }
    ]
  });
});

// GET /users/:id
router.get('/:id', (req, res) => {
  res.json({
    message: 'একটি user',
    userId: req.params.id
  });
});

// POST /users
router.post('/', (req, res) => {
  res.status(201).json({
    message: 'নতুন user তৈরি হয়েছে',
    user: req.body
  });
});

// PUT /users/:id
router.put('/:id', (req, res) => {
  res.json({
    message: 'User update হয়েছে',
    userId: req.params.id,
    data: req.body
  });
});

// DELETE /users/:id
router.delete('/:id', (req, res) => {
  res.json({
    message: 'User মুছে ফেলা হয়েছে',
    userId: req.params.id
  });
});

module.exports = router;
```

**index.js:**
```javascript
const express = require('express');
const app = express();

// Middleware
app.use(express.json());

// Import routes
const userRoutes = require('./routes/users');

// Use routes
app.use('/users', userRoutes);

// এখন সব user routes '/users' prefix এর সাথে কাজ করবে
// /users, /users/:id ইত্যাদি

app.listen(3000, () => {
  console.log('Server চলছে: http://localhost:3000');
});
```

## সম্পূর্ণ Modular Structure

### Project Structure:

```
my-express-app/
│
├── routes/
│   ├── index.js          # Main router
│   ├── users.js          # User routes
│   ├── products.js       # Product routes
│   └── orders.js         # Order routes
│
├── controllers/
│   ├── userController.js
│   ├── productController.js
│   └── orderController.js
│
├── middleware/
│   ├── auth.js
│   └── validation.js
│
├── index.js              # Main app file
└── package.json
```

### routes/users.js (Controller সহ):

```javascript
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const authMiddleware = require('../middleware/auth');

// Public routes
router.get('/', userController.getAllUsers);
router.get('/:id', userController.getUserById);

// Protected routes (authentication required)
router.post('/', authMiddleware, userController.createUser);
router.put('/:id', authMiddleware, userController.updateUser);
router.delete('/:id', authMiddleware, userController.deleteUser);

module.exports = router;
```

### controllers/userController.js:

```javascript
// Dummy data
let users = [
  { id: 1, name: 'রহিম', email: 'rahim@example.com' },
  { id: 2, name: 'করিম', email: 'karim@example.com' }
];

// Get all users
exports.getAllUsers = (req, res) => {
  res.json({
    success: true,
    count: users.length,
    data: users
  });
};

// Get single user
exports.getUserById = (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({
      success: false,
      error: 'User পাওয়া যায়নি'
    });
  }
  
  res.json({
    success: true,
    data: user
  });
};

// Create user
exports.createUser = (req, res) => {
  const { name, email } = req.body;
  
  // Validation
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
    message: 'User তৈরি হয়েছে',
    data: newUser
  });
};

// Update user
exports.updateUser = (req, res) => {
  const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
  
  if (userIndex === -1) {
    return res.status(404).json({
      success: false,
      error: 'User পাওয়া যায়নি'
    });
  }
  
  users[userIndex] = {
    ...users[userIndex],
    ...req.body,
    id: parseInt(req.params.id)
  };
  
  res.json({
    success: true,
    message: 'User update হয়েছে',
    data: users[userIndex]
  });
};

// Delete user
exports.deleteUser = (req, res) => {
  const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
  
  if (userIndex === -1) {
    return res.status(404).json({
      success: false,
      error: 'User পাওয়া যায়নি'
    });
  }
  
  const deletedUser = users.splice(userIndex, 1)[0];
  
  res.json({
    success: true,
    message: 'User মুছে ফেলা হয়েছে',
    data: deletedUser
  });
};
```

### routes/products.js:

```javascript
const express = require('express');
const router = express.Router();
const productController = require('../controllers/productController');

router.get('/', productController.getAllProducts);
router.get('/:id', productController.getProductById);
router.post('/', productController.createProduct);
router.put('/:id', productController.updateProduct);
router.delete('/:id', productController.deleteProduct);

// Special routes
router.get('/category/:category', productController.getProductsByCategory);
router.get('/search', productController.searchProducts);

module.exports = router;
```

### controllers/productController.js:

```javascript
let products = [
  { id: 1, name: 'ল্যাপটপ', price: 50000, category: 'electronics' },
  { id: 2, name: 'মোবাইল', price: 15000, category: 'electronics' },
  { id: 3, name: 'বই', price: 300, category: 'books' }
];

exports.getAllProducts = (req, res) => {
  // Query string filtering
  let filtered = products;
  
  if (req.query.minPrice) {
    filtered = filtered.filter(p => p.price >= parseInt(req.query.minPrice));
  }
  
  if (req.query.maxPrice) {
    filtered = filtered.filter(p => p.price <= parseInt(req.query.maxPrice));
  }
  
  res.json({
    success: true,
    count: filtered.length,
    data: filtered
  });
};

exports.getProductById = (req, res) => {
  const product = products.find(p => p.id === parseInt(req.params.id));
  
  if (!product) {
    return res.status(404).json({
      success: false,
      error: 'Product পাওয়া যায়নি'
    });
  }
  
  res.json({
    success: true,
    data: product
  });
};

exports.createProduct = (req, res) => {
  const { name, price, category } = req.body;
  
  if (!name || !price || !category) {
    return res.status(400).json({
      success: false,
      error: 'Name, price এবং category প্রয়োজন'
    });
  }
  
  const newProduct = {
    id: products.length + 1,
    name,
    price: parseFloat(price),
    category
  };
  
  products.push(newProduct);
  
  res.status(201).json({
    success: true,
    message: 'Product তৈরি হয়েছে',
    data: newProduct
  });
};

exports.updateProduct = (req, res) => {
  const productIndex = products.findIndex(p => p.id === parseInt(req.params.id));
  
  if (productIndex === -1) {
    return res.status(404).json({
      success: false,
      error: 'Product পাওয়া যায়নি'
    });
  }
  
  products[productIndex] = {
    ...products[productIndex],
    ...req.body,
    id: parseInt(req.params.id)
  };
  
  res.json({
    success: true,
    message: 'Product update হয়েছে',
    data: products[productIndex]
  });
};

exports.deleteProduct = (req, res) => {
  const productIndex = products.findIndex(p => p.id === parseInt(req.params.id));
  
  if (productIndex === -1) {
    return res.status(404).json({
      success: false,
      error: 'Product পাওয়া যায়নি'
    });
  }
  
  const deletedProduct = products.splice(productIndex, 1)[0];
  
  res.json({
    success: true,
    message: 'Product মুছে ফেলা হয়েছে',
    data: deletedProduct
  });
};

exports.getProductsByCategory = (req, res) => {
  const categoryProducts = products.filter(p => 
    p.category.toLowerCase() === req.params.category.toLowerCase()
  );
  
  res.json({
    success: true,
    category: req.params.category,
    count: categoryProducts.length,
    data: categoryProducts
  });
};

exports.searchProducts = (req, res) => {
  const { q } = req.query;
  
  if (!q) {
    return res.status(400).json({
      success: false,
      error: 'Search query প্রয়োজন'
    });
  }
  
  const results = products.filter(p => 
    p.name.toLowerCase().includes(q.toLowerCase())
  );
  
  res.json({
    success: true,
    query: q,
    count: results.length,
    data: results
  });
};
```

### routes/index.js (Main Router):

```javascript
const express = require('express');
const router = express.Router();

// Import sub-routers
const userRoutes = require('./users');
const productRoutes = require('./products');
const orderRoutes = require('./orders');

// API info
router.get('/', (req, res) => {
  res.json({
    message: 'Welcome to API',
    version: '1.0.0',
    endpoints: {
      users: '/api/users',
      products: '/api/products',
      orders: '/api/orders'
    }
  });
});

// Mount sub-routers
router.use('/users', userRoutes);
router.use('/products', productRoutes);
router.use('/orders', orderRoutes);

module.exports = router;
```

### index.js (Main Application):

```javascript
require('dotenv').config();
const express = require('express');
const app = express();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Logger middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// Import main router
const apiRouter = require('./routes');

// Mount API routes
app.use('/api', apiRouter);

// Home route
app.get('/', (req, res) => {
  res.json({
    message: 'Server চলছে',
    api: '/api'
  });
});

// 404 Handler
app.use((req, res) => {
  res.status(404).json({
    success: false,
    error: 'Route পাওয়া যায়নি'
  });
});

// Error Handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    success: false,
    error: 'Server error'
  });
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`✅ Server চলছে: http://localhost:${PORT}`);
});
```

## Router-Level Middleware

Router এর সব routes এর জন্য middleware apply করুন:

```javascript
const express = require('express');
const router = express.Router();

// Router-level middleware
router.use((req, res, next) => {
  console.log('User router accessed');
  console.log('Time:', Date.now());
  next();
});

// Specific route middleware
router.use('/:id', (req, res, next) => {
  console.log('User ID:', req.params.id);
  next();
});

// Routes
router.get('/', (req, res) => {
  res.send('All users');
});

router.get('/:id', (req, res) => {
  res.send(`User ${req.params.id}`);
});

module.exports = router;
```

## Nested Routers

Router এর মধ্যে আরেকটি router:

**routes/users.js:**
```javascript
const express = require('express');
const router = express.Router();

// Import nested router
const profileRouter = require('./userProfile');
const postsRouter = require('./userPosts');

// Main user routes
router.get('/', (req, res) => {
  res.send('All users');
});

router.get('/:id', (req, res) => {
  res.send(`User ${req.params.id}`);
});

// Nest routers
// /users/:userId/profile/*
router.use('/:userId/profile', profileRouter);

// /users/:userId/posts/*
router.use('/:userId/posts', postsRouter);

module.exports = router;
```

**routes/userProfile.js:**
```javascript
const express = require('express');
const router = express.Router({ mergeParams: true }); // mergeParams important!

// GET /users/:userId/profile
router.get('/', (req, res) => {
  res.json({
    message: 'User profile',
    userId: req.params.userId // parent router থেকে
  });
});

// PUT /users/:userId/profile
router.put('/', (req, res) => {
  res.json({
    message: 'Profile updated',
    userId: req.params.userId,
    data: req.body
  });
});

module.exports = router;
```

**routes/userPosts.js:**
```javascript
const express = require('express');
const router = express.Router({ mergeParams: true });

// GET /users/:userId/posts
router.get('/', (req, res) => {
  res.json({
    message: 'User posts',
    userId: req.params.userId
  });
});

// POST /users/:userId/posts
router.post('/', (req, res) => {
  res.json({
    message: 'New post created',
    userId: req.params.userId,
    post: req.body
  });
});

// GET /users/:userId/posts/:postId
router.get('/:postId', (req, res) => {
  res.json({
    userId: req.params.userId,
    postId: req.params.postId
  });
});

module.exports = router;
```

## Route Versioning

API versioning এর জন্য router ব্যবহার:

**routes/v1/users.js:**
```javascript
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({
    version: 'v1',
    users: ['রহিম', 'করিম']
  });
});

module.exports = router;
```

**routes/v2/users.js:**
```javascript
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({
    version: 'v2',
    users: [
      { id: 1, name: 'রহিম' },
      { id: 2, name: 'করিম' }
    ]
  });
});

module.exports = router;
```

**index.js:**
```javascript
const express = require('express');
const app = express();

// Import versions
const v1Routes = require('./routes/v1/users');
const v2Routes = require('./routes/v2/users');

// Mount versions
app.use('/api/v1/users', v1Routes);
app.use('/api/v2/users', v2Routes);

app.listen(3000);
```

## Middleware Authentication Example

**middleware/auth.js:**
```javascript
const auth = (req, res, next) => {
  const token = req.headers['authorization'];
  
  if (!token) {
    return res.status(401).json({
      success: false,
      error: 'Authorization token প্রয়োজন'
    });
  }
  
  // Token verification (simplified)
  if (token !== 'Bearer valid-token') {
    return res.status(403).json({
      success: false,
      error: 'Invalid token'
    });
  }
  
  // Add user info to request
  req.user = { id: 1, name: 'রহিম' };
  next();
};

module.exports = auth;
```

**routes/users.js (auth সহ):**
```javascript
const express = require('express');
const router = express.Router();
const auth = require('../middleware/auth');

// Public route
router.get('/', (req, res) => {
  res.send('Public: All users');
});

// Protected route
router.post('/', auth, (req, res) => {
  res.json({
    message: 'Protected: User created',
    createdBy: req.user.name
  });
});

// Multiple middleware
router.delete('/:id', [auth, isAdmin], (req, res) => {
  res.send('User deleted');
});

module.exports = router;
```

## পূর্ণাঙ্গ উদাহরণ: E-commerce API Structure

```
ecommerce-api/
│
├── routes/
│   ├── index.js
│   ├── auth.js
│   ├── users.js
│   ├── products.js
│   ├── categories.js
│   ├── cart.js
│   └── orders.js
│
├── controllers/
│   ├── authController.js
│   ├── userController.js
│   ├── productController.js
│   └── orderController.js
│
├── middleware/
│   ├── auth.js
│   ├── admin.js
│   └── validation.js
│
└── index.js
```

## সাধারণ ভুল এবং সমাধান

### ❌ ভুল ১: mergeParams ভুলে যাওয়া

```javascript
// Nested router এ parent params access করতে পারবেন না
const router = express.Router(); // ❌ Wrong

// সঠিক
const router = express.Router({ mergeParams: true }); // ✅
```

### ❌ ভুল ২: Router export করতে ভুলে যাওয়া

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.send('Users');
});

// ❌ module.exports ভুলে গেছেন
```

**সমাধান:**
```javascript
module.exports = router; // ✅ Always export
```

### ❌ ভুল ৩: Middleware order ভুল

```javascript
// ভুল - routes আগে, middleware পরে
app.use('/users', userRoutes);
app.use(express.json()); // ❌ Too late!

// সঠিক - middleware আগে, routes পরে
app.use(express.json()); // ✅
app.use('/users', userRoutes);
```

## Best Practices

### ✅ ১. Feature-based Organization

```
routes/
  users/
    index.js
    profile.js
    posts.js
  products/
    index.js
    reviews.js
```

### ✅ ২. Consistent Naming

```javascript
// File: userController.js
exports.getUsers = ...
exports.getUser = ...
exports.createUser = ...
exports.updateUser = ...
exports.deleteUser = ...
```

### ✅ ৩. Router Prefix সঠিকভাবে ব্যবহার করুন

```javascript
// index.js
app.use('/api/v1/users', userRoutes);

// routes/users.js - '/' means '/api/v1/users'
router.get('/', getAllUsers);
```

### ✅ ৪. Error Handling Middleware

```javascript
// routes/users.js
router.get('/:id', asyncHandler(getUserById));

// middleware/asyncHandler.js
const asyncHandler = fn => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};
```

## পরবর্তী পদক্ষেপ

এখন আপনি:
- ✅ Express Router তৈরি এবং ব্যবহার করতে পারেন
- ✅ Routes modularize করতে পারেন
- ✅ Nested routers তৈরি করতে পারেন
- ✅ Large-scale application structure তৈরি করতে পারেন

পরবর্তী lesson এ আমরা **Middleware** সম্পর্কে বিস্তারিত শিখব।

## রেফারেন্স

- **Express Router**: https://expressjs.com/en/guide/routing.html#express-router
- **Router API**: https://expressjs.com/en/5x/api.html#router

---

**পরবর্তী Lesson**: Middleware - Concept এবং Implementation

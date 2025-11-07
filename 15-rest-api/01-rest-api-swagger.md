# REST API Design এবং Documentation (Swagger/OpenAPI)

## সংক্ষিপ্ত পরিচিতি (Brief Overview)

**REST (Representational State Transfer)** হলো web services তৈরির একটি architectural style। এই lesson এ আমরা শিখব কীভাবে production-ready REST API তৈরি করতে হয়, সঠিক design principles follow করতে হয় এবং Swagger দিয়ে professional documentation তৈরি করতে হয়।

### এই Lesson এ যা শিখবেন:
- ✅ REST API principles এবং best practices
- ✅ RESTful URL structure design
- ✅ HTTP methods সঠিক ব্যবহার
- ✅ API versioning strategies
- ✅ Swagger/OpenAPI documentation setup (ES6)
- ✅ HATEOAS implementation
- ✅ Pagination, filtering, sorting
- ✅ Error responses standardization
- ✅ API security best practices

---

## REST API কী?

**REST (Representational State Transfer)** হলো একটি architectural style যেখানে:

- **Resources** হলো মূল concept (e.g., users, products, orders)
- প্রতিটি resource এর একটি unique **URI** থাকে
- **HTTP methods** দিয়ে operations করা হয় (GET, POST, PUT, DELETE)
- **Stateless** communication (server কোনো client state store করে না)
- **JSON** বা **XML** format এ data transfer

---

## REST Principles

### 1. Client-Server Architecture
- Client এবং Server আলাদা
- দুইটি independently evolve করতে পারে

### 2. Statelessness
- প্রতিটি request এ সব information থাকে
- Server কোনো session state রাখে না

### 3. Cacheability
- Responses cacheable বা non-cacheable হিসেবে mark করা থাকে
- Performance improve হয়

### 4. Uniform Interface
- Consistent URL structure
- Standard HTTP methods
- Predictable responses

### 5. Layered System
- Client জানে না directly server এর সাথে connected কিনা
- Load balancers, proxies থাকতে পারে

### 6. Code on Demand (Optional)
- Server executable code send করতে পারে

---

## RESTful URL Design Best Practices

### ✅ সঠিক URL Structure

```
# Resources হলো plural nouns
GET    /api/users              # সব users
GET    /api/users/123          # Specific user
POST   /api/users              # নতুন user তৈরি
PUT    /api/users/123          # User update (full)
PATCH  /api/users/123          # User update (partial)
DELETE /api/users/123          # User delete

# Nested resources
GET    /api/users/123/posts    # User 123 এর সব posts
GET    /api/users/123/posts/456 # User 123 এর post 456
POST   /api/users/123/posts    # User 123 এর জন্য নতুন post

# Query parameters for filtering/sorting
GET    /api/users?role=admin&status=active
GET    /api/products?sort=price&order=desc&limit=20
```

### ❌ ভুল URL Structure

```
# Verbs ব্যবহার করবেন না
❌ GET    /api/getUsers
❌ POST   /api/createUser
❌ POST   /api/deleteUser

# Inconsistent naming
❌ GET    /api/user        # Singular
❌ GET    /api/Users       # Capital letter
❌ GET    /api/user-list   # Hyphens in resource name

# Deep nesting (3+ levels avoid করুন)
❌ GET    /api/users/123/posts/456/comments/789/likes
```

---

## HTTP Methods সঠিক ব্যবহার

| Method | Purpose | Idempotent | Safe | Request Body | Response Body |
|--------|---------|------------|------|--------------|---------------|
| **GET** | Read/Retrieve | ✅ Yes | ✅ Yes | ❌ No | ✅ Yes |
| **POST** | Create | ❌ No | ❌ No | ✅ Yes | ✅ Yes |
| **PUT** | Update (full replace) | ✅ Yes | ❌ No | ✅ Yes | ✅ Yes |
| **PATCH** | Update (partial) | ❌ No | ❌ No | ✅ Yes | ✅ Yes |
| **DELETE** | Delete | ✅ Yes | ❌ No | ❌ Optional | ✅ Optional |

**Idempotent** মানে একই request multiple times করলে result একই থাকবে।

---

## Complete REST API Example (ES6)

### Project Structure

```
rest-api/
├── server.js
├── package.json
├── .env
│
├── src/
│   ├── config/
│   │   ├── database.js
│   │   └── swagger.js
│   │
│   ├── controllers/
│   │   └── userController.js
│   │
│   ├── models/
│   │   └── User.js
│   │
│   ├── routes/
│   │   └── userRoutes.js
│   │
│   ├── middleware/
│   │   ├── auth.js
│   │   └── validator.js
│   │
│   ├── utils/
│   │   ├── ApiError.js
│   │   └── ApiResponse.js
│   │
│   └── docs/
│       └── swagger.yaml
│
└── tests/
    └── user.test.js
```

### `package.json`

```json
{
  "name": "rest-api-express",
  "version": "1.0.0",
  "type": "module",
  "description": "RESTful API with Express and Swagger",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^5.1.0",
    "mongoose": "^8.0.3",
    "dotenv": "^16.4.5",
    "express-validator": "^7.0.1",
    "swagger-jsdoc": "^6.2.8",
    "swagger-ui-express": "^5.0.0",
    "cors": "^2.8.5",
    "helmet": "^7.1.0",
    "express-rate-limit": "^7.1.5",
    "jsonwebtoken": "^9.0.2",
    "bcryptjs": "^2.4.3"
  },
  "devDependencies": {
    "nodemon": "^3.1.0",
    "jest": "^29.7.0",
    "supertest": "^6.3.3"
  }
}
```

### `src/utils/ApiResponse.js`

```javascript
/**
 * Standardized API Response
 */
class ApiResponse {
  constructor(statusCode, data, message = 'Success', meta = {}) {
    this.success = statusCode < 400;
    this.statusCode = statusCode;
    this.message = message;
    this.data = data;
    
    if (Object.keys(meta).length > 0) {
      this.meta = meta;
    }
  }
  
  static success(data, message = 'Success', statusCode = 200, meta = {}) {
    return new ApiResponse(statusCode, data, message, meta);
  }
  
  static created(data, message = 'Resource created successfully') {
    return new ApiResponse(201, data, message);
  }
  
  static noContent(message = 'No content') {
    return new ApiResponse(204, null, message);
  }
}

export default ApiResponse;
```

### `src/utils/ApiError.js`

```javascript
/**
 * Standardized API Error
 */
class ApiError extends Error {
  constructor(statusCode, message, errors = [], stack = '') {
    super(message);
    this.statusCode = statusCode;
    this.success = false;
    this.errors = errors;
    
    if (stack) {
      this.stack = stack;
    } else {
      Error.captureStackTrace(this, this.constructor);
    }
  }
  
  static badRequest(message = 'Bad Request', errors = []) {
    return new ApiError(400, message, errors);
  }
  
  static unauthorized(message = 'Unauthorized') {
    return new ApiError(401, message);
  }
  
  static forbidden(message = 'Forbidden') {
    return new ApiError(403, message);
  }
  
  static notFound(message = 'Resource not found') {
    return new ApiError(404, message);
  }
  
  static conflict(message = 'Conflict') {
    return new ApiError(409, message);
  }
  
  static internal(message = 'Internal Server Error') {
    return new ApiError(500, message);
  }
}

export default ApiError;
```

### `src/models/User.js`

```javascript
import mongoose from 'mongoose';
import bcrypt from 'bcryptjs';

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    minlength: [2, 'Name must be at least 2 characters'],
    maxlength: [50, 'Name cannot exceed 50 characters']
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    trim: true,
    match: [/^\S+@\S+\.\S+$/, 'Please provide a valid email']
  },
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [6, 'Password must be at least 6 characters'],
    select: false // Password default এ return হবে না
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user'
  },
  avatar: {
    type: String,
    default: null
  },
  isActive: {
    type: Boolean,
    default: true
  },
  lastLogin: {
    type: Date,
    default: null
  }
}, {
  timestamps: true, // createdAt, updatedAt automatically add হবে
  toJSON: {
    transform: (doc, ret) => {
      delete ret.password;
      delete ret.__v;
      return ret;
    }
  }
});

// Password hashing middleware
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  
  this.password = await bcrypt.hash(this.password, 10);
  next();
});

// Password comparison method
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

// User statistics virtual field
userSchema.virtual('stats').get(function() {
  return {
    id: this._id,
    memberSince: this.createdAt,
    lastUpdated: this.updatedAt
  };
});

const User = mongoose.model('User', userSchema);

export default User;
```

### `src/controllers/userController.js`

```javascript
import User from '../models/User.js';
import ApiResponse from '../utils/ApiResponse.js';
import ApiError from '../utils/ApiError.js';

/**
 * @desc    Get all users with pagination, filtering, sorting
 * @route   GET /api/v1/users
 * @access  Public
 */
export const getUsers = async (req, res, next) => {
  try {
    // Pagination
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const skip = (page - 1) * limit;
    
    // Filtering
    const filter = {};
    if (req.query.role) filter.role = req.query.role;
    if (req.query.isActive !== undefined) filter.isActive = req.query.isActive === 'true';
    if (req.query.search) {
      filter.$or = [
        { name: { $regex: req.query.search, $options: 'i' } },
        { email: { $regex: req.query.search, $options: 'i' } }
      ];
    }
    
    // Sorting
    const sortBy = req.query.sort || 'createdAt';
    const sortOrder = req.query.order === 'asc' ? 1 : -1;
    const sort = { [sortBy]: sortOrder };
    
    // Query execution
    const users = await User.find(filter)
      .sort(sort)
      .skip(skip)
      .limit(limit)
      .select('-password');
    
    const total = await User.countDocuments(filter);
    
    // Pagination meta
    const meta = {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit),
      hasNextPage: page < Math.ceil(total / limit),
      hasPrevPage: page > 1
    };
    
    // HATEOAS links
    const links = {
      self: `/api/v1/users?page=${page}&limit=${limit}`,
      first: `/api/v1/users?page=1&limit=${limit}`,
      last: `/api/v1/users?page=${meta.pages}&limit=${limit}`
    };
    
    if (meta.hasNextPage) {
      links.next = `/api/v1/users?page=${page + 1}&limit=${limit}`;
    }
    
    if (meta.hasPrevPage) {
      links.prev = `/api/v1/users?page=${page - 1}&limit=${limit}`;
    }
    
    res.status(200).json(
      ApiResponse.success(
        users,
        'Users retrieved successfully',
        200,
        { pagination: meta, links }
      )
    );
  } catch (error) {
    next(error);
  }
};

/**
 * @desc    Get single user by ID
 * @route   GET /api/v1/users/:id
 * @access  Public
 */
export const getUserById = async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id).select('-password');
    
    if (!user) {
      throw ApiError.notFound('User not found');
    }
    
    res.status(200).json(
      ApiResponse.success(user, 'User retrieved successfully')
    );
  } catch (error) {
    next(error);
  }
};

/**
 * @desc    Create new user
 * @route   POST /api/v1/users
 * @access  Public
 */
export const createUser = async (req, res, next) => {
  try {
    const { name, email, password, role } = req.body;
    
    // Check if user already exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      throw ApiError.conflict('User with this email already exists');
    }
    
    // Create user
    const user = await User.create({
      name,
      email,
      password,
      role: role || 'user'
    });
    
    res.status(201).json(
      ApiResponse.created(user, 'User created successfully')
    );
  } catch (error) {
    next(error);
  }
};

/**
 * @desc    Update user (full update)
 * @route   PUT /api/v1/users/:id
 * @access  Private
 */
export const updateUser = async (req, res, next) => {
  try {
    const { name, email, role, isActive } = req.body;
    
    const user = await User.findById(req.params.id);
    
    if (!user) {
      throw ApiError.notFound('User not found');
    }
    
    // Update fields
    user.name = name;
    user.email = email;
    user.role = role;
    user.isActive = isActive;
    
    await user.save();
    
    res.status(200).json(
      ApiResponse.success(user, 'User updated successfully')
    );
  } catch (error) {
    next(error);
  }
};

/**
 * @desc    Update user (partial update)
 * @route   PATCH /api/v1/users/:id
 * @access  Private
 */
export const patchUser = async (req, res, next) => {
  try {
    const updates = req.body;
    
    // Don't allow password update through PATCH
    delete updates.password;
    
    const user = await User.findByIdAndUpdate(
      req.params.id,
      { $set: updates },
      { new: true, runValidators: true }
    ).select('-password');
    
    if (!user) {
      throw ApiError.notFound('User not found');
    }
    
    res.status(200).json(
      ApiResponse.success(user, 'User updated successfully')
    );
  } catch (error) {
    next(error);
  }
};

/**
 * @desc    Delete user
 * @route   DELETE /api/v1/users/:id
 * @access  Private/Admin
 */
export const deleteUser = async (req, res, next) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);
    
    if (!user) {
      throw ApiError.notFound('User not found');
    }
    
    res.status(200).json(
      ApiResponse.success(null, 'User deleted successfully')
    );
  } catch (error) {
    next(error);
  }
};

/**
 * @desc    Get user statistics
 * @route   GET /api/v1/users/stats
 * @access  Private/Admin
 */
export const getUserStats = async (req, res, next) => {
  try {
    const stats = await User.aggregate([
      {
        $group: {
          _id: '$role',
          count: { $sum: 1 }
        }
      },
      {
        $project: {
          _id: 0,
          role: '$_id',
          count: 1
        }
      }
    ]);
    
    const totalUsers = await User.countDocuments();
    const activeUsers = await User.countDocuments({ isActive: true });
    const inactiveUsers = await User.countDocuments({ isActive: false });
    
    res.status(200).json(
      ApiResponse.success({
        total: totalUsers,
        active: activeUsers,
        inactive: inactiveUsers,
        byRole: stats
      }, 'Statistics retrieved successfully')
    );
  } catch (error) {
    next(error);
  }
};
```

### `src/routes/userRoutes.js`

```javascript
import express from 'express';
import {
  getUsers,
  getUserById,
  createUser,
  updateUser,
  patchUser,
  deleteUser,
  getUserStats
} from '../controllers/userController.js';
import { validateUser } from '../middleware/validator.js';

const router = express.Router();

/**
 * @swagger
 * components:
 *   schemas:
 *     User:
 *       type: object
 *       required:
 *         - name
 *         - email
 *         - password
 *       properties:
 *         _id:
 *           type: string
 *           description: User ID
 *         name:
 *           type: string
 *           description: User's full name
 *         email:
 *           type: string
 *           format: email
 *           description: User's email address
 *         role:
 *           type: string
 *           enum: [user, admin, moderator]
 *           description: User role
 *         avatar:
 *           type: string
 *           description: Avatar URL
 *         isActive:
 *           type: boolean
 *           description: Account status
 *         createdAt:
 *           type: string
 *           format: date-time
 *         updatedAt:
 *           type: string
 *           format: date-time
 *       example:
 *         _id: 507f1f77bcf86cd799439011
 *         name: রহিম আহমেদ
 *         email: rahim@example.com
 *         role: user
 *         isActive: true
 *         createdAt: 2024-01-01T00:00:00.000Z
 */

/**
 * @swagger
 * /api/v1/users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *         description: Page number
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *         description: Items per page
 *       - in: query
 *         name: role
 *         schema:
 *           type: string
 *           enum: [user, admin, moderator]
 *         description: Filter by role
 *       - in: query
 *         name: search
 *         schema:
 *           type: string
 *         description: Search by name or email
 *     responses:
 *       200:
 *         description: Users retrieved successfully
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 message:
 *                   type: string
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 */
router.get('/', getUsers);

/**
 * @swagger
 * /api/v1/users/stats:
 *   get:
 *     summary: Get user statistics
 *     tags: [Users]
 *     responses:
 *       200:
 *         description: Statistics retrieved successfully
 */
router.get('/stats', getUserStats);

/**
 * @swagger
 * /api/v1/users/{id}:
 *   get:
 *     summary: Get user by ID
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *         description: User ID
 *     responses:
 *       200:
 *         description: User retrieved successfully
 *       404:
 *         description: User not found
 */
router.get('/:id', getUserById);

/**
 * @swagger
 * /api/v1/users:
 *   post:
 *     summary: Create new user
 *     tags: [Users]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *               - email
 *               - password
 *             properties:
 *               name:
 *                 type: string
 *               email:
 *                 type: string
 *                 format: email
 *               password:
 *                 type: string
 *                 format: password
 *               role:
 *                 type: string
 *                 enum: [user, admin, moderator]
 *     responses:
 *       201:
 *         description: User created successfully
 *       400:
 *         description: Validation error
 *       409:
 *         description: User already exists
 */
router.post('/', validateUser, createUser);

/**
 * @swagger
 * /api/v1/users/{id}:
 *   put:
 *     summary: Update user (full update)
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/User'
 *     responses:
 *       200:
 *         description: User updated successfully
 *       404:
 *         description: User not found
 */
router.put('/:id', updateUser);

/**
 * @swagger
 * /api/v1/users/{id}:
 *   patch:
 *     summary: Update user (partial update)
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               name:
 *                 type: string
 *               email:
 *                 type: string
 *               isActive:
 *                 type: boolean
 *     responses:
 *       200:
 *         description: User updated successfully
 *       404:
 *         description: User not found
 */
router.patch('/:id', patchUser);

/**
 * @swagger
 * /api/v1/users/{id}:
 *   delete:
 *     summary: Delete user
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *     responses:
 *       200:
 *         description: User deleted successfully
 *       404:
 *         description: User not found
 */
router.delete('/:id', deleteUser);

export default router;
```

### `src/middleware/validator.js`

```javascript
import { body, validationResult } from 'express-validator';
import ApiError from '../utils/ApiError.js';

export const validateUser = [
  body('name')
    .trim()
    .isLength({ min: 2, max: 50 })
    .withMessage('Name must be between 2 and 50 characters'),
  
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please provide a valid email'),
  
  body('password')
    .isLength({ min: 6 })
    .withMessage('Password must be at least 6 characters'),
  
  body('role')
    .optional()
    .isIn(['user', 'admin', 'moderator'])
    .withMessage('Invalid role'),
  
  (req, res, next) => {
    const errors = validationResult(req);
    
    if (!errors.isEmpty()) {
      const errorMessages = errors.array().map(err => ({
        field: err.path,
        message: err.msg
      }));
      
      throw ApiError.badRequest('Validation failed', errorMessages);
    }
    
    next();
  }
];
```

### `src/config/swagger.js`

```javascript
import swaggerJsdoc from 'swagger-jsdoc';

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'Express REST API Documentation',
      version: '1.0.0',
      description: 'Complete REST API documentation with Swagger/OpenAPI 3.0',
      contact: {
        name: 'API Support',
        email: 'support@example.com',
        url: 'https://example.com/support'
      },
      license: {
        name: 'MIT',
        url: 'https://opensource.org/licenses/MIT'
      }
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development server'
      },
      {
        url: 'https://api.example.com',
        description: 'Production server'
      }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      }
    },
    security: [
      {
        bearerAuth: []
      }
    ],
    tags: [
      {
        name: 'Users',
        description: 'User management endpoints'
      },
      {
        name: 'Auth',
        description: 'Authentication endpoints'
      }
    ]
  },
  apis: ['./src/routes/*.js'], // Path to route files
};

const swaggerSpec = swaggerJsdoc(options);

export default swaggerSpec;
```

### `server.js`

```javascript
import express from 'express';
import mongoose from 'mongoose';
import cors from 'cors';
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
import swaggerUi from 'swagger-ui-express';
import dotenv from 'dotenv';

import swaggerSpec from './src/config/swagger.js';
import userRoutes from './src/routes/userRoutes.js';
import ApiError from './src/utils/ApiError.js';

// Load environment variables
dotenv.config();

const app = express();

// Security middleware
app.use(helmet());
app.use(cors());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Max 100 requests per windowMs
  message: 'Too many requests from this IP, please try again later.'
});
app.use('/api', limiter);

// Body parser
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// API Documentation
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec, {
  customCss: '.swagger-ui .topbar { display: none }',
  customSiteTitle: 'REST API Documentation',
  customfavIcon: '/favicon.ico'
}));

// API v1 routes
app.use('/api/v1/users', userRoutes);

// Root route
app.get('/', (req, res) => {
  res.json({
    message: 'Welcome to Express REST API',
    version: '1.0.0',
    documentation: '/api-docs',
    endpoints: {
      users: '/api/v1/users',
      swagger: '/api-docs'
    }
  });
});

// Health check
app.get('/health', (req, res) => {
  res.json({
    status: 'OK',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    environment: process.env.NODE_ENV || 'development'
  });
});

// 404 Handler
app.use((req, res, next) => {
  next(ApiError.notFound(`Route ${req.originalUrl} not found`));
});

// Global error handler
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';
  
  res.status(statusCode).json({
    success: false,
    statusCode,
    message,
    errors: err.errors || [],
    stack: process.env.NODE_ENV === 'development' ? err.stack : undefined
  });
});

// Database connection
const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/rest-api');
    console.log('✅ MongoDB connected successfully');
  } catch (error) {
    console.error('❌ MongoDB connection error:', error);
    process.exit(1);
  }
};

// Start server
const PORT = process.env.PORT || 3000;

connectDB().then(() => {
  app.listen(PORT, () => {
    console.log(`🚀 Server running on http://localhost:${PORT}`);
    console.log(`📚 API Documentation: http://localhost:${PORT}/api-docs`);
    console.log(`🏥 Health check: http://localhost:${PORT}/health`);
  });
});

// Handle unhandled promise rejections
process.on('unhandledRejection', (err) => {
  console.error('❌ Unhandled Promise Rejection:', err);
  process.exit(1);
});
```

---

## API Versioning Strategies

### 1. URI Versioning (Most Common)

```javascript
// সবচেয়ে জনপ্রিয় এবং সহজ
app.use('/api/v1/users', userRoutesV1);
app.use('/api/v2/users', userRoutesV2);

// Example URLs:
// GET /api/v1/users
// GET /api/v2/users
```

### 2. Header Versioning

```javascript
app.use('/api/users', (req, res, next) => {
  const version = req.headers['api-version'] || '1';
  
  if (version === '1') {
    return userRoutesV1(req, res, next);
  } else if (version === '2') {
    return userRoutesV2(req, res, next);
  }
  
  res.status(400).json({ error: 'Unsupported API version' });
});

// Request:
// GET /api/users
// Header: api-version: 2
```

### 3. Accept Header Versioning

```javascript
app.use('/api/users', (req, res, next) => {
  const accept = req.headers.accept || '';
  
  if (accept.includes('application/vnd.api.v1+json')) {
    return userRoutesV1(req, res, next);
  } else if (accept.includes('application/vnd.api.v2+json')) {
    return userRoutesV2(req, res, next);
  }
  
  // Default version
  return userRoutesV1(req, res, next);
});

// Request:
// GET /api/users
// Header: Accept: application/vnd.api.v2+json
```

### 4. Query Parameter Versioning

```javascript
app.use('/api/users', (req, res, next) => {
  const version = req.query.version || '1';
  
  if (version === '1') {
    return userRoutesV1(req, res, next);
  } else if (version === '2') {
    return userRoutesV2(req, res, next);
  }
  
  res.status(400).json({ error: 'Invalid version' });
});

// Example URL:
// GET /api/users?version=2
```

**Recommendation**: URI versioning ব্যবহার করুন - এটি সবচেয়ে clear এবং developer-friendly।

---

## HATEOAS Implementation

**HATEOAS (Hypermedia As The Engine Of Application State)** মানে API response এ related links include করা:

```javascript
// Example HATEOAS response
{
  "success": true,
  "data": {
    "_id": "507f1f77bcf86cd799439011",
    "name": "রহিম আহমেদ",
    "email": "rahim@example.com",
    "role": "user"
  },
  "links": {
    "self": "/api/v1/users/507f1f77bcf86cd799439011",
    "posts": "/api/v1/users/507f1f77bcf86cd799439011/posts",
    "update": {
      "href": "/api/v1/users/507f1f77bcf86cd799439011",
      "method": "PUT"
    },
    "delete": {
      "href": "/api/v1/users/507f1f77bcf86cd799439011",
      "method": "DELETE"
    }
  }
}
```

### HATEOAS Helper Function

```javascript
export const addHATEOASLinks = (resource, resourceType, baseUrl = '/api/v1') => {
  const links = {
    self: `${baseUrl}/${resourceType}/${resource._id}`
  };
  
  // Add common actions
  links.update = {
    href: `${baseUrl}/${resourceType}/${resource._id}`,
    method: 'PUT'
  };
  
  links.patch = {
    href: `${baseUrl}/${resourceType}/${resource._id}`,
    method: 'PATCH'
  };
  
  links.delete = {
    href: `${baseUrl}/${resourceType}/${resource._id}`,
    method: 'DELETE'
  };
  
  // Add related resources
  if (resourceType === 'users') {
    links.posts = `${baseUrl}/users/${resource._id}/posts`;
    links.comments = `${baseUrl}/users/${resource._id}/comments`;
  }
  
  return { ...resource.toJSON(), links };
};

// Usage in controller
const user = await User.findById(id);
const userWithLinks = addHATEOASLinks(user, 'users');
res.json(ApiResponse.success(userWithLinks));
```

---

## ⚠️ Common Mistakes & Fixes

### ভুল ১: Inconsistent response structure

```javascript
// ❌ ভুল - Different response structures
app.get('/users', (req, res) => {
  res.json(users); // Just array
});

app.get('/users/:id', (req, res) => {
  res.json({ user }); // Wrapped object
});

// ✅ সঠিক - Consistent structure
app.get('/users', (req, res) => {
  res.json({
    success: true,
    data: users,
    message: 'Users retrieved'
  });
});

app.get('/users/:id', (req, res) => {
  res.json({
    success: true,
    data: user,
    message: 'User retrieved'
  });
});
```

### ভুল ২: Using verbs in URLs

```javascript
// ❌ ভুল
POST /api/createUser
GET  /api/getUser/123
POST /api/deleteUser/123

// ✅ সঠিক - Use HTTP methods
POST   /api/users          # Create
GET    /api/users/123      # Read
PUT    /api/users/123      # Update (full)
PATCH  /api/users/123      # Update (partial)
DELETE /api/users/123      # Delete
```

### ভুল ৩: Wrong HTTP status codes

```javascript
// ❌ ভুল - সব কিছুতেই 200 return
app.post('/users', (req, res) => {
  const user = createUser(req.body);
  res.status(200).json(user); // Wrong!
});

app.get('/users/999', (req, res) => {
  res.status(200).json({ error: 'Not found' }); // Wrong!
});

// ✅ সঠিক - Proper status codes
app.post('/users', (req, res) => {
  const user = createUser(req.body);
  res.status(201).json(user); // 201 Created
});

app.get('/users/999', (req, res) => {
  res.status(404).json({ error: 'Not found' }); // 404 Not Found
});
```

### ভুল ৪: Not handling validation errors properly

```javascript
// ❌ ভুল - Generic error message
app.post('/users', (req, res) => {
  if (!req.body.email) {
    return res.status(400).json({ error: 'Invalid data' });
  }
});

// ✅ সঠিক - Specific field errors
app.post('/users', (req, res) => {
  const errors = [];
  
  if (!req.body.email) {
    errors.push({ field: 'email', message: 'Email is required' });
  }
  
  if (!req.body.name) {
    errors.push({ field: 'name', message: 'Name is required' });
  }
  
  if (errors.length > 0) {
    return res.status(400).json({
      success: false,
      message: 'Validation failed',
      errors
    });
  }
});
```

### ভুল ৫: Not implementing pagination

```javascript
// ❌ ভুল - সব data একবারে return
app.get('/users', async (req, res) => {
  const users = await User.find(); // Might be thousands!
  res.json(users);
});

// ✅ সঠিক - Pagination implement
app.get('/users', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;
  
  const users = await User.find()
    .skip(skip)
    .limit(limit);
  
  const total = await User.countDocuments();
  
  res.json({
    data: users,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit)
    }
  });
});
```

### ভুল ৬: Exposing sensitive data

```javascript
// ❌ ভুল - Password return হচ্ছে
app.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json(user); // Includes password field!
});

// ✅ সঠিক - Sensitive data exclude
app.get('/users/:id', async (req, res) => {
  const user = await User.findById(req.params.id).select('-password');
  res.json(user);
});

// অথবা model এ
userSchema.set('toJSON', {
  transform: (doc, ret) => {
    delete ret.password;
    delete ret.__v;
    return ret;
  }
});
```

### ভুল ৭: Not versioning API

```javascript
// ❌ ভুল - No versioning
app.use('/api/users', userRoutes);

// ✅ সঠিক - Version in URL
app.use('/api/v1/users', userRoutesV1);
app.use('/api/v2/users', userRoutesV2);
```

### ভুল ৮: Improper error responses

```javascript
// ❌ ভুল - Stack trace exposed in production
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
    stack: err.stack // Exposes internal details!
  });
});

// ✅ সঠিক - Hide stack in production
app.use((err, req, res, next) => {
  res.status(err.statusCode || 500).json({
    success: false,
    message: err.message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
});
```

### ভুল ৯: Using GET for non-safe operations

```javascript
// ❌ ভুল - GET দিয়ে data delete
app.get('/users/:id/delete', async (req, res) => {
  await User.findByIdAndDelete(req.params.id);
  res.json({ message: 'Deleted' });
});

// ✅ সঠিক - DELETE method ব্যবহার
app.delete('/users/:id', async (req, res) => {
  await User.findByIdAndDelete(req.params.id);
  res.json({ message: 'Deleted' });
});
```

### ভুল ১০: Not documenting API

```javascript
// ❌ ভুল - No documentation
// Developers have to guess endpoints

// ✅ সঠিক - Swagger documentation
/**
 * @swagger
 * /api/v1/users:
 *   get:
 *     summary: Get all users
 *     description: Returns a list of users with pagination
 *     tags: [Users]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *     responses:
 *       200:
 *         description: Success
 */
router.get('/', getUsers);
```

---

## API Security Best Practices

### 1. Rate Limiting

```javascript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  message: 'Too many requests'
});

app.use('/api/', limiter);
```

### 2. Input Validation

```javascript
import { body, validationResult } from 'express-validator';

const validateInput = [
  body('email').isEmail(),
  body('password').isLength({ min: 6 }),
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    next();
  }
];
```

### 3. Authentication & Authorization

```javascript
import jwt from 'jsonwebtoken';

const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};

const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Access denied' });
    }
    next();
  };
};

// Usage
router.delete('/users/:id', authenticate, authorize('admin'), deleteUser);
```

### 4. CORS Configuration

```javascript
import cors from 'cors';

const corsOptions = {
  origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 86400 // 24 hours
};

app.use(cors(corsOptions));
```

---

## HTTP Status Codes Reference

| Code | Name | Usage |
|------|------|-------|
| **2xx Success** |||
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST (resource created) |
| 204 | No Content | Successful DELETE (no response body) |
| **4xx Client Errors** |||
| 400 | Bad Request | Validation errors, malformed data |
| 401 | Unauthorized | No/invalid authentication token |
| 403 | Forbidden | Authenticated but no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource already exists |
| 422 | Unprocessable Entity | Validation error (semantic) |
| 429 | Too Many Requests | Rate limit exceeded |
| **5xx Server Errors** |||
| 500 | Internal Server Error | Unexpected server error |
| 502 | Bad Gateway | Invalid response from upstream |
| 503 | Service Unavailable | Server overloaded/maintenance |

---

## Testing REST API

```bash
# Install testing dependencies
npm install --save-dev jest supertest
```

### `tests/user.test.js`

```javascript
import request from 'supertest';
import app from '../server.js';
import User from '../src/models/User.js';

describe('User API', () => {
  beforeAll(async () => {
    // Connect to test database
  });
  
  afterAll(async () => {
    // Close database connection
  });
  
  beforeEach(async () => {
    // Clear users collection
    await User.deleteMany({});
  });
  
  describe('GET /api/v1/users', () => {
    it('should return all users', async () => {
      const res = await request(app)
        .get('/api/v1/users')
        .expect(200);
      
      expect(res.body.success).toBe(true);
      expect(Array.isArray(res.body.data)).toBe(true);
    });
    
    it('should return paginated users', async () => {
      const res = await request(app)
        .get('/api/v1/users?page=1&limit=10')
        .expect(200);
      
      expect(res.body.meta.pagination).toBeDefined();
      expect(res.body.meta.pagination.page).toBe(1);
      expect(res.body.meta.pagination.limit).toBe(10);
    });
  });
  
  describe('POST /api/v1/users', () => {
    it('should create a new user', async () => {
      const userData = {
        name: 'Test User',
        email: 'test@example.com',
        password: 'password123'
      };
      
      const res = await request(app)
        .post('/api/v1/users')
        .send(userData)
        .expect(201);
      
      expect(res.body.success).toBe(true);
      expect(res.body.data.email).toBe(userData.email);
      expect(res.body.data.password).toBeUndefined();
    });
    
    it('should return 400 for invalid email', async () => {
      const userData = {
        name: 'Test User',
        email: 'invalid-email',
        password: 'password123'
      };
      
      const res = await request(app)
        .post('/api/v1/users')
        .send(userData)
        .expect(400);
      
      expect(res.body.success).toBe(false);
    });
  });
});
```

---

## রেফারেন্স এবং দরকারী লিংক

- **REST API Design Best Practices**: https://restfulapi.net/
- **HTTP Status Codes**: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status
- **Swagger/OpenAPI Specification**: https://swagger.io/specification/
- **swagger-jsdoc**: https://github.com/Surnet/swagger-jsdoc
- **swagger-ui-express**: https://github.com/scottie1984/swagger-ui-express
- **express-validator**: https://express-validator.github.io/docs/
- **REST API Tutorial**: https://www.restapitutorial.com/
- **API Design Guide (Google)**: https://cloud.google.com/apis/design

---

**সম্পূর্ণ Express.js Documentation শেষ!** 🎉

এখন আপনি জানেন কীভাবে production-ready REST APIs তৈরি করতে হয়, সঠিক design principles follow করতে হয় এবং professional documentation তৈরি করতে হয়।

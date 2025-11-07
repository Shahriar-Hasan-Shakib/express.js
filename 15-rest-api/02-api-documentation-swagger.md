# Swagger/OpenAPI Documentation

## সংক্ষিপ্ত পরিচিতি

**Swagger** (এখন **OpenAPI specification** নামে পরিচিত) হলো একটি standard যা REST APIs কে document এবং visualize করার জন্য ব্যবহৃত হয়।

## কেন API Documentation প্রয়োজন?

### ✅ সুবিধা:

1. **Developer Experience**: Developers সহজেই API বুঝতে পারে
2. **Interactive Testing**: Swagger UI তে directly test করতে পারেন
3. **Auto-generated**: Documentation automatically generate হয়
4. **Standardized Format**: OpenAPI standard follow করে
5. **Contract**: API consumer এবং producer এর মধ্যে contract
6. **Client Generation**: Tools দিয়ে client library generate করা যায়

## Swagger/OpenAPI Setup

### Step 1: Dependencies Install করুন

```bash
npm install swagger-jsdoc swagger-ui-express
```

### Step 2: Swagger Configuration

```javascript
import swaggerJsdoc from 'swagger-jsdoc';
import swaggerUi from 'swagger-ui-express';

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'REST API Documentation',
      version: '1.0.0',
      description: 'Express.js REST API with Swagger/OpenAPI'
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
    ]
  },
  apis: ['./src/routes/*.js'] // Route files path
};

const swaggerSpec = swaggerJsdoc(options);

// Swagger UI mount করুন
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

## JSDoc Comments দিয়ে Document করুন

### GET Endpoint

```javascript
/**
 * @swagger
 * /api/v1/users:
 *   get:
 *     summary: Get all users
 *     description: Retrieve a list of all users with pagination, filtering, and sorting
 *     tags:
 *       - Users
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *         description: Page number (default 1)
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *         description: Items per page (default 10)
 *       - in: query
 *         name: role
 *         schema:
 *           type: string
 *           enum: [user, admin, moderator]
 *         description: Filter by user role
 *       - in: query
 *         name: search
 *         schema:
 *           type: string
 *         description: Search in name and email
 *       - in: query
 *         name: sortBy
 *         schema:
 *           type: string
 *         description: Sort field (name, email, createdAt)
 *       - in: query
 *         name: order
 *         schema:
 *           type: string
 *           enum: [asc, desc]
 *         description: Sort order
 *     responses:
 *       200:
 *         description: Successfully retrieved users
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 count:
 *                   type: integer
 *                 total:
 *                   type: integer
 *                 totalPages:
 *                   type: integer
 *                 currentPage:
 *                   type: integer
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 *       400:
 *         description: Invalid query parameters
 *       500:
 *         description: Server error
 */
app.get('/api/v1/users', getUsers);
```

### POST Endpoint

```javascript
/**
 * @swagger
 * /api/v1/users:
 *   post:
 *     summary: Create a new user
 *     tags:
 *       - Users
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/UserInput'
 *           example:
 *             name: "রহিম"
 *             email: "rahim@example.com"
 *             password: "password123"
 *             role: "user"
 *     responses:
 *       201:
 *         description: User created successfully
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
 *                   $ref: '#/components/schemas/User'
 *       400:
 *         description: Validation error
 *       409:
 *         description: Email already exists
 */
app.post('/api/v1/users', createUser);
```

### GET by ID Endpoint

```javascript
/**
 * @swagger
 * /api/v1/users/{id}:
 *   get:
 *     summary: Get a user by ID
 *     tags:
 *       - Users
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *         description: User ID
 *     responses:
 *       200:
 *         description: User found
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   $ref: '#/components/schemas/User'
 *       404:
 *         description: User not found
 */
app.get('/api/v1/users/:id', getUserById);
```

### PUT Endpoint (Update)

```javascript
/**
 * @swagger
 * /api/v1/users/{id}:
 *   put:
 *     summary: Update a user (complete replacement)
 *     tags:
 *       - Users
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
 *             $ref: '#/components/schemas/UserInput'
 *     responses:
 *       200:
 *         description: User updated
 *       404:
 *         description: User not found
 */
app.put('/api/v1/users/:id', updateUser);
```

### DELETE Endpoint

```javascript
/**
 * @swagger
 * /api/v1/users/{id}:
 *   delete:
 *     summary: Delete a user
 *     tags:
 *       - Users
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
app.delete('/api/v1/users/:id', deleteUser);
```

## Component Schemas Define করুন

```javascript
/**
 * @swagger
 * components:
 *   schemas:
 *     User:
 *       type: object
 *       properties:
 *         _id:
 *           type: string
 *           description: Unique identifier
 *         name:
 *           type: string
 *           description: User full name
 *         email:
 *           type: string
 *           format: email
 *           description: User email address
 *         role:
 *           type: string
 *           enum: [user, admin, moderator]
 *           description: User role
 *         createdAt:
 *           type: string
 *           format: date-time
 *         updatedAt:
 *           type: string
 *           format: date-time
 *       required:
 *         - name
 *         - email
 *
 *     UserInput:
 *       type: object
 *       properties:
 *         name:
 *           type: string
 *           minLength: 2
 *           maxLength: 50
 *         email:
 *           type: string
 *           format: email
 *         password:
 *           type: string
 *           minLength: 6
 *         role:
 *           type: string
 *           enum: [user, admin, moderator]
 *       required:
 *         - name
 *         - email
 *         - password
 *
 *     Error:
 *       type: object
 *       properties:
 *         success:
 *           type: boolean
 *           example: false
 *         error:
 *           type: string
 *         code:
 *           type: string
 *       required:
 *         - success
 *         - error
 */
```

## Security Schemes

```javascript
/**
 * @swagger
 * components:
 *   securitySchemes:
 *     bearerAuth:
 *       type: http
 *       scheme: bearer
 *       bearerFormat: JWT
 *     apiKey:
 *       type: apiKey
 *       in: header
 *       name: X-API-Key
 */

/**
 * @swagger
 * /api/v1/admin/users:
 *   get:
 *     summary: Get all users (admin only)
 *     tags:
 *       - Admin
 *     security:
 *       - bearerAuth: []
 *     responses:
 *       200:
 *         description: Success
 *       401:
 *         description: Unauthorized
 */
app.get('/api/v1/admin/users', authenticateAdmin, getUsers);
```

## সম্পূর্ণ Example Server

```javascript
import express from 'express';
import swaggerJsdoc from 'swagger-jsdoc';
import swaggerUi from 'swagger-ui-express';

const app = express();

app.use(express.json());

// Swagger configuration
const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'User Management API',
      version: '1.0.0',
      description: 'Complete REST API with Swagger documentation',
      contact: {
        name: 'API Support',
        email: 'support@example.com'
      },
      license: {
        name: 'MIT'
      }
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development'
      }
    ],
    tags: [
      {
        name: 'Users',
        description: 'User management'
      }
    ]
  },
  apis: [__filename] // Current file
};

const swaggerSpec = swaggerJsdoc(options);

// Mount Swagger UI
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));

// Sample data
let users = [
  { id: 1, name: 'রহিম', email: 'rahim@example.com', role: 'admin' },
  { id: 2, name: 'করিম', email: 'karim@example.com', role: 'user' }
];

/**
 * @swagger
 * /api/v1/users:
 *   get:
 *     summary: Get all users
 *     tags:
 *       - Users
 *     responses:
 *       200:
 *         description: Users list
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 type: object
 */
app.get('/api/v1/users', (req, res) => {
  res.json(users);
});

/**
 * @swagger
 * /api/v1/users:
 *   post:
 *     summary: Create user
 *     tags:
 *       - Users
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
 */
app.post('/api/v1/users', (req, res) => {
  const newUser = { id: users.length + 1, ...req.body };
  users.push(newUser);
  res.status(201).json(newUser);
});

app.listen(3000, () => {
  console.log('Server: http://localhost:3000');
  console.log('API Docs: http://localhost:3000/api-docs');
});
```

## Swagger UI Customization

```javascript
// Custom CSS এবং options
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec, {
  customCss: '.swagger-ui .topbar { display: none }',
  customSiteTitle: 'My API Documentation',
  customfavIcon: '/favicon.ico',
  swaggerOptions: {
    persistAuthorization: true,
    displayOperationId: false,
    defaultModelsExpandDepth: 1,
    docExpansion: 'list'
  }
}));
```

## Best Practices

### ✅ 1. Clear Descriptions রাখুন

```javascript
/**
 * @swagger
 * /api/users:
 *   get:
 *     summary: Retrieve all users
 *     description: Get a paginated list of users with optional filtering and sorting
 */
```

### ✅ 2. Examples প্রদান করুন

```javascript
/**
 * @swagger
 * requestBody:
 *   content:
 *     application/json:
 *       example:
 *         name: "রহিম"
 *         email: "rahim@example.com"
 */
```

### ✅ 3. Error Responses Document করুন

```javascript
/**
 * @swagger
 * responses:
 *   400:
 *     description: Validation error
 *   404:
 *     description: Resource not found
 *   500:
 *     description: Server error
 */
```

### ✅ 4. Reusable Schemas

```javascript
/**
 * @swagger
 * components:
 *   schemas:
 *     User:
 *       type: object
 *       properties:
 *         id: { type: string }
 *         name: { type: string }
 */

// Use করুন:
// $ref: '#/components/schemas/User'
```

## HATEOAS (Optional but Recommended)

**HATEOAS (Hypermedia As The Engine Of Application State)** হলো REST এর একটি constraint যেখানে response এ links থাকে।

```javascript
app.get('/api/v1/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  
  res.json({
    data: user,
    links: {
      self: `/api/v1/users/${user.id}`,
      all: '/api/v1/users',
      update: {
        href: `/api/v1/users/${user.id}`,
        method: 'PUT'
      },
      delete: {
        href: `/api/v1/users/${user.id}`,
        method: 'DELETE'
      }
    }
  });
});

// Response:
{
  "data": { "id": 1, "name": "রহিম" },
  "links": {
    "self": "/api/v1/users/1",
    "all": "/api/v1/users",
    "update": { "href": "/api/v1/users/1", "method": "PUT" },
    "delete": { "href": "/api/v1/users/1", "method": "DELETE" }
  }
}
```

## API Testing with Swagger UI

1. `/api-docs` এ যান
2. Endpoint এ click করুন
3. "Try it out" বাটন click করুন
4. Parameters fill করুন
5. "Execute" click করুন
6. Response দেখবেন

## Postman/Swagger Export

Swagger spec থেকে Postman collection export করা যায়:

1. Swagger UI তে Export option
2. OpenAPI spec download করুন
3. Postman এ import করুন

## পরবর্তী পদক্ষেপ

Advanced API patterns এবং security features শিখব।

## রেফারেন্স

- **OpenAPI Specification**: https://spec.openapis.org/
- **Swagger/OpenAPI**: https://swagger.io/
- **swagger-jsdoc**: https://github.com/Surnet/swagger-jsdoc
- **swagger-ui-express**: https://github.com/scottie1984/swagger-ui-express

---

**পরবর্তী Chapter**: Advanced API Features এবং Security

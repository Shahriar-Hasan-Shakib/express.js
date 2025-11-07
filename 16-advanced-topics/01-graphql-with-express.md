# GraphQL with Express.js

## সংক্ষিপ্ত পরিচিতি (Brief Overview)

**GraphQL** হলো API এর জন্য একটি query language এবং runtime যা Facebook তৈরি করেছে। REST API এর alternative হিসেবে GraphQL ব্যবহার করে আপনি exactly যা চান তা request করতে পারেন - না বেশি, না কম।

### এই Lesson এ যা শিখবেন:
- ✅ GraphQL কী এবং কেন ব্যবহার করবেন
- ✅ GraphQL vs REST API comparison
- ✅ Apollo Server setup with Express (ES6)
- ✅ Schema এবং Type definitions
- ✅ Resolvers এবং Query/Mutation
- ✅ GraphQL authentication
- ✅ DataLoader for optimization
- ✅ Complete CRUD example

---

## GraphQL কী?

**GraphQL** হলো:
- একটি **query language** for APIs
- **Strongly typed** schema system
- **Single endpoint** (`/graphql`)
- Client নির্ধারণ করে কোন data চাই
- **No over-fetching or under-fetching**

### REST vs GraphQL

| Feature | REST | GraphQL |
|---------|------|---------|
| Endpoints | Multiple (`/users`, `/posts`) | Single (`/graphql`) |
| Data Fetching | Fixed structure | Flexible, client decides |
| Over-fetching | ✅ Yes (gets extra data) | ❌ No |
| Under-fetching | ✅ Yes (multiple requests) | ❌ No |
| Versioning | URL versioning (`/v1`, `/v2`) | Schema evolution |
| Learning Curve | Easy | Moderate |

---

## Installation এবং Setup

### Dependencies Install

```bash
# Apollo Server এবং GraphQL
npm install @apollo/server graphql

# Express integration
npm install express cors body-parser

# Additional (for database)
npm install mongoose
```

### `package.json` Configuration

```json
{
  "name": "graphql-express-api",
  "version": "1.0.0",
  "type": "module",
  "description": "GraphQL API with Express and Apollo Server",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "@apollo/server": "^4.9.5",
    "graphql": "^16.8.1",
    "express": "^5.1.0",
    "mongoose": "^8.0.3",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2"
  },
  "devDependencies": {
    "nodemon": "^3.1.0"
  }
}
```

---

## Basic GraphQL Server Setup (ES6)

### `server.js`:

```javascript
import express from 'express';
import { ApolloServer } from '@apollo/server';
import { expressMiddleware } from '@apollo/server/express4';
import cors from 'cors';
import bodyParser from 'body-parser';

// Type definitions (Schema)
const typeDefs = `#graphql
  type Query {
    hello: String
    greeting(name: String!): String
  }
`;

// Resolvers
const resolvers = {
  Query: {
    hello: () => 'হ্যালো GraphQL! 🚀',
    greeting: (parent, args) => `হ্যালো, ${args.name}!`
  }
};

// Apollo Server instance
const server = new ApolloServer({
  typeDefs,
  resolvers,
});

// Express app
const app = express();
const PORT = process.env.PORT || 4000;

// Start Apollo Server
await server.start();

// Middleware
app.use(cors());
app.use(bodyParser.json());

// GraphQL endpoint
app.use('/graphql', expressMiddleware(server));

// Root endpoint
app.get('/', (req, res) => {
  res.json({
    message: 'GraphQL Server Running',
    graphql: `http://localhost:${PORT}/graphql`
  });
});

// Start server
app.listen(PORT, () => {
  console.log(`🚀 Server ready at http://localhost:${PORT}`);
  console.log(`📊 GraphQL endpoint: http://localhost:${PORT}/graphql`);
});
```

### Testing the Server:

```bash
npm run dev
```

Browser এ যান: `http://localhost:4000/graphql`

Apollo Studio খুলবে যেখানে আপনি query test করতে পারবেন:

```graphql
# Simple query
query {
  hello
}

# Query with arguments
query {
  greeting(name: "রহিম")
}
```

---

## Complete CRUD Example with Database

### Project Structure:

```
graphql-api/
├── server.js
├── package.json
│
├── src/
│   ├── schema/
│   │   ├── typeDefs.js
│   │   └── resolvers.js
│   │
│   ├── models/
│   │   └── User.js
│   │
│   ├── datasources/
│   │   └── userAPI.js
│   │
│   └── utils/
│       └── auth.js
│
└── .env
```

### `src/models/User.js`:

```javascript
import mongoose from 'mongoose';

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  age: {
    type: Number,
    min: 0
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  },
  isActive: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true
});

const User = mongoose.model('User', userSchema);

export default User;
```

### `src/schema/typeDefs.js`:

```javascript
export const typeDefs = `#graphql
  # User Type
  type User {
    id: ID!
    name: String!
    email: String!
    age: Int
    role: String!
    isActive: Boolean!
    createdAt: String!
    updatedAt: String!
  }

  # Input Types
  input CreateUserInput {
    name: String!
    email: String!
    age: Int
    role: String
  }

  input UpdateUserInput {
    name: String
    email: String
    age: Int
    role: String
    isActive: Boolean
  }

  # Query Type
  type Query {
    # Get all users
    users(limit: Int, offset: Int): [User!]!
    
    # Get user by ID
    user(id: ID!): User
    
    # Search users by name
    searchUsers(name: String!): [User!]!
    
    # Get user count
    userCount: Int!
  }

  # Mutation Type
  type Mutation {
    # Create new user
    createUser(input: CreateUserInput!): User!
    
    # Update user
    updateUser(id: ID!, input: UpdateUserInput!): User
    
    # Delete user
    deleteUser(id: ID!): Boolean!
    
    # Toggle user status
    toggleUserStatus(id: ID!): User
  }

  # Subscription Type (for real-time updates)
  type Subscription {
    userCreated: User!
    userUpdated: User!
  }
`;
```

### `src/schema/resolvers.js`:

```javascript
import User from '../models/User.js';

export const resolvers = {
  Query: {
    // Get all users with pagination
    users: async (parent, args) => {
      const { limit = 10, offset = 0 } = args;
      
      try {
        const users = await User.find()
          .skip(offset)
          .limit(limit)
          .sort({ createdAt: -1 });
        
        return users;
      } catch (error) {
        throw new Error(`Error fetching users: ${error.message}`);
      }
    },

    // Get single user by ID
    user: async (parent, args) => {
      try {
        const user = await User.findById(args.id);
        
        if (!user) {
          throw new Error('User not found');
        }
        
        return user;
      } catch (error) {
        throw new Error(`Error fetching user: ${error.message}`);
      }
    },

    // Search users by name
    searchUsers: async (parent, args) => {
      try {
        const users = await User.find({
          name: { $regex: args.name, $options: 'i' }
        });
        
        return users;
      } catch (error) {
        throw new Error(`Error searching users: ${error.message}`);
      }
    },

    // Get total user count
    userCount: async () => {
      try {
        return await User.countDocuments();
      } catch (error) {
        throw new Error(`Error counting users: ${error.message}`);
      }
    }
  },

  Mutation: {
    // Create new user
    createUser: async (parent, args) => {
      try {
        const { input } = args;
        
        // Check if user already exists
        const existingUser = await User.findOne({ email: input.email });
        if (existingUser) {
          throw new Error('User with this email already exists');
        }
        
        // Create new user
        const user = new User(input);
        await user.save();
        
        return user;
      } catch (error) {
        throw new Error(`Error creating user: ${error.message}`);
      }
    },

    // Update user
    updateUser: async (parent, args) => {
      try {
        const { id, input } = args;
        
        const user = await User.findByIdAndUpdate(
          id,
          { $set: input },
          { new: true, runValidators: true }
        );
        
        if (!user) {
          throw new Error('User not found');
        }
        
        return user;
      } catch (error) {
        throw new Error(`Error updating user: ${error.message}`);
      }
    },

    // Delete user
    deleteUser: async (parent, args) => {
      try {
        const user = await User.findByIdAndDelete(args.id);
        
        if (!user) {
          throw new Error('User not found');
        }
        
        return true;
      } catch (error) {
        throw new Error(`Error deleting user: ${error.message}`);
      }
    },

    // Toggle user active status
    toggleUserStatus: async (parent, args) => {
      try {
        const user = await User.findById(args.id);
        
        if (!user) {
          throw new Error('User not found');
        }
        
        user.isActive = !user.isActive;
        await user.save();
        
        return user;
      } catch (error) {
        throw new Error(`Error toggling user status: ${error.message}`);
      }
    }
  },

  // Field resolvers (if needed for computed fields)
  User: {
    id: (parent) => parent._id.toString(),
    createdAt: (parent) => parent.createdAt.toISOString(),
    updatedAt: (parent) => parent.updatedAt.toISOString()
  }
};
```

### Complete `server.js` with Database:

```javascript
import express from 'express';
import { ApolloServer } from '@apollo/server';
import { expressMiddleware } from '@apollo/server/express4';
import mongoose from 'mongoose';
import cors from 'cors';
import bodyParser from 'body-parser';
import dotenv from 'dotenv';

import { typeDefs } from './src/schema/typeDefs.js';
import { resolvers } from './src/schema/resolvers.js';

// Load environment variables
dotenv.config();

// MongoDB connection
const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/graphql-api');
    console.log('✅ MongoDB connected');
  } catch (error) {
    console.error('❌ MongoDB connection error:', error);
    process.exit(1);
  }
};

// Apollo Server setup
const server = new ApolloServer({
  typeDefs,
  resolvers,
  formatError: (error) => {
    // Custom error formatting
    return {
      message: error.message,
      code: error.extensions?.code || 'INTERNAL_SERVER_ERROR',
      path: error.path
    };
  }
});

// Express app
const app = express();
const PORT = process.env.PORT || 4000;

// Connect to database
await connectDB();

// Start Apollo Server
await server.start();

// Middleware
app.use(cors());
app.use(bodyParser.json());

// GraphQL endpoint with context
app.use('/graphql', expressMiddleware(server, {
  context: async ({ req }) => {
    // Add context (auth, database, etc.)
    return {
      user: req.user || null,
      // Add dataloaders here if needed
    };
  }
}));

// Health check
app.get('/health', (req, res) => {
  res.json({
    status: 'OK',
    database: mongoose.connection.readyState === 1 ? 'Connected' : 'Disconnected',
    timestamp: new Date().toISOString()
  });
});

// Root endpoint
app.get('/', (req, res) => {
  res.json({
    message: 'GraphQL API Server',
    endpoints: {
      graphql: `http://localhost:${PORT}/graphql`,
      health: `http://localhost:${PORT}/health`
    }
  });
});

// Start server
app.listen(PORT, () => {
  console.log(`🚀 Server ready at http://localhost:${PORT}`);
  console.log(`📊 GraphQL endpoint: http://localhost:${PORT}/graphql`);
});
```

---

## GraphQL Queries এবং Mutations Examples

### 1. Create User:

```graphql
mutation CreateUser {
  createUser(input: {
    name: "রহিম আহমেদ"
    email: "rahim@example.com"
    age: 25
    role: "user"
  }) {
    id
    name
    email
    age
    role
    createdAt
  }
}
```

### 2. Get All Users:

```graphql
query GetUsers {
  users(limit: 10, offset: 0) {
    id
    name
    email
    role
    isActive
  }
}
```

### 3. Get Single User:

```graphql
query GetUser {
  user(id: "507f1f77bcf86cd799439011") {
    id
    name
    email
    age
    role
    createdAt
    updatedAt
  }
}
```

### 4. Update User:

```graphql
mutation UpdateUser {
  updateUser(
    id: "507f1f77bcf86cd799439011"
    input: {
      name: "রহিম আহমেদ খান"
      age: 26
    }
  ) {
    id
    name
    age
    updatedAt
  }
}
```

### 5. Delete User:

```graphql
mutation DeleteUser {
  deleteUser(id: "507f1f77bcf86cd799439011")
}
```

### 6. Search Users:

```graphql
query SearchUsers {
  searchUsers(name: "রহিম") {
    id
    name
    email
  }
}
```

### 7. Get User Count:

```graphql
query UserCount {
  userCount
}
```

---

## Authentication with GraphQL

### `src/utils/auth.js`:

```javascript
import jwt from 'jsonwebtoken';

const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';

export const generateToken = (user) => {
  return jwt.sign(
    { id: user._id, email: user.email, role: user.role },
    JWT_SECRET,
    { expiresIn: '24h' }
  );
};

export const verifyToken = (token) => {
  try {
    return jwt.verify(token, JWT_SECRET);
  } catch (error) {
    throw new Error('Invalid or expired token');
  }
};
```

### Add Authentication to Schema:

```javascript
// In typeDefs.js
export const typeDefs = `#graphql
  type AuthPayload {
    token: String!
    user: User!
  }

  type Mutation {
    # ... existing mutations
    
    login(email: String!, password: String!): AuthPayload!
    register(input: CreateUserInput!): AuthPayload!
  }
`;
```

### Add Auth Resolvers:

```javascript
// In resolvers.js
import { generateToken, verifyToken } from '../utils/auth.js';
import bcrypt from 'bcryptjs';

export const resolvers = {
  // ... existing resolvers
  
  Mutation: {
    // ... existing mutations
    
    login: async (parent, args) => {
      const { email, password } = args;
      
      // Find user
      const user = await User.findOne({ email });
      if (!user) {
        throw new Error('Invalid credentials');
      }
      
      // Verify password (assuming you have password field)
      const isValid = await bcrypt.compare(password, user.password);
      if (!isValid) {
        throw new Error('Invalid credentials');
      }
      
      // Generate token
      const token = generateToken(user);
      
      return { token, user };
    },
    
    register: async (parent, args) => {
      const { input } = args;
      
      // Check if user exists
      const existingUser = await User.findOne({ email: input.email });
      if (existingUser) {
        throw new Error('User already exists');
      }
      
      // Hash password
      const hashedPassword = await bcrypt.hash(input.password, 10);
      
      // Create user
      const user = await User.create({
        ...input,
        password: hashedPassword
      });
      
      // Generate token
      const token = generateToken(user);
      
      return { token, user };
    }
  }
};
```

### Protected Queries with Context:

```javascript
// In server.js - update context
app.use('/graphql', expressMiddleware(server, {
  context: async ({ req }) => {
    const token = req.headers.authorization?.replace('Bearer ', '');
    
    let user = null;
    if (token) {
      try {
        user = verifyToken(token);
      } catch (error) {
        console.error('Token verification failed:', error.message);
      }
    }
    
    return { user };
  }
}));
```

### Protected Resolver Example:

```javascript
// In resolvers.js
Query: {
  // Protected query
  me: async (parent, args, context) => {
    if (!context.user) {
      throw new Error('Not authenticated');
    }
    
    const user = await User.findById(context.user.id);
    return user;
  },
  
  // Admin only query
  allUsersAdmin: async (parent, args, context) => {
    if (!context.user || context.user.role !== 'admin') {
      throw new Error('Not authorized');
    }
    
    return await User.find();
  }
}
```

---

## DataLoader for N+1 Query Optimization

### Install DataLoader:

```bash
npm install dataloader
```

### `src/datasources/userDataLoader.js`:

```javascript
import DataLoader from 'dataloader';
import User from '../models/User.js';

export const createUserLoader = () => {
  return new DataLoader(async (userIds) => {
    const users = await User.find({ _id: { $in: userIds } });
    
    // Return users in the same order as requested IDs
    const userMap = {};
    users.forEach(user => {
      userMap[user._id.toString()] = user;
    });
    
    return userIds.map(id => userMap[id.toString()] || null);
  });
};
```

### Use DataLoader in Context:

```javascript
import { createUserLoader } from './src/datasources/userDataLoader.js';

app.use('/graphql', expressMiddleware(server, {
  context: async ({ req }) => {
    return {
      user: req.user,
      loaders: {
        userLoader: createUserLoader()
      }
    };
  }
}));
```

### Use in Resolvers:

```javascript
Query: {
  user: async (parent, args, context) => {
    return await context.loaders.userLoader.load(args.id);
  }
}
```

---

## ⚠️ Common Mistakes & Fixes

### ভুল ১: Query/Mutation এ type mismatch

```graphql
# ❌ ভুল - String পাঠাচ্ছেন কিন্তু Int expected
mutation {
  createUser(input: {
    age: "25"  # Should be number, not string
  })
}

# ✅ সঠিক
mutation {
  createUser(input: {
    age: 25
  })
}
```

### ভুল ২: Missing required fields

```graphql
# ❌ ভুল - name required কিন্তু missing
mutation {
  createUser(input: {
    email: "test@example.com"
  })
}

# ✅ সঠিক - All required fields provided
mutation {
  createUser(input: {
    name: "Test User"
    email: "test@example.com"
  })
}
```

### ভুল ৩: Not handling errors properly

```javascript
// ❌ ভুল - Generic error
const user = await User.findById(id);
return user; // Returns null if not found

// ✅ সঠিক - Throw descriptive error
const user = await User.findById(id);
if (!user) {
  throw new Error('User not found');
}
return user;
```

### ভুল ৪: Over-fetching with nested queries

```graphql
# ❌ ভুল - Fetching unnecessary data
query {
  users {
    id
    name
    email
    age
    role
    isActive
    createdAt
    updatedAt
    # ... all fields
  }
}

# ✅ সঠিক - শুধু প্রয়োজনীয় fields
query {
  users {
    id
    name
    email
  }
}
```

### ভুল ৫: Not using DataLoader (N+1 problem)

```javascript
// ❌ ভুল - N+1 queries
Post: {
  author: async (parent) => {
    return await User.findById(parent.authorId); // Called N times!
  }
}

// ✅ সঠিক - Use DataLoader
Post: {
  author: async (parent, args, context) => {
    return await context.loaders.userLoader.load(parent.authorId);
  }
}
```

### ভুল ৬: Exposing sensitive data

```javascript
// ❌ ভুল - Password field exposed
type User {
  id: ID!
  email: String!
  password: String!  # Never expose password!
}

// ✅ সঠিক - Exclude sensitive fields
type User {
  id: ID!
  email: String!
  # No password field
}
```

### ভুল ৭: Not validating input

```javascript
// ❌ ভুল - No validation
createUser: async (parent, args) => {
  return await User.create(args.input);
}

// ✅ সঠিক - Validate before creating
createUser: async (parent, args) => {
  const { input } = args;
  
  // Validate email format
  if (!/\S+@\S+\.\S+/.test(input.email)) {
    throw new Error('Invalid email format');
  }
  
  // Validate age
  if (input.age && (input.age < 0 || input.age > 150)) {
    throw new Error('Invalid age');
  }
  
  return await User.create(input);
}
```

### ভুল ৮: Not handling authentication in context

```javascript
// ❌ ভুল - Checking auth in every resolver
Query: {
  user: async (parent, args) => {
    const token = /* get from somewhere */;
    if (!token) throw new Error('Not authenticated');
    // ... rest of code
  }
}

// ✅ সঠিক - Handle in context
// In server.js
context: async ({ req }) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  const user = token ? verifyToken(token) : null;
  return { user };
}

// In resolver
Query: {
  user: async (parent, args, context) => {
    if (!context.user) throw new Error('Not authenticated');
    // ... rest of code
  }
}
```

### ভুল ৯: Mutation without returning data

```graphql
# ❌ ভুল - Mutation returns nothing
type Mutation {
  createUser(input: CreateUserInput!): Boolean
}

# ✅ সঠিক - Return created object
type Mutation {
  createUser(input: CreateUserInput!): User!
}
```

### ভুল ১০: Not using variables in queries

```graphql
# ❌ ভুল - Hardcoded values
mutation {
  createUser(input: {
    name: "Test User"
    email: "test@example.com"
  })
}

# ✅ সঠিক - Use variables
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
    name
  }
}

# Variables (separate JSON):
{
  "input": {
    "name": "Test User",
    "email": "test@example.com"
  }
}
```

---

## GraphQL Best Practices

1. **Use Input Types** for mutations
2. **Implement DataLoader** to avoid N+1 queries
3. **Paginate** large data sets
4. **Handle errors** gracefully
5. **Validate input** before processing
6. **Use fragments** for reusable query parts
7. **Implement rate limiting**
8. **Cache responses** when possible
9. **Document your schema** with descriptions
10. **Version your schema** carefully

---

## রেফারেন্স এবং দরকারী লিংক

- **GraphQL Official**: https://graphql.org/
- **Apollo Server Docs**: https://www.apollographql.com/docs/apollo-server/
- **GraphQL Best Practices**: https://graphql.org/learn/best-practices/
- **DataLoader GitHub**: https://github.com/graphql/dataloader
- **GraphQL Playground**: https://github.com/graphql/graphql-playground
- **Apollo Client**: https://www.apollographql.com/docs/react/

---

**পরবর্তী Lesson**: TypeScript with Express.js

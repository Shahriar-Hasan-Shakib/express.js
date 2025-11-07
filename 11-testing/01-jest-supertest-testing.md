# Testing Express.js Applications

## সংক্ষিপ্ত পরিচিতি

Testing আপনার application এর reliability এবং maintainability নিশ্চিত করে।

## Testing Tools

```bash
npm install --save-dev jest supertest @types/jest
# অথবা
npm install --save-dev mocha chai supertest
```

## Jest + Supertest Setup

### package.json:
```json
{
  "scripts": {
    "test": "jest --watchAll --verbose",
    "test:ci": "jest --coverage"
  },
  "jest": {
    "testEnvironment": "node",
    "coveragePathIgnorePatterns": ["/node_modules/"]
  }
}
```

### Basic Test Example:

```javascript
// app.js
const express = require('express');
const app = express();

app.use(express.json());

app.get('/', (req, res) => {
  res.json({ message: 'Hello World' });
});

app.get('/users', (req, res) => {
  res.json({ users: ['রহিম', 'করিম'] });
});

module.exports = app;
```

```javascript
// app.test.js
const request = require('supertest');
const app = require('./app');

describe('GET /', () => {
  it('should return hello message', async () => {
    const response = await request(app).get('/');
    
    expect(response.status).toBe(200);
    expect(response.body.message).toBe('Hello World');
  });
});

describe('GET /users', () => {
  it('should return users list', async () => {
    const response = await request(app).get('/users');
    
    expect(response.status).toBe(200);
    expect(response.body.users).toHaveLength(2);
  });
});
```

## CRUD API Testing

```javascript
// tests/users.test.js
const request = require('supertest');
const app = require('../app');
const User = require('../models/User');

beforeAll(async () => {
  // Connect to test database
  await mongoose.connect(process.env.TEST_DATABASE_URL);
});

afterAll(async () => {
  await mongoose.connection.close();
});

beforeEach(async () => {
  // Clear database before each test
  await User.deleteMany({});
});

describe('User API', () => {
  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const userData = {
        name: 'রহিম',
        email: 'rahim@example.com',
        password: 'password123'
      };
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);
      
      expect(response.body.success).toBe(true);
      expect(response.body.data.name).toBe(userData.name);
      expect(response.body.data.email).toBe(userData.email);
      expect(response.body.data.password).toBeUndefined(); // Password should not be returned
    });
    
    it('should return 400 if email is missing', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({ name: 'রহিম', password: 'password123' })
        .expect(400);
      
      expect(response.body.success).toBe(false);
    });
    
    it('should return 409 if email already exists', async () => {
      const userData = {
        name: 'রহিম',
        email: 'rahim@example.com',
        password: 'password123'
      };
      
      await User.create(userData);
      
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(409);
    });
  });
  
  describe('GET /api/users', () => {
    it('should return all users', async () => {
      await User.create([
        { name: 'রহিম', email: 'rahim@example.com', password: 'pass' },
        { name: 'করিম', email: 'karim@example.com', password: 'pass' }
      ]);
      
      const response = await request(app)
        .get('/api/users')
        .expect(200);
      
      expect(response.body.data).toHaveLength(2);
    });
  });
  
  describe('GET /api/users/:id', () => {
    it('should return a single user', async () => {
      const user = await User.create({
        name: 'রহিম',
        email: 'rahim@example.com',
        password: 'pass'
      });
      
      const response = await request(app)
        .get(`/api/users/${user._id}`)
        .expect(200);
      
      expect(response.body.data.name).toBe('রহিম');
    });
    
    it('should return 404 if user not found', async () => {
      const fakeId = '507f1f77bcf86cd799439011';
      
      await request(app)
        .get(`/api/users/${fakeId}`)
        .expect(404);
    });
  });
  
  describe('PUT /api/users/:id', () => {
    it('should update a user', async () => {
      const user = await User.create({
        name: 'রহিম',
        email: 'rahim@example.com',
        password: 'pass'
      });
      
      const response = await request(app)
        .put(`/api/users/${user._id}`)
        .send({ name: 'Updated Name' })
        .expect(200);
      
      expect(response.body.data.name).toBe('Updated Name');
    });
  });
  
  describe('DELETE /api/users/:id', () => {
    it('should delete a user', async () => {
      const user = await User.create({
        name: 'রহিম',
        email: 'rahim@example.com',
        password: 'pass'
      });
      
      await request(app)
        .delete(`/api/users/${user._id}`)
        .expect(200);
      
      const deletedUser = await User.findById(user._id);
      expect(deletedUser).toBeNull();
    });
  });
});
```

## Authentication Testing

```javascript
describe('Authentication', () => {
  describe('POST /api/auth/register', () => {
    it('should register a new user', async () => {
      const response = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'রহিম',
          email: 'rahim@example.com',
          password: 'password123'
        })
        .expect(201);
      
      expect(response.body.token).toBeDefined();
    });
  });
  
  describe('POST /api/auth/login', () => {
    it('should login with valid credentials', async () => {
      // Create user first
      await request(app)
        .post('/api/auth/register')
        .send({
          name: 'রহিম',
          email: 'rahim@example.com',
          password: 'password123'
        });
      
      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'rahim@example.com',
          password: 'password123'
        })
        .expect(200);
      
      expect(response.body.token).toBeDefined();
    });
    
    it('should fail with invalid credentials', async () => {
      await request(app)
        .post('/api/auth/login')
        .send({
          email: 'wrong@example.com',
          password: 'wrongpass'
        })
        .expect(401);
    });
  });
  
  describe('Protected Routes', () => {
    let token;
    
    beforeEach(async () => {
      // Register and get token
      const response = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'রহিম',
          email: 'rahim@example.com',
          password: 'password123'
        });
      
      token = response.body.token;
    });
    
    it('should access protected route with valid token', async () => {
      const response = await request(app)
        .get('/api/profile')
        .set('Authorization', `Bearer ${token}`)
        .expect(200);
      
      expect(response.body.user).toBeDefined();
    });
    
    it('should deny access without token', async () => {
      await request(app)
        .get('/api/profile')
        .expect(401);
    });
    
    it('should deny access with invalid token', async () => {
      await request(app)
        .get('/api/profile')
        .set('Authorization', 'Bearer invalid-token')
        .expect(403);
    });
  });
});
```

## Mocking

```javascript
jest.mock('../models/User');

const User = require('../models/User');

describe('User Controller', () => {
  it('should return users', async () => {
    const mockUsers = [
      { name: 'রহিম', email: 'rahim@example.com' }
    ];
    
    User.find.mockResolvedValue(mockUsers);
    
    const response = await request(app).get('/api/users');
    
    expect(response.body.data).toEqual(mockUsers);
    expect(User.find).toHaveBeenCalledTimes(1);
  });
});
```

## Integration Testing

```javascript
describe('Full User Flow', () => {
  it('should register, login, and access profile', async () => {
    // 1. Register
    const registerResponse = await request(app)
      .post('/api/auth/register')
      .send({
        name: 'রহিম',
        email: 'rahim@example.com',
        password: 'password123'
      });
    
    expect(registerResponse.status).toBe(201);
    const token = registerResponse.body.token;
    
    // 2. Login
    const loginResponse = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'rahim@example.com',
        password: 'password123'
      });
    
    expect(loginResponse.status).toBe(200);
    
    // 3. Access profile
    const profileResponse = await request(app)
      .get('/api/profile')
      .set('Authorization', `Bearer ${token}`);
    
    expect(profileResponse.status).toBe(200);
    expect(profileResponse.body.user.email).toBe('rahim@example.com');
  });
});
```

## Test Coverage

```bash
npm test -- --coverage
```

## Best Practices

### ✅ ১. Test Isolation
```javascript
beforeEach(async () => {
  await clearDatabase();
});
```

### ✅ ২. Use Test Database
```javascript
const testDB = process.env.TEST_DATABASE_URL;
```

### ✅ ৩. Descriptive Test Names
```javascript
it('should return 400 when email is missing', async () => {});
```

### ✅ ৪. Test Edge Cases
```javascript
it('should handle empty array', async () => {});
it('should handle null values', async () => {});
```

## রেফারেন্স

- **Jest**: https://jestjs.io/
- **Supertest**: https://www.npmjs.com/package/supertest
- **Mocha**: https://mochajs.org/

---

**পরবর্তী Chapter**: Performance & Optimization

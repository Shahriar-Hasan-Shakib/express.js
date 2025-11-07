# Database Integration - MongoDB, PostgreSQL, MySQL

## সংক্ষিপ্ত পরিচিতি

Express.js এর সাথে যেকোনো database ব্যবহার করা যায়। এই lesson এ আমরা popular databases integrate করা শিখব।

## MongoDB with Mongoose

### Installation:
```bash
npm install mongoose
```

### Connection:

```javascript
const express = require('express');
const mongoose = require('mongoose');
const app = express();

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true
})
.then(() => console.log('✅ MongoDB connected'))
.catch(err => console.error('❌ MongoDB connection error:', err));

app.listen(3000);
```

### Model Definition:

```javascript
// models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name প্রয়োজন'],
    trim: true
  },
  email: {
    type: String,
    required: [true, 'Email প্রয়োজন'],
    unique: true,
    lowercase: true,
    match: [/^\S+@\S+\.\S+$/, 'Valid email প্রয়োজন']
  },
  age: {
    type: Number,
    min: [0, 'Age 0 এর কম হতে পারে না'],
    max: [150, 'Age 150 এর বেশি হতে পারে না']
  },
  password: {
    type: String,
    required: true,
    minlength: 6
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
  timestamps: true // createdAt, updatedAt automatically
});

module.exports = mongoose.model('User', userSchema);
```

### CRUD Operations:

```javascript
const User = require('./models/User');

// CREATE
app.post('/users', async (req, res) => {
  try {
    const user = await User.create(req.body);
    res.status(201).json({ success: true, data: user });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
});

// READ ALL
app.get('/users', async (req, res) => {
  try {
    const users = await User.find()
      .select('-password') // password বাদ দিন
      .sort('-createdAt')  // নতুন আগে
      .limit(10);
    
    res.json({ success: true, count: users.length, data: users });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// READ ONE
app.get('/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id).select('-password');
    
    if (!user) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }
    
    res.json({ success: true, data: user });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// UPDATE
app.put('/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true } // নতুন data return এবং validation চালান
    );
    
    if (!user) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }
    
    res.json({ success: true, data: user });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
});

// DELETE
app.delete('/users/:id', async (req, res) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);
    
    if (!user) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }
    
    res.json({ success: true, message: 'User deleted' });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});
```

### Advanced Queries:

```javascript
// Filtering
app.get('/users/filter', async (req, res) => {
  const { minAge, role, search } = req.query;
  
  const query = {};
  
  if (minAge) query.age = { $gte: parseInt(minAge) };
  if (role) query.role = role;
  if (search) query.name = { $regex: search, $options: 'i' };
  
  const users = await User.find(query);
  res.json(users);
});

// Pagination
app.get('/users/paginate', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;
  
  const users = await User.find()
    .skip(skip)
    .limit(limit);
  
  const total = await User.countDocuments();
  
  res.json({
    page,
    limit,
    total,
    totalPages: Math.ceil(total / limit),
    data: users
  });
});
```

## PostgreSQL with pg (node-postgres)

### Installation:
```bash
npm install pg
```

### Connection:

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  user: 'postgres',
  host: 'localhost',
  database: 'myapp',
  password: 'password',
  port: 5432,
});

// Test connection
pool.query('SELECT NOW()', (err, res) => {
  if (err) {
    console.error('Database connection error:', err);
  } else {
    console.log('✅ PostgreSQL connected');
  }
});
```

### Create Table:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  age INTEGER CHECK (age >= 0 AND age <= 150),
  password VARCHAR(255) NOT NULL,
  role VARCHAR(20) DEFAULT 'user',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### CRUD Operations:

```javascript
// CREATE
app.post('/users', async (req, res) => {
  const { name, email, age, password } = req.body;
  
  try {
    const result = await pool.query(
      'INSERT INTO users (name, email, age, password) VALUES ($1, $2, $3, $4) RETURNING *',
      [name, email, age, password]
    );
    
    res.status(201).json({ success: true, data: result.rows[0] });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
});

// READ ALL
app.get('/users', async (req, res) => {
  try {
    const result = await pool.query(
      'SELECT id, name, email, age, role, created_at FROM users ORDER BY created_at DESC'
    );
    
    res.json({ success: true, count: result.rowCount, data: result.rows });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// READ ONE
app.get('/users/:id', async (req, res) => {
  try {
    const result = await pool.query(
      'SELECT id, name, email, age, role FROM users WHERE id = $1',
      [req.params.id]
    );
    
    if (result.rowCount === 0) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }
    
    res.json({ success: true, data: result.rows[0] });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// UPDATE
app.put('/users/:id', async (req, res) => {
  const { name, email, age } = req.body;
  
  try {
    const result = await pool.query(
      'UPDATE users SET name = $1, email = $2, age = $3, updated_at = CURRENT_TIMESTAMP WHERE id = $4 RETURNING *',
      [name, email, age, req.params.id]
    );
    
    if (result.rowCount === 0) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }
    
    res.json({ success: true, data: result.rows[0] });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
});

// DELETE
app.delete('/users/:id', async (req, res) => {
  try {
    const result = await pool.query(
      'DELETE FROM users WHERE id = $1',
      [req.params.id]
    );
    
    if (result.rowCount === 0) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }
    
    res.json({ success: true, message: 'User deleted' });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});
```

## MySQL with mysql2

### Installation:
```bash
npm install mysql2
```

### Connection:

```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: 'localhost',
  user: 'root',
  password: 'password',
  database: 'myapp',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

// Test connection
pool.query('SELECT 1')
  .then(() => console.log('✅ MySQL connected'))
  .catch(err => console.error('❌ MySQL connection error:', err));
```

### CRUD Operations:

```javascript
// CREATE
app.post('/users', async (req, res) => {
  const { name, email, age, password } = req.body;
  
  try {
    const [result] = await pool.query(
      'INSERT INTO users (name, email, age, password) VALUES (?, ?, ?, ?)',
      [name, email, age, password]
    );
    
    const [user] = await pool.query('SELECT * FROM users WHERE id = ?', [result.insertId]);
    
    res.status(201).json({ success: true, data: user[0] });
  } catch (error) {
    res.status(400).json({ success: false, error: error.message });
  }
});

// READ ALL
app.get('/users', async (req, res) => {
  try {
    const [users] = await pool.query(
      'SELECT id, name, email, age, role FROM users ORDER BY created_at DESC'
    );
    
    res.json({ success: true, count: users.length, data: users });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// READ ONE
app.get('/users/:id', async (req, res) => {
  try {
    const [users] = await pool.query(
      'SELECT id, name, email, age, role FROM users WHERE id = ?',
      [req.params.id]
    );
    
    if (users.length === 0) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }
    
    res.json({ success: true, data: users[0] });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});
```

## Database Connection Best Practices

### Environment Variables:

```javascript
require('dotenv').config();

const dbConfig = {
  host: process.env.DB_HOST || 'localhost',
  user: process.env.DB_USER || 'root',
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  port: process.env.DB_PORT || 5432
};
```

### Connection Pool:

```javascript
// Always use connection pooling for better performance
const pool = mysql.createPool({
  // ... config
  connectionLimit: 10 // Adjust based on your needs
});
```

### Error Handling:

```javascript
// Graceful shutdown
process.on('SIGINT', async () => {
  await pool.end();
  process.exit(0);
});
```

## ORM/Query Builders

### Sequelize (for SQL databases):

```bash
npm install sequelize mysql2
# or
npm install sequelize pg pg-hstore
```

```javascript
const { Sequelize, DataTypes } = require('sequelize');

const sequelize = new Sequelize('myapp', 'root', 'password', {
  host: 'localhost',
  dialect: 'mysql' // or 'postgres', 'sqlite', 'mariadb', 'mssql'
});

// Define model
const User = sequelize.define('User', {
  name: {
    type: DataTypes.STRING,
    allowNull: false
  },
  email: {
    type: DataTypes.STRING,
    unique: true,
    allowNull: false
  },
  age: {
    type: DataTypes.INTEGER
  }
});

// Sync models
await sequelize.sync();

// Use it
app.post('/users', async (req, res) => {
  const user = await User.create(req.body);
  res.json(user);
});
```

### Prisma (Modern ORM):

```bash
npm install prisma @prisma/client
npx prisma init
```

```prisma
// schema.prisma
model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  age       Int?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

```javascript
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

app.post('/users', async (req, res) => {
  const user = await prisma.user.create({
    data: req.body
  });
  res.json(user);
});
```

## রেফারেন্স

- **Mongoose**: https://mongoosejs.com/
- **node-postgres**: https://node-postgres.com/
- **mysql2**: https://www.npmjs.com/package/mysql2
- **Sequelize**: https://sequelize.org/
- **Prisma**: https://www.prisma.io/

---

**পরবর্তী Chapter**: Authentication & Authorization

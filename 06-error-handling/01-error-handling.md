# Error Handling in Express.js

## সংক্ষিপ্ত পরিচিতি

Proper error handling আপনার application কে stable এবং user-friendly করে তোলে।

## Basic Error Handling

### Synchronous Errors

```javascript
app.get('/user/:id', (req, res, next) => {
  const user = getUserById(req.params.id);
  
  if (!user) {
    const error = new Error('User not found');
    error.status = 404;
    throw error; // Synchronous error - Express catch করবে
  }
  
  res.json(user);
});
```

### Asynchronous Errors

```javascript
// ❌ ভুল - async errors automatically caught হয় না
app.get('/users', async (req, res) => {
  const users = await User.find(); // Error হলে app crash!
  res.json(users);
});

// ✅ সঠিক - try-catch ব্যবহার করুন
app.get('/users', async (req, res, next) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (error) {
    next(error); // Error handler এ পাঠান
  }
});
```

## Error Handling Middleware

### Basic Error Handler

```javascript
// সব routes এর পরে
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  res.status(err.status || 500).json({
    success: false,
    error: err.message || 'Server Error'
  });
});
```

### Advanced Error Handler

```javascript
const errorHandler = (err, req, res, next) => {
  // Log error
  console.error('Error:', {
    message: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method
  });
  
  // Set status code
  const statusCode = err.statusCode || err.status || 500;
  
  // Development vs Production
  const response = {
    success: false,
    error: err.message,
    ...(process.env.NODE_ENV === 'development' && {
      stack: err.stack,
      details: err
    })
  };
  
  res.status(statusCode).json(response);
};

app.use(errorHandler);
```

## Custom Error Classes

```javascript
// errors/AppError.js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404);
  }
}

class ValidationError extends AppError {
  constructor(message = 'Validation failed') {
    super(message, 400);
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401);
  }
}

module.exports = { AppError, NotFoundError, ValidationError, UnauthorizedError };
```

### Usage:

```javascript
const { NotFoundError, ValidationError } = require('./errors/AppError');

app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    
    if (!user) {
      throw new NotFoundError('User not found');
    }
    
    res.json(user);
  } catch (error) {
    next(error);
  }
});

app.post('/users', async (req, res, next) => {
  try {
    if (!req.body.email) {
      throw new ValidationError('Email প্রয়োজন');
    }
    
    const user = await User.create(req.body);
    res.status(201).json(user);
  } catch (error) {
    next(error);
  }
});
```

## Async Handler Wrapper

```javascript
// utils/asyncHandler.js
const asyncHandler = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

module.exports = asyncHandler;
```

### Usage:

```javascript
const asyncHandler = require('./utils/asyncHandler');

// আগে:
app.get('/users', async (req, res, next) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (error) {
    next(error);
  }
});

// পরে (cleaner):
app.get('/users', asyncHandler(async (req, res) => {
  const users = await User.find();
  res.json(users);
}));
```

## 404 Not Found Handler

```javascript
// সব routes এর পরে, error handler এর আগে
app.use((req, res, next) => {
  res.status(404).json({
    success: false,
    error: 'Route পাওয়া যায়নি',
    path: req.originalUrl
  });
});
```

## Validation Errors

```javascript
const { validationResult } = require('express-validator');

const validate = (req, res, next) => {
  const errors = validationResult(req);
  
  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      errors: errors.array().map(err => ({
        field: err.path,
        message: err.msg
      }))
    });
  }
  
  next();
};

app.post('/users',
  body('email').isEmail(),
  body('password').isLength({ min: 6 }),
  validate,
  asyncHandler(async (req, res) => {
    const user = await User.create(req.body);
    res.status(201).json(user);
  })
);
```

## Database Errors

```javascript
const handleDatabaseError = (err) => {
  // Mongoose duplicate key error
  if (err.code === 11000) {
    const field = Object.keys(err.keyPattern)[0];
    return new AppError(`${field} already exists`, 409);
  }
  
  // Mongoose validation error
  if (err.name === 'ValidationError') {
    const messages = Object.values(err.errors).map(e => e.message);
    return new AppError(messages.join(', '), 400);
  }
  
  // Mongoose cast error (invalid ID)
  if (err.name === 'CastError') {
    return new AppError('Invalid ID format', 400);
  }
  
  return err;
};

// Error handler middleware
app.use((err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;
  
  // Handle specific errors
  error = handleDatabaseError(error);
  
  res.status(error.statusCode || 500).json({
    success: false,
    error: error.message || 'Server Error'
  });
});
```

## সম্পূর্ণ উদাহরণ

```javascript
const express = require('express');
const asyncHandler = require('./utils/asyncHandler');
const { NotFoundError, ValidationError } = require('./errors/AppError');
const app = express();

app.use(express.json());

// Routes
app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  
  if (!user) {
    throw new NotFoundError('User not found');
  }
  
  res.json(user);
}));

app.post('/users', asyncHandler(async (req, res) => {
  if (!req.body.email) {
    throw new ValidationError('Email required');
  }
  
  const user = await User.create(req.body);
  res.status(201).json(user);
}));

// 404 Handler
app.use((req, res, next) => {
  res.status(404).json({
    success: false,
    error: 'Route not found'
  });
});

// Error Handler
app.use((err, req, res, next) => {
  console.error(err);
  
  const statusCode = err.statusCode || 500;
  const message = err.isOperational ? err.message : 'Server Error';
  
  res.status(statusCode).json({
    success: false,
    error: message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
});

app.listen(3000);
```

## Best Practices

### ✅ ১. Always use try-catch for async code
### ✅ ২. Use custom error classes
### ✅ ৩. Log errors properly
### ✅ ৪. Don't expose sensitive info in production
### ✅ ৫. Use error monitoring tools (Sentry, etc.)

## রেফারেন্স

- **Express Error Handling**: https://expressjs.com/en/guide/error-handling.html

---

**পরবর্তী Chapter**: Database Integration

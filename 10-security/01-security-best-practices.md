# Security Best Practices

## সংক্ষিপ্ত পরিচিতি

Web application security অত্যন্ত গুরুত্বপূর্ণ। এই chapter এ আমরা Express.js application secure করার best practices শিখব।

## Essential Security Packages

```bash
npm install helmet cors express-rate-limit express-validator hpp xss-clean express-mongo-sanitize
```

## 1. Helmet - Security Headers

```javascript
const helmet = require('helmet');

// Basic usage
app.use(helmet());

// Custom configuration
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'", "fonts.googleapis.com"],
      scriptSrc: ["'self'", "trusted-cdn.com"],
      imgSrc: ["'self'", "data:", "https:"],
      fontSrc: ["'self'", "fonts.gstatic.com"]
    }
  },
  hsts: {
    maxAge: 31536000, // 1 year
    includeSubDomains: true,
    preload: true
  }
}));

// Individual middlewares
app.use(helmet.noSniff());
app.use(helmet.frameguard({ action: 'deny' }));
app.use(helmet.xssFilter());
app.use(helmet.hidePoweredBy());
```

## 2. CORS Configuration

```javascript
const cors = require('cors');

// Specific origins only
const corsOptions = {
  origin: function (origin, callback) {
    const whitelist = [
      'https://example.com',
      'https://app.example.com'
    ];
    
    if (!origin || whitelist.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  maxAge: 86400 // 24 hours
};

app.use(cors(corsOptions));
```

## 3. Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

// Global rate limiter
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests',
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/', globalLimiter);

// Strict limiter for auth routes
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  message: 'Too many login attempts'
});

app.post('/api/login', authLimiter, loginHandler);
```

## 4. Input Validation & Sanitization

```javascript
const { body, validationResult } = require('express-validator');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');

// Prevent MongoDB injection
app.use(mongoSanitize());

// Prevent XSS attacks
app.use(xss());

// Prevent HTTP Parameter Pollution
app.use(hpp({
  whitelist: ['sort', 'filter'] // Allow these params to be repeated
}));

// Validation example
app.post('/api/users',
  body('email')
    .isEmail()
    .normalizeEmail()
    .trim()
    .escape(),
  body('name')
    .isLength({ min: 3, max: 50 })
    .trim()
    .escape(),
  body('password')
    .isLength({ min: 8 })
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/),
  async (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // Process request
  }
);
```

## 5. SQL Injection Prevention

```javascript
// ❌ ভুল - SQL Injection vulnerable
app.get('/users', async (req, res) => {
  const query = `SELECT * FROM users WHERE email = '${req.query.email}'`;
  // Dangerous!
});

// ✅ সঠিক - Parameterized queries
app.get('/users', async (req, res) => {
  const query = 'SELECT * FROM users WHERE email = $1';
  const result = await pool.query(query, [req.query.email]);
  res.json(result.rows);
});

// With MongoDB (Mongoose automatically protects)
const user = await User.findOne({ email: req.query.email });
```

## 6. Password Security

```javascript
const bcrypt = require('bcryptjs');

// Hash password
const hashPassword = async (password) => {
  const salt = await bcrypt.genSalt(10);
  return await bcrypt.hash(password, salt);
};

// Verify password
const verifyPassword = async (password, hashedPassword) => {
  return await bcrypt.compare(password, hashedPassword);
};

// Registration
app.post('/register', async (req, res) => {
  const { email, password } = req.body;
  
  // Password requirements
  if (password.length < 8) {
    return res.status(400).json({ error: 'Password must be at least 8 characters' });
  }
  
  const hashedPassword = await hashPassword(password);
  
  const user = await User.create({
    email,
    password: hashedPassword
  });
  
  res.json({ success: true });
});
```

## 7. JWT Security

```javascript
const jwt = require('jsonwebtoken');

// Strong secret
const JWT_SECRET = process.env.JWT_SECRET; // min 256 bits

// Generate token
const generateToken = (payload) => {
  return jwt.sign(payload, JWT_SECRET, {
    expiresIn: '15m', // Short-lived
    issuer: 'my-app',
    audience: 'my-app-users'
  });
};

// Verify token
const verifyToken = (req, res, next) => {
  const token = req.headers['authorization']?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'Token required' });
  }
  
  try {
    const decoded = jwt.verify(token, JWT_SECRET, {
      issuer: 'my-app',
      audience: 'my-app-users'
    });
    req.user = decoded;
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    return res.status(403).json({ error: 'Invalid token' });
  }
};
```

## 8. HTTPS Enforcement

```javascript
// Redirect HTTP to HTTPS
app.use((req, res, next) => {
  if (process.env.NODE_ENV === 'production' && !req.secure) {
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});

// Or use helmet's HSTS
app.use(helmet.hsts({
  maxAge: 31536000,
  includeSubDomains: true,
  preload: true
}));
```

## 9. Session Security

```javascript
const session = require('express-session');
const MongoStore = require('connect-mongo');

app.use(session({
  secret: process.env.SESSION_SECRET, // Strong random string
  resave: false,
  saveUninitialized: false,
  store: MongoStore.create({
    mongoUrl: process.env.MONGODB_URI,
    touchAfter: 24 * 3600 // Lazy update
  }),
  cookie: {
    secure: process.env.NODE_ENV === 'production', // HTTPS only
    httpOnly: true, // Prevent XSS
    maxAge: 1000 * 60 * 60 * 24, // 24 hours
    sameSite: 'strict' // CSRF protection
  },
  name: 'sessionId' // Don't use default 'connect.sid'
}));
```

## 10. CSRF Protection

```javascript
const csrf = require('csurf');

const csrfProtection = csrf({ cookie: true });

app.use(csrfProtection);

// Send CSRF token
app.get('/form', (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
});

// Verify CSRF token
app.post('/process', csrfProtection, (req, res) => {
  res.send('Data processed');
});
```

## 11. File Upload Security

```javascript
const multer = require('multer');
const path = require('path');

const fileFilter = (req, file, cb) => {
  // Whitelist approach
  const allowedMimes = ['image/jpeg', 'image/png', 'application/pdf'];
  
  if (allowedMimes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type'), false);
  }
};

const upload = multer({
  storage: multer.diskStorage({
    destination: 'uploads/',
    filename: (req, file, cb) => {
      // Generate secure random filename
      const uniqueName = crypto.randomBytes(16).toString('hex');
      const ext = path.extname(file.originalname);
      cb(null, uniqueName + ext);
    }
  }),
  fileFilter: fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB
    files: 1
  }
});
```

## 12. Error Handling

```javascript
// Development vs Production errors
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  const statusCode = err.statusCode || 500;
  
  if (process.env.NODE_ENV === 'production') {
    // Don't expose internal errors
    res.status(statusCode).json({
      success: false,
      error: err.isOperational ? err.message : 'Internal server error'
    });
  } else {
    // Development - show full error
    res.status(statusCode).json({
      success: false,
      error: err.message,
      stack: err.stack
    });
  }
});
```

## 13. Environment Variables

```javascript
// ❌ Never hardcode secrets
const API_KEY = 'abc123';

// ✅ Use environment variables
require('dotenv').config();
const API_KEY = process.env.API_KEY;

// Validate critical env vars
const requiredEnv = ['DATABASE_URL', 'JWT_SECRET', 'SESSION_SECRET'];
requiredEnv.forEach(key => {
  if (!process.env[key]) {
    throw new Error(`Missing required environment variable: ${key}`);
  }
});
```

## 14. Logging & Monitoring

```javascript
const morgan = require('morgan');
const winston = require('winston');

// Request logging
app.use(morgan('combined'));

// Application logging
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Log security events
app.post('/login', async (req, res) => {
  try {
    const user = await authenticate(req.body);
    logger.info(`Login successful: ${user.email}`);
    res.json({ success: true });
  } catch (error) {
    logger.warn(`Failed login attempt: ${req.body.email}`);
    res.status(401).json({ error: 'Invalid credentials' });
  }
});
```

## Complete Secure Application Example

```javascript
require('dotenv').config();
const express = require('express');
const helmet = require('helmet');
const cors = require('cors');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');

const app = express();

// Security headers
app.use(helmet());

// CORS
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(','),
  credentials: true
}));

// Body parsing
app.use(express.json({ limit: '10kb' })); // Limit payload size
app.use(express.urlencoded({ extended: true, limit: '10kb' }));

// Sanitization
app.use(mongoSanitize());
app.use(xss());
app.use(hpp());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100
});
app.use('/api/', limiter);

// Routes
app.use('/api/auth', require('./routes/auth'));
app.use('/api/users', require('./routes/users'));

// Error handling
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.statusCode || 500).json({
    success: false,
    error: process.env.NODE_ENV === 'production' 
      ? 'Server error' 
      : err.message
  });
});

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`✅ Secure server running on port ${PORT}`);
});
```

## Security Checklist

### ✅ সবসময় করুন:
- Use HTTPS in production
- Hash passwords (bcrypt)
- Validate & sanitize all input
- Use parameterized queries
- Set security headers (Helmet)
- Rate limit API endpoints
- Use strong JWT secrets
- Enable CORS properly
- Keep dependencies updated
- Log security events
- Use environment variables
- Implement proper error handling

### ❌ কখনোই করবেন না:
- Expose sensitive errors
- Store passwords in plain text
- Trust user input
- Use weak secrets
- Hardcode credentials
- Ignore security warnings
- Skip input validation

## রেফারেন্স

- **OWASP Top 10**: https://owasp.org/www-project-top-ten/
- **Express Security**: https://expressjs.com/en/advanced/best-practice-security.html
- **Helmet**: https://helmetjs.github.io/

---

**পরবর্তী Chapter**: Testing

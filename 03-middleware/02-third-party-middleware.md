# Popular Third-Party Middleware

## সংক্ষিপ্ত পরিচিতি (Brief Overview)

Express.js এ হাজার হাজার third-party middleware উপলব্ধ যা বিভিন্ন কাজ সহজ করে দেয়। এই lesson এ আমরা সবচেয়ে জনপ্রিয় এবং প্রয়োজনীয় middleware গুলো শিখব।

এই lesson এ থাকবে:
- Morgan (HTTP request logger)
- CORS (Cross-Origin Resource Sharing)
- Helmet (Security headers)
- Compression (Response compression)
- Cookie-parser
- Express-validator
- Multer (File upload)
- Express-rate-limit

## 1. Morgan - HTTP Request Logger

### Installation:
```bash
npm install morgan
```

### Basic Usage:

```javascript
const express = require('express');
const morgan = require('morgan');
const app = express();

// Predefined formats
app.use(morgan('dev'));      // Development
app.use(morgan('combined')); // Apache style
app.use(morgan('common'));   // Standard Apache format
app.use(morgan('short'));    // Shorter output
app.use(morgan('tiny'));     // Minimal output

app.get('/', (req, res) => {
  res.send('হোম পেজ');
});

app.listen(3000);
```

### Output Examples:

**dev format:**
```
GET / 200 5.123 ms - 27
POST /users 201 15.456 ms - 145
```

**combined format:**
```
::1 - - [07/Nov/2025:10:30:45 +0000] "GET / HTTP/1.1" 200 27
```

### Custom Format:

```javascript
// Custom tokens
morgan.token('user-id', (req) => {
  return req.user ? req.user.id : 'guest';
});

// Custom format
app.use(morgan(':method :url :status :response-time ms - User: :user-id'));
```

### Log to File:

```javascript
const fs = require('fs');
const path = require('path');

// Create write stream
const accessLogStream = fs.createWriteStream(
  path.join(__dirname, 'access.log'),
  { flags: 'a' }
);

// Log to file
app.use(morgan('combined', { stream: accessLogStream }));

// Log only errors to file
app.use(morgan('combined', {
  skip: (req, res) => res.statusCode < 400,
  stream: accessLogStream
}));
```

## 2. CORS - Cross-Origin Resource Sharing

### Installation:
```bash
npm install cors
```

### Basic Usage:

```javascript
const express = require('express');
const cors = require('cors');
const app = express();

// সব domain এর জন্য CORS enable
app.use(cors());

app.get('/api/data', (req, res) => {
  res.json({ message: 'CORS enabled' });
});

app.listen(3000);
```

### Specific Origin:

```javascript
// শুধু specific origin এর জন্য
app.use(cors({
  origin: 'https://example.com'
}));

// Multiple origins
app.use(cors({
  origin: ['https://example.com', 'https://app.example.com']
}));

// Dynamic origin
app.use(cors({
  origin: (origin, callback) => {
    const allowedOrigins = ['https://example.com', 'https://app.example.com'];
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  }
}));
```

### Advanced Configuration:

```javascript
const corsOptions = {
  origin: 'https://example.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  optionsSuccessStatus: 200,
  maxAge: 86400 // 24 hours
};

app.use(cors(corsOptions));
```

### Route-Specific CORS:

```javascript
// Specific route এ CORS
app.get('/api/public', cors(), (req, res) => {
  res.json({ message: 'Public API' });
});

// Different CORS for different routes
app.get('/api/admin', cors({
  origin: 'https://admin.example.com'
}), (req, res) => {
  res.json({ message: 'Admin API' });
});
```

## 3. Helmet - Security Headers

### Installation:
```bash
npm install helmet
```

### Basic Usage:

```javascript
const express = require('express');
const helmet = require('helmet');
const app = express();

// সব default security headers enable
app.use(helmet());

app.listen(3000);
```

### What Helmet Does:

Helmet নিচের headers set করে:
- `X-DNS-Prefetch-Control`
- `X-Frame-Options`
- `X-Content-Type-Options`
- `Strict-Transport-Security`
- `X-Download-Options`
- `X-Permitted-Cross-Domain-Policies`
- `Referrer-Policy`
- `Content-Security-Policy`

### Custom Configuration:

```javascript
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'", "trusted-cdn.com"],
      imgSrc: ["'self'", "data:", "https:"],
    },
  },
  hsts: {
    maxAge: 31536000, // 1 year
    includeSubDomains: true,
    preload: true
  }
}));
```

### Individual Middlewares:

```javascript
// শুধু specific protections
app.use(helmet.noSniff());              // X-Content-Type-Options
app.use(helmet.frameguard());           // X-Frame-Options
app.use(helmet.xssFilter());            // X-XSS-Protection
app.use(helmet.hidePoweredBy());        // Remove X-Powered-By
app.use(helmet.ieNoOpen());             // X-Download-Options
```

## 4. Compression - Response Compression

### Installation:
```bash
npm install compression
```

### Basic Usage:

```javascript
const express = require('express');
const compression = require('compression');
const app = express();

// Response compress করুন
app.use(compression());

app.get('/data', (req, res) => {
  // বড় response - compressed হবে
  const data = Array(10000).fill({ name: 'Test', value: 123 });
  res.json(data);
});

app.listen(3000);
```

### Custom Configuration:

```javascript
app.use(compression({
  // শুধু 1KB এর বেশি response compress করুন
  threshold: 1024,
  
  // Compression level (0-9, 9 = maximum)
  level: 6,
  
  // Filter function
  filter: (req, res) => {
    // Don't compress if request has 'x-no-compression' header
    if (req.headers['x-no-compression']) {
      return false;
    }
    
    // Use compression's default filter
    return compression.filter(req, res);
  }
}));
```

## 5. Cookie-Parser

### Installation:
```bash
npm install cookie-parser
```

### Basic Usage:

```javascript
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();

app.use(cookieParser());

// Set cookie
app.get('/set-cookie', (req, res) => {
  res.cookie('username', 'রহিম', {
    maxAge: 900000, // 15 minutes
    httpOnly: true
  });
  res.send('Cookie set');
});

// Read cookie
app.get('/get-cookie', (req, res) => {
  const username = req.cookies.username;
  res.send(`Cookie value: ${username}`);
});

// Delete cookie
app.get('/delete-cookie', (req, res) => {
  res.clearCookie('username');
  res.send('Cookie deleted');
});

app.listen(3000);
```

### Signed Cookies:

```javascript
// Secret দিয়ে initialize
app.use(cookieParser('my-secret-key'));

// Set signed cookie
app.get('/set-signed', (req, res) => {
  res.cookie('userId', '12345', { signed: true });
  res.send('Signed cookie set');
});

// Read signed cookie
app.get('/get-signed', (req, res) => {
  const userId = req.signedCookies.userId;
  res.send(`Signed cookie: ${userId}`);
});
```

### Cookie Options:

```javascript
res.cookie('name', 'value', {
  maxAge: 900000,           // মিলিসেকেন্ডে lifetime
  expires: new Date(Date.now() + 900000), // অথবা expire date
  httpOnly: true,           // JavaScript থেকে access করা যাবে না
  secure: true,             // শুধু HTTPS এ পাঠান
  sameSite: 'strict',       // CSRF protection
  path: '/',                // Cookie path
  domain: 'example.com'     // Cookie domain
});
```

## 6. Express-Validator

### Installation:
```bash
npm install express-validator
```

### Basic Validation:

```javascript
const express = require('express');
const { body, validationResult } = require('express-validator');
const app = express();

app.use(express.json());

app.post('/register',
  // Validation rules
  body('email').isEmail().withMessage('Valid email প্রয়োজন'),
  body('password').isLength({ min: 6 }).withMessage('Password কমপক্ষে 6 characters'),
  body('name').trim().notEmpty().withMessage('Name প্রয়োজন'),
  body('age').optional().isInt({ min: 18 }).withMessage('বয়স কমপক্ষে 18'),
  
  // Request handler
  (req, res) => {
    const errors = validationResult(req);
    
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    
    res.json({
      message: 'Validation successful',
      user: req.body
    });
  }
);

app.listen(3000);
```

### Advanced Validation:

```javascript
const { body, param, query, validationResult } = require('express-validator');

// Custom validator
const isStrongPassword = (value) => {
  const strongRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/;
  return strongRegex.test(value);
};

app.post('/signup',
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Valid email প্রয়োজন'),
  
  body('password')
    .isLength({ min: 8 })
    .custom(isStrongPassword)
    .withMessage('Password খুব দুর্বল'),
  
  body('confirmPassword')
    .custom((value, { req }) => value === req.body.password)
    .withMessage('Passwords মিলছে না'),
  
  body('phone')
    .optional()
    .matches(/^01[3-9]\d{8}$/)
    .withMessage('Valid বাংলাদেশী phone number দিন'),
  
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    res.json({ message: 'User registered' });
  }
);

// URL parameter validation
app.get('/users/:id',
  param('id').isInt().withMessage('User ID সংখ্যা হতে হবে'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    res.json({ userId: req.params.id });
  }
);

// Query string validation
app.get('/search',
  query('q').notEmpty().withMessage('Search query প্রয়োজন'),
  query('page').optional().isInt({ min: 1 }).withMessage('Page অবশ্যই positive integer'),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    res.json({ query: req.query });
  }
);
```

### Sanitization:

```javascript
const { body, sanitize } = require('express-validator');

app.post('/create-post',
  body('title').trim().escape(),
  body('content').trim(),
  body('tags').toArray(),
  body('email').normalizeEmail(),
  (req, res) => {
    res.json(req.body);
  }
);
```

## 7. Express-Rate-Limit

### Installation:
```bash
npm install express-rate-limit
```

### Basic Usage:

```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');
const app = express();

// Rate limiter
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // প্রতি IP এর জন্য 100 requests
  message: 'অনেক বেশি requests! 15 মিনিট পরে আবার চেষ্টা করুন।',
  standardHeaders: true,
  legacyHeaders: false,
});

// সব routes এ apply করুন
app.use(limiter);

app.get('/', (req, res) => {
  res.send('হোম পেজ');
});

app.listen(3000);
```

### Route-Specific Rate Limiting:

```javascript
// Login route এর জন্য strict limit
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // শুধু 5 attempts
  message: 'অনেক বেশি login attempts! 15 মিনিট পরে চেষ্টা করুন।',
  skipSuccessfulRequests: true // successful login count করবে না
});

app.post('/login', loginLimiter, (req, res) => {
  res.send('Login');
});

// API route এর জন্য different limit
const apiLimiter = rateLimit({
  windowMs: 1 * 60 * 1000, // 1 minute
  max: 30
});

app.use('/api/', apiLimiter);
```

### Advanced Configuration:

```javascript
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  
  // Custom key generator (default: IP address)
  keyGenerator: (req) => {
    return req.headers['x-api-key'] || req.ip;
  },
  
  // Custom handler
  handler: (req, res) => {
    res.status(429).json({
      error: 'Rate limit exceeded',
      retryAfter: Math.ceil(req.rateLimit.resetTime / 1000)
    });
  },
  
  // Skip function
  skip: (req) => {
    // Admin users কে skip করুন
    return req.user && req.user.role === 'admin';
  },
  
  // Store (default: memory)
  // Production এ Redis/Memcached use করুন
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:'
  })
});
```

## সম্পূর্ণ উদাহরণ: সব Middleware একসাথে

```javascript
require('dotenv').config();
const express = require('express');
const morgan = require('morgan');
const cors = require('cors');
const helmet = require('helmet');
const compression = require('compression');
const cookieParser = require('cookie-parser');
const rateLimit = require('express-rate-limit');
const { body, validationResult } = require('express-validator');

const app = express();

// 1. Security - Helmet (প্রথমে)
app.use(helmet());

// 2. CORS
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
  credentials: true
}));

// 3. Logging - Morgan
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
} else {
  app.use(morgan('combined'));
}

// 4. Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// 5. Cookie parsing
app.use(cookieParser(process.env.COOKIE_SECRET));

// 6. Compression
app.use(compression());

// 7. Rate limiting
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: 'অনেক বেশি requests!'
});
app.use(globalLimiter);

// Strict limiter for auth routes
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true
});

// Routes
app.get('/', (req, res) => {
  res.json({
    message: 'API চলছে',
    timestamp: new Date().toISOString()
  });
});

// Auth route with validation
app.post('/login',
  authLimiter,
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 6 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    
    // Login logic here
    res.cookie('session', 'abc123', {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      maxAge: 24 * 60 * 60 * 1000 // 24 hours
    });
    
    res.json({ message: 'Login successful' });
  }
);

// Protected route
app.get('/profile', (req, res) => {
  const session = req.cookies.session;
  
  if (!session) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  res.json({
    user: { id: 1, name: 'রহিম' }
  });
});

// 404 Handler
app.use((req, res) => {
  res.status(404).json({ error: 'Route পাওয়া যায়নি' });
});

// Error Handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).json({
    error: err.message || 'Server error'
  });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`✅ Server চলছে: http://localhost:${PORT}`);
});
```

## Package.json Dependencies:

```json
{
  "dependencies": {
    "express": "^5.1.0",
    "morgan": "^1.10.0",
    "cors": "^2.8.5",
    "helmet": "^7.1.0",
    "compression": "^1.7.4",
    "cookie-parser": "^1.4.6",
    "express-rate-limit": "^7.1.5",
    "express-validator": "^7.0.1",
    "dotenv": "^16.4.5"
  }
}
```

## Best Practices

### ✅ ১. Middleware Order

```javascript
// সঠিক order
app.use(helmet());              // 1. Security first
app.use(cors());                // 2. CORS
app.use(morgan());              // 3. Logging
app.use(express.json());        // 4. Body parsing
app.use(cookieParser());        // 5. Cookie parsing
app.use(compression());         // 6. Compression
app.use(rateLimiter);           // 7. Rate limiting
app.use('/api', routes);        // 8. Routes
app.use(errorHandler);          // 9. Error handling (শেষে)
```

### ✅ ২. Environment-based Configuration

```javascript
// Development
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
}

// Production
if (process.env.NODE_ENV === 'production') {
  app.use(compression());
  app.use(helmet());
}
```

### ✅ ৩. Proper Error Handling

```javascript
app.use((err, req, res, next) => {
  // Development এ full error
  if (process.env.NODE_ENV === 'development') {
    return res.status(500).json({
      error: err.message,
      stack: err.stack
    });
  }
  
  // Production এ minimal error
  res.status(500).json({
    error: 'Server error'
  });
});
```

## পরবর্তী পদক্ষেপ

এখন আপনি:
- ✅ Popular middleware গুলো জানেন
- ✅ Security best practices follow করতে পারেন
- ✅ Production-ready application setup করতে পারেন

পরবর্তী chapter এ আমরা **Request এবং Response Handling** শিখব।

## রেফারেন্স

- **Morgan**: https://www.npmjs.com/package/morgan
- **CORS**: https://www.npmjs.com/package/cors
- **Helmet**: https://helmetjs.github.io/
- **Compression**: https://www.npmjs.com/package/compression
- **Express-validator**: https://express-validator.github.io/
- **Express-rate-limit**: https://www.npmjs.com/package/express-rate-limit

---

**পরবর্তী Chapter**: Request এবং Response Handling

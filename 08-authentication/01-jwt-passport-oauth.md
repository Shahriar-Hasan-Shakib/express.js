# Authentication & Authorization

## সংক্ষিপ্ত পরিচিতি

Authentication (কে আপনি?) এবং Authorization (আপনার কী অনুমতি আছে?) Express applications এর অত্যন্ত গুরুত্বপূর্ণ অংশ।

## JWT (JSON Web Token) Authentication

### Installation:
```bash
npm install jsonwebtoken bcryptjs
```

### User Registration:

```javascript
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('./models/User');

// Register
app.post('/auth/register', async (req, res) => {
  try {
    const { name, email, password } = req.body;
    
    // Check if user exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(409).json({ error: 'Email already exists' });
    }
    
    // Hash password
    const hashedPassword = await bcrypt.hash(password, 10);
    
    // Create user
    const user = await User.create({
      name,
      email,
      password: hashedPassword
    });
    
    // Generate JWT
    const token = jwt.sign(
      { userId: user._id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    );
    
    res.status(201).json({
      message: 'User registered successfully',
      token,
      user: { id: user._id, name: user.name, email: user.email }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### User Login:

```javascript
// Login
app.post('/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    
    // Find user
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Check password
    const isValidPassword = await bcrypt.compare(password, user.password);
    if (!isValidPassword) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Generate JWT
    const token = jwt.sign(
      { userId: user._id, email: user.email, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    );
    
    res.json({
      message: 'Login successful',
      token,
      user: { id: user._id, name: user.name, email: user.email, role: user.role }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### Authentication Middleware:

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');

const authenticate = (req, res, next) => {
  try {
    // Get token from header
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN
    
    if (!token) {
      return res.status(401).json({ error: 'Access token প্রয়োজন' });
    }
    
    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid or expired token' });
  }
};

module.exports = authenticate;
```

### Usage:

```javascript
const authenticate = require('./middleware/auth');

// Protected route
app.get('/profile', authenticate, async (req, res) => {
  try {
    const user = await User.findById(req.user.userId).select('-password');
    res.json({ user });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Update profile
app.put('/profile', authenticate, async (req, res) => {
  try {
    const user = await User.findByIdAndUpdate(
      req.user.userId,
      req.body,
      { new: true }
    ).select('-password');
    
    res.json({ message: 'Profile updated', user });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

## Role-Based Authorization

```javascript
// middleware/authorize.js
const authorize = (...allowedRoles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: 'আপনার এই access নেই' });
    }
    
    next();
  };
};

module.exports = authorize;
```

### Usage:

```javascript
const authenticate = require('./middleware/auth');
const authorize = require('./middleware/authorize');

// Only admin can access
app.get('/admin/users', authenticate, authorize('admin'), async (req, res) => {
  const users = await User.find();
  res.json({ users });
});

// Admin and moderator can access
app.delete('/posts/:id', 
  authenticate, 
  authorize('admin', 'moderator'), 
  async (req, res) => {
    // Delete post logic
  }
);
```

## Session-Based Authentication

### Installation:
```bash
npm install express-session connect-mongo
```

### Setup:

```javascript
const session = require('express-session');
const MongoStore = require('connect-mongo');

app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  store: MongoStore.create({
    mongoUrl: process.env.MONGODB_URI,
    ttl: 14 * 24 * 60 * 60 // 14 days
  }),
  cookie: {
    maxAge: 14 * 24 * 60 * 60 * 1000, // 14 days
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production', // HTTPS only in production
    sameSite: 'strict'
  }
}));

// Login
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  
  const user = await User.findOne({ email });
  if (!user || !await bcrypt.compare(password, user.password)) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Save user in session
  req.session.userId = user._id;
  req.session.user = {
    id: user._id,
    name: user.name,
    email: user.email,
    role: user.role
  };
  
  res.json({ message: 'Login successful', user: req.session.user });
});

// Logout
app.post('/logout', (req, res) => {
  req.session.destroy(err => {
    if (err) {
      return res.status(500).json({ error: 'Logout failed' });
    }
    res.clearCookie('connect.sid');
    res.json({ message: 'Logout successful' });
  });
});

// Check auth middleware
const requireAuth = (req, res, next) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  next();
};

// Protected route
app.get('/dashboard', requireAuth, (req, res) => {
  res.json({ user: req.session.user });
});
```

## Passport.js

### Installation:
```bash
npm install passport passport-local passport-jwt
```

### Local Strategy:

```javascript
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const bcrypt = require('bcryptjs');
const User = require('./models/User');

// Configure passport
passport.use(new LocalStrategy(
  { usernameField: 'email' },
  async (email, password, done) => {
    try {
      const user = await User.findOne({ email });
      
      if (!user) {
        return done(null, false, { message: 'Invalid credentials' });
      }
      
      const isMatch = await bcrypt.compare(password, user.password);
      if (!isMatch) {
        return done(null, false, { message: 'Invalid credentials' });
      }
      
      return done(null, user);
    } catch (error) {
      return done(error);
    }
  }
));

// Serialize user
passport.serializeUser((user, done) => {
  done(null, user.id);
});

// Deserialize user
passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    done(null, user);
  } catch (error) {
    done(error);
  }
});

// Initialize passport
app.use(passport.initialize());
app.use(passport.session());

// Login route
app.post('/login', 
  passport.authenticate('local'),
  (req, res) => {
    res.json({ message: 'Login successful', user: req.user });
  }
);
```

## OAuth 2.0 (Google, Facebook, GitHub)

### Google OAuth:

```bash
npm install passport-google-oauth20
```

```javascript
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: '/auth/google/callback'
  },
  async (accessToken, refreshToken, profile, done) => {
    try {
      // Find or create user
      let user = await User.findOne({ googleId: profile.id });
      
      if (!user) {
        user = await User.create({
          googleId: profile.id,
          name: profile.displayName,
          email: profile.emails[0].value,
          avatar: profile.photos[0].value
        });
      }
      
      return done(null, user);
    } catch (error) {
      return done(error, null);
    }
  }
));

// Routes
app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] })
);

app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    res.redirect('/dashboard');
  }
);
```

## Refresh Tokens

```javascript
const jwt = require('jsonwebtoken');

// Generate tokens
const generateTokens = (user) => {
  const accessToken = jwt.sign(
    { userId: user._id, email: user.email },
    process.env.JWT_SECRET,
    { expiresIn: '15m' } // Short-lived
  );
  
  const refreshToken = jwt.sign(
    { userId: user._id },
    process.env.REFRESH_TOKEN_SECRET,
    { expiresIn: '7d' } // Long-lived
  );
  
  return { accessToken, refreshToken };
};

// Login with refresh token
app.post('/auth/login', async (req, res) => {
  const { email, password } = req.body;
  
  const user = await User.findOne({ email });
  if (!user || !await bcrypt.compare(password, user.password)) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  const { accessToken, refreshToken } = generateTokens(user);
  
  // Save refresh token to database
  user.refreshToken = refreshToken;
  await user.save();
  
  res.json({ accessToken, refreshToken });
});

// Refresh access token
app.post('/auth/refresh', async (req, res) => {
  const { refreshToken } = req.body;
  
  if (!refreshToken) {
    return res.status(401).json({ error: 'Refresh token required' });
  }
  
  try {
    const decoded = jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET);
    const user = await User.findById(decoded.userId);
    
    if (!user || user.refreshToken !== refreshToken) {
      return res.status(403).json({ error: 'Invalid refresh token' });
    }
    
    const { accessToken, refreshToken: newRefreshToken } = generateTokens(user);
    
    user.refreshToken = newRefreshToken;
    await user.save();
    
    res.json({ accessToken, refreshToken: newRefreshToken });
  } catch (error) {
    res.status(403).json({ error: 'Invalid refresh token' });
  }
});
```

## Password Reset

```javascript
const crypto = require('crypto');

// Request password reset
app.post('/auth/forgot-password', async (req, res) => {
  const { email } = req.body;
  
  const user = await User.findOne({ email });
  if (!user) {
    return res.json({ message: 'If email exists, reset link sent' });
  }
  
  // Generate reset token
  const resetToken = crypto.randomBytes(32).toString('hex');
  user.resetPasswordToken = crypto.createHash('sha256').update(resetToken).digest('hex');
  user.resetPasswordExpires = Date.now() + 3600000; // 1 hour
  await user.save();
  
  // Send email (use nodemailer)
  const resetUrl = `${req.protocol}://${req.get('host')}/auth/reset-password/${resetToken}`;
  // sendEmail(user.email, 'Password Reset', resetUrl);
  
  res.json({ message: 'Reset email sent' });
});

// Reset password
app.post('/auth/reset-password/:token', async (req, res) => {
  const hashedToken = crypto.createHash('sha256').update(req.params.token).digest('hex');
  
  const user = await User.findOne({
    resetPasswordToken: hashedToken,
    resetPasswordExpires: { $gt: Date.now() }
  });
  
  if (!user) {
    return res.status(400).json({ error: 'Invalid or expired token' });
  }
  
  user.password = await bcrypt.hash(req.body.password, 10);
  user.resetPasswordToken = undefined;
  user.resetPasswordExpires = undefined;
  await user.save();
  
  res.json({ message: 'Password reset successful' });
});
```

## Security Best Practices

### ✅ ১. Password Hashing
```javascript
const hashedPassword = await bcrypt.hash(password, 10);
```

### ✅ ২. JWT Secret Management
```javascript
// Use strong, random secrets
JWT_SECRET=your-very-long-random-secret-key-here
```

### ✅ ৩. Token Expiration
```javascript
jwt.sign(payload, secret, { expiresIn: '15m' });
```

### ✅ ৪. HTTPS Only Cookies
```javascript
cookie: { secure: true, httpOnly: true }
```

### ✅ ৫. Rate Limiting on Auth Routes
```javascript
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5
});
app.post('/login', authLimiter, loginHandler);
```

## রেফারেন্স

- **JWT**: https://jwt.io/
- **Passport.js**: http://www.passportjs.org/
- **bcryptjs**: https://www.npmjs.com/package/bcryptjs
- **OAuth 2.0**: https://oauth.net/2/

---

**পরবর্তী Chapter**: File Upload & Static Files

# Request এবং Response Object

## সংক্ষিপ্ত পরিচিতি

Request (`req`) এবং Response (`res`) object হলো Express.js এর সবচেয়ে গুরুত্বপূর্ণ components। এই lesson এ আমরা এগুলোর সব features বিস্তারিত শিখব।

## Request Object (req)

### req.params - URL Parameters

```javascript
app.get('/users/:userId/posts/:postId', (req, res) => {
  console.log(req.params.userId);  // URL থেকে userId
  console.log(req.params.postId);  // URL থেকে postId
  res.json(req.params);
});
// GET /users/123/posts/456
// Output: { userId: "123", postId: "456" }
```

### req.query - Query Strings

```javascript
app.get('/search', (req, res) => {
  console.log(req.query.q);        // Search query
  console.log(req.query.page);     // Page number
  console.log(req.query.limit);    // Results per page
  res.json(req.query);
});
// GET /search?q=express&page=2&limit=10
// Output: { q: "express", page: "2", limit: "10" }
```

### req.body - Request Body

```javascript
app.use(express.json());

app.post('/users', (req, res) => {
  const { name, email, age } = req.body;
  res.json({
    message: 'User created',
    data: req.body
  });
});
// POST /users
// Body: { "name": "রহিম", "email": "rahim@example.com", "age": 25 }
```

### req.headers - HTTP Headers

```javascript
app.get('/check-headers', (req, res) => {
  console.log(req.headers['content-type']);
  console.log(req.headers['authorization']);
  console.log(req.headers['user-agent']);
  
  res.json({
    contentType: req.headers['content-type'],
    auth: req.headers['authorization'],
    userAgent: req.headers['user-agent']
  });
});
```

### req.cookies - Cookies

```javascript
const cookieParser = require('cookie-parser');
app.use(cookieParser());

app.get('/profile', (req, res) => {
  const sessionId = req.cookies.sessionId;
  const user = req.cookies.user;
  
  res.json({
    session: sessionId,
    user: user
  });
});
```

### req এর অন্যান্য Properties

```javascript
app.get('/info', (req, res) => {
  res.json({
    method: req.method,                    // GET, POST, etc.
    url: req.url,                          // /info?page=1
    originalUrl: req.originalUrl,          // Full URL
    path: req.path,                        // /info
    hostname: req.hostname,                // example.com
    ip: req.ip,                            // Client IP
    protocol: req.protocol,                // http or https
    secure: req.secure,                    // true if HTTPS
    xhr: req.xhr,                          // true if AJAX request
    subdomains: req.subdomains,            // ['api', 'v1'] from api.v1.example.com
  });
});
```

## Response Object (res)

### res.send() - যেকোনো টাইপের Data

```javascript
app.get('/send-examples', (req, res) => {
  res.send('Text string');                    // String
  res.send({ message: 'Object' });            // Object
  res.send([1, 2, 3]);                        // Array
  res.send(Buffer.from('Buffer'));            // Buffer
  res.send('<h1>HTML</h1>');                  // HTML
});
```

### res.json() - JSON Response

```javascript
app.get('/users', (req, res) => {
  res.json({
    success: true,
    count: 2,
    data: [
      { id: 1, name: 'রহিম' },
      { id: 2, name: 'করিম' }
    ]
  });
});
```

### res.status() - HTTP Status Code

```javascript
app.get('/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.status(200).json(user);
});

// সাধারণ status codes:
// 200 - OK
// 201 - Created
// 204 - No Content
// 400 - Bad Request
// 401 - Unauthorized
// 403 - Forbidden
// 404 - Not Found
// 500 - Internal Server Error
```

### res.redirect() - Redirect

```javascript
app.get('/old-url', (req, res) => {
  res.redirect('/new-url');                    // 302 redirect
  res.redirect(301, '/permanent-url');         // 301 permanent
  res.redirect('https://example.com');         // External
  res.redirect('..');                          // Parent path
  res.redirect('back');                        // Previous page
});
```

### res.render() - Template Rendering

```javascript
app.set('view engine', 'ejs');

app.get('/profile', (req, res) => {
  res.render('profile', {
    name: 'রহিম',
    age: 25,
    email: 'rahim@example.com'
  });
});
```

### res.sendFile() - File পাঠান

```javascript
const path = require('path');

app.get('/download-pdf', (req, res) => {
  const filePath = path.join(__dirname, 'files', 'document.pdf');
  res.sendFile(filePath);
});
```

### res.download() - File Download

```javascript
app.get('/download', (req, res) => {
  const file = path.join(__dirname, 'files', 'report.pdf');
  
  // Default filename
  res.download(file);
  
  // Custom filename
  res.download(file, 'my-report.pdf');
  
  // With callback
  res.download(file, 'report.pdf', (err) => {
    if (err) {
      console.error('Download error:', err);
    }
  });
});
```

### res.set() / res.header() - Headers Set করা

```javascript
app.get('/custom-headers', (req, res) => {
  res.set('Content-Type', 'application/json');
  res.set('X-Custom-Header', 'MyValue');
  
  // অথবা multiple headers
  res.set({
    'Content-Type': 'application/json',
    'X-Powered-By': 'Express',
    'Cache-Control': 'no-cache'
  });
  
  res.json({ message: 'Headers set' });
});
```

### res.cookie() - Cookie Set করা

```javascript
app.get('/set-cookies', (req, res) => {
  // Simple cookie
  res.cookie('name', 'value');
  
  // With options
  res.cookie('userId', '12345', {
    maxAge: 900000,      // 15 minutes
    httpOnly: true,      // Client-side JS access না
    secure: true,        // HTTPS only
    sameSite: 'strict'   // CSRF protection
  });
  
  res.send('Cookies set');
});

app.get('/clear-cookies', (req, res) => {
  res.clearCookie('userId');
  res.send('Cookie cleared');
});
```

### res.type() - Content-Type Set করা

```javascript
app.get('/content-types', (req, res) => {
  res.type('json');          // application/json
  res.type('html');          // text/html
  res.type('txt');           // text/plain
  res.type('png');           // image/png
  res.type('.pdf');          // application/pdf
  
  res.send('Content type set');
});
```

### res.format() - Content Negotiation

```javascript
app.get('/data', (req, res) => {
  res.format({
    'text/plain': () => {
      res.send('Plain text data');
    },
    'text/html': () => {
      res.send('<p>HTML data</p>');
    },
    'application/json': () => {
      res.json({ message: 'JSON data' });
    },
    'default': () => {
      res.status(406).send('Not Acceptable');
    }
  });
});
```

### res.locals - Template এ Data পাঠান

```javascript
app.use((req, res, next) => {
  res.locals.currentUser = req.user;
  res.locals.siteName = 'My Site';
  next();
});

app.get('/page', (req, res) => {
  res.render('page'); // Template এ currentUser এবং siteName available
});
```

## সম্পূর্ণ CRUD উদাহরণ

```javascript
const express = require('express');
const app = express();

app.use(express.json());

let users = [
  { id: 1, name: 'রহিম', email: 'rahim@example.com', age: 25 },
  { id: 2, name: 'করিম', email: 'karim@example.com', age: 30 }
];

// GET সব users (with pagination এবং filtering)
app.get('/api/users', (req, res) => {
  let filtered = users;
  
  // Query filtering
  if (req.query.minAge) {
    filtered = filtered.filter(u => u.age >= parseInt(req.query.minAge));
  }
  
  if (req.query.search) {
    filtered = filtered.filter(u => 
      u.name.toLowerCase().includes(req.query.search.toLowerCase())
    );
  }
  
  // Pagination
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const startIndex = (page - 1) * limit;
  const endIndex = page * limit;
  
  const paginated = filtered.slice(startIndex, endIndex);
  
  res.status(200).json({
    success: true,
    count: filtered.length,
    page: page,
    totalPages: Math.ceil(filtered.length / limit),
    data: paginated
  });
});

// GET single user
app.get('/api/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  
  if (!user) {
    return res.status(404).json({
      success: false,
      error: 'User পাওয়া যায়নি'
    });
  }
  
  res.status(200).json({
    success: true,
    data: user
  });
});

// POST নতুন user
app.post('/api/users', (req, res) => {
  const { name, email, age } = req.body;
  
  // Validation
  if (!name || !email) {
    return res.status(400).json({
      success: false,
      error: 'Name এবং email প্রয়োজন'
    });
  }
  
  // Check duplicate email
  const exists = users.find(u => u.email === email);
  if (exists) {
    return res.status(409).json({
      success: false,
      error: 'Email already exists'
    });
  }
  
  const newUser = {
    id: users.length + 1,
    name,
    email,
    age: age || null,
    createdAt: new Date()
  };
  
  users.push(newUser);
  
  res.status(201).json({
    success: true,
    message: 'User created successfully',
    data: newUser
  });
});

// PUT update user
app.put('/api/users/:id', (req, res) => {
  const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
  
  if (userIndex === -1) {
    return res.status(404).json({
      success: false,
      error: 'User not found'
    });
  }
  
  users[userIndex] = {
    ...users[userIndex],
    ...req.body,
    id: parseInt(req.params.id),
    updatedAt: new Date()
  };
  
  res.status(200).json({
    success: true,
    message: 'User updated',
    data: users[userIndex]
  });
});

// DELETE user
app.delete('/api/users/:id', (req, res) => {
  const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));
  
  if (userIndex === -1) {
    return res.status(404).json({
      success: false,
      error: 'User not found'
    });
  }
  
  const deletedUser = users.splice(userIndex, 1)[0];
  
  res.status(200).json({
    success: true,
    message: 'User deleted',
    data: deletedUser
  });
});

app.listen(3000, () => {
  console.log('Server চলছে: http://localhost:3000');
});
```

## File Upload Handling

```javascript
const express = require('express');
const multer = require('multer');
const path = require('path');
const app = express();

// Multer configuration
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
  }
});

const upload = multer({
  storage: storage,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
  fileFilter: (req, file, cb) => {
    const allowedTypes = /jpeg|jpg|png|gif/;
    const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
    const mimetype = allowedTypes.test(file.mimetype);
    
    if (extname && mimetype) {
      return cb(null, true);
    }
    cb(new Error('শুধুমাত্র image files allowed!'));
  }
});

// Single file upload
app.post('/upload/single', upload.single('image'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  
  res.json({
    message: 'File uploaded successfully',
    file: {
      filename: req.file.filename,
      originalName: req.file.originalname,
      size: req.file.size,
      path: req.file.path
    }
  });
});

// Multiple files upload
app.post('/upload/multiple', upload.array('images', 10), (req, res) => {
  if (!req.files || req.files.length === 0) {
    return res.status(400).json({ error: 'No files uploaded' });
  }
  
  const fileInfo = req.files.map(file => ({
    filename: file.filename,
    size: file.size,
    path: file.path
  }));
  
  res.json({
    message: `${req.files.length} files uploaded`,
    files: fileInfo
  });
});

app.listen(3000);
```

## Best Practices

### ✅ ১. Consistent Response Structure

```javascript
// Success response
{
  "success": true,
  "data": { ... },
  "message": "Operation successful"
}

// Error response
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE"
}
```

### ✅ ২. Proper Status Codes

```javascript
res.status(200).json(data);    // OK
res.status(201).json(data);    // Created
res.status(400).json(error);   // Bad Request
res.status(404).json(error);   // Not Found
res.status(500).json(error);   // Server Error
```

### ✅ ৩. Input Validation

```javascript
const { body, validationResult } = require('express-validator');

app.post('/users',
  body('email').isEmail(),
  body('age').optional().isInt({ min: 0, max: 150 }),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // Process request
  }
);
```

## পরবর্তী পদক্ষেপ

পরবর্তী chapter এ আমরা **Template Engines** শিখব।

## রেফারেন্স

- **Request API**: https://expressjs.com/en/5x/api.html#req
- **Response API**: https://expressjs.com/en/5x/api.html#res

---

**পরবর্তী Chapter**: Template Engines (EJS, Pug, Handlebars)

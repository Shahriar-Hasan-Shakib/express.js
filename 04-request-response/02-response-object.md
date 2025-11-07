# Response Object (res)

## সংক্ষিপ্ত পরিচিতি

Response object (`res`) client-কে data পাঠাতে ব্যবহৃত হয়। JSON, HTML, files, redirects ইত্যাদি পাঠানো যায়।

## res.send() - যেকোনো ধরনের Data পাঠান

`res.send()` বিভিন্ন ধরনের data পাঠাতে পারে।

```javascript
// String পাঠান
app.get('/text', (req, res) => {
  res.send('This is a text message');
});

// Object পাঠান (automatic JSON conversion)
app.get('/object', (req, res) => {
  res.send({ message: 'This is an object', status: 'success' });
});

// Array পাঠান
app.get('/array', (req, res) => {
  res.send([1, 2, 3, 4, 5]);
});

// HTML পাঠান
app.get('/html', (req, res) => {
  res.send('<h1>স্বাগতম!</h1><p>এটি HTML content</p>');
});

// Buffer পাঠান
app.get('/buffer', (req, res) => {
  res.send(Buffer.from('Binary data'));
});
```

## res.json() - JSON Response

JSON format এ data পাঠাতে `res.json()` ব্যবহার করুন। এটি JSON.stringify() করে এবং Content-Type সেট করে।

```javascript
app.get('/api/users', (req, res) => {
  res.json({
    success: true,
    count: 2,
    data: [
      { id: 1, name: 'রহিম', city: 'ঢাকা' },
      { id: 2, name: 'করিম', city: 'চট্টগ্রাম' }
    ]
  });
});
```

**Response:**
```json
{
  "success": true,
  "count": 2,
  "data": [
    { "id": 1, "name": "রহিম", "city": "ঢাকা" },
    { "id": 2, "name": "করিম", "city": "চট্টগ্রাম" }
  ]
}
```

## res.status() - HTTP Status Code সেট করুন

Response এর HTTP status code নির্ধারণ করুন।

```javascript
app.get('/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  
  if (!user) {
    // 404 status code সহ error পাঠান
    return res.status(404).json({ 
      error: 'User পাওয়া যায়নি',
      code: 'USER_NOT_FOUND'
    });
  }
  
  // 200 status code (default) সহ success
  res.status(200).json({
    success: true,
    data: user
  });
});
```

### সাধারণ HTTP Status Codes

| Code | অর্থ | ব্যবহার |
|------|------|--------|
| 200 | OK | সফল GET request |
| 201 | Created | সফল POST (নতুন resource) |
| 204 | No Content | সফল DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Access denied |
| 404 | Not Found | Resource exists করে না |
| 409 | Conflict | Resource conflict (duplicate) |
| 500 | Server Error | Server-side error |

```javascript
app.post('/users', (req, res) => {
  res.status(201).json({ message: 'User created' });
});

app.delete('/users/:id', (req, res) => {
  res.status(204).send(); // No content
});
```

## res.redirect() - Redirect করুন

Client-কে অন্য URL এ redirect করুন।

```javascript
// 302 Temporary Redirect (default)
app.get('/old-page', (req, res) => {
  res.redirect('/new-page');
});

// 301 Permanent Redirect
app.get('/old-domain', (req, res) => {
  res.redirect(301, 'https://newdomain.com');
});

// External URL এ redirect
app.get('/go-external', (req, res) => {
  res.redirect('https://example.com');
});

// Previous page এ redirect
app.get('/back', (req, res) => {
  res.redirect('back');
});
```

### Redirect vs Send পার্থক্য

```javascript
// ❌ ভুল - response পাঠানোর পরে redirect করতে পারবেন না
app.get('/wrong', (req, res) => {
  res.send('Hello');
  res.redirect('/other');  // Error!
});

// ✅ সঠিক - return ব্যবহার করুন
app.get('/correct', (req, res) => {
  res.redirect('/other');
  // নিচের কোড execute হবে না
  console.log('This won\'t run');
});
```

## res.render() - Template Rendering

Template engine দিয়ে dynamic HTML পেজ তৈরি করুন।

```javascript
// View engine configure করুন
app.set('view engine', 'ejs');
app.set('views', './views');

app.get('/profile/:id', (req, res) => {
  res.render('profile', {
    id: req.params.id,
    name: 'রহিম',
    age: 25,
    email: 'rahim@example.com'
  });
});
```

**views/profile.ejs:**
```ejs
<h1><%= name %></h1>
<p>বয়স: <%= age %></p>
<p>ইমেইল: <%= email %></p>
```

## res.sendFile() - File পাঠান

Server থেকে file content পাঠান।

```javascript
import { fileURLToPath } from 'url';
import { dirname, join } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

app.get('/download-pdf', (req, res) => {
  const filePath = join(__dirname, 'files', 'document.pdf');
  res.sendFile(filePath);
});

// Callback error handling সহ
app.get('/send-file', (req, res) => {
  const filePath = join(__dirname, 'files', 'report.pdf');
  
  res.sendFile(filePath, (err) => {
    if (err) {
      console.error('File send error:', err);
      res.status(404).send('File not found');
    }
  });
});
```

## res.download() - File Download করান

File download করার জন্য browser এ appropriate headers set করুন।

```javascript
app.get('/download', (req, res) => {
  const file = join(__dirname, 'files', 'report.pdf');
  
  // Default filename দিয়ে download করুন
  res.download(file);
  
  // Custom filename দিয়ে download করুন
  res.download(file, 'my-report.pdf');
  
  // Error handling সহ
  res.download(file, 'my-report.pdf', (err) => {
    if (err) {
      console.error('Download failed:', err);
    }
  });
});
```

## res.set() / res.header() - Response Headers সেট করুন

HTTP headers কাস্টমাইজ করুন।

```javascript
// Single header সেট করুন
app.get('/custom-header', (req, res) => {
  res.set('X-Custom-Header', 'MyValue');
  res.set('Content-Type', 'application/json');
  
  res.json({ message: 'Headers set' });
});

// Multiple headers একসাথে
app.get('/multi-headers', (req, res) => {
  res.set({
    'X-Custom-Header': 'CustomValue',
    'X-Powered-By': 'Express',
    'Cache-Control': 'no-cache',
    'X-API-Version': '1.0'
  });
  
  res.json({ message: 'Multiple headers set' });
});
```

### সাধারণ Headers

```javascript
app.get('/security-headers', (req, res) => {
  res.set({
    'X-Content-Type-Options': 'nosniff',
    'X-Frame-Options': 'DENY',
    'X-XSS-Protection': '1; mode=block',
    'Strict-Transport-Security': 'max-age=31536000',
    'Content-Security-Policy': "default-src 'self'"
  });
  
  res.json({ message: 'Security headers set' });
});
```

## res.cookie() - Cookies সেট করুন

Client-side এ cookies store করুন।

```javascript
app.use(cookieParser());

// Simple cookie
app.get('/set-cookie', (req, res) => {
  res.cookie('name', 'value');
  res.send('Cookie set');
});

// Options সহ cookie
app.get('/set-secure-cookie', (req, res) => {
  res.cookie('sessionId', 'abc123xyz', {
    maxAge: 3600000,         // 1 hour (milliseconds)
    httpOnly: true,          // JavaScript access block করবে
    secure: true,            // HTTPS only
    sameSite: 'strict'       // CSRF protection
  });
  
  res.send('Secure cookie set');
});

// Cookie clear করুন
app.get('/clear-cookie', (req, res) => {
  res.clearCookie('sessionId');
  res.send('Cookie cleared');
});
```

### Cookie Options

```javascript
{
  domain: '.example.com',    // Cookie এর জন্য domain
  path: '/',                 // Cookie এর জন্য path
  maxAge: 3600000,          // Cookie validity (milliseconds)
  expires: new Date(),      // Expiration date
  httpOnly: true,           // JavaScript access prevent
  secure: true,             // HTTPS only
  sameSite: 'strict'        // CSRF protection (strict/lax/none)
}
```

## res.type() - Content-Type সেট করুন

Response content type স্বয়ংক্রিয়ভাবে সেট করুন।

```javascript
app.get('/json', (req, res) => {
  res.type('json').send({ message: 'JSON response' });
});

app.get('/html', (req, res) => {
  res.type('html').send('<h1>HTML response</h1>');
});

app.get('/text', (req, res) => {
  res.type('txt').send('Plain text response');
});

app.get('/pdf', (req, res) => {
  res.type('.pdf').sendFile('./file.pdf');
});
```

## res.format() - Content Negotiation

Client এর Accept header অনুযায়ী different format পাঠান।

```javascript
app.get('/data', (req, res) => {
  res.format({
    'application/json': () => {
      res.json({ message: 'JSON data' });
    },
    'text/html': () => {
      res.send('<p>HTML data</p>');
    },
    'text/plain': () => {
      res.send('Plain text data');
    },
    'default': () => {
      res.status(406).send('Not Acceptable');
    }
  });
});
```

## res.locals - Template এ Data পাঠান

Middleware থেকে template এ data পাঠান।

```javascript
// Middleware এ data set করুন
app.use((req, res, next) => {
  res.locals.currentUser = { id: 1, name: 'রহিম' };
  res.locals.siteName = 'My Express App';
  res.locals.year = new Date().getFullYear();
  next();
});

// Template এ automatically available
app.get('/page', (req, res) => {
  res.render('page'); // currentUser, siteName, year available
});
```

## সম্পূর্ণ Example

```javascript
import express from 'express';
import cookieParser from 'cookie-parser';

const app = express();

app.use(express.json());
app.use(cookieParser());

// API endpoints
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  
  // Validation
  if (!userId) {
    return res.status(400).json({ 
      error: 'User ID প্রয়োজন',
      code: 'INVALID_ID'
    });
  }
  
  // Success response
  res.status(200).json({
    success: true,
    data: {
      id: userId,
      name: 'রহিম',
      email: 'rahim@example.com'
    }
  });
});

app.post('/login', (req, res) => {
  const { email, password } = req.body;
  
  // Validate credentials
  if (!email || !password) {
    return res.status(400).json({ error: 'Email এবং password প্রয়োজন' });
  }
  
  // Set cookie
  res.cookie('sessionId', 'token123', { httpOnly: true });
  
  // Response with headers
  res.set('X-Login-Token', 'token123');
  res.status(200).json({
    success: true,
    message: 'Login successful'
  });
});

app.listen(3000, () => console.log('Server চলছে: 3000'));
```

## Best Practices

### ✅ 1. সবসময় Status Code সেট করুন

```javascript
app.post('/users', (req, res) => {
  res.status(201).json({ /* ... */ });  // Created
  // ❌ res.json({ /* ... */ });        // Default 200
});
```

### ✅ 2. Consistent Response Structure

```javascript
// Success
{ success: true, data: {...}, message: "OK" }

// Error
{ success: false, error: "Error message", code: "ERROR_CODE" }
```

### ✅ 3. চেইনিং ব্যবহার করুন

```javascript
app.get('/efficient', (req, res) => {
  res
    .status(200)
    .set('X-Custom', 'value')
    .cookie('session', 'token')
    .json({ message: 'Chained response' });
});
```

## পরবর্তী Lesson

CRUD operations সম্পর্কে বিস্তারিত শিখব।

## রেফারেন্স

- **Response API**: https://expressjs.com/en/5x/api.html#res

---

**পরবর্তী Lesson**: CRUD Operations এবং Error Handling

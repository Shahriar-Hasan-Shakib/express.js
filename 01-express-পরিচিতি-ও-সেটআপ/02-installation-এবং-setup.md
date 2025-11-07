# Express.js Installation এবং Setup

## সংক্ষিপ্ত পরিচিতি (Brief Overview)

এই lesson এ আমরা শিখব কীভাবে:
- Node.js এবং npm install করতে হয়
- নতুন Express project তৈরি করতে হয়
- Express.js install এবং configure করতে হয়
- প্রথম Express application চালাতে হয়

## পূর্বশর্ত (Prerequisites)

Express.js ব্যবহার করার আগে আপনার কম্পিউটারে থাকা দরকার:

1. **Node.js** (v18.0.0 বা তার বেশি - recommended)
2. **npm** (Node Package Manager) - Node.js এর সাথে automatically আসে
3. একটি **Code Editor** (VS Code recommended)
4. **Terminal/Command Line** এর মৌলিক জ্ঞান

## ধাপ ১: Node.js এবং npm Installation

### Linux (Ubuntu/Debian) এ Installation:

```bash
# NodeSource repository থেকে latest LTS version
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# অথবা nvm (Node Version Manager) ব্যবহার করে
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install --lts
nvm use --lts
```

### Windows এ Installation:

1. https://nodejs.org/ থেকে LTS version download করুন
2. Installer চালান এবং instructions follow করুন
3. "Add to PATH" option টি check করুন

### macOS এ Installation:

```bash
# Homebrew ব্যবহার করে
brew install node

# অথবা official installer download করুন
# https://nodejs.org/
```

### Installation যাচাই করুন:

```bash
# Node.js version check
node --version
# Output: v20.x.x বা v18.x.x

# npm version check
npm --version
# Output: 10.x.x বা 9.x.x
```

## ধাপ ২: নতুন Project তৈরি করা

### প্রজেক্ট ফোল্ডার তৈরি:

```bash
# Project directory তৈরি
mkdir my-express-app
cd my-express-app

# npm project initialize করুন
npm init -y
```

`npm init -y` command টি একটি `package.json` file তৈরি করবে:

```json
{
  "name": "my-express-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

### package.json সম্পাদনা:

আপনার project এর তথ্য যোগ করুন:

```json
{
  "name": "my-express-app",
  "version": "1.0.0",
  "description": "আমার প্রথম Express.js অ্যাপ্লিকেশন",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": ["express", "nodejs", "web-app"],
  "author": "আপনার নাম",
  "license": "MIT"
}
```

## ধাপ ৩: Express.js Installation

### Express install করুন:

```bash
# Express.js latest version install
npm install express

# অথবা specific version install করতে চাইলে
npm install express@5.1.0
```

Installation সফল হলে আপনার `package.json` এ dependencies যোগ হবে:

```json
{
  "dependencies": {
    "express": "^5.1.0"
  }
}
```

এবং একটি `node_modules` folder এবং `package-lock.json` file তৈরি হবে।

### Development Dependencies (ঐচ্ছিক কিন্তু recommended):

```bash
# Nodemon - auto-restart server on file changes
npm install --save-dev nodemon

# dotenv - environment variables management
npm install dotenv
```

Final `package.json`:

```json
{
  "name": "my-express-app",
  "version": "1.0.0",
  "description": "আমার প্রথম Express.js অ্যাপ্লিকেশন",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "keywords": ["express", "nodejs", "web-app"],
  "author": "আপনার নাম",
  "license": "MIT",
  "dependencies": {
    "express": "^5.1.0",
    "dotenv": "^16.4.5"
  },
  "devDependencies": {
    "nodemon": "^3.1.0"
  }
}
```

## ধাপ ৪: প্রথম Express Application তৈরি

### `package.json` এ ES6 Modules Enable করুন:

```json
{
  "name": "my-express-app",
  "version": "1.0.0",
  "type": "module",  // ⭐ এটি যোগ করুন ES6 modules এর জন্য
  "description": "আমার প্রথম Express.js অ্যাপ্লিকেশন",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  }
}
```

### `index.js` file তৈরি করুন (ES6 Syntax):

```javascript
// Express module import করুন (ES6 syntax)
import express from 'express';

// Express application তৈরি করুন
const app = express();

// Port number নির্ধারণ
const PORT = process.env.PORT || 3000;

// Root route (হোম পেজ)
app.get('/', (req, res) => {
  res.send('হ্যালো! এটি আমার প্রথম Express অ্যাপ্লিকেশন! 🚀');
});

// About route
app.get('/about', (req, res) => {
  res.send('এটি About পেজ');
});

// Contact route
app.get('/contact', (req, res) => {
  res.json({
    message: 'যোগাযোগ করুন',
    email: 'example@email.com',
    phone: '০১৭১১-১২৩৪৫৬'
  });
});

// 404 Handler
app.use((req, res) => {
  res.status(404).send('পেজ খুঁজে পাওয়া যায়নি! 😕');
});

// Server চালু করুন
app.listen(PORT, () => {
  console.log(`✅ সার্ভার চলছে: http://localhost:${PORT}`);
  console.log(`📡 Environment: ${process.env.NODE_ENV || 'development'}`);
});
```

> **💡 Note:** যদি CommonJS (require/module.exports) ব্যবহার করতে চান, তাহলে `package.json` থেকে `"type": "module"` সরিয়ে দিন এবং `import` এর জায়গায় `require()` ব্যবহার করুন।

### Application চালান:

```bash
# Normal mode
npm start

# Development mode (nodemon দিয়ে - auto-restart)
npm run dev
```

### Browser এ পরীক্ষা করুন:

1. `http://localhost:3000/` - হোম পেজ দেখবেন
2. `http://localhost:3000/about` - About পেজ
3. `http://localhost:3000/contact` - JSON response

## ধাপ ৫: Project Structure তৈরি করা

### Recommended Folder Structure:

```
my-express-app/
│
├── node_modules/          # Dependencies (git এ যোগ করবেন না)
├── public/                # Static files (CSS, JS, images)
│   ├── css/
│   ├── js/
│   └── images/
│
├── views/                 # Template files (EJS, Pug, etc.)
│   ├── index.ejs
│   └── about.ejs
│
├── routes/                # Route definitions
│   ├── index.js
│   ├── users.js
│   └── products.js
│
├── controllers/           # Route handlers (business logic)
│   ├── userController.js
│   └── productController.js
│
├── models/                # Database models
│   ├── User.js
│   └── Product.js
│
├── middleware/            # Custom middleware
│   ├── auth.js
│   └── logger.js
│
├── config/                # Configuration files
│   └── database.js
│
├── .env                   # Environment variables (git এ যোগ করবেন না)
├── .gitignore            # Git ignore file
├── package.json          # Project metadata and dependencies
├── package-lock.json     # Locked versions of dependencies
└── index.js              # Main application file
```

### `.gitignore` file তৈরি করুন:

```gitignore
# Dependencies
node_modules/
npm-debug.log
yarn-error.log

# Environment variables
.env
.env.local
.env.production

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Build files
dist/
build/
```

### `.env` file তৈরি করুন:

```env
# Server Configuration
PORT=3000
NODE_ENV=development

# Database Configuration (পরবর্তীতে ব্যবহার করবেন)
# DB_HOST=localhost
# DB_PORT=5432
# DB_NAME=myapp
# DB_USER=postgres
# DB_PASSWORD=password

# JWT Secret (পরবর্তীতে ব্যবহার করবেন)
# JWT_SECRET=your-secret-key
```

### `.env` ব্যবহার করে `index.js` update:

```javascript
// Load environment variables
require('dotenv').config();

const express = require('express');
const app = express();

// Use PORT from environment variable or default to 3000
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.send('হ্যালো! এটি আমার প্রথম Express অ্যাপ্লিকেশন! 🚀');
});

app.listen(PORT, () => {
  console.log(`✅ সার্ভার চলছে: http://localhost:${PORT}`);
  console.log(`🌍 Environment: ${process.env.NODE_ENV}`);
});
```

## সম্পূর্ণ উদাহরণ: Organized Structure সহ

### `index.js` (Main file):

```javascript
require('dotenv').config();
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(express.json()); // JSON parsing
app.use(express.urlencoded({ extended: true })); // URL-encoded data parsing
app.use(express.static('public')); // Static files

// Routes
app.get('/', (req, res) => {
  res.json({
    message: 'Welcome to Express API',
    version: '1.0.0',
    endpoints: {
      home: '/',
      about: '/about',
      users: '/api/users'
    }
  });
});

app.get('/about', (req, res) => {
  res.json({
    app: 'My Express App',
    description: 'A simple Express.js application',
    author: 'Your Name'
  });
});

// 404 Handler
app.use((req, res) => {
  res.status(404).json({
    error: 'পেজ পাওয়া যায়নি',
    message: `${req.url} এই route টি exist করে না`
  });
});

// Error Handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'কিছু একটা ভুল হয়েছে!',
    message: err.message
  });
});

// Start Server
app.listen(PORT, () => {
  console.log(`✅ সার্ভার চলছে: http://localhost:${PORT}`);
});
```

## সাধারণ সমস্যা এবং সমাধান (Common Mistakes & Fixes)

### ❌ সমস্যা ১: ES6 modules error - `require is not defined`

**কারণ**: package.json এ `"type": "module"` আছে কিন্তু `require()` ব্যবহার করছেন

**ভুল কোড**:
```javascript
// ❌ ভুল - ES6 modules enabled কিন্তু CommonJS syntax
const express = require('express'); // Error!
```

**সমাধান**:
```javascript
// ✅ সঠিক - ES6 import ব্যবহার করুন
import express from 'express';

// অথবা package.json থেকে "type": "module" সরিয়ে দিন
```

### ❌ সমস্যা ২: `Cannot find module 'express'`

**কারণ**: Express install করা হয়নি বা node_modules missing

**সমাধান**:
```bash
# Express install করুন
npm install express

# যদি node_modules delete হয়ে যায়
npm install  # সব dependencies reinstall হবে
```

### ❌ সমস্যা ৩: `Port 3000 is already in use`

**কারণ**: অন্য কোনো application ইতিমধ্যে port 3000 ব্যবহার করছে

**সমাধান**:
```bash
# Linux/Mac এ process খুঁজে বের করুন
lsof -i :3000
kill -9 <PID>

# Windows এ
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# অথবা অন্য port ব্যবহার করুন
# .env ফাইলে
PORT=4000
```

### ❌ সমস্যা ৪: `nodemon: command not found`

**কারণ**: nodemon globally install করা নেই

**সমাধান**:
```bash
# Locally install করুন (recommended)
npm install --save-dev nodemon

# package.json এ script যোগ করুন
"scripts": {
  "dev": "nodemon index.js"
}

# তারপর চালান
npm run dev

# Global install করতে চাইলে (not recommended)
npm install -g nodemon
```

### ❌ সমস্যা ৫: `.env` file কাজ করছে না

**কারণ**: dotenv package install করা হয়নি বা load করা হয়নি

**সমাধান**:
```bash
# Install করুন
npm install dotenv
```

```javascript
// CommonJS এ
require('dotenv').config();

// ES6 modules এ
import 'dotenv/config';
// অথবা
import dotenv from 'dotenv';
dotenv.config();
```

### ❌ সমস্যা ৬: `app.listen is not a function`

**কারণ**: Express সঠিকভাবে instantiate করা হয়নি

**ভুল কোড**:
```javascript
// ❌ ভুল
import express from 'express';
const app = express; // Forgot parentheses!
```

**সঠিক কোড**:
```javascript
// ✅ সঠিক
import express from 'express';
const app = express(); // Call the function
```

### ❌ সমস্যা ৭: File extension error (.mjs vs .js)

**কারণ**: ES6 modules ব্যবহার করলে file extension নিয়ে সমস্যা

**সমাধান**:
```json
// Option 1: package.json এ "type": "module" যোগ করুন
{
  "type": "module"
}
// এখন .js files ES6 modules হিসেবে কাজ করবে

// Option 2: .mjs extension ব্যবহার করুন
// index.mjs (ES6 modules)
// index.js (CommonJS)
```

### ❌ সমস্যা ৮: __dirname is not defined (ES6 modules এ)

**কারণ**: ES6 modules এ `__dirname` এবং `__filename` available নেই

**সমাধান**:
```javascript
// ✅ ES6 modules এ
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

### ❌ সমস্যা ৯: npm start কাজ করছে না

**কারণ**: package.json এ start script missing বা ভুল

**সমাধান**:
```json
{
  "scripts": {
    "start": "node index.js",  // ✅ Correct entry point
    "dev": "nodemon index.js"
  }
}
```

### ❌ সমস্যা ১০: Permission denied error (Linux/Mac)

**কারণ**: Port 80 বা 443 ব্যবহার করতে root permission দরকার

**সমাধান**:
```bash
# Option 1: Higher port ব্যবহার করুন (1024+)
PORT=3000 npm start

# Option 2: sudo ব্যবহার করুন (not recommended)
sudo npm start

# Option 3: Nginx reverse proxy ব্যবহার করুন (best practice)
```

## Best Practices

### ✅ ১. Environment Variables ব্যবহার করুন
```javascript
require('dotenv').config();
const PORT = process.env.PORT || 3000;
```

### ✅ ২. Error Handling করুন
```javascript
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).send('Server Error');
});
```

### ✅ ৩. Proper Project Structure রাখুন
- Routes আলাদা file এ রাখুন
- Controllers আলাদা করুন
- Config files আলাদা folder এ রাখুন

### ✅ ৪. Version Control ব্যবহার করুন
```bash
git init
git add .
git commit -m "Initial Express setup"
```

### ✅ ৫. Nodemon ব্যবহার করুন Development এ
```bash
npm run dev
```

## পরবর্তী পদক্ষেপ

এখন আপনি:
- ✅ Node.js এবং npm install করতে পারেন
- ✅ Express.js project তৈরি করতে পারেন
- ✅ Basic Express application চালাতে পারেন
- ✅ Proper project structure follow করতে পারেন

পরবর্তী lesson এ আমরা Express.js এর **Routing** সম্পর্কে বিস্তারিত শিখব।

## রেফারেন্স এবং দরকারী লিংক

- **Node.js Download**: https://nodejs.org/
- **Express Installation Guide**: https://expressjs.com/en/starter/installing.html
- **npm Documentation**: https://docs.npmjs.com/
- **nodemon**: https://www.npmjs.com/package/nodemon
- **dotenv**: https://www.npmjs.com/package/dotenv

---

**পরবর্তী Lesson**: Basic Routing এবং HTTP Methods

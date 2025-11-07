# Express.js সম্পূর্ণ বাংলা ডকুমেন্টেশন

**Express.js 5.1.0** - Fast, unopinionated, minimalist web framework for Node.js

এই ডকুমেন্টেশনটি Express.js শেখার জন্য একটি সম্পূর্ণ বাংলা গাইড। নতুন এবং অভিজ্ঞ - সবার জন্য উপযুক্ত।

---

## 📚 বিষয়সূচি (Table of Contents)

### [অধ্যায় ১: Express.js পরিচিতি ও সেটআপ](./01-express-পরিচিতি-ও-সেটআপ/)
- [১.১ Express.js কী এবং কেন ব্যবহার করবেন](./01-express-পরিচিতি-ও-সেটআপ/01-express-কি-এবং-কেন.md)
  - Express.js কী?
  - কেন Express.js ব্যবহার করবেন?
  - প্রধান Features
  - ইতিহাস ও বর্তমান সংস্করণ
  
- [১.২ Installation এবং Setup](./01-express-পরিচিতি-ও-সেটআপ/02-installation-এবং-setup.md)
  - Node.js এবং npm Installation
  - প্রথম Express Project তৈরি
  - Project Structure
  - সাধারণ সমস্যা ও সমাধান

---

### [অধ্যায় ২: Routing](./02-routing/)
- [২.১ Basic Routing](./02-routing/01-basic-routing.md)
  - HTTP Methods (GET, POST, PUT, DELETE)
  - Route Parameters
  - Query Strings
  - Response Methods
  - CRUD Operations

- [২.২ Express Router](./02-routing/02-express-router.md)
  - Router তৈরি করা
  - Route Modularization
  - Nested Routers
  - Route Versioning
  - Best Practices

---

### [অধ্যায় ৩: Middleware](./03-middleware/)
- [৩.১ Middleware Concept](./03-middleware/01-middleware-concept.md)
  - Middleware কী?
  - কীভাবে কাজ করে?
  - Custom Middleware তৈরি
  - Middleware Types
  - Execution Order

- [৩.২ Third-Party Middleware](./03-middleware/02-third-party-middleware.md)
  - Morgan (HTTP Logger)
  - CORS
  - Helmet (Security)
  - Compression
  - Cookie-Parser
  - Express-Validator
  - Express-Rate-Limit

---

### [অধ্যায় ৪: Request এবং Response](./04-request-response/)
- [৪.১ Request এবং Response Objects](./04-request-response/01-request-response-objects.md)
  - Request Object (req.params, req.query, req.body)
  - Response Methods (res.send, res.json, res.status)
  - Headers, Cookies
  - File Upload

---

### [অধ্যায় ৫: Template Engines](./05-template-engines/)
- [৫.১ EJS, Pug, Handlebars](./05-template-engines/01-ejs-pug-handlebars.md)
  - EJS (Embedded JavaScript)
  - Pug (formerly Jade)
  - Handlebars
  - Partials এবং Layouts

---

### [অধ্যায় ৬: Error Handling](./06-error-handling/)
- [৬.১ Error Handling](./06-error-handling/01-error-handling.md)
  - Error Handling Middleware
  - Custom Error Classes
  - Async Error Handling
  - Database Errors
  - Validation Errors

---

### [অধ্যায় ৭: Database Integration](./07-database/)
- [৭.১ MongoDB, PostgreSQL, MySQL](./07-database/01-mongodb-postgresql-mysql.md)
  - MongoDB with Mongoose
  - PostgreSQL with pg
  - MySQL with mysql2
  - CRUD Operations
  - Query Optimization
  - ORM/Query Builders (Sequelize, Prisma)

---

### [অধ্যায় ৮: Authentication & Authorization](./08-authentication/)
- [৮.১ JWT, Passport, OAuth](./08-authentication/01-jwt-passport-oauth.md)
  - JWT Authentication
  - Session-Based Authentication
  - Passport.js
  - OAuth 2.0 (Google, Facebook, GitHub)
  - Refresh Tokens
  - Password Reset
  - Role-Based Authorization

---

### [অধ্যায় ৯: File Upload & Static Files](./09-file-upload/)
- [৯.১ Multer & Static Files](./09-file-upload/01-multer-static-files.md)
  - Static Files Serving
  - Multer Configuration
  - File Type Filtering
  - Image Processing (Sharp)
  - Cloud Storage (Cloudinary, AWS S3)

---

### [অধ্যায় ১০: Security Best Practices](./10-security/)
- [১০.১ Security Best Practices](./10-security/01-security-best-practices.md)
  - Helmet (Security Headers)
  - CORS Configuration
  - Rate Limiting
  - Input Validation & Sanitization
  - SQL Injection Prevention
  - XSS Protection
  - CSRF Protection
  - Password Security
  - JWT Security
  - HTTPS Enforcement

---

### [অধ্যায় ১১: Testing](./11-testing/)
- [১১.১ Jest & Supertest Testing](./11-testing/01-jest-supertest-testing.md)
  - Unit Testing
  - Integration Testing
  - API Testing
  - Authentication Testing
  - Mocking
  - Test Coverage

---

### [অধ্যায় ১২: Performance & Optimization](./12-performance/)
- [১২.১ Caching, Clustering, Optimization](./12-performance/01-caching-clustering-optimization.md)
  - Compression
  - Caching (In-Memory, Redis)
  - Database Optimization
  - Clustering
  - PM2 Process Manager
  - Connection Pooling
  - Monitoring

---

### [অধ্যায় ১৩: Deployment](./13-deployment/)
- [১৩.১ PM2, Docker, Heroku, AWS](./13-deployment/01-pm2-docker-heroku-aws.md)
  - PM2 Deployment
  - Docker & Docker Compose
  - Nginx Reverse Proxy
  - Heroku Deployment
  - VPS Deployment (DigitalOcean)
  - AWS EC2 Deployment
  - SSL/TLS Certificates
  - CI/CD Pipeline
  - Monitoring & Logging

---

## 🎯 কীভাবে শিখবেন

১. **ক্রমানুসারে পড়ুন**: Chapter 1 থেকে শুরু করে ক্রমানুসারে এগিয়ে যান
২. **Code Practice করুন**: প্রতিটি উদাহরণ নিজে লিখে পরীক্ষা করুন
৩. **Project তৈরি করুন**: শেখার সাথে সাথে ছোট ছোট প্রজেক্ট তৈরি করুন
৪. **Official Documentation পড়ুন**: এই guide এর সাথে official docs ও পড়ুন

---

## 💡 Prerequisites (পূর্বশর্ত)

- JavaScript এর মৌলিক জ্ঞান
- Node.js সম্পর্কে ধারণা
- HTML/CSS (Template Engines এর জন্য)
- Database এর মৌলিক ধারণা (ঐচ্ছিক)

---

## 🛠️ প্রয়োজনীয় Software

- **Node.js** (v18.0.0 বা তার বেশি)
- **npm** বা **yarn**
- **Code Editor** (VS Code recommended)
- **MongoDB** / **PostgreSQL** / **MySQL** (যেকোনো একটি)
- **Postman** বা **Insomnia** (API testing এর জন্য)

---

## 📦 Quick Start

```bash
# Project তৈরি করুন
mkdir my-express-app
cd my-express-app

# npm initialize করুন
npm init -y

# Express install করুন
npm install express

# প্রথম server তৈরি করুন
echo "const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('হ্যালো Express! 🚀');
});

app.listen(3000, () => {
  console.log('Server চলছে: http://localhost:3000');
});" > index.js

# Server চালান
node index.js
```

Browser এ যান: `http://localhost:3000`

---

## 📖 প্রতিটি Chapter এ যা থাকবে

- **সংক্ষিপ্ত পরিচিতি** (Brief Overview)
- **Step-by-step Installation/Setup**
- **Runnable Code Examples**
- **সম্পূর্ণ উদাহরণ প্রজেক্ট**
- **Common Mistakes & Fixes**
- **Best Practices**
- **References/Links**

---

## 🎓 এই ডকুমেন্টেশন থেকে আপনি শিখবেন

✅ Express.js দিয়ে RESTful API তৈরি করা  
✅ Authentication & Authorization implement করা  
✅ Database integration (MongoDB, PostgreSQL, MySQL)  
✅ File upload এবং processing  
✅ Security best practices  
✅ Testing (Unit, Integration)  
✅ Performance optimization  
✅ Production deployment  
✅ Real-world project structure  

---

## 🌟 বৈশিষ্ট্য

- ✅ **সম্পূর্ণ বাংলায়**: সহজ ভাষায় বিস্তারিত ব্যাখ্যা
- ✅ **Latest Version**: Express 5.1.0 (November 2025)
- ✅ **Practical Examples**: প্রতিটি concept এর জন্য working code
- ✅ **Production Ready**: Real-world best practices
- ✅ **Comprehensive**: Basic থেকে Advanced সব topics
- ✅ **Verified Content**: Official documentation থেকে যাচাইকৃত

---

## 🤝 অবদান (Contribution)

এই ডকুমেন্টেশন open-source এবং community-driven। আপনিও অবদান রাখতে পারেন:

- Issues report করুন
- Suggestions দিন
- Examples যোগ করুন
- Translation improve করুন

---

## 📞 সাহায্য প্রয়োজন?

- **Official Express Documentation**: https://expressjs.com/
- **GitHub Issues**: Report bugs or ask questions
- **Stack Overflow**: Tag `express` এবং `node.js`

---

## 📜 License

এই ডকুমেন্টেশন শিক্ষামূলক উদ্দেশ্যে তৈরি। Express.js এর license: MIT License

---

## ⭐ শেষ কথা

Express.js শেখা একটি journey। এই ডকুমেন্টেশন আপনার সেই journey কে সহজ এবং আনন্দদায়ক করার জন্য তৈরি করা হয়েছে। ধৈর্য ধরুন, practice করুন, এবং মজা করে শিখুন!

**Happy Coding! 🚀**

---

**Documentation Version**: 1.0.0  
**Last Updated**: November 7, 2025  
**Express.js Version**: 5.1.0

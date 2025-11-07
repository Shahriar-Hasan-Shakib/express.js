# Input Validation in Express.js

## কেন Input Validation গুরুত্বপূর্ণ?

Input validation হলো API এবং ওয়েব অ্যাপ্লিকেশনের নিরাপত্তা ও স্থিতিশীলতা নিশ্চিত করার অন্যতম প্রধান ধাপ।

- **Security**: SQL Injection, NoSQL Injection, XSS-এর মতো আক্রমণ থেকে রক্ষা পেতে সাহায্য করে।
- **Data Integrity**: ডেটাবেসে সঠিক ও প্রত্যাশিত মান ঢুকছে কিনা নিশ্চিত করে।
- **User Experience**: ব্যবহারকারীদের পরিষ্কার error message দেয়া যায়।
- **Stability**: invalid request থেকে অ্যাপ ক্র্যাশ হওয়ার সম্ভাবনা কমে যায়।

এই lesson এ আমরা Express.js এ input validation করার বিভিন্ন পদ্ধতি শিখব।

---

## express-validator দিয়ে Validation

`express-validator` একটি জনপ্রিয় validation লাইব্রেরি যা middleware হিসেবে কাজ করে।

### Installation

```bash
npm install express-validator
```

### Simple Example

```javascript
import express from 'express';
import { body, validationResult } from 'express-validator';

const app = express();
app.use(express.json());

app.post('/api/users',
  [
    body('name')
      .trim()
      .notEmpty()
      .withMessage('Name প্রয়োজন')
      .isLength({ min: 2, max: 50 })
      .withMessage('Name 2-50 character এর মধ্যে হতে হবে'),
    body('email')
      .isEmail()
      .withMessage('Valid email দিন')
      .normalizeEmail(),
    body('password')
      .isLength({ min: 6 })
      .withMessage('Password কমপক্ষে 6 character হতে হবে')
  ],
  (req, res) => {
    const errors = validationResult(req);

    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        errors: errors.array().map(err => ({
          field: err.param,
          message: err.msg
        }))
      });
    }

    res.status(201).json({ success: true, data: req.body });
  }
);

app.listen(3000, () => console.log('Server running on port 3000'));
```

### Nested Validation

```javascript
app.post('/api/orders', [
  body('items').isArray({ min: 1 }).withMessage('কমপক্ষে একটি item প্রয়োজন'),
  body('items.*.productId')
    .isString()
    .withMessage('productId string হতে হবে'),
  body('items.*.quantity')
    .isInt({ gt: 0 })
    .withMessage('quantity একটি positive integer হতে হবে'),
  body('shipping.address')
    .notEmpty()
    .withMessage('shipping address প্রয়োজন'),
  body('shipping.postcode')
    .matches(/^[0-9]{4}$/)
    .withMessage('Valid পোস্টকোড দিন')
], handler);
```

### Custom Validators

```javascript
import { body } from 'express-validator';

const isStrongPassword = value => {
  const pattern = /^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[!@#$%^&*]).{8,}$/;
  if (!pattern.test(value)) {
    throw new Error('Password অবশ্যই 8 character, uppercase, lowercase, number এবং special character থাকতে হবে');
  }
  return true;
};

app.post('/auth/register', [
  body('password').custom(isStrongPassword)
], handler);
```

---

## Joi দিয়ে Schema Validation

[Joi](https://joi.dev/) হলো একটি declarative schema validation লাইব্রেরি যা object validation এর জন্য শক্তিশালী।

### Installation

```bash
npm install joi
```

### Basic Schema

```javascript
import Joi from 'joi';

const userSchema = Joi.object({
  name: Joi.string().trim().min(2).max(50).required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(6).required(),
  role: Joi.string().valid('user', 'admin', 'moderator').default('user'),
  profile: Joi.object({
    bio: Joi.string().max(160).allow(null, ''),
    website: Joi.string().uri().optional()
  }).optional()
});

app.post('/api/users', async (req, res) => {
  try {
    const value = await userSchema.validateAsync(req.body, {
      abortEarly: false,
      stripUnknown: true
    });

    res.status(201).json({ success: true, data: value });
  } catch (err) {
    res.status(400).json({
      success: false,
      errors: err.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message
      }))
    });
  }
});
```

### Schema Reuse

```javascript
const createUserSchema = userSchema;

const updateUserSchema = userSchema.fork(
  ['name', 'email', 'password'],
  field => field.optional()
);

app.put('/api/users/:id', async (req, res) => {
  const value = await updateUserSchema.validateAsync(req.body, { abortEarly: false });
  // Update logic
});
```

### Conditional Validation

```javascript
const paymentSchema = Joi.object({
  method: Joi.string().valid('card', 'bkash', 'cod').required(),
  card: Joi.object({
    number: Joi.string().creditCard().required(),
    expiry: Joi.string().pattern(/^(0[1-9]|1[0-2])\/(2[3-9]|3\d)$/).required(),
    cvv: Joi.string().length(3).pattern(/^[0-9]+$/).required()
  }).when('method', {
    is: 'card',
    then: Joi.required(),
    otherwise: Joi.forbidden()
  }),
  transactionId: Joi.string().when('method', {
    is: 'bkash',
    then: Joi.required(),
    otherwise: Joi.optional()
  })
});
```

---

## Validation Error Responses

একটি standard error response structure রাখা উচিত, যাতে client সহজে বুঝতে পারে।

```javascript
const formatValidationErrors = (errors = []) => errors.map(err => ({
  field: err.path || err.param,
  message: err.message || err.msg,
  value: err.value
}));

res.status(400).json({
  success: false,
  error: 'Validation failed',
  errors: formatValidationErrors(validationErrors)
});
```

### Best Practices
- **Consistent Structure**: সব validation error response একই structure এ রাখুন।
- **Localization**: Error message localization করা যায় (i18n)।
- **Logging**: Invalid data logging করলে security issue এড়াতে sanitized data ব্যবহার করুন।

---

## Sanitization Best Practices

Validation এর পাশাপাশি sanitization গুরুত্বপূর্ণ:

- `express-validator` এর `.escape()`, `.trim()`, `.normalizeEmail()` ইত্যাদি ব্যবহার করুন।
- Dangerous characters মুছে ফেলুন (`<`, `>`, `'`, `"` ইত্যাদি)।
- `DOMPurify`, `sanitize-html` এর মতো লাইব্রেরি ব্যবহার করতে পারেন user-generated HTML content এর জন্য।
- Database-specific sanitization করুন (SQL parameterized queries, MongoDB অপারেটর ব্লক ইত্যাদি)।

```javascript
body('username')
  .trim()
  .escape()
  .isAlphanumeric()
  .withMessage('Username শুধুমাত্র letters এবং numbers হতে পারে');
```

---

## Request Validation Middleware Architecture

### Reusable Middleware Factory

```javascript
const validate = validations => {
  return async (req, res, next) => {
    await Promise.all(validations.map(validation => validation.run(req)));

    const errors = validationResult(req);
    if (errors.isEmpty()) {
      return next();
    }

    res.status(400).json({
      success: false,
      error: 'Validation failed',
      errors: errors.array()
    });
  };
};

app.post('/api/products', validate([
  body('name').notEmpty(),
  body('price').isFloat({ gt: 0 }),
  body('stock').optional().isInt({ min: 0 })
]), handler);
```

### Layered Validation

- **Request Layer**: express-validator / Joi দিয়ে basic schema validation
- **Service Layer**: business rule validation (unique email check, state transitions)
- **Database Layer**: database constraints (unique index, foreign key)

---

## File Upload এবং Multipart Validation

```javascript
import multer from 'multer';

const upload = multer({
  limits: { fileSize: 5 * 1024 * 1024 },
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png'];
    if (!allowed.includes(file.mimetype)) {
      return cb(new Error('শুধুমাত্র jpg/png ফাইল অনুমোদিত'), false);
    }
    cb(null, true);
  }
});

app.post('/api/profile',
  upload.single('avatar'),
  [ body('name').notEmpty() ],
  handler
);
```

---

## Validation Rules Versioning

API version অনুযায়ী validation rules পরিবর্তন হতে পারে।

```javascript
const v1Rules = [ body('username').isAlphanumeric() ];
const v2Rules = [ body('username').matches(/^[a-z0-9_]+$/i) ];

app.post('/api/v1/users', validate(v1Rules), handlerV1);
app.post('/api/v2/users', validate(v2Rules), handlerV2);
```

---

## Testing Validation Logic

```javascript
import request from 'supertest';
import app from '../app.js';

describe('POST /api/users validation', () => {
  it('should return 400 if email invalid', async () => {
    const res = await request(app)
      .post('/api/users')
      .send({ name: 'রহিম', email: 'invalid', password: 'secret123' });

    expect(res.statusCode).toBe(400);
    expect(res.body.success).toBe(false);
  });
});
```

---

## Summary Checklist

- [x] Why validation is necessary
- [x] express-validator basics, nested, custom
- [x] Joi schema validation এবং reuse
- [x] Standardized error responses
- [x] Sanitization best practices
- [x] Reusable validation middleware
- [x] File upload validation
- [x] Versioned validation rules
- [x] Validation tests

---

## পরবর্তী Lesson

**Lesson 26: Express Security Fundamentals** – এখানে আমরা Helmet.js, CORS, rate limiting সহ নিরাপত্তা বিষয়ক আলোচনা করব।

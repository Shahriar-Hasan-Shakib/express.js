# REST API CRUD Operations

## সংক্ষিপ্ত পরিচিতি

CRUD (Create, Read, Update, Delete) হলো RESTful API এর foundation। প্রতিটি resource-কে ব্যাবস্থাপনাযোগ্য ও পর্যাপ্ত কভারেজ দেওয়ার জন্য CRUD operations সঠিকভাবে ডিজাইন করা জরুরি।

এই lesson এ আমরা Express.js ব্যবহার করে production-grade CRUD operations তৈরি করার best practices শিখব।

---

## RESTful Route Design

| Operation | HTTP Method | Endpoint | Description |
|-----------|-------------|----------|-------------|
| Create    | POST        | `/api/v1/users` | নতুন user তৈরি |
| Read (all) | GET        | `/api/v1/users` | সব user দেখুন |
| Read (single) | GET     | `/api/v1/users/:id` | নির্দিষ্ট user দেখুন |
| Update (replace) | PUT  | `/api/v1/users/:id` | সম্পূর্ণ resource update |
| Update (partial) | PATCH | `/api/v1/users/:id` | আংশিক update |
| Delete    | DELETE      | `/api/v1/users/:id` | Resource delete |

Best Practice:
- Endpoint শুধু resource নির্দেশ করে, action নয়।
- Plural nouns ব্যবহার করুন (`/users`, `/products`).

---

## Create (POST) Operations

```javascript
import express from 'express';
import { body, validationResult } from 'express-validator';
import User from '../models/User.js';

const router = express.Router();

router.post('/', [
  body('name').trim().notEmpty(),
  body('email').isEmail(),
  body('password').isLength({ min: 6 })
], async (req, res, next) => {
  try {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ success: false, errors: errors.array() });
    }

    const { name, email, password } = req.body;

    const existing = await User.findOne({ email });
    if (existing) {
      return res.status(409).json({ success: false, error: 'Email already taken' });
    }

    const user = await User.create({ name, email, password });

    res.status(201).json({
      success: true,
      message: 'User created successfully',
      data: user
    });
  } catch (error) {
    next(error);
  }
});
```

**Tips**
- Duplicate data চেক করুন (unique constraints)।
- Created resource এর বিবরণ ও location header পাঠান।

```javascript
res.status(201)
  .location(`/api/v1/users/${user.id}`)
  .json({ success: true, data: user });
```

---

## Read (GET) Operations

### সব Resource Fetch করা

```javascript
router.get('/', async (req, res, next) => {
  try {
    const { page = 1, limit = 10 } = req.query;
    const skip = (page - 1) * limit;

    const [users, total] = await Promise.all([
      User.find().skip(skip).limit(parseInt(limit)).sort('-createdAt'),
      User.countDocuments()
    ]);

    res.json({
      success: true,
      count: users.length,
      total,
      page: parseInt(page),
      totalPages: Math.ceil(total / limit),
      data: users
    });
  } catch (error) {
    next(error);
  }
});
```

### একক Resource Fetch করা

```javascript
router.get('/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);

    if (!user) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }

    res.json({ success: true, data: user });
  } catch (error) {
    next(error);
  }
});
```

**Tips**
- Query parameters দিয়ে filtering, sorting, field selection করুন।
- Aggregation pipeline ব্যবহার করে analytics data প্রদান করুন।

---

## Update Operations (PUT/PATCH)

### PUT: সম্পূর্ণ replace

```javascript
router.put('/:id', async (req, res, next) => {
  try {
    const user = await User.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true, overwrite: true }
    );

    if (!user) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }

    res.json({ success: true, data: user });
  } catch (error) {
    next(error);
  }
});
```

### PATCH: আংশিক update

```javascript
router.patch('/:id', async (req, res, next) => {
  try {
    const updates = Object.keys(req.body);
    const allowed = ['name', 'email', 'password', 'role'];

    const isValid = updates.every(update => allowed.includes(update));
    if (!isValid) {
      return res.status(400).json({ success: false, error: 'Invalid fields' });
    }

    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }

    updates.forEach(field => {
      user[field] = req.body[field];
    });

    await user.save();

    res.json({ success: true, data: user });
  } catch (error) {
    next(error);
  }
});
```

**Tips**
- PUT/ PATCH response সবার জন্য consistent রাখুন।
- Optimistic locking / versioning ব্যবহার করুন concurrency হ্যান্ডেল করতে।

```javascript
userSchema.pre('save', function(next) {
  this.increment();
  next();
});
```

---

## Delete (DELETE) Operations

```javascript
router.delete('/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);

    if (!user) {
      return res.status(404).json({ success: false, error: 'User not found' });
    }

    await user.deleteOne();

    res.status(200).json({
      success: true,
      message: 'User deleted successfully'
    });
  } catch (error) {
    next(error);
  }
});
```

**Soft Delete**

```javascript
// Model field
isDeleted: { type: Boolean, default: false }

router.delete('/:id', async (req, res, next) => {
  const user = await User.findByIdAndUpdate(req.params.id, {
    isDeleted: true,
    deletedAt: new Date()
  }, { new: true });

  res.status(200).json({ success: true, data: user });
});
```

---

## Bulk Operations

### Bulk Create

```javascript
router.post('/bulk', async (req, res, next) => {
  try {
    const users = await User.insertMany(req.body, { ordered: false });

    res.status(201).json({ success: true, inserted: users.length });
  } catch (error) {
    // ordered: false হলে কিছু insert সফল হতে পারে, failure list থেকে জানা যায়
    res.status(207).json({ success: false, message: 'Some inserts failed', error });
  }
});
```

### Bulk Update

```javascript
router.patch('/bulk/status', async (req, res, next) => {
  const { ids, isActive } = req.body;

  const result = await User.updateMany(
    { _id: { $in: ids } },
    { $set: { isActive } }
  );

  res.json({ success: true, matched: result.matchedCount, modified: result.modifiedCount });
});
```

### Bulk Delete

```javascript
router.post('/bulk-delete', async (req, res, next) => {
  const { ids } = req.body;

  const result = await User.deleteMany({ _id: { $in: ids } });

  res.json({ success: true, deleted: result.deletedCount });
});
```

---

## Partial Updates

### JSON Patch (RFC 6902)

```javascript
import jsonpatch from 'fast-json-patch';

router.patch('/:id/json-patch', async (req, res, next) => {
  const user = await User.findById(req.params.id).lean();

  if (!user) {
    return res.status(404).json({ success: false, error: 'User not found' });
  }

  try {
    const patched = jsonpatch.applyPatch(user, req.body).newDocument;
    const updated = await User.findByIdAndUpdate(req.params.id, patched, { new: true });

    res.json({ success: true, data: updated });
  } catch (error) {
    res.status(400).json({ success: false, error: 'Invalid JSON Patch instructions' });
  }
});
```

### Field Masking

```javascript
router.patch('/:id', async (req, res) => {
  const allowed = ['name', 'email', 'settings.theme'];

  const filteredBody = Object.keys(req.body)
    .filter(key => allowed.includes(key))
    .reduce((obj, key) => {
      obj[key] = req.body[key];
      return obj;
    }, {});

  const user = await User.findByIdAndUpdate(req.params.id, filteredBody, { new: true });
  res.json({ success: true, data: user });
});
```

---

## Idempotency & Concurrency

- **Idempotent methods**: GET, PUT, DELETE (same request multiple times → same outcome)
- **Non-idempotent**: POST (multiple identical requests → multiple resources)

### Idempotency Key Example

```javascript
router.post('/payments', async (req, res, next) => {
  const idempotencyKey = req.headers['idempotency-key'];
  if (!idempotencyKey) {
    return res.status(400).json({ error: 'Idempotency key required' });
  }

  const existing = await Payment.findOne({ idempotencyKey });
  if (existing) {
    return res.status(200).json(existing.responseBody);
  }

  const response = await paymentService.charge(req.body);
  await Payment.create({ idempotencyKey, responseBody: response });

  res.status(201).json(response);
});
```

---

## Transaction Management

### MongoDB Transaction (Mongoose)

```javascript
import mongoose from 'mongoose';

router.post('/transfer', async (req, res, next) => {
  const session = await mongoose.startSession();
  session.startTransaction();

  try {
    const { fromUserId, toUserId, amount } = req.body;

    await Account.updateOne({ user: fromUserId }, { $inc: { balance: -amount } }, { session });
    await Account.updateOne({ user: toUserId }, { $inc: { balance: amount } }, { session });

    await session.commitTransaction();
    res.status(200).json({ success: true, message: 'Transfer complete' });
  } catch (error) {
    await session.abortTransaction();
    next(error);
  } finally {
    session.endSession();
  }
});
```

---

## Response Examples

### সফল Response Structure

```json
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": "64f9f5...",
    "name": "রহিম",
    "email": "rahim@example.com"
  }
}
```

### Error Response Structure

```json
{
  "success": false,
  "error": "Validation failed",
  "errors": [
    { "field": "email", "message": "Valid email দিন" }
  ]
}
```

---

## Swagger/OpenAPI Documentation

প্রতিটি CRUD endpoint সঠিকভাবে document করুন যাতে consumer সহজে বুঝতে পারে।

```javascript
/**
 * @swagger
 * /api/v1/users:
 *   post:
 *     summary: Create a new user
 *     tags: [Users]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/UserInput'
 *     responses:
 *       201:
 *         description: User created
 *       400:
 *         description: Validation error
 */
```

---

## End-to-End Router উদাহরণ

```javascript
import express from 'express';
import * as userController from '../controllers/userController.js';
import { createUserValidator, updateUserValidator } from '../validators/userValidator.js';

const router = express.Router();

router
  .route('/')
  .post(createUserValidator, userController.createUser)
  .get(userController.getUsers);

router
  .route('/bulk')
  .post(userController.bulkCreateUsers)
  .patch(userController.bulkUpdateUsers)
  .delete(userController.bulkDeleteUsers);

router
  .route('/:id')
  .get(userController.getUser)
  .put(updateUserValidator, userController.replaceUser)
  .patch(updateUserValidator, userController.updateUser)
  .delete(userController.deleteUser);

export default router;
```

---

## Checklist

- [x] Create operations with validation এবং duplicate checks
- [x] Read operations with pagination এবং filtering
- [x] PUT vs PATCH differences
- [x] Delete এবং soft delete strategies
- [x] Bulk operations (insert/update/delete)
- [x] Partial updates (field masking, JSON Patch)
- [x] Idempotency key handling
- [x] Transaction management
- [x] Standardized responses

---

## পরবর্তী Lesson

**Lesson 23: Advanced API Features** – Pagination, filtering, sorting, field selection, rate limiting, response formatting এবং HATEOAS সম্পর্কে আলোচনা করা হবে।

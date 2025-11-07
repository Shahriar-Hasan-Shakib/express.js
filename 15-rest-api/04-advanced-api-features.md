# Advanced REST API Features

## সংক্ষিপ্ত পরিচিতি

বেসিক CRUD এর পাশাপাশি উন্নত REST API তে pagination, filtering, sorting, field selection, rate limiting, response formatting এবং HATEOAS গুরুত্বপূর্ণ ভূমিকা পালন করে। এই Lesson এ আমরা এসব features-এর বাস্তবায়ন এবং best practices শিখব।

---

## Pagination

Pagination বৃহৎ dataset কে ছোট ছোট অংশে ভাগ করে client-কে manageable data দেয়।

### Query Parameters
- `page`: বর্তমান পেজ (default 1)
- `limit`: প্রতি পেজে কতটি item (default 10)

### Implementation Example

```javascript
router.get('/api/v1/products', async (req, res, next) => {
  const page = parseInt(req.query.page) || 1;
  const limit = Math.min(parseInt(req.query.limit) || 10, 100);
  const skip = (page - 1) * limit;

  const [products, total] = await Promise.all([
    Product.find().skip(skip).limit(limit),
    Product.countDocuments()
  ]);

  res.json({
    success: true,
    page,
    limit,
    total,
    totalPages: Math.ceil(total / limit),
    hasNextPage: page * limit < total,
    hasPrevPage: page > 1,
    data: products
  });
});
```

### Cursor-based Pagination

```javascript
router.get('/api/v1/feeds', async (req, res) => {
  const { cursor, limit = 20 } = req.query;

  const query = cursor ? { createdAt: { $lt: new Date(cursor) } } : {};

  const items = await Feed.find(query)
    .sort({ createdAt: -1 })
    .limit(parseInt(limit) + 1);

  const hasNext = items.length > limit;
  if (hasNext) items.pop();

  res.json({
    success: true,
    nextCursor: hasNext ? items[items.length - 1].createdAt.toISOString() : null,
    data: items
  });
});
```

---

## Filtering এবং Searching

### Dynamic Filter Builder

```javascript
const buildFilters = (query) => {
  const filters = {};

  if (query.status) filters.status = query.status; // exact match
  if (query.category) filters.category = { $in: query.category.split(',') };
  if (query.minPrice || query.maxPrice) {
    filters.price = {};
    if (query.minPrice) filters.price.$gte = Number(query.minPrice);
    if (query.maxPrice) filters.price.$lte = Number(query.maxPrice);
  }
  if (query.search) {
    filters.$or = [
      { name: { $regex: query.search, $options: 'i' } },
      { description: { $regex: query.search, $options: 'i' } }
    ];
  }

  return filters;
};

router.get('/api/v1/products', async (req, res, next) => {
  const filters = buildFilters(req.query);
  const products = await Product.find(filters);
  res.json({ success: true, data: products });
});
```

### Advanced Filtering Syntax

Client থেকে `filter[price][gte]=100` ধরনের query গ্রহণ করতে পারেন।

```javascript
const queryFilters = JSON.parse(
  JSON.stringify(req.query)
    .replace(/(gte|gt|lte|lt|in|ne)/g, match => `$${match}`)
);
```

---

## Sorting

### Multiple Field Sorting

```javascript
const sort = req.query.sort?.split(',').join(' ') || '-createdAt';
const products = await Product.find(filters).sort(sort);
```

Examples:
- `?sort=price` (ascending)
- `?sort=-price` (descending)
- `?sort=price,-rating` (multi-field)

### Stable Sorting

Ensure stable sorting by including a unique secondary sort key (`createdAt` + `_id`).

```javascript
.sort({ price: 1, createdAt: -1, _id: 1 });
```

---

## Field Selection (Projection)

Client কে শুধুমাত্র প্রয়োজনীয় field পাঠিয়ে bandwidth বাঁচানো যায়।

```javascript
const fields = req.query.fields?.split(',').join(' ');
const users = await User.find(filters).select(fields || '-password -__v');
```

Examples:
- `?fields=name,email` (specific fields)
- `?fields=-createdAt,-updatedAt` (exclude fields)

---

## Request Rate Limiting

Rate limiting abusive requests ব্লক করে ও API কে নিরাপদ রাখে।

### express-rate-limit

```javascript
import rateLimit from 'express-rate-limit';

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
  message: {
    success: false,
    error: 'Too many requests, please try again later.'
  }
});

app.use('/api', apiLimiter);
```

### Dynamic Limits (per user)

```javascript
const createDynamicLimiter = (getLimit) => rateLimit({
  windowMs: 60 * 1000,
  max: async (req) => getLimit(req.user),
  keyGenerator: req => req.user?.id || req.ip
});
```

---

## API Response Formatting

### Enveloping

```javascript
res.json({
  success: true,
  data: users,
  meta: {
    total,
    page,
    limit
  },
  links: {
    self: req.originalUrl,
    next: hasNext ? `/api/v1/users?page=${page + 1}` : null
  }
});
```

### Transformation Layer

```javascript
const transformUser = (user) => ({
  id: user._id,
  name: user.name,
  email: user.email,
  role: user.role,
  createdAt: user.createdAt
});

res.json({
  success: true,
  data: users.map(transformUser)
});
```

### Versioned Responses

```javascript
const responseBuilders = {
  v1: data => ({ success: true, data }),
  v2: data => ({ success: true, meta: { count: data.length }, data })
};

const version = req.headers['api-version'] || 'v1';
const response = responseBuilders[version](users);
res.json(response);
```

---

## HATEOAS Implementation

HATEOAS (Hypermedia As The Engine Of Application State) response এর মধ্যে next actions নির্দেশ করে।

```javascript
const buildUserLinks = (user) => ({
  self: { href: `/api/v1/users/${user._id}`, method: 'GET' },
  update: { href: `/api/v1/users/${user._id}`, method: 'PATCH' },
  delete: { href: `/api/v1/users/${user._id}`, method: 'DELETE' },
  posts: { href: `/api/v1/users/${user._id}/posts`, method: 'GET' }
});

router.get('/api/v1/users/:id', async (req, res, next) => {
  const user = await User.findById(req.params.id);
  if (!user) {
    return res.status(404).json({ success: false, error: 'User not found' });
  }

  res.json({
    success: true,
    data: user,
    links: buildUserLinks(user)
  });
});
```

### Collection-level HATEOAS

```javascript
res.json({
  success: true,
  data: users,
  links: {
    self: { href: `/api/v1/users?page=${page}`, method: 'GET' },
    next: hasNextPage ? { href: `/api/v1/users?page=${page + 1}`, method: 'GET' } : null,
    create: { href: '/api/v1/users', method: 'POST' }
  }
});
```

---

## Performance Considerations

- **Caching**: Use HTTP cache headers (`ETag`, `Cache-Control`) অথবা Redis.
- **Projection**: Only necessary fields fetch করে database load কমান।
- **Indexes**: Filtering/Sorting fields এর উপর index তৈরি করুন।
- **Aggregation**: `$facet`, `$match`, `$group` ইত্যাদি ব্যবহার করে complex analytics।

```javascript
const pipeline = [
  { $match: buildFilters(req.query) },
  { $sort: sortStage },
  { $skip: skip },
  { $limit: limit },
  { $project: projection }
];
```

---

## Monitoring এবং Observability

- **Metrics**: Request latency, throughput, error rate track করুন (Prometheus, StatsD)।
- **Logging**: structured logs (request id, user id, response time)।
- **Tracing**: Distributed tracing (OpenTelemetry)।

```javascript
app.use((req, res, next) => {
  const start = process.hrtime.bigint();
  res.on('finish', () => {
    const duration = Number(process.hrtime.bigint() - start) / 1e6;
    console.log(`${req.method} ${req.originalUrl} ${res.statusCode} - ${duration.toFixed(2)}ms`);
  });
  next();
});
```

---

## Checklist

- [x] Pagination (offset & cursor)
- [x] Filtering এবং searching strategy
- [x] Sorting with multiple fields
- [x] Field projection/selection
- [x] Rate limiting এবং dynamic limits
- [x] Response formatting এবং versioning
- [x] HATEOAS (resource & collection links)
- [x] Performance best practices
- [x] Monitoring ও observability hints

---

## পরবর্তী Lesson

**Lesson 24: Error Handling** – এখানে আমরা API তে ত্রুটি হ্যান্ডেল করার উন্নত প্যাটার্ন নিয়ে আলোচনা করব।

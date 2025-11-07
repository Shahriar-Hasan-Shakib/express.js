# Performance & Optimization

## সংক্ষিপ্ত পরিচিতি

Performance optimization আপনার Express application কে দ্রুত এবং scalable করে তোলে।

## 1. Compression

```bash
npm install compression
```

```javascript
const compression = require('compression');

app.use(compression({
  level: 6, // Compression level (0-9)
  threshold: 1024, // Only compress if response > 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));
```

## 2. Caching

### In-Memory Caching:

```javascript
const cache = new Map();

const cacheMiddleware = (duration) => {
  return (req, res, next) => {
    const key = req.originalUrl;
    const cached = cache.get(key);
    
    if (cached && Date.now() < cached.expiry) {
      return res.json(cached.data);
    }
    
    res.originalJson = res.json;
    res.json = (data) => {
      cache.set(key, {
        data,
        expiry: Date.now() + duration
      });
      res.originalJson(data);
    };
    
    next();
  };
};

// Usage
app.get('/api/products', cacheMiddleware(60000), async (req, res) => {
  const products = await Product.find();
  res.json(products);
});
```

### Redis Caching:

```bash
npm install redis
```

```javascript
const redis = require('redis');
const client = redis.createClient();

client.connect();

const cacheData = async (key, data, ttl = 3600) => {
  await client.setEx(key, ttl, JSON.stringify(data));
};

const getCachedData = async (key) => {
  const data = await client.get(key);
  return data ? JSON.parse(data) : null;
};

app.get('/api/users', async (req, res) => {
  const cacheKey = 'users:all';
  
  // Check cache
  const cached = await getCachedData(cacheKey);
  if (cached) {
    return res.json({ source: 'cache', data: cached });
  }
  
  // Get from database
  const users = await User.find();
  
  // Cache result
  await cacheData(cacheKey, users, 300); // 5 minutes
  
  res.json({ source: 'database', data: users });
});
```

## 3. Database Optimization

### Indexing:

```javascript
// Mongoose
const userSchema = new mongoose.Schema({
  email: { type: String, index: true, unique: true },
  name: { type: String, index: true },
  createdAt: { type: Date, index: true }
});

// Compound index
userSchema.index({ email: 1, name: 1 });
```

### Query Optimization:

```javascript
// ❌ Bad - Multiple queries
const users = await User.find();
const posts = await Post.find({ userId: users[0]._id });

// ✅ Good - Single query with populate
const users = await User.find().populate('posts');

// Select only needed fields
const users = await User.find().select('name email');

// Pagination
const page = parseInt(req.query.page) || 1;
const limit = 20;
const skip = (page - 1) * limit;

const users = await User.find()
  .limit(limit)
  .skip(skip)
  .lean(); // Return plain objects (faster)
```

## 4. Clustering

```javascript
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) {
  const numCPUs = os.cpus().length;
  
  console.log(`Master process ${process.pid} is running`);
  console.log(`Forking ${numCPUs} workers`);
  
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // Replace dead worker
  });
} else {
  // Workers share the TCP connection
  const app = express();
  
  // Your app code here
  
  app.listen(3000, () => {
    console.log(`Worker ${process.pid} started`);
  });
}
```

## 5. PM2 Process Manager

```bash
npm install -g pm2
```

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'my-app',
    script: './index.js',
    instances: 'max', // Use all CPU cores
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    max_memory_restart: '300M',
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z'
  }]
};
```

```bash
pm2 start ecosystem.config.js
pm2 list
pm2 logs
pm2 restart my-app
pm2 stop my-app
```

## 6. Connection Pooling

```javascript
// MongoDB
mongoose.connect(process.env.MONGODB_URI, {
  maxPoolSize: 10,
  minPoolSize: 5,
  socketTimeoutMS: 45000
});

// PostgreSQL
const pool = new Pool({
  max: 20, // Maximum number of clients
  min: 5,  // Minimum number of clients
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
});
```

## 7. Async Operations

```javascript
// ❌ Synchronous file operations block event loop
const data = fs.readFileSync('file.txt');

// ✅ Use async operations
const data = await fs.promises.readFile('file.txt');
```

## 8. Lazy Loading

```javascript
// ❌ Load all at startup
const heavy = require('./heavy-module');

// ✅ Load on demand
app.get('/heavy-operation', async (req, res) => {
  const heavy = require('./heavy-module');
  const result = await heavy.process();
  res.json(result);
});
```

## 9. Response Optimization

```javascript
// Streaming large files
app.get('/download', (req, res) => {
  const stream = fs.createReadStream('large-file.pdf');
  stream.pipe(res);
});

// ETags for caching
app.use((req, res, next) => {
  res.setHeader('ETag', generateETag());
  next();
});
```

## 10. Monitoring

```bash
npm install express-status-monitor
```

```javascript
const monitor = require('express-status-monitor');

app.use(monitor({
  title: 'API Status',
  path: '/status',
  spans: [{
    interval: 1,
    retention: 60
  }]
}));
```

## Performance Checklist

### ✅ কর্যন:
- Use compression
- Enable caching (Redis/Memcached)
- Database indexing
- Connection pooling
- Clustering/PM2
- Async operations
- Load balancing
- CDN for static files
- Gzip compression
- HTTP/2

### ❌ এড়িয়ে চলুন:
- Synchronous operations
- N+1 queries
- Large payloads
- Memory leaks
- Blocking event loop

## রেফারেন্স

- **PM2**: https://pm2.keymetrics.io/
- **Redis**: https://redis.io/
- **Node.js Performance**: https://nodejs.org/en/docs/guides/simple-profiling/

---

**পরবর্তী Chapter**: Deployment

# File Upload & Static Files

## সংক্ষিপ্ত পরিচিতি

File upload এবং static files serving Express applications এ commonly প্রয়োজন হয়।

## Static Files Serving

### Basic Static Files:

```javascript
const express = require('express');
const path = require('path');
const app = express();

// Static files থেকে serve করুন
app.use(express.static('public'));

// এখন public folder এর files সরাসরি access করা যাবে:
// http://localhost:3000/css/style.css
// http://localhost:3000/js/app.js
// http://localhost:3000/images/logo.png

app.listen(3000);
```

### Multiple Static Directories:

```javascript
app.use(express.static('public'));
app.use(express.static('uploads'));
app.use(express.static('assets'));
```

### Virtual Path Prefix:

```javascript
app.use('/static', express.static('public'));
// এখন: http://localhost:3000/static/css/style.css

app.use('/uploads', express.static('uploads'));
// এখন: http://localhost:3000/uploads/image.jpg
```

### Absolute Paths:

```javascript
const path = require('path');

app.use(express.static(path.join(__dirname, 'public')));
```

## File Upload with Multer

### Installation:
```bash
npm install multer
```

### Basic Upload:

```javascript
const express = require('express');
const multer = require('multer');
const path = require('path');
const app = express();

// Simple storage
const upload = multer({ dest: 'uploads/' });

// Single file upload
app.post('/upload', upload.single('file'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  
  res.json({
    message: 'File uploaded',
    file: {
      filename: req.file.filename,
      originalName: req.file.originalname,
      size: req.file.size,
      path: req.file.path
    }
  });
});

app.listen(3000);
```

### Custom Storage Configuration:

```javascript
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/'); // Destination folder
  },
  filename: (req, file, cb) => {
    // Generate unique filename
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const ext = path.extname(file.originalname);
    cb(null, file.fieldname + '-' + uniqueSuffix + ext);
  }
});

const upload = multer({ storage: storage });
```

### File Type Filtering:

```javascript
const fileFilter = (req, file, cb) => {
  // Allowed extensions
  const allowedTypes = /jpeg|jpg|png|gif|pdf/;
  const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
  const mimetype = allowedTypes.test(file.mimetype);
  
  if (extname && mimetype) {
    return cb(null, true);
  }
  
  cb(new Error('শুধুমাত্র images এবং PDF files allowed!'));
};

const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024 // 5MB limit
  }
});
```

### Multiple Files Upload:

```javascript
// Multiple files (same field)
app.post('/upload/multiple', upload.array('photos', 10), (req, res) => {
  if (!req.files || req.files.length === 0) {
    return res.status(400).json({ error: 'No files uploaded' });
  }
  
  const fileInfo = req.files.map(file => ({
    filename: file.filename,
    originalName: file.originalname,
    size: file.size,
    path: file.path
  }));
  
  res.json({
    message: `${req.files.length} files uploaded`,
    files: fileInfo
  });
});
```

### Multiple Fields:

```javascript
const uploadFields = upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'photos', maxCount: 5 },
  { name: 'documents', maxCount: 3 }
]);

app.post('/upload/fields', uploadFields, (req, res) => {
  res.json({
    avatar: req.files['avatar'],
    photos: req.files['photos'],
    documents: req.files['documents']
  });
});
```

## Complete File Upload Example

```javascript
const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');
const app = express();

app.use(express.json());
app.use('/uploads', express.static('uploads'));

// Ensure uploads directory exists
if (!fs.existsSync('uploads')) {
  fs.mkdirSync('uploads');
}

// Storage configuration
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    // Organize by file type
    let folder = 'uploads/';
    
    if (file.mimetype.startsWith('image/')) {
      folder += 'images/';
    } else if (file.mimetype === 'application/pdf') {
      folder += 'documents/';
    } else {
      folder += 'others/';
    }
    
    // Create folder if doesn't exist
    if (!fs.existsSync(folder)) {
      fs.mkdirSync(folder, { recursive: true });
    }
    
    cb(null, folder);
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const ext = path.extname(file.originalname);
    const nameWithoutExt = path.basename(file.originalname, ext);
    cb(null, nameWithoutExt + '-' + uniqueSuffix + ext);
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowedMimes = [
    'image/jpeg',
    'image/jpg',
    'image/png',
    'image/gif',
    'application/pdf'
  ];
  
  if (allowedMimes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type. Only JPEG, PNG, GIF and PDF allowed!'));
  }
};

// Multer configuration
const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: {
    fileSize: 10 * 1024 * 1024, // 10MB
    files: 5 // Max 5 files
  }
});

// Single file upload
app.post('/api/upload/single', upload.single('file'), (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }
    
    res.json({
      success: true,
      message: 'File uploaded successfully',
      file: {
        filename: req.file.filename,
        originalName: req.file.originalname,
        mimetype: req.file.mimetype,
        size: req.file.size,
        path: req.file.path,
        url: `${req.protocol}://${req.get('host')}/uploads/${req.file.filename}`
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Multiple files upload
app.post('/api/upload/multiple', upload.array('files', 5), (req, res) => {
  try {
    if (!req.files || req.files.length === 0) {
      return res.status(400).json({ error: 'No files uploaded' });
    }
    
    const filesInfo = req.files.map(file => ({
      filename: file.filename,
      originalName: file.originalname,
      size: file.size,
      url: `${req.protocol}://${req.get('host')}/${file.path.replace(/\\/g, '/')}`
    }));
    
    res.json({
      success: true,
      message: `${req.files.length} files uploaded`,
      files: filesInfo
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Delete file
app.delete('/api/files/:filename', (req, res) => {
  const filename = req.params.filename;
  
  // Search in all folders
  const folders = ['uploads/images/', 'uploads/documents/', 'uploads/others/'];
  let fileDeleted = false;
  
  for (const folder of folders) {
    const filePath = path.join(__dirname, folder, filename);
    
    if (fs.existsSync(filePath)) {
      fs.unlinkSync(filePath);
      fileDeleted = true;
      break;
    }
  }
  
  if (fileDeleted) {
    res.json({ success: true, message: 'File deleted' });
  } else {
    res.status(404).json({ error: 'File not found' });
  }
});

// Error handling middleware
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    if (err.code === 'LIMIT_FILE_SIZE') {
      return res.status(400).json({ error: 'File too large. Max 10MB allowed.' });
    }
    if (err.code === 'LIMIT_FILE_COUNT') {
      return res.status(400).json({ error: 'Too many files. Max 5 allowed.' });
    }
    if (err.code === 'LIMIT_UNEXPECTED_FILE') {
      return res.status(400).json({ error: 'Unexpected field.' });
    }
  }
  
  res.status(500).json({ error: err.message });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

## Image Upload with Processing

### Installation:
```bash
npm install sharp
```

### Image Resize & Optimization:

```javascript
const sharp = require('sharp');

app.post('/upload/image', upload.single('image'), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No image uploaded' });
    }
    
    const filename = `processed-${Date.now()}.jpg`;
    const outputPath = path.join('uploads', 'images', filename);
    
    // Process image
    await sharp(req.file.path)
      .resize(800, 600, { fit: 'inside' }) // Resize
      .jpeg({ quality: 80 }) // Convert to JPEG
      .toFile(outputPath);
    
    // Delete original
    fs.unlinkSync(req.file.path);
    
    res.json({
      success: true,
      message: 'Image processed and uploaded',
      url: `/uploads/images/${filename}`
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

## Cloud Storage (AWS S3, Cloudinary)

### Cloudinary:

```bash
npm install cloudinary multer-storage-cloudinary
```

```javascript
const cloudinary = require('cloudinary').v2;
const { CloudinaryStorage } = require('multer-storage-cloudinary');

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET
});

const storage = new CloudinaryStorage({
  cloudinary: cloudinary,
  params: {
    folder: 'express-uploads',
    allowed_formats: ['jpg', 'png', 'pdf'],
    transformation: [{ width: 500, height: 500, crop: 'limit' }]
  }
});

const upload = multer({ storage: storage });

app.post('/upload/cloud', upload.single('file'), (req, res) => {
  res.json({
    success: true,
    url: req.file.path,
    public_id: req.file.filename
  });
});
```

## Best Practices

### ✅ ১. File Size Limits
```javascript
limits: { fileSize: 5 * 1024 * 1024 } // 5MB
```

### ✅ ২. File Type Validation
```javascript
fileFilter: (req, file, cb) => {
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type'));
  }
}
```

### ✅ ৩. Unique Filenames
```javascript
filename: (req, file, cb) => {
  cb(null, Date.now() + '-' + Math.random() + '-' + file.originalname);
}
```

### ✅ ৪. Security
```javascript
// Don't trust user input
// Validate MIME types
// Scan for viruses (in production)
// Use CDN for serving files
```

### ✅ ৫. Error Handling
```javascript
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    // Handle multer errors
  }
  next(err);
});
```

## রেফারেন্স

- **Multer**: https://www.npmjs.com/package/multer
- **Sharp**: https://sharp.pixelplumbing.com/
- **Cloudinary**: https://cloudinary.com/documentation/node_integration

---

**পরবর্তী Chapter**: Security Best Practices

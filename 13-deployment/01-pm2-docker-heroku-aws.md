# Production Deployment

## সংক্ষিপ্ত পরিচিতি

Production deployment এর জন্য proper configuration এবং best practices follow করা অত্যন্ত গুরুত্বপূর্ণ।

## 1. Environment Configuration

### .env.production:
```env
NODE_ENV=production
PORT=3000
DATABASE_URL=mongodb://production-db-url
JWT_SECRET=very-strong-random-secret-key
SESSION_SECRET=another-strong-random-key
ALLOWED_ORIGINS=https://yourdomain.com
```

### config.js:
```javascript
module.exports = {
  port: process.env.PORT || 3000,
  env: process.env.NODE_ENV || 'development',
  db: {
    url: process.env.DATABASE_URL,
    options: {
      useNewUrlParser: true,
      useUnifiedTopology: true,
      maxPoolSize: 10
    }
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: '7d'
  }
};
```

## 2. PM2 Deployment

### Installation:
```bash
npm install -g pm2
```

### ecosystem.config.js:
```javascript
module.exports = {
  apps: [{
    name: 'my-app',
    script: './server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: './logs/error.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm Z',
    max_memory_restart: '500M',
    watch: false,
    merge_logs: true,
    autorestart: true
  }],
  deploy: {
    production: {
      user: 'ubuntu',
      host: 'your-server-ip',
      ref: 'origin/main',
      repo: 'git@github.com:username/repo.git',
      path: '/var/www/my-app',
      'post-deploy': 'npm install && pm2 reload ecosystem.config.js --env production'
    }
  }
};
```

### Commands:
```bash
# Start
pm2 start ecosystem.config.js --env production

# Monitor
pm2 monit

# Logs
pm2 logs

# Restart
pm2 restart my-app

# Stop
pm2 stop my-app

# Delete
pm2 delete my-app

# Startup script
pm2 startup
pm2 save
```

## 3. Docker Deployment

### Dockerfile:
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy app files
COPY . .

# Expose port
EXPOSE 3000

# Set user
USER node

# Start app
CMD ["node", "server.js"]
```

### docker-compose.yml:
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=mongodb://mongo:27017/myapp
    depends_on:
      - mongo
    restart: always
  
  mongo:
    image: mongo:6
    volumes:
      - mongo-data:/data/db
    restart: always
  
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - app
    restart: always

volumes:
  mongo-data:
```

### Commands:
```bash
# Build and start
docker-compose up -d

# View logs
docker-compose logs -f

# Stop
docker-compose down

# Rebuild
docker-compose up -d --build
```

## 4. Nginx Reverse Proxy

### nginx.conf:
```nginx
upstream app {
    least_conn;
    server app:3000;
}

server {
    listen 80;
    server_name yourdomain.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com;
    
    # SSL certificates
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    
    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
    
    # Proxy settings
    location / {
        proxy_pass http://app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
    
    # Static files
    location /uploads {
        alias /var/www/uploads;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

## 5. Heroku Deployment

### Procfile:
```
web: node server.js
```

### package.json:
```json
{
  "scripts": {
    "start": "node server.js",
    "heroku-postbuild": "npm run build"
  },
  "engines": {
    "node": "18.x",
    "npm": "9.x"
  }
}
```

### Commands:
```bash
# Login
heroku login

# Create app
heroku create my-app

# Set env vars
heroku config:set NODE_ENV=production
heroku config:set DATABASE_URL=your-db-url

# Deploy
git push heroku main

# View logs
heroku logs --tail

# Scale
heroku ps:scale web=2
```

## 6. DigitalOcean / VPS Deployment

### Setup Script:
```bash
#!/bin/bash

# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install PM2
sudo npm install -g pm2

# Install Nginx
sudo apt install -y nginx

# Setup firewall
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
sudo ufw enable

# Clone repository
git clone https://github.com/username/repo.git /var/www/my-app
cd /var/www/my-app

# Install dependencies
npm ci --only=production

# Setup environment
cp .env.example .env
nano .env  # Edit environment variables

# Start with PM2
pm2 start ecosystem.config.js --env production
pm2 startup
pm2 save

# Configure Nginx
sudo cp nginx.conf /etc/nginx/sites-available/my-app
sudo ln -s /etc/nginx/sites-available/my-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## 7. AWS Deployment (EC2)

### User Data Script:
```bash
#!/bin/bash
cd /home/ubuntu
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install 18
npm install -g pm2
git clone https://github.com/username/repo.git app
cd app
npm ci --only=production
pm2 start ecosystem.config.js
pm2 startup
pm2 save
```

## 8. SSL/TLS Certificate (Let's Encrypt)

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Get certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Auto-renewal
sudo certbot renew --dry-run
```

## 9. Monitoring & Logging

### Winston Logger:
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}
```

### Health Check Endpoint:
```javascript
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'OK',
    uptime: process.uptime(),
    timestamp: Date.now()
  });
});
```

## 10. CI/CD with GitHub Actions

### .github/workflows/deploy.yml:
```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Deploy to server
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_KEY }}
        script: |
          cd /var/www/my-app
          git pull origin main
          npm ci --only=production
          pm2 reload ecosystem.config.js
```

## Deployment Checklist

### ✅ প্রোডাকশনে Deploy করার আগে:
- [ ] Environment variables configured
- [ ] Database backups setup
- [ ] SSL/TLS certificates installed
- [ ] Monitoring setup
- [ ] Logging configured
- [ ] Error tracking (Sentry)
- [ ] Performance monitoring
- [ ] Security headers configured
- [ ] Rate limiting enabled
- [ ] CORS configured
- [ ] PM2/cluster mode enabled
- [ ] Nginx/reverse proxy setup
- [ ] Health check endpoint
- [ ] CI/CD pipeline
- [ ] Documentation updated

## Best Practices

### ✅ ১. Never commit secrets
```bash
echo ".env" >> .gitignore
```

### ✅ ২. Use process manager
```bash
pm2 start app.js -i max
```

### ✅ ৩. Enable HTTPS
```bash
certbot --nginx -d yourdomain.com
```

### ✅ ৪. Monitor application
```bash
pm2 monit
pm2 logs
```

### ✅ ৫. Backup database regularly
```bash
mongodump --uri="mongodb://..." --out=/backups/
```

## রেফারেন্স

- **PM2**: https://pm2.keymetrics.io/
- **Docker**: https://docs.docker.com/
- **Nginx**: https://nginx.org/en/docs/
- **Let's Encrypt**: https://letsencrypt.org/
- **Heroku**: https://devcenter.heroku.com/

---

**সম্পূর্ণ ডকুমেন্টেশন সমাপ্ত!** 🎉

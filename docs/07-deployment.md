# WMS-Lite 部署运维文档

## 一、部署概述

### 1.1 部署方式

| 方式 | 适用场景 | 优点 | 缺点 |
|------|----------|------|------|
| **源码部署** | 开发/生产环境 | 灵活、可调试、易于定制 | 需要配置环境 |
| **Docker 部署** | 生产环境（可选） | 环境一致、易于迁移 | 需要 Docker 环境 |
| **单文件部署** | 小型部署 | 简单、开箱即用 | 打包体积大 |

### 1.2 环境要求

#### 开发环境

| 软件 | 最低版本 | 推荐版本 |
|------|----------|----------|
| Node.js | 18.0 | 20.x LTS |
| MySQL | 5.7 | 8.0 |
| Git | 2.0 | 最新版 |

#### 生产环境

| 软件 | 最低版本 | 推荐版本 |
|------|----------|----------|
| Node.js | 18.0 | 20.x LTS |
| MySQL | 5.7 | 8.0 |
| Docker (可选) | 20.0 | 24.x |
| Nginx | 1.20 | 1.24 |
| Redis (可选) | 6.0 | 7.0 |

---

## 二、源码部署（推荐）

### 2.1 环境准备

```bash
# 安装 Node.js (Ubuntu)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# 安装 Node.js (CentOS)
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
sudo yum install -y nodejs

# 安装 MySQL (Ubuntu)
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation

# 安装 MySQL (CentOS)
sudo yum install mysql-server
sudo systemctl start mysqld
sudo mysql_secure_installation

# 验证安装
node -v
npm -v
mysql --version
```

### 2.2 项目部署

```bash
# 1. 克隆项目
git clone https://github.com/your-org/wms-lite.git
cd wms-lite

# 2. 安装依赖
npm install

# 3. 配置环境变量
cp .env.example .env
# 编辑 .env 文件，配置 MySQL 连接参数
```

### 2.3 环境变量配置

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑 .env 文件，配置 MySQL 连接参数
# 根据实际环境修改配置值
```

详细的 `.env.example` 模板文件已创建在项目根目录，包含以下配置项：

| 配置分类 | 配置项 | 说明 |
|---------|---------|------|
| 应用配置 | NODE_ENV, PORT, APP_NAME | 运行环境、端口、应用名称 |
| 数据库配置 | DATABASE_URL, DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD | MySQL 数据库连接参数 |
| JWT 认证 | JWT_SECRET, JWT_EXPIRES_IN | JWT 密钥和过期时间 |
| 日志配置 | LOG_LEVEL, LOG_DIR, LOG_MAX_SIZE, LOG_MAX_FILES | 日志级别、目录和文件配置 |
| 文件上传 | UPLOAD_DIR, MAX_FILE_SIZE, ALLOWED_FILE_TYPES | 上传目录、大小限制、允许类型 |
| CORS 配置 | CORS_ORIGIN, CORS_METHODS, CORS_HEADERS | 跨域配置 |
| 速率限制 | RATE_LIMIT_MAX, RATE_LIMIT_WINDOW | API 速率限制 |
| 打印配置 | DEFAULT_PRINTER_NAME, PRINT_SERVICE_TYPE | 打印机和服务类型 |
| 条码配置 | BARCODE_SERVICE_URL, BARCODE_CACHE_DIR | 条码服务和缓存目录 |
| 邮件配置 | SMTP_HOST, SMTP_PORT, SMTP_USER, SMTP_PASS, SMTP_FROM | SMTP 邮件服务配置 |
| Redis 配置 | REDIS_URL, REDIS_PASSWORD | Redis 缓存配置 |
| 系统配置 | TZ, DEFAULT_LANGUAGE, ORDER_PREFIX_* | 时区、语言、单据前缀 |
| 安全配置 | ENABLE_HTTPS, SSL_CERT_PATH, SSL_KEY_PATH, SESSION_SECRET | HTTPS 和会话安全 |
| 监控配置 | ENABLE_METRICS, METRICS_PORT | Prometheus 监控配置 |
| 备份配置 | BACKUP_DIR, BACKUP_RETENTION_DAYS | 备份目录和保留天数 |
| 开发配置 | ENABLE_HOT_RELOAD, DEBUG, ENABLE_PRISMA_STUDIO | 开发环境专用配置 |

### 2.4 环境变量配置（示例）

```bash
# .env.example
# 应用配置
NODE_ENV=production
PORT=3000
APP_NAME=WMS-Lite

# 数据库配置 (MySQL)
DATABASE_URL="mysql://wms:wms123@localhost:3306/wms_lite"
DB_HOST=localhost
DB_PORT=3306
DB_NAME=wms_lite
DB_USER=wms
DB_PASSWORD=wms123

# JWT 配置
JWT_SECRET=your-super-secret-key-change-in-production
JWT_EXPIRES_IN=7d

# 日志配置
LOG_LEVEL=info
LOG_DIR=./logs

# 文件上传配置
UPLOAD_DIR=./uploads
MAX_FILE_SIZE=10485760
ALLOWED_FILE_TYPES=.jpg,.jpeg,.png,.gif,.pdf,.xlsx,.xls,.csv

# CORS 配置
CORS_ORIGIN=*
CORS_METHODS=GET,POST,PUT,DELETE,OPTIONS
CORS_HEADERS=Content-Type,Authorization

# 速率限制配置
RATE_LIMIT_MAX=100
RATE_LIMIT_WINDOW=900000

# 打印配置
DEFAULT_PRINTER_NAME=Microsoft Print to PDF
PRINT_SERVICE_TYPE=browser

# 条码配置
BARCODE_CACHE_DIR=./cache/barcode

# 系统配置
TZ=Asia/Shanghai
DEFAULT_LANGUAGE=zh-CN
ORDER_PREFIX_INBOUND=RK
ORDER_PREFIX_OUTBOUND=CK
ORDER_PREFIX_INVENTORY=PD
ORDER_PREFIX_MOVE=YK

# 安全配置
ENABLE_HTTPS=false
SESSION_SECRET=your-session-secret-key

# 备份配置
BACKUP_DIR=./backup
BACKUP_RETENTION_DAYS=30

# 开发环境专用配置
ENABLE_HOT_RELOAD=true
DEBUG=false
ENABLE_PRISMA_STUDIO=true
PRISMA_STUDIO_PORT=5555
```

### 2.4 数据库初始化

```bash
# 创建数据库
mysql -u root -p -e "CREATE DATABASE wms_lite CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 创建用户
mysql -u root -p -e "CREATE USER 'wms'@'localhost' IDENTIFIED BY 'wms123';"
mysql -u root -p -e "GRANT ALL PRIVILEGES ON wms_lite.* TO 'wms'@'localhost';"
mysql -u root -p -e "FLUSH PRIVILEGES;"

# 初始化数据库
cd backend
npx prisma generate
npx prisma migrate deploy
npx prisma db seed
```

### 2.5 构建与启动

```bash
# 构建前端
cd frontend
npm run build

# 构建后端
cd ../backend
npm run build

# 启动服务
npm start
```

### 2.6 PM2 部署

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'wms-lite',
      script: 'backend/server.js',
      instances: 'max',
      exec_mode: 'cluster',
      autorestart: true,
      watch: false,
      max_memory_restart: '1G',
      env: {
        NODE_ENV: 'production',
        PORT: 3000
      },
      env_development: {
        NODE_ENV: 'development',
        PORT: 3000
      },
      error_file: './logs/pm2-error.log',
      out_file: './logs/pm2-out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z'
    }
  ]
}
```

```bash
# 安装 PM2
npm install -g pm2

# 启动服务
pm2 start ecosystem.config.js

# 查看状态
pm2 status

# 查看日志
pm2 logs wms-lite

# 重启服务
pm2 restart wms-lite

# 停止服务
pm2 stop wms-lite

# 开机自启动
pm2 startup
pm2 save
```

---

## 三、Docker 部署（可选）

> Docker 部署可作为后续扩展方案，当前推荐使用源码部署。

### 3.1 Dockerfile

```dockerfile
# docker/Dockerfile

# 构建阶段
FROM node:20-alpine AS builder

WORKDIR /app

# 复制 package 文件
COPY package*.json ./
COPY frontend/package*.json ./frontend/
COPY backend/package*.json ./backend/

# 安装依赖
RUN npm ci --legacy-peer-deps

# 复制源码
COPY . .

# 构建前端
WORKDIR /app/frontend
RUN npm run build

# 构建后端
WORKDIR /app/backend
RUN npm run build

# 生产阶段
FROM node:20-alpine AS production

WORKDIR /app

# 复制构建产物
COPY --from=builder /app/backend/dist ./backend
COPY --from=builder /app/backend/node_modules ./backend/node_modules
COPY --from=builder /app/backend/package.json ./backend/
COPY --from=builder /app/frontend/dist ./frontend

# 复制启动脚本
COPY docker/start.sh /start.sh
RUN chmod +x /start.sh

# 暴露端口
EXPOSE 3000

# 启动服务
CMD ["/start.sh"]
```

### 3.2 Docker Compose

```yaml
# docker/docker-compose.yml

version: '3.8'

services:
  # 应用服务
  wms-app:
    build:
      context: ..
      dockerfile: docker/Dockerfile
    container_name: wms-lite
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=file:/app/data/wms.db
      - JWT_SECRET=${JWT_SECRET:-your-secret-key}
      - JWT_EXPIRES_IN=7d
    volumes:
      - ./data:/app/data
      - ./uploads:/app/uploads
      - ./logs:/app/logs
    networks:
      - wms-network
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # MySQL 服务（可选）
  mysql:
    image: mysql:8.0
    container_name: wms-mysql
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD:-root123}
      - MYSQL_DATABASE=wms_lite
      - MYSQL_USER=wms
      - MYSQL_PASSWORD=${MYSQL_PASSWORD:-wms123}
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - wms-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis 服务（可选）
  redis:
    image: redis:7-alpine
    container_name: wms-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - wms-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Nginx 反向代理（可选）
  nginx:
    image: nginx:alpine
    container_name: wms-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - wms-app
    networks:
      - wms-network

networks:
  wms-network:
    driver: bridge

volumes:
  mysql-data:
  redis-data:
```

### 3.3 启动脚本

```bash
#!/bin/bash
# docker/start.sh

set -e

# 初始化数据库
if [ ! -f /app/data/wms.db ]; then
  echo "Initializing database..."
  cd /app/backend
  npx prisma migrate deploy
  npx prisma db seed
fi

# 启动服务
echo "Starting WMS-Lite..."
cd /app/backend
node server.js
```

### 3.4 部署命令

```bash
# 构建镜像
docker-compose build

# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f wms-app

# 停止服务
docker-compose down

# 重启服务
docker-compose restart wms-app

# 进入容器
docker exec -it wms-lite sh

# 数据备份
docker exec wms-lite sqlite3 /app/data/wms.db ".backup /app/data/backup.db"
docker cp wms-lite:/app/data/backup.db ./backup_$(date +%Y%m%d).db
```

---

## 四、Nginx 配置

### 4.1 基础配置

```nginx
# /etc/nginx/nginx.conf

user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 65;
  types_hash_max_size 2048;

  # Gzip 压缩
  gzip on;
  gzip_vary on;
  gzip_proxied any;
  gzip_comp_level 6;
  gzip_types text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;

  # 包含站点配置
  include /etc/nginx/sites-enabled/*;
}
```

### 4.2 站点配置

```nginx
# /etc/nginx/sites-available/wms-lite

server {
  listen 80;
  server_name your-domain.com;

  # 重定向到 HTTPS
  return 301 https://$server_name$request_uri;
}

server {
  listen 443 ssl http2;
  server_name your-domain.com;

  # SSL 证书
  ssl_certificate /etc/nginx/ssl/cert.pem;
  ssl_certificate_key /etc/nginx/ssl/key.pem;
  ssl_session_timeout 1d;
  ssl_session_cache shared:SSL:50m;
  ssl_session_tickets off;

  # SSL 配置
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
  ssl_prefer_server_ciphers off;

  # HSTS
  add_header Strict-Transport-Security "max-age=63072000" always;

  # 安全头
  add_header X-Frame-Options "SAMEORIGIN" always;
  add_header X-Content-Type-Options "nosniff" always;
  add_header X-XSS-Protection "1; mode=block" always;

  # 静态文件
  root /var/www/wms-lite/frontend/dist;
  index index.html;

  # 前端路由
  location / {
    try_files $uri $uri/ /index.html;
    
    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
      expires 1y;
      add_header Cache-Control "public, immutable";
    }
  }

  # API 代理
  location /api {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_cache_bypass $http_upgrade;
    
    # 超时设置
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
  }

  # WebSocket 代理
  location /ws {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # 上传文件
  location /uploads {
    alias /var/www/wms-lite/uploads;
    expires 30d;
    add_header Cache-Control "public";
  }

  # 日志
  access_log /var/log/nginx/wms-access.log;
  error_log /var/log/nginx/wms-error.log;
}
```

### 4.3 启用配置

```bash
# 创建软链接
sudo ln -s /etc/nginx/sites-available/wms-lite /etc/nginx/sites-enabled/

# 测试配置
sudo nginx -t

# 重载配置
sudo nginx -s reload
```

---

## 五、数据库运维

### 5.1 MySQL 运维

```bash
# 创建数据库
mysql -u root -p -e "CREATE DATABASE wms_lite CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 创建用户
mysql -u root -p -e "CREATE USER 'wms'@'%' IDENTIFIED BY 'password';"
mysql -u root -p -e "GRANT ALL PRIVILEGES ON wms_lite.* TO 'wms'@'%';"
mysql -u root -p -e "FLUSH PRIVILEGES;"

# 数据库备份
mysqldump -u wms -p wms_lite > backup_$(date +%Y%m%d).sql

# 数据库恢复
mysql -u wms -p wms_lite < backup.sql

# 定时备份 (crontab)
0 2 * * * mysqldump -u wms -ppassword wms_lite > /backup/wms_$(date +\%Y\%m\%d).sql

# 查看数据库状态
mysql -u root -p -e "SHOW STATUS LIKE 'Threads_connected';"
mysql -u root -p -e "SHOW PROCESSLIST;"

# 优化表
mysql -u root -p wms_lite -e "OPTIMIZE TABLE product, stock, inbound_order, outbound_order;"
```

### 5.2 Prisma 迁移

```bash
# 创建迁移
npx prisma migrate dev --name migration_name

# 应用迁移
npx prisma migrate deploy

# 重置数据库
npx prisma migrate reset

# 生成客户端
npx prisma generate

# 查看数据
npx prisma studio
```

### 5.3 SQLite 运维（可选）

> SQLite 仅适用于开发测试或小型部署场景。

```bash
# 数据库备份
sqlite3 data/wms.db ".backup data/backup_$(date +%Y%m%d).db"

# 数据库压缩
sqlite3 data/wms.db "VACUUM;"

# 数据库完整性检查
sqlite3 data/wms.db "PRAGMA integrity_check;"

# 导出 SQL
sqlite3 data/wms.db .dump > backup.sql

# 导入 SQL
sqlite3 data/wms.db < backup.sql
```

---

## 六、监控与日志

### 6.1 日志配置

```javascript
// src/utils/logger.js
const winston = require('winston')
const path = require('path')

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: 'wms-lite' },
  transports: [
    // 错误日志
    new winston.transports.File({
      filename: path.join(__dirname, '../../logs/error.log'),
      level: 'error',
      maxsize: 5242880, // 5MB
      maxFiles: 5
    }),
    // 全部日志
    new winston.transports.File({
      filename: path.join(__dirname, '../../logs/combined.log'),
      maxsize: 5242880,
      maxFiles: 5
    })
  ]
})

// 开发环境输出到控制台
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    )
  }))
}

module.exports = logger
```

### 6.2 健康检查

```javascript
// src/routes/health.routes.js
const express = require('express')
const router = express.Router()
const { PrismaClient } = require('@prisma/client')
const prisma = new PrismaClient()

router.get('/health', async (req, res) => {
  const health = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    checks: {
      database: 'unknown',
      memory: 'unknown'
    }
  }

  // 检查数据库连接
  try {
    await prisma.$queryRaw`SELECT 1`
    health.checks.database = 'ok'
  } catch (error) {
    health.checks.database = 'error'
    health.status = 'degraded'
  }

  // 检查内存使用
  const memoryUsage = process.memoryUsage()
  health.checks.memory = {
    heapUsed: Math.round(memoryUsage.heapUsed / 1024 / 1024) + 'MB',
    heapTotal: Math.round(memoryUsage.heapTotal / 1024 / 1024) + 'MB',
    rss: Math.round(memoryUsage.rss / 1024 / 1024) + 'MB'
  }

  const statusCode = health.status === 'ok' ? 200 : 503
  res.status(statusCode).json(health)
})

module.exports = router
```

### 6.3 Prometheus 监控

```javascript
// src/middlewares/metrics.middleware.js
const promClient = require('prom-client')

// 创建 Registry
const register = new promClient.Registry()

// 默认指标
promClient.collectDefaultMetrics({ register })

// 自定义指标
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
})

const httpRequestTotal = new promClient.Counter({
  name: 'http_request_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
})

register.registerMetric(httpRequestDuration)
register.registerMetric(httpRequestTotal)

// 中间件
function metricsMiddleware(req, res, next) {
  const start = Date.now()

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000
    const route = req.route ? req.route.path : req.path

    httpRequestDuration.labels(req.method, route, res.statusCode).observe(duration)
    httpRequestTotal.labels(req.method, route, res.statusCode).inc()
  })

  next()
}

module.exports = { register, metricsMiddleware }
```

---

## 七、备份策略

### 7.1 备份脚本

```bash
#!/bin/bash
# scripts/backup.sh

# 配置
BACKUP_DIR="/backup/wms-lite"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# 创建备份目录
mkdir -p $BACKUP_DIR

# 备份数据库
echo "Backing up database..."
sqlite3 data/wms.db ".backup $BACKUP_DIR/db_$DATE.db"

# 备份上传文件
echo "Backing up uploads..."
tar -czf $BACKUP_DIR/uploads_$DATE.tar.gz uploads/

# 备份配置文件
echo "Backing up config..."
tar -czf $BACKUP_DIR/config_$DATE.tar.gz .env

# 清理旧备份
echo "Cleaning old backups..."
find $BACKUP_DIR -type f -mtime +$RETENTION_DAYS -delete

echo "Backup completed: $DATE"
```

### 7.2 定时备份

```bash
# 编辑 crontab
crontab -e

# 每天凌晨 2 点执行备份
0 2 * * * /path/to/scripts/backup.sh >> /var/log/wms-backup.log 2>&1
```

---

## 八、故障排查

### 8.1 常见问题

#### 服务无法启动

```bash
# 检查端口占用
lsof -i :3000
netstat -tlnp | grep 3000

# 检查进程
ps aux | grep node

# 查看日志
tail -f logs/error.log
pm2 logs wms-lite
```

#### 数据库连接失败

```bash
# 检查数据库文件权限
ls -la data/wms.db

# 检查数据库完整性
sqlite3 data/wms.db "PRAGMA integrity_check;"

# 检查 MySQL 连接
mysql -u wms -p -h localhost wms_lite -e "SELECT 1;"
```

#### 内存不足

```bash
# 查看内存使用
free -h

# 查看进程内存
ps aux --sort=-%mem | head -10

# 重启服务
pm2 restart wms-lite
```

### 8.2 日志分析

```bash
# 查看错误日志
grep "error" logs/combined.log | tail -50

# 统计错误类型
grep "error" logs/combined.log | awk '{print $5}' | sort | uniq -c

# 查看慢请求
grep "duration" logs/combined.log | awk -F'duration":' '{print $2}' | awk '$1 > 1000'
```

### 8.3 性能优化

```bash
# Node.js 性能分析
node --prof backend/server.js
node --prof-process isolate-*.log > profile.txt

# 内存分析
node --inspect backend/server.js
# 使用 Chrome DevTools 进行内存分析
```

---

## 九、安全加固

### 9.1 系统安全

```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 配置防火墙
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

# 禁用 root 登录
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### 9.2 应用安全

```bash
# 设置文件权限
chmod 600 .env
chmod 644 data/wms.db

# 创建专用用户
sudo useradd -r -s /bin/false wms
sudo chown -R wms:wms /var/www/wms-lite
```

### 9.3 SSL 证书

```bash
# 安装 Certbot
sudo apt install certbot python3-certbot-nginx

# 获取证书
sudo certbot --nginx -d your-domain.com

# 自动续期
sudo certbot renew --dry-run
```

---

## 十、升级指南

### 10.1 版本升级

```bash
# 1. 备份数据
./scripts/backup.sh

# 2. 拉取最新代码
git fetch origin
git checkout v1.1.0

# 3. 安装依赖
npm install

# 4. 执行数据库迁移
cd backend
npx prisma migrate deploy

# 5. 重新构建
npm run build

# 6. 重启服务
pm2 restart wms-lite
```

### 10.2 回滚操作

```bash
# 1. 停止服务
pm2 stop wms-lite

# 2. 切换到旧版本
git checkout v1.0.0

# 3. 恢复数据库
cp /backup/wms-lite/db_xxx.db data/wms.db

# 4. 重启服务
pm2 start wms-lite
```

# Deployment & Configuration Guide

## Overview

This guide covers deployment strategies, environment configuration, security best practices, and production monitoring for the Charging Station API and related applications.

## Table of Contents

- [Environment Setup](#environment-setup)
- [Configuration Management](#configuration-management)
- [Security Configuration](#security-configuration)
- [Deployment Strategies](#deployment-strategies)
- [Monitoring & Logging](#monitoring--logging)
- [Performance Optimization](#performance-optimization)
- [Backup & Recovery](#backup--recovery)
- [Troubleshooting](#troubleshooting)

---

## Environment Setup

### Development Environment

```bash
# Clone repository
git clone https://github.com/laadpaalnodig/api-client.git
cd api-client

# Install dependencies
npm install

# Copy environment template
cp .env.example .env.development

# Configure development environment
cat << EOF > .env.development
NODE_ENV=development
API_KEY=dev_api_key_12345
API_BASE_URL=https://staging-api.laadpaalnodig.nl/v1
API_TIMEOUT=30000
LOG_LEVEL=debug
ENABLE_CORS=true
DATABASE_URL=postgresql://dev_user:dev_pass@localhost:5432/charging_dev
REDIS_URL=redis://localhost:6379
EOF

# Start development server
npm run dev
```

### Staging Environment

```yaml
# docker-compose.staging.yml
version: '3.8'

services:
  api:
    build: .
    environment:
      - NODE_ENV=staging
      - API_KEY=${STAGING_API_KEY}
      - API_BASE_URL=https://staging-api.laadpaalnodig.nl/v1
      - DATABASE_URL=${STAGING_DATABASE_URL}
      - REDIS_URL=${STAGING_REDIS_URL}
      - LOG_LEVEL=info
    ports:
      - "3000:3000"
    depends_on:
      - database
      - redis
    restart: unless-stopped

  database:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=charging_staging
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - api
    restart: unless-stopped

volumes:
  postgres_data:
```

### Production Environment

```yaml
# docker-compose.production.yml
version: '3.8'

services:
  api:
    image: laadpaalnodig/api:${VERSION}
    environment:
      - NODE_ENV=production
      - API_KEY=${PRODUCTION_API_KEY}
      - API_BASE_URL=https://api.laadpaalnodig.nl/v1
      - DATABASE_URL=${PRODUCTION_DATABASE_URL}
      - REDIS_URL=${PRODUCTION_REDIS_URL}
      - LOG_LEVEL=warn
      - ENABLE_MONITORING=true
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
    depends_on:
      - database
      - redis

  database:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=charging_production
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backups:/backups
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    deploy:
      replicas: 1

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.production.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    deploy:
      replicas: 2

  monitoring:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

volumes:
  postgres_data:
  redis_data:
```

---

## Configuration Management

### Environment Variables

```bash
# .env.production
# Application
NODE_ENV=production
PORT=3000
APP_VERSION=1.0.0

# API Configuration
API_KEY=prod_api_key_secure_12345
API_BASE_URL=https://api.laadpaalnodig.nl/v1
API_TIMEOUT=30000
API_RATE_LIMIT=1000
API_RATE_WINDOW=3600

# Database
DATABASE_URL=postgresql://prod_user:secure_password@db-cluster:5432/charging_prod
DATABASE_POOL_MIN=5
DATABASE_POOL_MAX=20
DATABASE_SSL=true

# Redis
REDIS_URL=redis://redis-cluster:6379
REDIS_PASSWORD=secure_redis_password
REDIS_DB=0
REDIS_KEY_PREFIX=charging:

# Security
JWT_SECRET=very_secure_jwt_secret_key_here
API_SECRET_KEY=super_secure_api_secret_key
ENCRYPTION_KEY=32_character_encryption_key_here
CORS_ORIGINS=https://app.laadpaalnodig.nl,https://admin.laadpaalnodig.nl

# External Services
PAYMENT_API_KEY=secure_payment_api_key
EMAIL_API_KEY=secure_email_api_key
SMS_API_KEY=secure_sms_api_key

# Monitoring
LOG_LEVEL=info
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project
DATADOG_API_KEY=your_datadog_api_key
NEWRELIC_LICENSE_KEY=your_newrelic_license

# File Storage
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_REGION=eu-west-1
S3_BUCKET=charging-station-documents

# Webhooks
WEBHOOK_SECRET=secure_webhook_secret_key
WEBHOOK_TIMEOUT=10000
```

### Configuration Classes

```javascript
// config/index.js
const convict = require('convict');
const convictFormatWithValidator = require('convict-format-with-validator');

convict.addFormats(convictFormatWithValidator);

const config = convict({
  env: {
    doc: 'The application environment',
    format: ['production', 'development', 'staging', 'test'],
    default: 'development',
    env: 'NODE_ENV'
  },
  port: {
    doc: 'The port to bind',
    format: 'port',
    default: 3000,
    env: 'PORT'
  },
  api: {
    key: {
      doc: 'API key for external services',
      format: String,
      default: '',
      env: 'API_KEY',
      sensitive: true
    },
    baseUrl: {
      doc: 'Base URL for API',
      format: 'url',
      default: 'https://api.laadpaalnodig.nl/v1',
      env: 'API_BASE_URL'
    },
    timeout: {
      doc: 'API request timeout in milliseconds',
      format: 'int',
      default: 30000,
      env: 'API_TIMEOUT'
    },
    rateLimit: {
      requests: {
        doc: 'Number of requests allowed per window',
        format: 'int',
        default: 1000,
        env: 'API_RATE_LIMIT'
      },
      window: {
        doc: 'Rate limit window in seconds',
        format: 'int',
        default: 3600,
        env: 'API_RATE_WINDOW'
      }
    }
  },
  database: {
    url: {
      doc: 'Database connection URL',
      format: String,
      default: 'postgresql://localhost:5432/charging',
      env: 'DATABASE_URL',
      sensitive: true
    },
    pool: {
      min: {
        doc: 'Minimum pool size',
        format: 'int',
        default: 5,
        env: 'DATABASE_POOL_MIN'
      },
      max: {
        doc: 'Maximum pool size',
        format: 'int',
        default: 20,
        env: 'DATABASE_POOL_MAX'
      }
    },
    ssl: {
      doc: 'Enable SSL for database connection',
      format: Boolean,
      default: false,
      env: 'DATABASE_SSL'
    }
  },
  redis: {
    url: {
      doc: 'Redis connection URL',
      format: String,
      default: 'redis://localhost:6379',
      env: 'REDIS_URL'
    },
    password: {
      doc: 'Redis password',
      format: String,
      default: '',
      env: 'REDIS_PASSWORD',
      sensitive: true
    },
    keyPrefix: {
      doc: 'Redis key prefix',
      format: String,
      default: 'charging:',
      env: 'REDIS_KEY_PREFIX'
    }
  },
  security: {
    jwtSecret: {
      doc: 'JWT signing secret',
      format: String,
      default: '',
      env: 'JWT_SECRET',
      sensitive: true
    },
    apiSecretKey: {
      doc: 'API secret key',
      format: String,
      default: '',
      env: 'API_SECRET_KEY',
      sensitive: true
    },
    corsOrigins: {
      doc: 'Allowed CORS origins',
      format: Array,
      default: [],
      env: 'CORS_ORIGINS'
    }
  },
  monitoring: {
    logLevel: {
      doc: 'Logging level',
      format: ['debug', 'info', 'warn', 'error'],
      default: 'info',
      env: 'LOG_LEVEL'
    },
    sentryDsn: {
      doc: 'Sentry DSN for error tracking',
      format: String,
      default: '',
      env: 'SENTRY_DSN',
      sensitive: true
    },
    datadogApiKey: {
      doc: 'Datadog API key',
      format: String,
      default: '',
      env: 'DATADOG_API_KEY',
      sensitive: true
    }
  }
});

// Validate configuration
config.validate({ allowed: 'strict' });

module.exports = config;
```

---

## Security Configuration

### SSL/TLS Setup

```nginx
# nginx.production.conf
server {
    listen 80;
    server_name api.laadpaalnodig.nl;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.laadpaalnodig.nl;

    # SSL Configuration
    ssl_certificate /etc/nginx/ssl/laadpaalnodig.crt;
    ssl_certificate_key /etc/nginx/ssl/laadpaalnodig.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options DENY always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # Rate Limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req zone=api burst=20 nodelay;

    # Proxy Configuration
    location / {
        proxy_pass http://api:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 300s;
        proxy_connect_timeout 75s;
    }

    # Health Check
    location /health {
        access_log off;
        proxy_pass http://api:3000/health;
    }
}
```

### API Key Management

```javascript
// middleware/auth.js
const crypto = require('crypto');
const rateLimit = require('express-rate-limit');
const { config } = require('../config');

class ApiKeyManager {
  constructor() {
    this.keys = new Map();
    this.loadApiKeys();
  }

  loadApiKeys() {
    // Load API keys from secure storage
    const keys = [
      {
        id: 'client_001',
        key: 'ak_live_123456789abcdef',
        name: 'Production Client',
        permissions: ['read', 'write'],
        rateLimit: { requests: 1000, window: 3600 }
      },
      {
        id: 'client_002',
        key: 'ak_test_987654321fedcba',
        name: 'Test Client',
        permissions: ['read'],
        rateLimit: { requests: 100, window: 3600 }
      }
    ];

    keys.forEach(keyData => {
      this.keys.set(keyData.key, keyData);
    });
  }

  validateKey(apiKey) {
    return this.keys.get(apiKey);
  }

  hasPermission(keyData, permission) {
    return keyData.permissions.includes(permission);
  }
}

const keyManager = new ApiKeyManager();

// API Key Authentication Middleware
const authenticateApiKey = (req, res, next) => {
  const apiKey = req.headers.authorization?.replace('Bearer ', '') || 
                 req.query.api_key;

  if (!apiKey) {
    return res.status(401).json({
      success: false,
      errors: [{ code: 'UNAUTHORIZED', message: 'API key required' }]
    });
  }

  const keyData = keyManager.validateKey(apiKey);
  
  if (!keyData) {
    return res.status(401).json({
      success: false,
      errors: [{ code: 'INVALID_API_KEY', message: 'Invalid API key' }]
    });
  }

  // Attach key data to request
  req.apiKey = keyData;
  next();
};

// Permission-based authorization
const requirePermission = (permission) => {
  return (req, res, next) => {
    if (!keyManager.hasPermission(req.apiKey, permission)) {
      return res.status(403).json({
        success: false,
        errors: [{ code: 'FORBIDDEN', message: 'Insufficient permissions' }]
      });
    }
    next();
  };
};

// Dynamic rate limiting based on API key
const createRateLimiter = () => {
  return rateLimit({
    windowMs: (req) => (req.apiKey?.rateLimit.window || 3600) * 1000,
    max: (req) => req.apiKey?.rateLimit.requests || 100,
    keyGenerator: (req) => req.apiKey?.id || req.ip,
    message: {
      success: false,
      errors: [{ code: 'RATE_LIMITED', message: 'Rate limit exceeded' }]
    }
  });
};

module.exports = {
  authenticateApiKey,
  requirePermission,
  createRateLimiter
};
```

### Data Encryption

```javascript
// utils/encryption.js
const crypto = require('crypto');
const { config } = require('../config');

class EncryptionService {
  constructor() {
    this.algorithm = 'aes-256-gcm';
    this.keyLength = 32;
    this.ivLength = 16;
    this.tagLength = 16;
    this.key = Buffer.from(config.get('security.encryptionKey'), 'hex');
  }

  encrypt(text) {
    const iv = crypto.randomBytes(this.ivLength);
    const cipher = crypto.createCipher(this.algorithm, this.key);
    cipher.setAAD(Buffer.from('charging-api', 'utf8'));

    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const tag = cipher.getAuthTag();
    
    return {
      encrypted,
      iv: iv.toString('hex'),
      tag: tag.toString('hex')
    };
  }

  decrypt(encryptedData) {
    const { encrypted, iv, tag } = encryptedData;
    
    const decipher = crypto.createDecipher(this.algorithm, this.key);
    decipher.setAAD(Buffer.from('charging-api', 'utf8'));
    decipher.setAuthTag(Buffer.from(tag, 'hex'));

    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }

  hashPassword(password, salt) {
    if (!salt) {
      salt = crypto.randomBytes(16).toString('hex');
    }
    
    const hash = crypto.pbkdf2Sync(password, salt, 10000, 64, 'sha512');
    return {
      hash: hash.toString('hex'),
      salt
    };
  }

  verifyPassword(password, hash, salt) {
    const { hash: newHash } = this.hashPassword(password, salt);
    return newHash === hash;
  }
}

module.exports = new EncryptionService();
```

---

## Deployment Strategies

### Blue-Green Deployment

```bash
#!/bin/bash
# deploy.sh - Blue-Green deployment script

set -e

ENVIRONMENT=${1:-staging}
VERSION=${2:-latest}

echo "Starting Blue-Green deployment for $ENVIRONMENT environment"

# Configuration
BLUE_SERVICE="charging-api-blue"
GREEN_SERVICE="charging-api-green"
NGINX_CONFIG="/etc/nginx/sites-available/charging-api"

# Determine current active service
CURRENT_ACTIVE=$(docker ps --filter "name=charging-api" --filter "status=running" --format "{{.Names}}" | head -n1)

if [[ $CURRENT_ACTIVE == *"blue"* ]]; then
    ACTIVE_SERVICE=$BLUE_SERVICE
    NEW_SERVICE=$GREEN_SERVICE
    ACTIVE_COLOR="blue"
    NEW_COLOR="green"
else
    ACTIVE_SERVICE=$GREEN_SERVICE
    NEW_SERVICE=$BLUE_SERVICE
    ACTIVE_COLOR="green"
    NEW_COLOR="blue"
fi

echo "Current active service: $ACTIVE_COLOR"
echo "Deploying to: $NEW_COLOR"

# Stop and remove existing new service
docker stop $NEW_SERVICE 2>/dev/null || true
docker rm $NEW_SERVICE 2>/dev/null || true

# Pull latest image
docker pull laadpaalnodig/api:$VERSION

# Start new service
docker run -d \
  --name $NEW_SERVICE \
  --env-file .env.$ENVIRONMENT \
  -e VERSION=$VERSION \
  --network charging-network \
  laadpaalnodig/api:$VERSION

# Wait for service to be ready
echo "Waiting for $NEW_SERVICE to be ready..."
for i in {1..30}; do
  if docker exec $NEW_SERVICE curl -f http://localhost:3000/health > /dev/null 2>&1; then
    echo "$NEW_SERVICE is ready!"
    break
  fi
  
  if [ $i -eq 30 ]; then
    echo "ERROR: $NEW_SERVICE failed to start properly"
    docker logs $NEW_SERVICE
    exit 1
  fi
  
  sleep 2
done

# Run smoke tests
echo "Running smoke tests..."
docker exec $NEW_SERVICE npm run test:smoke

if [ $? -ne 0 ]; then
  echo "ERROR: Smoke tests failed"
  docker stop $NEW_SERVICE
  docker rm $NEW_SERVICE
  exit 1
fi

# Update nginx configuration
echo "Updating nginx configuration..."
sed -i "s/$ACTIVE_SERVICE/$NEW_SERVICE/g" $NGINX_CONFIG
nginx -t && nginx -s reload

# Wait for traffic to switch
sleep 10

# Run integration tests against new service
echo "Running integration tests..."
npm run test:integration -- --baseUrl=https://api-$ENVIRONMENT.laadpaalnodig.nl

if [ $? -ne 0 ]; then
  echo "ERROR: Integration tests failed, rolling back..."
  sed -i "s/$NEW_SERVICE/$ACTIVE_SERVICE/g" $NGINX_CONFIG
  nginx -s reload
  docker stop $NEW_SERVICE
  docker rm $NEW_SERVICE
  exit 1
fi

# Deployment successful - stop old service
echo "Deployment successful! Stopping old service..."
docker stop $ACTIVE_SERVICE
docker rm $ACTIVE_SERVICE

echo "Blue-Green deployment completed successfully!"
```

### Rolling Deployment

```yaml
# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: charging-api
  labels:
    app: charging-api
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: charging-api
  template:
    metadata:
      labels:
        app: charging-api
    spec:
      containers:
      - name: api
        image: laadpaalnodig/api:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: api-secrets
              key: api-key
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: api-secrets
              key: database-url
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          limits:
            cpu: 1000m
            memory: 1Gi
          requests:
            cpu: 500m
            memory: 512Mi
```

### Canary Deployment

```yaml
# istio/canary-deployment.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: charging-api-rollout
spec:
  replicas: 5
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {duration: 2m}
      - setWeight: 40
      - pause: {duration: 2m}
      - setWeight: 60
      - pause: {duration: 2m}
      - setWeight: 80
      - pause: {duration: 2m}
      canaryService: charging-api-canary
      stableService: charging-api-stable
      trafficRouting:
        istio:
          virtualService:
            name: charging-api-vs
  selector:
    matchLabels:
      app: charging-api
  template:
    metadata:
      labels:
        app: charging-api
    spec:
      containers:
      - name: api
        image: laadpaalnodig/api:latest
        ports:
        - containerPort: 3000
```

---

## Monitoring & Logging

### Application Monitoring

```javascript
// monitoring/metrics.js
const prometheus = require('prom-client');

// Create metrics registry
const register = new prometheus.Registry();

// Default metrics
prometheus.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.3, 0.5, 0.7, 1, 3, 5, 7, 10]
});

const httpRequestTotal = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

const activeConnections = new prometheus.Gauge({
  name: 'active_connections',
  help: 'Number of active connections'
});

const apiKeyUsage = new prometheus.Counter({
  name: 'api_key_usage_total',
  help: 'Total API key usage',
  labelNames: ['key_id', 'endpoint']
});

const applicationCreated = new prometheus.Counter({
  name: 'applications_created_total',
  help: 'Total number of applications created',
  labelNames: ['category']
});

const documentUploads = new prometheus.Counter({
  name: 'documents_uploaded_total',
  help: 'Total number of documents uploaded',
  labelNames: ['type', 'status']
});

// Register metrics
register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(activeConnections);
register.registerMetric(apiKeyUsage);
register.registerMetric(applicationCreated);
register.registerMetric(documentUploads);

// Metrics middleware
const metricsMiddleware = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const route = req.route?.path || req.path;
    
    httpRequestDuration
      .labels(req.method, route, res.statusCode)
      .observe(duration);
    
    httpRequestTotal
      .labels(req.method, route, res.statusCode)
      .inc();
    
    // Track API key usage
    if (req.apiKey) {
      apiKeyUsage
        .labels(req.apiKey.id, route)
        .inc();
    }
  });
  
  next();
};

module.exports = {
  register,
  metricsMiddleware,
  metrics: {
    httpRequestDuration,
    httpRequestTotal,
    activeConnections,
    apiKeyUsage,
    applicationCreated,
    documentUploads
  }
};
```

### Structured Logging

```javascript
// utils/logger.js
const winston = require('winston');
const { config } = require('../config');

const logFormat = winston.format.combine(
  winston.format.timestamp(),
  winston.format.errors({ stack: true }),
  winston.format.json(),
  winston.format.printf(({ timestamp, level, message, ...meta }) => {
    return JSON.stringify({
      timestamp,
      level,
      message,
      service: 'charging-api',
      version: process.env.APP_VERSION,
      environment: config.get('env'),
      ...meta
    });
  })
);

const logger = winston.createLogger({
  level: config.get('monitoring.logLevel'),
  format: logFormat,
  defaultMeta: {
    service: 'charging-api',
    version: process.env.APP_VERSION
  },
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ 
      filename: 'logs/error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'logs/combined.log' 
    })
  ]
});

// Request logging middleware
const requestLogger = (req, res, next) => {
  const start = Date.now();
  
  // Log request
  logger.info('HTTP Request', {
    method: req.method,
    url: req.url,
    userAgent: req.get('User-Agent'),
    ip: req.ip,
    apiKeyId: req.apiKey?.id,
    requestId: req.id
  });
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    
    logger.info('HTTP Response', {
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration,
      requestId: req.id
    });
  });
  
  next();
};

// Error logging middleware
const errorLogger = (err, req, res, next) => {
  logger.error('Request Error', {
    error: {
      message: err.message,
      stack: err.stack,
      code: err.code
    },
    method: req.method,
    url: req.url,
    requestId: req.id,
    apiKeyId: req.apiKey?.id
  });
  
  next(err);
};

module.exports = {
  logger,
  requestLogger,
  errorLogger
};
```

### Health Checks

```javascript
// routes/health.js
const express = require('express');
const { config } = require('../config');
const { pool } = require('../database');
const redis = require('../redis');

const router = express.Router();

// Basic health check
router.get('/health', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: process.env.APP_VERSION,
    environment: config.get('env'),
    uptime: process.uptime(),
    memory: process.memoryUsage()
  };
  
  res.json(health);
});

// Detailed readiness check
router.get('/ready', async (req, res) => {
  const checks = {
    database: false,
    redis: false,
    externalApi: false
  };
  
  let overallStatus = 'ready';
  
  try {
    // Database check
    await pool.query('SELECT 1');
    checks.database = true;
  } catch (error) {
    checks.database = false;
    overallStatus = 'not_ready';
  }
  
  try {
    // Redis check
    await redis.ping();
    checks.redis = true;
  } catch (error) {
    checks.redis = false;
    overallStatus = 'not_ready';
  }
  
  try {
    // External API check
    const response = await fetch(config.get('api.baseUrl') + '/health', {
      timeout: 5000
    });
    checks.externalApi = response.ok;
  } catch (error) {
    checks.externalApi = false;
    overallStatus = 'not_ready';
  }
  
  const statusCode = overallStatus === 'ready' ? 200 : 503;
  
  res.status(statusCode).json({
    status: overallStatus,
    timestamp: new Date().toISOString(),
    checks
  });
});

// Liveness probe
router.get('/live', (req, res) => {
  res.json({
    status: 'alive',
    timestamp: new Date().toISOString()
  });
});

module.exports = router;
```

---

## Performance Optimization

### Caching Strategy

```javascript
// cache/redis-cache.js
const redis = require('redis');
const { config } = require('../config');

class RedisCache {
  constructor() {
    this.client = redis.createClient({
      url: config.get('redis.url'),
      password: config.get('redis.password')
    });
    
    this.client.on('error', (err) => {
      console.error('Redis Client Error:', err);
    });
    
    this.keyPrefix = config.get('redis.keyPrefix');
    this.defaultTTL = 3600; // 1 hour
  }

  async connect() {
    await this.client.connect();
  }

  formatKey(key) {
    return `${this.keyPrefix}${key}`;
  }

  async get(key) {
    try {
      const data = await this.client.get(this.formatKey(key));
      return data ? JSON.parse(data) : null;
    } catch (error) {
      console.error('Cache get error:', error);
      return null;
    }
  }

  async set(key, value, ttl = this.defaultTTL) {
    try {
      await this.client.setEx(
        this.formatKey(key),
        ttl,
        JSON.stringify(value)
      );
      return true;
    } catch (error) {
      console.error('Cache set error:', error);
      return false;
    }
  }

  async del(key) {
    try {
      await this.client.del(this.formatKey(key));
      return true;
    } catch (error) {
      console.error('Cache delete error:', error);
      return false;
    }
  }

  async invalidatePattern(pattern) {
    try {
      const keys = await this.client.keys(this.formatKey(pattern));
      if (keys.length > 0) {
        await this.client.del(keys);
      }
      return true;
    } catch (error) {
      console.error('Cache invalidate pattern error:', error);
      return false;
    }
  }

  // Cache middleware for Express
  middleware(options = {}) {
    const { ttl = this.defaultTTL, keyGenerator } = options;
    
    return async (req, res, next) => {
      const key = keyGenerator ? 
        keyGenerator(req) : 
        `${req.method}:${req.originalUrl}`;
      
      try {
        const cached = await this.get(key);
        
        if (cached) {
          res.set('X-Cache', 'HIT');
          return res.json(cached);
        }
        
        // Override res.json to cache the response
        const originalJson = res.json;
        res.json = function(data) {
          res.set('X-Cache', 'MISS');
          
          // Cache successful responses
          if (res.statusCode === 200 && data.success) {
            cache.set(key, data, ttl);
          }
          
          return originalJson.call(this, data);
        };
        
        next();
      } catch (error) {
        console.error('Cache middleware error:', error);
        next();
      }
    };
  }
}

const cache = new RedisCache();

module.exports = cache;
```

### Database Optimization

```javascript
// database/pool.js
const { Pool } = require('pg');
const { config } = require('../config');

const pool = new Pool({
  connectionString: config.get('database.url'),
  min: config.get('database.pool.min'),
  max: config.get('database.pool.max'),
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
  ssl: config.get('database.ssl') ? { rejectUnauthorized: false } : false
});

// Connection monitoring
pool.on('connect', () => {
  console.log('Connected to database');
});

pool.on('error', (err) => {
  console.error('Database pool error:', err);
});

// Query logging and performance monitoring
const originalQuery = pool.query;
pool.query = function(text, params, callback) {
  const start = Date.now();
  
  return originalQuery.call(this, text, params, (err, result) => {
    const duration = Date.now() - start;
    
    if (duration > 1000) {
      console.warn('Slow query detected:', {
        query: text,
        duration,
        params: params?.length
      });
    }
    
    if (callback) callback(err, result);
  });
};

module.exports = pool;
```

---

## Backup & Recovery

### Automated Backups

```bash
#!/bin/bash
# backup.sh - Database backup script

set -e

# Configuration
DB_HOST=${DB_HOST:-localhost}
DB_PORT=${DB_PORT:-5432}
DB_NAME=${DB_NAME:-charging_production}
DB_USER=${DB_USER:-postgres}
BACKUP_DIR=${BACKUP_DIR:-/backups}
S3_BUCKET=${S3_BUCKET:-charging-backups}
RETENTION_DAYS=${RETENTION_DAYS:-30}

# Create backup directory
mkdir -p $BACKUP_DIR

# Generate backup filename
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/charging_db_$TIMESTAMP.sql"

echo "Starting database backup..."

# Create database backup
pg_dump -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME \
  --verbose --clean --no-owner --no-privileges \
  --format=custom --compress=9 \
  --file=$BACKUP_FILE

# Compress backup
gzip $BACKUP_FILE
BACKUP_FILE="$BACKUP_FILE.gz"

echo "Backup created: $BACKUP_FILE"

# Upload to S3
if [ ! -z "$S3_BUCKET" ]; then
  echo "Uploading backup to S3..."
  aws s3 cp $BACKUP_FILE s3://$S3_BUCKET/database/
  echo "Backup uploaded to S3"
fi

# Clean up old local backups
find $BACKUP_DIR -name "charging_db_*.sql.gz" -mtime +$RETENTION_DAYS -delete

echo "Database backup completed successfully"
```

### Recovery Procedures

```bash
#!/bin/bash
# restore.sh - Database restore script

set -e

BACKUP_FILE=$1
DB_NAME=${DB_NAME:-charging_production}
DB_USER=${DB_USER:-postgres}

if [ -z "$BACKUP_FILE" ]; then
  echo "Usage: $0 <backup_file>"
  exit 1
fi

echo "Restoring database from $BACKUP_FILE..."

# Download from S3 if needed
if [[ $BACKUP_FILE == s3://* ]]; then
  LOCAL_FILE="/tmp/$(basename $BACKUP_FILE)"
  aws s3 cp $BACKUP_FILE $LOCAL_FILE
  BACKUP_FILE=$LOCAL_FILE
fi

# Decompress if needed
if [[ $BACKUP_FILE == *.gz ]]; then
  gunzip $BACKUP_FILE
  BACKUP_FILE=${BACKUP_FILE%.gz}
fi

# Stop application
docker stop charging-api || true

# Create new database
createdb -U $DB_USER ${DB_NAME}_new

# Restore backup
pg_restore -U $DB_USER -d ${DB_NAME}_new \
  --verbose --clean --no-owner --no-privileges \
  $BACKUP_FILE

# Rename databases
psql -U $DB_USER -c "ALTER DATABASE $DB_NAME RENAME TO ${DB_NAME}_old;"
psql -U $DB_USER -c "ALTER DATABASE ${DB_NAME}_new RENAME TO $DB_NAME;"

# Start application
docker start charging-api

echo "Database restore completed successfully"
echo "Old database available as ${DB_NAME}_old"
```

---

## Troubleshooting

### Common Issues

#### 1. Database Connection Issues

```bash
# Check database connectivity
docker exec charging-api pg_isready -h database -p 5432

# View database logs
docker logs charging-database

# Check connection pool status
curl -s http://localhost:3000/debug/pool-status
```

#### 2. Memory Issues

```bash
# Monitor memory usage
docker stats charging-api

# Check for memory leaks
curl -s http://localhost:3000/debug/memory

# Restart with memory limit
docker run --memory=1g --name charging-api-new laadpaalnodig/api:latest
```

#### 3. Performance Issues

```bash
# Check response times
curl -w "@curl-format.txt" -s http://localhost:3000/applications

# Monitor database queries
tail -f logs/slow-queries.log

# Check cache hit rates
curl -s http://localhost:3000/debug/cache-stats
```

### Debugging Tools

```javascript
// routes/debug.js (only enable in non-production)
const express = require('express');
const { pool } = require('../database');
const cache = require('../cache');

const router = express.Router();

// Only enable in development/staging
if (process.env.NODE_ENV !== 'production') {
  
  router.get('/pool-status', async (req, res) => {
    res.json({
      totalCount: pool.totalCount,
      idleCount: pool.idleCount,
      waitingCount: pool.waitingCount
    });
  });

  router.get('/memory', (req, res) => {
    const usage = process.memoryUsage();
    res.json({
      rss: `${Math.round(usage.rss / 1024 / 1024)} MB`,
      heapTotal: `${Math.round(usage.heapTotal / 1024 / 1024)} MB`,
      heapUsed: `${Math.round(usage.heapUsed / 1024 / 1024)} MB`,
      external: `${Math.round(usage.external / 1024 / 1024)} MB`
    });
  });

  router.get('/cache-stats', async (req, res) => {
    const stats = await cache.client.info('stats');
    res.json({ stats });
  });
}

module.exports = router;
```

This comprehensive deployment guide covers all aspects of production deployment, from environment setup to monitoring and troubleshooting. The documentation provides practical, working examples that can be adapted to specific deployment requirements.
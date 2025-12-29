# Part 9: Performance Optimization and Caching

**Author**: Saiprasad Hegde

## Table of Contents
- [Introduction](#introduction)
- [Understanding Performance](#understanding-performance)
- [Caching Strategies](#caching-strategies)
- [Database Optimization](#database-optimization)
- [API Rate Limiting](#api-rate-limiting)
- [Load Balancing and Scaling](#load-balancing-and-scaling)
- [Performance Monitoring](#performance-monitoring)
- [Best Practices](#best-practices)

---

## Introduction

### What is Performance Optimization?

**Analogy**: Think of your backend as a highway system:
- **Without optimization**: Single-lane roads with traffic jams
- **With optimization**: Multi-lane highways with efficient traffic flow, rest stops (caches), and alternative routes (load balancing)

Performance optimization ensures your application:
- Responds quickly to user requests
- Handles many concurrent users
- Uses resources efficiently
- Scales as demand grows

### Why Does Performance Matter?

1. **User Experience**: Users expect responses in under 200ms
2. **Cost Efficiency**: Better performance = fewer servers needed
3. **Scalability**: Handle growth without rewriting code
4. **SEO**: Search engines favor faster sites
5. **Business Impact**: Every 100ms of delay can reduce conversions by 1%

---

## Caching Strategies

### What is Caching?

**Analogy**: Caching is like keeping frequently used books on your desk instead of walking to the library every time.

Cache stores frequently accessed data in fast storage (memory) instead of slow storage (database, disk).

### Types of Caching

#### 1. In-Memory Caching (Application Level)

Simple caching using JavaScript objects or Map:

```javascript
// Simple in-memory cache
class SimpleCache {
    constructor() {
        this.cache = new Map();
        this.ttl = new Map(); // Time to live
    }

    set(key, value, ttlSeconds = 300) {
        this.cache.set(key, value);
        this.ttl.set(key, Date.now() + (ttlSeconds * 1000));
    }

    get(key) {
        // Check if expired
        const expiry = this.ttl.get(key);
        if (expiry && Date.now() > expiry) {
            this.cache.delete(key);
            this.ttl.delete(key);
            return null;
        }
        return this.cache.get(key) || null;
    }

    delete(key) {
        this.cache.delete(key);
        this.ttl.delete(key);
    }

    clear() {
        this.cache.clear();
        this.ttl.clear();
    }
}

// Usage in service layer
const cache = new SimpleCache();

class StudentService {
    async getStudentById(id) {
        // Check cache first
        const cacheKey = `student:${id}`;
        const cached = cache.get(cacheKey);

        if (cached) {
            console.log('Cache hit!');
            return cached;
        }

        console.log('Cache miss - querying database');
        const student = await studentRepository.findById(id);

        // Store in cache for 5 minutes
        cache.set(cacheKey, student, 300);

        return student;
    }

    async updateStudent(id, data) {
        const updated = await studentRepository.update(id, data);

        // Invalidate cache when data changes
        cache.delete(`student:${id}`);

        return updated;
    }
}
```

**Pros**: Simple, no external dependencies
**Cons**: Limited to single server, lost on restart, no memory management

#### 2. Redis Caching (Distributed)

Redis is an in-memory data store that can be shared across multiple servers.

**Installation**:
```bash
npm install redis ioredis
```

**Redis Setup**:

```javascript
// config/redis.js
const Redis = require('ioredis');

const redis = new Redis({
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379,
    password: process.env.REDIS_PASSWORD,
    retryStrategy: (times) => {
        const delay = Math.min(times * 50, 2000);
        return delay;
    }
});

redis.on('connect', () => {
    console.log('✅ Redis connected');
});

redis.on('error', (err) => {
    console.error('❌ Redis error:', err);
});

module.exports = redis;
```

**Cache Service with Redis**:

```javascript
// services/cacheService.js
const redis = require('../config/redis');

class CacheService {
    /**
     * Get value from cache
     */
    async get(key) {
        try {
            const data = await redis.get(key);
            return data ? JSON.parse(data) : null;
        } catch (error) {
            console.error('Cache get error:', error);
            return null;
        }
    }

    /**
     * Set value in cache with TTL
     */
    async set(key, value, ttlSeconds = 300) {
        try {
            await redis.setex(key, ttlSeconds, JSON.stringify(value));
            return true;
        } catch (error) {
            console.error('Cache set error:', error);
            return false;
        }
    }

    /**
     * Delete from cache
     */
    async delete(key) {
        try {
            await redis.del(key);
            return true;
        } catch (error) {
            console.error('Cache delete error:', error);
            return false;
        }
    }

    /**
     * Delete multiple keys matching pattern
     */
    async deletePattern(pattern) {
        try {
            const keys = await redis.keys(pattern);
            if (keys.length > 0) {
                await redis.del(...keys);
            }
            return true;
        } catch (error) {
            console.error('Cache delete pattern error:', error);
            return false;
        }
    }

    /**
     * Get or set pattern (cache aside)
     */
    async getOrSet(key, fetchFunction, ttlSeconds = 300) {
        // Try to get from cache
        const cached = await this.get(key);
        if (cached !== null) {
            return cached;
        }

        // If not in cache, fetch data
        const data = await fetchFunction();

        // Store in cache
        await this.set(key, data, ttlSeconds);

        return data;
    }
}

module.exports = new CacheService();
```

**Using Cache in Service Layer**:

```javascript
// services/studentService.js
const studentRepository = require('../repositories/studentRepository');
const cacheService = require('./cacheService');

class StudentService {
    async getStudentById(id) {
        const cacheKey = `student:${id}`;

        // Use getOrSet pattern
        return await cacheService.getOrSet(
            cacheKey,
            () => studentRepository.findById(id),
            600 // 10 minutes
        );
    }

    async getAllStudents(filters = {}) {
        // Create cache key from filters
        const cacheKey = `students:list:${JSON.stringify(filters)}`;

        return await cacheService.getOrSet(
            cacheKey,
            () => studentRepository.findAll(filters),
            300 // 5 minutes
        );
    }

    async updateStudent(id, data) {
        const updated = await studentRepository.update(id, data);

        // Invalidate related caches
        await cacheService.delete(`student:${id}`);
        await cacheService.deletePattern('students:list:*');

        return updated;
    }

    async getStudentWithEnrollments(id) {
        const cacheKey = `student:${id}:enrollments`;

        return await cacheService.getOrSet(
            cacheKey,
            async () => {
                const student = await studentRepository.findById(id);
                const enrollments = await enrollmentRepository.findByStudentId(id);
                return { ...student, enrollments };
            },
            600
        );
    }
}

module.exports = new StudentService();
```

**Python/FastAPI Redis Caching**:

```python
# config/redis.py
import redis.asyncio as redis
from typing import Optional, Any
import json
import os

class RedisCache:
    def __init__(self):
        self.redis: Optional[redis.Redis] = None

    async def connect(self):
        self.redis = await redis.from_url(
            os.getenv('REDIS_URL', 'redis://localhost:6379'),
            encoding='utf-8',
            decode_responses=True
        )
        print("✅ Redis connected")

    async def disconnect(self):
        if self.redis:
            await self.redis.close()

    async def get(self, key: str) -> Optional[Any]:
        try:
            data = await self.redis.get(key)
            return json.loads(data) if data else None
        except Exception as e:
            print(f"Cache get error: {e}")
            return None

    async def set(self, key: str, value: Any, ttl: int = 300):
        try:
            await self.redis.setex(key, ttl, json.dumps(value))
            return True
        except Exception as e:
            print(f"Cache set error: {e}")
            return False

    async def delete(self, key: str):
        try:
            await self.redis.delete(key)
            return True
        except Exception as e:
            print(f"Cache delete error: {e}")
            return False

    async def delete_pattern(self, pattern: str):
        try:
            keys = await self.redis.keys(pattern)
            if keys:
                await self.redis.delete(*keys)
            return True
        except Exception as e:
            print(f"Cache delete pattern error: {e}")
            return False

cache = RedisCache()

# Service using cache
class StudentService:
    async def get_student_by_id(self, student_id: str):
        cache_key = f"student:{student_id}"

        # Try cache first
        cached = await cache.get(cache_key)
        if cached:
            return cached

        # Fetch from database
        student = await student_repository.find_by_id(student_id)

        # Store in cache
        await cache.set(cache_key, student, ttl=600)

        return student
```

### Caching Strategies

#### 1. Cache-Aside (Lazy Loading)
Application checks cache, if miss, loads from DB and updates cache.

```javascript
async function getCourse(id) {
    // 1. Check cache
    let course = await cache.get(`course:${id}`);

    if (!course) {
        // 2. Cache miss - load from DB
        course = await db.courses.findById(id);

        // 3. Update cache
        await cache.set(`course:${id}`, course, 600);
    }

    return course;
}
```

**Pros**: Only caches what's needed
**Cons**: Cache miss penalty, stale data possible

#### 2. Write-Through
Write to cache and database simultaneously.

```javascript
async function updateCourse(id, data) {
    // 1. Update database
    const updated = await db.courses.update(id, data);

    // 2. Update cache immediately
    await cache.set(`course:${id}`, updated, 600);

    return updated;
}
```

**Pros**: Cache always fresh
**Cons**: Write latency, caches unused data

#### 3. Write-Behind (Write-Back)
Write to cache immediately, update database asynchronously.

```javascript
async function updateCourse(id, data) {
    // 1. Update cache immediately
    await cache.set(`course:${id}`, data, 600);

    // 2. Queue database update (async)
    await updateQueue.add({ type: 'course', id, data });

    return data;
}

// Worker processes queue
async function processQueue(job) {
    await db.courses.update(job.id, job.data);
}
```

**Pros**: Fast writes, reduced DB load
**Cons**: Data loss risk, complexity

### Cache Invalidation

**The two hardest problems in CS**: Naming things and cache invalidation.

#### Time-Based Expiration (TTL)

```javascript
// Short TTL for frequently changing data
await cache.set('student:count', count, 60); // 1 minute

// Long TTL for rarely changing data
await cache.set('departments', departments, 3600); // 1 hour
```

#### Event-Based Invalidation

```javascript
class StudentService {
    async createStudent(data) {
        const student = await studentRepository.create(data);

        // Invalidate list caches
        await cache.deletePattern('students:list:*');
        await cache.delete('students:count');

        return student;
    }

    async updateStudent(id, data) {
        const student = await studentRepository.update(id, data);

        // Invalidate specific student cache
        await cache.delete(`student:${id}`);

        // Invalidate lists that might contain this student
        await cache.deletePattern('students:list:*');

        return student;
    }
}
```

#### Tag-Based Invalidation

```javascript
// When setting cache, also maintain tags
async function cacheWithTags(key, value, tags, ttl) {
    await cache.set(key, value, ttl);

    // Add key to each tag set
    for (const tag of tags) {
        await redis.sadd(`tag:${tag}`, key);
    }
}

// Invalidate by tag
async function invalidateTag(tag) {
    const keys = await redis.smembers(`tag:${tag}`);
    if (keys.length > 0) {
        await redis.del(...keys);
    }
    await redis.del(`tag:${tag}`);
}

// Usage
await cacheWithTags(
    'student:123:enrollments',
    enrollments,
    ['student:123', 'enrollments', 'course:456'],
    600
);

// When course 456 updates, invalidate all related caches
await invalidateTag('course:456');
```

---

## Database Optimization

### 1. Indexing

**Analogy**: Indexes are like a book's index - instead of reading every page to find "Redis", you check the index and jump to page 247.

#### Creating Indexes

**PostgreSQL**:
```sql
-- Index on frequently queried column
CREATE INDEX idx_students_email ON students(email);

-- Composite index for multi-column queries
CREATE INDEX idx_enrollments_student_course
ON enrollments(student_id, course_id);

-- Partial index (only index subset)
CREATE INDEX idx_active_students
ON students(department_id)
WHERE is_active = true;

-- Index for text search
CREATE INDEX idx_students_name_search
ON students USING gin(to_tsvector('english', first_name || ' ' || last_name));
```

#### Sequelize Indexes

```javascript
// models/student.js
const Student = sequelize.define('Student', {
    firstName: DataTypes.STRING,
    lastName: DataTypes.STRING,
    email: {
        type: DataTypes.STRING,
        unique: true // Automatically creates unique index
    },
    departmentId: DataTypes.UUID
}, {
    indexes: [
        // Simple index
        {
            fields: ['email']
        },
        // Composite index
        {
            fields: ['department_id', 'is_active']
        },
        // Unique composite index
        {
            unique: true,
            fields: ['email', 'department_id']
        },
        // Partial index
        {
            fields: ['last_name'],
            where: {
                is_active: true
            }
        }
    ]
});
```

#### Drizzle Indexes

```typescript
// schema/students.ts
import { pgTable, uuid, varchar, boolean, index } from 'drizzle-orm/pg-core';

export const students = pgTable('students', {
    id: uuid('id').defaultRandom().primaryKey(),
    firstName: varchar('first_name', { length: 50 }).notNull(),
    lastName: varchar('last_name', { length: 50 }).notNull(),
    email: varchar('email', { length: 255 }).notNull().unique(),
    departmentId: uuid('department_id').notNull(),
    isActive: boolean('is_active').default(true)
}, (table) => ({
    // Create indexes
    emailIdx: index('idx_students_email').on(table.email),
    departmentIdx: index('idx_students_department').on(table.departmentId),
    compositeIdx: index('idx_students_dept_active').on(table.departmentId, table.isActive)
}));
```

#### When to Use Indexes

**Use indexes for**:
- Primary keys (automatic)
- Foreign keys
- Columns in WHERE clauses
- Columns in JOIN conditions
- Columns in ORDER BY
- Columns with high cardinality (many unique values)

**Don't index**:
- Small tables (< 1000 rows)
- Columns rarely used in queries
- Columns with low cardinality (few unique values, like boolean)
- Tables with frequent writes (indexes slow down writes)

### 2. Query Optimization

#### N+1 Query Problem

**Bad** (N+1 queries):
```javascript
// 1 query for students
const students = await Student.findAll();

// N queries for enrollments (one per student!)
for (const student of students) {
    student.enrollments = await Enrollment.findAll({
        where: { studentId: student.id }
    });
}
// Total: 1 + N queries
```

**Good** (2 queries with proper joins):
```javascript
// Single query with join
const students = await Student.findAll({
    include: [{
        model: Enrollment,
        include: [Course]
    }]
});
// Total: 1-2 queries
```

#### Eager Loading vs Lazy Loading

**Eager Loading** (load everything upfront):
```javascript
const students = await Student.findAll({
    include: [
        { model: Enrollment, include: [Course] },
        { model: Department }
    ]
});
// All data loaded in single query
```

**Lazy Loading** (load on demand):
```javascript
const students = await Student.findAll();
// Later, when needed:
const firstStudentEnrollments = await students[0].getEnrollments();
```

#### Pagination

Always paginate large result sets:

```javascript
// services/studentService.js
async function getStudents(page = 1, limit = 20, filters = {}) {
    const offset = (page - 1) * limit;

    const { count, rows } = await Student.findAndCountAll({
        where: filters,
        limit,
        offset,
        order: [['lastName', 'ASC'], ['firstName', 'ASC']]
    });

    return {
        data: rows,
        pagination: {
            page,
            limit,
            totalItems: count,
            totalPages: Math.ceil(count / limit),
            hasMore: offset + rows.length < count
        }
    };
}
```

#### Cursor-Based Pagination (Better for real-time data)

```javascript
async function getStudentsCursor(cursor = null, limit = 20) {
    const where = cursor ? {
        id: { [Op.gt]: cursor } // Get IDs greater than cursor
    } : {};

    const students = await Student.findAll({
        where,
        limit: limit + 1, // Fetch one extra to check if there's more
        order: [['id', 'ASC']]
    });

    const hasMore = students.length > limit;
    const data = hasMore ? students.slice(0, -1) : students;
    const nextCursor = hasMore ? data[data.length - 1].id : null;

    return {
        data,
        nextCursor,
        hasMore
    };
}
```

#### Select Only Needed Fields

```javascript
// Bad - loads all columns
const students = await Student.findAll();

// Good - only needed fields
const students = await Student.findAll({
    attributes: ['id', 'firstName', 'lastName', 'email']
});

// Exclude specific fields
const students = await Student.findAll({
    attributes: { exclude: ['password', 'passwordResetToken'] }
});
```

### 3. Database Connection Pooling

**Analogy**: Connection pooling is like a taxi stand. Instead of calling a new taxi (creating connection) each time, you use taxis waiting at the stand (connection pool).

**Sequelize Connection Pool**:

```javascript
const sequelize = new Sequelize(DATABASE_URL, {
    pool: {
        max: 20,        // Maximum connections in pool
        min: 5,         // Minimum connections
        acquire: 30000, // Max time (ms) to get connection
        idle: 10000     // Max time connection can be idle
    },
    logging: false
});
```

**Drizzle with node-postgres Pool**:

```typescript
import { Pool } from 'pg';
import { drizzle } from 'drizzle-orm/node-postgres';

const pool = new Pool({
    connectionString: process.env.DATABASE_URL,
    max: 20,
    min: 5,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000
});

const db = drizzle(pool);
```

### 4. Database Query Analysis

Use `EXPLAIN ANALYZE` to understand query performance:

```javascript
// Enable query logging
const sequelize = new Sequelize(DATABASE_URL, {
    logging: console.log,
    benchmark: true
});

// Or manual EXPLAIN
const [results] = await sequelize.query(`
    EXPLAIN ANALYZE
    SELECT s.*, e.*, c.*
    FROM students s
    JOIN enrollments e ON s.id = e.student_id
    JOIN courses c ON e.course_id = c.id
    WHERE s.department_id = '123';
`);

console.log(results);
```

Look for:
- **Seq Scan** (sequential scan) → Add index
- **High cost** → Optimize query
- **Nested loops** on large tables → Consider different join

---

## API Rate Limiting

Rate limiting prevents abuse and ensures fair usage.

**Analogy**: Rate limiting is like a bouncer at a club - "You can enter 100 times per hour, not more."

### Express Rate Limiting

```bash
npm install express-rate-limit redis rate-limit-redis
```

**Basic Rate Limiting**:

```javascript
// middleware/rateLimiter.js
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const redis = require('../config/redis');

// General API rate limiter
const apiLimiter = rateLimit({
    store: new RedisStore({
        client: redis,
        prefix: 'rl:api:'
    }),
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // Max 100 requests per window
    message: {
        error: 'Too many requests, please try again later.',
        retryAfter: '15 minutes'
    },
    standardHeaders: true, // Return rate limit info in headers
    legacyHeaders: false
});

// Strict limiter for expensive operations
const strictLimiter = rateLimit({
    store: new RedisStore({
        client: redis,
        prefix: 'rl:strict:'
    }),
    windowMs: 60 * 1000, // 1 minute
    max: 10, // Max 10 requests per minute
    message: {
        error: 'Rate limit exceeded for this operation.'
    }
});

// Auth endpoints limiter (prevent brute force)
const authLimiter = rateLimit({
    store: new RedisStore({
        client: redis,
        prefix: 'rl:auth:'
    }),
    windowMs: 15 * 60 * 1000,
    max: 5, // Only 5 login attempts per 15 minutes
    skipSuccessfulRequests: true, // Don't count successful logins
    message: {
        error: 'Too many login attempts, please try again later.'
    }
});

module.exports = { apiLimiter, strictLimiter, authLimiter };
```

**Usage**:

```javascript
// app.js
const { apiLimiter, strictLimiter, authLimiter } = require('./middleware/rateLimiter');

// Apply to all API routes
app.use('/api/', apiLimiter);

// Apply strict limiter to expensive operations
app.post('/api/enrollments', strictLimiter, enrollmentController.create);

// Apply to auth routes
app.post('/api/auth/login', authLimiter, authController.login);
app.post('/api/auth/register', authLimiter, authController.register);
```

### Advanced Rate Limiting

**Per-User Rate Limiting**:

```javascript
const userRateLimiter = rateLimit({
    store: new RedisStore({
        client: redis,
        prefix: 'rl:user:'
    }),
    windowMs: 60 * 1000,
    max: 30,
    keyGenerator: (req) => {
        // Use user ID if authenticated, IP otherwise
        return req.user?.id || req.ip;
    }
});
```

**Tiered Rate Limiting**:

```javascript
function tierBasedLimiter(req, res, next) {
    const user = req.user;

    let maxRequests = 100; // Free tier
    if (user?.subscription === 'premium') {
        maxRequests = 1000;
    } else if (user?.subscription === 'enterprise') {
        maxRequests = 10000;
    }

    const limiter = rateLimit({
        store: new RedisStore({
            client: redis,
            prefix: `rl:tier:${user?.id || req.ip}:`
        }),
        windowMs: 60 * 60 * 1000, // 1 hour
        max: maxRequests
    });

    return limiter(req, res, next);
}
```

### FastAPI Rate Limiting

```python
from fastapi import FastAPI, Request, HTTPException
from fastapi.responses import JSONResponse
from datetime import datetime, timedelta
import redis.asyncio as redis

app = FastAPI()
redis_client = redis.from_url("redis://localhost:6379")

async def rate_limit(request: Request, max_requests: int = 100, window_seconds: int = 900):
    # Use user ID or IP as key
    user_id = request.state.user.id if hasattr(request.state, 'user') else request.client.host
    key = f"rl:{user_id}"

    # Increment counter
    count = await redis_client.incr(key)

    # Set expiry on first request
    if count == 1:
        await redis_client.expire(key, window_seconds)

    # Check limit
    if count > max_requests:
        ttl = await redis_client.ttl(key)
        raise HTTPException(
            status_code=429,
            detail=f"Rate limit exceeded. Try again in {ttl} seconds."
        )

    return count

@app.get("/api/students")
async def get_students(request: Request):
    await rate_limit(request, max_requests=100, window_seconds=900)
    # ... handle request
```

---

## Load Balancing and Scaling

### Vertical Scaling vs Horizontal Scaling

**Vertical Scaling** (Scale Up):
- Add more CPU/RAM to single server
- **Analogy**: Making one worker stronger
- **Pros**: Simple, no code changes
- **Cons**: Limited, expensive, single point of failure

**Horizontal Scaling** (Scale Out):
- Add more servers
- **Analogy**: Hiring more workers
- **Pros**: Unlimited scaling, fault tolerant
- **Cons**: Requires load balancer, session management, data consistency

### Load Balancing

**Analogy**: Load balancer is like a restaurant host who seats customers at different tables to balance the workload among servers.

**Common Load Balancing Algorithms**:

1. **Round Robin**: Distribute requests evenly (Server1 → Server2 → Server3 → Server1...)
2. **Least Connections**: Send to server with fewest active connections
3. **IP Hash**: Same client always goes to same server (sticky sessions)
4. **Weighted**: Some servers handle more load based on capacity

**Nginx Load Balancer Configuration**:

```nginx
# nginx.conf
upstream backend {
    # Least connections algorithm
    least_conn;

    # Backend servers
    server backend1.example.com:3000 weight=3;
    server backend2.example.com:3000 weight=2;
    server backend3.example.com:3000 weight=1;

    # Health checks
    server backend4.example.com:3000 backup; # Only if others fail
}

server {
    listen 80;

    location /api/ {
        proxy_pass http://backend;

        # Headers to pass
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

### Session Management in Scaled Environment

When you have multiple servers, sessions must be shared:

**Option 1: Redis Session Store**

```javascript
const session = require('express-session');
const RedisStore = require('connect-redis')(session);
const redis = require('./config/redis');

app.use(session({
    store: new RedisStore({ client: redis }),
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: process.env.NODE_ENV === 'production',
        maxAge: 24 * 60 * 60 * 1000 // 24 hours
    }
}));
```

**Option 2: JWT (Stateless)**

```javascript
// No server-side session needed
// Token is self-contained and can be verified by any server
const token = jwt.sign({ userId: user.id }, JWT_SECRET);
```

### Application-Level Scaling

**PM2 Cluster Mode** (Node.js):

```bash
npm install -g pm2
```

```javascript
// ecosystem.config.js
module.exports = {
    apps: [{
        name: 'college-api',
        script: './server.js',
        instances: 'max', // Use all CPU cores
        exec_mode: 'cluster',
        env: {
            NODE_ENV: 'production',
            PORT: 3000
        }
    }]
};
```

```bash
# Start with PM2
pm2 start ecosystem.config.js

# Monitor
pm2 monit

# Logs
pm2 logs
```

**Gunicorn Workers** (Python):

```bash
# Install
pip install gunicorn

# Run with multiple workers
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app --bind 0.0.0.0:8000

# -w 4: 4 worker processes
# -k: Worker class (uvicorn for async)
```

---

## Performance Monitoring

### Application Performance Monitoring (APM)

**Key Metrics to Track**:

1. **Response Time**: How long requests take
2. **Throughput**: Requests per second
3. **Error Rate**: Percentage of failed requests
4. **CPU/Memory Usage**: Resource utilization
5. **Database Query Time**: Slow query detection

### Performance Middleware

```javascript
// middleware/performanceMonitor.js
const logger = require('../config/logger');

function performanceMonitor(req, res, next) {
    const start = Date.now();

    // Capture response
    res.on('finish', () => {
        const duration = Date.now() - start;

        // Log slow requests
        if (duration > 1000) {
            logger.warn('Slow request detected', {
                method: req.method,
                url: req.url,
                duration: `${duration}ms`,
                statusCode: res.statusCode
            });
        }

        // Log all requests
        logger.info('Request completed', {
            method: req.method,
            url: req.url,
            duration: `${duration}ms`,
            statusCode: res.statusCode,
            userAgent: req.get('user-agent')
        });
    });

    next();
}

module.exports = performanceMonitor;
```

### Database Query Performance Tracking

```javascript
// Track slow queries
const sequelize = new Sequelize(DATABASE_URL, {
    logging: (sql, timing) => {
        if (timing > 1000) { // Queries over 1 second
            logger.warn('Slow query detected', {
                query: sql,
                duration: `${timing}ms`
            });
        }
    },
    benchmark: true
});
```

### Memory Leak Detection

```javascript
// Monitor memory usage
setInterval(() => {
    const usage = process.memoryUsage();

    logger.info('Memory usage', {
        rss: `${Math.round(usage.rss / 1024 / 1024)}MB`,
        heapTotal: `${Math.round(usage.heapTotal / 1024 / 1024)}MB`,
        heapUsed: `${Math.round(usage.heapUsed / 1024 / 1024)}MB`,
        external: `${Math.round(usage.external / 1024 / 1024)}MB`
    });

    // Alert if heap usage is high
    const heapPercent = (usage.heapUsed / usage.heapTotal) * 100;
    if (heapPercent > 90) {
        logger.error('High heap usage detected', { heapPercent });
    }
}, 60000); // Every minute
```

### Response Time Histogram

```javascript
// Track response time distribution
const responseTimeBuckets = {
    '<100ms': 0,
    '100-300ms': 0,
    '300-500ms': 0,
    '500-1000ms': 0,
    '>1000ms': 0
};

function trackResponseTime(duration) {
    if (duration < 100) responseTimeBuckets['<100ms']++;
    else if (duration < 300) responseTimeBuckets['100-300ms']++;
    else if (duration < 500) responseTimeBuckets['300-500ms']++;
    else if (duration < 1000) responseTimeBuckets['500-1000ms']++;
    else responseTimeBuckets['>1000ms']++;
}

// Log histogram every 5 minutes
setInterval(() => {
    logger.info('Response time distribution', responseTimeBuckets);
}, 5 * 60 * 1000);
```

### Health Check Endpoint

```javascript
// routes/health.js
const router = require('express').Router();
const sequelize = require('../config/database');
const redis = require('../config/redis');

router.get('/health', async (req, res) => {
    const health = {
        status: 'healthy',
        timestamp: new Date().toISOString(),
        uptime: process.uptime(),
        memory: process.memoryUsage(),
        services: {}
    };

    // Check database
    try {
        await sequelize.authenticate();
        health.services.database = 'connected';
    } catch (error) {
        health.services.database = 'disconnected';
        health.status = 'unhealthy';
    }

    // Check Redis
    try {
        await redis.ping();
        health.services.redis = 'connected';
    } catch (error) {
        health.services.redis = 'disconnected';
        health.status = 'degraded';
    }

    const statusCode = health.status === 'healthy' ? 200 : 503;
    res.status(statusCode).json(health);
});

module.exports = router;
```

---

## Best Practices

### 1. Cache Strategy Guidelines

- ✅ Cache frequently accessed, rarely changing data
- ✅ Set appropriate TTL based on data freshness requirements
- ✅ Invalidate cache on data updates
- ✅ Use cache-aside pattern for most cases
- ✅ Monitor cache hit/miss ratios
- ❌ Don't cache user-specific sensitive data without encryption
- ❌ Don't cache everything (wastes memory)

### 2. Database Optimization

- ✅ Add indexes to columns used in WHERE, JOIN, ORDER BY
- ✅ Use connection pooling
- ✅ Paginate large result sets
- ✅ Select only needed columns
- ✅ Use eager loading to prevent N+1 queries
- ✅ Analyze slow queries with EXPLAIN
- ❌ Don't over-index (slows writes)
- ❌ Don't load unnecessary data

### 3. API Performance

- ✅ Implement rate limiting
- ✅ Use compression (gzip)
- ✅ Enable HTTP/2
- ✅ Cache responses when possible
- ✅ Use CDN for static assets
- ❌ Don't expose internal errors to clients
- ❌ Don't perform heavy computation in request handlers

### 4. Monitoring

- ✅ Track response times and error rates
- ✅ Set up alerts for anomalies
- ✅ Monitor database query performance
- ✅ Track memory and CPU usage
- ✅ Implement health check endpoints
- ❌ Don't ignore slow query warnings
- ❌ Don't deploy without monitoring

### 5. Scaling Checklist

Before scaling:
- [ ] Database is optimized (indexes, queries)
- [ ] Caching is implemented
- [ ] Sessions are stored in Redis/JWT
- [ ] Static assets are on CDN
- [ ] Rate limiting is configured
- [ ] Health checks are implemented
- [ ] Logging and monitoring are set up
- [ ] Load balancer is configured
- [ ] Horizontal scaling tested

---

## Complete Example: Optimized Student Service

```javascript
// services/studentService.js
const studentRepository = require('../repositories/studentRepository');
const cacheService = require('./cacheService');
const logger = require('../config/logger');

class StudentService {
    /**
     * Get student by ID (with caching)
     */
    async getStudentById(id) {
        const startTime = Date.now();
        const cacheKey = `student:${id}`;

        try {
            // Try cache first
            const cached = await cacheService.get(cacheKey);
            if (cached) {
                logger.debug('Cache hit', { key: cacheKey });
                return cached;
            }

            // Cache miss - fetch from database
            logger.debug('Cache miss', { key: cacheKey });
            const student = await studentRepository.findById(id);

            if (!student) {
                throw new NotFoundError(`Student with ID ${id} not found`);
            }

            // Cache for 10 minutes
            await cacheService.set(cacheKey, student, 600);

            return student;
        } finally {
            const duration = Date.now() - startTime;
            logger.info('getStudentById completed', { id, duration: `${duration}ms` });
        }
    }

    /**
     * Get students with pagination and filtering
     */
    async getStudents(options = {}) {
        const {
            page = 1,
            limit = 20,
            departmentId,
            isActive = true,
            search
        } = options;

        // Create cache key from options
        const cacheKey = `students:list:${JSON.stringify(options)}`;

        // Try cache
        const cached = await cacheService.get(cacheKey);
        if (cached) {
            return cached;
        }

        // Build filters
        const filters = { isActive };
        if (departmentId) filters.departmentId = departmentId;
        if (search) filters.search = search;

        // Fetch with pagination
        const result = await studentRepository.findAllPaginated({
            filters,
            page,
            limit,
            include: ['department'] // Eager load to prevent N+1
        });

        // Cache for 5 minutes
        await cacheService.set(cacheKey, result, 300);

        return result;
    }

    /**
     * Update student (invalidates caches)
     */
    async updateStudent(id, data) {
        const student = await studentRepository.update(id, data);

        // Invalidate caches
        await cacheService.delete(`student:${id}`);
        await cacheService.deletePattern('students:list:*');

        logger.info('Student updated and caches invalidated', { id });

        return student;
    }

    /**
     * Get student with enrollments (complex query with caching)
     */
    async getStudentWithEnrollments(id) {
        const cacheKey = `student:${id}:enrollments`;

        return await cacheService.getOrSet(
            cacheKey,
            async () => {
                // Single query with joins to prevent N+1
                return await studentRepository.findByIdWithEnrollments(id);
            },
            600 // 10 minutes
        );
    }

    /**
     * Get student statistics (expensive query, long cache)
     */
    async getStatistics(departmentId) {
        const cacheKey = `stats:department:${departmentId}`;

        return await cacheService.getOrSet(
            cacheKey,
            async () => {
                return await studentRepository.getDepartmentStatistics(departmentId);
            },
            3600 // 1 hour
        );
    }
}

module.exports = new StudentService();
```

---

## Summary

### Performance Optimization Hierarchy

1. **Database** (Biggest impact)
   - Add indexes
   - Optimize queries
   - Use connection pooling
   - Paginate results

2. **Caching** (High impact)
   - Redis for distributed caching
   - Cache frequently accessed data
   - Implement cache invalidation

3. **API Design** (Medium impact)
   - Rate limiting
   - Compression
   - Efficient serialization

4. **Scaling** (When needed)
   - Horizontal scaling with load balancer
   - PM2 cluster mode
   - CDN for static assets

### Key Takeaways

- **Cache intelligently**: Not everything needs caching
- **Index strategically**: Too many indexes hurt write performance
- **Monitor always**: Can't optimize what you don't measure
- **Scale when ready**: Optimize first, then scale
- **Think ahead**: Design for scalability from the start

---

**Next**: [Part 10: Testing and Deployment](./10_Testing_Deployment.md)

**Previous**: [Part 8: Validation, Error Handling, and Logging](./08_Validation_Error_Handling.md)

[← Back to Index](./README.md)

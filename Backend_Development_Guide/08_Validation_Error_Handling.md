# Part 8: Validation, Error Handling, and Logging

**Author**: Saiprasad Hegde

---

## Table of Contents
1. [Input Validation](#input-validation)
2. [Validation Libraries](#validation-libraries)
3. [Error Handling Patterns](#error-handling-patterns)
4. [Custom Error Classes](#custom-error-classes)
5. [Logging](#logging)
6. [Monitoring and Debugging](#monitoring-and-debugging)

---

## Input Validation

### Why Validate?

```
User Input → Validation → Process → Database
                ↓
         Block bad data
         before it causes
         damage!
```

**Without Validation**:
```javascript
// ❌ Dangerous!
const student = await Student.create({
    email: req.body.email,  // Could be "not-an-email"
    age: req.body.age      // Could be -999 or "abc"
});
```

**With Validation**:
```javascript
// ✅ Safe
const validatedData = validateStudent(req.body);
if (!validatedData.valid) {
    return res.status(400).json({ errors: validatedData.errors });
}

const student = await Student.create(validatedData.data);
```

### Types of Validation

**1. Format Validation**: Is it the right format?
```javascript
// Email format
email: "not-an-email"  // ❌ Invalid
email: "john@college.edu"  // ✅ Valid

// Date format
dateOfBirth: "abc"  // ❌ Invalid
dateOfBirth: "2000-01-15"  // ✅ Valid
```

**2. Type Validation**: Is it the right type?
```javascript
age: "twenty"  // ❌ Should be number
age: 20  // ✅ Correct type
```

**3. Range Validation**: Is it within acceptable range?
```javascript
age: -5  // ❌ Out of range
age: 500  // ❌ Out of range
age: 20  // ✅ In range
```

**4. Business Validation**: Does it follow business rules?
```javascript
credits: 25  // ❌ Exceeds max (18 credits/semester)
credits: 15  // ✅ Within limit
```

---

## Validation Libraries

### Joi (Node.js)

```bash
npm install joi
```

**validators/studentValidator.js**:
```javascript
const Joi = require('joi');

const studentSchema = Joi.object({
    firstName: Joi.string()
        .min(1)
        .max(50)
        .required()
        .messages({
            'string.empty': 'First name is required',
            'string.max': 'First name must be at most 50 characters'
        }),

    lastName: Joi.string()
        .min(1)
        .max(50)
        .required(),

    email: Joi.string()
        .email()
        .required()
        .messages({
            'string.email': 'Please provide a valid email address'
        }),

    dateOfBirth: Joi.date()
        .max('now')
        .optional()
        .messages({
            'date.max': 'Date of birth cannot be in the future'
        }),

    department: Joi.string()
        .valid('CS', 'Math', 'Physics', 'English')
        .required()
        .messages({
            'any.only': 'Department must be one of CS, Math, Physics, or English'
        }),

    enrollmentYear: Joi.number()
        .integer()
        .min(2000)
        .max(2100)
        .required(),

    gpa: Joi.number()
        .min(0.0)
        .max(4.0)
        .precision(2)
        .optional()
}).options({ abortEarly: false });  // Return all errors, not just first

function validateStudent(data) {
    const { error, value } = studentSchema.validate(data);

    if (error) {
        const errors = error.details.map(detail => ({
            field: detail.path.join('.'),
            message: detail.message
        }));

        return {
            valid: false,
            errors
        };
    }

    return {
        valid: true,
        data: value
    };
}

module.exports = { validateStudent };
```

**Middleware**:
```javascript
function validateStudentMiddleware(req, res, next) {
    const result = validateStudent(req.body);

    if (!result.valid) {
        return res.status(400).json({
            success: false,
            error: 'Validation failed',
            details: result.errors
        });
    }

    req.validatedData = result.data;
    next();
}

// Usage
router.post('/students',
    validateStudentMiddleware,
    studentController.createStudent
);
```

### Complex Validations

```javascript
const enrollmentSchema = Joi.object({
    studentId: Joi.string()
        .uuid()
        .required(),

    courseId: Joi.string()
        .uuid()
        .required(),

    semester: Joi.string()
        .pattern(/^(Spring|Summer|Fall) \d{4}$/)
        .required()
        .messages({
            'string.pattern.base': 'Semester must be in format "Spring 2025"'
        })
});

const gradeSchema = Joi.object({
    grade: Joi.string()
        .valid('A', 'A-', 'B+', 'B', 'B-', 'C+', 'C', 'C-', 'D+', 'D', 'F')
        .required()
        .messages({
            'any.only': 'Invalid grade. Must be A, A-, B+, B, B-, C+, C, C-, D+, D, or F'
        })
});

// Custom validation function
const passwordSchema = Joi.string()
    .min(8)
    .custom((value, helpers) => {
        if (!/[A-Z]/.test(value)) {
            return helpers.error('password.uppercase');
        }
        if (!/[a-z]/.test(value)) {
            return helpers.error('password.lowercase');
        }
        if (!/\d/.test(value)) {
            return helpers.error('password.number');
        }
        if (!/[!@#$%^&*]/.test(value)) {
            return helpers.error('password.special');
        }
        return value;
    })
    .messages({
        'password.uppercase': 'Password must contain at least one uppercase letter',
        'password.lowercase': 'Password must contain at least one lowercase letter',
        'password.number': 'Password must contain at least one number',
        'password.special': 'Password must contain at least one special character (!@#$%^&*)'
    });
```

### Pydantic (Python/FastAPI)

```python
from pydantic import BaseModel, EmailStr, Field, validator
from datetime import date
from typing import Optional
from enum import Enum

class Department(str, Enum):
    CS = "CS"
    MATH = "Math"
    PHYSICS = "Physics"
    ENGLISH = "English"

class StudentCreate(BaseModel):
    first_name: str = Field(..., min_length=1, max_length=50)
    last_name: str = Field(..., min_length=1, max_length=50)
    email: EmailStr
    date_of_birth: Optional[date] = None
    department: Department
    enrollment_year: int = Field(..., ge=2000, le=2100)

    @validator('date_of_birth')
    def validate_date_of_birth(cls, v):
        if v and v > date.today():
            raise ValueError('Date of birth cannot be in the future')
        return v

    @validator('enrollment_year')
    def validate_enrollment_year(cls, v):
        current_year = date.today().year
        if v > current_year + 1:
            raise ValueError('Enrollment year cannot be more than 1 year in future')
        return v

class EnrollmentCreate(BaseModel):
    student_id: str = Field(..., regex="^[0-9a-f-]{36}$")
    course_id: str = Field(..., regex="^[0-9a-f-]{36}$")
    semester: str = Field(..., regex="^(Spring|Summer|Fall) \\d{4}$")

class GradeSubmit(BaseModel):
    grade: str = Field(...)

    @validator('grade')
    def validate_grade(cls, v):
        valid_grades = ['A', 'A-', 'B+', 'B', 'B-', 'C+', 'C', 'C-', 'D+', 'D', 'F']
        if v not in valid_grades:
            raise ValueError(f'Invalid grade. Must be one of: {", ".join(valid_grades)}')
        return v

# FastAPI automatically validates
@router.post("/students")
async def create_student(student: StudentCreate):  # Automatically validated!
    return await student_service.create(student.dict())
```

---

## Error Handling Patterns

### Centralized Error Handling

**errors/AppError.js**:
```javascript
class AppError extends Error {
    constructor(message, statusCode = 500, isOperational = true) {
        super(message);
        this.statusCode = statusCode;
        this.isOperational = isOperational;
        this.timestamp = new Date().toISOString();

        Error.captureStackTrace(this, this.constructor);
    }
}

class ValidationError extends AppError {
    constructor(message, details = []) {
        super(message, 400);
        this.details = details;
    }
}

class NotFoundError extends AppError {
    constructor(resource = 'Resource') {
        super(`${resource} not found`, 404);
    }
}

class UnauthorizedError extends AppError {
    constructor(message = 'Unauthorized') {
        super(message, 401);
    }
}

class ForbiddenError extends AppError {
    constructor(message = 'Forbidden') {
        super(message, 403);
    }
}

class ConflictError extends AppError {
    constructor(message) {
        super(message, 409);
    }
}

module.exports = {
    AppError,
    ValidationError,
    NotFoundError,
    UnauthorizedError,
    ForbiddenError,
    ConflictError
};
```

### Error Handling Middleware

**middlewares/errorHandler.js**:
```javascript
const { AppError } = require('../errors/AppError');
const logger = require('../utils/logger');

function errorHandler(err, req, res, next) {
    // Log error
    logger.error({
        message: err.message,
        stack: err.stack,
        url: req.url,
        method: req.method,
        ip: req.ip,
        userId: req.user?.id
    });

    // Operational errors (expected)
    if (err.isOperational) {
        return res.status(err.statusCode).json({
            success: false,
            error: err.message,
            ...(err.details && { details: err.details }),
            ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
        });
    }

    // Programming errors (unexpected)
    console.error('UNEXPECTED ERROR:', err);

    // Don't leak error details in production
    res.status(500).json({
        success: false,
        error: process.env.NODE_ENV === 'production'
            ? 'Internal server error'
            : err.message,
        ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    });
}

// 404 handler
function notFoundHandler(req, res) {
    res.status(404).json({
        success: false,
        error: `Route ${req.method} ${req.url} not found`
    });
}

// Async error wrapper
function asyncHandler(fn) {
    return (req, res, next) => {
        Promise.resolve(fn(req, res, next)).catch(next);
    };
}

module.exports = { errorHandler, notFoundHandler, asyncHandler };
```

### Using Error Classes

```javascript
const { NotFoundError, ValidationError, ConflictError } = require('../errors/AppError');

class StudentService {
    async getById(id) {
        const student = await this.studentRepo.findById(id);

        if (!student) {
            throw new NotFoundError('Student');
        }

        return student;
    }

    async create(data) {
        // Check email exists
        const existing = await this.studentRepo.findByEmail(data.email);

        if (existing) {
            throw new ConflictError('Email already registered');
        }

        // Validate age
        if (data.age < 16) {
            throw new ValidationError('Student must be at least 16 years old');
        }

        return await this.studentRepo.create(data);
    }
}

// In controller
const { asyncHandler } = require('../middlewares/errorHandler');

router.get('/students/:id', asyncHandler(async (req, res) => {
    const student = await studentService.getById(req.params.id);
    // Errors automatically caught and handled!

    res.json({ success: true, data: student });
}));
```

### FastAPI Error Handling

```python
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from pydantic import ValidationError

app = FastAPI()

# Custom exception classes
class AppError(Exception):
    def __init__(self, message: str, status_code: int = 500):
        self.message = message
        self.status_code = status_code

class NotFoundError(AppError):
    def __init__(self, resource: str = "Resource"):
        super().__init__(f"{resource} not found", 404)

class ConflictError(AppError):
    def __init__(self, message: str):
        super().__init__(message, 409)

# Global exception handlers
@app.exception_handler(AppError)
async def app_error_handler(request: Request, exc: AppError):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "success": False,
            "error": exc.message
        }
    )

@app.exception_handler(RequestValidationError)
async def validation_error_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "success": False,
            "error": "Validation error",
            "details": exc.errors()
        }
    )

@app.exception_handler(Exception)
async def global_error_handler(request: Request, exc: Exception):
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={
            "success": False,
            "error": "Internal server error"
        }
    )

# Usage
async def get_student(student_id: str):
    student = await student_repo.find_by_id(student_id)
    if not student:
        raise NotFoundError("Student")
    return student
```

---

## Custom Error Classes

### Domain-Specific Errors

```javascript
// errors/enrollmentErrors.js
class EnrollmentError extends AppError {
    constructor(message) {
        super(message, 400);
    }
}

class PrerequisitesNotMetError extends EnrollmentError {
    constructor(missingCourses) {
        super(`Prerequisites not met: ${missingCourses.join(', ')}`);
        this.missingCourses = missingCourses;
    }
}

class CourseFull error extends EnrollmentError {
    constructor(courseName) {
        super(`Course ${courseName} is at maximum capacity`);
    }
}

class CreditLimitExceededError extends EnrollmentError {
    constructor(currentCredits, attemptedCredits, maxCredits) {
        super(`Adding ${attemptedCredits} credits would exceed maximum of ${maxCredits} (currently enrolled in ${currentCredits})`);
        this.currentCredits = currentCredits;
        this.attemptedCredits = attemptedCredits;
        this.maxCredits = maxCredits;
    }
}

// Usage
async enrollStudent(studentId, courseId) {
    // ...checks...

    if (missingPrereqs.length > 0) {
        throw new PrerequisitesNotMetError(missingPrereqs);
    }

    if (enrolledCount >= course.maxCapacity) {
        throw new CourseFullError(course.name);
    }

    if (currentCredits + course.credits > MAX_CREDITS) {
        throw new CreditLimitExceededError(
            currentCredits,
            course.credits,
            MAX_CREDITS
        );
    }
}
```

---

## Logging

### Winston (Node.js)

```bash
npm install winston
```

**utils/logger.js**:
```javascript
const winston = require('winston');

const logger = winston.createLogger({
    level: process.env.LOG_LEVEL || 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
    ),
    defaultMeta: { service: 'college-api' },
    transports: [
        // Write all logs to files
        new winston.transports.File({
            filename: 'logs/error.log',
            level: 'error'
        }),
        new winston.transports.File({
            filename: 'logs/combined.log'
        })
    ]
});

// Console logging in development
if (process.env.NODE_ENV !== 'production') {
    logger.add(new winston.transports.Console({
        format: winston.format.combine(
            winston.format.colorize(),
            winston.format.simple()
        )
    }));
}

module.exports = logger;
```

**Usage**:
```javascript
const logger = require('./utils/logger');

// Different log levels
logger.error('Database connection failed', { error: err });
logger.warn('Slow query detected', { query: sql, duration: 1500 });
logger.info('Student enrolled', { studentId, courseId });
logger.debug('Cache hit', { key: cacheKey });

// Log HTTP requests
app.use((req, res, next) => {
    const start = Date.now();

    res.on('finish', () => {
        const duration = Date.now() - start;

        logger.info('HTTP Request', {
            method: req.method,
            url: req.url,
            status: res.statusCode,
            duration,
            ip: req.ip,
            userId: req.user?.id
        });
    });

    next();
});

// Log errors
logger.error('Enrollment failed', {
    error: err.message,
    stack: err.stack,
    studentId,
    courseId,
    userId: req.user?.id
});
```

### Structured Logging

```javascript
// ❌ Bad - hard to parse
logger.info(`Student ${studentId} enrolled in ${courseId}`);

// ✅ Good - structured, easy to query
logger.info('Student enrolled', {
    studentId,
    courseId,
    semester,
    timestamp: new Date().toISOString()
});
```

### Python Logging

```python
import logging
from logging.handlers import RotatingFileHandler
import json
from datetime import datetime

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger(__name__)

# File handler with rotation
file_handler = RotatingFileHandler(
    'logs/app.log',
    maxBytes=10485760,  # 10MB
    backupCount=5
)

file_handler.setFormatter(
    logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
)

logger.addHandler(file_handler)

# Usage
logger.info(f"Student enrolled", extra={
    "student_id": student_id,
    "course_id": course_id,
    "semester": semester
})

logger.error(f"Enrollment failed", extra={
    "error": str(error),
    "student_id": student_id
}, exc_info=True)
```

---

## Monitoring and Debugging

### Request ID Tracking

```javascript
const { v4: uuidv4 } = require('uuid');

// Middleware to add request ID
app.use((req, res, next) => {
    req.id = uuidv4();
    res.setHeader('X-Request-ID', req.id);
    next();
});

// Log with request ID
logger.info('Request started', {
    requestId: req.id,
    method: req.method,
    url: req.url
});

logger.error('Error occurred', {
    requestId: req.id,
    error: err.message
});
```

### Health Check Endpoint

```javascript
router.get('/health', async (req, res) => {
    const health = {
        status: 'OK',
        timestamp: new Date().toISOString(),
        uptime: process.uptime(),
        environment: process.env.NODE_ENV
    };

    // Check database connection
    try {
        await sequelize.authenticate();
        health.database = 'connected';
    } catch (error) {
        health.database = 'disconnected';
        health.status = 'ERROR';
    }

    // Check Redis connection
    try {
        await redis.ping();
        health.cache = 'connected';
    } catch (error) {
        health.cache = 'disconnected';
    }

    const statusCode = health.status === 'OK' ? 200 : 503;
    res.status(statusCode).json(health);
});
```

### Performance Monitoring

```javascript
// Track slow queries
function trackQuery(sql, duration) {
    if (duration > 1000) {  // Slower than 1 second
        logger.warn('Slow query detected', {
            sql,
            duration,
            threshold: 1000
        });
    }
}

// Track memory usage
setInterval(() => {
    const usage = process.memoryUsage();
    logger.info('Memory usage', {
        rss: Math.round(usage.rss / 1024 / 1024),
        heapUsed: Math.round(usage.heapUsed / 1024 / 1024),
        heapTotal: Math.round(usage.heapTotal / 1024 / 1024)
    });
}, 60000);  // Every minute
```

---

## Key Takeaways

1. **Validate early** - At API layer, before processing
2. **Use validation libraries** - Joi, Pydantic for consistency
3. **Custom error classes** - Domain-specific, informative
4. **Centralized error handling** - One place to handle all errors
5. **Structured logging** - JSON format, easy to query
6. **Monitor performance** - Track slow queries, memory usage
7. **Never expose sensitive info** - Stack traces only in development

---

**Next**: [09 - Performance and Caching →](09_Performance_Caching.md)

---

**Previous**: [← 07 - Authentication](07_Authentication_Authorization.md) | **Next**: [09 - Performance →](09_Performance_Caching.md)

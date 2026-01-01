# Part 3: API Layer Deep Dive

**Author**: Saiprasad Hegde

---

## Table of Contents
1. [What is an API?](#what-is-an-api)
2. [RESTful API Design Principles](#restful-api-design-principles)
3. [Express.js Implementation](#expressjs-implementation)
4. [Middleware and Request Processing](#middleware-and-request-processing)
5. [Error Handling in API Layer](#error-handling-in-api-layer)
6. [API Documentation](#api-documentation)

---

## What is an API?

**API (Application Programming Interface)** is a contract that defines how clients communicate with your backend.

### Analogy: Restaurant Menu

Think of an API as a restaurant menu:

- **Menu (API Documentation)**: Lists what you can order and how
- **Menu Items (Endpoints)**: Specific dishes you can request
- **Order (HTTP Request)**: You tell the waiter what you want
- **Food (HTTP Response)**: The kitchen prepares and delivers it

**Example**:
```
Menu Item: "Caesar Salad - $12"
↓
API Endpoint: "GET /api/courses/:id"

Your Order: "One Caesar Salad, please"
↓
HTTP Request: GET /api/courses/CS101

Kitchen Prepares: Chef makes the salad
↓
Service Layer: Fetches course data from database

Food Delivered: Waiter brings your salad
↓
HTTP Response: { "id": "CS101", "name": "Introduction to Programming", ... }
```

### HTTP Methods (Verbs)

| Method | Purpose | Example | Analogy |
|--------|---------|---------|---------|
| GET | Retrieve data | Get student info | Read a book |
| POST | Create new data | Add new student | Write a new chapter |
| PUT | Update entire resource | Replace student info | Rewrite entire chapter |
| PATCH | Update partial resource | Change student email | Edit part of chapter |
| DELETE | Remove data | Delete student | Tear out a page |

---

## RESTful API Design Principles

**REST (Representational State Transfer)** is an architectural style for designing APIs.

### 1. Resource-Based URLs

**Good** ✅:
```
GET    /api/students              - Get all students
GET    /api/students/123          - Get specific student
POST   /api/students              - Create student
PUT    /api/students/123          - Update student
DELETE /api/students/123          - Delete student

GET    /api/courses               - Get all courses
POST   /api/courses/CS101/enroll  - Enroll in course
```

**Bad** ❌:
```
GET    /api/getStudents           - Don't use verbs in URLs
POST   /api/createStudent         - HTTP method already indicates action
GET    /api/student?action=delete - Don't use query params for actions
```

### 2. Use HTTP Status Codes Correctly

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH, DELETE |
| 201 | Created | Successful POST (resource created) |
| 204 | No Content | Successful DELETE (no response body) |
| 400 | Bad Request | Invalid input data |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource (email already exists) |
| 500 | Internal Server Error | Server error |

### 3. Use Proper HTTP Methods

```javascript
// ✅ Correct
GET    /api/students       - List students (safe, idempotent)
POST   /api/students       - Create student (not idempotent)
PUT    /api/students/123   - Update student (idempotent)
DELETE /api/students/123   - Delete student (idempotent)

// ❌ Wrong
POST   /api/getStudents    - Should be GET
GET    /api/deleteStudent  - Should be DELETE
```

**Idempotent**: Multiple identical requests have the same effect as one request

### 4. Versioning

```
/api/v1/students
/api/v2/students
```

### 5. Filtering, Sorting, Pagination

```
GET /api/students?department=CS&year=2024          - Filter
GET /api/students?sort=lastName&order=asc          - Sort
GET /api/students?page=2&limit=20                  - Pagination
GET /api/students?search=john                      - Search
```

### 6. Nested Resources

```
GET /api/students/123/courses           - Get student's courses
GET /api/courses/CS101/students         - Get course's students
POST /api/students/123/courses/CS101    - Enroll student in course
```

---

## Express.js Implementation

### Project Structure

```
college-api/
├── src/
│   ├── controllers/        # Handle HTTP requests
│   │   ├── studentController.js
│   │   ├── courseController.js
│   │   └── enrollmentController.js
│   ├── routes/            # Define endpoints
│   │   ├── index.js
│   │   ├── studentRoutes.js
│   │   ├── courseRoutes.js
│   │   └── enrollmentRoutes.js
│   ├── middlewares/       # Request processing
│   │   ├── auth.js
│   │   ├── validation.js
│   │   └── errorHandler.js
│   ├── services/          # Business logic
│   ├── repositories/      # Data access
│   ├── models/           # Database models
│   └── app.js            # Express setup
├── package.json
└── server.js             # Entry point
```

### Setting Up Express

**server.js**:
```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(helmet());                          // Security headers
app.use(cors());                            // Enable CORS
app.use(morgan('dev'));                     // Logging
app.use(express.json());                    // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies

// Routes
const routes = require('./routes');
app.use('/api/v1', routes);

// Error handling
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(err.statusCode || 500).json({
        success: false,
        error: err.message || 'Internal Server Error'
    });
});

// 404 handler
app.use((req, res) => {
    res.status(404).json({
        success: false,
        error: 'Route not found'
    });
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

### Routes

**routes/index.js**:
```javascript
const express = require('express');
const router = express.Router();

const studentRoutes = require('./studentRoutes');
const courseRoutes = require('./courseRoutes');
const enrollmentRoutes = require('./enrollmentRoutes');
const authRoutes = require('./authRoutes');

// Mount routes
router.use('/auth', authRoutes);
router.use('/students', studentRoutes);
router.use('/courses', courseRoutes);
router.use('/enrollments', enrollmentRoutes);

// Health check
router.get('/health', (req, res) => {
    res.json({ status: 'OK', timestamp: new Date().toISOString() });
});

module.exports = router;
```

**routes/studentRoutes.js**:
```javascript
const express = require('express');
const router = express.Router();
const studentController = require('../controllers/studentController');
const { authenticate, authorize } = require('../middlewares/auth');
const { validateStudent } = require('../middlewares/validation');

// Public routes
router.get('/', studentController.getAllStudents);
router.get('/:id', studentController.getStudentById);

// Protected routes (require authentication)
router.use(authenticate);

// Student-specific routes
router.get('/me', studentController.getCurrentStudent);
router.put('/me', validateStudent, studentController.updateCurrentStudent);

// Admin-only routes
router.post('/',
    authorize(['admin']),
    validateStudent,
    studentController.createStudent
);

router.put('/:id',
    authorize(['admin']),
    validateStudent,
    studentController.updateStudent
);

router.delete('/:id',
    authorize(['admin']),
    studentController.deleteStudent
);

// Nested resources
router.get('/:id/courses', studentController.getStudentCourses);
router.get('/:id/grades', studentController.getStudentGrades);
router.get('/:id/transcript', studentController.getTranscript);

module.exports = router;
```

### Controllers

**controllers/studentController.js**:
```javascript
const studentService = require('../services/studentService');

class StudentController {
    // GET /api/students
    async getAllStudents(req, res, next) {
        try {
            const { page = 1, limit = 20, department, year, search } = req.query;

            const options = {
                page: parseInt(page),
                limit: parseInt(limit),
                department,
                year: year ? parseInt(year) : undefined,
                search
            };

            const result = await studentService.getAllStudents(options);

            res.json({
                success: true,
                data: result.students,
                pagination: {
                    page: result.page,
                    limit: result.limit,
                    total: result.total,
                    pages: result.pages
                }
            });
        } catch (error) {
            next(error);
        }
    }

    // GET /api/students/:id
    async getStudentById(req, res, next) {
        try {
            const student = await studentService.getStudentById(req.params.id);

            if (!student) {
                return res.status(404).json({
                    success: false,
                    error: 'Student not found'
                });
            }

            res.json({
                success: true,
                data: student
            });
        } catch (error) {
            next(error);
        }
    }

    // POST /api/students
    async createStudent(req, res, next) {
        try {
            const studentData = req.body;
            const student = await studentService.createStudent(studentData);

            res.status(201).json({
                success: true,
                data: student,
                message: 'Student created successfully'
            });
        } catch (error) {
            if (error.name === 'ValidationError') {
                return res.status(400).json({
                    success: false,
                    error: error.message,
                    details: error.details
                });
            }
            if (error.code === 'DUPLICATE_EMAIL') {
                return res.status(409).json({
                    success: false,
                    error: 'Email already exists'
                });
            }
            next(error);
        }
    }

    // PUT /api/students/:id
    async updateStudent(req, res, next) {
        try {
            const { id } = req.params;
            const updates = req.body;

            const student = await studentService.updateStudent(id, updates);

            if (!student) {
                return res.status(404).json({
                    success: false,
                    error: 'Student not found'
                });
            }

            res.json({
                success: true,
                data: student,
                message: 'Student updated successfully'
            });
        } catch (error) {
            next(error);
        }
    }

    // DELETE /api/students/:id
    async deleteStudent(req, res, next) {
        try {
            const { id } = req.params;
            const deleted = await studentService.deleteStudent(id);

            if (!deleted) {
                return res.status(404).json({
                    success: false,
                    error: 'Student not found'
                });
            }

            res.status(204).send(); // No content
        } catch (error) {
            next(error);
        }
    }

    // GET /api/students/:id/courses
    async getStudentCourses(req, res, next) {
        try {
            const { id } = req.params;
            const { semester, status } = req.query;

            const courses = await studentService.getStudentCourses(id, {
                semester,
                status
            });

            res.json({
                success: true,
                data: courses
            });
        } catch (error) {
            next(error);
        }
    }

    // GET /api/students/:id/grades
    async getStudentGrades(req, res, next) {
        try {
            const { id } = req.params;
            const { semester } = req.query;

            const grades = await studentService.getStudentGrades(id, semester);

            res.json({
                success: true,
                data: grades
            });
        } catch (error) {
            next(error);
        }
    }

    // GET /api/students/:id/transcript
    async getTranscript(req, res, next) {
        try {
            const { id } = req.params;
            const transcript = await studentService.generateTranscript(id);

            res.json({
                success: true,
                data: transcript
            });
        } catch (error) {
            next(error);
        }
    }

    // GET /api/students/me
    async getCurrentStudent(req, res, next) {
        try {
            // req.user is set by authenticate middleware
            const student = await studentService.getStudentById(req.user.id);

            res.json({
                success: true,
                data: student
            });
        } catch (error) {
            next(error);
        }
    }

    // PUT /api/students/me
    async updateCurrentStudent(req, res, next) {
        try {
            const updates = req.body;

            // Prevent updating sensitive fields
            delete updates.id;
            delete updates.enrollmentYear;
            delete updates.status;

            const student = await studentService.updateStudent(req.user.id, updates);

            res.json({
                success: true,
                data: student,
                message: 'Profile updated successfully'
            });
        } catch (error) {
            next(error);
        }
    }
}

module.exports = new StudentController();
```

**controllers/enrollmentController.js**:
```javascript
const enrollmentService = require('../services/enrollmentService');

class EnrollmentController {
    // POST /api/enrollments
    async enrollStudent(req, res, next) {
        try {
            const { studentId, courseId } = req.body;

            // If regular user, can only enroll themselves
            const actualStudentId = req.user.role === 'student'
                ? req.user.id
                : studentId;

            const enrollment = await enrollmentService.enrollStudent(
                actualStudentId,
                courseId
            );

            res.status(201).json({
                success: true,
                data: enrollment,
                message: 'Enrolled successfully'
            });
        } catch (error) {
            if (error.message.includes('not found')) {
                return res.status(404).json({
                    success: false,
                    error: error.message
                });
            }
            if (error.message.includes('already enrolled') ||
                error.message.includes('prerequisites') ||
                error.message.includes('capacity') ||
                error.message.includes('credit limit')) {
                return res.status(400).json({
                    success: false,
                    error: error.message
                });
            }
            next(error);
        }
    }

    // DELETE /api/enrollments/:id (drop course)
    async dropCourse(req, res, next) {
        try {
            const { id } = req.params;

            const dropped = await enrollmentService.dropCourse(
                id,
                req.user.id,
                req.user.role
            );

            if (!dropped) {
                return res.status(404).json({
                    success: false,
                    error: 'Enrollment not found'
                });
            }

            res.json({
                success: true,
                message: 'Course dropped successfully'
            });
        } catch (error) {
            if (error.message.includes('deadline')) {
                return res.status(400).json({
                    success: false,
                    error: error.message
                });
            }
            next(error);
        }
    }

    // GET /api/enrollments/:id
    async getEnrollment(req, res, next) {
        try {
            const enrollment = await enrollmentService.getEnrollmentById(
                req.params.id
            );

            if (!enrollment) {
                return res.status(404).json({
                    success: false,
                    error: 'Enrollment not found'
                });
            }

            res.json({
                success: true,
                data: enrollment
            });
        } catch (error) {
            next(error);
        }
    }
}

module.exports = new EnrollmentController();
```

---

## Middleware and Request Processing

### What is Middleware?

**Middleware** is code that runs BEFORE your route handlers.

**Analogy**: Security checkpoint at airport
1. You arrive (HTTP request)
2. Security checks ID (authentication middleware)
3. TSA scans bags (validation middleware)
4. You board plane (controller handles request)

### Middleware Flow

```
Request → Middleware 1 → Middleware 2 → Controller → Response
          ↓              ↓                ↓
          Auth           Validation       Business Logic
```

### Express Middleware

**middlewares/auth.js**:
```javascript
const jwt = require('jsonwebtoken');

// Authentication middleware
async function authenticate(req, res, next) {
    try {
        // Get token from header
        const authHeader = req.headers.authorization;

        if (!authHeader || !authHeader.startsWith('Bearer ')) {
            return res.status(401).json({
                success: false,
                error: 'No token provided'
            });
        }

        const token = authHeader.split(' ')[1];

        // Verify token
        const decoded = jwt.verify(token, process.env.JWT_SECRET);

        // Attach user to request
        req.user = decoded;

        next(); // Continue to next middleware or route
    } catch (error) {
        if (error.name === 'TokenExpiredError') {
            return res.status(401).json({
                success: false,
                error: 'Token expired'
            });
        }
        return res.status(401).json({
            success: false,
            error: 'Invalid token'
        });
    }
}

// Authorization middleware
function authorize(roles = []) {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({
                success: false,
                error: 'Unauthenticated'
            });
        }

        if (roles.length && !roles.includes(req.user.role)) {
            return res.status(403).json({
                success: false,
                error: 'Insufficient permissions'
            });
        }

        next();
    };
}

module.exports = { authenticate, authorize };
```

**middlewares/validation.js**:
```javascript
const Joi = require('joi');

const studentSchema = Joi.object({
    firstName: Joi.string().min(1).max(50).required(),
    lastName: Joi.string().min(1).max(50).required(),
    email: Joi.string().email().required(),
    department: Joi.string().required(),
    dateOfBirth: Joi.date().optional(),
    enrollmentYear: Joi.number().integer().min(2000).max(2100).required()
});

function validateStudent(req, res, next) {
    const { error, value } = studentSchema.validate(req.body, {
        abortEarly: false
    });

    if (error) {
        const errors = error.details.map(detail => ({
            field: detail.path.join('.'),
            message: detail.message
        }));

        return res.status(400).json({
            success: false,
            error: 'Validation failed',
            details: errors
        });
    }

    req.validatedData = value;
    next();
}

module.exports = { validateStudent };
```

**Using Middleware**:
```javascript
const { authenticate, authorize } = require('./middlewares/auth');
const { validateStudent } = require('./middlewares/validation');

// Apply to specific route
router.post('/students',
    authenticate,                    // First: check auth
    authorize(['admin']),            // Second: check role
    validateStudent,                 // Third: validate data
    studentController.createStudent  // Finally: handle request
);

// Apply to all routes after this point
router.use(authenticate);

router.get('/students/me', studentController.getCurrentStudent);
router.put('/students/me', studentController.updateCurrentStudent);
```

### Request Logging Middleware

```javascript
function requestLogger(req, res, next) {
    const start = Date.now();

    // Log when response finishes
    res.on('finish', () => {
        const duration = Date.now() - start;
        console.log(
            `${req.method} ${req.path} ${res.statusCode} - ${duration}ms`
        );
    });

    next();
}

app.use(requestLogger);
```

### Rate Limiting Middleware

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // Limit each IP to 100 requests per windowMs
    message: 'Too many requests, please try again later'
});

// Apply to all routes
app.use('/api/', limiter);

// Stricter limit for auth routes
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5
});

app.use('/api/auth/login', authLimiter);
```

---

## Error Handling in API Layer

### Centralized Error Handler (Express)

```javascript
// Custom error class
class AppError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode;
        this.isOperational = true;
        Error.captureStackTrace(this, this.constructor);
    }
}

// Error handler middleware
function errorHandler(err, req, res, next) {
    let { statusCode = 500, message } = err;

    // Log error
    console.error('Error:', {
        message: err.message,
        stack: err.stack,
        url: req.url,
        method: req.method
    });

    // Don't leak error details in production
    if (process.env.NODE_ENV === 'production' && !err.isOperational) {
        message = 'Internal server error';
    }

    res.status(statusCode).json({
        success: false,
        error: message,
        ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    });
}

// Usage in controller
async function getStudent(req, res, next) {
    try {
        const student = await studentService.getStudentById(req.params.id);

        if (!student) {
            throw new AppError('Student not found', 404);
        }

        res.json({ success: true, data: student });
    } catch (error) {
        next(error); // Pass to error handler
    }
}

// Apply error handler (must be last middleware)
app.use(errorHandler);
```

---

## API Documentation

### Swagger/OpenAPI (Express)

```javascript
const swaggerJsDoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const swaggerOptions = {
    definition: {
        openapi: '3.0.0',
        info: {
            title: 'College Management API',
            version: '1.0.0',
            description: 'API for managing students, courses, and enrollments'
        },
        servers: [
            { url: 'http://localhost:3000/api/v1' }
        ]
    },
    apis: ['./routes/*.js', './controllers/*.js']
};

const swaggerDocs = swaggerJsDoc(swaggerOptions);
app.use('/api/docs', swaggerUi.serve, swaggerUi.setup(swaggerDocs));
```

**Documenting endpoints**:
```javascript
/**
 * @swagger
 * /students:
 *   get:
 *     summary: Get all students
 *     tags: [Students]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *         description: Page number
 *       - in: query
 *         name: department
 *         schema:
 *           type: string
 *         description: Filter by department
 *     responses:
 *       200:
 *         description: Success
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/Student'
 */
router.get('/students', studentController.getAllStudents);
```

---

## Key Takeaways

1. **API Layer handles HTTP** - Not business logic
2. **RESTful principles** - Resource-based URLs, proper HTTP methods
3. **Middleware** - Process requests before controllers
4. **Error handling** - Centralized, consistent error responses
5. **Validation** - Validate input at API layer
6. **Documentation** - Swagger/OpenAPI for Express.js
7. **Security** - Authentication, authorization, rate limiting

---

**Next**: [04 - Business Logic Layer →](04_Business_Logic_Layer.md)

---

**Previous**: [← 02 - Layered Architecture](02_Layered_Architecture.md) | **Next**: [04 - Business Logic Layer →](04_Business_Logic_Layer.md)

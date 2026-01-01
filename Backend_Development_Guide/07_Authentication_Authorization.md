# Part 7: Authentication and Authorization

**Author**: Saiprasad Hegde

---

## Table of Contents
1. [Authentication vs Authorization](#authentication-vs-authorization)
2. [Password Security](#password-security)
3. [JWT (JSON Web Tokens)](#jwt-json-web-tokens)
4. [Session-Based Authentication](#session-based-authentication)
5. [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
6. [OAuth 2.0](#oauth-20)
7. [Implementation Examples](#implementation-examples)

---

## Authentication vs Authorization

### Analogy: Airport Security

```
┌────────────────────────────────────────┐
│  AUTHENTICATION                        │
│  "Who are you?"                        │
│  Show ID card/passport                 │
│  Verify identity                       │
└────────────────────────────────────────┘
                 ↓
┌────────────────────────────────────────┐
│  AUTHORIZATION                         │
│  "What can you access?"                │
│  Check boarding pass                   │
│  Business class vs Economy             │
│  Lounge access?                        │
└────────────────────────────────────────┘
```

### College System Example

**Authentication**:
```javascript
POST /api/auth/login
{
    "email": "john@college.edu",
    "password": "secret123"
}
→ "Yes, you are John Smith (Student ID: S12345)"
```

**Authorization**:
```javascript
// Student John trying to access different resources
GET /api/students/me              → ✅ Allowed (own data)
GET /api/students/S99999          → ❌ Forbidden (other student)
GET /api/admin/reports            → ❌ Forbidden (not admin)
POST /api/courses/CS101/enroll    → ✅ Allowed (students can enroll)
POST /api/grades                  → ❌ Forbidden (only professors)
```

---

## Password Security

### Never Store Plain Passwords!

```javascript
// ❌ NEVER DO THIS!
{
    email: "john@college.edu",
    password: "mypassword123"  // Plain text - DANGEROUS!
}

// ✅ Always hash passwords
{
    email: "john@college.edu",
    passwordHash: "$2b$10$N9qo8uL..." // Hashed with bcrypt
}
```

### Hashing with bcrypt

**Node.js**:
```javascript
const bcrypt = require('bcrypt');

// Hash password during registration
async function hashPassword(plainPassword) {
    const saltRounds = 10;
    return await bcrypt.hash(plainPassword, saltRounds);
}

// Verify password during login
async function verifyPassword(plainPassword, hashedPassword) {
    return await bcrypt.compare(plainPassword, hashedPassword);
}

// Usage
const hashedPassword = await hashPassword("mypassword123");
// $2b$10$N9qo8uL...

const isValid = await verifyPassword("mypassword123", hashedPassword);
// true
```

### Complete Registration Flow

**services/authService.js**:
```javascript
const bcrypt = require('bcrypt');
const { v4: uuidv4 } = require('uuid');

class AuthService {
    constructor(studentRepository) {
        this.studentRepo = studentRepository;
    }

    async register(userData) {
        // 1. Validate input
        if (!userData.email || !userData.password) {
            throw new Error('Email and password required');
        }

        // 2. Check if email already exists
        const existingUser = await this.studentRepo.findByEmail(userData.email);
        if (existingUser) {
            throw new Error('Email already registered');
        }

        // 3. Validate password strength
        if (userData.password.length < 8) {
            throw new Error('Password must be at least 8 characters');
        }

        // 4. Hash password
        const passwordHash = await bcrypt.hash(userData.password, 10);

        // 5. Create student
        const student = await this.studentRepo.create({
            id: uuidv4(),
            firstName: userData.firstName,
            lastName: userData.lastName,
            email: userData.email,
            passwordHash,  // Store hash, not plain password
            department: userData.department,
            enrollmentYear: new Date().getFullYear(),
            status: 'active'
        });

        // 6. Don't return password hash!
        const { passwordHash: _, ...studentData } = student.toJSON();
        return studentData;
    }

    async login(email, password) {
        // 1. Find user by email
        const student = await this.studentRepo.findByEmail(email);
        if (!student) {
            throw new Error('Invalid credentials');
        }

        // 2. Verify password
        const isValidPassword = await bcrypt.compare(
            password,
            student.passwordHash
        );

        if (!isValidPassword) {
            throw new Error('Invalid credentials');
        }

        // 3. Check account status
        if (student.status !== 'active') {
            throw new Error('Account is not active');
        }

        // 4. Return user data (without password hash)
        const { passwordHash: _, ...studentData } = student.toJSON();
        return studentData;
    }
}

module.exports = AuthService;
```

---

## JWT (JSON Web Tokens)

### What is JWT?

**JWT** is a token that contains encoded user information, signed by the server.

**Structure**:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjEyMzQ1Iiwicm9sZSI6InN0dWRlbnQifQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

    Header          Payload           Signature
      ↓                ↓                  ↓
{                {                  Signed with
  "alg": "HS256",  "id": "12345",      secret key
  "typ": "JWT"     "role": "student"
}                }
```

### Analogy: Concert Wristband

```
1. You buy ticket → Register/Login
2. Get wristband → Receive JWT token
3. Show wristband to enter different areas → Send token with each request
4. Wristband proves you paid → Token proves you're authenticated
```

### Creating JWTs (Node.js)

```bash
npm install jsonwebtoken
```

**utils/jwt.js**:
```javascript
const jwt = require('jsonwebtoken');

const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';
const JWT_EXPIRES_IN = '7d'; // Token expires in 7 days

function generateToken(payload) {
    return jwt.sign(payload, JWT_SECRET, {
        expiresIn: JWT_EXPIRES_IN
    });
}

function verifyToken(token) {
    try {
        return jwt.verify(token, JWT_SECRET);
    } catch (error) {
        if (error.name === 'TokenExpiredError') {
            throw new Error('Token expired');
        }
        throw new Error('Invalid token');
    }
}

module.exports = { generateToken, verifyToken };
```

### Login with JWT

**controllers/authController.js**:
```javascript
const { generateToken } = require('../utils/jwt');

class AuthController {
    async login(req, res) {
        try {
            const { email, password } = req.body;

            // Authenticate user
            const student = await authService.login(email, password);

            // Generate JWT token
            const token = generateToken({
                id: student.id,
                email: student.email,
                role: 'student',
                department: student.department
            });

            res.json({
                success: true,
                data: {
                    user: student,
                    token
                },
                message: 'Login successful'
            });
        } catch (error) {
            res.status(401).json({
                success: false,
                error: error.message
            });
        }
    }

    async register(req, res) {
        try {
            const student = await authService.register(req.body);

            // Generate token for new user
            const token = generateToken({
                id: student.id,
                email: student.email,
                role: 'student',
                department: student.department
            });

            res.status(201).json({
                success: true,
                data: {
                    user: student,
                    token
                },
                message: 'Registration successful'
            });
        } catch (error) {
            res.status(400).json({
                success: false,
                error: error.message
            });
        }
    }
}

module.exports = new AuthController();
```

### Authentication Middleware

**middlewares/auth.js**:
```javascript
const { verifyToken } = require('../utils/jwt');

async function authenticate(req, res, next) {
    try {
        // 1. Get token from header
        const authHeader = req.headers.authorization;

        if (!authHeader || !authHeader.startsWith('Bearer ')) {
            return res.status(401).json({
                success: false,
                error: 'No token provided'
            });
        }

        const token = authHeader.split(' ')[1];

        // 2. Verify token
        const decoded = verifyToken(token);

        // 3. Attach user to request
        req.user = decoded;

        next();
    } catch (error) {
        res.status(401).json({
            success: false,
            error: error.message
        });
    }
}

module.exports = { authenticate };
```

### Using Authentication

```javascript
const { authenticate } = require('./middlewares/auth');

// Public route - no auth needed
router.get('/api/courses', courseController.getAllCourses);

// Protected route - auth required
router.get('/api/students/me', authenticate, studentController.getCurrentStudent);

// Example request:
// GET /api/students/me
// Headers:
//   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Session-Based Authentication

### How Sessions Work

```
1. User logs in → Server creates session → Stores in database/Redis
2. Server sends session ID as cookie → Browser stores cookie
3. Browser sends cookie with each request → Server looks up session
4. Server knows who you are → Process request
```

### Express Session Example

```bash
npm install express-session connect-redis redis
```

**server.js**:
```javascript
const session = require('express-session');
const RedisStore = require('connect-redis').default;
const { createClient } = require('redis');

// Redis client
const redisClient = createClient({
    host: 'localhost',
    port: 6379
});

redisClient.connect();

// Session middleware
app.use(session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: process.env.NODE_ENV === 'production', // HTTPS only in prod
        httpOnly: true,  // Prevent XSS attacks
        maxAge: 1000 * 60 * 60 * 24 * 7  // 7 days
    }
}));
```

**Login with Session**:
```javascript
router.post('/login', async (req, res) => {
    const { email, password } = req.body;

    const student = await authService.login(email, password);

    // Store user in session
    req.session.user = {
        id: student.id,
        email: student.email,
        role: 'student'
    };

    res.json({
        success: true,
        data: student
    });
});

// Logout
router.post('/logout', (req, res) => {
    req.session.destroy((err) => {
        if (err) {
            return res.status(500).json({ error: 'Logout failed' });
        }
        res.clearCookie('connect.sid');
        res.json({ success: true, message: 'Logged out' });
    });
});

// Session auth middleware
function requireSession(req, res, next) {
    if (!req.session.user) {
        return res.status(401).json({ error: 'Not authenticated' });
    }
    next();
}
```

### JWT vs Sessions

| Feature | JWT | Sessions |
|---------|-----|----------|
| **Storage** | Client (localStorage/cookie) | Server (database/Redis) |
| **Scalability** | Easy (stateless) | Harder (need shared storage) |
| **Revocation** | Hard (can't invalidate) | Easy (delete session) |
| **Size** | Larger (sent with every request) | Smaller (just session ID) |
| **Use Case** | APIs, mobile apps, microservices | Traditional web apps |

---

## Role-Based Access Control (RBAC)

### Define Roles

```javascript
const ROLES = {
    STUDENT: 'student',
    PROFESSOR: 'professor',
    ADMIN: 'admin'
};

const PERMISSIONS = {
    // Students
    VIEW_OWN_GRADES: 'view_own_grades',
    ENROLL_IN_COURSES: 'enroll_in_courses',

    // Professors
    VIEW_COURSE_STUDENTS: 'view_course_students',
    SUBMIT_GRADES: 'submit_grades',

    // Admins
    MANAGE_STUDENTS: 'manage_students',
    MANAGE_COURSES: 'manage_courses',
    VIEW_ALL_DATA: 'view_all_data'
};

const ROLE_PERMISSIONS = {
    [ROLES.STUDENT]: [
        PERMISSIONS.VIEW_OWN_GRADES,
        PERMISSIONS.ENROLL_IN_COURSES
    ],
    [ROLES.PROFESSOR]: [
        PERMISSIONS.VIEW_COURSE_STUDENTS,
        PERMISSIONS.SUBMIT_GRADES
    ],
    [ROLES.ADMIN]: Object.values(PERMISSIONS)  // All permissions
};
```

### Authorization Middleware

**middlewares/authorize.js**:
```javascript
function authorize(allowedRoles = []) {
    return (req, res, next) => {
        // Check if user is authenticated
        if (!req.user) {
            return res.status(401).json({
                success: false,
                error: 'Authentication required'
            });
        }

        // Check if user has required role
        if (allowedRoles.length && !allowedRoles.includes(req.user.role)) {
            return res.status(403).json({
                success: false,
                error: 'Insufficient permissions'
            });
        }

        next();
    };
}

// Permission-based authorization
function requirePermission(permission) {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Not authenticated' });
        }

        const userPermissions = ROLE_PERMISSIONS[req.user.role] || [];

        if (!userPermissions.includes(permission)) {
            return res.status(403).json({ error: 'Permission denied' });
        }

        next();
    };
}

module.exports = { authorize, requirePermission };
```

### Using Authorization

```javascript
const { authenticate } = require('./middlewares/auth');
const { authorize, requirePermission } = require('./middlewares/authorize');

// Only students can enroll
router.post('/enrollments',
    authenticate,
    authorize(['student']),
    enrollmentController.enroll
);

// Only professors can submit grades
router.post('/grades',
    authenticate,
    authorize(['professor', 'admin']),
    gradeController.submitGrade
);

// Only admins can create courses
router.post('/courses',
    authenticate,
    authorize(['admin']),
    courseController.createCourse
);

// Permission-based
router.get('/admin/reports',
    authenticate,
    requirePermission(PERMISSIONS.VIEW_ALL_DATA),
    reportController.getReports
);
```

### Resource-Level Authorization

```javascript
// Students can only view their own grades
async getGrades(req, res) {
    const { studentId } = req.params;

    // Authorization check
    if (req.user.role === 'student' && req.user.id !== studentId) {
        return res.status(403).json({
            error: 'You can only view your own grades'
        });
    }

    const grades = await gradeService.getStudentGrades(studentId);
    res.json({ data: grades });
}

// Professors can only submit grades for their courses
async submitGrade(req, res) {
    const { courseId, studentId, grade } = req.body;

    // Check if professor teaches this course
    const course = await courseRepo.findById(courseId);

    if (course.professorId !== req.user.id) {
        return res.status(403).json({
            error: 'You can only grade students in your courses'
        });
    }

    await gradeService.submitGrade(studentId, courseId, grade);
    res.json({ success: true });
}
```

---

## OAuth 2.0

### What is OAuth?

**OAuth 2.0** allows users to login with third-party providers (Google, GitHub, etc.) without sharing passwords.

**Flow**:
```
1. User clicks "Login with Google"
2. Redirect to Google login
3. User logs in and approves
4. Google redirects back with authorization code
5. Exchange code for access token
6. Use token to get user info
7. Create/update user in your database
```

### OAuth with Passport.js

```bash
npm install passport passport-google-oauth20
```

**config/passport.js**:
```javascript
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: '/api/auth/google/callback'
}, async (accessToken, refreshToken, profile, done) => {
    try {
        // Check if user exists
        let student = await studentRepo.findByEmail(profile.emails[0].value);

        if (!student) {
            // Create new student from Google profile
            student = await studentRepo.create({
                email: profile.emails[0].value,
                firstName: profile.name.givenName,
                lastName: profile.name.familyName,
                googleId: profile.id,
                // ... other fields
            });
        }

        done(null, student);
    } catch (error) {
        done(error, null);
    }
}));

module.exports = passport;
```

**Routes**:
```javascript
const passport = require('./config/passport');

// Initiate OAuth flow
router.get('/auth/google',
    passport.authenticate('google', {
        scope: ['profile', 'email']
    })
);

// OAuth callback
router.get('/auth/google/callback',
    passport.authenticate('google', { session: false }),
    (req, res) => {
        // Generate JWT for the user
        const token = generateToken({
            id: req.user.id,
            email: req.user.email,
            role: 'student'
        });

        // Redirect to frontend with token
        res.redirect(`${process.env.FRONTEND_URL}?token=${token}`);
    }
);
```

---

## Implementation Examples

### Complete Auth System (Express)

**routes/authRoutes.js**:
```javascript
const router = require('express').Router();
const authController = require('../controllers/authController');
const { authenticate } = require('../middlewares/auth');

router.post('/register', authController.register);
router.post('/login', authController.login);
router.post('/logout', authenticate, authController.logout);
router.post('/refresh', authController.refreshToken);
router.get('/me', authenticate, authController.getCurrentUser);
router.post('/change-password', authenticate, authController.changePassword);
router.post('/forgot-password', authController.forgotPassword);
router.post('/reset-password', authController.resetPassword);

module.exports = router;
```

**controllers/authController.js**:
```javascript
class AuthController {
    async register(req, res, next) {
        try {
            const result = await authService.register(req.body);
            res.status(201).json({
                success: true,
                data: result
            });
        } catch (error) {
            next(error);
        }
    }

    async login(req, res, next) {
        try {
            const { email, password } = req.body;
            const result = await authService.login(email, password);

            res.json({
                success: true,
                data: result
            });
        } catch (error) {
            next(error);
        }
    }

    async getCurrentUser(req, res, next) {
        try {
            const student = await studentService.getById(req.user.id);
            res.json({
                success: true,
                data: student
            });
        } catch (error) {
            next(error);
        }
    }

    async changePassword(req, res, next) {
        try {
            const { currentPassword, newPassword } = req.body;

            await authService.changePassword(
                req.user.id,
                currentPassword,
                newPassword
            );

            res.json({
                success: true,
                message: 'Password changed successfully'
            });
        } catch (error) {
            next(error);
        }
    }
}

module.exports = new AuthController();
```

---

## Security Best Practices

### 1. Always Use HTTPS in Production

```javascript
// Force HTTPS
app.use((req, res, next) => {
    if (process.env.NODE_ENV === 'production' && !req.secure) {
        return res.redirect('https://' + req.headers.host + req.url);
    }
    next();
});
```

### 2. Set Secure Cookie Flags

```javascript
res.cookie('token', token, {
    httpOnly: true,    // Prevent XSS
    secure: true,      // HTTPS only
    sameSite: 'strict' // Prevent CSRF
});
```

### 3. Rate Limiting for Auth Endpoints

```javascript
const rateLimit = require('express-rate-limit');

const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // 5 requests per window
    message: 'Too many login attempts, please try again later'
});

router.post('/login', authLimiter, authController.login);
```

### 4. Password Requirements

```javascript
function validatePassword(password) {
    const minLength = 8;
    const hasUpperCase = /[A-Z]/.test(password);
    const hasLowerCase = /[a-z]/.test(password);
    const hasNumber = /\d/.test(password);
    const hasSpecialChar = /[!@#$%^&*]/.test(password);

    if (password.length < minLength) {
        throw new Error('Password must be at least 8 characters');
    }

    if (!hasUpperCase || !hasLowerCase) {
        throw new Error('Password must contain uppercase and lowercase letters');
    }

    if (!hasNumber) {
        throw new Error('Password must contain at least one number');
    }

    if (!hasSpecialChar) {
        throw new Error('Password must contain at least one special character');
    }
}
```

### 5. Token Expiration and Refresh

```javascript
// Short-lived access token (15 minutes)
const accessToken = generateToken(payload, '15m');

// Long-lived refresh token (7 days)
const refreshToken = generateToken({ id: user.id }, '7d');

// Store refresh token in database
await refreshTokenRepo.create({
    userId: user.id,
    token: refreshToken,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
});

res.json({ accessToken, refreshToken });
```

---

## Key Takeaways

1. **Authentication**: Verify user identity (who are you?)
2. **Authorization**: Check permissions (what can you do?)
3. **Never store plain passwords** - always hash with bcrypt
4. **JWT**: Stateless, scalable, good for APIs and mobile apps
5. **Sessions**: Stateful, easy to revoke, good for traditional web apps
6. **RBAC**: Role-based permissions (student, professor, admin)
7. **OAuth 2.0**: Login with third-party providers (Google, GitHub, etc.)
8. **Security**: HTTPS, secure cookies, rate limiting, strong passwords

---

**Next**: [08 - Validation and Error Handling →](08_Validation_Error_Handling.md)

---

**Previous**: [← 06 - Database Layer](06_Database_Layer.md) | **Next**: [08 - Validation and Error Handling →](08_Validation_Error_Handling.md)

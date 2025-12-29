# Part 10: Testing and Deployment

**Author**: Saiprasad Hegde

## Table of Contents
- [Introduction](#introduction)
- [Testing Fundamentals](#testing-fundamentals)
- [Unit Testing](#unit-testing)
- [Integration Testing](#integration-testing)
- [End-to-End Testing](#end-to-end-testing)
- [Test-Driven Development](#test-driven-development)
- [Docker and Containerization](#docker-and-containerization)
- [CI/CD Pipelines](#cicd-pipelines)
- [Deployment Strategies](#deployment-strategies)
- [Monitoring and Logging](#monitoring-and-logging)
- [Best Practices](#best-practices)

---

## Introduction

### Why Testing Matters

**Analogy**: Testing is like quality control in a car factory. You wouldn't ship a car without testing the brakes, engine, and safety features.

**Benefits of Testing**:
- **Confidence**: Know your code works as expected
- **Refactoring Safety**: Change code without breaking functionality
- **Documentation**: Tests show how code should be used
- **Bug Prevention**: Catch issues before production
- **Faster Development**: Find bugs early (cheap to fix) vs in production (expensive)

### Testing Pyramid

```
        /\
       /  \     E2E Tests (Few, Slow, Expensive)
      /____\
     /      \   Integration Tests (Some, Medium)
    /________\
   /          \ Unit Tests (Many, Fast, Cheap)
  /__________\
```

- **70% Unit Tests**: Test individual functions/methods
- **20% Integration Tests**: Test modules working together
- **10% E2E Tests**: Test entire application flow

---

## Testing Fundamentals

### Test Structure: AAA Pattern

```javascript
test('should enroll student in course', async () => {
    // 1. ARRANGE - Set up test data
    const student = await createTestStudent();
    const course = await createTestCourse();

    // 2. ACT - Perform the action
    const enrollment = await enrollmentService.enrollStudent(student.id, course.id);

    // 3. ASSERT - Verify the result
    expect(enrollment).toBeDefined();
    expect(enrollment.studentId).toBe(student.id);
    expect(enrollment.courseId).toBe(course.id);
});
```

### Testing Frameworks

**Node.js/JavaScript**:
- **Jest**: Most popular, built-in mocking, coverage
- **Mocha + Chai**: Flexible, require separate assertion library
- **Vitest**: Fast, Vite-powered

**Python**:
- **pytest**: Most popular, powerful fixtures
- **unittest**: Built-in, more verbose
- **FastAPI TestClient**: Built on top of Starlette

---

## Unit Testing

Unit tests verify individual functions/methods in isolation.

### Setup Jest (Node.js)

```bash
npm install --save-dev jest @types/jest supertest
```

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "testEnvironment": "node",
    "coverageDirectory": "coverage",
    "collectCoverageFrom": [
      "src/**/*.js",
      "!src/server.js",
      "!src/config/**"
    ],
    "testMatch": [
      "**/__tests__/**/*.test.js"
    ]
  }
}
```

### Unit Test Examples

#### Testing Service Layer

```javascript
// __tests__/services/studentService.test.js
const studentService = require('../../src/services/studentService');
const studentRepository = require('../../src/repositories/studentRepository');
const cacheService = require('../../src/services/cacheService');

// Mock dependencies
jest.mock('../../src/repositories/studentRepository');
jest.mock('../../src/services/cacheService');

describe('StudentService', () => {
    beforeEach(() => {
        // Clear all mocks before each test
        jest.clearAllMocks();
    });

    describe('getStudentById', () => {
        it('should return student from cache if available', async () => {
            // Arrange
            const mockStudent = {
                id: '123',
                firstName: 'John',
                lastName: 'Doe',
                email: 'john@example.com'
            };
            cacheService.get.mockResolvedValue(mockStudent);

            // Act
            const result = await studentService.getStudentById('123');

            // Assert
            expect(result).toEqual(mockStudent);
            expect(cacheService.get).toHaveBeenCalledWith('student:123');
            expect(studentRepository.findById).not.toHaveBeenCalled(); // DB not called
        });

        it('should fetch from database on cache miss', async () => {
            // Arrange
            const mockStudent = {
                id: '123',
                firstName: 'John',
                lastName: 'Doe',
                email: 'john@example.com'
            };
            cacheService.get.mockResolvedValue(null); // Cache miss
            studentRepository.findById.mockResolvedValue(mockStudent);

            // Act
            const result = await studentService.getStudentById('123');

            // Assert
            expect(result).toEqual(mockStudent);
            expect(cacheService.get).toHaveBeenCalledWith('student:123');
            expect(studentRepository.findById).toHaveBeenCalledWith('123');
            expect(cacheService.set).toHaveBeenCalledWith('student:123', mockStudent, 600);
        });

        it('should throw NotFoundError when student does not exist', async () => {
            // Arrange
            cacheService.get.mockResolvedValue(null);
            studentRepository.findById.mockResolvedValue(null);

            // Act & Assert
            await expect(studentService.getStudentById('999'))
                .rejects
                .toThrow('Student with ID 999 not found');
        });
    });

    describe('updateStudent', () => {
        it('should update student and invalidate caches', async () => {
            // Arrange
            const mockStudent = { id: '123', firstName: 'Jane' };
            const updateData = { firstName: 'Jane' };
            studentRepository.update.mockResolvedValue(mockStudent);

            // Act
            const result = await studentService.updateStudent('123', updateData);

            // Assert
            expect(result).toEqual(mockStudent);
            expect(studentRepository.update).toHaveBeenCalledWith('123', updateData);
            expect(cacheService.delete).toHaveBeenCalledWith('student:123');
            expect(cacheService.deletePattern).toHaveBeenCalledWith('students:list:*');
        });
    });
});
```

#### Testing Business Logic

```javascript
// __tests__/services/enrollmentService.test.js
const enrollmentService = require('../../src/services/enrollmentService');
const studentRepository = require('../../src/repositories/studentRepository');
const courseRepository = require('../../src/repositories/courseRepository');
const enrollmentRepository = require('../../src/repositories/enrollmentRepository');

jest.mock('../../src/repositories/studentRepository');
jest.mock('../../src/repositories/courseRepository');
jest.mock('../../src/repositories/enrollmentRepository');

describe('EnrollmentService', () => {
    describe('enrollStudent', () => {
        it('should successfully enroll student', async () => {
            // Arrange
            const student = {
                id: 'student-1',
                totalCredits: 10,
                department: 'CS'
            };
            const course = {
                id: 'course-1',
                code: 'CS101',
                credits: 3,
                maxEnrollments: 30,
                prerequisites: []
            };
            const enrollmentCount = 20;

            studentRepository.findById.mockResolvedValue(student);
            courseRepository.findById.mockResolvedValue(course);
            enrollmentRepository.countByCourseId.mockResolvedValue(enrollmentCount);
            enrollmentRepository.findByStudentAndCourse.mockResolvedValue(null);
            enrollmentRepository.findByStudentId.mockResolvedValue([]);
            enrollmentRepository.create.mockResolvedValue({
                id: 'enrollment-1',
                studentId: student.id,
                courseId: course.id,
                status: 'active'
            });

            // Act
            const result = await enrollmentService.enrollStudent('student-1', 'course-1');

            // Assert
            expect(result).toBeDefined();
            expect(result.status).toBe('active');
            expect(enrollmentRepository.create).toHaveBeenCalledWith({
                studentId: 'student-1',
                courseId: 'course-1',
                status: 'active',
                enrolledAt: expect.any(Date)
            });
        });

        it('should throw error if course is full', async () => {
            // Arrange
            const student = { id: 'student-1', totalCredits: 10 };
            const course = {
                id: 'course-1',
                maxEnrollments: 30,
                prerequisites: []
            };
            const enrollmentCount = 30; // Course is full

            studentRepository.findById.mockResolvedValue(student);
            courseRepository.findById.mockResolvedValue(course);
            enrollmentRepository.countByCourseId.mockResolvedValue(enrollmentCount);

            // Act & Assert
            await expect(enrollmentService.enrollStudent('student-1', 'course-1'))
                .rejects
                .toThrow('Course is full');
        });

        it('should throw error if already enrolled', async () => {
            // Arrange
            const student = { id: 'student-1', totalCredits: 10 };
            const course = {
                id: 'course-1',
                maxEnrollments: 30,
                prerequisites: []
            };
            const existingEnrollment = { id: 'enrollment-1' };

            studentRepository.findById.mockResolvedValue(student);
            courseRepository.findById.mockResolvedValue(course);
            enrollmentRepository.countByCourseId.mockResolvedValue(20);
            enrollmentRepository.findByStudentAndCourse.mockResolvedValue(existingEnrollment);

            // Act & Assert
            await expect(enrollmentService.enrollStudent('student-1', 'course-1'))
                .rejects
                .toThrow('Student is already enrolled in this course');
        });

        it('should throw error if prerequisites not met', async () => {
            // Arrange
            const student = { id: 'student-1', totalCredits: 10 };
            const course = {
                id: 'course-1',
                maxEnrollments: 30,
                prerequisites: ['course-0'] // Requires CS100
            };

            studentRepository.findById.mockResolvedValue(student);
            courseRepository.findById.mockResolvedValue(course);
            enrollmentRepository.countByCourseId.mockResolvedValue(20);
            enrollmentRepository.findByStudentAndCourse.mockResolvedValue(null);
            enrollmentRepository.findByStudentId.mockResolvedValue([]); // No completed courses

            // Act & Assert
            await expect(enrollmentService.enrollStudent('student-1', 'course-1'))
                .rejects
                .toThrow('Prerequisites not met');
        });
    });
});
```

### Testing Utilities and Helpers

```javascript
// __tests__/utils/validation.test.js
const { validateEmail, validatePhoneNumber } = require('../../src/utils/validation');

describe('Validation Utils', () => {
    describe('validateEmail', () => {
        it('should accept valid emails', () => {
            expect(validateEmail('user@example.com')).toBe(true);
            expect(validateEmail('user.name@example.co.uk')).toBe(true);
            expect(validateEmail('user+tag@example.com')).toBe(true);
        });

        it('should reject invalid emails', () => {
            expect(validateEmail('invalid')).toBe(false);
            expect(validateEmail('invalid@')).toBe(false);
            expect(validateEmail('@example.com')).toBe(false);
            expect(validateEmail('user@.com')).toBe(false);
        });

        it('should reject empty or null values', () => {
            expect(validateEmail('')).toBe(false);
            expect(validateEmail(null)).toBe(false);
            expect(validateEmail(undefined)).toBe(false);
        });
    });
});
```

### Python Unit Tests with pytest

```python
# tests/test_student_service.py
import pytest
from unittest.mock import Mock, patch, AsyncMock
from services.student_service import StudentService
from models.student import Student

@pytest.fixture
def student_service():
    return StudentService()

@pytest.fixture
def mock_student():
    return {
        'id': '123',
        'first_name': 'John',
        'last_name': 'Doe',
        'email': 'john@example.com'
    }

class TestStudentService:
    @patch('services.student_service.student_repository')
    @patch('services.student_service.cache_service')
    async def test_get_student_by_id_cache_hit(
        self, mock_cache, mock_repo, student_service, mock_student
    ):
        """Should return student from cache if available"""
        # Arrange
        mock_cache.get = AsyncMock(return_value=mock_student)

        # Act
        result = await student_service.get_student_by_id('123')

        # Assert
        assert result == mock_student
        mock_cache.get.assert_called_once_with('student:123')
        mock_repo.find_by_id.assert_not_called()

    @patch('services.student_service.student_repository')
    @patch('services.student_service.cache_service')
    async def test_get_student_by_id_cache_miss(
        self, mock_cache, mock_repo, student_service, mock_student
    ):
        """Should fetch from database on cache miss"""
        # Arrange
        mock_cache.get = AsyncMock(return_value=None)
        mock_repo.find_by_id = AsyncMock(return_value=mock_student)

        # Act
        result = await student_service.get_student_by_id('123')

        # Assert
        assert result == mock_student
        mock_repo.find_by_id.assert_called_once_with('123')
        mock_cache.set.assert_called_once()

    @patch('services.student_service.student_repository')
    @patch('services.student_service.cache_service')
    async def test_get_student_not_found(
        self, mock_cache, mock_repo, student_service
    ):
        """Should raise NotFoundError when student doesn't exist"""
        # Arrange
        mock_cache.get = AsyncMock(return_value=None)
        mock_repo.find_by_id = AsyncMock(return_value=None)

        # Act & Assert
        with pytest.raises(NotFoundError, match="Student with ID 999 not found"):
            await student_service.get_student_by_id('999')
```

---

## Integration Testing

Integration tests verify that multiple modules work together correctly.

### Testing API Endpoints with Supertest

```javascript
// __tests__/integration/students.test.js
const request = require('supertest');
const app = require('../../src/app');
const { sequelize } = require('../../src/config/database');
const Student = require('../../src/models/student');

describe('Student API Integration Tests', () => {
    // Setup: Run before all tests
    beforeAll(async () => {
        await sequelize.sync({ force: true }); // Reset database
    });

    // Cleanup: Run after all tests
    afterAll(async () => {
        await sequelize.close();
    });

    // Clean database between tests
    afterEach(async () => {
        await Student.destroy({ where: {}, force: true });
    });

    describe('POST /api/students', () => {
        it('should create a new student', async () => {
            // Arrange
            const studentData = {
                firstName: 'John',
                lastName: 'Doe',
                email: 'john@example.com',
                department: 'CS',
                dateOfBirth: '2000-01-01'
            };

            // Act
            const response = await request(app)
                .post('/api/students')
                .send(studentData)
                .expect('Content-Type', /json/)
                .expect(201);

            // Assert
            expect(response.body.success).toBe(true);
            expect(response.body.data).toMatchObject({
                firstName: 'John',
                lastName: 'Doe',
                email: 'john@example.com'
            });
            expect(response.body.data.id).toBeDefined();

            // Verify in database
            const studentInDb = await Student.findByPk(response.body.data.id);
            expect(studentInDb).toBeDefined();
            expect(studentInDb.email).toBe('john@example.com');
        });

        it('should return 400 for invalid data', async () => {
            // Arrange
            const invalidData = {
                firstName: 'John'
                // Missing required fields
            };

            // Act
            const response = await request(app)
                .post('/api/students')
                .send(invalidData)
                .expect('Content-Type', /json/)
                .expect(400);

            // Assert
            expect(response.body.success).toBe(false);
            expect(response.body.error).toBeDefined();
        });

        it('should return 409 for duplicate email', async () => {
            // Arrange - Create existing student
            await Student.create({
                firstName: 'Jane',
                lastName: 'Doe',
                email: 'jane@example.com',
                department: 'CS',
                dateOfBirth: '2000-01-01'
            });

            const duplicateData = {
                firstName: 'John',
                lastName: 'Smith',
                email: 'jane@example.com', // Duplicate
                department: 'Math',
                dateOfBirth: '2001-01-01'
            };

            // Act
            const response = await request(app)
                .post('/api/students')
                .send(duplicateData)
                .expect(409);

            // Assert
            expect(response.body.error).toMatch(/email.*already exists/i);
        });
    });

    describe('GET /api/students/:id', () => {
        it('should return student by ID', async () => {
            // Arrange - Create student
            const student = await Student.create({
                firstName: 'John',
                lastName: 'Doe',
                email: 'john@example.com',
                department: 'CS',
                dateOfBirth: '2000-01-01'
            });

            // Act
            const response = await request(app)
                .get(`/api/students/${student.id}`)
                .expect(200);

            // Assert
            expect(response.body.data).toMatchObject({
                id: student.id,
                firstName: 'John',
                lastName: 'Doe',
                email: 'john@example.com'
            });
        });

        it('should return 404 for non-existent student', async () => {
            const fakeId = '00000000-0000-0000-0000-000000000000';

            const response = await request(app)
                .get(`/api/students/${fakeId}`)
                .expect(404);

            expect(response.body.error).toMatch(/not found/i);
        });
    });

    describe('GET /api/students', () => {
        beforeEach(async () => {
            // Create test data
            await Student.bulkCreate([
                { firstName: 'John', lastName: 'Doe', email: 'john@example.com', department: 'CS', dateOfBirth: '2000-01-01' },
                { firstName: 'Jane', lastName: 'Smith', email: 'jane@example.com', department: 'Math', dateOfBirth: '2000-02-01' },
                { firstName: 'Bob', lastName: 'Johnson', email: 'bob@example.com', department: 'CS', dateOfBirth: '2000-03-01' }
            ]);
        });

        it('should return paginated students', async () => {
            const response = await request(app)
                .get('/api/students?page=1&limit=2')
                .expect(200);

            expect(response.body.data).toHaveLength(2);
            expect(response.body.pagination).toMatchObject({
                page: 1,
                limit: 2,
                totalItems: 3,
                totalPages: 2
            });
        });

        it('should filter students by department', async () => {
            const response = await request(app)
                .get('/api/students?department=CS')
                .expect(200);

            expect(response.body.data).toHaveLength(2);
            expect(response.body.data.every(s => s.department === 'CS')).toBe(true);
        });
    });

    describe('Authentication', () => {
        it('should require authentication for protected routes', async () => {
            await request(app)
                .delete('/api/students/123')
                .expect(401);
        });

        it('should allow access with valid token', async () => {
            // Arrange - Create student and get auth token
            const student = await Student.create({
                firstName: 'John',
                lastName: 'Doe',
                email: 'john@example.com',
                department: 'CS',
                dateOfBirth: '2000-01-01'
            });

            const loginResponse = await request(app)
                .post('/api/auth/login')
                .send({ email: 'john@example.com', password: 'password123' });

            const token = loginResponse.body.token;

            // Act
            const response = await request(app)
                .get('/api/students/me')
                .set('Authorization', `Bearer ${token}`)
                .expect(200);

            // Assert
            expect(response.body.data.email).toBe('john@example.com');
        });
    });
});
```

### FastAPI Integration Tests

```python
# tests/test_students_api.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from main import app
from database import Base, get_db

# Test database
TEST_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(TEST_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Override dependency
def override_get_db():
    try:
        db = TestingSessionLocal()
        yield db
    finally:
        db.close()

app.dependency_overrides[get_db] = override_get_db

@pytest.fixture
def client():
    # Create tables
    Base.metadata.create_all(bind=engine)

    with TestClient(app) as test_client:
        yield test_client

    # Drop tables
    Base.metadata.drop_all(bind=engine)

def test_create_student(client):
    """Should create a new student"""
    response = client.post("/api/students", json={
        "first_name": "John",
        "last_name": "Doe",
        "email": "john@example.com",
        "department": "CS",
        "date_of_birth": "2000-01-01"
    })

    assert response.status_code == 201
    data = response.json()
    assert data["first_name"] == "John"
    assert data["email"] == "john@example.com"
    assert "id" in data

def test_get_student(client):
    """Should retrieve student by ID"""
    # Create student
    create_response = client.post("/api/students", json={
        "first_name": "Jane",
        "last_name": "Smith",
        "email": "jane@example.com",
        "department": "Math",
        "date_of_birth": "2000-02-01"
    })
    student_id = create_response.json()["id"]

    # Get student
    response = client.get(f"/api/students/{student_id}")

    assert response.status_code == 200
    assert response.json()["email"] == "jane@example.com"

def test_get_nonexistent_student(client):
    """Should return 404 for non-existent student"""
    response = client.get("/api/students/99999")
    assert response.status_code == 404
```

---

## End-to-End Testing

E2E tests simulate real user interactions, testing the entire application stack.

### Playwright for E2E Testing

```bash
npm install --save-dev @playwright/test
npx playwright install
```

```javascript
// e2e/student-enrollment.spec.js
const { test, expect } = require('@playwright/test');

test.describe('Student Enrollment Flow', () => {
    test.beforeEach(async ({ page }) => {
        // Navigate to app
        await page.goto('http://localhost:3000');

        // Login
        await page.click('text=Login');
        await page.fill('input[name="email"]', 'test@example.com');
        await page.fill('input[name="password"]', 'password123');
        await page.click('button[type="submit"]');

        // Wait for dashboard
        await page.waitForURL('**/dashboard');
    });

    test('should enroll in a course', async ({ page }) => {
        // Navigate to courses
        await page.click('text=Courses');
        await page.waitForURL('**/courses');

        // Select a course
        await page.click('text=Introduction to Computer Science');

        // Enroll
        await page.click('button:has-text("Enroll")');

        // Verify success message
        await expect(page.locator('.success-message')).toContainText('Successfully enrolled');

        // Verify course appears in "My Courses"
        await page.click('text=My Courses');
        await expect(page.locator('.course-list')).toContainText('Introduction to Computer Science');
    });

    test('should not allow enrollment when prerequisites not met', async ({ page }) => {
        // Navigate to advanced course
        await page.click('text=Courses');
        await page.click('text=Advanced Algorithms');

        // Try to enroll
        await page.click('button:has-text("Enroll")');

        // Verify error message
        await expect(page.locator('.error-message')).toContainText('Prerequisites not met');
    });

    test('should show enrollment capacity', async ({ page }) => {
        await page.click('text=Courses');
        await page.click('text=Introduction to Computer Science');

        // Check capacity display
        const capacity = page.locator('.course-capacity');
        await expect(capacity).toBeVisible();
        await expect(capacity).toContainText(/\d+\/\d+ enrolled/);
    });
});
```

---

## Test-Driven Development (TDD)

**TDD Cycle**: Red → Green → Refactor

**Analogy**: TDD is like writing a shopping list (test) before going to the store (writing code). You know exactly what you need.

### TDD Example: Password Validation

**Step 1: Write failing test (RED)**

```javascript
// __tests__/utils/passwordValidator.test.js
const { validatePassword } = require('../../src/utils/passwordValidator');

describe('Password Validator', () => {
    test('should accept valid password', () => {
        expect(validatePassword('SecureP@ss123')).toBe(true);
    });

    test('should reject password shorter than 8 characters', () => {
        expect(validatePassword('Short1!')).toBe(false);
    });

    test('should reject password without uppercase', () => {
        expect(validatePassword('lowercase123!')).toBe(false);
    });

    test('should reject password without lowercase', () => {
        expect(validatePassword('UPPERCASE123!')).toBe(false);
    });

    test('should reject password without number', () => {
        expect(validatePassword('NoNumbers!')).toBe(false);
    });

    test('should reject password without special character', () => {
        expect(validatePassword('NoSpecial123')).toBe(false);
    });
});
```

Run test: `npm test` → **Tests FAIL** (function doesn't exist yet)

**Step 2: Write minimal code to pass (GREEN)**

```javascript
// src/utils/passwordValidator.js
function validatePassword(password) {
    if (!password || password.length < 8) {
        return false;
    }

    const hasUppercase = /[A-Z]/.test(password);
    const hasLowercase = /[a-z]/.test(password);
    const hasNumber = /\d/.test(password);
    const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);

    return hasUppercase && hasLowercase && hasNumber && hasSpecialChar;
}

module.exports = { validatePassword };
```

Run test: `npm test` → **Tests PASS**

**Step 3: Refactor**

```javascript
// src/utils/passwordValidator.js
const PASSWORD_RULES = {
    minLength: 8,
    patterns: {
        uppercase: /[A-Z]/,
        lowercase: /[a-z]/,
        number: /\d/,
        specialChar: /[!@#$%^&*(),.?":{}|<>]/
    }
};

function validatePassword(password) {
    if (!password || password.length < PASSWORD_RULES.minLength) {
        return false;
    }

    return Object.values(PASSWORD_RULES.patterns).every(pattern =>
        pattern.test(password)
    );
}

module.exports = { validatePassword, PASSWORD_RULES };
```

Run test again: **Still PASS** (refactoring successful)

---

## Docker and Containerization

Docker packages your application with all dependencies into a container that runs anywhere.

**Analogy**: Docker is like a shipping container. Just as a shipping container can be loaded onto any ship, truck, or train, a Docker container runs on any machine.

### Dockerfile for Node.js

```dockerfile
# Dockerfile
# Use official Node.js image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node healthcheck.js || exit 1

# Start application
CMD ["node", "server.js"]
```

### Multi-stage Dockerfile (Smaller Image)

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Copy built artifacts from builder
COPY --from=builder /app/dist ./dist

EXPOSE 3000
CMD ["node", "dist/server.js"]
```

### Docker Compose (Multi-Container)

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Node.js API
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:password@db:5432/college
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "node", "healthcheck.js"]
      interval: 30s
      timeout: 3s
      retries: 3

  # PostgreSQL Database
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=college
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped

  # Redis Cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

  # Nginx Load Balancer
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

### Running with Docker

```bash
# Build image
docker build -t college-api .

# Run container
docker run -p 3000:3000 --env-file .env college-api

# With Docker Compose (recommended)
docker-compose up -d

# View logs
docker-compose logs -f api

# Stop all services
docker-compose down

# Rebuild and restart
docker-compose up -d --build
```

### Python/FastAPI Dockerfile

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Expose port
EXPOSE 8000

# Run application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## CI/CD Pipelines

Continuous Integration/Continuous Deployment automates testing and deployment.

**Analogy**: CI/CD is like an assembly line in a factory. Every step is automated, from testing to packaging to shipping.

### GitHub Actions Workflow

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  # Job 1: Test
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run unit tests
        run: npm test
        env:
          NODE_ENV: test
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Run integration tests
        run: npm run test:integration
        env:
          NODE_ENV: test
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379

      - name: Generate coverage report
        run: npm run test:coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/coverage-final.json

  # Job 2: Build Docker Image
  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.DOCKER_USERNAME }}/college-api:latest
            ${{ secrets.DOCKER_USERNAME }}/college-api:${{ github.sha }}
          cache-from: type=registry,ref=${{ secrets.DOCKER_USERNAME }}/college-api:buildcache
          cache-to: type=registry,ref=${{ secrets.DOCKER_USERNAME }}/college-api:buildcache,mode=max

  # Job 3: Deploy to Production
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.DEPLOY_HOST }}
          username: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.DEPLOY_KEY }}
          script: |
            cd /opt/college-api
            docker-compose pull
            docker-compose up -d
            docker-compose exec -T api npm run migrate
            docker system prune -f
```

### GitLab CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  POSTGRES_DB: test_db
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: postgres

# Run tests
test:
  stage: test
  image: node:18
  services:
    - postgres:15
    - redis:7-alpine
  variables:
    DATABASE_URL: "postgresql://postgres:postgres@postgres:5432/test_db"
    REDIS_URL: "redis://redis:6379"
  script:
    - npm ci
    - npm run lint
    - npm test
    - npm run test:integration
  coverage: '/Statements\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# Build Docker image
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main

# Deploy to production
deploy:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh -o StrictHostKeyChecking=no $DEPLOY_USER@$DEPLOY_HOST "cd /opt/college-api && docker-compose pull && docker-compose up -d"
  only:
    - main
  when: manual
```

---

## Deployment Strategies

### 1. Blue-Green Deployment

**Analogy**: Like having two identical stages. While audience watches Stage A (Blue), prepare Stage B (Green). When ready, instantly switch spotlight to Stage B.

```yaml
# docker-compose.blue-green.yml
version: '3.8'

services:
  # Blue (current production)
  api-blue:
    image: college-api:v1.0
    ports:
      - "3001:3000"
    environment:
      - DEPLOYMENT=blue

  # Green (new version)
  api-green:
    image: college-api:v1.1
    ports:
      - "3002:3000"
    environment:
      - DEPLOYMENT=green

  # Load balancer switches between blue and green
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx-blue.conf:/etc/nginx/nginx.conf:ro
```

**Deployment Steps**:
1. Deploy new version to Green (3002)
2. Test Green thoroughly
3. Switch Nginx config to route traffic to Green
4. Reload Nginx (zero downtime)
5. Keep Blue running for quick rollback if needed
6. After verification, decommission Blue

### 2. Rolling Deployment

Gradually replace old instances with new ones.

```bash
# Using Docker Swarm
docker service update --image college-api:v1.1 --update-parallelism 2 --update-delay 10s college-api
```

### 3. Canary Deployment

Route small percentage of traffic to new version.

```nginx
# nginx.conf - Canary routing
upstream backend {
    server api-v1:3000 weight=9;  # 90% traffic
    server api-v2:3000 weight=1;  # 10% traffic (canary)
}
```

**Process**:
1. Deploy new version alongside old
2. Route 10% traffic to new version
3. Monitor metrics (errors, latency)
4. Gradually increase traffic (10% → 25% → 50% → 100%)
5. Decommission old version

### 4. Feature Flags

Control feature visibility without redeploying.

```javascript
// Feature flag service
const featureFlags = {
    newEnrollmentFlow: {
        enabled: true,
        rolloutPercentage: 50 // 50% of users
    },
    advancedSearch: {
        enabled: false
    }
};

function isFeatureEnabled(featureName, userId) {
    const feature = featureFlags[featureName];

    if (!feature || !feature.enabled) {
        return false;
    }

    // Gradual rollout based on user ID
    if (feature.rolloutPercentage) {
        const hash = hashUserId(userId);
        return (hash % 100) < feature.rolloutPercentage;
    }

    return true;
}

// Usage in code
app.post('/api/enrollments', async (req, res) => {
    const userId = req.user.id;

    if (isFeatureEnabled('newEnrollmentFlow', userId)) {
        // Use new enrollment flow
        return newEnrollmentController.enroll(req, res);
    } else {
        // Use old enrollment flow
        return oldEnrollmentController.enroll(req, res);
    }
});
```

---

## Monitoring and Logging

### Structured Logging with Winston

```javascript
// config/logger.js
const winston = require('winston');

const logger = winston.createLogger({
    level: process.env.LOG_LEVEL || 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
    ),
    defaultMeta: {
        service: 'college-api',
        environment: process.env.NODE_ENV
    },
    transports: [
        // Write to console
        new winston.transports.Console({
            format: winston.format.combine(
                winston.format.colorize(),
                winston.format.simple()
            )
        }),
        // Write errors to file
        new winston.transports.File({
            filename: 'logs/error.log',
            level: 'error'
        }),
        // Write all logs to file
        new winston.transports.File({
            filename: 'logs/combined.log'
        })
    ]
});

module.exports = logger;
```

### Application Monitoring

**Prometheus + Grafana**:

```javascript
// monitoring/metrics.js
const client = require('prom-client');

// Create metrics
const httpRequestDuration = new client.Histogram({
    name: 'http_request_duration_seconds',
    help: 'Duration of HTTP requests in seconds',
    labelNames: ['method', 'route', 'status_code']
});

const httpRequestTotal = new client.Counter({
    name: 'http_requests_total',
    help: 'Total number of HTTP requests',
    labelNames: ['method', 'route', 'status_code']
});

const activeConnections = new client.Gauge({
    name: 'active_connections',
    help: 'Number of active connections'
});

// Middleware to track metrics
function metricsMiddleware(req, res, next) {
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
    });

    next();
}

// Expose metrics endpoint
app.get('/metrics', async (req, res) => {
    res.set('Content-Type', client.register.contentType);
    res.end(await client.register.metrics());
});

module.exports = { metricsMiddleware };
```

### Error Tracking with Sentry

```javascript
// config/sentry.js
const Sentry = require('@sentry/node');

Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    tracesSampleRate: 1.0
});

// Use in Express
app.use(Sentry.Handlers.requestHandler());
app.use(Sentry.Handlers.tracingHandler());

// ... your routes ...

// Error handler
app.use(Sentry.Handlers.errorHandler());
```

---

## Best Practices

### Testing Best Practices

- ✅ Write tests before or alongside code (TDD)
- ✅ Follow AAA pattern (Arrange, Act, Assert)
- ✅ Test behavior, not implementation
- ✅ Keep tests independent and isolated
- ✅ Use descriptive test names
- ✅ Mock external dependencies
- ✅ Aim for >80% code coverage
- ✅ Run tests in CI/CD pipeline
- ❌ Don't test third-party libraries
- ❌ Don't write flaky tests
- ❌ Don't skip failing tests

### Deployment Best Practices

- ✅ Use environment variables for configuration
- ✅ Implement health check endpoints
- ✅ Use Docker for consistency
- ✅ Automate with CI/CD
- ✅ Deploy during low-traffic periods
- ✅ Have rollback plan ready
- ✅ Monitor metrics after deployment
- ✅ Use database migrations
- ✅ Keep deployment scripts in version control
- ❌ Don't deploy on Fridays (hard to fix issues over weekend)
- ❌ Don't skip testing in staging environment
- ❌ Don't deploy without backups

### Docker Best Practices

- ✅ Use official base images
- ✅ Use multi-stage builds for smaller images
- ✅ Use .dockerignore to exclude unnecessary files
- ✅ Run as non-root user
- ✅ Use specific image tags (not :latest)
- ✅ Implement health checks
- ✅ Use docker-compose for local development
- ❌ Don't store secrets in images
- ❌ Don't install unnecessary packages

### CI/CD Best Practices

- ✅ Run tests on every commit
- ✅ Fail fast (stop on first error)
- ✅ Cache dependencies
- ✅ Parallelize when possible
- ✅ Keep pipelines fast (<10 minutes)
- ✅ Separate build and deploy stages
- ✅ Notify team of failures
- ❌ Don't deploy automatically to production without approval
- ❌ Don't ignore test failures

---

## Complete Testing Example

```javascript
// __tests__/enrollment-flow.test.js
const request = require('supertest');
const app = require('../../src/app');
const { sequelize } = require('../../src/config/database');
const Student = require('../../src/models/student');
const Course = require('../../src/models/course');
const Enrollment = require('../../src/models/enrollment');

describe('Complete Enrollment Flow', () => {
    let student, course1, course2, authToken;

    beforeAll(async () => {
        await sequelize.sync({ force: true });
    });

    afterAll(async () => {
        await sequelize.close();
    });

    beforeEach(async () => {
        // Clean database
        await Enrollment.destroy({ where: {}, force: true });
        await Student.destroy({ where: {}, force: true });
        await Course.destroy({ where: {}, force: true });

        // Create test student
        student = await Student.create({
            firstName: 'John',
            lastName: 'Doe',
            email: 'john@example.com',
            password: 'SecurePass123!',
            department: 'CS',
            dateOfBirth: '2000-01-01'
        });

        // Create test courses
        course1 = await Course.create({
            code: 'CS101',
            name: 'Intro to CS',
            credits: 3,
            maxEnrollments: 30,
            prerequisites: []
        });

        course2 = await Course.create({
            code: 'CS201',
            name: 'Data Structures',
            credits: 4,
            maxEnrollments: 25,
            prerequisites: [course1.id]
        });

        // Get auth token
        const loginRes = await request(app)
            .post('/api/auth/login')
            .send({
                email: 'john@example.com',
                password: 'SecurePass123!'
            });

        authToken = loginRes.body.token;
    });

    describe('Successful Enrollment', () => {
        it('should complete full enrollment flow', async () => {
            // 1. Browse available courses
            const coursesRes = await request(app)
                .get('/api/courses')
                .set('Authorization', `Bearer ${authToken}`)
                .expect(200);

            expect(coursesRes.body.data).toHaveLength(2);

            // 2. Get course details
            const courseDetailRes = await request(app)
                .get(`/api/courses/${course1.id}`)
                .set('Authorization', `Bearer ${authToken}`)
                .expect(200);

            expect(courseDetailRes.body.data.code).toBe('CS101');

            // 3. Enroll in course
            const enrollRes = await request(app)
                .post('/api/enrollments')
                .set('Authorization', `Bearer ${authToken}`)
                .send({
                    courseId: course1.id
                })
                .expect(201);

            expect(enrollRes.body.data.status).toBe('active');

            // 4. Verify enrollment appears in student's courses
            const myCoursesRes = await request(app)
                .get('/api/students/me/enrollments')
                .set('Authorization', `Bearer ${authToken}`)
                .expect(200);

            expect(myCoursesRes.body.data).toHaveLength(1);
            expect(myCoursesRes.body.data[0].course.code).toBe('CS101');

            // 5. Verify enrollment count increased
            const updatedCourseRes = await request(app)
                .get(`/api/courses/${course1.id}`)
                .set('Authorization', `Bearer ${authToken}`)
                .expect(200);

            expect(updatedCourseRes.body.data.currentEnrollments).toBe(1);
        });
    });

    describe('Enrollment Business Rules', () => {
        it('should prevent duplicate enrollment', async () => {
            // First enrollment
            await request(app)
                .post('/api/enrollments')
                .set('Authorization', `Bearer ${authToken}`)
                .send({ courseId: course1.id })
                .expect(201);

            // Duplicate enrollment
            const duplicateRes = await request(app)
                .post('/api/enrollments')
                .set('Authorization', `Bearer ${authToken}`)
                .send({ courseId: course1.id })
                .expect(409);

            expect(duplicateRes.body.error).toMatch(/already enrolled/i);
        });

        it('should enforce prerequisites', async () => {
            // Try to enroll in CS201 without completing CS101
            const res = await request(app)
                .post('/api/enrollments')
                .set('Authorization', `Bearer ${authToken}`)
                .send({ courseId: course2.id })
                .expect(400);

            expect(res.body.error).toMatch(/prerequisite/i);
        });

        it('should allow enrollment after completing prerequisite', async () => {
            // Enroll in CS101
            await request(app)
                .post('/api/enrollments')
                .set('Authorization', `Bearer ${authToken}`)
                .send({ courseId: course1.id })
                .expect(201);

            // Complete CS101
            const enrollment = await Enrollment.findOne({
                where: { studentId: student.id, courseId: course1.id }
            });
            await enrollment.update({ status: 'completed', grade: 'A' });

            // Now enroll in CS201
            const res = await request(app)
                .post('/api/enrollments')
                .set('Authorization', `Bearer ${authToken}`)
                .send({ courseId: course2.id })
                .expect(201);

            expect(res.body.data.courseId).toBe(course2.id);
        });

        it('should prevent enrollment when course is full', async () => {
            // Fill course to capacity
            course1.maxEnrollments = 1;
            await course1.save();

            // Create another student and enroll them
            const otherStudent = await Student.create({
                firstName: 'Jane',
                lastName: 'Smith',
                email: 'jane@example.com',
                password: 'SecurePass123!',
                department: 'CS',
                dateOfBirth: '2000-02-01'
            });

            await Enrollment.create({
                studentId: otherStudent.id,
                courseId: course1.id,
                status: 'active'
            });

            // Try to enroll
            const res = await request(app)
                .post('/api/enrollments')
                .set('Authorization', `Bearer ${authToken}`)
                .send({ courseId: course1.id })
                .expect(400);

            expect(res.body.error).toMatch(/full/i);
        });
    });
});
```

---

## Summary

### Testing Checklist

- [ ] Unit tests for all services
- [ ] Integration tests for API endpoints
- [ ] E2E tests for critical user flows
- [ ] Test coverage > 80%
- [ ] Tests run in CI/CD pipeline
- [ ] Mocked external dependencies
- [ ] Database properly cleaned between tests

### Deployment Checklist

- [ ] Docker images built and tagged
- [ ] Environment variables configured
- [ ] Database migrations prepared
- [ ] Health checks implemented
- [ ] Monitoring and logging set up
- [ ] Backup and rollback plan ready
- [ ] CI/CD pipeline configured
- [ ] Staging environment tested
- [ ] Load testing completed
- [ ] Security scan passed

### Post-Deployment Checklist

- [ ] Monitor error rates
- [ ] Check response times
- [ ] Verify database queries
- [ ] Monitor resource usage (CPU, memory)
- [ ] Check application logs
- [ ] Verify health check endpoints
- [ ] Test critical user flows
- [ ] Monitor cache hit rates
- [ ] Keep rollback plan ready

---

## Congratulations!

You've completed the comprehensive Backend Development Guide! You now understand:

- ✅ **Testing**: Unit, integration, E2E, and TDD
- ✅ **Containerization**: Docker and Docker Compose
- ✅ **CI/CD**: Automated testing and deployment
- ✅ **Deployment**: Blue-green, rolling, canary strategies
- ✅ **Monitoring**: Logging, metrics, and error tracking

### What's Next?

1. **Practice**: Build a complete application with tests
2. **Explore Advanced Topics**:
   - Microservices architecture
   - Message queues (RabbitMQ, Kafka)
   - GraphQL
   - WebSockets for real-time features
   - Serverless architecture
3. **Learn Cloud Platforms**:
   - AWS (EC2, RDS, S3, Lambda)
   - Google Cloud Platform
   - Azure
   - Kubernetes for orchestration

---

**Previous**: [Part 9: Performance Optimization and Caching](./09_Performance_Caching.md)

[← Back to Index](./README.md)

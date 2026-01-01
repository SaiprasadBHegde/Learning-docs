# Part 4: Business Logic Layer (Service Layer)

**Author**: Saiprasad Hegde

---

## Table of Contents
1. [What is Business Logic?](#what-is-business-logic)
2. [Service Layer Pattern](#service-layer-pattern)
3. [Implementing Services](#implementing-services)
4. [Dependency Injection](#dependency-injection)
5. [Business Rules and Validation](#business-rules-and-validation)
6. [Transaction Management](#transaction-management)

---

## What is Business Logic?

**Business Logic** is the set of rules that define how your application works - the "what" and "why" of your app.

### Analogy: Restaurant Kitchen Rules

```
Business Logic = Chef's Recipe Rules
┌────────────────────────────────────────┐
│ Rules for making a Caesar Salad:      │
│ 1. Must have romaine lettuce           │
│ 2. Dressing must be made fresh daily   │
│ 3. Croutons must be toasted            │
│ 4. Parmesan must be freshly grated     │
│ 5. Cannot substitute ingredients       │
└────────────────────────────────────────┘
```

### Business Logic vs Other Layers

| Layer | Responsibility | Example |
|-------|---------------|---------|
| API | HTTP handling | "Received POST request with body {..}" |
| **Service** | **Business rules** | **"Check if student meets prerequisites"** |
| Repository | Data access | "Query database for student record" |
| Database | Data storage | "Store enrollment record in table" |

---

## Service Layer Pattern

### Purpose

The Service Layer:
- Encapsulates business logic
- Coordinates operations across multiple repositories
- Enforces business rules
- Manages transactions
- Is reusable across different APIs (REST, GraphQL, CLI)

### Structure

```
┌─────────────────────────────────────────┐
│         API Layer                       │
│   (Express/FastAPI Controllers)         │
└──────────────┬──────────────────────────┘
               │
               ↓
┌─────────────────────────────────────────┐
│      Business Logic Layer               │
│         (Services)                      │
│                                         │
│  - enrollmentService                    │
│  - studentService                       │
│  - gradeService                         │
└──────────────┬──────────────────────────┘
               │
               ↓
┌─────────────────────────────────────────┐
│      Data Access Layer                  │
│       (Repositories)                    │
└─────────────────────────────────────────┘
```

---

## Implementing Services

### Service Class Structure (Node.js)

```javascript
// services/enrollmentService.js
class EnrollmentService {
    constructor(
        studentRepository,
        courseRepository,
        enrollmentRepository
    ) {
        this.studentRepo = studentRepository;
        this.courseRepo = courseRepository;
        this.enrollmentRepo = enrollmentRepository;
    }

    // Public methods - use cases
    async enrollStudent(studentId, courseId) { }
    async dropCourse(enrollmentId, userId, userRole) { }
    async submitGrade(enrollmentId, grade, professorId) { }
    async getEnrollmentById(enrollmentId) { }

    // Private methods - business rules
    async _validateEnrollment(student, course) { }
    async _checkPrerequisites(student, course) { }
    async _checkCreditLimit(studentId, courseCredits) { }
    _getCurrentSemester() { }
}

module.exports = EnrollmentService;
```

### Complete Service Implementation

**services/enrollmentService.js**:
```javascript
const { AppError } = require('../utils/errors');

class EnrollmentService {
    constructor(studentRepo, courseRepo, enrollmentRepo, gradeRepo) {
        this.studentRepo = studentRepo;
        this.courseRepo = courseRepo;
        this.enrollmentRepo = enrollmentRepo;
        this.gradeRepo = gradeRepo;
    }

    /**
     * Enroll a student in a course
     * Business Rules:
     * 1. Student and course must exist
     * 2. Student must not be already enrolled
     * 3. Student must meet prerequisites
     * 4. Course must have available capacity
     * 5. Student must not exceed credit limit (18 credits)
     */
    async enrollStudent(studentId, courseId) {
        // 1. Fetch student and course
        const [student, course] = await Promise.all([
            this.studentRepo.findById(studentId),
            this.courseRepo.findById(courseId)
        ]);

        if (!student) {
            throw new AppError('Student not found', 404);
        }

        if (!course) {
            throw new AppError('Course not found', 404);
        }

        // 2. Check if already enrolled
        const existingEnrollment = await this.enrollmentRepo
            .findByStudentAndCourse(studentId, courseId);

        if (existingEnrollment) {
            throw new AppError('Student already enrolled in this course', 400);
        }

        // 3. Validate enrollment business rules
        await this._validateEnrollment(student, course);

        // 4. Create enrollment
        const semester = this._getCurrentSemester();
        const enrollment = await this.enrollmentRepo.create({
            studentId,
            courseId,
            semester,
            status: 'enrolled',
            enrolledAt: new Date()
        });

        // 5. Update course enrollment count
        await this.courseRepo.incrementEnrollmentCount(courseId);

        // 6. Send notification (side effect)
        await this._sendEnrollmentConfirmation(student, course);

        return enrollment;
    }

    /**
     * Drop a course (student unenrolls)
     * Business Rules:
     * 1. Enrollment must exist
     * 2. Must be before drop deadline
     * 3. Student can only drop their own courses (or admin)
     */
    async dropCourse(enrollmentId, userId, userRole) {
        const enrollment = await this.enrollmentRepo.findById(enrollmentId);

        if (!enrollment) {
            throw new AppError('Enrollment not found', 404);
        }

        // Authorization: student can only drop their own courses
        if (userRole === 'student' && enrollment.studentId !== userId) {
            throw new AppError('Cannot drop another student\'s course', 403);
        }

        // Check drop deadline
        if (!this._isBeforeDropDeadline(enrollment.semester)) {
            throw new AppError('Drop deadline has passed', 400);
        }

        // Update enrollment status
        await this.enrollmentRepo.update(enrollmentId, {
            status: 'dropped',
            droppedAt: new Date()
        });

        // Decrement course enrollment count
        await this.courseRepo.decrementEnrollmentCount(enrollment.courseId);

        return true;
    }

    /**
     * Submit grade for a course
     * Business Rules:
     * 1. Enrollment must exist
     * 2. Course must be completed
     * 3. Only assigned professor can submit grade
     * 4. Grade must be valid (A, B+, B, etc.)
     */
    async submitGrade(enrollmentId, grade, professorId) {
        const enrollment = await this.enrollmentRepo.findById(enrollmentId);

        if (!enrollment) {
            throw new AppError('Enrollment not found', 404);
        }

        // Get course to verify professor
        const course = await this.courseRepo.findById(enrollment.courseId);

        if (course.professorId !== professorId) {
            throw new AppError('Only assigned professor can submit grades', 403);
        }

        // Validate grade
        if (!this._isValidGrade(grade)) {
            throw new AppError('Invalid grade format', 400);
        }

        // Update enrollment with grade
        await this.enrollmentRepo.update(enrollmentId, {
            grade,
            status: 'completed',
            completedAt: new Date()
        });

        // Recalculate student GPA
        const student = await this.studentRepo.findById(enrollment.studentId);
        const newGPA = await this.gradeRepo.calculateGPA(student.id);

        await this.studentRepo.update(student.id, {
            gpa: newGPA
        });

        return { grade, gpa: newGPA };
    }

    // ========== PRIVATE METHODS (Business Rules) ==========

    /**
     * Validate all enrollment business rules
     */
    async _validateEnrollment(student, course) {
        // Rule: Check prerequisites
        const hasPrereqs = await this._checkPrerequisites(student, course);
        if (!hasPrereqs) {
            const prereqCourses = await this._getPrerequisiteCourseNames(
                course.prerequisites
            );
            throw new AppError(
                `Missing prerequisites: ${prereqCourses.join(', ')}`,
                400
            );
        }

        // Rule: Check course capacity
        const enrolledCount = await this.enrollmentRepo
            .countByCourse(course.id);

        if (enrolledCount >= course.maxCapacity) {
            throw new AppError('Course is at maximum capacity', 400);
        }

        // Rule: Check credit limit
        const exceedsLimit = await this._checkCreditLimit(
            student.id,
            course.credits
        );

        if (exceedsLimit) {
            throw new AppError(
                'Exceeds maximum credit limit (18 credits per semester)',
                400
            );
        }

        // Rule: Check student status
        if (student.status !== 'active') {
            throw new AppError(
                `Cannot enroll: student status is ${student.status}`,
                400
            );
        }
    }

    /**
     * Check if student has completed all prerequisites
     */
    async _checkPrerequisites(student, course) {
        if (!course.prerequisites || course.prerequisites.length === 0) {
            return true;
        }

        const completedCourses = await this.enrollmentRepo
            .getCompletedCourses(student.id);

        const completedCourseIds = completedCourses
            .map(enrollment => enrollment.courseId);

        // Student must have completed ALL prerequisites
        return course.prerequisites.every(prereqId =>
            completedCourseIds.includes(prereqId)
        );
    }

    /**
     * Check if adding this course would exceed credit limit
     */
    async _checkCreditLimit(studentId, newCourseCredits) {
        const MAX_CREDITS = 18;
        const semester = this._getCurrentSemester();

        const currentCredits = await this.enrollmentRepo
            .getCurrentSemesterCredits(studentId, semester);

        return (currentCredits + newCourseCredits) > MAX_CREDITS;
    }

    /**
     * Get current semester (Spring, Summer, Fall)
     */
    _getCurrentSemester() {
        const now = new Date();
        const year = now.getFullYear();
        const month = now.getMonth() + 1;

        if (month >= 1 && month <= 5) return `Spring ${year}`;
        if (month >= 6 && month <= 8) return `Summer ${year}`;
        return `Fall ${year}`;
    }

    /**
     * Check if before drop deadline
     */
    _isBeforeDropDeadline(semester) {
        const deadlines = {
            'Spring 2025': new Date('2025-03-01'),
            'Summer 2025': new Date('2025-07-01'),
            'Fall 2025': new Date('2025-11-01')
        };

        const deadline = deadlines[semester];
        if (!deadline) return false;

        return new Date() < deadline;
    }

    /**
     * Validate grade format
     */
    _isValidGrade(grade) {
        const validGrades = [
            'A', 'A-',
            'B+', 'B', 'B-',
            'C+', 'C', 'C-',
            'D+', 'D',
            'F'
        ];

        return validGrades.includes(grade);
    }

    /**
     * Get prerequisite course names for error message
     */
    async _getPrerequisiteCourseNames(prerequisiteIds) {
        const courses = await Promise.all(
            prerequisiteIds.map(id => this.courseRepo.findById(id))
        );

        return courses.map(course => course.code);
    }

    /**
     * Send enrollment confirmation email
     */
    async _sendEnrollmentConfirmation(student, course) {
        // In real app, integrate with email service
        console.log(
            `Email sent to ${student.email}: ` +
            `You are enrolled in ${course.code} - ${course.name}`
        );
    }
}

module.exports = EnrollmentService;
```

---

## Dependency Injection

### What is Dependency Injection?

**Dependency Injection (DI)** is passing dependencies to a class instead of creating them inside.

**Analogy**: Chef's Kitchen
- **Without DI**: Chef makes their own knives, pots, and ingredients
- **With DI**: Kitchen manager provides chef with tools and ingredients

### Without DI (Bad)

```javascript
class EnrollmentService {
    constructor() {
        // ❌ Creates dependencies inside - tightly coupled
        this.studentRepo = new StudentRepository();
        this.courseRepo = new CourseRepository();
        this.enrollmentRepo = new EnrollmentRepository();
    }
}
```

**Problems**:
- Hard to test (can't mock repositories)
- Hard to change implementations
- Creates unwanted dependencies

### With DI (Good)

```javascript
class EnrollmentService {
    constructor(studentRepo, courseRepo, enrollmentRepo) {
        // ✅ Dependencies injected from outside
        this.studentRepo = studentRepo;
        this.courseRepo = courseRepo;
        this.enrollmentRepo = enrollmentRepo;
    }
}

// Usage
const studentRepo = new StudentRepository(db);
const courseRepo = new CourseRepository(db);
const enrollmentRepo = new EnrollmentRepository(db);

const enrollmentService = new EnrollmentService(
    studentRepo,
    courseRepo,
    enrollmentRepo
);
```

**Benefits**:
- Easy to test (inject mock repositories)
- Loose coupling
- Flexible (can swap implementations)

### DI Container (Advanced)

**services/container.js**:
```javascript
const StudentRepository = require('../repositories/studentRepository');
const CourseRepository = require('../repositories/courseRepository');
const EnrollmentRepository = require('../repositories/enrollmentRepository');

const StudentService = require('./studentService');
const CourseService = require('./courseService');
const EnrollmentService = require('./enrollmentService');

class Container {
    constructor(db) {
        this.db = db;
        this.repositories = {};
        this.services = {};
    }

    getStudentRepository() {
        if (!this.repositories.student) {
            this.repositories.student = new StudentRepository(this.db);
        }
        return this.repositories.student;
    }

    getCourseRepository() {
        if (!this.repositories.course) {
            this.repositories.course = new CourseRepository(this.db);
        }
        return this.repositories.course;
    }

    getEnrollmentRepository() {
        if (!this.repositories.enrollment) {
            this.repositories.enrollment = new EnrollmentRepository(this.db);
        }
        return this.repositories.enrollment;
    }

    getStudentService() {
        if (!this.services.student) {
            this.services.student = new StudentService(
                this.getStudentRepository()
            );
        }
        return this.services.student;
    }

    getEnrollmentService() {
        if (!this.services.enrollment) {
            this.services.enrollment = new EnrollmentService(
                this.getStudentRepository(),
                this.getCourseRepository(),
                this.getEnrollmentRepository()
            );
        }
        return this.services.enrollment;
    }
}

module.exports = Container;
```

**Usage**:
```javascript
const container = new Container(db);

// In controllers
const enrollmentService = container.getEnrollmentService();
await enrollmentService.enrollStudent(studentId, courseId);
```

---

## Business Rules and Validation

### Types of Validation

1. **Input Validation** (API Layer): Format, type, required fields
2. **Business Validation** (Service Layer): Business rules, context

**Example**:
```javascript
// API Layer - Input validation
if (!req.body.studentId) {
    return res.status(400).json({ error: 'studentId is required' });
}

// Service Layer - Business validation
if (student.status !== 'active') {
    throw new Error('Student is not active');
}
```

### Complex Business Rules

**GPA Calculation Service**:
```javascript
class GradeService {
    constructor(enrollmentRepo) {
        this.enrollmentRepo = enrollmentRepo;
    }

    async calculateGPA(studentId) {
        const enrollments = await this.enrollmentRepo
            .getGradedEnrollments(studentId);

        if (enrollments.length === 0) return 0;

        let totalPoints = 0;
        let totalCredits = 0;

        for (const enrollment of enrollments) {
            const gradePoints = this._getGradePoints(enrollment.grade);
            const courseCredits = enrollment.course.credits;

            totalPoints += gradePoints * courseCredits;
            totalCredits += courseCredits;
        }

        return totalCredits > 0
            ? (totalPoints / totalCredits).toFixed(2)
            : 0;
    }

    _getGradePoints(grade) {
        const gradeScale = {
            'A': 4.0, 'A-': 3.7,
            'B+': 3.3, 'B': 3.0, 'B-': 2.7,
            'C+': 2.3, 'C': 2.0, 'C-': 1.7,
            'D+': 1.3, 'D': 1.0,
            'F': 0.0
        };

        return gradeScale[grade] || 0;
    }

    /**
     * Business Rule: Determine academic standing
     */
    async getAcademicStanding(studentId) {
        const gpa = await this.calculateGPA(studentId);

        if (gpa >= 3.5) return 'Dean\'s List';
        if (gpa >= 3.0) return 'Good Standing';
        if (gpa >= 2.0) return 'Satisfactory';
        if (gpa >= 1.0) return 'Academic Warning';
        return 'Academic Probation';
    }
}
```

---

## Transaction Management

### What are Transactions?

**Transaction**: A group of operations that must ALL succeed or ALL fail.

**Analogy**: Bank transfer
- Debit $100 from Account A
- Credit $100 to Account B
- Both must happen or neither (no half-transfers!)

### Without Transactions (Dangerous)

```javascript
async enrollStudent(studentId, courseId) {
    // 1. Create enrollment
    await this.enrollmentRepo.create({ studentId, courseId });

    // 2. Increment course count
    await this.courseRepo.incrementCount(courseId);

    // ⚠️ If step 2 fails, enrollment exists but count is wrong!
}
```

### With Transactions (Safe)

**Node.js (Sequelize)**:
```javascript
const { sequelize } = require('../models');

async enrollStudent(studentId, courseId) {
    const transaction = await sequelize.transaction();

    try {
        // All operations in same transaction
        const enrollment = await this.enrollmentRepo.create(
            { studentId, courseId },
            { transaction }
        );

        await this.courseRepo.incrementCount(
            courseId,
            { transaction }
        );

        // Commit if all succeed
        await transaction.commit();

        return enrollment;
    } catch (error) {
        // Rollback if any fail
        await transaction.rollback();
        throw error;
    }
}
```

---

## Key Takeaways

1. **Service Layer** contains business logic and rules
2. **Coordinate** multiple repositories
3. **Dependency Injection** makes code testable and flexible
4. **Validate** business rules in services, not API layer
5. **Transactions** ensure data consistency
6. **Reusable** across different API types (REST, GraphQL, CLI)

---

**Next**: [05 - Data Access Layer →](05_Data_Access_Layer.md)

---

**Previous**: [← 03 - API Layer](03_API_Layer.md) | **Next**: [05 - Data Access Layer →](05_Data_Access_Layer.md)

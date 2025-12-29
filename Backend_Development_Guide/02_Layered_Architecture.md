# Part 2: Layered Architecture

**Author**: Saiprasad Hegde

---

## Table of Contents
1. [What is Layered Architecture?](#what-is-layered-architecture)
2. [Why Decoupling Matters](#why-decoupling-matters)
3. [The Four Main Layers](#the-four-main-layers)
4. [Layer Communication](#layer-communication)
5. [Complete Example: Student Enrollment](#complete-example-student-enrollment)

---

## What is Layered Architecture?

**Layered architecture** (also called N-tier architecture) organizes code into separate layers, each with specific responsibilities.

### Analogy: Building Construction

Think of a building:

```
┌─────────────────────────────────────┐
│     ROOF (Presentation Layer)       │  ← What users see and interact with
├─────────────────────────────────────┤
│     WALLS (Business Logic Layer)    │  ← Rules and operations
├─────────────────────────────────────┤
│  PLUMBING (Data Access Layer)       │  ← How to get/store data
├─────────────────────────────────────┤
│  FOUNDATION (Database Layer)        │  ← Where data is stored
└─────────────────────────────────────┘
```

Each layer:
- Has a **specific purpose**
- Can be **modified independently**
- Only **communicates with adjacent layers**

### Without Layers (Bad)

```javascript
// Everything in one file - MESSY!
app.post('/api/enroll', async (req, res) => {
    // Validate input
    if (!req.body.studentId) return res.status(400).send("Missing studentId");

    // Database query
    const student = await db.query("SELECT * FROM students WHERE id = ?", [req.body.studentId]);

    // Business logic
    if (student.creditsCompleted < 30) {
        return res.status(400).send("Need 30 credits");
    }

    // More database queries
    const course = await db.query("SELECT * FROM courses WHERE id = ?", [req.body.courseId]);
    const enrolled = await db.query("SELECT COUNT(*) FROM enrollments WHERE course_id = ?", [req.body.courseId]);

    // More business logic
    if (enrolled >= course.maxCapacity) {
        return res.status(400).send("Course full");
    }

    // Insert enrollment
    await db.query("INSERT INTO enrollments (student_id, course_id) VALUES (?, ?)",
        [req.body.studentId, req.body.courseId]);

    res.json({ success: true });
});
```

**Problems**:
- ❌ Hard to test
- ❌ Hard to maintain
- ❌ Can't reuse logic
- ❌ Mixing concerns

### With Layers (Good)

```javascript
// API Layer - handles HTTP
app.post('/api/enroll', async (req, res) => {
    try {
        const result = await enrollmentService.enrollStudent(
            req.body.studentId,
            req.body.courseId
        );
        res.json(result);
    } catch (error) {
        res.status(error.statusCode || 500).json({ error: error.message });
    }
});

// Service Layer - business logic
class EnrollmentService {
    async enrollStudent(studentId, courseId) {
        const student = await studentRepository.findById(studentId);
        const course = await courseRepository.findById(courseId);

        if (!this.canEnroll(student, course)) {
            throw new Error("Cannot enroll");
        }

        return await enrollmentRepository.create(studentId, courseId);
    }

    canEnroll(student, course) {
        return student.creditsCompleted >= 30 &&
               course.enrolledCount < course.maxCapacity;
    }
}

// Repository Layer - data access
class StudentRepository {
    async findById(id) {
        return await Student.findByPk(id); // ORM call
    }
}
```

**Benefits**:
- ✅ Easy to test each layer
- ✅ Easy to maintain
- ✅ Reusable logic
- ✅ Clear separation of concerns

---

## Why Decoupling Matters

**Decoupling** means making layers independent so changes in one don't break others.

### Analogy: Restaurant Kitchen

```
┌──────────────────────────────────────────────┐
│  DINING ROOM (Presentation Layer)           │
│  - Waiters take orders                       │
│  - Serve food                                │
└──────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────┐
│  HEAD CHEF (Business Logic Layer)           │
│  - Decides how to prepare dishes            │
│  - Ensures quality standards                 │
│  - Manages cooking workflow                  │
└──────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────┐
│  LINE COOKS (Data Access Layer)             │
│  - Execute cooking instructions              │
│  - Get ingredients from pantry               │
└──────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────┐
│  PANTRY (Database Layer)                     │
│  - Stores all ingredients                    │
│  - Organized for easy access                 │
└──────────────────────────────────────────────┘
```

**Benefits of Decoupling**:

1. **Change Database**: Switch from PostgreSQL to MongoDB? Only change Data Access Layer
2. **Change Business Rules**: Update enrollment logic? Only change Service Layer
3. **Add New API**: Add GraphQL alongside REST? Only add new Presentation Layer
4. **Testing**: Test business logic without touching database

### Example: Switching Databases

**Without Decoupling**:
```javascript
// SQL queries everywhere
app.get('/students/:id', async (req, res) => {
    const result = await db.query("SELECT * FROM students WHERE id = ?", [req.params.id]);
    res.json(result);
});

// To switch to MongoDB, you'd have to change EVERY route!
```

**With Decoupling**:
```javascript
// API layer stays the same
app.get('/students/:id', async (req, res) => {
    const student = await studentService.getById(req.params.id);
    res.json(student);
});

// Only change repository layer
class StudentRepository {
    async findById(id) {
        // Before: SQL
        // return await db.query("SELECT * FROM students WHERE id = ?", [id]);

        // After: MongoDB
        return await Student.findOne({ _id: id });
    }
}
```

---

## The Four Main Layers

### Layer 1: Presentation Layer (API Layer)

**Responsibility**: Handle HTTP requests and responses

**Analogy**: Restaurant waiter - takes orders, serves food

**What it does**:
- Receive HTTP requests
- Validate request format (not business rules)
- Call service layer
- Format responses
- Handle HTTP status codes

**What it does NOT do**:
- Business logic
- Direct database access
- Data transformation logic

**Express.js Example**:
```javascript
// routes/studentRoutes.js
const express = require('express');
const router = express.Router();
const studentController = require('../controllers/studentController');

router.get('/', studentController.getAllStudents);
router.get('/:id', studentController.getStudent);
router.post('/', studentController.createStudent);
router.put('/:id', studentController.updateStudent);
router.delete('/:id', studentController.deleteStudent);

module.exports = router;
```

```javascript
// controllers/studentController.js
class StudentController {
    async getAllStudents(req, res) {
        try {
            const students = await studentService.getAll();
            res.json(students);
        } catch (error) {
            res.status(500).json({ error: error.message });
        }
    }

    async getStudent(req, res) {
        try {
            const student = await studentService.getById(req.params.id);
            if (!student) {
                return res.status(404).json({ error: 'Student not found' });
            }
            res.json(student);
        } catch (error) {
            res.status(500).json({ error: error.message });
        }
    }

    async createStudent(req, res) {
        try {
            const student = await studentService.create(req.body);
            res.status(201).json(student);
        } catch (error) {
            if (error.name === 'ValidationError') {
                return res.status(400).json({ error: error.message });
            }
            res.status(500).json({ error: error.message });
        }
    }
}

module.exports = new StudentController();
```

**FastAPI Example**:
```python
# routes/student_routes.py
from fastapi import APIRouter, HTTPException, status
from services.student_service import StudentService
from schemas.student import StudentCreate, StudentResponse

router = APIRouter()
student_service = StudentService()

@router.get("/", response_model=list[StudentResponse])
async def get_all_students():
    return await student_service.get_all()

@router.get("/{student_id}", response_model=StudentResponse)
async def get_student(student_id: str):
    student = await student_service.get_by_id(student_id)
    if not student:
        raise HTTPException(status_code=404, detail="Student not found")
    return student

@router.post("/", response_model=StudentResponse, status_code=status.HTTP_201_CREATED)
async def create_student(student: StudentCreate):
    return await student_service.create(student)
```

---

### Layer 2: Business Logic Layer (Service Layer)

**Responsibility**: Implement business rules and workflows

**Analogy**: Head chef - decides HOW to prepare food according to recipes

**What it does**:
- Enforce business rules
- Coordinate multiple operations
- Transform data for business needs
- Validate business logic
- Orchestrate workflows

**What it does NOT do**:
- HTTP-specific code
- Direct database queries
- Handle request/response

**Example: Enrollment Business Logic**

```javascript
// services/enrollmentService.js
class EnrollmentService {
    constructor(studentRepo, courseRepo, enrollmentRepo) {
        this.studentRepo = studentRepo;
        this.courseRepo = courseRepo;
        this.enrollmentRepo = enrollmentRepo;
    }

    async enrollStudent(studentId, courseId) {
        // 1. Get student and course
        const student = await this.studentRepo.findById(studentId);
        const course = await this.courseRepo.findById(courseId);

        if (!student) throw new Error('Student not found');
        if (!course) throw new Error('Course not found');

        // 2. Business Rule: Check if student already enrolled
        const existingEnrollment = await this.enrollmentRepo.findByStudentAndCourse(
            studentId,
            courseId
        );
        if (existingEnrollment) {
            throw new Error('Student already enrolled in this course');
        }

        // 3. Business Rule: Check prerequisites
        if (!this.hasPrerequisites(student, course)) {
            throw new Error('Student does not meet prerequisites');
        }

        // 4. Business Rule: Check course capacity
        const enrolledCount = await this.enrollmentRepo.countByCourse(courseId);
        if (enrolledCount >= course.maxCapacity) {
            throw new Error('Course is at maximum capacity');
        }

        // 5. Business Rule: Check credit limit
        const currentCredits = await this.enrollmentRepo.getCurrentSemesterCredits(studentId);
        if (currentCredits + course.credits > 18) {
            throw new Error('Exceeds maximum credit limit (18 credits)');
        }

        // 6. Create enrollment
        const enrollment = await this.enrollmentRepo.create({
            studentId,
            courseId,
            semester: this.getCurrentSemester(),
            status: 'enrolled'
        });

        // 7. Update course enrollment count
        await this.courseRepo.incrementEnrollmentCount(courseId);

        return enrollment;
    }

    hasPrerequisites(student, course) {
        if (!course.prerequisites || course.prerequisites.length === 0) {
            return true;
        }

        const completedCourses = student.completedCourses.map(c => c.courseId);
        return course.prerequisites.every(prereq =>
            completedCourses.includes(prereq)
        );
    }

    getCurrentSemester() {
        const now = new Date();
        const year = now.getFullYear();
        const month = now.getMonth() + 1;

        if (month >= 1 && month <= 5) return `Spring ${year}`;
        if (month >= 6 && month <= 8) return `Summer ${year}`;
        return `Fall ${year}`;
    }
}

module.exports = EnrollmentService;
```

**Python Example**:
```python
# services/enrollment_service.py
from datetime import datetime
from repositories.student_repository import StudentRepository
from repositories.course_repository import CourseRepository
from repositories.enrollment_repository import EnrollmentRepository

class EnrollmentService:
    def __init__(
        self,
        student_repo: StudentRepository,
        course_repo: CourseRepository,
        enrollment_repo: EnrollmentRepository
    ):
        self.student_repo = student_repo
        self.course_repo = course_repo
        self.enrollment_repo = enrollment_repo

    async def enroll_student(self, student_id: str, course_id: str):
        # 1. Get student and course
        student = await self.student_repo.find_by_id(student_id)
        course = await self.course_repo.find_by_id(course_id)

        if not student:
            raise ValueError("Student not found")
        if not course:
            raise ValueError("Course not found")

        # 2. Check if already enrolled
        existing = await self.enrollment_repo.find_by_student_and_course(
            student_id, course_id
        )
        if existing:
            raise ValueError("Student already enrolled in this course")

        # 3. Check prerequisites
        if not self._has_prerequisites(student, course):
            raise ValueError("Student does not meet prerequisites")

        # 4. Check capacity
        enrolled_count = await self.enrollment_repo.count_by_course(course_id)
        if enrolled_count >= course.max_capacity:
            raise ValueError("Course is at maximum capacity")

        # 5. Check credit limit
        current_credits = await self.enrollment_repo.get_current_semester_credits(
            student_id
        )
        if current_credits + course.credits > 18:
            raise ValueError("Exceeds maximum credit limit (18 credits)")

        # 6. Create enrollment
        enrollment = await self.enrollment_repo.create({
            "student_id": student_id,
            "course_id": course_id,
            "semester": self._get_current_semester(),
            "status": "enrolled"
        })

        # 7. Update course count
        await self.course_repo.increment_enrollment_count(course_id)

        return enrollment

    def _has_prerequisites(self, student, course):
        if not course.prerequisites:
            return True

        completed_course_ids = [c.course_id for c in student.completed_courses]
        return all(prereq in completed_course_ids for prereq in course.prerequisites)

    def _get_current_semester(self):
        now = datetime.now()
        year = now.year
        month = now.month

        if 1 <= month <= 5:
            return f"Spring {year}"
        elif 6 <= month <= 8:
            return f"Summer {year}"
        else:
            return f"Fall {year}"
```

---

### Layer 3: Data Access Layer (Repository Pattern)

**Responsibility**: Abstract database operations

**Analogy**: Line cooks - know HOW to get ingredients from pantry

**What it does**:
- Encapsulate database queries
- Provide clean interface for data operations
- Handle data mapping (ORM to domain objects)
- Abstract database-specific logic

**What it does NOT do**:
- Business logic
- HTTP handling
- Data validation (beyond basic constraints)

**Repository Pattern**:
```javascript
// repositories/studentRepository.js
class StudentRepository {
    constructor(db) {
        this.db = db; // Database connection or ORM
    }

    async findAll() {
        return await this.db.Student.findAll({
            attributes: ['id', 'firstName', 'lastName', 'email', 'department'],
            order: [['lastName', 'ASC']]
        });
    }

    async findById(id) {
        return await this.db.Student.findByPk(id, {
            include: [
                { model: this.db.Enrollment, include: [this.db.Course] }
            ]
        });
    }

    async findByEmail(email) {
        return await this.db.Student.findOne({
            where: { email }
        });
    }

    async create(studentData) {
        return await this.db.Student.create(studentData);
    }

    async update(id, updates) {
        const student = await this.findById(id);
        if (!student) return null;

        return await student.update(updates);
    }

    async delete(id) {
        const student = await this.findById(id);
        if (!student) return false;

        await student.destroy();
        return true;
    }

    async getCompletedCourses(studentId) {
        return await this.db.Enrollment.findAll({
            where: {
                studentId,
                status: 'completed'
            },
            include: [this.db.Course]
        });
    }

    async calculateGPA(studentId) {
        const enrollments = await this.db.Enrollment.findAll({
            where: {
                studentId,
                grade: { [this.db.Sequelize.Op.ne]: null }
            }
        });

        if (enrollments.length === 0) return 0;

        const gradePoints = {
            'A': 4.0, 'A-': 3.7,
            'B+': 3.3, 'B': 3.0, 'B-': 2.7,
            'C+': 2.3, 'C': 2.0, 'C-': 1.7,
            'D': 1.0, 'F': 0.0
        };

        let totalPoints = 0;
        let totalCredits = 0;

        for (const enrollment of enrollments) {
            const course = await enrollment.getCourse();
            totalPoints += gradePoints[enrollment.grade] * course.credits;
            totalCredits += course.credits;
        }

        return totalCredits > 0 ? totalPoints / totalCredits : 0;
    }
}

module.exports = StudentRepository;
```

**Python Example**:
```python
# repositories/student_repository.py
from typing import List, Optional
from sqlalchemy.orm import Session, joinedload
from models.student import Student
from models.enrollment import Enrollment

class StudentRepository:
    def __init__(self, db: Session):
        self.db = db

    async def find_all(self) -> List[Student]:
        return self.db.query(Student).order_by(Student.last_name).all()

    async def find_by_id(self, student_id: str) -> Optional[Student]:
        return self.db.query(Student)\
            .options(joinedload(Student.enrollments))\
            .filter(Student.id == student_id)\
            .first()

    async def find_by_email(self, email: str) -> Optional[Student]:
        return self.db.query(Student)\
            .filter(Student.email == email)\
            .first()

    async def create(self, student_data: dict) -> Student:
        student = Student(**student_data)
        self.db.add(student)
        self.db.commit()
        self.db.refresh(student)
        return student

    async def update(self, student_id: str, updates: dict) -> Optional[Student]:
        student = await self.find_by_id(student_id)
        if not student:
            return None

        for key, value in updates.items():
            setattr(student, key, value)

        self.db.commit()
        self.db.refresh(student)
        return student

    async def delete(self, student_id: str) -> bool:
        student = await self.find_by_id(student_id)
        if not student:
            return False

        self.db.delete(student)
        self.db.commit()
        return True

    async def get_completed_courses(self, student_id: str):
        return self.db.query(Enrollment)\
            .filter(
                Enrollment.student_id == student_id,
                Enrollment.status == "completed"
            )\
            .all()

    async def calculate_gpa(self, student_id: str) -> float:
        enrollments = self.db.query(Enrollment)\
            .filter(
                Enrollment.student_id == student_id,
                Enrollment.grade.isnot(None)
            )\
            .all()

        if not enrollments:
            return 0.0

        grade_points = {
            'A': 4.0, 'A-': 3.7,
            'B+': 3.3, 'B': 3.0, 'B-': 2.7,
            'C+': 2.3, 'C': 2.0, 'C-': 1.7,
            'D': 1.0, 'F': 0.0
        }

        total_points = 0
        total_credits = 0

        for enrollment in enrollments:
            total_points += grade_points[enrollment.grade] * enrollment.course.credits
            total_credits += enrollment.course.credits

        return total_points / total_credits if total_credits > 0 else 0.0
```

---

### Layer 4: Database Layer (Models/Entities)

**Responsibility**: Define data structure and relationships

**Analogy**: Pantry organization - how ingredients are stored

**What it does**:
- Define database schema
- Define relationships between entities
- Provide data validation at database level
- Handle migrations

**What it does NOT do**:
- Business logic
- HTTP handling
- Complex queries (that's repository's job)

**Sequelize Example**:
```javascript
// models/Student.js
const { DataTypes } = require('sequelize');

module.exports = (sequelize) => {
    const Student = sequelize.define('Student', {
        id: {
            type: DataTypes.UUID,
            defaultValue: DataTypes.UUIDV4,
            primaryKey: true
        },
        firstName: {
            type: DataTypes.STRING,
            allowNull: false,
            field: 'first_name'
        },
        lastName: {
            type: DataTypes.STRING,
            allowNull: false,
            field: 'last_name'
        },
        email: {
            type: DataTypes.STRING,
            allowNull: false,
            unique: true,
            validate: {
                isEmail: true
            }
        },
        dateOfBirth: {
            type: DataTypes.DATE,
            field: 'date_of_birth'
        },
        department: {
            type: DataTypes.STRING,
            allowNull: false
        },
        enrollmentYear: {
            type: DataTypes.INTEGER,
            field: 'enrollment_year'
        },
        status: {
            type: DataTypes.ENUM('active', 'inactive', 'graduated', 'suspended'),
            defaultValue: 'active'
        }
    }, {
        tableName: 'students',
        timestamps: true,
        underscored: true
    });

    // Associations
    Student.associate = (models) => {
        Student.hasMany(models.Enrollment, {
            foreignKey: 'student_id',
            as: 'enrollments'
        });

        Student.belongsToMany(models.Course, {
            through: models.Enrollment,
            foreignKey: 'student_id',
            as: 'courses'
        });
    };

    return Student;
};
```

```javascript
// models/Course.js
module.exports = (sequelize) => {
    const Course = sequelize.define('Course', {
        id: {
            type: DataTypes.UUID,
            defaultValue: DataTypes.UUIDV4,
            primaryKey: true
        },
        code: {
            type: DataTypes.STRING,
            allowNull: false,
            unique: true
        },
        name: {
            type: DataTypes.STRING,
            allowNull: false
        },
        description: {
            type: DataTypes.TEXT
        },
        credits: {
            type: DataTypes.INTEGER,
            allowNull: false,
            validate: {
                min: 1,
                max: 6
            }
        },
        maxCapacity: {
            type: DataTypes.INTEGER,
            field: 'max_capacity',
            defaultValue: 30
        },
        prerequisites: {
            type: DataTypes.ARRAY(DataTypes.UUID),
            defaultValue: []
        },
        department: {
            type: DataTypes.STRING,
            allowNull: false
        }
    }, {
        tableName: 'courses',
        timestamps: true,
        underscored: true
    });

    Course.associate = (models) => {
        Course.hasMany(models.Enrollment, {
            foreignKey: 'course_id',
            as: 'enrollments'
        });

        Course.belongsToMany(models.Student, {
            through: models.Enrollment,
            foreignKey: 'course_id',
            as: 'students'
        });

        Course.belongsTo(models.Professor, {
            foreignKey: 'professor_id',
            as: 'professor'
        });
    };

    return Course;
};
```

**SQLAlchemy Example**:
```python
# models/student.py
from sqlalchemy import Column, String, Date, Integer, Enum
from sqlalchemy.orm import relationship
from database import Base
import uuid

class Student(Base):
    __tablename__ = "students"

    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    first_name = Column(String, nullable=False)
    last_name = Column(String, nullable=False)
    email = Column(String, unique=True, nullable=False, index=True)
    date_of_birth = Column(Date)
    department = Column(String, nullable=False)
    enrollment_year = Column(Integer)
    status = Column(
        Enum('active', 'inactive', 'graduated', 'suspended', name='student_status'),
        default='active'
    )

    # Relationships
    enrollments = relationship("Enrollment", back_populates="student", cascade="all, delete-orphan")
    courses = relationship("Course", secondary="enrollments", back_populates="students")

    def __repr__(self):
        return f"<Student {self.first_name} {self.last_name}>"
```

```python
# models/course.py
from sqlalchemy import Column, String, Integer, Text, ARRAY
from sqlalchemy.orm import relationship
from database import Base
import uuid

class Course(Base):
    __tablename__ = "courses"

    id = Column(String, primary_key=True, default=lambda: str(uuid.uuid4()))
    code = Column(String, unique=True, nullable=False, index=True)
    name = Column(String, nullable=False)
    description = Column(Text)
    credits = Column(Integer, nullable=False)
    max_capacity = Column(Integer, default=30)
    prerequisites = Column(ARRAY(String), default=[])
    department = Column(String, nullable=False)
    professor_id = Column(String, ForeignKey("professors.id"))

    # Relationships
    enrollments = relationship("Enrollment", back_populates="course", cascade="all, delete-orphan")
    students = relationship("Student", secondary="enrollments", back_populates="courses")
    professor = relationship("Professor", back_populates="courses")

    def __repr__(self):
        return f"<Course {self.code}: {self.name}>"
```

---

## Layer Communication

### The Dependency Rule

**Rule**: Outer layers depend on inner layers, never the reverse

```
┌────────────────────────────────────────┐
│     API Layer (Controllers)            │  ← Knows about Services
├────────────────────────────────────────┤
│     Service Layer (Business Logic)     │  ← Knows about Repositories
├────────────────────────────────────────┤
│     Repository Layer (Data Access)     │  ← Knows about Models
├────────────────────────────────────────┤
│     Database Layer (Models/ORM)        │  ← Knows nothing above it
└────────────────────────────────────────┘
```

**Good** ✅:
```javascript
// Controller uses Service
class StudentController {
    constructor(studentService) {
        this.studentService = studentService; // ✅ Depends on layer below
    }
}

// Service uses Repository
class StudentService {
    constructor(studentRepository) {
        this.studentRepository = studentRepository; // ✅ Depends on layer below
    }
}
```

**Bad** ❌:
```javascript
// Service depends on Controller - WRONG!
class StudentService {
    constructor(studentController) { // ❌ Wrong direction!
        this.controller = studentController;
    }
}
```

### Dependency Injection

Pass dependencies instead of creating them inside:

**Bad** ❌:
```javascript
class StudentController {
    constructor() {
        this.service = new StudentService(); // ❌ Tightly coupled
    }
}
```

**Good** ✅:
```javascript
class StudentController {
    constructor(studentService) {
        this.service = studentService; // ✅ Dependency injected
    }
}

// Usage
const studentRepo = new StudentRepository(db);
const studentService = new StudentService(studentRepo);
const studentController = new StudentController(studentService);
```

---

## Complete Example: Student Enrollment

Let's see all layers working together:

### Request Flow

```
1. HTTP Request
   POST /api/enrollments
   Body: { studentId: "123", courseId: "456" }

2. API Layer (Controller)
   - Receives request
   - Validates format
   - Calls service

3. Service Layer
   - Gets student from repository
   - Gets course from repository
   - Checks business rules
   - Creates enrollment via repository

4. Repository Layer
   - Executes database queries
   - Maps results to objects
   - Returns data

5. Database Layer
   - Stores/retrieves data

6. Response flows back up
   Repository → Service → Controller → HTTP Response
```

### Complete Code

**1. API Layer**:
```javascript
// controllers/enrollmentController.js
router.post('/enrollments', async (req, res) => {
    try {
        const { studentId, courseId } = req.body;

        const enrollment = await enrollmentService.enrollStudent(
            studentId,
            courseId
        );

        res.status(201).json({
            success: true,
            data: enrollment
        });
    } catch (error) {
        if (error.message.includes('not found')) {
            return res.status(404).json({ error: error.message });
        }
        if (error.message.includes('already enrolled') ||
            error.message.includes('prerequisites') ||
            error.message.includes('capacity')) {
            return res.status(400).json({ error: error.message });
        }
        res.status(500).json({ error: 'Internal server error' });
    }
});
```

**2. Service Layer**:
```javascript
// services/enrollmentService.js
async enrollStudent(studentId, courseId) {
    // Get entities
    const student = await this.studentRepo.findById(studentId);
    const course = await this.courseRepo.findById(courseId);

    if (!student) throw new Error('Student not found');
    if (!course) throw new Error('Course not found');

    // Business validations
    const validation = await this.validateEnrollment(student, course);
    if (!validation.valid) {
        throw new Error(validation.error);
    }

    // Create enrollment
    return await this.enrollmentRepo.create({
        studentId,
        courseId,
        semester: this.getCurrentSemester(),
        status: 'enrolled'
    });
}

async validateEnrollment(student, course) {
    // Check if already enrolled
    const existing = await this.enrollmentRepo.findByStudentAndCourse(
        student.id,
        course.id
    );
    if (existing) {
        return { valid: false, error: 'Student already enrolled in this course' };
    }

    // Check prerequisites
    if (!this.hasPrerequisites(student, course)) {
        return { valid: false, error: 'Prerequisites not met' };
    }

    // Check capacity
    const enrolledCount = await this.enrollmentRepo.countByCourse(course.id);
    if (enrolledCount >= course.maxCapacity) {
        return { valid: false, error: 'Course is at maximum capacity' };
    }

    return { valid: true };
}
```

**3. Repository Layer**:
```javascript
// repositories/enrollmentRepository.js
async create(enrollmentData) {
    return await this.db.Enrollment.create(enrollmentData);
}

async findByStudentAndCourse(studentId, courseId) {
    return await this.db.Enrollment.findOne({
        where: { studentId, courseId }
    });
}

async countByCourse(courseId) {
    return await this.db.Enrollment.count({
        where: { courseId, status: 'enrolled' }
    });
}
```

**4. Database Layer**:
```javascript
// models/Enrollment.js
const Enrollment = sequelize.define('Enrollment', {
    id: {
        type: DataTypes.UUID,
        defaultValue: DataTypes.UUIDV4,
        primaryKey: true
    },
    studentId: {
        type: DataTypes.UUID,
        allowNull: false,
        field: 'student_id'
    },
    courseId: {
        type: DataTypes.UUID,
        allowNull: false,
        field: 'course_id'
    },
    semester: {
        type: DataTypes.STRING,
        allowNull: false
    },
    grade: {
        type: DataTypes.STRING
    },
    status: {
        type: DataTypes.ENUM('enrolled', 'dropped', 'completed'),
        defaultValue: 'enrolled'
    }
});
```

---

## Key Takeaways

1. **Separation of Concerns**: Each layer has ONE responsibility
2. **Decoupling**: Layers are independent, changes isolated
3. **Testability**: Easy to test each layer separately
4. **Maintainability**: Clear structure, easy to find code
5. **Scalability**: Can replace/upgrade layers independently
6. **Dependency Rule**: Outer layers depend on inner, not reverse

---

## Next Steps

Now that you understand the architecture, let's dive deep into each layer:

**Next**: [03 - API Layer Deep Dive →](03_API_Layer.md)

---

**Previous**: [← 01 - Introduction](01_Introduction.md) | **Next**: [03 - API Layer →](03_API_Layer.md)

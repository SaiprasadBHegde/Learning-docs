# Part 5: Data Access Layer (Repository Pattern)

**Author**: Saiprasad Hegde

---

## Table of Contents
1. [What is Data Access Layer?](#what-is-data-access-layer)
2. [Repository Pattern](#repository-pattern)
3. [Implementing Repositories](#implementing-repositories)
4. [Query Patterns](#query-patterns)
5. [Data Mapping](#data-mapping)
6. [Repository Best Practices](#repository-best-practices)

---

## What is Data Access Layer?

**Data Access Layer (DAL)** abstracts all database operations, providing a clean interface for the service layer.

### Analogy: Library Catalog System

```
Service Layer: "I need a book about JavaScript"
      ↓
Repository: "Let me search the catalog and retrieve it for you"
      ↓
Database: "Here's the book from shelf 23.4B"
```

### Why DAL Matters

**Without DAL** ❌:
```javascript
// Service directly using database
async function enrollStudent(studentId, courseId) {
    const student = await db.query(
        "SELECT * FROM students WHERE id = $1",
        [studentId]
    );

    const course = await db.query(
        "SELECT * FROM courses WHERE id = $1",
        [courseId]
    );

    // Business logic mixed with SQL
}
```

**Problems**:
- SQL scattered everywhere
- Hard to change database
- Can't test without real database
- Violates Single Responsibility Principle

**With DAL** ✅:
```javascript
// Service using repository
async function enrollStudent(studentId, courseId) {
    const student = await this.studentRepo.findById(studentId);
    const course = await this.courseRepo.findById(courseId);

    // Clean, testable, maintainable
}
```

---

## Repository Pattern

### What is Repository Pattern?

**Repository Pattern** treats data access like a collection in memory.

**Key Idea**: Your service shouldn't know whether data comes from PostgreSQL, MongoDB, or an API - it just asks the repository.

### Interface

```javascript
// Repository interface (what service layer sees)
interface IStudentRepository {
    findAll(): Promise<Student[]>
    findById(id: string): Promise<Student | null>
    create(data: StudentData): Promise<Student>
    update(id: string, data: Partial<StudentData>): Promise<Student | null>
    delete(id: string): Promise<boolean>

    // Domain-specific methods
    findByEmail(email: string): Promise<Student | null>
    findByDepartment(dept: string): Promise<Student[]>
    getCompletedCourses(studentId: string): Promise<Course[]>
}
```

### Benefits

1. **Abstraction**: Service doesn't know about database
2. **Testability**: Easy to mock for testing
3. **Flexibility**: Switch databases without changing services
4. **Centralized**: All queries in one place
5. **Reusability**: Same repository used by multiple services

---

## Implementing Repositories

### Basic Repository Structure

**repositories/studentRepository.js**:
```javascript
const { Student, Enrollment, Course } = require('../models');
const { Op } = require('sequelize');

class StudentRepository {
    // ========== CRUD OPERATIONS ==========

    /**
     * Get all students with pagination and filtering
     */
    async findAll(options = {}) {
        const {
            page = 1,
            limit = 20,
            department,
            year,
            search,
            status = 'active'
        } = options;

        const offset = (page - 1) * limit;

        // Build query conditions
        const where = { status };

        if (department) {
            where.department = department;
        }

        if (year) {
            where.enrollmentYear = year;
        }

        if (search) {
            where[Op.or] = [
                { firstName: { [Op.iLike]: `%${search}%` } },
                { lastName: { [Op.iLike]: `%${search}%` } },
                { email: { [Op.iLike]: `%${search}%` } }
            ];
        }

        // Execute query
        const { rows: students, count: total } = await Student.findAndCountAll({
            where,
            limit,
            offset,
            order: [['lastName', 'ASC'], ['firstName', 'ASC']],
            attributes: {
                exclude: ['passwordHash'] // Don't return sensitive data
            }
        });

        return {
            students,
            total,
            page,
            pages: Math.ceil(total / limit)
        };
    }

    /**
     * Find student by ID
     */
    async findById(id) {
        return await Student.findByPk(id, {
            attributes: { exclude: ['passwordHash'] },
            include: [
                {
                    model: Enrollment,
                    as: 'enrollments',
                    include: [
                        {
                            model: Course,
                            as: 'course'
                        }
                    ]
                }
            ]
        });
    }

    /**
     * Create new student
     */
    async create(studentData) {
        return await Student.create(studentData);
    }

    /**
     * Update student
     */
    async update(id, updates) {
        const student = await this.findById(id);

        if (!student) {
            return null;
        }

        return await student.update(updates);
    }

    /**
     * Delete student
     */
    async delete(id) {
        const student = await this.findById(id);

        if (!student) {
            return false;
        }

        await student.destroy();
        return true;
    }

    // ========== CUSTOM QUERIES ==========

    /**
     * Find student by email
     */
    async findByEmail(email) {
        return await Student.findOne({
            where: { email },
            attributes: { exclude: ['passwordHash'] }
        });
    }

    /**
     * Find students by department
     */
    async findByDepartment(department) {
        return await Student.findAll({
            where: { department, status: 'active' },
            order: [['lastName', 'ASC']]
        });
    }

    /**
     * Get student's completed courses
     */
    async getCompletedCourses(studentId) {
        const enrollments = await Enrollment.findAll({
            where: {
                studentId,
                status: 'completed'
            },
            include: [
                {
                    model: Course,
                    as: 'course'
                }
            ],
            order: [['completedAt', 'DESC']]
        });

        return enrollments.map(e => ({
            ...e.course.toJSON(),
            grade: e.grade,
            semester: e.semester,
            completedAt: e.completedAt
        }));
    }

    /**
     * Calculate student's GPA
     */
    async calculateGPA(studentId) {
        const enrollments = await Enrollment.findAll({
            where: {
                studentId,
                status: 'completed',
                grade: { [Op.ne]: null }
            },
            include: [
                {
                    model: Course,
                    as: 'course',
                    attributes: ['credits']
                }
            ]
        });

        if (enrollments.length === 0) return 0;

        const gradePoints = {
            'A': 4.0, 'A-': 3.7,
            'B+': 3.3, 'B': 3.0, 'B-': 2.7,
            'C+': 2.3, 'C': 2.0, 'C-': 1.7,
            'D+': 1.3, 'D': 1.0,
            'F': 0.0
        };

        let totalPoints = 0;
        let totalCredits = 0;

        for (const enrollment of enrollments) {
            const points = gradePoints[enrollment.grade] || 0;
            const credits = enrollment.course.credits;

            totalPoints += points * credits;
            totalCredits += credits;
        }

        return totalCredits > 0
            ? parseFloat((totalPoints / totalCredits).toFixed(2))
            : 0;
    }

    /**
     * Get students with GPA above threshold
     */
    async findDeansListStudents(minGPA = 3.5) {
        // This is complex - would need raw SQL or compute in application
        const students = await this.findAll({ status: 'active' });

        const results = [];
        for (const student of students.students) {
            const gpa = await this.calculateGPA(student.id);
            if (gpa >= minGPA) {
                results.push({ ...student.toJSON(), gpa });
            }
        }

        return results.sort((a, b) => b.gpa - a.gpa);
    }

    /**
     * Check if email exists
     */
    async emailExists(email, excludeId = null) {
        const where = { email };

        if (excludeId) {
            where.id = { [Op.ne]: excludeId };
        }

        const count = await Student.count({ where });
        return count > 0;
    }

    /**
     * Get student count by department
     */
    async getCountByDepartment() {
        return await Student.findAll({
            attributes: [
                'department',
                [sequelize.fn('COUNT', sequelize.col('id')), 'count']
            ],
            where: { status: 'active' },
            group: ['department'],
            order: [[sequelize.literal('count'), 'DESC']]
        });
    }
}

module.exports = StudentRepository;
```

---

## Query Patterns

### 1. Simple Queries

```javascript
// Find by primary key
async findById(id) {
    return await Student.findByPk(id);
}

// Find one by condition
async findByEmail(email) {
    return await Student.findOne({ where: { email } });
}

// Find all by condition
async findByDepartment(department) {
    return await Student.findAll({ where: { department } });
}

// Count
async countActive() {
    return await Student.count({ where: { status: 'active' } });
}
```

### 2. Complex Queries with Joins

```javascript
async getStudentWithEnrollments(studentId) {
    return await Student.findByPk(studentId, {
        include: [
            {
                model: Enrollment,
                as: 'enrollments',
                include: [
                    {
                        model: Course,
                        as: 'course',
                        include: [
                            {
                                model: Professor,
                                as: 'professor'
                            }
                        ]
                    }
                ],
                where: { status: 'enrolled' },
                required: false // LEFT JOIN
            }
        ]
    });
}
```

### 3. Aggregation Queries

```javascript
async getEnrollmentStats() {
    return await Enrollment.findAll({
        attributes: [
            'semester',
            [sequelize.fn('COUNT', sequelize.col('id')), 'totalEnrollments'],
            [sequelize.fn('AVG', sequelize.col('grade')), 'averageGrade']
        ],
        group: ['semester'],
        order: [['semester', 'DESC']]
    });
}
```

### 4. Raw SQL (When Needed)

```javascript
async getComplexReport() {
    const [results] = await sequelize.query(`
        SELECT
            s.department,
            COUNT(DISTINCT s.id) as student_count,
            COUNT(DISTINCT e.id) as enrollment_count,
            AVG(CASE
                WHEN e.grade = 'A' THEN 4.0
                WHEN e.grade = 'B' THEN 3.0
                ELSE 2.0
            END) as avg_gpa
        FROM students s
        LEFT JOIN enrollments e ON s.id = e.student_id
        WHERE s.status = 'active'
        GROUP BY s.department
        ORDER BY avg_gpa DESC
    `);

    return results;
}
```

### 5. Pagination Pattern

```javascript
async findAllPaginated(page = 1, limit = 20, filters = {}) {
    const offset = (page - 1) * limit;

    const { rows, count } = await Student.findAndCountAll({
        where: filters,
        limit,
        offset,
        order: [['createdAt', 'DESC']]
    });

    return {
        data: rows,
        pagination: {
            page,
            limit,
            total: count,
            pages: Math.ceil(count / limit)
        }
    };
}
```

---

## Data Mapping

### ORM to Domain Objects

**Problem**: ORM models have database-specific properties we don't want in our application.

**Solution**: Map ORM objects to clean domain objects.

```javascript
// models/Student.js (ORM model)
class Student extends Model {
    id: string;
    firstName: string;
    lastName: string;
    email: string;
    passwordHash: string; // ❌ Don't expose!
    createdAt: Date;
    updatedAt: Date;
}

// domain/Student.js (Domain model)
class StudentDTO {
    constructor(ormStudent) {
        this.id = ormStudent.id;
        this.firstName = ormStudent.firstName;
        this.lastName = ormStudent.lastName;
        this.fullName = `${ormStudent.firstName} ${ormStudent.lastName}`;
        this.email = ormStudent.email;
        this.department = ormStudent.department;
        // passwordHash intentionally excluded
    }

    toJSON() {
        return {
            id: this.id,
            firstName: this.firstName,
            lastName: this.lastName,
            fullName: this.fullName,
            email: this.email,
            department: this.department
        };
    }
}

// In repository
async findById(id) {
    const student = await Student.findByPk(id);
    return student ? new StudentDTO(student) : null;
}
```

### Projection (Select Specific Fields)

```javascript
// Only get fields you need
async getStudentNames() {
    return await Student.findAll({
        attributes: ['id', 'firstName', 'lastName']
    });
}

// Exclude sensitive fields
async findAllPublic() {
    return await Student.findAll({
        attributes: {
            exclude: ['passwordHash', 'ssn']
        }
    });
}

// Computed fields
async getStudentsWithAge() {
    return await Student.findAll({
        attributes: [
            'id',
            'firstName',
            'lastName',
            [
                sequelize.fn(
                    'DATE_PART',
                    'year',
                    sequelize.fn('AGE', sequelize.col('date_of_birth'))
                ),
                'age'
            ]
        ]
    });
}
```

---

## Repository Best Practices

### 1. Keep Repositories Focused

**Good** ✅:
```javascript
class StudentRepository {
    // Student-specific operations only
    async findById(id) { }
    async findByEmail(email) { }
    async getCompletedCourses(studentId) { }
}

class CourseRepository {
    // Course-specific operations only
    async findById(id) { }
    async findByCode(code) { }
    async getEnrolledStudents(courseId) { }
}
```

**Bad** ❌:
```javascript
class Repository {
    // ❌ Too many responsibilities
    async findStudent(id) { }
    async findCourse(id) { }
    async enrollStudentInCourse() { }
    async calculateGrades() { }
}
```

### 2. Return Domain Objects, Not ORM Objects

```javascript
// ❌ Bad - exposes ORM implementation
async findById(id) {
    return await Student.findByPk(id); // Sequelize model
}

// ✅ Good - returns plain object
async findById(id) {
    const student = await Student.findByPk(id);
    return student ? student.toJSON() : null;
}
```

### 3. Use Descriptive Method Names

```javascript
// ✅ Clear intent
async findActiveStudentsByDepartment(department) { }
async getStudentsEnrolledInCourse(courseId) { }
async countStudentsOnDeansListThisSemester() { }

// ❌ Unclear
async getStudents(dept) { }
async find(id, opts) { }
```

### 4. Handle Not Found Cases Consistently

```javascript
// Option 1: Return null
async findById(id) {
    const student = await Student.findByPk(id);
    return student || null;
}

// Option 2: Throw error
async findByIdOrFail(id) {
    const student = await Student.findByPk(id);
    if (!student) {
        throw new NotFoundError('Student not found');
    }
    return student;
}

// Use consistently across all repositories
```

### 5. Don't Put Business Logic in Repositories

```javascript
// ❌ Bad - business logic in repository
async enrollStudentInCourse(studentId, courseId) {
    const student = await this.findById(studentId);

    // ❌ This is business logic, not data access!
    if (student.creditsThisSemester >= 18) {
        throw new Error('Exceeds credit limit');
    }

    // Create enrollment...
}

// ✅ Good - just data access
async create(enrollmentData) {
    return await Enrollment.create(enrollmentData);
}

// Business logic belongs in service layer!
```

### 6. Use Transactions for Multi-Step Operations

```javascript
async transferStudentDepartment(studentId, newDepartment, transaction) {
    // Accept transaction from service layer
    await Student.update(
        { department: newDepartment },
        { where: { id: studentId }, transaction }
    );

    // Log history
    await DepartmentChangeLog.create(
        { studentId, newDepartment, changedAt: new Date() },
        { transaction }
    );
}
```

### 7. Create Reusable Query Builders

```javascript
class StudentRepository {
    _baseQuery() {
        return Student.findAll({
            attributes: { exclude: ['passwordHash'] },
            where: { status: 'active' }
        });
    }

    async findByDepartment(department) {
        return await this._baseQuery()
            .where({ department });
    }

    async findEnrolledThisSemester(semester) {
        return await this._baseQuery()
            .include([{
                model: Enrollment,
                where: { semester }
            }]);
    }
}
```

---

## Key Takeaways

1. **Repository Pattern** abstracts database operations
2. **One repository per entity** (Student, Course, Enrollment)
3. **Return domain objects**, not ORM models
4. **Keep it simple** - just data access, no business logic
5. **Descriptive names** - clear intent
6. **Handle transactions** properly
7. **Easy to test** - mock repositories in tests

---

**Next**: [06 - Database Layer and ORMs →](06_Database_Layer.md)

---

**Previous**: [← 04 - Business Logic Layer](04_Business_Logic_Layer.md) | **Next**: [06 - Database Layer →](06_Database_Layer.md)

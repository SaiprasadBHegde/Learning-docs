# Part 6: Database Layer and ORMs

**Author**: Saiprasad Hegde

---

## Table of Contents
1. [Database Design Fundamentals](#database-design-fundamentals)
2. [What is an ORM?](#what-is-an-orm)
3. [Sequelize (Node.js)](#sequelize-nodejs)
4. [Drizzle ORM (Node.js)](#drizzle-orm-nodejs)
5. [Migrations](#migrations)
6. [Database Relationships](#database-relationships)

---

## Database Design Fundamentals

### Schema Design for College System

```sql
-- Students Table
CREATE TABLE students (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    date_of_birth DATE,
    department VARCHAR(100) NOT NULL,
    enrollment_year INTEGER NOT NULL,
    status VARCHAR(20) DEFAULT 'active',
    gpa DECIMAL(3,2) DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Courses Table
CREATE TABLE courses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    credits INTEGER NOT NULL CHECK (credits BETWEEN 1 AND 6),
    max_capacity INTEGER DEFAULT 30,
    department VARCHAR(100) NOT NULL,
    professor_id UUID REFERENCES professors(id),
    prerequisites UUID[],
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Enrollments Table (Junction)
CREATE TABLE enrollments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID REFERENCES students(id) ON DELETE CASCADE,
    course_id UUID REFERENCES courses(id) ON DELETE CASCADE,
    semester VARCHAR(50) NOT NULL,
    status VARCHAR(20) DEFAULT 'enrolled',
    grade VARCHAR(2),
    enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP,
    dropped_at TIMESTAMP,
    UNIQUE(student_id, course_id, semester)
);

-- Professors Table
CREATE TABLE professors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    department VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for performance
CREATE INDEX idx_students_email ON students(email);
CREATE INDEX idx_students_department ON students(department);
CREATE INDEX idx_students_status ON students(status);
CREATE INDEX idx_courses_code ON courses(code);
CREATE INDEX idx_courses_department ON courses(department);
CREATE INDEX idx_enrollments_student ON enrollments(student_id);
CREATE INDEX idx_enrollments_course ON enrollments(course_id);
CREATE INDEX idx_enrollments_semester ON enrollments(semester);
```

### Database Normalization

**Normalization**: Organizing data to reduce redundancy.

**1NF (First Normal Form)**: No repeating groups
```javascript
// ❌ Bad
student = {
    id: 1,
    name: "John",
    courses: "CS101,CS102,CS103" // Multiple values in one field!
}

// ✅ Good - separate enrollments table
student = { id: 1, name: "John" }
enrollments = [
    { student_id: 1, course_id: "CS101" },
    { student_id: 1, course_id: "CS102" }
]
```

**2NF (Second Normal Form)**: No partial dependencies
**3NF (Third Normal Form)**: No transitive dependencies

---

## What is an ORM?

**ORM (Object-Relational Mapping)** bridges the gap between objects (code) and tables (database).

### Analogy: Translator

```
Your Code (JavaScript/Python) ↔ ORM ↔ Database (SQL)
     "student.save()"           →    "INSERT INTO students..."
```

### Benefits of ORMs

✅ **Write less SQL**
✅ **Database-agnostic** (switch from PostgreSQL to MySQL easily)
✅ **Type safety** (especially with TypeScript/Drizzle)
✅ **Migrations** handled automatically
✅ **Relationships** easy to define

### Drawbacks

❌ **Performance overhead** (for complex queries)
❌ **Learning curve**
❌ **Less control** over queries
❌ **N+1 query problem** if not careful

---

## Sequelize (Node.js)

### Setup

```bash
npm install sequelize pg pg-hstore
npm install --save-dev sequelize-cli
```

**config/database.js**:
```javascript
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize(
    process.env.DB_NAME || 'college_db',
    process.env.DB_USER || 'postgres',
    process.env.DB_PASS || 'password',
    {
        host: process.env.DB_HOST || 'localhost',
        port: process.env.DB_PORT || 5432,
        dialect: 'postgres',
        logging: process.env.NODE_ENV === 'development' ? console.log : false,
        pool: {
            max: 5,
            min: 0,
            acquire: 30000,
            idle: 10000
        }
    }
);

module.exports = sequelize;
```

### Defining Models

**models/Student.js**:
```javascript
const { DataTypes, Model } = require('sequelize');
const sequelize = require('../config/database');

class Student extends Model {
    // Instance methods
    get fullName() {
        return `${this.firstName} ${this.lastName}`;
    }

    async getEnrollments() {
        return await this.getEnrollments({
            where: { status: 'enrolled' }
        });
    }

    // Class methods
    static async findActiveStudents() {
        return await this.findAll({
            where: { status: 'active' }
        });
    }
}

Student.init({
    id: {
        type: DataTypes.UUID,
        defaultValue: DataTypes.UUIDV4,
        primaryKey: true
    },
    firstName: {
        type: DataTypes.STRING(50),
        allowNull: false,
        field: 'first_name',
        validate: {
            notEmpty: true,
            len: [1, 50]
        }
    },
    lastName: {
        type: DataTypes.STRING(50),
        allowNull: false,
        field: 'last_name',
        validate: {
            notEmpty: true,
            len: [1, 50]
        }
    },
    email: {
        type: DataTypes.STRING(255),
        allowNull: false,
        unique: true,
        validate: {
            isEmail: true
        }
    },
    dateOfBirth: {
        type: DataTypes.DATEONLY,
        field: 'date_of_birth'
    },
    department: {
        type: DataTypes.STRING(100),
        allowNull: false
    },
    enrollmentYear: {
        type: DataTypes.INTEGER,
        allowNull: false,
        field: 'enrollment_year',
        validate: {
            min: 2000,
            max: 2100
        }
    },
    status: {
        type: DataTypes.ENUM('active', 'inactive', 'graduated', 'suspended'),
        defaultValue: 'active'
    },
    gpa: {
        type: DataTypes.DECIMAL(3, 2),
        defaultValue: 0.00,
        validate: {
            min: 0.00,
            max: 4.00
        }
    }
}, {
    sequelize,
    modelName: 'Student',
    tableName: 'students',
    timestamps: true,
    underscored: true,
    indexes: [
        { fields: ['email'] },
        { fields: ['department'] },
        { fields: ['status'] }
    ]
});

module.exports = Student;
```

**models/Course.js**:
```javascript
class Course extends Model {}

Course.init({
    id: {
        type: DataTypes.UUID,
        defaultValue: DataTypes.UUIDV4,
        primaryKey: true
    },
    code: {
        type: DataTypes.STRING(20),
        allowNull: false,
        unique: true,
        validate: {
            is: /^[A-Z]{2,4}\d{3}$/  // Like CS101, MATH201
        }
    },
    name: {
        type: DataTypes.STRING(200),
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
        defaultValue: 30,
        field: 'max_capacity'
    },
    department: {
        type: DataTypes.STRING(100),
        allowNull: false
    },
    professorId: {
        type: DataTypes.UUID,
        field: 'professor_id',
        references: {
            model: 'professors',
            key: 'id'
        }
    },
    prerequisites: {
        type: DataTypes.ARRAY(DataTypes.UUID),
        defaultValue: []
    }
}, {
    sequelize,
    modelName: 'Course',
    tableName: 'courses',
    timestamps: true,
    underscored: true
});

module.exports = Course;
```

**models/Enrollment.js**:
```javascript
class Enrollment extends Model {}

Enrollment.init({
    id: {
        type: DataTypes.UUID,
        defaultValue: DataTypes.UUIDV4,
        primaryKey: true
    },
    studentId: {
        type: DataTypes.UUID,
        allowNull: false,
        field: 'student_id',
        references: {
            model: 'students',
            key: 'id'
        },
        onDelete: 'CASCADE'
    },
    courseId: {
        type: DataTypes.UUID,
        allowNull: false,
        field: 'course_id',
        references: {
            model: 'courses',
            key: 'id'
        },
        onDelete: 'CASCADE'
    },
    semester: {
        type: DataTypes.STRING(50),
        allowNull: false
    },
    status: {
        type: DataTypes.ENUM('enrolled', 'dropped', 'completed'),
        defaultValue: 'enrolled'
    },
    grade: {
        type: DataTypes.STRING(2),
        validate: {
            isIn: [['A', 'A-', 'B+', 'B', 'B-', 'C+', 'C', 'C-', 'D+', 'D', 'F']]
        }
    },
    enrolledAt: {
        type: DataTypes.DATE,
        defaultValue: DataTypes.NOW,
        field: 'enrolled_at'
    },
    completedAt: {
        type: DataTypes.DATE,
        field: 'completed_at'
    },
    droppedAt: {
        type: DataTypes.DATE,
        field: 'dropped_at'
    }
}, {
    sequelize,
    modelName: 'Enrollment',
    tableName: 'enrollments',
    timestamps: false,
    indexes: [
        {
            unique: true,
            fields: ['student_id', 'course_id', 'semester']
        }
    ]
});

module.exports = Enrollment;
```

### Associations

**models/index.js**:
```javascript
const Student = require('./Student');
const Course = require('./Course');
const Enrollment = require('./Enrollment');
const Professor = require('./Professor');

// Student <-> Enrollment <-> Course (Many-to-Many)
Student.hasMany(Enrollment, {
    foreignKey: 'student_id',
    as: 'enrollments'
});

Enrollment.belongsTo(Student, {
    foreignKey: 'student_id',
    as: 'student'
});

Course.hasMany(Enrollment, {
    foreignKey: 'course_id',
    as: 'enrollments'
});

Enrollment.belongsTo(Course, {
    foreignKey: 'course_id',
    as: 'course'
});

// Student <-> Course (through Enrollment)
Student.belongsToMany(Course, {
    through: Enrollment,
    foreignKey: 'student_id',
    otherKey: 'course_id',
    as: 'courses'
});

Course.belongsToMany(Student, {
    through: Enrollment,
    foreignKey: 'course_id',
    otherKey: 'student_id',
    as: 'students'
});

// Course -> Professor (Many-to-One)
Course.belongsTo(Professor, {
    foreignKey: 'professor_id',
    as: 'professor'
});

Professor.hasMany(Course, {
    foreignKey: 'professor_id',
    as: 'courses'
});

module.exports = {
    Student,
    Course,
    Enrollment,
    Professor
};
```

---

## Drizzle ORM (Node.js)

### Why Drizzle?

- **TypeScript-first**: Full type safety
- **SQL-like**: Feels like writing SQL
- **Lightweight**: Smaller bundle size
- **Performance**: Faster than Sequelize
- **Zero dependencies**

### Setup

```bash
npm install drizzle-orm pg
npm install -D drizzle-kit
```

**drizzle.config.ts**:
```typescript
import type { Config } from 'drizzle-kit';

export default {
    schema: './src/db/schema.ts',
    out: './drizzle',
    driver: 'pg',
    dbCredentials: {
        connectionString: process.env.DATABASE_URL!
    }
} satisfies Config;
```

### Schema Definition

**db/schema.ts**:
```typescript
import {
    pgTable,
    uuid,
    varchar,
    integer,
    text,
    date,
    timestamp,
    decimal,
    pgEnum,
    unique
} from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// Enums
export const studentStatusEnum = pgEnum('student_status', [
    'active',
    'inactive',
    'graduated',
    'suspended'
]);

export const enrollmentStatusEnum = pgEnum('enrollment_status', [
    'enrolled',
    'dropped',
    'completed'
]);

// Students Table
export const students = pgTable('students', {
    id: uuid('id').defaultRandom().primaryKey(),
    firstName: varchar('first_name', { length: 50 }).notNull(),
    lastName: varchar('last_name', { length: 50 }).notNull(),
    email: varchar('email', { length: 255 }).notNull().unique(),
    dateOfBirth: date('date_of_birth'),
    department: varchar('department', { length: 100 }).notNull(),
    enrollmentYear: integer('enrollment_year').notNull(),
    status: studentStatusEnum('status').default('active'),
    gpa: decimal('gpa', { precision: 3, scale: 2 }).default('0.00'),
    createdAt: timestamp('created_at').defaultNow(),
    updatedAt: timestamp('updated_at').defaultNow()
}, (table) => ({
    emailIdx: unique().on(table.email),
}));

// Courses Table
export const courses = pgTable('courses', {
    id: uuid('id').defaultRandom().primaryKey(),
    code: varchar('code', { length: 20 }).notNull().unique(),
    name: varchar('name', { length: 200 }).notNull(),
    description: text('description'),
    credits: integer('credits').notNull(),
    maxCapacity: integer('max_capacity').default(30),
    department: varchar('department', { length: 100 }).notNull(),
    professorId: uuid('professor_id').references(() => professors.id),
    prerequisites: uuid('prerequisites').array(),
    createdAt: timestamp('created_at').defaultNow(),
    updatedAt: timestamp('updated_at').defaultNow()
});

// Enrollments Table
export const enrollments = pgTable('enrollments', {
    id: uuid('id').defaultRandom().primaryKey(),
    studentId: uuid('student_id')
        .notNull()
        .references(() => students.id, { onDelete: 'cascade' }),
    courseId: uuid('course_id')
        .notNull()
        .references(() => courses.id, { onDelete: 'cascade' }),
    semester: varchar('semester', { length: 50 }).notNull(),
    status: enrollmentStatusEnum('status').default('enrolled'),
    grade: varchar('grade', { length: 2 }),
    enrolledAt: timestamp('enrolled_at').defaultNow(),
    completedAt: timestamp('completed_at'),
    droppedAt: timestamp('dropped_at')
}, (table) => ({
    uniqueEnrollment: unique().on(table.studentId, table.courseId, table.semester)
}));

// Professors Table
export const professors = pgTable('professors', {
    id: uuid('id').defaultRandom().primaryKey(),
    name: varchar('name', { length: 100 }).notNull(),
    email: varchar('email', { length: 255 }).notNull().unique(),
    department: varchar('department', { length: 100 }).notNull(),
    createdAt: timestamp('created_at').defaultNow()
});

// Relations
export const studentsRelations = relations(students, ({ many }) => ({
    enrollments: many(enrollments)
}));

export const coursesRelations = relations(courses, ({ one, many }) => ({
    professor: one(professors, {
        fields: [courses.professorId],
        references: [professors.id]
    }),
    enrollments: many(enrollments)
}));

export const enrollmentsRelations = relations(enrollments, ({ one }) => ({
    student: one(students, {
        fields: [enrollments.studentId],
        references: [students.id]
    }),
    course: one(courses, {
        fields: [enrollments.courseId],
        references: [courses.id]
    })
}));

export const professorsRelations = relations(professors, ({ many }) => ({
    courses: many(courses)
}));

// Type exports
export type Student = typeof students.$inferSelect;
export type NewStudent = typeof students.$inferInsert;
export type Course = typeof courses.$inferSelect;
export type NewCourse = typeof courses.$inferInsert;
export type Enrollment = typeof enrollments.$inferSelect;
export type NewEnrollment = typeof enrollments.$inferInsert;
```

### Database Connection

**db/index.ts**:
```typescript
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import * as schema from './schema';

const pool = new Pool({
    connectionString: process.env.DATABASE_URL
});

export const db = drizzle(pool, { schema });
```

### Querying with Drizzle

**repositories/studentRepository.ts**:
```typescript
import { eq, and, or, like, desc, sql } from 'drizzle-orm';
import { db } from '../db';
import { students, enrollments, courses } from '../db/schema';
import type { Student, NewStudent } from '../db/schema';

export class StudentRepository {
    // Find all students
    async findAll(filters?: {
        department?: string;
        status?: string;
        search?: string;
    }) {
        let query = db.select().from(students);

        if (filters?.department) {
            query = query.where(eq(students.department, filters.department));
        }

        if (filters?.status) {
            query = query.where(eq(students.status, filters.status as any));
        }

        if (filters?.search) {
            query = query.where(
                or(
                    like(students.firstName, `%${filters.search}%`),
                    like(students.lastName, `%${filters.search}%`),
                    like(students.email, `%${filters.search}%`)
                )
            );
        }

        return await query;
    }

    // Find by ID with enrollments
    async findById(id: string) {
        const result = await db.query.students.findFirst({
            where: eq(students.id, id),
            with: {
                enrollments: {
                    with: {
                        course: true
                    }
                }
            }
        });

        return result;
    }

    // Find by email
    async findByEmail(email: string) {
        return await db.query.students.findFirst({
            where: eq(students.email, email)
        });
    }

    // Create student
    async create(data: NewStudent) {
        const [student] = await db
            .insert(students)
            .values(data)
            .returning();

        return student;
    }

    // Update student
    async update(id: string, data: Partial<NewStudent>) {
        const [student] = await db
            .update(students)
            .set({ ...data, updatedAt: new Date() })
            .where(eq(students.id, id))
            .returning();

        return student;
    }

    // Delete student
    async delete(id: string) {
        const [deleted] = await db
            .delete(students)
            .where(eq(students.id, id))
            .returning();

        return !!deleted;
    }

    // Get student's completed courses
    async getCompletedCourses(studentId: string) {
        return await db
            .select({
                id: courses.id,
                code: courses.code,
                name: courses.name,
                credits: courses.credits,
                grade: enrollments.grade,
                semester: enrollments.semester,
                completedAt: enrollments.completedAt
            })
            .from(enrollments)
            .innerJoin(courses, eq(enrollments.courseId, courses.id))
            .where(
                and(
                    eq(enrollments.studentId, studentId),
                    eq(enrollments.status, 'completed')
                )
            )
            .orderBy(desc(enrollments.completedAt));
    }

    // Calculate GPA
    async calculateGPA(studentId: string): Promise<number> {
        const gradePoints: Record<string, number> = {
            'A': 4.0, 'A-': 3.7,
            'B+': 3.3, 'B': 3.0, 'B-': 2.7,
            'C+': 2.3, 'C': 2.0, 'C-': 1.7,
            'D+': 1.3, 'D': 1.0,
            'F': 0.0
        };

        const completedCourses = await db
            .select({
                grade: enrollments.grade,
                credits: courses.credits
            })
            .from(enrollments)
            .innerJoin(courses, eq(enrollments.courseId, courses.id))
            .where(
                and(
                    eq(enrollments.studentId, studentId),
                    eq(enrollments.status, 'completed'),
                    sql`${enrollments.grade} IS NOT NULL`
                )
            );

        if (completedCourses.length === 0) return 0;

        let totalPoints = 0;
        let totalCredits = 0;

        for (const course of completedCourses) {
            const points = gradePoints[course.grade!] || 0;
            totalPoints += points * course.credits;
            totalCredits += course.credits;
        }

        return totalCredits > 0
            ? parseFloat((totalPoints / totalCredits).toFixed(2))
            : 0;
    }

    // Count by department
    async getCountByDepartment() {
        return await db
            .select({
                department: students.department,
                count: sql<number>`count(*)::int`
            })
            .from(students)
            .where(eq(students.status, 'active'))
            .groupBy(students.department)
            .orderBy(desc(sql`count(*)`));
    }
}
```

### Drizzle Advantages

```typescript
// ✅ Full type safety
const student = await db.query.students.findFirst({
    where: eq(students.id, id)
});

// student.firstName is typed as string
// student.unknownField // ❌ TypeScript error!

// ✅ SQL-like syntax
const results = await db
    .select()
    .from(students)
    .where(eq(students.department, 'CS'))
    .orderBy(desc(students.gpa))
    .limit(10);

// ✅ Join queries
const studentsWithCourses = await db
    .select()
    .from(students)
    .leftJoin(enrollments, eq(students.id, enrollments.studentId))
    .leftJoin(courses, eq(enrollments.courseId, courses.id));
```

---

## Migrations

### Why Migrations?

**Migrations** track database schema changes over time.

**Benefits**:
- Version control for database schema
- Easy rollback
- Consistent across environments
- Team collaboration

### Sequelize Migrations

```bash
npx sequelize-cli migration:generate --name create-students-table
```

**migrations/20250115-create-students.js**:
```javascript
module.exports = {
    up: async (queryInterface, Sequelize) => {
        await queryInterface.createTable('students', {
            id: {
                type: Sequelize.UUID,
                defaultValue: Sequelize.UUIDV4,
                primaryKey: true
            },
            first_name: {
                type: Sequelize.STRING(50),
                allowNull: false
            },
            last_name: {
                type: Sequelize.STRING(50),
                allowNull: false
            },
            email: {
                type: Sequelize.STRING(255),
                allowNull: false,
                unique: true
            },
            department: {
                type: Sequelize.STRING(100),
                allowNull: false
            },
            status: {
                type: Sequelize.ENUM('active', 'inactive', 'graduated', 'suspended'),
                defaultValue: 'active'
            },
            gpa: {
                type: Sequelize.DECIMAL(3, 2),
                defaultValue: 0.00
            },
            created_at: {
                type: Sequelize.DATE,
                allowNull: false
            },
            updated_at: {
                type: Sequelize.DATE,
                allowNull: false
            }
        });

        // Add indexes
        await queryInterface.addIndex('students', ['email']);
        await queryInterface.addIndex('students', ['department']);
    },

    down: async (queryInterface, Sequelize) => {
        await queryInterface.dropTable('students');
    }
};
```

```bash
# Run migrations
npx sequelize-cli db:migrate

# Rollback
npx sequelize-cli db:migrate:undo
```

### Drizzle Migrations

```bash
# Generate migration
npx drizzle-kit generate:pg

# Apply migration
npx drizzle-kit push:pg
```

---

## Database Relationships

### One-to-Many

**One Professor has Many Courses**

```javascript
// Sequelize
Professor.hasMany(Course, { foreignKey: 'professor_id' });
Course.belongsTo(Professor, { foreignKey: 'professor_id' });
```

```typescript
// Drizzle
export const professorsRelations = relations(professors, ({ many }) => ({
    courses: many(courses)
}));
```

### Many-to-Many

**Students and Courses (through Enrollments)**

```javascript
// Sequelize
Student.belongsToMany(Course, {
    through: Enrollment,
    foreignKey: 'student_id'
});

Course.belongsToMany(Student, {
    through: Enrollment,
    foreignKey: 'course_id'
});
```

---

## Key Takeaways

1. **ORMs** map objects to database tables
2. **Sequelize**: Mature, feature-rich Node.js ORM
3. **Drizzle**: Type-safe, modern, performant Node.js ORM
4. **Migrations** version control your schema
5. **Relationships** define how data connects

---

**Next**: [07 - Authentication and Authorization →](07_Authentication_Authorization.md)

---

**Previous**: [← 05 - Data Access Layer](05_Data_Access_Layer.md) | **Next**: [07 - Authentication →](07_Authentication_Authorization.md)

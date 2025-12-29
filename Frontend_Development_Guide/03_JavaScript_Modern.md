# Part 3: Modern JavaScript & ES6+

**Author: Saiprasad Hegde**

## Table of Contents
1. [Introduction to Modern JavaScript](#introduction)
2. [Arrow Functions](#arrow-functions)
3. [Template Literals](#template-literals)
4. [Destructuring](#destructuring)
5. [Spread and Rest Operators](#spread-rest-operators)
6. [Array Methods](#array-methods)
7. [Promises and Async/Await](#promises-async-await)
8. [Modules (Import/Export)](#modules)
9. [Optional Chaining and Nullish Coalescing](#optional-chaining)
10. [DOM Manipulation Basics](#dom-manipulation)
11. [Event Handling](#event-handling)
12. [Practical Examples: College Portal](#practical-examples)

---

## Introduction to Modern JavaScript {#introduction}

Modern JavaScript (ES6+) introduced features that make code more readable, maintainable, and powerful. Think of ES6+ as JavaScript getting a major upgrade - like going from a bicycle to a Tesla. The destination is the same, but the journey is much smoother.

### Why Modern JavaScript Matters

**Analogy**: Imagine you're writing letters. Old JavaScript is like writing with a typewriter - functional but tedious. Modern JavaScript is like having a word processor with autocomplete, templates, and shortcuts.

---

## Arrow Functions {#arrow-functions}

Arrow functions provide a shorter syntax for writing functions and handle the `this` keyword differently.

### Basic Syntax

```typescript
// Traditional function
function addTraditional(a: number, b: number): number {
  return a + b;
}

// Arrow function
const addArrow = (a: number, b: number): number => {
  return a + b;
};

// Concise arrow function (implicit return)
const addConcise = (a: number, b: number): number => a + b;

// Single parameter (parentheses optional)
const square = (n: number): number => n * n;

// No parameters
const getGreeting = (): string => "Hello, Students!";
```

### College Management System Examples

```typescript
// Student data type
interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
  enrolledCourses: string[];
}

// Calculate average GPA
const calculateAverageGPA = (students: Student[]): number => {
  const total = students.reduce((sum, student) => sum + student.gpa, 0);
  return total / students.length;
};

// Find student by ID
const findStudentById = (students: Student[], id: string): Student | undefined => {
  return students.find(student => student.id === id);
};

// Format student name
const formatStudentName = (student: Student): string =>
  `${student.name} (${student.email})`;

// Check if student is enrolled in course
const isEnrolled = (student: Student, courseId: string): boolean =>
  student.enrolledCourses.includes(courseId);
```

### When to Use Arrow Functions

**Use arrow functions for:**
- Short, simple functions
- Array methods (map, filter, reduce)
- Callbacks
- Functions that don't need their own `this`

**Use traditional functions for:**
- Methods in classes
- Functions that need `arguments` object
- Constructor functions

---

## Template Literals {#template-literals}

Template literals allow embedded expressions and multi-line strings using backticks.

### Basic Syntax

```typescript
// Old way (string concatenation)
const studentName = "Alice Johnson";
const courseCount = 5;
const message1 = "Student " + studentName + " is enrolled in " + courseCount + " courses.";

// Modern way (template literals)
const message2 = `Student ${studentName} is enrolled in ${courseCount} courses.`;

// Multi-line strings
const emailTemplate = `
Dear ${studentName},

Welcome to your college portal!
You are currently enrolled in ${courseCount} courses.

Best regards,
Administration
`;
```

### College Portal Examples

```typescript
interface Course {
  id: string;
  name: string;
  code: string;
  credits: number;
  instructor: string;
  schedule: {
    days: string[];
    time: string;
  };
}

// Generate course description
const getCourseDescription = (course: Course): string =>
  `${course.code}: ${course.name} (${course.credits} credits)`;

// Generate schedule text
const getScheduleText = (course: Course): string =>
  `${course.schedule.days.join(', ')} at ${course.schedule.time}`;

// Generate enrollment confirmation
const getEnrollmentConfirmation = (
  student: Student,
  course: Course
): string => `
Enrollment Confirmation
-----------------------
Student: ${student.name}
Email: ${student.email}
Course: ${course.name}
Code: ${course.code}
Schedule: ${getScheduleText(course)}
Instructor: ${course.instructor}

Total Enrolled Courses: ${student.enrolledCourses.length}
`;

// Generate transcript header
const getTranscriptHeader = (student: Student, semester: string): string => `
╔════════════════════════════════════════════════════════╗
║              OFFICIAL TRANSCRIPT                       ║
║════════════════════════════════════════════════════════║
║  Student: ${student.name.padEnd(42)} ║
║  ID: ${student.id.padEnd(47)} ║
║  GPA: ${student.gpa.toFixed(2).padEnd(46)} ║
║  Semester: ${semester.padEnd(41)} ║
╚════════════════════════════════════════════════════════╝
`;
```

### Expression Interpolation

```typescript
// Complex expressions inside template literals
const student: Student = {
  id: "STU001",
  name: "Alice Johnson",
  email: "alice@college.edu",
  gpa: 3.85,
  enrolledCourses: ["CS101", "MATH201", "PHYS101"]
};

// Conditional in template
const gpaStatus = `GPA: ${student.gpa} - ${student.gpa >= 3.5 ? 'Honors' : 'Regular'}`;

// Function call in template
const courseList = `Enrolled in: ${student.enrolledCourses.join(', ')}`;

// Arithmetic in template
const remainingSlots = `Available slots: ${8 - student.enrolledCourses.length}`;
```

---

## Destructuring {#destructuring}

Destructuring allows unpacking values from arrays or properties from objects into distinct variables.

### Object Destructuring

```typescript
// Student object
const student: Student = {
  id: "STU001",
  name: "Alice Johnson",
  email: "alice@college.edu",
  gpa: 3.85,
  enrolledCourses: ["CS101", "MATH201"]
};

// Old way
const name = student.name;
const email = student.email;
const gpa = student.gpa;

// Modern way (destructuring)
const { name, email, gpa } = student;

// Renaming variables
const { name: studentName, email: studentEmail } = student;

// Default values
const { name, phone = "Not provided" } = student;

// Nested destructuring
interface StudentWithAddress {
  id: string;
  name: string;
  address: {
    street: string;
    city: string;
    zipCode: string;
  };
}

const studentWithAddr: StudentWithAddress = {
  id: "STU001",
  name: "Alice Johnson",
  address: {
    street: "123 College Ave",
    city: "Boston",
    zipCode: "02101"
  }
};

const {
  name: fullName,
  address: { city, zipCode }
} = studentWithAddr;
```

### Array Destructuring

```typescript
// Array of top students
const topStudents = ["Alice Johnson", "Bob Smith", "Carol White"];

// Old way
const first = topStudents[0];
const second = topStudents[1];

// Modern way
const [first, second, third] = topStudents;

// Skip elements
const [winner, , thirdPlace] = topStudents;

// Rest elements
const [topStudent, ...otherStudents] = topStudents;

// Swapping variables
let studentA = "Alice";
let studentB = "Bob";
[studentA, studentB] = [studentB, studentA]; // Swap!
```

### Function Parameter Destructuring

```typescript
// Without destructuring
function displayStudent(student: Student): string {
  return `${student.name} - GPA: ${student.gpa}`;
}

// With destructuring
function displayStudentModern({ name, gpa }: Student): string {
  return `${name} - GPA: ${gpa}`;
}

// With default values
function createCourse({
  name,
  code,
  credits = 3,
  instructor = "TBA"
}: Partial<Course> & Pick<Course, 'name' | 'code'>): Course {
  return {
    id: crypto.randomUUID(),
    name,
    code,
    credits,
    instructor,
    schedule: { days: [], time: "" }
  };
}

// Practical example: Enrollment function
interface EnrollmentRequest {
  studentId: string;
  courseId: string;
  semester: string;
  notes?: string;
}

function processEnrollment({
  studentId,
  courseId,
  semester,
  notes = "No notes"
}: EnrollmentRequest): string {
  return `Enrolling student ${studentId} in ${courseId} for ${semester}. Notes: ${notes}`;
}
```

---

## Spread and Rest Operators {#spread-rest-operators}

The spread (`...`) operator expands elements, while rest collects them.

### Spread Operator

```typescript
// Array spreading
const coreCourses = ["CS101", "MATH201", "ENG101"];
const electiveCourses = ["ART101", "MUS101"];

// Combine arrays
const allCourses = [...coreCourses, ...electiveCourses];

// Copy array
const coursesCopy = [...coreCourses];

// Add elements
const coursesWithNew = [...coreCourses, "PHYS101", "CHEM101"];

// Object spreading
const student: Student = {
  id: "STU001",
  name: "Alice Johnson",
  email: "alice@college.edu",
  gpa: 3.85,
  enrolledCourses: ["CS101"]
};

// Copy object
const studentCopy = { ...student };

// Update properties (immutable)
const updatedStudent = {
  ...student,
  gpa: 3.90,
  enrolledCourses: [...student.enrolledCourses, "MATH201"]
};

// Merge objects
const contactInfo = {
  phone: "555-0100",
  address: "123 College Ave"
};

const studentWithContact = {
  ...student,
  ...contactInfo
};
```

### Rest Operator

```typescript
// Function with rest parameters
function calculateTotalCredits(...courseCredits: number[]): number {
  return courseCredits.reduce((sum, credits) => sum + credits, 0);
}

const total = calculateTotalCredits(3, 4, 3, 3, 4); // 17

// Array destructuring with rest
const grades = [95, 88, 92, 85, 90];
const [highest, secondHighest, ...otherGrades] = grades;

// Object destructuring with rest
const course: Course = {
  id: "CRS001",
  name: "Introduction to Programming",
  code: "CS101",
  credits: 4,
  instructor: "Dr. Smith",
  schedule: { days: ["Mon", "Wed"], time: "10:00 AM" }
};

const { id, name, ...courseDetails } = course;
// courseDetails contains: code, credits, instructor, schedule
```

### Practical College Portal Examples

```typescript
// Add course to student's enrollment
function enrollInCourse(student: Student, courseId: string): Student {
  return {
    ...student,
    enrolledCourses: [...student.enrolledCourses, courseId]
  };
}

// Remove course from enrollment
function dropCourse(student: Student, courseId: string): Student {
  return {
    ...student,
    enrolledCourses: student.enrolledCourses.filter(id => id !== courseId)
  };
}

// Update multiple student fields
function updateStudentInfo(
  student: Student,
  updates: Partial<Student>
): Student {
  return {
    ...student,
    ...updates
  };
}

// Merge course schedules
function mergeCourseSchedules(...courses: Course[]): string[] {
  const allDays = courses.flatMap(course => course.schedule.days);
  return [...new Set(allDays)]; // Remove duplicates
}

// Create enrollment with optional fields
interface Enrollment {
  id: string;
  studentId: string;
  courseId: string;
  semester: string;
  grade?: string;
  status: "active" | "dropped" | "completed";
}

function createEnrollment(
  required: Pick<Enrollment, 'studentId' | 'courseId' | 'semester'>,
  optional: Partial<Enrollment> = {}
): Enrollment {
  return {
    id: crypto.randomUUID(),
    status: "active",
    ...required,
    ...optional
  };
}
```

---

## Array Methods {#array-methods}

Modern JavaScript provides powerful array methods for data manipulation.

### map() - Transform Elements

**Analogy**: Think of `map()` as a factory assembly line. Each item goes in, gets transformed, and comes out as something new.

```typescript
const students: Student[] = [
  { id: "1", name: "Alice", email: "alice@college.edu", gpa: 3.85, enrolledCourses: [] },
  { id: "2", name: "Bob", email: "bob@college.edu", gpa: 3.60, enrolledCourses: [] },
  { id: "3", name: "Carol", email: "carol@college.edu", gpa: 3.92, enrolledCourses: [] }
];

// Extract student names
const studentNames = students.map(student => student.name);
// ["Alice", "Bob", "Carol"]

// Format for display
const displayList = students.map(student => ({
  label: `${student.name} (GPA: ${student.gpa})`,
  value: student.id
}));

// Add computed properties
const studentsWithStatus = students.map(student => ({
  ...student,
  status: student.gpa >= 3.5 ? "Honors" : "Regular",
  enrollmentCount: student.enrolledCourses.length
}));

// Transform courses for dropdown
const courses: Course[] = [
  {
    id: "1",
    name: "Intro to Programming",
    code: "CS101",
    credits: 4,
    instructor: "Dr. Smith",
    schedule: { days: ["Mon", "Wed"], time: "10:00 AM" }
  }
];

const courseOptions = courses.map(course => ({
  id: course.id,
  label: `${course.code} - ${course.name}`,
  description: `${course.credits} credits, ${course.instructor}`
}));
```

### filter() - Select Elements

**Analogy**: Think of `filter()` as a bouncer at a club - only items that meet the criteria get through.

```typescript
// Find honors students
const honorsStudents = students.filter(student => student.gpa >= 3.5);

// Find students with specific enrollment count
const fullTimeStudents = students.filter(
  student => student.enrolledCourses.length >= 4
);

// Find available courses
interface CourseWithCapacity extends Course {
  enrolled: number;
  capacity: number;
}

const coursesWithCapacity: CourseWithCapacity[] = [
  {
    id: "1",
    name: "Intro to Programming",
    code: "CS101",
    credits: 4,
    instructor: "Dr. Smith",
    schedule: { days: ["Mon", "Wed"], time: "10:00 AM" },
    enrolled: 25,
    capacity: 30
  },
  {
    id: "2",
    name: "Data Structures",
    code: "CS201",
    credits: 4,
    instructor: "Dr. Johnson",
    schedule: { days: ["Tue", "Thu"], time: "2:00 PM" },
    enrolled: 30,
    capacity: 30
  }
];

const availableCourses = coursesWithCapacity.filter(
  course => course.enrolled < course.capacity
);

// Complex filtering
const morningCSCourses = coursesWithCapacity.filter(course =>
  course.code.startsWith("CS") &&
  course.schedule.time.includes("AM")
);

// Chaining filter and map
const honorsStudentNames = students
  .filter(student => student.gpa >= 3.5)
  .map(student => student.name);
```

### reduce() - Aggregate Data

**Analogy**: Think of `reduce()` as a blender - it takes many items and combines them into one result.

```typescript
// Calculate total credits
const totalCredits = courses.reduce(
  (sum, course) => sum + course.credits,
  0
);

// Calculate average GPA
const averageGPA = students.reduce(
  (sum, student) => sum + student.gpa,
  0
) / students.length;

// Count students by status
const studentsByStatus = students.reduce((acc, student) => {
  const status = student.gpa >= 3.5 ? "Honors" : "Regular";
  acc[status] = (acc[status] || 0) + 1;
  return acc;
}, {} as Record<string, number>);
// Result: { Honors: 2, Regular: 1 }

// Group courses by instructor
const coursesByInstructor = courses.reduce((acc, course) => {
  if (!acc[course.instructor]) {
    acc[course.instructor] = [];
  }
  acc[course.instructor].push(course);
  return acc;
}, {} as Record<string, Course[]>);

// Build enrollment map
interface Enrollment {
  studentId: string;
  courseId: string;
  grade: number;
}

const enrollments: Enrollment[] = [
  { studentId: "1", courseId: "CS101", grade: 95 },
  { studentId: "1", courseId: "MATH201", grade: 88 },
  { studentId: "2", courseId: "CS101", grade: 92 }
];

const enrollmentMap = enrollments.reduce((acc, enrollment) => {
  if (!acc[enrollment.studentId]) {
    acc[enrollment.studentId] = [];
  }
  acc[enrollment.studentId].push(enrollment);
  return acc;
}, {} as Record<string, Enrollment[]>);

// Calculate statistics
interface Stats {
  total: number;
  average: number;
  highest: number;
  lowest: number;
}

const gradeStats = enrollments.reduce((stats, enrollment) => {
  const grade = enrollment.grade;
  return {
    total: stats.total + grade,
    average: 0, // Calculate after
    highest: Math.max(stats.highest, grade),
    lowest: Math.min(stats.lowest, grade)
  };
}, { total: 0, average: 0, highest: -Infinity, lowest: Infinity } as Stats);

gradeStats.average = gradeStats.total / enrollments.length;
```

### find() and findIndex()

```typescript
// Find first honors student
const firstHonorsStudent = students.find(student => student.gpa >= 3.5);

// Find student by ID
const findStudent = (id: string) =>
  students.find(student => student.id === id);

// Find course by code
const findCourse = (code: string) =>
  courses.find(course => course.code === code);

// Get index of student
const studentIndex = students.findIndex(student => student.id === "2");

// Update student using findIndex
function updateStudentById(students: Student[], id: string, updates: Partial<Student>): Student[] {
  const index = students.findIndex(s => s.id === id);
  if (index === -1) return students;

  const newStudents = [...students];
  newStudents[index] = { ...newStudents[index], ...updates };
  return newStudents;
}
```

### some() and every()

```typescript
// Check if any student has GPA below 2.0
const hasFailingStudent = students.some(student => student.gpa < 2.0);

// Check if any course is full
const hasFullCourse = coursesWithCapacity.some(
  course => course.enrolled >= course.capacity
);

// Check if all students are passing
const allPassing = students.every(student => student.gpa >= 2.0);

// Check if all courses have instructors
const allCoursesHaveInstructors = courses.every(
  course => course.instructor && course.instructor !== "TBA"
);

// Validate enrollment prerequisites
interface EnrollmentValidation {
  hasPrerequisites: boolean;
  hasAvailableSlots: boolean;
  meetsGPARequirement: boolean;
}

function canEnroll(student: Student, course: CourseWithCapacity): boolean {
  const validations: EnrollmentValidation = {
    hasPrerequisites: true, // Simplified
    hasAvailableSlots: course.enrolled < course.capacity,
    meetsGPARequirement: student.gpa >= 2.0
  };

  return Object.values(validations).every(v => v === true);
}
```

### forEach() - Iteration

```typescript
// Log all student names
students.forEach(student => {
  console.log(`Student: ${student.name}, GPA: ${student.gpa}`);
});

// Send enrollment confirmations
function sendEnrollmentConfirmations(students: Student[], course: Course): void {
  students.forEach(student => {
    const message = `Dear ${student.name}, you are enrolled in ${course.name}`;
    console.log(`Sending email to ${student.email}: ${message}`);
    // In real app: sendEmail(student.email, message);
  });
}

// Update DOM elements (side effects)
function updateStudentCards(students: Student[]): void {
  students.forEach((student, index) => {
    const element = document.getElementById(`student-${index}`);
    if (element) {
      element.textContent = `${student.name} - GPA: ${student.gpa}`;
    }
  });
}
```

### Method Chaining

```typescript
// Complex data transformation
const topStudentSummary = students
  .filter(student => student.gpa >= 3.5)
  .map(student => ({
    name: student.name,
    gpa: student.gpa,
    courseCount: student.enrolledCourses.length
  }))
  .sort((a, b) => b.gpa - a.gpa)
  .slice(0, 5)
  .map((student, index) => `${index + 1}. ${student.name} (${student.gpa})`)
  .join('\n');

// Calculate enrollment statistics
const enrollmentStats = coursesWithCapacity
  .map(course => ({
    code: course.code,
    fillRate: (course.enrolled / course.capacity) * 100
  }))
  .filter(stat => stat.fillRate >= 80)
  .sort((a, b) => b.fillRate - a.fillRate);

// Get popular instructors
const popularInstructors = courses
  .reduce((acc, course) => {
    acc[course.instructor] = (acc[course.instructor] || 0) + 1;
    return acc;
  }, {} as Record<string, number>)
  |> Object.entries(%) // Proposed pipeline operator
  .map(([instructor, count]) => ({ instructor, courseCount: count }))
  .filter(item => item.courseCount >= 3)
  .sort((a, b) => b.courseCount - a.courseCount);
```

---

## Promises and Async/Await {#promises-async-await}

Asynchronous programming handles operations that take time (API calls, file reading, etc.).

**Analogy**: Think of ordering food at a restaurant. You don't stand at the counter waiting for your food (blocking). Instead, you get a number (promise), sit down, and wait for your order to be ready (resolved) or cancelled (rejected).

### Promises Basics

```typescript
// Creating a promise
function fetchStudent(id: string): Promise<Student> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const student: Student = {
        id,
        name: "Alice Johnson",
        email: "alice@college.edu",
        gpa: 3.85,
        enrolledCourses: ["CS101", "MATH201"]
      };

      if (student) {
        resolve(student);
      } else {
        reject(new Error("Student not found"));
      }
    }, 1000);
  });
}

// Using promises with .then()
fetchStudent("STU001")
  .then(student => {
    console.log(`Found: ${student.name}`);
    return student.gpa;
  })
  .then(gpa => {
    console.log(`GPA: ${gpa}`);
  })
  .catch(error => {
    console.error("Error:", error.message);
  })
  .finally(() => {
    console.log("Operation complete");
  });
```

### Async/Await Syntax

```typescript
// Async function automatically returns a Promise
async function getStudentInfo(id: string): Promise<string> {
  try {
    const student = await fetchStudent(id);
    return `${student.name} has a GPA of ${student.gpa}`;
  } catch (error) {
    console.error("Failed to fetch student:", error);
    throw error;
  }
}

// Using async/await
async function displayStudentDashboard(studentId: string): Promise<void> {
  try {
    const student = await fetchStudent(studentId);
    console.log(`Welcome, ${student.name}!`);
    console.log(`Your GPA: ${student.gpa}`);
    console.log(`Enrolled in ${student.enrolledCourses.length} courses`);
  } catch (error) {
    console.error("Error loading dashboard:", error);
  }
}
```

### Real API Examples

```typescript
// API types
interface APIResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface CourseEnrollment {
  id: string;
  studentId: string;
  courseId: string;
  enrolledAt: string;
  status: "active" | "dropped" | "completed";
}

// Fetch student from API
async function fetchStudentFromAPI(id: string): Promise<Student> {
  const response = await fetch(`https://api.college.edu/students/${id}`);

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }

  const data: APIResponse<Student> = await response.json();
  return data.data;
}

// Fetch multiple students
async function fetchAllStudents(): Promise<Student[]> {
  const response = await fetch('https://api.college.edu/students');
  const data: APIResponse<Student[]> = await response.json();
  return data.data;
}

// Create new enrollment
async function enrollStudent(
  studentId: string,
  courseId: string
): Promise<CourseEnrollment> {
  const response = await fetch('https://api.college.edu/enrollments', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ studentId, courseId })
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Enrollment failed');
  }

  const data: APIResponse<CourseEnrollment> = await response.json();
  return data.data;
}

// Update student information
async function updateStudent(
  id: string,
  updates: Partial<Student>
): Promise<Student> {
  const response = await fetch(`https://api.college.edu/students/${id}`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(updates)
  });

  if (!response.ok) {
    throw new Error('Update failed');
  }

  const data: APIResponse<Student> = await response.json();
  return data.data;
}

// Delete enrollment
async function dropCourse(enrollmentId: string): Promise<void> {
  const response = await fetch(
    `https://api.college.edu/enrollments/${enrollmentId}`,
    { method: 'DELETE' }
  );

  if (!response.ok) {
    throw new Error('Failed to drop course');
  }
}
```

### Parallel Requests with Promise.all()

```typescript
// Fetch student and their courses in parallel
async function loadStudentDashboard(studentId: string): Promise<{
  student: Student;
  courses: Course[];
  enrollments: CourseEnrollment[];
}> {
  try {
    const [student, courses, enrollments] = await Promise.all([
      fetchStudentFromAPI(studentId),
      fetch('https://api.college.edu/courses').then(r => r.json()),
      fetch(`https://api.college.edu/enrollments?studentId=${studentId}`)
        .then(r => r.json())
    ]);

    return {
      student,
      courses: courses.data,
      enrollments: enrollments.data
    };
  } catch (error) {
    console.error("Error loading dashboard:", error);
    throw error;
  }
}

// Load multiple students in parallel
async function loadMultipleStudents(ids: string[]): Promise<Student[]> {
  const promises = ids.map(id => fetchStudentFromAPI(id));
  return Promise.all(promises);
}

// Use Promise.allSettled for handling failures gracefully
async function loadStudentsWithFallback(
  ids: string[]
): Promise<{ successful: Student[]; failed: string[] }> {
  const promises = ids.map(id => fetchStudentFromAPI(id));
  const results = await Promise.allSettled(promises);

  const successful: Student[] = [];
  const failed: string[] = [];

  results.forEach((result, index) => {
    if (result.status === 'fulfilled') {
      successful.push(result.value);
    } else {
      failed.push(ids[index]);
    }
  });

  return { successful, failed };
}
```

### Error Handling

```typescript
// Custom error types
class APIError extends Error {
  constructor(public status: number, message: string) {
    super(message);
    this.name = 'APIError';
  }
}

class NetworkError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'NetworkError';
  }
}

// Comprehensive error handling
async function fetchWithErrorHandling<T>(url: string): Promise<T> {
  try {
    const response = await fetch(url);

    if (!response.ok) {
      throw new APIError(
        response.status,
        `API returned ${response.status}: ${response.statusText}`
      );
    }

    return await response.json();
  } catch (error) {
    if (error instanceof APIError) {
      console.error(`API Error [${error.status}]:`, error.message);
    } else if (error instanceof TypeError) {
      throw new NetworkError("Network connection failed");
    } else {
      console.error("Unexpected error:", error);
    }
    throw error;
  }
}

// Retry logic
async function fetchWithRetry<T>(
  url: string,
  maxRetries: number = 3
): Promise<T> {
  let lastError: Error;

  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return await response.json();
    } catch (error) {
      lastError = error as Error;
      console.log(`Attempt ${i + 1} failed, retrying...`);
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }

  throw lastError!;
}
```

---

## Modules (Import/Export) {#modules}

Modules allow you to organize code into separate files and import/export functionality.

**Analogy**: Think of modules as a library organization system. Each book (module) contains specific knowledge, and you can borrow (import) the books you need.

### Exporting

```typescript
// student.types.ts - Named exports
export interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
  enrolledCourses: string[];
}

export interface StudentFilters {
  minGPA?: number;
  maxGPA?: number;
  enrolledIn?: string;
}

export type StudentStatus = "active" | "inactive" | "graduated";

// student.utils.ts - Named exports
export function calculateAverageGPA(students: Student[]): number {
  if (students.length === 0) return 0;
  const total = students.reduce((sum, s) => sum + s.gpa, 0);
  return total / students.length;
}

export function filterStudentsByGPA(
  students: Student[],
  minGPA: number
): Student[] {
  return students.filter(s => s.gpa >= minGPA);
}

export const MAX_COURSES = 6;
export const MIN_GPA_FOR_HONORS = 3.5;

// student.service.ts - Default export
export default class StudentService {
  private baseURL: string;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
  }

  async fetchStudent(id: string): Promise<Student> {
    const response = await fetch(`${this.baseURL}/students/${id}`);
    return response.json();
  }

  async updateStudent(id: string, updates: Partial<Student>): Promise<Student> {
    const response = await fetch(`${this.baseURL}/students/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(updates)
    });
    return response.json();
  }
}

// Mixed exports
export { StudentService as default };
export { calculateAverageGPA, filterStudentsByGPA };
```

### Importing

```typescript
// Import named exports
import { Student, StudentFilters } from './student.types';
import { calculateAverageGPA, filterStudentsByGPA } from './student.utils';

// Import with aliases
import {
  Student as StudentType,
  StudentFilters as Filters
} from './student.types';

// Import all as namespace
import * as StudentUtils from './student.utils';
const avgGPA = StudentUtils.calculateAverageGPA(students);

// Import default export
import StudentService from './student.service';

// Import default with named exports
import StudentService, { calculateAverageGPA } from './student.service';

// Side-effect imports (for CSS, initialization)
import './styles.css';
import './init-analytics';
```

### Module Organization Example

```typescript
// types/student.types.ts
export interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
  enrolledCourses: string[];
}

export interface Course {
  id: string;
  name: string;
  code: string;
  credits: number;
  instructor: string;
}

// types/index.ts - Barrel export
export * from './student.types';
export * from './course.types';
export * from './enrollment.types';

// utils/validation.ts
export function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

export function validateGPA(gpa: number): boolean {
  return gpa >= 0 && gpa <= 4.0;
}

// services/api.service.ts
import { Student, Course } from '../types';

export class APIService {
  constructor(private baseURL: string) {}

  async fetchStudents(): Promise<Student[]> {
    const response = await fetch(`${this.baseURL}/students`);
    return response.json();
  }

  async fetchCourses(): Promise<Course[]> {
    const response = await fetch(`${this.baseURL}/courses`);
    return response.json();
  }
}

// main.ts - Using all modules
import { Student, Course } from './types';
import { validateEmail, validateGPA } from './utils/validation';
import { APIService } from './services/api.service';

async function initializeApp() {
  const api = new APIService('https://api.college.edu');
  const students = await api.fetchStudents();
  console.log(`Loaded ${students.length} students`);
}
```

---

## Optional Chaining and Nullish Coalescing {#optional-chaining}

These features handle undefined/null values elegantly.

### Optional Chaining (?.)

**Analogy**: Like checking if each floor of a building exists before taking the elevator up, instead of crashing when a floor is missing.

```typescript
interface Address {
  street: string;
  city: string;
  zipCode: string;
}

interface StudentProfile {
  id: string;
  name: string;
  address?: Address;
  contactInfo?: {
    phone?: string;
    alternateEmail?: string;
  };
}

const student: StudentProfile = {
  id: "STU001",
  name: "Alice Johnson",
  // address and contactInfo are optional
};

// Old way - verbose null checking
const city = student.address && student.address.city;

// Modern way - optional chaining
const city2 = student.address?.city;

// Deep optional chaining
const phone = student.contactInfo?.phone;
const altEmail = student.contactInfo?.alternateEmail;

// Optional chaining with arrays
const firstCourse = student.enrolledCourses?.[0];

// Optional chaining with functions
const formatName = student.formatName?.();

// Real example
interface CourseWithPrereqs extends Course {
  prerequisites?: {
    courses?: string[];
    minGPA?: number;
  };
}

function checkPrerequisites(course: CourseWithPrereqs, student: Student): boolean {
  // Check if course has prerequisites and if student meets them
  const requiredGPA = course.prerequisites?.minGPA ?? 0;
  const requiredCourses = course.prerequisites?.courses ?? [];

  const meetsGPARequirement = student.gpa >= requiredGPA;
  const hasRequiredCourses = requiredCourses.every(
    courseId => student.enrolledCourses?.includes(courseId)
  );

  return meetsGPARequirement && hasRequiredCourses;
}
```

### Nullish Coalescing (??)

**Analogy**: Like having a backup plan - use the primary option if it exists, otherwise use the fallback.

```typescript
// Difference between || and ??
const count1 = 0 || 10;  // 10 (0 is falsy)
const count2 = 0 ?? 10;  // 0 (0 is not null/undefined)

const name1 = "" || "Anonymous";  // "Anonymous" ("" is falsy)
const name2 = "" ?? "Anonymous";  // "" ("" is not null/undefined)

// Practical examples
function getStudentDisplay(student: StudentProfile): string {
  const phone = student.contactInfo?.phone ?? "No phone provided";
  const city = student.address?.city ?? "Unknown city";

  return `${student.name} from ${city}, contact: ${phone}`;
}

// Default configuration
interface AppConfig {
  maxEnrollments?: number;
  allowWaitlist?: boolean;
  semesterLength?: number;
}

function getConfig(customConfig?: AppConfig): Required<AppConfig> {
  return {
    maxEnrollments: customConfig?.maxEnrollments ?? 6,
    allowWaitlist: customConfig?.allowWaitlist ?? true,
    semesterLength: customConfig?.semesterLength ?? 16
  };
}

// Form field defaults
interface EnrollmentForm {
  studentId: string;
  courseId: string;
  semester: string;
  notes?: string;
  priority?: number;
}

function createEnrollment(formData: Partial<EnrollmentForm>): EnrollmentForm {
  return {
    studentId: formData.studentId ?? "",
    courseId: formData.courseId ?? "",
    semester: formData.semester ?? "Fall 2024",
    notes: formData.notes ?? "",
    priority: formData.priority ?? 1
  };
}
```

### Combining Both Operators

```typescript
interface StudentWithMetadata {
  id: string;
  name: string;
  metadata?: {
    lastLogin?: Date;
    preferences?: {
      theme?: "light" | "dark";
      notifications?: boolean;
    };
  };
}

function getStudentPreferences(student: StudentWithMetadata) {
  return {
    theme: student.metadata?.preferences?.theme ?? "light",
    notifications: student.metadata?.preferences?.notifications ?? true,
    lastLogin: student.metadata?.lastLogin ?? new Date()
  };
}

// API response handling
async function fetchStudentSafely(id: string): Promise<Student | null> {
  try {
    const response = await fetch(`/api/students/${id}`);
    const data = await response.json();
    return data?.student ?? null;
  } catch (error) {
    console.error("Fetch failed:", error);
    return null;
  }
}

// Display with fallbacks
function renderStudentCard(student?: Student): string {
  const name = student?.name ?? "Unknown Student";
  const gpa = student?.gpa?.toFixed(2) ?? "N/A";
  const courseCount = student?.enrolledCourses?.length ?? 0;

  return `
    <div class="student-card">
      <h3>${name}</h3>
      <p>GPA: ${gpa}</p>
      <p>Courses: ${courseCount}</p>
    </div>
  `;
}
```

---

## DOM Manipulation Basics {#dom-manipulation}

The DOM (Document Object Model) is how JavaScript interacts with HTML.

**Analogy**: The DOM is like a family tree of HTML elements. JavaScript can find, modify, add, or remove members of this tree.

### Selecting Elements

```typescript
// Get single element by ID
const loginButton = document.getElementById('login-btn');

// Query selector (CSS selectors)
const firstStudent = document.querySelector('.student-card');
const emailInput = document.querySelector<HTMLInputElement>('#email-input');

// Query all elements
const allStudents = document.querySelectorAll('.student-card');
const allButtons = document.querySelectorAll<HTMLButtonElement>('button');

// Type-safe element selection
function getElement<T extends HTMLElement>(selector: string): T | null {
  return document.querySelector<T>(selector);
}

const input = getElement<HTMLInputElement>('#student-name');
if (input) {
  input.value = "Alice Johnson";
}
```

### Modifying Content

```typescript
// Text content
const heading = document.getElementById('page-title');
if (heading) {
  heading.textContent = "Student Portal";
}

// HTML content
const container = document.getElementById('student-container');
if (container) {
  container.innerHTML = `
    <div class="student-card">
      <h3>Alice Johnson</h3>
      <p>GPA: 3.85</p>
    </div>
  `;
}

// Safe HTML rendering with template
function renderStudentCard(student: Student): string {
  return `
    <div class="student-card" data-student-id="${student.id}">
      <h3>${escapeHTML(student.name)}</h3>
      <p>Email: ${escapeHTML(student.email)}</p>
      <p>GPA: ${student.gpa.toFixed(2)}</p>
      <p>Enrolled Courses: ${student.enrolledCourses.length}</p>
    </div>
  `;
}

function escapeHTML(str: string): string {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}

// Render multiple students
function renderStudentList(students: Student[]): void {
  const container = document.getElementById('student-list');
  if (!container) return;

  container.innerHTML = students.map(renderStudentCard).join('');
}
```

### Modifying Attributes and Styles

```typescript
// Attributes
const link = document.querySelector<HTMLAnchorElement>('.course-link');
if (link) {
  link.href = "/courses/CS101";
  link.target = "_blank";
  link.setAttribute('data-course-id', 'CS101');

  const courseId = link.getAttribute('data-course-id');
}

// Classes
const card = document.querySelector('.student-card');
if (card) {
  card.classList.add('active');
  card.classList.remove('inactive');
  card.classList.toggle('highlighted');
  const hasActive = card.classList.contains('active');
}

// Styles
const header = document.getElementById('header');
if (header) {
  header.style.backgroundColor = '#1a202c';
  header.style.color = 'white';
  header.style.padding = '1rem';
}

// Style with TypeScript
function highlightHonorsStudent(element: HTMLElement): void {
  element.style.borderColor = '#f59e0b';
  element.style.borderWidth = '2px';
  element.style.backgroundColor = '#fffbeb';
}
```

### Creating and Adding Elements

```typescript
// Create new element
const newCard = document.createElement('div');
newCard.className = 'student-card';
newCard.innerHTML = `
  <h3>New Student</h3>
  <p>Just enrolled!</p>
`;

// Append to parent
const container = document.getElementById('student-list');
container?.appendChild(newCard);

// Insert at specific position
container?.insertBefore(newCard, container.firstChild);

// Create element with function
function createStudentCard(student: Student): HTMLDivElement {
  const card = document.createElement('div');
  card.className = 'student-card';
  card.dataset.studentId = student.id;

  const name = document.createElement('h3');
  name.textContent = student.name;

  const gpa = document.createElement('p');
  gpa.textContent = `GPA: ${student.gpa.toFixed(2)}`;

  const coursesCount = document.createElement('p');
  coursesCount.textContent = `Enrolled in ${student.enrolledCourses.length} courses`;

  card.appendChild(name);
  card.appendChild(gpa);
  card.appendChild(coursesCount);

  return card;
}

// Render list of students
function renderStudents(students: Student[]): void {
  const container = document.getElementById('student-list');
  if (!container) return;

  // Clear existing content
  container.innerHTML = '';

  // Add each student card
  students.forEach(student => {
    const card = createStudentCard(student);
    container.appendChild(card);
  });
}
```

### Removing Elements

```typescript
// Remove element
const oldCard = document.getElementById('student-123');
oldCard?.remove();

// Remove all children
const container = document.getElementById('student-list');
if (container) {
  while (container.firstChild) {
    container.removeChild(container.firstChild);
  }
  // Or simply: container.innerHTML = '';
}

// Remove specific student card
function removeStudentCard(studentId: string): void {
  const card = document.querySelector(`[data-student-id="${studentId}"]`);
  card?.remove();
}
```

---

## Event Handling {#event-handling}

Events allow JavaScript to respond to user interactions.

**Analogy**: Events are like doorbells - when someone presses the button (event happens), the bell rings (your code runs).

### Adding Event Listeners

```typescript
// Basic event listener
const button = document.getElementById('enroll-btn');
button?.addEventListener('click', () => {
  console.log('Enroll button clicked!');
});

// With event parameter
button?.addEventListener('click', (event: MouseEvent) => {
  console.log('Clicked at:', event.clientX, event.clientY);
  event.preventDefault(); // Prevent default behavior
});

// Form submission
const form = document.getElementById('enrollment-form');
form?.addEventListener('submit', (event: SubmitEvent) => {
  event.preventDefault(); // Don't reload page

  const formData = new FormData(event.target as HTMLFormElement);
  const studentId = formData.get('studentId') as string;
  const courseId = formData.get('courseId') as string;

  console.log('Enrolling:', studentId, 'in', courseId);
});

// Input changes
const searchInput = document.getElementById('search-input');
searchInput?.addEventListener('input', (event: Event) => {
  const target = event.target as HTMLInputElement;
  console.log('Search query:', target.value);
});
```

### Event Delegation

```typescript
// Instead of adding listener to each card
const container = document.getElementById('student-list');

container?.addEventListener('click', (event: MouseEvent) => {
  const target = event.target as HTMLElement;

  // Find closest student card
  const card = target.closest('.student-card');
  if (!card) return;

  const studentId = card.getAttribute('data-student-id');
  console.log('Clicked student:', studentId);
});

// Handle different button clicks
container?.addEventListener('click', (event: MouseEvent) => {
  const target = event.target as HTMLElement;

  if (target.matches('.view-btn')) {
    const studentId = target.dataset.studentId;
    viewStudent(studentId);
  } else if (target.matches('.edit-btn')) {
    const studentId = target.dataset.studentId;
    editStudent(studentId);
  } else if (target.matches('.delete-btn')) {
    const studentId = target.dataset.studentId;
    deleteStudent(studentId);
  }
});

function viewStudent(id: string | undefined): void {
  if (!id) return;
  console.log('Viewing student:', id);
}

function editStudent(id: string | undefined): void {
  if (!id) return;
  console.log('Editing student:', id);
}

function deleteStudent(id: string | undefined): void {
  if (!id) return;
  console.log('Deleting student:', id);
}
```

### Common Event Types

```typescript
// Mouse events
element?.addEventListener('click', handleClick);
element?.addEventListener('dblclick', handleDoubleClick);
element?.addEventListener('mouseenter', handleMouseEnter);
element?.addEventListener('mouseleave', handleMouseLeave);
element?.addEventListener('mousemove', handleMouseMove);

// Keyboard events
input?.addEventListener('keydown', (e: KeyboardEvent) => {
  if (e.key === 'Enter') {
    console.log('Enter pressed');
  }
});

input?.addEventListener('keyup', handleKeyUp);

// Form events
form?.addEventListener('submit', handleSubmit);
input?.addEventListener('change', handleChange);
input?.addEventListener('input', handleInput);
input?.addEventListener('focus', handleFocus);
input?.addEventListener('blur', handleBlur);

// Window events
window.addEventListener('load', handlePageLoad);
window.addEventListener('resize', handleResize);
window.addEventListener('scroll', handleScroll);

function handleClick(event: MouseEvent): void {
  console.log('Clicked!');
}

function handleDoubleClick(event: MouseEvent): void {
  console.log('Double clicked!');
}

function handleMouseEnter(event: MouseEvent): void {
  const target = event.target as HTMLElement;
  target.style.backgroundColor = '#f3f4f6';
}

function handleMouseLeave(event: MouseEvent): void {
  const target = event.target as HTMLElement;
  target.style.backgroundColor = '';
}

function handleMouseMove(event: MouseEvent): void {
  console.log('Mouse at:', event.clientX, event.clientY);
}

function handleKeyUp(event: KeyboardEvent): void {
  console.log('Key released:', event.key);
}

function handleSubmit(event: SubmitEvent): void {
  event.preventDefault();
}

function handleChange(event: Event): void {
  const target = event.target as HTMLInputElement;
  console.log('Changed to:', target.value);
}

function handleInput(event: Event): void {
  const target = event.target as HTMLInputElement;
  console.log('Input:', target.value);
}

function handleFocus(event: FocusEvent): void {
  const target = event.target as HTMLElement;
  target.style.outline = '2px solid #3b82f6';
}

function handleBlur(event: FocusEvent): void {
  const target = event.target as HTMLElement;
  target.style.outline = '';
}

function handlePageLoad(): void {
  console.log('Page loaded');
}

function handleResize(): void {
  console.log('Window resized to:', window.innerWidth, window.innerHeight);
}

function handleScroll(): void {
  console.log('Scrolled to:', window.scrollY);
}
```

### Removing Event Listeners

```typescript
function handleButtonClick(): void {
  console.log('Button clicked');
}

const button = document.getElementById('my-button');

// Add listener
button?.addEventListener('click', handleButtonClick);

// Remove listener (must be same function reference)
button?.removeEventListener('click', handleButtonClick);

// Using AbortController for cleanup
const controller = new AbortController();

button?.addEventListener('click', handleButtonClick, {
  signal: controller.signal
});

// Later: remove all listeners added with this signal
controller.abort();
```

---

## Practical Examples: College Portal {#practical-examples}

Let's build a complete interactive student dashboard.

### HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>College Portal</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: system-ui; padding: 2rem; background: #f9fafb; }
    .container { max-width: 1200px; margin: 0 auto; }
    .header { background: #1f2937; color: white; padding: 2rem; border-radius: 8px; margin-bottom: 2rem; }
    .filters { background: white; padding: 1.5rem; border-radius: 8px; margin-bottom: 2rem; display: flex; gap: 1rem; }
    .filters input { padding: 0.5rem; border: 1px solid #d1d5db; border-radius: 4px; flex: 1; }
    .students-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 1.5rem; }
    .student-card { background: white; padding: 1.5rem; border-radius: 8px; border-left: 4px solid #3b82f6; }
    .student-card.honors { border-left-color: #f59e0b; }
    .student-card h3 { margin-bottom: 0.5rem; color: #1f2937; }
    .student-card p { color: #6b7280; margin: 0.25rem 0; }
    .student-card .gpa { font-size: 1.5rem; font-weight: bold; color: #3b82f6; margin: 0.5rem 0; }
    .badge { display: inline-block; padding: 0.25rem 0.75rem; border-radius: 9999px; font-size: 0.875rem; font-weight: 500; }
    .badge.honors { background: #fef3c7; color: #92400e; }
    .badge.regular { background: #dbeafe; color: #1e40af; }
    .actions { margin-top: 1rem; display: flex; gap: 0.5rem; }
    .btn { padding: 0.5rem 1rem; border: none; border-radius: 4px; cursor: pointer; font-size: 0.875rem; }
    .btn-primary { background: #3b82f6; color: white; }
    .btn-secondary { background: #e5e7eb; color: #374151; }
    .btn:hover { opacity: 0.9; }
    .stats { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1rem; margin-bottom: 2rem; }
    .stat-card { background: white; padding: 1.5rem; border-radius: 8px; }
    .stat-card .label { color: #6b7280; font-size: 0.875rem; }
    .stat-card .value { font-size: 2rem; font-weight: bold; color: #1f2937; margin-top: 0.5rem; }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>College Student Portal</h1>
      <p>Manage and view student information</p>
    </div>

    <div class="stats" id="stats"></div>

    <div class="filters">
      <input type="text" id="search-input" placeholder="Search by name...">
      <select id="status-filter">
        <option value="all">All Students</option>
        <option value="honors">Honors Only</option>
        <option value="regular">Regular Only</option>
      </select>
    </div>

    <div class="students-grid" id="students-grid"></div>
  </div>

  <script src="app.js"></script>
</body>
</html>
```

### Complete TypeScript Application

```typescript
// types.ts
interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
  enrolledCourses: string[];
  major: string;
}

interface Stats {
  total: number;
  averageGPA: number;
  honorsCount: number;
}

// data.ts - Mock data
const students: Student[] = [
  {
    id: "STU001",
    name: "Alice Johnson",
    email: "alice@college.edu",
    gpa: 3.85,
    enrolledCourses: ["CS101", "MATH201", "PHYS101", "ENG101"],
    major: "Computer Science"
  },
  {
    id: "STU002",
    name: "Bob Smith",
    email: "bob@college.edu",
    gpa: 3.60,
    enrolledCourses: ["BIO101", "CHEM101", "MATH101"],
    major: "Biology"
  },
  {
    id: "STU003",
    name: "Carol White",
    email: "carol@college.edu",
    gpa: 3.92,
    enrolledCourses: ["CS101", "CS201", "MATH201", "PHYS201", "ENG101"],
    major: "Computer Science"
  },
  {
    id: "STU004",
    name: "David Brown",
    email: "david@college.edu",
    gpa: 3.25,
    enrolledCourses: ["HIST101", "ENG101", "ART101"],
    major: "History"
  },
  {
    id: "STU005",
    name: "Eve Davis",
    email: "eve@college.edu",
    gpa: 3.78,
    enrolledCourses: ["MATH301", "CS201", "STAT201"],
    major: "Mathematics"
  }
];

// utils.ts
function calculateStats(students: Student[]): Stats {
  return {
    total: students.length,
    averageGPA: calculateAverageGPA(students),
    honorsCount: students.filter(s => s.gpa >= 3.5).length
  };
}

function calculateAverageGPA(students: Student[]): number {
  if (students.length === 0) return 0;
  const total = students.reduce((sum, s) => sum + s.gpa, 0);
  return Number((total / students.length).toFixed(2));
}

function isHonorsStudent(student: Student): boolean {
  return student.gpa >= 3.5;
}

function getStudentStatus(student: Student): "honors" | "regular" {
  return isHonorsStudent(student) ? "honors" : "regular";
}

// dom.ts - DOM manipulation functions
function createStudentCard(student: Student): HTMLDivElement {
  const card = document.createElement('div');
  card.className = `student-card ${isHonorsStudent(student) ? 'honors' : ''}`;
  card.dataset.studentId = student.id;

  const status = getStudentStatus(student);
  const statusLabel = status === 'honors' ? 'Honors Student' : 'Regular Student';

  card.innerHTML = `
    <h3>${escapeHTML(student.name)}</h3>
    <p>${escapeHTML(student.email)}</p>
    <p><strong>Major:</strong> ${escapeHTML(student.major)}</p>
    <div class="gpa">${student.gpa.toFixed(2)}</div>
    <span class="badge ${status}">${statusLabel}</span>
    <p><strong>Enrolled Courses:</strong> ${student.enrolledCourses.length}</p>
    <div class="actions">
      <button class="btn btn-primary view-btn" data-student-id="${student.id}">
        View Details
      </button>
      <button class="btn btn-secondary edit-btn" data-student-id="${student.id}">
        Edit
      </button>
    </div>
  `;

  return card;
}

function escapeHTML(str: string): string {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}

function renderStudents(students: Student[]): void {
  const container = document.getElementById('students-grid');
  if (!container) return;

  container.innerHTML = '';

  if (students.length === 0) {
    container.innerHTML = '<p style="grid-column: 1/-1; text-align: center; color: #6b7280;">No students found</p>';
    return;
  }

  students.forEach(student => {
    const card = createStudentCard(student);
    container.appendChild(card);
  });
}

function renderStats(stats: Stats): void {
  const container = document.getElementById('stats');
  if (!container) return;

  container.innerHTML = `
    <div class="stat-card">
      <div class="label">Total Students</div>
      <div class="value">${stats.total}</div>
    </div>
    <div class="stat-card">
      <div class="label">Average GPA</div>
      <div class="value">${stats.averageGPA}</div>
    </div>
    <div class="stat-card">
      <div class="label">Honors Students</div>
      <div class="value">${stats.honorsCount}</div>
    </div>
  `;
}

// app.ts - Main application logic
class StudentPortal {
  private students: Student[];
  private filteredStudents: Student[];

  constructor(students: Student[]) {
    this.students = students;
    this.filteredStudents = students;
    this.initialize();
  }

  private initialize(): void {
    this.render();
    this.attachEventListeners();
  }

  private render(): void {
    renderStats(calculateStats(this.filteredStudents));
    renderStudents(this.filteredStudents);
  }

  private attachEventListeners(): void {
    // Search functionality
    const searchInput = document.getElementById('search-input') as HTMLInputElement;
    searchInput?.addEventListener('input', (e) => {
      const query = (e.target as HTMLInputElement).value.toLowerCase();
      this.filterStudents(query, this.getStatusFilter());
    });

    // Status filter
    const statusFilter = document.getElementById('status-filter') as HTMLSelectElement;
    statusFilter?.addEventListener('change', (e) => {
      const status = (e.target as HTMLSelectElement).value;
      this.filterStudents(this.getSearchQuery(), status);
    });

    // Event delegation for buttons
    const container = document.getElementById('students-grid');
    container?.addEventListener('click', (e) => {
      const target = e.target as HTMLElement;

      if (target.matches('.view-btn')) {
        const studentId = target.dataset.studentId;
        this.viewStudent(studentId);
      } else if (target.matches('.edit-btn')) {
        const studentId = target.dataset.studentId;
        this.editStudent(studentId);
      }
    });
  }

  private filterStudents(searchQuery: string, status: string): void {
    this.filteredStudents = this.students.filter(student => {
      const matchesSearch = student.name.toLowerCase().includes(searchQuery) ||
                           student.email.toLowerCase().includes(searchQuery) ||
                           student.major.toLowerCase().includes(searchQuery);

      const matchesStatus = status === 'all' ||
                           (status === 'honors' && isHonorsStudent(student)) ||
                           (status === 'regular' && !isHonorsStudent(student));

      return matchesSearch && matchesStatus;
    });

    this.render();
  }

  private getSearchQuery(): string {
    const input = document.getElementById('search-input') as HTMLInputElement;
    return input?.value.toLowerCase() || '';
  }

  private getStatusFilter(): string {
    const select = document.getElementById('status-filter') as HTMLSelectElement;
    return select?.value || 'all';
  }

  private viewStudent(id: string | undefined): void {
    if (!id) return;
    const student = this.students.find(s => s.id === id);
    if (student) {
      alert(`
Student Details:
Name: ${student.name}
Email: ${student.email}
Major: ${student.major}
GPA: ${student.gpa}
Enrolled Courses: ${student.enrolledCourses.join(', ')}
Status: ${getStudentStatus(student)}
      `);
    }
  }

  private editStudent(id: string | undefined): void {
    if (!id) return;
    console.log('Edit student:', id);
    // In a real app, this would open an edit modal
  }
}

// Initialize the application
document.addEventListener('DOMContentLoaded', () => {
  new StudentPortal(students);
});
```

### Async Version with API

```typescript
// api.service.ts
class StudentAPI {
  private baseURL: string;

  constructor(baseURL: string = 'https://api.college.edu') {
    this.baseURL = baseURL;
  }

  async fetchStudents(): Promise<Student[]> {
    try {
      const response = await fetch(`${this.baseURL}/students`);
      if (!response.ok) throw new Error('Failed to fetch students');
      const data = await response.json();
      return data.students;
    } catch (error) {
      console.error('Error fetching students:', error);
      return [];
    }
  }

  async fetchStudent(id: string): Promise<Student | null> {
    try {
      const response = await fetch(`${this.baseURL}/students/${id}`);
      if (!response.ok) throw new Error('Student not found');
      const data = await response.json();
      return data.student;
    } catch (error) {
      console.error('Error fetching student:', error);
      return null;
    }
  }

  async updateStudent(id: string, updates: Partial<Student>): Promise<Student | null> {
    try {
      const response = await fetch(`${this.baseURL}/students/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(updates)
      });
      if (!response.ok) throw new Error('Update failed');
      const data = await response.json();
      return data.student;
    } catch (error) {
      console.error('Error updating student:', error);
      return null;
    }
  }
}

// app-async.ts
class AsyncStudentPortal extends StudentPortal {
  private api: StudentAPI;
  private loading: boolean = false;

  constructor(api: StudentAPI) {
    super([]);
    this.api = api;
    this.loadStudents();
  }

  private async loadStudents(): Promise<void> {
    this.setLoading(true);
    const students = await this.api.fetchStudents();
    this.students = students;
    this.filteredStudents = students;
    this.render();
    this.setLoading(false);
  }

  private setLoading(loading: boolean): void {
    this.loading = loading;
    const container = document.getElementById('students-grid');
    if (!container) return;

    if (loading) {
      container.innerHTML = '<p style="grid-column: 1/-1; text-align: center;">Loading students...</p>';
    }
  }

  protected async viewStudent(id: string | undefined): Promise<void> {
    if (!id) return;
    const student = await this.api.fetchStudent(id);
    if (student) {
      // Display student details
      console.log('Viewing student:', student);
    }
  }
}

// Initialize with API
document.addEventListener('DOMContentLoaded', () => {
  const api = new StudentAPI();
  new AsyncStudentPortal(api);
});
```

---

## Summary

Modern JavaScript (ES6+) provides powerful features that make code more readable and maintainable:

1. **Arrow Functions**: Concise function syntax
2. **Template Literals**: String interpolation and multi-line strings
3. **Destructuring**: Extracting values from objects and arrays
4. **Spread/Rest**: Expanding and collecting elements
5. **Array Methods**: Powerful data transformation (map, filter, reduce)
6. **Promises/Async-Await**: Clean asynchronous code
7. **Modules**: Code organization with import/export
8. **Optional Chaining**: Safe property access
9. **Nullish Coalescing**: Default values for null/undefined
10. **DOM Manipulation**: Interacting with HTML elements
11. **Event Handling**: Responding to user interactions

These features form the foundation for modern frontend development with React and Next.js.

---

## Navigation

- **Previous**: [Part 2: HTML & CSS Fundamentals](./02_HTML_CSS_Fundamentals.md)
- **Next**: [Part 4: React Fundamentals](./04_React_Fundamentals.md)
- **[Back to Index](./README.md)**

---

**Author: Saiprasad Hegde**

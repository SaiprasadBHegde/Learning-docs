# Part 4: React Fundamentals

**Author: Saiprasad Hegde**

## Table of Contents
1. [What is React?](#what-is-react)
2. [What is a Component?](#what-is-component)
3. [Functional Components](#functional-components)
4. [JSX Syntax and Expressions](#jsx-syntax)
5. [Props and Prop Types](#props)
6. [Children Prop](#children-prop)
7. [Conditional Rendering](#conditional-rendering)
8. [Lists and Keys](#lists-keys)
9. [Component Composition](#component-composition)
10. [Thinking in React](#thinking-in-react)
11. [College Portal Examples](#college-examples)

---

## What is React? {#what-is-react}

React is a JavaScript library for building user interfaces, created by Facebook (Meta) in 2013.

**Analogy**: Think of React like building with LEGO blocks. Instead of building a house all at once, you create individual pieces (components) that snap together to form the complete structure. You can reuse the same blocks in different ways, and if one piece breaks, you only need to replace that piece, not rebuild the entire house.

### Why React?

```typescript
// Without React: Manual DOM manipulation
function updateStudentList(students) {
  const container = document.getElementById('student-list');
  container.innerHTML = ''; // Clear

  students.forEach(student => {
    const div = document.createElement('div');
    div.innerHTML = `<h3>${student.name}</h3><p>${student.gpa}</p>`;
    container.appendChild(div);
  });
}

// With React: Declarative UI
function StudentList({ students }) {
  return (
    <div>
      {students.map(student => (
        <div key={student.id}>
          <h3>{student.name}</h3>
          <p>{student.gpa}</p>
        </div>
      ))}
    </div>
  );
}
```

### React's Core Principles

1. **Component-Based**: Build encapsulated components that manage their own state
2. **Declarative**: Describe what the UI should look like, React handles updates
3. **Learn Once, Write Anywhere**: Use React for web, mobile (React Native), desktop

---

## What is a Component? {#what-is-component}

A component is a self-contained piece of UI that can be reused throughout your application.

**Analogy**: Think of components like recipes. A "Button" component is like a recipe for making a button - it defines the ingredients (props) and instructions (render logic). You can use the same recipe many times with different ingredients (different props).

### Component Example

```typescript
// A simple Student Card component
interface StudentCardProps {
  name: string;
  email: string;
  gpa: number;
}

function StudentCard({ name, email, gpa }: StudentCardProps) {
  return (
    <div className="student-card">
      <h3>{name}</h3>
      <p>{email}</p>
      <p>GPA: {gpa}</p>
    </div>
  );
}

// Using the component
function App() {
  return (
    <div>
      <StudentCard name="Alice Johnson" email="alice@college.edu" gpa={3.85} />
      <StudentCard name="Bob Smith" email="bob@college.edu" gpa={3.60} />
    </div>
  );
}
```

### Components Are Functions

In modern React, components are just JavaScript functions that return JSX (React's HTML-like syntax).

```typescript
// Component = Function that returns JSX
function Greeting() {
  return <h1>Hello, Students!</h1>;
}

// With TypeScript types
function TypedGreeting(): JSX.Element {
  return <h1>Hello, Students!</h1>;
}

// Or using React.FC (Functional Component)
import { FC } from 'react';

const FCGreeting: FC = () => {
  return <h1>Hello, Students!</h1>;
};
```

---

## Functional Components {#functional-components}

Functional components are the modern way to write React components.

### Basic Functional Component

```typescript
// components/StudentCard.tsx
import { FC } from 'react';

interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
  major: string;
}

interface StudentCardProps {
  student: Student;
}

const StudentCard: FC<StudentCardProps> = ({ student }) => {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 border-l-4 border-blue-500">
      <h3 className="text-xl font-bold text-gray-900">{student.name}</h3>
      <p className="text-gray-600">{student.email}</p>
      <p className="text-sm text-gray-500">{student.major}</p>
      <div className="mt-4">
        <span className="text-2xl font-bold text-blue-600">
          {student.gpa.toFixed(2)}
        </span>
        <span className="text-sm text-gray-500 ml-2">GPA</span>
      </div>
    </div>
  );
};

export default StudentCard;
```

### Component with Logic

```typescript
// components/CourseCard.tsx
import { FC } from 'react';

interface Course {
  id: string;
  code: string;
  name: string;
  credits: number;
  instructor: string;
  enrolled: number;
  capacity: number;
}

interface CourseCardProps {
  course: Course;
  onEnroll?: (courseId: string) => void;
}

const CourseCard: FC<CourseCardProps> = ({ course, onEnroll }) => {
  // Calculate availability
  const availableSeats = course.capacity - course.enrolled;
  const isFull = availableSeats <= 0;
  const isAlmostFull = availableSeats <= 5 && !isFull;

  // Handle enrollment
  const handleEnrollClick = () => {
    if (!isFull && onEnroll) {
      onEnroll(course.id);
    }
  };

  // Determine badge color
  const getBadgeColor = () => {
    if (isFull) return 'bg-red-100 text-red-800';
    if (isAlmostFull) return 'bg-yellow-100 text-yellow-800';
    return 'bg-green-100 text-green-800';
  };

  return (
    <div className="bg-white rounded-lg shadow-md p-6">
      <div className="flex justify-between items-start mb-4">
        <div>
          <h3 className="text-lg font-bold text-gray-900">{course.name}</h3>
          <p className="text-sm text-gray-500">{course.code}</p>
        </div>
        <span className={`px-3 py-1 rounded-full text-xs font-medium ${getBadgeColor()}`}>
          {availableSeats} seats
        </span>
      </div>

      <div className="space-y-2 text-sm text-gray-600">
        <p>Instructor: {course.instructor}</p>
        <p>Credits: {course.credits}</p>
        <p>Enrolled: {course.enrolled}/{course.capacity}</p>
      </div>

      <button
        onClick={handleEnrollClick}
        disabled={isFull}
        className={`mt-4 w-full py-2 px-4 rounded-md font-medium transition-colors ${
          isFull
            ? 'bg-gray-300 text-gray-500 cursor-not-allowed'
            : 'bg-blue-600 text-white hover:bg-blue-700'
        }`}
      >
        {isFull ? 'Course Full' : 'Enroll Now'}
      </button>
    </div>
  );
};

export default CourseCard;
```

### Component File Organization

```typescript
// Good: One component per file
// components/StudentCard.tsx
export const StudentCard = () => { /* ... */ };

// components/CourseCard.tsx
export const CourseCard = () => { /* ... */ };

// Better: Barrel exports for related components
// components/cards/index.ts
export { StudentCard } from './StudentCard';
export { CourseCard } from './CourseCard';
export { EnrollmentCard } from './EnrollmentCard';

// Usage
import { StudentCard, CourseCard } from '@/components/cards';
```

---

## JSX Syntax and Expressions {#jsx-syntax}

JSX is a syntax extension for JavaScript that looks like HTML but has the power of JavaScript.

**Analogy**: JSX is like a bilingual translator. You write something that looks like HTML, and React translates it into JavaScript function calls that create DOM elements.

### Basic JSX

```typescript
// JSX looks like HTML
const element = <h1>Hello, World!</h1>;

// But it's actually JavaScript
const element2 = React.createElement('h1', null, 'Hello, World!');

// JSX with attributes
const image = <img src="/logo.png" alt="College Logo" />;

// JSX with children
const card = (
  <div className="card">
    <h2>Title</h2>
    <p>Description</p>
  </div>
);
```

### JSX Expressions

```typescript
// JavaScript expressions in JSX using {}
const student = {
  name: "Alice Johnson",
  gpa: 3.85
};

const StudentInfo = () => {
  return (
    <div>
      <h2>{student.name}</h2>
      <p>GPA: {student.gpa}</p>

      {/* Arithmetic */}
      <p>Percentage: {student.gpa * 25}%</p>

      {/* Function calls */}
      <p>{student.name.toUpperCase()}</p>

      {/* Ternary operator */}
      <p>{student.gpa >= 3.5 ? 'Honors' : 'Regular'}</p>

      {/* Template literals */}
      <p>{`Student: ${student.name}`}</p>
    </div>
  );
};
```

### JSX Rules and Gotchas

```typescript
// ✅ CORRECT: Single root element
function Component1() {
  return (
    <div>
      <h1>Title</h1>
      <p>Content</p>
    </div>
  );
}

// ✅ CORRECT: Fragment (no extra div)
function Component2() {
  return (
    <>
      <h1>Title</h1>
      <p>Content</p>
    </>
  );
}

// ❌ WRONG: Multiple root elements
function Component3() {
  return (
    <h1>Title</h1>
    <p>Content</p>
  );
}

// ✅ CORRECT: className (not class)
const element = <div className="container">Content</div>;

// ✅ CORRECT: Self-closing tags
const input = <input type="text" />;
const image = <img src="/logo.png" alt="Logo" />;

// ✅ CORRECT: camelCase for attributes
const element2 = <div onClick={handleClick} tabIndex={0}>Click me</div>;

// ✅ CORRECT: Comments in JSX
const element3 = (
  <div>
    {/* This is a comment */}
    <p>Content</p>
  </div>
);
```

### Dynamic Attributes

```typescript
interface StudentCardProps {
  student: {
    id: string;
    name: string;
    status: 'active' | 'inactive' | 'graduated';
  };
}

const StudentCard: FC<StudentCardProps> = ({ student }) => {
  // Dynamic className
  const cardClass = `student-card ${student.status}`;

  // Computed styles
  const statusColor = {
    active: '#10b981',
    inactive: '#6b7280',
    graduated: '#3b82f6'
  }[student.status];

  return (
    <div className={cardClass} style={{ borderLeftColor: statusColor }}>
      <h3>{student.name}</h3>
      <span className={`badge badge-${student.status}`}>
        {student.status}
      </span>
    </div>
  );
};
```

### Conditional Attributes

```typescript
const Button: FC<{ disabled?: boolean; primary?: boolean }> = ({
  disabled,
  primary
}) => {
  return (
    <button
      className={`btn ${primary ? 'btn-primary' : 'btn-secondary'}`}
      disabled={disabled}
      aria-disabled={disabled}
      // Spread operator for conditional attributes
      {...(disabled && { tabIndex: -1 })}
    >
      Click Me
    </button>
  );
};
```

---

## Props and Prop Types {#props}

Props (properties) are how you pass data from parent to child components.

**Analogy**: Props are like arguments to a function. When you call a function, you pass in arguments. When you use a component, you pass in props.

### Basic Props

```typescript
// Defining prop types with interface
interface GreetingProps {
  name: string;
  age: number;
}

// Using props
const Greeting: FC<GreetingProps> = (props) => {
  return (
    <div>
      <h1>Hello, {props.name}!</h1>
      <p>You are {props.age} years old.</p>
    </div>
  );
};

// Destructuring props (preferred)
const GreetingDestructured: FC<GreetingProps> = ({ name, age }) => {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>You are {age} years old.</p>
    </div>
  );
};

// Usage
<Greeting name="Alice" age={20} />
```

### Optional Props

```typescript
interface StudentCardProps {
  name: string;
  email: string;
  gpa?: number; // Optional
  photoUrl?: string; // Optional
}

const StudentCard: FC<StudentCardProps> = ({
  name,
  email,
  gpa,
  photoUrl
}) => {
  return (
    <div className="student-card">
      {photoUrl && <img src={photoUrl} alt={name} />}
      <h3>{name}</h3>
      <p>{email}</p>
      {gpa !== undefined && <p>GPA: {gpa.toFixed(2)}</p>}
    </div>
  );
};
```

### Default Props

```typescript
// Method 1: Default parameters
interface ButtonProps {
  text: string;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
}

const Button: FC<ButtonProps> = ({
  text,
  variant = 'primary',
  size = 'medium'
}) => {
  return (
    <button className={`btn btn-${variant} btn-${size}`}>
      {text}
    </button>
  );
};

// Method 2: defaultProps (older approach, still valid)
const Button2: FC<ButtonProps> = ({ text, variant, size }) => {
  return <button className={`btn btn-${variant} btn-${size}`}>{text}</button>;
};

Button2.defaultProps = {
  variant: 'primary',
  size: 'medium'
};
```

### Complex Props

```typescript
// Object props
interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
  enrolledCourses: string[];
}

interface StudentCardProps {
  student: Student;
  onEdit?: (studentId: string) => void;
  onDelete?: (studentId: string) => void;
}

const StudentCard: FC<StudentCardProps> = ({ student, onEdit, onDelete }) => {
  return (
    <div className="student-card">
      <h3>{student.name}</h3>
      <p>{student.email}</p>
      <p>GPA: {student.gpa}</p>
      <p>Enrolled in {student.enrolledCourses.length} courses</p>

      <div className="actions">
        {onEdit && (
          <button onClick={() => onEdit(student.id)}>Edit</button>
        )}
        {onDelete && (
          <button onClick={() => onDelete(student.id)}>Delete</button>
        )}
      </div>
    </div>
  );
};
```

### Props Validation with TypeScript

```typescript
// Union types
interface BadgeProps {
  status: 'success' | 'warning' | 'error' | 'info';
  text: string;
}

// Literal types
interface HeadingProps {
  level: 1 | 2 | 3 | 4 | 5 | 6;
  text: string;
}

const Heading: FC<HeadingProps> = ({ level, text }) => {
  const Tag = `h${level}` as keyof JSX.IntrinsicElements;
  return <Tag>{text}</Tag>;
};

// Required vs Optional
interface FormInputProps {
  label: string;           // Required
  value: string;           // Required
  onChange: (value: string) => void;  // Required
  placeholder?: string;    // Optional
  error?: string;          // Optional
  disabled?: boolean;      // Optional
}
```

### Prop Spreading

```typescript
// Spreading props to child components
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary';
}

const Button: FC<ButtonProps> = ({ variant = 'primary', children, ...rest }) => {
  return (
    <button className={`btn btn-${variant}`} {...rest}>
      {children}
    </button>
  );
};

// Usage - all native button props work
<Button variant="primary" onClick={handleClick} disabled={isLoading}>
  Submit
</Button>
```

---

## Children Prop {#children-prop}

The `children` prop is special - it represents the content between component tags.

**Analogy**: Think of a component as a picture frame. The `children` prop is the picture you put inside the frame. The frame provides structure and styling, while the picture is the content.

### Basic Children

```typescript
// Simple children
interface CardProps {
  children: React.ReactNode;
}

const Card: FC<CardProps> = ({ children }) => {
  return (
    <div className="card">
      {children}
    </div>
  );
};

// Usage
<Card>
  <h2>Title</h2>
  <p>This is the content inside the card.</p>
</Card>
```

### Children with Additional Props

```typescript
interface PanelProps {
  title: string;
  children: React.ReactNode;
  footer?: React.ReactNode;
}

const Panel: FC<PanelProps> = ({ title, children, footer }) => {
  return (
    <div className="panel">
      <div className="panel-header">
        <h3>{title}</h3>
      </div>
      <div className="panel-body">
        {children}
      </div>
      {footer && (
        <div className="panel-footer">
          {footer}
        </div>
      )}
    </div>
  );
};

// Usage
<Panel
  title="Student Information"
  footer={<button>Save Changes</button>}
>
  <p>Name: Alice Johnson</p>
  <p>GPA: 3.85</p>
</Panel>
```

### Layout Components with Children

```typescript
// Container component
interface ContainerProps {
  children: React.ReactNode;
  maxWidth?: 'sm' | 'md' | 'lg' | 'xl';
}

const Container: FC<ContainerProps> = ({ children, maxWidth = 'lg' }) => {
  const maxWidthClass = {
    sm: 'max-w-screen-sm',
    md: 'max-w-screen-md',
    lg: 'max-w-screen-lg',
    xl: 'max-w-screen-xl'
  }[maxWidth];

  return (
    <div className={`container mx-auto px-4 ${maxWidthClass}`}>
      {children}
    </div>
  );
};

// PageLayout component
interface PageLayoutProps {
  children: React.ReactNode;
  sidebar?: React.ReactNode;
}

const PageLayout: FC<PageLayoutProps> = ({ children, sidebar }) => {
  return (
    <div className="flex min-h-screen">
      {sidebar && (
        <aside className="w-64 bg-gray-100 p-4">
          {sidebar}
        </aside>
      )}
      <main className="flex-1 p-8">
        {children}
      </main>
    </div>
  );
};

// Usage
<PageLayout sidebar={<Navigation />}>
  <h1>Dashboard</h1>
  <Container>
    <StudentList students={students} />
  </Container>
</PageLayout>
```

### Render Props Pattern (Alternative to Children)

```typescript
interface DataFetcherProps<T> {
  url: string;
  children: (data: T | null, loading: boolean, error: Error | null) => React.ReactNode;
}

function DataFetcher<T>({ url, children }: DataFetcherProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);

  return <>{children(data, loading, error)}</>;
}

// Usage
<DataFetcher<Student[]> url="/api/students">
  {(students, loading, error) => {
    if (loading) return <p>Loading...</p>;
    if (error) return <p>Error: {error.message}</p>;
    if (!students) return null;
    return <StudentList students={students} />;
  }}
</DataFetcher>
```

---

## Conditional Rendering {#conditional-rendering}

Showing different UI based on conditions is fundamental to dynamic interfaces.

**Analogy**: Conditional rendering is like a traffic light system. Based on conditions (red, yellow, green), different things happen (stop, slow down, go).

### && Operator (Short-Circuit)

```typescript
// Show element only if condition is true
const StudentCard: FC<{ student: Student }> = ({ student }) => {
  const isHonors = student.gpa >= 3.5;

  return (
    <div className="student-card">
      <h3>{student.name}</h3>
      <p>GPA: {student.gpa}</p>

      {/* Show honors badge only for honors students */}
      {isHonors && (
        <span className="badge badge-honors">Honors Student</span>
      )}

      {/* Show warning if no courses */}
      {student.enrolledCourses.length === 0 && (
        <p className="text-red-500">No courses enrolled!</p>
      )}
    </div>
  );
};
```

### Ternary Operator (if-else)

```typescript
const CourseStatus: FC<{ available: number }> = ({ available }) => {
  return (
    <span className={available > 0 ? 'text-green-600' : 'text-red-600'}>
      {available > 0 ? `${available} seats available` : 'Course Full'}
    </span>
  );
};

// More complex ternary
const StudentStatus: FC<{ student: Student }> = ({ student }) => {
  return (
    <div>
      {student.gpa >= 3.5 ? (
        <div className="honors-section">
          <h4>Honors Student</h4>
          <p>Eligible for advanced courses</p>
        </div>
      ) : (
        <div className="regular-section">
          <h4>Regular Student</h4>
          <p>Maintain good academic standing</p>
        </div>
      )}
    </div>
  );
};
```

### If-Else Statements (Early Return)

```typescript
const StudentCard: FC<{ student: Student | null }> = ({ student }) => {
  // Early return for null case
  if (!student) {
    return <p>No student data available</p>;
  }

  // Early return for error case
  if (student.gpa < 0 || student.gpa > 4) {
    return <p className="text-red-500">Invalid student data</p>;
  }

  // Main render
  return (
    <div className="student-card">
      <h3>{student.name}</h3>
      <p>GPA: {student.gpa}</p>
    </div>
  );
};
```

### Switch/Match Pattern

```typescript
type EnrollmentStatus = 'pending' | 'approved' | 'rejected' | 'waitlist';

interface EnrollmentStatusBadgeProps {
  status: EnrollmentStatus;
}

const EnrollmentStatusBadge: FC<EnrollmentStatusBadgeProps> = ({ status }) => {
  // Using object lookup
  const statusConfig = {
    pending: {
      color: 'bg-yellow-100 text-yellow-800',
      icon: '⏳',
      text: 'Pending Approval'
    },
    approved: {
      color: 'bg-green-100 text-green-800',
      icon: '✓',
      text: 'Approved'
    },
    rejected: {
      color: 'bg-red-100 text-red-800',
      icon: '✗',
      text: 'Rejected'
    },
    waitlist: {
      color: 'bg-blue-100 text-blue-800',
      icon: '⏸',
      text: 'Waitlisted'
    }
  }[status];

  return (
    <span className={`px-3 py-1 rounded-full text-sm ${statusConfig.color}`}>
      {statusConfig.icon} {statusConfig.text}
    </span>
  );
};

// Using switch statement
const EnrollmentMessage: FC<{ status: EnrollmentStatus }> = ({ status }) => {
  let message: string;

  switch (status) {
    case 'pending':
      message = 'Your enrollment request is being reviewed.';
      break;
    case 'approved':
      message = 'Congratulations! You are enrolled in this course.';
      break;
    case 'rejected':
      message = 'Unfortunately, your enrollment was not approved.';
      break;
    case 'waitlist':
      message = 'You are on the waitlist. We will notify you if a spot opens.';
      break;
    default:
      message = 'Unknown status';
  }

  return <p>{message}</p>;
};
```

### Complex Conditional Rendering

```typescript
interface DashboardProps {
  user: {
    role: 'student' | 'instructor' | 'admin';
    courses: Course[];
  };
  loading: boolean;
  error: Error | null;
}

const Dashboard: FC<DashboardProps> = ({ user, loading, error }) => {
  // Loading state
  if (loading) {
    return (
      <div className="flex items-center justify-center h-screen">
        <div className="text-center">
          <div className="spinner"></div>
          <p>Loading dashboard...</p>
        </div>
      </div>
    );
  }

  // Error state
  if (error) {
    return (
      <div className="error-container">
        <h2>Something went wrong</h2>
        <p>{error.message}</p>
        <button onClick={() => window.location.reload()}>Retry</button>
      </div>
    );
  }

  // Empty state
  if (user.courses.length === 0) {
    return (
      <div className="empty-state">
        <h2>No Courses Yet</h2>
        <p>Get started by enrolling in your first course!</p>
        <button>Browse Courses</button>
      </div>
    );
  }

  // Main content - different based on role
  return (
    <div className="dashboard">
      <h1>Welcome, {user.role}!</h1>

      {user.role === 'student' && <StudentDashboard courses={user.courses} />}
      {user.role === 'instructor' && <InstructorDashboard courses={user.courses} />}
      {user.role === 'admin' && <AdminDashboard />}
    </div>
  );
};
```

---

## Lists and Keys {#lists-keys}

Rendering lists of data is one of the most common operations in React.

**Analogy**: Think of keys like student ID numbers. When the teacher takes attendance, they use ID numbers to uniquely identify each student, even if two students have the same name. React uses keys the same way to track list items.

### Basic List Rendering

```typescript
interface Student {
  id: string;
  name: string;
  gpa: number;
}

interface StudentListProps {
  students: Student[];
}

const StudentList: FC<StudentListProps> = ({ students }) => {
  return (
    <div className="student-list">
      {students.map(student => (
        <div key={student.id} className="student-item">
          <h3>{student.name}</h3>
          <p>GPA: {student.gpa}</p>
        </div>
      ))}
    </div>
  );
};
```

### Why Keys Matter

```typescript
// ❌ BAD: No key
students.map(student => (
  <StudentCard student={student} />
))

// ❌ BAD: Index as key (only okay for static lists that never reorder)
students.map((student, index) => (
  <StudentCard key={index} student={student} />
))

// ✅ GOOD: Unique ID as key
students.map(student => (
  <StudentCard key={student.id} student={student} />
))

// ✅ GOOD: Composite key if no unique ID
students.map(student => (
  <StudentCard key={`${student.name}-${student.email}`} student={student} />
))
```

### List with Components

```typescript
const StudentList: FC<StudentListProps> = ({ students }) => {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      {students.map(student => (
        <StudentCard
          key={student.id}
          student={student}
          onEdit={(id) => console.log('Edit', id)}
          onDelete={(id) => console.log('Delete', id)}
        />
      ))}
    </div>
  );
};
```

### Filtering and Sorting Lists

```typescript
interface CourseListProps {
  courses: Course[];
  filter?: {
    searchTerm?: string;
    minCredits?: number;
    availableOnly?: boolean;
  };
}

const CourseList: FC<CourseListProps> = ({ courses, filter = {} }) => {
  // Filter courses
  let filteredCourses = courses;

  if (filter.searchTerm) {
    filteredCourses = filteredCourses.filter(course =>
      course.name.toLowerCase().includes(filter.searchTerm!.toLowerCase()) ||
      course.code.toLowerCase().includes(filter.searchTerm!.toLowerCase())
    );
  }

  if (filter.minCredits) {
    filteredCourses = filteredCourses.filter(
      course => course.credits >= filter.minCredits!
    );
  }

  if (filter.availableOnly) {
    filteredCourses = filteredCourses.filter(
      course => course.enrolled < course.capacity
    );
  }

  // Sort courses by name
  filteredCourses = [...filteredCourses].sort((a, b) =>
    a.name.localeCompare(b.name)
  );

  // Empty state
  if (filteredCourses.length === 0) {
    return (
      <div className="text-center py-12">
        <p className="text-gray-500">No courses found matching your criteria</p>
      </div>
    );
  }

  // Render list
  return (
    <div className="course-list">
      <p className="mb-4 text-sm text-gray-600">
        Showing {filteredCourses.length} of {courses.length} courses
      </p>
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        {filteredCourses.map(course => (
          <CourseCard key={course.id} course={course} />
        ))}
      </div>
    </div>
  );
};
```

### Grouped Lists

```typescript
interface GroupedCoursesProps {
  courses: Course[];
}

const GroupedCourses: FC<GroupedCoursesProps> = ({ courses }) => {
  // Group courses by instructor
  const coursesByInstructor = courses.reduce((acc, course) => {
    if (!acc[course.instructor]) {
      acc[course.instructor] = [];
    }
    acc[course.instructor].push(course);
    return acc;
  }, {} as Record<string, Course[]>);

  return (
    <div className="space-y-6">
      {Object.entries(coursesByInstructor).map(([instructor, instructorCourses]) => (
        <div key={instructor} className="bg-white rounded-lg p-6 shadow">
          <h3 className="text-xl font-bold mb-4">{instructor}</h3>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            {instructorCourses.map(course => (
              <CourseCard key={course.id} course={course} />
            ))}
          </div>
        </div>
      ))}
    </div>
  );
};
```

### Nested Lists

```typescript
interface Department {
  id: string;
  name: string;
  courses: Course[];
}

interface DepartmentListProps {
  departments: Department[];
}

const DepartmentList: FC<DepartmentListProps> = ({ departments }) => {
  return (
    <div className="space-y-6">
      {departments.map(department => (
        <div key={department.id} className="bg-white rounded-lg p-6 shadow">
          <h2 className="text-2xl font-bold mb-4">{department.name}</h2>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
            {department.courses.map(course => (
              <CourseCard key={course.id} course={course} />
            ))}
          </div>
        </div>
      ))}
    </div>
  );
};
```

---

## Component Composition {#component-composition}

Component composition is about building complex UIs by combining simpler components.

**Analogy**: Think of composition like building a sandwich. You don't create one massive "Sandwich" component. Instead, you have Bread, Lettuce, Tomato, Cheese components that you stack together to make different types of sandwiches.

### Basic Composition

```typescript
// Small, reusable components
const Avatar: FC<{ src: string; alt: string; size?: 'sm' | 'md' | 'lg' }> = ({
  src,
  alt,
  size = 'md'
}) => {
  const sizeClasses = {
    sm: 'w-8 h-8',
    md: 'w-12 h-12',
    lg: 'w-16 h-16'
  };

  return (
    <img
      src={src}
      alt={alt}
      className={`${sizeClasses[size]} rounded-full object-cover`}
    />
  );
};

const Badge: FC<{ children: React.ReactNode; variant: 'success' | 'warning' | 'error' }> = ({
  children,
  variant
}) => {
  const variantClasses = {
    success: 'bg-green-100 text-green-800',
    warning: 'bg-yellow-100 text-yellow-800',
    error: 'bg-red-100 text-red-800'
  };

  return (
    <span className={`px-2 py-1 rounded-full text-xs font-medium ${variantClasses[variant]}`}>
      {children}
    </span>
  );
};

// Composed component
const StudentProfile: FC<{ student: Student }> = ({ student }) => {
  return (
    <div className="flex items-center gap-4">
      <Avatar
        src={`https://api.dicebear.com/7.x/avataaars/svg?seed=${student.id}`}
        alt={student.name}
        size="lg"
      />
      <div>
        <h3 className="font-bold">{student.name}</h3>
        <p className="text-sm text-gray-600">{student.email}</p>
        <div className="flex gap-2 mt-2">
          {student.gpa >= 3.5 && <Badge variant="success">Honors</Badge>}
          <Badge variant="warning">GPA: {student.gpa}</Badge>
        </div>
      </div>
    </div>
  );
};
```

### Container and Presentational Components

```typescript
// Presentational Component (UI only, no logic)
interface StudentCardViewProps {
  student: Student;
  isHonors: boolean;
  onEnroll: () => void;
  onDrop: () => void;
}

const StudentCardView: FC<StudentCardViewProps> = ({
  student,
  isHonors,
  onEnroll,
  onDrop
}) => {
  return (
    <div className={`student-card ${isHonors ? 'honors' : ''}`}>
      <h3>{student.name}</h3>
      <p>{student.email}</p>
      <p>GPA: {student.gpa}</p>
      <div className="actions">
        <button onClick={onEnroll}>Enroll in Course</button>
        <button onClick={onDrop}>Drop Course</button>
      </div>
    </div>
  );
};

// Container Component (logic, data fetching)
interface StudentCardContainerProps {
  studentId: string;
}

const StudentCardContainer: FC<StudentCardContainerProps> = ({ studentId }) => {
  const [student, setStudent] = useState<Student | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/students/${studentId}`)
      .then(res => res.json())
      .then(setStudent)
      .finally(() => setLoading(false));
  }, [studentId]);

  const handleEnroll = () => {
    console.log('Enrolling student', studentId);
  };

  const handleDrop = () => {
    console.log('Dropping course for student', studentId);
  };

  if (loading) return <p>Loading...</p>;
  if (!student) return <p>Student not found</p>;

  return (
    <StudentCardView
      student={student}
      isHonors={student.gpa >= 3.5}
      onEnroll={handleEnroll}
      onDrop={handleDrop}
    />
  );
};
```

### Compound Components Pattern

```typescript
// Tabs compound component
interface TabsContextValue {
  activeTab: string;
  setActiveTab: (tab: string) => void;
}

const TabsContext = createContext<TabsContextValue | undefined>(undefined);

const Tabs: FC<{ children: React.ReactNode; defaultTab: string }> = ({
  children,
  defaultTab
}) => {
  const [activeTab, setActiveTab] = useState(defaultTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
};

const TabList: FC<{ children: React.ReactNode }> = ({ children }) => {
  return <div className="flex border-b">{children}</div>;
};

const Tab: FC<{ value: string; children: React.ReactNode }> = ({ value, children }) => {
  const context = useContext(TabsContext);
  if (!context) throw new Error('Tab must be used within Tabs');

  const { activeTab, setActiveTab } = context;
  const isActive = activeTab === value;

  return (
    <button
      className={`px-4 py-2 ${isActive ? 'border-b-2 border-blue-500' : ''}`}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
};

const TabPanel: FC<{ value: string; children: React.ReactNode }> = ({ value, children }) => {
  const context = useContext(TabsContext);
  if (!context) throw new Error('TabPanel must be used within Tabs');

  const { activeTab } = context;
  if (activeTab !== value) return null;

  return <div className="p-4">{children}</div>;
};

// Usage
<Tabs defaultTab="profile">
  <TabList>
    <Tab value="profile">Profile</Tab>
    <Tab value="courses">Courses</Tab>
    <Tab value="grades">Grades</Tab>
  </TabList>

  <TabPanel value="profile">
    <StudentProfile student={student} />
  </TabPanel>
  <TabPanel value="courses">
    <CourseList courses={student.enrolledCourses} />
  </TabPanel>
  <TabPanel value="grades">
    <GradeTable grades={student.grades} />
  </TabPanel>
</Tabs>
```

---

## Thinking in React {#thinking-in-react}

A methodology for building React applications.

**The Process:**

1. **Break the UI into components**
2. **Build a static version**
3. **Identify state**
4. **Identify where state should live**
5. **Add inverse data flow**

### Example: Student Enrollment Dashboard

```typescript
// Step 1: Component Hierarchy
/*
StudentDashboard
├── DashboardHeader
├── SearchBar
├── FilterPanel
│   ├── GPAFilter
│   └── StatusFilter
├── StudentList
│   └── StudentCard
│       ├── Avatar
│       ├── StudentInfo
│       └── ActionButtons
└── Pagination
*/

// Step 2: Static Version (no state yet)
const StudentCard: FC<{ student: Student }> = ({ student }) => {
  return (
    <div className="student-card">
      <Avatar src={student.avatar} alt={student.name} />
      <div>
        <h3>{student.name}</h3>
        <p>{student.email}</p>
        <p>GPA: {student.gpa}</p>
      </div>
      <button>View Details</button>
    </div>
  );
};

const StudentList: FC<{ students: Student[] }> = ({ students }) => {
  return (
    <div className="grid grid-cols-3 gap-4">
      {students.map(student => (
        <StudentCard key={student.id} student={student} />
      ))}
    </div>
  );
};

const SearchBar: FC = () => {
  return <input type="text" placeholder="Search students..." />;
};

const FilterPanel: FC = () => {
  return (
    <div>
      <select>
        <option>All Students</option>
        <option>Honors</option>
        <option>Regular</option>
      </select>
    </div>
  );
};

const StudentDashboard: FC<{ students: Student[] }> = ({ students }) => {
  return (
    <div>
      <h1>Student Dashboard</h1>
      <SearchBar />
      <FilterPanel />
      <StudentList students={students} />
    </div>
  );
};

// Step 3 & 4: Add State (in next part - React Hooks)
// We'll cover this in Part 5: React Hooks & State
```

---

## College Portal Examples {#college-examples}

Let's build a complete College Portal with all concepts combined.

### Complete Student Card Component

```typescript
// components/StudentCard.tsx
import { FC } from 'react';

interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
  major: string;
  enrolledCourses: string[];
  avatar?: string;
  status: 'active' | 'inactive' | 'graduated';
}

interface StudentCardProps {
  student: Student;
  onViewDetails?: (id: string) => void;
  onEdit?: (id: string) => void;
  onDelete?: (id: string) => void;
  showActions?: boolean;
}

export const StudentCard: FC<StudentCardProps> = ({
  student,
  onViewDetails,
  onEdit,
  onDelete,
  showActions = true
}) => {
  const isHonors = student.gpa >= 3.5;
  const statusColor = {
    active: 'border-green-500',
    inactive: 'border-gray-500',
    graduated: 'border-blue-500'
  }[student.status];

  return (
    <div className={`bg-white rounded-lg shadow-md p-6 border-l-4 ${statusColor}`}>
      {/* Header */}
      <div className="flex items-start justify-between mb-4">
        <div className="flex items-center gap-3">
          {student.avatar ? (
            <img
              src={student.avatar}
              alt={student.name}
              className="w-12 h-12 rounded-full object-cover"
            />
          ) : (
            <div className="w-12 h-12 rounded-full bg-blue-500 flex items-center justify-center text-white font-bold">
              {student.name.charAt(0)}
            </div>
          )}
          <div>
            <h3 className="text-lg font-bold text-gray-900">{student.name}</h3>
            <p className="text-sm text-gray-600">{student.email}</p>
          </div>
        </div>
        {isHonors && (
          <span className="px-2 py-1 bg-yellow-100 text-yellow-800 text-xs font-medium rounded-full">
            ⭐ Honors
          </span>
        )}
      </div>

      {/* Info */}
      <div className="space-y-2 mb-4">
        <p className="text-sm text-gray-700">
          <span className="font-medium">Major:</span> {student.major}
        </p>
        <p className="text-sm text-gray-700">
          <span className="font-medium">GPA:</span>{' '}
          <span className="text-lg font-bold text-blue-600">
            {student.gpa.toFixed(2)}
          </span>
        </p>
        <p className="text-sm text-gray-700">
          <span className="font-medium">Enrolled:</span>{' '}
          {student.enrolledCourses.length} courses
        </p>
        <p className="text-sm text-gray-700">
          <span className="font-medium">Status:</span>{' '}
          <span className="capitalize">{student.status}</span>
        </p>
      </div>

      {/* Actions */}
      {showActions && (onViewDetails || onEdit || onDelete) && (
        <div className="flex gap-2 pt-4 border-t">
          {onViewDetails && (
            <button
              onClick={() => onViewDetails(student.id)}
              className="flex-1 px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition-colors text-sm font-medium"
            >
              View Details
            </button>
          )}
          {onEdit && (
            <button
              onClick={() => onEdit(student.id)}
              className="px-4 py-2 bg-gray-200 text-gray-700 rounded-md hover:bg-gray-300 transition-colors text-sm font-medium"
            >
              Edit
            </button>
          )}
          {onDelete && (
            <button
              onClick={() => onDelete(student.id)}
              className="px-4 py-2 bg-red-100 text-red-700 rounded-md hover:bg-red-200 transition-colors text-sm font-medium"
            >
              Delete
            </button>
          )}
        </div>
      )}
    </div>
  );
};
```

### Course List Component

```typescript
// components/CourseList.tsx
import { FC } from 'react';
import { CourseCard } from './CourseCard';

interface Course {
  id: string;
  code: string;
  name: string;
  credits: number;
  instructor: string;
  enrolled: number;
  capacity: number;
  schedule: {
    days: string[];
    time: string;
  };
}

interface CourseListProps {
  courses: Course[];
  onEnroll?: (courseId: string) => void;
  emptyMessage?: string;
}

export const CourseList: FC<CourseListProps> = ({
  courses,
  onEnroll,
  emptyMessage = 'No courses available'
}) => {
  if (courses.length === 0) {
    return (
      <div className="text-center py-12 bg-gray-50 rounded-lg">
        <p className="text-gray-500 text-lg">{emptyMessage}</p>
      </div>
    );
  }

  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center mb-4">
        <h2 className="text-2xl font-bold text-gray-900">
          Available Courses
        </h2>
        <p className="text-sm text-gray-600">
          {courses.length} course{courses.length !== 1 ? 's' : ''} found
        </p>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {courses.map(course => (
          <CourseCard
            key={course.id}
            course={course}
            onEnroll={onEnroll}
          />
        ))}
      </div>
    </div>
  );
};
```

### Complete Dashboard Page

```typescript
// pages/StudentDashboard.tsx
import { FC } from 'react';
import { StudentCard } from '@/components/StudentCard';
import { CourseList } from '@/components/CourseList';

interface DashboardProps {
  currentStudent: Student;
  recommendedCourses: Course[];
}

export const StudentDashboard: FC<DashboardProps> = ({
  currentStudent,
  recommendedCourses
}) => {
  const handleEnroll = (courseId: string) => {
    console.log('Enrolling in course:', courseId);
  };

  const handleEditProfile = (id: string) => {
    console.log('Edit profile:', id);
  };

  return (
    <div className="min-h-screen bg-gray-50">
      {/* Header */}
      <header className="bg-white shadow">
        <div className="max-w-7xl mx-auto px-4 py-6">
          <h1 className="text-3xl font-bold text-gray-900">
            Student Dashboard
          </h1>
        </div>
      </header>

      {/* Main Content */}
      <main className="max-w-7xl mx-auto px-4 py-8">
        {/* Student Profile Section */}
        <section className="mb-8">
          <h2 className="text-2xl font-bold mb-4">My Profile</h2>
          <div className="max-w-2xl">
            <StudentCard
              student={currentStudent}
              onEdit={handleEditProfile}
              showActions
            />
          </div>
        </section>

        {/* Enrolled Courses */}
        <section className="mb-8">
          <h2 className="text-2xl font-bold mb-4">
            My Courses ({currentStudent.enrolledCourses.length})
          </h2>
          {currentStudent.enrolledCourses.length > 0 ? (
            <div className="bg-white rounded-lg shadow p-6">
              <ul className="space-y-2">
                {currentStudent.enrolledCourses.map(courseId => (
                  <li key={courseId} className="flex items-center gap-2">
                    <span className="text-green-500">✓</span>
                    <span>{courseId}</span>
                  </li>
                ))}
              </ul>
            </div>
          ) : (
            <p className="text-gray-500">No courses enrolled yet.</p>
          )}
        </section>

        {/* Recommended Courses */}
        <section>
          <h2 className="text-2xl font-bold mb-4">Recommended Courses</h2>
          <CourseList
            courses={recommendedCourses}
            onEnroll={handleEnroll}
            emptyMessage="No recommendations available"
          />
        </section>
      </main>
    </div>
  );
};
```

---

## Summary

React fundamentals covered:

1. **Components**: Reusable pieces of UI
2. **Functional Components**: Modern way using functions
3. **JSX**: HTML-like syntax in JavaScript
4. **Props**: Passing data to components
5. **Children**: Content between component tags
6. **Conditional Rendering**: Showing UI based on conditions
7. **Lists and Keys**: Rendering arrays of data
8. **Composition**: Building complex UIs from simple components
9. **Thinking in React**: Methodology for building apps

**Key Takeaways:**
- Components are like LEGO blocks - small, reusable pieces
- Props flow down from parent to child (one-way data flow)
- Always use keys when rendering lists
- Compose small components to build complex UIs
- TypeScript makes components more reliable and maintainable

---

## Navigation

- **Previous**: [Part 3: Modern JavaScript & ES6+](./03_JavaScript_Modern.md)
- **Next**: [Part 5: React Hooks & State](./05_React_Hooks_State.md)
- **[Back to Index](./README.md)**

---

**Author: Saiprasad Hegde**

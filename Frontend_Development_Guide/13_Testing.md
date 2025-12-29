# Part 13: Testing

**Author: Saiprasad Hegde**

## Table of Contents
1. [Introduction to Testing](#introduction-to-testing)
2. [Testing Fundamentals and Philosophy](#testing-fundamentals-and-philosophy)
3. [Jest Setup and Configuration](#jest-setup-and-configuration)
4. [React Testing Library](#react-testing-library)
   - [Basic Component Testing](#basic-component-testing)
   - [Testing User Interactions](#testing-user-interactions)
   - [Testing Async Behavior](#testing-async-behavior)
   - [Mock Functions](#mock-functions)
5. [Testing Hooks](#testing-hooks)
6. [Mocking API Calls (MSW)](#mocking-api-calls-msw)
7. [Integration Testing](#integration-testing)
8. [E2E Testing with Playwright](#e2e-testing-with-playwright)
9. [Test Coverage](#test-coverage)
10. [TDD Approach](#tdd-approach)
11. [Complete Examples](#complete-examples)
12. [Best Practices](#best-practices)
13. [Common Pitfalls](#common-pitfalls)

---

## Introduction to Testing

Testing ensures your code works as expected and continues to work as you make changes. It's like quality control in a college - making sure every course, enrollment, and grade is processed correctly.

**Real-World Analogy**: Testing is like a professor grading exams:
- **Unit Tests**: Individual quiz questions (test one function)
- **Integration Tests**: Essay questions (test multiple parts working together)
- **E2E Tests**: Final project (test the entire user experience)

In our College Management System, we'll test:
- Student registration forms
- Course enrollment flows
- Grade calculations
- User authentication
- API integrations

---

## Testing Fundamentals and Philosophy

### Types of Tests

1. **Unit Tests**: Test individual functions/components in isolation
2. **Integration Tests**: Test multiple components working together
3. **E2E Tests**: Test complete user flows in a browser
4. **Snapshot Tests**: Test component output doesn't change unexpectedly

### Testing Pyramid

```
       /\
      /  \     E2E Tests (Few, slow, expensive)
     /____\
    /      \   Integration Tests (Some, medium speed)
   /________\
  /          \ Unit Tests (Many, fast, cheap)
 /____________\
```

### The Testing Trophy (Modern Approach)

```
       /\
      /  \     E2E Tests (Some)
     /    \
    / Integ \  Integration Tests (Most - Focus here!)
   /________\
  /  Unit    \ Unit Tests (Some)
 /____________\
  Static Tests (TypeScript, Linting)
```

**Philosophy**: Write tests that give you confidence. Test behavior, not implementation.

---

## Jest Setup and Configuration

Jest is a JavaScript testing framework. It comes pre-configured with Create React App and Next.js.

### Installation

```bash
# For Next.js
npm install -D jest jest-environment-jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event

# For Vite
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

### Jest Configuration (jest.config.js)

```javascript
const nextJest = require('next/jest')

const createJestConfig = nextJest({
  // Provide the path to your Next.js app to load next.config.js and .env files in your test environment
  dir: './',
})

// Add any custom config to be passed to Jest
const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  testEnvironment: 'jest-environment-jsdom',
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/$1',
  },
  collectCoverageFrom: [
    'app/**/*.{js,jsx,ts,tsx}',
    'components/**/*.{js,jsx,ts,tsx}',
    '!**/*.d.ts',
    '!**/node_modules/**',
  ],
}

// createJestConfig is exported this way to ensure that next/jest can load the Next.js config which is async
module.exports = createJestConfig(customJestConfig)
```

### Jest Setup (jest.setup.js)

```javascript
import '@testing-library/jest-dom'

// Mock Next.js router
jest.mock('next/navigation', () => ({
  useRouter() {
    return {
      push: jest.fn(),
      replace: jest.fn(),
      prefetch: jest.fn(),
    }
  },
  useSearchParams() {
    return new URLSearchParams()
  },
  usePathname() {
    return ''
  },
}))

// Mock Next.js Image component
jest.mock('next/image', () => ({
  __esModule: true,
  default: (props) => {
    // eslint-disable-next-line jsx-a11y/alt-text
    return <img {...props} />
  },
}))
```

### package.json Scripts

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

---

## React Testing Library

React Testing Library helps you test components the way users interact with them.

**Philosophy**: "The more your tests resemble the way your software is used, the more confidence they can give you."

### Basic Component Testing

```typescript
// components/StudentCard.tsx
interface StudentCardProps {
  name: string;
  email: string;
  department: string;
  gpa: number;
}

export default function StudentCard({ name, email, department, gpa }: StudentCardProps) {
  return (
    <div data-testid="student-card">
      <h3>{name}</h3>
      <p>{email}</p>
      <p>Department: {department}</p>
      <p>GPA: {gpa.toFixed(2)}</p>
      {gpa >= 3.5 && <span data-testid="honors-badge">Honors</span>}
    </div>
  );
}
```

```typescript
// components/StudentCard.test.tsx
import { render, screen } from '@testing-library/react';
import StudentCard from './StudentCard';

describe('StudentCard', () => {
  it('renders student information correctly', () => {
    render(
      <StudentCard
        name="Alice Johnson"
        email="alice@college.edu"
        department="Computer Science"
        gpa={3.8}
      />
    );

    // Query by text
    expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
    expect(screen.getByText('alice@college.edu')).toBeInTheDocument();
    expect(screen.getByText('Department: Computer Science')).toBeInTheDocument();
    expect(screen.getByText('GPA: 3.80')).toBeInTheDocument();
  });

  it('shows honors badge for GPA >= 3.5', () => {
    render(
      <StudentCard
        name="Alice Johnson"
        email="alice@college.edu"
        department="Computer Science"
        gpa={3.8}
      />
    );

    expect(screen.getByTestId('honors-badge')).toBeInTheDocument();
  });

  it('does not show honors badge for GPA < 3.5', () => {
    render(
      <StudentCard
        name="Bob Smith"
        email="bob@college.edu"
        department="Engineering"
        gpa={3.2}
      />
    );

    expect(screen.queryByTestId('honors-badge')).not.toBeInTheDocument();
  });
});
```

### Query Methods

```typescript
import { render, screen } from '@testing-library/react';

// getBy - Throws error if not found (use for elements that should exist)
const element = screen.getByText('Hello');
const button = screen.getByRole('button', { name: 'Submit' });
const input = screen.getByLabelText('Email');

// queryBy - Returns null if not found (use for elements that might not exist)
const optional = screen.queryByText('Optional Text');
if (optional) {
  // Do something
}

// findBy - Returns promise (use for async elements)
const asyncElement = await screen.findByText('Loaded Data');

// getAllBy - Returns array of all matches
const listItems = screen.getAllByRole('listitem');
expect(listItems).toHaveLength(5);

// Common queries
screen.getByRole('button')           // Accessible role
screen.getByLabelText('Email')       // Form label
screen.getByPlaceholderText('Search')// Placeholder text
screen.getByText('Hello')            // Text content
screen.getByDisplayValue('Value')    // Form input value
screen.getByAltText('Logo')          // Image alt text
screen.getByTitle('Close')           // Title attribute
screen.getByTestId('custom-id')      // data-testid attribute
```

### Testing User Interactions

```typescript
// components/EnrollmentForm.tsx
import { useState } from 'react';

interface EnrollmentFormProps {
  onSubmit: (data: { studentId: string; courseId: string }) => void;
}

export default function EnrollmentForm({ onSubmit }: EnrollmentFormProps) {
  const [studentId, setStudentId] = useState('');
  const [courseId, setCourseId] = useState('');
  const [error, setError] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    if (!studentId || !courseId) {
      setError('Both fields are required');
      return;
    }

    onSubmit({ studentId, courseId });
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="studentId">Student ID</label>
        <input
          id="studentId"
          value={studentId}
          onChange={(e) => setStudentId(e.target.value)}
        />
      </div>

      <div>
        <label htmlFor="courseId">Course ID</label>
        <input
          id="courseId"
          value={courseId}
          onChange={(e) => setCourseId(e.target.value)}
        />
      </div>

      {error && <div role="alert">{error}</div>}

      <button type="submit">Enroll</button>
    </form>
  );
}
```

```typescript
// components/EnrollmentForm.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import EnrollmentForm from './EnrollmentForm';

describe('EnrollmentForm', () => {
  it('calls onSubmit with form data when valid', async () => {
    const user = userEvent.setup();
    const mockOnSubmit = jest.fn();

    render(<EnrollmentForm onSubmit={mockOnSubmit} />);

    // Type into inputs
    await user.type(screen.getByLabelText('Student ID'), 'STU123456');
    await user.type(screen.getByLabelText('Course ID'), 'CS101');

    // Click submit button
    await user.click(screen.getByRole('button', { name: 'Enroll' }));

    // Verify onSubmit was called with correct data
    expect(mockOnSubmit).toHaveBeenCalledWith({
      studentId: 'STU123456',
      courseId: 'CS101',
    });
    expect(mockOnSubmit).toHaveBeenCalledTimes(1);
  });

  it('shows error when submitting with empty fields', async () => {
    const user = userEvent.setup();
    const mockOnSubmit = jest.fn();

    render(<EnrollmentForm onSubmit={mockOnSubmit} />);

    // Click submit without filling fields
    await user.click(screen.getByRole('button', { name: 'Enroll' }));

    // Verify error is shown
    expect(screen.getByRole('alert')).toHaveTextContent('Both fields are required');

    // Verify onSubmit was NOT called
    expect(mockOnSubmit).not.toHaveBeenCalled();
  });

  it('clears error when user starts typing', async () => {
    const user = userEvent.setup();
    const mockOnSubmit = jest.fn();

    render(<EnrollmentForm onSubmit={mockOnSubmit} />);

    // Submit empty form to show error
    await user.click(screen.getByRole('button', { name: 'Enroll' }));
    expect(screen.getByRole('alert')).toBeInTheDocument();

    // Start typing
    await user.type(screen.getByLabelText('Student ID'), 'STU');

    // Error should be cleared
    expect(screen.queryByRole('alert')).not.toBeInTheDocument();
  });
});
```

### Testing Async Behavior

```typescript
// components/StudentList.tsx
import { useState, useEffect } from 'react';

interface Student {
  id: string;
  name: string;
  email: string;
}

export default function StudentList() {
  const [students, setStudents] = useState<Student[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetch('/api/students')
      .then(res => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then(data => {
        setStudents(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div role="alert">Error: {error}</div>;

  return (
    <ul>
      {students.map(student => (
        <li key={student.id}>{student.name}</li>
      ))}
    </ul>
  );
}
```

```typescript
// components/StudentList.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import StudentList from './StudentList';

// Mock fetch
global.fetch = jest.fn();

describe('StudentList', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('shows loading state initially', () => {
    (global.fetch as jest.Mock).mockImplementation(() => new Promise(() => {}));

    render(<StudentList />);

    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });

  it('displays students when fetch succeeds', async () => {
    const mockStudents = [
      { id: '1', name: 'Alice Johnson', email: 'alice@college.edu' },
      { id: '2', name: 'Bob Smith', email: 'bob@college.edu' },
    ];

    (global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => mockStudents,
    });

    render(<StudentList />);

    // Wait for students to appear
    await waitFor(() => {
      expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
    });

    expect(screen.getByText('Bob Smith')).toBeInTheDocument();
    expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
  });

  it('shows error when fetch fails', async () => {
    (global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: false,
    });

    render(<StudentList />);

    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('Error: Failed to fetch');
    });

    expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
  });
});
```

### Mock Functions

```typescript
// Testing callbacks
it('calls onClick when button is clicked', async () => {
  const user = userEvent.setup();
  const mockOnClick = jest.fn();

  render(<Button onClick={mockOnClick}>Click Me</Button>);

  await user.click(screen.getByRole('button'));

  expect(mockOnClick).toHaveBeenCalledTimes(1);
});

// Testing with arguments
it('calls onDelete with student id', async () => {
  const user = userEvent.setup();
  const mockOnDelete = jest.fn();

  render(<StudentCard id="123" onDelete={mockOnDelete} />);

  await user.click(screen.getByRole('button', { name: 'Delete' }));

  expect(mockOnDelete).toHaveBeenCalledWith('123');
});

// Testing async functions
it('calls async submit handler', async () => {
  const user = userEvent.setup();
  const mockSubmit = jest.fn().mockResolvedValue({ success: true });

  render(<Form onSubmit={mockSubmit} />);

  await user.click(screen.getByRole('button', { name: 'Submit' }));

  expect(mockSubmit).toHaveBeenCalled();
  await waitFor(() => {
    expect(screen.getByText('Success!')).toBeInTheDocument();
  });
});
```

---

## Testing Hooks

```typescript
// hooks/useStudents.ts
import { useState, useEffect } from 'react';

interface Student {
  id: string;
  name: string;
}

export function useStudents() {
  const [students, setStudents] = useState<Student[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetch('/api/students')
      .then(res => res.json())
      .then(data => {
        setStudents(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  const addStudent = (student: Student) => {
    setStudents(prev => [...prev, student]);
  };

  return { students, loading, error, addStudent };
}
```

```typescript
// hooks/useStudents.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { act } from 'react-dom/test-utils';
import { useStudents } from './useStudents';

global.fetch = jest.fn();

describe('useStudents', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('fetches students on mount', async () => {
    const mockStudents = [
      { id: '1', name: 'Alice' },
      { id: '2', name: 'Bob' },
    ];

    (global.fetch as jest.Mock).mockResolvedValueOnce({
      json: async () => mockStudents,
    });

    const { result } = renderHook(() => useStudents());

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.students).toEqual(mockStudents);
    expect(result.current.error).toBeNull();
  });

  it('adds student to list', async () => {
    (global.fetch as jest.Mock).mockResolvedValueOnce({
      json: async () => [],
    });

    const { result } = renderHook(() => useStudents());

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    const newStudent = { id: '3', name: 'Charlie' };

    act(() => {
      result.current.addStudent(newStudent);
    });

    expect(result.current.students).toContainEqual(newStudent);
  });
});
```

---

## Mocking API Calls (MSW)

Mock Service Worker (MSW) intercepts network requests for realistic API mocking.

### Setup MSW

```bash
npm install -D msw
```

```typescript
// mocks/handlers.ts
import { rest } from 'msw';

export const handlers = [
  // GET /api/students
  rest.get('/api/students', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json([
        { id: '1', name: 'Alice Johnson', email: 'alice@college.edu', gpa: 3.8 },
        { id: '2', name: 'Bob Smith', email: 'bob@college.edu', gpa: 3.5 },
      ])
    );
  }),

  // POST /api/students
  rest.post('/api/students', async (req, res, ctx) => {
    const newStudent = await req.json();
    return res(
      ctx.status(201),
      ctx.json({ ...newStudent, id: '3' })
    );
  }),

  // DELETE /api/students/:id
  rest.delete('/api/students/:id', (req, res, ctx) => {
    const { id } = req.params;
    return res(ctx.status(204));
  }),

  // Error scenario
  rest.get('/api/courses', (req, res, ctx) => {
    return res(
      ctx.status(500),
      ctx.json({ message: 'Internal server error' })
    );
  }),
];
```

```typescript
// mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```typescript
// jest.setup.js
import { server } from './mocks/server';

// Establish API mocking before all tests
beforeAll(() => server.listen());

// Reset handlers after each test
afterEach(() => server.resetHandlers());

// Clean up after all tests
afterAll(() => server.close());
```

```typescript
// components/StudentList.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { rest } from 'msw';
import { server } from '@/mocks/server';
import StudentList from './StudentList';

describe('StudentList with MSW', () => {
  it('loads and displays students', async () => {
    render(<StudentList />);

    // Initially shows loading
    expect(screen.getByText('Loading...')).toBeInTheDocument();

    // Wait for students to load (MSW will intercept the request)
    await waitFor(() => {
      expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
    });

    expect(screen.getByText('Bob Smith')).toBeInTheDocument();
  });

  it('handles server error', async () => {
    // Override handler for this test
    server.use(
      rest.get('/api/students', (req, res, ctx) => {
        return res(ctx.status(500), ctx.json({ message: 'Server error' }));
      })
    );

    render(<StudentList />);

    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('Server error');
    });
  });
});
```

---

## Integration Testing

Integration tests verify multiple components work together.

```typescript
// app/enrollments/page.tsx
'use client';

import { useState, useEffect } from 'react';
import StudentSelect from '@/components/StudentSelect';
import CourseSelect from '@/components/CourseSelect';
import EnrollmentSummary from '@/components/EnrollmentSummary';

export default function EnrollmentPage() {
  const [selectedStudent, setSelectedStudent] = useState<string | null>(null);
  const [selectedCourse, setSelectedCourse] = useState<string | null>(null);
  const [enrollments, setEnrollments] = useState<Enrollment[]>([]);

  const handleEnroll = async () => {
    if (!selectedStudent || !selectedCourse) return;

    const response = await fetch('/api/enrollments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        studentId: selectedStudent,
        courseId: selectedCourse,
      }),
    });

    if (response.ok) {
      const newEnrollment = await response.json();
      setEnrollments(prev => [...prev, newEnrollment]);
      setSelectedStudent(null);
      setSelectedCourse(null);
    }
  };

  return (
    <div>
      <h1>Course Enrollment</h1>
      <StudentSelect value={selectedStudent} onChange={setSelectedStudent} />
      <CourseSelect value={selectedCourse} onChange={setSelectedCourse} />
      <button onClick={handleEnroll} disabled={!selectedStudent || !selectedCourse}>
        Enroll
      </button>
      <EnrollmentSummary enrollments={enrollments} />
    </div>
  );
}
```

```typescript
// app/enrollments/page.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { rest } from 'msw';
import { server } from '@/mocks/server';
import EnrollmentPage from './page';

describe('EnrollmentPage Integration', () => {
  it('completes full enrollment flow', async () => {
    const user = userEvent.setup();

    // Setup mock handlers
    server.use(
      rest.get('/api/students', (req, res, ctx) => {
        return res(ctx.json([
          { id: '1', name: 'Alice Johnson' },
          { id: '2', name: 'Bob Smith' },
        ]));
      }),
      rest.get('/api/courses', (req, res, ctx) => {
        return res(ctx.json([
          { id: 'CS101', name: 'Intro to CS' },
          { id: 'CS201', name: 'Data Structures' },
        ]));
      }),
      rest.post('/api/enrollments', async (req, res, ctx) => {
        const body = await req.json();
        return res(ctx.json({
          id: '1',
          studentId: body.studentId,
          courseId: body.courseId,
          enrolledAt: new Date().toISOString(),
        }));
      })
    );

    render(<EnrollmentPage />);

    // Wait for students and courses to load
    await waitFor(() => {
      expect(screen.getByRole('combobox', { name: /student/i })).toBeInTheDocument();
    });

    // Select student
    await user.selectOptions(
      screen.getByRole('combobox', { name: /student/i }),
      'Alice Johnson'
    );

    // Select course
    await user.selectOptions(
      screen.getByRole('combobox', { name: /course/i }),
      'Intro to CS'
    );

    // Click enroll button
    await user.click(screen.getByRole('button', { name: /enroll/i }));

    // Verify enrollment appears in summary
    await waitFor(() => {
      expect(screen.getByText(/Alice Johnson/)).toBeInTheDocument();
      expect(screen.getByText(/Intro to CS/)).toBeInTheDocument();
    });

    // Verify form was reset
    expect(screen.getByRole('combobox', { name: /student/i })).toHaveValue('');
    expect(screen.getByRole('combobox', { name: /course/i })).toHaveValue('');
  });
});
```

---

## E2E Testing with Playwright

Playwright tests your app in a real browser, simulating actual user behavior.

### Setup Playwright

```bash
npm init playwright@latest
```

### Basic E2E Test

```typescript
// e2e/enrollment.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Student Enrollment Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Navigate to enrollment page
    await page.goto('http://localhost:3000/enrollments');
  });

  test('user can enroll student in course', async ({ page }) => {
    // Wait for page to load
    await expect(page.getByRole('heading', { name: 'Course Enrollment' })).toBeVisible();

    // Select student
    await page.getByLabel('Student').selectOption('Alice Johnson');

    // Select course
    await page.getByLabel('Course').selectOption('CS101');

    // Click enroll button
    await page.getByRole('button', { name: 'Enroll' }).click();

    // Verify success message
    await expect(page.getByText('Successfully enrolled')).toBeVisible();

    // Verify enrollment appears in list
    await expect(page.getByText('Alice Johnson - CS101')).toBeVisible();
  });

  test('shows error for duplicate enrollment', async ({ page }) => {
    // Enroll once
    await page.getByLabel('Student').selectOption('Alice Johnson');
    await page.getByLabel('Course').selectOption('CS101');
    await page.getByRole('button', { name: 'Enroll' }).click();
    await expect(page.getByText('Successfully enrolled')).toBeVisible();

    // Try to enroll again
    await page.getByLabel('Student').selectOption('Alice Johnson');
    await page.getByLabel('Course').selectOption('CS101');
    await page.getByRole('button', { name: 'Enroll' }).click();

    // Verify error message
    await expect(page.getByRole('alert')).toContainText('Student already enrolled');
  });

  test('form validation works correctly', async ({ page }) => {
    // Try to submit without selections
    await page.getByRole('button', { name: 'Enroll' }).click();

    // Verify button is disabled or error is shown
    await expect(page.getByText('Please select both student and course')).toBeVisible();
  });
});
```

### Advanced E2E Test with Authentication

```typescript
// e2e/admin-dashboard.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Admin Dashboard', () => {
  test.beforeEach(async ({ page }) => {
    // Login as admin
    await page.goto('http://localhost:3000/login');
    await page.getByLabel('Email').fill('admin@college.edu');
    await page.getByLabel('Password').fill('admin123');
    await page.getByRole('button', { name: 'Login' }).click();

    // Wait for redirect to dashboard
    await expect(page).toHaveURL(/.*dashboard/);
  });

  test('admin can view all students', async ({ page }) => {
    await page.goto('http://localhost:3000/admin/students');

    // Wait for students to load
    await expect(page.getByRole('table')).toBeVisible();

    // Verify table has data
    const rows = await page.getByRole('row').count();
    expect(rows).toBeGreaterThan(1); // Header + at least one student
  });

  test('admin can create new course', async ({ page }) => {
    await page.goto('http://localhost:3000/admin/courses/new');

    // Fill form
    await page.getByLabel('Course Name').fill('Advanced React');
    await page.getByLabel('Course Code').fill('CS401');
    await page.getByLabel('Credits').fill('4');
    await page.getByLabel('Instructor').fill('Dr. Smith');

    // Submit
    await page.getByRole('button', { name: 'Create Course' }).click();

    // Verify success and redirect
    await expect(page).toHaveURL(/.*courses$/);
    await expect(page.getByText('Advanced React')).toBeVisible();
  });

  test('admin can delete student', async ({ page }) => {
    await page.goto('http://localhost:3000/admin/students');

    // Get first student name
    const firstStudent = await page.getByRole('row').nth(1).textContent();

    // Click delete button
    await page.getByRole('row').nth(1).getByRole('button', { name: 'Delete' }).click();

    // Confirm deletion
    await page.getByRole('button', { name: 'Confirm' }).click();

    // Verify student is removed
    await expect(page.getByText(firstStudent!)).not.toBeVisible();
  });
});
```

### Page Object Pattern

```typescript
// e2e/pages/LoginPage.ts
import { Page, expect } from '@playwright/test';

export class LoginPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('http://localhost:3000/login');
  }

  async login(email: string, password: string) {
    await this.page.getByLabel('Email').fill(email);
    await this.page.getByLabel('Password').fill(password);
    await this.page.getByRole('button', { name: 'Login' }).click();
  }

  async expectError(message: string) {
    await expect(this.page.getByRole('alert')).toContainText(message);
  }

  async expectRedirectToDashboard() {
    await expect(this.page).toHaveURL(/.*dashboard/);
  }
}

// Usage in test
import { LoginPage } from './pages/LoginPage';

test('user can login', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await loginPage.goto();
  await loginPage.login('student@college.edu', 'password123');
  await loginPage.expectRedirectToDashboard();
});
```

---

## Test Coverage

Track how much of your code is tested.

```bash
# Run tests with coverage
npm run test:coverage
```

**jest.config.js**:
```javascript
module.exports = {
  collectCoverageFrom: [
    'components/**/*.{ts,tsx}',
    'app/**/*.{ts,tsx}',
    'lib/**/*.{ts,tsx}',
    '!**/*.d.ts',
    '!**/node_modules/**',
    '!**/.next/**',
  ],
  coverageThresholds: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

**Interpreting Coverage**:
- **Statements**: % of code statements executed
- **Branches**: % of if/else branches tested
- **Functions**: % of functions called
- **Lines**: % of lines executed

**Aim for**:
- Critical code: 90%+ coverage
- Utilities: 80%+ coverage
- UI components: 70%+ coverage
- Overall: 80%+ coverage

---

## TDD Approach

Test-Driven Development: Write tests before code.

**TDD Cycle (Red-Green-Refactor)**:
1. **Red**: Write a failing test
2. **Green**: Write minimal code to pass
3. **Refactor**: Improve code while keeping tests green

### TDD Example

```typescript
// 1. RED - Write failing test
describe('calculateGPA', () => {
  it('calculates GPA from grades', () => {
    const grades = [
      { course: 'CS101', grade: 'A', credits: 3 },
      { course: 'MATH101', grade: 'B', credits: 4 },
    ];

    expect(calculateGPA(grades)).toBe(3.43);
  });
});

// Test fails - calculateGPA doesn't exist

// 2. GREEN - Write minimal code
function calculateGPA(grades: Grade[]): number {
  const gradePoints = { A: 4.0, B: 3.0, C: 2.0, D: 1.0, F: 0.0 };

  let totalPoints = 0;
  let totalCredits = 0;

  for (const grade of grades) {
    totalPoints += gradePoints[grade.grade] * grade.credits;
    totalCredits += grade.credits;
  }

  return parseFloat((totalPoints / totalCredits).toFixed(2));
}

// Test passes!

// 3. REFACTOR - Improve code
function calculateGPA(grades: Grade[]): number {
  const gradePoints: Record<string, number> = {
    'A': 4.0, 'B': 3.0, 'C': 2.0, 'D': 1.0, 'F': 0.0
  };

  const { totalPoints, totalCredits } = grades.reduce(
    (acc, { grade, credits }) => ({
      totalPoints: acc.totalPoints + gradePoints[grade] * credits,
      totalCredits: acc.totalCredits + credits,
    }),
    { totalPoints: 0, totalCredits: 0 }
  );

  return parseFloat((totalPoints / totalCredits).toFixed(2));
}

// Test still passes! Refactoring successful
```

---

## Complete Examples

### Testing a Complete Feature

```typescript
// features/student-registration/StudentRegistration.tsx
'use client';

import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  department: z.string().min(1),
});

type FormData = z.infer<typeof schema>;

export default function StudentRegistration() {
  const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
  const [errorMessage, setErrorMessage] = useState('');

  const { register, handleSubmit, formState: { errors }, reset } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = async (data: FormData) => {
    setStatus('loading');
    setErrorMessage('');

    try {
      const response = await fetch('/api/students', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });

      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.message);
      }

      setStatus('success');
      reset();
    } catch (error) {
      setStatus('error');
      setErrorMessage(error instanceof Error ? error.message : 'Unknown error');
    }
  };

  return (
    <div>
      <h1>Student Registration</h1>

      {status === 'success' && (
        <div role="alert" data-testid="success-message">
          Student registered successfully!
        </div>
      )}

      {status === 'error' && (
        <div role="alert" data-testid="error-message">
          {errorMessage}
        </div>
      )}

      <form onSubmit={handleSubmit(onSubmit)}>
        <div>
          <label htmlFor="name">Name</label>
          <input id="name" {...register('name')} />
          {errors.name && <span role="alert">{errors.name.message}</span>}
        </div>

        <div>
          <label htmlFor="email">Email</label>
          <input id="email" type="email" {...register('email')} />
          {errors.email && <span role="alert">{errors.email.message}</span>}
        </div>

        <div>
          <label htmlFor="department">Department</label>
          <select id="department" {...register('department')}>
            <option value="">Select Department</option>
            <option value="CS">Computer Science</option>
            <option value="EE">Electrical Engineering</option>
          </select>
          {errors.department && <span role="alert">{errors.department.message}</span>}
        </div>

        <button type="submit" disabled={status === 'loading'}>
          {status === 'loading' ? 'Registering...' : 'Register'}
        </button>
      </form>
    </div>
  );
}
```

```typescript
// features/student-registration/StudentRegistration.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { rest } from 'msw';
import { server } from '@/mocks/server';
import StudentRegistration from './StudentRegistration';

describe('StudentRegistration', () => {
  it('registers student successfully', async () => {
    const user = userEvent.setup();

    server.use(
      rest.post('/api/students', async (req, res, ctx) => {
        const body = await req.json();
        return res(ctx.json({ id: '1', ...body }));
      })
    );

    render(<StudentRegistration />);

    // Fill form
    await user.type(screen.getByLabelText('Name'), 'Alice Johnson');
    await user.type(screen.getByLabelText('Email'), 'alice@college.edu');
    await user.selectOptions(screen.getByLabelText('Department'), 'CS');

    // Submit
    await user.click(screen.getByRole('button', { name: 'Register' }));

    // Verify success
    await waitFor(() => {
      expect(screen.getByTestId('success-message')).toBeInTheDocument();
    });

    // Verify form is reset
    expect(screen.getByLabelText('Name')).toHaveValue('');
  });

  it('shows validation errors', async () => {
    const user = userEvent.setup();

    render(<StudentRegistration />);

    // Submit empty form
    await user.click(screen.getByRole('button', { name: 'Register' }));

    // Verify errors
    await waitFor(() => {
      expect(screen.getAllByRole('alert')).toHaveLength(3);
    });
  });

  it('handles API errors', async () => {
    const user = userEvent.setup();

    server.use(
      rest.post('/api/students', (req, res, ctx) => {
        return res(ctx.status(400), ctx.json({ message: 'Email already exists' }));
      })
    );

    render(<StudentRegistration />);

    // Fill and submit form
    await user.type(screen.getByLabelText('Name'), 'Alice Johnson');
    await user.type(screen.getByLabelText('Email'), 'alice@college.edu');
    await user.selectOptions(screen.getByLabelText('Department'), 'CS');
    await user.click(screen.getByRole('button', { name: 'Register' }));

    // Verify error message
    await waitFor(() => {
      expect(screen.getByTestId('error-message')).toHaveTextContent('Email already exists');
    });
  });
});
```

---

## Best Practices

1. **Test Behavior, Not Implementation**: Test what users see and do
2. **Use Semantic Queries**: getByRole, getByLabelText over getByTestId
3. **Write Descriptive Test Names**: "user can enroll in course" not "test1"
4. **Arrange-Act-Assert**: Setup, perform action, verify result
5. **One Assertion Per Concept**: Multiple expects for same behavior is OK
6. **Mock External Dependencies**: APIs, third-party libraries
7. **Test Error States**: Don't just test happy paths
8. **Keep Tests Independent**: Each test should run alone
9. **Use Test Data Builders**: Create reusable test data
10. **Continuous Integration**: Run tests on every commit

---

## Common Pitfalls

1. **Testing Implementation Details**: Testing state instead of behavior
2. **Too Many Mocks**: Over-mocking makes tests brittle
3. **Flaky Tests**: Tests that randomly fail
4. **Slow Tests**: Tests taking too long to run
5. **Not Testing Edge Cases**: Only testing happy paths
6. **Ignoring Accessibility**: Not using semantic queries
7. **Large Test Files**: Breaking single responsibility
8. **No Test Organization**: Poor describe/it structure
9. **Snapshot Abuse**: Using snapshots for everything
10. **Not Running Tests**: Writing tests but not running them regularly

---

## Next Steps

Now that you understand testing, you're ready for the final part: **Part 14: Build & Deployment**. You'll learn how to deploy your tested, production-ready application to the world!

---

**Navigation:**
- **Previous**: [Part 12: Performance Optimization](12_Performance_Optimization.md)
- **Next**: [Part 14: Build & Deployment](14_Build_Deployment.md)
- **[Back to Index](README.md)**

---

**Author: Saiprasad Hegde**

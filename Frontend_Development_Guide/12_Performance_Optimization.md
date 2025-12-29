# Part 12: Performance Optimization

**Author: Saiprasad Hegde**

## Table of Contents
1. [Introduction to Performance](#introduction-to-performance)
2. [Performance Basics and Metrics](#performance-basics-and-metrics)
3. [React.lazy and Suspense](#reactlazy-and-suspense)
4. [React.memo for Component Memoization](#reactmemo-for-component-memoization)
5. [useMemo for Expensive Calculations](#usememo-for-expensive-calculations)
6. [useCallback for Function Memoization](#usecallback-for-function-memoization)
7. [Next.js Image Component](#nextjs-image-component)
8. [Bundle Analysis](#bundle-analysis)
9. [Lighthouse and Core Web Vitals](#lighthouse-and-core-web-vitals)
10. [Virtual Scrolling](#virtual-scrolling)
11. [Debouncing and Throttling](#debouncing-and-throttling)
12. [Performance Monitoring](#performance-monitoring)
13. [Complete Examples](#complete-examples)
14. [Best Practices](#best-practices)
15. [Common Pitfalls](#common-pitfalls)

---

## Introduction to Performance

Performance optimization is about making your application fast and responsive. Users expect instant feedback, and slow applications lead to abandonment.

**Real-World Analogy**: Think of performance optimization like organizing a college library:
- **Code Splitting**: Don't bring all books at once, fetch sections as needed
- **Memoization**: Remember where you put frequently used books
- **Lazy Loading**: Only retrieve books when students ask for them
- **Virtual Scrolling**: Show only visible shelves, not the entire library

In our College Management System, we need to optimize:
- Loading large student lists
- Image-heavy course catalogs
- Complex enrollment dashboards
- Real-time notifications

---

## Performance Basics and Metrics

### Key Performance Metrics

1. **First Contentful Paint (FCP)**: When first content appears (< 1.8s good)
2. **Largest Contentful Paint (LCP)**: When main content loads (< 2.5s good)
3. **Time to Interactive (TTI)**: When page becomes interactive (< 3.8s good)
4. **Cumulative Layout Shift (CLS)**: Visual stability (< 0.1 good)
5. **First Input Delay (FID)**: Response to first interaction (< 100ms good)

### Measuring Performance

**Using Performance API**:
```typescript
'use client';

import { useEffect } from 'react';

export default function PerformanceMonitor() {
  useEffect(() => {
    // Measure component mount time
    const startTime = performance.now();

    return () => {
      const endTime = performance.now();
      console.log(`Component was mounted for ${endTime - startTime}ms`);
    };
  }, []);

  // Measure specific operations
  const handleExpensiveOperation = () => {
    performance.mark('operation-start');

    // Expensive operation here
    const result = complexCalculation();

    performance.mark('operation-end');
    performance.measure('operation', 'operation-start', 'operation-end');

    const measure = performance.getEntriesByName('operation')[0];
    console.log(`Operation took ${measure.duration}ms`);
  };

  return <div>Performance Monitoring Active</div>;
}
```

**React DevTools Profiler**:
```typescript
import { Profiler } from 'react';

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number,
  startTime: number,
  commitTime: number
) {
  console.log(`${id} (${phase}) took ${actualDuration}ms`);
}

export default function App() {
  return (
    <Profiler id="StudentList" onRender={onRenderCallback}>
      <StudentList />
    </Profiler>
  );
}
```

---

## React.lazy and Suspense

Code splitting allows you to split your code into smaller chunks that are loaded on demand.

**Analogy**: Instead of downloading the entire course catalog at once, download each department's courses when students select that department.

### Basic Code Splitting

```typescript
import { lazy, Suspense } from 'react';

// Lazy load components
const StudentDashboard = lazy(() => import('@/components/StudentDashboard'));
const CourseCatalog = lazy(() => import('@/components/CourseCatalog'));
const EnrollmentForm = lazy(() => import('@/components/EnrollmentForm'));

export default function App() {
  const [activeTab, setActiveTab] = useState('dashboard');

  return (
    <div>
      <nav>
        <button onClick={() => setActiveTab('dashboard')}>Dashboard</button>
        <button onClick={() => setActiveTab('catalog')}>Courses</button>
        <button onClick={() => setActiveTab('enroll')}>Enroll</button>
      </nav>

      <Suspense fallback={<LoadingSpinner />}>
        {activeTab === 'dashboard' && <StudentDashboard />}
        {activeTab === 'catalog' && <CourseCatalog />}
        {activeTab === 'enroll' && <EnrollmentForm />}
      </Suspense>
    </div>
  );
}

function LoadingSpinner() {
  return (
    <div className="flex justify-center items-center h-64">
      <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-500"></div>
    </div>
  );
}
```

### Route-Based Code Splitting (Next.js)

Next.js automatically code splits by routes. Each page is its own chunk.

```typescript
// app/students/page.tsx - Automatically code split
export default function StudentsPage() {
  return <div>Students Page</div>;
}

// app/courses/page.tsx - Separate chunk
export default function CoursesPage() {
  return <div>Courses Page</div>;
}
```

### Named Exports with Lazy

```typescript
import { lazy, Suspense } from 'react';

// For default exports
const StudentCard = lazy(() => import('@/components/StudentCard'));

// For named exports
const StudentList = lazy(() =>
  import('@/components/StudentComponents').then(module => ({
    default: module.StudentList
  }))
);

export default function Students() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <StudentList />
    </Suspense>
  );
}
```

### Nested Suspense Boundaries

```typescript
import { lazy, Suspense } from 'react';

const Header = lazy(() => import('@/components/Header'));
const Sidebar = lazy(() => import('@/components/Sidebar'));
const StudentList = lazy(() => import('@/components/StudentList'));
const CourseList = lazy(() => import('@/components/CourseList'));

export default function Dashboard() {
  return (
    <div>
      {/* Header loads independently */}
      <Suspense fallback={<div>Loading header...</div>}>
        <Header />
      </Suspense>

      <div className="flex">
        {/* Sidebar loads independently */}
        <Suspense fallback={<div>Loading sidebar...</div>}>
          <Sidebar />
        </Suspense>

        <main>
          {/* Main content loads independently */}
          <Suspense fallback={<div>Loading content...</div>}>
            <StudentList />
            <CourseList />
          </Suspense>
        </main>
      </div>
    </div>
  );
}
```

---

## React.memo for Component Memoization

React.memo prevents unnecessary re-renders by memoizing component output.

**Analogy**: If a student's information hasn't changed, don't reprint their ID card.

### Basic Usage

```typescript
import { memo } from 'react';

interface StudentCardProps {
  id: string;
  name: string;
  email: string;
  gpa: number;
}

// Without memo - re-renders every time parent re-renders
function StudentCard({ id, name, email, gpa }: StudentCardProps) {
  console.log(`Rendering ${name}`);
  return (
    <div className="border p-4 rounded">
      <h3>{name}</h3>
      <p>{email}</p>
      <p>GPA: {gpa}</p>
    </div>
  );
}

// With memo - only re-renders when props change
const MemoizedStudentCard = memo(StudentCard);

export default function StudentList() {
  const [filter, setFilter] = useState('');
  const students = [
    { id: '1', name: 'Alice', email: 'alice@college.edu', gpa: 3.8 },
    { id: '2', name: 'Bob', email: 'bob@college.edu', gpa: 3.5 },
  ];

  return (
    <div>
      <input
        type="text"
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter students..."
      />
      {/* Only filtered students re-render when filter changes */}
      {students.map((student) => (
        <MemoizedStudentCard key={student.id} {...student} />
      ))}
    </div>
  );
}
```

### Custom Comparison Function

```typescript
import { memo } from 'react';

interface CourseCardProps {
  course: {
    id: string;
    name: string;
    instructor: string;
    enrolledCount: number;
    lastUpdated: Date;
  };
}

const CourseCard = memo(
  ({ course }: CourseCardProps) => {
    return (
      <div>
        <h3>{course.name}</h3>
        <p>Instructor: {course.instructor}</p>
        <p>Enrolled: {course.enrolledCount}</p>
      </div>
    );
  },
  // Custom comparison - only re-render if important props changed
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    // Return false if props are different (re-render)
    return (
      prevProps.course.id === nextProps.course.id &&
      prevProps.course.name === nextProps.course.name &&
      prevProps.course.instructor === nextProps.course.instructor &&
      prevProps.course.enrolledCount === nextProps.course.enrolledCount
      // Ignore lastUpdated - don't re-render for it
    );
  }
);

export default CourseCard;
```

---

## useMemo for Expensive Calculations

useMemo caches the result of expensive calculations between re-renders.

**Analogy**: Like calculating a student's cumulative GPA once and remembering it, rather than recalculating every time you open their record.

### Basic Usage

```typescript
import { useMemo, useState } from 'react';

interface Student {
  id: string;
  name: string;
  gpa: number;
  department: string;
}

export default function StudentStatistics({ students }: { students: Student[] }) {
  const [filter, setFilter] = useState('');

  // Expensive calculation - only runs when students array changes
  const statistics = useMemo(() => {
    console.log('Calculating statistics...');

    const totalStudents = students.length;
    const averageGPA = students.reduce((sum, s) => sum + s.gpa, 0) / totalStudents;
    const departmentCounts = students.reduce((acc, s) => {
      acc[s.department] = (acc[s.department] || 0) + 1;
      return acc;
    }, {} as Record<string, number>);

    return {
      totalStudents,
      averageGPA: averageGPA.toFixed(2),
      departmentCounts,
      topPerformers: students
        .sort((a, b) => b.gpa - a.gpa)
        .slice(0, 10),
    };
  }, [students]); // Only recalculate when students changes

  // Filtered students - recalculates when filter or students change
  const filteredStudents = useMemo(() => {
    return students.filter(s =>
      s.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [students, filter]);

  return (
    <div>
      <div className="stats">
        <p>Total Students: {statistics.totalStudents}</p>
        <p>Average GPA: {statistics.averageGPA}</p>
        <div>
          <h4>Department Distribution:</h4>
          {Object.entries(statistics.departmentCounts).map(([dept, count]) => (
            <p key={dept}>{dept}: {count}</p>
          ))}
        </div>
      </div>

      <input
        type="text"
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter students..."
      />

      <div>
        {filteredStudents.map(student => (
          <div key={student.id}>{student.name}</div>
        ))}
      </div>
    </div>
  );
}
```

### When to Use useMemo

**Use useMemo when:**
- Calculation is computationally expensive
- Calculating derived data from large arrays
- Creating complex objects/arrays that are used as dependencies
- Preventing expensive re-renders in child components

**Don't use useMemo when:**
- Calculation is simple (x + y)
- Working with small arrays (< 100 items)
- Premature optimization without measuring

```typescript
// Good use of useMemo
const sortedAndFilteredCourses = useMemo(() => {
  return courses
    .filter(c => c.department === selectedDepartment)
    .sort((a, b) => a.name.localeCompare(b.name))
    .map(c => ({
      ...c,
      formattedSchedule: formatSchedule(c.schedule), // Expensive formatting
    }));
}, [courses, selectedDepartment]);

// Bad use of useMemo - calculation is too simple
const fullName = useMemo(() => {
  return `${firstName} ${lastName}`;
}, [firstName, lastName]); // Overkill!

// Just do:
const fullName = `${firstName} ${lastName}`;
```

---

## useCallback for Function Memoization

useCallback memoizes function references to prevent unnecessary re-renders.

**Analogy**: Like having a permanent phone number for the registrar's office instead of changing it daily.

### Basic Usage

```typescript
import { useState, useCallback, memo } from 'react';

interface Student {
  id: string;
  name: string;
}

// Child component wrapped in memo
const StudentCard = memo(({ student, onDelete }: {
  student: Student;
  onDelete: (id: string) => void;
}) => {
  console.log(`Rendering ${student.name}`);
  return (
    <div>
      <h3>{student.name}</h3>
      <button onClick={() => onDelete(student.id)}>Delete</button>
    </div>
  );
});

export default function StudentList() {
  const [students, setStudents] = useState<Student[]>([
    { id: '1', name: 'Alice' },
    { id: '2', name: 'Bob' },
  ]);
  const [filter, setFilter] = useState('');

  // Without useCallback - new function on every render
  // This causes StudentCard to re-render even with memo
  const handleDeleteBad = (id: string) => {
    setStudents(prev => prev.filter(s => s.id !== id));
  };

  // With useCallback - same function reference
  // StudentCard won't re-render unless student prop changes
  const handleDeleteGood = useCallback((id: string) => {
    setStudents(prev => prev.filter(s => s.id !== id));
  }, []); // Empty deps - function never changes

  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
      />
      {students.map(student => (
        <StudentCard
          key={student.id}
          student={student}
          onDelete={handleDeleteGood}
        />
      ))}
    </div>
  );
}
```

### useCallback with Dependencies

```typescript
import { useState, useCallback } from 'react';

export default function CourseEnrollment() {
  const [maxStudents, setMaxStudents] = useState(30);
  const [courses, setCourses] = useState<Course[]>([]);

  // Function depends on maxStudents
  const handleEnroll = useCallback((courseId: string, studentId: string) => {
    setCourses(prev => prev.map(course => {
      if (course.id !== courseId) return course;

      // Use maxStudents from dependency
      if (course.enrolledCount >= maxStudents) {
        alert('Course is full!');
        return course;
      }

      return {
        ...course,
        enrolledCount: course.enrolledCount + 1,
        students: [...course.students, studentId],
      };
    }));
  }, [maxStudents]); // Recreate when maxStudents changes

  return (
    <div>
      <input
        type="number"
        value={maxStudents}
        onChange={(e) => setMaxStudents(Number(e.target.value))}
      />
      {/* Course components use handleEnroll */}
    </div>
  );
}
```

### Combining useMemo and useCallback

```typescript
import { useState, useMemo, useCallback } from 'react';

export default function StudentGradeCalculator() {
  const [students, setStudents] = useState<Student[]>([]);
  const [gradingScale, setGradingScale] = useState('standard');

  // Memoize the grading calculation
  const gradeCalculations = useMemo(() => {
    return students.map(student => ({
      id: student.id,
      letterGrade: calculateLetterGrade(student.gpa, gradingScale),
      rank: calculateRank(student, students),
    }));
  }, [students, gradingScale]);

  // Memoize the update function
  const updateStudentGPA = useCallback((id: string, newGPA: number) => {
    setStudents(prev => prev.map(s =>
      s.id === id ? { ...s, gpa: newGPA } : s
    ));
  }, []);

  return (
    <div>
      {/* UI using gradeCalculations and updateStudentGPA */}
    </div>
  );
}
```

---

## Next.js Image Component

The Next.js Image component automatically optimizes images.

**Analogy**: Like a smart photocopier that automatically adjusts quality and size based on who's viewing the document.

### Basic Usage

```typescript
import Image from 'next/image';

export default function StudentProfile() {
  return (
    <div>
      {/* Basic usage */}
      <Image
        src="/students/avatar.jpg"
        alt="Student Avatar"
        width={200}
        height={200}
        className="rounded-full"
      />

      {/* Remote images require domains in next.config.js */}
      <Image
        src="https://example.com/profile.jpg"
        alt="Profile"
        width={400}
        height={400}
      />

      {/* Fill container (like CSS object-fit) */}
      <div className="relative w-full h-64">
        <Image
          src="/banner.jpg"
          alt="Course Banner"
          fill
          className="object-cover"
        />
      </div>
    </div>
  );
}
```

### Advanced Image Optimization

```typescript
import Image from 'next/image';

export default function CourseCatalog({ courses }: { courses: Course[] }) {
  return (
    <div className="grid grid-cols-3 gap-4">
      {courses.map(course => (
        <div key={course.id} className="relative h-48">
          <Image
            src={course.imageUrl}
            alt={course.name}
            fill
            className="object-cover rounded-lg"
            sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
            priority={course.featured} // Load first
            placeholder="blur"
            blurDataURL={course.blurDataUrl}
          />
        </div>
      ))}
    </div>
  );
}
```

**next.config.js**:
```javascript
module.exports = {
  images: {
    domains: ['example.com', 'cdn.college.edu'],
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
};
```

---

## Bundle Analysis

Analyze your bundle size to identify large dependencies.

### Setup webpack-bundle-analyzer

```bash
npm install -D @next/bundle-analyzer
```

**next.config.js**:
```javascript
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // Your Next.js config
});
```

**package.json**:
```json
{
  "scripts": {
    "analyze": "ANALYZE=true npm run build"
  }
}
```

Run: `npm run analyze` to see visual bundle breakdown.

### Reducing Bundle Size

```typescript
// Bad - imports entire library
import _ from 'lodash';
const result = _.debounce(fn, 300);

// Good - imports only what you need
import debounce from 'lodash/debounce';
const result = debounce(fn, 300);

// Bad - importing large date library
import moment from 'moment';
const date = moment().format('YYYY-MM-DD');

// Good - using native or smaller alternative
import { format } from 'date-fns';
const date = format(new Date(), 'yyyy-MM-dd');
```

---

## Lighthouse and Core Web Vitals

Lighthouse audits your app's performance, accessibility, SEO, and more.

### Running Lighthouse

1. Open Chrome DevTools
2. Go to Lighthouse tab
3. Click "Analyze page load"
4. Review scores and suggestions

### Measuring Core Web Vitals in Code

```typescript
'use client';

import { useEffect } from 'react';

export default function WebVitalsReporter() {
  useEffect(() => {
    if (typeof window !== 'undefined') {
      import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
        getCLS(console.log); // Cumulative Layout Shift
        getFID(console.log); // First Input Delay
        getFCP(console.log); // First Contentful Paint
        getLCP(console.log); // Largest Contentful Paint
        getTTFB(console.log); // Time to First Byte
      });
    }
  }, []);

  return null;
}
```

**Next.js Built-in Web Vitals**:
```typescript
// app/layout.tsx
export function reportWebVitals(metric: any) {
  console.log(metric);

  // Send to analytics
  if (metric.label === 'web-vital') {
    fetch('/api/analytics', {
      method: 'POST',
      body: JSON.stringify(metric),
    });
  }
}
```

---

## Virtual Scrolling

Virtual scrolling renders only visible items in long lists.

**Analogy**: Like a college bookstore showing only the books on visible shelves, not loading the entire warehouse inventory.

### Using react-window

```bash
npm install react-window
```

```typescript
import { FixedSizeList } from 'react-window';

interface Student {
  id: string;
  name: string;
  email: string;
  department: string;
}

export default function VirtualizedStudentList({ students }: { students: Student[] }) {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => {
    const student = students[index];
    return (
      <div style={style} className="border-b p-4 flex items-center">
        <div className="flex-1">
          <h3 className="font-semibold">{student.name}</h3>
          <p className="text-sm text-gray-600">{student.email}</p>
          <p className="text-sm text-gray-500">{student.department}</p>
        </div>
      </div>
    );
  };

  return (
    <FixedSizeList
      height={600} // Visible height
      itemCount={students.length}
      itemSize={100} // Each row height
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

### Using @tanstack/react-virtual

```bash
npm install @tanstack/react-virtual
```

```typescript
import { useVirtualizer } from '@tanstack/react-virtual';
import { useRef } from 'react';

export default function EnhancedVirtualList({ students }: { students: Student[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: students.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 100, // Estimated row height
    overscan: 5, // Render 5 extra items above/below viewport
  });

  return (
    <div
      ref={parentRef}
      className="h-[600px] overflow-auto border rounded"
    >
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          width: '100%',
          position: 'relative',
        }}
      >
        {virtualizer.getVirtualItems().map((virtualRow) => {
          const student = students[virtualRow.index];
          return (
            <div
              key={virtualRow.key}
              style={{
                position: 'absolute',
                top: 0,
                left: 0,
                width: '100%',
                height: `${virtualRow.size}px`,
                transform: `translateY(${virtualRow.start}px)`,
              }}
              className="border-b p-4"
            >
              <h3 className="font-semibold">{student.name}</h3>
              <p className="text-sm">{student.email}</p>
            </div>
          );
        })}
      </div>
    </div>
  );
}
```

---

## Debouncing and Throttling

Control how often functions execute to improve performance.

**Analogy**:
- **Debouncing**: Like waiting for a student to finish typing before spell-checking
- **Throttling**: Like limiting elevator rides to once per minute during peak hours

### Debouncing (Search Input)

```typescript
import { useState, useEffect, useCallback } from 'react';

function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}

export default function StudentSearch() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);

  useEffect(() => {
    if (debouncedSearchTerm) {
      // Only search after user stops typing for 500ms
      fetch(`/api/students/search?q=${debouncedSearchTerm}`)
        .then(res => res.json())
        .then(data => console.log(data));
    }
  }, [debouncedSearchTerm]);

  return (
    <input
      type="text"
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search students..."
      className="w-full px-4 py-2 border rounded"
    />
  );
}
```

### Throttling (Scroll Event)

```typescript
import { useEffect, useRef, useCallback } from 'react';

function useThrottle(callback: Function, delay: number) {
  const lastRan = useRef(Date.now());

  return useCallback((...args: any[]) => {
    if (Date.now() - lastRan.current >= delay) {
      callback(...args);
      lastRan.current = Date.now();
    }
  }, [callback, delay]);
}

export default function InfiniteScrollList() {
  const [page, setPage] = useState(1);
  const [students, setStudents] = useState<Student[]>([]);

  const handleScroll = useThrottle(() => {
    if (window.innerHeight + window.scrollY >= document.body.offsetHeight - 500) {
      // Load more when near bottom
      setPage(prev => prev + 1);
    }
  }, 300); // Execute at most once every 300ms

  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [handleScroll]);

  useEffect(() => {
    fetch(`/api/students?page=${page}`)
      .then(res => res.json())
      .then(data => setStudents(prev => [...prev, ...data]));
  }, [page]);

  return (
    <div>
      {students.map(student => (
        <div key={student.id}>{student.name}</div>
      ))}
    </div>
  );
}
```

---

## Performance Monitoring

Monitor real-world performance of your production app.

### Using Vercel Analytics

```bash
npm install @vercel/analytics
```

```typescript
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

### Custom Performance Tracking

```typescript
// lib/analytics.ts
export function trackPerformance(metric: {
  name: string;
  value: number;
  rating: 'good' | 'needs-improvement' | 'poor';
}) {
  // Send to your analytics service
  fetch('/api/analytics/performance', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(metric),
  });
}

// Usage
import { trackPerformance } from '@/lib/analytics';

export default function StudentList() {
  useEffect(() => {
    const startTime = performance.now();

    // Load data
    fetchStudents().then(() => {
      const loadTime = performance.now() - startTime;
      trackPerformance({
        name: 'student-list-load',
        value: loadTime,
        rating: loadTime < 1000 ? 'good' : loadTime < 2500 ? 'needs-improvement' : 'poor',
      });
    });
  }, []);
}
```

---

## Complete Examples

### Optimized Student Dashboard

```typescript
'use client';

import { memo, useMemo, useCallback, lazy, Suspense } from 'react';
import { useVirtualizer } from '@tanstack/react-virtual';
import Image from 'next/image';
import { useDebounce } from '@/hooks/useDebounce';

// Lazy load heavy components
const StudentDetailsModal = lazy(() => import('@/components/StudentDetailsModal'));
const EnrollmentChart = lazy(() => import('@/components/EnrollmentChart'));

interface Student {
  id: string;
  name: string;
  email: string;
  avatar: string;
  department: string;
  gpa: number;
  enrolledCourses: number;
}

// Memoized student card
const StudentCard = memo(({ student, onSelect }: {
  student: Student;
  onSelect: (student: Student) => void;
}) => {
  return (
    <div
      className="border p-4 rounded-lg hover:shadow-lg transition-shadow cursor-pointer"
      onClick={() => onSelect(student)}
    >
      <div className="flex items-center gap-4">
        <Image
          src={student.avatar}
          alt={student.name}
          width={48}
          height={48}
          className="rounded-full"
        />
        <div className="flex-1">
          <h3 className="font-semibold">{student.name}</h3>
          <p className="text-sm text-gray-600">{student.email}</p>
        </div>
        <div className="text-right">
          <p className="text-lg font-bold">{student.gpa.toFixed(2)}</p>
          <p className="text-sm text-gray-500">{student.enrolledCourses} courses</p>
        </div>
      </div>
    </div>
  );
});

export default function OptimizedStudentDashboard({ students }: { students: Student[] }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [selectedStudent, setSelectedStudent] = useState<Student | null>(null);
  const [departmentFilter, setDepartmentFilter] = useState<string>('all');
  const parentRef = useRef<HTMLDivElement>(null);

  // Debounce search
  const debouncedSearch = useDebounce(searchTerm, 300);

  // Memoize filtered and sorted students
  const filteredStudents = useMemo(() => {
    return students
      .filter(student => {
        const matchesSearch = student.name.toLowerCase().includes(debouncedSearch.toLowerCase()) ||
                            student.email.toLowerCase().includes(debouncedSearch.toLowerCase());
        const matchesDepartment = departmentFilter === 'all' || student.department === departmentFilter;
        return matchesSearch && matchesDepartment;
      })
      .sort((a, b) => b.gpa - a.gpa); // Sort by GPA descending
  }, [students, debouncedSearch, departmentFilter]);

  // Memoize statistics
  const statistics = useMemo(() => {
    return {
      total: filteredStudents.length,
      averageGPA: (filteredStudents.reduce((sum, s) => sum + s.gpa, 0) / filteredStudents.length).toFixed(2),
      totalEnrollments: filteredStudents.reduce((sum, s) => sum + s.enrolledCourses, 0),
      departments: Array.from(new Set(students.map(s => s.department))),
    };
  }, [filteredStudents, students]);

  // Memoize selection handler
  const handleSelectStudent = useCallback((student: Student) => {
    setSelectedStudent(student);
  }, []);

  // Virtual scrolling for large lists
  const virtualizer = useVirtualizer({
    count: filteredStudents.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 100,
    overscan: 5,
  });

  return (
    <div className="container mx-auto p-6">
      {/* Statistics Cards */}
      <div className="grid grid-cols-1 md:grid-cols-4 gap-4 mb-6">
        <div className="bg-white p-6 rounded-lg shadow">
          <p className="text-sm text-gray-600">Total Students</p>
          <p className="text-3xl font-bold">{statistics.total}</p>
        </div>
        <div className="bg-white p-6 rounded-lg shadow">
          <p className="text-sm text-gray-600">Average GPA</p>
          <p className="text-3xl font-bold">{statistics.averageGPA}</p>
        </div>
        <div className="bg-white p-6 rounded-lg shadow">
          <p className="text-sm text-gray-600">Total Enrollments</p>
          <p className="text-3xl font-bold">{statistics.totalEnrollments}</p>
        </div>
        <div className="bg-white p-6 rounded-lg shadow">
          <p className="text-sm text-gray-600">Departments</p>
          <p className="text-3xl font-bold">{statistics.departments.length}</p>
        </div>
      </div>

      {/* Filters */}
      <div className="bg-white p-4 rounded-lg shadow mb-6 flex gap-4">
        <input
          type="text"
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          placeholder="Search students..."
          className="flex-1 px-4 py-2 border rounded"
        />
        <select
          value={departmentFilter}
          onChange={(e) => setDepartmentFilter(e.target.value)}
          className="px-4 py-2 border rounded"
        >
          <option value="all">All Departments</option>
          {statistics.departments.map(dept => (
            <option key={dept} value={dept}>{dept}</option>
          ))}
        </select>
      </div>

      {/* Lazy loaded chart */}
      <Suspense fallback={<div>Loading chart...</div>}>
        <EnrollmentChart students={filteredStudents} />
      </Suspense>

      {/* Virtual scrolling list */}
      <div
        ref={parentRef}
        className="h-[600px] overflow-auto bg-white rounded-lg shadow"
      >
        <div
          style={{
            height: `${virtualizer.getTotalSize()}px`,
            width: '100%',
            position: 'relative',
          }}
        >
          {virtualizer.getVirtualItems().map((virtualRow) => {
            const student = filteredStudents[virtualRow.index];
            return (
              <div
                key={virtualRow.key}
                style={{
                  position: 'absolute',
                  top: 0,
                  left: 0,
                  width: '100%',
                  height: `${virtualRow.size}px`,
                  transform: `translateY(${virtualRow.start}px)`,
                }}
              >
                <StudentCard student={student} onSelect={handleSelectStudent} />
              </div>
            );
          })}
        </div>
      </div>

      {/* Lazy loaded modal */}
      {selectedStudent && (
        <Suspense fallback={<div>Loading details...</div>}>
          <StudentDetailsModal
            student={selectedStudent}
            onClose={() => setSelectedStudent(null)}
          />
        </Suspense>
      )}
    </div>
  );
}
```

---

## Best Practices

1. **Measure First**: Don't optimize without measuring
2. **Code Split Routes**: Every route should be a separate chunk
3. **Lazy Load Heavy Components**: Charts, modals, rarely used features
4. **Memoize Expensive Calculations**: But don't over-memoize
5. **Use Next.js Image**: Automatic optimization
6. **Virtual Scrolling**: For lists > 100 items
7. **Debounce User Input**: Search, autocomplete
8. **Bundle Analysis**: Regularly check bundle size
9. **Monitor Production**: Track real-world performance
10. **Progressive Enhancement**: Core features work without JS

---

## Common Pitfalls

1. **Premature Optimization**: Optimizing before measuring
2. **Over-Memoization**: Using memo/useMemo everywhere
3. **Ignoring Dependencies**: Stale closures in useCallback
4. **Large Bundles**: Importing entire libraries
5. **No Code Splitting**: Shipping everything upfront
6. **Unoptimized Images**: Using regular img tags
7. **No Virtual Scrolling**: Rendering 1000s of items
8. **Synchronous Operations**: Blocking the main thread
9. **No Performance Monitoring**: Flying blind in production
10. **Ignoring Mobile**: Only testing on desktop

---

## Next Steps

Now that you understand performance optimization, you're ready to learn about **Part 13: Testing**. You'll learn how to ensure your optimized code actually works correctly!

---

**Navigation:**
- **Previous**: [Part 11: State Management](11_State_Management.md)
- **Next**: [Part 13: Testing](13_Testing.md)
- **[Back to Index](README.md)**

---

**Author: Saiprasad Hegde**

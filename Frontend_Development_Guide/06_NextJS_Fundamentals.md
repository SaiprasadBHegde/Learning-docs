# Part 6: Next.js Fundamentals

**Author: Saiprasad Hegde**

## Table of Contents
1. [What is Next.js?](#what-is-nextjs)
2. [Next.js vs React](#nextjs-vs-react)
3. [App Router vs Pages Router](#routers)
4. [File-Based Routing](#file-based-routing)
5. [Server Components vs Client Components](#server-client-components)
6. [Layouts and Templates](#layouts-templates)
7. [Loading States](#loading-states)
8. [Error Handling](#error-handling)
9. [Metadata and SEO](#metadata-seo)
10. [College Portal Example](#college-example)

---

## What is Next.js? {#what-is-nextjs}

Next.js is a React framework that provides additional features like server-side rendering, routing, and optimization out of the box.

**Analogy**: If React is like building blocks, Next.js is like a complete construction kit with instructions, tools, and pre-made templates. You still use React blocks, but Next.js gives you a better structure and extra tools.

### Key Features

```typescript
// React (Plain)
// - Client-side rendering only
// - Manual routing setup (React Router)
// - Manual SEO optimization
// - Manual image optimization
// - Manual code splitting

// Next.js (Framework)
// - Server-side rendering (SSR)
// - Static generation (SSG)
// - File-based routing (automatic)
// - Built-in SEO support
// - Automatic image optimization
// - Automatic code splitting
// - API routes
// - Middleware
```

### Why Use Next.js for College Portal?

```typescript
// Benefits for our College Management System:

// 1. SEO - Course catalog pages can be indexed by Google
// 2. Performance - Fast initial page loads for students
// 3. Easy routing - /students/[id], /courses/[courseId]
// 4. API routes - Backend endpoints in the same project
// 5. Image optimization - Student photos, course images
// 6. TypeScript - Built-in support
```

---

## Next.js vs React {#nextjs-vs-react}

### React App Structure

```typescript
// src/App.tsx (Plain React)
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/students" element={<Students />} />
        <Route path="/students/:id" element={<StudentDetail />} />
        <Route path="/courses" element={<Courses />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### Next.js App Structure

```typescript
// app/layout.tsx (Next.js - automatic routing)
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}

// File structure creates routes automatically:
// app/page.tsx → /
// app/students/page.tsx → /students
// app/students/[id]/page.tsx → /students/:id
// app/courses/page.tsx → /courses
```

---

## App Router vs Pages Router {#routers}

Next.js has two routing systems. We'll focus on **App Router** (modern, Next.js 13+).

### Pages Router (Old - Next.js 12 and below)

```typescript
// pages/index.tsx
export default function Home() {
  return <h1>Home Page</h1>;
}

// pages/students/[id].tsx
import { useRouter } from 'next/router';

export default function StudentPage() {
  const router = useRouter();
  const { id } = router.query;
  return <h1>Student {id}</h1>;
}
```

### App Router (Modern - Next.js 13+)

```typescript
// app/page.tsx
export default function Home() {
  return <h1>Home Page</h1>;
}

// app/students/[id]/page.tsx
export default function StudentPage({ params }: { params: { id: string } }) {
  return <h1>Student {params.id}</h1>;
}
```

**Use App Router for all new projects!**

---

## File-Based Routing {#file-based-routing}

Next.js automatically creates routes based on file structure in the `app/` directory.

### Basic Routes

```typescript
// Folder structure → URL routes

app/
├── page.tsx                  → /
├── about/
│   └── page.tsx             → /about
├── students/
│   ├── page.tsx             → /students
│   └── [id]/
│       └── page.tsx         → /students/:id
└── courses/
    ├── page.tsx             → /courses
    └── [courseId]/
        ├── page.tsx         → /courses/:courseId
        └── enroll/
            └── page.tsx     → /courses/:courseId/enroll
```

### Special Files

```typescript
// app/layout.tsx - Layout for all pages
// app/loading.tsx - Loading UI
// app/error.tsx - Error UI
// app/not-found.tsx - 404 page
// app/page.tsx - Page content
```

### Dynamic Routes

```typescript
// app/students/[id]/page.tsx
interface PageProps {
  params: { id: string };
  searchParams: { [key: string]: string | string[] | undefined };
}

export default function StudentPage({ params, searchParams }: PageProps) {
  // params.id → the dynamic segment value
  // searchParams → query string parameters

  return (
    <div>
      <h1>Student ID: {params.id}</h1>
      {searchParams.tab && <p>Active tab: {searchParams.tab}</p>}
    </div>
  );
}

// URL: /students/STU001?tab=courses
// params = { id: "STU001" }
// searchParams = { tab: "courses" }
```

### Catch-All Routes

```typescript
// app/docs/[...slug]/page.tsx
// Matches: /docs/a, /docs/a/b, /docs/a/b/c

export default function DocsPage({ params }: { params: { slug: string[] } }) {
  return (
    <div>
      <h1>Path: {params.slug.join('/')}</h1>
    </div>
  );
}

// URL: /docs/getting-started/installation
// params.slug = ["getting-started", "installation"]
```

### Optional Catch-All Routes

```typescript
// app/courses/[[...filters]]/page.tsx
// Matches: /courses, /courses/cs, /courses/cs/advanced

export default function CoursesPage({
  params
}: {
  params: { filters?: string[] };
}) {
  const filters = params.filters || [];

  return (
    <div>
      <h1>Courses</h1>
      {filters.length > 0 && <p>Filters: {filters.join(' > ')}</p>}
    </div>
  );
}
```

### Route Groups

```typescript
// Route groups organize files without affecting URL

app/
├── (auth)/              ← Group (not in URL)
│   ├── login/
│   │   └── page.tsx    → /login
│   └── register/
│       └── page.tsx    → /register
├── (dashboard)/         ← Group (not in URL)
│   ├── students/
│   │   └── page.tsx    → /students
│   └── courses/
│       └── page.tsx    → /courses
└── page.tsx            → /

// Each group can have its own layout.tsx
```

### Parallel Routes

```typescript
// app/dashboard/@students/page.tsx
// app/dashboard/@courses/page.tsx
// app/dashboard/layout.tsx

export default function DashboardLayout({
  students,
  courses
}: {
  students: React.ReactNode;
  courses: React.ReactNode;
}) {
  return (
    <div className="grid grid-cols-2 gap-4">
      <div>{students}</div>
      <div>{courses}</div>
    </div>
  );
}
```

---

## Server Components vs Client Components {#server-client-components}

Next.js 13+ uses **React Server Components** by default.

**Analogy**:
- **Server Components** = Food prepared in the restaurant kitchen (server). You get it ready to eat.
- **Client Components** = DIY meal kit. You prepare it at home (browser).

### Server Components (Default)

```typescript
// app/students/page.tsx
// This is a Server Component by default (no "use client")

interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
}

async function getStudents(): Promise<Student[]> {
  // This runs on the SERVER
  const res = await fetch('https://api.college.edu/students', {
    cache: 'no-store' // Always fresh data
  });
  return res.json();
}

export default async function StudentsPage() {
  // async component - can await directly!
  const students = await getStudents();

  return (
    <div>
      <h1>Students</h1>
      {students.map(student => (
        <div key={student.id}>
          <h3>{student.name}</h3>
          <p>{student.email}</p>
        </div>
      ))}
    </div>
  );
}

// Benefits:
// - Data fetching on server (fast, secure)
// - Smaller JavaScript bundle
// - Direct database access
// - SEO-friendly
```

### Client Components

```typescript
// components/EnrollmentForm.tsx
'use client'; // This directive makes it a Client Component

import { useState } from 'react';

export default function EnrollmentForm() {
  // Client-side state
  const [formData, setFormData] = useState({
    studentId: '',
    courseId: ''
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    // Client-side logic
    console.log('Submitting:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.studentId}
        onChange={(e) => setFormData({ ...formData, studentId: e.target.value })}
        placeholder="Student ID"
      />
      <button type="submit">Enroll</button>
    </form>
  );
}

// Use Client Components when you need:
// - useState, useEffect, other hooks
// - Event handlers (onClick, onChange)
// - Browser APIs (localStorage, window)
// - React Context
```

### When to Use Each

```typescript
// ✅ Server Component - Use for:
// - Data fetching
// - Direct database access
// - Accessing backend resources
// - Keeping sensitive data on server
// - Large dependencies (reduce bundle size)

// ✅ Client Component - Use for:
// - Interactivity (state, event handlers)
// - Browser APIs
// - React hooks
// - Real-time features
```

### Mixing Server and Client Components

```typescript
// app/students/page.tsx (Server Component)
import StudentCard from '@/components/StudentCard'; // Client Component

async function getStudents() {
  const res = await fetch('https://api.college.edu/students');
  return res.json();
}

export default async function StudentsPage() {
  const students = await getStudents(); // Server-side fetch

  return (
    <div>
      <h1>Students</h1>
      {students.map((student: any) => (
        <StudentCard key={student.id} student={student} />
      ))}
    </div>
  );
}

// components/StudentCard.tsx (Client Component)
'use client';

import { useState } from 'react';

export default function StudentCard({ student }: { student: any }) {
  const [expanded, setExpanded] = useState(false);

  return (
    <div onClick={() => setExpanded(!expanded)}>
      <h3>{student.name}</h3>
      {expanded && <p>{student.email}</p>}
    </div>
  );
}
```

---

## Layouts and Templates {#layouts-templates}

Layouts wrap pages with shared UI (navigation, footers).

### Root Layout

```typescript
// app/layout.tsx (Required root layout)
import './globals.css';
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export const metadata = {
  title: 'College Portal',
  description: 'Student and course management system'
};

export default function RootLayout({
  children
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <header className="bg-gray-900 text-white p-4">
          <h1>College Portal</h1>
        </header>
        <main className="container mx-auto p-4">
          {children}
        </main>
        <footer className="bg-gray-100 p-4 text-center">
          © 2024 College Portal
        </footer>
      </body>
    </html>
  );
}
```

### Nested Layouts

```typescript
// app/dashboard/layout.tsx
import { Sidebar } from '@/components/Sidebar';

export default function DashboardLayout({
  children
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex">
      <Sidebar />
      <div className="flex-1 p-8">
        {children}
      </div>
    </div>
  );
}

// app/dashboard/students/page.tsx
export default function StudentsPage() {
  return <h1>Students</h1>;
}

// Result: RootLayout > DashboardLayout > StudentsPage
```

### Templates (Re-render on Navigation)

```typescript
// app/dashboard/template.tsx
// Templates create a new instance on each navigation
// Layouts persist and don't re-render

export default function DashboardTemplate({
  children
}: {
  children: React.ReactNode;
}) {
  // This component re-renders on every navigation
  console.log('Template rendered');

  return (
    <div className="animate-fadeIn">
      {children}
    </div>
  );
}

// Use templates when you need:
// - Enter/exit animations
// - Features that rely on useEffect
// - Resetting state on navigation
```

### Shared Navigation Component

```typescript
// components/Navigation.tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';

const navLinks = [
  { href: '/', label: 'Home' },
  { href: '/students', label: 'Students' },
  { href: '/courses', label: 'Courses' },
  { href: '/about', label: 'About' }
];

export function Navigation() {
  const pathname = usePathname();

  return (
    <nav className="flex gap-4">
      {navLinks.map(link => {
        const isActive = pathname === link.href;

        return (
          <Link
            key={link.href}
            href={link.href}
            className={`px-3 py-2 rounded ${
              isActive ? 'bg-blue-600 text-white' : 'hover:bg-gray-100'
            }`}
          >
            {link.label}
          </Link>
        );
      })}
    </nav>
  );
}
```

---

## Loading States {#loading-states}

Next.js provides built-in loading UI with `loading.tsx`.

### Basic Loading State

```typescript
// app/students/loading.tsx
export default function Loading() {
  return (
    <div className="flex items-center justify-center h-screen">
      <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600"></div>
      <p className="ml-4">Loading students...</p>
    </div>
  );
}

// app/students/page.tsx (Server Component)
async function getStudents() {
  await new Promise(resolve => setTimeout(resolve, 2000)); // Simulate delay
  const res = await fetch('https://api.college.edu/students');
  return res.json();
}

export default async function StudentsPage() {
  const students = await getStudents();
  // While fetching, loading.tsx is shown automatically
  return <div>{/* students list */}</div>;
}
```

### Skeleton Loading

```typescript
// app/students/loading.tsx
export default function StudentsLoading() {
  return (
    <div className="space-y-4">
      <div className="h-8 w-48 bg-gray-200 rounded animate-pulse"></div>
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        {[...Array(6)].map((_, i) => (
          <div key={i} className="bg-white p-6 rounded-lg shadow">
            <div className="h-6 bg-gray-200 rounded animate-pulse mb-4"></div>
            <div className="h-4 bg-gray-200 rounded animate-pulse mb-2"></div>
            <div className="h-4 bg-gray-200 rounded animate-pulse w-2/3"></div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Streaming with Suspense

```typescript
// app/dashboard/page.tsx
import { Suspense } from 'react';

async function StudentCount() {
  const res = await fetch('https://api.college.edu/students/count');
  const { count } = await res.json();
  return <div>Total Students: {count}</div>;
}

async function CourseCount() {
  const res = await fetch('https://api.college.edu/courses/count');
  const { count } = await res.json();
  return <div>Total Courses: {count}</div>;
}

export default function Dashboard() {
  return (
    <div className="grid grid-cols-2 gap-4">
      <Suspense fallback={<div>Loading students...</div>}>
        <StudentCount />
      </Suspense>

      <Suspense fallback={<div>Loading courses...</div>}>
        <CourseCount />
      </Suspense>
    </div>
  );
}

// Each component streams independently!
```

---

## Error Handling {#error-handling}

Next.js provides automatic error boundaries with `error.tsx`.

### Basic Error Boundary

```typescript
// app/students/error.tsx
'use client'; // Error components must be Client Components

import { useEffect } from 'react';

export default function Error({
  error,
  reset
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log error to reporting service
    console.error('Error:', error);
  }, [error]);

  return (
    <div className="flex flex-col items-center justify-center h-screen">
      <h2 className="text-2xl font-bold mb-4">Something went wrong!</h2>
      <p className="text-gray-600 mb-4">{error.message}</p>
      <button
        onClick={reset}
        className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Try again
      </button>
    </div>
  );
}
```

### Not Found Page

```typescript
// app/students/[id]/not-found.tsx
import Link from 'next/link';

export default function NotFound() {
  return (
    <div className="text-center py-12">
      <h2 className="text-3xl font-bold mb-4">Student Not Found</h2>
      <p className="text-gray-600 mb-8">
        The student you're looking for doesn't exist.
      </p>
      <Link
        href="/students"
        className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Back to Students
      </Link>
    </div>
  );
}

// app/students/[id]/page.tsx
import { notFound } from 'next/navigation';

async function getStudent(id: string) {
  const res = await fetch(`https://api.college.edu/students/${id}`);
  if (!res.ok) return null;
  return res.json();
}

export default async function StudentPage({ params }: { params: { id: string } }) {
  const student = await getStudent(params.id);

  if (!student) {
    notFound(); // Triggers not-found.tsx
  }

  return <div>{student.name}</div>;
}
```

### Global Error Boundary

```typescript
// app/global-error.tsx
'use client';

export default function GlobalError({
  error,
  reset
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <html>
      <body>
        <h2>Application Error</h2>
        <button onClick={reset}>Try again</button>
      </body>
    </html>
  );
}
```

---

## Metadata and SEO {#metadata-seo}

Next.js provides powerful SEO features through metadata.

### Static Metadata

```typescript
// app/page.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'College Portal - Home',
  description: 'Manage students, courses, and enrollments',
  keywords: ['college', 'education', 'students', 'courses'],
  authors: [{ name: 'Saiprasad Hegde' }],
  openGraph: {
    title: 'College Portal',
    description: 'Student and course management system',
    type: 'website',
    images: ['/og-image.png']
  },
  twitter: {
    card: 'summary_large_image',
    title: 'College Portal',
    description: 'Student and course management system'
  }
};

export default function HomePage() {
  return <h1>Welcome to College Portal</h1>;
}
```

### Dynamic Metadata

```typescript
// app/students/[id]/page.tsx
import { Metadata } from 'next';

interface PageProps {
  params: { id: string };
}

async function getStudent(id: string) {
  const res = await fetch(`https://api.college.edu/students/${id}`);
  return res.json();
}

export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
  const student = await getStudent(params.id);

  return {
    title: `${student.name} - Student Profile`,
    description: `View ${student.name}'s academic profile, GPA: ${student.gpa}`,
    openGraph: {
      title: student.name,
      description: `${student.major} student with GPA ${student.gpa}`,
      images: [student.avatar || '/default-avatar.png']
    }
  };
}

export default async function StudentPage({ params }: PageProps) {
  const student = await getStudent(params.id);

  return (
    <div>
      <h1>{student.name}</h1>
      <p>GPA: {student.gpa}</p>
    </div>
  );
}
```

### generateStaticParams (Static Generation)

```typescript
// app/courses/[courseId]/page.tsx

// Generate pages at build time
export async function generateStaticParams() {
  const courses = await fetch('https://api.college.edu/courses').then(res => res.json());

  return courses.map((course: any) => ({
    courseId: course.id
  }));
}

// This page will be statically generated for each course
export default function CoursePage({ params }: { params: { courseId: string } }) {
  return <h1>Course {params.courseId}</h1>;
}
```

---

## College Portal Example {#college-example}

Let's build a complete Next.js app structure for our College Portal.

### Project Structure

```
college-portal/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── not-found.tsx
│   ├── (auth)/
│   │   ├── layout.tsx
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   ├── students/
│   │   │   ├── page.tsx
│   │   │   ├── loading.tsx
│   │   │   ├── error.tsx
│   │   │   └── [id]/
│   │   │       ├── page.tsx
│   │   │       ├── loading.tsx
│   │   │       └── not-found.tsx
│   │   └── courses/
│   │       ├── page.tsx
│   │       ├── loading.tsx
│   │       └── [courseId]/
│   │           ├── page.tsx
│   │           └── enroll/
│   │               └── page.tsx
│   └── api/
│       └── students/
│           └── route.ts
├── components/
│   ├── Navigation.tsx
│   ├── StudentCard.tsx
│   └── CourseCard.tsx
└── lib/
    └── api.ts
```

### Root Layout

```typescript
// app/layout.tsx
import './globals.css';
import { Inter } from 'next/font/google';
import type { Metadata } from 'next';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: {
    template: '%s | College Portal',
    default: 'College Portal'
  },
  description: 'Modern college management system'
};

export default function RootLayout({
  children
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

### Home Page

```typescript
// app/page.tsx
import Link from 'next/link';

export default function HomePage() {
  return (
    <div className="min-h-screen flex flex-col items-center justify-center bg-gradient-to-br from-blue-500 to-purple-600">
      <h1 className="text-6xl font-bold text-white mb-8">College Portal</h1>
      <p className="text-xl text-white/90 mb-12">
        Modern Student & Course Management
      </p>
      <div className="flex gap-4">
        <Link
          href="/students"
          className="px-8 py-3 bg-white text-blue-600 rounded-lg font-semibold hover:bg-gray-100 transition"
        >
          View Students
        </Link>
        <Link
          href="/courses"
          className="px-8 py-3 bg-blue-700 text-white rounded-lg font-semibold hover:bg-blue-800 transition"
        >
          Browse Courses
        </Link>
      </div>
    </div>
  );
}
```

### Dashboard Layout

```typescript
// app/(dashboard)/layout.tsx
import { Navigation } from '@/components/Navigation';

export default function DashboardLayout({
  children
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="min-h-screen bg-gray-50">
      <header className="bg-white shadow">
        <div className="max-w-7xl mx-auto px-4 py-4">
          <div className="flex items-center justify-between">
            <h1 className="text-2xl font-bold text-gray-900">College Portal</h1>
            <Navigation />
          </div>
        </div>
      </header>
      <main className="max-w-7xl mx-auto px-4 py-8">
        {children}
      </main>
    </div>
  );
}
```

### Students Page (Server Component)

```typescript
// app/(dashboard)/students/page.tsx
import { StudentCard } from '@/components/StudentCard';
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Students',
  description: 'Browse all students in the college'
};

interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
  major: string;
}

async function getStudents(): Promise<Student[]> {
  const res = await fetch('https://api.college.edu/students', {
    next: { revalidate: 60 } // Revalidate every 60 seconds
  });

  if (!res.ok) throw new Error('Failed to fetch students');
  return res.json();
}

export default async function StudentsPage() {
  const students = await getStudents();

  return (
    <div>
      <div className="mb-8">
        <h2 className="text-3xl font-bold text-gray-900">Students</h2>
        <p className="text-gray-600 mt-2">
          {students.length} students enrolled
        </p>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {students.map(student => (
          <StudentCard key={student.id} student={student} />
        ))}
      </div>
    </div>
  );
}
```

### Student Detail Page

```typescript
// app/(dashboard)/students/[id]/page.tsx
import { notFound } from 'next/navigation';
import type { Metadata } from 'next';

interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
  major: string;
  enrolledCourses: Array<{
    id: string;
    name: string;
    code: string;
  }>;
}

async function getStudent(id: string): Promise<Student | null> {
  const res = await fetch(`https://api.college.edu/students/${id}`, {
    cache: 'no-store'
  });

  if (!res.ok) return null;
  return res.json();
}

export async function generateMetadata({
  params
}: {
  params: { id: string };
}): Promise<Metadata> {
  const student = await getStudent(params.id);

  if (!student) {
    return { title: 'Student Not Found' };
  }

  return {
    title: student.name,
    description: `${student.name} - ${student.major} student with GPA ${student.gpa}`
  };
}

export default async function StudentDetailPage({
  params
}: {
  params: { id: string };
}) {
  const student = await getStudent(params.id);

  if (!student) {
    notFound();
  }

  return (
    <div className="max-w-4xl mx-auto">
      <div className="bg-white rounded-lg shadow-lg p-8">
        {/* Header */}
        <div className="border-b pb-6 mb-6">
          <h1 className="text-4xl font-bold text-gray-900">{student.name}</h1>
          <p className="text-gray-600 mt-2">{student.email}</p>
        </div>

        {/* Info Grid */}
        <div className="grid grid-cols-2 gap-6 mb-8">
          <div>
            <h3 className="text-sm font-medium text-gray-500">Major</h3>
            <p className="text-lg font-semibold text-gray-900">{student.major}</p>
          </div>
          <div>
            <h3 className="text-sm font-medium text-gray-500">GPA</h3>
            <p className="text-lg font-semibold text-blue-600">{student.gpa.toFixed(2)}</p>
          </div>
        </div>

        {/* Enrolled Courses */}
        <div>
          <h3 className="text-xl font-bold mb-4">Enrolled Courses</h3>
          <div className="space-y-3">
            {student.enrolledCourses.map(course => (
              <div
                key={course.id}
                className="flex items-center justify-between p-4 bg-gray-50 rounded-lg"
              >
                <div>
                  <p className="font-semibold">{course.name}</p>
                  <p className="text-sm text-gray-600">{course.code}</p>
                </div>
              </div>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
}
```

### API Route

```typescript
// app/api/students/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  // In real app: fetch from database
  const students = [
    { id: '1', name: 'Alice Johnson', email: 'alice@college.edu', gpa: 3.85, major: 'CS' },
    { id: '2', name: 'Bob Smith', email: 'bob@college.edu', gpa: 3.60, major: 'Math' }
  ];

  return NextResponse.json(students);
}

export async function POST(request: Request) {
  const body = await request.json();

  // Validate and create student
  const newStudent = {
    id: Date.now().toString(),
    ...body
  };

  return NextResponse.json(newStudent, { status: 201 });
}
```

---

## Summary

Next.js fundamentals covered:

1. **Framework Benefits**: SSR, routing, optimization
2. **App Router**: Modern file-based routing
3. **Server Components**: Default, performant
4. **Client Components**: Interactive, use hooks
5. **Layouts**: Shared UI structure
6. **Loading States**: Automatic loading UI
7. **Error Handling**: Built-in error boundaries
8. **Metadata**: SEO optimization

**Key Takeaways:**
- Use App Router for all new projects
- Server Components by default, Client when needed
- File structure = route structure
- Automatic code splitting and optimization
- Built-in SEO and performance features

---

## Navigation

- **Previous**: [Part 5: React Hooks & State](./05_React_Hooks_State.md)
- **Next**: [Part 7: Routing & Navigation](./07_Routing_Navigation.md)
- **[Back to Index](./README.md)**

---

**Author: Saiprasad Hegde**

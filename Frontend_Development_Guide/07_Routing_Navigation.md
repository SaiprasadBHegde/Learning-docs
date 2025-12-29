# Part 7: Routing & Navigation

**Author: Saiprasad Hegde**

## Table of Contents
1. [Introduction to Routing](#introduction)
2. [Next.js Routing](#nextjs-routing)
3. [Dynamic Routes](#dynamic-routes)
4. [Navigation Components](#navigation-components)
5. [URL Parameters and Query Strings](#url-parameters)
6. [useRouter, usePathname, useSearchParams](#hooks)
7. [Protected Routes](#protected-routes)
8. [Programmatic Navigation](#programmatic-navigation)
9. [College Portal Examples](#college-examples)

---

## Introduction to Routing {#introduction}

Routing determines which component/page displays for a given URL.

**Analogy**: Routing is like a GPS for your app. When you enter a destination (URL), the router figures out which route to take (which component to display).

---

## Next.js Routing {#nextjs-routing}

Next.js uses file-based routing in the `app/` directory.

### Basic Routes

```typescript
// File structure automatically creates routes:

app/
├── page.tsx                      → /
├── about/page.tsx               → /about
├── students/
│   ├── page.tsx                 → /students
│   └── [id]/
│       └── page.tsx             → /students/:id
└── courses/
    ├── page.tsx                 → /courses
    └── [courseId]/
        ├── page.tsx             → /courses/:courseId
        └── enroll/
            └── page.tsx         → /courses/:courseId/enroll
```

### Route Groups (Organization)

```typescript
// Route groups organize files without affecting URLs
// Use parentheses for groups: (groupName)

app/
├── (marketing)/
│   ├── page.tsx                 → /
│   ├── about/page.tsx           → /about
│   └── contact/page.tsx         → /contact
├── (shop)/
│   ├── products/page.tsx        → /products
│   └── cart/page.tsx            → /cart
└── (dashboard)/
    ├── students/page.tsx        → /students
    └── courses/page.tsx         → /courses

// Each group can have its own layout.tsx
```

### Private Folders

```typescript
// Prefix with underscore to exclude from routing

app/
├── _components/          // Not a route
│   ├── Header.tsx
│   └── Footer.tsx
├── _lib/                 // Not a route
│   └── utils.ts
└── students/
    └── page.tsx         → /students
```

---

## Dynamic Routes {#dynamic-routes}

Dynamic routes use brackets `[param]` for URL parameters.

### Single Dynamic Segment

```typescript
// app/students/[id]/page.tsx

interface PageProps {
  params: { id: string };
}

export default async function StudentPage({ params }: PageProps) {
  const { id } = params;

  // Fetch student data using id
  const student = await fetchStudent(id);

  return (
    <div>
      <h1>{student.name}</h1>
      <p>Student ID: {id}</p>
    </div>
  );
}

// URLs matched:
// /students/STU001
// /students/STU002
// /students/alice-johnson
```

### Multiple Dynamic Segments

```typescript
// app/departments/[dept]/courses/[courseId]/page.tsx

interface PageProps {
  params: {
    dept: string;
    courseId: string;
  };
}

export default function CoursePage({ params }: PageProps) {
  return (
    <div>
      <h1>Department: {params.dept}</h1>
      <p>Course ID: {params.courseId}</p>
    </div>
  );
}

// URL: /departments/computer-science/courses/CS101
// params = { dept: "computer-science", courseId: "CS101" }
```

### Catch-All Routes

```typescript
// app/docs/[...slug]/page.tsx
// Matches multiple path segments

interface PageProps {
  params: { slug: string[] };
}

export default function DocsPage({ params }: PageProps) {
  const path = params.slug.join('/');

  return (
    <div>
      <h1>Documentation</h1>
      <p>Path: {path}</p>
      <ul>
        {params.slug.map((segment, i) => (
          <li key={i}>{segment}</li>
        ))}
      </ul>
    </div>
  );
}

// Matches:
// /docs/getting-started → slug = ["getting-started"]
// /docs/api/authentication → slug = ["api", "authentication"]
// /docs/guides/deployment/vercel → slug = ["guides", "deployment", "vercel"]
```

### Optional Catch-All Routes

```typescript
// app/shop/[[...slug]]/page.tsx
// Also matches the base route

interface PageProps {
  params: { slug?: string[] };
}

export default function ShopPage({ params }: PageProps) {
  const segments = params.slug || [];

  if (segments.length === 0) {
    return <div>Shop Home</div>;
  }

  return <div>Category: {segments.join(' > ')}</div>;
}

// Matches:
// /shop → slug = undefined
// /shop/electronics → slug = ["electronics"]
// /shop/electronics/phones → slug = ["electronics", "phones"]
```

### generateStaticParams (SSG)

```typescript
// app/students/[id]/page.tsx

// Generate static pages at build time
export async function generateStaticParams() {
  const students = await fetchAllStudents();

  return students.map((student) => ({
    id: student.id
  }));
}

export default async function StudentPage({
  params
}: {
  params: { id: string };
}) {
  const student = await fetchStudent(params.id);
  return <div>{student.name}</div>;
}

// At build time, creates pages for each student:
// /students/STU001
// /students/STU002
// /students/STU003
```

---

## Navigation Components {#navigation-components}

### Link Component

```typescript
import Link from 'next/link';

export default function Navigation() {
  return (
    <nav>
      {/* Basic link */}
      <Link href="/">Home</Link>

      {/* Link with styling */}
      <Link
        href="/students"
        className="text-blue-600 hover:text-blue-800"
      >
        Students
      </Link>

      {/* Dynamic link */}
      <Link href={`/students/${studentId}`}>
        View Student
      </Link>

      {/* Link with query params */}
      <Link
        href={{
          pathname: '/courses',
          query: { semester: 'fall', year: '2024' }
        }}
      >
        Fall 2024 Courses
      </Link>

      {/* External link (opens in new tab) */}
      <Link
        href="https://college.edu"
        target="_blank"
        rel="noopener noreferrer"
      >
        College Website
      </Link>

      {/* Prefetch disabled */}
      <Link href="/slow-page" prefetch={false}>
        Slow Page
      </Link>

      {/* Replace history instead of push */}
      <Link href="/redirect" replace>
        Redirect
      </Link>
    </nav>
  );
}
```

### Active Link Styling

```typescript
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';

const navItems = [
  { href: '/', label: 'Home' },
  { href: '/students', label: 'Students' },
  { href: '/courses', label: 'Courses' },
  { href: '/about', label: 'About' }
];

export function Navigation() {
  const pathname = usePathname();

  return (
    <nav className="flex gap-4">
      {navItems.map((item) => {
        const isActive = pathname === item.href;

        return (
          <Link
            key={item.href}
            href={item.href}
            className={`px-4 py-2 rounded transition ${
              isActive
                ? 'bg-blue-600 text-white'
                : 'text-gray-700 hover:bg-gray-100'
            }`}
          >
            {item.label}
          </Link>
        );
      })}
    </nav>
  );
}
```

### Breadcrumbs

```typescript
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';

export function Breadcrumbs() {
  const pathname = usePathname();
  const segments = pathname.split('/').filter(Boolean);

  return (
    <nav className="flex items-center gap-2 text-sm">
      <Link href="/" className="text-blue-600 hover:underline">
        Home
      </Link>

      {segments.map((segment, index) => {
        const href = `/${segments.slice(0, index + 1).join('/')}`;
        const isLast = index === segments.length - 1;
        const label = segment.replace(/-/g, ' ');

        return (
          <div key={href} className="flex items-center gap-2">
            <span className="text-gray-400">/</span>
            {isLast ? (
              <span className="text-gray-600 capitalize">{label}</span>
            ) : (
              <Link href={href} className="text-blue-600 hover:underline capitalize">
                {label}
              </Link>
            )}
          </div>
        );
      })}
    </nav>
  );
}

// Usage: /students/STU001/courses
// Renders: Home / Students / STU001 / Courses
```

---

## URL Parameters and Query Strings {#url-parameters}

### Reading URL Params

```typescript
// app/students/[id]/page.tsx
export default function StudentPage({
  params
}: {
  params: { id: string };
}) {
  return <div>Student ID: {params.id}</div>;
}

// URL: /students/STU001
// params.id = "STU001"
```

### Reading Search Params (Server Component)

```typescript
// app/courses/page.tsx
interface PageProps {
  searchParams: { [key: string]: string | string[] | undefined };
}

export default function CoursesPage({ searchParams }: PageProps) {
  const semester = searchParams.semester || 'all';
  const department = searchParams.department;
  const page = Number(searchParams.page) || 1;

  return (
    <div>
      <h1>Courses</h1>
      <p>Semester: {semester}</p>
      <p>Department: {department}</p>
      <p>Page: {page}</p>
    </div>
  );
}

// URL: /courses?semester=fall&department=CS&page=2
// searchParams = { semester: "fall", department: "CS", page: "2" }
```

### Reading Search Params (Client Component)

```typescript
'use client';

import { useSearchParams } from 'next/navigation';

export function CourseFilters() {
  const searchParams = useSearchParams();

  const semester = searchParams.get('semester');
  const department = searchParams.get('department');
  const allParams = searchParams.toString();

  return (
    <div>
      <p>Semester: {semester}</p>
      <p>Department: {department}</p>
      <p>Query string: {allParams}</p>
    </div>
  );
}
```

### Updating Search Params

```typescript
'use client';

import { useRouter, useSearchParams, usePathname } from 'next/navigation';

export function SearchFilters() {
  const router = useRouter();
  const pathname = usePathname();
  const searchParams = useSearchParams();

  const updateFilters = (key: string, value: string) => {
    const params = new URLSearchParams(searchParams.toString());

    if (value) {
      params.set(key, value);
    } else {
      params.delete(key);
    }

    router.push(`${pathname}?${params.toString()}`);
  };

  return (
    <div className="flex gap-4">
      <select
        value={searchParams.get('semester') || ''}
        onChange={(e) => updateFilters('semester', e.target.value)}
      >
        <option value="">All Semesters</option>
        <option value="fall">Fall 2024</option>
        <option value="spring">Spring 2025</option>
      </select>

      <select
        value={searchParams.get('department') || ''}
        onChange={(e) => updateFilters('department', e.target.value)}
      >
        <option value="">All Departments</option>
        <option value="CS">Computer Science</option>
        <option value="MATH">Mathematics</option>
      </select>
    </div>
  );
}
```

---

## useRouter, usePathname, useSearchParams {#hooks}

### useRouter Hook

```typescript
'use client';

import { useRouter } from 'next/navigation';

export function EnrollButton({ courseId }: { courseId: string }) {
  const router = useRouter();

  const handleEnroll = async () => {
    // Enroll student
    await enrollInCourse(courseId);

    // Navigate to success page
    router.push(`/courses/${courseId}/enrolled`);
  };

  return <button onClick={handleEnroll}>Enroll Now</button>;
}

// Available methods:
// router.push(href) - Navigate to href
// router.replace(href) - Replace current URL
// router.refresh() - Refresh current page
// router.back() - Go back in history
// router.forward() - Go forward in history
// router.prefetch(href) - Prefetch a page
```

### usePathname Hook

```typescript
'use client';

import { usePathname } from 'next/navigation';

export function CurrentPath() {
  const pathname = usePathname();

  // URL: /students/STU001
  // pathname = "/students/STU001"

  return <div>Current path: {pathname}</div>;
}

// Use cases:
// - Active link highlighting
// - Breadcrumbs
// - Conditional rendering based on route
// - Analytics tracking
```

### useSearchParams Hook

```typescript
'use client';

import { useSearchParams } from 'next/navigation';

export function FilterDisplay() {
  const searchParams = useSearchParams();

  const semester = searchParams.get('semester');
  const department = searchParams.get('department');
  const hasFilters = searchParams.size > 0;

  // Get all as object
  const allParams = Object.fromEntries(searchParams.entries());

  return (
    <div>
      {hasFilters ? (
        <div>
          <p>Active Filters:</p>
          <ul>
            {Array.from(searchParams.entries()).map(([key, value]) => (
              <li key={key}>
                {key}: {value}
              </li>
            ))}
          </ul>
        </div>
      ) : (
        <p>No filters applied</p>
      )}
    </div>
  );
}
```

### Combined Example

```typescript
'use client';

import { useRouter, usePathname, useSearchParams } from 'next/navigation';

export function NavigationInfo() {
  const router = useRouter();
  const pathname = usePathname();
  const searchParams = useSearchParams();

  const goToCourses = () => {
    router.push('/courses?semester=fall&year=2024');
  };

  const refreshPage = () => {
    router.refresh();
  };

  return (
    <div className="space-y-4">
      <div>
        <strong>Current Path:</strong> {pathname}
      </div>
      <div>
        <strong>Query Params:</strong> {searchParams.toString()}
      </div>
      <div className="flex gap-2">
        <button onClick={goToCourses}>Go to Courses</button>
        <button onClick={() => router.back()}>Go Back</button>
        <button onClick={refreshPage}>Refresh</button>
      </div>
    </div>
  );
}
```

---

## Protected Routes {#protected-routes}

### Middleware Protection

```typescript
// middleware.ts (root of project)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token');
  const isAuthPage = request.nextUrl.pathname.startsWith('/login');
  const isProtected = request.nextUrl.pathname.startsWith('/dashboard');

  // Redirect to login if accessing protected route without token
  if (isProtected && !token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Redirect to dashboard if logged in user tries to access login
  if (isAuthPage && token) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/login']
};
```

### Component-Level Protection

```typescript
// app/(protected)/layout.tsx
import { redirect } from 'next/navigation';
import { getSession } from '@/lib/auth';

export default async function ProtectedLayout({
  children
}: {
  children: React.ReactNode;
}) {
  const session = await getSession();

  if (!session) {
    redirect('/login');
  }

  return <div>{children}</div>;
}
```

### Role-Based Access

```typescript
// app/admin/layout.tsx
import { redirect } from 'next/navigation';
import { getSession } from '@/lib/auth';

export default async function AdminLayout({
  children
}: {
  children: React.ReactNode;
}) {
  const session = await getSession();

  if (!session) {
    redirect('/login');
  }

  if (session.user.role !== 'admin') {
    redirect('/unauthorized');
  }

  return (
    <div>
      <h1>Admin Dashboard</h1>
      {children}
    </div>
  );
}
```

---

## Programmatic Navigation {#programmatic-navigation}

### After Form Submission

```typescript
'use client';

import { useRouter } from 'next/navigation';
import { useState } from 'react';

export function EnrollmentForm() {
  const router = useRouter();
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);

    try {
      const formData = new FormData(e.currentTarget);
      const response = await fetch('/api/enroll', {
        method: 'POST',
        body: JSON.stringify(Object.fromEntries(formData))
      });

      if (response.ok) {
        // Navigate to success page
        router.push('/enrollment/success');
      }
    } catch (error) {
      console.error('Enrollment failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
      <button type="submit" disabled={loading}>
        {loading ? 'Enrolling...' : 'Enroll'}
      </button>
    </form>
  );
}
```

### Conditional Navigation

```typescript
'use client';

import { useRouter } from 'next/navigation';

export function StudentActions({ student }: { student: any }) {
  const router = useRouter();

  const handleAction = (action: string) => {
    switch (action) {
      case 'view':
        router.push(`/students/${student.id}`);
        break;
      case 'edit':
        router.push(`/students/${student.id}/edit`);
        break;
      case 'courses':
        router.push(`/students/${student.id}/courses`);
        break;
      case 'grades':
        router.push(`/students/${student.id}/grades`);
        break;
    }
  };

  return (
    <div className="flex gap-2">
      <button onClick={() => handleAction('view')}>View</button>
      <button onClick={() => handleAction('edit')}>Edit</button>
      <button onClick={() => handleAction('courses')}>Courses</button>
      <button onClick={() => handleAction('grades')}>Grades</button>
    </div>
  );
}
```

### Navigation with State

```typescript
'use client';

import { useRouter } from 'next/navigation';

export function SearchResults({ query }: { query: string }) {
  const router = useRouter();

  const viewStudent = (studentId: string) => {
    // Navigate with query params
    router.push(`/students/${studentId}?from=search&query=${query}`);
  };

  return (
    <div>
      {/* results */}
      <button onClick={() => viewStudent('STU001')}>
        View Student
      </button>
    </div>
  );
}

// On student page, read the params:
// app/students/[id]/page.tsx
export default function StudentPage({
  params,
  searchParams
}: {
  params: { id: string };
  searchParams: { from?: string; query?: string };
}) {
  return (
    <div>
      <h1>Student {params.id}</h1>
      {searchParams.from === 'search' && (
        <p>From search: "{searchParams.query}"</p>
      )}
    </div>
  );
}
```

---

## College Portal Examples {#college-examples}

### Complete Navigation Component

```typescript
// components/Navigation.tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { useState } from 'react';

const navItems = [
  { href: '/', label: 'Home', icon: '🏠' },
  { href: '/students', label: 'Students', icon: '👨‍🎓' },
  { href: '/courses', label: 'Courses', icon: '📚' },
  { href: '/enrollments', label: 'Enrollments', icon: '✅' },
  { href: '/reports', label: 'Reports', icon: '📊' }
];

export function Navigation() {
  const pathname = usePathname();
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);

  const isActive = (href: string) => {
    if (href === '/') return pathname === '/';
    return pathname.startsWith(href);
  };

  return (
    <nav className="bg-white shadow">
      <div className="max-w-7xl mx-auto px-4">
        {/* Desktop Navigation */}
        <div className="hidden md:flex items-center gap-1">
          {navItems.map((item) => (
            <Link
              key={item.href}
              href={item.href}
              className={`px-4 py-3 rounded-lg transition flex items-center gap-2 ${
                isActive(item.href)
                  ? 'bg-blue-600 text-white'
                  : 'text-gray-700 hover:bg-gray-100'
              }`}
            >
              <span>{item.icon}</span>
              <span>{item.label}</span>
            </Link>
          ))}
        </div>

        {/* Mobile Navigation */}
        <div className="md:hidden">
          <button
            onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
            className="p-2"
          >
            {mobileMenuOpen ? '✕' : '☰'}
          </button>

          {mobileMenuOpen && (
            <div className="absolute top-16 left-0 right-0 bg-white shadow-lg">
              {navItems.map((item) => (
                <Link
                  key={item.href}
                  href={item.href}
                  onClick={() => setMobileMenuOpen(false)}
                  className={`block px-4 py-3 ${
                    isActive(item.href) ? 'bg-blue-600 text-white' : ''
                  }`}
                >
                  {item.icon} {item.label}
                </Link>
              ))}
            </div>
          )}
        </div>
      </div>
    </nav>
  );
}
```

### Course Browser with Filters

```typescript
// app/courses/page.tsx
import { CourseFilters } from '@/components/CourseFilters';
import { CourseList } from '@/components/CourseList';

interface PageProps {
  searchParams: {
    department?: string;
    semester?: string;
    level?: string;
    search?: string;
  };
}

export default async function CoursesPage({ searchParams }: PageProps) {
  const courses = await fetchCourses(searchParams);

  return (
    <div>
      <h1 className="text-3xl font-bold mb-6">Course Catalog</h1>
      <CourseFilters />
      <CourseList courses={courses} />
    </div>
  );
}

// components/CourseFilters.tsx
'use client';

import { useRouter, useSearchParams, usePathname } from 'next/navigation';

export function CourseFilters() {
  const router = useRouter();
  const pathname = usePathname();
  const searchParams = useSearchParams();

  const updateFilter = (key: string, value: string) => {
    const params = new URLSearchParams(searchParams.toString());
    if (value) {
      params.set(key, value);
    } else {
      params.delete(key);
    }
    router.push(`${pathname}?${params.toString()}`);
  };

  return (
    <div className="flex gap-4 mb-6">
      <input
        type="text"
        placeholder="Search courses..."
        defaultValue={searchParams.get('search') || ''}
        onChange={(e) => updateFilter('search', e.target.value)}
        className="px-4 py-2 border rounded"
      />

      <select
        value={searchParams.get('department') || ''}
        onChange={(e) => updateFilter('department', e.target.value)}
        className="px-4 py-2 border rounded"
      >
        <option value="">All Departments</option>
        <option value="CS">Computer Science</option>
        <option value="MATH">Mathematics</option>
        <option value="PHYS">Physics</option>
      </select>

      <select
        value={searchParams.get('level') || ''}
        onChange={(e) => updateFilter('level', e.target.value)}
        className="px-4 py-2 border rounded"
      >
        <option value="">All Levels</option>
        <option value="100">100-level</option>
        <option value="200">200-level</option>
        <option value="300">300-level</option>
      </select>
    </div>
  );
}
```

---

## Summary

Routing & Navigation covered:

1. **File-Based Routing**: Automatic route creation
2. **Dynamic Routes**: URL parameters with brackets
3. **Link Component**: Client-side navigation
4. **Navigation Hooks**: useRouter, usePathname, useSearchParams
5. **Protected Routes**: Middleware and layout protection
6. **Programmatic Navigation**: Router methods
7. **URL Parameters**: Reading and updating query strings

**Key Takeaways:**
- Use `<Link>` for navigation (not `<a>`)
- Server Components can read params directly
- Client Components use hooks for params
- Protect routes with middleware or layouts
- Update URL params for filterable lists
- Use route groups for organization

---

## Navigation

- **Previous**: [Part 6: Next.js Fundamentals](./06_NextJS_Fundamentals.md)
- **Next**: [Part 8: API Integration with TanStack Query](./08_API_Integration_TanStack.md)
- **[Back to Index](./README.md)**

---

**Author: Saiprasad Hegde**

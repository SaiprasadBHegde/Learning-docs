# Part 8: API Integration with TanStack Query

**Author: Saiprasad Hegde**

## Table of Contents
1. [Introduction to Data Fetching](#introduction)
2. [Fetch API Basics](#fetch-basics)
3. [Axios Setup](#axios-setup)
4. [TanStack Query Setup](#tanstack-setup)
5. [useQuery - Fetching Data](#usequery)
6. [useMutation - Create/Update/Delete](#usemutation)
7. [Query Keys and Invalidation](#query-keys)
8. [Optimistic Updates](#optimistic-updates)
9. [Error Handling and Retry](#error-handling)
10. [Infinite Queries](#infinite-queries)
11. [Next.js Server Actions](#server-actions)
12. [College Portal Examples](#college-examples)

---

## Introduction to Data Fetching {#introduction}

Modern apps need to fetch, cache, and update server data efficiently.

**Analogy**: TanStack Query is like a smart librarian. Instead of fetching the same book repeatedly, the librarian remembers what you read recently (caching), automatically gets new editions (refetching), and knows when to update outdated books (invalidation).

### Problems with Manual Fetching

```typescript
// ❌ Manual fetching problems:
const [students, setStudents] = useState([]);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);

useEffect(() => {
  setLoading(true);
  fetch('/api/students')
    .then(res => res.json())
    .then(setStudents)
    .catch(setError)
    .finally(() => setLoading(false));
}, []);

// Issues:
// - Boilerplate code
// - No caching
// - No automatic refetching
// - No background updates
// - Manual loading/error states
```

### With TanStack Query

```typescript
// ✅ TanStack Query solution:
const { data: students, isLoading, error } = useQuery({
  queryKey: ['students'],
  queryFn: () => fetch('/api/students').then(res => res.json())
});

// Benefits:
// - Automatic caching
// - Background refetching
// - Deduplication
// - Pagination/infinite scroll
// - Optimistic updates
// - Much less code!
```

---

## Fetch API Basics {#fetch-basics}

The native JavaScript Fetch API.

### Basic GET Request

```typescript
// Simple fetch
async function getStudents() {
  const response = await fetch('https://api.college.edu/students');

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }

  const data = await response.json();
  return data;
}

// With error handling
async function getStudentSafe(id: string) {
  try {
    const response = await fetch(`https://api.college.edu/students/${id}`);

    if (!response.ok) {
      throw new Error('Student not found');
    }

    return await response.json();
  } catch (error) {
    console.error('Failed to fetch student:', error);
    throw error;
  }
}
```

### POST Request

```typescript
async function createStudent(student: Student) {
  const response = await fetch('https://api.college.edu/students', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(student)
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Failed to create student');
  }

  return response.json();
}
```

### PUT/PATCH Request

```typescript
async function updateStudent(id: string, updates: Partial<Student>) {
  const response = await fetch(`https://api.college.edu/students/${id}`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(updates)
  });

  if (!response.ok) {
    throw new Error('Update failed');
  }

  return response.json();
}
```

### DELETE Request

```typescript
async function deleteStudent(id: string) {
  const response = await fetch(`https://api.college.edu/students/${id}`, {
    method: 'DELETE'
  });

  if (!response.ok) {
    throw new Error('Delete failed');
  }

  return true;
}
```

---

## Axios Setup {#axios-setup}

Axios is a popular HTTP client with better DX than fetch.

### Installation

```bash
npm install axios
```

### Basic Axios Instance

```typescript
// lib/axios.ts
import axios from 'axios';

export const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || 'https://api.college.edu',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
});

// Request interceptor (add auth token)
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('auth-token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor (handle errors globally)
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Redirect to login
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

### Axios API Functions

```typescript
// services/students.ts
import { api } from '@/lib/axios';

export const studentsAPI = {
  getAll: () => api.get<Student[]>('/students'),

  getById: (id: string) => api.get<Student>(`/students/${id}`),

  create: (student: CreateStudentDTO) =>
    api.post<Student>('/students', student),

  update: (id: string, updates: Partial<Student>) =>
    api.patch<Student>(`/students/${id}`, updates),

  delete: (id: string) => api.delete(`/students/${id}`)
};

// Usage
const students = await studentsAPI.getAll();
const student = await studentsAPI.getById('STU001');
```

---

## TanStack Query Setup {#tanstack-setup}

TanStack Query (formerly React Query) is the best library for server state management.

### Installation

```bash
npm install @tanstack/react-query
```

### Setup Provider

```typescript
// app/providers.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState } from 'react';

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 1 minute
            retry: 1
          }
        }
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}

// app/layout.tsx
import { Providers } from './providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

---

## useQuery - Fetching Data {#usequery}

`useQuery` fetches and caches data automatically.

### Basic Query

```typescript
'use client';

import { useQuery } from '@tanstack/react-query';

function StudentList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['students'],
    queryFn: async () => {
      const response = await fetch('/api/students');
      if (!response.ok) throw new Error('Failed to fetch');
      return response.json();
    }
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {data.map((student: Student) => (
        <div key={student.id}>{student.name}</div>
      ))}
    </div>
  );
}
```

### Query with Parameters

```typescript
function StudentDetails({ studentId }: { studentId: string }) {
  const { data: student, isLoading } = useQuery({
    queryKey: ['students', studentId],
    queryFn: async () => {
      const response = await fetch(`/api/students/${studentId}`);
      return response.json();
    },
    enabled: !!studentId // Only run if studentId exists
  });

  if (isLoading) return <div>Loading...</div>;
  if (!student) return <div>Student not found</div>;

  return (
    <div>
      <h2>{student.name}</h2>
      <p>{student.email}</p>
    </div>
  );
}
```

### Query Options

```typescript
const { data, isLoading, error, isFetching, isRefetching } = useQuery({
  queryKey: ['students'],
  queryFn: fetchStudents,

  // Caching
  staleTime: 5 * 60 * 1000, // 5 minutes - data stays fresh
  gcTime: 10 * 60 * 1000, // 10 minutes - garbage collection time (was cacheTime)

  // Refetching
  refetchOnWindowFocus: true,
  refetchOnMount: true,
  refetchInterval: false, // or 30000 for 30 seconds

  // Retry
  retry: 3,
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),

  // Conditional
  enabled: true, // or !!studentId to run conditionally

  // Initial data
  initialData: [],

  // Placeholder data (shown while loading)
  placeholderData: []
});
```

### Dependent Queries

```typescript
function StudentCourses({ studentId }: { studentId: string }) {
  // First query: get student
  const { data: student } = useQuery({
    queryKey: ['students', studentId],
    queryFn: () => fetchStudent(studentId)
  });

  // Second query: get student's enrollments (depends on first)
  const { data: enrollments } = useQuery({
    queryKey: ['enrollments', student?.id],
    queryFn: () => fetchEnrollments(student!.id),
    enabled: !!student // Only run when student is loaded
  });

  return <div>{/* ... */}</div>;
}
```

---

## useMutation - Create/Update/Delete {#usemutation}

`useMutation` handles data modifications.

### Basic Mutation

```typescript
'use client';

import { useMutation, useQueryClient } from '@tanstack/react-query';

function CreateStudentForm() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: async (newStudent: CreateStudentDTO) => {
      const response = await fetch('/api/students', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newStudent)
      });
      return response.json();
    },
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['students'] });
    }
  });

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);

    mutation.mutate({
      name: formData.get('name') as string,
      email: formData.get('email') as string,
      gpa: Number(formData.get('gpa')),
      major: formData.get('major') as string
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" required />
      <input name="email" type="email" required />
      <input name="gpa" type="number" step="0.01" required />
      <input name="major" required />

      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Creating...' : 'Create Student'}
      </button>

      {mutation.isError && <div>Error: {mutation.error.message}</div>}
      {mutation.isSuccess && <div>Student created successfully!</div>}
    </form>
  );
}
```

### Update Mutation

```typescript
function UpdateStudentGPA({ student }: { student: Student }) {
  const queryClient = useQueryClient();

  const updateMutation = useMutation({
    mutationFn: async (newGPA: number) => {
      const response = await fetch(`/api/students/${student.id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ gpa: newGPA })
      });
      return response.json();
    },
    onSuccess: (updatedStudent) => {
      // Update specific student in cache
      queryClient.setQueryData(['students', student.id], updatedStudent);
      // Invalidate students list
      queryClient.invalidateQueries({ queryKey: ['students'] });
    }
  });

  const [gpa, setGPA] = useState(student.gpa);

  return (
    <div>
      <input
        type="number"
        step="0.01"
        value={gpa}
        onChange={(e) => setGPA(Number(e.target.value))}
      />
      <button onClick={() => updateMutation.mutate(gpa)}>Update GPA</button>
    </div>
  );
}
```

### Delete Mutation

```typescript
function DeleteStudentButton({ studentId }: { studentId: string }) {
  const queryClient = useQueryClient();

  const deleteMutation = useMutation({
    mutationFn: async (id: string) => {
      await fetch(`/api/students/${id}`, { method: 'DELETE' });
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['students'] });
    }
  });

  const handleDelete = () => {
    if (confirm('Are you sure?')) {
      deleteMutation.mutate(studentId);
    }
  };

  return (
    <button onClick={handleDelete} disabled={deleteMutation.isPending}>
      {deleteMutation.isPending ? 'Deleting...' : 'Delete'}
    </button>
  );
}
```

---

## Query Keys and Invalidation {#query-keys}

Query keys are crucial for caching and invalidation.

### Query Key Patterns

```typescript
// String key (simple)
queryKey: ['students']

// Array key with ID (specific item)
queryKey: ['students', studentId]

// Array key with filters (filtered list)
queryKey: ['students', { semester: 'fall', department: 'CS' }]

// Nested keys (related data)
queryKey: ['students', studentId, 'courses']

// Complex key structure
queryKey: ['students', { filters: { gpa: { min: 3.5 } }, sort: 'name' }]
```

### Query Key Factory

```typescript
// lib/queryKeys.ts
export const queryKeys = {
  // Students
  students: {
    all: ['students'] as const,
    lists: () => [...queryKeys.students.all, 'list'] as const,
    list: (filters: StudentFilters) =>
      [...queryKeys.students.lists(), filters] as const,
    details: () => [...queryKeys.students.all, 'detail'] as const,
    detail: (id: string) => [...queryKeys.students.details(), id] as const,
    courses: (id: string) => [...queryKeys.students.detail(id), 'courses'] as const
  },

  // Courses
  courses: {
    all: ['courses'] as const,
    lists: () => [...queryKeys.courses.all, 'list'] as const,
    list: (filters: CourseFilters) =>
      [...queryKeys.courses.lists(), filters] as const,
    detail: (id: string) => [...queryKeys.courses.all, id] as const
  }
};

// Usage
const { data } = useQuery({
  queryKey: queryKeys.students.detail(studentId),
  queryFn: () => fetchStudent(studentId)
});
```

### Invalidation Strategies

```typescript
const queryClient = useQueryClient();

// Invalidate all students queries
queryClient.invalidateQueries({ queryKey: ['students'] });

// Invalidate specific student
queryClient.invalidateQueries({ queryKey: ['students', studentId] });

// Invalidate with exact match
queryClient.invalidateQueries({
  queryKey: ['students', { status: 'active' }],
  exact: true
});

// Refetch immediately
queryClient.invalidateQueries({
  queryKey: ['students'],
  refetchType: 'active' // 'active' | 'inactive' | 'all' | 'none'
});
```

### Manual Cache Updates

```typescript
// Set query data manually
queryClient.setQueryData(['students', studentId], newStudent);

// Update query data with function
queryClient.setQueryData<Student[]>(['students'], (old) => {
  if (!old) return [];
  return old.map((s) => (s.id === studentId ? { ...s, gpa: newGPA } : s));
});

// Get query data
const students = queryClient.getQueryData<Student[]>(['students']);
```

---

## Optimistic Updates {#optimistic-updates}

Update UI immediately before server responds.

### Optimistic Update Example

```typescript
function UpdateStudentName({ student }: { student: Student }) {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: async (newName: string) => {
      const response = await fetch(`/api/students/${student.id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name: newName })
      });
      return response.json();
    },

    // Before mutation
    onMutate: async (newName) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['students', student.id] });

      // Snapshot previous value
      const previousStudent = queryClient.getQueryData(['students', student.id]);

      // Optimistically update
      queryClient.setQueryData(['students', student.id], (old: Student) => ({
        ...old,
        name: newName
      }));

      // Return context with previous value
      return { previousStudent };
    },

    // On error, rollback
    onError: (err, newName, context) => {
      queryClient.setQueryData(
        ['students', student.id],
        context?.previousStudent
      );
    },

    // Always refetch after error or success
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['students', student.id] });
    }
  });

  const [name, setName] = useState(student.name);

  return (
    <div>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <button onClick={() => mutation.mutate(name)}>Save</button>
    </div>
  );
}
```

---

## Error Handling and Retry {#error-handling}

Handle errors gracefully with retries.

### Error Boundaries

```typescript
const { data, error, isError } = useQuery({
  queryKey: ['students'],
  queryFn: fetchStudents,
  retry: 3,
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000)
});

if (isError) {
  return (
    <div className="error-container">
      <h3>Failed to load students</h3>
      <p>{error.message}</p>
      <button onClick={() => refetch()}>Try Again</button>
    </div>
  );
}
```

### Global Error Handler

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      onError: (error) => {
        console.error('Query error:', error);
        // Show toast notification
      },
      retry: (failureCount, error: any) => {
        // Don't retry on 404
        if (error.response?.status === 404) return false;
        // Retry up to 3 times
        return failureCount < 3;
      }
    },
    mutations: {
      onError: (error) => {
        console.error('Mutation error:', error);
        // Show error toast
      }
    }
  }
});
```

---

## Infinite Queries {#infinite-queries}

Implement pagination and infinite scroll.

### Infinite Query Example

```typescript
'use client';

import { useInfiniteQuery } from '@tanstack/react-query';
import { useInView } from 'react-intersection-observer';
import { useEffect } from 'react';

function InfiniteStudentList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage
  } = useInfiniteQuery({
    queryKey: ['students', 'infinite'],
    queryFn: async ({ pageParam = 1 }) => {
      const response = await fetch(
        `/api/students?page=${pageParam}&limit=20`
      );
      return response.json();
    },
    getNextPageParam: (lastPage, pages) => {
      // If last page has fewer items than limit, no more pages
      if (lastPage.length < 20) return undefined;
      return pages.length + 1;
    },
    initialPageParam: 1
  });

  const { ref, inView } = useInView();

  // Load more when scrolled to bottom
  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);

  return (
    <div>
      {data?.pages.map((page, i) => (
        <div key={i}>
          {page.map((student: Student) => (
            <StudentCard key={student.id} student={student} />
          ))}
        </div>
      ))}

      {/* Loading trigger */}
      <div ref={ref}>
        {isFetchingNextPage ? (
          <p>Loading more...</p>
        ) : hasNextPage ? (
          <p>Scroll for more</p>
        ) : (
          <p>No more students</p>
        )}
      </div>
    </div>
  );
}
```

---

## Next.js Server Actions {#server-actions}

Server Actions provide a new way to mutate data in Next.js 14+.

### Basic Server Action

```typescript
// app/actions/students.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function createStudent(formData: FormData) {
  const student = {
    name: formData.get('name') as string,
    email: formData.get('email') as string,
    gpa: Number(formData.get('gpa')),
    major: formData.get('major') as string
  };

  const response = await fetch('https://api.college.edu/students', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(student)
  });

  if (!response.ok) {
    throw new Error('Failed to create student');
  }

  // Revalidate the students page
  revalidatePath('/students');

  return await response.json();
}

// app/students/new/page.tsx
import { createStudent } from '@/app/actions/students';

export default function NewStudentPage() {
  return (
    <form action={createStudent}>
      <input name="name" required />
      <input name="email" type="email" required />
      <input name="gpa" type="number" step="0.01" required />
      <input name="major" required />
      <button type="submit">Create Student</button>
    </form>
  );
}
```

### Server Action with useTransition

```typescript
'use client';

import { useTransition } from 'react';
import { createStudent } from '@/app/actions/students';

export function StudentForm() {
  const [isPending, startTransition] = useTransition();

  const handleSubmit = async (formData: FormData) => {
    startTransition(async () => {
      try {
        await createStudent(formData);
        alert('Student created!');
      } catch (error) {
        alert('Failed to create student');
      }
    });
  };

  return (
    <form action={handleSubmit}>
      {/* form fields */}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create'}
      </button>
    </form>
  );
}
```

---

## College Portal Examples {#college-examples}

### Complete Student Management

```typescript
// services/students.ts
import { api } from '@/lib/axios';

export const studentsAPI = {
  getAll: async (filters?: StudentFilters) => {
    const { data } = await api.get<Student[]>('/students', { params: filters });
    return data;
  },

  getById: async (id: string) => {
    const { data } = await api.get<Student>(`/students/${id}`);
    return data;
  },

  create: async (student: CreateStudentDTO) => {
    const { data } = await api.post<Student>('/students', student);
    return data;
  },

  update: async (id: string, updates: Partial<Student>) => {
    const { data } = await api.patch<Student>(`/students/${id}`, updates);
    return data;
  },

  delete: async (id: string) => {
    await api.delete(`/students/${id}`);
  }
};

// components/StudentList.tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { studentsAPI } from '@/services/students';
import { StudentCard } from './StudentCard';

export function StudentList() {
  const { data: students, isLoading, error } = useQuery({
    queryKey: ['students'],
    queryFn: () => studentsAPI.getAll()
  });

  if (isLoading) return <div>Loading students...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
      {students?.map((student) => (
        <StudentCard key={student.id} student={student} />
      ))}
    </div>
  );
}
```

---

## Summary

TanStack Query provides powerful server state management with:

1. **Automatic caching** and background refetching
2. **useQuery** for data fetching
3. **useMutation** for create/update/delete
4. **Query invalidation** for cache updates
5. **Optimistic updates** for better UX
6. **Error handling** and retries
7. **Infinite queries** for pagination

**Key Takeaways:**
- TanStack Query simplifies data fetching
- Use query keys for caching and invalidation
- Optimistic updates improve perceived performance
- Server Actions integrate well with Next.js 14+

---

## Navigation

- **Previous**: [Part 7: Routing & Navigation](./07_Routing_Navigation.md)
- **Next**: [Part 9: Forms & Validation](./09_Forms_Validation.md)
- **[Back to Index](./README.md)**

---

**Author: Saiprasad Hegde**

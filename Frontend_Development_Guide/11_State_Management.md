# Part 11: State Management

**Author: Saiprasad Hegde**

## Table of Contents
1. [Introduction to State Management](#introduction-to-state-management)
2. [State Management Fundamentals](#state-management-fundamentals)
3. [React Context API](#react-context-api)
   - [Creating Context](#creating-context)
   - [Provider and Consumer](#provider-and-consumer)
   - [useContext Hook](#usecontext-hook)
   - [Best Practices](#context-best-practices)
4. [Zustand (Recommended)](#zustand-recommended)
   - [Installation and Setup](#installation-and-setup)
   - [Creating Stores](#creating-stores)
   - [Using Stores in Components](#using-stores-in-components)
   - [Middleware](#middleware)
   - [Persistence](#persistence)
5. [Redux Toolkit](#redux-toolkit)
   - [Setup](#redux-setup)
   - [Slices](#slices)
   - [Store Configuration](#store-configuration)
   - [useSelector and useDispatch](#useselector-and-usedispatch)
6. [When to Use Which Solution](#when-to-use-which-solution)
7. [Server State vs Client State](#server-state-vs-client-state)
8. [Complete Examples](#complete-examples)
9. [Best Practices](#best-practices)
10. [Common Pitfalls](#common-pitfalls)

---

## Introduction to State Management

State management is about organizing and sharing data across your application. As applications grow, managing state becomes increasingly complex.

**Real-World Analogy**: Think of state management like a college's information system:
- **Local State (useState)**: A student's notebook - only they can see it
- **Context API**: Department bulletin board - everyone in the department can see it
- **Zustand/Redux**: College-wide database - everyone can access and update it

In our College Management System, we need to manage:
- User authentication status
- Student data
- Course information
- Enrollment records
- UI preferences (theme, language)

---

## State Management Fundamentals

### Types of State

1. **Local State**: Component-specific (useState, useReducer)
2. **Shared State**: Multiple components need it (Context, Zustand, Redux)
3. **Server State**: Data from APIs (React Query, SWR)
4. **URL State**: Router parameters and query strings
5. **Form State**: Form inputs and validation (React Hook Form)

### State Management Principles

1. **Single Source of Truth**: Don't duplicate state
2. **State is Read-Only**: Don't mutate state directly
3. **Changes with Pure Functions**: Predictable updates
4. **Keep State Minimal**: Derive data when possible
5. **Colocate State**: Keep state close to where it's used

---

## React Context API

Context API is React's built-in solution for sharing state without prop drilling.

**Analogy**: Context is like a school announcement system. Instead of telling each student individually (prop drilling), you make one announcement that everyone hears.

### Creating Context

```typescript
import { createContext, useState, useContext, ReactNode } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
  role: 'student' | 'instructor' | 'admin';
}

interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
  isLoading: boolean;
}

// Create context with default value
const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Provider component
export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(false);

  const login = async (email: string, password: string) => {
    setIsLoading(true);
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) throw new Error('Login failed');

      const userData = await response.json();
      setUser(userData);
      localStorage.setItem('user', JSON.stringify(userData));
    } catch (error) {
      console.error('Login error:', error);
      throw error;
    } finally {
      setIsLoading(false);
    }
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('user');
  };

  const value = {
    user,
    login,
    logout,
    isAuthenticated: !!user,
    isLoading,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

// Custom hook for using context
export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

### Provider and Consumer

**Setting Up Provider** (in app layout):
```typescript
// app/layout.tsx (Next.js)
import { AuthProvider } from '@/contexts/AuthContext';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <AuthProvider>
          {children}
        </AuthProvider>
      </body>
    </html>
  );
}
```

**Using in Components**:
```typescript
'use client';

import { useAuth } from '@/contexts/AuthContext';

export default function LoginPage() {
  const { login, isLoading, isAuthenticated } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      await login(email, password);
      // Redirect on success
    } catch (error) {
      console.error('Login failed');
    }
  };

  if (isAuthenticated) {
    return <div>Already logged in</div>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

### useContext Hook

The useContext hook is how you access context values.

```typescript
import { createContext, useContext, useState, ReactNode } from 'react';

interface Theme {
  mode: 'light' | 'dark';
  primaryColor: string;
}

interface ThemeContextType {
  theme: Theme;
  setTheme: (theme: Theme) => void;
  toggleMode: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>({
    mode: 'light',
    primaryColor: '#2563eb',
  });

  const toggleMode = () => {
    setTheme(prev => ({
      ...prev,
      mode: prev.mode === 'light' ? 'dark' : 'light',
    }));
  };

  return (
    <ThemeContext.Provider value={{ theme, setTheme, toggleMode }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Usage in component
function Header() {
  const { theme, toggleMode } = useTheme();

  return (
    <header style={{ background: theme.primaryColor }}>
      <button onClick={toggleMode}>
        {theme.mode === 'light' ? '🌙' : '☀️'}
      </button>
    </header>
  );
}
```

### Context Best Practices

1. **Split Contexts**: Don't put everything in one context
2. **Memoize Values**: Prevent unnecessary re-renders
3. **Custom Hooks**: Always create useX() hooks
4. **Error Boundaries**: Wrap providers in error boundaries
5. **TypeScript**: Always type your contexts

**Optimized Context with Memo**:
```typescript
import { createContext, useContext, useMemo, useState, ReactNode } from 'react';

export function OptimizedAuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  // Memoize the context value to prevent unnecessary re-renders
  const value = useMemo(
    () => ({
      user,
      login: async (email: string, password: string) => {
        // login logic
      },
      logout: () => setUser(null),
      isAuthenticated: !!user,
    }),
    [user] // Only recreate when user changes
  );

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}
```

---

## Zustand (Recommended)

Zustand is a small, fast, and scalable state management solution. It's simpler than Redux and more powerful than Context.

**Analogy**: Zustand is like a smart filing cabinet. It's organized, easy to access, and automatically updates anyone looking at a file when it changes.

### Installation and Setup

```bash
npm install zustand
```

### Creating Stores

**Auth Store**:
```typescript
// stores/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  name: string;
  email: string;
  role: 'student' | 'instructor' | 'admin';
}

interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  updateUser: (user: Partial<User>) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      isAuthenticated: false,

      login: async (email: string, password: string) => {
        try {
          const response = await fetch('/api/auth/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password }),
          });

          if (!response.ok) throw new Error('Login failed');

          const user = await response.json();
          set({ user, isAuthenticated: true });
        } catch (error) {
          console.error('Login error:', error);
          throw error;
        }
      },

      logout: () => {
        set({ user: null, isAuthenticated: false });
      },

      updateUser: (userData: Partial<User>) => {
        set((state) => ({
          user: state.user ? { ...state.user, ...userData } : null,
        }));
      },
    }),
    {
      name: 'auth-storage', // localStorage key
      partialize: (state) => ({ user: state.user }), // Only persist user
    }
  )
);
```

**Student Store**:
```typescript
// stores/studentStore.ts
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

interface Student {
  id: string;
  name: string;
  email: string;
  department: string;
  gpa: number;
}

interface StudentState {
  students: Student[];
  selectedStudent: Student | null;
  isLoading: boolean;
  error: string | null;

  // Actions
  fetchStudents: () => Promise<void>;
  addStudent: (student: Omit<Student, 'id'>) => Promise<void>;
  updateStudent: (id: string, data: Partial<Student>) => Promise<void>;
  deleteStudent: (id: string) => Promise<void>;
  selectStudent: (student: Student | null) => void;
}

export const useStudentStore = create<StudentState>()(
  devtools(
    (set, get) => ({
      students: [],
      selectedStudent: null,
      isLoading: false,
      error: null,

      fetchStudents: async () => {
        set({ isLoading: true, error: null });
        try {
          const response = await fetch('/api/students');
          if (!response.ok) throw new Error('Failed to fetch students');

          const students = await response.json();
          set({ students, isLoading: false });
        } catch (error) {
          set({
            error: error instanceof Error ? error.message : 'Unknown error',
            isLoading: false
          });
        }
      },

      addStudent: async (studentData) => {
        set({ isLoading: true, error: null });
        try {
          const response = await fetch('/api/students', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(studentData),
          });

          if (!response.ok) throw new Error('Failed to add student');

          const newStudent = await response.json();
          set((state) => ({
            students: [...state.students, newStudent],
            isLoading: false,
          }));
        } catch (error) {
          set({
            error: error instanceof Error ? error.message : 'Unknown error',
            isLoading: false
          });
          throw error;
        }
      },

      updateStudent: async (id, data) => {
        set({ isLoading: true, error: null });
        try {
          const response = await fetch(`/api/students/${id}`, {
            method: 'PATCH',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data),
          });

          if (!response.ok) throw new Error('Failed to update student');

          const updatedStudent = await response.json();
          set((state) => ({
            students: state.students.map((s) =>
              s.id === id ? updatedStudent : s
            ),
            isLoading: false,
          }));
        } catch (error) {
          set({
            error: error instanceof Error ? error.message : 'Unknown error',
            isLoading: false
          });
          throw error;
        }
      },

      deleteStudent: async (id) => {
        set({ isLoading: true, error: null });
        try {
          const response = await fetch(`/api/students/${id}`, {
            method: 'DELETE',
          });

          if (!response.ok) throw new Error('Failed to delete student');

          set((state) => ({
            students: state.students.filter((s) => s.id !== id),
            selectedStudent: state.selectedStudent?.id === id ? null : state.selectedStudent,
            isLoading: false,
          }));
        } catch (error) {
          set({
            error: error instanceof Error ? error.message : 'Unknown error',
            isLoading: false
          });
          throw error;
        }
      },

      selectStudent: (student) => {
        set({ selectedStudent: student });
      },
    }),
    { name: 'StudentStore' } // For Redux DevTools
  )
);
```

### Using Stores in Components

```typescript
'use client';

import { useEffect } from 'react';
import { useStudentStore } from '@/stores/studentStore';

export default function StudentList() {
  // Select only the data you need to prevent unnecessary re-renders
  const students = useStudentStore((state) => state.students);
  const isLoading = useStudentStore((state) => state.isLoading);
  const error = useStudentStore((state) => state.error);
  const fetchStudents = useStudentStore((state) => state.fetchStudents);
  const deleteStudent = useStudentStore((state) => state.deleteStudent);

  useEffect(() => {
    fetchStudents();
  }, [fetchStudents]);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="space-y-4">
      <h2 className="text-2xl font-bold">Students</h2>
      {students.map((student) => (
        <div key={student.id} className="border p-4 rounded">
          <h3 className="font-semibold">{student.name}</h3>
          <p>{student.email}</p>
          <p>Department: {student.department}</p>
          <p>GPA: {student.gpa}</p>
          <button
            onClick={() => deleteStudent(student.id)}
            className="mt-2 bg-red-500 text-white px-4 py-2 rounded"
          >
            Delete
          </button>
        </div>
      ))}
    </div>
  );
}
```

**Selecting Multiple Values Efficiently**:
```typescript
// Option 1: Select individually (re-renders only when specific values change)
const students = useStudentStore((state) => state.students);
const isLoading = useStudentStore((state) => state.isLoading);

// Option 2: Select multiple with shallow equality check
import { shallow } from 'zustand/shallow';

const { students, isLoading, fetchStudents } = useStudentStore(
  (state) => ({
    students: state.students,
    isLoading: state.isLoading,
    fetchStudents: state.fetchStudents,
  }),
  shallow
);
```

### Middleware

Zustand supports middleware for enhanced functionality.

**Persist Middleware** (localStorage):
```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface PreferencesState {
  theme: 'light' | 'dark';
  language: 'en' | 'es' | 'fr';
  notificationsEnabled: boolean;
  setTheme: (theme: 'light' | 'dark') => void;
  setLanguage: (lang: 'en' | 'es' | 'fr') => void;
  toggleNotifications: () => void;
}

export const usePreferencesStore = create<PreferencesState>()(
  persist(
    (set) => ({
      theme: 'light',
      language: 'en',
      notificationsEnabled: true,

      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
      toggleNotifications: () =>
        set((state) => ({ notificationsEnabled: !state.notificationsEnabled })),
    }),
    {
      name: 'user-preferences', // localStorage key
      // Optional: customize storage
      // storage: createJSONStorage(() => sessionStorage),
    }
  )
);
```

**DevTools Middleware**:
```typescript
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

export const useStore = create<State>()(
  devtools(
    (set) => ({
      // ... your state
    }),
    { name: 'MyStore' } // Name shown in Redux DevTools
  )
);
```

**Immer Middleware** (for easier immutable updates):
```typescript
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

interface Course {
  id: string;
  name: string;
  students: string[];
}

interface CourseState {
  courses: Course[];
  addStudentToCourse: (courseId: string, studentId: string) => void;
}

export const useCourseStore = create<CourseState>()(
  immer((set) => ({
    courses: [],

    addStudentToCourse: (courseId, studentId) =>
      set((state) => {
        // With immer, you can mutate draft state directly
        const course = state.courses.find((c) => c.id === courseId);
        if (course) {
          course.students.push(studentId);
        }
      }),
  }))
);
```

### Persistence

**Advanced Persistence with Versioning**:
```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface StudentState {
  students: Student[];
  version: number;
}

export const useStudentStore = create<StudentState>()(
  persist(
    (set) => ({
      students: [],
      version: 1,
      // ... actions
    }),
    {
      name: 'student-storage',
      version: 1,
      migrate: (persistedState: any, version: number) => {
        // Migrate old data to new structure
        if (version === 0) {
          // Upgrade from version 0 to 1
          return {
            ...persistedState,
            version: 1,
            // Transform data if needed
          };
        }
        return persistedState;
      },
      partialize: (state) => ({
        // Only persist students, not loading states
        students: state.students,
        version: state.version,
      }),
    }
  )
);
```

---

## Redux Toolkit

Redux Toolkit is the official, opinionated toolkit for Redux. Use it for very complex applications with intricate state logic.

**Analogy**: Redux is like a college's central administration building. Everything goes through proper channels, paperwork is filed, and there's a clear chain of command.

### Redux Setup

```bash
npm install @reduxjs/toolkit react-redux
```

**Store Setup** (store/index.ts):
```typescript
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './slices/authSlice';
import studentsReducer from './slices/studentsSlice';
import coursesReducer from './slices/coursesSlice';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    students: studentsReducer,
    courses: coursesReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        // Ignore these action types
        ignoredActions: ['persist/PERSIST'],
      },
    }),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

**Provider Setup** (app/layout.tsx):
```typescript
'use client';

import { Provider } from 'react-redux';
import { store } from '@/store';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Provider store={store}>
          {children}
        </Provider>
      </body>
    </html>
  );
}
```

### Slices

Slices contain reducer logic and actions for a specific feature.

**Auth Slice**:
```typescript
// store/slices/authSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: string;
  name: string;
  email: string;
  role: 'student' | 'instructor' | 'admin';
}

interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
}

const initialState: AuthState = {
  user: null,
  isAuthenticated: false,
  isLoading: false,
  error: null,
};

// Async thunk for login
export const loginAsync = createAsyncThunk(
  'auth/login',
  async ({ email, password }: { email: string; password: string }, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) {
        const error = await response.json();
        return rejectWithValue(error.message);
      }

      const user = await response.json();
      return user;
    } catch (error) {
      return rejectWithValue('Network error');
    }
  }
);

export const logoutAsync = createAsyncThunk('auth/logout', async () => {
  await fetch('/api/auth/logout', { method: 'POST' });
});

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    // Synchronous actions
    setUser: (state, action: PayloadAction<User>) => {
      state.user = action.payload;
      state.isAuthenticated = true;
    },
    clearUser: (state) => {
      state.user = null;
      state.isAuthenticated = false;
    },
  },
  extraReducers: (builder) => {
    // Login
    builder.addCase(loginAsync.pending, (state) => {
      state.isLoading = true;
      state.error = null;
    });
    builder.addCase(loginAsync.fulfilled, (state, action) => {
      state.user = action.payload;
      state.isAuthenticated = true;
      state.isLoading = false;
      state.error = null;
    });
    builder.addCase(loginAsync.rejected, (state, action) => {
      state.isLoading = false;
      state.error = action.payload as string;
    });

    // Logout
    builder.addCase(logoutAsync.fulfilled, (state) => {
      state.user = null;
      state.isAuthenticated = false;
    });
  },
});

export const { setUser, clearUser } = authSlice.actions;
export default authSlice.reducer;
```

**Students Slice**:
```typescript
// store/slices/studentsSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

interface Student {
  id: string;
  name: string;
  email: string;
  department: string;
  gpa: number;
}

interface StudentsState {
  students: Student[];
  selectedStudent: Student | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: StudentsState = {
  students: [],
  selectedStudent: null,
  isLoading: false,
  error: null,
};

// Async thunks
export const fetchStudents = createAsyncThunk(
  'students/fetchAll',
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/students');
      if (!response.ok) throw new Error('Failed to fetch students');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error instanceof Error ? error.message : 'Unknown error');
    }
  }
);

export const addStudent = createAsyncThunk(
  'students/add',
  async (studentData: Omit<Student, 'id'>, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/students', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(studentData),
      });
      if (!response.ok) throw new Error('Failed to add student');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error instanceof Error ? error.message : 'Unknown error');
    }
  }
);

export const updateStudent = createAsyncThunk(
  'students/update',
  async ({ id, data }: { id: string; data: Partial<Student> }, { rejectWithValue }) => {
    try {
      const response = await fetch(`/api/students/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });
      if (!response.ok) throw new Error('Failed to update student');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error instanceof Error ? error.message : 'Unknown error');
    }
  }
);

export const deleteStudent = createAsyncThunk(
  'students/delete',
  async (id: string, { rejectWithValue }) => {
    try {
      const response = await fetch(`/api/students/${id}`, {
        method: 'DELETE',
      });
      if (!response.ok) throw new Error('Failed to delete student');
      return id;
    } catch (error) {
      return rejectWithValue(error instanceof Error ? error.message : 'Unknown error');
    }
  }
);

const studentsSlice = createSlice({
  name: 'students',
  initialState,
  reducers: {
    selectStudent: (state, action: PayloadAction<Student | null>) => {
      state.selectedStudent = action.payload;
    },
    clearError: (state) => {
      state.error = null;
    },
  },
  extraReducers: (builder) => {
    // Fetch students
    builder.addCase(fetchStudents.pending, (state) => {
      state.isLoading = true;
      state.error = null;
    });
    builder.addCase(fetchStudents.fulfilled, (state, action) => {
      state.students = action.payload;
      state.isLoading = false;
    });
    builder.addCase(fetchStudents.rejected, (state, action) => {
      state.isLoading = false;
      state.error = action.payload as string;
    });

    // Add student
    builder.addCase(addStudent.fulfilled, (state, action) => {
      state.students.push(action.payload);
    });

    // Update student
    builder.addCase(updateStudent.fulfilled, (state, action) => {
      const index = state.students.findIndex((s) => s.id === action.payload.id);
      if (index !== -1) {
        state.students[index] = action.payload;
      }
    });

    // Delete student
    builder.addCase(deleteStudent.fulfilled, (state, action) => {
      state.students = state.students.filter((s) => s.id !== action.payload);
      if (state.selectedStudent?.id === action.payload) {
        state.selectedStudent = null;
      }
    });
  },
});

export const { selectStudent, clearError } = studentsSlice.actions;
export default studentsSlice.reducer;
```

### Store Configuration

**Custom Hooks** (store/hooks.ts):
```typescript
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './index';

// Use throughout your app instead of plain `useDispatch` and `useSelector`
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

### useSelector and useDispatch

**Using Redux in Components**:
```typescript
'use client';

import { useEffect } from 'react';
import { useAppDispatch, useAppSelector } from '@/store/hooks';
import { fetchStudents, deleteStudent, selectStudent } from '@/store/slices/studentsSlice';

export default function StudentList() {
  const dispatch = useAppDispatch();

  // Select state from store
  const students = useAppSelector((state) => state.students.students);
  const isLoading = useAppSelector((state) => state.students.isLoading);
  const error = useAppSelector((state) => state.students.error);

  useEffect(() => {
    dispatch(fetchStudents());
  }, [dispatch]);

  const handleDelete = async (id: string) => {
    if (confirm('Are you sure?')) {
      await dispatch(deleteStudent(id));
    }
  };

  const handleSelect = (student: Student) => {
    dispatch(selectStudent(student));
  };

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="space-y-4">
      <h2 className="text-2xl font-bold">Students</h2>
      {students.map((student) => (
        <div key={student.id} className="border p-4 rounded">
          <h3 className="font-semibold">{student.name}</h3>
          <p>{student.email}</p>
          <div className="flex gap-2 mt-2">
            <button
              onClick={() => handleSelect(student)}
              className="bg-blue-500 text-white px-4 py-2 rounded"
            >
              Select
            </button>
            <button
              onClick={() => handleDelete(student.id)}
              className="bg-red-500 text-white px-4 py-2 rounded"
            >
              Delete
            </button>
          </div>
        </div>
      ))}
    </div>
  );
}
```

---

## When to Use Which Solution

| Solution | Best For | Complexity | Bundle Size | TypeScript |
|----------|----------|------------|-------------|------------|
| **useState** | Local component state | Very Low | 0 KB | Excellent |
| **Context API** | Theme, Auth, Simple shared state | Low | 0 KB | Good |
| **Zustand** | Most applications, Moderate complexity | Low-Medium | 3 KB | Excellent |
| **Redux Toolkit** | Complex apps, Team standards | Medium-High | 12 KB | Excellent |

**Decision Tree**:
```
Is state only used in one component?
├─ Yes → useState
└─ No → Is it server data?
    ├─ Yes → React Query/SWR
    └─ No → How complex is the state logic?
        ├─ Simple (auth, theme) → Context API
        ├─ Moderate → Zustand
        └─ Complex → Redux Toolkit
```

**Recommendations for College Management System**:
- **Authentication**: Context API or Zustand
- **User Preferences**: Zustand with persistence
- **Student/Course Data**: React Query (server state)
- **Complex Workflows**: Redux Toolkit
- **UI State**: Local useState or Zustand

---

## Server State vs Client State

**Client State**: Lives in the browser (theme, form inputs, UI toggles)
**Server State**: Lives on the server (students, courses, enrollments)

**Don't use Redux/Zustand for server state!** Use React Query or SWR instead.

**Example with React Query**:
```typescript
'use client';

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

interface Student {
  id: string;
  name: string;
  email: string;
}

export default function StudentList() {
  const queryClient = useQueryClient();

  // Fetch students
  const { data: students, isLoading, error } = useQuery({
    queryKey: ['students'],
    queryFn: async () => {
      const response = await fetch('/api/students');
      if (!response.ok) throw new Error('Failed to fetch students');
      return response.json();
    },
  });

  // Delete mutation
  const deleteMutation = useMutation({
    mutationFn: async (id: string) => {
      const response = await fetch(`/api/students/${id}`, { method: 'DELETE' });
      if (!response.ok) throw new Error('Failed to delete');
    },
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['students'] });
    },
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {students.map((student: Student) => (
        <div key={student.id}>
          <h3>{student.name}</h3>
          <button onClick={() => deleteMutation.mutate(student.id)}>
            Delete
          </button>
        </div>
      ))}
    </div>
  );
}
```

---

## Complete Examples

### Complete College App with Zustand

**App Structure**:
```
stores/
├── authStore.ts
├── preferencesStore.ts
└── notificationStore.ts
```

**stores/authStore.ts**:
```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  name: string;
  email: string;
  role: 'student' | 'instructor' | 'admin';
}

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  refreshToken: () => Promise<void>;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      token: null,
      isAuthenticated: false,

      login: async (email, password) => {
        const response = await fetch('/api/auth/login', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ email, password }),
        });

        if (!response.ok) throw new Error('Login failed');

        const { user, token } = await response.json();
        set({ user, token, isAuthenticated: true });
      },

      logout: () => {
        set({ user: null, token: null, isAuthenticated: false });
      },

      refreshToken: async () => {
        const { token } = get();
        if (!token) return;

        try {
          const response = await fetch('/api/auth/refresh', {
            headers: { Authorization: `Bearer ${token}` },
          });

          if (!response.ok) throw new Error('Refresh failed');

          const { token: newToken } = await response.json();
          set({ token: newToken });
        } catch (error) {
          // Token expired, logout
          get().logout();
        }
      },
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({ user: state.user, token: state.token }),
    }
  )
);
```

**stores/notificationStore.ts**:
```typescript
import { create } from 'zustand';

interface Notification {
  id: string;
  type: 'success' | 'error' | 'warning' | 'info';
  message: string;
  duration?: number;
}

interface NotificationState {
  notifications: Notification[];
  addNotification: (notification: Omit<Notification, 'id'>) => void;
  removeNotification: (id: string) => void;
  clearAll: () => void;
}

export const useNotificationStore = create<NotificationState>((set) => ({
  notifications: [],

  addNotification: (notification) => {
    const id = Math.random().toString(36).substring(7);
    const newNotification = { ...notification, id };

    set((state) => ({
      notifications: [...state.notifications, newNotification],
    }));

    // Auto-remove after duration
    if (notification.duration) {
      setTimeout(() => {
        set((state) => ({
          notifications: state.notifications.filter((n) => n.id !== id),
        }));
      }, notification.duration);
    }
  },

  removeNotification: (id) => {
    set((state) => ({
      notifications: state.notifications.filter((n) => n.id !== id),
    }));
  },

  clearAll: () => {
    set({ notifications: [] });
  },
}));
```

**Using Multiple Stores**:
```typescript
'use client';

import { useAuthStore } from '@/stores/authStore';
import { useNotificationStore } from '@/stores/notificationStore';
import { usePreferencesStore } from '@/stores/preferencesStore';

export default function Dashboard() {
  const user = useAuthStore((state) => state.user);
  const logout = useAuthStore((state) => state.logout);
  const theme = usePreferencesStore((state) => state.theme);
  const addNotification = useNotificationStore((state) => state.addNotification);

  const handleLogout = () => {
    logout();
    addNotification({
      type: 'success',
      message: 'Logged out successfully',
      duration: 3000,
    });
  };

  return (
    <div className={theme === 'dark' ? 'dark' : ''}>
      <h1>Welcome, {user?.name}!</h1>
      <button onClick={handleLogout}>Logout</button>
    </div>
  );
}
```

---

## Best Practices

1. **Keep State Minimal**: Derive data when possible
2. **Normalize Data**: Avoid nested structures
3. **Split Stores**: One store per domain (auth, students, courses)
4. **Use Selectors**: Select only what you need
5. **Async in Actions**: Keep components clean
6. **Error Handling**: Always handle errors in state
7. **TypeScript**: Type everything
8. **DevTools**: Use Redux DevTools for debugging
9. **Persist Wisely**: Only persist what's necessary
10. **Test State Logic**: Unit test stores/reducers

---

## Common Pitfalls

1. **Over-Engineering**: Using Redux when Context would suffice
2. **State Duplication**: Keeping same data in multiple places
3. **Not Using Selectors**: Causing unnecessary re-renders
4. **Mutating State**: Always return new objects
5. **Too Much in State**: Store UI state locally when possible
6. **Poor Organization**: Mixing unrelated state
7. **Ignoring Loading States**: Always track loading/error
8. **No TypeScript**: Losing type safety
9. **Persisting Everything**: Large localStorage usage
10. **Mixing Server and Client State**: Use React Query for server data

---

## Next Steps

Now that you understand state management, you're ready to learn about **Part 12: Performance Optimization**. You'll learn how to make your applications blazingly fast!

---

**Navigation:**
- **Previous**: [Part 10: Styling Solutions](10_Styling_Solutions.md)
- **Next**: [Part 12: Performance Optimization](12_Performance_Optimization.md)
- **[Back to Index](README.md)**

---

**Author: Saiprasad Hegde**

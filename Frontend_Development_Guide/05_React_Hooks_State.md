# Part 5: React Hooks & State

**Author: Saiprasad Hegde**

## Table of Contents
1. [Introduction to Hooks](#introduction)
2. [useState - Local State Management](#usestate)
3. [useEffect - Side Effects & Lifecycle](#useeffect)
4. [useRef - DOM Refs & Mutable Values](#useref)
5. [useCallback - Memoizing Functions](#usecallback)
6. [useMemo - Memoizing Values](#usememo)
7. [Custom Hooks](#custom-hooks)
8. [Rules of Hooks](#rules-of-hooks)
9. [College Portal Examples](#college-examples)

---

## Introduction to Hooks {#introduction}

Hooks are functions that let you "hook into" React features like state and lifecycle from functional components.

**Analogy**: Think of hooks like power tools. Before hooks, you needed different tools (classes) for different jobs. Hooks let you use the same type of tool (functions) with different attachments (hooks) for different tasks.

### Why Hooks?

**Before Hooks (Class Components):**
```typescript
class StudentCard extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  componentDidMount() {
    document.title = `Clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `Clicked ${this.state.count} times`;
  }

  render() {
    return (
      <button onClick={() => this.setState({ count: this.state.count + 1 })}>
        Click me
      </button>
    );
  }
}
```

**With Hooks (Functional Components):**
```typescript
const StudentCard: FC = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Clicked ${count} times`;
  }, [count]);

  return <button onClick={() => setCount(count + 1)}>Click me</button>;
};
```

---

## useState - Local State Management {#usestate}

`useState` lets you add state to functional components.

**Analogy**: Think of `useState` like a notepad. You can write a value on it (state), and whenever you change the value, React re-renders your component with the new value.

### Basic Usage

```typescript
import { useState } from 'react';

const Counter: FC = () => {
  // Declare state variable
  const [count, setCount] = useState(0);
  //    [current value, setter function] = useState(initial value)

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
};
```

### State with Different Types

```typescript
// Number
const [gpa, setGPA] = useState<number>(0);

// String
const [name, setName] = useState<string>('');

// Boolean
const [isEnrolled, setIsEnrolled] = useState<boolean>(false);

// Array
const [courses, setCourses] = useState<string[]>([]);

// Object
interface Student {
  id: string;
  name: string;
  email: string;
}

const [student, setStudent] = useState<Student>({
  id: '',
  name: '',
  email: ''
});

// Null/Undefined
const [selectedCourse, setSelectedCourse] = useState<Course | null>(null);
```

### Updating State

```typescript
const StudentForm: FC = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    gpa: 0,
    major: ''
  });

  // ❌ WRONG: Mutating state directly
  const handleWrong = () => {
    formData.name = 'Alice'; // Don't do this!
  };

  // ✅ CORRECT: Creating new object
  const handleNameChange = (name: string) => {
    setFormData({
      ...formData,
      name // Shorthand for name: name
    });
  };

  // ✅ CORRECT: Updating multiple fields
  const handleChange = (field: string, value: any) => {
    setFormData(prev => ({
      ...prev,
      [field]: value
    }));
  };

  return (
    <div>
      <input
        value={formData.name}
        onChange={(e) => handleNameChange(e.target.value)}
        placeholder="Name"
      />
      <input
        value={formData.email}
        onChange={(e) => handleChange('email', e.target.value)}
        placeholder="Email"
      />
    </div>
  );
};
```

### Functional Updates

```typescript
const CourseEnrollment: FC = () => {
  const [enrolledCourses, setEnrolledCourses] = useState<string[]>([]);

  // When new state depends on previous state, use functional update
  const addCourse = (courseId: string) => {
    setEnrolledCourses(prev => [...prev, courseId]);
  };

  const removeCourse = (courseId: string) => {
    setEnrolledCourses(prev => prev.filter(id => id !== courseId));
  };

  const clearCourses = () => {
    setEnrolledCourses([]);
  };

  return (
    <div>
      <p>Enrolled: {enrolledCourses.length} courses</p>
      <button onClick={() => addCourse('CS101')}>Add CS101</button>
      <button onClick={() => removeCourse('CS101')}>Remove CS101</button>
      <button onClick={clearCourses}>Clear All</button>
    </div>
  );
};
```

### Lazy Initial State

```typescript
// Expensive computation
const getInitialStudents = () => {
  console.log('Computing initial students...');
  return JSON.parse(localStorage.getItem('students') || '[]');
};

const StudentList: FC = () => {
  // ❌ BAD: Function runs on every render
  const [students, setStudents] = useState(getInitialStudents());

  // ✅ GOOD: Function runs only once on mount
  const [students2, setStudents2] = useState(() => getInitialStudents());

  return <div>{/* ... */}</div>;
};
```

### Multiple State Variables

```typescript
const StudentDashboard: FC = () => {
  // Multiple separate state variables
  const [students, setStudents] = useState<Student[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  const [searchTerm, setSearchTerm] = useState('');
  const [filterStatus, setFilterStatus] = useState<'all' | 'honors'>('all');

  // Or group related state together
  const [state, setState] = useState({
    students: [] as Student[],
    loading: true,
    error: null as Error | null
  });

  return <div>{/* ... */}</div>;
};
```

### Real-World Example: Search and Filter

```typescript
interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
  major: string;
}

const StudentSearch: FC<{ students: Student[] }> = ({ students }) => {
  const [searchTerm, setSearchTerm] = useState('');
  const [minGPA, setMinGPA] = useState(0);
  const [selectedMajor, setSelectedMajor] = useState<string>('all');

  // Derived state (computed from other state/props)
  const filteredStudents = students.filter(student => {
    const matchesSearch = student.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
                         student.email.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesGPA = student.gpa >= minGPA;
    const matchesMajor = selectedMajor === 'all' || student.major === selectedMajor;

    return matchesSearch && matchesGPA && matchesMajor;
  });

  const majors = Array.from(new Set(students.map(s => s.major)));

  return (
    <div className="space-y-4">
      {/* Search Input */}
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search by name or email..."
        className="w-full px-4 py-2 border rounded-lg"
      />

      {/* Filters */}
      <div className="flex gap-4">
        <div>
          <label>Min GPA: {minGPA.toFixed(1)}</label>
          <input
            type="range"
            min="0"
            max="4"
            step="0.1"
            value={minGPA}
            onChange={(e) => setMinGPA(parseFloat(e.target.value))}
          />
        </div>

        <select
          value={selectedMajor}
          onChange={(e) => setSelectedMajor(e.target.value)}
          className="px-4 py-2 border rounded-lg"
        >
          <option value="all">All Majors</option>
          {majors.map(major => (
            <option key={major} value={major}>{major}</option>
          ))}
        </select>
      </div>

      {/* Results */}
      <p className="text-sm text-gray-600">
        Showing {filteredStudents.length} of {students.length} students
      </p>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {filteredStudents.map(student => (
          <StudentCard key={student.id} student={student} />
        ))}
      </div>
    </div>
  );
};
```

---

## useEffect - Side Effects & Lifecycle {#useeffect}

`useEffect` lets you perform side effects in functional components (data fetching, subscriptions, DOM manipulation).

**Analogy**: Think of `useEffect` like a security guard. The guard watches specific things (dependencies) and takes action (runs effect) when those things change.

### Basic Usage

```typescript
import { useEffect } from 'react';

const Component: FC = () => {
  useEffect(() => {
    // Effect code runs after render
    console.log('Component rendered');

    // Optional cleanup function
    return () => {
      console.log('Component will unmount or effect will re-run');
    };
  }, []); // Dependency array

  return <div>Hello</div>;
};
```

### Dependency Array Patterns

```typescript
const EffectExamples: FC = () => {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // 1. Run on every render (no dependency array)
  useEffect(() => {
    console.log('Runs on every render');
  });

  // 2. Run only once on mount (empty dependency array)
  useEffect(() => {
    console.log('Runs once on mount');
    return () => console.log('Cleanup on unmount');
  }, []);

  // 3. Run when specific dependencies change
  useEffect(() => {
    console.log('Runs when count changes:', count);
  }, [count]);

  // 4. Run when any of multiple dependencies change
  useEffect(() => {
    console.log('Runs when count or name changes');
  }, [count, name]);

  return <div>{/* ... */}</div>;
};
```

### Data Fetching

```typescript
interface Student {
  id: string;
  name: string;
  email: string;
  gpa: number;
}

const StudentProfile: FC<{ studentId: string }> = ({ studentId }) => {
  const [student, setStudent] = useState<Student | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    // Reset state when studentId changes
    setLoading(true);
    setError(null);

    // Fetch student data
    fetch(`https://api.college.edu/students/${studentId}`)
      .then(response => {
        if (!response.ok) throw new Error('Failed to fetch');
        return response.json();
      })
      .then(data => {
        setStudent(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [studentId]); // Re-fetch when studentId changes

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!student) return <div>No student found</div>;

  return (
    <div>
      <h2>{student.name}</h2>
      <p>{student.email}</p>
      <p>GPA: {student.gpa}</p>
    </div>
  );
};
```

### Async/Await in useEffect

```typescript
const StudentList: FC = () => {
  const [students, setStudents] = useState<Student[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Option 1: Create async function inside effect
    const fetchStudents = async () => {
      try {
        setLoading(true);
        const response = await fetch('https://api.college.edu/students');
        const data = await response.json();
        setStudents(data);
      } catch (error) {
        console.error('Failed to fetch students:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchStudents();
  }, []);

  // Option 2: IIFE (Immediately Invoked Function Expression)
  useEffect(() => {
    (async () => {
      const response = await fetch('https://api.college.edu/students');
      const data = await response.json();
      setStudents(data);
    })();
  }, []);

  return <div>{/* ... */}</div>;
};
```

### Cleanup Functions

```typescript
// Example 1: Event Listeners
const WindowSize: FC = () => {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    // Add event listener
    window.addEventListener('resize', handleResize);

    // Cleanup: Remove event listener
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  return <div>Window size: {size.width} x {size.height}</div>;
};

// Example 2: Timers
const AutoSave: FC<{ formData: any }> = ({ formData }) => {
  const [lastSaved, setLastSaved] = useState<Date | null>(null);

  useEffect(() => {
    // Auto-save every 30 seconds
    const interval = setInterval(() => {
      console.log('Auto-saving...', formData);
      // API call to save data
      setLastSaved(new Date());
    }, 30000);

    // Cleanup: Clear interval
    return () => clearInterval(interval);
  }, [formData]);

  return (
    <div>
      {lastSaved && <p>Last saved: {lastSaved.toLocaleTimeString()}</p>}
    </div>
  );
};

// Example 3: Subscriptions
const RealtimeEnrollment: FC<{ courseId: string }> = ({ courseId }) => {
  const [enrolled, setEnrolled] = useState(0);

  useEffect(() => {
    // Subscribe to real-time updates
    const unsubscribe = subscribeToEnrollment(courseId, (count) => {
      setEnrolled(count);
    });

    // Cleanup: Unsubscribe
    return unsubscribe;
  }, [courseId]);

  return <div>Currently enrolled: {enrolled}</div>;
};
```

### Abort Controller (Cancel Requests)

```typescript
const SearchStudents: FC = () => {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<Student[]>([]);

  useEffect(() => {
    // Don't search if query is empty
    if (!query) {
      setResults([]);
      return;
    }

    // Create AbortController for this request
    const controller = new AbortController();

    const searchStudents = async () => {
      try {
        const response = await fetch(
          `https://api.college.edu/students/search?q=${query}`,
          { signal: controller.signal }
        );
        const data = await response.json();
        setResults(data);
      } catch (error: any) {
        if (error.name === 'AbortError') {
          console.log('Request cancelled');
        } else {
          console.error('Search failed:', error);
        }
      }
    };

    searchStudents();

    // Cleanup: Abort request if component unmounts or query changes
    return () => controller.abort();
  }, [query]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search students..."
      />
      <div>
        {results.map(student => (
          <StudentCard key={student.id} student={student} />
        ))}
      </div>
    </div>
  );
};
```

### Document Title Updates

```typescript
const StudentDashboard: FC<{ student: Student }> = ({ student }) => {
  useEffect(() => {
    document.title = `${student.name} - Dashboard`;

    // Cleanup: Reset title on unmount
    return () => {
      document.title = 'College Portal';
    };
  }, [student.name]);

  return <div>{/* ... */}</div>;
};
```

---

## useRef - DOM Refs & Mutable Values {#useref}

`useRef` creates a mutable reference that persists across renders without causing re-renders.

**Analogy**: Think of `useRef` like a box where you can store things. Unlike state, putting new things in the box doesn't trigger a re-render.

### DOM References

```typescript
import { useRef, useEffect } from 'react';

const AutoFocusInput: FC = () => {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    // Focus input on mount
    inputRef.current?.focus();
  }, []);

  return (
    <input
      ref={inputRef}
      type="text"
      placeholder="This will auto-focus"
    />
  );
};
```

### Accessing DOM Elements

```typescript
const ScrollToBottom: FC = () => {
  const messagesEndRef = useRef<HTMLDivElement>(null);
  const [messages, setMessages] = useState<string[]>([]);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  useEffect(() => {
    scrollToBottom();
  }, [messages]);

  return (
    <div className="messages-container">
      {messages.map((msg, i) => (
        <div key={i}>{msg}</div>
      ))}
      <div ref={messagesEndRef} />
    </div>
  );
};
```

### Storing Mutable Values

```typescript
const Timer: FC = () => {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  const intervalRef = useRef<NodeJS.Timeout | null>(null);

  const start = () => {
    if (intervalRef.current) return; // Already running

    setIsRunning(true);
    intervalRef.current = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
  };

  const stop = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
    setIsRunning(false);
  };

  const reset = () => {
    stop();
    setSeconds(0);
  };

  useEffect(() => {
    // Cleanup on unmount
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);

  return (
    <div>
      <p>Time: {seconds}s</p>
      <button onClick={start} disabled={isRunning}>Start</button>
      <button onClick={stop} disabled={!isRunning}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
};
```

### Previous Value Tracking

```typescript
const usePrevious = <T,>(value: T): T | undefined => {
  const ref = useRef<T>();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
};

// Usage
const StudentGPA: FC<{ student: Student }> = ({ student }) => {
  const previousGPA = usePrevious(student.gpa);

  return (
    <div>
      <p>Current GPA: {student.gpa}</p>
      {previousGPA !== undefined && (
        <p>
          Previous GPA: {previousGPA}
          {student.gpa > previousGPA ? ' ⬆️' : ' ⬇️'}
        </p>
      )}
    </div>
  );
};
```

### Avoiding Unnecessary Re-renders

```typescript
const RenderCount: FC = () => {
  const [count, setCount] = useState(0);
  const renderCount = useRef(0);

  // Increment on every render (doesn't cause re-render)
  renderCount.current += 1;

  return (
    <div>
      <p>Count: {count}</p>
      <p>Render count: {renderCount.current}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

---

## useCallback - Memoizing Functions {#usecallback}

`useCallback` returns a memoized version of a callback function that only changes if dependencies change.

**Analogy**: Think of `useCallback` like saving a phone number in contacts. Instead of typing the number every time, you save it once and reuse the same saved version.

### Basic Usage

```typescript
import { useCallback } from 'react';

const StudentList: FC = () => {
  const [students, setStudents] = useState<Student[]>([]);

  // Without useCallback: New function created on every render
  const handleDelete = (id: string) => {
    setStudents(prev => prev.filter(s => s.id !== id));
  };

  // With useCallback: Same function reference unless dependencies change
  const handleDeleteMemoized = useCallback((id: string) => {
    setStudents(prev => prev.filter(s => s.id !== id));
  }, []); // Empty deps: function never changes

  return (
    <div>
      {students.map(student => (
        <StudentCard
          key={student.id}
          student={student}
          onDelete={handleDeleteMemoized}
        />
      ))}
    </div>
  );
};
```

### With Dependencies

```typescript
const SearchableList: FC = () => {
  const [students, setStudents] = useState<Student[]>([]);
  const [searchTerm, setSearchTerm] = useState('');

  // Function depends on searchTerm
  const filterStudents = useCallback(() => {
    return students.filter(student =>
      student.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [students, searchTerm]); // Re-create when these change

  return <div>{/* ... */}</div>;
};
```

### Performance Optimization

```typescript
import React, { memo } from 'react';

// Memoized child component (only re-renders if props change)
const StudentCard = memo<StudentCardProps>(({ student, onDelete }) => {
  console.log('StudentCard rendered:', student.name);
  return (
    <div>
      <h3>{student.name}</h3>
      <button onClick={() => onDelete(student.id)}>Delete</button>
    </div>
  );
});

const StudentList: FC = () => {
  const [students, setStudents] = useState<Student[]>([]);
  const [count, setCount] = useState(0);

  // ❌ Without useCallback: StudentCard re-renders even when student doesn't change
  const handleDelete = (id: string) => {
    setStudents(prev => prev.filter(s => s.id !== id));
  };

  // ✅ With useCallback: StudentCard only re-renders when student changes
  const handleDeleteOptimized = useCallback((id: string) => {
    setStudents(prev => prev.filter(s => s.id !== id));
  }, []);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Unrelated count: {count}
      </button>
      {students.map(student => (
        <StudentCard
          key={student.id}
          student={student}
          onDelete={handleDeleteOptimized}
        />
      ))}
    </div>
  );
};
```

### Event Handlers

```typescript
const EnrollmentForm: FC = () => {
  const [formData, setFormData] = useState({
    studentId: '',
    courseId: '',
    semester: ''
  });

  const handleChange = useCallback((field: keyof typeof formData, value: string) => {
    setFormData(prev => ({ ...prev, [field]: value }));
  }, []);

  const handleSubmit = useCallback(async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      await fetch('/api/enrollments', {
        method: 'POST',
        body: JSON.stringify(formData)
      });
    } catch (error) {
      console.error('Enrollment failed:', error);
    }
  }, [formData]);

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.studentId}
        onChange={(e) => handleChange('studentId', e.target.value)}
      />
      {/* ... */}
    </form>
  );
};
```

---

## useMemo - Memoizing Values {#usememo}

`useMemo` returns a memoized value that only recomputes when dependencies change.

**Analogy**: Think of `useMemo` like caching search results. Instead of searching the entire database every time, you save the results and only search again when the query changes.

### Basic Usage

```typescript
import { useMemo } from 'react';

const StudentStats: FC<{ students: Student[] }> = ({ students }) => {
  // Expensive calculation
  const stats = useMemo(() => {
    console.log('Calculating stats...');
    return {
      total: students.length,
      averageGPA: students.reduce((sum, s) => sum + s.gpa, 0) / students.length,
      honorsCount: students.filter(s => s.gpa >= 3.5).length
    };
  }, [students]); // Only recalculate when students array changes

  return (
    <div>
      <p>Total Students: {stats.total}</p>
      <p>Average GPA: {stats.averageGPA.toFixed(2)}</p>
      <p>Honors Students: {stats.honorsCount}</p>
    </div>
  );
};
```

### Filtering and Sorting

```typescript
const FilteredStudentList: FC = () => {
  const [students, setStudents] = useState<Student[]>([]);
  const [searchTerm, setSearchTerm] = useState('');
  const [sortBy, setSortBy] = useState<'name' | 'gpa'>('name');

  const filteredAndSorted = useMemo(() => {
    console.log('Filtering and sorting...');

    // Filter
    let result = students.filter(student =>
      student.name.toLowerCase().includes(searchTerm.toLowerCase())
    );

    // Sort
    result.sort((a, b) => {
      if (sortBy === 'name') {
        return a.name.localeCompare(b.name);
      } else {
        return b.gpa - a.gpa; // Descending GPA
      }
    });

    return result;
  }, [students, searchTerm, sortBy]);

  return (
    <div>
      <input
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value as any)}>
        <option value="name">Sort by Name</option>
        <option value="gpa">Sort by GPA</option>
      </select>

      {filteredAndSorted.map(student => (
        <StudentCard key={student.id} student={student} />
      ))}
    </div>
  );
};
```

### Complex Computations

```typescript
const CourseRecommendations: FC<{ student: Student; allCourses: Course[] }> = ({
  student,
  allCourses
}) => {
  const recommendations = useMemo(() => {
    console.log('Calculating recommendations...');

    // Complex algorithm to recommend courses
    return allCourses
      .filter(course => {
        // Not already enrolled
        if (student.enrolledCourses.includes(course.id)) return false;

        // Has capacity
        if (course.enrolled >= course.capacity) return false;

        // Matches major (simplified)
        if (course.code.startsWith(student.major.substring(0, 2))) return true;

        return false;
      })
      .sort((a, b) => b.credits - a.credits) // Prefer higher credit courses
      .slice(0, 5); // Top 5 recommendations
  }, [student, allCourses]);

  return (
    <div>
      <h3>Recommended Courses</h3>
      {recommendations.map(course => (
        <CourseCard key={course.id} course={course} />
      ))}
    </div>
  );
};
```

### useMemo vs useCallback

```typescript
// useMemo: Memoize a VALUE
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);

// useCallback: Memoize a FUNCTION
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

// They're related:
// useCallback(fn, deps) === useMemo(() => fn, deps)
```

---

## Custom Hooks {#custom-hooks}

Custom hooks let you extract component logic into reusable functions.

**Analogy**: Custom hooks are like creating your own power tool attachments. You combine basic tools (built-in hooks) to create specialized tools for specific jobs.

### Basic Custom Hook

```typescript
// hooks/useToggle.ts
const useToggle = (initialValue: boolean = false): [boolean, () => void] => {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => {
    setValue(v => !v);
  }, []);

  return [value, toggle];
};

// Usage
const Modal: FC = () => {
  const [isOpen, toggleOpen] = useToggle(false);

  return (
    <div>
      <button onClick={toggleOpen}>Open Modal</button>
      {isOpen && (
        <div className="modal">
          <p>Modal Content</p>
          <button onClick={toggleOpen}>Close</button>
        </div>
      )}
    </div>
  );
};
```

### Data Fetching Hook

```typescript
// hooks/useFetch.ts
interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

function useFetch<T>(url: string): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await fetch(url);
      if (!response.ok) throw new Error('Failed to fetch');
      const json = await response.json();
      setData(json);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
}

// Usage
const StudentList: FC = () => {
  const { data: students, loading, error, refetch } = useFetch<Student[]>(
    'https://api.college.edu/students'
  );

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <button onClick={refetch}>Refresh</button>
      {students?.map(student => (
        <StudentCard key={student.id} student={student} />
      ))}
    </div>
  );
};
```

### Local Storage Hook

```typescript
// hooks/useLocalStorage.ts
function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
  // Get from local storage or use initial value
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error('Error reading from localStorage:', error);
      return initialValue;
    }
  });

  // Update local storage when value changes
  const setValue = useCallback((value: T | ((prev: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error('Error writing to localStorage:', error);
    }
  }, [key, storedValue]);

  return [storedValue, setValue];
}

// Usage
const UserPreferences: FC = () => {
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
  const [fontSize, setFontSize] = useLocalStorage<number>('fontSize', 16);

  return (
    <div>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme (Current: {theme})
      </button>
      <input
        type="number"
        value={fontSize}
        onChange={(e) => setFontSize(Number(e.target.value))}
      />
    </div>
  );
};
```

### Debounce Hook

```typescript
// hooks/useDebounce.ts
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
const StudentSearch: FC = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearch = useDebounce(searchTerm, 500);
  const [results, setResults] = useState<Student[]>([]);

  useEffect(() => {
    if (debouncedSearch) {
      fetch(`/api/students/search?q=${debouncedSearch}`)
        .then(res => res.json())
        .then(setResults);
    }
  }, [debouncedSearch]);

  return (
    <div>
      <input
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search students... (debounced)"
      />
      <p>Searching for: {debouncedSearch}</p>
      {results.map(student => (
        <StudentCard key={student.id} student={student} />
      ))}
    </div>
  );
};
```

### Window Size Hook

```typescript
// hooks/useWindowSize.ts
interface WindowSize {
  width: number;
  height: number;
}

function useWindowSize(): WindowSize {
  const [windowSize, setWindowSize] = useState<WindowSize>({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return windowSize;
}

// Usage
const ResponsiveLayout: FC = () => {
  const { width } = useWindowSize();
  const isMobile = width < 768;

  return (
    <div>
      <p>Window width: {width}px</p>
      {isMobile ? <MobileNav /> : <DesktopNav />}
    </div>
  );
};
```

---

## Rules of Hooks {#rules-of-hooks}

Hooks have strict rules you must follow.

### Rule 1: Only Call Hooks at the Top Level

```typescript
// ❌ WRONG: Conditional hook
function Component({ condition }: { condition: boolean }) {
  if (condition) {
    const [state, setState] = useState(0); // Error!
  }
  return <div />;
}

// ❌ WRONG: Hook in loop
function Component({ items }: { items: string[] }) {
  items.forEach(item => {
    const [state, setState] = useState(item); // Error!
  });
  return <div />;
}

// ✅ CORRECT: Hooks at top level
function Component({ condition }: { condition: boolean }) {
  const [state, setState] = useState(0);

  if (condition) {
    // Use state here
  }

  return <div />;
}
```

### Rule 2: Only Call Hooks from React Functions

```typescript
// ❌ WRONG: Hook in regular function
function calculateGPA() {
  const [gpa, setGPA] = useState(0); // Error!
  return gpa;
}

// ✅ CORRECT: Hook in component
function GPACalculator() {
  const [gpa, setGPA] = useState(0);
  return <div>{gpa}</div>;
}

// ✅ CORRECT: Hook in custom hook
function useGPA() {
  const [gpa, setGPA] = useState(0);
  return gpa;
}
```

### ESLint Plugin

```json
// .eslintrc.json
{
  "plugins": ["react-hooks"],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

---

## College Portal Examples {#college-examples}

### Complete Enrollment Form

```typescript
const EnrollmentForm: FC = () => {
  const [formData, setFormData] = useState({
    studentId: '',
    courseId: '',
    semester: 'Fall 2024'
  });
  const [submitting, setSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [success, setSuccess] = useState(false);

  const handleChange = (field: string, value: string) => {
    setFormData(prev => ({ ...prev, [field]: value }));
    setError(null);
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setSubmitting(true);
    setError(null);

    try {
      const response = await fetch('/api/enrollments', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });

      if (!response.ok) {
        const data = await response.json();
        throw new Error(data.message || 'Enrollment failed');
      }

      setSuccess(true);
      setFormData({ studentId: '', courseId: '', semester: 'Fall 2024' });
    } catch (err: any) {
      setError(err.message);
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="max-w-md mx-auto p-6 bg-white rounded-lg shadow">
      <h2 className="text-2xl font-bold mb-6">Course Enrollment</h2>

      {error && (
        <div className="mb-4 p-3 bg-red-100 text-red-700 rounded">
          {error}
        </div>
      )}

      {success && (
        <div className="mb-4 p-3 bg-green-100 text-green-700 rounded">
          Enrollment successful!
        </div>
      )}

      <div className="mb-4">
        <label className="block mb-2 font-medium">Student ID</label>
        <input
          type="text"
          value={formData.studentId}
          onChange={(e) => handleChange('studentId', e.target.value)}
          className="w-full px-4 py-2 border rounded-lg"
          required
        />
      </div>

      <div className="mb-4">
        <label className="block mb-2 font-medium">Course ID</label>
        <input
          type="text"
          value={formData.courseId}
          onChange={(e) => handleChange('courseId', e.target.value)}
          className="w-full px-4 py-2 border rounded-lg"
          required
        />
      </div>

      <div className="mb-6">
        <label className="block mb-2 font-medium">Semester</label>
        <select
          value={formData.semester}
          onChange={(e) => handleChange('semester', e.target.value)}
          className="w-full px-4 py-2 border rounded-lg"
        >
          <option>Fall 2024</option>
          <option>Spring 2025</option>
          <option>Summer 2025</option>
        </select>
      </div>

      <button
        type="submit"
        disabled={submitting}
        className="w-full py-3 bg-blue-600 text-white rounded-lg font-medium hover:bg-blue-700 disabled:bg-gray-400"
      >
        {submitting ? 'Enrolling...' : 'Enroll'}
      </button>
    </form>
  );
};
```

---

## Summary

React Hooks covered:

1. **useState**: Local state management
2. **useEffect**: Side effects and lifecycle
3. **useRef**: DOM refs and mutable values
4. **useCallback**: Memoize functions
5. **useMemo**: Memoize values
6. **Custom Hooks**: Reusable logic
7. **Rules of Hooks**: Important constraints

**Key Takeaways:**
- Hooks let you use state and lifecycle in functional components
- Always follow the Rules of Hooks
- Use custom hooks to extract reusable logic
- Optimize with useCallback and useMemo when needed
- Clean up effects to prevent memory leaks

---

## Navigation

- **Previous**: [Part 4: React Fundamentals](./04_React_Fundamentals.md)
- **Next**: [Part 6: Next.js Fundamentals](./06_NextJS_Fundamentals.md)
- **[Back to Index](./README.md)**

---

**Author: Saiprasad Hegde**

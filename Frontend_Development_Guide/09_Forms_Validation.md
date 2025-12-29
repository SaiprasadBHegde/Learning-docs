# Part 9: Forms & Validation

**Author: Saiprasad Hegde**

## Table of Contents
1. [Introduction to Forms in React](#introduction-to-forms-in-react)
2. [Controlled vs Uncontrolled Components](#controlled-vs-uncontrolled-components)
3. [Basic Form State Management](#basic-form-state-management)
4. [React Hook Form](#react-hook-form)
   - [Installation and Setup](#installation-and-setup)
   - [useForm Hook](#useform-hook)
   - [register Function](#register-function)
   - [handleSubmit Function](#handlesubmit-function)
   - [formState Object](#formstate-object)
5. [Validation with Zod](#validation-with-zod)
   - [Zod Schema Basics](#zod-schema-basics)
   - [Integration with React Hook Form](#integration-with-react-hook-form)
6. [Error Handling and Display](#error-handling-and-display)
7. [Field Validation Rules](#field-validation-rules)
8. [Custom Validation](#custom-validation)
9. [File Uploads](#file-uploads)
10. [Multi-Step Forms](#multi-step-forms)
11. [Form Submission to API](#form-submission-to-api)
12. [Complete Examples](#complete-examples)
    - [Student Registration Form](#student-registration-form)
    - [Course Enrollment Form](#course-enrollment-form)
13. [Best Practices](#best-practices)
14. [Common Pitfalls](#common-pitfalls)

---

## Introduction to Forms in React

Forms are the primary way users interact with web applications. In our College Management System, forms are everywhere - registering students, creating courses, enrolling students in courses, and more.

**Real-World Analogy**: Think of a form like a restaurant order slip. The waiter (React) needs to know what you want (form data), check if your order makes sense (validation), and send it to the kitchen (API). If you order pizza with ice cream on top, the waiter should stop you (validation error)!

In React, forms work differently than traditional HTML forms because React maintains its own state. This gives us powerful control over user input and validation.

---

## Controlled vs Uncontrolled Components

### Uncontrolled Components

In uncontrolled components, the DOM itself maintains the form state. You access values using refs.

**Analogy**: Like a suggestion box where you only check what's inside when someone tells you to look.

```typescript
import { useRef } from 'react';

function UncontrolledForm() {
  const nameRef = useRef<HTMLInputElement>(null);
  const emailRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('Name:', nameRef.current?.value);
    console.log('Email:', emailRef.current?.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={nameRef} type="text" placeholder="Name" />
      <input ref={emailRef} type="email" placeholder="Email" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Controlled Components

In controlled components, React state controls the input value. Every keystroke updates state.

**Analogy**: Like watching someone fill out a form in real-time, seeing every letter they write as they write it.

```typescript
import { useState } from 'react';

function ControlledForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('Name:', name);
    console.log('Email:', email);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**When to Use Which?**
- **Controlled**: When you need instant validation, formatting, or conditional rendering based on input
- **Uncontrolled**: For simple forms or file inputs (files must be uncontrolled)

---

## Basic Form State Management

For simple forms, useState is sufficient. However, as forms grow complex, managing state becomes tedious.

```typescript
import { useState } from 'react';

interface StudentFormData {
  firstName: string;
  lastName: string;
  email: string;
  phone: string;
  dateOfBirth: string;
  department: string;
}

function BasicStudentForm() {
  const [formData, setFormData] = useState<StudentFormData>({
    firstName: '',
    lastName: '',
    email: '',
    phone: '',
    dateOfBirth: '',
    department: '',
  });

  const [errors, setErrors] = useState<Partial<StudentFormData>>({});

  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLSelectElement>) => {
    const { name, value } = e.target;
    setFormData((prev) => ({
      ...prev,
      [name]: value,
    }));
  };

  const validate = (): boolean => {
    const newErrors: Partial<StudentFormData> = {};

    if (!formData.firstName.trim()) {
      newErrors.firstName = 'First name is required';
    }

    if (!formData.email.includes('@')) {
      newErrors.email = 'Invalid email address';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (validate()) {
      console.log('Form submitted:', formData);
      // Submit to API
    }
  };

  return (
    <form onSubmit={handleSubmit} className="max-w-md mx-auto p-6">
      <div className="mb-4">
        <label className="block text-sm font-medium mb-1">First Name</label>
        <input
          type="text"
          name="firstName"
          value={formData.firstName}
          onChange={handleChange}
          className="w-full px-3 py-2 border rounded"
        />
        {errors.firstName && (
          <p className="text-red-500 text-sm mt-1">{errors.firstName}</p>
        )}
      </div>

      <div className="mb-4">
        <label className="block text-sm font-medium mb-1">Email</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          className="w-full px-3 py-2 border rounded"
        />
        {errors.email && (
          <p className="text-red-500 text-sm mt-1">{errors.email}</p>
        )}
      </div>

      <button
        type="submit"
        className="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600"
      >
        Register Student
      </button>
    </form>
  );
}
```

**Problems with This Approach:**
- Lots of boilerplate code
- Manual validation logic
- Error handling is verbose
- Hard to scale to complex forms

**Solution**: React Hook Form!

---

## React Hook Form

React Hook Form is a library that simplifies form handling with better performance and less code.

**Analogy**: Think of React Hook Form as a smart assistant who handles all the paperwork for you, checks everything is filled correctly, and only bothers you when there's a problem.

### Installation and Setup

```bash
npm install react-hook-form
npm install zod @hookform/resolvers
```

### useForm Hook

The `useForm` hook is the core of React Hook Form. It returns several useful functions and objects.

```typescript
import { useForm } from 'react-hook-form';

interface FormData {
  username: string;
  email: string;
}

function SimpleForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>();

  const onSubmit = (data: FormData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('username')} />
      <input {...register('email')} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### register Function

The `register` function connects your input to React Hook Form. It returns props that you spread onto your input element.

```typescript
import { useForm } from 'react-hook-form';

interface CourseFormData {
  courseName: string;
  courseCode: string;
  credits: number;
  department: string;
}

function CourseForm() {
  const { register, handleSubmit } = useForm<CourseFormData>();

  const onSubmit = (data: CourseFormData) => {
    console.log('Course Data:', data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Course Name</label>
        <input
          {...register('courseName', {
            required: 'Course name is required',
            minLength: {
              value: 3,
              message: 'Course name must be at least 3 characters',
            },
          })}
          className="border p-2 rounded"
        />
      </div>

      <div>
        <label>Course Code</label>
        <input
          {...register('courseCode', {
            required: 'Course code is required',
            pattern: {
              value: /^[A-Z]{2,4}\d{3}$/,
              message: 'Invalid course code format (e.g., CS101)',
            },
          })}
          className="border p-2 rounded"
          placeholder="CS101"
        />
      </div>

      <div>
        <label>Credits</label>
        <input
          type="number"
          {...register('credits', {
            required: 'Credits are required',
            min: { value: 1, message: 'Minimum 1 credit' },
            max: { value: 6, message: 'Maximum 6 credits' },
            valueAsNumber: true, // Automatically convert to number
          })}
          className="border p-2 rounded"
        />
      </div>

      <button type="submit" className="bg-blue-500 text-white px-4 py-2 rounded">
        Create Course
      </button>
    </form>
  );
}
```

### handleSubmit Function

`handleSubmit` wraps your submit handler and only calls it if validation passes.

```typescript
const { handleSubmit } = useForm<FormData>();

// Success handler - called only if validation passes
const onSubmit = (data: FormData) => {
  console.log('Valid data:', data);
};

// Error handler - called if validation fails
const onError = (errors: FieldErrors<FormData>) => {
  console.log('Validation errors:', errors);
};

// Use both handlers
<form onSubmit={handleSubmit(onSubmit, onError)}>
```

### formState Object

The `formState` object contains information about the form's state.

```typescript
import { useForm } from 'react-hook-form';

function FormWithState() {
  const {
    register,
    handleSubmit,
    formState: {
      errors,        // Validation errors
      isDirty,       // Form has been modified
      isValid,       // All fields are valid
      isSubmitting,  // Form is currently submitting
      isSubmitted,   // Form has been submitted
      touchedFields, // Fields that have been focused
      dirtyFields,   // Fields that have been modified
    }
  } = useForm<FormData>({ mode: 'onChange' }); // Validate on change

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}

      <button
        type="submit"
        disabled={!isValid || isSubmitting}
        className={`px-4 py-2 rounded ${
          isValid ? 'bg-blue-500' : 'bg-gray-300'
        }`}
      >
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>

      {isSubmitted && <p>Form submitted successfully!</p>}
    </form>
  );
}
```

**Validation Modes:**
- `onSubmit` (default): Validate on submit
- `onChange`: Validate on every change
- `onBlur`: Validate when field loses focus
- `onTouched`: Validate after blur, then on change
- `all`: Validate on all events

---

## Validation with Zod

Zod is a TypeScript-first schema validation library. It's powerful, type-safe, and works beautifully with React Hook Form.

**Analogy**: Zod is like a strict teacher who has a clear rubric. You define the rules once, and Zod checks every submission against those rules.

### Zod Schema Basics

```typescript
import { z } from 'zod';

// Simple schema
const userSchema = z.object({
  username: z.string().min(3).max(20),
  email: z.string().email(),
  age: z.number().min(18).max(100),
});

// TypeScript type from schema
type User = z.infer<typeof userSchema>;

// Validate data
const result = userSchema.safeParse({
  username: 'john',
  email: 'john@example.com',
  age: 25,
});

if (result.success) {
  console.log('Valid data:', result.data);
} else {
  console.log('Errors:', result.error);
}
```

### Student Registration Schema

```typescript
import { z } from 'zod';

export const studentRegistrationSchema = z.object({
  // Personal Information
  firstName: z
    .string()
    .min(2, 'First name must be at least 2 characters')
    .max(50, 'First name must be less than 50 characters')
    .regex(/^[a-zA-Z\s]+$/, 'First name can only contain letters'),

  lastName: z
    .string()
    .min(2, 'Last name must be at least 2 characters')
    .max(50, 'Last name must be less than 50 characters')
    .regex(/^[a-zA-Z\s]+$/, 'Last name can only contain letters'),

  email: z
    .string()
    .email('Invalid email address')
    .endsWith('@college.edu', 'Must use college email'),

  phone: z
    .string()
    .regex(/^\+?1?\d{10,14}$/, 'Invalid phone number'),

  dateOfBirth: z
    .string()
    .refine((date) => {
      const birthDate = new Date(date);
      const today = new Date();
      const age = today.getFullYear() - birthDate.getFullYear();
      return age >= 16 && age <= 100;
    }, 'Student must be between 16 and 100 years old'),

  // Academic Information
  department: z.enum([
    'Computer Science',
    'Electrical Engineering',
    'Mechanical Engineering',
    'Business Administration',
    'Biology',
    'Chemistry',
  ], {
    errorMap: () => ({ message: 'Please select a department' }),
  }),

  studentId: z
    .string()
    .regex(/^STU\d{6}$/, 'Student ID must be in format STU123456'),

  gpa: z
    .number()
    .min(0, 'GPA cannot be negative')
    .max(4.0, 'GPA cannot exceed 4.0')
    .optional(),

  // Address
  address: z.object({
    street: z.string().min(5, 'Street address is required'),
    city: z.string().min(2, 'City is required'),
    state: z.string().length(2, 'State must be 2 characters'),
    zipCode: z.string().regex(/^\d{5}(-\d{4})?$/, 'Invalid ZIP code'),
  }),

  // Emergency Contact
  emergencyContact: z.object({
    name: z.string().min(2, 'Emergency contact name is required'),
    relationship: z.string().min(2, 'Relationship is required'),
    phone: z.string().regex(/^\+?1?\d{10,14}$/, 'Invalid phone number'),
  }),

  // Agreements
  termsAccepted: z
    .boolean()
    .refine((val) => val === true, 'You must accept the terms and conditions'),
});

// TypeScript type from schema
export type StudentRegistrationData = z.infer<typeof studentRegistrationSchema>;
```

### Integration with React Hook Form

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  courseName: z.string().min(3, 'Course name must be at least 3 characters'),
  courseCode: z.string().regex(/^[A-Z]{2,4}\d{3}$/, 'Invalid course code'),
  credits: z.number().min(1).max(6),
  instructor: z.string().min(2, 'Instructor name is required'),
  maxStudents: z.number().min(1).max(200),
  description: z.string().max(500, 'Description is too long').optional(),
});

type CourseFormData = z.infer<typeof schema>;

function CourseCreationForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<CourseFormData>({
    resolver: zodResolver(schema), // Connect Zod schema
    defaultValues: {
      credits: 3,
      maxStudents: 30,
    },
  });

  const onSubmit = async (data: CourseFormData) => {
    try {
      const response = await fetch('/api/courses', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });

      if (!response.ok) throw new Error('Failed to create course');

      console.log('Course created successfully');
    } catch (error) {
      console.error('Error creating course:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="max-w-2xl mx-auto p-6 space-y-4">
      <h2 className="text-2xl font-bold mb-6">Create New Course</h2>

      <div>
        <label className="block text-sm font-medium mb-1">Course Name</label>
        <input
          {...register('courseName')}
          className="w-full px-3 py-2 border rounded focus:ring-2 focus:ring-blue-500"
          placeholder="Introduction to Computer Science"
        />
        {errors.courseName && (
          <p className="text-red-500 text-sm mt-1">{errors.courseName.message}</p>
        )}
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Course Code</label>
        <input
          {...register('courseCode')}
          className="w-full px-3 py-2 border rounded focus:ring-2 focus:ring-blue-500"
          placeholder="CS101"
        />
        {errors.courseCode && (
          <p className="text-red-500 text-sm mt-1">{errors.courseCode.message}</p>
        )}
      </div>

      <div className="grid grid-cols-2 gap-4">
        <div>
          <label className="block text-sm font-medium mb-1">Credits</label>
          <input
            type="number"
            {...register('credits', { valueAsNumber: true })}
            className="w-full px-3 py-2 border rounded focus:ring-2 focus:ring-blue-500"
          />
          {errors.credits && (
            <p className="text-red-500 text-sm mt-1">{errors.credits.message}</p>
          )}
        </div>

        <div>
          <label className="block text-sm font-medium mb-1">Max Students</label>
          <input
            type="number"
            {...register('maxStudents', { valueAsNumber: true })}
            className="w-full px-3 py-2 border rounded focus:ring-2 focus:ring-blue-500"
          />
          {errors.maxStudents && (
            <p className="text-red-500 text-sm mt-1">{errors.maxStudents.message}</p>
          )}
        </div>
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Instructor</label>
        <input
          {...register('instructor')}
          className="w-full px-3 py-2 border rounded focus:ring-2 focus:ring-blue-500"
          placeholder="Dr. Smith"
        />
        {errors.instructor && (
          <p className="text-red-500 text-sm mt-1">{errors.instructor.message}</p>
        )}
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Description (Optional)</label>
        <textarea
          {...register('description')}
          className="w-full px-3 py-2 border rounded focus:ring-2 focus:ring-blue-500"
          rows={4}
          placeholder="Course description..."
        />
        {errors.description && (
          <p className="text-red-500 text-sm mt-1">{errors.description.message}</p>
        )}
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600 disabled:bg-gray-300"
      >
        {isSubmitting ? 'Creating...' : 'Create Course'}
      </button>
    </form>
  );
}

export default CourseCreationForm;
```

---

## Error Handling and Display

Proper error display improves user experience significantly.

```typescript
import { useForm, FieldError } from 'react-hook-form';

interface ErrorMessageProps {
  error?: FieldError;
}

// Reusable error message component
function ErrorMessage({ error }: ErrorMessageProps) {
  if (!error) return null;

  return (
    <p className="text-red-500 text-sm mt-1 flex items-center">
      <svg className="w-4 h-4 mr-1" fill="currentColor" viewBox="0 0 20 20">
        <path fillRule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7 4a1 1 0 11-2 0 1 1 0 012 0zm-1-9a1 1 0 00-1 1v4a1 1 0 102 0V6a1 1 0 00-1-1z" clipRule="evenodd" />
      </svg>
      {error.message}
    </p>
  );
}

// Reusable input component with error handling
interface InputFieldProps {
  label: string;
  name: string;
  type?: string;
  placeholder?: string;
  register: any;
  error?: FieldError;
  required?: boolean;
}

function InputField({
  label,
  name,
  type = 'text',
  placeholder,
  register,
  error,
  required = false,
}: InputFieldProps) {
  return (
    <div className="mb-4">
      <label className="block text-sm font-medium mb-1">
        {label}
        {required && <span className="text-red-500 ml-1">*</span>}
      </label>
      <input
        type={type}
        {...register(name)}
        placeholder={placeholder}
        className={`w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 ${
          error
            ? 'border-red-500 focus:ring-red-500'
            : 'border-gray-300 focus:ring-blue-500'
        }`}
      />
      <ErrorMessage error={error} />
    </div>
  );
}

// Usage
function StudentForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<StudentFormData>({
    resolver: zodResolver(studentSchema),
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <InputField
        label="First Name"
        name="firstName"
        register={register}
        error={errors.firstName}
        required
      />
      <InputField
        label="Email"
        name="email"
        type="email"
        register={register}
        error={errors.email}
        required
      />
    </form>
  );
}
```

---

## Field Validation Rules

React Hook Form supports various validation rules out of the box.

```typescript
import { useForm } from 'react-hook-form';

function ValidationExamples() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Required field */}
      <input
        {...register('username', {
          required: 'Username is required',
        })}
      />

      {/* Min/Max length */}
      <input
        {...register('password', {
          required: 'Password is required',
          minLength: {
            value: 8,
            message: 'Password must be at least 8 characters',
          },
          maxLength: {
            value: 50,
            message: 'Password must be less than 50 characters',
          },
        })}
      />

      {/* Pattern (Regex) */}
      <input
        {...register('studentId', {
          pattern: {
            value: /^STU\d{6}$/,
            message: 'Invalid student ID format',
          },
        })}
      />

      {/* Min/Max value (for numbers) */}
      <input
        type="number"
        {...register('age', {
          min: {
            value: 16,
            message: 'Must be at least 16 years old',
          },
          max: {
            value: 100,
            message: 'Age must be less than 100',
          },
          valueAsNumber: true,
        })}
      />

      {/* Custom validation */}
      <input
        {...register('email', {
          required: 'Email is required',
          validate: {
            isCollegeEmail: (value) =>
              value.endsWith('@college.edu') || 'Must use college email',
            isNotBlocked: async (value) => {
              const blocked = await checkIfEmailBlocked(value);
              return !blocked || 'This email is blocked';
            },
          },
        })}
      />

      {/* Multiple validations */}
      <input
        {...register('courseCode', {
          required: 'Course code is required',
          pattern: {
            value: /^[A-Z]{2,4}\d{3}$/,
            message: 'Invalid format (e.g., CS101)',
          },
          validate: async (value) => {
            const exists = await checkCourseCodeExists(value);
            return !exists || 'Course code already exists';
          },
        })}
      />
    </form>
  );
}
```

---

## Custom Validation

Sometimes you need complex validation logic beyond simple rules.

```typescript
import { z } from 'zod';

// Custom Zod validation
const enrollmentSchema = z.object({
  studentId: z.string(),
  courseId: z.string(),
  semester: z.string(),
  year: z.number(),
}).refine(async (data) => {
  // Check if student is already enrolled
  const response = await fetch(`/api/enrollments/check`, {
    method: 'POST',
    body: JSON.stringify(data),
  });
  const { alreadyEnrolled } = await response.json();
  return !alreadyEnrolled;
}, {
  message: 'Student is already enrolled in this course',
  path: ['courseId'], // Which field to attach error to
});

// Cross-field validation
const passwordChangeSchema = z.object({
  currentPassword: z.string().min(8),
  newPassword: z.string().min(8),
  confirmPassword: z.string().min(8),
}).refine((data) => data.newPassword === data.confirmPassword, {
  message: "Passwords don't match",
  path: ['confirmPassword'],
}).refine((data) => data.currentPassword !== data.newPassword, {
  message: "New password must be different from current password",
  path: ['newPassword'],
});

// Conditional validation
const courseSchema = z.object({
  courseName: z.string(),
  isAdvanced: z.boolean(),
  prerequisiteCourses: z.array(z.string()),
}).refine(
  (data) => {
    // If advanced course, must have prerequisites
    if (data.isAdvanced) {
      return data.prerequisiteCourses.length > 0;
    }
    return true;
  },
  {
    message: 'Advanced courses must have at least one prerequisite',
    path: ['prerequisiteCourses'],
  }
);

// Async validation with React Hook Form
function AsyncValidationExample() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  const checkUsernameAvailable = async (username: string) => {
    // Simulate API call
    await new Promise((resolve) => setTimeout(resolve, 500));
    const taken = ['admin', 'test', 'user'].includes(username);
    return !taken || 'Username is already taken';
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('username', {
          required: 'Username is required',
          validate: checkUsernameAvailable,
        })}
      />
      {errors.username && <span>{errors.username.message}</span>}
    </form>
  );
}
```

---

## File Uploads

File inputs require special handling as they must be uncontrolled.

```typescript
import { useForm } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
const ACCEPTED_IMAGE_TYPES = ['image/jpeg', 'image/jpg', 'image/png', 'image/webp'];

const profileSchema = z.object({
  name: z.string().min(2),
  bio: z.string().max(500).optional(),
  avatar: z
    .custom<FileList>()
    .refine((files) => files?.length === 1, 'Avatar is required')
    .refine(
      (files) => files?.[0]?.size <= MAX_FILE_SIZE,
      'Max file size is 5MB'
    )
    .refine(
      (files) => ACCEPTED_IMAGE_TYPES.includes(files?.[0]?.type),
      'Only .jpg, .jpeg, .png and .webp formats are supported'
    ),
  resume: z
    .custom<FileList>()
    .refine((files) => files?.length === 1, 'Resume is required')
    .refine(
      (files) => files?.[0]?.size <= MAX_FILE_SIZE,
      'Max file size is 5MB'
    )
    .refine(
      (files) => files?.[0]?.type === 'application/pdf',
      'Only PDF files are accepted'
    ),
});

type ProfileFormData = z.infer<typeof profileSchema>;

function StudentProfileForm() {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors, isSubmitting },
  } = useForm<ProfileFormData>({
    resolver: zodResolver(profileSchema),
  });

  const avatarFile = watch('avatar');
  const resumeFile = watch('resume');

  const onSubmit = async (data: ProfileFormData) => {
    const formData = new FormData();
    formData.append('name', data.name);
    if (data.bio) formData.append('bio', data.bio);
    formData.append('avatar', data.avatar[0]);
    formData.append('resume', data.resume[0]);

    try {
      const response = await fetch('/api/students/profile', {
        method: 'POST',
        body: formData, // Don't set Content-Type header, browser will set it
      });

      if (!response.ok) throw new Error('Upload failed');

      console.log('Profile updated successfully');
    } catch (error) {
      console.error('Error uploading files:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="max-w-md mx-auto p-6 space-y-4">
      <h2 className="text-2xl font-bold">Update Profile</h2>

      <div>
        <label className="block text-sm font-medium mb-1">Name</label>
        <input
          {...register('name')}
          className="w-full px-3 py-2 border rounded"
        />
        {errors.name && (
          <p className="text-red-500 text-sm mt-1">{errors.name.message}</p>
        )}
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Bio</label>
        <textarea
          {...register('bio')}
          className="w-full px-3 py-2 border rounded"
          rows={3}
        />
        {errors.bio && (
          <p className="text-red-500 text-sm mt-1">{errors.bio.message}</p>
        )}
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Avatar Image</label>
        <input
          type="file"
          accept="image/*"
          {...register('avatar')}
          className="w-full px-3 py-2 border rounded"
        />
        {avatarFile?.[0] && (
          <p className="text-sm text-gray-600 mt-1">
            Selected: {avatarFile[0].name} ({(avatarFile[0].size / 1024).toFixed(2)} KB)
          </p>
        )}
        {errors.avatar && (
          <p className="text-red-500 text-sm mt-1">{errors.avatar.message as string}</p>
        )}
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Resume (PDF)</label>
        <input
          type="file"
          accept=".pdf"
          {...register('resume')}
          className="w-full px-3 py-2 border rounded"
        />
        {resumeFile?.[0] && (
          <p className="text-sm text-gray-600 mt-1">
            Selected: {resumeFile[0].name} ({(resumeFile[0].size / 1024).toFixed(2)} KB)
          </p>
        )}
        {errors.resume && (
          <p className="text-red-500 text-sm mt-1">{errors.resume.message as string}</p>
        )}
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600 disabled:bg-gray-300"
      >
        {isSubmitting ? 'Uploading...' : 'Update Profile'}
      </button>
    </form>
  );
}

export default StudentProfileForm;
```

---

## Multi-Step Forms

Multi-step forms break complex forms into manageable chunks.

**Analogy**: Like filling out a college application - personal info first, then academics, then preferences. One step at a time!

```typescript
import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Step 1 Schema: Personal Information
const personalInfoSchema = z.object({
  firstName: z.string().min(2),
  lastName: z.string().min(2),
  email: z.string().email(),
  phone: z.string().regex(/^\+?1?\d{10,14}$/),
  dateOfBirth: z.string(),
});

// Step 2 Schema: Academic Information
const academicInfoSchema = z.object({
  studentId: z.string().regex(/^STU\d{6}$/),
  department: z.string().min(1),
  expectedGraduation: z.string(),
  gpa: z.number().min(0).max(4.0).optional(),
});

// Step 3 Schema: Course Selection
const courseSelectionSchema = z.object({
  courses: z.array(z.string()).min(1, 'Select at least one course'),
  semester: z.string(),
  year: z.number(),
});

// Complete schema (for final submission)
const completeSchema = z.object({
  ...personalInfoSchema.shape,
  ...academicInfoSchema.shape,
  ...courseSelectionSchema.shape,
});

type PersonalInfo = z.infer<typeof personalInfoSchema>;
type AcademicInfo = z.infer<typeof academicInfoSchema>;
type CourseSelection = z.infer<typeof courseSelectionSchema>;
type CompleteFormData = z.infer<typeof completeSchema>;

function MultiStepEnrollmentForm() {
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState<Partial<CompleteFormData>>({});

  // Step 1: Personal Information
  const {
    register: registerPersonal,
    handleSubmit: handleSubmitPersonal,
    formState: { errors: errorsPersonal },
  } = useForm<PersonalInfo>({
    resolver: zodResolver(personalInfoSchema),
    defaultValues: formData as PersonalInfo,
  });

  // Step 2: Academic Information
  const {
    register: registerAcademic,
    handleSubmit: handleSubmitAcademic,
    formState: { errors: errorsAcademic },
  } = useForm<AcademicInfo>({
    resolver: zodResolver(academicInfoSchema),
    defaultValues: formData as AcademicInfo,
  });

  // Step 3: Course Selection
  const {
    register: registerCourses,
    handleSubmit: handleSubmitCourses,
    formState: { errors: errorsCourses },
  } = useForm<CourseSelection>({
    resolver: zodResolver(courseSelectionSchema),
    defaultValues: formData as CourseSelection,
  });

  const onSubmitPersonal = (data: PersonalInfo) => {
    setFormData((prev) => ({ ...prev, ...data }));
    setStep(2);
  };

  const onSubmitAcademic = (data: AcademicInfo) => {
    setFormData((prev) => ({ ...prev, ...data }));
    setStep(3);
  };

  const onSubmitFinal = async (data: CourseSelection) => {
    const completeData = { ...formData, ...data };

    try {
      const response = await fetch('/api/enrollments', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(completeData),
      });

      if (!response.ok) throw new Error('Enrollment failed');

      console.log('Enrollment successful!');
      // Reset form or redirect
    } catch (error) {
      console.error('Error submitting enrollment:', error);
    }
  };

  const availableCourses = [
    'CS101 - Intro to Computer Science',
    'CS201 - Data Structures',
    'CS301 - Algorithms',
    'MATH101 - Calculus I',
    'PHYS101 - Physics I',
  ];

  return (
    <div className="max-w-2xl mx-auto p-6">
      {/* Progress Indicator */}
      <div className="mb-8">
        <div className="flex justify-between mb-2">
          <span className={`text-sm font-medium ${step >= 1 ? 'text-blue-600' : 'text-gray-400'}`}>
            Personal Info
          </span>
          <span className={`text-sm font-medium ${step >= 2 ? 'text-blue-600' : 'text-gray-400'}`}>
            Academic Info
          </span>
          <span className={`text-sm font-medium ${step >= 3 ? 'text-blue-600' : 'text-gray-400'}`}>
            Course Selection
          </span>
        </div>
        <div className="w-full bg-gray-200 rounded-full h-2">
          <div
            className="bg-blue-600 h-2 rounded-full transition-all duration-300"
            style={{ width: `${(step / 3) * 100}%` }}
          />
        </div>
      </div>

      {/* Step 1: Personal Information */}
      {step === 1 && (
        <form onSubmit={handleSubmitPersonal(onSubmitPersonal)} className="space-y-4">
          <h2 className="text-2xl font-bold mb-4">Personal Information</h2>

          <div className="grid grid-cols-2 gap-4">
            <div>
              <label className="block text-sm font-medium mb-1">First Name</label>
              <input
                {...registerPersonal('firstName')}
                className="w-full px-3 py-2 border rounded"
              />
              {errorsPersonal.firstName && (
                <p className="text-red-500 text-sm mt-1">{errorsPersonal.firstName.message}</p>
              )}
            </div>

            <div>
              <label className="block text-sm font-medium mb-1">Last Name</label>
              <input
                {...registerPersonal('lastName')}
                className="w-full px-3 py-2 border rounded"
              />
              {errorsPersonal.lastName && (
                <p className="text-red-500 text-sm mt-1">{errorsPersonal.lastName.message}</p>
              )}
            </div>
          </div>

          <div>
            <label className="block text-sm font-medium mb-1">Email</label>
            <input
              type="email"
              {...registerPersonal('email')}
              className="w-full px-3 py-2 border rounded"
            />
            {errorsPersonal.email && (
              <p className="text-red-500 text-sm mt-1">{errorsPersonal.email.message}</p>
            )}
          </div>

          <div>
            <label className="block text-sm font-medium mb-1">Phone</label>
            <input
              {...registerPersonal('phone')}
              className="w-full px-3 py-2 border rounded"
            />
            {errorsPersonal.phone && (
              <p className="text-red-500 text-sm mt-1">{errorsPersonal.phone.message}</p>
            )}
          </div>

          <div>
            <label className="block text-sm font-medium mb-1">Date of Birth</label>
            <input
              type="date"
              {...registerPersonal('dateOfBirth')}
              className="w-full px-3 py-2 border rounded"
            />
            {errorsPersonal.dateOfBirth && (
              <p className="text-red-500 text-sm mt-1">{errorsPersonal.dateOfBirth.message}</p>
            )}
          </div>

          <button
            type="submit"
            className="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600"
          >
            Next: Academic Information
          </button>
        </form>
      )}

      {/* Step 2: Academic Information */}
      {step === 2 && (
        <form onSubmit={handleSubmitAcademic(onSubmitAcademic)} className="space-y-4">
          <h2 className="text-2xl font-bold mb-4">Academic Information</h2>

          <div>
            <label className="block text-sm font-medium mb-1">Student ID</label>
            <input
              {...registerAcademic('studentId')}
              placeholder="STU123456"
              className="w-full px-3 py-2 border rounded"
            />
            {errorsAcademic.studentId && (
              <p className="text-red-500 text-sm mt-1">{errorsAcademic.studentId.message}</p>
            )}
          </div>

          <div>
            <label className="block text-sm font-medium mb-1">Department</label>
            <select
              {...registerAcademic('department')}
              className="w-full px-3 py-2 border rounded"
            >
              <option value="">Select Department</option>
              <option value="Computer Science">Computer Science</option>
              <option value="Electrical Engineering">Electrical Engineering</option>
              <option value="Mechanical Engineering">Mechanical Engineering</option>
              <option value="Business Administration">Business Administration</option>
            </select>
            {errorsAcademic.department && (
              <p className="text-red-500 text-sm mt-1">{errorsAcademic.department.message}</p>
            )}
          </div>

          <div>
            <label className="block text-sm font-medium mb-1">Expected Graduation</label>
            <input
              type="month"
              {...registerAcademic('expectedGraduation')}
              className="w-full px-3 py-2 border rounded"
            />
            {errorsAcademic.expectedGraduation && (
              <p className="text-red-500 text-sm mt-1">{errorsAcademic.expectedGraduation.message}</p>
            )}
          </div>

          <div>
            <label className="block text-sm font-medium mb-1">Current GPA (Optional)</label>
            <input
              type="number"
              step="0.01"
              {...registerAcademic('gpa', { valueAsNumber: true })}
              className="w-full px-3 py-2 border rounded"
            />
            {errorsAcademic.gpa && (
              <p className="text-red-500 text-sm mt-1">{errorsAcademic.gpa.message}</p>
            )}
          </div>

          <div className="flex gap-4">
            <button
              type="button"
              onClick={() => setStep(1)}
              className="flex-1 bg-gray-300 text-gray-700 py-2 rounded hover:bg-gray-400"
            >
              Back
            </button>
            <button
              type="submit"
              className="flex-1 bg-blue-500 text-white py-2 rounded hover:bg-blue-600"
            >
              Next: Course Selection
            </button>
          </div>
        </form>
      )}

      {/* Step 3: Course Selection */}
      {step === 3 && (
        <form onSubmit={handleSubmitCourses(onSubmitFinal)} className="space-y-4">
          <h2 className="text-2xl font-bold mb-4">Course Selection</h2>

          <div>
            <label className="block text-sm font-medium mb-2">Select Courses</label>
            <div className="space-y-2">
              {availableCourses.map((course, index) => (
                <label key={course} className="flex items-center space-x-2">
                  <input
                    type="checkbox"
                    value={course}
                    {...registerCourses('courses')}
                    className="rounded"
                  />
                  <span>{course}</span>
                </label>
              ))}
            </div>
            {errorsCourses.courses && (
              <p className="text-red-500 text-sm mt-1">{errorsCourses.courses.message}</p>
            )}
          </div>

          <div className="grid grid-cols-2 gap-4">
            <div>
              <label className="block text-sm font-medium mb-1">Semester</label>
              <select
                {...registerCourses('semester')}
                className="w-full px-3 py-2 border rounded"
              >
                <option value="">Select Semester</option>
                <option value="Fall">Fall</option>
                <option value="Spring">Spring</option>
                <option value="Summer">Summer</option>
              </select>
              {errorsCourses.semester && (
                <p className="text-red-500 text-sm mt-1">{errorsCourses.semester.message}</p>
              )}
            </div>

            <div>
              <label className="block text-sm font-medium mb-1">Year</label>
              <input
                type="number"
                {...registerCourses('year', { valueAsNumber: true })}
                className="w-full px-3 py-2 border rounded"
                placeholder="2024"
              />
              {errorsCourses.year && (
                <p className="text-red-500 text-sm mt-1">{errorsCourses.year.message}</p>
              )}
            </div>
          </div>

          <div className="flex gap-4">
            <button
              type="button"
              onClick={() => setStep(2)}
              className="flex-1 bg-gray-300 text-gray-700 py-2 rounded hover:bg-gray-400"
            >
              Back
            </button>
            <button
              type="submit"
              className="flex-1 bg-green-500 text-white py-2 rounded hover:bg-green-600"
            >
              Complete Enrollment
            </button>
          </div>
        </form>
      )}
    </div>
  );
}

export default MultiStepEnrollmentForm;
```

---

## Form Submission to API

Proper form submission includes loading states, error handling, and success feedback.

```typescript
import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  studentId: z.string(),
  courseId: z.string(),
  semester: z.string(),
  year: z.number(),
});

type EnrollmentData = z.infer<typeof schema>;

interface ApiError {
  message: string;
  field?: string;
}

function CourseEnrollmentForm() {
  const [submitStatus, setSubmitStatus] = useState<'idle' | 'success' | 'error'>('idle');
  const [apiError, setApiError] = useState<string>('');

  const {
    register,
    handleSubmit,
    setError,
    reset,
    formState: { errors, isSubmitting },
  } = useForm<EnrollmentData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = async (data: EnrollmentData) => {
    try {
      setSubmitStatus('idle');
      setApiError('');

      const response = await fetch('/api/enrollments', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(data),
      });

      if (!response.ok) {
        const errorData = await response.json();

        // Handle field-specific errors
        if (errorData.field) {
          setError(errorData.field as keyof EnrollmentData, {
            message: errorData.message,
          });
        } else {
          // Handle general errors
          setApiError(errorData.message || 'Enrollment failed');
        }

        setSubmitStatus('error');
        return;
      }

      const result = await response.json();
      console.log('Enrollment successful:', result);

      setSubmitStatus('success');
      reset(); // Clear form

      // Redirect after success
      setTimeout(() => {
        window.location.href = '/enrollments';
      }, 2000);

    } catch (error) {
      console.error('Error submitting form:', error);
      setApiError('Network error. Please try again.');
      setSubmitStatus('error');
    }
  };

  return (
    <div className="max-w-md mx-auto p-6">
      <h2 className="text-2xl font-bold mb-6">Enroll in Course</h2>

      {/* Success Message */}
      {submitStatus === 'success' && (
        <div className="mb-4 p-4 bg-green-100 border border-green-400 text-green-700 rounded">
          Enrollment successful! Redirecting...
        </div>
      )}

      {/* Error Message */}
      {submitStatus === 'error' && apiError && (
        <div className="mb-4 p-4 bg-red-100 border border-red-400 text-red-700 rounded">
          {apiError}
        </div>
      )}

      <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
        <div>
          <label className="block text-sm font-medium mb-1">Student ID</label>
          <input
            {...register('studentId')}
            className="w-full px-3 py-2 border rounded"
            disabled={isSubmitting}
          />
          {errors.studentId && (
            <p className="text-red-500 text-sm mt-1">{errors.studentId.message}</p>
          )}
        </div>

        <div>
          <label className="block text-sm font-medium mb-1">Course ID</label>
          <input
            {...register('courseId')}
            className="w-full px-3 py-2 border rounded"
            disabled={isSubmitting}
          />
          {errors.courseId && (
            <p className="text-red-500 text-sm mt-1">{errors.courseId.message}</p>
          )}
        </div>

        <div>
          <label className="block text-sm font-medium mb-1">Semester</label>
          <select
            {...register('semester')}
            className="w-full px-3 py-2 border rounded"
            disabled={isSubmitting}
          >
            <option value="">Select Semester</option>
            <option value="Fall">Fall</option>
            <option value="Spring">Spring</option>
            <option value="Summer">Summer</option>
          </select>
          {errors.semester && (
            <p className="text-red-500 text-sm mt-1">{errors.semester.message}</p>
          )}
        </div>

        <div>
          <label className="block text-sm font-medium mb-1">Year</label>
          <input
            type="number"
            {...register('year', { valueAsNumber: true })}
            className="w-full px-3 py-2 border rounded"
            disabled={isSubmitting}
          />
          {errors.year && (
            <p className="text-red-500 text-sm mt-1">{errors.year.message}</p>
          )}
        </div>

        <button
          type="submit"
          disabled={isSubmitting}
          className="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600 disabled:bg-gray-300 disabled:cursor-not-allowed flex items-center justify-center"
        >
          {isSubmitting ? (
            <>
              <svg className="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
                <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
              </svg>
              Enrolling...
            </>
          ) : (
            'Enroll in Course'
          )}
        </button>
      </form>
    </div>
  );
}

export default CourseEnrollmentForm;
```

---

## Complete Examples

### Student Registration Form

A comprehensive student registration form with all features combined.

```typescript
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useState } from 'react';

const studentSchema = z.object({
  // Personal Information
  firstName: z.string().min(2, 'First name must be at least 2 characters'),
  lastName: z.string().min(2, 'Last name must be at least 2 characters'),
  email: z.string().email('Invalid email').endsWith('@college.edu', 'Must use college email'),
  phone: z.string().regex(/^\+?1?\d{10,14}$/, 'Invalid phone number'),
  dateOfBirth: z.string().refine((date) => {
    const age = new Date().getFullYear() - new Date(date).getFullYear();
    return age >= 16 && age <= 100;
  }, 'Must be between 16 and 100 years old'),

  // Academic Information
  studentId: z.string().regex(/^STU\d{6}$/, 'Student ID must be in format STU123456'),
  department: z.string().min(1, 'Please select a department'),
  expectedGraduation: z.string().min(1, 'Expected graduation is required'),

  // Address
  street: z.string().min(5, 'Street address is required'),
  city: z.string().min(2, 'City is required'),
  state: z.string().length(2, 'State must be 2 characters'),
  zipCode: z.string().regex(/^\d{5}(-\d{4})?$/, 'Invalid ZIP code'),

  // Emergency Contact
  emergencyName: z.string().min(2, 'Emergency contact name is required'),
  emergencyRelationship: z.string().min(2, 'Relationship is required'),
  emergencyPhone: z.string().regex(/^\+?1?\d{10,14}$/, 'Invalid phone number'),

  // Documents
  photo: z.custom<FileList>()
    .refine((files) => files?.length === 1, 'Photo is required')
    .refine((files) => files?.[0]?.size <= 5 * 1024 * 1024, 'Max file size is 5MB')
    .refine(
      (files) => ['image/jpeg', 'image/jpg', 'image/png'].includes(files?.[0]?.type),
      'Only .jpg, .jpeg, .png formats are supported'
    ),

  // Agreements
  termsAccepted: z.boolean().refine((val) => val === true, 'You must accept the terms'),
});

type StudentFormData = z.infer<typeof studentSchema>;

export default function StudentRegistrationForm() {
  const [submitStatus, setSubmitStatus] = useState<'idle' | 'success' | 'error'>('idle');
  const [apiError, setApiError] = useState('');

  const {
    register,
    handleSubmit,
    watch,
    formState: { errors, isSubmitting, isDirty },
  } = useForm<StudentFormData>({
    resolver: zodResolver(studentSchema),
    mode: 'onBlur',
  });

  const photoFile = watch('photo');

  const onSubmit = async (data: StudentFormData) => {
    try {
      const formData = new FormData();

      // Append all fields except file
      Object.entries(data).forEach(([key, value]) => {
        if (key !== 'photo') {
          formData.append(key, String(value));
        }
      });

      // Append file
      if (data.photo?.[0]) {
        formData.append('photo', data.photo[0]);
      }

      const response = await fetch('/api/students/register', {
        method: 'POST',
        body: formData,
      });

      if (!response.ok) {
        const error = await response.json();
        setApiError(error.message || 'Registration failed');
        setSubmitStatus('error');
        return;
      }

      setSubmitStatus('success');
      console.log('Student registered successfully');

    } catch (error) {
      console.error('Error submitting form:', error);
      setApiError('Network error. Please try again.');
      setSubmitStatus('error');
    }
  };

  return (
    <div className="min-h-screen bg-gray-50 py-12 px-4">
      <div className="max-w-3xl mx-auto">
        <div className="bg-white rounded-lg shadow-md p-8">
          <h1 className="text-3xl font-bold text-gray-900 mb-2">Student Registration</h1>
          <p className="text-gray-600 mb-8">Please fill out all required fields to register</p>

          {submitStatus === 'success' && (
            <div className="mb-6 p-4 bg-green-100 border border-green-400 text-green-700 rounded">
              Registration successful! Welcome to the college.
            </div>
          )}

          {submitStatus === 'error' && apiError && (
            <div className="mb-6 p-4 bg-red-100 border border-red-400 text-red-700 rounded">
              {apiError}
            </div>
          )}

          <form onSubmit={handleSubmit(onSubmit)} className="space-y-8">
            {/* Personal Information */}
            <section>
              <h2 className="text-xl font-semibold mb-4 text-gray-800">Personal Information</h2>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    First Name <span className="text-red-500">*</span>
                  </label>
                  <input
                    {...register('firstName')}
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                  {errors.firstName && (
                    <p className="text-red-500 text-sm mt-1">{errors.firstName.message}</p>
                  )}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Last Name <span className="text-red-500">*</span>
                  </label>
                  <input
                    {...register('lastName')}
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                  {errors.lastName && (
                    <p className="text-red-500 text-sm mt-1">{errors.lastName.message}</p>
                  )}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Email <span className="text-red-500">*</span>
                  </label>
                  <input
                    type="email"
                    {...register('email')}
                    placeholder="name@college.edu"
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                  {errors.email && (
                    <p className="text-red-500 text-sm mt-1">{errors.email.message}</p>
                  )}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Phone <span className="text-red-500">*</span>
                  </label>
                  <input
                    {...register('phone')}
                    placeholder="+1234567890"
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                  {errors.phone && (
                    <p className="text-red-500 text-sm mt-1">{errors.phone.message}</p>
                  )}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Date of Birth <span className="text-red-500">*</span>
                  </label>
                  <input
                    type="date"
                    {...register('dateOfBirth')}
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                  {errors.dateOfBirth && (
                    <p className="text-red-500 text-sm mt-1">{errors.dateOfBirth.message}</p>
                  )}
                </div>
              </div>
            </section>

            {/* Academic Information */}
            <section>
              <h2 className="text-xl font-semibold mb-4 text-gray-800">Academic Information</h2>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Student ID <span className="text-red-500">*</span>
                  </label>
                  <input
                    {...register('studentId')}
                    placeholder="STU123456"
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                  {errors.studentId && (
                    <p className="text-red-500 text-sm mt-1">{errors.studentId.message}</p>
                  )}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Department <span className="text-red-500">*</span>
                  </label>
                  <select
                    {...register('department')}
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  >
                    <option value="">Select Department</option>
                    <option value="Computer Science">Computer Science</option>
                    <option value="Electrical Engineering">Electrical Engineering</option>
                    <option value="Mechanical Engineering">Mechanical Engineering</option>
                    <option value="Business Administration">Business Administration</option>
                    <option value="Biology">Biology</option>
                    <option value="Chemistry">Chemistry</option>
                  </select>
                  {errors.department && (
                    <p className="text-red-500 text-sm mt-1">{errors.department.message}</p>
                  )}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Expected Graduation <span className="text-red-500">*</span>
                  </label>
                  <input
                    type="month"
                    {...register('expectedGraduation')}
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                  {errors.expectedGraduation && (
                    <p className="text-red-500 text-sm mt-1">{errors.expectedGraduation.message}</p>
                  )}
                </div>
              </div>
            </section>

            {/* Address */}
            <section>
              <h2 className="text-xl font-semibold mb-4 text-gray-800">Address</h2>
              <div className="grid grid-cols-1 gap-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Street Address <span className="text-red-500">*</span>
                  </label>
                  <input
                    {...register('street')}
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                  {errors.street && (
                    <p className="text-red-500 text-sm mt-1">{errors.street.message}</p>
                  )}
                </div>

                <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">
                      City <span className="text-red-500">*</span>
                    </label>
                    <input
                      {...register('city')}
                      className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                    />
                    {errors.city && (
                      <p className="text-red-500 text-sm mt-1">{errors.city.message}</p>
                    )}
                  </div>

                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">
                      State <span className="text-red-500">*</span>
                    </label>
                    <input
                      {...register('state')}
                      placeholder="CA"
                      maxLength={2}
                      className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                    />
                    {errors.state && (
                      <p className="text-red-500 text-sm mt-1">{errors.state.message}</p>
                    )}
                  </div>

                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">
                      ZIP Code <span className="text-red-500">*</span>
                    </label>
                    <input
                      {...register('zipCode')}
                      placeholder="12345"
                      className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                    />
                    {errors.zipCode && (
                      <p className="text-red-500 text-sm mt-1">{errors.zipCode.message}</p>
                    )}
                  </div>
                </div>
              </div>
            </section>

            {/* Emergency Contact */}
            <section>
              <h2 className="text-xl font-semibold mb-4 text-gray-800">Emergency Contact</h2>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Name <span className="text-red-500">*</span>
                  </label>
                  <input
                    {...register('emergencyName')}
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                  {errors.emergencyName && (
                    <p className="text-red-500 text-sm mt-1">{errors.emergencyName.message}</p>
                  )}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Relationship <span className="text-red-500">*</span>
                  </label>
                  <input
                    {...register('emergencyRelationship')}
                    placeholder="Parent, Spouse, etc."
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                  {errors.emergencyRelationship && (
                    <p className="text-red-500 text-sm mt-1">{errors.emergencyRelationship.message}</p>
                  )}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Phone <span className="text-red-500">*</span>
                  </label>
                  <input
                    {...register('emergencyPhone')}
                    placeholder="+1234567890"
                    className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                  {errors.emergencyPhone && (
                    <p className="text-red-500 text-sm mt-1">{errors.emergencyPhone.message}</p>
                  )}
                </div>
              </div>
            </section>

            {/* Photo Upload */}
            <section>
              <h2 className="text-xl font-semibold mb-4 text-gray-800">Documents</h2>
              <div>
                <label className="block text-sm font-medium text-gray-700 mb-1">
                  Student Photo <span className="text-red-500">*</span>
                </label>
                <input
                  type="file"
                  accept="image/*"
                  {...register('photo')}
                  className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                />
                {photoFile?.[0] && (
                  <p className="text-sm text-gray-600 mt-1">
                    Selected: {photoFile[0].name} ({(photoFile[0].size / 1024).toFixed(2)} KB)
                  </p>
                )}
                {errors.photo && (
                  <p className="text-red-500 text-sm mt-1">{errors.photo.message as string}</p>
                )}
              </div>
            </section>

            {/* Terms */}
            <section>
              <div className="flex items-start">
                <input
                  type="checkbox"
                  {...register('termsAccepted')}
                  className="mt-1 mr-2"
                />
                <label className="text-sm text-gray-700">
                  I accept the terms and conditions and privacy policy <span className="text-red-500">*</span>
                </label>
              </div>
              {errors.termsAccepted && (
                <p className="text-red-500 text-sm mt-1">{errors.termsAccepted.message}</p>
              )}
            </section>

            {/* Submit Button */}
            <div className="flex gap-4">
              <button
                type="submit"
                disabled={isSubmitting || !isDirty}
                className="flex-1 bg-blue-600 text-white py-3 rounded-md hover:bg-blue-700 disabled:bg-gray-300 disabled:cursor-not-allowed font-semibold transition-colors"
              >
                {isSubmitting ? 'Registering...' : 'Complete Registration'}
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  );
}
```

---

## Best Practices

1. **Always Use TypeScript**: Get type safety for form data
2. **Use React Hook Form**: Less boilerplate, better performance
3. **Use Zod for Validation**: Type-safe schemas that work everywhere
4. **Validate on Blur**: Balance between UX and instant feedback
5. **Show Clear Error Messages**: Tell users exactly what's wrong
6. **Disable Submit During Submission**: Prevent duplicate submissions
7. **Show Loading States**: Let users know something is happening
8. **Handle API Errors Gracefully**: Network issues, validation errors, etc.
9. **Reset Form After Success**: Clear form or redirect user
10. **Use Controlled Components for Complex Logic**: Uncontrolled for simple forms

---

## Common Pitfalls

1. **Not Using valueAsNumber**: Number inputs return strings by default
2. **Forgetting to Prevent Default**: Always `e.preventDefault()` in form handlers
3. **Not Handling File Uploads Properly**: Files must use FormData
4. **Validating Too Aggressively**: Don't validate every keystroke unless needed
5. **Not Resetting Form State**: Can cause stale data issues
6. **Ignoring Accessibility**: Always use proper labels and error associations
7. **Not Testing Form Validation**: Test all validation scenarios
8. **Hardcoding Error Messages**: Use schema-based messages for consistency

---

## Next Steps

Now that you understand forms and validation, you're ready to learn about styling solutions in **Part 10: Styling Solutions**. You'll learn how to make your forms (and all components) look professional with Tailwind CSS, CSS Modules, and Styled Components!

---

**Navigation:**
- **Previous**: [Part 8: Advanced React Patterns](08_Advanced_React_Patterns.md)
- **Next**: [Part 10: Styling Solutions](10_Styling_Solutions.md)
- **[Back to Index](README.md)**

---

**Author: Saiprasad Hegde**

# Part 2: HTML, CSS & Responsive Design

**Author**: Saiprasad Hegde

## Table of Contents
- [HTML5 Semantic Elements](#html5-semantic-elements)
- [CSS Fundamentals](#css-fundamentals)
- [CSS Flexbox](#css-flexbox)
- [CSS Grid](#css-grid)
- [Responsive Design](#responsive-design)
- [Mobile-First Approach](#mobile-first-approach)
- [Accessibility Basics](#accessibility-basics)
- [Building a College Portal Layout](#building-a-college-portal-layout)
- [Summary](#summary)

---

## HTML5 Semantic Elements

**Semantic HTML** uses meaningful tags that describe the content's purpose, not just its appearance.

### Why Semantic HTML?

**Analogy**: Think of a building blueprint. Instead of labeling everything as "box," you specify "bedroom," "kitchen," "bathroom." This helps builders, inspectors, and future owners understand the structure.

**Benefits**:
✅ **SEO**: Search engines understand content better
✅ **Accessibility**: Screen readers can navigate properly
✅ **Maintainability**: Code is easier to read and understand
✅ **Consistency**: Standard structure across projects

### Non-Semantic vs Semantic

**Bad (Non-semantic)**:
```html
<div id="header">
  <div id="nav">
    <div class="link">Home</div>
    <div class="link">Courses</div>
  </div>
</div>
<div id="content">
  <div class="post">
    <div class="title">Course Announcement</div>
    <div class="text">...</div>
  </div>
</div>
<div id="footer">
  <div>© 2025</div>
</div>
```

**Good (Semantic)**:
```html
<header>
  <nav>
    <a href="/">Home</a>
    <a href="/courses">Courses</a>
  </nav>
</header>
<main>
  <article>
    <h1>Course Announcement</h1>
    <p>...</p>
  </article>
</main>
<footer>
  <p>© 2025</p>
</footer>
```

### Common Semantic Elements

#### Structure Elements

```html
<!-- Page structure -->
<header>
  <!-- Site header: logo, navigation -->
</header>

<nav>
  <!-- Navigation links -->
</nav>

<main>
  <!-- Main content (only one per page) -->
</main>

<aside>
  <!-- Sidebar, related content -->
</aside>

<footer>
  <!-- Footer: copyright, links -->
</footer>

<section>
  <!-- Thematic grouping of content -->
</section>

<article>
  <!-- Self-contained content (blog post, news article) -->
</article>
```

#### Content Elements

```html
<!-- Headings (h1 to h6) -->
<h1>Main Page Title</h1>
<h2>Section Title</h2>
<h3>Subsection Title</h3>

<!-- Paragraph -->
<p>This is a paragraph of text.</p>

<!-- Lists -->
<ul>
  <li>Unordered list item</li>
</ul>

<ol>
  <li>Ordered list item</li>
</ol>

<!-- Links -->
<a href="/courses">View Courses</a>

<!-- Images -->
<img src="/image.jpg" alt="Description for screen readers" />

<!-- Figure with caption -->
<figure>
  <img src="/chart.png" alt="Enrollment statistics" />
  <figcaption>Student enrollment by department</figcaption>
</figure>

<!-- Time -->
<time datetime="2025-01-15">January 15, 2025</time>

<!-- Emphasis -->
<strong>Important text</strong>
<em>Emphasized text</em>

<!-- Code -->
<code>const x = 10;</code>

<!-- Preformatted text -->
<pre><code>function hello() {
  console.log('Hello!');
}</code></pre>
```

### College Portal Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>College Portal - Home</title>
</head>
<body>
  <!-- Site header -->
  <header>
    <div class="logo">
      <h1>College Portal</h1>
    </div>
    <nav>
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/courses">Courses</a></li>
        <li><a href="/students">Students</a></li>
        <li><a href="/profile">My Profile</a></li>
      </ul>
    </nav>
  </header>

  <!-- Main content -->
  <main>
    <!-- Welcome section -->
    <section class="welcome">
      <h2>Welcome Back, John!</h2>
      <p>You have 4 active enrollments this semester.</p>
    </section>

    <!-- Enrolled courses -->
    <section class="enrolled-courses">
      <h2>Your Courses</h2>
      <article class="course-card">
        <h3>Computer Science 101</h3>
        <p>Introduction to Programming</p>
        <p>Instructor: <strong>Dr. Smith</strong></p>
        <p>Schedule: <time>Mon, Wed, Fri 10:00 AM</time></p>
        <a href="/courses/cs101">View Details</a>
      </article>

      <article class="course-card">
        <h3>Mathematics 201</h3>
        <p>Calculus II</p>
        <p>Instructor: <strong>Dr. Johnson</strong></p>
        <p>Schedule: <time>Tue, Thu 2:00 PM</time></p>
        <a href="/courses/math201">View Details</a>
      </article>
    </section>

    <!-- Announcements -->
    <aside class="announcements">
      <h2>Recent Announcements</h2>
      <article>
        <h3>Midterm Exam Schedule</h3>
        <time datetime="2025-01-10">January 10, 2025</time>
        <p>Midterm exams will be held from March 15-20.</p>
      </article>
    </aside>
  </main>

  <!-- Site footer -->
  <footer>
    <p>&copy; 2025 College Portal. All rights reserved.</p>
    <nav>
      <a href="/privacy">Privacy Policy</a>
      <a href="/terms">Terms of Service</a>
      <a href="/contact">Contact Us</a>
    </nav>
  </footer>
</body>
</html>
```

---

## CSS Fundamentals

CSS (Cascading Style Sheets) controls how HTML elements look.

### CSS Syntax

```css
selector {
  property: value;
  another-property: value;
}
```

**Example**:
```css
h1 {
  color: blue;
  font-size: 32px;
  margin-bottom: 16px;
}
```

### Ways to Add CSS

#### 1. Inline CSS (Avoid!)
```html
<h1 style="color: blue; font-size: 32px;">Title</h1>
```

#### 2. Internal CSS
```html
<head>
  <style>
    h1 {
      color: blue;
      font-size: 32px;
    }
  </style>
</head>
```

#### 3. External CSS (Best Practice)
```html
<head>
  <link rel="stylesheet" href="styles.css">
</head>
```

```css
/* styles.css */
h1 {
  color: blue;
  font-size: 32px;
}
```

### CSS Selectors

```css
/* Element selector */
p {
  color: black;
}

/* Class selector */
.course-card {
  padding: 20px;
  border: 1px solid #ccc;
}

/* ID selector */
#main-header {
  background-color: navy;
}

/* Descendant selector */
nav a {
  text-decoration: none;
}

/* Child selector */
nav > ul {
  list-style: none;
}

/* Multiple selectors */
h1, h2, h3 {
  font-family: Arial, sans-serif;
}

/* Pseudo-class */
a:hover {
  color: red;
}

button:disabled {
  opacity: 0.5;
}

/* Pseudo-element */
p::first-line {
  font-weight: bold;
}
```

### The Box Model

Every HTML element is a box:

```
┌─────────────────────────────────────┐
│           MARGIN                    │
│  ┌──────────────────────────────┐   │
│  │       BORDER                 │   │
│  │  ┌────────────────────────┐  │   │
│  │  │     PADDING            │  │   │
│  │  │  ┌──────────────────┐  │  │   │
│  │  │  │    CONTENT       │  │  │   │
│  │  │  │   (width/height) │  │  │   │
│  │  │  └──────────────────┘  │  │   │
│  │  └────────────────────────┘  │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

```css
.course-card {
  /* Content dimensions */
  width: 300px;
  height: 200px;

  /* Space inside the border */
  padding: 20px;

  /* Border */
  border: 2px solid #333;

  /* Space outside the border */
  margin: 10px;
}
```

**Total width** = width + padding-left + padding-right + border-left + border-right + margin-left + margin-right

**Box-sizing fix**:
```css
* {
  box-sizing: border-box; /* Include padding and border in width/height */
}
```

### Common CSS Properties

```css
.element {
  /* Text */
  color: #333;
  font-size: 16px;
  font-weight: 700;
  font-family: Arial, sans-serif;
  text-align: center;
  line-height: 1.5;

  /* Background */
  background-color: #f0f0f0;
  background-image: url('image.jpg');

  /* Spacing */
  margin: 10px;           /* All sides */
  margin: 10px 20px;      /* Top/bottom, left/right */
  margin: 10px 20px 30px; /* Top, left/right, bottom */
  margin: 10px 20px 30px 40px; /* Top, right, bottom, left */

  padding: 15px;

  /* Border */
  border: 2px solid #333;
  border-radius: 8px;     /* Rounded corners */

  /* Display */
  display: block;         /* Block-level */
  display: inline;        /* Inline */
  display: inline-block;  /* Inline but can have width/height */
  display: flex;          /* Flexbox */
  display: grid;          /* Grid */
  display: none;          /* Hidden */

  /* Positioning */
  position: relative;
  position: absolute;
  position: fixed;
  position: sticky;

  /* Dimensions */
  width: 100%;
  max-width: 1200px;
  height: auto;
  min-height: 500px;

  /* Shadows */
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);

  /* Transitions */
  transition: all 0.3s ease;

  /* Cursor */
  cursor: pointer;
}
```

---

## CSS Flexbox

**Flexbox** is a layout model for arranging items in rows or columns.

**Analogy**: Think of Flexbox like organizing books on a shelf. You can stack them left-to-right, center them, space them evenly, or wrap them to a new shelf when full.

### Flex Container

```css
.container {
  display: flex;

  /* Direction */
  flex-direction: row;         /* Left to right (default) */
  flex-direction: row-reverse; /* Right to left */
  flex-direction: column;      /* Top to bottom */
  flex-direction: column-reverse;

  /* Wrapping */
  flex-wrap: nowrap;   /* Single line (default) */
  flex-wrap: wrap;     /* Multiple lines */

  /* Horizontal alignment (main axis) */
  justify-content: flex-start;    /* Start */
  justify-content: flex-end;      /* End */
  justify-content: center;        /* Center */
  justify-content: space-between; /* Space between items */
  justify-content: space-around;  /* Space around items */
  justify-content: space-evenly;  /* Equal space */

  /* Vertical alignment (cross axis) */
  align-items: stretch;     /* Stretch to fill (default) */
  align-items: flex-start;  /* Top */
  align-items: flex-end;    /* Bottom */
  align-items: center;      /* Center */
  align-items: baseline;    /* Align baselines */

  /* Gap between items */
  gap: 16px;
}
```

### Flex Items

```css
.item {
  /* Grow factor */
  flex-grow: 1; /* Take up remaining space */

  /* Shrink factor */
  flex-shrink: 1; /* Can shrink if needed */

  /* Initial size */
  flex-basis: 200px;

  /* Shorthand */
  flex: 1; /* flex-grow: 1, flex-shrink: 1, flex-basis: 0 */

  /* Individual alignment */
  align-self: center;
}
```

### Flexbox Examples

#### Horizontal Navigation

```html
<nav class="navbar">
  <div class="logo">College Portal</div>
  <ul class="nav-links">
    <li><a href="/">Home</a></li>
    <li><a href="/courses">Courses</a></li>
    <li><a href="/students">Students</a></li>
  </ul>
  <button class="login-btn">Login</button>
</nav>
```

```css
.navbar {
  display: flex;
  justify-content: space-between; /* Logo left, button right */
  align-items: center;
  padding: 16px;
  background-color: #1e40af;
  color: white;
}

.nav-links {
  display: flex;
  gap: 24px;
  list-style: none;
}
```

#### Course Cards Grid (with Flexbox)

```html
<div class="course-grid">
  <div class="course-card">CS 101</div>
  <div class="course-card">Math 201</div>
  <div class="course-card">Physics 301</div>
  <div class="course-card">English 101</div>
</div>
```

```css
.course-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.course-card {
  flex: 1 1 300px; /* Grow, shrink, base width 300px */
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}
```

#### Centered Content

```css
.center-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
```

---

## CSS Grid

**CSS Grid** is a 2D layout system for creating rows and columns.

**Analogy**: Grid is like a spreadsheet. You define rows and columns, then place items in specific cells.

### Grid Container

```css
.grid-container {
  display: grid;

  /* Define columns */
  grid-template-columns: 200px 200px 200px;  /* 3 fixed columns */
  grid-template-columns: 1fr 1fr 1fr;        /* 3 equal columns */
  grid-template-columns: 1fr 2fr 1fr;        /* Middle column twice as wide */
  grid-template-columns: repeat(3, 1fr);     /* 3 equal columns (shorthand) */
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); /* Responsive */

  /* Define rows */
  grid-template-rows: 100px 200px;
  grid-template-rows: auto auto;

  /* Gap between items */
  gap: 20px;
  row-gap: 20px;
  column-gap: 20px;
}
```

### Grid Items

```css
.grid-item {
  /* Span multiple columns */
  grid-column: 1 / 3;  /* Start at column 1, end before column 3 */
  grid-column: span 2; /* Span 2 columns */

  /* Span multiple rows */
  grid-row: 1 / 3;
  grid-row: span 2;

  /* Place in specific cell */
  grid-column: 2;
  grid-row: 1;
}
```

### Grid Examples

#### Dashboard Layout

```html
<div class="dashboard">
  <header class="header">Header</header>
  <aside class="sidebar">Sidebar</aside>
  <main class="main">Main Content</main>
  <footer class="footer">Footer</footer>
</div>
```

```css
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: 80px 1fr 60px;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  height: 100vh;
  gap: 16px;
}

.header {
  grid-area: header;
  background-color: #1e40af;
}

.sidebar {
  grid-area: sidebar;
  background-color: #f3f4f6;
}

.main {
  grid-area: main;
  background-color: white;
}

.footer {
  grid-area: footer;
  background-color: #374151;
}
```

#### Course Cards Grid

```html
<div class="courses-grid">
  <div class="course-card">CS 101</div>
  <div class="course-card">Math 201</div>
  <div class="course-card">Physics 301</div>
  <div class="course-card">English 101</div>
  <div class="course-card">History 201</div>
</div>
```

```css
.courses-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

.course-card {
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: white;
}
```

---

## Responsive Design

**Responsive design** makes websites work on all devices (desktop, tablet, mobile).

**Analogy**: Like water taking the shape of its container, responsive websites adapt to any screen size.

### Viewport Meta Tag

Always include this in `<head>`:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

### Media Queries

Media queries apply CSS based on screen size:

```css
/* Mobile styles (base styles) */
.container {
  padding: 10px;
  font-size: 14px;
}

/* Tablet and up */
@media (min-width: 768px) {
  .container {
    padding: 20px;
    font-size: 16px;
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .container {
    padding: 40px;
    max-width: 1200px;
    margin: 0 auto;
  }
}

/* Large desktop */
@media (min-width: 1440px) {
  .container {
    max-width: 1400px;
  }
}
```

### Common Breakpoints

```css
/* Mobile: 0-767px (default styles) */

/* Tablet: 768px and up */
@media (min-width: 768px) { }

/* Desktop: 1024px and up */
@media (min-width: 1024px) { }

/* Large desktop: 1440px and up */
@media (min-width: 1440px) { }
```

### Responsive Navigation

```html
<nav class="navbar">
  <div class="logo">College Portal</div>
  <button class="menu-toggle">☰</button>
  <ul class="nav-links">
    <li><a href="/">Home</a></li>
    <li><a href="/courses">Courses</a></li>
    <li><a href="/students">Students</a></li>
  </ul>
</nav>
```

```css
/* Mobile: Stack vertically */
.navbar {
  display: flex;
  flex-direction: column;
  padding: 16px;
}

.menu-toggle {
  display: block;
}

.nav-links {
  display: none; /* Hidden by default on mobile */
  flex-direction: column;
  gap: 8px;
}

.nav-links.active {
  display: flex;
}

/* Desktop: Horizontal layout */
@media (min-width: 768px) {
  .navbar {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }

  .menu-toggle {
    display: none; /* Hide hamburger menu */
  }

  .nav-links {
    display: flex; /* Always visible */
    flex-direction: row;
    gap: 24px;
  }
}
```

### Responsive Grid

```css
.course-grid {
  display: grid;
  gap: 20px;

  /* Mobile: 1 column */
  grid-template-columns: 1fr;
}

/* Tablet: 2 columns */
@media (min-width: 768px) {
  .course-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop: 3 columns */
@media (min-width: 1024px) {
  .course-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* Large desktop: 4 columns */
@media (min-width: 1440px) {
  .course-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}
```

### Responsive Images

```css
img {
  max-width: 100%;
  height: auto;
}
```

### Responsive Typography

```css
body {
  font-size: 14px;
}

h1 {
  font-size: 24px;
}

@media (min-width: 768px) {
  body {
    font-size: 16px;
  }

  h1 {
    font-size: 32px;
  }
}

@media (min-width: 1024px) {
  body {
    font-size: 18px;
  }

  h1 {
    font-size: 48px;
  }
}
```

---

## Mobile-First Approach

**Mobile-first** means writing CSS for mobile screens first, then adding styles for larger screens.

**Why mobile-first?**
✅ Forces you to prioritize content
✅ Easier to add features than remove them
✅ Better performance on mobile (don't download unused styles)
✅ Most users are on mobile

### Mobile-First Example

```css
/* Mobile styles (default) */
.container {
  padding: 16px;
  font-size: 14px;
}

.sidebar {
  display: none; /* Hide sidebar on mobile */
}

.main {
  width: 100%;
}

/* Tablet and larger: Add sidebar */
@media (min-width: 768px) {
  .container {
    display: grid;
    grid-template-columns: 250px 1fr;
    gap: 20px;
  }

  .sidebar {
    display: block; /* Show sidebar */
  }
}

/* Desktop: Larger typography */
@media (min-width: 1024px) {
  .container {
    font-size: 16px;
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

---

## Accessibility Basics

**Accessibility (a11y)** ensures everyone can use your website, including people with disabilities.

### Why Accessibility Matters

✅ **Legal requirement** in many countries
✅ **Larger audience** (15% of people have disabilities)
✅ **Better UX** for everyone
✅ **Better SEO** (semantic HTML helps search engines)

### Accessibility Best Practices

#### 1. Semantic HTML

```html
<!-- Good: Semantic button -->
<button>Click me</button>

<!-- Bad: Non-semantic div -->
<div onclick="handleClick()">Click me</div>
```

#### 2. Alt Text for Images

```html
<!-- Good: Descriptive alt text -->
<img src="chart.png" alt="Bar chart showing enrollment by department">

<!-- Bad: Missing or generic alt text -->
<img src="chart.png">
<img src="chart.png" alt="image">
```

#### 3. Keyboard Navigation

```css
/* Show focus outline */
button:focus,
a:focus {
  outline: 2px solid blue;
  outline-offset: 2px;
}

/* Don't remove outlines! */
/* button:focus { outline: none; } ❌ BAD */
```

#### 4. Color Contrast

```css
/* Good: High contrast (4.5:1 ratio minimum) */
.text {
  color: #333;
  background-color: #fff;
}

/* Bad: Low contrast */
.text {
  color: #ccc;
  background-color: #fff;
}
```

#### 5. Form Labels

```html
<!-- Good: Proper label association -->
<label for="email">Email address</label>
<input type="email" id="email" name="email">

<!-- Bad: No label -->
<input type="email" placeholder="Email">
```

#### 6. ARIA Labels (when needed)

```html
<!-- When visual text isn't enough -->
<button aria-label="Close dialog">×</button>

<!-- Indicate current page in navigation -->
<nav>
  <a href="/" aria-current="page">Home</a>
  <a href="/courses">Courses</a>
</nav>
```

---

## Building a College Portal Layout

Let's put it all together and build a complete responsive layout for our College Management System.

### HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>College Portal - Dashboard</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <!-- Navigation -->
  <nav class="navbar">
    <div class="logo">
      <h1>College Portal</h1>
    </div>
    <button class="menu-toggle" aria-label="Toggle menu">☰</button>
    <ul class="nav-links">
      <li><a href="/" aria-current="page">Dashboard</a></li>
      <li><a href="/courses">Courses</a></li>
      <li><a href="/students">Students</a></li>
      <li><a href="/profile">Profile</a></li>
    </ul>
    <div class="user-menu">
      <img src="/avatar.jpg" alt="John Doe profile picture" class="avatar">
      <span>John Doe</span>
    </div>
  </nav>

  <!-- Main content area -->
  <div class="main-container">
    <!-- Sidebar -->
    <aside class="sidebar">
      <h2>Quick Links</h2>
      <ul>
        <li><a href="/enrollments">My Enrollments</a></li>
        <li><a href="/grades">My Grades</a></li>
        <li><a href="/schedule">Class Schedule</a></li>
        <li><a href="/transcript">Transcript</a></li>
      </ul>

      <h2>Announcements</h2>
      <div class="announcement-card">
        <h3>Midterm Schedule</h3>
        <time datetime="2025-01-15">Jan 15, 2025</time>
        <p>Midterms will be held March 15-20.</p>
      </div>
    </aside>

    <!-- Main content -->
    <main class="main-content">
      <header class="page-header">
        <h1>Welcome back, John!</h1>
        <p>You have 4 active enrollments this semester.</p>
      </header>

      <!-- Stats cards -->
      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-icon">📚</div>
          <div class="stat-info">
            <h3>4</h3>
            <p>Enrolled Courses</p>
          </div>
        </div>
        <div class="stat-card">
          <div class="stat-icon">✅</div>
          <div class="stat-info">
            <h3>3.7</h3>
            <p>GPA</p>
          </div>
        </div>
        <div class="stat-card">
          <div class="stat-icon">📅</div>
          <div class="stat-info">
            <h3>12</h3>
            <p>Credits</p>
          </div>
        </div>
        <div class="stat-card">
          <div class="stat-icon">🎓</div>
          <div class="stat-info">
            <h3>45</h3>
            <p>Total Credits</p>
          </div>
        </div>
      </div>

      <!-- Course cards -->
      <section class="courses-section">
        <h2>Your Courses</h2>
        <div class="courses-grid">
          <article class="course-card">
            <div class="course-header">
              <h3>Computer Science 101</h3>
              <span class="course-code">CS101</span>
            </div>
            <p class="course-description">
              Introduction to Programming
            </p>
            <div class="course-details">
              <p><strong>Instructor:</strong> Dr. Sarah Smith</p>
              <p><strong>Schedule:</strong> Mon, Wed, Fri 10:00 AM</p>
              <p><strong>Credits:</strong> 3</p>
            </div>
            <div class="course-actions">
              <a href="/courses/cs101" class="btn btn-primary">View Course</a>
            </div>
          </article>

          <article class="course-card">
            <div class="course-header">
              <h3>Mathematics 201</h3>
              <span class="course-code">MATH201</span>
            </div>
            <p class="course-description">
              Calculus II
            </p>
            <div class="course-details">
              <p><strong>Instructor:</strong> Dr. Michael Johnson</p>
              <p><strong>Schedule:</strong> Tue, Thu 2:00 PM</p>
              <p><strong>Credits:</strong> 4</p>
            </div>
            <div class="course-actions">
              <a href="/courses/math201" class="btn btn-primary">View Course</a>
            </div>
          </article>

          <article class="course-card">
            <div class="course-header">
              <h3>Physics 301</h3>
              <span class="course-code">PHY301</span>
            </div>
            <p class="course-description">
              Classical Mechanics
            </p>
            <div class="course-details">
              <p><strong>Instructor:</strong> Dr. Emily Brown</p>
              <p><strong>Schedule:</strong> Mon, Wed 1:00 PM</p>
              <p><strong>Credits:</strong> 3</p>
            </div>
            <div class="course-actions">
              <a href="/courses/phy301" class="btn btn-primary">View Course</a>
            </div>
          </article>

          <article class="course-card">
            <div class="course-header">
              <h3>English 101</h3>
              <span class="course-code">ENG101</span>
            </div>
            <p class="course-description">
              English Composition
            </p>
            <div class="course-details">
              <p><strong>Instructor:</strong> Prof. David Lee</p>
              <p><strong>Schedule:</strong> Tue, Thu 10:00 AM</p>
              <p><strong>Credits:</strong> 3</p>
            </div>
            <div class="course-actions">
              <a href="/courses/eng101" class="btn btn-primary">View Course</a>
            </div>
          </article>
        </div>
      </section>
    </main>
  </div>

  <!-- Footer -->
  <footer class="footer">
    <p>&copy; 2025 College Portal. All rights reserved.</p>
    <nav>
      <a href="/privacy">Privacy</a>
      <a href="/terms">Terms</a>
      <a href="/contact">Contact</a>
    </nav>
  </footer>

  <script src="script.js"></script>
</body>
</html>
```

### Complete CSS (styles.css)

```css
/* ===== CSS RESET ===== */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* ===== ROOT VARIABLES ===== */
:root {
  /* Colors */
  --primary: #1e40af;
  --primary-dark: #1e3a8a;
  --secondary: #64748b;
  --success: #10b981;
  --danger: #ef4444;
  --warning: #f59e0b;

  --gray-50: #f9fafb;
  --gray-100: #f3f4f6;
  --gray-200: #e5e7eb;
  --gray-300: #d1d5db;
  --gray-600: #4b5563;
  --gray-700: #374151;
  --gray-900: #111827;

  /* Spacing */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;

  /* Border radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
}

/* ===== BASE STYLES ===== */
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  line-height: 1.6;
  color: var(--gray-900);
  background-color: var(--gray-50);
}

a {
  color: var(--primary);
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

ul {
  list-style: none;
}

/* ===== NAVBAR ===== */
.navbar {
  background-color: var(--primary);
  color: white;
  padding: var(--spacing-md);
  display: flex;
  flex-direction: column;
  gap: var(--spacing-md);
}

.logo h1 {
  font-size: 24px;
}

.menu-toggle {
  background: none;
  border: none;
  color: white;
  font-size: 24px;
  cursor: pointer;
  align-self: flex-end;
}

.nav-links {
  display: none;
  flex-direction: column;
  gap: var(--spacing-sm);
}

.nav-links.active {
  display: flex;
}

.nav-links a {
  color: white;
  padding: var(--spacing-sm);
  border-radius: var(--radius-sm);
}

.nav-links a:hover,
.nav-links a[aria-current="page"] {
  background-color: var(--primary-dark);
  text-decoration: none;
}

.user-menu {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
}

.avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
}

/* ===== MAIN CONTAINER ===== */
.main-container {
  display: flex;
  flex-direction: column;
  min-height: calc(100vh - 120px);
}

/* ===== SIDEBAR ===== */
.sidebar {
  background-color: white;
  padding: var(--spacing-lg);
  display: none; /* Hidden on mobile */
}

.sidebar h2 {
  font-size: 18px;
  margin-bottom: var(--spacing-md);
  color: var(--gray-700);
}

.sidebar ul {
  margin-bottom: var(--spacing-xl);
}

.sidebar li {
  margin-bottom: var(--spacing-sm);
}

.sidebar a {
  color: var(--gray-700);
  display: block;
  padding: var(--spacing-sm);
  border-radius: var(--radius-sm);
}

.sidebar a:hover {
  background-color: var(--gray-100);
  text-decoration: none;
}

.announcement-card {
  padding: var(--spacing-md);
  background-color: var(--gray-50);
  border-radius: var(--radius-md);
  border-left: 4px solid var(--primary);
}

.announcement-card h3 {
  font-size: 16px;
  margin-bottom: var(--spacing-xs);
}

.announcement-card time {
  font-size: 14px;
  color: var(--gray-600);
  display: block;
  margin-bottom: var(--spacing-sm);
}

/* ===== MAIN CONTENT ===== */
.main-content {
  flex: 1;
  padding: var(--spacing-lg);
}

.page-header {
  margin-bottom: var(--spacing-xl);
}

.page-header h1 {
  font-size: 28px;
  margin-bottom: var(--spacing-sm);
}

.page-header p {
  color: var(--gray-600);
}

/* ===== STATS GRID ===== */
.stats-grid {
  display: grid;
  gap: var(--spacing-md);
  margin-bottom: var(--spacing-xl);
  grid-template-columns: 1fr;
}

.stat-card {
  background-color: white;
  padding: var(--spacing-lg);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
  display: flex;
  align-items: center;
  gap: var(--spacing-md);
}

.stat-icon {
  font-size: 36px;
}

.stat-info h3 {
  font-size: 32px;
  font-weight: bold;
  color: var(--primary);
}

.stat-info p {
  color: var(--gray-600);
  font-size: 14px;
}

/* ===== COURSES SECTION ===== */
.courses-section h2 {
  font-size: 24px;
  margin-bottom: var(--spacing-lg);
}

.courses-grid {
  display: grid;
  gap: var(--spacing-lg);
  grid-template-columns: 1fr;
}

.course-card {
  background-color: white;
  border-radius: var(--radius-lg);
  padding: var(--spacing-lg);
  box-shadow: var(--shadow-md);
  transition: transform 0.2s, box-shadow 0.2s;
}

.course-card:hover {
  transform: translateY(-4px);
  box-shadow: var(--shadow-lg);
}

.course-header {
  display: flex;
  justify-content: space-between;
  align-items: start;
  margin-bottom: var(--spacing-md);
}

.course-header h3 {
  font-size: 20px;
  color: var(--gray-900);
}

.course-code {
  background-color: var(--primary);
  color: white;
  padding: 4px 8px;
  border-radius: var(--radius-sm);
  font-size: 12px;
  font-weight: bold;
}

.course-description {
  color: var(--gray-600);
  margin-bottom: var(--spacing-md);
}

.course-details {
  margin-bottom: var(--spacing-md);
  font-size: 14px;
  color: var(--gray-700);
}

.course-details p {
  margin-bottom: var(--spacing-xs);
}

.course-actions {
  display: flex;
  gap: var(--spacing-sm);
}

/* ===== BUTTONS ===== */
.btn {
  display: inline-block;
  padding: 10px 20px;
  border-radius: var(--radius-md);
  font-weight: 600;
  text-align: center;
  cursor: pointer;
  transition: all 0.2s;
  border: none;
  font-size: 14px;
}

.btn-primary {
  background-color: var(--primary);
  color: white;
}

.btn-primary:hover {
  background-color: var(--primary-dark);
  text-decoration: none;
}

/* ===== FOOTER ===== */
.footer {
  background-color: var(--gray-700);
  color: white;
  padding: var(--spacing-lg);
  text-align: center;
}

.footer nav {
  margin-top: var(--spacing-md);
  display: flex;
  justify-content: center;
  gap: var(--spacing-lg);
}

.footer a {
  color: white;
}

/* ===== TABLET & UP (768px+) ===== */
@media (min-width: 768px) {
  /* Navbar */
  .navbar {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }

  .menu-toggle {
    display: none;
  }

  .nav-links {
    display: flex;
    flex-direction: row;
    gap: var(--spacing-lg);
  }

  /* Main container */
  .main-container {
    display: grid;
    grid-template-columns: 250px 1fr;
  }

  .sidebar {
    display: block;
  }

  /* Stats grid: 2 columns */
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  /* Courses grid: 2 columns */
  .courses-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* ===== DESKTOP & UP (1024px+) ===== */
@media (min-width: 1024px) {
  /* Stats grid: 4 columns */
  .stats-grid {
    grid-template-columns: repeat(4, 1fr);
  }

  /* Main container */
  .main-container {
    grid-template-columns: 280px 1fr;
  }

  /* Courses grid: 3 columns */
  .courses-grid {
    grid-template-columns: repeat(3, 1fr);
  }

  .main-content {
    max-width: 1400px;
  }
}

/* ===== LARGE DESKTOP (1440px+) ===== */
@media (min-width: 1440px) {
  .main-content {
    padding: var(--spacing-xl);
  }
}

/* ===== FOCUS STYLES (Accessibility) ===== */
*:focus {
  outline: 2px solid var(--primary);
  outline-offset: 2px;
}

button:focus,
a:focus {
  outline: 2px solid var(--warning);
}
```

### JavaScript for Menu Toggle (script.js)

```javascript
// Mobile menu toggle
const menuToggle = document.querySelector('.menu-toggle');
const navLinks = document.querySelector('.nav-links');

if (menuToggle) {
  menuToggle.addEventListener('click', () => {
    navLinks.classList.toggle('active');
  });
}
```

---

## Summary

### Key Takeaways

✅ **Semantic HTML** improves SEO, accessibility, and code readability
✅ **CSS Flexbox** is perfect for 1D layouts (rows or columns)
✅ **CSS Grid** excels at 2D layouts (rows and columns)
✅ **Responsive design** adapts to all screen sizes using media queries
✅ **Mobile-first approach** starts with mobile styles and adds complexity
✅ **Accessibility** ensures everyone can use your website

### What We Covered

1. HTML5 semantic elements for better structure
2. CSS fundamentals (selectors, box model, properties)
3. Flexbox for flexible layouts
4. Grid for 2D grid layouts
5. Responsive design with media queries
6. Mobile-first development approach
7. Accessibility best practices
8. Complete College Portal layout example

### What's Next?

In **Part 3**, we'll dive into:
- Modern JavaScript (ES6+)
- Arrow functions, destructuring, spread operators
- Promises and async/await
- Array methods (map, filter, reduce)
- DOM manipulation

---

**Next**: [Part 3: Modern JavaScript & ES6+](./03_JavaScript_Modern.md)

**Previous**: [Part 1: Introduction to Frontend Development](./01_Introduction.md)

[← Back to Index](./README.md)

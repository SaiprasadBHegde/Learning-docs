# JavaScript & TypeScript Complete Guide
## From Basics to Advanced: A Beginner-Friendly Journey

**Author**: Saiprasad Hegde

---

## Table of Contents

### Part 1: JavaScript Fundamentals
1. [Introduction to JavaScript](#introduction-to-javascript)
2. [Getting Started](#getting-started)
3. [Variables and Data Types](#variables-and-data-types)
4. [Operators](#operators)
5. [Control Flow](#control-flow)
6. [Functions](#functions)
7. [Arrays](#arrays)
8. [Objects](#objects)
9. [Strings and Template Literals](#strings-and-template-literals)

### Part 2: JavaScript Intermediate
10. [How JavaScript Works](#how-javascript-works)
11. [The Event Loop](#the-event-loop)
12. [Call Stack, Web APIs, and Task Queue](#call-stack-web-apis-and-task-queue)
13. [Scope and Hoisting](#scope-and-hoisting)
14. [Closures](#closures)
15. [The 'this' Keyword](#the-this-keyword)
16. [Arrow Functions](#arrow-functions)
17. [Destructuring](#destructuring)
18. [Spread and Rest Operators](#spread-and-rest-operators)
19. [Array Methods](#array-methods)
20. [Object Methods](#object-methods)

### Part 3: JavaScript Advanced
18. [Asynchronous JavaScript](#asynchronous-javascript)
19. [Promises](#promises)
20. [Async/Await](#async-await)
21. [ES6 Modules](#es6-modules)
22. [Classes](#classes)
23. [Prototypes and Inheritance](#prototypes-and-inheritance)
24. [Error Handling](#error-handling)
25. [Regular Expressions](#regular-expressions)

### Part 4: TypeScript
26. [Introduction to TypeScript](#introduction-to-typescript)
27. [TypeScript Basics](#typescript-basics)
28. [Type Annotations](#type-annotations)
29. [Interfaces](#interfaces)
30. [Type Aliases](#type-aliases)
31. [Union and Intersection Types](#union-and-intersection-types)
32. [Generics](#generics)
33. [Advanced TypeScript](#advanced-typescript)

### Part 5: Modern JavaScript & Best Practices
34. [DOM Manipulation](#dom-manipulation)
35. [Fetch API and HTTP Requests](#fetch-api-and-http-requests)
36. [Local Storage and Session Storage](#local-storage-and-session-storage)
37. [Best Practices](#best-practices)
38. [Common Patterns](#common-patterns)
39. [Resources and Next Steps](#resources-and-next-steps)

---

## Introduction to JavaScript

### What is JavaScript?

**JavaScript** is a programming language that makes web pages interactive. It's one of the three core technologies of the web, alongside HTML and CSS.

**Analogy**: Think of a house:
- **HTML** is the structure (walls, roof, rooms)
- **CSS** is the decoration (paint, furniture, style)
- **JavaScript** is the electricity (lights that turn on/off, appliances that work)

### What Can JavaScript Do?

- Make websites interactive (buttons, forms, animations)
- Build web applications (Gmail, Facebook, Netflix)
- Create mobile apps (React Native)
- Build server applications (Node.js)
- Develop desktop apps (Electron)
- Control IoT devices
- Create games

### Why Learn JavaScript?

1. **Most Popular Language**: Used everywhere on the web
2. **Beginner-Friendly**: Relatively easy to start with
3. **Versatile**: Front-end, back-end, mobile, desktop
4. **Great Job Market**: High demand for JavaScript developers
5. **Active Community**: Tons of resources and libraries

### JavaScript vs TypeScript

**JavaScript**:
- Dynamic typing (types checked at runtime)
- Flexible but can lead to bugs
- No compilation needed

**TypeScript**:
- Static typing (types checked before running)
- Catches errors early
- Compiles to JavaScript
- Superset of JavaScript (all JS is valid TS)

**Analogy**:
- **JavaScript** is like speaking freely without grammar rules - fast but might be misunderstood
- **TypeScript** is like having a grammar checker - takes more effort but prevents mistakes

---

## Getting Started

### Setting Up Your Environment

#### Option 1: Browser Console (Easiest)

1. Open any web browser (Chrome, Firefox, Edge)
2. Press `F12` or right-click → "Inspect"
3. Go to "Console" tab
4. Start typing JavaScript!

```javascript
console.log("Hello, World!");
// Output: Hello, World!
```

#### Option 2: HTML File

Create a file called `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My First JavaScript</title>
</head>
<body>
    <h1>Check the console!</h1>

    <script>
        console.log("Hello from JavaScript!");
        alert("Welcome!");
    </script>
</body>
</html>
```

#### Option 3: Node.js (For Running JS Outside Browser)

1. Download from [nodejs.org](https://nodejs.org)
2. Install it
3. Create `app.js`:
```javascript
console.log("Hello from Node.js!");
```
4. Run: `node app.js`

### Your First JavaScript Program

```javascript
// This is a comment - JavaScript ignores this line

// Print to console
console.log("Hello, World!");

// Show a popup
alert("Welcome to JavaScript!");

// Get user input
let name = prompt("What's your name?");
console.log("Hello, " + name);
```

---

## Variables and Data Types

### What are Variables?

**Variables** are containers that store data values.

**Analogy**: Think of variables as labeled boxes where you can store things.

### Declaring Variables

JavaScript has three ways to declare variables:

```javascript
// 1. var (old way - avoid using)
var oldWay = "Don't use this";

// 2. let (modern way - can be reassigned)
let age = 25;
age = 26; // ✅ Works

// 3. const (modern way - cannot be reassigned)
const name = "John";
// name = "Jane"; // ❌ Error! Can't reassign const
```

**Rule of Thumb**:
- Use `const` by default
- Use `let` only if you need to reassign
- Never use `var`

### Naming Rules

```javascript
// ✅ Good variable names
let firstName = "John";
let age = 25;
let isStudent = true;
let totalPrice = 99.99;

// ❌ Invalid names
let 123abc = "nope";      // Can't start with number
let first-name = "nope";  // Can't use hyphens
let let = "nope";         // Can't use reserved words

// 📝 Naming conventions
let camelCase = "standard";     // ✅ Use this
let PascalCase = "for classes"; // ✅ Use for classes/constructors
let snake_case = "avoid";       // ❌ Not common in JS
```

### Data Types

JavaScript has 7 primitive data types:

#### 1. String (Text)

```javascript
let message = "Hello, World!";
let name = 'John';
let greeting = `Hi, ${name}!`; // Template literal (backticks)

console.log(greeting); // Hi, John!
```

#### 2. Number

```javascript
let age = 25;
let price = 19.99;
let negative = -10;
let infinity = Infinity;
let notANumber = NaN; // Special "not a number" value

// All numbers are floating point in JavaScript
let x = 5;
let y = 5.0; // Same as x
```

#### 3. Boolean (True/False)

```javascript
let isStudent = true;
let hasLicense = false;
let canVote = age >= 18; // true if age is 18 or more
```

**Analogy**: Like a light switch - it's either ON (true) or OFF (false).

#### 4. Undefined

```javascript
let x; // Declared but not assigned
console.log(x); // undefined

// Function with no return
function doNothing() {}
console.log(doNothing()); // undefined
```

**Analogy**: An empty box that exists but has nothing in it yet.

#### 5. Null

```javascript
let user = null; // Intentionally empty

// Difference from undefined:
let a; // undefined - "I forgot to put something here"
let b = null; // null - "I deliberately made this empty"
```

#### 6. Symbol (Unique Identifier)

```javascript
let id = Symbol("id");
let id2 = Symbol("id");

console.log(id === id2); // false - each Symbol is unique
```

**Note**: Symbols are advanced - you'll rarely use them as a beginner.

#### 7. BigInt (Large Numbers)

```javascript
let bigNumber = 123456789012345678901234567890n; // Add 'n' at end
let alsoBig = BigInt("999999999999999999999");
```

### Objects (Complex Data Type)

```javascript
let person = {
    name: "John",
    age: 30,
    isStudent: false
};

console.log(person.name); // John
```

### Arrays (List of Values)

```javascript
let fruits = ["apple", "banana", "orange"];
let numbers = [1, 2, 3, 4, 5];
let mixed = [1, "text", true, null]; // Can mix types

console.log(fruits[0]); // apple (arrays start at 0)
```

### Checking Types

```javascript
console.log(typeof "hello");     // string
console.log(typeof 42);          // number
console.log(typeof true);        // boolean
console.log(typeof undefined);   // undefined
console.log(typeof null);        // object (this is a bug in JS!)
console.log(typeof {});          // object
console.log(typeof []);          // object (arrays are objects)
```

### Type Conversion

```javascript
// String to Number
let str = "123";
let num = Number(str);      // 123
let num2 = parseInt(str);   // 123
let num3 = parseFloat("3.14"); // 3.14
let num4 = +"456";          // 456 (shorthand)

// Number to String
let age = 25;
let ageStr = String(age);   // "25"
let ageStr2 = age.toString(); // "25"
let ageStr3 = "" + age;     // "25" (shorthand)

// To Boolean
Boolean(1);        // true
Boolean(0);        // false
Boolean("text");   // true
Boolean("");       // false
Boolean(null);     // false
Boolean(undefined); // false

// Truthy and Falsy
// Falsy values: false, 0, "", null, undefined, NaN
// Everything else is truthy
```

---

## Operators

### Arithmetic Operators

```javascript
let a = 10;
let b = 3;

console.log(a + b);  // 13 (addition)
console.log(a - b);  // 7  (subtraction)
console.log(a * b);  // 30 (multiplication)
console.log(a / b);  // 3.333... (division)
console.log(a % b);  // 1  (modulus - remainder)
console.log(a ** b); // 1000 (exponentiation - 10³)

// Increment and Decrement
let x = 5;
x++;      // x = 6 (same as x = x + 1)
x--;      // x = 5 (same as x = x - 1)
++x;      // x = 6 (pre-increment)
```

### Assignment Operators

```javascript
let x = 10;

x += 5;  // x = x + 5  → 15
x -= 3;  // x = x - 3  → 12
x *= 2;  // x = x * 2  → 24
x /= 4;  // x = x / 4  → 6
x %= 4;  // x = x % 4  → 2
```

### Comparison Operators

```javascript
let a = 5;
let b = "5";

// Equal value
console.log(a == b);   // true (converts types)
console.log(a === b);  // false (strict - checks type too)

// Not equal
console.log(a != b);   // false
console.log(a !== b);  // true (strict)

// Greater/Less than
console.log(5 > 3);    // true
console.log(5 < 3);    // false
console.log(5 >= 5);   // true
console.log(5 <= 4);   // false
```

**Important**: Always use `===` and `!==` (strict comparison) instead of `==` and `!=`.

**Why?**
```javascript
// Weird behavior with ==
console.log(0 == false);      // true (wtf?)
console.log("" == false);     // true (wtf?)
console.log(null == undefined); // true (wtf?)

// Predictable with ===
console.log(0 === false);      // false ✅
console.log("" === false);     // false ✅
console.log(null === undefined); // false ✅
```

### Logical Operators

```javascript
let age = 20;
let hasID = true;

// AND - both must be true
console.log(age >= 18 && hasID);  // true

// OR - at least one must be true
console.log(age >= 21 || hasID);  // true

// NOT - reverses boolean
console.log(!hasID);              // false
console.log(!false);              // true
```

**Analogy**:
- **AND (&&)**: Like needing BOTH a key AND a password
- **OR (||)**: Like needing EITHER cash OR credit card
- **NOT (!)**: Like a light switch flip

### String Operators

```javascript
let firstName = "John";
let lastName = "Doe";

// Concatenation
let fullName = firstName + " " + lastName; // "John Doe"

// Template literals (better way)
let greeting = `Hello, ${firstName}!`; // "Hello, John!"

let age = 25;
let message = `I am ${age} years old`; // "I am 25 years old"

// Multi-line strings
let poem = `Roses are red
Violets are blue
JavaScript is awesome
And so are you!`;
```

### Ternary Operator (Shorthand if-else)

```javascript
// Syntax: condition ? valueIfTrue : valueIfFalse

let age = 20;
let status = age >= 18 ? "adult" : "minor";
console.log(status); // "adult"

// Without ternary (longer)
let status2;
if (age >= 18) {
    status2 = "adult";
} else {
    status2 = "minor";
}
```

---

## Control Flow

### If Statements

```javascript
let age = 18;

if (age >= 18) {
    console.log("You can vote!");
}

// if-else
let temperature = 25;

if (temperature > 30) {
    console.log("It's hot!");
} else {
    console.log("It's nice!");
}

// if-else if-else
let score = 85;

if (score >= 90) {
    console.log("Grade: A");
} else if (score >= 80) {
    console.log("Grade: B");
} else if (score >= 70) {
    console.log("Grade: C");
} else {
    console.log("Grade: F");
}
```

### Switch Statements

```javascript
let day = "Monday";

switch (day) {
    case "Monday":
        console.log("Start of work week");
        break;
    case "Friday":
        console.log("Almost weekend!");
        break;
    case "Saturday":
    case "Sunday":
        console.log("Weekend!");
        break;
    default:
        console.log("Midweek day");
}
```

**Note**: Don't forget `break` or it will "fall through" to the next case!

### For Loop

```javascript
// Basic for loop
for (let i = 0; i < 5; i++) {
    console.log(i); // 0, 1, 2, 3, 4
}

// Loop through array
let fruits = ["apple", "banana", "orange"];

for (let i = 0; i < fruits.length; i++) {
    console.log(fruits[i]);
}

// for...of (modern way for arrays)
for (let fruit of fruits) {
    console.log(fruit);
}

// for...in (for object properties)
let person = { name: "John", age: 30 };

for (let key in person) {
    console.log(key + ": " + person[key]);
}
// Output:
// name: John
// age: 30
```

### While Loop

```javascript
let count = 0;

while (count < 5) {
    console.log(count);
    count++;
}

// do...while (runs at least once)
let num = 10;

do {
    console.log(num);
    num++;
} while (num < 10); // Condition is false, but still runs once
```

### Break and Continue

```javascript
// break - exit loop
for (let i = 0; i < 10; i++) {
    if (i === 5) {
        break; // Stop loop when i is 5
    }
    console.log(i); // 0, 1, 2, 3, 4
}

// continue - skip iteration
for (let i = 0; i < 5; i++) {
    if (i === 2) {
        continue; // Skip when i is 2
    }
    console.log(i); // 0, 1, 3, 4 (skips 2)
}
```

---

## Functions

### What are Functions?

**Functions** are reusable blocks of code that perform a specific task.

**Analogy**: A function is like a recipe. You write it once, then use it many times.

### Function Declaration

```javascript
// Basic function
function greet() {
    console.log("Hello!");
}

greet(); // Call the function - Output: Hello!

// Function with parameters
function greetPerson(name) {
    console.log("Hello, " + name + "!");
}

greetPerson("John");  // Hello, John!
greetPerson("Jane");  // Hello, Jane!

// Function with return value
function add(a, b) {
    return a + b;
}

let result = add(5, 3);
console.log(result); // 8

// Multiple parameters
function calculateArea(width, height) {
    return width * height;
}

console.log(calculateArea(5, 10)); // 50
```

### Function Expression

```javascript
// Store function in a variable
const multiply = function(a, b) {
    return a * b;
};

console.log(multiply(4, 5)); // 20
```

### Arrow Functions (Modern)

```javascript
// Traditional function
function add(a, b) {
    return a + b;
}

// Arrow function (shorter)
const add2 = (a, b) => {
    return a + b;
};

// Even shorter (implicit return)
const add3 = (a, b) => a + b;

// Single parameter (no parentheses needed)
const double = x => x * 2;

// No parameters
const greet = () => console.log("Hello!");

console.log(add3(5, 3));  // 8
console.log(double(4));   // 8
greet();                  // Hello!
```

### Default Parameters

```javascript
function greet(name = "Guest") {
    console.log(`Hello, ${name}!`);
}

greet("John");  // Hello, John!
greet();        // Hello, Guest!

// Multiple defaults
function createUser(name = "Anonymous", age = 0) {
    return { name, age };
}

console.log(createUser());              // { name: "Anonymous", age: 0 }
console.log(createUser("John", 25));    // { name: "John", age: 25 }
```

### Rest Parameters

```javascript
// Collect all arguments into an array
function sum(...numbers) {
    let total = 0;
    for (let num of numbers) {
        total += num;
    }
    return total;
}

console.log(sum(1, 2, 3));        // 6
console.log(sum(1, 2, 3, 4, 5));  // 15

// Mix regular and rest parameters
function introduce(greeting, ...names) {
    return `${greeting}, ${names.join(" and ")}!`;
}

console.log(introduce("Hello", "John", "Jane", "Bob"));
// "Hello, John and Jane and Bob!"
```

### Callback Functions

```javascript
// Function passed as argument to another function
function doHomework(subject, callback) {
    console.log(`Starting ${subject} homework...`);
    callback();
}

doHomework("Math", function() {
    console.log("Finished homework!");
});

// With arrow function
doHomework("English", () => {
    console.log("All done!");
});
```

### Higher-Order Functions

```javascript
// Function that returns a function
function multiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = multiplier(2);
const triple = multiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

---

## Arrays

### Creating Arrays

```javascript
// Array literal (most common)
let fruits = ["apple", "banana", "orange"];
let numbers = [1, 2, 3, 4, 5];
let mixed = [1, "text", true, null, { name: "John" }];

// Array constructor
let arr = new Array(5); // Empty array with 5 slots
let arr2 = new Array(1, 2, 3); // [1, 2, 3]

// Empty array
let empty = [];
```

### Accessing Elements

```javascript
let fruits = ["apple", "banana", "orange"];

console.log(fruits[0]);   // apple (first element)
console.log(fruits[1]);   // banana
console.log(fruits[2]);   // orange
console.log(fruits[3]);   // undefined (doesn't exist)

// Last element
console.log(fruits[fruits.length - 1]); // orange

// Negative indexing (doesn't work in JS)
// console.log(fruits[-1]); // undefined ❌

// Use .at() method for negative indexing (modern)
console.log(fruits.at(-1));  // orange ✅
console.log(fruits.at(-2));  // banana
```

### Modifying Arrays

```javascript
let fruits = ["apple", "banana", "orange"];

// Change element
fruits[1] = "grape";
console.log(fruits); // ["apple", "grape", "orange"]

// Add to end
fruits.push("mango");
console.log(fruits); // ["apple", "grape", "orange", "mango"]

// Add to beginning
fruits.unshift("strawberry");
console.log(fruits); // ["strawberry", "apple", "grape", "orange", "mango"]

// Remove from end
let last = fruits.pop();
console.log(last);    // "mango"
console.log(fruits);  // ["strawberry", "apple", "grape", "orange"]

// Remove from beginning
let first = fruits.shift();
console.log(first);   // "strawberry"
console.log(fruits);  // ["apple", "grape", "orange"]
```

### Array Methods (Basic)

```javascript
let fruits = ["apple", "banana", "orange"];

// Length
console.log(fruits.length); // 3

// Check if array includes value
console.log(fruits.includes("banana")); // true
console.log(fruits.includes("grape"));  // false

// Find index
console.log(fruits.indexOf("orange"));  // 2
console.log(fruits.indexOf("grape"));   // -1 (not found)

// Join array into string
console.log(fruits.join(", "));  // "apple, banana, orange"
console.log(fruits.join(" - ")); // "apple - banana - orange"

// Reverse array
fruits.reverse();
console.log(fruits); // ["orange", "banana", "apple"]

// Sort array
let numbers = [3, 1, 4, 1, 5, 9];
numbers.sort();
console.log(numbers); // [1, 1, 3, 4, 5, 9]

// Slice (extract portion)
let fruits2 = ["apple", "banana", "orange", "mango", "grape"];
let citrus = fruits2.slice(1, 3); // From index 1 to 3 (not including 3)
console.log(citrus); // ["banana", "orange"]

// Splice (add/remove elements)
let fruits3 = ["apple", "banana", "orange"];
fruits3.splice(1, 1, "grape", "mango"); // At index 1, remove 1, add 2
console.log(fruits3); // ["apple", "grape", "mango", "orange"]
```

### Multidimensional Arrays

```javascript
// 2D array (array of arrays)
let matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

console.log(matrix[0]);     // [1, 2, 3]
console.log(matrix[1][1]);  // 5
console.log(matrix[2][0]);  // 7

// Practical example: tic-tac-toe board
let board = [
    ["X", "O", "X"],
    ["O", "X", "O"],
    ["X", "O", "X"]
];
```

---

## Objects

### What are Objects?

**Objects** are collections of related data and functions (called properties and methods).

**Analogy**: An object is like a real-world object (e.g., a car). A car has properties (color, brand, model) and methods (start, stop, accelerate).

### Creating Objects

```javascript
// Object literal (most common)
let person = {
    name: "John",
    age: 30,
    isStudent: false
};

// Empty object
let emptyObj = {};

// Object constructor
let person2 = new Object();
person2.name = "Jane";
person2.age = 25;
```

### Accessing Properties

```javascript
let person = {
    name: "John",
    age: 30,
    email: "john@example.com"
};

// Dot notation (preferred)
console.log(person.name);   // John
console.log(person.age);    // 30

// Bracket notation (when key has spaces or special chars)
console.log(person["email"]); // john@example.com

let key = "name";
console.log(person[key]);   // John (dynamic access)

// Access non-existent property
console.log(person.address); // undefined
```

### Modifying Objects

```javascript
let person = {
    name: "John",
    age: 30
};

// Change property
person.age = 31;

// Add property
person.email = "john@example.com";

// Delete property
delete person.age;

console.log(person); // { name: "John", email: "john@example.com" }
```

### Methods (Functions in Objects)

```javascript
let person = {
    name: "John",
    age: 30,

    // Method (old way)
    greet: function() {
        console.log("Hello, my name is " + this.name);
    },

    // Method (modern shorthand)
    celebrate() {
        this.age++;
        console.log("Happy birthday! I'm now " + this.age);
    }
};

person.greet();      // Hello, my name is John
person.celebrate();  // Happy birthday! I'm now 31
```

### Nested Objects

```javascript
let user = {
    name: "John",
    age: 30,
    address: {
        street: "123 Main St",
        city: "New York",
        country: "USA"
    },
    hobbies: ["reading", "gaming", "coding"]
};

console.log(user.address.city);     // New York
console.log(user.hobbies[0]);       // reading
```

### Object Destructuring

```javascript
let person = {
    name: "John",
    age: 30,
    email: "john@example.com"
};

// Extract properties
const { name, age } = person;
console.log(name); // John
console.log(age);  // 30

// Rename variables
const { name: personName, age: personAge } = person;
console.log(personName); // John

// Default values
const { name, address = "Unknown" } = person;
console.log(address); // Unknown
```

### Object Methods

```javascript
let person = {
    name: "John",
    age: 30,
    email: "john@example.com"
};

// Get all keys
console.log(Object.keys(person));
// ["name", "age", "email"]

// Get all values
console.log(Object.values(person));
// ["John", 30, "john@example.com"]

// Get key-value pairs
console.log(Object.entries(person));
// [["name", "John"], ["age", 30], ["email", "john@example.com"]]

// Check if property exists
console.log("name" in person);        // true
console.log("address" in person);     // false
console.log(person.hasOwnProperty("age")); // true

// Merge objects
let obj1 = { a: 1, b: 2 };
let obj2 = { c: 3, d: 4 };
let merged = Object.assign({}, obj1, obj2);
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// Modern way with spread
let merged2 = { ...obj1, ...obj2 };
console.log(merged2); // { a: 1, b: 2, c: 3, d: 4 }

// Freeze object (make immutable)
let frozen = Object.freeze({ name: "John" });
frozen.name = "Jane"; // Silently fails (or error in strict mode)
console.log(frozen.name); // John
```

---

## Strings and Template Literals

### String Basics

```javascript
let single = 'Hello';
let double = "World";
let template = `Hello, World!`;

// They're all the same type
console.log(typeof single); // string
```

### String Properties and Methods

```javascript
let text = "Hello, World!";

// Length
console.log(text.length); // 13

// Access characters
console.log(text[0]);       // H
console.log(text.charAt(0)); // H
console.log(text.at(-1));   // ! (last character)

// Case conversion
console.log(text.toLowerCase()); // hello, world!
console.log(text.toUpperCase()); // HELLO, WORLD!

// Search
console.log(text.includes("World"));  // true
console.log(text.startsWith("Hello")); // true
console.log(text.endsWith("!"));      // true
console.log(text.indexOf("o"));       // 4 (first occurrence)
console.log(text.lastIndexOf("o"));   // 8 (last occurrence)

// Extract
console.log(text.slice(0, 5));      // Hello
console.log(text.slice(7));         // World!
console.log(text.substring(0, 5));  // Hello
console.log(text.substr(7, 5));     // World (deprecated)

// Replace
console.log(text.replace("World", "JavaScript"));
// Hello, JavaScript!

// Replace all (modern)
let text2 = "cat cat cat";
console.log(text2.replaceAll("cat", "dog"));
// dog dog dog

// Split into array
let sentence = "The quick brown fox";
let words = sentence.split(" ");
console.log(words); // ["The", "quick", "brown", "fox"]

// Trim whitespace
let messy = "  hello  ";
console.log(messy.trim());      // "hello"
console.log(messy.trimStart()); // "hello  "
console.log(messy.trimEnd());   // "  hello"

// Repeat
console.log("ha".repeat(3)); // hahaha

// Pad
console.log("5".padStart(3, "0"));  // 005
console.log("5".padEnd(3, "0"));    // 500
```

### Template Literals

```javascript
let name = "John";
let age = 30;

// Old way (concatenation)
let message1 = "My name is " + name + " and I'm " + age + " years old.";

// Modern way (template literal)
let message2 = `My name is ${name} and I'm ${age} years old.`;

// Expressions inside ${}
let a = 5;
let b = 10;
console.log(`Sum: ${a + b}`); // Sum: 15

// Multi-line strings
let html = `
    <div>
        <h1>${name}</h1>
        <p>Age: ${age}</p>
    </div>
`;

// Function calls
function upper(str) {
    return str.toUpperCase();
}

console.log(`Hello, ${upper(name)}!`); // Hello, JOHN!
```

### Tagged Templates (Advanced)

```javascript
function highlight(strings, ...values) {
    return strings.reduce((result, str, i) => {
        return result + str + (values[i] ? `<mark>${values[i]}</mark>` : '');
    }, '');
}

let name = "John";
let age = 30;

let result = highlight`My name is ${name} and I'm ${age} years old.`;
console.log(result);
// My name is <mark>John</mark> and I'm <mark>30</mark> years old.
```

---

## How JavaScript Works

### JavaScript Runtime Environment

Before diving into code, it's crucial to understand **how JavaScript actually runs**.

**Analogy**: Think of JavaScript runtime like a restaurant kitchen:
- **Call Stack**: The chef's task list (one task at a time)
- **Web APIs**: Other kitchen staff doing tasks in parallel (timers, fetching ingredients)
- **Task Queue**: Completed tasks waiting to be served
- **Event Loop**: The manager who checks if the chef is free and assigns new tasks

### JavaScript is Single-Threaded

**Single-threaded** means JavaScript can only execute one piece of code at a time.

```javascript
console.log("First");
console.log("Second");
console.log("Third");

// Output (in order):
// First
// Second
// Third
```

**Why?** JavaScript has only **one call stack**, so it processes one function call at a time.

**Analogy**: Like having one checkout counter at a grocery store - customers must wait in line.

### Synchronous vs Asynchronous

#### Synchronous (Blocking)

Code executes line by line, waiting for each line to complete before moving to the next.

```javascript
console.log("Start");

// This blocks for 3 seconds
let startTime = Date.now();
while (Date.now() - startTime < 3000) {
    // Do nothing, just wait
}

console.log("End");

// Output:
// Start
// (3 second pause)
// End
```

**Problem**: The browser freezes during that 3-second wait!

#### Asynchronous (Non-Blocking)

Code can start a task and move on without waiting for it to complete.

```javascript
console.log("Start");

setTimeout(() => {
    console.log("Inside timeout");
}, 3000);

console.log("End");

// Output:
// Start
// End
// (3 seconds later)
// Inside timeout
```

**Magic!** The code didn't wait for setTimeout to finish!

---

## The Event Loop

### What is the Event Loop?

The **event loop** is the mechanism that allows JavaScript to perform asynchronous operations despite being single-threaded.

**Analogy**: Imagine a waiter (event loop) constantly checking:
1. Is the chef (call stack) free?
2. Are there any completed orders (callback queue)?
3. If yes to both, give the chef the next order

### Visual Model

```
┌─────────────────────────────────────────────┐
│                Call Stack                    │
│  (Where JavaScript executes code)            │
│                                              │
│  [Empty] or [function calls stacked]         │
└─────────────────────────────────────────────┘
                    ↑
                    │
    ┌───────────────┴───────────────┐
    │       Event Loop              │
    │  (Monitors stack & queue)     │
    └───────────────┬───────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│           Callback Queue                     │
│  (Tasks waiting to be executed)              │
│                                              │
│  [callback1] [callback2] [callback3]         │
└─────────────────────────────────────────────┘
                    ↑
                    │
┌─────────────────────────────────────────────┐
│            Web APIs                          │
│  (Browser-provided functionality)            │
│                                              │
│  • setTimeout / setInterval                  │
│  • fetch / XMLHttpRequest                    │
│  • DOM events                                │
│  • console.log                               │
└─────────────────────────────────────────────┘
```

### How It Works: Step by Step

```javascript
console.log("1");

setTimeout(() => {
    console.log("2");
}, 0);

console.log("3");

// Output:
// 1
// 3
// 2  (even though timeout is 0ms!)
```

**Execution breakdown**:

1. **Call Stack**: `console.log("1")` → executes → prints "1"
2. **Call Stack**: `setTimeout(...)` → registers callback with Web API → removed from stack
3. **Web APIs**: Timer starts (0ms)
4. **Call Stack**: `console.log("3")` → executes → prints "3"
5. **Web APIs**: Timer completes → sends callback to Callback Queue
6. **Event Loop**: Checks if call stack is empty → YES → moves callback from queue to stack
7. **Call Stack**: `console.log("2")` → executes → prints "2"

**Key insight**: Even with 0ms delay, `setTimeout` callback goes through the event loop and executes AFTER synchronous code!

### The Event Loop Algorithm

```
while (true) {
    if (callStack.isEmpty()) {
        if (callbackQueue.hasItems()) {
            let callback = callbackQueue.dequeue();
            callStack.push(callback);
        }
    }
}
```

The event loop continuously checks:
1. Is the call stack empty?
2. Are there callbacks waiting in the queue?
3. If both are true, move a callback to the stack

---

## Call Stack, Web APIs, and Task Queue

### The Call Stack

The **call stack** tracks function calls. It's a LIFO (Last In, First Out) data structure.

```javascript
function first() {
    console.log("First function");
    second();
    console.log("First function again");
}

function second() {
    console.log("Second function");
    third();
}

function third() {
    console.log("Third function");
}

first();

// Call Stack visualization:
//
// third()          ← top
// second()
// first()
// main()           ← bottom
```

**Execution order**:
1. `first()` is pushed to stack
2. Executes `console.log("First function")`
3. `second()` is pushed to stack
4. Executes `console.log("Second function")`
5. `third()` is pushed to stack
6. Executes `console.log("Third function")`
7. `third()` completes, popped from stack
8. `second()` completes, popped from stack
9. Back to `first()`, executes `console.log("First function again")`
10. `first()` completes, popped from stack

**Output**:
```
First function
Second function
Third function
First function again
```

### Stack Overflow

```javascript
function recurse() {
    recurse(); // Calls itself forever
}

recurse();
// Error: Maximum call stack size exceeded
```

**Why?** The call stack has a limit (~10,000 calls). Infinite recursion fills it up.

**Analogy**: Like stacking plates forever - eventually, they'll topple over!

### Web APIs

**Web APIs** are browser-provided features that run **outside** the JavaScript engine.

Common Web APIs:
- `setTimeout` / `setInterval`
- `fetch` / `XMLHttpRequest`
- DOM events (`click`, `keypress`, etc.)
- `localStorage`
- `console.log` (yes, even this!)

```javascript
console.log("Start");

// This goes to Web API
setTimeout(() => {
    console.log("Timeout");
}, 2000);

// This also goes to Web API
fetch("https://api.example.com/data")
    .then(data => console.log(data));

console.log("End");

// While setTimeout and fetch are "waiting",
// JavaScript continues executing other code!
```

### Task Queue (Callback Queue)

When Web API operations complete, their callbacks are placed in the **task queue** (also called callback queue or macrotask queue).

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0);
setTimeout(() => console.log("3"), 0);

console.log("4");

// Output: 1, 4, 2, 3
```

**Why this order?**
1. `console.log("1")` executes immediately
2. Both `setTimeout` callbacks go to Web API → then to queue
3. `console.log("4")` executes immediately
4. Event loop moves callbacks from queue to stack (FIFO - First In, First Out)
5. First `setTimeout` callback executes → prints "2"
6. Second `setTimeout` callback executes → prints "3"

### Microtask Queue

There's actually **another queue** with **higher priority**: the **microtask queue**.

**Microtasks** include:
- Promise callbacks (`.then`, `.catch`, `.finally`)
- `async`/`await`
- `queueMicrotask()`
- `MutationObserver`

**Macrotasks** (regular task queue) include:
- `setTimeout`
- `setInterval`
- `setImmediate` (Node.js)
- I/O operations

**Priority**: Microtasks execute **before** macrotasks!

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0); // Macrotask

Promise.resolve().then(() => console.log("3")); // Microtask

console.log("4");

// Output: 1, 4, 3, 2
```

**Execution order**:
1. Synchronous code: prints "1", then "4"
2. Call stack empty → check microtask queue
3. Promise callback (microtask) executes → prints "3"
4. Microtask queue empty → check macrotask queue
5. setTimeout callback (macrotask) executes → prints "2"

### Complete Example

```javascript
console.log("Start");

setTimeout(() => {
    console.log("Timeout 1");
}, 0);

Promise.resolve()
    .then(() => {
        console.log("Promise 1");
    })
    .then(() => {
        console.log("Promise 2");
    });

setTimeout(() => {
    console.log("Timeout 2");
}, 0);

console.log("End");

// Output:
// Start
// End
// Promise 1
// Promise 2
// Timeout 1
// Timeout 2
```

**Breakdown**:

| Step | Action | Call Stack | Microtask Queue | Macrotask Queue | Output |
|------|--------|-----------|-----------------|-----------------|--------|
| 1 | Execute | `console.log("Start")` | [] | [] | "Start" |
| 2 | Register | - | [] | [timeout1] | - |
| 3 | Register | - | [promise1] | [timeout1] | - |
| 4 | Register | - | [promise1] | [timeout1, timeout2] | - |
| 5 | Execute | `console.log("End")` | [promise1] | [timeout1, timeout2] | "End" |
| 6 | Execute microtask | `console.log("Promise 1")` | [promise2] | [timeout1, timeout2] | "Promise 1" |
| 7 | Execute microtask | `console.log("Promise 2")` | [] | [timeout1, timeout2] | "Promise 2" |
| 8 | Execute macrotask | `console.log("Timeout 1")` | [] | [timeout2] | "Timeout 1" |
| 9 | Execute macrotask | `console.log("Timeout 2")` | [] | [] | "Timeout 2" |

### Visualizing Async Code Flow

```javascript
function simulateAsync() {
    console.log("1: Synchronous start");

    setTimeout(() => {
        console.log("2: Macrotask (setTimeout)");
    }, 0);

    Promise.resolve().then(() => {
        console.log("3: Microtask (Promise)");
    });

    queueMicrotask(() => {
        console.log("4: Microtask (queueMicrotask)");
    });

    console.log("5: Synchronous end");
}

simulateAsync();

// Output:
// 1: Synchronous start
// 5: Synchronous end
// 3: Microtask (Promise)
// 4: Microtask (queueMicrotask)
// 2: Macrotask (setTimeout)
```

**Order of execution**:
1. **Synchronous code** (1, 5)
2. **All microtasks** (3, 4) - in order they were queued
3. **Macrotasks** (2) - one at a time

### Real-World Example: Button Click

```javascript
const button = document.getElementById("myButton");

button.addEventListener("click", () => {
    console.log("Click handler start");

    setTimeout(() => {
        console.log("Timeout in click handler");
    }, 0);

    Promise.resolve().then(() => {
        console.log("Promise in click handler");
    });

    console.log("Click handler end");
});

// When user clicks:
// Click handler start
// Click handler end
// Promise in click handler
// Timeout in click handler
```

### Why This Matters

Understanding the event loop helps you:

1. **Debug asynchronous code** - Know why things happen in unexpected order
2. **Avoid blocking the UI** - Keep the call stack clear for user interactions
3. **Optimize performance** - Use microtasks for high-priority work
4. **Prevent race conditions** - Understand timing of async operations

### Common Pitfalls

#### Pitfall 1: Assuming setTimeout is Precise

```javascript
console.log("Start");

setTimeout(() => {
    console.log("Should be 100ms later");
}, 100);

// Some heavy computation that takes 200ms
for (let i = 0; i < 1000000000; i++) {}

console.log("End");

// The timeout callback will NOT run at exactly 100ms!
// It can only run after the call stack is empty.
```

**setTimeout guarantees minimum delay, not exact timing!**

#### Pitfall 2: Infinite Microtask Loop

```javascript
// ❌ This will freeze the browser!
function infiniteMicrotasks() {
    Promise.resolve().then(() => {
        console.log("Microtask");
        infiniteMicrotasks(); // Queues another microtask
    });
}

infiniteMicrotasks();

// Microtasks keep executing, macrotasks never get a chance!
// Browser UI freezes because rendering is a macrotask.
```

#### Pitfall 3: Order of Operations

```javascript
async function example() {
    console.log("1");

    await Promise.resolve();

    console.log("2");
}

console.log("3");
example();
console.log("4");

// Output: 3, 1, 4, 2
// Many beginners expect: 3, 1, 2, 4
```

### Interactive Visualization Tool

To really understand the event loop, try this excellent visualization tool:
- [Loupe - Event Loop Visualizer](http://latentflip.com/loupe/)

You can paste code and see the call stack, Web APIs, and queues in action!

### Summary

**Key Concepts**:

1. **Call Stack**: Executes functions one at a time (LIFO)
2. **Web APIs**: Handle async operations outside JavaScript engine
3. **Microtask Queue**: High-priority tasks (Promises, async/await)
4. **Macrotask Queue**: Regular async tasks (setTimeout, events)
5. **Event Loop**: Coordinates everything

**Execution Order**:
1. Execute all **synchronous code**
2. Execute all **microtasks**
3. Execute **one macrotask**
4. Repeat from step 2

**Mental Model**:
```
while (true) {
    // 1. Execute synchronous code until call stack is empty

    // 2. Execute ALL microtasks
    while (microtaskQueue.hasItems()) {
        let microtask = microtaskQueue.dequeue();
        execute(microtask);
    }

    // 3. Execute ONE macrotask
    if (macrotaskQueue.hasItems()) {
        let macrotask = macrotaskQueue.dequeue();
        execute(macrotask);
    }

    // 4. Render if needed (browser only)
    if (needsRender) {
        render();
    }
}
```

---

## Scope and Hoisting

### What is Scope?

**Scope** determines where variables are accessible in your code.

**Analogy**: Think of scope like privacy levels:
- **Global**: Public announcement (everyone can hear)
- **Function**: Private conversation in a room
- **Block**: Whispering to person next to you

### Global Scope

```javascript
// Global variable - accessible everywhere
let globalVar = "I'm global!";

function test() {
    console.log(globalVar); // ✅ Can access
}

test();
console.log(globalVar); // ✅ Can access
```

### Function Scope

```javascript
function myFunction() {
    let functionVar = "I'm local to this function";
    console.log(functionVar); // ✅ Works
}

myFunction();
// console.log(functionVar); // ❌ Error! Not accessible outside
```

### Block Scope

```javascript
// let and const are block-scoped
if (true) {
    let blockVar = "I'm in a block";
    console.log(blockVar); // ✅ Works
}

// console.log(blockVar); // ❌ Error!

// var is NOT block-scoped (another reason to avoid it)
if (true) {
    var leakyVar = "I leak out!";
}
console.log(leakyVar); // ✅ Works (but shouldn't!)
```

### Scope Chain

```javascript
let global = "Global";

function outer() {
    let outerVar = "Outer";

    function inner() {
        let innerVar = "Inner";

        // Inner can access all three
        console.log(innerVar);  // ✅ Inner
        console.log(outerVar);  // ✅ Outer
        console.log(global);    // ✅ Global
    }

    inner();
    // console.log(innerVar); // ❌ Can't access inner's variables
}

outer();
```

### Hoisting

**Hoisting** is JavaScript's behavior of moving declarations to the top of their scope.

```javascript
// What you write:
console.log(x); // undefined (not an error!)
var x = 5;

// What JavaScript does:
var x;              // Declaration hoisted
console.log(x);     // undefined
x = 5;              // Assignment stays in place

// With let/const (not hoisted the same way)
// console.log(y); // ❌ Error! Cannot access before initialization
let y = 5;

// Function hoisting
sayHi(); // ✅ Works! Function declarations are fully hoisted

function sayHi() {
    console.log("Hi!");
}

// Function expressions are NOT hoisted
// greet(); // ❌ Error!
const greet = function() {
    console.log("Hello!");
};
```

**Best Practice**: Always declare variables at the top of their scope to avoid confusion.

---

## Closures

### What are Closures?

A **closure** is a function that has access to variables from its outer (enclosing) scope, even after the outer function has finished executing.

**Analogy**: A closure is like a backpack. When a function is created, it packs up all the variables it needs from its surrounding environment in a backpack and carries them along wherever it goes.

### Basic Closure Example

```javascript
function outer() {
    let count = 0; // This variable is "closed over"

    function inner() {
        count++; // inner() has access to count
        console.log(count);
    }

    return inner;
}

let counter = outer();
counter(); // 1
counter(); // 2
counter(); // 3

// count is still accessible even though outer() finished!
```

### Practical Use: Private Variables

```javascript
function createBankAccount(initialBalance) {
    let balance = initialBalance; // Private variable

    return {
        deposit(amount) {
            balance += amount;
            console.log(`Deposited $${amount}. New balance: $${balance}`);
        },

        withdraw(amount) {
            if (amount > balance) {
                console.log("Insufficient funds!");
            } else {
                balance -= amount;
                console.log(`Withdrew $${amount}. New balance: $${balance}`);
            }
        },

        getBalance() {
            return balance;
        }
    };
}

let account = createBankAccount(100);
account.deposit(50);    // Deposited $50. New balance: $150
account.withdraw(30);   // Withdrew $30. New balance: $120
console.log(account.getBalance()); // 120

// Can't directly access balance
console.log(account.balance); // undefined (it's private!)
```

### Closure in Loops (Common Gotcha)

```javascript
// ❌ Problem with var
for (var i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // 3, 3, 3 (not what we want!)
    }, 1000);
}

// ✅ Solution 1: Use let (block-scoped)
for (let i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // 0, 1, 2 (correct!)
    }, 1000);
}

// ✅ Solution 2: Create closure with function
for (var i = 0; i < 3; i++) {
    (function(index) {
        setTimeout(function() {
            console.log(index); // 0, 1, 2
        }, 1000);
    })(i);
}
```

### Closure Examples

```javascript
// Example 1: Function factory
function multiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = multiplier(2);
const triple = multiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15

// Example 2: Once function (execute only once)
function once(fn) {
    let called = false;
    let result;

    return function(...args) {
        if (!called) {
            called = true;
            result = fn(...args);
        }
        return result;
    };
}

const initialize = once(() => {
    console.log("Initialized!");
    return "Done";
});

initialize(); // Initialized!
initialize(); // (nothing happens)
initialize(); // (nothing happens)
```

---

## The 'this' Keyword

### What is 'this'?

**this** refers to the object that is executing the current function.

**Analogy**: "this" is like the word "me" - it refers to whoever is speaking (or in code, whatever object is calling the function).

### Global Context

```javascript
console.log(this); // Window object (in browser) or global (in Node.js)
```

### Object Method

```javascript
let person = {
    name: "John",
    age: 30,

    greet() {
        console.log(`Hi, I'm ${this.name}`);
        // 'this' refers to 'person' object
    }
};

person.greet(); // Hi, I'm John
```

### Regular Function

```javascript
function showThis() {
    console.log(this);
}

showThis(); // Window (in non-strict mode) or undefined (in strict mode)
```

### Arrow Functions and 'this'

**Important**: Arrow functions don't have their own `this`. They inherit `this` from their surrounding scope.

```javascript
let person = {
    name: "John",

    // Regular function
    greet1: function() {
        console.log(this.name); // ✅ John
    },

    // Arrow function
    greet2: () => {
        console.log(this.name); // ❌ undefined (this is NOT person)
    },

    // This is where arrow functions shine
    delayedGreet: function() {
        setTimeout(() => {
            // Arrow function inherits 'this' from delayedGreet
            console.log(`Hi, I'm ${this.name}`); // ✅ John
        }, 1000);
    }
};

person.greet1();  // John
person.greet2();  // undefined
person.delayedGreet(); // Hi, I'm John (after 1 second)
```

### Constructor Functions

```javascript
function Person(name, age) {
    this.name = name;
    this.age = age;

    this.greet = function() {
        console.log(`Hi, I'm ${this.name}`);
    };
}

let john = new Person("John", 30);
john.greet(); // Hi, I'm John
// 'this' refers to the new object being created
```

### Explicit Binding

```javascript
function greet() {
    console.log(`Hi, I'm ${this.name}`);
}

let person1 = { name: "John" };
let person2 = { name: "Jane" };

// call() - invoke function with specific 'this'
greet.call(person1); // Hi, I'm John
greet.call(person2); // Hi, I'm Jane

// apply() - same but arguments as array
function introduce(greeting, age) {
    console.log(`${greeting}, I'm ${this.name}, ${age} years old`);
}

introduce.apply(person1, ["Hello", 30]);
// Hello, I'm John, 30 years old

// bind() - create new function with bound 'this'
let greetJohn = greet.bind(person1);
greetJohn(); // Hi, I'm John (always uses person1)
```

### Event Handlers

```javascript
// In browser
document.getElementById("myButton").addEventListener("click", function() {
    console.log(this); // The button element
});

// With arrow function
document.getElementById("myButton").addEventListener("click", () => {
    console.log(this); // Window object (not the button!)
});
```

---

## Arrow Functions

### Syntax Variations

```javascript
// Traditional function
function add(a, b) {
    return a + b;
}

// Arrow function (full syntax)
const add1 = (a, b) => {
    return a + b;
};

// Implicit return (one expression)
const add2 = (a, b) => a + b;

// Single parameter (no parentheses needed)
const double = x => x * 2;

// No parameters
const greet = () => console.log("Hello!");

// Returning object literal (need parentheses)
const makePerson = (name, age) => ({ name, age });

console.log(makePerson("John", 30));
// { name: "John", age: 30 }
```

### When to Use Arrow Functions

✅ **Use arrow functions for**:
- Array methods (map, filter, reduce)
- Callbacks
- When you want to preserve outer `this`

```javascript
// Array methods
let numbers = [1, 2, 3, 4, 5];

let doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

let evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4]

// Preserving 'this'
class Counter {
    constructor() {
        this.count = 0;
    }

    start() {
        setInterval(() => {
            this.count++; // 'this' refers to Counter instance
            console.log(this.count);
        }, 1000);
    }
}
```

❌ **Don't use arrow functions for**:
- Object methods (if you need `this`)
- Prototype methods
- Constructors
- Event handlers (if you need element reference)

```javascript
// ❌ Bad: Object method
let person = {
    name: "John",
    greet: () => {
        console.log(this.name); // undefined!
    }
};

// ✅ Good: Regular function
let person2 = {
    name: "John",
    greet() {
        console.log(this.name); // John
    }
};
```

---

## Destructuring

### Array Destructuring

```javascript
// Basic
let fruits = ["apple", "banana", "orange"];
let [first, second, third] = fruits;

console.log(first);  // apple
console.log(second); // banana
console.log(third);  // orange

// Skip elements
let [a, , c] = fruits;
console.log(a); // apple
console.log(c); // orange

// Rest elements
let numbers = [1, 2, 3, 4, 5];
let [one, two, ...rest] = numbers;

console.log(one);  // 1
console.log(two);  // 2
console.log(rest); // [3, 4, 5]

// Default values
let [x, y, z = 0] = [1, 2];
console.log(z); // 0

// Swapping variables
let a1 = 1, b1 = 2;
[a1, b1] = [b1, a1];
console.log(a1, b1); // 2, 1
```

### Object Destructuring

```javascript
let person = {
    name: "John",
    age: 30,
    email: "john@example.com"
};

// Basic
let { name, age } = person;
console.log(name); // John
console.log(age);  // 30

// Rename variables
let { name: personName, age: personAge } = person;
console.log(personName); // John

// Default values
let { name, address = "Unknown" } = person;
console.log(address); // Unknown

// Nested destructuring
let user = {
    id: 1,
    name: "John",
    address: {
        city: "New York",
        country: "USA"
    }
};

let { address: { city, country } } = user;
console.log(city);    // New York
console.log(country); // USA

// Rest properties
let { name: n, ...details } = person;
console.log(n);       // John
console.log(details); // { age: 30, email: "john@example.com" }
```

### Function Parameters

```javascript
// Destructure object parameter
function greet({ name, age }) {
    console.log(`Hi, I'm ${name}, ${age} years old`);
}

greet({ name: "John", age: 30 });
// Hi, I'm John, 30 years old

// With defaults
function createUser({ name = "Anonymous", age = 0 } = {}) {
    return { name, age };
}

console.log(createUser({}));              // { name: "Anonymous", age: 0 }
console.log(createUser({ name: "John" })); // { name: "John", age: 0 }
```

---

## Spread and Rest Operators

Both use `...` but in different contexts.

### Spread Operator

**Spread** expands an array or object into individual elements.

**Analogy**: Like unpacking a suitcase and laying out all items.

```javascript
// Spread in arrays
let fruits = ["apple", "banana"];
let moreFruits = ["orange", "mango"];

// Combine arrays
let allFruits = [...fruits, ...moreFruits];
console.log(allFruits);
// ["apple", "banana", "orange", "mango"]

// Copy array
let fruitsCopy = [...fruits];
fruitsCopy.push("grape");
console.log(fruits);      // ["apple", "banana"] (unchanged)
console.log(fruitsCopy);  // ["apple", "banana", "grape"]

// Spread in function calls
let numbers = [1, 2, 3, 4, 5];
console.log(Math.max(...numbers)); // 5
// Same as: Math.max(1, 2, 3, 4, 5)

// Spread in objects
let person = { name: "John", age: 30 };
let employee = { ...person, job: "Developer" };

console.log(employee);
// { name: "John", age: 30, job: "Developer" }

// Merge objects
let obj1 = { a: 1, b: 2 };
let obj2 = { c: 3, d: 4 };
let merged = { ...obj1, ...obj2 };
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// Override properties
let defaults = { theme: "light", fontSize: 12 };
let userSettings = { fontSize: 16 };
let settings = { ...defaults, ...userSettings };
console.log(settings);
// { theme: "light", fontSize: 16 }
```

### Rest Operator

**Rest** collects multiple elements into an array.

**Analogy**: Like packing leftover items into a bag.

```javascript
// Rest in function parameters
function sum(...numbers) {
    return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2, 3));       // 6
console.log(sum(1, 2, 3, 4, 5)); // 15

// Rest with other parameters
function introduce(greeting, ...names) {
    return `${greeting}, ${names.join(" and ")}!`;
}

console.log(introduce("Hello", "John", "Jane", "Bob"));
// Hello, John and Jane and Bob!

// Rest in destructuring
let [first, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(rest);  // [2, 3, 4, 5]

let { a, ...others } = { a: 1, b: 2, c: 3 };
console.log(a);      // 1
console.log(others); // { b: 2, c: 3 }
```

---

## Array Methods

### Iteration Methods

#### forEach()

```javascript
let fruits = ["apple", "banana", "orange"];

fruits.forEach((fruit, index) => {
    console.log(`${index}: ${fruit}`);
});
// 0: apple
// 1: banana
// 2: orange

// Cannot break or return from forEach
```

#### map()

**Creates a new array** by transforming each element.

```javascript
let numbers = [1, 2, 3, 4, 5];

let doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

let names = ["john", "jane", "bob"];
let capitalized = names.map(name =>
    name.charAt(0).toUpperCase() + name.slice(1)
);
console.log(capitalized); // ["John", "Jane", "Bob"]
```

#### filter()

**Creates a new array** with elements that pass a test.

```javascript
let numbers = [1, 2, 3, 4, 5, 6];

let evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4, 6]

let people = [
    { name: "John", age: 17 },
    { name: "Jane", age: 22 },
    { name: "Bob", age: 19 }
];

let adults = people.filter(person => person.age >= 18);
console.log(adults);
// [{ name: "Jane", age: 22 }, { name: "Bob", age: 19 }]
```

#### reduce()

**Reduces array to a single value**.

```javascript
let numbers = [1, 2, 3, 4, 5];

// Sum
let sum = numbers.reduce((total, n) => total + n, 0);
console.log(sum); // 15

// Find max
let max = numbers.reduce((max, n) => n > max ? n : max, numbers[0]);
console.log(max); // 5

// Count occurrences
let fruits = ["apple", "banana", "apple", "orange", "banana", "apple"];
let count = fruits.reduce((acc, fruit) => {
    acc[fruit] = (acc[fruit] || 0) + 1;
    return acc;
}, {});
console.log(count);
// { apple: 3, banana: 2, orange: 1 }

// Flatten array
let nested = [[1, 2], [3, 4], [5, 6]];
let flat = nested.reduce((acc, arr) => acc.concat(arr), []);
console.log(flat); // [1, 2, 3, 4, 5, 6]
```

#### find()

**Returns first element** that passes test.

```javascript
let numbers = [1, 3, 5, 8, 10, 12];

let firstEven = numbers.find(n => n % 2 === 0);
console.log(firstEven); // 8

let people = [
    { name: "John", age: 17 },
    { name: "Jane", age: 22 }
];

let adult = people.find(person => person.age >= 18);
console.log(adult); // { name: "Jane", age: 22 }
```

#### findIndex()

**Returns index** of first element that passes test.

```javascript
let numbers = [1, 3, 5, 8, 10];

let index = numbers.findIndex(n => n > 5);
console.log(index); // 3 (the number 8)
```

#### some()

**Returns true** if at least one element passes test.

```javascript
let numbers = [1, 3, 5, 8];

let hasEven = numbers.some(n => n % 2 === 0);
console.log(hasEven); // true

let allEven = numbers.every(n => n % 2 === 0);
console.log(allEven); // false
```

#### every()

**Returns true** if all elements pass test.

```javascript
let numbers = [2, 4, 6, 8];

let allEven = numbers.every(n => n % 2 === 0);
console.log(allEven); // true
```

### Chaining Methods

```javascript
let people = [
    { name: "John", age: 17, score: 85 },
    { name: "Jane", age: 22, score: 92 },
    { name: "Bob", age: 19, score: 78 },
    { name: "Alice", age: 25, score: 95 }
];

// Get names of adults with score >= 90
let result = people
    .filter(p => p.age >= 18)
    .filter(p => p.score >= 90)
    .map(p => p.name);

console.log(result); // ["Jane", "Alice"]

// Calculate average score of adults
let avgScore = people
    .filter(p => p.age >= 18)
    .map(p => p.score)
    .reduce((sum, score, _, arr) => sum + score / arr.length, 0);

console.log(avgScore); // 88.33...
```

---

## Object Methods

### Object.keys(), values(), entries()

```javascript
let person = {
    name: "John",
    age: 30,
    city: "New York"
};

// Get all keys
let keys = Object.keys(person);
console.log(keys); // ["name", "age", "city"]

// Get all values
let values = Object.values(person);
console.log(values); // ["John", 30, "New York"]

// Get key-value pairs
let entries = Object.entries(person);
console.log(entries);
// [["name", "John"], ["age", 30], ["city", "New York"]]

// Use with forEach
Object.entries(person).forEach(([key, value]) => {
    console.log(`${key}: ${value}`);
});
```

### Object.assign()

```javascript
// Copy object
let original = { a: 1, b: 2 };
let copy = Object.assign({}, original);

// Merge objects
let obj1 = { a: 1, b: 2 };
let obj2 = { c: 3, d: 4 };
let merged = Object.assign({}, obj1, obj2);
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// Override properties
let defaults = { theme: "light", size: "medium" };
let custom = { size: "large" };
let result = Object.assign({}, defaults, custom);
console.log(result); // { theme: "light", size: "large" }
```

### Object.freeze() and seal()

```javascript
// freeze - completely immutable
let frozen = Object.freeze({ name: "John" });
frozen.name = "Jane";  // Silently fails
frozen.age = 30;       // Can't add
delete frozen.name;    // Can't delete
console.log(frozen);   // { name: "John" }

// seal - can modify but can't add/delete
let sealed = Object.seal({ name: "John" });
sealed.name = "Jane";  // ✅ Works
sealed.age = 30;       // ❌ Can't add
delete sealed.name;    // ❌ Can't delete
console.log(sealed);   // { name: "Jane" }
```

### Object.create()

```javascript
// Create object with specific prototype
let personPrototype = {
    greet() {
        console.log(`Hi, I'm ${this.name}`);
    }
};

let john = Object.create(personPrototype);
john.name = "John";
john.greet(); // Hi, I'm John
```

---

## Asynchronous JavaScript

### Why Asynchronous?

JavaScript is **single-threaded** - it can only do one thing at a time.

**Problem**: Some operations take time (fetching data, reading files, timers). If JavaScript waited for each operation to complete, the browser would freeze.

**Solution**: Asynchronous programming allows JavaScript to start a task and move on to other things while waiting for it to complete.

**Analogy**:
- **Synchronous**: You order coffee and stand at the counter until it's ready (blocking everyone behind you)
- **Asynchronous**: You order coffee, get a buzzer, and sit down. When it's ready, the buzzer alerts you

### setTimeout and setInterval

```javascript
// setTimeout - run once after delay
console.log("Start");

setTimeout(() => {
    console.log("This runs after 2 seconds");
}, 2000);

console.log("End");

// Output:
// Start
// End
// This runs after 2 seconds (after 2000ms)

// setInterval - run repeatedly
let count = 0;
let interval = setInterval(() => {
    count++;
    console.log(`Count: ${count}`);

    if (count === 5) {
        clearInterval(interval); // Stop interval
    }
}, 1000); // Every 1 second
```

### Callbacks

**Callback** is a function passed to another function to be executed later.

```javascript
function fetchData(callback) {
    console.log("Fetching data...");

    setTimeout(() => {
        let data = { id: 1, name: "John" };
        callback(data);
    }, 2000);
}

fetchData((data) => {
    console.log("Data received:", data);
});

// Output:
// Fetching data...
// (2 seconds later)
// Data received: { id: 1, name: "John" }
```

### Callback Hell

```javascript
// ❌ This is hard to read and maintain
getData((data) => {
    processData(data, (processed) => {
        saveData(processed, (result) => {
            sendNotification(result, (response) => {
                console.log("All done!");
            });
        });
    });
});

// This is called "Pyramid of Doom" or "Callback Hell"
```

---

## Promises

**Promise** is an object representing the eventual completion or failure of an asynchronous operation.

**Analogy**: A promise is like ordering food delivery:
- **Pending**: Order placed, food being prepared
- **Fulfilled**: Food delivered successfully
- **Rejected**: Restaurant cancelled your order

### Creating Promises

```javascript
let promise = new Promise((resolve, reject) => {
    // Simulate async operation
    let success = true;

    setTimeout(() => {
        if (success) {
            resolve("Operation successful!"); // Fulfilled
        } else {
            reject("Operation failed!"); // Rejected
        }
    }, 2000);
});

// Using the promise
promise
    .then(result => {
        console.log(result); // "Operation successful!"
    })
    .catch(error => {
        console.log(error);
    });
```

### Promise States

```javascript
// Pending
let pending = new Promise((resolve) => {
    // Not resolved yet
});

// Fulfilled
let fulfilled = Promise.resolve("Success!");

// Rejected
let rejected = Promise.reject("Error!");
```

### Chaining Promises

```javascript
function fetchUser() {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve({ id: 1, name: "John" });
        }, 1000);
    });
}

function fetchPosts(userId) {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve(["Post 1", "Post 2", "Post 3"]);
        }, 1000);
    });
}

// Chain promises
fetchUser()
    .then(user => {
        console.log("User:", user);
        return fetchPosts(user.id);
    })
    .then(posts => {
        console.log("Posts:", posts);
    })
    .catch(error => {
        console.log("Error:", error);
    });
```

### Promise Methods

```javascript
// Promise.all - wait for all promises
let p1 = Promise.resolve(1);
let p2 = Promise.resolve(2);
let p3 = Promise.resolve(3);

Promise.all([p1, p2, p3])
    .then(results => {
        console.log(results); // [1, 2, 3]
    });

// If one fails, all fail
Promise.all([
    Promise.resolve(1),
    Promise.reject("Error!"),
    Promise.resolve(3)
])
.catch(error => {
    console.log(error); // "Error!"
});

// Promise.race - first to complete wins
Promise.race([
    new Promise(resolve => setTimeout(() => resolve("First"), 1000)),
    new Promise(resolve => setTimeout(() => resolve("Second"), 2000))
])
.then(result => {
    console.log(result); // "First"
});

// Promise.allSettled - wait for all, regardless of outcome
Promise.allSettled([
    Promise.resolve(1),
    Promise.reject("Error"),
    Promise.resolve(3)
])
.then(results => {
    console.log(results);
    // [
    //   { status: "fulfilled", value: 1 },
    //   { status: "rejected", reason: "Error" },
    //   { status: "fulfilled", value: 3 }
    // ]
});

// Promise.any - first successful promise
Promise.any([
    Promise.reject("Error 1"),
    Promise.resolve("Success!"),
    Promise.resolve("Also success")
])
.then(result => {
    console.log(result); // "Success!"
});
```

---

## Async/Await

**async/await** is syntactic sugar over promises, making asynchronous code look synchronous.

### Basic Syntax

```javascript
// Old way with promises
function fetchData() {
    return fetch("https://api.example.com/data")
        .then(response => response.json())
        .then(data => {
            console.log(data);
        })
        .catch(error => {
            console.log(error);
        });
}

// New way with async/await
async function fetchData() {
    try {
        let response = await fetch("https://api.example.com/data");
        let data = await response.json();
        console.log(data);
    } catch (error) {
        console.log(error);
    }
}
```

### async Functions

```javascript
// async function always returns a promise
async function greet() {
    return "Hello!";
}

greet().then(message => console.log(message)); // Hello!

// Same as:
function greet2() {
    return Promise.resolve("Hello!");
}
```

### await Keyword

```javascript
// await pauses execution until promise resolves
async function example() {
    console.log("Start");

    let result = await someAsyncOperation(); // Wait here

    console.log("Result:", result);
    console.log("End");
}

// await can only be used inside async functions
// This is WRONG:
// let result = await fetch("..."); // ❌ Error!

// This is correct:
async function getData() {
    let result = await fetch("..."); // ✅ Works
}
```

### Error Handling

```javascript
// With try/catch
async function fetchUser() {
    try {
        let response = await fetch("https://api.example.com/user");
        let user = await response.json();
        return user;
    } catch (error) {
        console.log("Error fetching user:", error);
        return null;
    }
}

// Without try/catch (handle in then/catch)
async function fetchUser2() {
    let response = await fetch("https://api.example.com/user");
    let user = await response.json();
    return user;
}

fetchUser2()
    .then(user => console.log(user))
    .catch(error => console.log(error));
```

### Parallel Execution

```javascript
// ❌ Sequential (slow) - takes 6 seconds total
async function slow() {
    let user = await fetchUser();    // 2 seconds
    let posts = await fetchPosts();  // 2 seconds
    let comments = await fetchComments(); // 2 seconds
    return { user, posts, comments };
}

// ✅ Parallel (fast) - takes 2 seconds total
async function fast() {
    let [user, posts, comments] = await Promise.all([
        fetchUser(),
        fetchPosts(),
        fetchComments()
    ]);
    return { user, posts, comments };
}

// Another parallel pattern
async function fast2() {
    // Start all requests
    let userPromise = fetchUser();
    let postsPromise = fetchPosts();
    let commentsPromise = fetchComments();

    // Wait for all
    let user = await userPromise;
    let posts = await postsPromise;
    let comments = await commentsPromise;

    return { user, posts, comments };
}
```

### Real-World Example

```javascript
async function getUserData(userId) {
    try {
        // Fetch user
        console.log("Fetching user...");
        let userResponse = await fetch(`https://api.example.com/users/${userId}`);

        if (!userResponse.ok) {
            throw new Error("User not found");
        }

        let user = await userResponse.json();
        console.log("User:", user);

        // Fetch user's posts
        console.log("Fetching posts...");
        let postsResponse = await fetch(`https://api.example.com/users/${userId}/posts`);
        let posts = await postsResponse.json();
        console.log("Posts:", posts);

        // Return combined data
        return {
            user,
            posts,
            postCount: posts.length
        };

    } catch (error) {
        console.error("Error:", error.message);
        return null;
    }
}

// Usage
getUserData(1).then(data => {
    if (data) {
        console.log(`${data.user.name} has ${data.postCount} posts`);
    }
});
```

---

## ES6 Modules

### What are Modules?

**Modules** let you split code into separate files and import/export functionality.

**Benefits**:
- Better code organization
- Reusability
- Avoid naming conflicts
- Easier maintenance

### Export

```javascript
// math.js

// Named exports
export const PI = 3.14159;

export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}

// Or export all at once
const multiply = (a, b) => a * b;
const divide = (a, b) => a / b;

export { multiply, divide };

// Default export (one per file)
export default function square(x) {
    return x * x;
}
```

### Import

```javascript
// app.js

// Named imports
import { add, subtract, PI } from './math.js';

console.log(add(5, 3));      // 8
console.log(subtract(5, 3)); // 2
console.log(PI);             // 3.14159

// Import all as object
import * as math from './math.js';

console.log(math.add(5, 3));  // 8
console.log(math.PI);         // 3.14159

// Import default
import square from './math.js';
console.log(square(5)); // 25

// Rename imports
import { add as sum, subtract as diff } from './math.js';
console.log(sum(5, 3)); // 8

// Mix default and named
import square, { add, PI } from './math.js';
```

### Using Modules in HTML

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>ES6 Modules</title>
</head>
<body>
    <!-- Note: type="module" -->
    <script type="module" src="app.js"></script>
</body>
</html>
```

### CommonJS (Node.js)

```javascript
// math.js (CommonJS - older Node.js style)
function add(a, b) {
    return a + b;
}

module.exports = { add };

// Or
exports.add = add;

// app.js
const { add } = require('./math');
console.log(add(5, 3));
```

---

## Classes

### What are Classes?

**Classes** are templates for creating objects with shared properties and methods.

**Analogy**: A class is like a blueprint for a house. The blueprint defines the structure, but you can build many houses from it.

### Basic Class

```javascript
class Person {
    // Constructor - runs when creating new instance
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    // Method
    greet() {
        console.log(`Hi, I'm ${this.name}, ${this.age} years old`);
    }

    // Another method
    haveBirthday() {
        this.age++;
        console.log(`Happy birthday! I'm now ${this.age}`);
    }
}

// Create instances
let john = new Person("John", 30);
let jane = new Person("Jane", 25);

john.greet();        // Hi, I'm John, 30 years old
jane.haveBirthday(); // Happy birthday! I'm now 26
```

### Getters and Setters

```javascript
class Rectangle {
    constructor(width, height) {
        this.width = width;
        this.height = height;
    }

    // Getter - access like a property
    get area() {
        return this.width * this.height;
    }

    // Setter - set like a property
    set dimensions({ width, height }) {
        this.width = width;
        this.height = height;
    }
}

let rect = new Rectangle(10, 5);
console.log(rect.area); // 50 (no parentheses!)

rect.dimensions = { width: 20, height: 10 };
console.log(rect.area); // 200
```

### Static Methods

```javascript
class MathHelper {
    // Static method - called on class, not instance
    static add(a, b) {
        return a + b;
    }

    static PI = 3.14159;
}

// Call on class
console.log(MathHelper.add(5, 3)); // 8
console.log(MathHelper.PI);        // 3.14159

// Can't call on instance
let helper = new MathHelper();
// helper.add(5, 3); // ❌ Error!
```

### Inheritance

```javascript
// Parent class
class Animal {
    constructor(name) {
        this.name = name;
    }

    speak() {
        console.log(`${this.name} makes a sound`);
    }
}

// Child class
class Dog extends Animal {
    constructor(name, breed) {
        super(name); // Call parent constructor
        this.breed = breed;
    }

    // Override parent method
    speak() {
        console.log(`${this.name} barks`);
    }

    // New method
    fetch() {
        console.log(`${this.name} fetches the ball`);
    }
}

let dog = new Dog("Buddy", "Golden Retriever");
dog.speak();  // Buddy barks
dog.fetch();  // Buddy fetches the ball
```

### Private Fields (Modern)

```javascript
class BankAccount {
    #balance = 0; // Private field (starts with #)

    constructor(initialBalance) {
        this.#balance = initialBalance;
    }

    deposit(amount) {
        this.#balance += amount;
    }

    withdraw(amount) {
        if (amount > this.#balance) {
            console.log("Insufficient funds!");
        } else {
            this.#balance -= amount;
        }
    }

    getBalance() {
        return this.#balance;
    }
}

let account = new BankAccount(100);
account.deposit(50);
console.log(account.getBalance()); // 150

// Can't access private field
// console.log(account.#balance); // ❌ Error!
```

---

## Prototypes and Inheritance

### What are Prototypes?

Every JavaScript object has a hidden `[[Prototype]]` property that links to another object.

**Analogy**: Prototypes are like genetic inheritance - if you don't have something, you can look at your parent.

### Prototype Chain

```javascript
let animal = {
    eats: true,
    walk() {
        console.log("Animal walks");
    }
};

let dog = {
    barks: true
};

// Set prototype
dog.__proto__ = animal; // Don't use in production! Use Object.create()

console.log(dog.barks);  // true (own property)
console.log(dog.eats);   // true (inherited from animal)
dog.walk();              // Animal walks (inherited method)
```

### Constructor Functions

```javascript
// Constructor function (before classes)
function Person(name, age) {
    this.name = name;
    this.age = age;
}

// Add method to prototype
Person.prototype.greet = function() {
    console.log(`Hi, I'm ${this.name}`);
};

let john = new Person("John", 30);
john.greet(); // Hi, I'm John

// All instances share the same method
let jane = new Person("Jane", 25);
console.log(john.greet === jane.greet); // true (same function)
```

### Prototypal Inheritance

```javascript
function Animal(name) {
    this.name = name;
}

Animal.prototype.speak = function() {
    console.log(`${this.name} makes a sound`);
};

function Dog(name, breed) {
    Animal.call(this, name); // Call parent constructor
    this.breed = breed;
}

// Set up inheritance
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
    console.log(`${this.name} barks`);
};

let dog = new Dog("Buddy", "Golden Retriever");
dog.speak(); // Buddy makes a sound (inherited)
dog.bark();  // Buddy barks (own method)
```

---

## Error Handling

### Try/Catch

```javascript
try {
    // Code that might throw an error
    let result = riskyOperation();
    console.log(result);
} catch (error) {
    // Handle error
    console.log("Error occurred:", error.message);
} finally {
    // Always runs (optional)
    console.log("Cleanup code");
}
```

### Throwing Errors

```javascript
function divide(a, b) {
    if (b === 0) {
        throw new Error("Cannot divide by zero!");
    }
    return a / b;
}

try {
    let result = divide(10, 0);
} catch (error) {
    console.log(error.message); // Cannot divide by zero!
}
```

### Custom Errors

```javascript
class ValidationError extends Error {
    constructor(message) {
        super(message);
        this.name = "ValidationError";
    }
}

function validateAge(age) {
    if (age < 0) {
        throw new ValidationError("Age cannot be negative");
    }
    if (age > 150) {
        throw new ValidationError("Age is unrealistic");
    }
    return true;
}

try {
    validateAge(-5);
} catch (error) {
    if (error instanceof ValidationError) {
        console.log("Validation failed:", error.message);
    } else {
        console.log("Unknown error:", error);
    }
}
```

### Error Handling with Async/Await

```javascript
async function fetchData() {
    try {
        let response = await fetch("https://api.example.com/data");

        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        let data = await response.json();
        return data;
    } catch (error) {
        console.error("Fetch error:", error.message);
        return null;
    }
}
```

---

## Regular Expressions

### What are Regular Expressions?

**Regular expressions (regex)** are patterns used to match text.

**Use cases**:
- Validate input (email, phone, etc.)
- Search and replace text
- Extract information from strings

### Basic Syntax

```javascript
// Two ways to create regex
let regex1 = /pattern/;
let regex2 = new RegExp("pattern");

// Test if pattern matches
let text = "Hello World";
let hasHello = /Hello/.test(text);
console.log(hasHello); // true

// Find matches
let matches = text.match(/o/g);
console.log(matches); // ["o", "o"]
```

### Flags

```javascript
// i - case insensitive
let regex = /hello/i;
console.log(regex.test("HELLO")); // true

// g - global (find all matches)
let text = "cat bat rat";
console.log(text.match(/at/g)); // ["at", "at", "at"]

// m - multiline
let multiline = /^line/m;
```

### Common Patterns

```javascript
// Digit
let hasDigit = /\d/.test("abc123"); // true
let allDigits = /^\d+$/.test("123"); // true

// Word characters (letters, digits, underscore)
let isWord = /\w+/.test("hello"); // true

// Whitespace
let hasSpace = /\s/.test("hello world"); // true

// Email validation (simple)
let emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
console.log(emailRegex.test("user@example.com")); // true

// Phone number (US format)
let phoneRegex = /^\d{3}-\d{3}-\d{4}$/;
console.log(phoneRegex.test("555-123-4567")); // true

// URL
let urlRegex = /^https?:\/\/.+/;
console.log(urlRegex.test("https://example.com")); // true
```

### Replace with Regex

```javascript
let text = "Hello World";

// Simple replace
let replaced = text.replace(/World/, "JavaScript");
console.log(replaced); // Hello JavaScript

// Replace all (with g flag)
let text2 = "cat bat rat";
let replaced2 = text2.replace(/at/g, "oot");
console.log(replaced2); // coot boot root

// Replace with function
let text3 = "hello world";
let capitalized = text3.replace(/\b\w/g, char => char.toUpperCase());
console.log(capitalized); // Hello World
```

---

## Introduction to TypeScript

### What is TypeScript?

**TypeScript** is a superset of JavaScript that adds **static typing**.

**Key features**:
- Catches errors before running code
- Better IDE autocomplete
- Self-documenting code
- Scales better for large projects

**Analogy**: JavaScript is like writing without spell-check, TypeScript is like having a grammar checker that catches mistakes as you type.

### JavaScript vs TypeScript

```javascript
// JavaScript
function add(a, b) {
    return a + b;
}

add(5, "3"); // "53" (wtf?)

// TypeScript
function add(a: number, b: number): number {
    return a + b;
}

add(5, "3"); // ❌ Error! Type 'string' is not assignable to type 'number'
```

### Setting Up TypeScript

```bash
# Install TypeScript
npm install -g typescript

# Create tsconfig.json
tsc --init

# Compile TypeScript to JavaScript
tsc filename.ts

# Watch mode (auto-compile on save)
tsc --watch
```

---

## TypeScript Basics

### Type Annotations

```typescript
// Basic types
let name: string = "John";
let age: number = 30;
let isStudent: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Arrays
let numbers: number[] = [1, 2, 3, 4, 5];
let strings: Array<string> = ["a", "b", "c"];

// Tuples (fixed length, specific types)
let person: [string, number] = ["John", 30];

// Any (avoid when possible)
let anything: any = "could be anything";
anything = 123;
anything = true;

// Unknown (safer than any)
let something: unknown = "hello";
// something.toUpperCase(); // ❌ Error!
if (typeof something === "string") {
    something.toUpperCase(); // ✅ Works after type check
}

// Void (no return value)
function log(message: string): void {
    console.log(message);
}

// Never (function never returns)
function throwError(message: string): never {
    throw new Error(message);
}
```

### Function Types

```typescript
// Parameter and return types
function add(a: number, b: number): number {
    return a + b;
}

// Optional parameters
function greet(name: string, greeting?: string): string {
    return `${greeting || "Hello"}, ${name}!`;
}

// Default parameters
function greet2(name: string, greeting: string = "Hello"): string {
    return `${greeting}, ${name}!`;
}

// Rest parameters
function sum(...numbers: number[]): number {
    return numbers.reduce((total, n) => total + n, 0);
}

// Function type
let myFunc: (a: number, b: number) => number;
myFunc = add; // ✅ Works
// myFunc = greet; // ❌ Error! Wrong type
```

### Object Types

```typescript
// Object type annotation
let person: { name: string; age: number } = {
    name: "John",
    age: 30
};

// Optional properties
let user: { name: string; age?: number } = {
    name: "Jane"
};

// Readonly properties
let config: { readonly apiKey: string } = {
    apiKey: "abc123"
};
// config.apiKey = "new"; // ❌ Error! Can't modify readonly
```

### Type Inference

```typescript
// TypeScript can infer types
let name = "John"; // TypeScript knows it's a string
// name = 123; // ❌ Error!

let numbers = [1, 2, 3]; // number[]
let mixed = [1, "two", true]; // (number | string | boolean)[]

function add(a: number, b: number) {
    return a + b; // TypeScript infers return type is number
}
```

---

## Interfaces

### What are Interfaces?

**Interfaces** define the structure of objects.

```typescript
interface Person {
    name: string;
    age: number;
    email?: string; // Optional
    readonly id: number; // Readonly
}

let john: Person = {
    id: 1,
    name: "John",
    age: 30
};

// john.id = 2; // ❌ Error! Can't modify readonly
```

### Interface Methods

```typescript
interface User {
    name: string;
    age: number;
    greet(): void;
    sayAge(): string;
}

let user: User = {
    name: "John",
    age: 30,
    greet() {
        console.log(`Hi, I'm ${this.name}`);
    },
    sayAge() {
        return `I'm ${this.age} years old`;
    }
};
```

### Extending Interfaces

```typescript
interface Animal {
    name: string;
    speak(): void;
}

interface Dog extends Animal {
    breed: string;
    fetch(): void;
}

let dog: Dog = {
    name: "Buddy",
    breed: "Golden Retriever",
    speak() {
        console.log("Woof!");
    },
    fetch() {
        console.log("Fetching...");
    }
};
```

### Interface for Functions

```typescript
interface MathOperation {
    (a: number, b: number): number;
}

let add: MathOperation = (a, b) => a + b;
let multiply: MathOperation = (a, b) => a * b;
```

### Interface for Classes

```typescript
interface Drivable {
    speed: number;
    drive(): void;
    stop(): void;
}

class Car implements Drivable {
    speed: number = 0;

    drive() {
        this.speed = 60;
        console.log("Driving at", this.speed);
    }

    stop() {
        this.speed = 0;
        console.log("Stopped");
    }
}
```

---

## Type Aliases

### What are Type Aliases?

**Type aliases** create custom type names.

```typescript
// Basic alias
type Name = string;
type Age = number;

let myName: Name = "John";
let myAge: Age = 30;

// Object type
type Person = {
    name: string;
    age: number;
};

let john: Person = {
    name: "John",
    age: 30
};

// Function type
type MathOp = (a: number, b: number) => number;

let add: MathOp = (a, b) => a + b;
```

### Type vs Interface

```typescript
// Type can represent primitives, unions, tuples
type ID = string | number;
type Coordinates = [number, number];

// Interface is for object shapes
interface Person {
    name: string;
}

// Both can be extended
type Employee = Person & { employeeId: number };

interface Manager extends Person {
    department: string;
}

// Rule of thumb: Use interface for objects, type for everything else
```

---

## Union and Intersection Types

### Union Types

**Union** means "this OR that" (can be multiple types).

```typescript
// Simple union
let id: string | number;
id = "abc123"; // ✅ OK
id = 12345;    // ✅ OK
// id = true;  // ❌ Error!

// Function with union
function printId(id: string | number) {
    // Type narrowing
    if (typeof id === "string") {
        console.log(id.toUpperCase());
    } else {
        console.log(id.toFixed(2));
    }
}

// Array of mixed types
let mixed: (string | number)[] = [1, "two", 3, "four"];

// Literal types
type Status = "pending" | "approved" | "rejected";

let orderStatus: Status = "pending"; // ✅ OK
// orderStatus = "cancelled"; // ❌ Error! Not in union
```

### Intersection Types

**Intersection** means "this AND that" (combines types).

```typescript
type Person = {
    name: string;
    age: number;
};

type Employee = {
    employeeId: number;
    department: string;
};

// Intersection - has ALL properties
type EmployeePerson = Person & Employee;

let employee: EmployeePerson = {
    name: "John",
    age: 30,
    employeeId: 12345,
    department: "IT"
};

// Mixing interfaces and types
interface HasName {
    name: string;
}

type HasAge = {
    age: number;
};

type PersonInfo = HasName & HasAge;
```

---

## Generics

### What are Generics?

**Generics** allow you to write reusable code that works with multiple types.

**Analogy**: Generics are like a template - you define the structure once, then fill in the specific type later.

### Basic Generic Function

```typescript
// Without generics (not reusable)
function identityString(value: string): string {
    return value;
}

function identityNumber(value: number): number {
    return value;
}

// With generics (reusable!)
function identity<T>(value: T): T {
    return value;
}

let str = identity<string>("hello"); // string
let num = identity<number>(42);      // number

// Type inference (TypeScript figures it out)
let str2 = identity("hello"); // TypeScript knows it's string
```

### Generic Array Functions

```typescript
// First element of array
function first<T>(arr: T[]): T | undefined {
    return arr[0];
}

let numbers = [1, 2, 3];
let firstNum = first(numbers); // number | undefined

let strings = ["a", "b", "c"];
let firstStr = first(strings); // string | undefined

// Filter array
function filter<T>(arr: T[], predicate: (item: T) => boolean): T[] {
    return arr.filter(predicate);
}

let evenNumbers = filter([1, 2, 3, 4, 5], n => n % 2 === 0);
// [2, 4]
```

### Generic Interfaces

```typescript
interface Box<T> {
    value: T;
}

let stringBox: Box<string> = { value: "hello" };
let numberBox: Box<number> = { value: 42 };

// Generic with multiple types
interface Pair<K, V> {
    key: K;
    value: V;
}

let pair: Pair<string, number> = {
    key: "age",
    value: 30
};
```

### Generic Classes

```typescript
class DataStore<T> {
    private data: T[] = [];

    add(item: T): void {
        this.data.push(item);
    }

    get(index: number): T | undefined {
        return this.data[index];
    }

    getAll(): T[] {
        return this.data;
    }
}

let numberStore = new DataStore<number>();
numberStore.add(1);
numberStore.add(2);
console.log(numberStore.getAll()); // [1, 2]

let stringStore = new DataStore<string>();
stringStore.add("hello");
stringStore.add("world");
```

### Generic Constraints

```typescript
// Constrain T to have a length property
interface HasLength {
    length: number;
}

function logLength<T extends HasLength>(item: T): void {
    console.log(item.length);
}

logLength("hello");      // ✅ strings have length
logLength([1, 2, 3]);    // ✅ arrays have length
// logLength(123);       // ❌ Error! numbers don't have length

// Constrain to object with specific property
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

let person = { name: "John", age: 30 };
let name = getProperty(person, "name"); // "John"
// let invalid = getProperty(person, "email"); // ❌ Error!
```

---

## Advanced TypeScript

### Utility Types

TypeScript provides built-in utility types:

```typescript
interface Person {
    name: string;
    age: number;
    email: string;
}

// Partial - make all properties optional
type PartialPerson = Partial<Person>;
let person1: PartialPerson = { name: "John" }; // ✅ OK

// Required - make all properties required
type RequiredPerson = Required<Person>;

// Readonly - make all properties readonly
type ReadonlyPerson = Readonly<Person>;
let person2: ReadonlyPerson = { name: "John", age: 30, email: "j@ex.com" };
// person2.name = "Jane"; // ❌ Error!

// Pick - select specific properties
type PersonName = Pick<Person, "name" | "age">;
let person3: PersonName = { name: "John", age: 30 };

// Omit - exclude specific properties
type PersonWithoutEmail = Omit<Person, "email">;
let person4: PersonWithoutEmail = { name: "John", age: 30 };

// Record - create object type with specific keys and value type
type Roles = "admin" | "user" | "guest";
type Permissions = Record<Roles, boolean>;
let perms: Permissions = {
    admin: true,
    user: true,
    guest: false
};
```

### Type Guards

```typescript
// typeof guard
function processValue(value: string | number) {
    if (typeof value === "string") {
        return value.toUpperCase(); // TypeScript knows it's string
    } else {
        return value.toFixed(2); // TypeScript knows it's number
    }
}

// instanceof guard
class Dog {
    bark() {
        console.log("Woof!");
    }
}

class Cat {
    meow() {
        console.log("Meow!");
    }
}

function makeSound(animal: Dog | Cat) {
    if (animal instanceof Dog) {
        animal.bark();
    } else {
        animal.meow();
    }
}

// Custom type guard
interface Fish {
    swim: () => void;
}

interface Bird {
    fly: () => void;
}

function isFish(animal: Fish | Bird): animal is Fish {
    return (animal as Fish).swim !== undefined;
}

function move(animal: Fish | Bird) {
    if (isFish(animal)) {
        animal.swim();
    } else {
        animal.fly();
    }
}
```

### Mapped Types

```typescript
// Make all properties readonly
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};

// Make all properties optional
type Optional<T> = {
    [P in keyof T]?: T[P];
};

// Make all properties nullable
type Nullable<T> = {
    [P in keyof T]: T[P] | null;
};

interface Person {
    name: string;
    age: number;
}

type NullablePerson = Nullable<Person>;
// { name: string | null; age: number | null }
```

### Conditional Types

```typescript
// T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false

// Extract non-nullable types
type NonNullable<T> = T extends null | undefined ? never : T;

type C = NonNullable<string | null>; // string
type D = NonNullable<number | undefined>; // number
```

---

## DOM Manipulation

### Selecting Elements

```javascript
// Get element by ID
let header = document.getElementById("header");

// Get elements by class name
let buttons = document.getElementsByClassName("btn");

// Get elements by tag name
let paragraphs = document.getElementsByTagName("p");

// Query selector (CSS selector) - first match
let firstButton = document.querySelector(".btn");
let mainDiv = document.querySelector("#main");

// Query selector all - all matches
let allButtons = document.querySelectorAll(".btn");
```

### Modifying Content

```javascript
// Text content
let element = document.getElementById("demo");
element.textContent = "New text"; // Plain text
element.innerText = "New text";   // Respects styling

// HTML content
element.innerHTML = "<strong>Bold text</strong>";

// Attributes
let img = document.querySelector("img");
img.getAttribute("src");
img.setAttribute("src", "new-image.jpg");
img.removeAttribute("alt");

// Classes
element.classList.add("active");
element.classList.remove("hidden");
element.classList.toggle("dark-mode");
element.classList.contains("active"); // true/false

// Styles
element.style.color = "red";
element.style.backgroundColor = "blue";
element.style.fontSize = "20px";
```

### Creating and Removing Elements

```javascript
// Create element
let newDiv = document.createElement("div");
newDiv.textContent = "Hello!";
newDiv.className = "box";

// Append to document
document.body.appendChild(newDiv);

// Insert before
let parent = document.getElementById("container");
let reference = document.getElementById("existing");
parent.insertBefore(newDiv, reference);

// Remove element
let element = document.getElementById("remove-me");
element.remove();

// Or
element.parentElement.removeChild(element);
```

### Event Listeners

```javascript
let button = document.getElementById("myButton");

// Add event listener
button.addEventListener("click", function() {
    console.log("Button clicked!");
});

// With arrow function
button.addEventListener("click", () => {
    console.log("Button clicked!");
});

// Event object
button.addEventListener("click", (event) => {
    console.log("Clicked at:", event.clientX, event.clientY);
    event.preventDefault(); // Prevent default behavior
});

// Remove event listener (need named function)
function handleClick() {
    console.log("Clicked!");
}

button.addEventListener("click", handleClick);
button.removeEventListener("click", handleClick);

// Common events
element.addEventListener("click", handler);
element.addEventListener("dblclick", handler);
element.addEventListener("mouseenter", handler);
element.addEventListener("mouseleave", handler);
input.addEventListener("input", handler);
input.addEventListener("change", handler);
form.addEventListener("submit", handler);
input.addEventListener("keydown", handler);
input.addEventListener("keyup", handler);
```

### Form Handling

```javascript
let form = document.getElementById("myForm");

form.addEventListener("submit", (event) => {
    event.preventDefault(); // Don't submit form

    // Get form values
    let name = document.getElementById("name").value;
    let email = document.getElementById("email").value;

    console.log("Name:", name);
    console.log("Email:", email);

    // Validate
    if (!name || !email) {
        alert("Please fill all fields");
        return;
    }

    // Submit data (e.g., via fetch)
    // ...
});
```

---

## Fetch API and HTTP Requests

### Basic Fetch

```javascript
// GET request
fetch("https://api.example.com/users")
    .then(response => response.json())
    .then(data => {
        console.log(data);
    })
    .catch(error => {
        console.error("Error:", error);
    });

// With async/await (cleaner)
async function getUsers() {
    try {
        let response = await fetch("https://api.example.com/users");
        let data = await response.json();
        console.log(data);
    } catch (error) {
        console.error("Error:", error);
    }
}
```

### POST Request

```javascript
async function createUser(userData) {
    try {
        let response = await fetch("https://api.example.com/users", {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(userData)
        });

        let data = await response.json();
        console.log("Created:", data);
    } catch (error) {
        console.error("Error:", error);
    }
}

createUser({ name: "John", age: 30 });
```

### Handling Errors

```javascript
async function fetchWithErrorHandling(url) {
    try {
        let response = await fetch(url);

        // Check if response is OK
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }

        let data = await response.json();
        return data;
    } catch (error) {
        console.error("Fetch error:", error.message);
        return null;
    }
}
```

### Full CRUD Example

```javascript
const API_URL = "https://jsonplaceholder.typicode.com/posts";

// GET all
async function getPosts() {
    let response = await fetch(API_URL);
    return await response.json();
}

// GET one
async function getPost(id) {
    let response = await fetch(`${API_URL}/${id}`);
    return await response.json();
}

// POST (create)
async function createPost(post) {
    let response = await fetch(API_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(post)
    });
    return await response.json();
}

// PUT (update)
async function updatePost(id, post) {
    let response = await fetch(`${API_URL}/${id}`, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(post)
    });
    return await response.json();
}

// DELETE
async function deletePost(id) {
    let response = await fetch(`${API_URL}/${id}`, {
        method: "DELETE"
    });
    return response.ok;
}

// Usage
getPosts().then(posts => console.log(posts));
createPost({ title: "New Post", body: "Content" });
```

---

## Local Storage and Session Storage

### Local Storage

**Persists** even after closing browser.

```javascript
// Set item
localStorage.setItem("username", "John");

// Get item
let username = localStorage.getItem("username");
console.log(username); // John

// Remove item
localStorage.removeItem("username");

// Clear all
localStorage.clear();

// Store objects (convert to JSON)
let user = { name: "John", age: 30 };
localStorage.setItem("user", JSON.stringify(user));

// Retrieve object
let storedUser = JSON.parse(localStorage.getItem("user"));
console.log(storedUser.name); // John
```

### Session Storage

**Clears** when browser tab is closed.

```javascript
// Same API as localStorage
sessionStorage.setItem("tempData", "value");
let data = sessionStorage.getItem("tempData");
sessionStorage.removeItem("tempData");
sessionStorage.clear();
```

### Practical Example

```javascript
// Save form data
function saveFormData() {
    let formData = {
        name: document.getElementById("name").value,
        email: document.getElementById("email").value
    };

    localStorage.setItem("formData", JSON.stringify(formData));
}

// Load form data
function loadFormData() {
    let saved = localStorage.getItem("formData");

    if (saved) {
        let formData = JSON.parse(saved);
        document.getElementById("name").value = formData.name;
        document.getElementById("email").value = formData.email;
    }
}

// Load on page load
window.addEventListener("load", loadFormData);

// Save on input
document.getElementById("name").addEventListener("input", saveFormData);
document.getElementById("email").addEventListener("input", saveFormData);
```

---

## Best Practices

### Code Style

```javascript
// ✅ Use const by default, let when needed
const PI = 3.14159;
let count = 0;

// ✅ Use meaningful names
const userAge = 25;
const isLoggedIn = true;

// ❌ Avoid cryptic names
const x = 25;
const flg = true;

// ✅ Use camelCase for variables and functions
const firstName = "John";
function getUserName() {}

// ✅ Use PascalCase for classes
class UserAccount {}

// ✅ Use UPPER_CASE for constants
const MAX_SIZE = 100;
const API_KEY = "abc123";
```

### Modern JavaScript Features

```javascript
// ✅ Use arrow functions
const double = x => x * 2;

// ✅ Use template literals
const message = `Hello, ${name}!`;

// ✅ Use destructuring
const { name, age } = person;
const [first, second] = array;

// ✅ Use spread operator
const newArray = [...oldArray, newItem];
const merged = { ...obj1, ...obj2 };

// ✅ Use default parameters
function greet(name = "Guest") {}

// ✅ Use async/await
async function fetchData() {
    const data = await fetch(url);
}
```

### Error Handling

```javascript
// ✅ Always handle errors
async function fetchData() {
    try {
        const response = await fetch(url);
        const data = await response.json();
        return data;
    } catch (error) {
        console.error("Error:", error);
        return null;
    }
}

// ✅ Validate input
function divide(a, b) {
    if (b === 0) {
        throw new Error("Cannot divide by zero");
    }
    return a / b;
}

// ✅ Check for null/undefined
if (user && user.name) {
    console.log(user.name);
}

// Or use optional chaining
console.log(user?.name);
```

### Performance

```javascript
// ✅ Cache DOM queries
const button = document.getElementById("btn");
button.addEventListener("click", handler);

// ❌ Don't query repeatedly
document.getElementById("btn").addEventListener("click", () => {
    document.getElementById("btn").textContent = "Clicked";
});

// ✅ Use event delegation
document.getElementById("list").addEventListener("click", (e) => {
    if (e.target.tagName === "LI") {
        console.log("List item clicked");
    }
});

// ✅ Debounce expensive operations
function debounce(func, delay) {
    let timeout;
    return function(...args) {
        clearTimeout(timeout);
        timeout = setTimeout(() => func.apply(this, args), delay);
    };
}

window.addEventListener("resize", debounce(() => {
    console.log("Resized!");
}, 500));
```

### Security

```javascript
// ✅ Sanitize user input
function sanitizeInput(input) {
    return input.replace(/[<>]/g, "");
}

// ❌ Never use eval()
// eval(userInput); // DANGEROUS!

// ✅ Use textContent, not innerHTML for user data
element.textContent = userInput; // Safe

// ❌ innerHTML with user input
// element.innerHTML = userInput; // Risky!

// ✅ Validate on both client and server
function validateEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

---

## Common Patterns

### Module Pattern

```javascript
const UserModule = (function() {
    // Private variables
    let users = [];

    // Private function
    function validate(user) {
        return user.name && user.email;
    }

    // Public API
    return {
        addUser(user) {
            if (validate(user)) {
                users.push(user);
                return true;
            }
            return false;
        },

        getUsers() {
            return [...users]; // Return copy
        },

        getUserCount() {
            return users.length;
        }
    };
})();

UserModule.addUser({ name: "John", email: "john@ex.com" });
console.log(UserModule.getUserCount());
```

### Observer Pattern

```javascript
class EventEmitter {
    constructor() {
        this.events = {};
    }

    on(event, listener) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(listener);
    }

    emit(event, data) {
        if (this.events[event]) {
            this.events[event].forEach(listener => listener(data));
        }
    }

    off(event, listenerToRemove) {
        if (this.events[event]) {
            this.events[event] = this.events[event].filter(
                listener => listener !== listenerToRemove
            );
        }
    }
}

// Usage
const emitter = new EventEmitter();

emitter.on("userLoggedIn", (user) => {
    console.log(`${user.name} logged in`);
});

emitter.emit("userLoggedIn", { name: "John" });
```

### Factory Pattern

```javascript
class User {
    constructor(name, role) {
        this.name = name;
        this.role = role;
    }
}

class UserFactory {
    static createUser(name, type) {
        switch(type) {
            case "admin":
                return new User(name, "admin");
            case "moderator":
                return new User(name, "moderator");
            default:
                return new User(name, "user");
        }
    }
}

const admin = UserFactory.createUser("John", "admin");
const user = UserFactory.createUser("Jane", "user");
```

### Singleton Pattern

```javascript
class Database {
    constructor() {
        if (Database.instance) {
            return Database.instance;
        }

        this.connection = null;
        Database.instance = this;
    }

    connect() {
        if (!this.connection) {
            this.connection = "Connected";
            console.log("Database connected");
        }
    }

    getConnection() {
        return this.connection;
    }
}

const db1 = new Database();
const db2 = new Database();

console.log(db1 === db2); // true (same instance)
```

---

## Resources and Next Steps

### Free Learning Resources

**JavaScript**:
- [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript) - Best reference
- [JavaScript.info](https://javascript.info/) - Comprehensive tutorial
- [Eloquent JavaScript](https://eloquentjavascript.net/) - Free book
- [freeCodeCamp](https://www.freecodecamp.org/) - Interactive learning

**TypeScript**:
- [TypeScript Official Docs](https://www.typescriptlang.org/docs/)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/) - Free book
- [Execute Program](https://www.executeprogram.com/) - TypeScript course

### Practice Platforms

- [LeetCode](https://leetcode.com/) - Algorithm challenges
- [Codewars](https://www.codewars.com/) - Code katas
- [Frontend Mentor](https://www.frontendmentor.io/) - Real projects
- [JavaScript30](https://javascript30.com/) - 30 day challenge

### Recommended Project Ideas

#### Beginner
1. Todo list app
2. Calculator
3. Weather app (using API)
4. Quiz app
5. Tip calculator

#### Intermediate
6. Expense tracker
7. Recipe finder
8. Movie search app
9. Markdown previewer
10. Pomodoro timer

#### Advanced
11. Chat application
12. E-commerce cart system
13. Kanban board
14. Social media dashboard
15. Real-time collaborative editor

### What to Learn Next

After mastering JavaScript and TypeScript:

1. **Frontend Framework**: React, Vue, or Angular
2. **Backend**: Node.js with Express
3. **Database**: MongoDB or PostgreSQL
4. **Testing**: Jest, Vitest
5. **Build Tools**: Vite, Webpack
6. **State Management**: Redux, Zustand
7. **CSS Framework**: Tailwind CSS
8. **Version Control**: Git and GitHub

### Tips for Success

1. **Code every day**: Even 30 minutes helps
2. **Build projects**: Don't just watch tutorials
3. **Read others' code**: Learn from open source
4. **Ask for help**: Use Stack Overflow, Discord communities
5. **Write clean code**: Focus on readability
6. **Learn debugging**: Master browser DevTools
7. **Stay updated**: JavaScript evolves constantly
8. **Practice problems**: Sharpen your logic skills
9. **Teach others**: Best way to solidify understanding
10. **Be patient**: Mastery takes time

### Communities

- [r/javascript](https://reddit.com/r/javascript) - Reddit community
- [JavaScript Discord](https://discord.gg/javascript)
- [Dev.to](https://dev.to/) - Developer community
- [Stack Overflow](https://stackoverflow.com/)
- [GitHub Discussions](https://github.com/)

---

## Conclusion

Congratulations on completing this comprehensive JavaScript and TypeScript guide!

### Key Takeaways

**JavaScript**:
- Dynamic, versatile language powering the web
- Master fundamentals: variables, functions, arrays, objects
- Understand asynchronous programming: Promises, async/await
- Learn modern features: arrow functions, destructuring, spread
- Practice DOM manipulation and API calls

**TypeScript**:
- Adds static typing to JavaScript
- Catches errors before runtime
- Better IDE support and autocomplete
- Essential for large-scale applications
- Use interfaces, generics, and utility types

### Your Journey Ahead

1. **Master the basics** before moving to frameworks
2. **Build real projects** to apply what you've learned
3. **Read documentation** regularly
4. **Join communities** and help others
5. **Stay curious** and keep learning

Remember: Every expert was once a beginner. The key is consistent practice and never stop learning!

Happy coding!

---

**Document Version**: 1.0
**Author**: Saiprasad Hegde
**Last Updated**: 2025
**License**: Free to use for educational purposes


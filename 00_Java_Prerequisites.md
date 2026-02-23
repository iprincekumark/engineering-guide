# Java Prerequisites

> **File:** `00_Java_Prerequisites.md`
> A beginner-to-DSA-ready handbook in Java. Covers fundamentals, OOP, complexity analysis, arrays, problem-solving techniques, and interview readiness.

---

## Table of Contents

| # | Chapter | Topics |
|---|---------|--------|
| 1 | [Java Fundamentals](#chapter-1-java-fundamentals) | Programming basics, setup, syntax, data types, operators, I/O, control flow |
| 2 | [Methods and Recursion](#chapter-2-methods-and-recursion) | Methods, call stack, recursion, iteration vs recursion, complexity |
| 3 | [Object-Oriented Programming](#chapter-3-object-oriented-programming) | Classes, objects, encapsulation, inheritance, polymorphism, abstraction |
| 4 | [Complexity Analysis](#chapter-4-complexity-analysis-deep-dive) | Big-O, time/space, loop analysis, recursion trees |
| 5 | [Arrays — DSA Foundation](#chapter-5-arrays--dsa-foundation) | Traversal, prefix sum, suffix, difference arrays, interview patterns |
| 6 | [Problem-Solving Techniques](#chapter-6-problem-solving-techniques) | Two pointers, sliding window, pattern recognition |
| 7 | [DSA Readiness](#chapter-7-dsa-readiness) | Problem approach, debugging, optimization, common mistakes |

---

# Chapter 1: Java Fundamentals

---

## 1.1 What Is Programming?

### Definition

Programming is the process of writing a set of instructions that a computer can understand and execute to perform a specific task.

### Intuition

Think of it like writing a recipe. A recipe tells a cook **what ingredients** to use and **what steps** to follow. Similarly, a program tells a computer what data to use and what operations to perform, step by step.

### Key Ideas

- Computers only understand **binary** (0s and 1s).
- Programming languages like Java let us write instructions in a **human-readable** form.
- A special tool (compiler/interpreter) translates our instructions into binary.

---

## 1.2 What Is Java and How It Works

### Definition

Java is a **high-level, object-oriented, platform-independent** programming language created by James Gosling at Sun Microsystems in 1995 (now owned by Oracle).

### How Java Works — The Lifecycle

```
Source Code (.java)
       │
       ▼
   javac (Compiler)
       │
       ▼
 Bytecode (.class)
       │
       ▼
  JVM (Java Virtual Machine)
       │
       ▼
  Machine Code (runs on your OS)
```

| Term | Meaning |
|------|---------|
| **JDK** | Java Development Kit — tools to write and compile Java |
| **JRE** | Java Runtime Environment — tools to run Java programs |
| **JVM** | Java Virtual Machine — executes bytecode on any OS |

### Why "Write Once, Run Anywhere"?

Java source code is compiled into **bytecode**, not native machine code. The JVM on each operating system interprets this bytecode. So the same `.class` file runs on Windows, Mac, and Linux.

### Real-Life Analogy

Think of bytecode like a **universal recipe** written in a standard format. Each country (OS) has its own chef (JVM) who can read and cook from that recipe.

---

## 1.3 Installation and Setup

### Step-by-Step

1. **Download JDK** from [https://www.oracle.com/java/technologies/downloads/](https://www.oracle.com/java/technologies/downloads/) or use OpenJDK.
2. **Install** the JDK (follow installer instructions for your OS).
3. **Set `JAVA_HOME`** environment variable to the JDK install path.
4. **Verify Installation:**

```bash
java -version
javac -version
```

### Recommended IDEs

| IDE | Best For |
|-----|----------|
| IntelliJ IDEA | Professional Java development |
| VS Code + Java Extension Pack | Lightweight, fast |
| Eclipse | Traditional Java IDE |

### Your First Program (Terminal)

```bash
# 1. Create a file
echo 'public class Hello { public static void main(String[] args) { System.out.println("Hello, World!"); } }' > Hello.java

# 2. Compile
javac Hello.java

# 3. Run
java Hello
```

---

## 1.4 Structure of a Java Program

### Syntax

```java
public class MyProgram {               // Class declaration
    public static void main(String[] args) {  // Entry point
        System.out.println("Hello!");   // Statement
    }
}
```

### Breakdown

| Part | Purpose |
|------|---------|
| `public class MyProgram` | Declares a class named `MyProgram`. File name must match. |
| `public static void main(String[] args)` | The main method — Java starts execution here. |
| `System.out.println(...)` | Prints text to the console. |
| `{ }` | Curly braces define code blocks. |
| `;` | Every statement ends with a semicolon. |

### Rules

- **File name** must exactly match the **public class name** (case-sensitive).
- Every Java program must have a `main` method to run.
- Java is **case-sensitive**: `Main` ≠ `main`.

### Common Mistake

```java
// ❌ File named "myprogram.java" but class is "MyProgram"
// This will cause a compilation error!
public class MyProgram { ... }
```

---

## 1.5 Keywords, Identifiers, and Variables

### Keywords

Keywords are **reserved words** in Java that have special meaning. You **cannot** use them as variable or class names.

| Category | Keywords |
|----------|----------|
| Data types | `int`, `float`, `double`, `char`, `boolean`, `long`, `short`, `byte` |
| Control flow | `if`, `else`, `switch`, `case`, `for`, `while`, `do`, `break`, `continue` |
| Access | `public`, `private`, `protected` |
| Class-related | `class`, `interface`, `extends`, `implements`, `abstract` |
| Other | `static`, `void`, `return`, `new`, `this`, `super`, `final`, `null` |

### Identifiers

An identifier is a **name** you choose for variables, methods, classes, etc.

**Rules:**
- Must start with a letter, `_`, or `$`.
- Cannot start with a digit.
- Cannot be a Java keyword.
- Case-sensitive.

```java
int age;        // ✅ valid
int _count;     // ✅ valid
int $price;     // ✅ valid
int 2ndPlace;   // ❌ cannot start with digit
int class;      // ❌ 'class' is a keyword
```

### Variables

A variable is a **named container** that stores a value in memory.

```java
// Syntax: dataType variableName = value;
int age = 21;
String name = "Prince";
double gpa = 3.8;
boolean isStudent = true;
```

### Real-Life Analogy

A variable is like a **labeled box**. The label is the variable name, and the item inside is the value. The type of box determines what you can put inside (integers, text, etc.).

---

## 1.6 Data Types

Java has two categories of data types:

### Primitive Data Types

| Type | Size | Range | Default | Example |
|------|------|-------|---------|---------|
| `byte` | 1 byte | -128 to 127 | 0 | `byte b = 100;` |
| `short` | 2 bytes | -32,768 to 32,767 | 0 | `short s = 1000;` |
| `int` | 4 bytes | -2³¹ to 2³¹-1 | 0 | `int x = 42;` |
| `long` | 8 bytes | -2⁶³ to 2⁶³-1 | 0L | `long l = 99999L;` |
| `float` | 4 bytes | ~6-7 decimal digits | 0.0f | `float f = 3.14f;` |
| `double` | 8 bytes | ~15 decimal digits | 0.0d | `double d = 3.14159;` |
| `char` | 2 bytes | 0 to 65,535 (Unicode) | '\u0000' | `char c = 'A';` |
| `boolean` | 1 bit* | `true` or `false` | false | `boolean flag = true;` |

> *The actual memory used by `boolean` is JVM-dependent (often 1 byte).

### Non-Primitive (Reference) Types

| Type | Description | Example |
|------|-------------|---------|
| `String` | Sequence of characters | `String s = "Hello";` |
| Arrays | Collection of same-type elements | `int[] arr = {1,2,3};` |
| Classes | Custom types | `Scanner sc = new Scanner(System.in);` |

### Type Casting

```java
// Implicit (widening) — smaller to larger, automatic
int a = 10;
double b = a;  // int → double, no data loss

// Explicit (narrowing) — larger to smaller, manual
double x = 9.7;
int y = (int) x;  // y = 9 (decimal part is truncated, NOT rounded)
```

### Common Mistake

```java
// ❌ Forgetting 'f' suffix for float
float price = 9.99;   // ERROR: 9.99 is treated as double
float price = 9.99f;  // ✅ Correct

// ❌ Forgetting 'L' suffix for long
long big = 9999999999;   // ERROR: literal too large for int
long big = 9999999999L;  // ✅ Correct
```

---

## 1.7 Operators

### Arithmetic Operators

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `+` | Addition | `5 + 3` | `8` |
| `-` | Subtraction | `5 - 3` | `2` |
| `*` | Multiplication | `5 * 3` | `15` |
| `/` | Division | `7 / 2` | `3` (integer division!) |
| `%` | Modulus (remainder) | `7 % 2` | `1` |

```java
// ⚠️ Integer division truncates!
System.out.println(7 / 2);    // 3 (not 3.5)
System.out.println(7.0 / 2);  // 3.5 (at least one operand is double)
```

### Comparison (Relational) Operators

| Operator | Meaning | Example | Result |
|----------|---------|---------|--------|
| `==` | Equal to | `5 == 5` | `true` |
| `!=` | Not equal to | `5 != 3` | `true` |
| `>` | Greater than | `5 > 3` | `true` |
| `<` | Less than | `5 < 3` | `false` |
| `>=` | Greater or equal | `5 >= 5` | `true` |
| `<=` | Less or equal | `3 <= 5` | `true` |

### Logical Operators

| Operator | Meaning | Example | Result |
|----------|---------|---------|--------|
| `&&` | AND | `true && false` | `false` |
| `\|\|` | OR | `true \|\| false` | `true` |
| `!` | NOT | `!true` | `false` |

### Assignment Operators

```java
int x = 10;
x += 5;   // x = x + 5  → 15
x -= 3;   // x = x - 3  → 12
x *= 2;   // x = x * 2  → 24
x /= 4;   // x = x / 4  → 6
x %= 4;   // x = x % 4  → 2
```

### Increment / Decrement

```java
int a = 5;
a++;       // Post-increment: use first, then add 1 → a becomes 6
++a;       // Pre-increment: add 1 first, then use → a becomes 7
a--;       // Post-decrement
--a;       // Pre-decrement

// Tricky example:
int x = 5;
int y = x++;  // y = 5 (old value), x = 6
int z = ++x;  // x = 7, z = 7 (new value)
```

### Ternary Operator

```java
// Syntax: condition ? valueIfTrue : valueIfFalse
int age = 20;
String status = (age >= 18) ? "Adult" : "Minor";  // "Adult"
```

---

## 1.8 Input / Output

### Output

```java
System.out.println("Hello World");  // Prints + newline
System.out.print("Hello ");         // Prints without newline
System.out.print("World");
// Output: Hello World

// Formatted output
String name = "Prince";
int age = 21;
System.out.printf("Name: %s, Age: %d%n", name, age);
```

| Format Specifier | Type |
|------------------|------|
| `%d` | Integer |
| `%f` | Float/Double |
| `%s` | String |
| `%c` | Character |
| `%b` | Boolean |
| `%n` | Newline (platform-independent) |

### Input (Scanner)

```java
import java.util.Scanner;

public class InputExample {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter your name: ");
        String name = sc.nextLine();       // Reads a full line

        System.out.print("Enter your age: ");
        int age = sc.nextInt();            // Reads an integer

        System.out.print("Enter your GPA: ");
        double gpa = sc.nextDouble();      // Reads a double

        System.out.println("Name: " + name + ", Age: " + age + ", GPA: " + gpa);
        sc.close();
    }
}
```

### Common Mistake — `nextLine()` after `nextInt()`

```java
Scanner sc = new Scanner(System.in);
int age = sc.nextInt();
// ❌ BUG: nextLine() reads the leftover newline from nextInt()
String name = sc.nextLine();  // This reads an empty string!

// ✅ FIX: Add an extra nextLine() to consume the leftover newline
int age2 = sc.nextInt();
sc.nextLine();                // consume the leftover newline
String name2 = sc.nextLine(); // now this works correctly
```

---

## 1.9 Control Flow

### 1.9.1 if / else if / else

```java
int marks = 75;

if (marks >= 90) {
    System.out.println("Grade: A");
} else if (marks >= 80) {
    System.out.println("Grade: B");
} else if (marks >= 70) {
    System.out.println("Grade: C");
} else {
    System.out.println("Grade: F");
}
// Output: Grade: C
```

### 1.9.2 switch Statement

```java
int day = 3;

switch (day) {
    case 1:
        System.out.println("Monday");
        break;
    case 2:
        System.out.println("Tuesday");
        break;
    case 3:
        System.out.println("Wednesday");
        break;
    default:
        System.out.println("Other day");
}
// Output: Wednesday
```

**Enhanced switch (Java 14+):**

```java
String result = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    case 3 -> "Wednesday";
    default -> "Other day";
};
System.out.println(result);  // Wednesday
```

### Common Mistake — Forgetting `break`

```java
// ❌ Without break, execution "falls through" to the next case!
switch (day) {
    case 1:
        System.out.println("Monday");
        // missing break → falls through!
    case 2:
        System.out.println("Tuesday");
        break;
}
// If day=1, prints BOTH "Monday" AND "Tuesday"
```

### 1.9.3 Loops

#### `for` loop

```java
// Syntax: for (initialization; condition; update)
for (int i = 0; i < 5; i++) {
    System.out.print(i + " ");
}
// Output: 0 1 2 3 4
```

#### `while` loop

```java
int i = 0;
while (i < 5) {
    System.out.print(i + " ");
    i++;
}
// Output: 0 1 2 3 4
```

#### `do-while` loop

```java
// Body executes AT LEAST ONCE, even if condition is false
int i = 10;
do {
    System.out.print(i + " ");
    i++;
} while (i < 5);
// Output: 10 (executes once because condition is checked AFTER body)
```

#### `for-each` loop

```java
int[] numbers = {10, 20, 30, 40};
for (int num : numbers) {
    System.out.print(num + " ");
}
// Output: 10 20 30 40
```

### Loop Control: `break` and `continue`

```java
// break — exits the loop entirely
for (int i = 0; i < 10; i++) {
    if (i == 5) break;
    System.out.print(i + " ");
}
// Output: 0 1 2 3 4

// continue — skips the current iteration
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) continue;  // skip even numbers
    System.out.print(i + " ");
}
// Output: 1 3 5 7 9
```

### Nested Loops — Pattern Example

```java
// Print a right triangle of stars
int n = 5;
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= i; j++) {
        System.out.print("* ");
    }
    System.out.println();
}
/*
Output:
*
* *
* * *
* * * *
* * * * *
*/
```

---

### 📝 Chapter 1 — Practice Problems

| # | Problem | Hint |
|---|---------|------|
| 1 | Print all even numbers from 1 to 100 | Use `%` operator |
| 2 | Find the largest of three numbers | Use nested `if-else` |
| 3 | Check if a number is prime | Loop from 2 to √n |
| 4 | Print multiplication table for a given number | Use a `for` loop |
| 5 | Calculate factorial of a number | Use a loop with accumulator |
| 6 | Reverse the digits of an integer | Use `% 10` and `/ 10` |
| 7 | Check if a number is a palindrome | Compare with reversed number |
| 8 | Print Fibonacci sequence up to n terms | Track previous two values |
| 9 | Sum of digits of a number | Use loop with `%` and `/` |
| 10 | Print a diamond star pattern | Combine two triangle patterns |

---

# Chapter 2: Methods and Recursion

---

## 2.1 Methods and Parameters

### Definition

A **method** (also called a function) is a reusable block of code that performs a specific task. Methods help us organize code, avoid repetition, and make programs easier to read and maintain.

### Real-Life Analogy

A method is like a **vending machine**: you put something in (input/parameters), it does some work internally, and it gives you something back (return value). You don't need to know how it works inside — you just use it.

### Syntax

```java
accessModifier returnType methodName(parameterList) {
    // method body
    return value;  // if return type is not void
}
```

### Example

```java
public class Calculator {

    // Method with parameters and return value
    public static int add(int a, int b) {
        return a + b;
    }

    // Method with no return value (void)
    public static void greet(String name) {
        System.out.println("Hello, " + name + "!");
    }

    public static void main(String[] args) {
        int sum = add(5, 3);       // Calling add method
        System.out.println(sum);   // 8

        greet("Prince");           // Hello, Prince!
    }
}
```

### Parameter vs Argument

| Term | Meaning | Example |
|------|---------|---------|
| **Parameter** | Variable in the method **definition** | `int a, int b` in `add(int a, int b)` |
| **Argument** | Actual value **passed** when calling | `5, 3` in `add(5, 3)` |

### Method Overloading

Methods can have the **same name** but **different parameter lists**. The compiler decides which to call based on arguments.

```java
public static int add(int a, int b) { return a + b; }
public static double add(double a, double b) { return a + b; }
public static int add(int a, int b, int c) { return a + b + c; }

// Usage:
add(2, 3);       // calls int version → 5
add(2.5, 3.5);   // calls double version → 6.0
add(1, 2, 3);    // calls three-param version → 6
```

---

## 2.2 Return Values

### Definition

A method's **return type** specifies what type of data the method sends back to the caller. If a method returns nothing, its return type is `void`.

### Rules

- A method can return **at most one** value.
- The `return` statement **immediately exits** the method.
- The return type must match the declared type.

```java
// Returns the maximum of two numbers
public static int max(int a, int b) {
    if (a > b) {
        return a;   // exits method here if a > b
    }
    return b;       // exits method here otherwise
}

// Void method — no return value
public static void printEven(int n) {
    for (int i = 2; i <= n; i += 2) {
        System.out.print(i + " ");
    }
    // return; is optional for void methods
}
```

### Common Mistake

```java
// ❌ Code after return is unreachable
public static int getValue() {
    return 5;
    System.out.println("This never runs!");  // COMPILE ERROR
}
```

---

## 2.3 The Call Stack

### Definition

The **call stack** is a memory structure (LIFO — Last In, First Out) that Java uses to manage method calls. Every time a method is called, a new **stack frame** is pushed onto the stack. When the method returns, its frame is popped off.

### Visual Example

```java
public static void main(String[] args) {
    int result = square(5);
    System.out.println(result);
}

public static int square(int n) {
    return multiply(n, n);
}

public static int multiply(int a, int b) {
    return a * b;
}
```

```
Call Stack (grows upward):

│ multiply(5, 5)  │  ← currently executing
│ square(5)       │
│ main()          │
└─────────────────┘

After multiply returns 25:
│ square(5) → 25  │  ← currently executing
│ main()          │
└─────────────────┘

After square returns 25:
│ main()          │  ← currently executing
└─────────────────┘
```

### Stack Overflow

If there are too many method calls (e.g., infinite recursion), the stack runs out of space and Java throws a `StackOverflowError`.

```java
// ❌ Infinite recursion — causes StackOverflowError
public static void infinite() {
    infinite();  // calls itself forever
}
```

---

## 2.4 Recursion (Deep Explanation)

### Definition

**Recursion** is when a method calls itself to solve a smaller version of the same problem. Every recursive method needs:

1. **Base case** — the condition where recursion stops.
2. **Recursive case** — the method calls itself with a smaller/simpler input.

### Real-Life Analogy

Imagine you're in a line of people and want to know your position. You ask the person in front of you, "What's your position?" They ask the person in front of them, and so on, until the person at the front says "I'm position 1." Then each person adds 1 to the answer they received and passes it back. That's recursion!

### Example 1: Factorial

```
Factorial(5) = 5 × 4 × 3 × 2 × 1 = 120

Recursive definition:
  factorial(0) = 1             ← base case
  factorial(n) = n × factorial(n-1)  ← recursive case
```

```java
public static int factorial(int n) {
    // Base case
    if (n == 0 || n == 1) {
        return 1;
    }
    // Recursive case
    return n * factorial(n - 1);
}

// Trace:
// factorial(5) → 5 * factorial(4)
//                    → 4 * factorial(3)
//                          → 3 * factorial(2)
//                                → 2 * factorial(1)
//                                      → 1 (base case!)
//                                → 2 * 1 = 2
//                          → 3 * 2 = 6
//                    → 4 * 6 = 24
//              → 5 * 24 = 120
```

### Example 2: Fibonacci

```java
public static int fibonacci(int n) {
    if (n <= 1) return n;       // base case: fib(0)=0, fib(1)=1
    return fibonacci(n - 1) + fibonacci(n - 2);  // recursive case
}

// fib(5) → fib(4) + fib(3)
//          fib(3)+fib(2)   fib(2)+fib(1)
//          ... and so on (many repeated calculations!)
```

### Example 3: Sum of Array

```java
public static int arraySum(int[] arr, int index) {
    if (index == arr.length) return 0;           // base case
    return arr[index] + arraySum(arr, index + 1); // recursive case
}

// Usage:
int[] nums = {1, 2, 3, 4, 5};
System.out.println(arraySum(nums, 0));  // 15
```

### Example 4: Reverse a String

```java
public static String reverse(String str) {
    if (str.isEmpty()) return "";                 // base case
    return reverse(str.substring(1)) + str.charAt(0);  // recursive case
}
// reverse("hello") → reverse("ello") + 'h'
//                   → reverse("llo") + 'e' + 'h'
//                   → ... → "olleh"
```

### The Three Laws of Recursion

1. A recursive method **must have a base case**.
2. A recursive method **must move toward the base case**.
3. A recursive method **must call itself**.

---

## 2.5 Recursion vs Iteration

| Aspect | Recursion | Iteration |
|--------|-----------|-----------|
| **Definition** | Method calls itself | Uses loops (`for`, `while`) |
| **Memory** | Uses call stack (extra space) | Uses fixed variables |
| **Readability** | Often cleaner for tree/divide problems | Simpler for linear tasks |
| **Performance** | Can be slower (overhead of calls) | Usually faster |
| **Risk** | StackOverflowError if too deep | Infinite loop if no termination |
| **When to use** | Trees, graphs, divide & conquer | Simple iterations, counting |

### Same Problem — Both Ways

```java
// Iterative factorial
public static long factorialIterative(int n) {
    long result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}

// Recursive factorial
public static long factorialRecursive(int n) {
    if (n <= 1) return 1;
    return n * factorialRecursive(n - 1);
}
```

### Tail Recursion

A recursive call is **tail-recursive** if the recursive call is the **very last thing** the method does — there's no pending operation after it returns. Some languages optimize tail recursion into iteration, but **Java does NOT** optimize tail recursion.

```java
// ❌ NOT tail-recursive (multiplication is pending after recursive call)
public static int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);  // must multiply AFTER call returns
}

// ✅ Tail-recursive version (result carried via accumulator)
public static int factorialTail(int n, int accumulator) {
    if (n <= 1) return accumulator;
    return factorialTail(n - 1, n * accumulator);  // last operation is the call
}
```

---

## 2.6 Complexity of Recursion

### How to Analyze

1. **Count the number of recursive calls** per invocation.
2. **Determine the depth** of recursion (how many levels deep it goes).
3. **Identify the work done** at each level.

### Common Patterns

| Pattern | Example | Time | Space |
|---------|---------|------|-------|
| Linear recursion (1 call, reduces by 1) | `factorial(n)` | O(n) | O(n) |
| Binary recursion (2 calls, reduces by 1) | `fibonacci(n)` | O(2ⁿ) | O(n) |
| Divide & conquer (2 calls, halves input) | `mergeSort(n)` | O(n log n) | O(n) |
| Linear search recursion | `search(arr, n)` | O(n) | O(n) |
| Binary search recursion | `binarySearch(arr, n)` | O(log n) | O(log n) |

### Recursion Tree — Fibonacci Example

```
                    fib(5)                     → Level 0: 1 call
                   /      \
              fib(4)      fib(3)               → Level 1: 2 calls
             /    \       /    \
         fib(3) fib(2) fib(2) fib(1)           → Level 2: 4 calls
         /  \    / \    / \
     fib(2) fib(1) ...                         → Level 3: 8 calls
     ...

Total calls ≈ 2⁰ + 2¹ + 2² + ... + 2ⁿ = O(2ⁿ)
Space = O(n) (max stack depth = n)
```

### Factorial — Complexity Proof

```
factorial(n) → factorial(n-1) → factorial(n-2) → ... → factorial(0)

Number of calls: n + 1 → O(n)
Work per call: O(1) (just one multiplication)
Total Time: O(n)
Space: O(n) (n frames on the call stack)
```

### Common Mistake

> "Fibonacci is O(n) because the depth is n."

**Wrong!** The depth is n, but at each level the number of calls **doubles**. Total calls ≈ 2ⁿ. Time is **O(2ⁿ)**, not O(n). Space is O(n) because the call stack only holds one path at a time.

---

### 📝 Chapter 2 — Practice Problems

| # | Problem | Hint |
|---|---------|------|
| 1 | Write a method to check if a number is prime | Return `boolean` |
| 2 | Write a recursive method to compute power(base, exp) | `base * power(base, exp-1)` |
| 3 | Print numbers 1 to n using recursion (no loop) | Base case: `n == 0` |
| 4 | Reverse an array using recursion | Swap ends, recurse on middle |
| 5 | Check if a string is a palindrome using recursion | Compare first and last chars |
| 6 | Count digits of a number using recursion | `n / 10` reduces digits |
| 7 | Tower of Hanoi | Classic 3-peg problem |
| 8 | Recursive binary search | Compare mid, recurse on half |
| 9 | Sum of digits using recursion | `n%10 + f(n/10)` |
| 10 | Generate all subsets of a string | Include/exclude each char |

---

# Chapter 3: Object-Oriented Programming (OOP)

---

## 3.1 Classes and Objects

### Definition

- A **class** is a blueprint or template that defines the structure and behavior of objects.
- An **object** is a specific instance created from a class.

### Real-Life Analogy

A class is like an **architectural blueprint** for a house. The blueprint itself is not a house — it describes what a house looks like. Each actual house built from that blueprint is an **object**.

### Syntax

```java
// Defining a class
public class Car {
    // Fields (attributes/properties)
    String brand;
    String color;
    int speed;

    // Method (behavior)
    void accelerate() {
        speed += 10;
        System.out.println(brand + " is now going " + speed + " km/h");
    }
}

// Creating objects
public class Main {
    public static void main(String[] args) {
        Car car1 = new Car();        // Object 1
        car1.brand = "Toyota";
        car1.color = "Red";
        car1.speed = 0;
        car1.accelerate();           // Toyota is now going 10 km/h

        Car car2 = new Car();        // Object 2
        car2.brand = "Honda";
        car2.color = "Blue";
        car2.speed = 50;
        car2.accelerate();           // Honda is now going 60 km/h
    }
}
```

### `new` Keyword

The `new` keyword allocates memory on the **heap** for the object and returns a **reference** (pointer) to that memory.

```java
Car car1 = new Car();
//  ↑         ↑
//  reference  actual object in heap memory
```

---

## 3.2 Constructors

### Definition

A **constructor** is a special method that is called **automatically** when an object is created. It has the **same name as the class** and **no return type**.

### Types of Constructors

```java
public class Student {
    String name;
    int age;

    // 1. Default constructor (no arguments)
    public Student() {
        this.name = "Unknown";
        this.age = 0;
    }

    // 2. Parameterized constructor
    public Student(String name, int age) {
        this.name = name;    // 'this' refers to the current object
        this.age = age;
    }

    // 3. Copy constructor (copies another object's data)
    public Student(Student other) {
        this.name = other.name;
        this.age = other.age;
    }

    void display() {
        System.out.println(name + ", Age: " + age);
    }
}

// Usage:
Student s1 = new Student();                // Default constructor
Student s2 = new Student("Prince", 21);    // Parameterized
Student s3 = new Student(s2);              // Copy constructor

s1.display();  // Unknown, Age: 0
s2.display();  // Prince, Age: 21
s3.display();  // Prince, Age: 21
```

### The `this` Keyword

`this` refers to the **current object**. It is used to:
- Distinguish between instance variables and parameters with the same name.
- Call one constructor from another (constructor chaining).

```java
public class Box {
    int length, width;

    public Box(int length, int width) {
        this.length = length;  // 'this.length' is the field; 'length' is the parameter
        this.width = width;
    }

    // Constructor chaining
    public Box(int side) {
        this(side, side);      // calls the two-parameter constructor
    }
}
```

### Common Mistake

```java
// ❌ If you define ANY constructor, Java does NOT provide a default one
public class Person {
    String name;
    public Person(String name) { this.name = name; }
}

Person p = new Person();  // COMPILE ERROR — no default constructor exists!
```

---

## 3.3 Encapsulation

### Definition

**Encapsulation** is the practice of wrapping data (fields) and the methods that operate on that data into a single unit (class), and **restricting direct access** to the data using access modifiers.

### Real-Life Analogy

Think of a **capsule** (medicine). The contents are hidden inside a protective shell. You take the capsule to get the benefit without touching the contents directly. Similarly, encapsulation hides the internal state and exposes only controlled access through methods.

### Access Modifiers

| Modifier | Class | Package | Subclass | World |
|----------|:-----:|:-------:|:--------:|:-----:|
| `public` | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| (default/package) | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ |

### Implementation — Getters and Setters

```java
public class BankAccount {
    private double balance;  // private — cannot be accessed directly

    // Constructor
    public BankAccount(double initialBalance) {
        if (initialBalance >= 0) {
            this.balance = initialBalance;
        }
    }

    // Getter — controlled read access
    public double getBalance() {
        return balance;
    }

    // Setter — controlled write access with validation
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposited: " + amount);
        } else {
            System.out.println("Invalid amount!");
        }
    }
}

// Usage:
BankAccount acc = new BankAccount(1000);
// acc.balance = -500;     // ❌ COMPILE ERROR — balance is private
acc.deposit(500);          // ✅ Deposited: 500
System.out.println(acc.getBalance());  // 1500.0
```

### Why Encapsulation Matters for DSA

When you build data structures like `Stack`, `Queue`, or `LinkedList`, encapsulation ensures that internal structure (arrays, pointers) cannot be corrupted by external code.

---

## 3.4 Inheritance

### Definition

**Inheritance** allows a class (child/subclass) to **acquire** the fields and methods of another class (parent/superclass). It promotes **code reuse** and models "is-a" relationships.

### Real-Life Analogy

A **child inherits traits** from a parent — eye color, height, etc. Similarly, a `Dog` class inherits common properties from an `Animal` class (like `name`, `eat()`) and adds its own (`bark()`).

### Syntax

```java
// Parent class
public class Animal {
    String name;

    public void eat() {
        System.out.println(name + " is eating.");
    }

    public void sleep() {
        System.out.println(name + " is sleeping.");
    }
}

// Child class — inherits from Animal
public class Dog extends Animal {
    String breed;

    public void bark() {
        System.out.println(name + " says: Woof!");
    }
}

// Usage:
Dog dog = new Dog();
dog.name = "Buddy";       // inherited field
dog.breed = "Labrador";   // Dog's own field
dog.eat();                // inherited method → Buddy is eating.
dog.bark();               // Dog's own method → Buddy says: Woof!
```

### The `super` Keyword

`super` refers to the **parent class**. Used to:
- Call the parent's constructor.
- Access overridden parent methods.

```java
public class Animal {
    String name;
    public Animal(String name) {
        this.name = name;
    }
}

public class Cat extends Animal {
    String color;

    public Cat(String name, String color) {
        super(name);          // calls Animal's constructor — MUST be first line
        this.color = color;
    }
}
```

### Types of Inheritance in Java

| Type | Supported? | Example |
|------|:----------:|---------|
| Single | ✅ | `Dog extends Animal` |
| Multilevel | ✅ | `Puppy extends Dog extends Animal` |
| Hierarchical | ✅ | `Dog extends Animal`, `Cat extends Animal` |
| Multiple (classes) | ❌ | Not supported (use interfaces instead) |

---

## 3.5 Polymorphism

### Definition

**Polymorphism** means "many forms." In Java, the **same method name** can behave differently based on the object or arguments.

### Two Types

#### 1. Compile-Time Polymorphism (Method Overloading)

Same method name, **different parameters**, resolved at **compile time**.

```java
public class MathUtils {
    public static int add(int a, int b) { return a + b; }
    public static double add(double a, double b) { return a + b; }
    public static int add(int a, int b, int c) { return a + b + c; }
}
```

#### 2. Runtime Polymorphism (Method Overriding)

Child class provides its own version of a parent method, resolved at **runtime**.

```java
public class Animal {
    public void sound() {
        System.out.println("Some generic animal sound");
    }
}

public class Dog extends Animal {
    @Override
    public void sound() {
        System.out.println("Woof!");
    }
}

public class Cat extends Animal {
    @Override
    public void sound() {
        System.out.println("Meow!");
    }
}

// Runtime polymorphism in action:
public class Main {
    public static void main(String[] args) {
        Animal myAnimal;

        myAnimal = new Dog();
        myAnimal.sound();    // Woof! (Dog's version)

        myAnimal = new Cat();
        myAnimal.sound();    // Meow! (Cat's version)
    }
}
```

### Why Polymorphism Matters for DSA

When implementing `Comparable` or `Comparator` interfaces for sorting, you override `compareTo()` or `compare()` — that's polymorphism at work.

---

## 3.6 Abstraction

### Definition

**Abstraction** means showing only the **essential features** and hiding the implementation details.

### Real-Life Analogy

When you drive a car, you use the **steering wheel, pedals, and gear shift** (interface). You don't need to know how the engine, transmission, or fuel injection system works internally. The complexity is **abstracted away**.

### Abstract Classes

An abstract class **cannot be instantiated** and may contain both abstract (unimplemented) and concrete (implemented) methods.

```java
public abstract class Shape {
    String color;

    // Abstract method — no body, MUST be overridden by subclasses
    public abstract double area();

    // Concrete method — has a body, inherited as-is
    public void displayColor() {
        System.out.println("Color: " + color);
    }
}

public class Circle extends Shape {
    double radius;

    public Circle(double radius, String color) {
        this.radius = radius;
        this.color = color;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle extends Shape {
    double length, width;

    public Rectangle(double length, double width, String color) {
        this.length = length;
        this.width = width;
        this.color = color;
    }

    @Override
    public double area() {
        return length * width;
    }
}

// Usage:
Shape s1 = new Circle(5, "Red");
Shape s2 = new Rectangle(4, 6, "Blue");
System.out.println(s1.area());  // 78.539...
System.out.println(s2.area());  // 24.0
```

---

## 3.7 Interfaces vs Abstract Classes

### Interfaces

An **interface** is a contract that a class must follow. It defines **what** a class must do, but not **how**.

```java
public interface Sortable {
    void sort();   // abstract by default
}

public interface Searchable {
    boolean search(int key);
}

// A class can implement MULTIPLE interfaces
public class MyArray implements Sortable, Searchable {
    int[] data;

    @Override
    public void sort() {
        java.util.Arrays.sort(data);
    }

    @Override
    public boolean search(int key) {
        for (int val : data) {
            if (val == key) return true;
        }
        return false;
    }
}
```

### Comparison Table

| Feature | Abstract Class | Interface |
|---------|:--------------:|:---------:|
| Can have constructors | ✅ | ❌ |
| Can have instance fields | ✅ | ❌ (only `static final`) |
| Can have concrete methods | ✅ | ✅ (default methods, Java 8+) |
| Can have abstract methods | ✅ | ✅ |
| Multiple inheritance | ❌ | ✅ (a class can implement many) |
| Instantiable | ❌ | ❌ |
| When to use | Shared state + partial implementation | Pure contract / capability |

### When to Use Which

- **Abstract class**: When classes share **common code** and **state** (fields). Use when there is a true "is-a" relationship (e.g., `Animal` → `Dog`).
- **Interface**: When you want to define a **capability** that unrelated classes can share (e.g., `Comparable`, `Serializable`, `Iterable`).

---

### 📝 Chapter 3 — Practice Problems

| # | Problem | Hint |
|---|---------|------|
| 1 | Create a `Student` class with name, roll, marks. Add method to display | Basic class practice |
| 2 | Implement a `BankAccount` class with deposit/withdraw using encapsulation | Use private fields + validation |
| 3 | Create `Shape` → `Circle`, `Rectangle`, `Triangle` hierarchy | Use abstract class + override `area()` |
| 4 | Demonstrate method overloading with a `Calculator` class | Same method, different params |
| 5 | Demonstrate runtime polymorphism with `Vehicle` → `Car`, `Bike` | Override `drive()` |
| 6 | Create an interface `Playable` with `play()` and `stop()` methods | Implement in `MusicPlayer` and `VideoPlayer` |
| 7 | Build a simple `Stack` class using encapsulation | Private array, public push/pop |
| 8 | Create a `Person` → `Employee` → `Manager` multilevel hierarchy | Use `super()` for constructor chaining |

---

# Chapter 4: Complexity Analysis (Deep Dive)

---

## 4.1 Time Complexity

### Definition

**Time complexity** measures how the **number of operations** an algorithm performs grows as the **input size (n)** increases. It does NOT measure actual wall-clock time — it measures scalability.

### Real-Life Analogy

Imagine you have a list of phone contacts and need to find someone. If you check each name one by one, it takes longer as the list grows — that's **linear** time. If you use alphabetical ordering and jump to the middle each time (like a dictionary), it's much faster — that's **logarithmic** time.

### Why We Care

Two algorithms might both solve the same problem, but one could handle 1 million items while the other chokes at 10,000. Time complexity helps us predict this behavior **before** running the code.

---

## 4.2 Space Complexity

### Definition

**Space complexity** measures how much **extra memory** an algorithm uses relative to the input size. This includes:

1. **Auxiliary space** — extra space used by the algorithm (temporary variables, arrays, call stack).
2. **Input space** — space for storing the input itself.

> Convention: When we say "space complexity," we usually mean **auxiliary space** (extra space beyond the input).

### Example

```java
// Space: O(1) — only uses a fixed number of variables
public static int sum(int[] arr) {
    int total = 0;             // O(1) extra space
    for (int num : arr) {
        total += num;
    }
    return total;
}

// Space: O(n) — creates a new array of size n
public static int[] doubled(int[] arr) {
    int[] result = new int[arr.length];  // O(n) extra space
    for (int i = 0; i < arr.length; i++) {
        result[i] = arr[i] * 2;
    }
    return result;
}
```

---

## 4.3 Big-O Notation

### Definition

**Big-O notation** describes the **upper bound** of an algorithm's growth rate. It tells us the **worst-case** performance as input grows.

### Formal Definition

We say `f(n) = O(g(n))` if there exist constants `c > 0` and `n₀ ≥ 0` such that:

```
f(n) ≤ c · g(n)  for all n ≥ n₀
```

In simple terms: after a certain point, `f(n)` grows **no faster than** a constant multiple of `g(n)`.

### Common Complexities (Sorted Best → Worst)

| Big-O | Name | Example | 1K items | 1M items |
|-------|------|---------|----------|----------|
| O(1) | Constant | Array access by index | 1 | 1 |
| O(log n) | Logarithmic | Binary search | ~10 | ~20 |
| O(n) | Linear | Linear search | 1,000 | 1,000,000 |
| O(n log n) | Linearithmic | Merge sort | ~10,000 | ~20,000,000 |
| O(n²) | Quadratic | Bubble sort | 1,000,000 | 10¹² ❌ |
| O(n³) | Cubic | Matrix multiplication (naive) | 10⁹ ❌ | 10¹⁸ ❌ |
| O(2ⁿ) | Exponential | Recursive fibonacci | 10³⁰⁰ ❌ | ∞ ❌ |
| O(n!) | Factorial | Permutations | ∞ ❌ | ∞ ❌ |

### Rules for Calculating Big-O

1. **Drop constants:** O(2n) → O(n), O(500) → O(1)
2. **Drop lower-order terms:** O(n² + n) → O(n²)
3. **Different inputs → different variables:** O(a + b), not O(2n)
4. **Nested loops multiply:** O(n × m) for two nested loops with different bounds

---

## 4.4 Best / Average / Worst Case

### Definition

For the same algorithm, performance can vary depending on the input:

| Case | Meaning | Example (Linear Search for target in array) |
|------|---------|----------------------------------------------|
| **Best case (Ω)** | Minimum operations | Target is at index 0 → O(1) |
| **Average case (Θ)** | Expected operations on random input | Target is somewhere in the middle → O(n/2) ≈ O(n) |
| **Worst case (O)** | Maximum operations | Target is last or not present → O(n) |

> In interviews, we almost always discuss **worst case** unless stated otherwise.

### Example — Linear Search

```java
public static int linearSearch(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) return i;  // found
    }
    return -1;  // not found
}
// Best: O(1), Average: O(n), Worst: O(n)
```

---

## 4.5 Loop Analysis Patterns

### Pattern 1: Simple Loop → O(n)

```java
for (int i = 0; i < n; i++) {     // runs n times
    System.out.println(i);         // O(1) work
}
// Total: n × O(1) = O(n)
```

### Pattern 2: Nested Loops → O(n²)

```java
for (int i = 0; i < n; i++) {         // n times
    for (int j = 0; j < n; j++) {     // n times each
        System.out.println(i + j);     // O(1) work
    }
}
// Total: n × n = O(n²)
```

### Pattern 3: Nested Loop (Dependent) → O(n²)

```java
for (int i = 0; i < n; i++) {         // n times
    for (int j = 0; j < i; j++) {     // 0 + 1 + 2 + ... + (n-1) times
        System.out.println(i + j);
    }
}
// Total: 0+1+2+...+(n-1) = n(n-1)/2 = O(n²)
```

### Pattern 4: Halving Loop → O(log n)

```java
for (int i = n; i > 0; i /= 2) {     // divides by 2 each time
    System.out.println(i);
}
// How many times? n → n/2 → n/4 → ... → 1
// That's log₂(n) iterations = O(log n)
```

### Pattern 5: Doubling Loop → O(log n)

```java
for (int i = 1; i < n; i *= 2) {     // doubles each time
    System.out.println(i);
}
// 1 → 2 → 4 → 8 → ... → n
// That's log₂(n) iterations = O(log n)
```

### Pattern 6: Loop + Halving = O(n log n)

```java
for (int i = 0; i < n; i++) {         // O(n)
    for (int j = n; j > 0; j /= 2) { // O(log n)
        System.out.println(i + j);
    }
}
// Total: O(n) × O(log n) = O(n log n)
```

### Pattern 7: Two Sequential Loops → O(n)

```java
for (int i = 0; i < n; i++) { ... }  // O(n)
for (int j = 0; j < n; j++) { ... }  // O(n)
// Total: O(n) + O(n) = O(2n) = O(n) — add, don't multiply!
```

### Pattern 8: Two Independent Inputs → O(a + b)

```java
for (int i = 0; i < a; i++) { ... }  // O(a)
for (int j = 0; j < b; j++) { ... }  // O(b)
// Total: O(a + b) — DO NOT simplify to O(n)!
```

### Summary Table

| Pattern | Code Signature | Complexity |
|---------|---------------|------------|
| Simple loop | `for i: 0→n` | O(n) |
| Nested (independent) | `for i * for j` | O(n²) |
| Nested (dependent, 0→i) | `for i * for j:0→i` | O(n²) |
| Halving | `i /= 2` | O(log n) |
| Doubling | `i *= 2` | O(log n) |
| Linear × log | `for i * for j/=2` | O(n log n) |
| Sequential | `loop1; loop2` | O(n + m) = O(max) |
| Nested 3-deep | `for × for × for` | O(n³) |

---

## 4.6 Recursion Tree Analysis

### How to Analyze Recursive Complexity

**Step 1:** Draw the recursion tree (each node = one function call).
**Step 2:** Determine the **depth** (number of levels).
**Step 3:** Count **nodes per level**.
**Step 4:** Calculate **work per node**.
**Step 5:** Sum across all levels.

### Example: Merge Sort

```
mergeSort(n)
├── mergeSort(n/2)
│   ├── mergeSort(n/4) ...
│   └── mergeSort(n/4) ...
└── mergeSort(n/2)
    ├── mergeSort(n/4) ...
    └── mergeSort(n/4) ...

Level 0: 1 call,   total work = n    (merge n elements)
Level 1: 2 calls,  total work = n    (merge n/2 + n/2)
Level 2: 4 calls,  total work = n    (merge n/4 × 4)
...
Level k: 2ᵏ calls, total work = n

Depth = log₂(n) levels
Total = n × log₂(n) = O(n log n)
Space = O(n) (temporary array for merging)
```

### Master Theorem (Simplified)

For recurrences of the form: `T(n) = a · T(n/b) + O(nᶜ)`

| Condition | Complexity |
|-----------|------------|
| `log_b(a) > c` | O(n^(log_b(a))) |
| `log_b(a) = c` | O(nᶜ · log n) |
| `log_b(a) < c` | O(nᶜ) |

**Examples:**
- Binary search: `T(n) = T(n/2) + O(1)` → a=1, b=2, c=0 → log₂(1)=0=c → **O(log n)**
- Merge sort: `T(n) = 2T(n/2) + O(n)` → a=2, b=2, c=1 → log₂(2)=1=c → **O(n log n)**

---

## 4.7 Analyzing Java Code — Step-by-Step

### Example 1: Two Sum (Brute Force)

```java
public static int[] twoSum(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++) {           // O(n)
        for (int j = i + 1; j < nums.length; j++) {   // O(n) worst case
            if (nums[i] + nums[j] == target) {        // O(1)
                return new int[]{i, j};                 // O(1)
            }
        }
    }
    return new int[]{};                                 // O(1)
}
// Time: O(n²)  — nested loops
// Space: O(1)  — only a constant-size output array
```

### Example 2: Checking for Duplicates

```java
// Approach 1: Brute Force
public static boolean hasDuplicateBrute(int[] arr) {
    for (int i = 0; i < arr.length; i++) {
        for (int j = i + 1; j < arr.length; j++) {
            if (arr[i] == arr[j]) return true;
        }
    }
    return false;
}
// Time: O(n²)   Space: O(1)

// Approach 2: Sorting
public static boolean hasDuplicateSort(int[] arr) {
    Arrays.sort(arr);  // O(n log n)
    for (int i = 1; i < arr.length; i++) {
        if (arr[i] == arr[i - 1]) return true;
    }
    return false;
}
// Time: O(n log n)   Space: O(1) or O(n) depending on sort

// Approach 3: HashSet
public static boolean hasDuplicateSet(int[] arr) {
    HashSet<Integer> seen = new HashSet<>();
    for (int num : arr) {
        if (!seen.add(num)) return true;  // add returns false if already present
    }
    return false;
}
// Time: O(n)   Space: O(n)
```

### Complexity Comparison

| Approach | Time | Space | Trade-off |
|----------|------|-------|-----------|
| Brute Force | O(n²) | O(1) | Slow but no extra memory |
| Sorting | O(n log n) | O(1)* | Modifies input array |
| HashSet | O(n) | O(n) | Fastest but uses extra memory |

> This is a classic **time-space trade-off**: you can often make an algorithm faster by using more memory.

---

### Common Mistake

> "O(n/2) is faster than O(n)."

**Both are O(n).** Constants are dropped in Big-O. O(n/2) = O(n). However, in practice, O(n/2) does run about twice as fast, so constants matter for real performance — just not for asymptotic analysis.

---

### 📝 Chapter 4 — Practice Problems

| # | Problem | Expected Complexity |
|---|---------|-------------------|
| 1 | Analyze: `for(i=0; i<n; i++) for(j=i; j<n; j++)` | O(n²) |
| 2 | Analyze: `for(i=1; i<n; i*=3)` | O(log₃ n) |
| 3 | Analyze: `for(i=0;i<n;i++) for(j=0;j<m;j++)` | O(n·m) |
| 4 | Analyze: `T(n) = 2T(n/2) + n²` | O(n²) — Master Thm |
| 5 | Which is faster for large n: O(n log n) or O(n√n)? | O(n log n) |
| 6 | Analyze `fibonacci(n)` recursive | Time: O(2ⁿ), Space: O(n) |
| 7 | What's the complexity of accessing `arr[5]`? | O(1) |
| 8 | Analyze three nested loops over n, m, k | O(n·m·k) |

---

# Chapter 5: Arrays — DSA Foundation

---

## 5.1 Array Basics

### Definition

An **array** is a fixed-size, contiguous block of memory that stores elements of the **same data type**. Each element is accessed by its **index** (starting from 0).

### Real-Life Analogy

An array is like a row of **numbered lockers**. Each locker holds one item, and you find items by their locker number (index). You can instantly go to locker #5 without checking lockers 1-4 — that's **O(1) access**.

### Declaration and Initialization

```java
// Declaration
int[] arr;

// Initialization with size (default values: 0 for int)
int[] arr = new int[5];        // [0, 0, 0, 0, 0]

// Initialization with values
int[] arr = {10, 20, 30, 40, 50};

// Access
System.out.println(arr[0]);    // 10 (first element)
System.out.println(arr[4]);    // 50 (last element)
System.out.println(arr.length); // 5

// Modify
arr[2] = 99;                   // [10, 20, 99, 40, 50]
```

### Array Memory Layout

```
Index:   [0]   [1]   [2]   [3]   [4]
Value:   10    20    30    40    50
Address: 1000  1004  1008  1012  1016   (4 bytes per int)
```

Access is O(1) because: `address = base + (index × element_size)`

### Default Values

| Type | Default |
|------|---------|
| `int[]` | 0 |
| `double[]` | 0.0 |
| `boolean[]` | false |
| `char[]` | '\u0000' |
| `String[]` / Object | null |

### Common Mistake

```java
int[] arr = new int[5];
System.out.println(arr[5]);  // ❌ ArrayIndexOutOfBoundsException!
// Valid indices: 0 to arr.length - 1 (i.e., 0 to 4)
```

---

## 5.2 Traversal and Operations

### Traversal

```java
int[] arr = {1, 2, 3, 4, 5};

// Method 1: for loop
for (int i = 0; i < arr.length; i++) {
    System.out.print(arr[i] + " ");
}

// Method 2: for-each loop
for (int num : arr) {
    System.out.print(num + " ");
}

// Method 3: Reverse traversal
for (int i = arr.length - 1; i >= 0; i--) {
    System.out.print(arr[i] + " ");
}
```

### Common Operations

```java
// 1. Find Maximum
public static int findMax(int[] arr) {
    int max = arr[0];
    for (int i = 1; i < arr.length; i++) {
        if (arr[i] > max) max = arr[i];
    }
    return max;
}
// Time: O(n)  Space: O(1)

// 2. Reverse Array (In-Place)
public static void reverse(int[] arr) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        int temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        left++;
        right--;
    }
}
// Time: O(n)  Space: O(1)

// 3. Linear Search
public static int search(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) return i;
    }
    return -1;
}
// Time: O(n)  Space: O(1)

// 4. Count Occurrences
public static int count(int[] arr, int target) {
    int cnt = 0;
    for (int num : arr) {
        if (num == target) cnt++;
    }
    return cnt;
}
// Time: O(n)  Space: O(1)
```

### 2D Arrays

```java
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// Access: matrix[row][col]
System.out.println(matrix[1][2]);  // 6

// Traverse
for (int i = 0; i < matrix.length; i++) {
    for (int j = 0; j < matrix[i].length; j++) {
        System.out.print(matrix[i][j] + " ");
    }
    System.out.println();
}
```

---

## 5.3 Prefix Sum Arrays

### Definition

A **prefix sum array** stores the cumulative sum from index 0 to each index i. It allows you to compute the **sum of any subarray** in **O(1)** time after an **O(n)** preprocessing step.

### Intuition

Instead of adding up elements every time someone asks "What's the sum from index L to R?", we pre-compute all running totals so we can answer instantly.

### Real-Life Analogy

Imagine a running total on a receipt. If you want to know the cost of items 3 through 7, you subtract the total after item 2 from the total after item 7. No need to re-add each item.

### Construction

```java
int[] arr = {2, 4, 6, 8, 10};
int[] prefix = new int[arr.length];

prefix[0] = arr[0];
for (int i = 1; i < arr.length; i++) {
    prefix[i] = prefix[i - 1] + arr[i];
}
// arr:    [2,  4,  6,  8, 10]
// prefix: [2,  6, 12, 20, 30]
```

### Range Sum Query

```java
// Sum of arr[L..R] = prefix[R] - prefix[L-1]
// Special case: if L == 0, sum = prefix[R]

public static int rangeSum(int[] prefix, int L, int R) {
    if (L == 0) return prefix[R];
    return prefix[R] - prefix[L - 1];
}

// Sum of arr[1..3] = prefix[3] - prefix[0] = 20 - 2 = 18
// Verify: 4 + 6 + 8 = 18 ✅
```

### Complexity

| Operation | Without Prefix Sum | With Prefix Sum |
|-----------|-------------------|----------------|
| Build | — | O(n) (one-time) |
| Range sum query | O(n) per query | O(1) per query |
| Q queries | O(n × Q) | O(n + Q) |

### Solved Problem: Equilibrium Index

Find an index where the sum of elements on the left equals the sum on the right.

```java
public static int equilibriumIndex(int[] arr) {
    int totalSum = 0;
    for (int num : arr) totalSum += num;

    int leftSum = 0;
    for (int i = 0; i < arr.length; i++) {
        int rightSum = totalSum - leftSum - arr[i];
        if (leftSum == rightSum) return i;
        leftSum += arr[i];
    }
    return -1;
}
// Time: O(n)  Space: O(1)
```

---

## 5.4 Suffix Arrays (Suffix Sum)

### Definition

A **suffix sum array** stores the cumulative sum from each index i to the **end** of the array. It's the reverse of prefix sum.

### Construction

```java
int[] arr = {2, 4, 6, 8, 10};
int[] suffix = new int[arr.length];

suffix[arr.length - 1] = arr[arr.length - 1];
for (int i = arr.length - 2; i >= 0; i--) {
    suffix[i] = suffix[i + 1] + arr[i];
}
// arr:    [ 2,  4,  6,  8, 10]
// suffix: [30, 28, 24, 18, 10]
```

### Use Case: Product Except Self

```java
// Given arr, return an array where result[i] = product of all elements except arr[i]
// WITHOUT using division
public static int[] productExceptSelf(int[] arr) {
    int n = arr.length;
    int[] result = new int[n];

    // Left pass: result[i] = product of all elements to the left
    result[0] = 1;
    for (int i = 1; i < n; i++) {
        result[i] = result[i - 1] * arr[i - 1];
    }

    // Right pass: multiply by product of all elements to the right
    int rightProduct = 1;
    for (int i = n - 1; i >= 0; i--) {
        result[i] *= rightProduct;
        rightProduct *= arr[i];
    }
    return result;
}
// Time: O(n)  Space: O(1) (output array not counted)
```

---

## 5.5 Difference Arrays

### Definition

A **difference array** is used to efficiently apply **multiple range update operations** on an array. Instead of updating every element in a range, you mark the start and end, then reconstruct the array in one pass.

### Intuition

If you want to add 5 to every element from index 2 to index 5, instead of visiting 4 elements, you just:
- Add 5 at index 2 (start)
- Subtract 5 at index 6 (one past end)

Then a prefix sum reconstruction gives the final result.

### Implementation

```java
// Add 'val' to all elements from index L to R
public static void rangeUpdate(int[] diff, int L, int R, int val) {
    diff[L] += val;
    if (R + 1 < diff.length) {
        diff[R + 1] -= val;
    }
}

// Reconstruct the actual array from the difference array
public static int[] reconstruct(int[] diff) {
    int[] result = new int[diff.length];
    result[0] = diff[0];
    for (int i = 1; i < diff.length; i++) {
        result[i] = result[i - 1] + diff[i];
    }
    return result;
}

// Example:
int[] diff = new int[6];               // [0, 0, 0, 0, 0, 0]
rangeUpdate(diff, 1, 3, 10);           // Add 10 to indices 1-3
rangeUpdate(diff, 2, 5, 5);            // Add 5  to indices 2-5
int[] result = reconstruct(diff);
// result: [0, 10, 15, 15, 5, 5]
```

### Complexity

| Operation | Brute Force | Difference Array |
|-----------|-------------|-----------------|
| Single range update | O(n) | O(1) |
| Q updates | O(n × Q) | O(Q + n) |

---

## 5.6 Interview Patterns

### Pattern 1: Kadane's Algorithm — Maximum Subarray Sum

```java
public static int maxSubarraySum(int[] arr) {
    int maxSoFar = arr[0];
    int maxEndingHere = arr[0];

    for (int i = 1; i < arr.length; i++) {
        maxEndingHere = Math.max(arr[i], maxEndingHere + arr[i]);
        maxSoFar = Math.max(maxSoFar, maxEndingHere);
    }
    return maxSoFar;
}
// Input:  [-2, 1, -3, 4, -1, 2, 1, -5, 4]
// Output: 6 (subarray [4, -1, 2, 1])
// Time: O(n)   Space: O(1)
```

**Key insight:** At each position, either **extend the current subarray** or **start a new one**. If the running sum becomes negative, it's better to start fresh.

### Pattern 2: Dutch National Flag — Sort 0s, 1s, 2s

```java
public static void sortColors(int[] arr) {
    int low = 0, mid = 0, high = arr.length - 1;

    while (mid <= high) {
        if (arr[mid] == 0) {
            swap(arr, low++, mid++);
        } else if (arr[mid] == 1) {
            mid++;
        } else {  // arr[mid] == 2
            swap(arr, mid, high--);
        }
    }
}

private static void swap(int[] arr, int i, int j) {
    int temp = arr[i]; arr[i] = arr[j]; arr[j] = temp;
}
// Time: O(n)   Space: O(1)
```

### Pattern 3: Binary Search on Sorted Array

```java
public static int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;  // avoids overflow
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;  // not found
}
// Time: O(log n)   Space: O(1)
```

> **Common mistake:** Using `(left + right) / 2` can cause integer overflow for large arrays. Always use `left + (right - left) / 2`.

### Pattern 4: Move Zeroes to End

```java
public static void moveZeroes(int[] arr) {
    int insertPos = 0;
    for (int num : arr) {
        if (num != 0) {
            arr[insertPos++] = num;
        }
    }
    while (insertPos < arr.length) {
        arr[insertPos++] = 0;
    }
}
// Time: O(n)   Space: O(1)
```

### Pattern 5: Rotate Array by K Positions

```java
public static void rotate(int[] arr, int k) {
    int n = arr.length;
    k = k % n;  // handle k > n
    reverse(arr, 0, n - 1);       // reverse entire array
    reverse(arr, 0, k - 1);       // reverse first k elements
    reverse(arr, k, n - 1);       // reverse remaining elements
}

private static void reverse(int[] arr, int start, int end) {
    while (start < end) {
        int temp = arr[start];
        arr[start++] = arr[end];
        arr[end--] = temp;
    }
}
// Input: [1,2,3,4,5,6,7], k=3
// Step 1: [7,6,5,4,3,2,1]
// Step 2: [5,6,7,4,3,2,1]
// Step 3: [5,6,7,1,2,3,4] ✅
// Time: O(n)   Space: O(1)
```

---

### 📝 Chapter 5 — Practice Problems

| # | Problem | Difficulty | Hint |
|---|---------|:----------:|------|
| 1 | Find second largest element | Easy | Track max and secondMax |
| 2 | Check if array is sorted | Easy | Compare adjacent elements |
| 3 | Remove duplicates from sorted array (in-place) | Easy | Two-pointer approach |
| 4 | Maximum subarray sum (Kadane's) | Medium | Extend or restart decision |
| 5 | Find missing number in [0..n] | Easy | Sum formula or XOR |
| 6 | Merge two sorted arrays | Medium | Two pointers from start |
| 7 | Subarray with given sum (positive) | Medium | Sliding window |
| 8 | Trapping rain water | Hard | Prefix max + suffix max |
| 9 | Best time to buy & sell stock | Easy | Track min price, max profit |
| 10 | Next permutation | Hard | Find rightmost ascending pair |

---

# Chapter 6: Problem-Solving Techniques

---

## 6.1 Two Pointer Technique

### Definition

The **two pointer technique** uses two indices (pointers) that traverse the data structure — typically from opposite ends (converging) or same direction (fast/slow) — to solve problems efficiently.

### Intuition

Instead of checking every possible pair O(n²), two pointers intelligently skip unnecessary comparisons by leveraging some property (like sorted order) to move pointers toward each other.

### Real-Life Analogy

Imagine searching for a pair of shoes that fit — one person starts from the smallest sizes, another from the largest. They meet in the middle, eliminating sizes much faster than trying every single one.

### When to Use

- Array is **sorted** (or can be sorted).
- Looking for **pairs** or **triplets** that satisfy a condition.
- Need to compare elements from **both ends**.
- In-place **partitioning** problems.

### Pattern 1: Opposite-End Pointers (Two Sum on Sorted Array)

```java
public static int[] twoSumSorted(int[] arr, int target) {
    int left = 0, right = arr.length - 1;

    while (left < right) {
        int sum = arr[left] + arr[right];
        if (sum == target) {
            return new int[]{left, right};
        } else if (sum < target) {
            left++;    // need a larger sum → move left forward
        } else {
            right--;   // need a smaller sum → move right backward
        }
    }
    return new int[]{-1, -1};  // no pair found
}
// Time: O(n)   Space: O(1)
// Compare: Brute force would be O(n²)
```

**Why it works:** Since the array is sorted, if `sum < target`, increasing `left` gives a larger value. If `sum > target`, decreasing `right` gives a smaller value. We never need to go back.

### Pattern 2: Same-Direction Pointers (Remove Duplicates)

```java
public static int removeDuplicates(int[] arr) {
    if (arr.length == 0) return 0;

    int slow = 0;  // position to place next unique element
    for (int fast = 1; fast < arr.length; fast++) {
        if (arr[fast] != arr[slow]) {
            slow++;
            arr[slow] = arr[fast];
        }
    }
    return slow + 1;  // count of unique elements
}
// Input:  [1, 1, 2, 2, 3, 4, 4]
// Output: 4, arr becomes [1, 2, 3, 4, ...]
// Time: O(n)   Space: O(1)
```

### Pattern 3: Three Pointers (3Sum)

```java
public static List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    Arrays.sort(nums);  // sort first

    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;  // skip duplicates

        int left = i + 1, right = nums.length - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                while (left < right && nums[left] == nums[left + 1]) left++;
                while (left < right && nums[right] == nums[right - 1]) right--;
                left++;
                right--;
            } else if (sum < 0) {
                left++;
            } else {
                right--;
            }
        }
    }
    return result;
}
// Time: O(n²)   Space: O(1) (excluding output)
```

### Pattern 4: Container With Most Water

```java
public static int maxArea(int[] height) {
    int left = 0, right = height.length - 1;
    int maxWater = 0;

    while (left < right) {
        int width = right - left;
        int h = Math.min(height[left], height[right]);
        maxWater = Math.max(maxWater, width * h);

        // Move the shorter line inward (greedy choice)
        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }
    return maxWater;
}
// Time: O(n)   Space: O(1)
```

**Why move the shorter side?** The width decreases each step. The only way to potentially find more water is with a taller line. Moving the taller side can only decrease or maintain height — never improve.

---

## 6.2 Sliding Window Technique

### Definition

The **sliding window technique** maintains a **window** (contiguous subarray/substring) that slides across the data. Instead of recalculating from scratch when the window moves, we **add the new element** and **remove the old one**.

### Intuition

Think of looking through a train window. As the train moves, new scenery appears on one side and old scenery disappears on the other. You don't need to look at the entire landscape each time — just update based on what changes.

### When to Use

- Problem asks for **contiguous subarray/substring**.
- You need to find **maximum/minimum/count** within a window.
- Keywords: "subarray of size k", "smallest subarray with sum ≥ S", "longest substring without repeating".

---

### 6.2.1 Fixed-Size Sliding Window

The window size is **constant** (given as `k`).

**Template:**

```java
// Maximum sum of subarray of size k
public static int maxSumSubarray(int[] arr, int k) {
    // Step 1: Calculate sum of first window
    int windowSum = 0;
    for (int i = 0; i < k; i++) {
        windowSum += arr[i];
    }

    int maxSum = windowSum;

    // Step 2: Slide the window — add right, remove left
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i];       // add new element entering window
        windowSum -= arr[i - k];   // remove element leaving window
        maxSum = Math.max(maxSum, windowSum);
    }

    return maxSum;
}
// Input: [2, 1, 5, 1, 3, 2], k=3
// Windows: [2,1,5]=8, [1,5,1]=7, [5,1,3]=9, [1,3,2]=6
// Output: 9
// Time: O(n)   Space: O(1)
// Compare: Brute force O(n × k) recalculates each window
```

**Solved Problem: First Negative in Every Window of Size K**

```java
public static List<Integer> firstNegative(int[] arr, int k) {
    List<Integer> result = new ArrayList<>();
    Deque<Integer> negatives = new LinkedList<>();  // indices of negatives

    for (int i = 0; i < arr.length; i++) {
        if (arr[i] < 0) negatives.addLast(i);

        // Remove elements outside the current window
        if (!negatives.isEmpty() && negatives.peekFirst() < i - k + 1) {
            negatives.pollFirst();
        }

        // Window is fully formed
        if (i >= k - 1) {
            result.add(negatives.isEmpty() ? 0 : arr[negatives.peekFirst()]);
        }
    }
    return result;
}
// Time: O(n)   Space: O(k)
```

---

### 6.2.2 Variable-Size Sliding Window

The window size **grows and shrinks** based on a condition. These are often the trickier problems.

**Template:**

```java
int left = 0;
int result = 0;  // or Integer.MAX_VALUE, depending on problem

for (int right = 0; right < arr.length; right++) {
    // Step 1: Expand — add arr[right] to the window

    // Step 2: Shrink — while window violates condition
    while (windowIsInvalid()) {
        // remove arr[left] from the window
        left++;
    }

    // Step 3: Update result
    result = Math.max(result, right - left + 1);
}
```

**Solved Problem 1: Smallest Subarray with Sum ≥ S**

```java
public static int minSubarrayLen(int target, int[] arr) {
    int left = 0, windowSum = 0;
    int minLen = Integer.MAX_VALUE;

    for (int right = 0; right < arr.length; right++) {
        windowSum += arr[right];               // expand

        while (windowSum >= target) {          // shrink while valid
            minLen = Math.min(minLen, right - left + 1);
            windowSum -= arr[left++];           // shrink from left
        }
    }

    return minLen == Integer.MAX_VALUE ? 0 : minLen;
}
// Input: target=7, arr=[2, 3, 1, 2, 4, 3]
// Subarray [4, 3] has sum 7's and length 2
// Output: 2
// Time: O(n)   Space: O(1)
```

**Solved Problem 2: Longest Substring Without Repeating Characters**

```java
public static int lengthOfLongestSubstring(String s) {
    Set<Character> window = new HashSet<>();
    int left = 0, maxLen = 0;

    for (int right = 0; right < s.length(); right++) {
        // Shrink window until no duplicate
        while (window.contains(s.charAt(right))) {
            window.remove(s.charAt(left));
            left++;
        }

        window.add(s.charAt(right));           // expand
        maxLen = Math.max(maxLen, right - left + 1);
    }

    return maxLen;
}
// Input: "abcabcbb"
// Output: 3 ("abc")
// Time: O(n)   Space: O(min(n, alphabet_size))
```

**Solved Problem 3: Maximum of All Subarrays of Size K (Using Deque)**

```java
public static int[] maxSlidingWindow(int[] arr, int k) {
    int n = arr.length;
    int[] result = new int[n - k + 1];
    Deque<Integer> deque = new ArrayDeque<>();  // stores indices

    for (int i = 0; i < n; i++) {
        // Remove elements outside the window
        if (!deque.isEmpty() && deque.peekFirst() < i - k + 1) {
            deque.pollFirst();
        }

        // Remove smaller elements from back (they can never be max)
        while (!deque.isEmpty() && arr[deque.peekLast()] <= arr[i]) {
            deque.pollLast();
        }

        deque.addLast(i);

        // Window is fully formed
        if (i >= k - 1) {
            result[i - k + 1] = arr[deque.peekFirst()];
        }
    }
    return result;
}
// Time: O(n)   Space: O(k)
```

---

## 6.3 Pattern Recognition — Choosing the Right Technique

### Decision Framework

```
Is the problem about a contiguous subarray / substring?
├── YES → Sliding Window
│   ├── Fixed size given? → Fixed Sliding Window
│   └── Need to find min/max size? → Variable Sliding Window
│
├── NO → Is the array sorted (or can be sorted)?
│   ├── YES → Two Pointers
│   │   ├── Find pair with sum? → Opposite-end pointers
│   │   ├── Find triplet? → Fix one + two pointers
│   │   └── Partition/remove? → Same-direction pointers
│   └── NO → Consider HashMap, Sorting, or Brute Force
│
└── Is it about ranges or batch updates?
    ├── Range sum queries → Prefix Sum
    └── Range updates → Difference Array
```

### Quick Reference Table

| Problem Type | Technique | Time |
|-------------|-----------|------|
| Pair sum (sorted array) | Two Pointers | O(n) |
| Pair sum (unsorted) | HashMap | O(n) |
| Triplet sum | Sort + Two Pointers | O(n²) |
| Max/min subarray of size k | Fixed Sliding Window | O(n) |
| Longest/shortest subarray with condition | Variable Sliding Window | O(n) |
| Range sum queries | Prefix Sum | O(1) per query |
| Multiple range updates | Difference Array | O(1) per update |
| Remove duplicates (sorted) | Two Pointers | O(n) |
| Container with most water | Two Pointers | O(n) |

---

### 📝 Chapter 6 — Practice Problems

| # | Problem | Technique | Difficulty |
|---|---------|-----------|:----------:|
| 1 | Two Sum (sorted array) | Two Pointers | Easy |
| 2 | Max sum subarray of size K | Fixed Sliding Window | Easy |
| 3 | Longest substring without repeating chars | Variable Sliding Window | Medium |
| 4 | 3Sum (find triplets summing to 0) | Sort + Two Pointers | Medium |
| 5 | Minimum window substring | Variable Sliding Window | Hard |
| 6 | Container with most water | Two Pointers | Medium |
| 7 | Subarray product less than K | Variable Sliding Window | Medium |
| 8 | Trapping rain water | Two Pointers | Hard |
| 9 | Fruits into baskets (max 2 types) | Variable Sliding Window | Medium |
| 10 | Smallest subarray with sum ≥ S | Variable Sliding Window | Medium |

---

# Chapter 7: DSA Readiness

---

## 7.1 How to Approach Problems

### The UMPIRE Framework

Use this 6-step framework for **every** DSA problem:

| Step | Name | Action |
|:----:|------|--------|
| **U** | **Understand** | Read the problem 2-3 times. Identify inputs, outputs, constraints, edge cases. |
| **M** | **Match** | Match the problem to a known pattern (two pointers, sliding window, etc.). |
| **P** | **Plan** | Write pseudocode or step-by-step logic before touching Java. |
| **I** | **Implement** | Translate your plan into clean Java code. |
| **R** | **Review** | Dry run your code with examples. Check edge cases. |
| **E** | **Evaluate** | Analyze time and space complexity. Can you optimize? |

### Step-by-Step Example: "Find pair with target sum in sorted array"

**U (Understand):**
- Input: sorted array `[1, 3, 5, 7, 9]`, target `= 10`
- Output: indices of the pair, e.g., `[1, 3]` (3+7=10)
- Constraint: array is sorted, find one pair

**M (Match):**
- Sorted array + pair sum → Two Pointer technique

**P (Plan):**
```
1. Place left pointer at start, right pointer at end
2. Calculate sum of arr[left] + arr[right]
3. If sum == target → return [left, right]
4. If sum < target → left++
5. If sum > target → right--
6. If pointers cross → no pair found
```

**I (Implement):**
```java
public static int[] findPair(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        int sum = arr[left] + arr[right];
        if (sum == target) return new int[]{left, right};
        else if (sum < target) left++;
        else right--;
    }
    return new int[]{-1, -1};
}
```

**R (Review):**
- Test with `[1,3,5,7,9], target=10` → left=0(1), right=4(9) → 10 ✅
- Test with `[1,2,3], target=7` → no pair → `[-1,-1]` ✅
- Test with `[5], target=10` → single element → `[-1,-1]` ✅

**E (Evaluate):**
- Time: O(n) — each pointer moves at most n times
- Space: O(1) — only two variables

---

## 7.2 Converting Logic to Java

### Common Translations

| Logic | Java Code |
|-------|-----------|
| "For each element" | `for (int i = 0; i < arr.length; i++)` |
| "Track the maximum" | `int max = Integer.MIN_VALUE; max = Math.max(max, val);` |
| "Store seen elements" | `HashSet<Integer> seen = new HashSet<>();` |
| "Count occurrences" | `HashMap<Integer, Integer> map = new HashMap<>();` |
| "Sort the array" | `Arrays.sort(arr);` |
| "Swap two elements" | `int temp = a; a = b; b = temp;` |
| "Check if exists" | `set.contains(key)` or `map.containsKey(key)` |
| "Get or default" | `map.getOrDefault(key, 0)` |

### Java Collections Cheat Sheet for DSA

| Data Structure | Java Class | Key Operations | When to Use |
|---------------|------------|---------------|-------------|
| Dynamic array | `ArrayList<T>` | `add`, `get`, `remove` O(1)/O(n) | When size changes |
| Stack | `Stack<T>` or `Deque<T>` | `push`, `pop`, `peek` O(1) | LIFO, matching brackets |
| Queue | `LinkedList<T>` or `ArrayDeque<T>` | `offer`, `poll`, `peek` O(1) | FIFO, BFS |
| Hash set | `HashSet<T>` | `add`, `contains`, `remove` O(1) | Fast lookups, uniqueness |
| Hash map | `HashMap<K,V>` | `put`, `get`, `containsKey` O(1) | Key-value pairs, counting |
| Sorted set | `TreeSet<T>` | `add`, `first`, `last` O(log n) | Sorted unique elements |
| Sorted map | `TreeMap<K,V>` | `put`, `firstKey`, `lastKey` O(log n) | Sorted keys |
| Priority queue | `PriorityQueue<T>` | `offer`, `poll` O(log n) | Min/max heap, top-K |

### HashMap — Frequency Counter Pattern

```java
// Count frequency of each element
public static Map<Integer, Integer> frequency(int[] arr) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int num : arr) {
        map.put(num, map.getOrDefault(num, 0) + 1);
    }
    return map;
}
// Time: O(n)   Space: O(n)
```

---

## 7.3 Debugging Strategies

### The 5-Point Debugging Checklist

1. **Read the error message carefully** — Java error messages tell you the exact file, line number, and exception type.
2. **Dry run with a simple example** — trace your code line by line on paper.
3. **Check boundary conditions** — empty arrays, single elements, all-same values.
4. **Print intermediate values** — add `System.out.println()` to see variable states.
5. **Check off-by-one errors** — most bugs! Is it `< n` or `<= n`? Is it `i - 1` or `i`?

### Common Java Runtime Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `ArrayIndexOutOfBoundsException` | Accessing index ≥ length or < 0 | Check loop bounds |
| `NullPointerException` | Calling method on `null` | Add null checks |
| `StackOverflowError` | Infinite recursion | Verify base case |
| `ArithmeticException` | Division by zero | Check denominator |
| `ConcurrentModificationException` | Modifying collection while iterating | Use iterator's `remove()` |

### Debugging Example

```java
// Bug: This should return the index of the target, but returns -1
public static int search(int[] arr, int target) {
    for (int i = 0; i <= arr.length; i++) {  // ❌ BUG: <= should be <
        if (arr[i] == target) return i;
    }
    return -1;
}
// When i == arr.length → ArrayIndexOutOfBoundsException!
// Fix: change i <= arr.length to i < arr.length
```

---

## 7.4 Optimization Mindset

### The Optimization Ladder

Always start with the **brute force**, then optimize step by step:

```
Level 1: Brute Force     → O(n²) or O(n³)
   ↓  Can I sort first?
Level 2: Sort + Optimize  → O(n log n)
   ↓  Can I use a hash table?
Level 3: Hash Map/Set     → O(n)
   ↓  Can I use two pointers or sliding window?
Level 4: Optimal          → O(n) or O(log n)
```

### Optimization Techniques Summary

| Technique | What It Replaces | Time Improvement |
|-----------|-----------------|------------------|
| Sort + Two Pointers | Nested loops for pairs | O(n²) → O(n log n) |
| HashMap | Nested loops for lookups | O(n²) → O(n) |
| Binary Search | Linear search on sorted data | O(n) → O(log n) |
| Prefix Sum | Repeated range sum queries | O(n·Q) → O(n + Q) |
| Sliding Window | Recalculating subarray metrics | O(n·k) → O(n) |
| Difference Array | Repeated range updates | O(n·Q) → O(Q + n) |

### The "Can I do better?" Checklist

1. Am I doing **redundant work**? (Recalculating something I already know?)
2. Am I using the right **data structure**? (Array vs HashMap vs Set?)
3. Can I **preprocess** the data? (Sort, prefix sum, frequency map?)
4. Am I exploring **all possibilities** when I don't need to? (Can I prune/skip?)
5. Is there a **mathematical formula** instead of a loop? (Sum of n = n(n+1)/2?)

---

## 7.5 Common Beginner Mistakes

### Mistake 1: Comparing Strings with `==`

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);       // ❌ false (compares references)
System.out.println(a.equals(b));  // ✅ true (compares content)
```

### Mistake 2: Integer Overflow

```java
int a = Integer.MAX_VALUE;  // 2,147,483,647
int b = a + 1;              // ❌ -2,147,483,648 (overflow!)

// Fix: use long
long c = (long) a + 1;      // ✅ 2,147,483,648

// Common in: computing mid index, sum of large arrays, factorial
int mid = (left + right) / 2;           // ❌ can overflow
int mid = left + (right - left) / 2;    // ✅ safe
```

### Mistake 3: Modifying Collection While Iterating

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));

// ❌ ConcurrentModificationException
for (int num : list) {
    if (num % 2 == 0) list.remove(Integer.valueOf(num));
}

// ✅ Use Iterator
Iterator<Integer> it = list.iterator();
while (it.hasNext()) {
    if (it.next() % 2 == 0) it.remove();
}
```

### Mistake 4: Not Handling Edge Cases

Always consider:
- **Empty input** — `arr.length == 0`, `str.isEmpty()`
- **Single element** — `arr.length == 1`
- **All same values** — `[5, 5, 5, 5]`
- **Already sorted** or **reverse sorted**
- **Negative numbers**
- **Very large values** (overflow risk)

### Mistake 5: Off-by-One Errors in Loops

```java
// ❌ Accessing arr[n] when only indices 0 to n-1 exist
for (int i = 0; i <= arr.length; i++) { ... }

// ✅ Correct
for (int i = 0; i < arr.length; i++) { ... }

// ❌ Binary search: wrong mid calculation or boundary
int mid = (left + right) / 2;  // overflow risk
while (left < right) { ... }   // may skip element at left == right

// ✅ Correct
int mid = left + (right - left) / 2;
while (left <= right) { ... }
```

### Mistake 6: Returning Early Without Checking All Cases

```java
// ❌ Returns on first pair, misses 'no pair' case
public static boolean hasPairWithSum(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        for (int j = i+1; j < arr.length; j++) {
            if (arr[i] + arr[j] == target) return true;
            // What if we add: else return false; ← BUG!
            // This would return false after checking just the first pair
        }
    }
    return false;  // ✅ Only return false AFTER checking ALL pairs
}
```

---

### 📝 Chapter 7 — Final Practice Exercises

| # | Exercise | Purpose |
|---|----------|---------|
| 1 | Solve "Two Sum" three ways: brute force → sort → HashMap | Practice optimization ladder |
| 2 | Debug a given buggy binary search implementation | Practice debugging |
| 3 | Identify time complexity of 5 given code snippets | Practice analysis |
| 4 | Convert pseudocode for "merge two sorted arrays" to Java | Practice logic → code |
| 5 | Write edge case test inputs for "find max subarray sum" | Practice edge case thinking |

---

# Appendix: Complexity Cheat Sheet

| Algorithm / Operation | Time (Average) | Time (Worst) | Space |
|----------------------|:--------------:|:------------:|:-----:|
| Array access by index | O(1) | O(1) | — |
| Linear search | O(n) | O(n) | O(1) |
| Binary search | O(log n) | O(log n) | O(1) |
| Bubble sort | O(n²) | O(n²) | O(1) |
| Selection sort | O(n²) | O(n²) | O(1) |
| Insertion sort | O(n²) | O(n²) | O(1) |
| Merge sort | O(n log n) | O(n log n) | O(n) |
| Quick sort | O(n log n) | O(n²) | O(log n) |
| HashMap get/put | O(1) | O(n) | O(n) |
| HashSet add/contains | O(1) | O(n) | O(n) |
| TreeMap get/put | O(log n) | O(log n) | O(n) |
| PriorityQueue offer/poll | O(log n) | O(log n) | O(n) |
| Arrays.sort() (primitives) | O(n log n) | O(n log n) | O(log n) |
| Arrays.sort() (objects) | O(n log n) | O(n log n) | O(n) |

---

# Appendix: Java DSA Starter Template

```java
import java.util.*;

public class Solution {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        // Read input
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) {
            arr[i] = sc.nextInt();
        }

        // Solve and print result
        System.out.println(solve(arr, n));
        sc.close();
    }

    public static int solve(int[] arr, int n) {
        // Your solution here
        return 0;
    }
}
```

---

> **End of Handbook**
>
> This handbook covers the complete journey from Java basics to DSA readiness. Practice each topic thoroughly, solve the recommended problems, and always analyze complexity before submitting solutions. Happy coding! 🚀

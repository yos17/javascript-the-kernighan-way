# Chapter 3: Control Flow

The order in which statements are executed is called *control flow*. Without it, every program would run the same sequence of instructions every time. Control flow lets a program make decisions, repeat operations, and exit early. This chapter covers all the mechanisms JavaScript provides.

---

## 3.1 Statements and Blocks

A *statement* is a complete instruction. In JavaScript, statements are terminated by semicolons (which are often optional due to automatic semicolon insertion, but should be written explicitly).

A *block* is a sequence of statements enclosed in braces:

```js
{
    let x = 1;
    let y = 2;
    console.log(x + y);
}
```

Blocks are used wherever a single statement is expected but you want to group multiple statements. Variables declared with `let` and `const` inside a block are local to that block.

---

## 3.2 if-else

The `if` statement executes a block conditionally:

```js
if (condition) {
    // executed when condition is truthy
}
```

The `else` clause runs when the condition is falsy:

```js
if (x > 0) {
    console.log("positive");
} else if (x < 0) {
    console.log("negative");
} else {
    console.log("zero");
}
```

The `else if` is not a distinct construct — it is simply an `else` followed by another `if`. The chain can be as long as needed.

Always use braces, even for single-statement bodies. Omitting them is a common source of bugs:

```js
// Don't do this:
if (x > 0)
    console.log("positive");
    console.log("always runs");   // not part of the if!

// Do this:
if (x > 0) {
    console.log("positive");
}
console.log("always runs");
```

---

## 3.3 The Ternary Operator

The conditional expression `? :` is a compact if-else for expressions:

```js
const abs = x >= 0 ? x : -x;
const grade = score >= 60 ? "pass" : "fail";
```

Use it for simple two-way choices that produce a value. Do not nest it; nested ternaries are difficult to read.

---

## 3.4 switch

When a single expression is compared against many constant values, `switch` can be clearer than a chain of `else if`:

```js
switch (day) {
    case "Monday":
    case "Tuesday":
    case "Wednesday":
    case "Thursday":
    case "Friday":
        console.log("weekday");
        break;
    case "Saturday":
    case "Sunday":
        console.log("weekend");
        break;
    default:
        console.log("unknown day");
}
```

Each `case` clause falls through to the next unless terminated by `break`. This is usually undesirable; always include `break` at the end of each `case`. The `default` clause runs if no `case` matches.

`switch` uses strict equality (`===`) for comparisons.

---

## 3.5 while

The `while` loop executes its body repeatedly as long as the condition is truthy:

```js
let n = 1;
while (n < 1000) {
    n *= 2;
}
console.log(n);   // 1024
```

The condition is tested before each iteration. If it is false initially, the body never runs.

---

## 3.6 do-while

The `do-while` loop tests the condition *after* each iteration, so the body always runs at least once:

```js
let input;
do {
    input = prompt("Enter a positive number:");
} while (input <= 0);
```

This pattern is useful when you must execute the body at least once before you have a value to check.

---

## 3.7 for

The `for` loop is the workhorse of iteration:

```js
for (initialization; condition; update) {
    body;
}
```

Which is equivalent to:

```js
initialization;
while (condition) {
    body;
    update;
}
```

The three parts of the `for` header may contain any expressions, or may be omitted entirely:

```js
// Infinite loop (requires break or return to exit)
for (;;) { ... }

// Looping backward
for (let i = arr.length - 1; i >= 0; i--) { ... }

// Multiple variables
for (let i = 0, j = arr.length - 1; i < j; i++, j--) { ... }
```

---

## 3.8 for-of

`for...of` iterates over the *values* of any iterable (arrays, strings, sets, maps, generators):

```js
const fruits = ["apple", "banana", "cherry"];
for (const fruit of fruits) {
    console.log(fruit);
}

for (const ch of "hello") {
    console.log(ch);
}
```

To get both index and value from an array, use `entries()`:

```js
for (const [i, val] of fruits.entries()) {
    console.log(`${i}: ${val}`);
}
```

---

## 3.9 for-in

`for...in` iterates over the *enumerable property keys* of an object:

```js
const obj = { a: 1, b: 2, c: 3 };
for (const key in obj) {
    console.log(`${key}: ${obj[key]}`);
}
```

Do not use `for...in` to iterate over arrays; use `for...of` or a numeric `for` loop. `for...in` iterates over all enumerable properties, including inherited ones, which can produce surprising results with arrays.

---

## 3.10 break and continue

`break` exits the innermost enclosing loop or `switch`:

```js
for (let i = 0; i < 100; i++) {
    if (i * i > 50) {
        console.log(`First integer whose square exceeds 50: ${i}`);
        break;
    }
}
```

`continue` skips the rest of the current iteration and begins the next:

```js
for (let i = 0; i < 20; i++) {
    if (i % 2 === 0) continue;   // skip even numbers
    console.log(i);
}
```

Labels allow `break` and `continue` to target an outer loop:

```js
outer:
for (let i = 0; i < 5; i++) {
    for (let j = 0; j < 5; j++) {
        if (j === 2) break outer;
        console.log(i, j);
    }
}
```

Labels are rare and should be avoided except when breaking nested loops more clearly.

---

## 3.11 Exceptions: try-catch-finally

JavaScript uses exceptions to handle runtime errors:

```js
try {
    const result = JSON.parse(userInput);
    console.log(result);
} catch (error) {
    console.log("Invalid JSON:", error.message);
} finally {
    console.log("This always runs.");
}
```

The `try` block contains code that might throw. The `catch` block handles the error. The `finally` block runs regardless of whether an error occurred — useful for cleanup.

To throw your own errors:

```js
function divide(a, b) {
    if (b === 0) throw new Error("Division by zero");
    return a / b;
}
```

Throw `Error` objects, not strings or other values. `Error` objects carry a `message` property and a stack trace.

---

## 3.12 A Worked Example: Binary Search

Binary search finds a target value in a sorted array in O(log n) time by repeatedly halving the search interval:

```js
function binarySearch(arr, target) {
    let low = 0;
    let high = arr.length - 1;

    while (low <= high) {
        const mid = Math.floor((low + high) / 2);
        if (arr[mid] === target) {
            return mid;           // found
        } else if (arr[mid] < target) {
            low = mid + 1;        // target is in right half
        } else {
            high = mid - 1;       // target is in left half
        }
    }
    return -1;                    // not found
}
```

This function uses a `while` loop, `if-else if-else`, arithmetic, and `return`. It is a canonical example of control flow in service of an algorithm.

---

## Exercises

**3-1.** Rewrite the FizzBuzz loop from Chapter 1 using `switch` instead of `if-else`. Which is clearer?

**3-2.** Write a function `collatz(n)` that applies the Collatz sequence: if n is even, divide by 2; if odd, multiply by 3 and add 1. Return the number of steps required to reach 1. What is the longest sequence for n < 1000?

**3-3.** Write a function `bubbleSort(arr)` that sorts an array in place using the bubble sort algorithm. The outer loop should stop early if no swaps were made in an inner pass.

**3-4.** Binary search assumes a sorted array. What happens if the array is not sorted? Write a test that reveals the failure.

**3-5.** Write a function `matrixMultiply(A, B)` that multiplies two 2D arrays (matrices). Use nested `for` loops. What conditions must A and B satisfy?

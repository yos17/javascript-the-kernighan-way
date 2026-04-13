# Chapter 4: Functions

Functions are the basic unit of program structure in JavaScript. A function encapsulates a computation, gives it a name, and makes it reusable. Programs are built by writing functions that call other functions; the art of programming is largely the art of decomposing problems into functions of the right size.

---

## 4.1 Defining Functions

There are three ways to define a function:

**Function declaration:**

```js
function add(a, b) {
    return a + b;
}
```

**Function expression:**

```js
const add = function(a, b) {
    return a + b;
};
```

**Arrow function:**

```js
const add = (a, b) => a + b;
```

All three create a function that adds two numbers. The differences are subtle but important.

Function declarations are *hoisted*: the function exists throughout its enclosing scope, even before the declaration line. Function expressions and arrow functions are not hoisted.

```js
console.log(square(5));    // 25 — works, declaration is hoisted
function square(x) { return x * x; }

console.log(cube(5));      // ReferenceError — not hoisted
const cube = x => x * x * x;
```

Use function declarations for top-level named functions. Use arrow functions for callbacks and short expressions. Avoid anonymous function expressions.

---

## 4.2 Parameters and Arguments

Parameters are the names listed in the function definition. Arguments are the values passed in the call. JavaScript does not enforce that the number of arguments matches the number of parameters.

```js
function f(a, b, c) {
    console.log(a, b, c);
}

f(1, 2, 3);    // 1 2 3
f(1, 2);       // 1 2 undefined
f(1, 2, 3, 4); // 1 2 3  (extra argument ignored)
```

**Default parameters** provide a value when an argument is omitted or `undefined`:

```js
function greet(name, greeting = "Hello") {
    return `${greeting}, ${name}!`;
}

greet("Alice");          // "Hello, Alice!"
greet("Alice", "Hi");    // "Hi, Alice!"
```

**Rest parameters** collect remaining arguments into an array:

```js
function sum(...numbers) {
    let total = 0;
    for (const n of numbers) total += n;
    return total;
}

sum(1, 2, 3, 4, 5);   // 15
```

---

## 4.3 Return Values

A function returns when it encounters a `return` statement, or when execution falls off the end. A function with no `return` statement returns `undefined`.

```js
function sign(x) {
    if (x > 0) return 1;
    if (x < 0) return -1;
    return 0;
}
```

Multiple return points are fine when they make the logic clearer. Forcing a single return point often produces needless complexity.

Functions can return any value — including other functions:

```js
function makeMultiplier(factor) {
    return function(x) {
        return x * factor;
    };
}

const double = makeMultiplier(2);
const triple = makeMultiplier(3);
console.log(double(5));   // 10
console.log(triple(5));   // 15
```

---

## 4.4 Scope

A variable declared inside a function is *local* to that function. It does not exist outside:

```js
function f() {
    const x = 10;
    console.log(x);   // 10
}

f();
console.log(x);       // ReferenceError: x is not defined
```

A function can read variables from its enclosing scope — this is how closures work (Chapter 6). But it cannot create variables in the outer scope simply by assigning to them (unless they were declared there).

Variables declared with `let` and `const` are *block-scoped*: they exist only within the `{}` that encloses them. Variables declared with `var` are *function-scoped* — they exist throughout the enclosing function regardless of the block in which they were declared. Avoid `var`.

---

## 4.5 Recursion

A function may call itself. This is called *recursion*, and it is a natural way to express problems that have self-similar structure.

The factorial function: n! = n × (n-1) × (n-2) × ... × 1:

```js
function factorial(n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

factorial(5);   // 120
```

Every recursive function needs a *base case* — a condition that stops the recursion. Without it, the function calls itself forever until the stack overflows.

The Fibonacci sequence: each number is the sum of the two before it.

```js
function fib(n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```

This is correct but inefficient for large `n` — it recomputes the same values repeatedly. We can fix this with *memoization*:

```js
function makeFib() {
    const cache = new Map();
    return function fib(n) {
        if (n <= 1) return n;
        if (cache.has(n)) return cache.get(n);
        const result = fib(n - 1) + fib(n - 2);
        cache.set(n, result);
        return result;
    };
}

const fib = makeFib();
fib(50);   // fast
```

---

## 4.6 Higher-Order Functions

A *higher-order function* is one that takes a function as an argument, or returns a function, or both.

The three most important higher-order functions for arrays are `map`, `filter`, and `reduce`.

**map** transforms each element:

```js
const nums = [1, 2, 3, 4, 5];
const squares = nums.map(x => x * x);
// [1, 4, 9, 16, 25]
```

**filter** keeps elements that pass a test:

```js
const evens = nums.filter(x => x % 2 === 0);
// [2, 4]
```

**reduce** combines elements into a single value:

```js
const sum = nums.reduce((acc, x) => acc + x, 0);
// 15

const product = nums.reduce((acc, x) => acc * x, 1);
// 120
```

These three functions replace most explicit loops. A pipeline of `map`, `filter`, and `reduce` often expresses the intent of a computation more clearly than an equivalent `for` loop.

---

## 4.7 Function Composition

Functions can be combined to build more complex behavior:

```js
const compose = (f, g) => x => f(g(x));

const double = x => x * 2;
const addOne = x => x + 1;

const doubleThenAddOne = compose(addOne, double);
console.log(doubleThenAddOne(5));   // 11
```

`compose` applies `g` first, then `f`. This mirrors mathematical notation: `(f ∘ g)(x) = f(g(x))`.

A more general compose handles any number of functions:

```js
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);

const process = pipe(
    x => x * 2,
    x => x + 10,
    x => x.toString(),
    s => `Result: ${s}`
);

console.log(process(5));   // "Result: 20"
```

`pipe` is `compose` in left-to-right order, which is often more readable.

---

## 4.8 Closures

A *closure* is a function that retains access to the variables of its enclosing scope, even after that scope has finished executing.

```js
function makeCounter(start = 0) {
    let count = start;
    return {
        increment() { return ++count; },
        decrement() { return --count; },
        reset()     { count = start; return count; },
        value()     { return count; }
    };
}

const counter = makeCounter(10);
counter.increment();   // 11
counter.increment();   // 12
counter.decrement();   // 11
counter.value();       // 11
counter.reset();       // 10
```

`count` is private to the closure — it cannot be accessed or modified from outside except through the returned methods. This is encapsulation without classes.

---

## 4.9 The Call Stack

When a function is called, a *stack frame* is pushed onto the call stack. When it returns, the frame is popped. The stack holds the return address and local variables for each active call.

Deeply recursive functions can exhaust the stack, causing a "Maximum call stack size exceeded" error. JavaScript does not (generally) optimize tail calls, so loops are preferable to recursion for large inputs.

The stack is visible in browser devtools. When an error occurs, the stack trace shows the sequence of function calls that led to it.

---

## 4.10 A Worked Example: Sorting with a Comparator

`Array.prototype.sort` takes an optional comparator function. The comparator receives two elements and returns:
- a negative number if the first should come first,
- a positive number if the second should come first,
- zero if they are equal.

```js
const people = [
    { name: "Alice", age: 30 },
    { name: "Bob",   age: 25 },
    { name: "Carol", age: 35 },
];

// Sort by age, ascending
people.sort((a, b) => a.age - b.age);

// Sort by name, alphabetically
people.sort((a, b) => a.name.localeCompare(b.name));

// Sort by age descending, then by name ascending
people.sort((a, b) => b.age - a.age || a.name.localeCompare(b.name));
```

The comparator is a function passed to a higher-order function. This pattern — parameterizing behavior with a function — is one of the most powerful ideas in programming.

---

## Exercises

**4-1.** Write a function `memoize(fn)` that takes any single-argument function and returns a memoized version of it. Test it by memoizing the naive Fibonacci function.

**4-2.** Write `map`, `filter`, and `reduce` from scratch — do not use the built-in array methods. Test each against a known input.

**4-3.** Write a function `curry(fn)` that converts a two-argument function into a curried function: `curry(add)(2)(3)` should return `5`.

**4-4.** Write a function `once(fn)` that returns a new function that calls `fn` at most once; subsequent calls return the result of the first call. Useful for initialization.

**4-5.** The Tower of Hanoi: three pegs, n disks. Move all disks from peg A to peg C using peg B, never placing a larger disk on a smaller one. Write a recursive function `hanoi(n, from, to, via)` that prints the required moves. How many moves does it take for n disks?

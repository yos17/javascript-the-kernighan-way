# Chapter 1: A Tutorial Introduction

The best way to learn a new language is to write programs in it. This chapter introduces the essential elements of JavaScript through a sequence of programs, each slightly more capable than the last. We will not explain everything — that comes later. For now, the goal is to see JavaScript doing useful things as quickly as possible.

Open `index.html` in your browser. Open the browser's developer console (`F12` or `Cmd+Option+I`). Type along.

---

## 1.1 Hello, World

Every introduction must begin here:

```js
console.log("hello, world");
```

`console.log` prints its arguments to the console. The string `"hello, world"` is its argument. The semicolon ends the statement. This is the smallest complete JavaScript program that does anything visible.

Run it. See it print. Move on.

---

## 1.2 Variables

```js
let message = "hello, world";
console.log(message);
```

`let` declares a variable. The `=` assigns a value to it. Variables can be reassigned:

```js
let x = 10;
x = 20;
console.log(x);   // 20
```

Use `const` when the value will not change:

```js
const PI = 3.14159;
console.log(PI);
```

Attempting to reassign a `const` is an error. Use `const` by default; use `let` when you need to reassign.

---

## 1.3 Arithmetic

JavaScript does arithmetic in the expected way:

```js
console.log(2 + 3);    // 5
console.log(10 - 4);   // 6
console.log(3 * 7);    // 21
console.log(22 / 7);   // 3.142857142857143
console.log(22 % 7);   // 1  (remainder)
console.log(2 ** 10);  // 1024 (exponentiation)
```

The `%` operator gives the remainder after integer division. `2 ** 10` is 2 raised to the 10th power.

---

## 1.4 A Simple Loop

To print the numbers from 1 to 10:

```js
for (let i = 1; i <= 10; i++) {
    console.log(i);
}
```

The `for` statement has three parts separated by semicolons: an initialization (`let i = 1`), a condition (`i <= 10`), and an increment (`i++`). The loop runs as long as the condition is true.

`i++` is shorthand for `i = i + 1`.

To compute the sum of integers from 1 to 100:

```js
let sum = 0;
for (let i = 1; i <= 100; i++) {
    sum += i;
}
console.log(sum);   // 5050
```

Gauss figured this out as a child. The computer needs a loop.

---

## 1.5 Functions

A function is a named, reusable block of code:

```js
function square(x) {
    return x * x;
}

console.log(square(5));    // 25
console.log(square(12));   // 144
```

`function` is the keyword. `square` is the name. `x` is the parameter. `return` sends a value back to the caller.

Functions can call other functions:

```js
function sumOfSquares(a, b) {
    return square(a) + square(b);
}

console.log(sumOfSquares(3, 4));   // 25
```

---

## 1.6 Strings

Strings hold text:

```js
let name = "world";
console.log("hello, " + name);   // hello, world
```

The `+` operator joins strings. This is called *concatenation*.

More useful is the template literal, which embeds expressions inside backtick strings:

```js
let name = "world";
console.log(`hello, ${name}`);   // hello, world

let x = 6;
let y = 7;
console.log(`${x} times ${y} is ${x * y}`);   // 6 times 7 is 42
```

`${...}` can contain any expression. The result is converted to a string and spliced in.

Strings have a `length` property and many useful methods:

```js
let s = "hello";
console.log(s.length);           // 5
console.log(s.toUpperCase());    // HELLO
console.log(s.includes("ell")); // true
console.log(s[0]);               // h
```

---

## 1.7 Arrays

An array holds a sequence of values:

```js
let primes = [2, 3, 5, 7, 11, 13];
console.log(primes[0]);    // 2
console.log(primes[5]);    // 13
console.log(primes.length);   // 6
```

Arrays are zero-indexed. The first element is at index 0.

To iterate over every element:

```js
for (let i = 0; i < primes.length; i++) {
    console.log(primes[i]);
}
```

Or more simply:

```js
for (const p of primes) {
    console.log(p);
}
```

`for...of` iterates over the values of any iterable. Prefer it when you don't need the index.

---

## 1.8 A Complete Program: FizzBuzz

The canonical programmer interview problem: print numbers from 1 to 100, but print "Fizz" for multiples of 3, "Buzz" for multiples of 5, and "FizzBuzz" for multiples of both.

```js
for (let i = 1; i <= 100; i++) {
    if (i % 15 === 0) {
        console.log("FizzBuzz");
    } else if (i % 3 === 0) {
        console.log("Fizz");
    } else if (i % 5 === 0) {
        console.log("Buzz");
    } else {
        console.log(i);
    }
}
```

Notice that we test `i % 15 === 0` first. If we tested `i % 3 === 0` first and printed "Fizz" on that branch, we would never reach the FizzBuzz case.

`===` is the strict equality operator. Use it instead of `==`. The difference will be explained in Chapter 2; for now, use `===` always.

---

## 1.9 The DOM: Talking to the Page

JavaScript was invented to manipulate web pages. The **Document Object Model** (DOM) is the interface between JavaScript and HTML. Every element on the page is a node in a tree, and JavaScript can read and change any of it.

```js
// Find an element by its id
const el = document.getElementById("output");

// Change its text
el.textContent = "hello, world";

// Change its style
el.style.color = "tomato";
```

This is the foundation of interactive web pages. We will use it throughout the book.

---

## 1.10 A First Interactive Program: Celsius to Fahrenheit

The companion `index.html` file for this chapter shows a complete interactive program: enter a temperature in Celsius, see Fahrenheit. It uses everything from this chapter: variables, arithmetic, functions, and a touch of DOM manipulation.

The conversion formula is `F = (C × 9/5) + 32`.

```js
function celsiusToFahrenheit(c) {
    return (c * 9 / 5) + 32;
}
```

Open `index.html` and study how each piece connects.

---

## Exercises

**1-1.** Modify the loop from 1.4 to print only even numbers from 2 to 20.

**1-2.** Write a function `cube(x)` that returns the cube of its argument. Write a second function `sumOfCubes(a, b)` that uses it.

**1-3.** FizzBuzz without the `i % 15` shortcut: can you write a correct version using only `i % 3` and `i % 5`? Is it clearer or less clear?

**1-4.** The formula for compound interest is `A = P(1 + r/n)^(nt)`. Write a function `compoundInterest(P, r, n, t)` and print a table showing the growth of $1000 at 5% interest compounded monthly for 10 years.

**1-5.** Write a function `isPrime(n)` that returns `true` if `n` is prime, `false` otherwise. Use it to print all primes less than 100.

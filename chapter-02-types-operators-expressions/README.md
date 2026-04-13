# Chapter 2: Types, Operators, and Expressions

Variables and constants are the basic objects manipulated in a program. Operators specify what is done to them. Expressions combine them to produce new values. This chapter covers all the building blocks that programs are made from, carefully and completely.

---

## 2.1 Variable Names

Variables are named with letters, digits, underscores, and dollar signs. They must begin with a letter, underscore, or dollar sign. Case matters: `count`, `Count`, and `COUNT` are three different variables.

By convention:
- Variables and functions: `camelCase`
- Constants that are truly fixed: `UPPER_SNAKE_CASE`
- Classes: `PascalCase`

Choose names that are descriptive. `elapsedTimeSeconds` is better than `ets`. `i`, `j`, `n` are acceptable for loop counters and short-lived numeric variables by long-standing convention.

---

## 2.2 Data Types

JavaScript has seven primitive types:

| Type | Example | Notes |
|------|---------|-------|
| `number` | `42`, `3.14`, `-7` | All numeric values, including integers |
| `string` | `"hello"`, `'world'`, `` `hi` `` | Text |
| `boolean` | `true`, `false` | Logical values |
| `undefined` | `undefined` | Declared but not assigned |
| `null` | `null` | Intentional absence of value |
| `bigint` | `9007199254740993n` | Arbitrarily large integers |
| `symbol` | `Symbol("id")` | Unique identifiers |

And one compound type: **object** — arrays, functions, dates, regular expressions, and plain objects are all objects.

```js
typeof 42          // "number"
typeof "hello"     // "string"
typeof true        // "boolean"
typeof undefined   // "undefined"
typeof null        // "object"   ← historical bug, not fixable
typeof []          // "object"
typeof function(){} // "function"
```

---

## 2.3 Numbers

JavaScript uses IEEE 754 double-precision floating-point for all numbers. This means integers up to 2^53 − 1 are represented exactly, and most decimal fractions are approximated.

```js
console.log(0.1 + 0.2);         // 0.30000000000000004
console.log(0.1 + 0.2 === 0.3); // false
```

This surprises programmers. The remedy when exact decimal arithmetic matters (money, for example) is to work in integers (cents, not dollars).

Special numeric values:

```js
Infinity           // larger than any number
-Infinity          // smaller than any number
NaN                // Not a Number — result of invalid operations
Number.MAX_SAFE_INTEGER   // 9007199254740991
Number.EPSILON     // 2.220446049250313e-16
```

`NaN` is the only value not equal to itself:

```js
NaN === NaN   // false
isNaN(NaN)    // true
```

Number literals may be written in several bases:

```js
255     // decimal
0xff    // hexadecimal (255)
0o377   // octal (255)
0b11111111  // binary (255)
```

The `Math` object provides mathematical functions:

```js
Math.floor(3.7)     // 3
Math.ceil(3.2)      // 4
Math.round(3.5)     // 4
Math.abs(-5)        // 5
Math.sqrt(16)       // 4
Math.pow(2, 10)     // 1024
Math.max(1, 5, 3)   // 5
Math.min(1, 5, 3)   // 1
Math.random()       // a float in [0, 1)
Math.trunc(3.9)     // 3 (toward zero)
Math.log(Math.E)    // 1
```

---

## 2.4 Strings

A string is an immutable sequence of Unicode characters. Single quotes, double quotes, and backticks all create strings; backticks additionally enable template literals.

```js
"hello"
'hello'
`hello`
`hello, ${name}`   // template literal: expression interpolated
```

Strings are zero-indexed and have a `length`:

```js
const s = "hello";
s[0]        // "h"
s[4]        // "o"
s.length    // 5
s[-1]       // undefined — negative indexes don't work in strings
```

Useful string methods (none modify the original — strings are immutable):

```js
"hello".toUpperCase()            // "HELLO"
"  hello  ".trim()               // "hello"
"hello world".split(" ")         // ["hello", "world"]
"hello".includes("ell")          // true
"hello".startsWith("he")         // true
"hello".indexOf("l")             // 2
"hello".slice(1, 3)              // "el"
"ha".repeat(3)                   // "hahaha"
"hello".replace("l", "r")        // "herlo"
"hello".replaceAll("l", "r")     // "herro"
String(42)                       // "42"
(42).toString()                  // "42"
(255).toString(16)               // "ff"
```

Converting in the other direction:

```js
parseInt("42px")     // 42
parseInt("0xff", 16) // 255
parseFloat("3.14")   // 3.14
Number("42")         // 42
Number("")           // 0
Number("hello")      // NaN
+"42"                // 42  (unary + coerces to number)
```

---

## 2.5 Booleans

```js
true
false
Boolean(0)       // false
Boolean("")      // false
Boolean(null)    // false
Boolean(undefined) // false
Boolean(NaN)     // false
Boolean(false)   // false

Boolean(1)       // true
Boolean("a")     // true
Boolean([])      // true  ← empty array is truthy
Boolean({})      // true  ← empty object is truthy
```

The six falsy values are: `false`, `0`, `""`, `null`, `undefined`, `NaN`. Everything else is truthy. This list is short enough to memorize.

---

## 2.6 Arithmetic Operators

```js
+   // addition (also string concatenation)
-   // subtraction
*   // multiplication
/   // division (always floating-point)
%   // remainder
**  // exponentiation (right-associative)
++  // increment (prefix or postfix)
--  // decrement (prefix or postfix)
```

Integer division is not a built-in operator; use `Math.trunc(a / b)` or `Math.floor(a / b)` depending on whether you want truncation toward zero or floor behavior.

Prefix vs. postfix increment:

```js
let a = 5;
let b = a++;   // b = 5, a = 6 (postfix: use then increment)
let c = ++a;   // c = 7, a = 7 (prefix: increment then use)
```

In practice, use `i++` in `for` loops and avoid relying on the return value.

---

## 2.7 Comparison Operators

```js
===   // strict equality (checks value AND type)
!==   // strict inequality
<     // less than
<=    // less than or equal
>     // greater than
>=    // greater than or equal
```

Always use `===` and `!==`. The `==` and `!=` operators perform type coercion with unintuitive results:

```js
0 == ""     // true  — don't rely on this
0 == false  // true
null == undefined  // true

0 === ""    // false
0 === false // false
null === undefined  // false
```

String comparison uses Unicode code point order:

```js
"apple" < "banana"  // true
"b" < "a"           // false
"Z" < "a"           // true — uppercase before lowercase in Unicode
```

---

## 2.8 Logical Operators

```js
&&   // AND: true if both operands are truthy
||   // OR: true if either operand is truthy
!    // NOT: negates a boolean value
??   // nullish coalescing
```

`&&` and `||` short-circuit and return one of their operands, not necessarily `true` or `false`:

```js
"hello" && "world"   // "world" (returns last truthy value)
"hello" || "world"   // "hello" (returns first truthy value)
0 && "world"         // 0 (returns first falsy value)
0 || "world"         // "world" (skips first falsy, returns next)
```

The **nullish coalescing** operator `??` returns the right side only when the left side is `null` or `undefined` (not when it's merely falsy):

```js
null ?? "default"      // "default"
undefined ?? "default" // "default"
0 ?? "default"         // 0  ← preserves 0; || would give "default"
"" ?? "default"        // "" ← preserves ""; || would give "default"
```

The **optional chaining** operator `?.` short-circuits on `null` or `undefined`:

```js
const user = null;
user.name           // TypeError: Cannot read property 'name' of null
user?.name          // undefined (safe)
user?.address?.city // undefined (safe, chained)
```

---

## 2.9 Assignment Operators

```js
=    // simple assignment
+=   // x += y means x = x + y
-=   // x -= y means x = x - y
*=   // etc.
/=
%=
**=
&&=  // logical AND assignment: x &&= y means x && (x = y)
||=  // logical OR assignment:  x ||= y means x || (x = y)
??=  // nullish assignment:     x ??= y means x ?? (x = y)
```

Assignment is an expression and has a value (the assigned value). Do not use this in conditions; it produces code that is hard to read.

---

## 2.10 Type Conversions

JavaScript converts types automatically in many contexts. The rules are consistent but numerous. The most important ones:

```js
// To number:
+"3"        // 3
+true       // 1
+false      // 0
+null       // 0
+undefined  // NaN
+[]         // 0
+[3]        // 3
+[1,2]      // NaN

// To string (template literals are explicit and predictable):
`${42}`     // "42"
`${true}`   // "true"
`${null}`   // "null"
`${[1,2]}`  // "1,2"

// To boolean (the six falsy values):
!0          // true
!""         // true
!null       // true
!undefined  // true
!NaN        // true
!false      // true
```

Implicit coercions are a source of bugs. Prefer explicit conversion with `Number()`, `String()`, `Boolean()`, or template literals.

---

## Exercises

**2-1.** What is `typeof typeof 42`? Explain why.

**2-2.** Write a function `clamp(value, min, max)` that returns `min` if `value < min`, `max` if `value > max`, and `value` otherwise. Write it three ways: with `if/else`, with the ternary operator `? :`, and with `Math.min`/`Math.max`.

**2-3.** Why does `0.1 + 0.2 !== 0.3`? Write a function `nearlyEqual(a, b, epsilon)` that returns `true` when two numbers are within `epsilon` of each other. What should the default value of `epsilon` be?

**2-4.** Given a string representing an integer in any base from 2 to 36, write a function that parses it and returns the decimal value. Handle invalid input by returning `NaN`.

**2-5.** JavaScript has no built-in integer type. Write a function `safeInt(n)` that returns the integer part of a number if it is within the safe integer range (`Number.MIN_SAFE_INTEGER` to `Number.MAX_SAFE_INTEGER`), and throws an error otherwise.

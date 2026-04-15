# Chapter 3 — Build Your Own Array Methods

Array methods can feel magical when you first see them. `map`, `filter`, and `reduce` look compact, but they are not magic. They are just loops with clear jobs. In this chapter, you rebuild them from scratch so the short modern syntax makes sense instead of feeling mysterious.

---

## The Problem

A beginner often starts with a plain loop like this:

```javascript
// Old way: explicit loops everywhere
var numbers = [1, 2, 3, 4, 5];
var squared = [];

for (var i = 0; i < numbers.length; i++) {
  squared.push(numbers[i] * numbers[i]);
}
```

This works, but every `for` loop is slightly different: different accumulator names, different index variables, different loop bodies. You can't tell at a glance what a loop is *for* — you have to read the whole body.

ECMAScript 5 (2009) introduced `Array.prototype.map`, `filter`, and `reduce`. The same operation became:

```javascript
const squared = numbers.map(n => n * n);
```

One line. Self-documenting. The word "map" tells you exactly what's happening: *transform every element*. No index variable, no push, no manual accumulator.

The point of this chapter is not just to memorize the method names. The point is to see the pattern behind them. Once you understand that pattern, modern array code becomes much easier to read and write.

One more beginner idea matters here: a callback is just a function you hand to another function so it can decide what to do on each item. That is all `map`, `filter`, and `reduce` are doing. They loop for you, and they call your function at the right moment.

---

## Building It Step by Step

### v1 — The Three Loops

The first version implements each method as a plain function. Notice the loop structure:

```javascript
// v1: the three fundamental collection patterns
const arr = {};

// map: transform — apply fn to each, collect results
arr.map = function(array, fn) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    result.push(fn(array[i], i, array));  // collect RETURN VALUE
  }
  return result;
};

// filter: select — keep elements where fn returns true
arr.filter = function(array, fn) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    if (fn(array[i], i, array)) {   // test RETURN VALUE as boolean
      result.push(array[i]);        // push ORIGINAL element, not fn's result
    }
  }
  return result;
};

// reduce: accumulate — collapse array to a single value
arr.reduce = function(array, fn, initial) {
  let acc = initial;
  for (let i = 0; i < array.length; i++) {
    acc = fn(acc, array[i], i, array);  // acc updates every iteration
  }
  return acc;
};
```

Three loops with three slightly different bodies. The key distinction:
- `map` pushes `fn(item)` — the **return value**
- `filter` pushes `array[i]` — the **original item** — but only when `fn(item)` is truthy
- `reduce` reassigns `acc` to `fn(acc, item)` — the accumulator **replaces itself**

If you get confused, ask one question: what is being collected on each loop iteration?

- in `map`, you collect transformed items
- in `filter`, you collect kept items
- in `reduce`, you do not collect a list at all, you keep updating one running value

### v2 — The Short-Circuit Methods

The second group of methods can stop early. `find` stops at the first match; `every` stops at the first failure; `some` stops at the first success. These use `return` inside the loop:

```javascript
// v2: methods that stop early
arr.find = function(array, fn) {
  for (let i = 0; i < array.length; i++) {
    if (fn(array[i], i, array)) {
      return array[i];    // found — return immediately, loop ends
    }
  }
  return undefined;       // not found
};

arr.every = function(array, fn) {
  for (let i = 0; i < array.length; i++) {
    if (!fn(array[i], i, array)) {
      return false;   // one failure: return false immediately
    }
  }
  return true;        // all passed
};

arr.some = function(array, fn) {
  for (let i = 0; i < array.length; i++) {
    if (fn(array[i], i, array)) {
      return true;    // one match: return true immediately
    }
  }
  return false;       // no match found
};
```

`every` and `some` are logical duals: `every` returns `false` early, `some` returns `true` early. An empty array: `every` returns `true` (vacuously — no element failed), `some` returns `false` (no element matched).

### v3 — reduce Builds Everything Else

The third insight: `reduce` is powerful enough to implement all the others. This is the conceptual payoff:

```javascript
// v3: map and filter as reduce — prove reduce is fundamental
arr.mapViaReduce = function(array, fn) {
  return arr.reduce(array, function(result, item, i) {
    result.push(fn(item, i, array));
    return result;
  }, []);   // start with empty array
};

arr.filterViaReduce = function(array, fn) {
  return arr.reduce(array, function(result, item, i) {
    if (fn(item, i, array)) result.push(item);
    return result;
  }, []);
};

// groupBy — not in the native API, built from reduce
arr.groupBy = function(array, keyFn) {
  return arr.reduce(array, function(groups, item) {
    const key = typeof keyFn === 'function' ? keyFn(item) : item[keyFn];
    if (!groups[key]) groups[key] = [];
    groups[key].push(item);
    return groups;
  }, {});   // start with empty object
};

arr.groupBy(['cat', 'car', 'bat', 'bar', 'can'], word => word[0]);
// → { c: ['cat', 'car', 'can'], b: ['bat', 'bar'] }
```

`reduce`'s accumulator can be anything — a number, string, array, or object. That flexibility is what makes it universal.

---

## The Complete Program

`array_methods.html` — open it in your browser. Enter any JSON array, pick a method, write a callback expression, and run it. The right panel shows your callback applied to each element step by step.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Build Your Own Array Methods — Chapter 3</title>
<!-- styles in array_methods.html -->
</head>
<body>
<script>
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: arr
// ═══════════════════════════════════════════════════════════════════════

const arr = {};

arr.forEach = function(array, fn) {
  for (let i = 0; i < array.length; i++) fn(array[i], i, array);
};

arr.map = function(array, fn) {
  const result = [];
  for (let i = 0; i < array.length; i++) result.push(fn(array[i], i, array));
  return result;
};

arr.filter = function(array, fn) {
  const result = [];
  for (let i = 0; i < array.length; i++)
    if (fn(array[i], i, array)) result.push(array[i]);
  return result;
};

arr.reduce = function(array, fn, initial) {
  let acc = initial;
  for (let i = 0; i < array.length; i++) acc = fn(acc, array[i], i, array);
  return acc;
};

arr.find = function(array, fn) {
  for (let i = 0; i < array.length; i++)
    if (fn(array[i], i, array)) return array[i];
  return undefined;
};

arr.every = function(array, fn) {
  for (const item of array) if (!fn(item)) return false;
  return true;
};

arr.some = function(array, fn) {
  for (const item of array) if (fn(item)) return true;
  return false;
};
</script>
</body>
</html>
```

---

## Walkthrough

### Arrays — Creation, Access, Mutation

An array is an ordered collection of values. Any value can be an element: numbers, strings, booleans, objects, other arrays.

```javascript
// Creation
const nums  = [1, 2, 3, 4, 5];
const mixed = ['alice', 42, true, null, { role: 'admin' }];
const empty = [];

// Access by index (zero-based)
nums[0]     // 1  — first element
nums[4]     // 5  — last element
nums[99]    // undefined — past the end

// Length
nums.length   // 5

// Mutation methods (modify the array in place):
nums.push(6);      // add to end    → [1,2,3,4,5,6]
nums.pop();        // remove from end → [1,2,3,4,5]

// Non-mutation methods (return a new array):
nums.slice(1, 3);  // extract [2, 3]  — original unchanged
nums.concat([6]);  // [1,2,3,4,5,6]   — new array
[...nums, 6];      // [1,2,3,4,5,6]   — spread syntax
```

**Important**: `map`, `filter`, and `reduce` never modify the original array. They always return new ones. This is by design — it means you can chain them safely.

### for loops — Three Forms

```javascript
// 1. Classic for (index) — gives you i, the index
for (let i = 0; i < array.length; i++) {
  console.log(i, array[i]);
}
// Best when: you need the index, or you need to break/return early

// 2. for...of — cleaner, no index
for (const item of array) {
  console.log(item);
}
// Best when: you just need the value, order matters

// 3. for...in — loops over keys (avoid for arrays!)
for (const key in obj) {   // use for plain objects, not arrays
  console.log(key, obj[key]);
}
```

The array methods use the classic `for` because they pass `(item, index, array)` to the callback — they need `i` to build the index argument.

### Functions as Values — Callbacks

In JavaScript, functions are values. You can store them in variables, pass them to other functions, return them from functions.

```javascript
// A function stored in a variable
const double = function(n) { return n * 2; };

// Arrow function syntax — shorter, same thing
const double = n => n * 2;

// Multi-line arrow function — needs { } and return
const describe = (n) => {
  if (n > 0) return 'positive';
  if (n < 0) return 'negative';
  return 'zero';
};

// Passing a function to another function (callback)
arr.map([1, 2, 3], double);      // [2, 4, 6]
arr.map([1, 2, 3], n => n * 2);  // same — inline arrow
```

A **callback** is a function you pass to another function to be called later. `map`'s callback is called once per element. `reduce`'s callback is called once per element too, but it receives the accumulated result as its first argument.

```
The Three Callback Signatures
─────────────────────────────────────────────────────
arr.map(array, fn)
   fn receives:  (item, index, array)
   fn returns:   the new value for that slot
   ↑ "transform" pattern

arr.filter(array, fn)
   fn receives:  (item, index, array)
   fn returns:   boolean (keep this element?)
   ↑ "select" pattern

arr.reduce(array, fn, initial)
   fn receives:  (accumulator, item, index, array)
   fn returns:   the new accumulator value
   ↑ "accumulate" pattern

arr.find / arr.every / arr.some
   fn receives:  (item, index, array)
   fn returns:   boolean (did I find/pass/match?)
   ↑ "search" pattern
```

### map vs filter — The Key Difference

A common mistake: confusing what map and filter push.

```javascript
// map: pushes fn's RETURN VALUE
arr.map([1,2,3,4], n => n > 2);  // [false, false, true, true]
//                                       fn returned false, false, true, true

// filter: pushes the ORIGINAL ITEM (when fn returns truthy)
arr.filter([1,2,3,4], n => n > 2);  // [3, 4]
//                                       original items where fn returned true
```

`map` always returns an array of the same length as the input. `filter` may return a shorter (or even empty) array.

### reduce — The Universal Accumulator

`reduce` is harder to understand at first because it has two inputs per iteration: the item *and* the running total. The key insight is that the accumulator can be anything:

```javascript
// Accumulate into a number: sum
arr.reduce([1,2,3,4,5], (acc, n) => acc + n, 0);
// i=0: acc=0, n=1 → acc=1
// i=1: acc=1, n=2 → acc=3
// i=2: acc=3, n=3 → acc=6
// ...
// → 15

// Accumulate into a string: join with separator
arr.reduce(['a','b','c'], (acc, s) => acc + '-' + s, '');
// → '-a-b-c'  (note the leading dash — .join() is usually cleaner for this)

// Accumulate into an array: same as map
arr.reduce([1,2,3], (acc, n) => { acc.push(n * 2); return acc; }, []);
// → [2, 4, 6]

// Accumulate into an object: count occurrences
arr.reduce(['a','b','a','c','b','a'], (acc, letter) => {
  acc[letter] = (acc[letter] || 0) + 1;
  return acc;
}, {});
// → { a: 3, b: 2, c: 1 }
```

The `(acc[letter] || 0) + 1` pattern deserves attention: `acc[letter]` is `undefined` on the first visit. `undefined || 0` evaluates to `0` (because `undefined` is falsy). Then `0 + 1` gives `1`. On the next visit, `acc[letter]` is `1`, and `1 || 0` evaluates to `1` (because `1` is truthy). Then `1 + 1` gives `2`. This is a clean way to increment a counter that starts at 0.

### every vs some — Logical Duals

```
arr.every — "do ALL elements pass?"
  [✓, ✓, ✓, ✓] → true
  [✓, ✓, ✗, ✓] → false  (stops at ✗)

arr.some  — "does ANY element pass?"
  [✗, ✗, ✗, ✓] → true   (stops at ✓)
  [✗, ✗, ✗, ✗] → false

Edge cases:
  arr.every([], fn) → true   (vacuously: no element failed)
  arr.some([],  fn) → false  (no element matched)
```

Both stop early: `every` at the first failure, `some` at the first success. This makes them efficient for large arrays where you expect early termination.

### for...of vs index-based for

`every` and `some` in the library use `for...of` because they don't need the index:

```javascript
arr.every = function(array, fn) {
  for (const item of array) {   // cleaner — no i, no array[i]
    if (!fn(item)) return false;
  }
  return true;
};
```

`map`, `filter`, and `reduce` use the index-based `for` because their callbacks receive `(item, index, array)`. If the index isn't needed, `for...of` is cleaner.

---

## Guided Try It — Implement `arr.flatMap`

**The goal**: `flatMap` is a method that maps and then flattens one level. It's like calling `map` and then `flat` in sequence — but it's so common that browsers provide it as a single method.

```javascript
// What flatMap does:
[1, 2, 3].flatMap(n => [n, n * 2]);
// → [1, 2, 2, 4, 3, 6]

// Equivalent with map then flat:
[1, 2, 3].map(n => [n, n * 2]).flat();
// → [1, 2, 2, 4, 3, 6]
```

**Step 1 — Implement using map and flat**

The simplest correct implementation: use the two methods you already have.

```javascript
arr.flatMap = function(array, fn) {
  const mapped = arr.map(array, fn);    // map first
  return arr.flat(mapped);              // then flatten one level
};

// Test:
arr.flatMap([1, 2, 3], n => [n, n * 2]);
// → [1, 2, 2, 4, 3, 6]

// flatMap to expand sentences into words:
arr.flatMap(['hello world', 'foo bar'], s => s.split(' '));
// → ['hello', 'world', 'foo', 'bar']
```

**Step 2 — Implement in a single pass**

The two-pass version creates an intermediate array. Do it in one pass:

```javascript
arr.flatMap = function(array, fn) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    const mapped = fn(array[i], i, array);
    if (Array.isArray(mapped)) {
      for (const item of mapped) result.push(item);  // inline flat
    } else {
      result.push(mapped);
    }
  }
  return result;
};
```

Same result, no intermediate array.

**Step 3 — Implement using reduce**

Since `reduce` can build any output, implement `flatMap` with it:

```javascript
arr.flatMap = function(array, fn) {
  return arr.reduce(array, function(result, item, i) {
    const mapped = fn(item, i, array);
    if (Array.isArray(mapped)) {
      for (const x of mapped) result.push(x);
    } else {
      result.push(mapped);
    }
    return result;
  }, []);
};
```

**Think about it**: All three implementations produce the same output. Which is most readable? Which is most efficient? In which situation would the two-pass version (Step 1) actually be preferable to the single-pass version (Step 2)? (Hint: consider a `flat` implementation that handles arbitrary depth.)

---

## Exercises

1. **Implement `arr.indexOf`**: Return the index of the first element strictly equal to `target`, or `-1` if not found. Use a classic `for` loop. `arr.indexOf([10, 20, 30, 20], 20)` should return `1`. Do not use the native `.indexOf()`.

2. **Implement `arr.flatten(depth)`**: Flatten an array up to `depth` levels. `arr.flatten([[1,[2]],[3]], 1)` → `[1,[2],3]`. `arr.flatten([[1,[2,[3]]]], Infinity)` → `[1,2,3]`. Use recursion: if an element is an array *and* depth > 0, recursively flatten it with `depth - 1`.

3. **Implement `arr.zip`**: `arr.zip([1,2,3], ['a','b','c'])` → `[[1,'a'],[2,'b'],[3,'c']]`. Pair elements by index and stop at the shorter array's length. Use a `for` loop with `Math.min(a.length, b.length)` as the limit.

4. **Implement `arr.groupBy`**: `arr.groupBy(['cat','car','bat','bar'], w => w[0])` → `{ c: ['cat','car'], b: ['bat','bar'] }`. Use `arr.reduce` with an object accumulator. The key function receives each item and returns the group key.

5. **Implement `arr.pipe`**: `arr.pipe([double, square, addOne])` returns a function that chains those transformations. `arr.pipe([double, square, addOne])(3)` should compute `addOne(square(double(3)))` = `addOne(square(6))` = `addOne(36)` = `37`. Use `arr.reduce` over the functions array. (This is Chapter 5's `_.pipe` — you now have the tools to build it.)

---

## Solutions

### Exercise 1 — `arr.indexOf`

```javascript
arr.indexOf = function(array, target) {
  for (let i = 0; i < array.length; i++) {
    if (array[i] === target) return i;   // strict equality: ===
  }
  return -1;
};

// Tests:
arr.indexOf([10, 20, 30, 20], 20);   // 1  (first occurrence)
arr.indexOf([10, 20, 30, 20], 99);   // -1 (not found)
arr.indexOf([10, 20, 30, 20], '20'); // -1 (strict: number !== string)
arr.indexOf([], 1);                  // -1 (empty array)
```

### Exercise 2 — `arr.flatten(depth)`

```javascript
arr.flatten = function(array, depth) {
  if (depth === undefined) depth = 1;   // default: one level

  return arr.reduce(array, function(result, item) {
    if (Array.isArray(item) && depth > 0) {
      // Item is an array and we still have depth — recurse
      const flattened = arr.flatten(item, depth - 1);
      for (const x of flattened) result.push(x);
    } else {
      result.push(item);
    }
    return result;
  }, []);
};

// Tests:
arr.flatten([1, [2, 3], [4, [5, 6]]]);          // [1,2,3,4,[5,6]]  (depth=1)
arr.flatten([1, [2, 3], [4, [5, 6]]], 2);       // [1,2,3,4,5,6]
arr.flatten([[1,[2,[3,[4]]]]],    Infinity);     // [1,2,3,4]
arr.flatten([1, 2, 3],            0);            // [1,2,3]  (no flattening)
```

### Exercise 3 — `arr.zip`

```javascript
arr.zip = function(a, b) {
  const len    = Math.min(a.length, b.length);
  const result = [];
  for (let i = 0; i < len; i++) {
    result.push([a[i], b[i]]);
  }
  return result;
};

// Tests:
arr.zip([1, 2, 3], ['a', 'b', 'c']);   // [[1,'a'],[2,'b'],[3,'c']]
arr.zip([1, 2],    ['a', 'b', 'c']);   // [[1,'a'],[2,'b']]  (stops at shorter)
arr.zip([], [1, 2]);                   // []

// Zip more than two arrays:
arr.zipAll = function(...arrays) {
  const len = Math.min(...arrays.map(a => a.length));
  const result = [];
  for (let i = 0; i < len; i++) {
    result.push(arrays.map(a => a[i]));
  }
  return result;
};
arr.zipAll([1,2,3], ['a','b','c'], [true,false,true]);
// [[1,'a',true],[2,'b',false],[3,'c',true]]
```

### Exercise 4 — `arr.groupBy`

```javascript
arr.groupBy = function(array, keyFn) {
  return arr.reduce(array, function(groups, item) {
    const key = typeof keyFn === 'function' ? keyFn(item) : item[keyFn];
    // typeof check: keyFn can be a function OR a property name string
    if (!groups[key]) groups[key] = [];
    groups[key].push(item);
    return groups;
  }, {});
};

// Tests:
arr.groupBy(['cat','car','bat','bar'], w => w[0]);
// { c: ['cat','car'], b: ['bat','bar'] }

arr.groupBy([1,2,3,4,5,6], n => n % 2 === 0 ? 'even' : 'odd');
// { odd: [1,3,5], even: [2,4,6] }

// With property name string (like Lodash's _.groupBy):
const people = [{ name: 'Ada', dept: 'eng' }, { name: 'Bob', dept: 'mkt' }, { name: 'Cal', dept: 'eng' }];
arr.groupBy(people, 'dept');
// { eng: [{name:'Ada',...},{name:'Cal',...}], mkt: [{name:'Bob',...}] }
```

### Exercise 5 — `arr.pipe`

```javascript
arr.pipe = function(fns) {
  return function(value) {
    return arr.reduce(fns, function(acc, fn) {
      return fn(acc);
    }, value);
    // acc starts as 'value', then each function transforms it
  };
};

// Tests:
const double  = n => n * 2;
const square  = n => n * n;
const addOne  = n => n + 1;

const transform = arr.pipe([double, square, addOne]);
transform(3);   // double(3)=6, square(6)=36, addOne(36)=37 ✓
transform(2);   // double(2)=4, square(4)=16, addOne(16)=17

// With strings:
const normalize = arr.pipe([
  s => s.trim(),
  s => s.toLowerCase(),
  s => s.replace(/\s+/g, '-')
]);
normalize('  Hello World  ');   // 'hello-world'
```

The inner `arr.reduce` uses the functions array as the collection and the running value as the accumulator. At each step, it calls the next function with the current value. This is exactly how `_.pipe` works in Chapter 5.

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| Array indexing | Zero-based; out-of-bounds returns `undefined`, not an error |
| `.push()` / `.pop()` | Mutate the array in place; `map`/`filter`/`reduce` do not |
| Classic `for` loop | `for (let i = 0; i < array.length; i++)` — use when you need index |
| `for...of` loop | `for (const item of array)` — cleaner when you only need the value |
| Functions as values | Functions can be stored in variables and passed as arguments |
| Callback signature | `map`/`filter` receive `(item, index, array)`; `reduce` gets `(acc, item, index, array)` |
| `arr.map` | Transforms every element; result is always same length as input |
| `arr.filter` | Selects elements; pushes the *original* item, not fn's return value |
| `arr.reduce` | Accumulates to a single value; accumulator can be number, string, array, or object |
| `arr.find` | Returns first match or `undefined`; stops early |
| `arr.every` | Returns `false` at first failure; empty array → `true` |
| `arr.some` | Returns `true` at first match; empty array → `false` |
| Short-circuit | `find`/`every`/`some` return early — `map`/`filter`/`reduce` always visit all elements |

---

## Building with Claude

Bad prompt:
> "Explain map, filter, and reduce."

Good prompt:
> "I'm implementing `arr.reduce` from scratch using a `for` loop. My implementation is: `let acc = initial; for (let i = 0; i < array.length; i++) { acc = fn(acc, array[i], i, array); } return acc;`. It works for summing numbers but breaks when I don't pass an `initial` value — the native `Array.prototype.reduce` handles missing `initial` by using the first element as the accumulator and starting the loop at index 1. How do I detect the missing-initial case, and what's the cleanest way to handle it without changing the external API?"

The good prompt shows your exact implementation, specifies what works and what doesn't, and asks a concrete question about a specific edge case. Claude will give you a targeted answer — not a general tutorial.

# Chapter 5 — Build Your Own Lodash

Now that you have built some core language tools, you can start building the helpers that bigger programs depend on. Lodash is a collection of reusable utility functions. This chapter keeps that idea beginner-friendly: instead of treating utility helpers as magic, you will build them from patterns you already know.

---

## The Problem

Once programs get a little bigger, you start solving the same kinds of problems again and again: transform a list, group items, remove duplicates, delay a function call. Writing all of that from scratch every time is wasteful.

John-David Dalton released Lodash in 2012 to solve three things at once: (1) give every browser the same set of reliable collection utilities, (2) fill the gaps the standard library left (there was no built-in `groupBy`, `flatten`, or `debounce`), and (3) make the code read like the problem domain — `_.filter(books, b => b.genre === 'Sci-Fi')` is self-documenting in a way that raw `for` loops are not.

The important beginner idea is this: many programming tasks are really the same shape with different data. If you build a helper once, you can reuse it everywhere.

You have a dataset of books. You need to search them, filter by genre, sort by rating, group by category, and compute aggregate statistics — all driven by user input. You could write custom loops everywhere. Or you build the abstractions once, and compose them.

---

## Building It Step by Step

### v1 — The Core Three: map, filter, reduce

The first observation is that almost every collection operation is "loop + accumulate." Notice that map, filter, and reduce share the same skeleton:

```javascript
// v1: three operations, same pattern
const _ = {};

_.map = function(array, fn) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    result.push(fn(array[i], i, array));
  }
  return result;
};

_.filter = function(array, fn) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    if (fn(array[i], i, array)) result.push(array[i]);
  }
  return result;
};

_.reduce = function(array, fn, initial) {
  let acc = initial;
  for (let i = 0; i < array.length; i++) {
    acc = fn(acc, array[i], i, array);
  }
  return acc;
};
```

Why is `reduce` listed third but described as most powerful? Because with it, the other two are derivable:

```javascript
// map as reduce:
xs => _.reduce(xs, (acc, x) => { acc.push(fn(x)); return acc; }, []);
// filter as reduce:
xs => _.reduce(xs, (acc, x) => { if (pred(x)) acc.push(x); return acc; }, []);
```

This version is usable. You can already process the book dataset. But writing pipeline logic is still verbose.

### v2 — reduce Powers the Higher-Level Operations

The second observation: `_.reduce` is so general that `groupBy`, `flatten`, and `uniq` are all just reductions.

```javascript
// v2: build higher-level operations from reduce

_.groupBy = function(array, key) {
  return _.reduce(array, function(groups, item) {
    const k = typeof key === 'function' ? key(item) : item[key];
    if (!groups[k]) groups[k] = [];
    groups[k].push(item);
    return groups;
  }, {});
  // The accumulator starts as {}, and we build it up one item at a time.
  // reduce's acc is the *running result* — here it's the groups object.
};

_.flatten = function(array) {
  return _.reduce(array, function(result, item) {
    return result.concat(Array.isArray(item) ? _.flatten(item) : item);
  }, []);
  // Recursive: if the item is itself an array, flatten it first before concat.
};

_.uniq = function(array) {
  const seen = new Set();
  return _.filter(array, function(item) {
    if (seen.has(item)) return false;
    seen.add(item);
    return true;
  });
  // Built from _.filter — new library functions can use old library functions.
};
```

This is the payoff of building a library: `groupBy` and `flatten` are implemented in terms of `reduce`, and `uniq` is implemented in terms of `filter`. The library grows by composition, not by copy-pasting loops.

### v3 — A Different Category: Function Utilities

The third observation is that `debounce`, `memoize`, `once`, and `pipe` are not about data at all. They are about *functions themselves* — when they fire, how they cache, how they compose. This is a new category of utility.

```javascript
// v3: function utilities — a completely different kind of tool

// debounce: don't fire until input has paused
_.debounce = function(fn, wait) {
  let timer = null;
  return function() {            // returns a NEW function
    const args = arguments;
    const ctx  = this;
    clearTimeout(timer);         // cancel the previous pending call
    timer = setTimeout(function() { fn.apply(ctx, args); }, wait);
  };
};

// memoize: cache the result so expensive work runs only once per unique input
_.memoize = function(fn) {
  const cache = new Map();
  return function() {
    const key = JSON.stringify(Array.from(arguments));
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, arguments);
    cache.set(key, result);
    return result;
  };
};

// pipe: feed output of each function into input of the next (left-to-right)
_.pipe = function() {
  const fns = Array.prototype.slice.call(arguments);
  return function(value) {
    return _.reduce(fns, function(acc, fn) { return fn(acc); }, value);
  };
};
```

Why separate category? Because `debounce` and `memoize` don't care what kind of data you have — they work on *any* function. The pattern is: take a function, return a smarter function. This is called a **higher-order function**. Understanding that distinction — collection utilities vs. function utilities — is understanding the full shape of Lodash.

---

## The Complete Program

`lodash.html` — open it in your browser to see the Book Explorer powered entirely by `_`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Build Your Own Lodash — Chapter 5</title>
  <!-- styles omitted for brevity — see lodash.html -->
</head>
<body>
<script>
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: _ (Mini Lodash)
// ═══════════════════════════════════════════════════════════════════════

const _ = {};

// ─── COLLECTION METHODS ──────────────────────────────────────────────

_.map = function(array, fn) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    result.push(fn(array[i], i, array));
  }
  return result;
};

_.filter = function(array, fn) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    if (fn(array[i], i, array)) result.push(array[i]);
  }
  return result;
};

_.reduce = function(array, fn, initial) {
  let acc = initial;
  for (let i = 0; i < array.length; i++) {
    acc = fn(acc, array[i], i, array);
  }
  return acc;
};

_.find = function(array, fn) {
  for (let i = 0; i < array.length; i++) {
    if (fn(array[i], i, array)) return array[i];
  }
  return undefined;
};

_.every = function(array, fn) {
  for (const item of array) {
    if (!fn(item)) return false;
  }
  return true;
};

_.some = function(array, fn) {
  for (const item of array) {
    if (fn(item)) return true;
  }
  return false;
};

_.chunk = function(array, size) {
  const result = [];
  for (let i = 0; i < array.length; i += size) {
    result.push(array.slice(i, i + size));
  }
  return result;
};

_.groupBy = function(array, key) {
  return _.reduce(array, function(groups, item) {
    const k = typeof key === 'function' ? key(item) : item[key];
    if (!groups[k]) groups[k] = [];
    groups[k].push(item);
    return groups;
  }, {});
};

_.sortBy = function(array, key) {
  return [...array].sort(function(a, b) {
    const va = typeof key === 'function' ? key(a) : a[key];
    const vb = typeof key === 'function' ? key(b) : b[key];
    return va < vb ? -1 : va > vb ? 1 : 0;
  });
};

_.flatten = function(array) {
  return _.reduce(array, function(result, item) {
    return result.concat(Array.isArray(item) ? _.flatten(item) : item);
  }, []);
};

_.uniq = function(array) {
  const seen = new Set();
  return _.filter(array, function(item) {
    if (seen.has(item)) return false;
    seen.add(item);
    return true;
  });
};

// ─── FUNCTION UTILITIES ───────────────────────────────────────────────

_.debounce = function(fn, wait) {
  let timer = null;
  return function() {
    const args = arguments;
    const ctx  = this;
    clearTimeout(timer);
    timer = setTimeout(function() { fn.apply(ctx, args); }, wait);
  };
};

_.memoize = function(fn) {
  const cache = new Map();
  const memoized = function() {
    const key = JSON.stringify(Array.from(arguments));
    if (cache.has(key)) {
      memoized.cacheHit = true;
      return cache.get(key);
    }
    memoized.cacheHit = false;
    const result = fn.apply(this, arguments);
    cache.set(key, result);
    return result;
  };
  memoized.cache = cache;
  memoized.clearCache = function() { cache.clear(); };
  return memoized;
};

_.once = function(fn) {
  let called = false;
  let result;
  return function() {
    if (!called) {
      called = true;
      result = fn.apply(this, arguments);
    }
    return result;
  };
};

_.partial = function(fn) {
  const preArgs = Array.prototype.slice.call(arguments, 1);
  return function() {
    const laterArgs = Array.prototype.slice.call(arguments);
    return fn.apply(this, preArgs.concat(laterArgs));
  };
};

_.pipe = function() {
  const fns = Array.prototype.slice.call(arguments);
  return function(value) {
    return _.reduce(fns, function(acc, fn) { return fn(acc); }, value);
  };
};

// ─── OBJECT UTILITIES ─────────────────────────────────────────────────

_.pick = function(obj) {
  const keys = _.flatten(Array.prototype.slice.call(arguments, 1));
  const result = {};
  for (const key of keys) {
    if (key in obj) result[key] = obj[key];
  }
  return result;
};

_.omit = function(obj) {
  const omitKeys = new Set(_.flatten(Array.prototype.slice.call(arguments, 1)));
  const result = {};
  for (const key of Object.keys(obj)) {
    if (!omitKeys.has(key)) result[key] = obj[key];
  }
  return result;
};

_.merge = function(target, source) {
  const result = Object.assign({}, target);
  for (const [k, v] of Object.entries(source)) {
    if (v && typeof v === 'object' && !Array.isArray(v) &&
        result[k] && typeof result[k] === 'object') {
      result[k] = _.merge(result[k], v);
    } else {
      result[k] = v;
    }
  }
  return result;
};
</script>
</body>
</html>
```

---

## Walkthrough

### Data Pipeline with _

```
Data Pipeline with _
─────────────────────────────────────────────────────
BOOKS array
   │
   ├─ _.filter(books, b => b.genre === 'Sci-Fi')
   │       keeps matching elements → new array
   │
   ├─ _.sortBy(result, b => -b.rating)
   │       sorts by key/fn → new array (spread prevents mutation)
   │
   ├─ _.map(result, b => b.title)
   │       transforms every element → ['Dune', 'Foundation', ...]
   │
   └─ _.groupBy(result, 'genre')
           builds object → { 'Sci-Fi': [...], 'Fiction': [...] }
```

Each step takes an array and returns a new array (or object). Nothing is mutated. Each step is independent and testable. This is a pipeline.

### `_.map` — Transform Every Element

`_.map` takes an array and a function, applies the function to each element, and returns a new array of results. The original array is never modified.

```javascript
_.map = function(array, fn) {
  const result = [];
  for (let i = 0; i < array.length; i++) {
    result.push(fn(array[i], i, array));
  }
  return result;
};
```

The callback receives three arguments: the current item, its index, and the whole array. This matches how the native `Array.prototype.map` works. Knowing that you pass `(item, index, array)` lets you write callbacks that use position — something often forgotten.

```javascript
_.map(['a', 'b', 'c'], (item, i) => `${i}: ${item}`)
// → ['0: a', '1: b', '2: c']
```

### `_.filter` and `_.reduce` — The Foundation

`_.filter` keeps elements where the predicate returns true. `_.reduce` is the most powerful: it collapses an array to a single value by accumulating through it. Almost every other collection function can be built from `reduce`.

```javascript
// Use reduce to sum ratings:
const avg = _.reduce(books, (sum, b) => sum + b.rating, 0) / books.length;

// Use reduce to implement groupBy:
_.groupBy = function(array, key) {
  return _.reduce(array, function(groups, item) {
    const k = typeof key === 'function' ? key(item) : item[key];
    if (!groups[k]) groups[k] = [];
    groups[k].push(item);
    return groups;
  }, {});
};
```

Notice that `_.groupBy` is built entirely using `_.reduce`. This is the power of building a library — your new functions can use your old ones.

### `_.debounce` — Controlling When a Function Fires

`_.debounce` wraps a function so that it only runs after a pause. Call it ten times in a row, and only the last call fires (after `wait` milliseconds of silence).

```javascript
_.debounce = function(fn, wait) {
  let timer = null;
  return function() {
    const args = arguments;
    const ctx  = this;
    clearTimeout(timer);
    timer = setTimeout(function() { fn.apply(ctx, args); }, wait);
  };
};
```

The returned function is a closure — it "closes over" (remembers) `timer` even after `_.debounce` has returned. Every call clears the previous timer and starts a new one. The actual `fn` only runs when a full `wait` milliseconds pass with no new calls. This is exactly how search boxes avoid hammering a server on every keystroke.

```javascript
const debouncedUpdate = _.debounce(updateDisplay, 250);
input.addEventListener('input', debouncedUpdate); // only fires 250ms after typing stops
```

### `_.memoize` — Cache Expensive Results

`_.memoize` wraps a function with a cache. The first time you call it with a given argument, it runs the function and stores the result. The second time, it returns the cached result immediately.

```javascript
_.memoize = function(fn) {
  const cache = new Map();
  const memoized = function() {
    const key = JSON.stringify(Array.from(arguments));
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, arguments);
    cache.set(key, result);
    return result;
  };
  memoized.cache = cache;
  return memoized;
};
```

The cache is a `Map` (not an object) because `Map` handles any key type and has fast `.has()` / `.get()` / `.set()`. The key is the JSON-stringified arguments — so `memoFib(10)` and `memoFib(20)` get different cache entries.

The classic demonstration is recursive Fibonacci. Without memoization, `fib(40)` requires over a billion function calls. With memoization, it requires 40.

```javascript
function slowFib(n) {
  if (n <= 1) return n;
  return memoFib(n - 1) + memoFib(n - 2);
}
const memoFib = _.memoize(slowFib);
memoFib(40); // instant, even though slowFib alone would hang the browser
```

### `_.pipe` — Left-to-Right Function Composition

`_.pipe` takes any number of functions and returns a new function that feeds the output of each into the input of the next.

```javascript
_.pipe = function() {
  const fns = Array.prototype.slice.call(arguments);
  return function(value) {
    return _.reduce(fns, function(acc, fn) { return fn(acc); }, value);
  };
};
```

```javascript
const process = _.pipe(
  xs => _.filter(xs, x => x > 2),    // keep elements > 2
  xs => _.map(xs, x => x * x),        // square them
  xs => _.reduce(xs, (s, x) => s + x, 0) // sum
);

process([1, 2, 3, 4, 5]); // → 41  (9 + 16 + 25)
```

`_.pipe` is implemented using `_.reduce` — the accumulator starts as the initial value, and each step applies the next function. This is the cleanest example of reduce in action.

### `_.once` and `_.partial` — Controlling Function Calls

`_.once` wraps a function so it can only execute once. Useful for initialisation code that must run exactly one time.

```javascript
const initDatabase = _.once(() => {
  console.log('DB connected');
  // expensive setup
});
initDatabase(); // 'DB connected'
initDatabase(); // nothing — returns first result
initDatabase(); // nothing
```

`_.partial` pre-fills some arguments:

```javascript
function multiply(a, b) { return a * b; }
const double = _.partial(multiply, 2);
double(5);  // → 10
double(21); // → 42
```

This is a lightweight form of currying. You take a general function and specialise it for a particular use.

---

## Guided Try It — Build `_.throttle` Step by Step

**The problem**: You want to handle `mousemove` events, but firing a handler 60 times per second is too expensive. `_.debounce` doesn't work because it only fires *after* the mouse stops — you want to fire at most once every 200ms *while* the mouse is moving.

This is `_.throttle`: the first call fires immediately, subsequent calls within the window are ignored, and after the window expires the next call fires again.

**Hint**: The key difference from debounce is tracking *when the last call happened*, not *resetting a timer on every call*.

**Step 1 — Check if we're inside a throttle window**

The simplest possible throttle tracks whether a timer is running. If it is, skip the call:

```javascript
_.throttle = function(fn, wait) {
  let waiting = false;
  return function() {
    if (waiting) return;   // inside the window — ignore
    waiting = true;
    fn.apply(this, arguments);
    setTimeout(function() {
      waiting = false;     // window expired — allow next call
    }, wait);
  };
};
```

This fires immediately and suppresses calls for `wait` ms. But notice: the call that arrives *just before* the window expires is lost entirely. That's fine for many cases, but scroll handlers and progress indicators want the final position too.

**Step 2 — Track the last-fire timestamp instead of a boolean**

Using `Date.now()` lets you calculate exactly how long ago the last call fired, and skip the `setTimeout` overhead:

```javascript
_.throttle = function(fn, wait) {
  let lastFired = 0;
  return function() {
    const now = Date.now();
    if (now - lastFired < wait) return;   // still inside window
    lastFired = now;
    fn.apply(this, arguments);
  };
};
```

**Step 3 — The full implementation (trailing call)**

To also fire once after the window closes with the most recent arguments, save them and schedule a trailing call:

```javascript
_.throttle = function(fn, wait) {
  let lastFired = 0;
  let trailingTimer = null;
  let trailingArgs = null;
  let trailingCtx  = null;

  return function() {
    const now = Date.now();
    const remaining = wait - (now - lastFired);

    if (remaining <= 0) {
      // Window has expired — fire immediately
      clearTimeout(trailingTimer);
      trailingTimer = null;
      lastFired = now;
      fn.apply(this, arguments);
    } else {
      // Inside window — save args for the trailing call
      trailingArgs = arguments;
      trailingCtx  = this;
      clearTimeout(trailingTimer);
      trailingTimer = setTimeout(function() {
        lastFired = Date.now();
        trailingTimer = null;
        fn.apply(trailingCtx, trailingArgs);
      }, remaining);
    }
  };
};
```

**Think about it**: Step 1 (boolean flag) and Step 2 (timestamp) both suppress mid-window calls. Why does Step 2 handle clock drift and tab-switching better? And what does the trailing call in Step 3 give you that Steps 1 and 2 don't? (Consider a progress bar that needs to show the final scroll position.)

---

## Exercises

1. **Implement `_.zip`**: `_.zip([1, 2, 3], ['a', 'b', 'c'])` should return `[[1, 'a'], [2, 'b'], [3, 'c']]`. Handle arrays of different lengths by stopping at the shortest. Implement using a `for` loop and no other `_` methods.

2. **Implement `_.intersection`**: `_.intersection([1, 2, 3], [2, 3, 4])` returns `[2, 3]` — elements present in all arrays. Accept any number of arrays. Use `_.filter` and `_.every` internally.

3. **Implement `_.compose`**: Like `_.pipe` but right-to-left. `_.compose(f, g, h)(x)` should equal `f(g(h(x)))`. Implement it by reversing the functions array before reducing. Write one test case showing it produces the opposite order from `_.pipe`.

4. **Implement `_.flatten` with a depth argument**: `_.flatten([1, [2, [3, [4]]]], 1)` should return `[1, 2, [3, [4]]]` — only flattening one level. `_.flatten([1, [2, [3]]], Infinity)` should flatten completely. Modify the existing implementation to support this.

5. **Build a `_.curry` function**: `_.curry(fn)` returns a function that collects arguments until it has enough to call `fn`. `_.curry(add)(1)(2)` and `_.curry(add)(1, 2)` and `_.curry(add, 1)(2)` should all return `3`. Use `fn.length` to know how many arguments are needed.

---

## Solutions

### Exercise 1 — `_.zip`

```javascript
_.zip = function(a, b) {
  const len = Math.min(a.length, b.length);
  const result = [];
  for (let i = 0; i < len; i++) {
    result.push([a[i], b[i]]);
  }
  return result;
};

// Test:
_.zip([1, 2, 3], ['a', 'b', 'c']); // [[1,'a'],[2,'b'],[3,'c']]
_.zip([1, 2], ['a', 'b', 'c']);     // [[1,'a'],[2,'b']] — stops at shortest
```

For multiple arrays, collect them all and zip across all at once:

```javascript
_.zipAll = function() {
  const arrays = Array.from(arguments);
  const len = Math.min(...arrays.map(a => a.length));
  const result = [];
  for (let i = 0; i < len; i++) {
    result.push(arrays.map(a => a[i]));
  }
  return result;
};
```

### Exercise 2 — `_.intersection`

```javascript
_.intersection = function() {
  const arrays = Array.from(arguments);
  if (arrays.length === 0) return [];
  const [first, ...rest] = arrays;
  return _.filter(first, item =>
    _.every(rest, arr => _.some(arr, x => x === item))
  );
};

// Test:
_.intersection([1, 2, 3], [2, 3, 4]);          // [2, 3]
_.intersection([1, 2, 3], [2, 3, 4], [3, 4, 5]); // [3]
```

### Exercise 3 — `_.compose`

```javascript
_.compose = function() {
  const fns = Array.prototype.slice.call(arguments).reverse();
  return _.pipe.apply(null, fns);
};

// Demonstration — pipe vs compose:
const addOne  = x => x + 1;
const double  = x => x * 2;
const square  = x => x * x;

_.pipe(addOne, double, square)(3);    // addOne(3)=4 → double=8 → square=64
_.compose(square, double, addOne)(3); // same result: 64 (right-to-left = left-to-right when reversed)
```

### Exercise 4 — `_.flatten` with depth

```javascript
_.flatten = function(array, depth) {
  if (depth === undefined) depth = Infinity;
  return _.reduce(array, function(result, item) {
    if (Array.isArray(item) && depth > 0) {
      return result.concat(_.flatten(item, depth - 1));
    }
    return result.concat(item);
  }, []);
};

// Tests:
_.flatten([1, [2, [3, [4]]]]);       // [1,2,3,4] (full depth)
_.flatten([1, [2, [3, [4]]]], 1);    // [1,2,[3,[4]]]
_.flatten([1, [2, [3, [4]]]], 2);    // [1,2,3,[4]]
```

### Exercise 5 — `_.curry`

```javascript
_.curry = function(fn) {
  const arity = fn.length;
  return function curried() {
    const args = Array.from(arguments);
    if (args.length >= arity) {
      return fn.apply(this, args);
    }
    return function() {
      return curried.apply(this, args.concat(Array.from(arguments)));
    };
  };
};

// Tests:
function add(a, b) { return a + b; }
const curriedAdd = _.curry(add);

curriedAdd(1)(2);     // 3
curriedAdd(1, 2);     // 3
curriedAdd(3)(4);     // 7

function multiply3(a, b, c) { return a * b * c; }
const cm = _.curry(multiply3);
cm(2)(3)(4);  // 24
cm(2, 3)(4);  // 24
cm(2)(3, 4);  // 24
```

The key insight: `fn.length` gives the number of declared parameters. `curried` collects arguments across multiple calls until it has enough, then fires the original function.

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `_.map` | Transforms every element; returns a new array; never mutates |
| `_.filter` | Keeps elements where predicate returns true |
| `_.reduce` | Folds array to one value; most powerful collection function |
| `_.groupBy` | Built from `_.reduce`; shows how library functions compose |
| `_.sortBy` | Uses spread `[...array]` to avoid mutating the original |
| `_.debounce` | Closure over `timer`; delays execution until input pauses |
| `_.throttle` | Closure over timestamp; limits execution rate while still firing |
| `_.memoize` | `Map` cache keyed by `JSON.stringify(args)`; trades memory for speed |
| `_.once` | Closure over `called` flag; guards one-time initialisation |
| `_.partial` | Pre-fills arguments; specialises a general function |
| `_.pipe` | Implemented with `_.reduce`; feeds output of each fn into next |
| Higher-order function | A function that takes or returns another function |
| Closure | Inner function that "remembers" variables from its creation scope |

---

## Building with Claude

Bad prompt:
> "Give me a utility library."

Good prompt:
> "I'm building a mini Lodash in plain JavaScript (no dependencies). I've implemented `_.map`, `_.filter`, and `_.reduce` using manual `for` loops. Now I want to implement `_.debounce`. I understand it should delay the function call until the user stops typing, using `clearTimeout` and `setTimeout`. Can you show me the closure-based implementation and explain exactly which variables the inner function closes over and why they persist between calls?"

The good prompt shows what you've already built, specifies the exact function you want, and asks for the *explanation* rather than just the code. Claude's answer will teach you — not just give you something to paste.

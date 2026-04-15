# Chapter 1 — Build Your Own Logger

A logger is a great first JavaScript tool because it does something useful right away. You give it a value, and it prints something readable. While building it, you learn one of the most important beginner skills in JavaScript: understanding what kind of value you are working with.

---

## The Problem

`console.log` is enough to get started, but it quickly becomes messy. If you log a string, an array, `null`, or an object, the output can look inconsistent. If something goes wrong, you also want to know how serious it is and when it happened.

To build a better logger, you need to answer three basic JavaScript questions:

```
What type is this value?   →  typeof, Array.isArray, the null quirk
How should I display it?   →  template literals, String(), JSON.stringify
Where should I store it?   →  objects, arrays, factory functions
```

These questions show up in almost every JavaScript program, not just loggers. That is why this chapter is a good place to begin.

If you are new, focus on one idea first: a JavaScript value is not just "data". It always has a type, and that type changes what you can do with it. A string behaves differently from a number. An array behaves differently from an object. Good JavaScript starts with noticing that difference.

---

## Building It Step by Step

### v1 — A Type-Aware print

The simplest possible logger: take any value, detect its type, format it as a string.

```javascript
// v1: the core insight — every value has a type, and type determines display
function log(value) {
  const type = typeof value;           // 'string', 'number', 'boolean', 'object', ...
  const formatted = `[${type}] ${value}`;
  console.log(formatted);
}

log("hello");    // [string] hello
log(42);         // [number] 42
log(true);       // [boolean] true
log(null);       // [object] null    ← uh oh
```

The last line exposes JavaScript's most famous quirk: `typeof null === 'object'`. This is a bug from 1995 that can never be fixed because millions of programs depend on it. We fix it ourselves:

```javascript
function log(value) {
  const type = value === null ? 'null' : typeof value;
  // Now: log(null) → '[null] null'  ✓
  const formatted = `[${type}] ${value}`;
  console.log(formatted);
}
```

v1 is a real tool. But every message looks the same — there's no way to tell a debug trace from a crash report.

### v2 — Add Levels and Timestamps

Real loggers assign severity levels. A debug trace has low priority; an error has high priority. Add levels and a timestamp:

```javascript
// v2: levels turn messages into events
const LEVELS = { DEBUG: 0, INFO: 1, WARN: 2, ERROR: 3 };

function getType(value) {
  if (value === null)       return 'null';
  if (Array.isArray(value)) return 'array';   // arrays are objects — be specific
  return typeof value;
}

function formatValue(value) {
  const type = getType(value);
  if (type === 'string')                return `"${value}"`;
  if (type === 'null' || type === 'undefined') return String(value);
  if (type === 'array' || type === 'object')   return JSON.stringify(value);
  if (type === 'function') return value.name ? `[Function: ${value.name}]` : '[Function]';
  return String(value);
}

function log(level, value) {
  const time = new Date().toTimeString().slice(0, 8);   // "14:23:07"
  const type = getType(value);
  const text = formatValue(value);
  console.log(`${time} [${level.padEnd(5)}] [${type}] ${text}`);
}

log('INFO',  'Server started');   // 14:23:07 [INFO ] [string] "Server started"
log('WARN',  null);               // 14:23:07 [WARN ] [null] null
log('ERROR', { code: 503 });      // 14:23:07 [ERROR] [object] {"code":503}
```

`padEnd(5)` left-aligns the level label so all output lines up in columns — that's the string method `.padEnd()` at work. Now messages are visually scannable.

### v3 — A Logger Factory with History

Production apps create multiple loggers (one per module), each with its own name and a minimum level filter. A *factory function* — a function that returns an object — is the right tool:

```javascript
// v3: createLogger() returns a logger object with private history
function createLogger(name, minLevel = 'DEBUG') {
  const history = [];  // private — only accessible through the returned object

  function log(level, value) {
    if (LEVELS[level] < LEVELS[minLevel]) return;   // below threshold: skip

    const entry = {
      time:      new Date().toTimeString().slice(0, 8),
      level,
      type:      getType(value),
      formatted: formatValue(value),
      raw:       value,        // keep the original too
      name
    };

    history.push(entry);
    return entry;
  }

  return {
    debug: (v) => log('DEBUG', v),
    info:  (v) => log('INFO',  v),
    warn:  (v) => log('WARN',  v),
    error: (v) => log('ERROR', v),
    get history() { return history; },
    clear: () => { history.length = 0; }
  };
}

const log = createLogger('auth', 'WARN');  // only WARN and above
log.debug('checking token');   // silently skipped
log.warn('token expiring');    // logged
log.error('token invalid');    // logged
```

`history.length = 0` clears an array *in place* — it doesn't create a new array, it truncates the existing one. That `get history()` syntax is a **getter** — it looks like a property but calls a function, which lets you read `logger.history` without the `()`.

---

## The Complete Program

`logger.html` — open it in your browser. Type any value and click "Log It". Click "Load Samples" to see every JavaScript type logged at once.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Build Your Own Logger — Chapter 1</title>
<!-- styles in logger.html -->
</head>
<body>
<script>
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: miniLogger
// ═══════════════════════════════════════════════════════════════════════

const LEVELS = { DEBUG: 0, INFO: 1, WARN: 2, ERROR: 3 };

function getType(value) {
  if (value === null)       return 'null';
  if (Array.isArray(value)) return 'array';
  return typeof value;
}

function formatValue(value) {
  const type = getType(value);
  if (type === 'string')                     return `"${value}"`;
  if (type === 'null' || type === 'undefined') return String(value);
  if (type === 'array' || type === 'object') return JSON.stringify(value);
  if (type === 'function')
    return value.name ? `[Function: ${value.name}]` : '[Function (anonymous)]';
  return String(value);
}

function createLogger(name, minLevel = 'DEBUG') {
  const history = [];

  function log(level, value) {
    if (LEVELS[level] < LEVELS[minLevel]) return;
    const entry = {
      time:      new Date().toTimeString().slice(0, 8),
      level, name,
      type:      getType(value),
      formatted: formatValue(value),
      raw:       value
    };
    history.push(entry);
    return entry;
  }

  return {
    debug: (v) => log('DEBUG', v),
    info:  (v) => log('INFO',  v),
    warn:  (v) => log('WARN',  v),
    error: (v) => log('ERROR', v),
    get history() { return history; },
    clear: () => { history.length = 0; }
  };
}
</script>
</body>
</html>
```

---

## Walkthrough

### The Seven Types

JavaScript values have exactly eight types (seven primitives plus object). `typeof` reveals six of them:

```
JavaScript Type System
───────────────────────────────────────────────────────────
Value            typeof result    getType() result
─────────────    ─────────────    ─────────────────
"hello"          'string'         'string'
42               'number'         'number'
3.14             'number'         'number'
true / false     'boolean'        'boolean'
undefined        'undefined'      'undefined'
null             'object'  ← bug  'null'       ← fixed
{ a: 1 }         'object'         'object'
[1, 2, 3]        'object'  ← vague 'array'     ← precise
function f() {}  'function'       'function'
```

`null` and `[]` both return `'object'` from `typeof`. Our `getType()` handles both correctly — checking `null` first (with `===`), then checking `Array.isArray()`, then falling through to `typeof`.

**Why `===` instead of `==`?**

`===` is strict equality: it checks value *and* type. `==` is loose equality: it coerces types before comparing, which produces surprises:

```javascript
0 == false    // true  — loose: coerces false to 0
0 === false   // false — strict: different types

null == undefined   // true  — loose
null === undefined  // false — strict

"" == 0    // true  — loose
"" === 0   // false — strict
```

Always use `===`. The loose equality rules are a famous source of bugs.

### Template Literals

The backtick syntax `` `${expression}` `` embeds any JavaScript expression inside a string:

```javascript
const level = 'WARN';
const type  = 'string';
const text  = '"hello"';

// Old way (concatenation):
const line1 = '[' + level + '] [' + type + '] ' + text;

// Template literal:
const line2 = `[${level}] [${type}] ${text}`;

// Both produce:  '[WARN] [string] "hello"'
```

The expression inside `${}` can be anything: a variable, a function call, arithmetic, a ternary:

```javascript
`Count: ${items.length}`
`Time: ${new Date().toTimeString().slice(0, 8)}`
`${n > 0 ? 'positive' : 'non-positive'}`
```

Template literals also support multi-line strings without `\n`:

```javascript
const report = `
  Level: ${level}
  Type:  ${type}
  Value: ${text}
`;
```

### String Methods

The logger uses three string methods. Here they are with context:

```javascript
// .slice(start, end) — extract a substring
new Date().toTimeString()           // "14:23:07 GMT+0000 ..."
new Date().toTimeString().slice(0, 8) // "14:23:07"

// .padEnd(width) — pad with spaces to reach target width
'INFO'.padEnd(5)    // "INFO "   (1 space added)
'DEBUG'.padEnd(5)   // "DEBUG"   (already 5 chars)
'WARN'.padEnd(5)    // "WARN "   (1 space added)
// result: all level labels align in a column

// String() — convert anything to a string
String(null)        // "null"
String(undefined)   // "undefined"
String(42)          // "42"
String(true)        // "true"
String(false)       // "false"
```

`JSON.stringify()` is not a string method — it's a global function. It converts an object or array to a JSON string:

```javascript
JSON.stringify({ name: 'ada', age: 36 })   // '{"name":"ada","age":36}'
JSON.stringify([1, 2, 3])                   // '[1,2,3]'
JSON.stringify(null)                        // 'null'
JSON.stringify("hello")                     // '"hello"'  (adds quotes!)
```

### Objects as Return Values

`createLogger()` returns an object — a collection of named methods. This pattern is called a **factory function**:

```
createLogger('auth') returns:
┌────────────────────────────────────────────┐
│  {                                         │
│    debug:   function(v) { log('DEBUG',v) } │
│    info:    function(v) { log('INFO', v) } │
│    warn:    function(v) { log('WARN', v) } │
│    error:   function(v) { log('ERROR',v) } │
│    history: [getter → history array]        │
│    clear:   function() { history.length=0 }│
│  }                                         │
└────────────────────────────────────────────┘
  All methods share the same private `history` array
  via closure — they were all created in the same
  function call, so they all close over the same scope.
```

The object returned by `createLogger()` is the *public API*. The `history` array is *private* — there's no way to access it from outside except through the `history` getter and `clear()` method. This is how JavaScript implements encapsulation without classes.

### null vs undefined

Two values represent "nothing" — and they mean different things:

```javascript
let x;              // x is undefined — declared but never assigned
let y = null;       // y is null — intentionally empty

typeof undefined    // 'undefined'
typeof null         // 'object' (the famous bug)

null === undefined  // false (strict: different types)
null == undefined   // true  (loose: both "nothing")
```

Convention: use `null` when you *set* something to empty. Use `undefined` when something has never been set. The logger distinguishes them in `getType()` because they mean different things to the caller.

---

## Guided Try It — Add a `count()` method

**The goal**: add a `logger.count(level)` method that returns how many entries have been logged at a given level. `logger.count('ERROR')` should return the number of error entries.

**Why this is useful**: production monitoring dashboards show error counts. You'd check `logger.count('ERROR') > 0` before sending an alert.

**Step 1 — Loop through history and count matching entries**

The `history` array holds every entry. Each entry has a `.level` property. Count the ones that match:

```javascript
// Add this to the returned object inside createLogger():
count: function(level) {
  let n = 0;
  for (const entry of history) {
    if (entry.level === level) n++;
  }
  return n;
}
```

Try it:
```javascript
const log = createLogger('app');
log.info('started');
log.warn('slow response');
log.error('timeout');
log.error('retry failed');

log.count('ERROR');  // → 2
log.count('INFO');   // → 1
log.count('DEBUG');  // → 0
```

**Step 2 — Support counting all levels at once**

If called with no argument, return an object with counts for every level:

```javascript
count: function(level) {
  if (level !== undefined) {
    // Count specific level (code from step 1)
    let n = 0;
    for (const entry of history) {
      if (entry.level === level) n++;
    }
    return n;
  }

  // No argument: count all levels
  const counts = { DEBUG: 0, INFO: 0, WARN: 0, ERROR: 0 };
  for (const entry of history) {
    counts[entry.level]++;
  }
  return counts;
}
```

Test:
```javascript
log.count();
// → { DEBUG: 0, INFO: 1, WARN: 1, ERROR: 2 }
```

**Step 3 — Use the count to trigger a warning**

Now use `count()` to build a "circuit breaker" — if too many errors pile up, escalate:

```javascript
function logWithEscalation(logger, level, value) {
  logger[level.toLowerCase()](value);

  if (level === 'ERROR' && logger.count('ERROR') >= 3) {
    // Three errors in a row: escalate to a notice
    console.warn(`⚠ ${logger.count('ERROR')} errors logged — consider alerting`);
  }
}
```

**Think about it**: the `for...of` loop in `count()` scans the entire history every time it's called. If you logged 10,000 entries and called `count()` after each one, that's an O(n²) operation. How would you make `count()` O(1)? (Hint: maintain a separate `counts` object that you update inside the `log()` function instead of scanning history.)

---

## Exercises

1. **Add a `minLevel` filter**: Modify `createLogger` to accept a second argument, `minLevel`. When `minLevel = 'WARN'`, calls to `.debug()` and `.info()` should be silently ignored. Use the `LEVELS` object to compare numeric priorities. Test: `createLogger('app', 'WARN').debug('ignored')` should log nothing.

2. **Format numbers with precision**: Update `formatValue` so numbers with decimal points are displayed with exactly 2 decimal places. `formatValue(3.14159)` → `"3.14"`, `formatValue(42)` → `"42"` (no decimal). Use `Number.isInteger()` to decide, and `.toFixed(2)` to format.

3. **Add a `last(n)` method**: Add `logger.last(3)` that returns the last 3 entries from history. If there are fewer than `n` entries, return all of them. Use `Array.prototype.slice` with a negative index.

4. **Add a `search(keyword)` method**: Add `logger.search('error')` that returns all entries where the formatted value contains the keyword (case-insensitive). Use `.toLowerCase()` and `.includes()` on the `formatted` field of each entry.

5. **Build a `createChild(name)` method**: Add `logger.createChild('auth')` that returns a new logger whose entries are also pushed into the *parent's* history. When the child logs something, it should appear in both `child.history` and `parent.history`. Think carefully about how closure and the shared `history` array make this possible.

---

## Solutions

### Exercise 1 — `minLevel` filter

```javascript
function createLogger(name, minLevel = 'DEBUG') {
  const history = [];

  function log(level, value) {
    if (LEVELS[level] < LEVELS[minLevel]) return;  // ← new: skip if below threshold

    const entry = {
      time:      new Date().toTimeString().slice(0, 8),
      level, name,
      type:      getType(value),
      formatted: formatValue(value),
      raw:       value
    };
    history.push(entry);
    return entry;
  }

  return {
    debug: (v) => log('DEBUG', v),
    info:  (v) => log('INFO',  v),
    warn:  (v) => log('WARN',  v),
    error: (v) => log('ERROR', v),
    get history() { return history; },
    clear: () => { history.length = 0; }
  };
}

// Test:
const prodLog = createLogger('api', 'WARN');
prodLog.debug('connection pool initialized');  // silently dropped
prodLog.info('request received');              // silently dropped
prodLog.warn('slow query: 850ms');             // logged
prodLog.error('query timeout');                // logged
prodLog.history.length;  // → 2
```

### Exercise 2 — Number precision

```javascript
function formatValue(value) {
  const type = getType(value);

  if (type === 'string')                     return `"${value}"`;
  if (type === 'null' || type === 'undefined') return String(value);
  if (type === 'array' || type === 'object') return JSON.stringify(value);
  if (type === 'function')
    return value.name ? `[Function: ${value.name}]` : '[Function (anonymous)]';

  // Numbers: show decimals only when present
  if (type === 'number') {
    return Number.isInteger(value) ? String(value) : value.toFixed(2);
  }

  return String(value);
}

// Tests:
formatValue(42);        // "42"
formatValue(3.14159);   // "3.14"
formatValue(1.0);       // "1"      (integer)
formatValue(-0.005);    // "0.00"   (.toFixed rounds to even)
formatValue(NaN);       // "NaN"    (NaN is a number type!)
formatValue(Infinity);  // "Infinity"
```

### Exercise 3 — `last(n)`

```javascript
// Inside createLogger, add to the returned object:
last: function(n = 1) {
  return history.slice(-n);
  // slice(-3) on [a,b,c,d,e] → [c,d,e]
  // slice(-99) on [a,b] → [a,b]  (negative bigger than length: returns all)
}

// Test:
const log = createLogger('app');
['one','two','three','four','five'].forEach(v => log.info(v));
log.last(3).map(e => e.formatted);  // ['"three"', '"four"', '"five"']
log.last(1)[0].formatted;           // '"five"'
log.last(99).length;                // 5  (all of them)
```

### Exercise 4 — `search(keyword)`

```javascript
// Inside createLogger, add to the returned object:
search: function(keyword) {
  const lower = keyword.toLowerCase();
  const results = [];
  for (const entry of history) {
    if (entry.formatted.toLowerCase().includes(lower)) {
      results.push(entry);
    }
  }
  return results;
}

// Test:
const log = createLogger('app');
log.info('User login: alice');
log.warn('User alice made 3 failed attempts');
log.error('User bob locked out');
log.debug(42);

log.search('alice').length;   // 2
log.search('user').length;    // 3
log.search('42').length;      // 1  (number formatted as "42")
log.search('xyz').length;     // 0
```

### Exercise 5 — `createChild(name)`

```javascript
function createLogger(name, minLevel = 'DEBUG', parentHistory = null) {
  // If a parent passed its history, we write to it too
  const history = parentHistory || [];

  function log(level, value) {
    if (LEVELS[level] < LEVELS[minLevel]) return;

    const entry = {
      time:      new Date().toTimeString().slice(0, 8),
      level, name,
      type:      getType(value),
      formatted: formatValue(value),
      raw:       value
    };
    history.push(entry);

    // If we have a parent history, also push there
    // (Only relevant for child loggers — they pass parentHistory below)
    return entry;
  }

  return {
    debug: (v) => log('DEBUG', v),
    info:  (v) => log('INFO',  v),
    warn:  (v) => log('WARN',  v),
    error: (v) => log('ERROR', v),
    get history() { return history; },
    clear: () => { history.length = 0; },

    createChild: function(childName) {
      // Child shares the same history array — same reference, not a copy
      return createLogger(`${name}:${childName}`, minLevel, history);
    }
  };
}

// Test:
const root  = createLogger('app');
const auth  = root.createChild('auth');
const db    = root.createChild('db');

root.info('app started');
auth.warn('token expiring');
db.error('query failed');

root.history.length;  // 3 — sees all entries
auth.history.length;  // 3 — shares the same array
db.history.length;    // 3 — shares the same array

// Each entry's .name tells you which logger wrote it:
root.history.map(e => e.name);  // ['app', 'app:auth', 'app:db']
```

The key insight: `createChild` passes the parent's `history` array by *reference*, not by value. Both the parent and child hold a reference to the *same* array object in memory. When either pushes an entry, both see it.

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `typeof` | Returns the type name as a string; `typeof null === 'object'` is a bug |
| `===` vs `==` | Always use `===`; loose `==` coerces types in surprising ways |
| `null` vs `undefined` | `null` = intentionally empty; `undefined` = never set |
| `Array.isArray()` | The right way to check for arrays (typeof says 'object') |
| Template literals | `` `${expr}` `` embeds any expression; supports multi-line |
| `String()` | Converts any value to a string safely |
| `JSON.stringify()` | Converts objects/arrays to JSON notation strings |
| `.slice(start, end)` | Extracts a substring or subarray |
| `.padEnd(n)` | Pads a string to a minimum width with spaces |
| Factory function | A function that returns an object with methods |
| Closure | The returned methods "remember" `history` from their creation scope |
| `history.length = 0` | Truncates an array in-place without creating a new one |

---

## Building with Claude

Bad prompt:
> "How do I log things in JavaScript?"

Good prompt:
> "I'm building a logger in plain JavaScript (no libraries). I have `getType(value)` that fixes the `typeof null` bug and detects arrays, and `formatValue(value)` that wraps strings in quotes and JSON-stringifies objects. Now I want to add a `minLevel` filter so `createLogger('app', 'WARN')` silently drops DEBUG and INFO messages. I have a `LEVELS` object mapping level names to numbers. Should I check `LEVELS[level] < LEVELS[minLevel]` at the start of the `log()` function, or store the numeric threshold at creation time? What's the trade-off?"

The good prompt shows your exact code, explains what you've already built, and frames a real design question. Claude's answer will be specific to *your* implementation — not a generic tutorial on logging.

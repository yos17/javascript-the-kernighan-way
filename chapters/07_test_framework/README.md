# Chapter 7 — Build Your Own Test Framework

Once you start writing larger programs, you need a way to check that they still work after changes. A test framework sounds advanced, but the core idea is beginner-friendly: run a function, and if it throws, the test failed. This chapter builds from that small idea step by step.

---

## The Problem

You have a math utility library — `add`, `multiply`, `factorial`, `isPrime`, `arraySum`. How do you know it works?

You could run it manually and check the output. But that doesn't scale, doesn't repeat automatically, and doesn't tell you which specific case broke. And when a bug gets fixed, how do you know the fix didn't break something else that was working before?

This is the real problem test frameworks solve: not just running checks once, but making correctness *verifiable on demand, for every case, with precise failure messages*. A test suite is a machine you build once and run forever. Build the machine yourself and the black box becomes a handful of familiar ideas: arrays, try/catch, and functions that throw.

---

## Test Lifecycle

```
Test Lifecycle
──────────────────────────────────────────────────────────
describe('Math', () => {        ← runs IMMEDIATELY
  it('add works', () => { ... }) ← REGISTERS test (doesn't run yet)
  it('mul works', () => { ... }) ← REGISTERS test
})                               ← describe returns

runner.run()                     ← NOW tests execute
  ├─ beforeEach() hooks
  ├─ try { test.fn() }           ← if no throw → PASS ✓
  │        └─ expect(x).toBe(y)  ← throws AssertionError if wrong
  └─ catch (err) → FAIL ✗
       └─ err.message → "Expected 4 to be 5"
```

`describe` runs *immediately* to collect registrations. `run()` executes them later. This two-phase design — collect first, execute second — is why test frameworks can report results for all tests even when early ones fail.

---

## Building It Step by Step

### v1 — The Core Insight (15 lines)

The entire foundation of any test framework is this: **a passing test is a function that doesn't throw; a failing test is a function that does.** Nothing else.

```javascript
function run(tests) {
  let passed = 0, failed = 0;

  for (const { name, fn } of tests) {
    try {
      fn();
      console.log(`  ✓ ${name}`);
      passed++;
    } catch (err) {
      console.log(`  ✗ ${name}: ${err.message}`);
      failed++;
    }
  }

  console.log(`\n${passed} passed, ${failed} failed`);
}

// Usage:
run([
  { name: 'add(2,3) is 5', fn: () => { if (add(2, 3) !== 5) throw new Error('Expected 5'); } },
  { name: 'add(0,0) is 0', fn: () => { if (add(0, 0) !== 0) throw new Error('Expected 0'); } },
]);
```

Why this works: `run` doesn't know or care what the test does. It only cares whether calling it throws. If the test function throws, that's a failure. If it returns normally, that's a pass.

The limitation of v1: tests live in a flat list, assertions are verbose, and there's no grouping. The structure doesn't match how we think about software — "the `add` function" contains several related cases.

### v2 — Structure: `describe()` and `it()`

Add two functions: `describe` names a group of related tests, and `it` registers a single test case. Neither runs tests yet — they just organize the collection phase.

```javascript
class MiniTest {
  constructor() {
    this.suites = [];
    this.currentSuite = null;
  }

  describe(name, fn) {
    const suite = { name, tests: [] };
    this.suites.push(suite);
    const prev = this.currentSuite;
    this.currentSuite = suite;
    fn();                      // runs immediately — collects it() calls
    this.currentSuite = prev;  // restore, so describe() can nest
  }

  it(name, fn) {
    this.currentSuite.tests.push({ name, fn });
  }

  run() {
    for (const suite of this.suites) {
      console.log(`\n${suite.name}`);
      for (const test of suite.tests) {
        try {
          test.fn();
          console.log(`  ✓ ${test.name}`);
        } catch (err) {
          console.log(`  ✗ ${test.name}: ${err.message}`);
        }
      }
    }
  }
}

// Usage:
const t = new MiniTest();
t.describe('add', () => {
  t.it('returns sum of two numbers', () => {
    if (add(2, 3) !== 5) throw new Error('Expected 5');
  });
});
t.run();
```

The key move: `describe`'s callback runs *synchronously right now*, not later. This is what populates `suite.tests`. The `prev`/restore pattern for `currentSuite` is what lets you nest `describe` blocks.

The remaining problem: throwing raw `new Error('Expected 5')` inside every test is tedious and inconsistent. We need an assertion API.

### v3 — Assertions: `expect()` with Matchers and `.not`

Add `expect(actual)` which returns an object of *matcher functions*. Each matcher throws a descriptive `AssertionError` on failure and does nothing on success.

```javascript
function expect(actual) {
  function fail(msg) {
    const err = new Error(msg);
    err.name = 'AssertionError';
    throw err;
  }

  const matchers = {
    toBe(expected) {
      if (!Object.is(actual, expected))
        fail(`Expected ${JSON.stringify(actual)} to be ${JSON.stringify(expected)}`);
    },
    toEqual(expected) {
      if (JSON.stringify(actual) !== JSON.stringify(expected))
        fail(`Expected ${JSON.stringify(actual)} to equal ${JSON.stringify(expected)}`);
    },
    toBeTruthy() {
      if (!actual) fail(`Expected ${JSON.stringify(actual)} to be truthy`);
    },
    // ... more matchers
  };

  // Auto-generate .not inverses for every matcher
  matchers.not = {};
  for (const [name, fn] of Object.entries(matchers)) {
    if (name === 'not') continue;
    matchers.not[name] = function(...args) {
      let threw = false;
      try { fn.apply(null, args); } catch (e) { threw = true; }
      if (!threw) fail(`Expected NOT to satisfy .${name}()`);
    };
  }

  return matchers;
}

// Usage is now readable:
t.it('returns sum', () => expect(add(2, 3)).toBe(5));
t.it('not negative', () => expect(add(2, 3)).not.toBeLessThan(0));
```

Why `.not` is auto-generated rather than hand-written: if you have 12 matchers, writing 12 inverses by hand means 24 functions that must stay in sync. The loop generates all 12 inverses from one rule — "if the original wouldn't have thrown, I throw instead."

---

## The Complete Program

`test-framework.html` — open it in your browser to see a live test runner with an editable test file.

```javascript
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: MiniTest
// ═══════════════════════════════════════════════════════════════════════

class MiniTest {
  constructor() { this.reset(); }

  reset() {
    this.suites       = [];
    this.currentSuite = null;
    this._beforeEachStack = [[]];
    this._afterEachStack  = [[]];
  }

  describe(name, fn) {
    const suite = { name, tests: [] };
    this.suites.push(suite);
    const prev = this.currentSuite;
    this.currentSuite = suite;

    this._beforeEachStack.push([]);
    this._afterEachStack.push([]);
    fn(); // collect it() calls
    this._beforeEachStack.pop();
    this._afterEachStack.pop();

    this.currentSuite = prev;
  }

  beforeEach(fn) {
    this._beforeEachStack[this._beforeEachStack.length - 1].push(fn);
  }
  afterEach(fn) {
    this._afterEachStack[this._afterEachStack.length - 1].push(fn);
  }

  it(name, fn) {
    if (!this.currentSuite) {
      this.currentSuite = { name: '(top level)', tests: [] };
      this.suites.push(this.currentSuite);
    }
    this.currentSuite.tests.push({ name, fn, skip: fn === undefined });
  }

  run() {
    const results = [];
    for (const suite of this.suites) {
      const suiteResult = { name: suite.name, tests: [] };
      for (const test of suite.tests) {
        if (test.skip) {
          suiteResult.tests.push({ name: test.name, status: 'skip', duration: 0 });
          continue;
        }
        for (const hook of suite._beforeEach || []) {
          try { hook(); } catch (e) { /* ignore */ }
        }
        const t0 = performance.now();
        try {
          test.fn();
          suiteResult.tests.push({
            name: test.name, status: 'pass',
            duration: Math.round(performance.now() - t0)
          });
        } catch (err) {
          suiteResult.tests.push({
            name: test.name, status: 'fail',
            error: err.message,
            duration: Math.round(performance.now() - t0)
          });
        }
        for (const hook of suite._afterEach || []) {
          try { hook(); } catch (e) { /* ignore */ }
        }
      }
      results.push(suiteResult);
    }
    return results;
  }
}

// ─── expect() ─────────────────────────────────────────────────────────

function expect(actual) {
  function fail(msg) {
    const err = new Error(msg);
    err.name = 'AssertionError';
    throw err;
  }

  const matchers = {
    toBe(expected) {
      if (!Object.is(actual, expected))
        fail(`Expected ${format(actual)} to be ${format(expected)}`);
    },
    toEqual(expected) {
      if (JSON.stringify(actual) !== JSON.stringify(expected))
        fail(`Expected ${format(actual)} to equal ${format(expected)}`);
    },
    toContain(item) {
      if (Array.isArray(actual) && !actual.includes(item))
        fail(`Expected ${format(actual)} to contain ${format(item)}`);
      else if (typeof actual === 'string' && !actual.includes(item))
        fail(`Expected "${actual}" to contain "${item}"`);
    },
    toThrow(expectedMsg) {
      if (typeof actual !== 'function') fail(`Expected a function`);
      let threw = false;
      try { actual(); } catch (err) {
        threw = true;
        if (expectedMsg !== undefined && !err.message.includes(expectedMsg))
          fail(`Expected to throw "${expectedMsg}", got "${err.message}"`);
      }
      if (!threw) fail(`Expected function to throw but it did not`);
    },
    toBeTruthy()   { if (!actual) fail(`Expected ${format(actual)} to be truthy`); },
    toBeFalsy()    { if (actual)  fail(`Expected ${format(actual)} to be falsy`); },
    toBeNull()     { if (actual !== null) fail(`Expected null`); },
    toBeUndefined(){ if (actual !== undefined) fail(`Expected undefined`); },
    toBeGreaterThan(n) { if (actual <= n) fail(`Expected ${actual} > ${n}`); },
    toBeLessThan(n)    { if (actual >= n) fail(`Expected ${actual} < ${n}`); },
    toBeCloseTo(n, p = 2) {
      const f = Math.pow(10, p);
      if (Math.round(actual * f) !== Math.round(n * f))
        fail(`Expected ${actual} to be close to ${n}`);
    },
    toMatch(pattern) {
      if (!new RegExp(pattern).test(String(actual)))
        fail(`Expected "${actual}" to match ${pattern}`);
    },
    toHaveLength(len) {
      if (!actual || actual.length !== len)
        fail(`Expected length ${len}, got ${actual?.length}`);
    },
  };

  // .not inverts each matcher
  matchers.not = {};
  for (const [name, fn] of Object.entries(matchers)) {
    if (name === 'not') continue;
    matchers.not[name] = function() {
      let threw = false;
      try { fn.apply(null, arguments); } catch (e) { threw = true; }
      if (!threw) fail(`Expected ${format(actual)} NOT to satisfy .${name}()`);
    };
  }

  return matchers;
}
```

---

## Walkthrough

### `describe()` — Grouping Tests

`describe` has one job: give a name to a group of tests. Its second argument is a function that runs immediately — not later, not asynchronously. It runs right now, synchronously, to *collect* the tests defined inside it.

```javascript
describe('Math utilities', () => {
  it('add works', () => { expect(add(2, 3)).toBe(5); });
  it('multiply works', () => { expect(multiply(3, 4)).toBe(12); });
});
```

When `describe` runs, it sets `this.currentSuite` so that every `it()` call inside the function knows which suite to attach itself to. When the function finishes, `currentSuite` is restored. This is why describe blocks can be nested — each call pushes and pops a context:

```javascript
describe(name, fn) {
  const suite = { name, tests: [] };
  this.suites.push(suite);
  const prev = this.currentSuite;
  this.currentSuite = suite;
  fn();                     // ← runs immediately to collect tests
  this.currentSuite = prev; // ← restore, enabling nested describes
}
```

### `it()` — Registering a Test

`it()` doesn't run the test. It *registers* it — pushes a `{ name, fn }` object onto the current suite's `tests` array. The actual execution happens later in `run()`.

```javascript
it('add returns the sum', () => {
  expect(add(2, 3)).toBe(5);
});
```

This separation — collect first, execute second — is what allows the framework to report results for the whole suite, even if one test fails. If tests ran immediately inside `it()`, an error in the first test would prevent the rest from registering.

### `run()` — The Engine

`run()` loops through every registered test and executes it inside a `try/catch`. That's the core insight: **a passing test is one that doesn't throw. A failing test is one that does.**

```javascript
run() {
  for (const suite of this.suites) {
    for (const test of suite.tests) {
      const t0 = performance.now();
      try {
        test.fn();          // ← run the test function
        // if we get here: PASS
        results.push({ status: 'pass', ... });
      } catch (err) {
        // if an error was thrown: FAIL
        results.push({ status: 'fail', error: err.message, ... });
      }
    }
  }
}
```

`performance.now()` measures each test's duration in milliseconds — useful for spotting slow tests.

### `expect()` — Assertions

`expect(actual)` returns an object of *matchers*. Each matcher is a function that throws if the assertion fails, and does nothing if it passes.

```javascript
function expect(actual) {
  return {
    toBe(expected) {
      if (!Object.is(actual, expected)) {
        throw new Error(`Expected ${actual} to be ${expected}`);
      }
      // if we reach here: assertion passed — no throw
    },
    // ...more matchers
  };
}
```

`Object.is` is used instead of `===` because it handles two edge cases correctly: `Object.is(NaN, NaN)` is `true` (whereas `NaN === NaN` is `false`), and `Object.is(0, -0)` is `false` (whereas `0 === -0` is `true`).

`toEqual` uses `JSON.stringify` for deep comparison — simple and effective for plain objects and arrays, though it won't handle functions, `undefined` in objects, or circular references:

```javascript
toEqual(expected) {
  if (JSON.stringify(actual) !== JSON.stringify(expected)) {
    throw new Error(`Expected ${format(actual)} to equal ${format(expected)}`);
  }
}
```

### `.not` — Inverting Matchers

The `.not` property is generated automatically by wrapping each matcher in a `try/catch` and flipping the logic:

```javascript
matchers.not = {};
for (const [name, fn] of Object.entries(matchers)) {
  if (name === 'not') continue;
  matchers.not[name] = function() {
    let threw = false;
    try { fn.apply(null, arguments); } catch (e) { threw = true; }
    if (!threw) throw new Error(`Expected NOT to satisfy .${name}()`);
  };
}
```

If the original matcher *would have thrown* (assertion failed), `.not` catches that and considers it a pass. If the original *didn't* throw (assertion passed), `.not` throws — inverting the result.

```javascript
expect(nums).not.toContain(99);  // passes if 99 is not in nums
expect('hello').not.toMatch(/\d/); // passes if no digits found
```

### `beforeEach()` — Setup Per Test

`beforeEach` registers a function that runs before every `it()` in the current suite. The pattern is essential for resetting state between tests:

```javascript
describe('Array utilities', () => {
  let nums;

  beforeEach(() => {
    nums = [3, 1, 4, 1, 5, 9, 2, 6]; // ← fresh array for each test
  });

  it('sum adds all elements', () => {
    expect(arraySum(nums)).toBe(31);
    // nums hasn't been mutated by any prior test
  });
});
```

Without `beforeEach`, if one test modified `nums`, the next test would start with the modified version — and the test results would depend on execution order, which is a testing anti-pattern.

### `toThrow()` — Testing Error Cases

Testing that a function throws is a first-class use case. `toThrow` wraps the call in its own `try/catch`:

```javascript
toThrow(expectedMsg) {
  if (typeof actual !== 'function') fail(`Expected a function`);
  let threw = false;
  try { actual(); } catch (err) {
    threw = true;
    if (expectedMsg && !err.message.includes(expectedMsg))
      fail(`Expected "${expectedMsg}", got "${err.message}"`);
  }
  if (!threw) fail(`Expected function to throw but it did not`);
},
```

The value passed to `expect()` must be a *function*, not the result of calling it — because we need to control when it runs:

```javascript
// Wrong — factorial(-1) throws before expect() sees anything
expect(factorial(-1)).toThrow();

// Correct — we wrap it in a function; toThrow() calls it safely
expect(() => factorial(-1)).toThrow();
```

---

## Guided Try It — Adding Async Test Support

**The problem**: You want to test `fetchUser(1)`, which returns a Promise. You write:

```javascript
it('fetches user data', async () => {
  const user = await fetchUser(1);
  expect(user.name).toBe('Ada');
});
```

You run the tests. They all show green. But `fetchUser` is broken — the server is down and it always rejects. Why did the test pass?

**Hint: look at `run()` in the complete program.** Find the line that calls `test.fn()`. What does it do with the return value?

**Step 1 — Spot the bug.**

The current `run()` ignores the return value:

```javascript
try {
  test.fn();   // ← return value discarded
  suiteResult.tests.push({ status: 'pass', ... });
} catch (err) {
  suiteResult.tests.push({ status: 'fail', ... });
}
```

When `test.fn` is an `async` function, calling it starts the async work and immediately returns a Promise — it doesn't throw. So `run()` sees no throw, records a pass, and moves on. The Promise rejects later, but nobody is listening.

**Step 2 — Capture the return value.**

```javascript
const result = test.fn();
```

Now we have the return value. How do we know if it's a Promise? We check for a `.then` method — the "thenable" duck-type check used throughout JavaScript:

```javascript
if (result && typeof result.then === 'function') {
  // it's a Promise (or thenable)
}
```

**Step 3 — Await the Promise.**

```javascript
const result = test.fn();
if (result && typeof result.then === 'function') {
  await result;
}
```

But `await` requires us to be inside an `async` function. That means `run()` itself must become `async`:

```javascript
async run() {
  const results = [];
  for (const suite of this.suites) {
    const suiteResult = { name: suite.name, tests: [] };
    for (const test of suite.tests) {
      const t0 = performance.now();
      try {
        const result = test.fn();
        if (result && typeof result.then === 'function') {
          await result;   // ← wait for the Promise to settle
        }
        suiteResult.tests.push({ name: test.name, status: 'pass',
          duration: Math.round(performance.now() - t0) });
      } catch (err) {
        suiteResult.tests.push({ name: test.name, status: 'fail',
          error: err.message, duration: Math.round(performance.now() - t0) });
      }
    }
    results.push(suiteResult);
  }
  return results;
}
```

When the Promise rejects, `await result` throws — and the `catch` block catches it. The test now correctly fails.

**Step 4 — Call site.**

Because `run()` is now `async`, you need to `await` it at the call site too:

```javascript
const results = await runner.run();
renderResults(results);
```

**Think about it**: What would happen if you forgot the `await` in `await result`? Write a concrete test case — a function that returns a rejected Promise — and trace through what `run()` would record without the `await`. What status would it show, and why would that be a misleading result?

---

## Exercises

1. **Implement `test.todo(name)`**: A way to mark planned but unwritten tests. `test.todo('validate email format')` should appear in results with a distinct "todo" status (neither pass nor fail) and no function required. Add it to the MiniTest class and render it with a different icon in the UI.

2. **Implement `expect.assertions(n)`**: Called at the top of a test to declare how many assertions are expected. If the test finishes with a different number of assertions having run, it should fail. Add an assertion counter to `expect()` and check it in `run()` after each test.

3. **Add async test support**: Currently all tests are synchronous. Modify `it()` to detect if the test function returns a `Promise`, and modify `run()` to `await` it. Tests that return promises should still be caught and reported correctly. Make `run()` itself `async`.

4. **Implement a `spy(fn)` function**: Returns a wrapper around `fn` that records every call. The spy should have `.calls` (array of argument lists), `.callCount`, and `.returnValues`. Add matchers `toHaveBeenCalled()` and `toHaveBeenCalledWith(...args)` to `expect()`.

5. **Implement `mock(obj, method, replacement)`**: Temporarily replaces `obj[method]` with `replacement` for the duration of a test. Returns a spy wrapping `replacement`. Provide a `.restore()` method that puts the original back. Wire it up so `afterEach` automatically calls `.restore()` on all active mocks.

---

## Solutions

### Exercise 1 — `test.todo(name)`

```javascript
// In MiniTest class:
it(name, fn) {
  const suite = this.currentSuite || this._getOrCreateTopLevel();
  suite.tests.push({ name, fn, skip: fn === undefined, todo: fn === undefined });
}
```

Add a static property:

```javascript
// Expose as test.todo outside the class:
const runner = new MiniTest();
const test = runner.it.bind(runner);
test.todo = (name) => runner.it(name, undefined); // fn is undefined → todo
```

In `run()`, detect the `todo` flag:

```javascript
if (test.skip || test.todo) {
  suiteResult.tests.push({ name: test.name, status: test.todo ? 'todo' : 'skip' });
  continue;
}
```

Render with a `☐` icon and distinct styling.

### Exercise 2 — `expect.assertions(n)`

```javascript
// Track assertion count per test in a module-level variable
let _expectedAssertions = null;
let _actualAssertions   = 0;

function expect(actual) {
  const matchers = {
    toBe(expected) {
      _actualAssertions++;
      if (!Object.is(actual, expected)) fail(`Expected ${format(actual)} to be ${format(expected)}`);
    },
    // ... increment _actualAssertions in every matcher
  };
  return matchers;
}

expect.assertions = function(n) {
  _expectedAssertions = n;
};

// In run(), after each test:
if (_expectedAssertions !== null && _actualAssertions !== _expectedAssertions) {
  // treat as failure
  suiteResult.tests.push({
    name: test.name, status: 'fail',
    error: `Expected ${_expectedAssertions} assertions, got ${_actualAssertions}`
  });
} else {
  suiteResult.tests.push({ name: test.name, status: 'pass' });
}
// Reset for next test:
_expectedAssertions = null;
_actualAssertions   = 0;
```

### Exercise 3 — Async tests

```javascript
async run() {
  const results = [];
  for (const suite of this.suites) {
    const suiteResult = { name: suite.name, tests: [] };
    for (const test of suite.tests) {
      const t0 = performance.now();
      try {
        const result = test.fn();
        // If the test returns a Promise, await it
        if (result && typeof result.then === 'function') {
          await result;
        }
        suiteResult.tests.push({ name: test.name, status: 'pass', duration: Math.round(performance.now() - t0) });
      } catch (err) {
        suiteResult.tests.push({ name: test.name, status: 'fail', error: err.message, duration: Math.round(performance.now() - t0) });
      }
    }
    results.push(suiteResult);
  }
  return results;
}
```

Usage:

```javascript
it('fetches user data', async () => {
  const user = await fetchUser(1);
  expect(user.name).toBe('Ada');
});
```

### Exercise 4 — `spy(fn)`

```javascript
function spy(fn = () => {}) {
  const calls = [];
  const returnValues = [];

  const wrapper = function() {
    const args = Array.from(arguments);
    let returnValue;
    try {
      returnValue = fn.apply(this, args);
    } catch(e) {
      calls.push(args);
      throw e;
    }
    calls.push(args);
    returnValues.push(returnValue);
    return returnValue;
  };

  wrapper.calls        = calls;
  wrapper.returnValues = returnValues;
  Object.defineProperty(wrapper, 'callCount', { get: () => calls.length });

  return wrapper;
}

// Matchers (add to the matchers object in expect()):
toHaveBeenCalled() {
  if (!actual.calls || actual.calls.length === 0)
    fail(`Expected spy to have been called`);
},
toHaveBeenCalledWith(...expectedArgs) {
  if (!actual.calls) fail(`Expected a spy`);
  const found = actual.calls.some(
    args => JSON.stringify(args) === JSON.stringify(expectedArgs)
  );
  if (!found)
    fail(`Expected spy to have been called with ${format(expectedArgs)}`);
},
```

### Exercise 5 — `mock(obj, method, replacement)`

```javascript
function mock(obj, method, replacement) {
  const original = obj[method];
  const wrapper  = spy(replacement || original);
  obj[method]    = wrapper;

  wrapper.restore = function() {
    obj[method] = original;
  };

  return wrapper;
}

// Usage:
describe('with mocked console', () => {
  let mockLog;

  beforeEach(() => {
    mockLog = mock(console, 'log', () => {});
  });

  afterEach(() => {
    mockLog.restore();
  });

  it('calls console.log', () => {
    someFunction(); // internally calls console.log
    expect(mockLog).toHaveBeenCalled();
  });
});
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `describe(name, fn)` | Runs `fn` immediately to collect tests; sets context for `it()` |
| `it(name, fn)` | Registers a test — does NOT run it yet |
| `run()` | The engine: loops through tests, wraps each in `try/catch` |
| Pass vs fail | A test passes if `fn()` doesn't throw; fails if it does |
| `expect(actual)` | Returns an object of matcher functions |
| Matcher | Throws `AssertionError` if check fails; does nothing if it passes |
| `Object.is` | Strict equality that handles `NaN` and `-0` correctly |
| `.not` | Auto-generated by inverting each matcher's throw/no-throw |
| `beforeEach` | Registers setup code; runs before every test in the suite |
| `toThrow` | Wraps the function call in `try/catch` — requires `() => fn()` |
| `performance.now()` | Microsecond timing for test duration measurement |
| Async tests | Check `typeof result.then === 'function'`; make `run()` async |

---

## Building with Claude

Bad prompt:
> "Write me a test framework."

Good prompt:
> "I've built a test framework with `describe`, `it`, `expect`, and a `run()` method that wraps each test in `try/catch`. It works for synchronous tests. Now I want to support async tests — if a test function returns a Promise, I need `run()` to await it before moving on. I think I need to make `run()` itself `async` and check `if (result && typeof result.then === 'function')`. Can you confirm this approach is correct and explain what would happen if I forgot the `await` — would the test silently pass even if the promise rejects?"

The prompt shows working code, names the mechanism, proposes a solution, and asks whether a specific mistake would cause a silent failure — the kind of subtle bug question that reveals real understanding.

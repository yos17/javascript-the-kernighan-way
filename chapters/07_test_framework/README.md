# Chapter 7 — Build Your Own Test Framework

Jest. Mocha. Vitest. Every JavaScript project uses one of them. But they're not magic — they're programs built from three simple ideas: a function that groups tests (`describe`), a function that runs one test (`it`), and a function that checks a value (`expect`). Build those three things yourself and you'll understand every test framework you ever encounter. You'll also understand why tests are written the way they are, what "assertion" actually means, and how `try/catch` becomes the engine of every test runner.

---

## The Problem

You have a math utility library — `add`, `multiply`, `factorial`, `isPrime`, `arraySum`. How do you know it works? You could run it manually and check the output. But that doesn't scale, doesn't repeat automatically, and doesn't tell you which specific case broke. A test framework solves this: it runs every case for you, catches failures, and reports exactly what went wrong and where.

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

## Try It

1. **Add a `.skip` test**: In the test editor, write `it.skip('this test is skipped', () => { ... })`. Add support for `it.skip` by checking if the test's `fn` is `undefined` — or add an `it.skip` property that sets `skip: true` without a function.

2. **Add `describe.only`**: When any `describe.only` block exists, only run suites marked with `.only`. Implement this by adding an `only` flag to suites and filtering in `run()`.

3. **Make `toEqual` handle `undefined`**: `JSON.stringify({a: undefined})` produces `"{}"`, losing the key. Write a custom deep equality function `deepEqual(a, b)` that handles `undefined`, `NaN`, and circular references (use a `Set` to track visited objects).

4. **Add test timing thresholds**: Add `.toBeFasterThan(ms)` — an assertion that the wrapped function runs in under `ms` milliseconds. Use `performance.now()` before and after calling it.

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

---

## Building with Claude

Bad prompt:
> "Write me a test framework."

Good prompt:
> "I've built a test framework with `describe`, `it`, `expect`, and a `run()` method that wraps each test in `try/catch`. It works for synchronous tests. Now I want to support async tests — if a test function returns a Promise, I need `run()` to await it before moving on. I think I need to make `run()` itself `async` and check `if (result && typeof result.then === 'function')`. Can you confirm this approach is correct and explain what would happen if I forgot the `await` — would the test silently pass even if the promise rejects?"

The prompt shows working code, names the mechanism, proposes a solution, and asks whether a specific mistake would cause a silent failure — the kind of subtle bug question that reveals real understanding.

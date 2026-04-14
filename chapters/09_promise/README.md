# Chapter 9 — Build Your Own Promise

`async/await` is the most important addition to modern JavaScript. But it's syntax sugar over Promises, and Promises are the most misunderstood part of the language. Developers use them daily without knowing why `.then()` returns a new Promise, what `queueMicrotask` does, or why an unhandled rejection silently swallows errors. In this chapter you build a Promise implementation from scratch — state machine, microtask queue, chaining, `.all()`, `.race()`, and `.finally()`. After this chapter, `async/await` will never be mysterious again.

---

## The Problem

Your app needs to fetch a user, then fetch their posts, then display both. These are sequential async operations — each depends on the last. With callbacks, this creates deeply nested "callback hell." Promises solve this by representing a future value as an object you can chain. Build the Promise implementation yourself and you'll understand every `.then()` you ever write.

---

## The Complete Program

`promise.html` — open it to see an interactive Promise timeline showing state transitions, microtask scheduling, and chained operations in real time.

```javascript
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: MyPromise
// ═══════════════════════════════════════════════════════════════════════

const PENDING   = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED  = 'rejected';

class MyPromise {
  constructor(executor) {
    this.state    = PENDING;
    this.value    = undefined;
    this.handlers = [];

    const resolve = (value) => {
      if (this.state !== PENDING) return;
      // If the resolved value is itself a thenable, adopt its state
      if (value && typeof value.then === 'function') {
        value.then(resolve, reject);
        return;
      }
      this.state = FULFILLED;
      this.value = value;
      this._runHandlers();
    };

    const reject = (reason) => {
      if (this.state !== PENDING) return;
      this.state = REJECTED;
      this.value = reason;
      this._runHandlers();
    };

    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }

  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      this.handlers.push({
        onFulfilled: typeof onFulfilled === 'function' ? onFulfilled : null,
        onRejected:  typeof onRejected  === 'function' ? onRejected  : null,
        resolve,
        reject,
      });
      this._runHandlers();
    });
  }

  catch(onRejected) {
    return this.then(null, onRejected);
  }

  finally(fn) {
    return this.then(
      (value)  => MyPromise.resolve(fn()).then(() => value),
      (reason) => MyPromise.resolve(fn()).then(() => { throw reason; })
    );
  }

  _runHandlers() {
    if (this.state === PENDING) return;

    for (const handler of this.handlers) {
      queueMicrotask(() => {
        const { onFulfilled, onRejected, resolve, reject } = handler;

        if (this.state === FULFILLED) {
          if (!onFulfilled) { resolve(this.value); return; }
          try   { resolve(onFulfilled(this.value)); }
          catch (err) { reject(err); }
        } else {
          if (!onRejected)  { reject(this.value); return; }
          try   { resolve(onRejected(this.value)); }
          catch (err) { reject(err); }
        }
      });
    }

    this.handlers = [];
  }

  static resolve(value) {
    if (value instanceof MyPromise) return value;
    return new MyPromise(resolve => resolve(value));
  }

  static reject(reason) {
    return new MyPromise((_, reject) => reject(reason));
  }

  static all(promises) {
    return new MyPromise((resolve, reject) => {
      const results = new Array(promises.length);
      let remaining = promises.length;
      if (remaining === 0) { resolve(results); return; }
      promises.forEach((p, i) => {
        MyPromise.resolve(p).then(
          value  => { results[i] = value; if (--remaining === 0) resolve(results); },
          reason => reject(reason)
        );
      });
    });
  }

  static race(promises) {
    return new MyPromise((resolve, reject) => {
      for (const p of promises) MyPromise.resolve(p).then(resolve, reject);
    });
  }

  static allSettled(promises) {
    return MyPromise.all(
      promises.map(p =>
        MyPromise.resolve(p).then(
          value  => ({ status: 'fulfilled', value }),
          reason => ({ status: 'rejected',  reason })
        )
      )
    );
  }
}
```

---

## Walkthrough

### A Promise Is a State Machine

A Promise can only ever be in one of three states, and transitions only move forward — never backward:

```
pending → fulfilled (resolved with a value)
pending → rejected  (rejected with a reason)
```

Once fulfilled or rejected, a Promise is **settled** and never changes state again. This is enforced by the guard at the top of both `resolve` and `reject`:

```javascript
const resolve = (value) => {
  if (this.state !== PENDING) return;  // ← ignore if already settled
  this.state = FULFILLED;
  this.value = value;
  this._runHandlers();
};
```

The state and value are stored as instance properties. Any number of `.then()` handlers can read them — the state machine makes this safe, because once set, they never change.

### The Executor — Synchronous Bridge to Async

The function you pass to `new Promise()` is the **executor**. It runs *synchronously*, and receives two functions: `resolve` and `reject`. Your code decides which one to call, and when.

```javascript
constructor(executor) {
  // ...set up state...
  try {
    executor(resolve, reject);  // ← runs RIGHT NOW, synchronously
  } catch (err) {
    reject(err);                // ← unhandled throws become rejections
  }
}
```

The `try/catch` around the executor is important: if you throw inside the executor, the Promise is rejected. Without it, the throw would escape the constructor and crash your program.

```javascript
// The executor can call resolve synchronously:
new MyPromise(resolve => resolve(42));  // settled immediately

// Or asynchronously, after some delay:
new MyPromise(resolve => setTimeout(resolve, 1000, 42));  // settled in 1 second
```

### `.then()` — The Heart of Chaining

Every call to `.then()` returns a **brand new Promise**. This is what enables chaining. The new Promise resolves with whatever value `onFulfilled` returns:

```javascript
then(onFulfilled, onRejected) {
  return new MyPromise((resolve, reject) => {
    this.handlers.push({
      onFulfilled: typeof onFulfilled === 'function' ? onFulfilled : null,
      onRejected:  typeof onRejected  === 'function' ? onRejected  : null,
      resolve,   // ← resolve/reject from the NEW promise
      reject,
    });
    this._runHandlers();
  });
}
```

The handler stores both the callback (`onFulfilled`) and the `resolve`/`reject` of the new promise. When the handler runs, it calls `onFulfilled` with the current value, then calls `resolve` with the *result* of that callback. This threads values through the chain.

```javascript
MyPromise.resolve(1)
  .then(n => n + 1)   // returns 2 → becomes value of next promise
  .then(n => n * 10)  // returns 20 → becomes value of next promise
  .then(n => console.log(n)); // logs 20
```

If `onFulfilled` throws, the error is caught and passed to `reject` of the new promise — so the next `.catch()` in the chain receives it.

### Thenable Resolution — Flattening Promise Chains

What happens when you return a Promise from `.then()`? If `resolve` receives a thenable (anything with a `.then` method), it delegates:

```javascript
const resolve = (value) => {
  if (this.state !== PENDING) return;
  if (value && typeof value.then === 'function') {
    value.then(resolve, reject);  // ← adopt the inner promise's state
    return;
  }
  // ... settle normally
};
```

This is why you can return a Promise from `.then()` and the chain flattens automatically:

```javascript
fetchUser(1)
  .then(user => fetchPosts(user.id))  // returns a Promise — not nested!
  .then(posts => renderPosts(posts)); // receives the posts, not the promise
```

Without this thenable check, returning a Promise from `.then()` would give you a Promise-of-a-Promise — and you'd have to unwrap it manually. The spec mandates this behaviour, and our implementation matches it.

### `_runHandlers()` — The Microtask Queue

Handlers are never called synchronously, even if the Promise is already settled. They're scheduled as **microtasks** using `queueMicrotask()`:

```javascript
_runHandlers() {
  if (this.state === PENDING) return;

  for (const handler of this.handlers) {
    queueMicrotask(() => {        // ← schedule, don't call immediately
      const { onFulfilled, onRejected, resolve, reject } = handler;
      if (this.state === FULFILLED) {
        if (!onFulfilled) { resolve(this.value); return; }
        try   { resolve(onFulfilled(this.value)); }
        catch (err) { reject(err); }
      } else {
        // ... similar for rejected
      }
    });
  }

  this.handlers = [];
}
```

Microtasks run **after the current synchronous code finishes**, but **before any setTimeout or UI event**. This gives you consistent, predictable ordering:

```javascript
console.log('1 — sync');
MyPromise.resolve('hello').then(v => console.log('3 — microtask:', v));
console.log('2 — sync');
// Logs: 1 → 2 → 3
```

If handlers ran synchronously, `.then()` callbacks would interleave with your synchronous code in unpredictable ways.

### `Promise.all()` — Parallelism

`Promise.all()` runs multiple promises in parallel and resolves when all have settled:

```javascript
static all(promises) {
  return new MyPromise((resolve, reject) => {
    const results  = new Array(promises.length);
    let remaining  = promises.length;

    promises.forEach((p, i) => {
      MyPromise.resolve(p).then(
        value => {
          results[i] = value;          // store at correct index
          if (--remaining === 0) resolve(results); // all done!
        },
        reason => reject(reason)       // fail fast — one rejection rejects all
      );
    });
  });
}
```

Key design choices: results are stored at their original index (not arrival order), and a single rejection immediately rejects the whole `.all()` — even if other promises are still running. They still run to completion, but their results are ignored.

### `Promise.race()` — First Wins

```javascript
static race(promises) {
  return new MyPromise((resolve, reject) => {
    for (const p of promises) {
      MyPromise.resolve(p).then(resolve, reject);
    }
  });
}
```

`race()` is three lines because the state machine does the work: after the first promise calls `resolve` or `reject`, all subsequent calls are ignored by the guard `if (this.state !== PENDING) return`.

---

## Try It

1. **Add a timeout utility**: Write `withTimeout(promise, ms)` that races `promise` against a timer that rejects after `ms` milliseconds. Use `MyPromise.race()` and a `delay()` function. Test it with a slow `fetchUser` call.

2. **Add `MyPromise.any()`**: Like `.race()`, but ignores rejections — resolves when the first promise *fulfills*, rejects only if all promises reject. The rejection reason should be an `AggregateError` containing all individual reasons.

3. **Track handler call order**: Add a `debug` mode to `MyPromise` that logs every state transition and handler invocation with timestamps. Use this to visualise the microtask order when three chained `.then()` calls each return a new Promise.

4. **Make `finally` truly final**: Currently `.finally(fn)` passes through the value. Modify it so if `fn` itself returns a rejected promise, that rejection takes precedence over the original value or reason.

---

## Exercises

1. **Implement `retry(fn, attempts)`**: Takes a function `fn` that returns a Promise and a number of `attempts`. Returns a Promise that tries `fn()` up to `attempts` times, waiting 200ms between tries. Resolves with the first successful result. Rejects with the last error if all attempts fail.

2. **Implement `promisify(fn)`**: Takes a Node.js-style callback function (where the last argument is `(err, result) => void`) and returns a new function that returns a Promise. `promisify(fs.readFile)('/path', 'utf8')` should return a Promise that resolves with the file contents or rejects with the error.

3. **Implement `waterfall(tasks)`**: Takes an array of functions, each receiving the result of the previous one. Runs them in sequence (each waits for the prior to resolve). Returns a Promise that resolves with the final result. Equivalent to a chain of `.then()` calls built dynamically.

4. **Implement `pool(tasks, concurrency)`**: Runs async tasks with a concurrency limit. Given 20 tasks and `concurrency=3`, at most 3 run simultaneously. When one finishes, the next starts. Returns a Promise that resolves with all results in order.

5. **Add unhandled rejection detection**: Track all promises that are rejected but have no `.catch()` handler attached within the same microtask turn. After a tick, warn in the console: `"UnhandledPromiseRejection: <reason>"`. This mirrors what Node.js and browsers do natively. Use `queueMicrotask` to schedule the check.

---

## Solutions

### Exercise 1 — `retry(fn, attempts)`

```javascript
function retry(fn, attempts, delay = 200) {
  return new MyPromise((resolve, reject) => {
    function attempt(remaining) {
      fn().then(resolve).catch(err => {
        if (remaining <= 1) {
          reject(err);
        } else {
          setTimeout(() => attempt(remaining - 1), delay);
        }
      });
    }
    attempt(attempts);
  });
}

// Usage:
retry(() => fetchUser(1), 3).then(user => console.log(user));
```

### Exercise 2 — `promisify(fn)`

```javascript
function promisify(fn) {
  return function() {
    const args = Array.from(arguments);
    return new MyPromise((resolve, reject) => {
      args.push(function(err, result) {
        if (err) reject(err);
        else     resolve(result);
      });
      fn.apply(this, args);
    });
  };
}

// Usage (Node.js-style):
// const readFile = promisify(fs.readFile);
// readFile('/path/to/file', 'utf8').then(contents => console.log(contents));
```

### Exercise 3 — `waterfall(tasks)`

```javascript
function waterfall(tasks) {
  return tasks.reduce(
    (promise, task) => promise.then(result => task(result)),
    MyPromise.resolve(undefined)
  );
}

// Usage:
waterfall([
  ()     => fetchUser(1),
  user   => fetchPosts(user.id),
  posts  => MyPromise.resolve(posts.map(p => p.title)),
]).then(titles => console.log(titles));
```

The `reduce` builds a chain: `Promise.resolve().then(task1).then(task2).then(task3)`. Each task receives the result of the previous one.

### Exercise 4 — `pool(tasks, concurrency)`

```javascript
function pool(tasks, concurrency) {
  return new MyPromise((resolve, reject) => {
    const results = new Array(tasks.length);
    let index     = 0;
    let completed = 0;
    let running   = 0;

    function next() {
      while (running < concurrency && index < tasks.length) {
        const i = index++;
        running++;
        MyPromise.resolve(tasks[i]()).then(
          value => {
            results[i] = value;
            running--;
            completed++;
            if (completed === tasks.length) resolve(results);
            else next();
          },
          reason => reject(reason)
        );
      }
    }

    next();
  });
}

// Usage:
const tasks = [1,2,3,4,5,6,7,8].map(id => () => fetchUser(id % 3 + 1));
pool(tasks, 3).then(users => console.log('All fetched:', users.length));
```

### Exercise 5 — Unhandled rejection detection

```javascript
// Patch the constructor to track unhandled rejections:
const _originalThen = MyPromise.prototype.then;
MyPromise.prototype.then = function(onFulfilled, onRejected) {
  if (onRejected) this._handled = true;
  return _originalThen.call(this, onFulfilled, onRejected);
};

// In _runHandlers, after settling as rejected:
_runHandlers() {
  if (this.state === PENDING) return;

  if (this.state === REJECTED && this.handlers.length === 0 && !this._handled) {
    queueMicrotask(() => {
      if (!this._handled) {
        console.warn('UnhandledPromiseRejection:', this.value);
      }
    });
  }

  // ... rest of _runHandlers
}
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| Three states | `pending → fulfilled` or `pending → rejected` — transitions are one-way |
| Executor | Runs synchronously; calls `resolve` or `reject` to settle the promise |
| `this.handlers` | Array of callbacks registered by `.then()` before the promise settles |
| `_runHandlers()` | Called whenever state changes; dispatches queued handlers as microtasks |
| `queueMicrotask` | Schedules work after current sync code but before the next setTimeout |
| `.then()` returns new Promise | The chaining primitive — `resolve`/`reject` of the new promise thread values through |
| Thenable resolution | `if value.then === 'function': value.then(resolve, reject)` — flattens nested promises |
| `Promise.all` | Parallel — resolves when all resolve; rejects fast on first rejection |
| `Promise.race` | First to settle (either way) wins; state guard ignores subsequent calls |
| `Promise.allSettled` | Never rejects — waits for all, reports `{status, value/reason}` per promise |
| `async/await` | Syntax sugar over `.then()` — `await p` unwraps a promise inside an async function |

---

## Building with Claude

Bad prompt:
> "Explain Promises."

Good prompt:
> "I've built a Promise implementation. When I call `resolve()` with another Promise as the value, I check `if (value && typeof value.then === 'function')` and call `value.then(resolve, reject)`. But I'm not sure why I need this check — what would actually go wrong if I removed it and just stored the Promise object as the resolved value? I want to understand what specific behaviour would break in a chain like `fetchUser(1).then(user => fetchPosts(user.id)).then(posts => render(posts))`."

The prompt proves you read the code, identifies the exact mechanism you're uncertain about, and asks for a concrete failure case — not abstract theory.

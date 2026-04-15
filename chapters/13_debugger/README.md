# Chapter 13 — Build Your Own Debugger

A debugger feels like an expert tool, but it becomes much less scary once you understand what it is showing you. This chapter does two things: it teaches the real browser debugging tools, and then it builds a small debugger so the moving parts become visible instead of mysterious.

---

## Using JavaScript Debugging Tools

Before building a debugger, you need to be fluent with the ones you already have. A developer who can't use the debugger effectively spends ten times longer finding bugs.

For beginners, a debugger is best understood as a pause button plus an inspection window. It stops the program at a specific moment and lets you look around. That is much easier than guessing from `console.log` output after the fact.

---

### The `debugger` Statement

The fastest way to pause JavaScript execution is one keyword:

```javascript
function divide(a, b) {
  debugger;           // execution pauses here when DevTools is open
  return a / b;
}

divide(10, 0);
```

Open DevTools (F12), then run the code. Execution stops at `debugger` and the Sources panel opens. You're now in the debugger — all controls and panels are live.

When DevTools is closed, `debugger` does nothing and is silently ignored. It is safe to commit (but bad practice — strip them before shipping).

The `debugger` statement works anywhere a statement can go:

```javascript
// Inside a loop — pauses on every iteration
for (let i = 0; i < users.length; i++) {
  debugger;
  processUser(users[i]);
}

// Inside a condition — pauses only when condition is true
if (response.status !== 200) {
  debugger;   // pause only on unexpected responses
}

// Inside an async function
async function fetchUser(id) {
  const data = await api.get(`/users/${id}`);
  debugger;   // pause after the await resolves
  return data;
}
```

---

### Chrome DevTools: Setting Breakpoints

`debugger` is a code breakpoint. DevTools gives you five more kinds — all without touching the source:

**Line breakpoints**: Click a line number in the Sources panel. A blue marker appears. Execution pauses every time that line is about to run.

**Conditional breakpoints**: Right-click a line number → "Add conditional breakpoint." Enter any JavaScript expression. Execution pauses only when the expression evaluates to `true`.

```
// Conditional breakpoint expression examples:
i === 50           // pause on the 51st loop iteration
user.role !== 'admin'   // pause for non-admins
arr.length > 100   // pause on large arrays
response.data === null   // pause on empty responses
```

**Logpoints**: Right-click → "Add logpoint." Enter an expression. Chrome logs the value without pausing. Like `console.log` but without touching the source. Useful in production where you can't modify code.

**DOM breakpoints**: In the Elements panel, right-click a DOM node → "Break on…" → subtree modifications / attribute changes / node removal. Chrome pauses when JavaScript modifies that node.

**Event listener breakpoints**: Sources → Event Listener Breakpoints. Check "click" under Mouse, and Chrome pauses whenever any click handler fires. Useful for finding which code runs when you interact with the UI.

---

### Stepping Through Code

When execution is paused, four navigation controls become active:

```
DevTools Controls (keyboard shortcuts in parentheses):

  ⟳ Resume (F8)    — run until the next breakpoint (or end)
  ↷ Step Over (F10) — run next line; if it calls a function, finish the whole call
  ↓ Step Into (F11) — run next line; if it calls a function, enter that function
  ↑ Step Out  (⇧F11) — run to the end of the current function, then pause

  ↯ Step (F9)      — one machine instruction (rarely needed)
```

When to use each:

- **Step Over** most of the time. Run the next statement. If it's `const x = helper()`, you trust `helper` and just want to see the result.
- **Step Into** when the bug might be inside the function being called. Enter `helper()` and step through it line by line.
- **Step Out** when you accidentally stepped into something you didn't mean to (like a library function). Jump back to the caller.
- **Resume** when you've set a second breakpoint further down and want to skip to it.

A useful beginner habit is to predict what the next line will do before you step. Then compare your prediction with what actually happens. That turns debugging into active learning, not just bug hunting.

---

### Inspecting Variables and Scope

While paused, the right sidebar shows three scope panels:

```
Scope Panels:
┌──────────────────────────────────────────┐
│ ▼ Local                                  │
│     n:  4                                │
│     left:  2                             │
│     right:  undefined                    │
├──────────────────────────────────────────┤
│ ▼ Closure (fibonacci)                    │
│     — (empty for this function)          │
├──────────────────────────────────────────┤
│ ▼ Global                                 │
│     window, document, ...               │
└──────────────────────────────────────────┘
```

**Local** — variables declared in the current function.
**Closure** — variables from outer functions that the current function closes over.
**Global** — everything on `window`.

If closures still feel confusing, read that middle panel as "variables this function can still see from outside itself." That mental model is usually enough to make the scope view useful.

Hover over any variable in the source code panel — its current value appears in a tooltip. Click a value to edit it and change it mid-execution. This is useful for testing: "what happens if `n` is -1 here?"

**Watch expressions**: In the Watch panel (+), add any JavaScript expression. It re-evaluates at every pause. Useful for derived values: `left + right`, `arr.slice(0, 5)`, `JSON.stringify(state)`.

**Call Stack panel**: The stack of active function calls. Click any frame to jump to that scope and see its local variables. Every entry corresponds to a function that called the current one and is waiting for it to return.

---

### Console Methods for Debugging

Beyond `console.log`, there are several methods that make debugging faster:

**`console.table`** — renders arrays of objects as a table:

```javascript
const users = [
  { name: 'Ada', age: 36, role: 'engineer' },
  { name: 'Grace', age: 85, role: 'admiral' },
  { name: 'Alan', age: 41, role: 'codebreaker' },
];

console.table(users);
// ┌─────────┬──────────┬─────┬──────────────┐
// │ (index) │  name    │ age │    role       │
// ├─────────┼──────────┼─────┼──────────────┤
// │    0    │  'Ada'   │ 36  │ 'engineer'    │
// │    1    │  'Grace' │ 85  │ 'admiral'     │
// │    2    │  'Alan'  │ 41  │ 'codebreaker' │
// └─────────┴──────────┴─────┴──────────────┘
```

**`console.trace`** — prints the current call stack. No breakpoint needed — just call it from anywhere:

```javascript
function inner() {
  console.trace('where am I being called from?');
}

function middle() { inner(); }
function outer()  { middle(); }

outer();
// Trace: where am I being called from?
//     at inner (<anonymous>:2:11)
//     at middle (<anonymous>:5:18)
//     at outer (<anonymous>:6:21)
//     at <anonymous>:8:1
```

**`console.group` and `console.groupEnd`** — indent related log lines together:

```javascript
function processOrder(order) {
  console.group(`Order #${order.id}`);
  console.log('items:', order.items.length);
  console.log('total:', order.total);
  console.log('status:', order.status);
  console.groupEnd();
}

// Output:
// ▼ Order #42
//     items: 3
//     total: 97.50
//     status: pending
```

**`console.time` and `console.timeEnd`** — measure elapsed time:

```javascript
console.time('sort');
arr.sort((a, b) => a - b);
console.timeEnd('sort');
// sort: 2.3ms
```

**`console.assert`** — logs an error if the condition is false:

```javascript
console.assert(arr.length > 0, 'Expected non-empty array', arr);
// If arr is empty: Assertion failed: Expected non-empty array []
// If arr is non-empty: nothing logged
```

**`console.dir`** — shows an object's properties as a tree (more useful than `log` for DOM nodes):

```javascript
console.dir(document.body);   // expandable property tree in DevTools
```

---

### Node.js Debugging with `--inspect`

Node.js has the same V8 debugging protocol as Chrome DevTools. To debug a Node script:

```bash
# Start Node with the inspector
node --inspect your-script.js

# Or pause immediately on the first line:
node --inspect-brk your-script.js
```

Then open `chrome://inspect` in Chrome, click "Open dedicated DevTools for Node." The same Sources panel, same breakpoints, same step controls — but running Node.

For long-running servers:

```bash
# Attach to a running Node process
node --inspect=0.0.0.0:9229 server.js
```

The `--inspect-brk` flag is especially useful when you have startup code that runs before you can set breakpoints: it pauses on the very first line, giving you time to set breakpoints before anything runs.

---

### VS Code Debugger

VS Code has a built-in debugger that shows scope, call stack, and variable values directly in the editor — without switching to a browser window.

Create `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Script",
      "program": "${workspaceFolder}/your-script.js",
      "console": "integratedTerminal"
    },
    {
      "type": "chrome",
      "request": "launch",
      "name": "Debug in Chrome",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}"
    }
  ]
}
```

Press F5 to start debugging. Set breakpoints by clicking in the gutter (left of line numbers). The Debug sidebar shows Variables, Watch, Call Stack, and Breakpoints panels — all identical to DevTools, but inside the editor.

VS Code breakpoints have conditions and hit counts (right-click a breakpoint to configure):
- **Condition**: `i === 50` — same as a DevTools conditional breakpoint
- **Hit count**: `>= 5` — pause only after the breakpoint has been hit 5 times
- **Log message**: `Value is {i}` — logpoint without pausing

---

## How Debuggers Work

Now that you can use the real tools, here's the question: *how do they work?*

Chrome DevTools doesn't pause JavaScript by magic. V8 (Chrome's JavaScript engine) implements the **Chrome DevTools Protocol (CDP)** — a WebSocket API that lets external programs tell V8 to pause, set breakpoints, and inspect the heap. DevTools is just a web page that talks to V8 over this protocol.

Native debuggers (GDB, LLDB) work at the CPU level — they patch the executable to insert trap instructions, and the OS pauses the process when a trap fires. JavaScript debuggers work higher up: at the engine level, not the OS level.

In plain JavaScript running in the browser, you don't have access to V8 internals. But you can achieve the same effect through a different route: **code instrumentation** — transform the source code to insert pause points before executing it.

```
Native Debugger:           Your Debugger:
CPU instruction trap  ←→  __step__() call inserted at each line
OS pauses the process ←→  generator yields (cooperative pause)
Debugger reads memory ←→  Debugger reads captured vars object
```

This is also how many production tools work. Istanbul (code coverage) instruments your code to count line executions. Babel transforms your code before the browser runs it. The `debugger` statement itself is just a well-known signal to the engine — it's syntactic sugar for "tell the debugger we're here."

---

## Execution Context, Call Stack, Scope Chain

Three structures that every debugger reads:

```
JavaScript Runtime: Three Key Structures
─────────────────────────────────────────────────────────────────
                      CALL STACK
              ┌──────────────────────────────┐
              │ fibonacci(1)                 │ ← top (running now)
              │   local: { n: 1 }            │
              ├──────────────────────────────┤
              │ fibonacci(2)                 │
              │   local: { n: 2, left: ? }   │   each frame is one
              ├──────────────────────────────┤   execution context
              │ fibonacci(3)                 │
              │   local: { n: 3, left: ? }   │
              ├──────────────────────────────┤
              │ fibonacci(4)                 │
              │   local: { n: 4, left: ? }   │
              ├──────────────────────────────┤
              │ <global>                     │ ← bottom (always here)
              └──────────────────────────────┘
                             ↕
            each frame's scope chain points to the frame below it

                   SCOPE CHAIN for fibonacci(1)
        ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
        │ fibonacci(1) │→→→│ fibonacci(2) │→→→│   <global>   │
        │   { n: 1 }   │   │ { n: 2, ... }│   │ { Math, ... }│
        └──────────────┘   └──────────────┘   └──────────────┘
            Looking up 'Math': not in fibonacci(1) → not in fibonacci(2)
            → found in global ✓
```

**Execution context**: A record created for each function call. Contains: local variables, `this`, and a reference to the outer scope (for the scope chain).

**Call stack**: The stack of active execution contexts. Each function call pushes a new frame; each return pops one. "Stack overflow" means the stack exceeded the engine's limit (≈10,000 frames in V8).

**Scope chain**: When you reference a variable, JavaScript looks in the current execution context first, then walks up to the parent, until it finds the name or reaches global scope. This is what makes closures work.

---

## Beginner Summary

Before you dive into the code, keep this in mind:

- a debugger pauses a program at useful moments
- while paused, you can inspect variables, scope, and call stack
- stepping means moving through execution in controlled small steps
- building a debugger means recording and replaying execution state

## Building It Step by Step

### v1 — A Trace Logger

In plain English: we start with the easiest debugging tool possible, which is just recording what happened.

The simplest debugger is a transparent wrapper — it logs state without changing behavior:

```javascript
// v1: trace logger — observe without stopping
function trace(label, value) {
  console.log(`[TRACE] ${label} =`, JSON.stringify(value));
  return value;   // returns value unchanged: trace('n', n) can wrap any expression
}

// Usage: wrap any expression to log it
function fibonacci(n) {
  trace('fibonacci called with', n);
  if (trace('n <= 1?', n <= 1)) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}
```

This works, but the output is a scroll of text. You can't pause, navigate, or inspect arbitrary expressions without re-running the program.

### v2 — Generators: Cooperative Execution

JavaScript generators (`function*`) can pause at `yield` and resume exactly where they left off. A generator hands execution back to the caller at each `yield`, and the caller decides when to continue.

```javascript
// v2: generators make execution controllable
function* stepThrough(n) {
  let step = 0;
  while (n > 0) {
    yield { step: ++step, line: 3, vars: { n } };  // pause: hand state to debugger
    n--;
  }
  yield { step: ++step, line: 5, vars: { n } };    // final state
}

const gen = stepThrough(3);
gen.next().value;   // { step: 1, line: 3, vars: { n: 3 } }
gen.next().value;   // { step: 2, line: 3, vars: { n: 2 } }
gen.next().value;   // { step: 3, line: 3, vars: { n: 1 } }
gen.next().value;   // { step: 4, line: 5, vars: { n: 0 } }
gen.next().done;    // true — execution finished
```

The caller controls exactly when the function proceeds. Between `.next()` calls, execution is completely paused. This is the engine your debugger uses.

### v3 — Breakpoints and Execution Control

Add a `Debugger` class that manages the trace and supports step/breakpoint commands:

```javascript
// v3: step controls and breakpoints
class Debugger {
  constructor() {
    this.trace  = [];           // recorded execution frames
    this.cursor = 0;            // current position in the trace
    this.breakpoints = new Set();  // set of line numbers
    this.watches = [];          // watch expressions
  }

  // Navigate
  stepInto()  { if (this.cursor < this.trace.length - 1) this.cursor++; }
  stepOver()  { /* advance until stack depth <= current */ }
  stepOut()   { /* advance until stack depth <  current */ }
  continue()  { /* advance until breakpoint */ }
  stepBack()  { if (this.cursor > 0) this.cursor--; }  // time travel!

  currentFrame() { return this.trace[this.cursor]; }

  // Breakpoints
  setBreakpoint(line)   { this.breakpoints.add(line); }
  clearBreakpoint(line) { this.breakpoints.delete(line); }

  // Watch expressions
  addWatch(expr) { this.watches.push(expr); }
  evalWatch(expr, vars) {
    try {
      const fn = new Function(...Object.keys(vars), `return (${expr})`);
      return fn(...Object.values(vars));
    } catch (e) { return `⚠ ${e.message}`; }
  }
}
```

### v4 — Source Instrumentation

In plain English: we rewrite the code slightly so it reports its own progress while running.

The most powerful piece: automatically transform source code to capture its own state as it runs.

```javascript
// v4: instrument source to record every line's state
function instrument(source) {
  const lines = source.split('\n');
  const names = extractVarNames(source);   // find all let/const declarations

  return lines.map((line, i) => {
    const t = line.trim();
    if (!t || t.startsWith('//') || t === '{' || t === '}') return line;

    // After each non-trivial line, inject a __step__ call
    const capture = names
      .map(n => `"${n}":typeof ${n}!=="undefined"?${n}:undefined`)
      .join(',');
    return line + `\n__step__(${i + 1},{${capture}});`;
  }).join('\n');
}

function extractVarNames(source) {
  const seen = new Set();
  for (const m of source.matchAll(/\b(?:let|const|var)\s+(\w+)/g)) seen.add(m[1]);
  return [...seen];
}

// The instrumented code calls __step__ at each line.
// __step__ records the frame and pauses until the UI calls next():
function __step__(line, vars) {
  debugRecord.push({ line, vars: { ...vars }, stack: captureStack() });
}
```

This is identical to how Istanbul captures code coverage — it transforms your source before execution and counts how many times each line runs.

---

## The Complete Program

`debugger.html` — open it in your browser. Select a program (fibonacci, insertion sort, or closure counter), set breakpoints by clicking line numbers, and step through execution. The call stack grows and shrinks in real time as functions are called and return. Watch expressions update automatically at each step.

---

## Walkthrough

### Generators — Functions That Pause and Resume

A regular function runs to completion. A generator can pause at `yield` and resume later from the same position, with all its local variables intact.

```javascript
function regular() {
  const x = 1;
  const y = 2;
  return x + y;      // runs all at once — you can't see inside while it's running
}

function* generator() {
  const x = 1;
  yield { line: 2, vars: { x } };       // pause: give caller the state
  const y = 2;
  yield { line: 3, vars: { x, y } };    // pause again
  return x + y;                          // done
}

const g = generator();
g.next();   // { value: { line: 2, vars: { x: 1 } }, done: false }
g.next();   // { value: { line: 3, vars: { x: 1, y: 2 } }, done: false }
g.next();   // { value: 3, done: true }  ← return value and done flag
```

The key: **the generator object holds the paused execution state**. It's a first-class value you can store, pass around, and resume later. Calling `.next()` resumes execution until the next `yield` (or `return`).

Generators are also how JavaScript `async/await` works under the hood. An `async` function is a generator whose `yield` points are the `await` expressions, and the runtime drives it automatically.

### The Call Stack — Frames and Depth

Every function call allocates a **stack frame** — a record of the function's local variables, the return address, and a pointer to the outer scope. Frames stack on top of each other as functions call functions.

```javascript
function a() {
  b();          // push b's frame
}               // pop b's frame, return here

function b() {
  c();          // push c's frame
}               // pop c's frame, return here

function c() {
  // call stack here: [c, b, a, <global>]
  console.trace();
}
```

The debugger's call stack panel reads these frames from top to bottom. The top frame is where execution is currently paused. Clicking a lower frame shows its local variables — that's the scope at that level, frozen at the point where it called the function above.

**Stack overflow** occurs when the stack grows faster than it shrinks — infinite recursion is the classic cause. The engine imposes a depth limit (about 10,000–15,000 frames in V8) to prevent stack memory from growing without bound.

### Scope Chain and Closures

Each execution context has a **scope** — a map of variable names to values. When you reference a name, JavaScript looks in the current scope, then the parent scope, then the parent's parent, until it finds the name or exhausts the chain.

```
Scope chain lookup: 'count' inside increment()
─────────────────────────────────────────────────────────────────
function makeCounter(start) {
  let count = start;               ← count lives here
  function increment(step) {
    count += step;                 ← 'count' not in increment's scope
    return count;                  ← walk up → found in makeCounter's scope ✓
  }
  return increment;
}
─────────────────────────────────────────────────────────────────
increment scope:    { step }
    ↓ walk up
makeCounter scope:  { start, count }   ← 'count' found here
    ↓ walk up
global scope:       { Math, Array, ... }
```

A **closure** is a function that captures a reference to its outer scope, not a copy. When `increment` reads `count`, it sees the current value of `count` in `makeCounter`'s scope. When it writes `count += step`, it modifies that same variable — and any other function that closes over the same scope sees the change.

This is why the Closure Counter demo in `debugger.html` shows `count` persisting across three calls to `increment`: all three calls close over the *same* `makeCounter` scope, and `count` in that scope accumulates changes.

### Step Into, Step Over, Step Out

These three navigation modes become clear once you understand call depth:

```
fibonacci(3) is about to call fibonacci(2)

  ┌──────────────────────────────────────────────────────┐
  │                          fibonacci(3) ← paused here  │
  │                          const left = fibonacci(2)   │
  │                          ↑ this call is about to run │
  └──────────────────────────────────────────────────────┘

  Step Into:  enter fibonacci(2), now paused at its first line
              call stack: [fibonacci(2), fibonacci(3), global]

  Step Over:  run fibonacci(2) completely, stay in fibonacci(3)
              after fibonacci(2) returns, paused at next line of fibonacci(3)
              call stack: [fibonacci(3), global]   (depth unchanged)

  Step Out:   run fibonacci(3) to completion, pause in its caller
              call stack: [global]   (one level shallower)
```

Implemented in terms of the trace:

```javascript
stepInto() {
  // Advance exactly one frame — enters sub-calls
  this.cursor++;
}

stepOver() {
  const depth = this.trace[this.cursor].stack.length;
  // Advance until we're back at the same or shallower depth
  while (this.cursor < this.trace.length - 1) {
    this.cursor++;
    if (this.trace[this.cursor].stack.length <= depth) break;
  }
}

stepOut() {
  const depth = this.trace[this.cursor].stack.length;
  // Advance until one level shallower
  while (this.cursor < this.trace.length - 1) {
    this.cursor++;
    if (this.trace[this.cursor].stack.length < depth) break;
  }
}
```

The difference is just how many frames you skip and what stack depth you stop at.

### Watch Expressions and `new Function`

A watch expression evaluates arbitrary JavaScript against the current scope on every pause. The trick is `new Function`:

```javascript
function evalWatch(expr, vars) {
  // Turn { n: 4, left: 2 } into a function that knows about n and left:
  // new Function('n', 'left', 'return (n * left)') → (n, left) => n * left
  try {
    const paramNames  = Object.keys(vars);
    const paramValues = Object.values(vars);
    const fn = new Function(...paramNames, `return (${expr})`);
    return fn(...paramValues);
  } catch (e) {
    return `⚠ ${e.message}`;
  }
}

evalWatch('n * 2', { n: 4, left: 2 });   // 8
evalWatch('n + left', { n: 4, left: 2 }); // 6
evalWatch('n > 1 ? "recurse" : "base"', { n: 4 }); // "recurse"
```

`new Function(params, body)` creates a function from strings at runtime. The watch expression `"n * 2"` becomes the body; the variable names become the parameter list. When called with the current values, it evaluates in that scope.

This is exactly how Chrome DevTools' Watch panel works: each watch expression is re-evaluated in the current scope every time the debugger pauses. It's also how template engines in Chapter 8 compile templates — `new Function` turns a string into executable code.

Security note: `new Function` with user-supplied strings is equivalent to `eval` — never use it with untrusted input in production code. In a local debugger tool, it is the right tool.

---

## Guided Try It — Add Conditional Breakpoints

**The goal**: allow breakpoints that only pause when a condition is true. Clicking a line number adds a regular breakpoint; double-clicking opens a condition prompt.

**Step 1 — Change the breakpoints data structure**

Right now, `breakpoints` is a `Set` of line numbers. Change it to a `Map` where the value is the condition string (or `null` for unconditional):

```javascript
// Before:
this.breakpoints = new Set();

// After:
this.breakpoints = new Map();   // line → condition string | null

setBreakpoint(line, condition = null) {
  this.breakpoints.set(line, condition);
}

clearBreakpoint(line) {
  this.breakpoints.delete(line);
}

isBreakpoint(line, vars) {
  if (!this.breakpoints.has(line)) return false;
  const cond = this.breakpoints.get(line);
  if (cond === null) return true;               // unconditional
  return !!this.evalWatch(cond, vars);          // conditional — evaluate
}
```

**Step 2 — Update `continue()` to use `isBreakpoint`**

```javascript
continue() {
  while (this.cursor < this.trace.length - 1) {
    this.cursor++;
    const f = this.trace[this.cursor];
    if (this.isBreakpoint(f.line, f.vars)) break;
  }
}
```

**Step 3 — Add the double-click handler in the UI**

```javascript
lineEl.addEventListener('dblclick', () => {
  const cond = prompt('Break when (leave blank for always):');
  if (cond !== null) {
    dbg.setBreakpoint(lineNum, cond.trim() || null);
    renderCode(dbg.currentFrame());
  }
});
```

**Step 4 — Try it**

Load the fibonacci program. Double-click line 2 (`if (n <= 1) return n`). Enter `n === 2` as the condition. Click Continue. Execution pauses only when `fibonacci(2)` reaches that line — skipping the deeper `fibonacci(1)` and `fibonacci(0)` calls. The call stack confirms: `fibonacci(2)` is on top with `n = 2`.

**Think about it**: Chrome DevTools conditional breakpoints work exactly this way — V8 evaluates the condition expression in the current scope on every hit. What happens if the condition expression itself throws an error? Your `evalWatch` already handles this with a `try/catch`. How should it affect whether the breakpoint fires?

---

## Exercises

1. **Step back in time**: The trace array holds all past steps. Add a `stepBack()` method to the debugger that decrements the cursor. Add a "Step Back" button to the UI. Verify: step forward three times and back three times should return to the starting frame. This is the core of time-travel debugging.

2. **Highlight changed variables**: Compare the current frame's `vars` with the previous frame's `vars`. Any variable whose value changed should be highlighted in yellow in the Variables panel. The format `prev !== curr` suffices for primitives. Chrome DevTools shows changed values as bright green.

3. **Add a step counter and progress bar**: Below the code panel, add a progress bar showing position `cursor / trace.length`. Add a label showing "Step N of M." The progress bar gives a visual sense of how far through execution you are — useful for programs with hundreds of steps.

4. **Build a call depth chart**: Record the call stack depth (`frame.stack.length`) at each step. After the program completes, render a small ASCII bar chart or colored timeline showing how the depth changed. For fibonacci(4), it should show a wave shape — deep during recursive expansion, shallow during returns.

5. **Add a simple profiler**: Record a `timestamp` (via `performance.now()`) in each frame when it's generated. After the trace is built, compute how long each function spent executing (sum of time between its call frame and return frame). Display the top 3 slowest functions. For recursive fibonacci, `fibonacci(1)` and `fibonacci(0)` together will dominate.

---

## Solutions

### Exercise 1 — Step back

```javascript
// In the Debugger class:
stepBack() {
  if (this.cursor > 0) {
    this.cursor--;
    return this.currentFrame();
  }
  return null;
}

// In the UI, add a button:
// <button id="btn-back">Step Back</button>
document.getElementById('btn-back').addEventListener('click', () => {
  const f = dbg.stepBack();
  if (f) render(f);
  updateButtons();
});

// Verify:
// 1. Load fibonacci(4)
// 2. Step forward 3 times: cursor should be 3
// 3. Step back 3 times: cursor should be 0 again
// 4. Frame at cursor 0 should be the first frame (line 1 of fibonacci, n=4)
```

### Exercise 2 — Highlight changed variables

```javascript
function changedVars(prev, curr) {
  // Returns a Set of variable names whose value changed
  const changed = new Set();
  if (!prev) return changed;
  const allKeys = new Set([...Object.keys(prev.vars || {}), ...Object.keys(curr.vars || {})]);
  for (const k of allKeys) {
    // JSON.stringify handles arrays and objects; primitives compare directly
    if (JSON.stringify(prev.vars[k]) !== JSON.stringify(curr.vars[k])) {
      changed.add(k);
    }
  }
  return changed;
}

function renderVars(curr, prev) {
  const changed = changedVars(prev, curr);
  const vars = curr.vars || {};
  return Object.entries(vars).map(([k, v]) => {
    const highlight = changed.has(k) ? ' changed' : '';
    return `<div class="var-row${highlight}">
      <span class="var-name">${k}</span>
      <span class="var-val">${JSON.stringify(v)}</span>
    </div>`;
  }).join('');
}

// CSS:
// .var-row.changed .var-val { background: #3a2d00; color: #e3b341; }
```

### Exercise 3 — Progress bar

```javascript
function renderProgress() {
  const pct = ((dbg.cursor + 1) / dbg.trace.length * 100).toFixed(1);
  document.getElementById('step-label').textContent =
    `Step ${dbg.cursor + 1} of ${dbg.trace.length}`;
  document.getElementById('progress-fill').style.width = pct + '%';
}

// HTML:
// <div id="step-label"></div>
// <div id="progress-bar">
//   <div id="progress-fill"></div>
// </div>

// CSS:
// #progress-bar { background:#1a1f2e; border-radius:3px; height:6px; }
// #progress-fill { background:#d2a8ff; height:100%; border-radius:3px; transition:width 0.1s; }
```

### Exercise 4 — Call depth chart

```javascript
function renderDepthChart() {
  if (dbg.trace.length === 0) return '';
  const depths = dbg.trace.map(f => f.stack.length);
  const maxDepth = Math.max(...depths);
  const width = Math.min(dbg.trace.length, 80);
  const step = dbg.trace.length / width;

  // Build ASCII bar per column
  const bars = Array.from({ length: width }, (_, col) => {
    const frameIdx = Math.floor(col * step);
    const depth = depths[frameIdx];
    const height = Math.round((depth / maxDepth) * 6);
    return '▁▂▃▄▅▆▇█'[Math.min(height, 7)] || '▁';
  }).join('');

  return `<pre class="depth-chart">${bars}</pre>`;
  // For fibonacci(4): ▁▃▅▇▅▇▅▃▅▃▁▃▅▃▁▃▁▁ (roughly)
}
```

### Exercise 5 — Profiler

```javascript
// During trace generation, record timing:
const t0 = performance.now();

function buildTrace(gen) {
  const trace = [];
  for (const frame of gen) {
    trace.push({ ...frame, ts: performance.now() - t0 });
  }
  return trace;
}

// After trace is built, measure per-function time:
function profileTrace(trace) {
  const totals = {};   // fnName → total ms

  for (let i = 0; i < trace.length - 1; i++) {
    const curr = trace[i];
    const next = trace[i + 1];
    const fn   = curr.stack[0];             // innermost function name
    const dt   = (next.ts || 0) - (curr.ts || 0);
    totals[fn] = (totals[fn] || 0) + dt;
  }

  // Sort by time descending
  return Object.entries(totals)
    .sort((a, b) => b[1] - a[1])
    .slice(0, 5)
    .map(([fn, ms]) => `${fn}: ${ms.toFixed(3)}ms`);
}

// Note: for tiny in-memory operations, timings may be 0ms due to timer resolution.
// The structure is correct; the numbers are most useful for programs that do real work.
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `debugger` statement | Pauses execution when DevTools is open; silently ignored otherwise |
| Chrome DevTools breakpoints | Line, conditional, logpoint, DOM, event listener — all without touching source |
| Step Into / Over / Out | Into enters a call; Over skips it; Out exits the current function |
| `console.table` | Renders arrays of objects as a readable table |
| `console.trace` | Prints the current call stack without a breakpoint |
| `console.group` | Indents related log lines; use `groupEnd` to close |
| Node.js `--inspect` | Same V8 protocol as Chrome DevTools; connect at `chrome://inspect` |
| VS Code launch.json | Configures the built-in debugger for Node or Chrome targets |
| Execution context | Created for each function call; contains locals, `this`, outer scope ref |
| Call stack | Stack of active execution contexts; grows on call, shrinks on return |
| Scope chain | Variable lookup walks from current scope up to global; closures freeze the chain at creation |
| Generator (`function*`) | A function that can pause at `yield` and resume; holds execution state as an object |
| Code instrumentation | Transforming source before execution to insert monitoring calls |
| `new Function` | Creates a function from strings at runtime; the mechanism behind watch expressions |
| Step Into vs Over vs Out | All three are "advance cursor" with different stopping conditions on call-stack depth |
| Conditional breakpoints | A breakpoint with a condition expression evaluated in the current scope |
| Time-travel debugging | Trace-based execution: record all frames upfront, navigate freely |

---

## Building with Claude

Bad prompt:
> "How do I debug JavaScript?"

Good prompt:
> "I'm building a step-through debugger for a JavaScript learning tool. It works by pre-running a generator function to collect a trace — an array of `{ line, vars, stack }` frames — then letting the user navigate with step controls. Step Into and Continue work correctly. Step Over is buggy: when I'm inside `fibonacci(2)` and step over a recursive call to `fibonacci(1)`, it sometimes stops one frame *into* `fibonacci(1)` instead of waiting for it to return. My `stepOver()` advances until `stack.length <= currentDepth`, but the frame immediately after the recursive call has the same depth as the recursive entry. Here's my current `stepOver()`: [code]. What's wrong with my stopping condition?"

The good prompt names the architecture (pre-recorded trace, not live execution), describes the exact misbehavior with a concrete example, gives the expected behavior ("wait for it to return"), and shows the specific implementation. Claude can diagnose the off-by-one in the depth comparison immediately.

---

*The patterns in this chapter connect directly to earlier ones. The generator pausing mechanism is the same cooperative-multitasking model as Chapter 9's Promise (a Promise's `.then` chain is driven by a microtask queue that "resumes" the chain one step at a time). The `new Function` watch evaluator is the same code-generation trick as Chapter 8's template engine. And the scope chain you read in the debugger is the closure mechanism from Chapter 1's logger — the same chain of outer references, now made visible instead of invisible.*

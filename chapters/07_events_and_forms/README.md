# Chapter 7: Events and Forms

## Why Events Exist

JavaScript runs in a single thread. There's no way to "pause and wait" for the user to click a button — the program would freeze. Instead, JavaScript uses an **event model**: you register a function, the browser stores it, and when something happens (a click, a keypress, a form submit), the browser calls your function.

This is called the **event loop** model. While nothing is happening, JavaScript is idle. The moment a user acts, the browser queues an event, picks up your registered function, and runs it.

```
User clicks button
        ↓
Browser creates a click event
        ↓
Browser looks up registered listeners for that element
        ↓
Browser calls your addEventListener callback
        ↓
Your code runs
        ↓
JavaScript goes idle again, waiting for the next event
```

This is why JavaScript can handle 20 buttons with 20 different behaviors without "blocking" — it doesn't watch them all simultaneously, it just responds when things happen.

## The Program: Calculator

A working calculator that responds to both mouse clicks and keyboard input. Press keys like `7 + 3 =` or click buttons. It handles decimals, operations, and shows both the current number and the expression you're building.

## The Complete Program

See `calculator.html` for the full source. The key JavaScript concepts are explained below.

## How It Works

### addEventListener and Event Types

An event listener attaches a function to an element for a specific event type:

```javascript
// Fires when user clicks the button
document.getElementById('addBtn').addEventListener('click', e => {
  console.log('clicked!');
});

// Fires when user presses any key while focused on the document
document.addEventListener('keydown', e => {
  console.log('key pressed:', e.key);
});
```

The function receives an **event object** (`e`) with details about what happened. For keyboard events, `e.key` holds the pressed key name (`'7'`, `'Enter'`, `'Escape'`). For click events, `e.target` holds the clicked element.

### Event Phases: Capturing and Bubbling

When you click a button inside a div, the event travels in two phases:

```
Phase 1 — CAPTURING (down the tree):
document → body → div.buttons → button

Phase 2 — BUBBLING (back up the tree):
button → div.buttons → body → document
```

`addEventListener` by default listens during the **bubbling** phase (phase 2). This is why event delegation works: a click on a button bubbles up to its parent container, where your single listener catches it.

You rarely need capturing. The third argument to `addEventListener` (the options object or a boolean) can switch to capturing phase, but for 99% of cases, bubbling is what you want.

### Event Delegation (Same Pattern as Chapter 6)

Instead of one listener per button (17 buttons = 17 listeners), one listener on the container:

```javascript
document.getElementById('buttons').addEventListener('click', e => {
  const btn = e.target.closest('button');  // which button was clicked?
  if (!btn) return;

  const { action, value } = btn.dataset;   // what should happen?

  if (!action) inputDigit(value);
  else if (action === 'op') chooseOp(value);
  else if (action === 'equals') doEquals();
  // ...
});
```

The `data-action` and `data-value` attributes on each button tell this one handler exactly what to do. No if/else based on button text (fragile), no IDs for each button (verbose) — just data attributes that describe intent.

### preventDefault

Some events have default browser behaviors. Keyboard events on number keys can trigger browser shortcuts. Form submission events refresh the page. Arrow keys scroll the viewport.

`e.preventDefault()` stops the default behavior so only your code runs:

```javascript
document.addEventListener('keydown', e => {
  if (e.key >= '0' && e.key <= '9') {
    e.preventDefault();  // stop any browser default for this key
    inputDigit(e.key);
  }
  // ...
});
```

Without `preventDefault()`, pressing `7` might trigger a browser function AND call `inputDigit('7')`. With it, only your code runs.

### The Calculator as a State Machine

The calculator keeps four variables:

```
State diagram:
                    ┌─────────────────────────────────────┐
                    │              CALCULATOR              │
                    │                                      │
  press digit  ───► │  current: "7"                        │
                    │  previous: null                      │
                    │  operator: null                      │
                    │  fresh: false                        │
                    └──────────────────┬──────────────────┘
                                       │ press "+"
                                       ▼
                    ┌─────────────────────────────────────┐
                    │  current: "7"  ← stays displayed    │
                    │  previous: "7" ← stored             │
                    │  operator: "+"                       │
                    │  fresh: true   ← next digit replaces│
                    └──────────────────┬──────────────────┘
                                       │ press "3"
                                       ▼
                    ┌─────────────────────────────────────┐
                    │  current: "3"  ← replaced (fresh!)  │
                    │  previous: "7"                       │
                    │  operator: "+"                       │
                    │  fresh: false                        │
                    └──────────────────┬──────────────────┘
                                       │ press "="
                                       ▼
                    ┌─────────────────────────────────────┐
                    │  current: "10" ← result             │
                    │  previous: null ← reset             │
                    │  operator: null ← reset             │
                    │  fresh: true   ← ready for next     │
                    └─────────────────────────────────────┘
```

The `fresh` flag is the key insight. When the user presses an operator, the next digit should *replace* the display, not extend it. Without `fresh`, pressing `7 + 3` would show `73` instead of `3`.

```javascript
function inputDigit(d) {
  if (fresh) {
    // Start fresh — replace display entirely
    current = d === '.' ? '0.' : d;
    fresh = false;
  } else if (current === '0' && d !== '.') {
    // Leading zero: "07" → replace with "7"
    current = d;
  } else {
    // Normal: append digit
    current += d;
  }
  updateDisplay();
}
```

### Keyboard Mapping

The keyboard uses character keys that don't match the display symbols:

```javascript
const KEY_MAP = {
  '/': '÷',  // keyboard slash → display division symbol
  '*': '×',  // keyboard asterisk → display multiplication
  '-': '−',  // keyboard minus → display minus (different Unicode!)
  '+': '+',  // these match
};

document.addEventListener('keydown', e => {
  if (KEY_MAP[e.key]) {
    chooseOp(KEY_MAP[e.key]);
  }
});
```

The object-as-lookup pattern again: instead of four if/else branches, one lookup that maps keyboard chars to operator symbols.

---

## Guided Exercises

### Exercise 1: Add a Square Button

**The Challenge:** Add an `x²` button that squares the currently displayed number. Pressing it with `5` on screen should show `25`. It should work like the `%` button — an immediate transformation, not a two-number operation.

**Where to start:** Look at how `doPercent()` works. It reads `current`, transforms it, and updates `current`. That's the exact pattern you need.

*(Look at `doPercent()` in the code before reading on.)*

---

**Step 1: Write the doSquare() function.**

```javascript
function doSquare() {
  const n = parseFloat(current);
  current = String(n * n);
  fresh = true;   // next digit starts a new number
  updateDisplay();
}
```

**Why set `fresh = true`?** After squaring 5 to get 25, the user probably wants to use 25 in a new operation. If `fresh` stays `false`, typing another digit would append to "25", giving "253". Setting `fresh = true` means the next digit starts fresh.

---

**Step 2: Add the button to the HTML.**

Replace the `+/−` button (or add a new slot):

```html
<button class="func" data-action="square">x²</button>
```

---

**Step 3: Wire it in the click handler.**

In the `click` event handler, add one line:

```javascript
else if (action === 'square') doSquare();
```

**Test it:** Click `5`, then `x²`. The display should show `25`. Click `x²` again — it should show `625`.

**Now think:** What happens if you press `x²` when the display shows `Error`? The current code would produce `NaN`. How would you guard against that? *(Add a check: `if (current === 'Error') return;` at the top of `doSquare()`.)*

---

### Exercise 2: Memory Functions (M+, M−, MR, MC)

**The Challenge:** Add four memory buttons. M+ adds the current number to a stored memory. M− subtracts it. MR recalls the memory. MC clears it to zero. Show the memory value (if non-zero) in a small display area above the main display.

**Where to start:** You need a new piece of state. What variable holds the memory? What type should it be?

*(Think before reading on. Answer: a single number, starting at 0.)*

---

**Step 1: Add the state variable.**

At the top of your script, with the other state variables:

```javascript
let memory = 0;
```

---

**Step 2: Write the memory functions.**

```javascript
function doMemAdd() {
  memory += parseFloat(current);
  updateMemoryDisplay();
}

function doMemSub() {
  memory -= parseFloat(current);
  updateMemoryDisplay();
}

function doMemRecall() {
  current = String(memory);
  fresh = true;  // ready for next operation
  updateDisplay();
}

function doMemClear() {
  memory = 0;
  updateMemoryDisplay();
}

function updateMemoryDisplay() {
  document.getElementById('memory').textContent = memory !== 0 ? `M: ${memory}` : '';
}
```

**Think about it:** `doMemAdd()` doesn't change `current` — it just stores from it. That's the right behavior. Why does `doMemRecall()` set `fresh = true`? What would happen without it?

*(If `fresh` stays `false`, the next digit typed after recall would append to the recalled number — so recalling 42 and pressing 3 would give "423" instead of "3".)*

---

**Step 3: Add HTML.**

Add a memory display above the `.display` div:

```html
<div class="memory" id="memory"></div>
```

Add four buttons. Memory buttons conventionally get their own row:

```html
<button class="mem" data-action="m-clear">MC</button>
<button class="mem" data-action="m-recall">MR</button>
<button class="mem" data-action="m-add">M+</button>
<button class="mem" data-action="m-sub">M−</button>
```

Style them differently from number and operator buttons — a blue-gray color works well.

---

**Step 4: Wire them in the click handler.**

```javascript
else if (action === 'm-add')    doMemAdd();
else if (action === 'm-sub')    doMemSub();
else if (action === 'm-recall') doMemRecall();
else if (action === 'm-clear')  doMemClear();
```

**The complete picture:** The memory system is just another state variable (`memory`) with four functions that read or write it. The pattern is identical to how `current`, `previous`, and `operator` work — state variables with functions that update them.

---

### Exercise 3: Calculation History

**The Challenge:** Keep a history of the last 5 calculations. Display them above the main display. Clicking a history entry loads that result as the current number. Clear history when the user presses AC.

**Where to start:** You need an array to store history entries. Each entry should record what the expression was and what the result was.

---

**Step 1: Add the history array.**

```javascript
const history = [];   // stores { expr: '7 + 3 =', result: '10' }
```

---

**Step 2: Record to history when = is pressed.**

In `doEquals()`, after computing the result:

```javascript
history.unshift({ expr: `${previous} ${operator} ${current} =`, result: current });
if (history.length > 5) history.pop();  // keep only the last 5
renderHistory();
```

`unshift` adds to the front (newest first). `pop` removes the oldest when the list exceeds 5.

---

**Step 3: Write renderHistory().**

```javascript
function renderHistory() {
  const histEl = document.getElementById('history');
  histEl.innerHTML = '';
  for (const entry of history) {
    const div = document.createElement('div');
    div.className = 'history-entry';
    div.textContent = `${entry.expr} ${entry.result}`;
    div.addEventListener('click', () => {
      current = entry.result;
      fresh = true;
      updateDisplay();
    });
    histEl.appendChild(div);
  }
}
```

---

**Step 4: Add HTML and clear in doClear().**

```html
<div id="history" style="text-align:right; color:#666; font-size:0.8rem; padding: 4px 8px; min-height: 80px;"></div>
```

In `doClear()`, add:

```javascript
history.length = 0;  // clears array in place
renderHistory();
```

**Check your understanding:** Why does `history.length = 0` clear the array instead of `history = []`? *(Setting `.length = 0` modifies the existing array. `history = []` creates a new array but doesn't affect the `history` variable's old binding if it was declared with `const`. Both work if declared with `let`, but `.length = 0` is a clear signal you're mutating, not replacing.)*

---

## What You Learned

| Concept | What It Does | Real-World Use |
|---------|-------------|----------------|
| `addEventListener` | Registers a function to run when an event fires | Foundation of all browser interactivity |
| `click` event | Fires on mouse button release over an element | Forms, menus, modals, toggles |
| `keydown` event | Fires when a key is pressed (repeats if held) | Keyboard shortcuts, game controls |
| Event delegation | One listener handles events from many children | React synthetic events, vanilla JS best practice |
| `e.preventDefault()` | Stops browser's default behavior | Form submission, link navigation, scroll |
| `e.key` | The key name that triggered the event | Keyboard navigation, shortcuts |
| State machine | Variables tracking which "mode" the program is in | Real calculators, text editors, games |
| Object as key map | Map keyboard chars to action strings | Shortcut systems, game keybindings |

### Real-World Connections

- **React's event system** wraps the browser's events in "synthetic events" but works identically — `onClick`, `onKeyDown`, `onChange` are just `addEventListener` calls managed by React.
- **State machines** power much more than calculators. XState is a popular JavaScript library built entirely around this concept. Traffic lights, authentication flows, media players — all state machines.
- **Form libraries** (React Hook Form, Formik) handle validation using the `input`, `change`, and `submit` events you've used here.
- **Game input systems** use the same keyboard tracking pattern you built: a `keys` object tracking which keys are held, checked on each frame.

## Building with Claude

- "Add a square root button that calculates the square root of the current number. Handle negative inputs gracefully."
- "Add localStorage so the calculator remembers the last memory value and history even after closing the tab."
- "Extend this calculator to support parentheses, like `(5 + 3) * 2`. Hint: this requires a real expression parser."
- "Add a history dropdown that shows the last 10 calculations. Clicking one loads that result."

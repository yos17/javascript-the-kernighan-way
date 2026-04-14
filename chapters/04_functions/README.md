# Chapter 4: Functions

## Why Functions Exist

Without functions, every program would be a long list of statements, top to bottom. You'd repeat the same code every place you needed it. Fixing a bug would mean finding every copy. Reading the code would mean understanding everything at once.

Functions solve this. A function is a named, reusable chunk of logic. You write it once, name it well, and call it whenever you need it. The name becomes documentation: `generatePassword()` tells you exactly what happens inside.

But functions aren't just about reuse. They're about **control**: what information goes in, what comes out, what can be seen from outside. Once you understand scope and closures — how functions control access to variables — the rest of JavaScript starts making sense.

**Coming up:** Functions are the unit of everything in Chapter 5 (every creature behavior is a function), Chapter 7 (every calculator operation is a function), Chapter 9 (the game loop *is* a function calling itself), and Chapter 11 (every drum sound is a synthesis function). The closures you learn here power the timer in Chapter 3, the audio scheduler in Chapter 11, and event handlers everywhere.

## The Program: Password Generator

A working password generator with character set selection, length slider, strength meter, and copy-to-clipboard. Open `password_generator.html` and generate a few passwords before reading the code.

Notice: every time you change a checkbox or move the slider, the password regenerates immediately. No "update" button needed — the UI stays in sync automatically. Understanding *how* requires understanding how functions compose.

## How It Works

### The Three Ways to Write a Function

JavaScript has three syntaxes for functions. They're not just stylistic — each has a different use case:

```javascript
// 1. Function declaration — the classic form
function generatePassword(length, options) {
  // body
}

// 2. Arrow function — concise, great for callbacks and short helpers
const upperChars = () => 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';

// 3. Arrow with body — for multi-line arrow functions
const getCharset = (options) => {
  const groups = [...];
  return groups.filter(...).map(...).join('');
};
```

**When to use which?**
- **Function declarations** for the main logic functions with names you'll search for: `generatePassword`, `calculateStrength`, `update`. They're also *hoisted* — the JavaScript engine reads them before executing, so you can call them before they appear in the file.
- **Arrow functions without a body** when a function is a single expression: `const double = n => n * 2`. The `return` is implicit.
- **Arrow functions with a body** when a callback needs multiple lines: `array.filter(x => { ... })`.

**Coming up:** In Chapter 7, event handlers use arrow functions: `btn.addEventListener('click', () => { ... })`. In Chapter 9, the game loop uses a named function declaration: `function gameLoop() { requestAnimationFrame(gameLoop); }` — because it needs to call itself by name.

### Default Parameters

Functions can have fallback values for parameters the caller doesn't provide:

```javascript
function generatePassword(length = 16, options = {}) {
  const charset = getCharset(options);
  if (charset.length === 0) return '';

  return Array.from(
    { length },
    () => charset[Math.floor(Math.random() * charset.length)]
  ).join('');
}
```

If you call `generatePassword()` with no arguments: `length` is 16, `options` is `{}`. If you call `generatePassword(32, { upper: true, lower: true })`: those values override the defaults.

**Why defaults?** They make functions useful with minimal input. `generatePassword()` just works. `generatePassword(64, allOptions)` gives full control. Same function, different levels of detail.

**`Array.from({ length }, fn)` — a concise way to build arrays:**

```javascript
Array.from({ length: 5 }, (_, i) => i * 2)
// → [0, 2, 4, 6, 8]

Array.from({ length: 10 }, () => charset[Math.floor(Math.random() * charset.length)])
// → 10 random characters from charset
```

`Array.from` takes anything with a `length` property and a mapping function. Here, `{ length }` is shorthand for `{ length: length }` — it creates an array of `length` elements, each produced by the callback.

### Scope: Who Can See What

Every variable has a **scope** — the region of code where it exists. Understanding scope prevents a whole class of bugs.

```
 Scope diagram:

 ┌─ File (module) scope ─────────────────────────────────────────┐
 │  const upperChars = ...    ← visible to all functions below   │
 │  const lowerChars = ...                                        │
 │  const formatStrength = ...                                    │
 │                                                                │
 │  ┌─ update() scope ─────────────────────────────────────────┐ │
 │  │  const options = getOptions()   ← only inside update()   │ │
 │  │  const length  = getLength()    ← only inside update()   │ │
 │  │  const password = ...           ← only inside update()   │ │
 │  │                                                          │ │
 │  │  ┌─ if block scope ────────────────────────────────────┐ │ │
 │  │  │  const message = 'Please select...'   ← narrowest   │ │ │
 │  │  └────────────────────────────────────────────────────┘ │ │
 │  └──────────────────────────────────────────────────────────┘ │
 └───────────────────────────────────────────────────────────────┘
```

Variables declared inside a function are **local** — they only exist while the function runs. Once `update()` finishes, `options`, `length`, and `password` are gone. The next call to `update()` creates them fresh.

Variables at the top of the file (outside all functions) are **module-level** — every function can read them. The password generator uses these for configuration constants that all functions need to share.

**Why does scope matter?** Keeping variables local prevents bugs. If `generatePassword` and `update` both used a global `length` variable, they'd interfere with each other. Local variables make functions independent.

**Coming up:** In Chapter 9, the game's ball position (`ballX`, `ballY`) is module-level state because multiple functions — `update`, `draw`, `handleCollision` — all need to read and write it. In Chapter 12, `gameState` is module-level for the same reason.

### Closures: Functions That Remember

Here's the part that feels like magic until you see how it works:

```javascript
const makeStrengthFormatter = (labels, colors) => {
  //   ↑ labels and colors are parameters of makeStrengthFormatter

  return (score) => ({       // ← this function is the closure
    label: score < 0 ? '—' : labels[score],   // uses labels from outer scope
    color: score < 0 ? '#64748b' : colors[score],  // uses colors from outer scope
  });
};

const formatStrength = makeStrengthFormatter(
  ['Very Weak', 'Weak', 'Fair', 'Strong', 'Very Strong'],
  ['#e17055',   '#e17055', '#fdcb6e', '#00b894', '#00d4aa']
);
```

`makeStrengthFormatter` runs once and finishes. Normally, its `labels` and `colors` parameters would disappear. But the inner function it returned *captured* them — it holds a reference to those arrays. Later when you call `formatStrength(2)`, it still has access to `labels` and `colors` from when it was created.

```
 Closure memory:

 makeStrengthFormatter finishes, but the returned function keeps:
 ┌────────────────────────────────────────────────┐
 │  Captured scope:                               │
 │    labels → ['Very Weak', 'Weak', ...]         │
 │    colors → ['#e17055', '#e17055', ...]        │
 └────────────────────────────────────────────────┘
          ↑
      formatStrength calls this
      with different scores each time
```

**Why is this useful?** Configuration-once, use-many. You pass the labels and colors *once* when creating the formatter. Every subsequent call to `formatStrength` uses those same arrays without you having to pass them again.

**The practical pattern:** Closures let you bake configuration into a function. Factory functions like `makeStrengthFormatter` produce configured functions you call later. You'll recognize this everywhere:

```javascript
// Same pattern — create a function configured for a specific task
const multiply = (factor) => (n) => n * factor;
const double = multiply(2);   // double(5) → 10
const triple = multiply(3);   // triple(5) → 15
```

**Coming up:** The `createTimer` function in Chapter 3 uses a closure to capture `timeLeft`. The audio scheduler in Chapter 11 uses closures to capture scheduling state. Event handlers — every `addEventListener` callback — are closures that capture surrounding variables.

### Higher-Order Functions: Functions That Take and Return Functions

A **higher-order function** is any function that:
1. Takes a function as an argument, OR
2. Returns a function

You've already used them in Chapter 3: `.filter()`, `.map()`, and `.forEach()` all take functions as arguments.

The password generator uses a more explicit example:

```javascript
// Takes an array of test functions, returns a new function
const checkPassword = (tests) => (password) => tests.every(test => test(password));

// Three small, single-purpose test functions
const isNonEmpty   = (pw) => pw.length > 0;
const isLongEnough = (pw) => pw.length >= 8;

// Compose them into a validator
const isValid = checkPassword([isNonEmpty, isLongEnough]);

// Now isValid is a function: isValid('hello') → false, isValid('helloworld') → true
```

Breaking this down:
- `checkPassword` takes `tests` (an array of functions)
- It returns `(password) => tests.every(test => test(password))` — a new function
- That new function runs every test and returns `true` only if all pass

```
 Data flow through checkPassword:

  isValid('abc')
       ↓
  tests.every(test => test('abc'))
       ↓ runs each test
  isNonEmpty('abc')   → true  ✓
  isLongEnough('abc') → false ✗
       ↓
  false (every requires all to pass)
```

**Why build validators this way?** Adding a new requirement is one line:

```javascript
const hasSymbol = (pw) => /[^A-Za-z0-9]/.test(pw);
const isStrongPassword = checkPassword([isNonEmpty, isLongEnough, hasSymbol]);
```

No if-statement chains. No duplicated logic. The structure makes the rules readable.

**Coming up:** Chapter 10's error handling composes checks similarly — a chain of conditions that all need to pass for a successful API response. Chapter 12's game validation uses the same pattern to check if a game can start.

### Pure Functions vs. Functions with Side Effects

The password generator cleanly separates two types of functions:

**Pure functions** — given the same inputs, always return the same output. No outside state changed:

```javascript
function generatePassword(length = 16, options = {}) { ... }
function calculateStrength(password) { ... }
const getCharset = (options) => { ... }
```

**Functions with side effects** — they reach outside themselves to change the world (the DOM, a global variable):

```javascript
function update() {
  // reads DOM, writes DOM, calls multiple other functions
}
function updateStrengthUI(password) {
  // writes to DOM elements directly
}
```

The pure functions are easy to test and reason about — `generatePassword(16, {upper: true})` always does the same thing. The side-effect functions are harder to test but necessary — something has to update the page.

**The design principle:** Push as much logic as possible into pure functions. Keep the DOM-touching functions thin — just read state, call the pure function, write the result.

```
 Good structure:

  update()                    ← thin: reads DOM, writes DOM
    ↓ calls
  generatePassword(length, options)   ← pure: just computes
    ↓ calls
  getCharset(options)         ← pure: just computes
```

---

## Guided Exercises

### Exercise 1: Pronounceable Password Mode

**The Challenge:** Add a "Pronounceable" checkbox. When checked, generate passwords using alternating consonants and vowels (like `tobuvex` or `kasipom`). These are weaker but easier to remember.

**Where to start:** How would you represent "consonants" and "vowels" as character sets? Where does the generation logic change — in `getCharset`, or in `generatePassword` itself?

*(The generation logic needs to change — you're not just picking from a combined pool, you're alternating. That's a new code path in `generatePassword`.)*

---

**Step 1: Define the character sets.**

```javascript
const vowels     = () => 'aeiou';
const consonants = () => 'bcdfghjklmnpqrstvwxyz';
```

---

**Step 2: Add the pronounceable option.**

In `getOptions()`, add the new checkbox:

```javascript
function getOptions() {
  return {
    upper:        document.getElementById('opt-upper').checked,
    lower:        document.getElementById('opt-lower').checked,
    numbers:      document.getElementById('opt-numbers').checked,
    symbols:      document.getElementById('opt-symbols').checked,
    pronounceable: document.getElementById('opt-pronounceable').checked,  // ← new
  };
}
```

Add the HTML checkbox to the options grid:
```html
<label class="option-label">
  <input type="checkbox" id="opt-pronounceable">
  Pronounceable
  <span class="char-preview">abc</span>
</label>
```

---

**Step 3: Branch on pronounceable in generatePassword.**

```javascript
function generatePassword(length = 16, options = {}) {
  if (options.pronounceable) {
    const v = vowels();
    const c = consonants();
    return Array.from(
      { length },
      (_, i) => {
        const pool = i % 2 === 0 ? c : v;  // alternate: consonant, vowel, consonant...
        return pool[Math.floor(Math.random() * pool.length)];
      }
    ).join('');
  }

  // Original path
  const charset = getCharset(options);
  if (charset.length === 0) return '';
  return Array.from(
    { length },
    () => charset[Math.floor(Math.random() * charset.length)]
  ).join('');
}
```

**Think about it:** When `pronounceable` is on, the strength meter will show these passwords as weak (no uppercase, no symbols, possibly short). That's accurate! Should you disable the strength meter in this mode, or leave it as useful information?

*(Leaving it as information is better — it tells users the tradeoff they're making.)*

**Test it:** Check the pronounceable box and generate a few passwords. They should be readable syllables. The strength bar should drop to Weak or Fair.

---

### Exercise 2: Password History

**The Challenge:** Keep a history of the last 5 generated passwords. Show them below the generator as clickable items — clicking one copies it to the clipboard.

**Where to start:** You need state that persists across calls to `update()`. Where does that live?

*(Outside all functions, at the module level — same reason game state lives at the top in Chapter 9.)*

---

**Step 1: Create the history array.**

At the top of the script:

```javascript
const passwordHistory = [];  // max 5 entries
```

---

**Step 2: Push to history in update().**

After generating a valid password, add it to the front:

```javascript
function update() {
  // ... existing code ...
  const password = generatePassword(length, options);

  passwordHistory.unshift(password);  // add to front (newest first)
  if (passwordHistory.length > 5) passwordHistory.pop();  // keep only 5

  document.getElementById('password-output').value = password;
  updateStrengthUI(password);
  renderHistory();  // ← call after updating
}
```

**Why `unshift` not `push`?** `push` adds to the end. `unshift` adds to the beginning, so newest passwords appear at the top of the history list.

---

**Step 3: Write renderHistory().**

```javascript
function renderHistory() {
  const container = document.getElementById('password-history');
  if (!container) return;

  container.innerHTML = passwordHistory
    .map(pw => `<div class="history-item" onclick="copyToClipboard('${pw}')">${pw}</div>`)
    .join('');
}

function copyToClipboard(text) {
  navigator.clipboard.writeText(text);
}
```

Add to the HTML (below the generate button):
```html
<div id="password-history"></div>
```

**Think about it:** The `onclick` approach embeds the password directly in the HTML attribute. What problem could this cause if the password contains a single quote character `'`? *(The `'` would break out of the attribute string.)*

How would you fix it? *(Use `addEventListener` instead of `onclick`, storing passwords in a data structure and referencing them by index: `data-index="0"` plus an event listener that reads `passwordHistory[el.dataset.index]`.)*

**The complete picture:** The history is a classic application of module-level state: declare outside functions, mutate inside `update()`, read in `renderHistory()`. Three functions, one shared piece of state, clearly defined responsibilities.

---

### Exercise 3: Build Your Own Validator

**The Challenge:** Create a `makeValidator` function that takes a list of rules (each rule is a function that returns `true`/`false` and a message) and returns a validator. The validator returns either `null` (all rules pass) or the *first* failing rule's message.

**Where to start:** This is a higher-order function — it takes functions and returns a function. Look at how `checkPassword` works as a model, but this version needs to return *which* rule failed.

---

**Step 1: Define the rule shape.**

Each rule is an object with a `test` function and a `message`:

```javascript
const rules = [
  { test: pw => pw.length >= 8,            message: 'Must be at least 8 characters' },
  { test: pw => /[A-Z]/.test(pw),          message: 'Must contain an uppercase letter' },
  { test: pw => /[0-9]/.test(pw),          message: 'Must contain a number' },
];
```

---

**Step 2: Write makeValidator.**

```javascript
function makeValidator(rules) {
  return function(password) {
    for (const rule of rules) {
      if (!rule.test(password)) {
        return rule.message;  // return the first failure message
      }
    }
    return null;  // null means "no failures — all rules passed"
  };
}
```

**What's happening here?** `makeValidator` returns a new function (a closure) that captures `rules`. When called with a password, it loops through the rules until one fails, returning that rule's message. If all pass, it returns `null`.

---

**Step 3: Use it.**

```javascript
const validatePassword = makeValidator(rules);

validatePassword('abc')      // → 'Must be at least 8 characters'
validatePassword('abcdefgh') // → 'Must contain an uppercase letter'
validatePassword('Abcdefgh') // → 'Must contain a number'
validatePassword('Abcdefg1') // → null (all rules pass)
```

Add live validation to the generator:

```javascript
function update() {
  // ... generate password ...

  const error = validatePassword(password);
  const warningEl = document.getElementById('warning');
  warningEl.textContent = error || '';  // show error, or clear if null
}
```

**Think about it:** The original `checkPassword` uses `.every()` which is elegant for "all must pass." The `makeValidator` uses a `for...of` loop with early return, which gives us the *message* of the first failure. Both are valid patterns — the right choice depends on what information you need.

**Try extending it:** Add a rule that checks entropy: `{ test: pw => new Set(pw).size >= 8, message: 'Too many repeated characters' }`. The `Set` constructor removes duplicates, so `new Set(pw).size` counts unique characters.

---

## What You Learned

| Concept | What It Does | Real-World Connection |
|---------|-------------|----------------------|
| **Function declaration** | Named, hoisted, callable before definition | The building block of all programs |
| **Arrow functions** | Concise inline functions; implicit return for expressions | React event handlers, array method callbacks |
| **Default parameters** | Fallback values when arguments are omitted | Makes APIs flexible — `fetch(url, { method: 'GET' })` |
| **Return values** | Send data out of a function | Pure functions always return; they're the easiest to test |
| **Scope** | Variables exist only in their declaring block | Prevents naming collisions; keeps functions independent |
| **Closures** | Inner functions capture outer variables | React hooks (`useState`, `useEffect`), event listeners, factories |
| **Higher-order functions** | Take or return functions | `.filter()`, `.map()`, React's `useMemo`, middleware in Express |
| **Pure functions** | Same input → same output, no side effects | Easy to test, easy to reason about, basis of functional programming |
| **Function composition** | Build complex behavior from simple functions | Lodash `_.compose`, Ramda, Redux middleware chain |

### Closures Are Everywhere

Once you see closures, you'll recognize them everywhere:

```javascript
// React hook — the callback closes over count
const [count, setCount] = useState(0);
useEffect(() => {
  console.log(count);  // ← closure: captures count from render scope
}, [count]);

// Event listener — closes over the button element
const btn = document.getElementById('submit');
btn.addEventListener('click', () => {
  btn.disabled = true;  // ← closure: btn is captured from outer scope
});

// setInterval — closes over variables from outer function
function startPolling(url) {
  let attempts = 0;
  const id = setInterval(() => {
    attempts++;               // ← closure: captures attempts
    fetch(url).then(check);
    if (attempts > 10) clearInterval(id);
  }, 1000);
}
```

The rule is simple: when a function accesses a variable from outside itself, that's a closure. You've been writing closures since Chapter 1 — now you know the name.

## Building with Claude

- "Add a 'password rules' display that lists which requirements the current password meets and which it doesn't. Use an array of rule objects, each with a `test` function, `label`, and pass/fail styling."
- "Add a master password strength calculator: test entropy using `Math.log2(charsetSize ** passwordLength)`. Display bits of entropy alongside the strength meter."
- "Add a passphrase generator mode: pick 4 random words from a wordlist separated by dashes. `correct-horse-battery-staple` style. Fetch the wordlist from a text file using `fetch`."
- "Save the last-used settings to `localStorage` so the checkboxes remember their state across page loads. Use `JSON.stringify` and `JSON.parse`."

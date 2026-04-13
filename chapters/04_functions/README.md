# Chapter 4: Functions — Password Generator

In this chapter, we build a working password generator to learn functions from the ground up: declarations, arrow functions, default parameters, return values, scope, closures, and higher-order functions. Every concept is demonstrated in real code.

## The Complete Program

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Password Generator — Chapter 4: Functions</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      background: #0d1b2a;
      color: #c9d6df;
      font-family: 'Segoe UI', system-ui, sans-serif;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 2rem;
    }

    .card {
      background: #112240;
      border: 1px solid #1e3a5f;
      border-radius: 12px;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.5);
      max-width: 520px;
      width: 100%;
      padding: 2rem;
    }

    .card-header {
      display: flex;
      align-items: center;
      gap: 0.75rem;
      margin-bottom: 1.75rem;
    }

    .lock-icon {
      font-size: 1.6rem;
      filter: drop-shadow(0 0 6px #00d4aa88);
    }

    h1 {
      font-size: 1.4rem;
      font-weight: 700;
      color: #e6f1ff;
      letter-spacing: 0.02em;
    }

    h1 span {
      color: #00d4aa;
    }

    .section-label {
      font-size: 0.72rem;
      font-weight: 600;
      letter-spacing: 0.12em;
      text-transform: uppercase;
      color: #64748b;
      margin-bottom: 0.5rem;
    }

    /* Password display */
    .password-display {
      position: relative;
      margin-bottom: 1.5rem;
    }

    #password-output {
      width: 100%;
      background: #0d1b2a;
      border: 1px solid #1e3a5f;
      border-radius: 8px;
      color: #00d4aa;
      font-family: 'Courier New', Courier, monospace;
      font-size: 1.1rem;
      font-weight: 600;
      letter-spacing: 0.06em;
      padding: 0.9rem 3rem 0.9rem 1rem;
      resize: none;
      word-break: break-all;
      min-height: 64px;
      line-height: 1.6;
      outline: none;
    }

    #copy-btn {
      position: absolute;
      top: 0.6rem;
      right: 0.6rem;
      background: #1e3a5f;
      border: 1px solid #2a5080;
      border-radius: 6px;
      color: #a0bcd8;
      cursor: pointer;
      font-size: 1rem;
      padding: 0.3rem 0.5rem;
      transition: background 0.15s, color 0.15s;
    }

    #copy-btn:hover {
      background: #2a5080;
      color: #e6f1ff;
    }

    #copy-btn.copied {
      background: #00d4aa22;
      border-color: #00d4aa;
      color: #00d4aa;
    }

    /* Strength meter */
    .strength-section {
      margin-bottom: 1.5rem;
    }

    .strength-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 0.5rem;
    }

    #strength-label {
      font-size: 0.8rem;
      font-weight: 600;
      letter-spacing: 0.05em;
      color: #64748b;
      transition: color 0.3s;
    }

    .strength-bar {
      display: flex;
      gap: 4px;
    }

    .strength-segment {
      flex: 1;
      height: 6px;
      background: #1e3a5f;
      border-radius: 3px;
      transition: background 0.3s;
    }

    /* Slider */
    .slider-section {
      margin-bottom: 1.25rem;
    }

    .slider-header {
      display: flex;
      justify-content: space-between;
      align-items: baseline;
      margin-bottom: 0.5rem;
    }

    #length-value {
      font-size: 1.1rem;
      font-weight: 700;
      color: #00d4aa;
      font-family: 'Courier New', Courier, monospace;
    }

    #length-slider {
      -webkit-appearance: none;
      width: 100%;
      height: 4px;
      background: #1e3a5f;
      border-radius: 2px;
      outline: none;
      cursor: pointer;
    }

    #length-slider::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 18px;
      height: 18px;
      background: #00d4aa;
      border-radius: 50%;
      cursor: pointer;
      box-shadow: 0 0 8px #00d4aa66;
    }

    /* Checkboxes */
    .options-section {
      margin-bottom: 1.75rem;
    }

    .options-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 0.6rem;
    }

    .option-label {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      cursor: pointer;
      font-size: 0.9rem;
      color: #a0bcd8;
      padding: 0.45rem 0.6rem;
      border-radius: 6px;
      border: 1px solid #1e3a5f;
      background: #0d1b2a;
      transition: border-color 0.15s, background 0.15s;
      user-select: none;
    }

    .option-label:hover {
      border-color: #2a5080;
      background: #112240;
    }

    .option-label input[type="checkbox"] {
      -webkit-appearance: none;
      width: 16px;
      height: 16px;
      border: 2px solid #2a5080;
      border-radius: 4px;
      background: #0d1b2a;
      cursor: pointer;
      flex-shrink: 0;
      position: relative;
      transition: background 0.15s, border-color 0.15s;
    }

    .option-label input[type="checkbox"]:checked {
      background: #00d4aa;
      border-color: #00d4aa;
    }

    .option-label input[type="checkbox"]:checked::after {
      content: '';
      position: absolute;
      left: 3px;
      top: 0px;
      width: 5px;
      height: 9px;
      border: 2px solid #0d1b2a;
      border-top: none;
      border-left: none;
      transform: rotate(45deg);
    }

    .char-preview {
      font-family: 'Courier New', Courier, monospace;
      font-size: 0.7rem;
      color: #64748b;
      margin-left: auto;
    }

    /* Generate button */
    #generate-btn {
      width: 100%;
      background: linear-gradient(135deg, #00b894, #0097e6);
      border: none;
      border-radius: 8px;
      color: #fff;
      cursor: pointer;
      font-size: 1rem;
      font-weight: 700;
      letter-spacing: 0.05em;
      padding: 0.85rem;
      text-transform: uppercase;
      transition: opacity 0.15s, transform 0.1s;
    }

    #generate-btn:hover {
      opacity: 0.9;
    }

    #generate-btn:active {
      transform: scale(0.98);
    }

    .warning {
      font-size: 0.75rem;
      color: #e17055;
      margin-top: 0.5rem;
      text-align: center;
      min-height: 1.2em;
    }
  </style>
</head>
<body>

<div class="card">
  <div class="card-header">
    <span class="lock-icon">&#x1F510;</span>
    <h1>Password <span>Generator</span></h1>
  </div>

  <!-- Password output -->
  <div class="password-display">
    <textarea id="password-output" readonly rows="2" spellcheck="false"></textarea>
    <button id="copy-btn" title="Copy to clipboard">&#x1F4CB;</button>
  </div>

  <!-- Strength meter -->
  <div class="strength-section">
    <div class="strength-header">
      <span class="section-label">Strength</span>
      <span id="strength-label">&#8212;</span>
    </div>
    <div class="strength-bar" id="strength-bar">
      <div class="strength-segment"></div>
      <div class="strength-segment"></div>
      <div class="strength-segment"></div>
      <div class="strength-segment"></div>
      <div class="strength-segment"></div>
    </div>
  </div>

  <!-- Length slider -->
  <div class="slider-section">
    <div class="slider-header">
      <span class="section-label">Length</span>
      <span id="length-value">16</span>
    </div>
    <input type="range" id="length-slider" min="8" max="64" value="16">
  </div>

  <!-- Options -->
  <div class="options-section">
    <p class="section-label">Character Types</p>
    <div class="options-grid">
      <label class="option-label">
        <input type="checkbox" id="opt-upper" checked>
        Uppercase
        <span class="char-preview">A&#8211;Z</span>
      </label>
      <label class="option-label">
        <input type="checkbox" id="opt-lower" checked>
        Lowercase
        <span class="char-preview">a&#8211;z</span>
      </label>
      <label class="option-label">
        <input type="checkbox" id="opt-numbers" checked>
        Numbers
        <span class="char-preview">0&#8211;9</span>
      </label>
      <label class="option-label">
        <input type="checkbox" id="opt-symbols" checked>
        Symbols
        <span class="char-preview">!@#&#8230;</span>
      </label>
    </div>
    <p class="warning" id="warning"></p>
  </div>

  <!-- Generate button -->
  <button id="generate-btn">Generate Password</button>
</div>

<script>
  // ─── Character set builder functions ─────────────────────────────────────

  // Each helper returns one group of characters (or empty string if disabled).
  // These are arrow functions — a short way to write a function.
  const upperChars  = () => 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  const lowerChars  = () => 'abcdefghijklmnopqrstuvwxyz';
  const numberChars = () => '0123456789';
  const symbolChars = () => '!@#$%^&*()-_=+[]{}|;:,.<>?';

  // getCharset uses .filter (a higher-order function) to keep only the groups
  // that are enabled, then maps each group to its characters, then joins them
  // all into one big string of allowed characters.
  const getCharset = (options) => {
    const groups = [
      { enabled: options.upper,   chars: upperChars() },
      { enabled: options.lower,   chars: lowerChars() },
      { enabled: options.numbers, chars: numberChars() },
      { enabled: options.symbols, chars: symbolChars() },
    ];
    return groups
      .filter(group => group.enabled)
      .map(group => group.chars)
      .join('');
  };

  // ─── Core generation function ─────────────────────────────────────────────

  // generatePassword is a function declaration (starts with the word function).
  // It uses default parameters: if you call generatePassword() with no
  // arguments, length defaults to 16 and options defaults to {}.
  // Array.from is a higher-order function — it takes a mapping function
  // and calls it once for each position to build the array.
  function generatePassword(length = 16, options = {}) {
    const charset = getCharset(options);
    if (charset.length === 0) return '';

    return Array.from(
      { length },
      () => charset[Math.floor(Math.random() * charset.length)]
    ).join('');
  }

  // ─── Strength calculation ─────────────────────────────────────────────────

  // calculateStrength returns a number from 0 to 4.
  // Each check that passes adds 1 to the score.
  // /[A-Z]/.test(password) uses a regular expression to check whether
  // the password contains at least one uppercase letter.
  function calculateStrength(password) {
    if (password.length === 0) return -1;

    let score = 0;
    if (password.length >= 12) score++;
    if (password.length >= 20) score++;
    if (/[A-Z]/.test(password)) score++;
    if (/[0-9]/.test(password)) score++;
    if (/[^A-Za-z0-9]/.test(password)) score++;

    return Math.min(score, 4);
  }

  // ─── Closure example: makeStrengthFormatter ──────────────────────────────

  // makeStrengthFormatter is a function that RETURNS another function.
  // The returned function "closes over" (remembers) the labels and colors
  // arrays even after makeStrengthFormatter has finished running.
  // This is called a closure.
  const makeStrengthFormatter = (labels, colors) => {
    return (score) => ({
      label: score < 0 ? '—' : labels[score],
      color: score < 0 ? '#64748b' : colors[score],
    });
  };

  const formatStrength = makeStrengthFormatter(
    ['Very Weak', 'Weak',    'Fair',    'Strong',  'Very Strong'],
    ['#e17055',   '#e17055', '#fdcb6e', '#00b894', '#00d4aa']
  );

  // ─── Higher-order function example: checkPassword ────────────────────────

  // checkPassword takes an ARRAY OF FUNCTIONS and returns a NEW FUNCTION.
  // The new function runs a password through every test.
  // It returns true only if every single test passes.
  // This is a higher-order function because it both receives functions
  // (tests) and returns a function.
  const checkPassword = (tests) => (password) => tests.every(test => test(password));

  const isNonEmpty   = (pw) => pw.length > 0;
  const isLongEnough = (pw) => pw.length >= 8;

  // isValid is a function built from checkPassword — we pass it an array
  // of test functions, and it gives us back a single validator.
  const isValid = checkPassword([isNonEmpty, isLongEnough]);

  // ─── DOM helpers ─────────────────────────────────────────────────────────

  // getOptions reads the current state of the four checkboxes.
  function getOptions() {
    return {
      upper:   document.getElementById('opt-upper').checked,
      lower:   document.getElementById('opt-lower').checked,
      numbers: document.getElementById('opt-numbers').checked,
      symbols: document.getElementById('opt-symbols').checked,
    };
  }

  // getLength reads the current value of the slider and converts it
  // from a string to an integer using parseInt.
  function getLength() {
    return parseInt(document.getElementById('length-slider').value, 10);
  }

  // updateStrengthUI paints the 5-segment bar and updates the label.
  // It uses the score from calculateStrength to decide how many
  // segments to fill in, and what color to use.
  function updateStrengthUI(password) {
    const score     = calculateStrength(password);
    const formatted = formatStrength(score);
    const segments  = document.querySelectorAll('.strength-segment');
    const label     = document.getElementById('strength-label');

    label.textContent = formatted.label;
    label.style.color = formatted.color;

    // Loop through each of the 5 segments
    segments.forEach((seg, i) => {
      seg.style.background = (i <= score && score >= 0)
        ? formatted.color
        : '#1e3a5f';
    });
  }

  // ─── Main update function ─────────────────────────────────────────────────

  // update is called whenever anything changes.
  // It reads the current options, generates a new password,
  // displays it, and updates the strength meter.
  function update() {
    const options = getOptions();
    const length  = getLength();
    const charset = getCharset(options);
    const warning = document.getElementById('warning');

    // If no character types are selected, show a warning instead
    if (charset.length === 0) {
      document.getElementById('password-output').value = '';
      warning.textContent = 'Please select at least one character type.';
      updateStrengthUI('');
      return;
    }

    warning.textContent = '';
    const password = generatePassword(length, options);
    document.getElementById('password-output').value = password;
    updateStrengthUI(password);
  }

  // ─── Event listeners ─────────────────────────────────────────────────────

  // When the slider moves, update the live number label AND regenerate
  document.getElementById('length-slider').addEventListener('input', (e) => {
    document.getElementById('length-value').textContent = e.target.value;
    update();
  });

  // When any checkbox changes, regenerate immediately
  ['opt-upper', 'opt-lower', 'opt-numbers', 'opt-symbols'].forEach(id => {
    document.getElementById(id).addEventListener('change', update);
  });

  // When the button is clicked, regenerate
  document.getElementById('generate-btn').addEventListener('click', update);

  // When the copy button is clicked, copy the password and show feedback
  document.getElementById('copy-btn').addEventListener('click', () => {
    const output = document.getElementById('password-output');
    if (!isValid(output.value)) return;

    navigator.clipboard.writeText(output.value).then(() => {
      const btn = document.getElementById('copy-btn');
      btn.textContent = 'Copied!';
      btn.classList.add('copied');
      setTimeout(() => {
        btn.textContent = '\u{1F4CB}';
        btn.classList.remove('copied');
      }, 1500);
    });
  });

  // ─── Run once on page load ────────────────────────────────────────────────
  update();
</script>
</body>
</html>
```

## Walkthrough

### Function Declarations

A **function declaration** is how we write a named function. Use this for the main logic of your program.

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

A function declaration starts with `function`, has a name (`generatePassword`), takes parameters in parentheses, and has a body in curly braces. We can call it anywhere in our code, even before it's defined. This one generates a random password of a given length using only the character types we ask for.

### Arrow Functions

**Arrow functions** are a shorter syntax for writing functions. Great for short callbacks.

```javascript
const upperChars  = () => 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
const lowerChars  = () => 'abcdefghijklmnopqrstuvwxyz';
const numberChars = () => '0123456789';
```

An arrow function uses `=>` (read: "arrow"). If the body is one expression, you can skip the curly braces and `return` keyword—the value is returned automatically. These three functions take no arguments and return their strings directly. Compare arrow functions to regular functions: both work, but arrows are cleaner when you're building something tiny.

### Parameters and Defaults

**Default parameters** let you specify what value a parameter should have if the caller doesn't provide one.

```javascript
function generatePassword(length = 16, options = {}) {
  // ...
}
```

If I call `generatePassword()` with no arguments, `length` becomes 16 and `options` becomes an empty object. If I call `generatePassword(20, { upper: true })`, those values override the defaults. This makes the function flexible and forgiving.

### Return Values

**Return values** are how a function sends a value back to whoever called it.

```javascript
function calculateStrength(password) {
  let score = 0;
  if (password.length >= 12) score++;
  if (password.length >= 20) score++;
  if (/[A-Z]/.test(password)) score++;
  if (/[0-9]/.test(password)) score++;
  if (/[^A-Za-z0-9]/.test(password)) score++;

  return Math.min(score, 4);
}
```

This function adds up points based on the password's features, then returns a single number from 0 to 4. The `return` statement stops the function and sends the value back. Without it, the function returns `undefined`.

### Scope

**Scope** is the rule for which code can see which variables. Variables declared with `const` and `let` are only visible inside the block where they're declared.

```javascript
function update() {
  const options = getOptions();    // visible only inside update()
  const length  = getLength();     // visible only inside update()
  const charset = getCharset(options);  // visible only inside update()
  const warning = document.getElementById('warning');  // visible only inside update()

  if (charset.length === 0) {
    const message = 'Please select at least one character type.';  // visible only inside this if block
    warning.textContent = message;
  }
}
```

Everything declared inside `update()` is local to `update()`. Once `update()` finishes, those variables disappear. This is good: it keeps your code clean and prevents accidents. Variables declared at the top level (outside any function) are global—every function can see them, but you should use them sparingly.

### Closures

A **closure** is a function that remembers variables from where it was created, even after that place has finished running. This sounds magical, but it's just how functions work.

```javascript
const makeStrengthFormatter = (labels, colors) => {
  return (score) => ({
    label: score < 0 ? '—' : labels[score],
    color: score < 0 ? '#64748b' : colors[score],
  });
};

const formatStrength = makeStrengthFormatter(
  ['Very Weak', 'Weak', 'Fair', 'Strong', 'Very Strong'],
  ['#e17055', '#e17055', '#fdcb6e', '#00b894', '#00d4aa']
);
```

`makeStrengthFormatter` returns a new function. That new function "remembers" the `labels` and `colors` arrays that were passed in, even though `makeStrengthFormatter` has finished running. Every time we call `formatStrength(score)`, it uses those remembered arrays. The inner function captured them—that's the closure.

### Higher-Order Functions

A **higher-order function** is a function that takes functions as arguments or returns a function. Arrays have many: `.filter()`, `.map()`, `.every()`.

```javascript
const getCharset = (options) => {
  const groups = [
    { enabled: options.upper,   chars: upperChars() },
    { enabled: options.lower,   chars: lowerChars() },
    { enabled: options.numbers, chars: numberChars() },
    { enabled: options.symbols, chars: symbolChars() },
  ];
  return groups
    .filter(group => group.enabled)
    .map(group => group.chars)
    .join('');
};
```

Here, `.filter()` takes a function that returns `true` or `false`, and keeps only the items where it's `true`. `.map()` takes a function that transforms each item. Both are higher-order functions. Later:

```javascript
const checkPassword = (tests) => (password) => tests.every(test => test(password));
```

`checkPassword` takes an array of test functions, and returns a new function that runs all tests. This is a higher-order function because it receives functions and returns a function.

## Try It

1. **Longer passwords**: Change the slider's `max` from 64 to 128. Generate a 100-character password and watch the strength meter. What happens?

2. **Add a fourth strength level**: In `calculateStrength`, add another check: `if (password.length >= 32) score++;` Should this be score 5 or 6? Check the strength bar—how many segments does it have?

3. **Time-based validity**: Modify `isValid` to also check that the password was generated "recently" (within the last 5 minutes). You'll need a helper that tracks when the password was created.

## Exercises

1. **Easy**: Add a "Copy and Regenerate" button that copies the password AND immediately generates a new one. Use `.click()` to call your existing functions.

2. **Medium**: Create a function `suggestPassword()` that returns a strong password without needing user input. Give it smart defaults (no symbols if you don't want them, always at least 16 characters). Decide: should `suggestPassword` use the UI checkboxes or ignore them?

3. **Hard**: Write a `passwordsMatch(pwd1, pwd2)` function that returns `true` only if two passwords are equally strong. Use `calculateStrength` inside it. Then build a "Verify Password" mode where you generate one password, and the user has to generate another just as strong.

## Solutions

**Easy: Copy and Regenerate button**

```javascript
document.getElementById('copy-regenerate-btn').addEventListener('click', () => {
  document.getElementById('copy-btn').click();
  document.getElementById('generate-btn').click();
});
```

Or with explicit calls:

```javascript
document.getElementById('copy-regenerate-btn').addEventListener('click', () => {
  const output = document.getElementById('password-output');
  navigator.clipboard.writeText(output.value);
  update();
});
```

**Medium: Smart password suggestion**

```javascript
function suggestPassword(minLength = 16) {
  return generatePassword(minLength, {
    upper: true,
    lower: true,
    numbers: true,
    symbols: true,
  });
}
```

This ignores the UI and always returns a strong password with all character types.

**Hard: Match strength**

```javascript
function passwordsMatch(pwd1, pwd2) {
  return calculateStrength(pwd1) === calculateStrength(pwd2);
}
```

For the game mode, store the target password and its strength, then check after each generation:

```javascript
let targetPassword = generatePassword();
let targetStrength = calculateStrength(targetPassword);

function handleGuess(guessPassword) {
  const guessStrength = calculateStrength(guessPassword);
  if (passwordsMatch(targetPassword, guessPassword)) {
    return 'Correct strength! You matched.';
  } else if (guessStrength < targetStrength) {
    return 'Too weak. Try again.';
  } else {
    return 'Too strong. Try again.';
  }
}
```

## What You Learned

| Concept | Where it appeared |
|---------|-------------------|
| **Function declaration** | `generatePassword(length = 16, options = {})` |
| **Arrow function** | `const upperChars = () => '...'` |
| **Default parameters** | `length = 16, options = {}` |
| **Return value** | `return Math.min(score, 4)` |
| **Scope (local)** | Variables inside `update()` disappear after it runs |
| **Scope (global)** | Variables declared at top level, visible everywhere |
| **Closure** | `makeStrengthFormatter` returns a function that remembers its arguments |
| **Higher-order function** | `.filter()`, `.map()`, `checkPassword` that takes and returns functions |
| **Function as callback** | `addEventListener('click', () => { ... })` |

## Building with Claude

1. **"Add a password history that remembers the last 10 passwords."** You'll need an array at the top level (not inside a function, so it persists), and a function to add to it. Where does the history display?

2. **"Make a 'strong passwords only' mode that refuses to generate anything weaker than 'Strong'."** Modify the generate button to check the strength first. Should it keep regenerating until it finds a strong one, or show an error?

3. **"Add a 'password rules' checker that shows what rules the password breaks."** Create a function `explainStrength(password)` that returns a list like `['No uppercase', 'No symbols']`. Update the UI to show this list below the strength meter.

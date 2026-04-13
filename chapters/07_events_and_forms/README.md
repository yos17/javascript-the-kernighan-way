# Chapter 7: Events and Forms

Events are how JavaScript listens to what the user does — clicks, key presses, form input. They're the bridge between HTML on the page and code that responds. Forms bundle data into a structured message. Together, events and forms let you build interactive programs.

## The Program: Calculator

A working calculator that responds to both mouse clicks and keyboard input. Press keys like `7 + 3 =` or click buttons. It handles decimals, operations, and shows both the current number and the expression you're building.

## The Complete Program

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Calculator</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { font-family: -apple-system, 'Segoe UI', system-ui, sans-serif; background: #111; min-height: 100vh; display: flex; align-items: center; justify-content: center; }

  .calc { width: 320px; background: #1c1c1e; border-radius: 20px; padding: 20px; box-shadow: 0 8px 40px rgba(0,0,0,0.6); }
  .display { text-align: right; padding: 10px 8px 20px; min-height: 90px; display: flex; flex-direction: column; justify-content: flex-end; }
  .display .expr { font-size: 0.95rem; color: #888; min-height: 1.2em; word-break: break-all; }
  .display .result { font-size: 2.8rem; font-weight: 300; color: #fff; word-break: break-all; }

  .buttons { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
  .buttons button { height: 62px; border: none; border-radius: 50%; font-size: 1.3rem; font-weight: 500; cursor: pointer; transition: filter 0.1s, transform 0.08s; }
  .buttons button:active { transform: scale(0.93); filter: brightness(1.3); }
  .buttons button.num  { background: #333; color: #fff; }
  .buttons button.op   { background: #ff9500; color: #fff; }
  .buttons button.func { background: #a5a5a5; color: #000; }
  .buttons button.zero { border-radius: 34px; grid-column: span 2; text-align: left; padding-left: 26px; }

  .hint { text-align: center; color: #555; font-size: 0.75rem; margin-top: 14px; }
</style>
</head>
<body>

<div class="calc">
  <div class="display">
    <div class="expr" id="expr"></div>
    <div class="result" id="result">0</div>
  </div>
  <div class="buttons" id="buttons">
    <button class="func" data-action="clear">AC</button>
    <button class="func" data-action="sign">+/−</button>
    <button class="func" data-action="percent">%</button>
    <button class="op"   data-action="op" data-value="÷">÷</button>

    <button class="num" data-value="7">7</button>
    <button class="num" data-value="8">8</button>
    <button class="num" data-value="9">9</button>
    <button class="op"  data-action="op" data-value="×">×</button>

    <button class="num" data-value="4">4</button>
    <button class="num" data-value="5">5</button>
    <button class="num" data-value="6">6</button>
    <button class="op"  data-action="op" data-value="−">−</button>

    <button class="num" data-value="1">1</button>
    <button class="num" data-value="2">2</button>
    <button class="num" data-value="3">3</button>
    <button class="op"  data-action="op" data-value="+">+</button>

    <button class="num zero" data-value="0">0</button>
    <button class="num" data-value=".">.</button>
    <button class="op"  data-action="equals">=</button>
  </div>
  <div class="hint">Keyboard: 0-9, +, -, *, /, %, Enter, Escape</div>
</div>

<script>
let current = '0';
let previous = null;
let operator = null;
let fresh = true; // screen shows a fresh result, next digit replaces it

const resultEl = document.getElementById('result');
const exprEl = document.getElementById('expr');

function updateDisplay() {
  resultEl.textContent = current;
  exprEl.textContent = previous !== null ? `${previous} ${operator}` : '';
}

function inputDigit(d) {
  if (fresh) { current = d === '.' ? '0.' : d; fresh = false; }
  else if (d === '.' && current.includes('.')) return;
  else if (current === '0' && d !== '.') current = d;
  else current += d;
  updateDisplay();
}

function chooseOp(op) {
  const value = parseFloat(current);
  if (previous !== null && !fresh) {
    current = String(calculate(parseFloat(previous), operator, value));
  }
  previous = current;
  operator = op;
  fresh = true;
  updateDisplay();
}

function calculate(a, op, b) {
  const ops = { '+': a + b, '−': a - b, '×': a * b, '÷': b !== 0 ? a / b : 'Error' };
  const result = ops[op];
  if (typeof result === 'number') {
    return Math.round(result * 1e12) / 1e12; // avoid float weirdness
  }
  return result;
}

function doEquals() {
  if (previous === null || fresh) return;
  const result = calculate(parseFloat(previous), operator, parseFloat(current));
  exprEl.textContent = `${previous} ${operator} ${current} =`;
  current = String(result);
  previous = null;
  operator = null;
  fresh = true;
  resultEl.textContent = current;
}

function doClear() {
  current = '0'; previous = null; operator = null; fresh = true;
  updateDisplay();
}

function doSign() {
  if (current !== '0' && current !== 'Error') {
    current = String(-parseFloat(current));
    updateDisplay();
  }
}

function doPercent() {
  current = String(parseFloat(current) / 100);
  updateDisplay();
}

// Button clicks — event delegation
document.getElementById('buttons').addEventListener('click', e => {
  const btn = e.target.closest('button');
  if (!btn) return;

  const { action, value } = btn.dataset;

  if (!action) inputDigit(value);
  else if (action === 'op') chooseOp(value);
  else if (action === 'equals') doEquals();
  else if (action === 'clear') doClear();
  else if (action === 'sign') doSign();
  else if (action === 'percent') doPercent();
});

// Keyboard support
const KEY_MAP = { '/': '÷', '*': '×', '-': '−', '+': '+' };

document.addEventListener('keydown', e => {
  if (e.key >= '0' && e.key <= '9' || e.key === '.') {
    e.preventDefault();
    inputDigit(e.key);
  } else if (KEY_MAP[e.key]) {
    e.preventDefault();
    chooseOp(KEY_MAP[e.key]);
  } else if (e.key === 'Enter' || e.key === '=') {
    e.preventDefault();
    doEquals();
  } else if (e.key === 'Escape') {
    e.preventDefault();
    doClear();
  } else if (e.key === 'Backspace') {
    e.preventDefault();
    if (!fresh && current.length > 1) current = current.slice(0, -1);
    else current = '0';
    updateDisplay();
  } else if (e.key === '%') {
    e.preventDefault();
    doPercent();
  }
});

updateDisplay();
</script>
</body>
</html>
```

## How It Works

### addEventListener and Event Types

An event listener tells JavaScript "when a specific thing happens, run this code." The most common events are `click` (user presses a button), `keydown` (user presses a key), and `keyup` (user releases a key).

```javascript
document.getElementById('buttons').addEventListener('click', e => { ... });
document.addEventListener('keydown', e => { ... });
```

The callback function receives an `event` object (here called `e`) packed with information about what happened.

### Event Delegation

Instead of adding a separate listener to each of 17 buttons, the code adds one listener to the container `.buttons`. This works because of **bubbling**: when you click a button, the click event bubbles up from that button to its parent to its grandparent, and so on.

The listener uses `e.target.closest('button')` to find the button that was actually clicked, even though the listener is on the container. This is much faster and cleaner than wiring up 17 separate listeners.

### preventDefault

Keyboard events can have default browser behavior (like arrow keys scrolling the page). `e.preventDefault()` stops that. Here it prevents the browser from doing anything with number keys, so only the calculator handles them:

```javascript
if (e.key >= '0' && e.key <= '9') {
  e.preventDefault();
  inputDigit(e.key);
}
```

### Event Object Properties

The event object carries details about what triggered it:
- `e.key`: the key pressed (`'7'`, `'Enter'`, `'Backspace'`)
- `e.target`: the element that was clicked or triggered the event
- `btn.dataset`: an object of all `data-*` attributes on an element (e.g., `data-action="op"` becomes `dataset.action`)

The calculator reads `e.key` to map keyboard presses to actions, and `dataset.action` and `dataset.value` to distinguish button types.

### Bubbling and Capturing

Events travel in two phases. **Capturing** goes down from the root toward the target. **Bubbling** goes up from the target toward the root. By default, `addEventListener` listens during the bubbling phase.

This is why event delegation works: a click on button #7 bubbles up to the container, where the listener catches it. If bubbling didn't exist, you'd have to attach a listener to every single button.

### Calculator State Machine

The calculator stores four pieces of state:
- `current`: the number you're typing now (displayed in `.result`)
- `previous`: the first number in an operation (shown in `.expr` along with the operator)
- `operator`: `+`, `−`, `×`, `÷`
- `fresh`: a boolean flag. When `true`, the next digit replaces the current display instead of appending to it

This flag matters when you press an operator: the display should switch to a fresh state so the next digit starts a new number, not extends the old one.

When you press `=`, the calculator clears `previous`, `operator`, and `fresh`, ready for a new calculation.

## Try It

1. **Add a backspace button.** Create a button with `data-action="backspace"` and code to remove the last digit from `current`. Bonus: make `Backspace` key work too (it already does—check the code!).

2. **Show calculation history.** Keep an array of past calculations. Display the last three results above the main display. Clear them when the user presses AC.

3. **Add keyboard support for more keys.** The code already maps `/` to `÷` and `*` to `×`. Add mappings for `Backspace` to delete the last digit, and test them.

4. **Build a theme toggle.** Add a button that switches between dark and light backgrounds. Store the choice in `localStorage` so it persists even after you close the tab.

## Exercises

**Exercise 1 (Easy).** Add a square button that raises the current number to the power of 2. For example, if the display shows `5`, pressing the square button should show `25`. Do not create a new operator; make it work like the percent button (an immediate transformation).

**Exercise 2 (Medium).** Extend the calculator with memory functions: M+ (add current to memory), M- (subtract current from memory), MR (recall memory), and MC (clear memory). Show the memory value (if non-zero) in a separate display area. Pressing M+ should NOT clear the display; it should store the value and keep the display the same.

**Exercise 3 (Hard).** Add support for chained operations. Right now, if you press `5 + 3 +`, the calculator immediately shows `8`. Modify the code so that pressing two operators in a row does NOT immediately calculate—it just switches operators. Only calculate when you press a different operator. Also add undo: pressing a new button labeled `<` removes the last action from the expression.

## Solutions

### Exercise 1 Solution: Square Button

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Calculator with Square</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { font-family: -apple-system, 'Segoe UI', system-ui, sans-serif; background: #111; min-height: 100vh; display: flex; align-items: center; justify-content: center; }

  .calc { width: 320px; background: #1c1c1e; border-radius: 20px; padding: 20px; box-shadow: 0 8px 40px rgba(0,0,0,0.6); }
  .display { text-align: right; padding: 10px 8px 20px; min-height: 90px; display: flex; flex-direction: column; justify-content: flex-end; }
  .display .expr { font-size: 0.95rem; color: #888; min-height: 1.2em; word-break: break-all; }
  .display .result { font-size: 2.8rem; font-weight: 300; color: #fff; word-break: break-all; }

  .buttons { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
  .buttons button { height: 62px; border: none; border-radius: 50%; font-size: 1.3rem; font-weight: 500; cursor: pointer; transition: filter 0.1s, transform 0.08s; }
  .buttons button:active { transform: scale(0.93); filter: brightness(1.3); }
  .buttons button.num  { background: #333; color: #fff; }
  .buttons button.op   { background: #ff9500; color: #fff; }
  .buttons button.func { background: #a5a5a5; color: #000; }
  .buttons button.zero { border-radius: 34px; grid-column: span 2; text-align: left; padding-left: 26px; }

  .hint { text-align: center; color: #555; font-size: 0.75rem; margin-top: 14px; }
</style>
</head>
<body>

<div class="calc">
  <div class="display">
    <div class="expr" id="expr"></div>
    <div class="result" id="result">0</div>
  </div>
  <div class="buttons" id="buttons">
    <button class="func" data-action="clear">AC</button>
    <button class="func" data-action="sign">+/−</button>
    <button class="func" data-action="square">x²</button>
    <button class="op"   data-action="op" data-value="÷">÷</button>

    <button class="num" data-value="7">7</button>
    <button class="num" data-value="8">8</button>
    <button class="num" data-value="9">9</button>
    <button class="op"  data-action="op" data-value="×">×</button>

    <button class="num" data-value="4">4</button>
    <button class="num" data-value="5">5</button>
    <button class="num" data-value="6">6</button>
    <button class="op"  data-action="op" data-value="−">−</button>

    <button class="num" data-value="1">1</button>
    <button class="num" data-value="2">2</button>
    <button class="num" data-value="3">3</button>
    <button class="op"  data-action="op" data-value="+">+</button>

    <button class="num zero" data-value="0">0</button>
    <button class="num" data-value=".">.</button>
    <button class="op"  data-action="equals">=</button>
  </div>
  <div class="hint">Keyboard: 0-9, +, -, *, /, %, Enter, Escape</div>
</div>

<script>
let current = '0';
let previous = null;
let operator = null;
let fresh = true;

const resultEl = document.getElementById('result');
const exprEl = document.getElementById('expr');

function updateDisplay() {
  resultEl.textContent = current;
  exprEl.textContent = previous !== null ? `${previous} ${operator}` : '';
}

function inputDigit(d) {
  if (fresh) { current = d === '.' ? '0.' : d; fresh = false; }
  else if (d === '.' && current.includes('.')) return;
  else if (current === '0' && d !== '.') current = d;
  else current += d;
  updateDisplay();
}

function chooseOp(op) {
  const value = parseFloat(current);
  if (previous !== null && !fresh) {
    current = String(calculate(parseFloat(previous), operator, value));
  }
  previous = current;
  operator = op;
  fresh = true;
  updateDisplay();
}

function calculate(a, op, b) {
  const ops = { '+': a + b, '−': a - b, '×': a * b, '÷': b !== 0 ? a / b : 'Error' };
  const result = ops[op];
  if (typeof result === 'number') {
    return Math.round(result * 1e12) / 1e12;
  }
  return result;
}

function doEquals() {
  if (previous === null || fresh) return;
  const result = calculate(parseFloat(previous), operator, parseFloat(current));
  exprEl.textContent = `${previous} ${operator} ${current} =`;
  current = String(result);
  previous = null;
  operator = null;
  fresh = true;
  resultEl.textContent = current;
}

function doClear() {
  current = '0'; previous = null; operator = null; fresh = true;
  updateDisplay();
}

function doSign() {
  if (current !== '0' && current !== 'Error') {
    current = String(-parseFloat(current));
    updateDisplay();
  }
}

function doSquare() {
  const n = parseFloat(current);
  current = String(n * n);
  fresh = true;
  updateDisplay();
}

document.getElementById('buttons').addEventListener('click', e => {
  const btn = e.target.closest('button');
  if (!btn) return;

  const { action, value } = btn.dataset;

  if (!action) inputDigit(value);
  else if (action === 'op') chooseOp(value);
  else if (action === 'equals') doEquals();
  else if (action === 'clear') doClear();
  else if (action === 'sign') doSign();
  else if (action === 'square') doSquare();
});

const KEY_MAP = { '/': '÷', '*': '×', '-': '−', '+': '+' };

document.addEventListener('keydown', e => {
  if (e.key >= '0' && e.key <= '9' || e.key === '.') {
    e.preventDefault();
    inputDigit(e.key);
  } else if (KEY_MAP[e.key]) {
    e.preventDefault();
    chooseOp(KEY_MAP[e.key]);
  } else if (e.key === 'Enter' || e.key === '=') {
    e.preventDefault();
    doEquals();
  } else if (e.key === 'Escape') {
    e.preventDefault();
    doClear();
  } else if (e.key === 'Backspace') {
    e.preventDefault();
    if (!fresh && current.length > 1) current = current.slice(0, -1);
    else current = '0';
    updateDisplay();
  }
});

updateDisplay();
</script>
</body>
</html>
```

### Exercise 2 Solution: Memory Functions

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Calculator with Memory</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { font-family: -apple-system, 'Segoe UI', system-ui, sans-serif; background: #111; min-height: 100vh; display: flex; align-items: center; justify-content: center; }

  .calc { width: 320px; background: #1c1c1e; border-radius: 20px; padding: 20px; box-shadow: 0 8px 40px rgba(0,0,0,0.6); }
  .memory { text-align: right; height: 20px; color: #888; font-size: 0.8rem; margin-bottom: 10px; }
  .display { text-align: right; padding: 10px 8px 20px; min-height: 90px; display: flex; flex-direction: column; justify-content: flex-end; }
  .display .expr { font-size: 0.95rem; color: #888; min-height: 1.2em; word-break: break-all; }
  .display .result { font-size: 2.8rem; font-weight: 300; color: #fff; word-break: break-all; }

  .buttons { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
  .buttons button { height: 62px; border: none; border-radius: 50%; font-size: 1.3rem; font-weight: 500; cursor: pointer; transition: filter 0.1s, transform 0.08s; }
  .buttons button:active { transform: scale(0.93); filter: brightness(1.3); }
  .buttons button.num  { background: #333; color: #fff; }
  .buttons button.op   { background: #ff9500; color: #fff; }
  .buttons button.func { background: #a5a5a5; color: #000; }
  .buttons button.mem  { background: #6c7a8f; color: #fff; font-size: 1rem; }
  .buttons button.zero { border-radius: 34px; grid-column: span 2; text-align: left; padding-left: 26px; }

  .hint { text-align: center; color: #555; font-size: 0.75rem; margin-top: 14px; }
</style>
</head>
<body>

<div class="calc">
  <div class="memory" id="memory"></div>
  <div class="display">
    <div class="expr" id="expr"></div>
    <div class="result" id="result">0</div>
  </div>
  <div class="buttons" id="buttons">
    <button class="mem" data-action="m-clear">MC</button>
    <button class="mem" data-action="m-recall">MR</button>
    <button class="mem" data-action="m-add">M+</button>
    <button class="mem" data-action="m-sub">M−</button>

    <button class="func" data-action="clear">AC</button>
    <button class="func" data-action="sign">+/−</button>
    <button class="func" data-action="percent">%</button>
    <button class="op"   data-action="op" data-value="÷">÷</button>

    <button class="num" data-value="7">7</button>
    <button class="num" data-value="8">8</button>
    <button class="num" data-value="9">9</button>
    <button class="op"  data-action="op" data-value="×">×</button>

    <button class="num" data-value="4">4</button>
    <button class="num" data-value="5">5</button>
    <button class="num" data-value="6">6</button>
    <button class="op"  data-action="op" data-value="−">−</button>

    <button class="num" data-value="1">1</button>
    <button class="num" data-value="2">2</button>
    <button class="num" data-value="3">3</button>
    <button class="op"  data-action="op" data-value="+">+</button>

    <button class="num zero" data-value="0">0</button>
    <button class="num" data-value=".">.</button>
    <button class="op"  data-action="equals">=</button>
  </div>
  <div class="hint">M+, M−, MR, MC for memory</div>
</div>

<script>
let current = '0';
let previous = null;
let operator = null;
let fresh = true;
let memory = 0;

const resultEl = document.getElementById('result');
const exprEl = document.getElementById('expr');
const memoryEl = document.getElementById('memory');

function updateDisplay() {
  resultEl.textContent = current;
  exprEl.textContent = previous !== null ? `${previous} ${operator}` : '';
}

function updateMemoryDisplay() {
  memoryEl.textContent = memory !== 0 ? `M: ${memory}` : '';
}

function inputDigit(d) {
  if (fresh) { current = d === '.' ? '0.' : d; fresh = false; }
  else if (d === '.' && current.includes('.')) return;
  else if (current === '0' && d !== '.') current = d;
  else current += d;
  updateDisplay();
}

function chooseOp(op) {
  const value = parseFloat(current);
  if (previous !== null && !fresh) {
    current = String(calculate(parseFloat(previous), operator, value));
  }
  previous = current;
  operator = op;
  fresh = true;
  updateDisplay();
}

function calculate(a, op, b) {
  const ops = { '+': a + b, '−': a - b, '×': a * b, '÷': b !== 0 ? a / b : 'Error' };
  const result = ops[op];
  if (typeof result === 'number') {
    return Math.round(result * 1e12) / 1e12;
  }
  return result;
}

function doEquals() {
  if (previous === null || fresh) return;
  const result = calculate(parseFloat(previous), operator, parseFloat(current));
  exprEl.textContent = `${previous} ${operator} ${current} =`;
  current = String(result);
  previous = null;
  operator = null;
  fresh = true;
  resultEl.textContent = current;
}

function doClear() {
  current = '0'; previous = null; operator = null; fresh = true;
  updateDisplay();
}

function doSign() {
  if (current !== '0' && current !== 'Error') {
    current = String(-parseFloat(current));
    updateDisplay();
  }
}

function doPercent() {
  current = String(parseFloat(current) / 100);
  updateDisplay();
}

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
  fresh = true;
  updateDisplay();
}

function doMemClear() {
  memory = 0;
  updateMemoryDisplay();
}

document.getElementById('buttons').addEventListener('click', e => {
  const btn = e.target.closest('button');
  if (!btn) return;

  const { action, value } = btn.dataset;

  if (!action) inputDigit(value);
  else if (action === 'op') chooseOp(value);
  else if (action === 'equals') doEquals();
  else if (action === 'clear') doClear();
  else if (action === 'sign') doSign();
  else if (action === 'percent') doPercent();
  else if (action === 'm-add') doMemAdd();
  else if (action === 'm-sub') doMemSub();
  else if (action === 'm-recall') doMemRecall();
  else if (action === 'm-clear') doMemClear();
});

const KEY_MAP = { '/': '÷', '*': '×', '-': '−', '+': '+' };

document.addEventListener('keydown', e => {
  if (e.key >= '0' && e.key <= '9' || e.key === '.') {
    e.preventDefault();
    inputDigit(e.key);
  } else if (KEY_MAP[e.key]) {
    e.preventDefault();
    chooseOp(KEY_MAP[e.key]);
  } else if (e.key === 'Enter' || e.key === '=') {
    e.preventDefault();
    doEquals();
  } else if (e.key === 'Escape') {
    e.preventDefault();
    doClear();
  } else if (e.key === 'Backspace') {
    e.preventDefault();
    if (!fresh && current.length > 1) current = current.slice(0, -1);
    else current = '0';
    updateDisplay();
  }
});

updateDisplay();
updateMemoryDisplay();
</script>
</body>
</html>
```

### Exercise 3 Solution: No Immediate Calc + Undo

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Calculator No-Immediate + Undo</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { font-family: -apple-system, 'Segoe UI', system-ui, sans-serif; background: #111; min-height: 100vh; display: flex; align-items: center; justify-content: center; }

  .calc { width: 320px; background: #1c1c1e; border-radius: 20px; padding: 20px; box-shadow: 0 8px 40px rgba(0,0,0,0.6); }
  .display { text-align: right; padding: 10px 8px 20px; min-height: 90px; display: flex; flex-direction: column; justify-content: flex-end; }
  .display .expr { font-size: 0.95rem; color: #888; min-height: 1.2em; word-break: break-all; }
  .display .result { font-size: 2.8rem; font-weight: 300; color: #fff; word-break: break-all; }

  .buttons { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
  .buttons button { height: 62px; border: none; border-radius: 50%; font-size: 1.3rem; font-weight: 500; cursor: pointer; transition: filter 0.1s, transform 0.08s; }
  .buttons button:active { transform: scale(0.93); filter: brightness(1.3); }
  .buttons button.num  { background: #333; color: #fff; }
  .buttons button.op   { background: #ff9500; color: #fff; }
  .buttons button.func { background: #a5a5a5; color: #000; }
  .buttons button.zero { border-radius: 34px; grid-column: span 2; text-align: left; padding-left: 26px; }

  .hint { text-align: center; color: #555; font-size: 0.75rem; margin-top: 14px; }
</style>
</head>
<body>

<div class="calc">
  <div class="display">
    <div class="expr" id="expr"></div>
    <div class="result" id="result">0</div>
  </div>
  <div class="buttons" id="buttons">
    <button class="func" data-action="clear">AC</button>
    <button class="func" data-action="undo">← Undo</button>
    <button class="func" data-action="percent">%</button>
    <button class="op"   data-action="op" data-value="÷">÷</button>

    <button class="num" data-value="7">7</button>
    <button class="num" data-value="8">8</button>
    <button class="num" data-value="9">9</button>
    <button class="op"  data-action="op" data-value="×">×</button>

    <button class="num" data-value="4">4</button>
    <button class="num" data-value="5">5</button>
    <button class="num" data-value="6">6</button>
    <button class="op"  data-action="op" data-value="−">−</button>

    <button class="num" data-value="1">1</button>
    <button class="num" data-value="2">2</button>
    <button class="num" data-value="3">3</button>
    <button class="op"  data-action="op" data-value="+">+</button>

    <button class="num zero" data-value="0">0</button>
    <button class="num" data-value=".">.</button>
    <button class="op"  data-action="equals">=</button>
  </div>
  <div class="hint">Pressing two ops in a row just switches. Press = to calculate.</div>
</div>

<script>
let current = '0';
let previous = null;
let operator = null;
let fresh = true;
let history = [];

const resultEl = document.getElementById('result');
const exprEl = document.getElementById('expr');

function updateDisplay() {
  resultEl.textContent = current;
  exprEl.textContent = previous !== null ? `${previous} ${operator}` : '';
}

function saveState() {
  history.push({ current, previous, operator, fresh });
}

function inputDigit(d) {
  if (fresh) { current = d === '.' ? '0.' : d; fresh = false; }
  else if (d === '.' && current.includes('.')) return;
  else if (current === '0' && d !== '.') current = d;
  else current += d;
  updateDisplay();
}

function chooseOp(op) {
  // If we already have an operator and current display is not fresh, do NOT calculate.
  // Just change the operator.
  if (operator !== null && !fresh) {
    // We pressed a second operator in a row. Just switch it without calculating.
    operator = op;
  } else if (operator === null) {
    // First operator.
    saveState();
    previous = current;
    operator = op;
    fresh = true;
  } else {
    // operator !== null and fresh === true (we just pressed an operator)
    // This is consecutive operators. Just switch.
    operator = op;
  }
  updateDisplay();
}

function calculate(a, op, b) {
  const ops = { '+': a + b, '−': a - b, '×': a * b, '÷': b !== 0 ? a / b : 'Error' };
  const result = ops[op];
  if (typeof result === 'number') {
    return Math.round(result * 1e12) / 1e12;
  }
  return result;
}

function doEquals() {
  if (previous === null || operator === null) return;
  const result = calculate(parseFloat(previous), operator, parseFloat(current));
  exprEl.textContent = `${previous} ${operator} ${current} =`;
  current = String(result);
  previous = null;
  operator = null;
  fresh = true;
  resultEl.textContent = current;
}

function doClear() {
  current = '0'; previous = null; operator = null; fresh = true;
  history = [];
  updateDisplay();
}

function doUndo() {
  if (history.length > 0) {
    const state = history.pop();
    current = state.current;
    previous = state.previous;
    operator = state.operator;
    fresh = state.fresh;
    updateDisplay();
  }
}

function doPercent() {
  current = String(parseFloat(current) / 100);
  updateDisplay();
}

document.getElementById('buttons').addEventListener('click', e => {
  const btn = e.target.closest('button');
  if (!btn) return;

  const { action, value } = btn.dataset;

  if (!action) inputDigit(value);
  else if (action === 'op') chooseOp(value);
  else if (action === 'equals') doEquals();
  else if (action === 'clear') doClear();
  else if (action === 'undo') doUndo();
  else if (action === 'percent') doPercent();
});

const KEY_MAP = { '/': '÷', '*': '×', '-': '−', '+': '+' };

document.addEventListener('keydown', e => {
  if (e.key >= '0' && e.key <= '9' || e.key === '.') {
    e.preventDefault();
    inputDigit(e.key);
  } else if (KEY_MAP[e.key]) {
    e.preventDefault();
    chooseOp(KEY_MAP[e.key]);
  } else if (e.key === 'Enter' || e.key === '=') {
    e.preventDefault();
    doEquals();
  } else if (e.key === 'Escape') {
    e.preventDefault();
    doClear();
  } else if (e.key === 'Backspace') {
    e.preventDefault();
    if (!fresh && current.length > 1) current = current.slice(0, -1);
    else current = '0';
    updateDisplay();
  }
});

updateDisplay();
</script>
</body>
</html>
```

## What You Learned

| Concept | What It Does |
|---------|--------------|
| `addEventListener` | Registers a function to run when an event happens on an element |
| `click` event | Fires when the user clicks a button or element |
| `keydown` event | Fires when the user presses a key down (fires repeatedly if held) |
| Event delegation | One listener on a container catches events from many children via bubbling |
| `e.target.closest()` | Walks up the DOM tree to find the nearest ancestor matching a selector |
| `e.preventDefault()` | Stops the browser's default action for an event (e.g., form submission, key behavior) |
| `e.key` | The keyboard key that triggered the event (e.g., `'7'`, `'Enter'`, `'Backspace'`) |
| `dataset` attribute | Reads `data-*` attributes as an object on an element |
| Event bubbling | Events travel up from the target element to its parents (enables delegation) |
| State machine | Variables (`current`, `previous`, `operator`, `fresh`) that track the calculator's mode |

## Building with Claude

- Ask Claude to "add a square root button that calculates the square root of the current number" and paste the solution into your HTML.
- Prompt Claude to "add localStorage so the calculator remembers the memory value even after you close the browser" and copy the modified script.
- Say "Build a 'calculation history' feature that shows the last 5 calculations in a popup when you press a history button" to see how to use an array and a modal dialog.
- Challenge Claude: "Extend this calculator to support parentheses, like (5 + 3) * 2" to learn about operator precedence and recursive parsing.

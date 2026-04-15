# Chapter 4 — Build Your Own Calculator

This is the hardest chapter in the first part of the book, but it is worth it. A calculator takes a string like `"(2 + 3) * 4"` and turns it into a result. To build that, you have to break a larger problem into smaller steps. That is one of the most important programming skills you can learn.

---

## The Problem

A beginner might try to evaluate `"2 + 3 * 4"` by splitting the string and working from left to right. But that gives the wrong answer. Multiplication must happen before addition, unless parentheses say otherwise.

String-splitting can't handle this. You need to understand the *structure* of the expression — that `2 + (3 * 4)` is an addition of `2` and the product `3 * 4`. That structure is a tree:

```
      +
     / \
    2   *
       / \
      3   4
```

Evaluate the tree bottom-up: `3 * 4 = 12`, then `2 + 12 = 14`. Operator precedence is captured by where things appear in the tree — multiplication is lower in the tree (evaluated first) because its parsing function is called deeper in the recursion.

This is also how real language tools work. They do not guess from the raw string. They break the input into pieces, understand the structure, and then compute the result.

For beginners, it helps to think of the calculator as three separate jobs:

1. **tokenizing**: turn the raw string into small pieces
2. **parsing**: figure out how those pieces belong together
3. **evaluating**: compute the final answer from that structure

If this chapter feels hard, do not try to hold the whole calculator in your head at once. Pick one of those three jobs and understand it in isolation.

---

## Beginner Summary

Before you dive into the code, keep this in mind:

- the calculator is not one big trick, it is three smaller jobs
- first break the input into pieces
- then understand how those pieces fit together
- then compute the answer from that structure

## Building It Step by Step

### v1 — A Function That Computes

The simplest calculator: functions that take numbers and return numbers.

```javascript
// v1: pure functions — inputs in, output out
function add(a, b)      { return a + b; }
function subtract(a, b) { return a - b; }
function multiply(a, b) { return a * b; }
function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}

// Compose them:
add(2, multiply(3, 4));   // 2 + (3 * 4) = 14
```

This works for hardcoded expressions, but can't parse strings. The input is already structured (function calls) — there's nothing to parse.

### v2 — A Tokenizer

In plain English: the tokenizer reads the string left to right and labels each piece so later code does not have to work with raw characters anymore.

The second step: split a string into typed tokens. `"2 + 3 * 4"` becomes a flat list:

```javascript
// v2: tokenize — split string into typed tokens
function tokenize(input) {
  const tokens = [];
  let i = 0;

  while (i < input.length) {
    const ch = input[i];

    if (ch === ' ') { i++; continue; }   // skip whitespace

    if (ch >= '0' && ch <= '9') {        // number
      let num = '';
      while (i < input.length && (input[i] >= '0' && input[i] <= '9' || input[i] === '.')) {
        num += input[i++];
      }
      tokens.push({ type: 'number', value: parseFloat(num) });
      continue;
    }

    if ('+-*/^()'.includes(ch)) {         // operator or paren
      tokens.push({ type: 'op', value: ch });
      i++;
      continue;
    }

    throw new Error(`Unexpected character: "${ch}"`);
  }

  tokens.push({ type: 'end', value: null });
  return tokens;
}

tokenize('2 + 3 * 4');
// [
//   { type: 'number', value: 2 },
//   { type: 'op', value: '+' },
//   { type: 'number', value: 3 },
//   { type: 'op', value: '*' },
//   { type: 'number', value: 4 },
//   { type: 'end', value: null }
// ]
```

The token stream is flat — all structural information (precedence, grouping) is gone. Recovering it is the parser's job.

That means tokenizing does not solve the whole problem. It only prepares the data for the next step. This is a common programming pattern: one stage simplifies the input, then the next stage does the deeper thinking.

### v3 — A Recursive-Descent Parser and Evaluator

In plain English: the parser turns the flat token list into a shape that matches mathematical meaning.

The full system: tokenize → parse into a tree → evaluate the tree.

The parser is a set of mutually-recursive functions, one per grammar rule. The rules encode precedence:

```
expr  (+ -)  →  calls term
term  (* /)  →  calls power
power (^)    →  calls unary
unary (-)    →  calls call
call (fn())  →  calls primary
primary      →  number or ( expr )
```

Lower in the chain = higher precedence. `*` binds tighter than `+` because `parseTerm` calls `parsePower` (which handles `*`'s operands), and `parseExpr` calls `parseTerm` (which handles `+`'s operands). The recursion naturally builds the right tree.

```javascript
// v3 skeleton: each grammar rule is one function
function parse(tokens) {
  let pos = 0;
  const peek    = () => tokens[pos];
  const consume = () => tokens[pos++];

  function parseExpr() {
    let left = parseTerm();
    while (peek().value === '+' || peek().value === '-') {
      const op    = consume().value;
      const right = parseTerm();
      left = { type: 'binary', op, left, right };
    }
    return left;
  }

  function parseTerm() {
    let left = parsePower();
    while (peek().value === '*' || peek().value === '/') {
      const op    = consume().value;
      const right = parsePower();
      left = { type: 'binary', op, left, right };
    }
    return left;
  }

  // ... parsePower, parseUnary, parsePrimary
  return parseExpr();
}

function evaluate(node) {
  if (node.type === 'number') return node.value;
  if (node.type === 'binary') {
    const left  = evaluate(node.left);   // recursion: evaluate subtrees first
    const right = evaluate(node.right);
    if (node.op === '+') return left + right;
    if (node.op === '-') return left - right;
    if (node.op === '*') return left * right;
    if (node.op === '/') return left / right;
    if (node.op === '^') return Math.pow(left, right);
  }
}
```

---

## The Complete Program

`calculator.html` — open it in your browser. Type any arithmetic expression and press Enter. The token stream and parse tree update in real time, showing exactly how the expression was analyzed.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Build Your Own Calculator — Chapter 4</title>
<!-- styles in calculator.html -->
</head>
<body>
<script>
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: calc
// ═══════════════════════════════════════════════════════════════════════

// (See calculator.html for the full tokenize/parse/evaluate implementation)

const FUNCTIONS = {
  sqrt:  args => Math.sqrt(args[0]),
  abs:   args => Math.abs(args[0]),
  floor: args => Math.floor(args[0]),
  ceil:  args => Math.ceil(args[0]),
  round: args => Math.round(args[0]),
  log:   args => Math.log(args[0]),
  max:   args => Math.max(...args),
  min:   args => Math.min(...args),
};

const CONSTANTS = { pi: Math.PI, e: Math.E };

const calc = {
  evaluate: function(input) {
    const tokens = tokenize(input.trim());
    const ast    = parse(tokens);
    return evaluate(ast);
  }
};
</script>
</body>
</html>
```

---

## Walkthrough

### Functions — Declarations, Expressions, Arrows

JavaScript has three syntaxes for defining functions. They differ in hoisting and `this` binding:

```javascript
// 1. Function declaration — hoisted to the top of the scope
function add(a, b) {
  return a + b;
}
add(2, 3);   // 5

// 2. Function expression — assigned to a variable; not hoisted
const multiply = function(a, b) {
  return a * b;
};

// 3. Arrow function — concise; no own `this`
const square = (n) => n * n;           // implicit return
const cube   = (n) => { return n * n * n; };  // explicit return

// Single-param arrows can omit parentheses:
const double = n => n * 2;
```

In the calculator, most functions are **declarations** (`function tokenize(...)`, `function parse(...)`) because they call each other by name, and declarations are available throughout their scope regardless of order. The inner parser functions (`parseExpr`, `parseTerm`) are also declarations, nested inside `parse`.

**Hoisting** means function declarations are available before their line in the code:

```javascript
greet('Ada');    // Works! greet is hoisted

function greet(name) {
  return `Hello, ${name}`;
}
```

Function expressions are not hoisted:

```javascript
greet('Ada');    // TypeError: greet is not a function

const greet = function(name) {
  return `Hello, ${name}`;
};
```

### Parameters, Return Values, Default Parameters

```javascript
// Parameters are local variables in the function's scope
function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}

// Default parameters — used when the argument is undefined
function power(base, exp = 2) {
  return Math.pow(base, exp);
}
power(3);     // 9  — exp defaults to 2
power(3, 3);  // 27 — exp is explicitly 3

// Rest parameters — collect remaining args into an array
function sum(...numbers) {
  let total = 0;
  for (const n of numbers) total += n;
  return total;
}
sum(1, 2, 3, 4);    // 10
sum(1, 2);          // 3
```

The calculator's `FUNCTIONS` object uses rest parameters implicitly — `max` and `min` receive an array of arguments:

```javascript
const FUNCTIONS = {
  max: (args) => Math.max(...args),   // spread the args array
  min: (args) => Math.min(...args),
};
```

`Math.max(...[3, 7, 2])` is equivalent to `Math.max(3, 7, 2)`. The spread operator `...` unpacks an array into individual arguments.

### Scope — Where Variables Live

Every function creates a new **scope** — a region where variables are defined. Inner functions can see outer variables (closure); outer functions cannot see inner variables.

```javascript
function parse(tokens) {
  let pos = 0;          // pos is scoped to parse()

  function peek() {
    return tokens[pos]; // peek() can see pos and tokens — they're in the outer scope
  }

  function consume() {
    return tokens[pos++]; // ++ increments pos — modifies the outer variable
  }

  function parseExpr() {
    // Can call parseTerm, peek, consume — all in the same scope
    let left = parseTerm();
    // ...
  }

  // pos is only accessible here — not from outside parse()
  return parseExpr();
}
```

The nested parser functions (`parseExpr`, `parseTerm`, etc.) all share `pos`, `peek`, and `consume` via closure. When `consume()` increments `pos`, all other functions see the updated value — they all close over the same `pos` variable.

```
Closure in parse()
──────────────────────────────────────────────────
parse() scope:
  pos      = 0        ← shared mutable state
  tokens   = [...]    ← the input
  peek     = fn       ← reads pos
  consume  = fn       ← reads and modifies pos
  parseExpr = fn      ← calls parseTerm, which calls peek/consume
  parseTerm = fn      ← calls parsePower, which calls peek/consume

All inner functions see the same pos.
When consume() does pos++, every function that
reads pos will see the new value on next call.
```

This is the same closure pattern as Chapter 1's `createLogger` — an outer function creates shared state, and inner functions close over it.

### Recursion — Functions That Call Themselves

The evaluator is recursive: to evaluate a binary node, it evaluates its left and right children first.

```javascript
function evaluate(node) {
  if (node.type === 'number') {
    return node.value;          // base case: just return the number
  }

  if (node.type === 'binary') {
    const left  = evaluate(node.left);   // ← recursive call
    const right = evaluate(node.right);  // ← recursive call
    // Both subtrees are now evaluated to numbers
    switch (node.op) {
      case '+': return left + right;
      case '*': return left * right;
      // ...
    }
  }
}
```

Every recursive function needs a **base case** (something that doesn't recurse) and a **recursive case** (something that does). Without a base case, the function would call itself forever — a stack overflow.

```
evaluate({ type: 'binary', op: '+',
  left:  { type: 'binary', op: '*',
    left:  { type: 'number', value: 3 },
    right: { type: 'number', value: 4 } },
  right: { type: 'number', value: 2 } })

  → evaluate left:  evaluate(3*4)
      → evaluate left:  evaluate(3) = 3   ← base case
      → evaluate right: evaluate(4) = 4   ← base case
      → return 3 * 4 = 12
  → evaluate right: evaluate(2) = 2       ← base case
  → return 12 + 2 = 14
```

The call stack grows as we go deeper into the tree, then unwinds as each call returns its value to its parent.

### throw and Error Objects

When something goes wrong, JavaScript throws an error. You create errors with `new Error(message)`:

```javascript
function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}
```

`throw` immediately stops the function and propagates up the call stack until something catches it. If nothing catches it, the browser shows an uncaught error.

The calculator throws in several places:
- `tokenize`: unexpected character
- `parse`: unexpected token, mismatched parentheses
- `evaluate`: division by zero, unknown function

The application catches them:

```javascript
try {
  const result = calc.evaluate(input);
  console.log(result);
} catch (e) {
  console.log('Error:', e.message);   // e.message is the string passed to new Error()
}
```

`try/catch` is the only way to handle thrown errors gracefully. Without it, a division-by-zero in the calculator would crash the whole page.

### switch Statements

`switch` compares a value against multiple cases:

```javascript
switch (node.op) {
  case '+': return left + right;
  case '-': return left - right;
  case '*': return left * right;
  case '/':
    if (right === 0) throw new Error('Division by zero');
    return left / right;
  case '^': return Math.pow(left, right);
  default:  throw new Error(`Unknown operator: ${node.op}`);
}
```

This is equivalent to `if/else if/else`, but cleaner when matching one variable against many exact values. The `default` case handles everything that doesn't match — same as `else`.

**Always include `default`** when the input could be anything unexpected. Without it, an unknown operator silently returns `undefined`.

### Objects as Function Registries

The `FUNCTIONS` object maps function names to implementations:

```javascript
const FUNCTIONS = {
  sqrt:  (args) => Math.sqrt(args[0]),
  abs:   (args) => Math.abs(args[0]),
  max:   (args) => Math.max(...args),
};

// Look up and call by string name:
const name = 'sqrt';
const fn   = FUNCTIONS[name];         // bracket notation: key is a variable
if (!fn) throw new Error(`Unknown function: ${name}`);
const result = fn([16]);              // 4
```

You built this pattern in Chapter 2 (the `RULES` registry) and Chapter 1 (the `LEVELS` lookup). Here it appears again, this time to dispatch function calls. It's more flexible than `switch` because you can add functions at runtime by adding properties to the object.

---

## Guided Try It — Add User-Defined Variables

**The goal**: allow the user to define variables in the REPL. `x = 10` stores `x`; subsequent expressions can use `x`. `x + 5` should evaluate to `15`.

**Step 1 — Detect assignment syntax**

Before parsing, check if the input looks like `name = expr`:

```javascript
function evalInput(input) {
  // Check for assignment: word, optional whitespace, =, expression
  const assignMatch = input.match(/^([a-zA-Z_]\w*)\s*=\s*(.+)$/);

  if (assignMatch) {
    const [, name, expr] = assignMatch;
    const value = calc.evaluate(expr);
    // Store the variable somewhere...
    return `${name} = ${value}`;
  }

  return calc.evaluate(input);
}
```

**Step 2 — Create a variable store**

A plain object maps variable names to values:

```javascript
const variables = { pi: Math.PI, e: Math.E };  // pre-seed constants

function setVariable(name, value) {
  variables[name] = value;
}

function getVariable(name) {
  if (name in variables) return variables[name];
  throw new Error(`Undefined variable: ${name}`);
}
```

**Step 3 — Thread the variable store into the evaluator**

Update `evaluate` to look up names in the variable store:

```javascript
function evaluate(node, vars) {    // vars parameter added
  if (node.type === 'number') return node.value;

  if (node.type === 'name') {
    // Look up the name in the variable store
    if (node.value in vars) return vars[node.value];
    if (node.value in CONSTANTS) return CONSTANTS[node.value];
    throw new Error(`Undefined variable: ${node.value}`);
  }

  if (node.type === 'binary') {
    const left  = evaluate(node.left, vars);   // pass vars through
    const right = evaluate(node.right, vars);
    // ...
  }

  if (node.type === 'call') {
    const fn   = FUNCTIONS[node.name];
    if (!fn) throw new Error(`Unknown function: ${node.name}`);
    const args = node.args.map(a => evaluate(a, vars));  // pass vars through
    return fn(args);
  }
}
```

**Step 4 — Try it**

```javascript
const vars = {};
calc.evaluate('x = 10', vars);    // stores x = 10
calc.evaluate('y = x + 5', vars); // stores y = 15
calc.evaluate('x * y', vars);     // 150
```

**Think about it**: the `variables` object is a very simple symbol table. Real calculators and programming languages use the same idea but with more structure — nested scopes, so `x` defined inside a function doesn't clobber `x` outside it. Chapter 5's `_.memoize` and Chapter 11's Redux both use similar "state in an object" patterns. How would you support scoped variables (functions that have their own local `x`)?

---

## Exercises

1. **Add the `%` (modulo) operator**: Add `%` to the tokenizer's operator list and add a `case '%': return left % right;` to the evaluator's switch. Update `parseTerm` to handle `%` at the same precedence as `*` and `/`. Test: `17 % 5` should return `2`.

2. **Add string support**: Extend the tokenizer to recognize quoted strings like `"hello"`. When the tokenizer sees `"`, it should collect characters until the closing `"` and push `{ type: 'string', value: 'hello' }`. The evaluator returns the string as-is for `string` nodes. Test: the parser should handle `"hello"` as a literal.

3. **Add a `define` form**: Support `define(name, expr)` syntax that stores a named constant. `define(tax, 0.08)` followed by `100 * tax` should return `8`. Add `define` as a special case in `parseCall` that adds to a global constants map rather than calling a function.

4. **Build `calc.explain(expression)`**: A function that returns a string showing the step-by-step evaluation. `calc.explain('2 + 3 * 4')` should return something like `"3 * 4 = 12, then 2 + 12 = 14"`. Walk the AST and build the explanation as you evaluate, using an array to collect steps.

5. **Add operator precedence climbing**: The current parser handles five precedence levels by having five functions. Research "Pratt parser" or "operator precedence climbing" and redesign the precedence levels as a table: `{ '+': 1, '-': 1, '*': 2, '/': 2, '^': 3 }`. Rewrite `parseExpr` and `parseTerm` as a single `parseInfix(minPrecedence)` function that uses the table instead of hardcoded checks.

---

## Solutions

### Exercise 1 — Modulo operator

```javascript
// In tokenize(): '+-*/^%(),' — add % to the includes check
if ('+-*/^%(),'.includes(ch)) {
  tokens.push({ type: ch === ',' ? 'comma' : 'op', value: ch });
  i++;
  continue;
}

// In parseTerm(): add % alongside * and /
function parseTerm() {
  let left = parsePower();
  while (peek().type === 'op' &&
         (peek().value === '*' || peek().value === '/' || peek().value === '%')) {
    const op    = consume().value;
    const right = parsePower();
    left = { type: 'binary', op, left, right };
  }
  return left;
}

// In evaluate(): add the % case
case '%':
  if (right === 0) throw new Error('Modulo by zero');
  return left % right;

// Tests:
calc.evaluate('17 % 5');    // 2
calc.evaluate('100 % 3');   // 1
calc.evaluate('10 % 10');   // 0
```

### Exercise 2 — String literals

```javascript
// Add to tokenize(): detect opening quote
if (ch === '"') {
  i++;  // skip opening quote
  let str = '';
  while (i < input.length && input[i] !== '"') {
    if (input[i] === '\\' && i + 1 < input.length) {
      // Simple escape sequences: \" and \\
      i++;
      str += input[i] === '"' ? '"' : '\\' + input[i];
    } else {
      str += input[i];
    }
    i++;
  }
  if (i >= input.length) throw new Error('Unterminated string');
  i++;  // skip closing quote
  tokens.push({ type: 'string', value: str });
  continue;
}

// Add to parsePrimary():
if (tok.type === 'string') {
  consume();
  return { type: 'string', value: tok.value };
}

// Add to evaluate():
if (node.type === 'string') {
  return node.value;   // strings evaluate to themselves
}

// Tests:
calc.evaluate('"hello"');           // "hello"
calc.evaluate('"world"');           // "world"
// Note: + on strings would require type checking in the binary evaluator
```

### Exercise 3 — `define` form

```javascript
const userConstants = {};   // separate from built-in CONSTANTS

// Add to parseCall() — special-case 'define':
if (tok.type === 'name' && tok.value === 'define') {
  consume();
  expect('op', '(');
  const nameTok = consume();
  if (nameTok.type !== 'name') throw new Error('define() requires a name');
  expect('comma');
  const valueExpr = parseExpr();
  expect('op', ')');
  return { type: 'define', name: nameTok.value, valueNode: valueExpr };
}

// Add to evaluate():
if (node.type === 'define') {
  const value = evaluate(node.valueNode);
  userConstants[node.name] = value;
  return value;
}

// Update the 'name' case to check userConstants:
if (node.type === 'name') {
  if (node.value in userConstants) return userConstants[node.value];
  if (node.value in CONSTANTS)     return CONSTANTS[node.value];
  throw new Error(`Undefined variable: ${node.value}`);
}

// Tests:
calc.evaluate('define(tax, 0.08)');   // 0.08
calc.evaluate('100 * tax');           // 8
calc.evaluate('define(dozen, 12)');   // 12
calc.evaluate('dozen * 2');           // 24
```

### Exercise 4 — `calc.explain()`

```javascript
calc.explain = function(input) {
  const tokens = tokenize(input.trim());
  const ast    = parse(tokens);
  const steps  = [];

  function evaluateWithSteps(node) {
    if (node.type === 'number') return node.value;

    if (node.type === 'binary') {
      const left  = evaluateWithSteps(node.left);
      const right = evaluateWithSteps(node.right);
      let result;
      switch (node.op) {
        case '+': result = left + right; break;
        case '-': result = left - right; break;
        case '*': result = left * right; break;
        case '/': result = left / right; break;
        case '^': result = Math.pow(left, right); break;
      }
      // Only record non-trivial steps (not leaf-level)
      if (node.left.type !== 'number' || node.right.type !== 'number') {
        steps.push(`${left} ${node.op} ${right} = ${result}`);
      } else {
        steps.push(`${left} ${node.op} ${right} = ${result}`);
      }
      return result;
    }

    if (node.type === 'call') {
      const fn   = FUNCTIONS[node.name];
      const args = node.args.map(evaluateWithSteps);
      const result = fn(args);
      steps.push(`${node.name}(${args.join(', ')}) = ${result}`);
      return result;
    }

    return evaluate(node);
  }

  const final = evaluateWithSteps(ast);
  return steps.length <= 1
    ? `${input} = ${final}`
    : steps.join(', then ') + ` → ${final}`;
};

// Tests:
calc.explain('2 + 3 * 4');
// "3 * 4 = 12, then 2 + 12 = 14 → 14"

calc.explain('sqrt(2 + 7)');
// "2 + 7 = 9, then sqrt(9) = 3 → 3"
```

### Exercise 5 — Pratt parser sketch

```javascript
// Precedence table — higher number = tighter binding
const PRECEDENCE = { '+': 1, '-': 1, '*': 2, '/': 2, '%': 2, '^': 3 };
const RIGHT_ASSOC = new Set(['^']);   // right-associative operators

function parseInfix(minPrec) {
  let left = parseUnary();

  while (true) {
    const tok = peek();
    if (tok.type !== 'op') break;

    const prec = PRECEDENCE[tok.value];
    if (prec === undefined || prec < minPrec) break;

    consume();
    // Right-associative: recurse with same precedence; left-assoc: recurse with prec+1
    const nextPrec = RIGHT_ASSOC.has(tok.value) ? prec : prec + 1;
    const right    = parseInfix(nextPrec);

    left = { type: 'binary', op: tok.value, left, right };
  }

  return left;
}

// Replace parseExpr/parseTerm/parsePower with just:
// function parseExpr() { return parseInfix(0); }

// Tests: same results as before — the table produces identical trees
calc.evaluate('2 + 3 * 4');      // 14
calc.evaluate('2 ^ 3 ^ 2');     // 512 (right-assoc: 2^(3^2) = 2^9 = 512)
```

The Pratt parser is elegant: instead of five functions, a single `parseInfix` with a precedence parameter handles all binary operators. The recursion depth is now controlled by the number, not by the call chain.

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| Function declarations | Hoisted — available before the line where they're defined |
| Function expressions | Not hoisted — assigned to a variable like any other value |
| Arrow functions | `(x) => expr` or `(x) => { return expr; }` — shorter syntax, no own `this` |
| Default parameters | `function f(x = 0)` — used when argument is `undefined` |
| Rest parameters | `function f(...args)` — collects remaining args into an array |
| Spread operator | `fn(...array)` — unpacks an array into separate arguments |
| Closure | Inner functions share the outer scope's variables; mutations are visible to all |
| Recursion | Function that calls itself; needs a base case to avoid infinite loops |
| `throw` | Immediately stops execution and propagates up the call stack |
| `try/catch` | Catches thrown errors so the program can handle them gracefully |
| `switch` | Clean multi-branch dispatch on a single value; always include `default` |
| Tokenizer | Splits a string into typed tokens — flat list, no structure |
| Parser | Builds a tree (AST) from tokens — structure captures precedence and grouping |
| Evaluator | Walks the AST recursively, computing the result bottom-up |

---

## Building with Claude

Bad prompt:
> "How do I make a calculator in JavaScript?"

Good prompt:
> "I'm building a recursive-descent parser for arithmetic expressions in plain JavaScript. I have `tokenize(input)` working, and my `parseExpr` / `parseTerm` functions handle `+`, `-`, `*`, `/` with correct precedence via mutual recursion. I'm adding exponentiation (`^`) as a new operator. The issue: `2 ^ 3 ^ 2` should evaluate as `2 ^ (3 ^ 2) = 512` (right-associative), not `(2 ^ 3) ^ 2 = 64` (left-associative). In my current `parsePower` function I call `parsePower` recursively on the right operand — which should give right-associativity. But I'm getting 64. Here's my `parsePower` function: [code]. Can you identify the bug?"

The good prompt shows you've already built the tokenizer and lower-precedence levels, describes the exact semantic you want (right-associativity), gives a concrete test case with expected vs actual output, and includes the specific function with the bug. Claude can pinpoint the problem immediately.

---

*In Chapters 5–12, you use these foundations to build real libraries. Chapter 5's Lodash builds on the array patterns from Chapter 3. Chapter 8's template engine uses a tokenizer almost identical to Chapter 4's. Chapter 9's Promise is a state machine built with the closure patterns from Chapter 1. The tools change; the patterns stay the same.*

# Chapter 2 — Build Your Own Validator

Validation is a good beginner problem because it is simple to describe and useful immediately. A validator checks a value against rules and tells you whether it passes. While building one, you practice conditionals, comparisons, and objects, which are core JavaScript tools.

---

## The Problem

Users do not always enter good data. Someone leaves a field empty. Someone types text where a number should go. Someone enters a weak password. HTML can help a little, but many real rules have to be written in JavaScript.

Libraries like [Yup](https://github.com/jquense/yup) and [Zod](https://zod.dev/) solve this with a schema: a description of what valid data looks like, separate from the code that checks it. Your data and your rules live in different places, so you can change one without touching the other.

There are two beginner-friendly ideas here:

1. one value may need to pass several checks
2. the same check should be reusable in more than one place

That is why we build rules separately and then apply them through a schema.

A schema is just data that describes other data. That phrase sounds abstract, but the idea is simple. Instead of hardcoding every check directly inside one function, you store the rules in an object. Then your validation function reads that object and decides what checks to run. In other words: the schema says what should happen, and the validator does the work.

---

## Building It Step by Step

### v1 — One Field, Hardcoded Checks

The first version validates a single username with conditions written directly:

```javascript
// v1: conditional logic, but nothing reusable yet
function validateUsername(value) {
  if (value === '' || value === null || value === undefined) {
    return 'Username is required';
  }
  if (value.length < 3) {
    return 'Username must be at least 3 characters';
  }
  if (value.length > 20) {
    return 'Username must be no more than 20 characters';
  }
  if (!/^[a-zA-Z0-9]+$/.test(value)) {
    return 'Username may only contain letters and numbers';
  }
  return null;   // null means "valid"
}
```

This works, but it's a dead end. You can't reuse any of these checks for the email field or the password field. Every new field means writing another function from scratch.

This is a common beginner moment in programming: the code works, but it does not scale. That usually means it is time to separate the repeated pattern from the specific case.

### v2 — Extract Reusable Rule Checkers

The second observation: each condition is an independent rule that could apply to many fields. Extract them:

```javascript
// v2: rule checkers as standalone functions
const RULES = {
  required: function(value) {
    if (value === null || value === undefined || value === '') {
      return 'This field is required';
    }
    return null;   // null = pass
  },

  minLength: function(value, min) {
    if (String(value).length < min) return `Must be at least ${min} characters`;
    return null;
  },

  maxLength: function(value, max) {
    if (String(value).length > max) return `Must be no more than ${max} characters`;
    return null;
  },

  isEmail: function(value) {
    const at = String(value).indexOf('@');
    if (at < 1) return 'Must contain @';
    const after = String(value).slice(at + 1);
    if (!after.includes('.')) return 'Must have a valid domain';
    return null;
  }
};
```

Now each checker is independent, testable, and reusable. `RULES.minLength('hi', 3)` returns an error; `RULES.minLength('hello', 3)` returns null.

Notice the shape of each rule function:

- it receives a value
- it checks one condition
- it returns either an error message or `null`

That consistent shape is what makes the whole system composable.

### v3 — A Schema-Driven validate() Function

The third step: define schemas as plain objects, and write `validate()` to apply them:

```javascript
// v3: the full system — schema + validate()
const SCHEMA = {
  username: { required: true, minLength: 3, maxLength: 20, alphanumeric: true },
  email:    { required: true, isEmail: true },
  password: { required: true, minLength: 8, hasUppercase: true, hasNumber: true }
};

function validate(value, schema) {
  const errors = [];

  for (const [ruleName, param] of Object.entries(schema)) {
    const checker = RULES[ruleName];
    if (!checker) continue;             // unknown rule: skip

    const error = checker(value, param); // run the checker
    if (error !== null) {
      errors.push(error);               // collect all failures
    }
  }

  return {
    valid:  errors.length === 0,
    errors
  };
}

// Now validate any value against any schema:
validate('hi', SCHEMA.username);
// → { valid: false, errors: ['Must be at least 3 characters'] }

validate('alice', SCHEMA.username);
// → { valid: true, errors: [] }

validate('not-an-email', SCHEMA.email);
// → { valid: false, errors: ['Must contain @'] }
```

The schema is just an object. The validator is just a loop. There's no magic — only `Object.entries`, a `for...of` loop, and an array of collected errors.

---

## The Complete Program

`validator.html` — open it in your browser. Fill out the sign-up form. Watch each field validate in real time as you type. The right panel shows the raw `validate()` return value.

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Build Your Own Validator — Chapter 2</title>
<!-- styles in validator.html -->
</head>
<body>
<script>
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: miniValidator
// ═══════════════════════════════════════════════════════════════════════

const RULES = {
  required:     (v)       => (v === null || v === undefined || v === '') ? 'Required' : null,
  minLength:    (v, min)  => String(v).length < min  ? `At least ${min} chars` : null,
  maxLength:    (v, max)  => String(v).length > max  ? `Max ${max} chars`      : null,
  isNumber:     (v)       => isNaN(Number(v))         ? 'Must be a number'     : null,
  min:          (v, n)    => Number(v) < n            ? `At least ${n}`        : null,
  max:          (v, n)    => Number(v) > n            ? `At most ${n}`         : null,
  isEmail:      (v)       => { /* ... */ },
  hasUppercase: (v)       => !/[A-Z]/.test(String(v)) ? 'Needs uppercase'      : null,
  hasNumber:    (v)       => !/[0-9]/.test(String(v)) ? 'Needs a number'       : null,
  alphanumeric: (v)       => !/^[a-zA-Z0-9]+$/.test(String(v)) ? 'Letters/numbers only' : null
};

function validate(value, schema) {
  const errors      = [];
  const ruleResults = {};
  for (const [ruleName, param] of Object.entries(schema)) {
    const error = RULES[ruleName]?.(value, param) ?? null;
    if (error !== null) { errors.push(error); ruleResults[ruleName] = { pass: false, message: error }; }
    else                { ruleResults[ruleName] = { pass: true }; }
  }
  return { valid: errors.length === 0, errors, ruleResults };
}

const SCHEMA = {
  username: { required: true, minLength: 3, maxLength: 20, alphanumeric: true },
  email:    { required: true, isEmail: true },
  age:      { required: true, isNumber: true, min: 13, max: 120 },
  password: { required: true, minLength: 8, hasUppercase: true, hasNumber: true }
};
</script>
</body>
</html>
```

---

## Walkthrough

### if / else if / else — The Decision Tree

`if` is JavaScript's core branching tool. The three forms:

```javascript
// 1. Simple if — do something or nothing
if (value === '') {
  console.log('empty');
}

// 2. if/else — do one of two things
if (value.length >= 8) {
  console.log('long enough');
} else {
  console.log('too short');
}

// 3. if/else if/else — chain of decisions
if      (value.length < 3)  console.log('too short');
else if (value.length > 20) console.log('too long');
else                        console.log('good length');
```

The validator's rule checkers are all `if` statements: check a condition, return an error string or `null`. The `validate()` function itself uses no `if` at all — the loop handles the control flow, and each checker handles its own logic.

### Comparison Operators

Six comparison operators produce booleans (`true` or `false`):

```
===   strict equal        "abc" === "abc"  → true
!==   strict not-equal    "abc" !== "xyz"  → true
<     less than           3 < 5            → true
>     greater than        5 > 3            → true
<=    less than or equal  3 <= 3           → true
>=    greater than/equal  5 >= 6           → false
```

**Always use `===` and `!==`**, not `==` and `!=`. Strict equality checks both value *and* type:

```javascript
0  === false   // false — different types
0  == false    // true  — loose equality coerces
"" === false   // false
"" == false    // true
null === undefined  // false
null == undefined   // true
```

The validator uses `=== null`, `=== undefined`, and `=== ''` in `required` — three separate strict checks, not one loose `== null`.

### Logical Operators

Three logical operators combine booleans:

```javascript
// && (AND): true only if BOTH sides are true
age >= 13 && age <= 120    // valid age range
value !== '' && value.includes('@')   // non-empty email-ish value

// || (OR): true if EITHER side is true
value === null || value === undefined || value === ''  // any kind of "empty"

// ! (NOT): flips true to false
!result.valid     // true when result is invalid
!/[A-Z]/.test(v)  // true when no uppercase letter found
```

**Short-circuit evaluation**: `&&` stops as soon as it sees `false`; `||` stops as soon as it sees `true`.

```javascript
// && short-circuit: if first is false, second never runs
false && expensiveCheck()   // expensiveCheck() is never called

// || short-circuit: if first is true, second never runs
true || expensiveCheck()    // expensiveCheck() is never called

// Practical use: guard against null before accessing properties
value !== null && value.length > 0
//              ↑ only runs if value is not null
```

The `required` checker uses `||` to catch all three "empty" states in one expression.

### Truthy and Falsy

JavaScript evaluates any value in a boolean context. **Falsy** values — values that behave like `false`:

```javascript
false
0
-0
0n          // BigInt zero
""          // empty string
null
undefined
NaN
```

Everything else is **truthy**. This lets you write shorter conditions:

```javascript
if (errors.length) { ... }     // same as errors.length !== 0
if (!value)        { ... }     // same as value === '' || value === null || etc.
if (checker)       { ... }     // same as checker !== undefined && checker !== null
```

But be careful — `"0"` is truthy (non-empty string), `0` is falsy. The validator uses explicit `=== ''` rather than `!value` in `required` precisely to avoid the `"0"` trap.

### Objects — Data + Behavior

In Chapter 1 you saw objects as *return values* (the logger's public API). Here they appear in two more roles:

**Objects as configuration (schemas):**
```javascript
const SCHEMA = {
  username: { required: true, minLength: 3, maxLength: 20 }
};
```

The schema is pure data. It describes what the rules are without running any code. This separates *describing* validation from *performing* it.

**Objects as lookup tables (rule registries):**
```javascript
const RULES = {
  required:  function(value) { ... },
  minLength: function(value, min) { ... },
  isEmail:   function(value) { ... }
};

// Look up a rule by name and call it:
const checker = RULES['minLength'];   // bracket notation: key is a string variable
checker('hi', 3);                     // → 'Must be at least 3 characters'
```

Bracket notation `obj[key]` accesses a property when `key` is a variable. Dot notation `obj.key` requires you to know the property name at write time. The validator uses bracket notation in the loop because `ruleName` is only known at runtime.

### Object.entries() — Looping Over Objects

To loop over an object's key-value pairs, use `Object.entries()`:

```
Object.entries({ a: 1, b: 2, c: 3 })
  →  [ ['a', 1], ['b', 2], ['c', 3] ]
```

Combined with destructuring in a `for...of` loop:

```javascript
for (const [ruleName, param] of Object.entries(schema)) {
  // ruleName = 'required', param = true
  // ruleName = 'minLength', param = 3
  // etc.
}
```

This is the exact pattern `validate()` uses. The loop visits every rule in the schema, calls the corresponding checker, and collects errors. The schema order determines the order errors are reported.

```
validate() control flow
─────────────────────────────────────────────────────
schema = { required: true, minLength: 3, maxLength: 20 }
   │
   ▼
Object.entries(schema)
   │
   ├─ ['required', true]  → RULES.required(value, true)
   │     error? → push to errors[]
   │     null?  → ruleResults[name] = { pass: true }
   │
   ├─ ['minLength', 3]    → RULES.minLength(value, 3)
   │     error? → push to errors[]
   │     null?  → ruleResults[name] = { pass: true }
   │
   └─ ['maxLength', 20]   → RULES.maxLength(value, 20)
         ...
   │
   ▼
return { valid: errors.length === 0, errors, ruleResults }
```

### The Ternary Operator

The ternary is a one-line if/else that *returns* a value:

```javascript
// Full form: condition ? valueIfTrue : valueIfFalse
const label = errors.length === 0 ? 'valid' : 'invalid';

// Equivalent if/else:
let label;
if (errors.length === 0) {
  label = 'valid';
} else {
  label = 'invalid';
}
```

The rule checkers could use ternary instead of `if`:

```javascript
minLength: (value, min) =>
  String(value).length < min ? `Must be at least ${min} characters` : null
```

Use ternary for simple two-branch decisions that fit on one line. Use `if/else` for anything more complex.

---

## Guided Try It — Add a `confirmPassword` Rule

**The goal**: add a `matches` rule that checks whether one field's value equals another field's value. This is used for "confirm password" fields: both must be the same string.

The tricky part: most rules only look at the *current* value. But `matches` needs to compare two values. You'll need to thread the second value through the system.

**Step 1 — Write the rule checker**

The simplest form: `matches` takes the current value and the value it must match:

```javascript
RULES.matches = function(value, targetValue) {
  if (value !== targetValue) {
    return 'Values do not match';
  }
  return null;
};
```

Test it directly:
```javascript
RULES.matches('secret123', 'secret123');   // null (pass)
RULES.matches('secret123', 'Secret123');   // 'Values do not match' (fail)
```

**Step 2 — Integrate it into the schema**

The challenge: schema values are usually primitives like `3` or `true`. For `matches`, the value you pass is the *content* of another field — which changes as the user types.

One solution: make the schema value a *function* that returns the current comparison value:

```javascript
const SCHEMA = {
  password:        { required: true, minLength: 8, hasUppercase: true, hasNumber: true },
  confirmPassword: { required: true, matches: () => document.getElementById('password').value }
};
```

**Step 3 — Handle function-valued schema params in validate()**

Update `validate()` to evaluate function params at call time:

```javascript
function validate(value, schema) {
  const errors = [];
  const ruleResults = {};

  for (const [ruleName, param] of Object.entries(schema)) {
    const checker = RULES[ruleName];
    if (!checker) continue;

    // If param is a function, call it to get the actual value
    const resolvedParam = typeof param === 'function' ? param() : param;

    const error = checker(value, resolvedParam);
    if (error !== null) {
      errors.push(error);
      ruleResults[ruleName] = { pass: false, message: error };
    } else {
      ruleResults[ruleName] = { pass: true };
    }
  }

  return { valid: errors.length === 0, errors, ruleResults };
}
```

**Step 4 — Try it**

Add a confirm password field to your HTML, wire it to `validateField`, and add `confirmPassword` to your schema. Type a password, then type the same thing in the confirm field — you should see the `matches` rule pass.

**Think about it**: the `matches` rule as implemented is "impure" — it reads from the DOM instead of just operating on its arguments. What's the downside of that? How would a library like Yup handle this differently, knowing only the field names from the schema? (Hint: Yup passes the *entire form's current values* to every validator, not just the single field value.)

---

## Exercises

1. **Add a `hasSpecialChar` rule**: The rule passes if the string contains at least one character from `!@#$%^&*`. Use a regular expression: `/[!@#$%^&*]/.test(value)`. Add it to the password schema and test that `'Secret1!'` passes but `'Secret1A'` fails.

2. **Add a `custom` rule**: Allow schemas to include an arbitrary function as a rule: `custom: (v) => v.startsWith('A') ? null : 'Must start with A'`. Update `validate()` to detect when a schema value is a function with `typeof param === 'function'` and call it directly as the checker (rather than looking it up in `RULES`).

3. **Collect all rule results**: Right now `validate()` stops collecting once it has all errors. Add a `summary` property to the return value: `summary: { passed: 2, failed: 1, total: 3 }`. Compute it by counting pass/fail entries in `ruleResults`.

4. **Build `validateForm(values, schema)`**: A function that takes an entire form's values and schema in one call. `values` is an object like `{ username: 'ada', email: 'ada@example.com' }`, and `schema` is an object with the same keys. Return `{ valid: boolean, fields: { [fieldName]: result } }` — a result per field. The form is valid only when all fields are valid.

5. **Add async validation**: Some rules require a server call — checking if a username is already taken. Design a `validateAsync(value, schema)` function that returns a `Promise`. Rules that are async return a Promise of `null | string`; sync rules still return `null | string`. The function should run all rules (sync and async) in parallel with `Promise.all` and collect errors from both. (You'll need Chapter 9's Promise knowledge to fully implement this — for now, plan the interface.)

---

## Solutions

### Exercise 1 — `hasSpecialChar`

```javascript
RULES.hasSpecialChar = function(value) {
  if (value === '' || value === null || value === undefined) return null;
  if (!/[!@#$%^&*]/.test(String(value))) {
    return 'Must contain at least one special character (!@#$%^&*)';
  }
  return null;
};

// Add to password schema:
const SCHEMA = {
  password: {
    required:       true,
    minLength:      8,
    hasUppercase:   true,
    hasNumber:      true,
    hasSpecialChar: true   // ← new
  }
};

// Tests:
RULES.hasSpecialChar('Secret1!');   // null  (pass)
RULES.hasSpecialChar('Secret1A');   // 'Must contain...'  (fail)
RULES.hasSpecialChar('');           // null  (empty — required handles this)
```

### Exercise 2 — `custom` rule

```javascript
function validate(value, schema) {
  const errors      = [];
  const ruleResults = {};

  for (const [ruleName, param] of Object.entries(schema)) {

    let error;
    if (ruleName === 'custom' && typeof param === 'function') {
      // The param IS the checker
      error = param(value);
    } else {
      const checker = RULES[ruleName];
      if (!checker) continue;
      error = checker(value, param);
    }

    if (error !== null) {
      errors.push(error);
      ruleResults[ruleName] = { pass: false, message: error };
    } else {
      ruleResults[ruleName] = { pass: true };
    }
  }

  return { valid: errors.length === 0, errors, ruleResults };
}

// Usage:
const codeSchema = {
  required: true,
  custom: (v) => v.startsWith('A') ? null : 'Must start with A'
};

validate('Alpha', codeSchema);  // { valid: true, errors: [] }
validate('Beta',  codeSchema);  // { valid: false, errors: ['Must start with A'] }
```

### Exercise 3 — `summary`

```javascript
function validate(value, schema) {
  const errors      = [];
  const ruleResults = {};

  for (const [ruleName, param] of Object.entries(schema)) {
    const checker = RULES[ruleName];
    if (!checker) continue;
    const error = checker(value, param);
    if (error !== null) {
      errors.push(error);
      ruleResults[ruleName] = { pass: false, message: error };
    } else {
      ruleResults[ruleName] = { pass: true };
    }
  }

  // Count results:
  let passed = 0, failed = 0;
  for (const r of Object.values(ruleResults)) {
    if (r.pass) passed++;
    else        failed++;
  }

  return {
    valid: errors.length === 0,
    errors,
    ruleResults,
    summary: { passed, failed, total: passed + failed }   // ← new
  };
}

// Test:
validate('hi', { required: true, minLength: 3, maxLength: 20 });
// { valid: false, errors: ['Must be at least 3 characters'],
//   summary: { passed: 2, failed: 1, total: 3 } }
```

### Exercise 4 — `validateForm()`

```javascript
function validateForm(values, schema) {
  const fields = {};
  let allValid = true;

  for (const [fieldName, fieldSchema] of Object.entries(schema)) {
    const value  = values[fieldName];
    const result = validate(value, fieldSchema);
    fields[fieldName] = result;
    if (!result.valid) allValid = false;
  }

  return { valid: allValid, fields };
}

// Test:
const result = validateForm(
  { username: 'ada', email: 'not-an-email', password: 'short' },
  SCHEMA
);

result.valid;               // false
result.fields.username.valid;   // true
result.fields.email.valid;      // false
result.fields.password.valid;   // false
result.fields.email.errors;     // ['Must contain @']
```

### Exercise 5 — Async validation design

```javascript
// Design: validateAsync returns a Promise
async function validateAsync(value, schema) {
  const errors      = [];
  const ruleResults = {};
  const promises    = [];

  for (const [ruleName, param] of Object.entries(schema)) {
    const checker = RULES[ruleName];
    if (!checker) continue;

    // Run checker — may return a string, null, or Promise<string|null>
    const result = checker(value, param);

    promises.push(
      Promise.resolve(result).then(error => {
        if (error !== null) {
          errors.push(error);
          ruleResults[ruleName] = { pass: false, message: error };
        } else {
          ruleResults[ruleName] = { pass: true };
        }
      })
    );
  }

  // Wait for all checks — sync ones resolve immediately, async ones wait
  await Promise.all(promises);

  return { valid: errors.length === 0, errors, ruleResults };
}

// An async rule example (checks server):
RULES.usernameAvailable = async function(value) {
  const response = await fetch(`/api/check-username?name=${value}`);
  const data     = await response.json();
  return data.available ? null : 'Username already taken';
};

// Usage (Chapter 9 covers Promise and async/await in full):
const result = await validateAsync('alice', {
  required:          true,
  minLength:         3,
  usernameAvailable: true
});
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `if / else if / else` | Branches execute the first block whose condition is true |
| `===` strict equality | Checks value AND type; always prefer over `==` |
| Logical operators | `&&` = both, `\|\|` = either, `!` = flip; short-circuit on first result |
| Truthy / falsy | 8 falsy values; everything else is truthy |
| Ternary `? :` | Inline if/else that returns a value |
| Objects as schemas | Plain objects describe rules declaratively, separate from execution |
| Objects as registries | `RULES['name']` looks up a function by string key at runtime |
| `Object.entries()` | Returns `[[key, value], ...]` pairs for looping over objects |
| `for...of` over entries | Destructure with `const [key, val] of Object.entries(obj)` |
| Return `null` for pass | Convention: null = no error, string = error message |
| Collecting all errors | Loop through all rules even after first failure — show everything |

---

## Building with Claude

Bad prompt:
> "How do I validate a form in JavaScript?"

Good prompt:
> "I'm building a schema-driven validator in plain JavaScript (no libraries). I have a `RULES` object where each key is a rule name and each value is a function `(value, param) => null | errorString`. My `validate(value, schema)` loops over `Object.entries(schema)` and calls `RULES[ruleName](value, param)`. The problem: I need a `matches` rule that compares the current field's value against a *different* field's value — but the schema is defined statically before the user types anything. My current idea is to make the schema param a function `() => otherField.value` and call it at validate-time. Is this the right approach, or is there a cleaner way that keeps the validator pure (no DOM reads inside the library)?"

The good prompt shows your architecture, identifies the exact tension (statically-defined schema vs. dynamically-compared values), and proposes a solution before asking. Claude will evaluate your proposal and suggest alternatives.

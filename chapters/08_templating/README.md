# Chapter 8 — Build Your Own Templating Engine

Handlebars. Mustache. Pug. EJS. Every major framework ships a templating engine. But what are they? A template is a string with special placeholders. The engine replaces those placeholders with real data. Under the hood, this is a three-step compiler: tokenize the string into parts, generate JavaScript code from those parts, and wrap that code in a function. The result is a render function you can call with any data. In this chapter you build exactly that — and understand what Handlebars does every time you type `{{ name }}`.

---

## The Problem

You need to produce HTML from data — a profile card, an invoice, an email. You could build strings by hand with template literals, but that gets unreadable fast when you add conditionals and loops. A templating engine solves this: write your HTML with `{{ }}` placeholders, and the engine figures out what to substitute. Build the engine yourself and the magic becomes mechanics.

---

## The Complete Program

`templating.html` — open it to see a live engine with four example templates and a token inspector.

```javascript
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: MiniTemplate
//  Syntax:
//    {{ expression }}                      — escaped output
//    {{{ expression }}}                    — raw HTML output
//    {{#if condition }}...{{/if}}          — conditional
//    {{#unless condition }}...{{/unless}}  — inverse conditional
//    {{#each item in list }}...{{/each}}   — loop (with $index, $first, $last)
// ═══════════════════════════════════════════════════════════════════════

// Step 1: Tokenize — split the template string into a flat list of tokens
function tokenize(template) {
  const tokens = [];
  const re = /\{\{\{([\s\S]*?)\}\}\}|\{\{([\s\S]*?)\}\}/g;
  let lastIndex = 0;
  let match;

  while ((match = re.exec(template)) !== null) {
    if (match.index > lastIndex) {
      tokens.push({ type: 'text', value: template.slice(lastIndex, match.index) });
    }
    if (match[1] !== undefined) {
      tokens.push({ type: 'raw', value: match[1].trim() });
    } else {
      const inner = match[2].trim();
      if (inner.startsWith('#if ')) {
        tokens.push({ type: 'if', value: inner.slice(4).trim() });
      } else if (inner.startsWith('#unless ')) {
        tokens.push({ type: 'unless', value: inner.slice(8).trim() });
      } else if (inner.startsWith('#each ')) {
        const parts = inner.slice(6).trim().split(/\s+in\s+/);
        tokens.push({ type: 'each', item: parts[0].trim(), list: parts[1].trim() });
      } else if (inner === '/if')      { tokens.push({ type: 'endif' }); }
      else if (inner === '/unless')    { tokens.push({ type: 'endunless' }); }
      else if (inner === '/each')      { tokens.push({ type: 'endeach' }); }
      else                             { tokens.push({ type: 'expr', value: inner }); }
    }
    lastIndex = match.index + match[0].length;
  }
  if (lastIndex < template.length) {
    tokens.push({ type: 'text', value: template.slice(lastIndex) });
  }
  return tokens;
}

// Step 2: Generate — turn tokens into a JavaScript code string
function generate(tokens) {
  let code = 'let __out = "";\n';
  let eachDepth = 0;

  for (const token of tokens) {
    switch (token.type) {
      case 'text':
        code += `__out += ${JSON.stringify(token.value)};\n`;
        break;
      case 'expr':
        code += `__out += __esc(${token.value});\n`;
        break;
      case 'raw':
        code += `__out += (${token.value});\n`;
        break;
      case 'if':
        code += `if (${token.value}) {\n`;
        break;
      case 'endif':
      case 'endunless':
        code += '}\n';
        break;
      case 'unless':
        code += `if (!(${token.value})) {\n`;
        break;
      case 'each':
        eachDepth++;
        code += `const __list${eachDepth} = (${token.list}) || [];\n`;
        code += `for (let __i${eachDepth} = 0; __i${eachDepth} < __list${eachDepth}.length; __i${eachDepth}++) {\n`;
        code += `  const ${token.item} = __list${eachDepth}[__i${eachDepth}];\n`;
        code += `  const $index = __i${eachDepth};\n`;
        code += `  const $first = __i${eachDepth} === 0;\n`;
        code += `  const $last  = __i${eachDepth} === __list${eachDepth}.length - 1;\n`;
        break;
      case 'endeach':
        code += '}\n';
        eachDepth--;
        break;
    }
  }

  code += 'return __out;';
  return code;
}

// Step 3: Compile — wrap the generated code in a callable function
function compile(templateStr) {
  const tokens = tokenize(templateStr);
  const code   = generate(tokens);
  return new Function('data', '__esc', `with(data) {\n${code}\n}`);
}

// HTML escaping — prevents XSS from user-provided data
function escapeHtml(str) {
  return String(str == null ? '' : str)
    .replace(/&/g, '&amp;').replace(/</g, '&lt;')
    .replace(/>/g, '&gt;').replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
}

// Public API
function render(templateStr, data) {
  return compile(templateStr)(data, escapeHtml);
}
```

---

## Walkthrough

### The Three Phases of a Compiler

Every compiler — from GCC to Babel to your template engine — does the same thing in three steps: **tokenize, generate, execute**. Understanding this pattern lets you read any compiler, parser, or language tool.

```
Template string
      │
      ▼
   tokenize()  → flat array of tokens (text, expr, if, each, …)
      │
      ▼
   generate()  → JavaScript code as a string
      │
      ▼
  new Function()  → callable function
      │
      ▼
   fn(data)    → HTML string
```

### Step 1: Tokenize

Tokenizing means splitting the template into meaningful chunks. The template `Hello, {{ name }}!` becomes three tokens: `{type:'text', value:'Hello, '}`, `{type:'expr', value:'name'}`, `{type:'text', value:'!'}`.

The engine uses a regex with two capture groups — one for triple braces `{{{ }}}` and one for double braces `{{ }}`:

```javascript
const re = /\{\{\{([\s\S]*?)\}\}\}|\{\{([\s\S]*?)\}\}/g;
```

The `|` means "try the left pattern first; if it fails, try the right." Triple braces are tried first so `{{{` isn't mistakenly consumed as `{{` followed by a `{`. The `[\s\S]*?` is a lazy match for anything including newlines.

```javascript
while ((match = re.exec(template)) !== null) {
  // Collect literal text before this tag
  if (match.index > lastIndex) {
    tokens.push({ type: 'text', value: template.slice(lastIndex, match.index) });
  }
  // match[1] is the triple-brace capture; match[2] is double-brace
  if (match[1] !== undefined) {
    tokens.push({ type: 'raw', value: match[1].trim() });
  } else {
    const inner = match[2].trim();
    // Dispatch to the right token type based on the first word
    if (inner.startsWith('#if '))    tokens.push({ type: 'if',   value: inner.slice(4) });
    else if (inner.startsWith('#each ')) { /* parse "item in list" */ }
    // ...etc
  }
  lastIndex = match.index + match[0].length;
}
```

The key insight: after each regex match, `lastIndex` tracks how far we've consumed. Any text between `lastIndex` and the start of the next match is a literal `text` token.

### Step 2: Generate

Code generation turns the flat token array into a JavaScript function body string. The generated function builds an output string `__out` by concatenating pieces:

```javascript
// Given the template: "Hello, {{ name }}!"
// generate() produces this code string:
let __out = "";
__out += "Hello, ";
__out += __esc(name);    // name comes from `with(data)`
__out += "!";
return __out;
```

For control flow tokens, the generated code uses real JavaScript `if` and `for`:

```javascript
case 'if':
  code += `if (${token.value}) {\n`;   // {{ #if verified }} → if (verified) {
  break;
case 'endif':
  code += '}\n';                        // {{ /if }} → }
  break;
case 'each':
  eachDepth++;
  code += `const __list${eachDepth} = (${token.list}) || [];\n`;
  code += `for (let __i${eachDepth} = 0; ...) {\n`;
  code += `  const ${token.item} = __list${eachDepth}[__i${eachDepth}];\n`;
  code += `  const $index = __i${eachDepth};\n`;
  // ... $first, $last
  break;
```

`eachDepth` is a counter that lets loops nest correctly — the inner loop uses `__list2` while the outer uses `__list1`, so their variables don't collide.

### Step 3: `new Function()` — Code as Data

The generated code string is turned into a callable function using `new Function()`:

```javascript
function compile(templateStr) {
  const tokens = tokenize(templateStr);
  const code   = generate(tokens);
  return new Function('data', '__esc', `with(data) {\n${code}\n}`);
}
```

`new Function('data', '__esc', body)` creates a function equivalent to:

```javascript
function(data, __esc) {
  with(data) {
    let __out = "";
    __out += __esc(name);  // `name` resolves from `data` via `with`
    return __out;
  }
}
```

The `with(data)` block is what allows template expressions to reference data properties without prefixing them with `data.`. Write `{{ name }}` in the template, not `{{ data.name }}`. `with` is considered bad style in general code because it makes scope unpredictable — but inside a sandboxed generated function it's the right tool: predictable scope, contained risk.

### HTML Escaping — The Security Layer

Without escaping, a template rendering user-provided data is a security hole:

```javascript
const bio = '<script>alert("XSS")</script>';
render('{{ bio }}', { bio }); // → executes the script!
```

The `escapeHtml` function converts dangerous characters to their HTML entity equivalents before they're inserted into the output:

```javascript
function escapeHtml(str) {
  return String(str == null ? '' : str)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
}
```

`&` must be replaced first — otherwise replacing `<` produces `&lt;` and then a second pass would escape the `&` in `&lt;` to `&amp;lt;`.

Triple braces `{{{ }}}` bypass escaping — for cases where you trust the HTML source (your own markup, a rich text editor output you've already sanitised).

### `$index`, `$first`, `$last` — Loop Context

Inside `{{#each}}` blocks, the engine provides three special variables:

```javascript
code += `const $index = __i${eachDepth};\n`;
code += `const $first = __i${eachDepth} === 0;\n`;
code += `const $last  = __i${eachDepth} === __list${eachDepth}.length - 1;\n`;
```

These let templates make decisions based on position without needing helper functions:

```handlebars
{{#each item in items}}
  {{#unless $first}}<hr>{{/unless}}
  <div class="{{#if $last}}last{{/if}}">{{item.name}}</div>
{{/each}}
```

---

## Try It

1. **Add `{{#else}}`**: After a `{{#if}}` block, allow `{{#else}}` to start an alternative section. You'll need a new token type `'else'` and generate `} else {` code for it.

2. **Add a string helper `{{upper name}}`**: When the first word in a tag is not a keyword, treat it as a helper function call. Register helpers in a `helpers` object and look them up before falling back to property access.

3. **Add `{{@index}}` syntax**: Some engines use `@` for loop metadata instead of `$`. Support both by generating both variable names in the `each` code.

4. **Add partial support `{{> partialName}}`**: Let users register named sub-templates with `registerPartial(name, template)`. When the tokenizer sees `{{> name}}`, generate a call to render that partial with the current data.

---

## Exercises

1. **Add `{{#with obj}}`**: A block that changes the data context. Inside `{{#with user}}...{{/with}}`, `{{ name }}` should resolve to `user.name`. Generate `{ const __ctx = obj; with(__ctx) { ... } }` code for it.

2. **Add a `{{log value}}` helper**: Prints the value to the browser console during rendering. Useful for debugging templates. Implement it as a built-in helper that generates `console.log(value);` code (no output to `__out`).

3. **Precompile to a reusable function**: The current `compile()` re-tokenizes and re-generates every call. Add a template cache: if the same template string has been compiled before, return the cached function instead of recompiling. Use a `Map` keyed by template string.

4. **Detect unclosed blocks**: If the template has `{{#if}}` without `{{/if}}`, the generated code will be syntactically broken. Add a validation pass after tokenizing that counts opening vs closing tokens and throws a descriptive error like `"Unclosed #if block"` before attempting code generation.

5. **Add `{{#each item in list separator=', '}}`**: Support a named `separator` option in `#each`. The separator string is inserted between iterations (not after the last). Generate code that uses `$last` to conditionally append the separator.

---

## Solutions

### Exercise 1 — `{{#with obj}}`

In `tokenize()`:
```javascript
else if (inner.startsWith('#with ')) {
  tokens.push({ type: 'with', value: inner.slice(6).trim() });
} else if (inner === '/with') {
  tokens.push({ type: 'endwith' });
}
```

In `generate()`:
```javascript
case 'with':
  code += `{ const __withCtx${++withDepth} = (${token.value}); with(__withCtx${withDepth}) {\n`;
  break;
case 'endwith':
  code += '}}\n';
  withDepth--;
  break;
```

Usage:
```handlebars
{{#with user}}
  <p>{{name}} — {{email}}</p>
{{/with}}
```

### Exercise 2 — `{{log value}}`

Add a special case in the tokenizer before the generic `expr` fallthrough:

```javascript
else if (inner.startsWith('log ')) {
  tokens.push({ type: 'log', value: inner.slice(4).trim() });
}
```

In `generate()`:
```javascript
case 'log':
  code += `console.log('[template]', ${token.value});\n`;
  break;
```

### Exercise 3 — Template cache

```javascript
const _cache = new Map();

function compile(templateStr) {
  if (_cache.has(templateStr)) return _cache.get(templateStr);

  const tokens = tokenize(templateStr);
  const code   = generate(tokens);
  const fn     = new Function('data', '__esc', `with(data) {\n${code}\n}`);

  _cache.set(templateStr, fn);
  return fn;
}

compile.clearCache = () => _cache.clear();
compile.cacheSize  = () => _cache.size;
```

### Exercise 4 — Unclosed block detection

```javascript
function validate(tokens) {
  const stack = [];
  const openers = { 'if': '/if', 'unless': '/unless', 'each': '/each', 'with': '/with' };
  const closers = new Set(['endif', 'endunless', 'endeach', 'endwith']);

  for (const token of tokens) {
    if (openers[token.type]) {
      stack.push({ type: token.type, closes: openers[token.type] });
    } else if (closers.has(token.type)) {
      const expected = stack.pop();
      if (!expected) throw new Error(`Unexpected {{/${token.type.replace('end','')}}}`);
    }
  }

  if (stack.length > 0) {
    throw new Error(`Unclosed #${stack[stack.length - 1].type} block`);
  }
}

// In compile(), call validate(tokens) before generate(tokens).
```

### Exercise 5 — `{{#each item in list separator=', '}}`

Extended tokenizer — parse the `separator=` option:

```javascript
else if (inner.startsWith('#each ')) {
  const body = inner.slice(6);
  const sepMatch = body.match(/\s+separator=["']([^"']*)["']/);
  const separator = sepMatch ? sepMatch[1] : null;
  const core = body.replace(/\s+separator=["'][^"']*["']/, '').trim();
  const [item, list] = core.split(/\s+in\s+/);
  tokens.push({ type: 'each', item: item.trim(), list: list.trim(), separator });
}
```

Generated code — append separator after each non-last item:

```javascript
case 'each':
  eachDepth++;
  code += `const __list${eachDepth} = (${token.list}) || [];\n`;
  code += `for (let __i${eachDepth} = 0; __i${eachDepth} < __list${eachDepth}.length; __i${eachDepth}++) {\n`;
  code += `  const ${token.item} = __list${eachDepth}[__i${eachDepth}];\n`;
  code += `  const $index = __i${eachDepth};\n`;
  code += `  const $first = __i${eachDepth} === 0;\n`;
  code += `  const $last  = __i${eachDepth} === __list${eachDepth}.length - 1;\n`;
  if (token.separator) {
    code += `  const __sep${eachDepth} = $last ? '' : ${JSON.stringify(token.separator)};\n`;
  }
  break;
```

Then in the template: `{{__sep1}}` appends the separator where needed.

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| Tokenizer | Regex + `exec` in a loop; tracks `lastIndex` to collect literal text between tags |
| Token types | `text`, `expr`, `raw`, `if`, `unless`, `each`, and their closing counterparts |
| Code generation | Converts each token to a JavaScript string; builds a complete function body |
| `new Function()` | Creates a function from a string of code at runtime |
| `with(data)` | Makes data properties directly accessible as variable names inside the function |
| HTML escaping | Replace `& < > " '` with entities; `&` must go first |
| `{{{ }}}` raw output | Bypasses escaping for trusted HTML |
| `$index`, `$first`, `$last` | Generated loop context variables — no extra syntax needed |
| Compile once, render many | `compile()` returns a reusable function; calling it multiple times is cheap |
| Three-phase compiler | Tokenize → Generate → Execute — the same pattern as Babel, TypeScript, and GCC |

---

## Building with Claude

Bad prompt:
> "Make me a template engine."

Good prompt:
> "I've built a template engine in three steps: tokenize (regex-based), generate (builds a JS code string), and compile (wraps it in `new Function`). I support `{{ expr }}`, `{{#if}}`, and `{{#each item in list}}`. I want to add `{{#else}}` as an alternative block after `{{#if}}`. My current `generate()` emits `if (condition) {` for the `if` token and `}` for `endif`. Can you explain what code the `else` token should generate, and whether I need to change anything about how `endif` works when an `else` is present?"

The prompt describes the architecture precisely, shows what already works, and asks for a targeted change rather than a full implementation. Claude can focus on the one new piece rather than reconstructing context.

# JavaScript: The Kernighan Way

**Learn JavaScript by building real programs — then building the tools that built them.**

---

## The Approach

Brian Kernighan taught a generation of programmers with a simple idea: *you learn to program by writing programs*. Not by memorizing syntax. Not by reading theory. You learn by building something real, understanding why it works, and making it your own.

This book takes that idea through two phases:

**Chapters 1–4: The Language** — fundamental JavaScript through building real tools. Types and strings, control flow, arrays and loops, functions. Each chapter builds a working library from scratch.

**Chapters 5–12: Build Your Own X** — each chapter builds a real library from scratch. Lodash. jQuery. A test framework. A template engine. Promises. A router. Redux. A virtual DOM. You use these tools every day; now you'll understand exactly how they work at the machine level.

The premise of chapters 5–12: the best way to understand a tool is to build it yourself. A router is a regex matcher and a `history.pushState` wrapper. Redux is a closure with a pub/sub system. A virtual DOM is a JavaScript object comparer that calls `setAttribute`. These aren't magic — they're patterns, and once you see the pattern, you can build anything.

## How Chapters Work

Every chapter follows the same structure:

1. **Intro paragraph** — the concept and why you're building it
2. **The Problem** — what you're solving and why the naive approach fails
3. **The Complete Program** — the library in full, readable in one sitting
4. **Walkthrough** — concept by concept, with code and explanation
5. **Try It** — 4 quick extensions to attempt before the exercises
6. **Exercises** — 5 progressively harder problems that deepen the concept
7. **Solutions** — complete working code for every exercise
8. **What You Learned** — a summary table of every key concept
9. **Building with Claude** — a bad prompt and a good prompt, to show how to get real help from an AI assistant

## What You Need

Two things:

1. **A web browser** — Chrome, Firefox, Safari, or Edge. You already have one.
2. **A text editor** — [VS Code](https://code.visualstudio.com/) is free and excellent.

No Node. No package managers. No terminal. Every program is a single `.html` file you open in a browser.

## Table of Contents

### Part 1: The Language

| Chapter | Title | You Build | Concepts |
|---------|-------|-----------|----------|
| 1 | [Build Your Own Logger](chapters/01_logger/) | Structured logger with levels, timestamps, type display | Types, `typeof`, template literals, string methods, factory functions |
| 2 | [Build Your Own Validator](chapters/02_validator/) | Schema-driven form validator with real-time feedback | if/else, comparisons, logical operators, truthy/falsy, objects |
| 3 | [Build Your Own Array Methods](chapters/03_array_methods/) | map, filter, reduce, find, every, some from scratch | Arrays, for loops, callbacks, higher-order functions |
| 4 | [Build Your Own Calculator](chapters/04_calculator/) | Expression parser and evaluator (REPL) | Functions, scope, closures, recursion, try/catch, switch |

### Part 2: Build Your Own X

| Chapter | Title | You Build | Key Concepts |
|---------|-------|-----------|--------------|
| 5 | [Build Your Own Lodash](chapters/05_lodash/) | Utility library with 20+ functions | Higher-order functions, closures, function composition |
| 6 | [Build Your Own jQuery](chapters/06_jquery/) | DOM wrapper with method chaining | Wrapper pattern, `return this`, event delegation |
| 7 | [Build Your Own Test Framework](chapters/07_test_framework/) | Mocha/Jest-style test runner | Collect-then-run, try/catch as pass/fail, matchers |
| 8 | [Build Your Own Template Engine](chapters/08_templating/) | Mustache-style compiler | Tokenizer, code generation, `new Function` |
| 9 | [Build Your Own Promise](chapters/09_promise/) | Promises/A+ state machine | State machine, microtask queue, thenable resolution |
| 10 | [Build Your Own Router](chapters/10_router/) | Client-side SPA router | `history.pushState`, RegExp named groups, `URLSearchParams` |
| 11 | [Build Your Own Redux](chapters/11_redux/) | Predictable state container | Pure reducers, pub/sub, middleware currying |
| 12 | [Build Your Own Virtual DOM](chapters/12_virtual_dom/) | React-style reconciler | vnode diffing, minimal DOM patching, reconciliation |

## Code Conventions

All code uses **modern JavaScript (ES2022+)**:

- `const` and `let` — never `var`
- Arrow functions, template literals, destructuring, spread operators
- `for...of` for clean iteration
- Named capture groups in RegExp: `(?<id>[^/]+)`
- `Map`, `Set`, `WeakMap` where they're the right tool
- `queueMicrotask`, `URLSearchParams`, `URLPattern` — platform builtins over polyfills

Every program is a **single HTML file** with embedded `<style>` and `<script>` tags. Not how production apps are structured — but the fastest path from an idea to a running program, with nothing between you and the JavaScript.

## Building with Claude

Every chapter ends with a **"Building with Claude"** section showing two prompts: a bad one and a good one. The difference is specificity.

The bad prompt asks a general question. The good prompt describes the specific system you built, proposes a solution, and asks a pointed question about the hardest part. That level of specificity is what gets you past "here's an overview" and into a real technical conversation.

General rule: the more precisely you can describe what you already built and what specifically isn't working, the better the answer.

## Getting Started

```bash
git clone https://github.com/yos17/javascript-the-kernighan-way.git
cd javascript-the-kernighan-way
```

Open `chapters/01_hello_browser/` and read the README. Open the `.html` file in your browser. Start there and work forward. By chapter 12, you'll have built the core of a modern UI framework from scratch.

---

*"The only way to learn a new programming language is by writing programs in it."*
— Brian W. Kernighan & Dennis M. Ritchie, *The C Programming Language* (1978)

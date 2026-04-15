# JavaScript: The Kernighan Way

**Learn JavaScript by building small, real programs in the browser.**

---

## What This Book Is

This book is for beginners who want JavaScript to feel understandable.

A lot of JavaScript material jumps too quickly from syntax to frameworks. This book takes a slower path. Each chapter builds one useful program or tool. You read a little, run it in the browser, change it, and see what happens.

The goal is not to memorize the whole language. The goal is to build enough small working programs that JavaScript stops feeling magical.

## How to Use This Book

Each chapter follows the same idea:

1. start with a real problem
2. build the simplest version that works
3. improve it step by step
4. explain the code in plain language

Keep the browser open while you read. Edit the code. Refresh often. Break things on purpose.

## What You Need

Only two things:

1. a web browser
2. a text editor such as VS Code

No Node. No package manager. No terminal setup. Every chapter is a single `.html` file you can open directly in the browser.

## The Learning Path

### Part 1: JavaScript Fundamentals Through Small Tools

| Chapter | Title | What You Build | What You Learn |
|---------|-------|----------------|----------------|
| 1 | [Build Your Own Logger](chapters/01_logger/) | A structured logger | values, types, strings, objects, factory functions |
| 2 | [Build Your Own Validator](chapters/02_validator/) | A reusable validator | conditionals, comparisons, truthy/falsy, objects |
| 3 | [Build Your Own Array Methods](chapters/03_array_methods/) | `map`, `filter`, `reduce`, and friends | arrays, loops, callbacks, reusable patterns |
| 4 | [Build Your Own Calculator](chapters/04_calculator/) | An expression evaluator | functions, recursion, parsing, error handling |

### Part 2: Build the Tools Behind Modern JavaScript

| Chapter | Title | What You Build | Main Idea |
|---------|-------|----------------|-----------|
| 5 | [Build Your Own Lodash](chapters/05_lodash/) | A utility library | reusable collection and function helpers |
| 6 | [Build Your Own jQuery](chapters/06_jquery/) | A DOM helper library | wrappers, chaining, DOM operations |
| 7 | [Build Your Own Test Framework](chapters/07_test_framework/) | A mini test runner | assertions, registration, pass/fail flow |
| 8 | [Build Your Own Template Engine](chapters/08_templating/) | A tiny rendering engine | tokenizing, compiling, safe output |
| 9 | [Build Your Own Promise](chapters/09_promise/) | A promise implementation | async state, chaining, microtasks |
| 10 | [Build Your Own Router](chapters/10_router/) | A browser router | URLs, history, route matching |
| 11 | [Build Your Own Redux](chapters/11_redux/) | A state container | reducers, subscriptions, predictable updates |
| 12 | [Build Your Own Virtual DOM](chapters/12_virtual_dom/) | A tiny UI reconciler | rendering, diffing, patching |
| 13 | [Build Your Own Debugger](chapters/13_debugger/) | A debugging toolkit | breakpoints, execution flow, inspection |

## How the Chapters Fit Together

The first four chapters are the foundation.

- Chapter 1 teaches you how JavaScript sees values.
- Chapter 2 teaches decision-making and reusable rules.
- Chapter 3 teaches the patterns behind array work.
- Chapter 4 teaches function design and structured problem solving.

After that, the later chapters build bigger tools, but they reuse the same ideas in more serious forms.

## If You Are Completely New

Remember these four ideas:

- JavaScript is mostly about **values**, **conditions**, **loops**, and **functions**
- most big libraries are built from small patterns repeated carefully
- reading code matters, but changing code teaches faster
- if a chapter feels hard, shrink the example and test one line at a time

You do not need to understand every chapter fully on the first pass.

A useful way to read this book is:

- first, ask what problem the chapter is solving
- second, identify the input and output of the code
- third, trace one small example by hand
- fourth, only then worry about the abstraction

For example:

- in the logger chapter, the input is a value and the output is a formatted message
- in the validator chapter, the input is a value plus rules and the output is a list of errors
- in the array chapter, the input is an array plus a callback and the output is a new array or a single final value
- in the calculator chapter, the input is a string expression and the output is a number

That input-output habit will help you through the entire book.

## Getting Started

```bash
git clone https://github.com/yos17/javascript-the-kernighan-way.git
cd javascript-the-kernighan-way
```

Then:

- start with `chapters/01_logger/README.md`
- open the chapter HTML file in your browser
- change one thing at a time
- move forward in order

---

*"The only way to learn a new programming language is by writing programs in it."*
— Brian W. Kernighan & Dennis M. Ritchie, *The C Programming Language* (1978)

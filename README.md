# JavaScript: The Kernighan Way

## Learn JavaScript by Building Real Programs

> *"The only way to learn a new programming language is by writing programs in it."*
> — Brian W. Kernighan & Dennis M. Ritchie, *The C Programming Language*

---

## What This Book Is

This book teaches JavaScript the way Brian Kernighan taught C: by building real programs from the very first page. There are no contrived toy examples, no endless theory before you touch code, and no "hello world" that does nothing interesting.

By Chapter 1 you will have a working Mad Libs generator running in your browser. By Chapter 4 you will have built a password generator with a live strength meter. By Chapter 12 you will have built a multiplayer quiz game that two people can play together.

Every concept is introduced *because a program needs it*. You learn `if/else` because your Choose Your Adventure game needs to branch. You learn arrays because your quiz game needs to store questions. You learn functions because your password generator needs to do the same thing in many different ways.

This is how real programmers learn.

---

## The Philosophy

Kernighan and Ritchie's *The C Programming Language* (1978) taught a generation of programmers by showing complete, working programs from page one. Not toy snippets — real programs that did real things.

This book applies that philosophy to JavaScript. Each chapter builds one complete program. The program introduces exactly the concepts you need to build it, in the order you need them. By the end, you have written twelve programs and learned JavaScript properly.

The programs get progressively more ambitious. Chapter 1 is a Mad Libs generator. Chapter 12 is a multiplayer quiz game. Every chapter in between adds tools to your toolkit.

Open a chapter. Run the program in your browser. Read the code. Change something. Break it. Fix it. That is how you learn to code.

---

## Who This Book Is For

Anyone who is roughly 12 years old (or older) and wants to learn programming. No prior experience needed. No special software beyond a browser and a text editor.

---

## Requirements

**Browser:** Any modern browser — Chrome, Firefox, Safari, or Edge (updated in the last two years).

**Text Editor:** Any text editor works:
- [Visual Studio Code](https://code.visualstudio.com/) — free, excellent, most popular
- [Notepad++](https://notepad-plus-plus.org/) — Windows, lightweight
- Notepad or TextEdit — already on your computer

**That's it.** Open the `.html` file in a browser. Edit it in your text editor. Save, refresh. That's the whole workflow.

---

## Table of Contents

| # | Chapter | Project | Key Concepts |
|---|---------|---------|--------------|
| 1 | [Hello, Browser](chapters/01_hello_browser/) | Mad Libs Generator | variables, template literals, getElementById, innerHTML |
| 2 | [Decisions](chapters/02_decisions/) | Choose Your Adventure | if/else, comparisons, logical operators, ternary, switch |
| 3 | [Loops and Arrays](chapters/03_loops_and_arrays/) | Quiz Game | arrays, for/for...of loops, .map, .filter, .forEach |
| 4 | [Functions](chapters/04_functions/) | Password Generator | declarations, arrow functions, parameters, scope |
| 5 | Objects | Creature Card Builder | object literals, properties, methods, destructuring, JSON |
| 6 | The DOM | To-Do List | querySelector, createElement, appendChild, classList |
| 7 | Events and Forms | Calculator | addEventListener, form events, keyboard events, validation |
| 8 | Canvas | Drawing App | canvas API, 2D context, paths, colors, mouse tracking |
| 9 | Animation | Breakout Game | requestAnimationFrame, game loops, collision detection |
| 10 | Async / Fetch | Weather Dashboard | fetch, Promises, async/await, JSON APIs, error handling |
| 11 | Sound | Drum Machine | Web Audio API, AudioContext, oscillators, scheduling |
| 12 | Final Project | Multiplayer Quiz | localStorage, BroadcastChannel, putting it all together |

---

## Code Conventions

Every program in this book is a single `.html` file with embedded CSS and JavaScript. No build tools. No npm. No configuration files. Just code you can read in one sitting.

**Why single files?** Because you are learning JavaScript, not toolchains. Build tools are useful once you know what you are building. They add complexity that gets in the way of learning.

**JavaScript version:** Modern JavaScript (ES2020+) throughout — `const`, `let`, arrow functions, template literals, optional chaining, destructuring, async/await. There is no reason to learn old-style JavaScript first.

```javascript
// Variables: const by default, let when the value must change
const playerName = 'Joanna';
let score = 0;

// Arrow functions for short, single-purpose operations
const double = n => n * 2;

// Named function declarations for multi-line, reusable logic
function calculateScore(answers, timeLeft) {
  const bonus = timeLeft > 10 ? 50 : 0;
  return answers.filter(a => a.correct).length * 100 + bonus;
}

// Template literals when inserting values into strings
const message = `${playerName} scored ${score} points!`;

// Destructuring when pulling multiple values from an object
const { name, type, hp } = creature;

// Array methods over manual index loops
const passing = scores.filter(s => s >= 60);
const names = players.map(p => p.name);
```

Two-space indentation. Semicolons always. Comments explain *why*, not *what* — if the code reads clearly, it does not need a comment.

---

## Building with Claude

Each chapter ends with a "Building with Claude" section — prompts you can paste into [Claude](https://claude.ai) to extend the project. This is not cheating. It is how professional programmers work today.

The key is that you must understand every line before you use it. After finishing a chapter, you know enough to evaluate what Claude produces. If it does something unexpected, you know enough to fix it. If it writes code you do not understand, ask it to explain. Then change one thing and see what breaks.

Example prompt after Chapter 4 (Password Generator):

> *"I have a JavaScript password generator (code below). Add a feature that checks whether the generated password appears in a list of the 100 most common passwords and shows a warning if it does. Explain each addition you make."*

That is the workflow: you build the foundation, Claude helps you extend it, you understand and own the result.

---

## Getting Started

```bash
git clone https://github.com/yos17/javascript-the-kernighan-way.git
cd javascript-the-kernighan-way
open chapters/01_hello_browser/mad_libs.html   # macOS
# Windows: double-click the file in Explorer
# Linux: xdg-open chapters/01_hello_browser/mad_libs.html
```

Open `chapters/01_hello_browser/mad_libs.html` in your browser. It already works. Read the code. Then read Chapter 1.

See you there.

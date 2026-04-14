# JavaScript: The Kernighan Way

**Learn JavaScript by Building Real Programs**

---

## The Idea Behind This Book

Brian Kernighan taught a generation of programmers with a single insight: *you learn to program by writing programs*. Not by reading about syntax. Not by studying theory. By building something real, watching it break, and fixing it.

This book applies that idea to JavaScript — but with a twist. Every chapter builds a program you actually want to use: a creature card builder, a to-do list, a drawing app, a drum machine, a quiz game. The JavaScript concepts aren't introduced because a curriculum demands them. They're introduced because *your program needs them right now*.

There's a second thing this book does that most don't: it explains **why**. Not just "here's how to write a loop" but "here's why you'd reach for a loop here instead of copying code four times." Not just "here's the Fetch API" but "here's why JavaScript can't just pause and wait for network data, and what `async/await` actually does about that." Understanding why makes the concepts stick — and makes you a programmer who can invent solutions, not just recall syntax.

## Why This Approach Works

Most JavaScript books teach the language first, programs second. You learn variables, then functions, then objects, then eventually — 200 pages in — you build something. By then you've forgotten the variables.

This book inverts that. The program comes first. The language follows. Every concept earns its place by solving a real problem in the program you're building.

The result: you never learn something in isolation. You learn destructuring because your creature card builder has nested stats. You learn `async/await` because your weather dashboard would freeze without it. You learn event delegation because your to-do list has a variable number of buttons and wiring each one individually would be a mess.

Programs are also *memorable*. A working drum machine teaches Web Audio API better than any definition, because you made the sounds yourself.

## What You Need

Two things:

1. **A web browser** — Chrome, Firefox, Safari, Edge. You already have one.
2. **A text editor** — [VS Code](https://code.visualstudio.com/) is free and excellent. Notepad works too.

No installing Node. No package managers. No terminal commands. Every program is a single HTML file you double-click to run.

## How Chapters Work

Every chapter follows this structure:

1. **The Concept** — What problem are we solving? Why does this feature of JavaScript exist?
2. **The Program** — A real, working application that demonstrates the concept in context.
3. **How It Works** — A guided tour of the code: not just what each piece does, but *why* it's designed that way.
4. **Progressive Build** — Many chapters show you how to arrive at the final design, step by step, so you see the thinking behind the code — not just the result.
5. **Try It** — Quick experiments that build intuition.
6. **Guided Exercises** — Step-by-step challenges with hints, partial solutions, and questions that make you think before the answer appears. Like pair programming with a patient tutor.
7. **What You Learned** — A summary of concepts, with connections to real-world tools that use them.
8. **Building with Claude** — Prompts to extend your program with AI assistance.

## Mission Log

| Chapter | You Build | Key Concepts |
|---------|-----------|--------------|
| [1 — Hello, Browser](chapters/01_hello_browser/) | Mad Libs generator | Variables, strings, template literals, basic DOM |
| [2 — Decisions](chapters/02_decisions/) | Choose Your Adventure story engine | if/else, comparisons, logical operators, switch |
| [3 — Loops and Arrays](chapters/03_loops_and_arrays/) | Quiz game with scoring | Arrays, for loops, .map/.filter/.forEach |
| [4 — Functions](chapters/04_functions/) | Password generator with strength meter | Functions, arrow functions, parameters, scope |
| [5 — Objects](chapters/05_objects/) | Creature card builder | Object literals, properties, methods, this, destructuring |
| [6 — The DOM](chapters/06_the_dom/) | Dynamic to-do list | querySelector, createElement, event delegation |
| [7 — Events and Forms](chapters/07_events_and_forms/) | Calculator app | Event listeners, keyboard events, state machines |
| [8 — Canvas](chapters/08_canvas/) | Drawing app | Canvas API, paths, shapes, undo/redo |
| [9 — Animation and Games](chapters/09_animation_and_games/) | Breakout clone | requestAnimationFrame, collision detection, game loops |
| [10 — Async and APIs](chapters/10_async_and_apis/) | Weather dashboard | fetch, promises, async/await, JSON, error handling |
| [11 — Sound and Media](chapters/11_sound_and_media/) | Drum machine | Web Audio API, synthesis, precise scheduling |
| [12 — Final Project](chapters/12_final_project/) | Multiplayer quiz game | Architecture, state management, putting it all together |

## Code Conventions

All code uses **modern JavaScript (ES2020+)**:

- `const` and `let` — never `var`
- Arrow functions `() => {}` where they make code clearer
- Template literals `` `Hello, ${name}!` `` instead of string concatenation
- Destructuring, spread operators, and optional chaining where they simplify code
- `for...of` loops for clean iteration

Every program is a **single HTML file** with embedded `<style>` and `<script>` tags. This isn't how professional apps are built, but it's the fastest way to go from zero to a working program. By Chapter 12, you'll understand why professionals split things up — and you'll be ready to do it yourself.

## Building with Claude

Every chapter ends with **"Building with Claude"** — prompts you can paste into [Claude](https://claude.ai) to take your program further.

Here's how to get the most out of it:

- **Show your code.** Paste your current program and say "I built this. Now I want to add [feature]."
- **Ask why, not just how.** "Why does this use `const` instead of `let`?" teaches more than "Make this work."
- **Debug together.** Paste an error message and your code. Claude will explain what went wrong.
- **Go beyond the chapters.** The prompts are starting points. Follow your curiosity.

The goal isn't to have Claude write code for you — it's to have a patient partner who can answer your questions at 2 AM.

## Getting Started

```bash
git clone https://github.com/yos17/javascript-the-kernighan-way.git
cd javascript-the-kernighan-way
```

Open `chapters/01_hello_browser/` in your text editor. Read the chapter. Start coding.

---

*"The only way to learn a new programming language is by writing programs in it."*
— Brian W. Kernighan & Dennis M. Ritchie, *The C Programming Language* (1978)

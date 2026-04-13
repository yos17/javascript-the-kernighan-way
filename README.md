# JavaScript: The Kernighan Way

**Learn JavaScript by Building Real Programs**

---

## Your Mission, Should You Choose to Accept It

Brian Kernighan taught a generation of programmers with a simple idea: *you learn to program by writing programs*. Not by memorizing syntax tables. Not by reading about theory. You learn by building something real, breaking it, fixing it, and making it your own.

This book takes that idea and turns it into **12 missions**.

Each mission drops you into a scenario: a broken machine at the school fair, a dungeon that needs a map, a spy agency that needs a password generator. Your job is to write the code that saves the day. Along the way, you'll learn every major concept in JavaScript — not because a textbook says so, but because your mission demands it.

No concept is introduced "for completeness." If it's in this book, it's because a real program needed it to work.

## How Missions Work

Every chapter is a **mission** with this structure:

1. **Mission Briefing** — The story. What's broken, what's at stake, why YOU need to fix it.
2. **Starter Code** — A partially working program with `// TODO` comments showing what you need to build. Not blank. Not complete. Just enough to get you going.
3. **Challenges** — 3–5 progressive steps that build toward the complete program. Each one gets you closer to mission success.
4. **Hints** — Stuck? Every challenge has three levels of help:
   - 🤔 **Think About It** — A guiding question
   - 💡 **Hint** — A more specific nudge
   - 🔓 **Solution** — The complete code
5. **The Complete Program** — What the finished mission looks like (also the standalone `.html` file in the folder).
6. **Bonus Missions** — Extra challenges for agents who want more.
7. **What You Unlocked** — Skills and abilities you've gained, like leveling up in a game.
8. **Building with Claude** — Prompts to extend your program with AI.

## What You Need

Two things:

1. **A web browser** — Chrome, Firefox, Safari, Edge. You already have one.
2. **A text editor** — [VS Code](https://code.visualstudio.com/) is free and excellent. Notepad works too.

That's it. No installing Node. No package managers. No terminal commands. Every program is a single HTML file you double-click to run.

## Mission Log (Table of Contents)

| Mission | Codename | You Build | Skills You Unlock |
|---------|----------|-----------|-------------------|
| 1 | [Hello, Browser](chapters/01_hello_browser/) | Mad Libs generator | Variables, strings, template literals, basic DOM |
| 2 | [Decisions](chapters/02_decisions/) | Choose Your Adventure story engine | if/else, comparisons, logical operators, switch |
| 3 | [Loops and Arrays](chapters/03_loops_and_arrays/) | Quiz game with scoring | Arrays, for loops, .map/.filter/.forEach |
| 4 | [Functions](chapters/04_functions/) | Password generator with strength meter | Functions, arrow functions, parameters, scope |
| 5 | Objects | Creature card builder | Object literals, properties, methods, this |
| 6 | The DOM | Dynamic to-do list | querySelector, createElement, event delegation |
| 7 | Events and Forms | Calculator app | Event listeners, form validation, keyboard events |
| 8 | Canvas and Drawing | Drawing app | Canvas API, mouse tracking, color and stroke |
| 9 | Animation and Games | Breakout clone | requestAnimationFrame, collision detection, game loops |
| 10 | Async and APIs | Weather dashboard | fetch, promises, async/await, JSON |
| 11 | Sound and Media | Drum machine | Web Audio API, media elements, timing |
| 12 | The Final Project | Multiplayer quiz game | WebSockets, modules, putting it all together |

## Code Conventions

All code uses **modern JavaScript (ES2024+)**:

- `const` and `let` — never `var`
- Arrow functions `() => {}` where they make code clearer
- Template literals `` `Hello, ${name}!` `` instead of string concatenation
- Destructuring, spread operators, and optional chaining where they simplify code
- `for...of` loops for clean iteration
- Semantic HTML5 elements
- CSS custom properties for theming

Every program is a **single HTML file** with embedded `<style>` and `<script>` tags. This isn't how professional apps are built, but it's the fastest way to go from zero to a working program. By Mission 12, you'll understand why professionals split things up — and you'll be ready to do it yourself.

## Building with Claude

Every mission ends with a **"Building with Claude"** section — prompts you can paste into [Claude](https://claude.ai) to take your program further. Claude is an AI assistant that's great at helping you code.

Here's how to get the most out of it:

- **Show your code.** Paste your current program and say "I built this. Now I want to add [feature]."
- **Ask why, not just how.** "Why does this use `const` instead of `let`?" teaches you more than "Make this work."
- **Debug together.** Paste an error message and your code. Claude will explain what went wrong.
- **Go beyond the missions.** The prompts are starting points. Follow your curiosity.

The goal isn't to have Claude write code for you — it's to have a patient partner who can answer your questions at 2 AM when no teacher is around.

## Getting Started

```bash
git clone https://github.com/yos17/javascript-the-kernighan-way.git
cd javascript-the-kernighan-way
```

Open `chapters/01_hello_browser/starter.html` in your text editor. Read the mission briefing. Start coding. Good luck, agent.

---

*"The only way to learn a new programming language is by writing programs in it."*
— Brian W. Kernighan & Dennis M. Ritchie, *The C Programming Language* (1978)

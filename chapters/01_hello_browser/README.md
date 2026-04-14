# Chapter 1: Hello, Browser

## Why This Matters

Every program you write in this book is a conversation between JavaScript and the browser. JavaScript holds the data — strings, numbers, objects. The browser holds the visible page — text, inputs, buttons. Your code reaches into the page, reads what the user typed, transforms it, and writes results back.

This chapter introduces the three-way relationship you'll use constantly:

```
JavaScript variables  ←→  DOM elements  ←→  What the user sees
```

By the end you'll know how to store data, build strings from pieces, and connect them to a live page. Every chapter from here on uses these exact same ideas — just in more interesting combinations.

## The Program: Mad Libs Generator

`mad_libs.html` takes seven words from the user (a name, some adjectives, a noun, a verb) and weaves them into a complete adventure story. Blank fields get funny random defaults so the story always works.

Open it in your browser first. Play with it. Then read the walkthrough.

## How It Works

### Variables: Naming Your Data

```javascript
const nameDefaults = ['Zorbax the Mighty', 'Princess Noodlearms', 'Bob the Unstoppable'];
```

`const` creates a **variable** — a named container. `nameDefaults` is the name; the array of strings is the value. `const` means "this container won't be replaced with something else." Use it for almost everything.

When should you use `let` instead? When the value needs to change:

```javascript
let count = 0;
count = count + 1;  // only works with let
```

**Why not `var`?** Older JavaScript used `var`, but it has confusing scope rules. Modern JavaScript uses `const` and `let` exclusively. You'll never need `var` in this book.

**Coming up:** In Chapter 5, every creature you create is stored in a `const` object. In Chapter 10, the weather data from an API arrives as a `const` variable. The naming changes; the concept stays the same.

### Template Literals: Building Strings from Parts

```javascript
const story = `
  Once upon a Tuesday, the brave hero ${h(name)} woke up to discover that their
  bedroom was completely filled with ${h(adjective1)} soup.
`;
```

A **template literal** starts and ends with backticks (`` ` ``), not quotes. The `${}` syntax inserts any JavaScript expression:

```javascript
const name = 'Alice';
const greeting = `Hello, ${name}!`;  // → "Hello, Alice!"

const a = 5, b = 3;
const math = `${a} + ${b} = ${a + b}`;  // → "5 + 3 = 8"
```

**Why template literals instead of `+` concatenation?**

```javascript
// Old way — hard to read
const story = "The hero " + name + " woke up in " + place + ".";

// Template literal — reads like a sentence
const story = `The hero ${name} woke up in ${place}.`;
```

The backtick version also spans multiple lines naturally, which is essential when building long HTML strings (you'll do this constantly in Chapters 6–12).

**Coming up:** In Chapter 6, you'll build entire HTML elements as template literals and insert them into the page. In Chapter 10, you'll build API URLs with template literals: `` `https://wttr.in/${city}?format=j1` ``.

### Reading from the Page: getElementById

```javascript
const name = document.getElementById('name').value.trim();
```

`document` is a special object the browser provides — it represents the entire page. `getElementById('name')` finds the `<input id="name">` element. `.value` reads whatever text the user typed. `.trim()` removes leading/trailing whitespace.

This is a **chain** of three operations on one line. You can read it left to right: "the document → find element with id 'name' → get its value → trim whitespace."

**The OR fallback:**

```javascript
const name = document.getElementById('name').value.trim() || pickRandom(nameDefaults);
```

An empty string `""` is **falsy** in JavaScript — it behaves like `false` in a boolean context. The `||` operator returns its right side when the left is falsy. So: if the field is empty, use a random default.

**Coming up:** `document.getElementById` and `querySelector` are the foundation of all DOM work in Chapters 5–8. Chapter 6 builds an entire to-do list using nothing but these operations.

### Writing to the Page: innerHTML

```javascript
document.getElementById('story-output').innerHTML = story;
```

`.innerHTML` is the write counterpart to `.value`. It sets the HTML content *inside* an element. We use `innerHTML` (not `.textContent`) because `story` contains real HTML tags — `<span>`, `<em>`, `<br>` — that we want the browser to render, not display as text.

**Why wrap highlighted words in `<span>` tags?**

```javascript
const h = (word) => `<span class="highlight">${word}</span>`;
```

The `h()` function wraps each user-provided word in a span. The CSS adds the yellow highlight. This separates concerns: JavaScript decides *which* words to highlight; CSS decides *how* highlights look.

**Coming up:** In Chapter 6, you'll write far more complex HTML into the page — complete list items with checkboxes, buttons, and data attributes. The pattern is the same; the content is richer.

### Showing and Hiding Elements: classList

```javascript
outputCard.classList.add('visible');  // show the result
```

The output card starts hidden (CSS: `display: none`). When the story is ready, adding the `visible` class makes it appear (CSS: `#output-card.visible { display: block }`). The JavaScript only manages the class; the CSS manages the appearance. This clean separation is a pattern you'll use throughout the book.

`classList` methods you'll see everywhere:
- `.add('name')` — add a class
- `.remove('name')` — remove a class
- `.toggle('name')` — add if absent, remove if present
- `.contains('name')` — check if it exists

### forEach and Arrow Functions: Processing Lists

```javascript
fieldIds.forEach((id) => {
  document.getElementById(id).value = '';
});
```

`forEach` runs a function once per item in an array. The `(id) => { ... }` is an **arrow function** — a concise way to write a function inline. For each string in `fieldIds`, the function runs with `id` set to that string, clearing that input.

Arrow functions are everywhere in modern JavaScript. The same function written three ways:

```javascript
// Full function declaration
function clear(id) { document.getElementById(id).value = ''; }

// Function expression
const clear = function(id) { document.getElementById(id).value = ''; };

// Arrow function — most concise
const clear = (id) => { document.getElementById(id).value = ''; };

// Arrow with implicit return (for single expressions)
const double = (n) => n * 2;
```

**Coming up:** Arrow functions appear in every subsequent chapter. In Chapter 3, `.map()`, `.filter()`, and `.forEach()` all take arrow functions. In Chapter 6, event listeners use arrow functions. By Chapter 5, you'll write them without thinking about it.

---

## Guided Exercises

### Exercise 1: Add a New Input Field

**The Challenge:** Add a "favorite color" input. It should appear in the story and be cleared by the Clear button.

**Where to start:** Three things need to change: the HTML form, the JavaScript that reads values, and the story template. Which should you add first?

*(Start with the HTML — you can see the change immediately. Then connect it.)*

---

**Step 1: Add the HTML input.**

In `mad_libs.html`, find one of the existing `<div class="field">` blocks. Copy it and change the `id`, label text, and placeholder:

```html
<div class="field">
  <label>Favorite Color</label>
  <input type="text" id="color" placeholder="e.g. electric blue">
</div>
```

**What do you see now?** A new input on the page. It doesn't affect the story yet.

---

**Step 2: Read the value in JavaScript.**

Inside `generate()`, after the other `const` declarations:

```javascript
const color = document.getElementById('color').value.trim() || 'neon chartreuse';
```

**Think about it:** What does the `|| 'neon chartreuse'` do? When does it trigger?

*(It provides a default when the field is empty. The empty string `''` is falsy, so `||` kicks in.)*

---

**Step 3: Use the color in the story.**

Find the `const story = \`...\`` template. Add `${h(color)}` somewhere that makes sense:

```
Their cape was the exact shade of ${h(color)}, which everyone agreed was questionable.
```

---

**Step 4: Clear it with the Clear button.**

Find the `fieldIds` array. Add `'color'` to it:

```javascript
const fieldIds = ['name', 'adjective1', 'noun1', 'verb', 'place', 'adjective2', 'noun2', 'color'];
```

**Test it:** Fill in a color, generate the story, clear, generate again without filling it in. The default should appear.

---

### Exercise 2: Random Highlight Colors

**The Challenge:** Each highlighted word gets its own random background color instead of all being yellow.

**Where to start:** Look at the `h()` function. It currently produces `<span class="highlight">word</span>`. What would you change to make each call produce a different color?

---

**Step 1: Create a color array.**

```javascript
const highlightColors = ['#fef08a', '#bbf7d0', '#bfdbfe', '#fecaca', '#e9d5ff'];
```

---

**Step 2: Modify `h()` to pick randomly.**

```javascript
const h = (word) => {
  const color = highlightColors[Math.floor(Math.random() * highlightColors.length)];
  return `<span style="background:${color}; color:#333; font-weight:700; padding:1px 5px; border-radius:5px;">${word}</span>`;
};
```

**What changed?** Instead of using the `.highlight` CSS class, we use an inline `style` with a randomly selected color. `Math.random()` returns a number from 0 to <1. Multiplying by `highlightColors.length` (5) gives 0–4.99. `Math.floor()` rounds down to 0, 1, 2, 3, or 4.

**Test it:** Generate a story. Each highlighted word should be a different color. Generate again — different colors each time.

**Think about it:** If you add more colors to `highlightColors`, does the code change? No — `highlightColors.length` adapts automatically. This is why you use array length instead of hardcoding `5`.

---

### Exercise 3: Calculation History → Headline History

**The Challenge:** Add a "Recent Stories" section that shows the last three story summaries (the hero's name and place) below the current story.

**Where to start:** You need to store history between button clicks. Where does that data live? You need a variable that persists between calls to `generate()`.

*(Hint: a variable declared outside `generate()` persists across calls.)*

---

**Step 1: Create the history array.**

At the top of the script, outside all functions:

```javascript
const storyHistory = [];  // stores { name, place } for each generated story
```

---

**Step 2: Push to history in generate().**

After reading `name` and `place`, push a summary:

```javascript
storyHistory.unshift({ name, place });
if (storyHistory.length > 3) storyHistory.pop();  // keep only the last 3
```

**Why `unshift` instead of `push`?** `push` adds to the end. `unshift` adds to the beginning, so newest appears first.

---

**Step 3: Render the history.**

After setting `innerHTML` for the story, render the history list:

```javascript
const historyEl = document.getElementById('story-history');
if (historyEl) {
  historyEl.innerHTML = storyHistory
    .map(s => `<li>${s.name} in ${s.place}</li>`)
    .join('');
}
```

Add `<ul id="story-history"></ul>` to the HTML near the output area.

**What does `.map().join('')` do?** `.map()` transforms each `{ name, place }` object into a `<li>` string. `.join('')` combines the array of strings into one string.

**Coming up:** This `.map().join('')` pattern for building HTML from data is used in Chapters 6, 7, and 12 to build lists, score cards, and option menus.

---

## What You Learned

| Concept | What It Does | Coming Up In |
|---------|-------------|-------------|
| `const` / `let` | Store data with a name | Every chapter |
| Template literals | Build strings with `${}` interpolation | Ch6 (building HTML), Ch10 (API URLs) |
| `document.getElementById` | Get an HTML element by ID | Ch6, Ch7 (extensively) |
| `.value` | Read text from an `<input>` | Ch7 (calculator inputs) |
| `.innerHTML` | Write HTML into an element | Ch6 (DOM manipulation) |
| `classList` | Add/remove CSS classes | Ch6, Ch7, Ch9, Ch12 |
| `\|\|` fallback | Default value when left side is falsy | Ch10 (API fallbacks), Ch12 |
| Arrow functions | Concise inline functions | Every chapter from Ch3 on |
| `forEach` | Run a function once per array item | Ch3, Ch5, Ch6 |
| `Math.random()` | Generate a random number 0–<1 | Ch5 (stats), Ch9 (game), Ch11 (noise) |

## Building with Claude

- "I built this Mad Libs generator. Add a 'Copy Story' button that copies the plain text (no HTML tags) to the clipboard using `navigator.clipboard.writeText()`."
- "Save the user's inputs to `localStorage` so they persist when the page is refreshed. Explain what `localStorage` is and when to use it."
- "Add three more story templates. Let the user choose which template they want from a `<select>` dropdown before generating."
- "Animate the highlighted words so they pop in one at a time with a staggered delay. Use CSS `@keyframes` and `setTimeout` inside a loop."

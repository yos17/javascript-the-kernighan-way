# Chapter 1: Hello, Browser

In this chapter you'll build a Mad Libs story generator — a program that takes words you type in and weaves them into a funny story. It sounds simple, but to make it work you'll touch almost every foundational idea in JavaScript: storing values, building strings, reading from the page, and writing back to it. Everything in this book builds on what you learn here.

---

## The Program

`mad_libs.html` is a single file that runs entirely in your browser — no server, no install, nothing to configure. It shows seven input fields (a hero's name, some adjectives, a noun, a verb, a place, and so on). You fill them in, click **Generate Story!**, and a complete adventure story appears below with your words highlighted in yellow. If you leave any field blank, the program picks a random funny default so the story always works.

---

## Open It First

Before you read another word: find `mad_libs.html` in this folder, double-click it, and try it in your browser. Type in some words, click the button, read the story. Click **Clear** and try again with different words, or leave everything blank and see what the random defaults come up with.

You'll understand the code much better once you've played with what it actually does.

---

## Line-by-Line Walkthrough

### `const` and `let` — storing values

```javascript
const nameDefaults = ['Zorbax the Mighty', 'Princess Noodlearms', 'Bob the Unstoppable', 'Captain Biscuit'];
```

`const` creates a **variable** — a named box that holds a value. Here the box is called `nameDefaults` and it holds an array (a list) of funny default names. The `const` keyword means "this box will always hold the same thing — don't let me accidentally replace it." Use `const` for almost everything.

```javascript
const fieldIds = ['name', 'adjective1', 'noun1', 'verb', 'place', 'adjective2', 'noun2'];
```

Same idea: a `const` variable holding a list of strings. The name `fieldIds` is descriptive — it tells you exactly what's in the box without having to read the value.

`let` works just like `const`, with one difference: you *can* replace what's inside it later. In this program we don't actually use `let` at the top level, but `forEach` and arrow functions use short-lived variables internally. The rule of thumb: start with `const`. If JavaScript complains that you're trying to reassign it, switch to `let`.

---

### `document.getElementById()` — finding things on the page

```javascript
const name = document.getElementById('name').value.trim() || pickRandom(nameDefaults);
```

`document` is a special object the browser gives you for free — it represents the entire web page. `getElementById('name')` searches through the page and returns the element whose `id` attribute equals `'name'`. In the HTML you can find this:

```html
<input type="text" id="name" placeholder="e.g. Zorbax the Mighty" />
```

Once we have that element, `.value` reads whatever text the user typed into it. `.trim()` removes any accidental spaces at the start or end. The whole chain — `document.getElementById('name').value.trim()` — is how JavaScript reaches into the page and grabs live user input.

---

### Template literals — building strings with variables

```javascript
const story = `
  Once upon a Tuesday, the brave hero ${h(name)} woke up to discover that their bedroom
  was completely filled with ${h(adjective1)} soup.
`;
```

A **template literal** is a string that starts and ends with backtick characters (`` ` ``), not quotes. The magic part is `${}`: anything you put between the curly braces gets evaluated and inserted into the string. So `${h(name)}` calls the function `h` with the value of `name`, and whatever that returns gets dropped right into the sentence.

Template literals can also span multiple lines without any tricks, which is perfect for building a long story. Compare this to the old way of joining strings with `+` signs everywhere — template literals are much easier to read and write.

---

### `innerHTML` — writing HTML into the page

```javascript
document.getElementById('story-output').innerHTML = story;
```

Just as `.value` reads text *from* an input, `.innerHTML` writes HTML *into* any element. Here we find the paragraph with `id="story-output"` and replace everything inside it with the story string we just built. We use `innerHTML` rather than `.textContent` because the story contains real HTML tags — `<span>`, `<br>`, `<em>`, `<strong>` — and `innerHTML` tells the browser to treat them as actual tags, not just display them as raw text.

The `.highlight` spans are what make the filled-in words appear with the yellow background. The `h()` function wraps any word in one:

```javascript
const h = (word) => `<span class="highlight">${word}</span>`;
```

And the CSS does the coloring:

```css
.highlight {
  background: linear-gradient(120deg, #fef08a 0%, #fde68a 100%);
  color: #92400e;
  font-weight: 700;
  padding: 1px 5px;
  border-radius: 5px;
}
```

---

### Showing and hiding the output card

```javascript
outputCard.classList.add('visible');
```

The output card starts invisible — the CSS rule `#output-card { display: none; }` hides it. When the story is ready, we call `.classList.add('visible')`, which adds the class `visible` to the element. The CSS then overrides the `display: none`:

```css
#output-card.visible {
  display: block;
}
```

`classList` is an object attached to every element that lets you add, remove, or check CSS classes without touching the style directly. It's a cleaner way to show/hide things than setting `style.display` manually.

---

### The `||` fallback — handling empty fields

```javascript
const name = document.getElementById('name').value.trim() || pickRandom(nameDefaults);
```

If the user leaves the Name field blank, `.value.trim()` returns an empty string `""`. In JavaScript, an empty string counts as **falsy** — it behaves like `false` when you need a yes/no answer. The `||` operator ("or") returns its right side when its left side is falsy. So: if the field is empty, use a random default. If the field has something in it, use that. This one line handles both cases.

---

### `forEach` and arrow functions — doing something to a list

```javascript
fieldIds.forEach((id) => {
  document.getElementById(id).value = '';
});
```

`forEach` runs a function once for each item in an array. The `(id) => { ... }` part is an **arrow function** — a short way to write a function. For each string in `fieldIds`, the arrow function runs with `id` set to that string, and it clears that input field. This is much tidier than writing seven separate clear lines.

---

## Key Concepts

| Concept | What It Does | Example |
|---|---|---|
| `const` | Creates a variable whose value won't be replaced | `const story = \`...\`` |
| `let` | Creates a variable whose value can change | `let count = 0;` |
| String | A piece of text, written in quotes or backticks | `'hello'` or `` `hello` `` |
| Template literal | A backtick string that can embed variables with `${}` | `` `Hello, ${name}!` `` |
| `getElementById` | Finds one HTML element on the page by its `id` | `document.getElementById('name')` |
| `.value` | Reads the current text inside an `<input>` element | `document.getElementById('verb').value` |
| `innerHTML` | Sets the HTML content inside any element | `el.innerHTML = '<b>hi</b>'` |
| Function call | Running a named function to do its work | `generate()`, `clearAll()` |

---

## Try It

These are things to actually do — open the file, make a change, save it, and refresh your browser to see what happens.

**1. Change the story text (easy)**
Open `mad_libs.html` in a text editor. Find the `const story = \`...\`` inside the `generate` function. Change one sentence. Maybe make the dragon's name something other than Kevin, or change where the hero lands. Save the file, refresh the browser, click Generate. See your change.

**2. Add a new input field (medium)**
Add a new input for "favorite color." In the HTML, copy one of the existing `<div class="field">` blocks, give the new input `id="color"`, and update its label and placeholder. In the JavaScript, add a new line inside `generate()`:
```javascript
const color = document.getElementById('color').value.trim() || 'neon green';
```
Then use `${h(color)}` somewhere in the story template. Also add `'color'` to the `fieldIds` array in `clearAll()` so the Clear button wipes it too.

**3. Make highlighted words appear in a random color each time (harder)**
Right now all highlights are yellow. Change the `h()` function to pick a random background color from a list each time it's called:
```javascript
const highlightColors = ['#fef08a', '#bbf7d0', '#bfdbfe', '#fecaca', '#e9d5ff'];
const h = (word) => {
  const color = pickRandom(highlightColors);
  return `<span style="background:${color}; color:#333; font-weight:700; padding:1px 5px; border-radius:5px;">${word}</span>`;
};
```
Each highlighted word will now get its own random color.

**4. Add a "Clear" button that also resets the output message (harder)**
The existing `clearAll()` function hides the output card, but it doesn't show any confirmation. Update `clearAll()` so that after clearing the fields, it briefly shows a small message like "Fields cleared!" inside the Generate button, then restores the original text after one second:
```javascript
const btn = document.getElementById('generateBtn');
btn.textContent = '✓ Cleared!';
setTimeout(() => { btn.textContent = '⚡ Generate Story!'; }, 1000);
```
`setTimeout` runs a function after a delay in milliseconds. 1000 milliseconds = 1 second.

---

## Exercises

### Exercise 1 — Fortune Cookie Generator

Build a page with a single button labeled **"Open Fortune Cookie"**. When clicked, it picks a random fortune from an array of at least eight fortunes and displays it in a styled box. The fortune should appear with a smooth fade-in effect (use a CSS `transition` on `opacity`). Add a second button labeled **"Another Fortune"** that picks a different one. The fortunes should be funny or absurd rather than serious — think along the lines of "You will step on a LEGO at 2am" or "A small cat will judge you this week."

### Exercise 2 — Compliment Machine

Build a page with a text input (asking for someone's name) and a button labeled **"Compliment Them!"**. When clicked, it picks a random compliment template from an array, inserts the name into it using a template literal, and displays the result. Have at least six compliment templates. Examples: `` `${name} has the laugh of a thousand golden retrievers.` `` or `` `Scientists agree that ${name} is 40% cooler than the average human.` `` If the name field is empty, use "you" as the default. Style the output so the name appears highlighted in a color different from the compliment text.

### Exercise 3 — News Headline Generator

Build a page with three inputs: **Subject** (e.g. "a local squirrel"), **Verb** (e.g. "defeats"), and **Object** (e.g. "the entire government"). When the user clicks **"Generate Headline!"**, combine the three inputs into a news headline format: `BREAKING: [Subject] [Verb] [Object], Sources Say`. Display it in a large, bold, newspaper-style font. Add a **"Generate Random Headline"** button that picks random subject, verb, and object from their own arrays (at least five options each) and generates a headline without the user typing anything. Show the last five generated headlines in a list below the main display.

---

## Solutions

### Solution: Fortune Cookie Generator

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Fortune Cookie</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(135deg, #f6d365, #fda085);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    .card {
      background: #fff;
      border-radius: 20px;
      padding: 40px;
      max-width: 500px;
      width: 100%;
      text-align: center;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15);
    }

    h1 {
      font-size: 2rem;
      margin-bottom: 8px;
      color: #92400e;
    }

    p.sub {
      color: #aaa;
      margin-bottom: 32px;
      font-size: 0.9rem;
    }

    .fortune-box {
      background: #fffbeb;
      border: 2px dashed #fbbf24;
      border-radius: 14px;
      padding: 24px;
      min-height: 80px;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: 28px;
      opacity: 0;
      transition: opacity 0.5s ease;
    }

    .fortune-box.show {
      opacity: 1;
    }

    .fortune-box p {
      font-size: 1.2rem;
      color: #78350f;
      font-style: italic;
      line-height: 1.6;
    }

    .btn-row {
      display: flex;
      gap: 12px;
    }

    button {
      flex: 1;
      padding: 14px;
      border: none;
      border-radius: 12px;
      font-size: 1rem;
      font-weight: 700;
      cursor: pointer;
    }

    #openBtn {
      background: linear-gradient(135deg, #f59e0b, #ef4444);
      color: #fff;
    }

    #anotherBtn {
      background: #fef3c7;
      color: #92400e;
    }
  </style>
</head>
<body>
  <div class="card">
    <h1>🥠 Fortune Cookie</h1>
    <p class="sub">Crack one open. Wisdom awaits.</p>

    <div class="fortune-box" id="fortuneBox">
      <p id="fortuneText">Your fortune will appear here...</p>
    </div>

    <div class="btn-row">
      <button id="openBtn" onclick="openCookie()">Open Fortune Cookie</button>
      <button id="anotherBtn" onclick="openCookie()">Another Fortune</button>
    </div>
  </div>

  <script>
    const fortunes = [
      'You will step on a LEGO at 2am. It was always going to happen.',
      'A small cat will judge you harshly this week. This is normal.',
      'The answer you seek is yes — unless the question involves spiders.',
      'Great success awaits you, probably on a Tuesday.',
      'You will find something you lost years ago. Unfortunately, it will be in the last place you look.',
      'Someone nearby is thinking about sandwiches. That someone is you.',
      'Your greatest adventure begins the moment you close this browser tab.',
      'The stars say: take a nap. The stars are very smart.',
      'Beware the man who says he is not hungry. He is always hungry.',
      'You will receive a compliment today. You will immediately forget it.',
    ];

    let lastIndex = -1;

    const openCookie = () => {
      // Pick a random fortune that isn't the same as the last one shown.
      let index;
      do {
        index = Math.floor(Math.random() * fortunes.length);
      } while (index === lastIndex && fortunes.length > 1);
      lastIndex = index;

      // Fade out, swap text, fade back in.
      const box = document.getElementById('fortuneBox');
      box.classList.remove('show');

      setTimeout(() => {
        document.getElementById('fortuneText').textContent = fortunes[index];
        box.classList.add('show');
      }, 200);
    };

    // Show a fortune immediately on load.
    openCookie();
  </script>
</body>
</html>
```

---

### Solution: Compliment Machine

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Compliment Machine</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(135deg, #a8edea, #fed6e3);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    .card {
      background: #fff;
      border-radius: 20px;
      padding: 36px;
      max-width: 520px;
      width: 100%;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.12);
    }

    h1 {
      text-align: center;
      font-size: 1.9rem;
      color: #be185d;
      margin-bottom: 6px;
    }

    p.sub {
      text-align: center;
      color: #aaa;
      font-size: 0.9rem;
      margin-bottom: 28px;
    }

    label {
      display: block;
      font-size: 0.8rem;
      font-weight: 700;
      text-transform: uppercase;
      letter-spacing: 0.6px;
      color: #555;
      margin-bottom: 6px;
    }

    input {
      width: 100%;
      padding: 11px 14px;
      border: 2px solid #f9a8d4;
      border-radius: 10px;
      font-size: 1rem;
      outline: none;
      margin-bottom: 16px;
      transition: border-color 0.2s;
    }

    input:focus {
      border-color: #ec4899;
    }

    button {
      width: 100%;
      padding: 14px;
      background: linear-gradient(135deg, #ec4899, #f43f5e);
      border: none;
      border-radius: 12px;
      color: #fff;
      font-size: 1.05rem;
      font-weight: 700;
      cursor: pointer;
      margin-bottom: 24px;
    }

    .result {
      background: #fdf2f8;
      border-radius: 14px;
      padding: 20px 24px;
      font-size: 1.1rem;
      line-height: 1.7;
      color: #374151;
      display: none;
    }

    .result.show {
      display: block;
    }

    .name-highlight {
      color: #be185d;
      font-weight: 700;
      background: #fce7f3;
      padding: 1px 5px;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <div class="card">
    <h1>🌟 Compliment Machine</h1>
    <p class="sub">Enter a name. Receive a scientifically verified compliment.</p>

    <label for="personName">Someone's Name</label>
    <input type="text" id="personName" placeholder="e.g. Jordan" />

    <button onclick="compliment()">Compliment Them!</button>

    <div class="result" id="result"></div>
  </div>

  <script>
    // Each template is a function that takes a name and returns a compliment string.
    const templates = [
      (name) => `<span class="name-highlight">${name}</span> has the laugh of a thousand golden retrievers.`,
      (name) => `Scientists agree that <span class="name-highlight">${name}</span> is at least 40% cooler than the average human.`,
      (name) => `Historians will one day write long books about the outstanding snack choices of <span class="name-highlight">${name}</span>.`,
      (name) => `<span class="name-highlight">${name}</span> could explain anything and make it sound interesting, even tax forms.`,
      (name) => `The moon asked me to tell <span class="name-highlight">${name}</span> that it thinks they are doing great.`,
      (name) => `Studies show that being near <span class="name-highlight">${name}</span> increases the average happiness of any room by 37%.`,
      (name) => `<span class="name-highlight">${name}</span>'s brain is essentially a library, but cooler and it has snacks.`,
    ];

    let lastTemplateIndex = -1;

    const compliment = () => {
      const rawName = document.getElementById('personName').value.trim();
      const name = rawName || 'you';

      // Pick a template that's different from the last one used.
      let index;
      do {
        index = Math.floor(Math.random() * templates.length);
      } while (index === lastTemplateIndex && templates.length > 1);
      lastTemplateIndex = index;

      const resultEl = document.getElementById('result');
      resultEl.innerHTML = templates[index](name);
      resultEl.classList.add('show');
    };

    // Let the user press Enter to generate.
    document.getElementById('personName').addEventListener('keydown', (e) => {
      if (e.key === 'Enter') compliment();
    });
  </script>
</body>
</html>
```

---

### Solution: News Headline Generator

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>News Headline Generator</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #f1f5f9;
      min-height: 100vh;
      padding: 24px 16px;
    }

    .container {
      max-width: 680px;
      margin: 0 auto;
    }

    h1 {
      font-size: 1.9rem;
      color: #1e293b;
      text-align: center;
      margin-bottom: 4px;
    }

    p.sub {
      text-align: center;
      color: #94a3b8;
      margin-bottom: 28px;
      font-size: 0.9rem;
    }

    .card {
      background: #fff;
      border-radius: 16px;
      padding: 28px;
      box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
      margin-bottom: 20px;
    }

    .fields-row {
      display: grid;
      grid-template-columns: 1fr 1fr 1fr;
      gap: 14px;
      margin-bottom: 18px;
    }

    @media (max-width: 480px) {
      .fields-row { grid-template-columns: 1fr; }
    }

    label {
      display: block;
      font-size: 0.75rem;
      font-weight: 700;
      text-transform: uppercase;
      letter-spacing: 0.6px;
      color: #64748b;
      margin-bottom: 5px;
    }

    input {
      width: 100%;
      padding: 10px 12px;
      border: 2px solid #e2e8f0;
      border-radius: 8px;
      font-size: 0.95rem;
      outline: none;
      transition: border-color 0.2s;
    }

    input:focus {
      border-color: #3b82f6;
    }

    .btn-row {
      display: flex;
      gap: 10px;
    }

    button {
      flex: 1;
      padding: 13px;
      border: none;
      border-radius: 10px;
      font-size: 0.95rem;
      font-weight: 700;
      cursor: pointer;
    }

    #customBtn {
      background: #1e293b;
      color: #fff;
    }

    #randomBtn {
      background: #eff6ff;
      color: #1d4ed8;
    }

    .headline-display {
      background: #1e293b;
      border-radius: 14px;
      padding: 28px;
      margin-bottom: 20px;
      display: none;
    }

    .headline-display.show {
      display: block;
    }

    .breaking {
      font-size: 0.7rem;
      font-weight: 900;
      letter-spacing: 3px;
      color: #ef4444;
      margin-bottom: 10px;
      text-transform: uppercase;
    }

    .headline-text {
      font-size: 1.5rem;
      font-weight: 900;
      color: #f8fafc;
      line-height: 1.3;
    }

    .headline-text .subject { color: #93c5fd; }
    .headline-text .verb    { color: #86efac; }
    .headline-text .object  { color: #fca5a5; }

    .source {
      margin-top: 10px;
      font-size: 0.8rem;
      color: #64748b;
      font-style: italic;
    }

    .history-card {
      background: #fff;
      border-radius: 16px;
      padding: 24px 28px;
      box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
      display: none;
    }

    .history-card.show {
      display: block;
    }

    .history-card h2 {
      font-size: 0.85rem;
      text-transform: uppercase;
      letter-spacing: 1px;
      color: #94a3b8;
      margin-bottom: 14px;
    }

    .history-list {
      list-style: none;
      display: flex;
      flex-direction: column;
      gap: 8px;
    }

    .history-list li {
      font-size: 0.9rem;
      color: #475569;
      padding: 8px 12px;
      background: #f8fafc;
      border-radius: 8px;
      border-left: 3px solid #3b82f6;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>📰 Headline Generator</h1>
    <p class="sub">Breaking news. Completely made up.</p>

    <div class="card">
      <div class="fields-row">
        <div>
          <label for="subject">Subject</label>
          <input type="text" id="subject" placeholder="e.g. a local squirrel" />
        </div>
        <div>
          <label for="verb">Verb</label>
          <input type="text" id="verb" placeholder="e.g. defeats" />
        </div>
        <div>
          <label for="object">Object</label>
          <input type="text" id="object" placeholder="e.g. the government" />
        </div>
      </div>
      <div class="btn-row">
        <button id="customBtn" onclick="generateCustom()">Generate Headline</button>
        <button id="randomBtn" onclick="generateRandom()">Random Headline</button>
      </div>
    </div>

    <div class="headline-display" id="headlineDisplay">
      <div class="breaking">⚡ Breaking News</div>
      <div class="headline-text" id="headlineText"></div>
      <div class="source" id="headlineSource"></div>
    </div>

    <div class="history-card" id="historyCard">
      <h2>Recent Headlines</h2>
      <ul class="history-list" id="historyList"></ul>
    </div>
  </div>

  <script>
    const randomSubjects = [
      'a confused pigeon',
      'three neighborhood raccoons',
      'an unnamed houseplant',
      'local middle schooler',
      'a deeply tired accountant',
    ];
    const randomVerbs = [
      'outwits',
      'legally purchases',
      'politely disagrees with',
      'accidentally invents',
      'challenges to a duel',
    ];
    const randomObjects = [
      'the entire financial system',
      'a surprisingly large sandwich',
      'the concept of Mondays',
      'a world-famous scientist',
      'seventeen confused tourists',
    ];

    const sources = [
      'Sources close to the situation say they have no idea what happened.',
      'Officials are baffled. More at 11.',
      'No one was available for comment, which only raises further questions.',
      'Experts describe the situation as "unprecedented and kind of funny."',
    ];

    const pickRandom = (arr) => arr[Math.floor(Math.random() * arr.length)];

    // history stores the last five headlines as plain strings.
    const history = [];

    const showHeadline = (subject, verb, obj) => {
      const headlineHTML = `
        <span class="subject">${subject}</span>
        <span class="verb"> ${verb} </span>
        <span class="object">${obj}</span>,
        Sources Say
      `;
      document.getElementById('headlineText').innerHTML = headlineHTML;
      document.getElementById('headlineSource').textContent = pickRandom(sources);
      document.getElementById('headlineDisplay').classList.add('show');

      // Add to history (plain text version).
      const plainHeadline = `${subject} ${verb} ${obj}, Sources Say`;
      history.unshift(plainHeadline);
      if (history.length > 5) history.pop();

      // Render history list.
      const listEl = document.getElementById('historyList');
      listEl.innerHTML = '';
      history.forEach((headline) => {
        const li = document.createElement('li');
        li.textContent = headline;
        listEl.appendChild(li);
      });
      document.getElementById('historyCard').classList.add('show');
    };

    const generateCustom = () => {
      const subject = document.getElementById('subject').value.trim() || pickRandom(randomSubjects);
      const verb = document.getElementById('verb').value.trim() || pickRandom(randomVerbs);
      const obj = document.getElementById('object').value.trim() || pickRandom(randomObjects);
      showHeadline(subject, verb, obj);
    };

    const generateRandom = () => {
      showHeadline(pickRandom(randomSubjects), pickRandom(randomVerbs), pickRandom(randomObjects));
    };
  </script>
</body>
</html>
```

---

## What You Learned

| Skill | Used When | Example from This Chapter |
|---|---|---|
| Declaring variables with `const` | Storing a value that won't be reassigned | `const story = \`...\`` |
| Declaring variables with `let` | Storing a value that will change | `let count = 0` in a loop |
| Building strings with template literals | Combining text and variable values | `` `Hello, ${name}!` `` |
| Wrapping HTML in a string | Creating styled output programmatically | `\`<span class="highlight">${word}</span>\`` |
| `document.getElementById` | Finding a specific element on the page | `document.getElementById('name')` |
| Reading `.value` | Getting what a user typed in an input | `document.getElementById('verb').value` |
| Writing `.innerHTML` | Displaying HTML content in an element | `el.innerHTML = story` |
| `.classList.add` / `.remove` | Showing and hiding elements with CSS | `outputCard.classList.add('visible')` |
| The `||` fallback operator | Providing defaults for empty inputs | `value.trim() \|\| 'default text'` |
| `Math.random()` with `Math.floor()` | Picking a random item from an array | `arr[Math.floor(Math.random() * arr.length)]` |
| `forEach` with arrow functions | Running the same code for each item in a list | `fieldIds.forEach((id) => { ... })` |

---

## Building with Claude

Once you've built and understood this project, here are four specific prompts you can paste directly into Claude to push it further. Be sure to paste your actual code along with each prompt.

**Prompt 1 — Add a copy button:**
> "I built a JavaScript Mad Libs generator. Here is the full HTML file: [paste your mad_libs.html here]. I want to add a 'Copy Story' button that appears after the story generates. When clicked, it should copy the story text (without the HTML tags, just the plain words) to the clipboard using the Clipboard API. Show me exactly where to add the HTML button and the JavaScript, and explain what `navigator.clipboard.writeText()` does."

**Prompt 2 — Save inputs with localStorage:**
> "Here is my Mad Libs generator: [paste code]. I want the values the user typed into the seven input fields to be saved automatically as they type, so if they refresh the page their inputs are still there. Use `localStorage` to do this. Add a `storage` event listener that saves each field's value when it changes. When the page loads, read back any saved values. Explain what localStorage is and why you need `JSON.stringify` only if you store objects."

**Prompt 3 — Add more stories with a selector:**
> "Here is my Mad Libs generator: [paste code]. Right now there is only one story template. I want to add two more story templates and let the user choose which one they want from a `<select>` dropdown before clicking Generate. Each story should use all seven of the same input fields but in different ways. Show me how to add the `<select>` to the HTML and how to use its `.value` in JavaScript to decide which template to use."

**Prompt 4 — Animate the highlighted words:**
> "I have this Mad Libs generator: [paste code]. When the story appears, I want each highlighted word (the ones in `.highlight` spans) to pop in one at a time with a short delay between them, rather than all appearing at once. Use CSS `@keyframes` for the pop animation and JavaScript's `querySelectorAll` and `setTimeout` to stagger the timing. Walk me through how `setTimeout` works when you use it inside a loop."

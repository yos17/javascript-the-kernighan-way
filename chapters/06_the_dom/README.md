# Chapter 6: The DOM

## Why the DOM Exists

When a browser loads an HTML page, it doesn't just render pixels — it builds a **tree** of objects in memory, one object per HTML element. That tree is the Document Object Model (DOM). JavaScript can reach into that tree at any time to read, change, add, or remove elements without reloading the page.

This is the foundation of every interactive web page. When you click a button and new content appears, JavaScript is modifying the DOM tree, and the browser reflects those changes on screen immediately.

Here's what the DOM tree looks like for a simple page:

```
document
└── html
    ├── head
    │   └── title: "To-Do List"
    └── body
        ├── h1: "✅ To-Do List"
        ├── div.input-row
        │   ├── input#taskInput
        │   └── button#addBtn: "Add"
        ├── div.filters
        │   ├── button[data-filter="all"]: "All"
        │   ├── button[data-filter="active"]: "Active"
        │   └── button[data-filter="done"]: "Done"
        └── ul#list
            ├── li[data-id="1"] ← dynamically added
            └── li[data-id="2"] ← dynamically added
```

The `ul#list` starts empty. Every time you add a task, JavaScript creates a new `li` node and attaches it to the tree. The browser then renders it. That's the whole mechanism.

## The Program: Dynamic To-Do List

A fully functional task manager where you can add, complete, and remove tasks, filter them by status, and drag to reorder. Everything happens through DOM manipulation — no page reloads.

## The Complete Program

See `todo_list.html` for the full source. Key concepts are explained below.

## How It Works

### Selecting Elements

Before you can manipulate an element, you need a JavaScript reference to it:

```javascript
// By ID — fastest, returns one element or null
const input = document.getElementById('taskInput');

// By CSS selector — flexible, returns first match
const filters = document.querySelector('.filters');

// By CSS selector — all matches, returns a NodeList (array-like)
const buttons = document.querySelectorAll('.filters button');
```

**Why use `getElementById` when `querySelector` does everything?** Speed and clarity. `getElementById` is faster and signals intent ("I'm looking for this specific element"). Use `querySelector` when you need CSS selector flexibility.

### createElement and Building New Nodes

New HTML elements are created in memory before being attached to the page:

```javascript
// 1. Create the element — it doesn't appear on screen yet
const li = document.createElement('li');

// 2. Configure it
li.dataset.id = task.id;   // sets data-id attribute
li.draggable = true;

// 3. Set its contents
li.innerHTML = `
  <input type="checkbox" ${task.done ? 'checked' : ''}>
  <span>${task.text}</span>
  <button title="Remove">✕</button>
`;

// 4. Attach it — NOW it appears on screen
list.appendChild(li);
```

**Why the two-step create/attach pattern?** You can build complex elements and set all their properties before touching the live page. This is more predictable and often faster than modifying elements after they're already visible.

### Event Delegation: One Listener for Many Elements

Here's the naive approach to handling clicks on 20 to-do items:

```javascript
// Bad — wires up a new listener every time you add a task
function addTask(text) {
  const li = document.createElement('li');
  li.querySelector('button').addEventListener('click', () => {
    // remove this task
  });
}
```

This works, but it creates a new listener for every task. With 100 tasks, that's 100 listeners. And when you re-render, you have to wire them all up again.

**Event delegation** solves this with one listener on the parent, which catches clicks from all children:

```javascript
// Good — one listener handles all tasks, present and future
list.addEventListener('click', e => {
  const li = e.target.closest('li');   // find the task that was clicked
  if (!li) return;
  const id = Number(li.dataset.id);

  if (e.target.type === 'checkbox') {
    // toggle done
  } else if (e.target.tagName === 'BUTTON') {
    // remove task
  }
});
```

This works because of **event bubbling**: when you click a checkbox inside a `<li>` inside a `<ul>`, the click fires on the checkbox and then travels *up* through the tree:

```
click on checkbox
        ↓  (event fires here first)
      <li>
        ↓  (bubbles up)
      <ul>  ← our listener catches it here
        ↓
     <div>
        ↓
     <body>
```

`e.target` tells you what was originally clicked. `closest('li')` walks *up* from `e.target` to find the nearest `<li>` ancestor, giving you the task container regardless of what specific element inside it was clicked.

**Why is event delegation better?**
1. **Performance**: One listener instead of N listeners.
2. **Works for dynamic content**: The `<ul>` listener catches clicks on tasks added *after* the listener was created.
3. **Less cleanup**: When tasks are deleted, there are no lingering listeners to remove.

### classList: Managing Visual State

CSS classes are the right tool for showing/hiding or styling based on state. JavaScript should only decide *what state* to apply — CSS decides *how it looks*:

```javascript
// Wrong — JavaScript doing CSS's job
li.style.textDecoration = 'line-through';
li.style.color = '#aaa';

// Right — JavaScript applies the class, CSS defines the style
if (task.done) li.classList.add('done');
```

Then in CSS:
```css
#list li.done span { text-decoration: line-through; color: #aaa; }
```

`classList` methods: `.add('x')`, `.remove('x')`, `.toggle('x')`, `.contains('x')`.

### dataset: Connecting DOM to Data

Custom data attributes (`data-*`) bridge the gap between HTML nodes and JavaScript data:

```javascript
// Store the task's ID on the DOM element
li.dataset.id = task.id;       // → <li data-id="1715000000">

// Later, read it back when the user clicks
const id = Number(li.dataset.id);   // → 1715000000 (number)
```

This is how event delegation knows *which* task was clicked — the element carries its own ID. All `data-*` values are strings, so convert to numbers with `Number()` when needed.

### The Render Pattern

This app uses a simple pattern for keeping the DOM in sync with data:

1. Keep all state in a JavaScript array (`tasks`)
2. When state changes, call `render()`
3. `render()` wipes the old DOM and rebuilds it from current state

```javascript
function render() {
  list.innerHTML = '';  // wipe everything
  for (const task of tasks) {
    // build and append each task
  }
}
```

**Why wipe and rebuild instead of updating in place?** For a small list, "destroy and redraw" is simple and correct. The alternative — surgically updating only what changed — is faster but much more complex to write correctly. (That's what React's virtual DOM does for you automatically, but it's a lot of machinery. For a to-do list, simple wins.)

---

## Guided Exercises

### Exercise 1: Add Due Dates

**The Challenge:** Tasks should have optional due dates. Overdue tasks (past their date, not done) should be highlighted in red. Add an "Overdue" filter button.

**Where to start:** This touches three things: the form (new input), the data (task object), and the render (display + filtering). Which should you add first?

*(Always start with data. The UI and render follow from the data model.)*

---

**Step 1: Add a date input to the form.**

Put it next to the task text input:

```html
<input type="date" id="dueInput" style="padding:12px 10px; border:2px solid #ddd; border-radius:10px;">
```

**What do you have now?** A date picker the user can interact with. No data changes yet.

---

**Step 2: Read the date in `addTask()`.**

Find `addTask()` and add the date to the task object:

```javascript
function addTask() {
  const text = input.value.trim();
  if (!text) return;
  const due = document.getElementById('dueInput').value || null;  // null if empty
  tasks.push({ text, done: false, id: Date.now(), due });
  input.value = '';
  document.getElementById('dueInput').value = '';
  render();
}
```

**Think about it:** `due` will be a string like `"2025-12-31"`. Dates stored this way sort alphabetically — and alphabetical order is the same as chronological order for YYYY-MM-DD format. Handy!

---

**Step 3: Display the date and highlight overdue tasks.**

In `render()`, compute today's date, then check each task:

```javascript
function render() {
  const today = new Date().toISOString().split('T')[0]; // → "2025-01-15"

  for (const task of visible) {
    const li = document.createElement('li');
    li.dataset.id = task.id;
    li.draggable = true;
    if (task.done) li.classList.add('done');

    // Overdue = has a due date, not done, and the date is in the past
    if (!task.done && task.due && task.due < today) {
      li.style.background = '#fff0f0';
      li.style.borderLeft = '4px solid #e74c3c';
    }

    const dueText = task.due
      ? `<small style="color:#999;margin-left:auto">${task.due}</small>`
      : '';

    li.innerHTML = `
      <input type="checkbox" ${task.done ? 'checked' : ''}>
      <span>${task.text}</span>
      ${dueText}
      <button title="Remove">✕</button>
    `;
    list.appendChild(li);
  }
}
```

**What's the string comparison doing?** `task.due < today` compares `"2025-01-10"` < `"2025-01-15"` as strings. This works because ISO date strings sort correctly as text.

---

**Step 4: Add the "Overdue" filter button.**

Add the button in the HTML filters div:

```html
<button data-filter="overdue">Overdue</button>
```

Then update the filter logic in `render()`:

```javascript
const visible = tasks.filter(t => {
  if (filter === 'all') return true;
  if (filter === 'done') return t.done;
  if (filter === 'active') return !t.done;
  if (filter === 'overdue') return !t.done && t.due && t.due < today;
  return true;
});
```

The filter click handler already exists (event delegation on `.filters`) — it reads `data-filter` from whichever button was clicked, so the "Overdue" button works automatically once you add it.

---

### Exercise 2: Keyboard Navigation

**The Challenge:** Make tasks focusable. Arrow Up/Down moves focus between tasks. Space toggles done. Delete removes the focused task (moving focus to the next one).

**Where to start:** Focus and keyboard navigation require two things: making elements focusable (via `tabindex`), and responding to key events. Start with focusability.

---

**Step 1: Make each task focusable.**

In `render()`, when building each `li`, add:

```javascript
li.setAttribute('tabindex', '0');
```

**Test it:** Tab through the page. The tasks should now receive focus (usually shown with an outline). You don't need to style this yet.

---

**Step 2: Add a keydown listener on the list.**

Using delegation again — one listener on `#list` handles keys for all tasks:

```javascript
list.addEventListener('keydown', e => {
  const li = e.target.closest('li');
  if (!li) return;
  const id = Number(li.dataset.id);
  // handle keys here
});
```

**Think about it:** What should `ArrowDown` do? It should move focus to `li.nextElementSibling`. What about `ArrowUp`? The previous sibling. Let's code it:

---

**Step 3: Handle arrow keys.**

```javascript
if (e.key === 'ArrowDown') {
  e.preventDefault();  // prevent page scroll
  const next = li.nextElementSibling;
  if (next) next.focus();
} else if (e.key === 'ArrowUp') {
  e.preventDefault();
  const prev = li.previousElementSibling;
  if (prev) prev.focus();
}
```

**What about Space and Delete?** For Space (toggle), you need to find the task in the array, flip its `done`, and re-render. After re-render, the `li` is a new DOM element — so you'll need to re-focus by finding the element with `data-id`:

```javascript
} else if (e.key === ' ') {
  e.preventDefault();
  const task = tasks.find(t => t.id === id);
  if (task) { task.done = !task.done; render(); }
  // Re-focus after re-render
  list.querySelector(`[data-id="${id}"]`)?.focus();
} else if (e.key === 'Delete') {
  e.preventDefault();
  const next = li.nextElementSibling || li.previousElementSibling;
  tasks = tasks.filter(t => t.id !== id);
  render();
  // Focus the neighbor, if any
  if (next) list.querySelector(`[data-id="${next.dataset.id}"]`)?.focus();
}
```

**The complete keyboard handler adds these four cases.** Notice that `?.focus()` uses optional chaining — if `querySelector` returns `null`, the `.focus()` is skipped safely instead of throwing an error.

---

### Exercise 3: LocalStorage Persistence

**The Challenge:** Save tasks to `localStorage` so they survive a page reload. Load them on startup. Add a "Reset" button that clears storage and restores sample tasks.

**Where to start:** `localStorage` stores strings only. JavaScript objects need to be serialized with `JSON.stringify` before saving and deserialized with `JSON.parse` after loading.

---

**Step 1: Write a save function.**

```javascript
const STORAGE_KEY = 'todo-app-tasks';

function saveTasks() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
}
```

Call `saveTasks()` at the end of any function that modifies `tasks`: `addTask()`, the click handler for checkboxes, the click handler for remove buttons.

**Think about it:** Is there a better way to do this automatically, without adding `saveTasks()` to every function that changes state?

---

**Step 2: Wrap `render()` to auto-save.**

Every state change calls `render()`. So wrapping render is a clean way to auto-save:

```javascript
const _render = render;  // save the original
render = function() {
  _render();      // run the original render
  saveTasks();    // then save
};
```

Now you only need to add saving in one place.

---

**Step 3: Load tasks on startup.**

```javascript
function loadTasks() {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (saved) {
      tasks = JSON.parse(saved);
      return true;
    }
  } catch {
    // JSON.parse can throw if stored data is corrupted
    localStorage.removeItem(STORAGE_KEY);
  }
  return false;
}
```

Replace the sample tasks initialization at the bottom:

```javascript
if (!loadTasks()) {
  // No saved data — start with sample tasks
  ['Learn JavaScript objects', 'Build a to-do app', 'Celebrate 🎉'].forEach(text =>
    tasks.push({ text, done: false, id: Date.now() + Math.random() })
  );
}
render();
```

**Why the try/catch?** `localStorage` data can be corrupted (browser bug, user edited it manually). `JSON.parse` throws on invalid JSON. Always wrap it.

---

## What You Learned

| Concept | What It Does | Real-World Use |
|---------|-------------|----------------|
| `getElementById` / `querySelector` | Gets a reference to a DOM element | Every UI framework still does this under the hood |
| `createElement` | Creates a new element in memory | React's `createElement` is literally the same idea |
| `appendChild` | Attaches a node to the page | React's `render()` does this for you |
| `innerHTML` | Sets HTML content as a string | Template engines (Handlebars, EJS) work this way |
| `classList` | Toggles CSS classes to reflect state | CSS-in-JS libraries do the same thing differently |
| `dataset` | Stores custom data on DOM elements | Used to pass context from HTML to JavaScript |
| Event delegation | One parent listener handles all child events | React uses synthetic events this way for performance |
| `e.target.closest()` | Walks up the tree to find an ancestor | Used in virtually all event delegation patterns |
| `getBoundingClientRect()` | Gets size and position of an element | Used for drag-and-drop, tooltips, scroll triggers |

### Real-World Connections

- **jQuery** was hugely popular for exactly this — it wrapped `querySelector`, `classList`, `addEventListener`, and `createElement` in simpler methods. You just learned what jQuery did.
- **React** adds a virtual DOM layer on top. Instead of manually calling `innerHTML = ''` and rebuilding, you describe what the output should look like and React figures out the minimal DOM changes. The underlying `createElement` and `appendChild` still happen — React just manages them.
- **Alpine.js and Vue** add a reactive layer: instead of calling `render()` manually when data changes, you declare data as reactive and the framework calls render for you. Same concept, automatic.

## Building with Claude

- "Add a Pomodoro timer to each task — clicking a tomato icon starts a 25-minute countdown displayed on the card."
- "Make tasks editable by double-clicking the text. Swap the span for an input and save on Enter or blur."
- "Add colored category tags that you can assign to tasks, with a filter dropdown to show only one category."
- "Add an undo system — when you delete or complete a task, show a toast notification with an Undo button that reverses it."

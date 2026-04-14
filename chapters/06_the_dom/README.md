# Chapter 6: The DOM

The DOM (Document Object Model) is the browser's live representation of your HTML page. JavaScript can reach into the DOM to create new elements, move them around, change their text, and delete them — all without reloading the page. Everything you see on screen is a node in the DOM tree, and you have full control over it.

## The Program: Dynamic To-Do List

A fully functional task manager where you can add, complete, and remove tasks, filter them by status, and drag to reorder. Everything happens through DOM manipulation — no page reloads.

## The Complete Program

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>To-Do List</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { font-family: 'Segoe UI', system-ui, sans-serif; background: #f0f2f5; min-height: 100vh; display: flex; justify-content: center; padding: 30px 16px; }
  .app { width: 100%; max-width: 520px; }
  h1 { font-size: 1.6rem; margin-bottom: 16px; color: #1a1a2e; }

  .input-row { display: flex; gap: 8px; margin-bottom: 16px; }
  .input-row input { flex: 1; padding: 12px 14px; border: 2px solid #ddd; border-radius: 10px; font-size: 1rem; outline: none; transition: border-color 0.2s; }
  .input-row input:focus { border-color: #6c5ce7; }
  .input-row button { padding: 12px 20px; border: none; border-radius: 10px; background: #6c5ce7; color: #fff; font-size: 1rem; font-weight: 600; cursor: pointer; }
  .input-row button:hover { background: #5b4cdb; }

  .filters { display: flex; gap: 6px; margin-bottom: 14px; }
  .filters button { padding: 6px 14px; border: 2px solid #ddd; border-radius: 20px; background: #fff; cursor: pointer; font-size: 0.85rem; font-weight: 500; }
  .filters button.active { border-color: #6c5ce7; background: #6c5ce7; color: #fff; }

  .count { font-size: 0.85rem; color: #888; margin-bottom: 10px; }

  #list { list-style: none; }
  #list li { display: flex; align-items: center; gap: 10px; padding: 12px 14px; background: #fff; border-radius: 10px; margin-bottom: 8px; box-shadow: 0 1px 4px rgba(0,0,0,0.06); cursor: grab; transition: opacity 0.2s, transform 0.15s; }
  #list li.dragging { opacity: 0.4; transform: scale(0.97); }
  #list li.done span { text-decoration: line-through; color: #aaa; }

  #list li input[type="checkbox"] { width: 20px; height: 20px; accent-color: #6c5ce7; cursor: pointer; flex-shrink: 0; }
  #list li span { flex: 1; font-size: 0.95rem; word-break: break-word; }
  #list li button { background: none; border: none; color: #ccc; font-size: 1.2rem; cursor: pointer; padding: 0 4px; flex-shrink: 0; }
  #list li button:hover { color: #e74c3c; }

  .empty { text-align: center; padding: 30px; color: #bbb; font-size: 0.95rem; }
</style>
</head>
<body>

<div class="app">
  <h1>✅ To-Do List</h1>

  <div class="input-row">
    <input type="text" id="taskInput" placeholder="What needs to be done?">
    <button id="addBtn">Add</button>
  </div>

  <div class="filters">
    <button class="active" data-filter="all">All</button>
    <button data-filter="active">Active</button>
    <button data-filter="done">Done</button>
  </div>

  <div class="count" id="count"></div>
  <ul id="list"></ul>
</div>

<script>
let tasks = [];
let filter = 'all';
let dragIdx = null;

const input = document.getElementById('taskInput');
const list = document.getElementById('list');
const countEl = document.getElementById('count');

// Add task
document.getElementById('addBtn').addEventListener('click', addTask);
input.addEventListener('keydown', e => { if (e.key === 'Enter') addTask(); });

function addTask() {
  const text = input.value.trim();
  if (!text) return;
  tasks.push({ text, done: false, id: Date.now() });
  input.value = '';
  render();
}

// Filter buttons — event delegation on parent
document.querySelector('.filters').addEventListener('click', e => {
  if (e.target.tagName !== 'BUTTON') return;
  filter = e.target.dataset.filter;
  document.querySelectorAll('.filters button').forEach(b => b.classList.remove('active'));
  e.target.classList.add('active');
  render();
});

// List clicks — event delegation
list.addEventListener('click', e => {
  const li = e.target.closest('li');
  if (!li) return;
  const id = Number(li.dataset.id);

  if (e.target.type === 'checkbox') {
    const task = tasks.find(t => t.id === id);
    task.done = e.target.checked;
    render();
  } else if (e.target.tagName === 'BUTTON') {
    tasks = tasks.filter(t => t.id !== id);
    render();
  }
});

// Drag-and-drop reordering
list.addEventListener('dragstart', e => {
  const li = e.target.closest('li');
  if (!li) return;
  dragIdx = [...list.children].indexOf(li);
  li.classList.add('dragging');
});

list.addEventListener('dragover', e => {
  e.preventDefault();
  const li = e.target.closest('li');
  if (!li) return;
  const rect = li.getBoundingClientRect();
  const mid = rect.top + rect.height / 2;
  const dropIdx = [...list.children].indexOf(li);
  if (e.clientY < mid && dropIdx < dragIdx) {
    list.insertBefore(list.children[dragIdx], li);
    dragIdx = dropIdx;
  } else if (e.clientY > mid && dropIdx > dragIdx) {
    list.insertBefore(list.children[dragIdx], li.nextSibling);
    dragIdx = dropIdx;
  }
});

list.addEventListener('dragend', e => {
  e.target.closest('li')?.classList.remove('dragging');
  // Sync task order from DOM
  const ids = [...list.children].map(li => Number(li.dataset.id));
  tasks = ids.map(id => tasks.find(t => t.id === id));
  dragIdx = null;
});

function render() {
  const visible = tasks.filter(t =>
    filter === 'all' ? true : filter === 'done' ? t.done : !t.done
  );

  const active = tasks.filter(t => !t.done).length;
  countEl.textContent = `${active} task${active !== 1 ? 's' : ''} remaining`;

  list.innerHTML = '';
  if (visible.length === 0) {
    list.innerHTML = `<div class="empty">${tasks.length === 0 ? 'Add your first task above!' : 'No tasks match this filter.'}</div>`;
    return;
  }

  for (const task of visible) {
    const li = document.createElement('li');
    li.dataset.id = task.id;
    li.draggable = true;
    if (task.done) li.classList.add('done');

    li.innerHTML = `
      <input type="checkbox" ${task.done ? 'checked' : ''}>
      <span>${task.text}</span>
      <button title="Remove">✕</button>
    `;
    list.appendChild(li);
  }
}

// Start with sample tasks
['Learn JavaScript objects', 'Build a to-do app', 'Celebrate 🎉'].forEach(text =>
  tasks.push({ text, done: false, id: Date.now() + Math.random() })
);
render();
</script>
</body>
</html>
```

## How It Works

### Selecting Elements

The program grabs references to key elements right away using `getElementById`:

```js
const input = document.getElementById('taskInput');
const list = document.getElementById('list');
const countEl = document.getElementById('count');
```

You can also use `querySelector` with any CSS selector — the filter buttons use `document.querySelector('.filters')` to find the container. `querySelectorAll` returns all matches as a NodeList you can loop over.

### createElement and innerHTML

New tasks are built with `createElement`:

```js
const li = document.createElement('li');
li.dataset.id = task.id;
li.draggable = true;
```

The element exists in memory until you attach it to the page. We set its inner HTML with a template literal containing the checkbox, text span, and remove button, then append it to the list. For clearing the list before re-render, `list.innerHTML = ''` wipes all children in one shot.

### appendChild and the DOM Tree

```js
list.appendChild(li);
```

This attaches the `<li>` as the last child of `<ul id="list">`. Each call adds one node to the tree. The browser repaints after the script finishes, so building many elements in a loop is fine — they all appear at once.

### classList

CSS classes control visual state. When a task is marked done:

```js
if (task.done) li.classList.add('done');
```

The CSS rule `#list li.done span { text-decoration: line-through; color: #aaa; }` handles the visual change. You can also use `classList.remove()`, `classList.toggle()`, and `classList.contains()`. This is cleaner than manipulating the full `className` string.

### dataset Attributes

Custom data lives on elements via `data-*` attributes:

```js
li.dataset.id = task.id;    // sets data-id="12345"
// later:
const id = Number(li.dataset.id);  // reads it back
```

The filter buttons use `data-filter="all"`, `data-filter="active"`, and `data-filter="done"`. When clicked, `e.target.dataset.filter` tells us which filter was chosen. This avoids messy if/else chains checking button text.

### Event Delegation

Instead of attaching a click handler to every checkbox and every remove button, one listener on the parent `<ul>` handles everything:

```js
list.addEventListener('click', e => {
  const li = e.target.closest('li');
  if (!li) return;
  const id = Number(li.dataset.id);

  if (e.target.type === 'checkbox') {
    // toggle done
  } else if (e.target.tagName === 'BUTTON') {
    // remove task
  }
});
```

This works because of **event bubbling** — when you click a checkbox inside a `<li>` inside a `<ul>`, the click event fires on the checkbox first, then bubbles up through `<li>` to `<ul>`. The `e.target` tells you what was actually clicked, and `closest('li')` walks up the tree to find the parent list item.

The filter buttons use the same pattern — one listener on `.filters` instead of three separate listeners.

### Drag-and-Drop Reordering

HTML5 drag-and-drop uses four events. First, make elements draggable:

```js
li.draggable = true;
```

Then handle the lifecycle. `dragstart` fires when the user begins dragging — we record which index they grabbed. `dragover` fires continuously as they drag over other items — we must call `e.preventDefault()` to allow dropping, and we use the mouse position relative to each item's midpoint to decide whether to insert above or below. `dragend` fires when they release — we sync the new DOM order back to our `tasks` array.

```js
list.addEventListener('dragover', e => {
  e.preventDefault();
  const li = e.target.closest('li');
  if (!li) return;
  const rect = li.getBoundingClientRect();
  const mid = rect.top + rect.height / 2;
  // insert above or below based on mouse position
});
```

## Try It

1. **Edit in place** — Double-click a task's text to replace the `<span>` with an `<input>`. When the user presses Enter or clicks away (blur), save the new text back.

2. **Priority colors** — Add a dropdown next to the input that lets you pick Low, Medium, or High priority. Store it in the task object and use a colored left border on each `<li>`.

3. **Clear completed** — Add a "Clear Done" button that removes all completed tasks at once with a confirmation.

4. **Task count animation** — When the count changes, briefly scale it up with a CSS transition to draw the eye.

## Exercises

**Exercise 1: Due Dates.** Add a date input next to the task input. Store the due date in each task object. Display it as small gray text on the right side of each task. Tasks past their due date should get a red background. The filter bar should get a fourth button: "Overdue".

**Exercise 2: Keyboard Navigation.** Make tasks focusable with `tabindex="0"`. Arrow Up/Down should move focus between tasks. Space should toggle done/undone. Delete should remove the focused task. When a task is removed, focus should move to the next one (or the previous if it was the last).

**Exercise 3: LocalStorage Persistence.** Save the entire tasks array to `localStorage` every time it changes. On page load, check for saved data and restore it. Add a "Reset" button that clears localStorage and reloads with the default sample tasks. Handle the case where saved data is corrupted (wrap JSON.parse in try/catch).

## Solutions

### Solution 1: Due Dates

Add this input to the `.input-row` in the HTML:

```html
<input type="date" id="dueInput" style="padding:12px 10px; border:2px solid #ddd; border-radius:10px;">
```

Modify the JavaScript:

```js
function addTask() {
  const text = input.value.trim();
  if (!text) return;
  const due = document.getElementById('dueInput').value || null;
  tasks.push({ text, done: false, id: Date.now(), due });
  input.value = '';
  document.getElementById('dueInput').value = '';
  render();
}

function render() {
  const today = new Date().toISOString().split('T')[0];
  const visible = tasks.filter(t => {
    if (filter === 'all') return true;
    if (filter === 'done') return t.done;
    if (filter === 'active') return !t.done;
    if (filter === 'overdue') return !t.done && t.due && t.due < today;
    return true;
  });

  // ... existing count logic ...

  for (const task of visible) {
    const li = document.createElement('li');
    li.dataset.id = task.id;
    li.draggable = true;
    if (task.done) li.classList.add('done');
    if (!task.done && task.due && task.due < today) {
      li.style.background = '#fff0f0';
      li.style.borderLeft = '4px solid #e74c3c';
    }

    const dueText = task.due ? `<small style="color:#999;margin-left:auto">${task.due}</small>` : '';
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

Add a fourth filter button in the HTML:

```html
<button data-filter="overdue">Overdue</button>
```

### Solution 2: Keyboard Navigation

Replace the list event delegation and add keyboard handling:

```js
list.addEventListener('keydown', e => {
  const li = e.target.closest('li');
  if (!li) return;
  const id = Number(li.dataset.id);

  if (e.key === ' ') {
    e.preventDefault();
    const task = tasks.find(t => t.id === id);
    if (task) { task.done = !task.done; render(); }
    // Re-focus the same task after re-render
    const refocused = list.querySelector(`[data-id="${id}"]`);
    if (refocused) refocused.focus();
  } else if (e.key === 'Delete' || e.key === 'Backspace') {
    e.preventDefault();
    const next = li.nextElementSibling || li.previousElementSibling;
    tasks = tasks.filter(t => t.id !== id);
    render();
    if (next) {
      const refocused = list.querySelector(`[data-id="${next.dataset.id}"]`);
      if (refocused) refocused.focus();
    }
  } else if (e.key === 'ArrowDown') {
    e.preventDefault();
    const next = li.nextElementSibling;
    if (next) next.focus();
  } else if (e.key === 'ArrowUp') {
    e.preventDefault();
    const prev = li.previousElementSibling;
    if (prev) prev.focus();
  }
});

// In the render function, add tabindex to each li:
// li.setAttribute('tabindex', '0');
```

### Solution 3: LocalStorage Persistence

```js
const STORAGE_KEY = 'todo-app-tasks';

function saveTasks() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
}

function loadTasks() {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (saved) {
      tasks = JSON.parse(saved);
      return true;
    }
  } catch {
    localStorage.removeItem(STORAGE_KEY);
  }
  return false;
}

// Wrap render to auto-save
const baseRender = render;
render = function() {
  baseRender();
  saveTasks();
};

// Add reset button in the HTML after the filters div:
// <button id="resetBtn" style="...">Reset</button>

document.getElementById('resetBtn')?.addEventListener('click', () => {
  localStorage.removeItem(STORAGE_KEY);
  tasks = [];
  ['Learn JavaScript objects', 'Build a to-do app', 'Celebrate 🎉'].forEach(text =>
    tasks.push({ text, done: false, id: Date.now() + Math.random() })
  );
  render();
});

// At startup, try loading saved data
if (!loadTasks()) {
  ['Learn JavaScript objects', 'Build a to-do app', 'Celebrate 🎉'].forEach(text =>
    tasks.push({ text, done: false, id: Date.now() + Math.random() })
  );
}
render();
```

## What You Learned

| Concept | What It Does |
|---------|-------------|
| `document.getElementById` | Finds a single element by its `id` attribute |
| `document.querySelector` | Finds the first element matching any CSS selector |
| `document.querySelectorAll` | Returns all matching elements as a NodeList |
| `createElement` | Creates a new DOM node in memory (not yet on screen) |
| `appendChild` | Attaches a node as the last child of a parent element |
| `innerHTML` | Gets or sets the HTML content of an element as a string |
| `classList` | Adds, removes, or toggles CSS classes on an element |
| `dataset` | Reads and writes `data-*` attributes as a JavaScript object |
| Event delegation | One listener on a parent handles events for all children |
| `e.target.closest()` | Walks up the DOM tree to find the nearest matching ancestor |
| `draggable` | HTML attribute that makes an element draggable |
| `getBoundingClientRect()` | Returns an element's size and position on screen |

## Building with Claude

- "Add a Pomodoro timer to each task — clicking a tomato icon starts a 25-minute countdown displayed on the card."
- "Make tasks editable by double-clicking the text. Swap the span for an input and save on Enter or blur."
- "Add colored category tags that you can assign to tasks, with a filter dropdown to show only one category."
- "Add an undo system — when you delete or complete a task, show a toast notification with an Undo button that reverses it."

# Chapter 6: The DOM

The DOM (Document Object Model) is the browser's live representation of your HTML. JavaScript can reach into the DOM, create new elements, move them around, change their text, and delete them — all without reloading the page.

We'll build a **Dynamic To-Do List** with add/remove/complete tasks, filters, and drag-and-drop reordering — all driven by DOM manipulation.

---

## The Program

Open **`todo_list.html`** in your browser to run the full program.

---

## Key Concepts Walkthrough

### Selecting Elements

`document.querySelector` finds the first element matching a CSS selector:

```js
const input   = document.querySelector('#taskInput');   // by ID
const firstLi = document.querySelector('li');           // first <li>
const active  = document.querySelector('.filter-btn.active'); // by class
```

`document.querySelectorAll` returns all matches as a NodeList:

```js
document.querySelectorAll('.filter-btn').forEach(btn => {
  btn.classList.remove('active');
});
```

### Creating and Adding Elements

```js
const li = document.createElement('li');     // creates <li> in memory
li.textContent = 'Buy milk';                 // sets text
li.className   = 'task-item';               // sets class

const list = document.querySelector('#taskList');
list.appendChild(li);                        // adds to the DOM
```

Elements exist in memory until you append them — only then do they appear on screen.

### Removing Elements

```js
li.remove();                  // self-remove (modern)
// or:
list.removeChild(li);         // remove a child from a parent
```

In our program we filter the array and re-render, which is often cleaner than hunting for specific nodes.

### classList

Add, remove, and toggle CSS classes without touching the full `className` string:

```js
li.classList.add('done');
li.classList.remove('done');
li.classList.toggle('done');        // adds if absent, removes if present
li.classList.contains('done');      // → true / false
```

### dataset Attributes

Store custom data on elements with `data-*` attributes, then read them in JS:

```html
<li data-id="42" data-priority="high">Task</li>
```

```js
const id       = li.dataset.id;        // '42' (always a string)
const priority = li.dataset.priority;  // 'high'
```

This avoids creating parallel arrays to track which DOM node corresponds to which data object.

### Event Delegation

Instead of adding a listener to every `<li>`, attach one listener to the parent `<ul>`:

```js
taskList.addEventListener('click', e => {
  const li = e.target.closest('.task-item');  // DOM traversal
  if (!li) return;

  const id = parseInt(li.dataset.id);
  // now handle the click
});
```

**Why**: Adding 100 listeners for 100 tasks is wasteful. One listener on the parent handles all of them — including new ones added later.

`e.target` is the exact element that was clicked. `closest()` walks up the DOM tree to find the nearest ancestor matching a selector.

### DOM Traversal

Navigate relative to a known element:

```js
li.parentElement          // the <ul>
li.children               // HTMLCollection of child elements
li.firstElementChild      // first child element
li.nextElementSibling     // the next <li>
e.target.closest('li')    // nearest ancestor that's a <li>
```

### HTML5 Drag and Drop API

Make an element draggable with `draggable="true"`, then handle events:

```js
li.draggable = true;

li.addEventListener('dragstart', e => {
  e.dataTransfer.setData('text/plain', task.id);
  li.classList.add('dragging');
});

li.addEventListener('dragover', e => {
  e.preventDefault();    // required to allow drop
});

li.addEventListener('drop', e => {
  e.preventDefault();
  const srcId = e.dataTransfer.getData('text/plain');
  // reorder logic...
});

li.addEventListener('dragend', () => {
  li.classList.remove('dragging');
});
```

---

## Try It

1. **Priority colors**: Add a `data-priority` attribute (`low`/`medium`/`high`) and use CSS to color the left border of each task.

2. **Edit in place**: Double-click a task's text to replace the `<span>` with an `<input>`, let the user type, then on blur swap it back.

3. **Due dates**: Add a `data-due` attribute. Tasks past their due date should automatically get a red background.

4. **Subtasks**: Add a "+" button that expands the task to show a nested list of subtasks.

---

## Exercises

### Exercise 1 — Task Counter Badge

Show separate counts for active and completed tasks in the header, like:
```
📝 My Tasks   •  3 active  •  2 done
```
Update both numbers every time a task is added, completed, or removed.

**Hint**: Use `tasks.filter(t => !t.done).length` and `tasks.filter(t => t.done).length`.

### Exercise 2 — Keyboard Navigation

Add full keyboard support:
- `Tab` should move focus between tasks
- `Space` on a focused task should toggle it done/undone
- `Delete` or `Backspace` on a focused task should remove it

**Hint**: Make each `<li>` focusable with `tabindex="0"` and listen for `keydown` on the list.

### Exercise 3 — LocalStorage Persistence

Save the tasks array to localStorage every time it changes, and restore it when the page loads:

```js
function saveTasks() {
  localStorage.setItem('tasks', JSON.stringify(tasks));
}

function loadTasks() {
  const saved = localStorage.getItem('tasks');
  if (saved) tasks = JSON.parse(saved);
}
```

Call `loadTasks()` at startup and `saveTasks()` after every mutation. Now tasks survive a page refresh.

---

## Solutions

### Solution 1 — Task Counter Badge

```js
function updateCounts() {
  const active = tasks.filter(t => !t.done).length;
  const done   = tasks.filter(t => t.done).length;

  document.getElementById('taskCount').innerHTML =
    `${active} active &nbsp;•&nbsp; ${done} done`;
}

// Call updateCounts() at the end of render()
```

### Solution 2 — Keyboard Navigation

```js
// In buildTaskEl(), add to the <li>:
li.setAttribute('tabindex', '0');

li.addEventListener('keydown', e => {
  if (e.key === ' ') {
    e.preventDefault();
    const task = tasks.find(t => t.id === parseInt(li.dataset.id));
    if (task) { task.done = !task.done; render(); }
  }
  if (e.key === 'Delete' || e.key === 'Backspace') {
    tasks = tasks.filter(t => t.id !== parseInt(li.dataset.id));
    render();
  }
});
```

### Solution 3 — LocalStorage Persistence

```js
const STORAGE_KEY = 'todo-tasks';

function saveTasks() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
}

function loadTasks() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) tasks = JSON.parse(raw);
  } catch {
    tasks = [];
  }
}

// Wrap render() to also save:
const originalRender = render;
function render() {
  originalRender();
  saveTasks();
}

// At startup:
loadTasks();
render();
```

---

## What You Learned

| Concept | Where it appeared |
|---------|-------------------|
| `document.querySelector` | Finding `#taskInput`, `.filters`, `#taskList` |
| `document.querySelectorAll` | Clearing active class on all filter buttons |
| `createElement` | Building `<li>` elements in `buildTaskEl()` |
| `appendChild` | Attaching tasks to the list |
| `innerHTML = ''` | Clearing the list before re-render |
| `classList.add/remove/toggle` | `done`, `dragging`, `drag-over` classes |
| `dataset` | `li.dataset.id` to link DOM to data |
| Event delegation | One `click` handler on `#taskList` for all tasks |
| `closest()` | `e.target.closest('.task-item')` for delegation |
| DOM traversal | `li.parentElement`, `li.children` |
| HTML5 Drag API | `dragstart`, `dragover`, `drop`, `dragend` |
| `e.preventDefault()` | Allowing drop, stopping default drag behavior |

---

## Building with Claude

- *"Add a 'pomodoro timer' to each task — clicking a tomato icon 🍅 starts a 25-minute countdown shown on the card."*
- *"Make tasks editable by double-clicking the text — swap the span for an input and save on blur."*
- *"Add categories with colored tags — each task can belong to one category, and you can filter by category."*

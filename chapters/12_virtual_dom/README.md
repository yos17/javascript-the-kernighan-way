# Chapter 12 — Build Your Own Virtual DOM

By this point, you are ready for one of the big ideas behind modern UI frameworks. A virtual DOM is just a structured description of the interface plus a way to compare the old version and the new one. It sounds fancy, but the core loop is small enough to build yourself.

---

## The Problem

You have a JavaScript object — the application state — and you want the DOM to reflect it at all times. The naive solution is: wipe the container and rebuild the entire DOM from scratch every time state changes. It works. It's also catastrophically slow for large UIs, breaks animations, loses focus, and scrolls the user back to the top.

The virtual DOM solution: describe the desired UI as a tree of plain objects, then *compare* the new tree against the previous one and touch only the DOM nodes that actually changed. A counter incrementing from 4 to 5 should update exactly one text node, nothing else.

If you are new, the most important idea is this: the virtual DOM is not the real DOM. It is just a JavaScript description of what you want the real DOM to look like. That makes it easier to compare two versions before touching the browser.

---

## Beginner Summary

Before you dive into the code, keep this in mind:

- the virtual DOM is just a JavaScript description of UI
- `render()` builds real DOM from that description
- `patch()` compares old and new descriptions
- the goal is to change as little real DOM as possible

## Building It Step by Step

### v1 — The Naive Approach: Wipe and Rebuild

In plain English: first we build the obvious version, even though it is inefficient, so we can clearly see what problem the better version is solving.

Start with the simplest possible thing that works:

```javascript
function render(vnode) {
  if (vnode.type === '#text') return document.createTextNode(vnode.value);
  const el = document.createElement(vnode.type);
  for (const [key, val] of Object.entries(vnode.props || {})) {
    if (key.startsWith('on')) el[key.toLowerCase()] = val;
    else el.setAttribute(key, val);
  }
  vnode.children.forEach(child => el.appendChild(render(child)));
  return el;
}

function update() {
  mountPoint.innerHTML = '';                        // wipe everything
  mountPoint.appendChild(render(view(state)));     // rebuild from scratch
}
```

This is fifteen words of logic and it works. Open the browser, click a button, the UI updates. But open DevTools and watch the Elements panel: every click destroys and recreates the entire DOM tree.

That means the first version is correct but wasteful. This is another useful beginner lesson: code can be logically right and still be a bad implementation because it does too much work.

Now stress-test it: add 500 todo items, type in the input field, then toggle a checkbox. Your cursor jumps out of the input — because the `<input>` element was destroyed and recreated. Add a CSS `transition: opacity 0.3s` on list items — transitions never play, because elements are replaced before they can animate. On a mobile device with 1000 items, the "rebuild" causes a full layout recalculation: 1000 `createElement` calls, 1000 `setAttribute` calls, one giant paint. A counter incrementing from 4 to 5 re-renders the entire page.

The bottleneck is not JavaScript — it's that every DOM operation is expensive because the browser must recalculate layout and repaint. The fix: touch only what changed.

### v2 — Add `patch()` for the Common Cases

The key insight: most state changes are small. A counter update changes one text node. A checkbox toggle changes one class. Rather than rebuilding everything, compare old and new vnodes and make surgical updates.

Handle the 80% case first — text changes and type mismatches — without recursing into children:

```javascript
function patch(parent, oldVNode, newVNode, i = 0) {
  // New node: add it
  if (!oldVNode) {
    parent.appendChild(render(newVNode));
    return;
  }

  const el = parent.childNodes[i];

  // Removed node: delete it
  if (!newVNode) {
    parent.removeChild(el);
    return;
  }

  // Text change: update in place
  if (oldVNode.type === '#text' && newVNode.type === '#text') {
    if (oldVNode.value !== newVNode.value) el.nodeValue = newVNode.value;
    return;
  }

  // Type changed (div → span): replace the whole subtree
  if (oldVNode.type !== newVNode.type) {
    parent.replaceChild(render(newVNode), el);
    return;
  }

  // Same type: we'd recurse here — for now, do nothing
}
```

This already handles the counter app perfectly. Incrementing `count` from 4 to 5 touches exactly one text node. The `<input>` element is never touched, so focus is preserved. CSS transitions play because elements are never destroyed.

That is the heart of the virtual DOM idea: do your thinking in JavaScript objects first, then make the smallest possible DOM change.

### v3 — Add `updateProps` and `reconcileChildren`

In plain English: now we teach the system how to update the same element instead of replacing it.

For the full diff, we need to update attributes when they change, and recursively diff the children list:

```javascript
// After the type-match case in patch():
updateProps(el, newVNode.props, oldVNode.props);
reconcileChildren(el, oldVNode.children, newVNode.children);
```

`updateProps` does two passes: first remove deleted props, then set new or changed ones. It uses `!==` comparison — which is why immutable state is so valuable here. If an object reference didn't change, the prop didn't change, and `setProp` is never called.

For beginners, this is the key connection: immutable data makes comparison easier. If you create a new object only when something changed, then the diffing code can spot change with a simple comparison instead of re-checking everything deeply.

`reconcileChildren` loops to `Math.max(old.length, new.length)` — handling both additions (old is shorter) and removals (new is shorter) in one loop. The `removals` counter adjusts the real DOM index as nodes are removed (see the diagram in the reconcileChildren section).

---

## The Complete Program

`vdom.html` — a todo list with a counter, filters, and a live inspector showing every DOM operation as it happens.

```javascript
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: Mini Virtual DOM  (~75 lines)
// ═══════════════════════════════════════════════════════════════════════

function h(type, props, ...children) {
  return {
    type,
    props: props || {},
    children: children.flat().map(c =>
      (typeof c === 'string' || typeof c === 'number')
        ? { type: '#text', props: {}, children: [], value: String(c) }
        : c
    ).filter(Boolean)
  };
}

function render(vnode) {
  if (vnode.type === '#text') return document.createTextNode(vnode.value);
  const el = document.createElement(vnode.type);
  updateProps(el, vnode.props, {});
  vnode.children.forEach(child => el.appendChild(render(child)));
  return el;
}

function updateProps(el, newProps, oldProps) {
  for (const key of Object.keys(oldProps)) {
    if (!(key in newProps)) removeProp(el, key);
  }
  for (const key of Object.keys(newProps)) {
    if (newProps[key] !== oldProps[key]) setProp(el, key, newProps[key]);
  }
}

function setProp(el, key, value) {
  if (key.startsWith('on') && typeof value === 'function') {
    el[key.toLowerCase()] = value;
  } else if (key === 'className') {
    el.className = value;
  } else if (key === 'style' && typeof value === 'object') {
    Object.assign(el.style, value);
  } else if (key !== 'key') {
    el.setAttribute(key, value);
  }
}

function removeProp(el, key) {
  if (key.startsWith('on')) { el[key.toLowerCase()] = null; }
  else if (key === 'className') { el.className = ''; }
  else if (key !== 'key') { el.removeAttribute(key); }
}

function patch(parent, oldVNode, newVNode, i = 0) {
  if (!oldVNode) {
    parent.appendChild(render(newVNode));
    return;
  }

  const el = parent.childNodes[i];

  if (!newVNode) {
    parent.removeChild(el);
    return;
  }

  if (oldVNode.type === '#text' && newVNode.type === '#text') {
    if (oldVNode.value !== newVNode.value) el.nodeValue = newVNode.value;
    return;
  }

  if (oldVNode.type !== newVNode.type) {
    parent.replaceChild(render(newVNode), el);
    return;
  }

  updateProps(el, newVNode.props, oldVNode.props);
  reconcileChildren(el, oldVNode.children, newVNode.children);
}

function reconcileChildren(parent, oldChildren, newChildren) {
  const maxLen = Math.max(oldChildren.length, newChildren.length);
  let removals = 0;
  for (let i = 0; i < maxLen; i++) {
    patch(parent, oldChildren[i], newChildren[i], i - removals);
    if (oldChildren[i] && !newChildren[i]) removals++;
  }
}

// ─── App ─────────────────────────────────────────────────────────────

let state = { todos: [...], count: 0, filter: 'all' };

function setState(updater) {
  state = typeof updater === 'function' ? updater(state) : { ...state, ...updater };
  update();
}

function view(s) {
  return h('div', { className: 'app' },
    h('div', { className: 'counter-row' },
      h('button', { onClick: () => setState(s => ({ ...s, count: s.count - 1 })) }, '−'),
      h('div', { className: 'count-display' }, String(s.count)),
      h('button', { onClick: () => setState(s => ({ ...s, count: s.count + 1 })) }, '+'),
    ),
    h('ul', { className: 'todo-list' },
      ...s.todos.map(todo =>
        h('li', { className: 'todo-item' + (todo.done ? ' done' : '') },
          h('div', { className: 'todo-checkbox', onClick: () => toggleTodo(todo.id) }, todo.done ? '✓' : ''),
          h('span', { className: 'todo-text' }, todo.text),
        )
      )
    ),
  );
}

let currentVNode = null;
const mountPoint = document.getElementById('app');

function update() {
  const newVNode = view(state);
  if (!currentVNode) {
    mountPoint.appendChild(render(newVNode));
  } else {
    patch(mountPoint, currentVNode, newVNode, 0);
  }
  currentVNode = newVNode;
}

update();
```

---

## Walkthrough

### Virtual Nodes — UI as Plain Objects

The entire virtual DOM rests on one insight: a DOM node can be represented as a plain JavaScript object. React calls it a React element; Preact calls it a VNode. Whatever the name, the structure is the same:

```javascript
{ type: 'div', props: { className: 'counter' }, children: [...] }
```

`h()` — short for *hyperscript* — constructs these objects:

```javascript
function h(type, props, ...children) {
  return {
    type,
    props: props || {},
    children: children.flat().map(c =>
      (typeof c === 'string' || typeof c === 'number')
        ? { type: '#text', props: {}, children: [], value: String(c) }
        : c
    ).filter(Boolean)
  };
}
```

Three things happen: `props` defaults to an empty object (never `null` — that would break every `Object.keys` call later), string and number children are normalized into text vnodes with a `value` field, and `children.flat()` flattens one level so you can spread arrays inline without nesting.

The result of calling `h('div', { className: 'app' }, h('p', null, 'Hello'))` is:

```javascript
{
  type: 'div',
  props: { className: 'app' },
  children: [
    {
      type: 'p',
      props: {},
      children: [
        { type: '#text', props: {}, children: [], value: 'Hello' }
      ]
    }
  ]
}
```

No DOM. No events. Just nested objects you can serialize, compare, and diff.

### `render()` — From Description to Reality

`render()` walks the vnode tree and creates real DOM nodes from it:

```javascript
function render(vnode) {
  if (vnode.type === '#text') return document.createTextNode(vnode.value);
  const el = document.createElement(vnode.type);
  updateProps(el, vnode.props, {});
  vnode.children.forEach(child => el.appendChild(render(child)));
  return el;
}
```

This is the first and only full DOM build. It's called once on the initial render. Every subsequent update goes through `patch()` instead.

The recursion mirrors the tree structure: `render` creates the element, calls `updateProps` to set its attributes, then recursively renders each child and appends it. The base case is text nodes, handled without recursion.

### The Naive Approach — Why Not Just Rebuild?

Before looking at `patch()`, it's worth understanding the alternative:

```javascript
function update() {
  mountPoint.innerHTML = '';             // wipe everything
  mountPoint.appendChild(render(view(state)));  // rebuild from scratch
}
```

This is fifteen words of code and it works. But run it on a 1000-item list and every keystroke causes 1000 `createElement` calls, 1000 attribute sets, 1000 `appendChild` calls, and a full browser layout recalculation. On a simple counter app, incrementing the number re-creates the entire page. The `<input>` loses focus mid-typing. Scroll position resets. CSS transitions are interrupted because the elements are destroyed and replaced.

The virtual DOM's value isn't that it avoids touching the DOM — it still touches the DOM. Its value is that it touches *only the parts that changed*.

### `patch()` — The Reconciler

`patch()` is the core of the library. It takes the parent element, the old vnode at position `i`, the new vnode at position `i`, and makes the real DOM match the new description with the minimum number of operations.

```
patch(parent, oldVNode, newVNode, i)
─────────────────────────────────────────────────────────
         ┌─────────────┐
         │  old exists?│
         └─────────────┘
            │         │
           NO         YES
            │         │
      append(new)   ┌─────────────┐
                    │  new exists?│
                    └─────────────┘
                       │       │
                      NO      YES
                       │       │
                  remove(old)  ┌─────────────────┐
                               │  both #text?    │
                               └─────────────────┘
                                  │           │
                                 YES          NO
                                  │           │
                         update          ┌─────────────┐
                         nodeValue       │ same type?  │
                                         └─────────────┘
                                            │       │
                                           NO      YES
                                            │       │
                                        replaceChild  updateProps()
                                                      reconcileChildren()
```

```javascript
function patch(parent, oldVNode, newVNode, i = 0) {
  // Case 1: Addition — no old node, just append
  if (!oldVNode) {
    parent.appendChild(render(newVNode));
    return;
  }

  const el = parent.childNodes[i];  // the actual DOM node

  // Case 2: Removal — no new node, remove it
  if (!newVNode) {
    parent.removeChild(el);
    return;
  }

  // Case 3: Text node change
  if (oldVNode.type === '#text' && newVNode.type === '#text') {
    if (oldVNode.value !== newVNode.value) el.nodeValue = newVNode.value;
    return;
  }

  // Case 4: Type changed — replace entirely
  if (oldVNode.type !== newVNode.type) {
    parent.replaceChild(render(newVNode), el);
    return;
  }

  // Case 5: Same type — update props, recurse into children
  updateProps(el, newVNode.props, oldVNode.props);
  reconcileChildren(el, oldVNode.children, newVNode.children);
}
```

Five cases, handled in order. The critical case is Case 5: when the node type is the same, `patch` doesn't recreate the element at all. It calls `updateProps` to sync only the attributes that changed, then calls `reconcileChildren` to diff the children. The recursion bottoms out at text nodes or leaf elements with no children.

Case 4 is an escape hatch: if a `<div>` needs to become a `<span>`, throw away the old subtree and rebuild fresh. This is expensive but rare — changing element type is uncommon in practice.

### `updateProps()` — Efficient Attribute Sync

Props diffing works in two passes — one to remove deleted props, one to set new or changed ones:

```javascript
function updateProps(el, newProps, oldProps) {
  for (const key of Object.keys(oldProps)) {
    if (!(key in newProps)) removeProp(el, key);    // deleted
  }
  for (const key of Object.keys(newProps)) {
    if (newProps[key] !== oldProps[key]) setProp(el, key, newProps[key]);  // added/changed
  }
}
```

The `!==` comparison is why virtual DOM works well with immutable state: if the object reference hasn't changed, the prop hasn't changed, and `setProp` is never called. This pairs naturally with the Redux pattern from Chapter 11 — spread operators in reducers create new objects only for changed data.

`setProp` handles the three special prop categories:

```javascript
function setProp(el, key, value) {
  if (key.startsWith('on') && typeof value === 'function') {
    el[key.toLowerCase()] = value;        // onClick → el.onclick
  } else if (key === 'className') {
    el.className = value;                 // JSX convention
  } else if (key === 'style' && typeof value === 'object') {
    Object.assign(el.style, value);       // style object → CSSStyleDeclaration
  } else if (key !== 'key') {
    el.setAttribute(key, value);          // everything else
  }
}
```

`onClick` instead of `onclick` because JSX uses camelCase by convention. `className` instead of `class` because `class` is a reserved word in JavaScript. `key` is silently skipped — it's a hint to the reconciler (covered in exercises), not a real DOM attribute.

### `reconcileChildren()` — List Diffing

After updating props, the reconciler diffs the children arrays:

```javascript
function reconcileChildren(parent, oldChildren, newChildren) {
  const maxLen = Math.max(oldChildren.length, newChildren.length);
  let removals = 0;
  for (let i = 0; i < maxLen; i++) {
    patch(parent, oldChildren[i], newChildren[i], i - removals);
    if (oldChildren[i] && !newChildren[i]) removals++;
  }
}
```

The loop goes to `maxLen` — the longer of the two arrays — so it handles both additions (old is shorter: `oldChildren[i]` is `undefined`) and removals (new is shorter: `newChildren[i]` is `undefined`).

The `removals` counter solves an index-shifting problem. The diagram makes it concrete:

```
Why `removals` matters
─────────────────────────────────────────────────────────
State: oldChildren=[A, B, C]  newChildren=[A, C]
       (B was removed from the list)

DOM before: [domA, domB, domC]  (indices 0, 1, 2)

Loop i=0: patch(domA, A)  → same, no change
Loop i=1: patch(domB, B, C) → different! replaces B with C
          removes B from DOM → DOM is now [domA, domC]
Loop i=2: patch(domC, C, undefined) → removes C!
          But wait — domC is now at index 1, not 2

Without `removals` counter: wrong node removed!
With `removals` counter (i - removals):
  i=1: index = 1-0 = 1 ✓  (domB at position 1)
  i=2: index = 2-1 = 1 ✓  (domC now at position 1 after removal)
```

When `patch` removes a child at index `j`, the real DOM's `childNodes` shifts: what was at `j+1` is now at `j`. Without the adjustment, the next call would grab the wrong node. Tracking removals and subtracting from `i` keeps the real DOM index aligned with the vnode index.

### The State → View → Patch Loop

Everything ties together in the update loop:

```javascript
let currentVNode = null;

function update() {
  const newVNode = view(state);         // 1. compute new vnode tree
  if (!currentVNode) {
    mountPoint.appendChild(render(newVNode));  // 2a. first render: build
  } else {
    patch(mountPoint, currentVNode, newVNode, 0);  // 2b. re-render: diff & patch
  }
  currentVNode = newVNode;              // 3. save for next diff
}
```

`view` is a pure function: state in, vnode tree out. `patch` diffs that tree against the previous one and mutates the real DOM. `currentVNode` holds the "last known state of the DOM" in object form. The cycle: state changes → `setState` calls `update` → new vnode computed → diffed against old → minimal DOM ops applied → `currentVNode` updated.

That full loop is worth reading several times. It is one of the central patterns in modern frontend programming. Once you understand that loop, React-style rendering stops feeling magical and starts feeling mechanical.

This is the pattern React, Preact, and Vue all follow under the hood. The surface API differs; the loop is the same.

---

## Try It

### Guided: Add Key-Based Reconciliation

**The problem**: The current reconciler matches old and new children purely by position. Adding 'Buy milk' to the *start* of a todo list `[A, B, C]` produces `newChildren = ['Buy milk', A, B, C]`. The loop compares:

- index 0: old=A, new='Buy milk' — different, update A's text
- index 1: old=B, new=A — different, update B's text
- index 2: old=C, new=B — different, update C's text
- index 3: old=undefined, new=C — append a new node

That's 4 DOM operations to prepend one item. With keys, the reconciler can see that A, B, and C haven't changed — they just moved. Result: 1 insertion.

**Why it matters**: Without keys, adding an item to the start of a 100-item list causes 100 text-node updates. With keys, it causes 1 insertion. More critically: without keys, input fields in list items lose their typed text when anything is prepended above them. With keys, the `<input>` DOM node follows its item regardless of position.

**Step 1** — In `reconcileChildren`, before the loop, build a map from key to old vnode index:

```javascript
function reconcileChildren(parent, oldChildren, newChildren) {
  const oldKeyMap = new Map();
  oldChildren.forEach((child, i) => {
    if (child?.props?.key != null) {
      oldKeyMap.set(child.props.key, i);
    }
  });
  // ... rest of function
}
```

This gives you O(1) lookup: given a key, find where it was in the old list.

**Step 2** — In the loop, when the new child has a key, look it up to find its old vnode:

```javascript
for (let i = 0; i < newChildren.length; i++) {
  const newChild = newChildren[i];
  const key = newChild?.props?.key;

  let oldChild;
  if (key != null && oldKeyMap.has(key)) {
    const oldIndex = oldKeyMap.get(key);
    oldChild = oldChildren[oldIndex];
    // oldChild is the same logical item — diff it against newChild
  } else {
    oldChild = oldChildren[i];  // fall back to position-based
  }

  patch(parent, oldChild, newChild, i);
}
```

**Think about it**: What should `key` values be — sequential numbers, or something stable like item IDs? Consider: if you have items with `key={0}`, `key={1}`, `key={2}` and you remove the first one, the remaining items still have keys 1 and 2 — which are now at indices 0 and 1. The keys are stable identifiers, not positions, so this works correctly. But if you used array indices as keys, removing item 0 makes what was item 1 now have the same key as the old item 0 — defeating the whole purpose. Use item IDs, not array indices.

What breaks if two siblings have the same key? The `Map` will only keep the last one — the first one is overwritten. The reconciler will try to reuse the same DOM node for two different items, causing incorrect display. Keys must be unique among siblings.

---

The other exercises, for independent exploration:

2. **Animate the counter**: In CSS, add a transition or animation class. Add a `counting` boolean to state, set it `true` when increment/decrement is clicked, and remove it 300ms later with `setTimeout`. The DOM log will show the class being added and removed.

3. **Implement `h` without JSX transformation**: Move the app into a separate file that uses `h()` directly. Then try installing Babel's JSX transform (or Preact's `@babel/plugin-transform-react-jsx`) and pointing it at your `h` function. Your `view` function written in JSX will compile directly to your virtual DOM.

4. **Serialize and restore state**: Because vnodes are plain objects, you can `JSON.stringify(currentVNode)` to inspect the entire virtual DOM tree as JSON. Add a "Snapshot" button that saves and a "Restore" button that calls `patch` against the saved tree.

---

## Exercises

1. **Implement key-based reconciliation**: The current reconciler matches old and new children purely by index. Adding an item to the *beginning* of a list causes every subsequent item to appear changed (because `oldChildren[1]` is compared to `newChildren[2]` etc). Add key-based matching: before reconciling, build a `Map<key, vnode>` from the old children. In `reconcileChildren`, look up each new child's `key` in the map to find its true old vnode, regardless of position. This is how React's reconciler avoids re-rendering the entire list on every prepend.

2. **Implement functional components**: Real React lets you write `h(MyComponent, { name: 'Alice' })` where `MyComponent` is a function that returns a vnode. Add support to `h` and `patch`: if `type` is a function, call it with `props` and recurse on the returned vnode. Handle the case where a function component is replaced with a different function component (treat as a type mismatch → replace).

3. **Add lifecycle hooks**: Add `onMount(el)` and `onUnmount(el)` as special props. When `patch` renders a new node (Case 1 or Case 4), call `props.onMount(el)` after inserting it. When `patch` removes a node, call `props.onUnmount(el)` before removing it. Implement a fade-in animation using `onMount` and an `opacity` CSS transition.

4. **Implement `useRef`-style mutable refs**: Sometimes you need direct access to the underlying DOM node — for focusing an input, measuring size, or integrating a third-party library. Add a `ref` prop: when `render` creates an element with a `ref` prop, call `ref.current = el`. Add a "Focus input" button to the app that uses a ref to call `inputEl.current.focus()`.

5. **Build a `createApp` abstraction**: Extract the update loop into a reusable function so apps don't have to manage `currentVNode` manually:

   ```javascript
   function createApp(viewFn, initialState, container) {
     // returns an object with .setState(updater)
   }
   ```

   The returned `setState` calls the view function, diffs, and patches automatically. Connect it to the todo app — the app code should have no reference to `currentVNode`, `render`, or `patch` directly.

---

## Solutions

### Exercise 1 — Key-based reconciliation

```javascript
function reconcileChildren(parent, oldChildren, newChildren) {
  // Build a map from key → old vnode for O(1) lookup
  const oldKeyedMap = new Map();
  oldChildren.forEach((child, i) => {
    if (child?.props?.key != null) {
      oldKeyedMap.set(child.props.key, { vnode: child, index: i });
    }
  });

  // Track which old children were matched (to detect removals)
  const matched = new Set();

  newChildren.forEach((newChild, newIndex) => {
    const key = newChild?.props?.key;
    if (key != null && oldKeyedMap.has(key)) {
      const { vnode: oldChild } = oldKeyedMap.get(key);
      matched.add(key);
      // Move the real DOM node to the right position if needed
      const el = parent.childNodes[newIndex];
      const oldEl = findDomNode(parent, oldChild, oldChildren);
      if (el !== oldEl) parent.insertBefore(oldEl, el || null);
      patch(parent, oldChild, newChild, newIndex);
    } else {
      patch(parent, oldChildren[newIndex], newChild, newIndex);
    }
  });

  // Remove old keyed nodes that are no longer in newChildren
  oldKeyedMap.forEach((val, key) => {
    if (!matched.has(key)) {
      const oldEl = findDomNode(parent, val.vnode, oldChildren);
      if (oldEl) parent.removeChild(oldEl);
    }
  });
}

// Helper: find a vnode's actual DOM node using its key
function findDomNode(parent, vnode, oldChildren) {
  const idx = oldChildren.indexOf(vnode);
  return idx >= 0 ? parent.childNodes[idx] : null;
}
```

This version handles the prepend case correctly: each keyed node is matched to its vnode by key, then moved to the correct position with `insertBefore` if necessary.

### Exercise 2 — Functional components

```javascript
// In h(): normalize functional components
function h(type, props, ...children) {
  // If type is a function, call it immediately and return its output
  // (stateless functional components)
  if (typeof type === 'function') {
    const allProps = { ...(props || {}), children: children.flat().filter(Boolean) };
    return type(allProps);
  }
  // ... rest of h unchanged
}
```

For components that need to be diffed rather than inlined (so you can optimize based on prop equality):

```javascript
// Keep component type as-is in the vnode, resolve in patch
function patch(parent, oldVNode, newVNode, i = 0) {
  // Resolve functional components before comparing
  const oldResolved = typeof oldVNode?.type === 'function'
    ? oldVNode.type(oldVNode.props)
    : oldVNode;
  const newResolved = typeof newVNode?.type === 'function'
    ? newVNode.type(newVNode.props)
    : newVNode;

  patch(parent, oldResolved, newResolved, i);
}
```

The first approach (inline in `h`) is simpler; the second allows skipping re-renders when props haven't changed.

### Exercise 3 — Lifecycle hooks

```javascript
function render(vnode) {
  if (vnode.type === '#text') return document.createTextNode(vnode.value);
  const el = document.createElement(vnode.type);
  updateProps(el, vnode.props, {});
  vnode.children.forEach(child => el.appendChild(render(child)));

  // Call onMount after element is ready
  if (typeof vnode.props.onMount === 'function') {
    // Use microtask to ensure element is in the DOM first
    queueMicrotask(() => vnode.props.onMount(el));
  }
  return el;
}

function patch(parent, oldVNode, newVNode, i = 0) {
  if (!newVNode) {
    const el = parent.childNodes[i];
    // Call onUnmount before removing
    if (typeof oldVNode?.props?.onUnmount === 'function') {
      oldVNode.props.onUnmount(el);
    }
    parent.removeChild(el);
    return;
  }
  // ... rest of patch unchanged
}

// Usage: fade-in animation
h('div', {
  className: 'card',
  onMount: el => {
    el.style.opacity = '0';
    el.style.transition = 'opacity 300ms';
    requestAnimationFrame(() => el.style.opacity = '1');
  },
  onUnmount: el => {
    el.style.opacity = '0';
  }
}, 'Hello')
```

### Exercise 4 — Refs

```javascript
function render(vnode) {
  if (vnode.type === '#text') return document.createTextNode(vnode.value);
  const el = document.createElement(vnode.type);
  updateProps(el, vnode.props, {});
  vnode.children.forEach(child => el.appendChild(render(child)));

  // Wire up ref
  if (vnode.props.ref && typeof vnode.props.ref === 'object') {
    vnode.props.ref.current = el;
  }
  return el;
}

// Skip 'ref' in setProp/removeProp to avoid setAttribute errors
function setProp(el, key, value) {
  if (key === 'ref' || key === 'key') return; // internal-only props
  // ... rest unchanged
}

// Usage:
const inputRef = { current: null };

h('input', { type: 'text', ref: inputRef }),
h('button', {
  onClick: () => inputRef.current?.focus()
}, 'Focus'),
```

### Exercise 5 — `createApp`

```javascript
function createApp(viewFn, initialState, container) {
  let state = initialState;
  let currentVNode = null;

  function update() {
    const newVNode = viewFn(state, setState);
    if (!currentVNode) {
      container.appendChild(render(newVNode));
    } else {
      patch(container, currentVNode, newVNode, 0);
    }
    currentVNode = newVNode;
  }

  function setState(updater) {
    state = typeof updater === 'function' ? updater(state) : { ...state, ...updater };
    update();
  }

  update(); // initial render
  return { setState, getState: () => state };
}

// Usage — app code has zero infrastructure:
const { setState } = createApp(
  (state, setState) => h('div', null,
    h('p', null, `Count: ${state.count}`),
    h('button', { onClick: () => setState(s => ({ ...s, count: s.count + 1 })) }, '+'),
  ),
  { count: 0 },
  document.getElementById('app')
);
```

`viewFn` receives both `state` and `setState` — no globals needed. The app is entirely self-contained.

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `h(type, props, ...children)` | Creates a vnode — a plain JS object describing a DOM node |
| Text vnodes | Strings become `{ type: '#text', value: '...' }` — same structure, special type |
| `render(vnode)` | Walks the vnode tree and creates real DOM; called once on first render |
| `patch(parent, old, new, i)` | Diffs two vnodes at position `i` and applies minimal DOM mutations |
| Type mismatch → replace | If `old.type !== new.type`, throw away the subtree and rebuild |
| `updateProps` two-pass | Pass 1: remove deleted props. Pass 2: set new/changed props. |
| `!==` prop comparison | Works because immutable state means unchanged data = same object reference |
| `removals` counter | Tracks nodes removed so far so index arithmetic stays correct during children diff |
| `key` prop | Signals identity across positions — tells the reconciler "this is the same item, just moved" |
| State → vnode → patch loop | Pure view function + stored `currentVNode` + patch = the entire framework model |

---

## Building with Claude

Bad prompt:
> "How does React's virtual DOM work?"

Good prompt:
> "I've built a virtual DOM library: `h()` creates vnode objects, `render()` builds real DOM from a vnode, and `patch()` diffs two vnodes and updates the real DOM. My `reconcileChildren()` uses index-based matching — it loops to `max(old.length, new.length)` and compares old[i] to new[i]. I'm seeing a bug: when I prepend an item to a list, every existing item gets re-rendered because their indices shifted. I want to add key-based matching. My plan: build a `Map<key, {vnode, domNode}>` from the old children, then for each new child with a key, look it up in that map. My question: when I find a keyed match at a different position, do I need to move the real DOM node, or can I just call patch in place? And after keyed reconciliation, how do I detect which old keyed nodes were removed so I can call `parent.removeChild` on them?"

That prompt explains the existing implementation, describes the specific symptom, proposes the solution, and asks about two precise sub-problems — DOM node movement and orphan detection — that a shallow answer would miss.

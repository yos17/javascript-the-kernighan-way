# Chapter 6 — Build Your Own jQuery

jQuery was the dominant JavaScript library for over a decade. At its peak, it ran on 78% of all websites. Today, most frameworks have replaced it — but its API design is still the clearest ever built for DOM manipulation. Understanding how `$()` works, how method chaining is implemented, and how event delegation scales reveals the patterns that every modern framework builds on. In this chapter you build a miniature version from scratch.

---

## The Problem

In the mid-2000s, writing cross-browser JavaScript was genuinely painful. Internet Explorer and Firefox implemented the DOM differently — `attachEvent` vs `addEventListener`, `innerText` vs `textContent`, broken `querySelectorAll` support. John Resig released jQuery in 2006 to solve that specific problem: one API that works everywhere.

But jQuery's enduring influence isn't the browser-normalisation layer — modern browsers solved that. It's the *design* of the API. Resig made three choices that shaped the web:

1. **Everything is a set.** `$('.btn')` doesn't give you one button — it gives you all of them. Every operation applies to the whole set automatically. You never write loops.

2. **Everything chains.** `$('.btn').addClass('active').css('color', 'red').on('click', fn)` reads like a sentence. It works because every method returns the same wrapped object.

3. **One listener, many elements.** Event delegation — attaching a single listener to a parent and identifying the real target when an event fires — means you can handle 1000 dynamic list items with one line of code.

These three choices — set operations, method chaining, event delegation — are not jQuery-specific tricks. They're patterns that React, Vue, Angular, and every other framework inherited. Understanding them in jQuery, where the implementation is small enough to hold in your head, gives you the mental model for all of them.

You have a web page with navigation, tabs, accordions, modals, and card filters. Wiring all of this with raw `document.querySelector` calls produces tangled, verbose code. Build the wrapper, and complex DOM work becomes simple.

---

## Building It Step by Step

### v1 — Wrap a Single Element, Add One Method

The simplest possible jQuery wraps one element and adds one method. It's barely useful, but the core idea is visible immediately:

```javascript
// v1: wrap one element, add .addClass()
function $(selector) {
  const element = document.querySelector(selector);  // just one element

  return {
    element,
    addClass(cls) {
      element.classList.add(cls);
      // no return — can't chain yet
    }
  };
}

$('.btn').addClass('active');
// Works, but only finds one element and can't chain
```

The key insight is already here: `$()` doesn't return an element — it returns a *plain object* that contains an element. This separation is what makes the API extensible.

### v2 — Wrap an Array of Elements, Loop Internally

The second observation: the real value of the wrapper is that every method loops over *all* matched elements, so callers never have to:

```javascript
// v2: wrap all matching elements
function $(selector) {
  const elements = Array.from(document.querySelectorAll(selector)); // all elements

  return {
    elements,
    length: elements.length,

    addClass(cls) {
      elements.forEach(el => el.classList.add(cls));
      // still no return — still can't chain
    },

    on(event, fn) {
      elements.forEach(el => el.addEventListener(event, fn));
      // loops internally — caller never sees the loop
    }
  };
}

$('.btn').addClass('active');
// Now applies to ALL .btn elements
// But $('.btn').addClass('active').on('click', fn) still crashes — returns undefined
```

Now the abstraction does real work. One call operates on many elements. But every method chain breaks because methods return `undefined`.

### v3 — Return `this` for Chaining, Add Traversal

The third observation: add one line to every method — `return this` — and the entire fluent API becomes possible. Then add traversal so chains can navigate the DOM:

```javascript
// v3: return this + traversal
function $(selector, context) {
  // normalise all input types to a flat array of elements
  let elements;
  if (typeof selector === 'string') {
    elements = Array.from((context || document).querySelectorAll(selector));
  } else if (selector instanceof Element) {
    elements = [selector];
  } else if (Array.isArray(selector)) {
    elements = selector;
  }

  const api = {
    elements,

    addClass(cls) {
      elements.forEach(el => el.classList.add(cls));
      return this;    // ← the one line that makes chaining possible
    },

    on(event, fn) {
      elements.forEach(el => el.addEventListener(event, fn));
      return this;
    },

    // Traversal returns a NEW wrapped set — chaining continues on the new set
    find(sel) {
      const found = [];
      elements.forEach(el => found.push(...el.querySelectorAll(sel)));
      return $(found);    // ← new wrapper, same API, chain continues
    },

    closest(sel) {
      return $(elements.map(el => el.closest(sel)).filter(Boolean));
    }
  };

  return api;
}

// Now this works:
$('.accordion-header')
  .closest('.accordion-item')   // traverses up — returns new wrapper
  .addClass('open')             // operates on new wrapper
  .find('.accordion-body')      // traverses down — returns another new wrapper
  .on('click', handleClick);    // attaches listeners to found elements
```

Adding `return this` to each method and making `find`/`closest` return new wrappers is the complete mechanism behind jQuery's fluent API. Everything else is filling in more methods.

---

## The Complete Program

`jquery.html` — open it in your browser to see a fully interactive page driven entirely by `$`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Build Your Own jQuery — Chapter 6</title>
  <!-- styles omitted for brevity — see jquery.html -->
</head>
<body>
<script>
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: $ (Mini jQuery)
// ═══════════════════════════════════════════════════════════════════════

function $(selector, context) {
  let elements;
  if (typeof selector === 'string') {
    elements = Array.from((context || document).querySelectorAll(selector));
  } else if (selector instanceof Element || selector instanceof Window ||
             selector instanceof Document) {
    elements = [selector];
  } else if (selector instanceof NodeList || Array.isArray(selector)) {
    elements = Array.from(selector);
  } else {
    elements = [];
  }

  const api = {
    elements,
    length: elements.length,

    // ─── Iteration ─────────────────────────────────────────────────────
    each(fn) {
      elements.forEach((el, i) => fn.call(el, el, i));
      return this;
    },
    get(i) { return elements[i]; },

    // ─── Classes ────────────────────────────────────────────────────────
    addClass(cls) {
      elements.forEach(el => el.classList.add(...cls.split(' ')));
      return this;
    },
    removeClass(cls) {
      elements.forEach(el => el.classList.remove(...cls.split(' ')));
      return this;
    },
    toggleClass(cls, force) {
      elements.forEach(el => el.classList.toggle(cls, force));
      return this;
    },
    hasClass(cls) {
      return elements.some(el => el.classList.contains(cls));
    },

    // ─── Content ────────────────────────────────────────────────────────
    html(content) {
      if (content === undefined) return elements[0] ? elements[0].innerHTML : '';
      elements.forEach(el => { el.innerHTML = content; });
      return this;
    },
    text(content) {
      if (content === undefined) return elements[0] ? elements[0].textContent : '';
      elements.forEach(el => { el.textContent = content; });
      return this;
    },
    val(value) {
      if (value === undefined) return elements[0] ? elements[0].value : '';
      elements.forEach(el => { el.value = value; });
      return this;
    },

    // ─── Styles ─────────────────────────────────────────────────────────
    css(prop, value) {
      if (typeof prop === 'object') {
        elements.forEach(el => Object.assign(el.style, prop));
        return this;
      }
      if (value === undefined) {
        return elements[0] ? getComputedStyle(elements[0])[prop] : '';
      }
      elements.forEach(el => { el.style[prop] = value; });
      return this;
    },

    // ─── Attributes ─────────────────────────────────────────────────────
    attr(name, value) {
      if (value === undefined) return elements[0] ? elements[0].getAttribute(name) : null;
      elements.forEach(el => el.setAttribute(name, value));
      return this;
    },
    removeAttr(name) {
      elements.forEach(el => el.removeAttribute(name));
      return this;
    },
    data(key, value) {
      if (value === undefined) return elements[0] ? elements[0].dataset[key] : undefined;
      elements.forEach(el => { el.dataset[key] = value; });
      return this;
    },

    // ─── Events ─────────────────────────────────────────────────────────
    on(event, fn) {
      elements.forEach(el => el.addEventListener(event, fn));
      return this;
    },
    off(event, fn) {
      elements.forEach(el => el.removeEventListener(event, fn));
      return this;
    },
    trigger(event) {
      elements.forEach(el => el.dispatchEvent(new Event(event, { bubbles: true })));
      return this;
    },

    // ─── Traversal ──────────────────────────────────────────────────────
    find(selector) {
      const found = [];
      elements.forEach(el => found.push(...el.querySelectorAll(selector)));
      return $(found);
    },
    closest(selector) {
      return $(elements.map(el => el.closest(selector)).filter(Boolean));
    },
    parent() {
      return $(elements.map(el => el.parentElement).filter(Boolean));
    },
    children(selector) {
      const kids = [];
      elements.forEach(el => kids.push(...el.children));
      return selector ? $(kids).filter(selector) : $(kids);
    },
    filter(selector) {
      if (typeof selector === 'function') return $(elements.filter(selector));
      return $(elements.filter(el => el.matches(selector)));
    },

    // ─── Visibility ─────────────────────────────────────────────────────
    show() { elements.forEach(el => { el.style.display = ''; }); return this; },
    hide() { elements.forEach(el => { el.style.display = 'none'; }); return this; },
    toggle(state) {
      if (state !== undefined) return state ? this.show() : this.hide();
      elements.forEach(el => {
        el.style.display = el.style.display === 'none' ? '' : 'none';
      });
      return this;
    },

    // ─── DOM manipulation ───────────────────────────────────────────────
    append(content) {
      elements.forEach(el => {
        if (typeof content === 'string') {
          el.insertAdjacentHTML('beforeend', content);
        } else if (content.elements) {
          content.elements.forEach(c => el.appendChild(c));
        } else {
          el.appendChild(content);
        }
      });
      return this;
    },
    prepend(content) {
      elements.forEach(el => {
        if (typeof content === 'string') {
          el.insertAdjacentHTML('afterbegin', content);
        } else {
          el.insertBefore(content.elements ? content.elements[0] : content, el.firstChild);
        }
      });
      return this;
    },
    remove() { elements.forEach(el => el.remove()); return this; },
    empty()  { elements.forEach(el => { el.innerHTML = ''; }); return this; },
  };

  return api;
}

// ─── Static helpers ───────────────────────────────────────────────────

$.ready = function(fn) {
  if (document.readyState !== 'loading') fn();
  else document.addEventListener('DOMContentLoaded', fn);
};

$.delegate = function(parent, event, selector, fn) {
  $(parent).on(event, function(e) {
    const target = e.target.closest(selector);
    if (target && parent.contains(target)) {
      fn.call(target, e, $(target));
    }
  });
};
</script>
</body>
</html>
```

---

## Walkthrough

### The Wrapper Pattern — How `$()` Works

```
$('.card')  →  Wrapped Set
               ┌─────────────────────────────────────┐
               │ elements: [div.card, div.card, div.card] │
               │                                      │
               │ .addClass()   → forEach + classList  │
               │ .on()         → forEach + addEventListener │
               │ .find()       → forEach + querySelectorAll │
               │ .closest()    → map + closest()       │
               │ .text()       → elements[0].textContent │
               └─────────────────────────────────────┘
                        each method returns `this` ↑
```

The key idea in jQuery is a **wrapper**: `$('.btn')` doesn't return a DOM element. It returns a plain JavaScript object that *contains* an array of DOM elements and has methods for manipulating them.

```javascript
function $(selector) {
  const elements = Array.from(document.querySelectorAll(selector));

  return {
    elements,
    length: elements.length,
    // methods...
  };
}
```

This object is not a class instance, not a special type — it's just a plain object with a property called `elements` and a bunch of functions. That simplicity is what makes it so extensible.

The function accepts multiple input types — string selectors, raw Elements, NodeLists, arrays — and normalises them all into a flat array of elements before building the wrapper:

```javascript
if (typeof selector === 'string') {
  elements = Array.from((context || document).querySelectorAll(selector));
} else if (selector instanceof Element) {
  elements = [selector];
} else if (selector instanceof NodeList || Array.isArray(selector)) {
  elements = Array.from(selector);
}
```

### Method Chaining — `return this`

Every method that modifies the DOM ends with `return this`. That one line makes chaining possible:

```javascript
addClass(cls) {
  elements.forEach(el => el.classList.add(...cls.split(' ')));
  return this;   // ← this is the wrapped set
},
```

Because `addClass` returns the same wrapped set, you can call another method immediately:

```javascript
$('.card')
  .addClass('highlighted')
  .css('border-color', '#34d399')
  .on('click', openDetail);
```

Without `return this`, each method call would return `undefined` and the chain would break. Chaining is not magic — it's a discipline of always returning the object.

Methods that *read* a value (`.html()`, `.text()`, `.val()` with no argument, `.css(prop)`) break the chain intentionally — they return the value itself, not the wrapper:

```javascript
html(content) {
  if (content === undefined) return elements[0].innerHTML;  // ← read: returns string
  elements.forEach(el => { el.innerHTML = content; });      // ← write: falls through to...
  return this;                                               // ← return wrapper
},
```

This is called the **getter/setter dual** — same method name, different behaviour based on whether an argument was passed.

### Operating on Sets — Not Single Elements

The most important difference from raw DOM APIs: `$('.btn')` selects *all* matching elements and every method operates on *all* of them automatically.

```javascript
// Raw DOM — have to loop manually
document.querySelectorAll('.btn').forEach(el => {
  el.classList.add('active');
  el.style.color = 'red';
});

// With $
$('.btn').addClass('active').css('color', 'red');
```

Internally, `addClass` is just:

```javascript
addClass(cls) {
  elements.forEach(el => el.classList.add(cls));
  return this;
}
```

But the abstraction means you never think about loops again.

### Traversal — `.find()`, `.closest()`, `.parent()`

DOM traversal methods navigate from the current set to a related set:

```javascript
find(selector) {
  const found = [];
  elements.forEach(el => found.push(...el.querySelectorAll(selector)));
  return $(found);  // ← returns a NEW wrapped set
},

closest(selector) {
  return $(elements.map(el => el.closest(selector)).filter(Boolean));
},
```

`find()` searches *inside* each element. `closest()` walks *up* the DOM tree. Both return a new wrapped set — so you can chain further methods onto the result:

```javascript
$('.accordion-header')
  .closest('.accordion-item')
  .toggleClass('open');
```

### Event Delegation — One Listener for Many Elements

Attaching listeners to every element in a list is wasteful, especially for dynamically added elements. **Event delegation** attaches one listener to a parent and checks each event's target:

```javascript
$.delegate = function(parent, event, selector, fn) {
  $(parent).on(event, function(e) {
    const target = e.target.closest(selector);
    if (target && parent.contains(target)) {
      fn.call(target, e, $(target));
    }
  });
};
```

When any click fires on `parent`, `e.target.closest(selector)` walks up from the actual click target looking for a match. If found, the callback fires. This is how the tab system and navigation work — one listener on `document.body` handles all nav clicks:

```javascript
$.delegate(document.querySelector('nav'), 'click', '.nav-link', function(e, $el) {
  const page = $el.data('page');
  $('.nav-link').removeClass('active');
  $el.addClass('active');
  $('.page').removeClass('active');
  $('#' + page).addClass('active');
});
```

That's the complete navigation implementation: 6 lines that read like English.

### `.data()` — Reading `data-*` Attributes

HTML5 lets you store arbitrary values in `data-*` attributes. The browser exposes them through `element.dataset`, and `$` wraps that:

```javascript
data(key, value) {
  if (value === undefined) return elements[0] ? elements[0].dataset[key] : undefined;
  elements.forEach(el => { el.dataset[key] = value; });
  return this;
},
```

```html
<button data-page="about">About</button>
```

```javascript
$('.nav-link').data('page'); // → 'about'
```

This is how the navigation tabs know which page to show without any hidden state variables — the information lives in the HTML.

---

## Guided Try It — Build `$.delegate` Step by Step

**The problem**: You have a list of 200 items. Each item has a delete button. You could attach 200 click listeners — one per button. Or you could attach one listener to the parent list. Event delegation is the second approach.

The tricky part: the user might click the icon *inside* the button, not the button element itself. Your listener needs to find the button regardless of which child element was actually clicked.

**Hint**: `e.target` is the exact element the user clicked. It might be a `<span>` inside your button. `e.target.closest(selector)` walks up from the click target until it finds a matching ancestor.

**Step 1 — Attach one listener to the parent**

Instead of looping over children, attach to the container:

```javascript
$.delegate = function(parent, event, selector, fn) {
  parent.addEventListener(event, function(e) {
    // e.target is what was clicked — might be deep inside an item
    console.log('clicked:', e.target);
  });
};
```

One listener covers all current *and future* children. If you add items to the list later, they're covered automatically with no extra code.

**Step 2 — Find the matching ancestor**

`e.target` may be a `<span>` or `<svg>` inside the item. Use `closest()` to walk up:

```javascript
$.delegate = function(parent, event, selector, fn) {
  parent.addEventListener(event, function(e) {
    const target = e.target.closest(selector);
    // If the click was inside a matching element, target is that element.
    // If not (user clicked the list background), target is null.
    if (target) {
      fn.call(target, e);
    }
  });
};
```

Now `$.delegate(list, 'click', '.delete-btn', fn)` fires `fn` whenever any `.delete-btn` (or a child of one) is clicked.

**Step 3 — Guard with `parent.contains`**

There's a subtle bug: `closest()` walks all the way up the DOM tree. If the selector matches something *outside* the parent (say, a `.delete-btn` in a completely different part of the page that happens to be an ancestor), you'd fire incorrectly.

```javascript
$.delegate = function(parent, event, selector, fn) {
  $(parent).on(event, function(e) {
    const target = e.target.closest(selector);
    if (target && parent.contains(target)) {   // ← guard: must be inside parent
      fn.call(target, e, $(target));
    }
  });
};
```

**Step 4 — Use it**

```javascript
$.delegate(document.querySelector('#item-list'), 'click', '.delete-btn', function(e, $btn) {
  e.preventDefault();
  $btn.closest('.list-item').remove();
});
```

One listener. Works for 200 items. Works for items added later. Works even if the button contains an icon element.

**Think about it**: What happens if you use `parent.addEventListener` instead of the `$(parent).on(...)` wrapper? What does wrapping in `$()` buy you here? And why is `parent.contains(target)` needed when `parent` is already the element we attached the listener to — can't `closest()` only return descendants?

(Answer: `closest()` matches on the element *itself* first, then walks up. So if the matched element is the `parent` itself or an ancestor of `parent`, `contains` catches it.)

---

## Exercises

1. **Implement `.index()`**: `$('.tab-btn').index()` should return the position of the first element among its siblings (0-based). `$('.tab-btn').index($('.tab-btn').get(2))` should return 2. Use `Array.from(el.parentElement.children).indexOf(el)` internally.

2. **Implement `.clone()`**: Returns a new wrapped set containing deep copies of all elements. `deep` parameter (default `true`) controls whether event listeners are also cloned (hint: `cloneNode(deep)` handles the structure; listeners can't be cloned with native APIs — just note this limitation).

3. **Implement `.animate(props, duration)`**: Transitions numeric CSS properties from current value to target value over `duration` milliseconds. Use `requestAnimationFrame` to update on each frame. Implement for at least `opacity` and `fontSize`. Hint: lerp (linear interpolate) between start and end values.

4. **Implement `.serialize()`**: Given a `<form>` element in the wrapped set, collect all `input`, `select`, and `textarea` values and return them as a URL-encoded string like `name=Ada&role=eng`. Use `new FormData(el)` and `URLSearchParams` for the conversion.

5. **Implement `$.ajax(url, options)`**: A thin wrapper around `fetch` that returns a plain Promise. Support `options.method` (default `'GET'`), `options.data` (sent as JSON body with `Content-Type: application/json`), and `options.headers`. On network error, reject; on HTTP error (status >= 400), reject with the status code.

---

## Solutions

### Exercise 1 — `.index()`

```javascript
index(el) {
  if (el === undefined) {
    // index of first element among its siblings
    const first = elements[0];
    if (!first || !first.parentElement) return -1;
    return Array.from(first.parentElement.children).indexOf(first);
  }
  // index of `el` within this set
  const raw = el instanceof Element ? el : (el.elements ? el.elements[0] : null);
  return elements.indexOf(raw);
},
```

Usage:

```javascript
const tabs = $('.tab-btn');
tabs.index();           // 0 — position of first tab among siblings
tabs.index(tabs.get(2)); // 2 — position of third tab within the set
```

### Exercise 2 — `.clone()`

```javascript
clone(deep = true) {
  return $(elements.map(el => el.cloneNode(deep)));
},
```

Important limitation: `cloneNode` copies the DOM structure but not event listeners attached via `addEventListener`. This is a fundamental browser constraint — there's no API to enumerate or copy listeners. Real jQuery works around this with its own internal listener registry.

```javascript
const copy = $('.card').clone();
$('#card-grid').append(copy);
```

### Exercise 3 — `.animate(props, duration)`

```javascript
animate(props, duration = 300) {
  elements.forEach(el => {
    const computed = getComputedStyle(el);
    const start = {};
    const end = {};
    const units = {};

    for (const [prop, targetVal] of Object.entries(props)) {
      const raw = computed[prop];
      const match = String(raw).match(/^([\d.]+)(.*)$/);
      start[prop] = match ? parseFloat(match[1]) : 0;
      units[prop] = match ? match[2] : '';
      end[prop]   = parseFloat(targetVal);
    }

    let startTime = null;
    function frame(ts) {
      if (!startTime) startTime = ts;
      const progress = Math.min((ts - startTime) / duration, 1);
      for (const prop of Object.keys(props)) {
        const val = start[prop] + (end[prop] - start[prop]) * progress;
        el.style[prop] = val + units[prop];
      }
      if (progress < 1) requestAnimationFrame(frame);
    }
    requestAnimationFrame(frame);
  });
  return this;
},
```

Usage:

```javascript
$('#hero').animate({ opacity: '0', fontSize: '8px' }, 500);
```

### Exercise 4 — `.serialize()`

```javascript
serialize() {
  const form = elements[0];
  if (!form) return '';
  return new URLSearchParams(new FormData(form)).toString();
},
```

`FormData` automatically collects all named inputs. `URLSearchParams` converts it to the `key=value&key=value` format. Two lines — the platform does the heavy lifting.

### Exercise 5 — `$.ajax()`

```javascript
$.ajax = function(url, options = {}) {
  const { method = 'GET', data, headers = {} } = options;

  const fetchOptions = { method, headers: { ...headers } };

  if (data !== undefined) {
    fetchOptions.body = JSON.stringify(data);
    fetchOptions.headers['Content-Type'] = 'application/json';
  }

  return fetch(url, fetchOptions).then(response => {
    if (!response.ok) {
      return Promise.reject(new Error('HTTP ' + response.status + ': ' + response.statusText));
    }
    const contentType = response.headers.get('content-type') || '';
    return contentType.includes('application/json')
      ? response.json()
      : response.text();
  });
};
```

Usage:

```javascript
$.ajax('/api/users')
  .then(users => $('ul').html(users.map(u => `<li>${u.name}</li>`).join('')))
  .catch(err  => console.error(err.message));

$.ajax('/api/users', { method: 'POST', data: { name: 'Ada' } })
  .then(created => console.log('Created:', created));
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| Wrapper pattern | `$()` returns a plain object containing an array of elements and methods |
| Method chaining | Every mutating method returns `this` — the same wrapper object |
| Getter/setter dual | Same method name, read or write based on whether an argument is passed |
| Set operations | Methods loop over all matched elements internally; callers never loop |
| `.find()` / `.closest()` | Traversal returns a new wrapped set — all subsequent methods apply to it |
| Event delegation | One listener on a parent, `closest()` to identify the true target |
| `parent.contains(target)` | Guards against `closest()` matching outside the parent boundary |
| `data-*` attributes | `element.dataset` provides a clean read/write interface to `data-*` |
| `insertAdjacentHTML` | Faster than `.innerHTML +=` — doesn't re-parse the whole element |
| `$.ready()` | Guards code that runs before the DOM is built |
| `return this` | The single rule that makes fluent APIs possible |

---

## Building with Claude

Bad prompt:
> "Build me a jQuery clone."

Good prompt:
> "I've built a `$()` function that wraps DOM elements and supports `.addClass()`, `.on()`, `.find()`, and `.closest()`. Method chaining works because every method returns `this`. Now I want to add event delegation — a `$.delegate(parent, event, selector, fn)` static method that attaches one listener to `parent` and calls `fn` only when the clicked element matches `selector`. I understand `e.target.closest(selector)` is the key — can you explain exactly why we also need `parent.contains(target)` and what would break without it?"

The good prompt demonstrates existing knowledge, names the mechanism, and asks for the *why* behind one specific design decision — the kind of thing Claude explains well and documentation glosses over.

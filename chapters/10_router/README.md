# Chapter 10 — Build Your Own Router

A router is the piece that makes a browser app feel like a multi-page site without full page reloads. That sounds advanced, but the main idea is simple: read the URL, match it, and run the right code. This chapter keeps that idea visible while building a more capable router step by step.

---

## The Problem

You have a multi-page app — a home page, a user list, user detail pages with dynamic IDs, a blog with tag filtering. In a traditional website, each URL triggers a server request. In a single-page app, JavaScript handles everything: the URL changes but no request is made. Build the router and you'll understand how React Router's `<Route path="/users/:id">` works at the machine level.

A beginner-friendly way to think about a router is: it is just an `if/else` system for URLs. Instead of asking "is this button clicked?" you ask "does this URL match this pattern?" Then you run the code for that page.

---

## Beginner Summary

Before you dive into the code, keep this in mind:

- a router watches the URL
- it decides which page handler matches that URL
- it passes route information to that handler
- then your app renders the right view

## Building It Step by Step

### v1 — Hash Router (15 lines)

In plain English: we are listening for URL changes and matching simple paths.

Before the History API existed, SPAs used `#`-based URLs. The hash portion of a URL (`#/users/1`) is never sent to the server — it's purely client-side. This makes it the simplest possible router: no `pushState`, no `popstate`, just `hashchange`.

```javascript
class HashRouter {
  constructor() {
    this.routes = [];
    window.addEventListener('hashchange', () => this._dispatch());
  }

  get(pattern, handler) {
    this.routes.push({ pattern, handler });
    return this;
  }

  _dispatch() {
    const path = location.hash.slice(1) || '/';  // '#/users/1' → '/users/1'
    for (const { pattern, handler } of this.routes) {
      if (path === pattern) {
        handler({ path });
        return;
      }
    }
  }

  start() { this._dispatch(); return this; }
}

// Usage:
const router = new HashRouter();
router
  .get('/',       ctx => { app.innerHTML = '<h1>Home</h1>'; })
  .get('/users',  ctx => { app.innerHTML = '<h1>Users</h1>'; })
  .start();
// URLs look like: myapp.com/#/users
```

This is already useful. You can pass the router around, register routes anywhere in your code, and the `hashchange` event fires reliably. The limitation: no dynamic params (`:id`), no query strings, and the `#` in every URL.

If you are new, do not worry about memorizing browser APIs here. Focus on the flow:

1. read the current URL
2. find a matching route
3. run the handler for that route
4. render the right view

That same flow survives through every version of the router.

### v2 — Upgrade to the History API

`history.pushState` gives you clean URLs (`/users/1` instead of `#/users/1`). The trade-off: the server must serve your `index.html` for all paths, because a direct browser request to `/users/1` would hit the server. In development (and most deployment setups), this is handled by config.

```javascript
class Router {
  constructor(outlet) {
    this.routes    = [];
    this._notFound = null;
    this._outlet   = outlet;

    // pushState doesn't fire popstate — but Back/Forward do
    window.addEventListener('popstate', () => {
      this._dispatch(location.pathname + location.search);
    });
  }

  get(path, handler) {
    this.routes.push({ pattern: path, handler });  // still exact match for now
    return this;
  }

  navigate(path) {
    history.pushState({}, '', path);  // ← update URL, no server request
    this._dispatch(path);              // ← call the matching handler
  }

  _dispatch(fullPath) {
    for (const { pattern, handler } of this.routes) {
      if (fullPath === pattern) {
        handler({ path: fullPath });
        return;
      }
    }
    if (this._notFound) this._notFound({ path: fullPath });
  }

  start() {
    this._dispatch(location.pathname + location.search);
    return this;
  }
}
```

Same route registration as before. Same patterns. The only change: `navigate()` uses `pushState` instead of setting `location.hash`, and we listen for `popstate` instead of `hashchange`.

### v3 — Named Params, Query Strings, and `_compile()`

In plain English: now the router can understand flexible URLs instead of only exact strings.

The final version replaces exact string matching with RegExp pattern matching, adds `URLSearchParams` for query strings, and moves route registration to use `_compile()`.

```javascript
get(path, handler) {
  this.routes.push({
    pattern:  this._compile(path),  // '/users/:id' → RegExp
    handler,
    original: path,
  });
  return this;
}

_compile(path) {
  const pattern = path
    .replace(/\//g, '\\/')
    .replace(/:(\w+)/g, '(?<$1>[^\\/]+)')  // :id → named capture group
    .replace(/\*/g, '.*');
  return new RegExp(`^${pattern}\\/?$`);
}

_dispatch(fullPath) {
  const [pathname, search] = fullPath.split('?');
  const query = this._parseQuery(search ? '?' + search : '');

  for (const route of this.routes) {
    const match = pathname.match(route.pattern);
    if (match) {
      route.handler({ path: pathname, params: match.groups || {}, query });
      return;
    }
  }

  if (this._notFound) this._notFound({ path: pathname, query, params: {} });
}
```

Now `/users/:id` matches `/users/42`, and `ctx.params.id === '42'`. Query strings like `?tag=history` land in `ctx.query.tag`.

That means the router is doing two jobs at once:

- deciding **which page** to show
- extracting **which data** the page needs from the URL

That is why routing matters so much in frontend apps.

---

## The Complete Program

`router.html` — open it to see a full SPA with navigation, dynamic user detail pages (try `/users/3`), and query-filtered blog posts. Watch the debug bar at the bottom.

```javascript
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: Router
// ═══════════════════════════════════════════════════════════════════════

class Router {
  constructor(outlet) {
    this.routes    = [];
    this._notFound = null;
    this._outlet   = outlet;
    this._onNavigate = null;

    // Handle browser Back/Forward
    window.addEventListener('popstate', () => {
      this._dispatch(location.pathname + location.search);
    });
  }

  get(path, handler) {
    this.routes.push({ pattern: this._compile(path), handler, original: path });
    return this; // enable chaining
  }

  on404(handler)     { this._notFound = handler;  return this; }
  onNavigate(hook)   { this._onNavigate = hook;   return this; }

  navigate(path) {
    history.pushState({}, '', path);
    this._dispatch(path);
  }

  back()    { history.back(); }
  forward() { history.forward(); }

  // Convert '/users/:id' → RegExp with named capture group (?<id>[^/]+)
  _compile(path) {
    const pattern = path
      .replace(/\//g, '\\/')
      .replace(/:(\w+)/g, '(?<$1>[^\\/]+)')
      .replace(/\*/g, '.*');
    return new RegExp(`^${pattern}\\/?$`);
  }

  _parseQuery(search) {
    const params = {};
    if (!search) return params;
    new URLSearchParams(search).forEach((value, key) => { params[key] = value; });
    return params;
  }

  _dispatch(fullPath) {
    const [pathname, search] = fullPath.split('?');
    const query = this._parseQuery(search ? '?' + search : '');
    const ctx   = { path: pathname, query };

    for (const route of this.routes) {
      const match = pathname.match(route.pattern);
      if (match) {
        ctx.params       = match.groups || {};
        ctx.matchedRoute = route.original;
        route.handler(ctx);
        if (this._onNavigate) this._onNavigate(ctx);
        return;
      }
    }

    if (this._notFound) this._notFound({ path: pathname, query, params: {} });
  }

  start() {
    this._dispatch(location.pathname + location.search);
    return this;
  }
}

// ─── Wire up ─────────────────────────────────────────────────────────

const router = new Router(document.getElementById('app'));

router
  .get('/',          ctx => { app.innerHTML = renderHome();          })
  .get('/users',     ctx => { app.innerHTML = renderUsers(ctx);      })
  .get('/users/:id', ctx => { app.innerHTML = renderUserDetail(ctx); })
  .get('/blog',      ctx => { app.innerHTML = renderBlog(ctx);       })
  .on404(           ctx => { app.innerHTML = render404(ctx);         })
  .onNavigate(ctx => updateNav(ctx))
  .start();
```

---

## Walkthrough

### The History API — URL Without Reload

The browser gives you `history.pushState(state, title, url)`. It updates the URL bar and adds an entry to the browser history — but it does not request the new URL from any server. The page stays put; JavaScript takes over.

```javascript
// Before: URL is /users
history.pushState({}, '', '/users/42');
// After:  URL is /users/42 — but the page never reloaded
```

That's the entire foundation of client-side routing. Everything else is just: know which path you're on, find the matching handler, call it.

```
Traditional (Server-Side) Navigation:
  User clicks /users/42
         │
         ▼
  Browser sends HTTP GET /users/42
         │
         ▼
  Server returns new HTML page
         │
         ▼
  Browser replaces entire page ← full reload

SPA (Client-Side) Navigation:
  User clicks /users/42
         │
         ▼
  e.preventDefault()  ← stops browser
         │
         ▼
  history.pushState({}, '', '/users/42')  ← URL changes, NO request
         │
         ▼
  router._dispatch('/users/42')  ← JS runs
         │
         ▼
  app.innerHTML = renderUserDetail(ctx)  ← only content changes
```

The `navigate` method wraps this up cleanly:

```javascript
navigate(path) {
  history.pushState({}, '', path);  // ← update URL
  this._dispatch(path);              // ← call the matching handler
}
```

### The `popstate` Event — Back and Forward

`pushState` doesn't fire any event — it just updates the URL. But the browser's Back and Forward buttons *do* fire an event: `popstate`. To handle Back/Forward, listen for it and dispatch again:

```javascript
window.addEventListener('popstate', () => {
  this._dispatch(location.pathname + location.search);
});
```

`location.pathname` always reflects the current URL, so after Back fires, it holds the previous path. Re-dispatching with it renders the correct page. Without this listener, Back/Forward would show the wrong content (or navigate away from the SPA entirely).

### `_compile()` — Path Patterns to RegExp

The most technical piece: converting a route pattern like `/users/:id` into a regular expression that matches real URLs and extracts the param values.

```
Route Pattern → RegExp
──────────────────────────────────────────────────────
'/users/:id/posts/:postId'
       │
  _compile()
       │
       ▼
/^\/users\/(?<id>[^\/]+)\/posts\/(?<postId>[^\/]+)\/?$/

URL: '/users/42/posts/7'
       │
  .match(pattern)
       │
       ▼
match.groups = { id: '42', postId: '7' }
       │
       ▼
ctx.params = { id: '42', postId: '7' }
```

```javascript
_compile(path) {
  const pattern = path
    .replace(/\//g, '\\/')               // escape / for RegExp
    .replace(/:(\w+)/g, '(?<$1>[^\\/]+)') // :id → (?<id>[^/]+)
    .replace(/\*/g, '.*');               // * → match anything
  return new RegExp(`^${pattern}\\/?$`);
}
```

Step by step for `/users/:id`:
1. `replace(/\//g, '\\/')` → `\\/users\\/:id`
2. `replace(/:(\w+)/g, '(?<$1>[^\\/]+)')` → `\\/users\\/(?<id>[^\\/]+)` — the `:id` becomes a named capture group
3. Wrapped in `^...\/?$` to match the full path with an optional trailing slash

Named capture groups (`(?<id>...)`) are the key: `match.groups.id` gives you the value directly without index arithmetic.

```javascript
'/users/42'.match(/^\/users\/(?<id>[^\/]+)\/?$/)
// → match.groups = { id: '42' }
```

### `_dispatch()` — Find and Call the Handler

Dispatching loops through routes in registration order and tries to match the current path:

```javascript
_dispatch(fullPath) {
  const [pathname, search] = fullPath.split('?');
  const query = this._parseQuery(search ? '?' + search : '');

  for (const route of this.routes) {
    const match = pathname.match(route.pattern);
    if (match) {
      const ctx = {
        path:    pathname,
        params:  match.groups || {},  // ← named capture groups
        query,                         // ← parsed query string
      };
      route.handler(ctx);
      return; // stop at first match
    }
  }

  if (this._notFound) this._notFound({ path: pathname, query, params: {} });
}
```

The `ctx` object passed to every handler gives three things:
- `ctx.params` — URL segments: `/users/42` → `{ id: '42' }`
- `ctx.query`  — query string: `/blog?tag=history` → `{ tag: 'history' }`
- `ctx.path`   — the full pathname

Handler functions receive this context and decide what to render. The router is agnostic about rendering — it just calls the function.

### Query Strings — `URLSearchParams`

The browser provides `URLSearchParams` — no parsing needed:

```javascript
_parseQuery(search) {
  const params = {};
  new URLSearchParams(search).forEach((value, key) => {
    params[key] = value;
  });
  return params;
}
```

`new URLSearchParams('?tag=history&page=2')` gives an iterable object. `.forEach` walks the pairs. The result is a plain object `{ tag: 'history', page: '2' }`.

Query params let you build filterable pages that are shareable and bookmarkable. Navigating to `/blog?tag=history` renders the filtered view and the URL can be copied:

```javascript
router.navigate('/blog?tag=' + encodeURIComponent(tag));
// handler receives ctx.query.tag === 'history'
```

### Intercepting Link Clicks

HTML `<a>` tags make real HTTP requests by default. To prevent that and hand off to the router, intercept the click at the nav level:

```javascript
nav.addEventListener('click', e => {
  const link = e.target.closest('[data-route]');
  if (link) {
    e.preventDefault();          // ← stop the browser navigation
    router.navigate(link.dataset.route);  // ← let the router handle it
  }
});
```

`e.preventDefault()` cancels the default browser action. `closest('[data-route]')` walks up the DOM from the click target looking for an element with a `data-route` attribute — handles clicks on children of the link too.

### Method Chaining — Route Registration DSL

Every route registration method returns `this`, enabling a fluent API:

```javascript
router
  .get('/',          homeHandler)
  .get('/users',     usersHandler)
  .get('/users/:id', userDetailHandler)
  .on404(            notFoundHandler)
  .start();
```

This is the same `return this` pattern from Chapter 6's jQuery, applied to a different problem. The result reads like a domain-specific language — a routing table that maps paths to handlers.

---

## Guided Try It — Auth Guard with `router.before(fn)`

**The problem**: Add `router.before(fn)` — a hook that runs before every navigation. If `fn` returns `false`, cancel the navigation. This is how auth guards work: `router.before(() => isLoggedIn() || router.navigate('/login'))`.

**Hint**: The hook needs to run inside `_dispatch`, before the route handler. Look at the flow of `_dispatch`: it splits the path, parses the query, then enters the `for` loop. Where should the hook check happen — before the loop, inside the loop, or after the matching route is found?

**Step 1 — Add the hook storage and registration method.**

In the constructor, add `this._beforeHook = null`. Add a `before` method that stores the hook:

```javascript
constructor(outlet) {
  this.routes       = [];
  this._notFound    = null;
  this._outlet      = outlet;
  this._onNavigate  = null;
  this._beforeHook  = null;  // ← add this

  window.addEventListener('popstate', () => {
    this._dispatch(location.pathname + location.search);
  });
}

before(fn) {
  this._beforeHook = fn;
  return this;
}
```

**Step 2 — Check the hook in `_dispatch`, before the route loop.**

Build `ctx` first (it gives the hook useful info about the destination), then check:

```javascript
_dispatch(fullPath) {
  const [pathname, search] = fullPath.split('?');
  const query = this._parseQuery(search ? '?' + search : '');
  const ctx   = { path: pathname, query };

  // ← add this block
  if (this._beforeHook) {
    const proceed = this._beforeHook(ctx);
    if (proceed === false) return;  // cancelled — do nothing
  }

  for (const route of this.routes) {
    const match = pathname.match(route.pattern);
    if (match) {
      ctx.params       = match.groups || {};
      ctx.matchedRoute = route.original;
      route.handler(ctx);
      if (this._onNavigate) this._onNavigate(ctx);
      return;
    }
  }

  if (this._notFound) this._notFound({ path: pathname, query, params: {} });
}
```

**Usage:**

```javascript
router.before(ctx => {
  if (ctx.path.startsWith('/admin') && !isLoggedIn()) {
    router.navigate('/login');
    return false;  // cancel the original navigation
  }
  // returning undefined (not false) lets navigation proceed
});
```

**Think about it**: What would happen if the `before` hook itself calls `router.navigate('/login')`? Is there a risk of infinite recursion, and under what conditions? How would you guard against it — and what does React Router do differently to avoid this problem?

---

## Exercises

1. **Add wildcard segments**: Support `/files/*` to match any path starting with `/files/`. The matched suffix should be available as `ctx.params.wildcard`. Modify `_compile()` to capture the wildcard: replace `*` with `(?<wildcard>.*)` instead of just `.*`.

2. **Add `router.redirect(from, to)`**: A shortcut for `router.get(from, ctx => router.navigate(to))`. Also implement a `replace` variant that uses `history.replaceState` instead of `pushState` (so Back doesn't go to the redirect source).

3. **Add hash-based routing**: As an alternative to the History API, support `#`-based routing (e.g., `#/users/1`). Use `window.addEventListener('hashchange', ...)` and `location.hash` instead of `pushState`. Make the router detect which mode to use based on a constructor option: `new Router(app, { mode: 'hash' })`.

4. **Add scroll restoration**: When navigating Back, browsers normally restore scroll position. SPA routers break this because the content changes programmatically. In `_dispatch`, save `scrollY` before rendering and restore it after, keyed by path. Store the scroll map in a `Map<path, scrollY>`.

5. **Add nested routes**: Support route groups where child routes inherit a prefix. `router.group('/admin', r => { r.get('/users', adminUsersHandler); r.get('/settings', adminSettingsHandler); })` should register `/admin/users` and `/admin/settings`. Implement `group(prefix, fn)` by temporarily prepending the prefix to all `.get()` calls inside the callback.

---

## Solutions

### Exercise 1 — Wildcard segments

```javascript
_compile(path) {
  const pattern = path
    .replace(/\//g, '\\/')
    .replace(/:(\w+)/g, '(?<$1>[^\\/]+)')
    .replace(/\*/g, '(?<wildcard>.*)');   // ← capture instead of just .*
  return new RegExp(`^${pattern}\\/?$`);
}

// Usage:
router.get('/files/*', ctx => {
  console.log(ctx.params.wildcard); // 'images/photo.jpg' for /files/images/photo.jpg
});
```

### Exercise 2 — `router.redirect(from, to)`

```javascript
redirect(from, to) {
  return this.get(from, () => this.navigate(to));
}

redirectReplace(from, to) {
  return this.get(from, () => {
    history.replaceState({}, '', to);
    this._dispatch(to);
  });
}

// Usage:
router.redirect('/home', '/');         // /home → navigate to /
router.redirectReplace('/old', '/new'); // /old → replace URL with /new, no Back entry
```

### Exercise 3 — Hash-based routing

```javascript
class Router {
  constructor(outlet, { mode = 'history' } = {}) {
    this.routes  = [];
    this._outlet = outlet;
    this._mode   = mode;
    this._notFound = null;

    if (mode === 'hash') {
      window.addEventListener('hashchange', () => {
        this._dispatch(location.hash.slice(1) || '/');
      });
    } else {
      window.addEventListener('popstate', () => {
        this._dispatch(location.pathname + location.search);
      });
    }
  }

  navigate(path) {
    if (this._mode === 'hash') {
      location.hash = path;  // triggers hashchange event
    } else {
      history.pushState({}, '', path);
      this._dispatch(path);
    }
  }

  start() {
    const path = this._mode === 'hash'
      ? (location.hash.slice(1) || '/')
      : (location.pathname + location.search);
    this._dispatch(path);
    return this;
  }
}
```

### Exercise 4 — Scroll restoration

```javascript
class Router {
  constructor(outlet) {
    this._scrollMap = new Map();
    // ...existing setup
  }

  _dispatch(fullPath) {
    const [pathname] = fullPath.split('?');

    // Save current scroll before navigating away
    if (this._currentPath) {
      this._scrollMap.set(this._currentPath, window.scrollY);
    }

    // ... find and call handler ...

    // Restore scroll for this path (or go to top)
    const savedScroll = this._scrollMap.get(pathname) ?? 0;
    requestAnimationFrame(() => window.scrollTo(0, savedScroll));

    this._currentPath = pathname;
  }
}
```

`requestAnimationFrame` ensures the DOM has been updated before we try to scroll.

### Exercise 5 — Nested routes with `group()`

```javascript
group(prefix, fn) {
  const originalGet = this.get.bind(this);

  // Temporarily override .get to prepend the prefix
  this.get = (path, handler) => {
    return originalGet(prefix + path, handler);
  };

  fn(this);  // call the registration function

  // Restore original .get
  this.get = originalGet;

  return this;
}

// Usage:
router.group('/admin', r => {
  r.get('/users',    adminUsersHandler);    // registers /admin/users
  r.get('/settings', adminSettingsHandler); // registers /admin/settings
});
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| `history.pushState` | Updates URL and history without making an HTTP request |
| `popstate` event | Fires on Back/Forward — re-dispatch to re-render the correct page |
| Named capture groups | `(?<id>[^/]+)` in RegExp — `match.groups.id` gives the value |
| `_compile(path)` | Converts `:param` syntax into RegExp named capture groups |
| `URLSearchParams` | Browser built-in that parses `?key=value&key2=value2` |
| `e.preventDefault()` | Stops browser from following the `<a href>` — hands control to JS |
| `e.target.closest()` | Walks up the DOM tree to find the nearest matching ancestor |
| Route registration order | First match wins — put specific routes before wildcards |
| `ctx` object | Every handler receives `{ params, query, path }` — the routing context |
| Method chaining | `return this` on every method creates a fluent registration DSL |

---

## Building with Claude

Bad prompt:
> "How does React Router work?"

Good prompt:
> "I've built a client-side router using `history.pushState` and `popstate`. Routes are compiled from strings like `/users/:id` into RegExp named capture groups using `(?<id>[^\\/]+)`. Navigation calls `pushState` then `_dispatch`. I want to add auth guards — a `before` hook where I can call `router.before(() => isLoggedIn() || router.navigate('/login'))`. Where in `_dispatch` should this hook run — before finding the matching route, or after? And what should happen if the hook calls `router.navigate('/login')` — will that cause infinite recursion if `/login` also has a `before` hook?"

The prompt describes the architecture clearly, proposes the solution, and asks a pointed question about a subtle bug that would only occur in practice.

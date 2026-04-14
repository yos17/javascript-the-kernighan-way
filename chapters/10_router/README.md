# Chapter 10 — Build Your Own Router

React Router. Vue Router. Svelte's router. Every single-page app framework ships one. But what does a router actually do? It maps URL paths to JavaScript functions — no server involved. When you click a link, instead of the browser requesting a new page, the router intercepts that click, updates the URL using `history.pushState`, and calls the matching handler to swap out the page content. In this chapter you build exactly that — a 60-line router that handles named params, query strings, browser back/forward, and 404 pages.

---

## The Problem

You have a multi-page app — a home page, a user list, user detail pages with dynamic IDs, a blog with tag filtering. In a traditional website, each URL triggers a server request. In a single-page app, JavaScript handles everything: the URL changes but no request is made. Build the router and you'll understand how React Router's `<Route path="/users/:id">` works at the machine level.

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

## Try It

1. **Add a `before` hook**: A function registered with `router.before(fn)` that runs before every dispatch. If `fn` returns `false`, the navigation is cancelled. Useful for auth guards: `router.before(() => isLoggedIn() || router.navigate('/login'))`.

2. **Add route aliases**: `router.alias('/home', '/')` — visiting `/home` dispatches the `/` handler without changing the URL further. Implement by adding a pre-dispatch check that rewrites paths.

3. **Add a loading state**: Between `navigate()` being called and the handler completing, show a loading indicator in the outlet. This matters when handlers are async (fetching data before rendering).

4. **Persist query state in links**: When navigating with `router.navigate`, preserve existing query params by default unless explicitly cleared. Useful for maintaining a `?theme=dark` param across page changes.

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

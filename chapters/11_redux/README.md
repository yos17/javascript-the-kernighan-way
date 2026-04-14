# Chapter 11 — Build Your Own Redux

Redux is one of the most influential JavaScript libraries ever written — not because of what it does (it's about 100 lines of code), but because of what it *enforces*: state lives in one place, state only changes through actions, and the same action always produces the same state. These three rules — single source of truth, immutability, pure reducers — solve the most common source of bugs in large apps: state that changes unexpectedly from multiple places. Build Redux from scratch and you'll understand why these constraints exist, not just how to follow them.

---

## The Problem

You have a shopping cart. The cart total affects the header badge, the cart icon, the checkout button, and a summary panel. Whenever an item is added or removed, all four need to update. With scattered `let cart = []` variables, keeping them in sync is a nightmare. Redux's insight: put all state in one object, enforce that it only changes through explicit actions, and notify every subscriber after every change. One source of truth, infinite observers.

---

## The Complete Program

`redux.html` — open it to see a shopping cart powered by a Redux store with `combineReducers`, middleware, and a live DevTools panel showing every dispatched action and the current state tree.

```javascript
// ═══════════════════════════════════════════════════════════════════════
//  THE LIBRARY: MiniRedux
// ═══════════════════════════════════════════════════════════════════════

function createStore(reducer, initialState, enhancer) {
  if (typeof enhancer !== 'undefined') {
    return enhancer(createStore)(reducer, initialState);
  }

  let state         = initialState;
  let listeners     = [];
  let isDispatching = false;

  function getState() {
    if (isDispatching) throw new Error('Do not call getState() inside a reducer');
    return state;
  }

  function subscribe(listener) {
    listeners.push(listener);
    let active = true;
    return function unsubscribe() {
      if (!active) return;
      active = false;
      listeners.splice(listeners.indexOf(listener), 1);
    };
  }

  function dispatch(action) {
    if (typeof action !== 'object' || action === null)
      throw new TypeError('Actions must be plain objects');
    if (typeof action.type !== 'string')
      throw new TypeError('Actions must have a string `type` property');
    if (isDispatching)
      throw new Error('Reducers may not dispatch actions');

    try {
      isDispatching = true;
      state = reducer(state, action);
    } finally {
      isDispatching = false;
    }

    for (const listener of [...listeners]) listener();
    return action;
  }

  dispatch({ type: '@@MINIREDUX/INIT' });
  return { getState, subscribe, dispatch };
}

// ─── combineReducers ──────────────────────────────────────────────────

function combineReducers(reducers) {
  const keys = Object.keys(reducers);
  return function combination(state = {}, action) {
    let hasChanged = false;
    const nextState = {};
    for (const key of keys) {
      const prevSlice = state[key];
      const nextSlice = reducers[key](prevSlice, action);
      nextState[key]  = nextSlice;
      hasChanged = hasChanged || nextSlice !== prevSlice;
    }
    return hasChanged ? nextState : state;
  };
}

// ─── applyMiddleware ──────────────────────────────────────────────────

function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, initialState) => {
    const store = createStore(reducer, initialState);
    const middlewareAPI = {
      getState: store.getState,
      dispatch: action => dispatch(action),
    };
    const chain    = middlewares.map(mw => mw(middlewareAPI));
    const dispatch = chain.reduceRight((next, mw) => mw(next), store.dispatch);
    return { ...store, dispatch };
  };
}

// ─── Reducers ─────────────────────────────────────────────────────────

function cartReducer(state = [], action) {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existing = state.find(i => i.id === action.payload.id);
      if (existing) {
        return state.map(i => i.id === action.payload.id
          ? { ...i, qty: i.qty + 1 } : i);
      }
      return [...state, { ...action.payload, qty: 1 }];
    }
    case 'REMOVE_ITEM':
      return state.filter(i => i.id !== action.payload.id);
    case 'INCREMENT_QTY':
      return state.map(i => i.id === action.payload.id ? { ...i, qty: i.qty + 1 } : i);
    case 'DECREMENT_QTY':
      return state.map(i => i.id === action.payload.id ? { ...i, qty: i.qty - 1 } : i)
                  .filter(i => i.qty > 0);
    case 'CLEAR_CART':
      return [];
    default:
      return state;
  }
}

// ─── Store ────────────────────────────────────────────────────────────

const loggerMiddleware = store => next => action => {
  console.log('dispatching', action.type);
  const result = next(action);
  console.log('next state', store.getState());
  return result;
};

const store = createStore(
  combineReducers({ cart: cartReducer }),
  undefined,
  applyMiddleware(loggerMiddleware)
);

// ─── Subscribe ────────────────────────────────────────────────────────

store.subscribe(() => renderCart(store.getState().cart));
```

---

## Walkthrough

### The Single Source of Truth

Redux puts all application state into one JavaScript object — the store. Every piece of state your app needs lives there. This sounds obvious but has a profound consequence: if you want to know anything about your app's current state, there's exactly one place to look.

```javascript
store.getState();
// → {
//     cart: [{ id: 1, name: '...', qty: 2, price: 24.99 }],
//     ui:   { totalDispatches: 7, checkoutDone: false }
//   }
```

One store. No hidden state in components. No variables scattered across modules.

### Actions — The Only Way to Change State

State never changes directly. You can't do `store.cart.push(item)`. The only way to change state is to `dispatch` an **action** — a plain object with a `type` property:

```javascript
store.dispatch({ type: 'ADD_ITEM', payload: { id: 1, name: 'Book', price: 24.99 } });
store.dispatch({ type: 'REMOVE_ITEM', payload: { id: 1 } });
store.dispatch({ type: 'CLEAR_CART' });
```

This constraint is what makes Redux debuggable. Every change is an explicit event with a name. The action log in the DevTools panel shows every dispatch, in order — like a history of everything that happened to your state.

### Reducers — Pure Functions

A **reducer** is a pure function that takes the current state and an action, and returns the next state:

```javascript
function cartReducer(state = [], action) {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existing = state.find(i => i.id === action.payload.id);
      if (existing) {
        return state.map(i =>
          i.id === action.payload.id
            ? { ...item, qty: i.qty + 1 }  // new object — immutable update
            : i
        );
      }
      return [...state, { ...action.payload, qty: 1 }]; // new array
    }
    default:
      return state; // always handle the default case
  }
}
```

Three things make a reducer pure:
1. **Same input → same output**: given the same state and action, it always returns the same result.
2. **No side effects**: no API calls, no `Date.now()`, no `Math.random()`.
3. **No mutation**: `state.push()` or `state[0].qty++` are forbidden — always return new objects.

The spread operator (`...`) is the primary tool for immutable updates:

```javascript
// Wrong — mutates state
state.push(newItem);

// Correct — returns new array
return [...state, newItem];

// Wrong — mutates object
item.qty++;

// Correct — returns new object
return { ...item, qty: item.qty + 1 };
```

Immutability makes change detection trivially cheap: `nextSlice !== prevSlice` is a reference comparison — O(1), no deep comparison needed.

### `createStore` — The Core

```javascript
function createStore(reducer, initialState, enhancer) {
  if (typeof enhancer !== 'undefined') {
    return enhancer(createStore)(reducer, initialState);
  }

  let state = initialState;
  let listeners = [];
  let isDispatching = false;

  function dispatch(action) {
    // ...validate action...
    try {
      isDispatching = true;
      state = reducer(state, action);  // ← pure function computes next state
    } finally {
      isDispatching = false;
    }
    for (const listener of [...listeners]) listener();  // ← notify subscribers
    return action;
  }

  dispatch({ type: '@@MINIREDUX/INIT' });  // ← seed initial state from reducer defaults
  return { getState, subscribe, dispatch };
}
```

The `isDispatching` flag guards against a reducer calling `store.dispatch()` — which would cause infinite recursion. The `@@MINIREDUX/INIT` dispatch on creation seeds the state: every reducer's `default` case returns its initial state (e.g., `state = []`), so the full state tree is populated before any user action.

### `subscribe()` — The Pub/Sub Pattern

`subscribe` implements the **observer pattern**: register a listener, get notified on every change.

```javascript
function subscribe(listener) {
  listeners.push(listener);
  let active = true;
  return function unsubscribe() {  // ← returns cleanup function
    if (!active) return;
    active = false;
    listeners.splice(listeners.indexOf(listener), 1);
  };
}
```

The return value — an `unsubscribe` function — is the cleanup mechanism. This is a common JavaScript pattern: a function that reverses whatever the outer function did.

```javascript
const unsubscribe = store.subscribe(() => renderCart(store.getState()));

// Later, when the component unmounts:
unsubscribe();
```

In the demo, two subscribers are registered: one renders the cart, one updates the DevTools state inspector. Both run after every dispatch.

### `combineReducers` — Splitting the State Tree

Large apps have complex state. `combineReducers` lets you split the root reducer into smaller functions, each managing one slice:

```javascript
const rootReducer = combineReducers({
  cart: cartReducer,   // manages state.cart
  ui:   uiReducer,     // manages state.ui
});
```

The combined reducer calls each slice reducer with the relevant portion of state and the same action:

```javascript
function combination(state = {}, action) {
  let hasChanged = false;
  const nextState = {};
  for (const key of keys) {
    const prevSlice = state[key];
    const nextSlice = reducers[key](prevSlice, action);
    nextState[key]  = nextSlice;
    hasChanged = hasChanged || nextSlice !== prevSlice; // reference check
  }
  return hasChanged ? nextState : state; // return same object if nothing changed
}
```

Every action is sent to every reducer — most reducers just return `state` for actions they don't handle. This is intentional: any reducer can respond to any action.

### Middleware — Wrapping Dispatch

Middleware intercepts every dispatch. The pattern uses function composition to build a chain:

```javascript
const loggerMiddleware = store => next => action => {
  console.log('dispatching', action.type);
  const result = next(action);  // ← call the next middleware (or real dispatch)
  console.log('next state', store.getState());
  return result;
};
```

This triple-arrow function is a curried function that takes three things:
1. `store` — the store API (`getState`, `dispatch`)
2. `next` — the next middleware in the chain (or the real `dispatch`)
3. `action` — the action being dispatched

The key line is `next(action)`: it passes the action down the chain. The middleware can run code before, after, or instead of calling `next`. The logger middleware runs before and after — logging the action type before dispatch and the new state after.

`applyMiddleware` composes the chain using `reduceRight`, so middlewares wrap each other like onion layers. The innermost layer is the real `store.dispatch`.

---

## Try It

1. **Add a `thunk` middleware**: Allows dispatching functions as actions. If the action is a function, call it with `dispatch` and `getState`. Otherwise, pass to `next`. This enables async action creators.

2. **Add undo/redo**: Wrap the root reducer in an `undoable(reducer)` higher-order function that stores past states in an array. Dispatch `{ type: 'UNDO' }` to roll back one step and `{ type: 'REDO' }` to go forward.

3. **Add a devtools middleware**: Serialises every action and state to `sessionStorage`. On page load, replay them to restore the session. This is exactly what the Redux DevTools Chrome extension does.

4. **Connect to the DOM automatically**: Write a `connect(selector, render)` function that subscribes to the store and calls `render(selector(store.getState()))` whenever the selected slice changes. Only re-render when the slice changes, not on every dispatch.

---

## Exercises

1. **Implement `bindActionCreators(actionCreators, dispatch)`**: Takes an object of action creator functions and returns a new object where each function is pre-wrapped with `dispatch`. `boundCreators.addItem(product)` should call `dispatch(addItem(product))` automatically.

2. **Implement a `createSelector(inputFn, computeFn)` memoised selector**: `inputFn` reads from state; `computeFn` derives a value from that input. The result is cached and only recomputed when the input changes. This is the core of Reselect — the memoisation library used with Redux.

3. **Add `replaceReducer(nextReducer)`**: Add this method to the store. It replaces the root reducer and re-initialises with an `@@REPLACE` action. Used for code splitting — loading reducers lazily as features are loaded.

4. **Implement `createSlice(config)`**: A Redux Toolkit-style helper that takes `{ name, initialState, reducers }` and returns `{ actions, reducer }`. Each key in `reducers` becomes both a reducer case and an auto-generated action creator. The action type is `name/reducerKey`.

5. **Add optimistic updates**: Add a middleware that marks certain actions as "optimistic". Before dispatching to the real reducer, apply the state change immediately. If a subsequent `ROLLBACK` action arrives with the same ID, undo the optimistic change by restoring a snapshot. Implement using a `Map` of pending optimistic changes.

---

## Solutions

### Exercise 1 — `bindActionCreators`

```javascript
function bindActionCreators(actionCreators, dispatch) {
  const bound = {};
  for (const [key, creator] of Object.entries(actionCreators)) {
    bound[key] = (...args) => dispatch(creator(...args));
  }
  return bound;
}

// Usage:
const actions = bindActionCreators(
  { addItem, removeItem, incrementQty, decrementQty, clearCart },
  store.dispatch
);

// Now instead of store.dispatch(addItem(product)):
actions.addItem(product);
actions.removeItem(id);
```

### Exercise 2 — `createSelector`

```javascript
function createSelector(inputFn, computeFn) {
  let lastInput  = undefined;
  let lastResult = undefined;
  let called     = false;

  return function(state) {
    const input = inputFn(state);
    if (called && input === lastInput) {
      return lastResult; // cache hit — return memoised value
    }
    lastInput  = input;
    lastResult = computeFn(input);
    called     = true;
    return lastResult;
  };
}

// Usage:
const selectCart      = state => state.cart;
const selectCartTotal = createSelector(
  selectCart,
  cart => cart.reduce((sum, item) => sum + item.price * item.qty, 0)
);

// Only recomputes when state.cart changes reference:
selectCartTotal(store.getState()); // computed
selectCartTotal(store.getState()); // cached — same cart reference
```

### Exercise 3 — `replaceReducer`

```javascript
// Add to createStore's returned object:
function replaceReducer(nextReducer) {
  reducer = nextReducer;
  dispatch({ type: '@@MINIREDUX/REPLACE' });
}

return { getState, subscribe, dispatch, replaceReducer };

// Usage (code splitting):
import(/* webpackChunkName: "admin" */ './adminReducer').then(({ adminReducer }) => {
  const newRootReducer = combineReducers({ cart: cartReducer, admin: adminReducer });
  store.replaceReducer(newRootReducer);
});
```

### Exercise 4 — `createSlice`

```javascript
function createSlice({ name, initialState, reducers }) {
  const actions = {};
  const cases   = {};

  for (const [key, reducerFn] of Object.entries(reducers)) {
    const type = `${name}/${key}`;
    actions[key] = (payload) => ({ type, payload });
    cases[type]  = reducerFn;
  }

  function reducer(state = initialState, action) {
    const handler = cases[action.type];
    return handler ? handler(state, action) : state;
  }

  return { actions, reducer };
}

// Usage:
const cartSlice = createSlice({
  name: 'cart',
  initialState: [],
  reducers: {
    addItem:      (state, action) => [...state, { ...action.payload, qty: 1 }],
    removeItem:   (state, action) => state.filter(i => i.id !== action.payload.id),
    clearCart:    () => [],
  },
});

const { addItem, removeItem, clearCart } = cartSlice.actions;
// addItem({ id: 1 }) → { type: 'cart/addItem', payload: { id: 1 } }
```

### Exercise 5 — Optimistic updates

```javascript
const optimisticMiddleware = store => next => action => {
  if (!action.optimistic) return next(action);

  // Store snapshot before optimistic update
  const snapshot = store.getState();
  const { id } = action.optimistic;

  // Track this optimistic change
  _optimisticSnapshots.set(id, snapshot);

  // Apply immediately
  return next(action);
};

// Handle rollback in reducer or middleware:
const rollbackMiddleware = store => next => action => {
  if (action.type !== 'ROLLBACK') return next(action);

  const snapshot = _optimisticSnapshots.get(action.id);
  if (snapshot) {
    // Force-set state to the snapshot
    // (requires a special action that bypasses normal reducer logic)
    _optimisticSnapshots.delete(action.id);
    next({ type: '@@RESTORE_SNAPSHOT', snapshot });
  }
};

const _optimisticSnapshots = new Map();
```

---

## What You Learned

| Concept | Key point |
|---------|-----------|
| Single source of truth | All state in one object — one place to read, one place to update |
| Actions | Plain objects with `type` — the only way to describe a state change |
| Reducers | Pure functions: `(state, action) → newState` — no mutation, no side effects |
| `dispatch(action)` | Sends action through middleware chain, then to reducer, then notifies subscribers |
| `subscribe(listener)` | Pub/sub — returns an `unsubscribe` function for cleanup |
| Immutable updates | Spread operator `{ ...obj, key: newVal }` — never mutate state directly |
| `combineReducers` | Splits root reducer into slices; each manages one key in the state tree |
| Reference equality | `nextSlice !== prevSlice` — O(1) change detection; requires immutability |
| Middleware | `store => next => action => { ... }` — wraps dispatch; composes with `reduceRight` |
| Action creators | Functions that return action objects — named, testable, refactorable |
| `@@INIT` dispatch | Seeds initial state from reducer `default` cases on store creation |

---

## Building with Claude

Bad prompt:
> "Explain Redux."

Good prompt:
> "I've built a Redux implementation with `createStore`, `combineReducers`, and `applyMiddleware`. My middleware uses the triple-arrow curried pattern: `store => next => action => { next(action) }`. I want to add a `thunk` middleware that allows dispatching functions instead of plain objects. My understanding is: if `action` is a function, call it with `(dispatch, getState)` instead of passing to `next`. If it's a plain object, pass to `next` as usual. Is this correct? And should the function action return anything — does `store.dispatch()` need to return the result?"

The prompt proves deep understanding of the existing code, proposes the solution precisely, and asks the specific follow-up that catches a real gotcha in the implementation.

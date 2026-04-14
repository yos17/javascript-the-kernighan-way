# Chapter 10: Async and APIs

## Why Asynchronous Programming Exists

JavaScript runs on a single thread — one piece of code at a time. If your code makes a network request that takes two seconds, and it blocks while waiting, the entire page freezes for two seconds. No animations. No button responses. Nothing.

This is the fundamental problem async programming solves: **how do you wait for something slow without stopping everything else?**

The answer: don't wait. Start the operation, register a callback to run when it finishes, and let JavaScript keep doing other things. When the operation completes, the callback runs.

Here's the evolution of how JavaScript has handled this:

**v1 — Callbacks (1995–2015):** Pass a function to run when done. Works, but nests badly ("callback hell").

**v2 — Promises (2015):** Return an object representing a future value. Chain with `.then()`. Cleaner, but still chained.

**v3 — async/await (2017):** Write async code that *looks* synchronous. `await` pauses only the current function, not the thread. This is what we use.

### The Promise State Machine

A Promise is an object that represents a value not yet available:

```
                    ┌─────────────┐
                    │   PENDING   │  ← operation in progress
                    └──────┬──────┘
                           │
               ┌───────────┴───────────┐
               │ success               │ failure
               ▼                       ▼
        ┌────────────┐          ┌────────────┐
        │ FULFILLED  │          │  REJECTED  │
        │ (result)   │          │  (error)   │
        └────────────┘          └────────────┘

Once a Promise settles (fulfills or rejects), it never changes state.
```

`async/await` wraps this: `await somePromise` pauses the function until the Promise fulfills, then returns the result. If it rejects, the error propagates (and `try/catch` catches it).

## The Program: Weather Dashboard

A single-page application that fetches weather data from a public API, displays current conditions and a 3-day forecast, stores search history, and handles all the things that can go wrong: no network, bad city name, API down.

[Open weather_dashboard.html](weather_dashboard.html) to try it.

## How It Works

### The Fetch Pattern

`fetch()` makes an HTTP request and returns a Promise:

```javascript
async function fetchWeather(city) {
  try {
    showLoading();

    const response = await fetch(`https://wttr.in/${city}?format=j1`);

    // fetch() doesn't throw for 404/500 — you must check response.ok
    if (!response.ok) {
      throw new Error(`City not found (${response.status})`);
    }

    const data = await response.json();  // parse JSON — also async!
    displayWeather(data, city);

  } catch (error) {
    showError(error.message);
  } finally {
    hideLoading();  // ALWAYS runs, success or failure
  }
}
```

**Critical detail:** `fetch()` only rejects (throws) on network failure. A `404 Not Found` or `500 Server Error` is still a "successful" fetch — the server responded, just with bad news. You must check `response.ok` (true for 200–299 status codes) and throw yourself if needed.

The flow:
```
User clicks Search
       ↓
showLoading() — spinner appears
       ↓
await fetch(url) — HTTP request begins; JS event loop is free
       ↓
[2 seconds pass; other JS runs; spinner animates]
       ↓
Response arrives — function resumes here
       ↓
await response.json() — parses body; very fast but still async
       ↓
displayWeather() — updates DOM with data
       ↓
hideLoading() — spinner disappears (runs via finally)
```

### async/await in Detail

`async` marks a function as asynchronous. Inside it, `await` pauses execution at that line until the Promise resolves:

```javascript
// This function returns a Promise<string>, not a string directly
async function getCity() {
  const response = await fetch('/api/location');
  const data = await response.json();
  return data.city;  // returned as a resolved Promise
}

// Calling it — it returns immediately, city arrives later
getCity().then(city => console.log(city));

// Or inside another async function:
const city = await getCity();
console.log(city);
```

**What does "pauses execution" mean?** Only *this function* pauses. JavaScript itself keeps running — processing other events, updating animations. When the awaited Promise settles, this function resumes from the `await` line.

### try/catch/finally for Async

With async/await, error handling looks like synchronous code:

```javascript
try {
  const data = await riskyOperation();  // might throw
  doSomethingWith(data);
} catch (error) {
  console.error('Something went wrong:', error.message);
} finally {
  cleanup();  // runs whether success or failure
}
```

Three kinds of errors the weather dashboard handles:
- **Network error**: No internet. `fetch()` rejects. Caught in `catch`.
- **HTTP error**: City not found (404). `response.ok` is false. We throw explicitly. Caught in `catch`.
- **Parse error**: API returns malformed JSON. `response.json()` rejects. Caught in `catch`.

All three end up in the same `catch` block with a user-friendly message. The `finally` block always hides the spinner, regardless of which path was taken.

### Working with Nested JSON

The wttr.in API returns deeply nested data. The shape looks like:

```
data
├── current_condition (array with one item)
│   └── [0]
│       ├── temp_C: "15"
│       ├── temp_F: "59"
│       ├── humidity: "76"
│       ├── weatherDesc (array)
│       │   └── [0].value: "Partly cloudy"
│       └── windspeedKmph: "18"
└── forecast (array with three items)
    ├── [0]  ← today
    │   ├── date: "2025-01-15"
    │   └── hourly (array of 8 items)
    │       └── ...
    ├── [1]  ← tomorrow
    └── [2]  ← day after
```

Destructuring makes this readable:

```javascript
const { current_condition, forecast } = data;
const {
  temp_C, temp_F, humidity,
  windspeedKmph, windDirection16Point,
  FeelsLikeC, FeelsLikeF
} = current_condition[0];

const weatherDesc = current_condition[0].weatherDesc[0].value;
```

Without destructuring, every variable access would be `data.current_condition[0].temp_C` — verbose and error-prone.

### Three UI States

Good async UIs always have three states:

```
IDLE → user hasn't searched yet → show placeholder
  ↓ user clicks Search
LOADING → request in progress → show spinner
  ↓ request finishes
SUCCESS or ERROR → show weather data OR error message
```

The dashboard manages this with CSS classes:

```javascript
function showLoading() {
  loading.classList.add('show');
  weatherContent.classList.remove('show');
  errorMessage.classList.remove('show');
}

function showWeather() {
  loading.classList.remove('show');
  weatherContent.classList.add('show');
  errorMessage.classList.remove('show');
}

function showError(message) {
  loading.classList.remove('show');
  weatherContent.classList.remove('show');
  errorMessage.textContent = message;
  errorMessage.classList.add('show');
}
```

At any moment, exactly one of `loading`, `weatherContent`, `errorMessage` has the `show` class. The CSS makes them visible or hidden accordingly.

---

## Guided Exercises

### Exercise 1: Celsius / Fahrenheit Toggle

**The Challenge:** Add a toggle button that switches between showing only °C, only °F, or both. Remember the preference in `localStorage` so it persists between page loads.

**Where to start:** Think about the state you need. What are the possible modes? How many are there?

*(Three modes: both, C only, F only. You could store a string like `'both'`, `'C'`, or `'F'`.)*

---

**Step 1: Define the state and add the button.**

```javascript
let tempMode = localStorage.getItem('tempMode') || 'both';
// 'both' | 'C' | 'F'
```

Add a button:
```html
<button id="tempToggle">°C / °F</button>
```

---

**Step 2: Cycle through modes on click.**

```javascript
document.getElementById('tempToggle').addEventListener('click', () => {
  const modes = ['both', 'C', 'F'];
  const idx = modes.indexOf(tempMode);
  tempMode = modes[(idx + 1) % modes.length];
  localStorage.setItem('tempMode', tempMode);
  applyTempMode();
});
```

**Why the modulo `% modes.length`?** When you reach the last mode (index 2) and add 1, you get 3. `3 % 3 = 0`, wrapping back to the first mode. This is the "cycling through options" pattern.

---

**Step 3: Apply the mode to the DOM.**

After displaying weather, call this function:

```javascript
function applyTempMode() {
  const showC = tempMode === 'both' || tempMode === 'C';
  const showF = tempMode === 'both' || tempMode === 'F';

  document.querySelectorAll('.temp-c').forEach(el => {
    el.style.display = showC ? '' : 'none';
  });
  document.querySelectorAll('.temp-f').forEach(el => {
    el.style.display = showF ? '' : 'none';
  });
}
```

This requires your temperature display elements to have `.temp-c` and `.temp-f` classes. Update your `displayWeather()` function to add these classes to the temperature spans.

**Think about it:** Why call `applyTempMode()` after displaying weather AND when the button is clicked? *(Because displaying weather resets the DOM to show both temperatures. Clicking toggle needs to re-apply the preference.)*

---

### Exercise 2: Multiple Cities Comparison

**The Challenge:** Let users add up to 3 cities. Display them side by side for comparison. Each city card shows current temp, weather description, and humidity. An "×" button removes a city from the comparison.

**Where to start:** You need a collection of city data. What data structure? What happens when a city is added or removed?

*(An array of `{ city, data }` objects. When it changes, re-render the comparison cards.)*

---

**Step 1: Add the state and render function.**

```javascript
let comparisons = [];  // [{ city: 'London', data: {...} }, ...]

function renderComparisons() {
  const container = document.getElementById('comparisons');
  container.innerHTML = '';

  for (const { city, data } of comparisons) {
    const { current_condition } = data;
    const { temp_C, humidity } = current_condition[0];
    const desc = current_condition[0].weatherDesc[0].value;

    const card = document.createElement('div');
    card.className = 'comparison-card';
    card.innerHTML = `
      <h3>${city} <button class="remove-btn" data-city="${city}">×</button></h3>
      <div>${temp_C}°C</div>
      <div>${desc}</div>
      <div>Humidity: ${humidity}%</div>
    `;
    container.appendChild(card);
  }
}
```

---

**Step 2: Add a city to comparisons.**

When a search succeeds and comparisons.length < 3:

```javascript
function addToComparison(city, data) {
  // Don't add duplicates
  if (comparisons.some(c => c.city.toLowerCase() === city.toLowerCase())) return;
  if (comparisons.length >= 3) comparisons.shift();  // remove oldest if full
  comparisons.push({ city, data });
  renderComparisons();
}
```

---

**Step 3: Handle the remove button.**

Using event delegation on the container:

```javascript
document.getElementById('comparisons').addEventListener('click', e => {
  if (e.target.classList.contains('remove-btn')) {
    const city = e.target.dataset.city;
    comparisons = comparisons.filter(c => c.city !== city);
    renderComparisons();
  }
});
```

**The complete picture:** The fetch result (an API response) gets stored in your app's state array. The render function reads from state and produces DOM. Removing a city filters the state array and re-renders. This is the same data → render → events cycle from Chapter 6, now working with async data.

---

### Exercise 3: Auto-Detect Location

**The Challenge:** On page load, ask for the user's location (via the browser Geolocation API). If granted, automatically fetch weather for their location. If denied, show a normal empty search box.

**Where to start:** The Geolocation API is asynchronous (like fetch) but uses callbacks, not Promises. You'll need to wrap it or use it directly.

---

**Step 1: Request location on load.**

```javascript
document.addEventListener('DOMContentLoaded', () => {
  if (!navigator.geolocation) return;  // not supported

  navigator.geolocation.getCurrentPosition(
    onLocationSuccess,   // called if user grants permission
    onLocationError      // called if user denies or error occurs
  );
});
```

---

**Step 2: Convert coordinates to a city name.**

```javascript
async function onLocationSuccess(position) {
  const { latitude, longitude } = position.coords;

  try {
    // wttr.in accepts lat,lon directly
    const city = `${latitude.toFixed(2)},${longitude.toFixed(2)}`;
    await fetchWeather(city);
  } catch (error) {
    // Silently fail — just show the empty search box
    console.log('Auto-location failed:', error.message);
  }
}

function onLocationError(error) {
  // User denied or error — that's fine, search box is already there
  console.log('Geolocation not available:', error.message);
}
```

**Think about it:** Notice we don't show an error to the user if geolocation fails — we just silently show the empty search state. Why? Because geolocation is a *bonus* feature. If it fails, the user still has the search box. Showing an error for a feature they didn't explicitly request would be confusing.

**Important:** The `try/catch` around `fetchWeather()` catches failures from the *weather fetch*, not from geolocation. The geolocation errors are handled by `onLocationError`. Each async operation has its own error boundary.

---

## What You Learned

| Concept | How We Used It | Real-World Use |
|---------|---------------|----------------|
| `async/await` | `fetchWeather` is non-blocking; UI stays responsive | Standard in every modern JS app |
| `fetch()` | HTTP GET requests to wttr.in | REST APIs, GraphQL, file uploads |
| `response.ok` | Check for 404/500 before reading body | Always check this — fetch doesn't throw on bad status |
| `response.json()` | Parse response body as JSON | Virtually all web APIs return JSON |
| `try/catch/finally` | Handle network/HTTP/parse errors; always hide spinner | Every async operation needs error handling |
| Destructuring | Extract nested API data cleanly | API response shapes are often deeply nested |
| Loading states | 3-state UI: idle/loading/done | Every data-fetching UI in production |
| `localStorage` | Search history without a server | User preferences, cached data, settings |

### Real-World Connections

- **React Query / TanStack Query** is a library that automates the loading/error/success state pattern you built manually here. It also caches results and refetches on focus. The underlying fetch is the same.
- **SWR** (stale-while-revalidate) is another React data-fetching library. Same concept: manage async state so you don't have to.
- **Axios** is a popular fetch alternative with slightly nicer defaults (throws on non-2xx, handles JSON automatically). Under the hood it uses XMLHttpRequest or fetch.
- **GraphQL clients** (Apollo, urql) wrap fetch with a query language, but the async pattern is identical.
- The **three-state UI** (loading/success/error) is so universal that React 18 added `<Suspense>` to handle it declaratively.

## Building with Claude

- "Add a 7-day forecast with daily high/low temperatures and a weather icon for each day."
- "Add a weather alert banner that appears when conditions include 'storm', 'thunderstorm', or 'blizzard'."
- "Cache the last-fetched weather for each city in localStorage so the app shows the cached data instantly, then updates in the background."
- "Add a sunrise/sunset display using the weather API data, with a simple sun arc visualization using Canvas."

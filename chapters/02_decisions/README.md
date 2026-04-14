# Chapter 2: Decisions

## Why Decision Logic Matters

Programs that do the same thing every time aren't very useful. A game where every choice leads to the same ending isn't a game — it's a story. An app that shows the same content regardless of who's using it isn't personalized — it's a flyer.

Decision logic — `if/else`, comparisons, logical operators — is how programs respond to context. It's also the foundation of more sophisticated patterns you'll see throughout this book: the state machine in Chapter 7 (the calculator knows whether you just pressed an operator or a digit), the game logic in Chapter 9 (is the ball above the paddle or below?), and the auth flow in Chapter 12 (which screen is currently showing?).

## The Program: Crystal Tower Adventure

A branching story engine with three levels of choices and four different endings. The entire story is stored as a data object — scenes point to other scenes via string keys. This architecture means adding new paths requires zero logic changes; just add new data.

Open `adventure.html` and play through at least two different paths before reading the code.

## How It Works

### if/else: The Fundamental Branch

```javascript
if (scene.isEnding) {
  cardClass += scene.type === 'victory' ? ' outcome-victory' : ' outcome-defeat';
}
```

`if` evaluates a condition. If true, the block runs. If false, it doesn't. `else` provides an alternative.

The condition can be any expression that produces `true` or `false`:

```javascript
if (score > 100) { showCelebration(); }
if (username === '') { showError('Name required'); }
if (inventory.includes('water')) { showWaterOption(); }
```

**Coming up:** In Chapter 7, `if` statements inside event handlers decide what the calculator should do based on which button was clicked. In Chapter 9, `if` checks whether the ball is past the bottom edge. Decision logic is the skeleton of every interactive program.

### Comparison Operators

```javascript
scene.type === 'victory'    // strictly equal (same type AND same value)
scene.choices.length === 0  // no choices = it's an ending
score > 100                 // greater than
health <= 0                 // less than or equal
```

**Why `===` instead of `==`?**

JavaScript's `==` (loose equality) does automatic type conversion, which leads to surprises:
```javascript
0 == false    // → true (bad!)
'' == false   // → true (bad!)
0 === false   // → false (correct)
```

Always use `===` and `!==`. The rare case for `==` isn't worth the confusion.

### Logical Operators: Combining Conditions

Three operators for building compound conditions:

```javascript
// AND (&&): both conditions must be true
if (hasWater && knowsWeakness) {
  enableSpecialAttack();
}

// OR (||): at least one must be true
if (lives === 0 || timeUp) {
  gameOver();
}

// NOT (!): flip true to false, or false to true
if (!scene.isEnding) {
  showChoiceButtons();
}
```

**Short-circuit evaluation:** `&&` stops as soon as it hits a false. `||` stops as soon as it hits a true. This matters for performance with expensive operations:

```javascript
// Only runs expensiveCheck() if user is logged in
if (isLoggedIn && expensiveCheck()) { ... }
```

**Coming up:** In Chapter 9, game loop conditions combine multiple checks: `if (ball.y > canvas.height && lives > 0)`. In Chapter 10, async error handling uses `if (!response.ok)`.

### The Ternary Operator: Inline if/else

```javascript
const label = score > 100 ? 'Expert' : 'Beginner';
//            ──────────── ↑ true    ↑ false
```

Ternary is `condition ? valueIfTrue : valueIfFalse`. It produces a *value*, unlike if/else which runs *statements*. Use it for simple two-way choices, not for complex logic.

### switch: Multiple Paths from One Value

When a single value determines many different behaviors:

```javascript
switch (scene.type) {
  case 'victory':
    cardClass += ' outcome-victory';
    break;
  case 'defeat':
    cardClass += ' outcome-defeat';
    break;
  case 'neutral':
    // no extra class
    break;
  default:
    console.warn('Unknown scene type:', scene.type);
}
```

**Why `break`?** Without it, execution "falls through" to the next case. This is almost never what you want. Always `break`, unless you intentionally want fall-through (rare).

**When switch vs. if/else?** Switch is cleaner when all branches depend on the *same variable*. If your conditions involve different variables or complex expressions, use if/else.

**Coming up:** The `switch` pattern appears everywhere in game code. In Chapter 7, the calculator's click handler uses a structure like this to route actions.

### Truthy and Falsy

JavaScript values are either truthy (behave like `true` in boolean context) or falsy:

**Falsy values:** `false`, `0`, `''`, `null`, `undefined`, `NaN`

**Everything else is truthy:** `1`, `'hello'`, `[]`, `{}`, any object

This powers common idioms:
```javascript
if (scene.choices.length) { ... }  // true if length > 0
const display = value || 'default';  // fallback pattern from Ch1
```

### The Story as a Data Object

The key architecture insight: the entire story lives in one object. Each scene is a key; the value describes the scene and its choices:

```javascript
const story = {
  start: {
    title: 'The Forest Edge',
    text: `...`,
    choices: [
      { label: 'Take the River Path', next: 'river' },
      { label: 'Take the Shadow Way', next: 'shadow' },
    ],
  },
  river: {
    title: 'The River Path',
    // ...
  },
  // ...
};
```

To advance to the next scene: `renderScene(choice.next)`. That's it. All the branching logic lives in the data, not in code. Adding a new scene or path means adding data — no `if/else` changes.

**Coming up:** This is the same pattern Chapter 12 uses for game state. Instead of `story[sceneKey]`, it's `gameState` accessed by property. Data-driven architectures scale better than logic-driven ones.

---

## Guided Exercises

### Exercise 1: Add a New Scene

**The Challenge:** Add a scene called `guardian_test` — a riddle posed by a hooded figure. Getting it right leads to `victory_smart`; getting it wrong leads to `sneak_attempt`. Connect it via a new choice in `tower_entrance`.

**Where to start:** Look at any existing scene in the `story` object. Copy its structure and modify it. Where does it connect?

---

**Step 1: Add the scene to the story object.**

Find the `story` object (it starts with `start:`). Add a new property after any existing scene:

```javascript
guardian_test: {
  title: "The Guardian's Test",
  text: `Before you stands a hooded figure made of mist. "Answer my riddle," it says.
    "I have cities but no houses, forests but no trees. What am I?"`,
  choices: [
    { label: "A map", next: "victory_smart" },
    { label: "A dream", next: "sneak_attempt" },
  ],
},
```

---

**Step 2: Connect it from an existing scene.**

Find `tower_entrance` in the story object. Add a new choice that leads to `guardian_test`:

```javascript
choices: [
  { label: "Hurl water at the sentinel's eyes", next: "victory_smart" },
  { label: "Face the guardian's test", next: "guardian_test" },  // ← new
  { label: "Try to sneak around it", next: "sneak_attempt" },
],
```

**Test it:** Reload the page. Navigate to the tower entrance. The new choice should appear.

**Think about it:** You added a scene with no JavaScript logic changes — just data. What does this tell you about data-driven architectures vs. logic-driven ones?

*(The story engine code doesn't care about specific scenes. It reads `choice.next` as a string and renders whatever scene has that key. Adding scenes is purely a data change.)*

---

### Exercise 2: Track a Player Decision

**The Challenge:** Add a `freedFox` variable that starts `false`. Set it to `true` when the player frees the fox. In `fox_ignored`, if `freedFox` is true, unlock a special "Use the fox's wisdom" choice that leads to `victory_smart`.

**Where to start:** You need state that persists across scene renders. Where should that variable be declared?

*(Outside all functions, at the top of the script — not inside `renderScene()`, which runs on every scene change.)*

---

**Step 1: Declare the state variable.**

At the top of the `<script>`, alongside `let currentScene`:

```javascript
let freedFox = false;
```

---

**Step 2: Set it when the relevant scene renders.**

In `renderScene()`, check the scene key and update state:

```javascript
function renderScene(sceneKey) {
  currentScene = sceneKey;
  if (sceneKey === 'fox_freed') freedFox = true;
  // ... rest of function
}
```

**Think about it:** Why set `freedFox = true` at the start of `renderScene`, rather than in the choice click handler?

*(The state should be set when the scene is reached, regardless of how. Keeping it in `renderScene` is cleaner — there's one place to check.)*

---

**Step 3: Use the state to conditionally add a choice.**

The story object is constant, but you can override choices at render time. In `renderScene()`, after getting the scene from the story object:

```javascript
const scene = { ...story[sceneKey] };  // shallow copy so we don't mutate original

if (sceneKey === 'fox_ignored' && freedFox) {
  scene.choices = [
    ...scene.choices,
    { label: "Wait — the fox said something about cold water...", next: "victory_smart" },
  ];
}
```

**What does `{ ...story[sceneKey] }` do?** The spread operator copies all properties of the scene object into a new object. This prevents mutating the original story data.

**Test it:** Play through fox_freed, then reach fox_ignored. The extra choice should appear. Play without freeing the fox — it shouldn't.

---

### Exercise 3: A Scoring System

**The Challenge:** Track "moral score" across decisions. Award +2 for freeing the fox, -1 for ignoring it. Display a moral rating on victory screens using a `switch` statement.

**Where to start:** You need to add `points` to certain choices, track a running total, and display the result. Which pieces need to change?

---

**Step 1: Add points to choices.**

In the story object, choices that should affect score get a `points` property:

```javascript
river: {
  choices: [
    { label: "Stop and free the fox", next: "fox_freed", points: 2 },
    { label: "Keep moving", next: "fox_ignored", points: -1 },
  ],
},
```

---

**Step 2: Track the score.**

Declare a score variable:
```javascript
let moralScore = 0;
```

In the button click handlers (where `renderScene(choice.next)` is called), add the points:

```javascript
// In the choice button onclick:
onclick="moralScore += ${c.points || 0}; renderScene('${c.next}')"
```

**Why `|| 0`?** Some choices don't have a `points` property. `undefined + 0` would give `NaN`. `(undefined || 0)` gives `0`. Safe fallback.

---

**Step 3: Display the rating on victory endings.**

Write a function that converts a number to a rating:

```javascript
function getMoralRating(score) {
  switch (true) {
    case score >= 5:  return 'Pure-Hearted';
    case score > 0:   return 'Compassionate';
    case score === 0: return 'Neutral';
    default:          return 'Selfish';
  }
}
```

**Why `switch (true)`?** Unlike a normal `switch` that compares one value, `switch (true)` evaluates each case as a boolean condition. It's an unusual pattern, but works well for range-based ratings. You could use if/else here too — either works.

In your victory scenes' text, display the rating:

```javascript
victory_smart: {
  text: `... You did it.
    <br><br><em>Moral Rating: ${getMoralRating(moralScore)}</em>`,
```

**The complete picture:** The scoring system threads through: data (points on choices) → event handling (add to running total on click) → display (function translates score to string at victory). Three concerns, separated.

---

## What You Learned

| Concept | What It Does | Coming Up In |
|---------|-------------|-------------|
| `if/else` | Run code conditionally | Every chapter — the skeleton of all logic |
| `===` / `!==` | Strict equality comparison | Ch7 (calculator ops), Ch9 (game state) |
| `&&` / `\|\|` / `!` | Combine and invert conditions | Ch9 (collision guards), Ch10 (error checks) |
| Ternary `? :` | Inline two-way choice producing a value | Ch6 (conditional HTML), Ch10 (loading states) |
| `switch` | Multiple branches from one value | Ch7 (calculator actions), Ch12 (game state) |
| Truthy/Falsy | Values that behave like true/false | Ch1's `\|\|` fallback; throughout |
| Data-driven design | Story lives in data, not in `if/else` | Ch12's game state object does the same thing |

## Building with Claude

- "Add a timer to each scene — if the player doesn't choose in 30 seconds, pick a random choice automatically. Use `setInterval` and clear it when a choice is made."
- "Add an inventory system: some scenes give items (like `'water_vial'`), and later scenes check `inventory.includes('item')` to unlock special choices."
- "Show the player all the endings they've found across multiple playthroughs. Track seen endings in `localStorage`."

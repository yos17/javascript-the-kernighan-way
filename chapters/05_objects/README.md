# Chapter 5: Objects

## Why Objects Exist

Imagine tracking a creature in your game with separate variables:

```javascript
const creatureName = 'Flamepaw';
const creatureType = 'fire';
const creatureHp = 60;
const creatureAttack = 50;
const creatureDefense = 40;
```

This works for one creature. What about twenty? You'd have `creature1Name`, `creature2Name`... and every function that touches creatures needs to accept five separate parameters.

Objects solve this by **bundling related data together**:

```javascript
const creature = {
  name: 'Flamepaw',
  type: 'fire',
  hp: 60,
  stats: { attack: 50, defense: 40 },
};
```

Now a creature is one thing. You can pass it to a function, store it in an array, and describe it as a unit. This is the fundamental purpose of objects: **grouping related data so you can treat it as a single thing**.

## The Program: Creature Card Builder

A Pokémon-style card builder. You create creatures by choosing a name, type (fire, water, grass, electric, dark, psychic), and stats (HP, attack, defense). The app displays each creature as a colorful card with stats and lets you delete cards. It demonstrates object literals, nested objects, destructuring, and lookup tables.

## The Complete Program

See `creature_cards.html` for the full source. The key JavaScript concepts are explained below.

## How It Works

### Object Literals

An object literal is data grouped with curly braces:

```javascript
const creature = {
  name: 'Flamepaw',      // key: value pairs, separated by commas
  type: 'fire',
  hp: 60,
  stats: { attack: 50, defense: 40 },   // nested object
  id: Date.now(),
  describe() {                            // method: a function inside an object
    return `${this.name} is a ${this.type}-type with ${this.hp} HP.`;
  },
};
```

The structure in memory looks like this:

```
creature
  ├── name: "Flamepaw"
  ├── type: "fire"
  ├── hp: 60
  ├── stats
  │     ├── attack: 50
  │     └── defense: 40
  ├── id: 1715000000000
  └── describe: [function]
```

You read values with dot notation: `creature.name` → `"Flamepaw"`. For nested values, chain the dots: `creature.stats.attack` → `50`.

### Properties and Methods

Properties store data. Methods are functions that live inside an object and can access the object's other data via `this`:

```javascript
const creature = {
  name: 'Flamepaw',
  describe() {
    // 'this' refers to the creature object this method is called on
    return `${this.name} is a ${this.type}-type.`;
  }
};

creature.describe(); // → "Flamepaw is a fire-type."
```

Why put the function *inside* the object instead of outside? Because it keeps behavior with the data it operates on. The `describe` function only makes sense for creatures — it belongs on the object.

### Destructuring

Instead of writing `creature.name` and `creature.type` repeatedly, destructuring pulls properties into local variables:

```javascript
// Without destructuring — repetitive
const name = creature.name;
const type = creature.type;
const hp = creature.hp;

// With destructuring — same thing, one line
const { name, type, hp } = creature;
```

You can also destructure nested objects:

```javascript
const { attack, defense } = creature.stats;
// attack is now 50, defense is now 40
```

In `renderCards()`, the full creature is unpacked in two lines, and the rest of the function reads cleanly:

```javascript
const { name, type, hp, stats, id } = creature;
const { attack, defense } = stats;
// Now we can just write: name, attack, defense — not creature.name, creature.stats.attack
```

### TYPE_DATA: Objects as Lookup Tables

Here's a pattern that replaces chains of if/else:

```javascript
// Without lookup — tedious and fragile
function getColor(type) {
  if (type === 'fire') return '#e74c3c';
  else if (type === 'water') return '#3498db';
  else if (type === 'grass') return '#27ae60';
  // ... 3 more ...
}

// With lookup — one line per type, easy to extend
const TYPE_DATA = {
  fire:     { color: '#c0392b', bg: '#e74c3c', emoji: '🔥' },
  water:    { color: '#2471a3', bg: '#3498db', emoji: '💧' },
  grass:    { color: '#1e8449', bg: '#27ae60', emoji: '🌿' },
  electric: { color: '#b7950b', bg: '#f1c40f', emoji: '⚡' },
  dark:     { color: '#2c3e50', bg: '#34495e', emoji: '🌑' },
  psychic:  { color: '#7d3c98', bg: '#9b59b6', emoji: '🔮' },
};

const { color, bg, emoji } = TYPE_DATA[type];  // type is the key
```

When `type` is `'fire'`, `TYPE_DATA['fire']` returns the fire object. This pattern is cleaner than if/else for any situation where you're mapping one string to a set of values. Real code uses this constantly — think theme objects, route configs, error message maps.

### Arrays of Objects

The app stores all creatures in an array:

```javascript
const creatures = [];

// Add
creatures.push(newCreature);

// Remove
const index = creatures.findIndex(c => c.id === id);
creatures.splice(index, 1);

// Loop and render
for (const creature of creatures) {
  // build a card for each creature
}
```

Each creature in the array is a full object with all its data. `findIndex` searches by a condition (creature id matches), unlike `indexOf` which only works for primitive values.

### Object.keys(), Object.values(), Object.entries()

These built-in methods extract data from objects:

```javascript
Object.keys(TYPE_DATA)    // → ['fire', 'water', 'grass', 'electric', 'dark', 'psychic']
Object.values(TYPE_DATA)  // → [{ color, bg, emoji }, ...]
Object.entries(TYPE_DATA) // → [['fire', { color, bg, emoji }], ...]
```

Use `Object.keys()` when you need to iterate over all possible types. Use `Object.entries()` when you need both the key and value at the same time.

---

## Guided Exercises

### Exercise 1: Add a Level Property

**The Challenge:** Add a `level` field (1–100) to each creature. Show it on the card.

**Where to start:** Think about what needs to change when you create a creature. Currently `createCreature()` reads name, type, hp, attack, defense. Where would level fit in?

*(Look at `createCreature()` in the code before reading on.)*

---

**Step 1: Add the input to the HTML form.**

You need a level input in the form. The stat row (HP, Attack, Defense) is a good model:

```html
<div class="stat-row">
  <div>
    <label for="level">Level</label>
    <input type="number" id="level" value="1" min="1" max="100">
  </div>
  <!-- ...existing stat inputs... -->
</div>
```

**What does this give you?** A field the user can fill in. But the JavaScript doesn't read it yet.

---

**Step 2: Read the level value in `createCreature()`.**

After you read `hp`, `attack`, and `defense`, add this:

```javascript
const level = Math.min(100, Math.max(1, +document.getElementById('level').value));
```

**Why `Math.min` and `Math.max`?** They clamp the value to the allowed range. `+` converts the string from the input into a number. This is the same pattern already used for hp, attack, and defense — so you're just following the existing convention.

**Think about it:** Where does `level` go in the creature object?

---

**Step 3: Add `level` to the creature object.**

```javascript
const creature = {
  name,
  type,
  hp,
  level,          // ← add this
  stats: { attack, defense },
  id: Date.now(),
  describe() {
    return `${this.name} (Lvl ${this.level}) is a ${this.type}-type with ${this.hp} HP.`;
  },
};
```

**What's happening now?** The creature stores its level. But the card doesn't display it yet.

---

**Step 4: Display the level on the card.**

In `renderCards()`, you destructure the creature: `const { name, type, hp, stats, id } = creature;`. Add `level` to that list:

```javascript
const { name, type, hp, stats, id, level } = creature;
```

Then update the card header in the `innerHTML` template:

```javascript
<h2>${name} <small style="font-size: 0.8rem; opacity: 0.8;">Lvl ${level}</small></h2>
```

**Test it:** Refresh the page, set a level, create a creature. The card should show "Flamepaw Lvl 42".

---

### Exercise 2: A Type Weakness System

**The Challenge:** Each type has a weakness. Fire is weak to water. Water is weak to grass. Add a `weakness` property to `TYPE_DATA` and display it on each card.

**Where to start:** Look at `TYPE_DATA`. It already holds per-type data (color, bg, emoji). Where would you add weakness?

*(Look at the `TYPE_DATA` object before reading on.)*

---

**Step 1: Extend TYPE_DATA with weaknesses.**

```javascript
const TYPE_DATA = {
  fire:     { color: '#c0392b', bg: '#e74c3c', emoji: '🔥', weakness: 'water' },
  water:    { color: '#2471a3', bg: '#3498db', emoji: '💧', weakness: 'grass' },
  grass:    { color: '#1e8449', bg: '#27ae60', emoji: '🌿', weakness: 'fire' },
  electric: { color: '#b7950b', bg: '#f1c40f', emoji: '⚡', weakness: 'dark' },
  dark:     { color: '#2c3e50', bg: '#34495e', emoji: '🌑', weakness: 'psychic' },
  psychic:  { color: '#7d3c98', bg: '#9b59b6', emoji: '🔮', weakness: 'electric' },
};
```

**Think about it:** Now `TYPE_DATA[type].weakness` exists. Where in `renderCards()` should you read it?

---

**Step 2: Destructure weakness in renderCards().**

Currently you have:

```javascript
const { color, bg, emoji } = TYPE_DATA[type];
```

Add `weakness` to the destructuring:

```javascript
const { color, bg, emoji, weakness } = TYPE_DATA[type];
```

---

**Step 3: Display weakness on the card.**

Add a line to the card's HTML, after the type label:

```javascript
<div class="card-type">${type} type · weak to ${weakness}</div>
```

**The complete picture:** `TYPE_DATA` is a lookup — you store all type attributes once, and look them up by name. Adding any new per-type attribute is just one line per type in `TYPE_DATA`, and one variable in the render code. This pattern scales to dozens of attributes without adding complexity.

---

### Exercise 3: Give Each Creature a Move

**The Challenge:** Each creature gets one move (like "Flame Burst" for fire types). Add a `move` property — look it up from the type — and display it on the card.

**Where to start:** This is similar to Exercise 2. The move depends on the creature's type. Where should move data live?

*(Think about it before reading on.)*

---

**Step 1: Add a MOVES lookup object.**

```javascript
const MOVES = {
  fire:     'Flame Burst',
  water:    'Tidal Wave',
  grass:    'Vine Whip',
  electric: 'Thunder Shock',
  dark:     'Shadow Ball',
  psychic:  'Psybeam',
};
```

**Why a separate object instead of adding to TYPE_DATA?** You could add it to TYPE_DATA, but keeping it separate keeps responsibilities clear: `TYPE_DATA` is about visual theming, `MOVES` is about gameplay. This will matter when your codebase grows.

---

**Step 2: Look up the move when creating a creature.**

In `createCreature()`, after reading the type:

```javascript
const move = MOVES[type];  // 'fire' → 'Flame Burst'
```

Then add `move` to the creature object.

---

**Step 3: Display the move on the card.**

Destructure `move` in `renderCards()` and add it to the card HTML:

```javascript
<div class="card-type">${type} type</div>
<div style="padding: 4px 16px; font-size: 0.8rem; opacity: 0.8;">⚔️ ${move}</div>
```

**Check your understanding:** Why did we look up the move from `MOVES[type]` when creating the creature, rather than at render time? Could it work either way? What's the difference?

*(Answer: Both work. Storing it on the creature lets each creature eventually have a custom move. Looking it up at render time means all creatures of the same type always show the same move — which might be what you want. The choice depends on the design.)*

---

## What You Learned

| Concept | What It Does | Real-World Use |
|---------|-------------|----------------|
| **Object Literal** | Groups related data with `{ key: value }` | Every React component has a `props` object; Redux uses a state object |
| **Properties** | Named values accessed with dot notation (`obj.prop`) | API responses are objects; you access them the same way |
| **Methods** | Functions inside objects that use `this` | Array methods like `.map()` and `.filter()` are methods on the Array object |
| **`this` Keyword** | Refers to the object when used inside a method | Used heavily in class-based components, event handlers |
| **Nested Objects** | Objects inside objects, chained dots (`obj.nested.prop`) | API responses are often deeply nested; destructuring tames them |
| **Destructuring** | Extracts properties into variables: `const { x, y } = obj` | Used everywhere in modern JS — function params, imports, API data |
| **Lookup Tables** | Object as dictionary: find values by key instead of if/else | Theme configs, route maps, feature flags in production apps |
| **`Object.keys()`** | Returns all keys as an array | Used to iterate over config objects, validate schemas |
| **`Object.entries()`** | Returns key-value pairs as `[key, value]` arrays | Used to build UI from config objects |

### Real-World Connections

- **React**: Component `props` and `state` are objects. Destructuring props (`const { name, age } = props`) is standard React style.
- **Redux**: The entire application state lives in one object — the same "single object as truth" idea you used with `creatures`.
- **APIs**: Every JSON API response is an object (or array of objects). The `TYPE_DATA` lookup pattern is how you convert API data to UI properties.
- **TypeScript**: TypeScript adds types to JavaScript objects. Once you understand objects, TypeScript types are just descriptions of their shape.

## Building with Claude

- "Add a battle system where two creatures fight based on their stats. The winner is whoever's attack exceeds the other's defense. Show the battle log step by step."

- "Create an inventory system where each creature has a list of held items. Items can boost stats. Let me choose items when creating a creature."

- "Add a trading feature where I can export a creature as JSON and import another creature's JSON to swap them."

- "Build a type-effectiveness system where attack damage is multiplied based on type matchups (fire vs. water = 0.5x damage). Show the multiplier when two creatures battle."

# Chapter 5: Objects

JavaScript objects let you bundle related data and behavior together. Instead of tracking a creature's name, type, and stats as separate variables, you wrap them into one thing — and that thing can even know how to describe itself.

We'll build a **Pokémon-style Creature Card Builder** where every creature is a JavaScript object, and the cards display type-based color schemes pulled from nested data.

---

## The Program

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Creature Card Builder</title>
  <style>
    /* ... see creature_cards.html for full styles ... */
  </style>
</head>
<body>
<!-- see creature_cards.html -->
</body>
</html>
```

Open **`creature_cards.html`** in your browser to run the full program.

---

## Key Concepts Walkthrough

### Object Literals

The simplest way to create an object is with curly braces:

```js
const creature = {
  name: 'Pyrocat',
  type: 'fire',
  hp: 95,
};
```

Each `key: value` pair is a **property**. Access them with dot notation (`creature.name`) or bracket notation (`creature['name']`).

### Nested Objects

Objects can contain other objects:

```js
const TYPE_DATA = {
  fire:  { emoji: '🔥', cssClass: 'type-fire', hpColor: '#fca5a5' },
  water: { emoji: '💧', cssClass: 'type-water', hpColor: '#93c5fd' },
};
```

Access nested values by chaining: `TYPE_DATA.fire.emoji` → `'🔥'`

### Methods

A property whose value is a function is called a **method**:

```js
const creature = {
  name: 'Pyrocat',
  attack: 88,
  defense: 55,
  maxHp: 95,
  powerRating() {
    return Math.round((this.attack * 2 + this.defense + this.maxHp / 2) / 4);
  }
};

creature.powerRating(); // 63
```

`this` inside a method refers to the object itself.

### Destructuring

Instead of writing `TYPE_DATA[type].emoji` over and over, pull values out in one line:

```js
const { emoji, cssClass, hpColor } = TYPE_DATA[type];
// Now use emoji, cssClass, hpColor directly
```

Works with nested objects and function parameters too:

```js
function createCreature({ name, type, hp, attack, defense }) {
  // name, type, hp, etc. are already local variables
}
```

### Computed Property Names

Use a variable as a property key by wrapping it in `[]`:

```js
const key = 'fire';
const data = { [key]: '🔥' };  // same as { fire: '🔥' }
```

Useful when building objects dynamically.

### Spread Operator

Copy an object and add or override properties:

```js
const base = { hp: 80, attack: 65 };
const stronger = { ...base, attack: 90 };  // { hp: 80, attack: 90 }
```

In the program we use it for immutable array updates:

```js
creatures = [...creatures, newCreature];
```

### Object.keys / values / entries

Iterate over an object's contents:

```js
const stats = { hp: 80, attack: 65, defense: 50 };

Object.keys(stats);    // ['hp', 'attack', 'defense']
Object.values(stats);  // [80, 65, 50]
Object.entries(stats); // [['hp', 80], ['attack', 65], ['defense', 50]]

for (const [key, val] of Object.entries(stats)) {
  console.log(`${key}: ${val}`);
}
```

---

## Try It

1. **Add a new type**: Add `poison` to `TYPE_DATA` with a purple gradient and the 🌿 emoji. Add it to the `<select>` dropdown.

2. **Add a Speed stat**: Add a `speed` input to the form, store it in the creature object, and display it on the card.

3. **Sort by power**: After `creatures = [...creatures, creature]`, sort the array by `powerRating()` before calling `renderAll()`.

4. **Edit a card**: Double-click a card's name to make it editable (use `contenteditable="true"`), then update the object when the user blurs out.

---

## Exercises

### Exercise 1 — Battle Score

Add a `battleScore()` method to the creature object that returns:

```
(attack × 1.5) + (defense × 1.2) + (hp × 0.5)
```

Display it on the card as a small badge in the corner. Round to the nearest integer.

### Exercise 2 — Type Matchups

Create a `TYPE_MATCHUPS` object where each type lists its strengths and weaknesses:

```js
const TYPE_MATCHUPS = {
  fire:  { strong: ['grass', 'ice'], weak: ['water', 'rock'] },
  water: { strong: ['fire', 'rock'], weak: ['grass', 'electric'] },
  // ...
};
```

When two cards are displayed, add a small "⚔️ vs" comparison button that pops up an alert saying which creature has the type advantage.

### Exercise 3 — Import / Export

Add a **"Export JSON"** button that calls `JSON.stringify(creatures, null, 2)` and shows the result in a `<textarea>`. Add an **"Import JSON"** button that reads from that textarea and uses `JSON.parse()` to restore creatures. This simulates a simple save/load system.

---

## Solutions

### Solution 1 — Battle Score

```js
function createCreature({ name, type, hp, attack, defense }) {
  const { emoji, cssClass, hpColor } = TYPE_DATA[type];
  return {
    id: nextId++,
    name, type, hp, maxHp: hp, attack, defense,
    emoji, cssClass, hpColor,
    powerRating() {
      return Math.round((this.attack * 2 + this.defense + this.maxHp / 2) / 4);
    },
    battleScore() {
      return Math.round(this.attack * 1.5 + this.defense * 1.2 + this.maxHp * 0.5);
    }
  };
}
```

In `renderCard`, add to the card HTML:

```js
`<div class="battle-badge">⚔️ ${creature.battleScore()}</div>`
```

### Solution 2 — Type Matchups

```js
const TYPE_MATCHUPS = {
  fire:     { strong: ['grass', 'ice'],      weak: ['water', 'rock']     },
  water:    { strong: ['fire', 'rock'],       weak: ['grass', 'electric'] },
  grass:    { strong: ['water', 'rock'],      weak: ['fire', 'ice']       },
  electric: { strong: ['water'],              weak: ['grass', 'ground']   },
  psychic:  { strong: ['fighting', 'ghost'],  weak: ['dark', 'ghost']     },
  ice:      { strong: ['grass', 'dragon'],    weak: ['fire', 'steel']     },
  dragon:   { strong: ['dragon'],             weak: ['ice', 'fairy']      },
  dark:     { strong: ['psychic', 'ghost'],   weak: ['fighting', 'fairy'] },
};

function compareCreatures(a, b) {
  const aMatchup = TYPE_MATCHUPS[a.type];
  const bMatchup = TYPE_MATCHUPS[b.type];

  if (aMatchup?.strong.includes(b.type)) {
    return `${a.name} (${a.type}) has the advantage over ${b.name} (${b.type})!`;
  } else if (bMatchup?.strong.includes(a.type)) {
    return `${b.name} (${b.type}) has the advantage over ${a.name} (${a.type})!`;
  } else {
    return `${a.name} vs ${b.name} — it's a fair fight!`;
  }
}
```

### Solution 3 — Import / Export

```js
document.getElementById('exportBtn').addEventListener('click', () => {
  const json = JSON.stringify(creatures, null, 2);
  document.getElementById('jsonArea').value = json;
});

document.getElementById('importBtn').addEventListener('click', () => {
  try {
    const imported = JSON.parse(document.getElementById('jsonArea').value);
    // Restore methods (they're stripped by JSON.stringify)
    creatures = imported.map(c => createCreature(c));
    renderAll();
  } catch (e) {
    alert('Invalid JSON!');
  }
});
```

Note: `JSON.stringify` strips methods — only data survives. That's why we re-run `createCreature` on import to restore the methods.

---

## What You Learned

| Concept | Where it appeared |
|---------|-------------------|
| Object literals `{}` | `createCreature()`, `TYPE_DATA` |
| Properties & dot notation | `creature.name`, `TYPE_DATA.fire.emoji` |
| Nested objects | `TYPE_DATA` with nested `{ emoji, cssClass, hpColor }` |
| Methods & `this` | `powerRating()` on creature |
| Destructuring | `const { emoji, cssClass } = TYPE_DATA[type]` |
| Parameter destructuring | `function createCreature({ name, type, hp })` |
| Computed property names | `{ [key]: value }` |
| Spread operator | `[...creatures, newCreature]` |
| `Object.keys/values/entries` | Iterating stats in console log |
| Optional chaining `?.` | `TYPE_MATCHUPS[type]?.strong` |

---

## Building with Claude

- *"Add a move list to each creature — an array of objects with `name` and `power` fields — and show the top move on the card."*
- *"Make the cards flip on hover to show stats on the front and a full description on the back using CSS 3D transforms."*
- *"Add a 'favorites' feature using a `favorite` boolean property — starred creatures float to the top of the grid."*

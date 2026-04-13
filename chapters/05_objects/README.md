# Chapter 5: Objects

Objects are collections of related data and behavior grouped together. In JavaScript, an object is a set of key-value pairs—think of it like a creature with a name, stats, and abilities all packed into one thing. Objects let you organize code so it's cleaner and easier to manage.

## The Program: Creature Card Builder

This is a Pokémon-style card builder. You create creatures by choosing a name, type (fire, water, grass, electric, dark, psychic), and stats (HP, attack, defense). The app displays each creature as a colorful card with stats and lets you delete cards. It demonstrates objects, nested objects, destructuring, and lookup tables.

## The Complete Program

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Creature Card Builder</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { font-family: 'Segoe UI', system-ui, sans-serif; background: #1a1a2e; color: #eee; min-height: 100vh; padding: 20px; }
  h1 { text-align: center; margin-bottom: 20px; font-size: 1.8rem; }

  .builder { max-width: 480px; margin: 0 auto 30px; background: #16213e; padding: 20px; border-radius: 12px; }
  .builder label { display: block; font-size: 0.85rem; margin-bottom: 4px; color: #aaa; }
  .builder input, .builder select { width: 100%; padding: 8px 10px; margin-bottom: 12px; border: 1px solid #333; border-radius: 6px; background: #0f3460; color: #eee; font-size: 0.95rem; }
  .builder button { width: 100%; padding: 10px; border: none; border-radius: 8px; background: #e94560; color: #fff; font-size: 1rem; font-weight: 600; cursor: pointer; transition: transform 0.1s; }
  .builder button:hover { transform: scale(1.02); }

  .stat-row { display: flex; gap: 10px; }
  .stat-row > div { flex: 1; }

  .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(260px, 1fr)); gap: 20px; max-width: 1100px; margin: 0 auto; }

  .card { border-radius: 14px; overflow: hidden; box-shadow: 0 4px 20px rgba(0,0,0,0.4); transition: transform 0.2s; }
  .card:hover { transform: translateY(-4px); }
  .card-header { padding: 14px 16px 8px; display: flex; justify-content: space-between; align-items: center; }
  .card-header h2 { font-size: 1.2rem; }
  .card-hp { font-weight: 700; font-size: 1.1rem; }
  .card-art { height: 120px; margin: 0 16px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-size: 4rem; background: rgba(255,255,255,0.15); }
  .card-type { padding: 6px 16px; font-size: 0.8rem; font-weight: 600; text-transform: uppercase; letter-spacing: 1px; }
  .card-stats { display: flex; justify-content: space-around; padding: 12px 16px 16px; }
  .card-stats div { text-align: center; }
  .card-stats .label { font-size: 0.7rem; text-transform: uppercase; opacity: 0.7; }
  .card-stats .value { font-size: 1.3rem; font-weight: 700; }
  .card-remove { background: none; border: none; color: rgba(255,255,255,0.5); cursor: pointer; font-size: 1.2rem; padding: 0; width: auto; margin: 0; }
  .card-remove:hover { color: #fff; }
</style>
</head>
<body>

<h1>Creature Card Builder</h1>

<div class="builder">
  <label for="name">Creature Name</label>
  <input type="text" id="name" placeholder="e.g. Flamepaw" maxlength="20">

  <label for="type">Type</label>
  <select id="type">
    <option value="fire">Fire</option>
    <option value="water">Water</option>
    <option value="grass">Grass</option>
    <option value="electric">Electric</option>
    <option value="dark">Dark</option>
    <option value="psychic">Psychic</option>
  </select>

  <div class="stat-row">
    <div><label for="hp">HP</label><input type="number" id="hp" value="60" min="10" max="200"></div>
    <div><label for="attack">Attack</label><input type="number" id="attack" value="50" min="5" max="150"></div>
    <div><label for="defense">Defense</label><input type="number" id="defense" value="40" min="5" max="150"></div>
  </div>

  <button onclick="createCreature()">Add Creature Card</button>
</div>

<div class="grid" id="grid"></div>

<script>
const TYPE_DATA = {
  fire:     { color: '#c0392b', bg: '#e74c3c', emoji: '🔥' },
  water:    { color: '#2471a3', bg: '#3498db', emoji: '💧' },
  grass:    { color: '#1e8449', bg: '#27ae60', emoji: '🌿' },
  electric: { color: '#b7950b', bg: '#f1c40f', emoji: '⚡' },
  dark:     { color: '#2c3e50', bg: '#34495e', emoji: '🌑' },
  psychic:  { color: '#7d3c98', bg: '#9b59b6', emoji: '🔮' },
};

const creatures = [];

function createCreature() {
  const name = document.getElementById('name').value.trim();
  if (!name) return alert('Give your creature a name!');

  const type = document.getElementById('type').value;
  const hp = Math.min(200, Math.max(10, +document.getElementById('hp').value));
  const attack = Math.min(150, Math.max(5, +document.getElementById('attack').value));
  const defense = Math.min(150, Math.max(5, +document.getElementById('defense').value));

  const creature = {
    name,
    type,
    hp,
    stats: { attack, defense },
    id: Date.now(),
    describe() {
      return `${this.name} is a ${this.type}-type with ${this.hp} HP.`;
    },
  };

  creatures.push(creature);
  renderCards();
  document.getElementById('name').value = '';
}

function removeCreature(id) {
  const index = creatures.findIndex(c => c.id === id);
  if (index !== -1) creatures.splice(index, 1);
  renderCards();
}

function renderCards() {
  const grid = document.getElementById('grid');
  grid.innerHTML = '';

  for (const creature of creatures) {
    const { name, type, hp, stats, id } = creature;
    const { color, bg, emoji } = TYPE_DATA[type];
    const { attack, defense } = stats;

    const card = document.createElement('div');
    card.className = 'card';
    card.style.background = `linear-gradient(135deg, ${bg}, ${color})`;

    card.innerHTML = `
      <div class="card-header">
        <h2>${name}</h2>
        <span class="card-hp">HP ${hp}</span>
      </div>
      <div class="card-art">${emoji}</div>
      <div class="card-type">${type} type</div>
      <div class="card-stats">
        <div><div class="label">Attack</div><div class="value">${attack}</div></div>
        <div><div class="label">Defense</div><div class="value">${defense}</div></div>
        <div><div class="label">Total</div><div class="value">${hp + attack + defense}</div></div>
      </div>
    `;

    const btn = document.createElement('button');
    btn.className = 'card-remove';
    btn.textContent = '✕';
    btn.onclick = () => removeCreature(id);
    card.querySelector('.card-header').appendChild(btn);

    grid.appendChild(card);
  }
}

// Start with a sample creature
document.getElementById('name').value = 'Flamepaw';
createCreature();
</script>
</body>
</html>
```

## How It Works

### Object Literals

An object literal is data grouped with curly braces. The creature object stores all information about one creature in one place:

```javascript
const creature = {
  name: 'Flamepaw',
  type: 'fire',
  hp: 60,
  stats: { attack: 50, defense: 40 },
  id: 1234567890,
  describe() { return `${this.name} is a ${this.type}-type with ${this.hp} HP.`; }
};
```

Each key (name, type, hp) maps to a value. You read values with dot notation: `creature.name`. Methods are functions inside objects—they get access to other properties via `this`.

### Properties and Methods

Properties store data. Methods are functions. In `createCreature()`, we build an object with both:

```javascript
const creature = {
  name,
  type,
  hp,
  describe() { return `${this.name} is a ${this.type}-type...`; }
};
```

The `describe()` method uses `this.name` and `this.type` to access the creature's own data. When you call `creature.describe()`, `this` refers to that creature.

### Nested Objects (Objects Inside Objects)

The `stats` property is itself an object:

```javascript
stats: { attack: 50, defense: 40 }
```

To access it, use two dots: `creature.stats.attack` returns 50. Nesting keeps related data together—all combat stats live in one place.

### Destructuring

Instead of writing `creature.name` and `creature.type` repeatedly, destructuring pulls properties out:

```javascript
const { name, type, hp, stats, id } = creature;
```

This is shorthand for `const name = creature.name;` etc. You can also destructure nested objects in one line:

```javascript
const { attack, defense } = stats;
```

### TYPE_DATA: A Lookup Object

`TYPE_DATA` is an object used as a lookup table. Instead of writing if-else chains, use the type as a key:

```javascript
const TYPE_DATA = {
  fire:     { color: '#c0392b', bg: '#e74c3c', emoji: '🔥' },
  water:    { color: '#2471a3', bg: '#3498db', emoji: '💧' },
};

const { color, bg, emoji } = TYPE_DATA[type];
```

When `type` is `'fire'`, `TYPE_DATA[type]` returns the fire object with its color, background, and emoji. This pattern is cleaner than switching through many if statements.

### Object.keys(), Object.values(), Object.entries()

These methods extract data from objects. `Object.keys(TYPE_DATA)` returns `['fire', 'water', 'grass', 'electric', 'dark', 'psychic']`. `Object.values(TYPE_DATA)` returns an array of all the type objects. `Object.entries(TYPE_DATA)` returns key-value pairs as arrays. Use these when you need to loop through or count properties.

### The Spread Operator with Objects

The spread operator `...` copies object properties. For example:

```javascript
const newCreature = { ...creature, name: 'NewName' };
```

This creates a new object with all of `creature`'s properties, but overwrites the name. Useful for avoiding accidental changes to the original.

### Array of Objects

`creatures` is an array holding multiple creature objects. The app manages this list with `push()`, `splice()`, and `findIndex()`. When rendering, a `for...of` loop walks through each creature and builds a card for it.

## Try It

1. **Add a Speed Stat**: In `createCreature()`, add a speed input field in the HTML and create a `speed` property. Update the stats display to show speed alongside attack and defense.

2. **Create a Type Weakness System**: Add a `weakness` property to each type in `TYPE_DATA` (e.g., fire is weak to water). Display the weakness on the card by looking it up from `TYPE_DATA`.

3. **Add a Power Move**: Add a `moves` array property to each creature (e.g., `moves: ['Flame Burst', 'Scratch']`). Display the first move below the stats.

4. **Color the Stats**: Use the creature's type color to highlight stat values. Wrap each stat value in a `<span>` with a background color pulled from `TYPE_DATA[type].bg`.

## Exercises

**Exercise 1 (Warm-up): Add a Level Property**

Modify `createCreature()` to add a `level` property (default to 1). Add a level input field to the form and display the level on the card. Use `Math.min()` and `Math.max()` to keep it between 1 and 100.

**Exercise 2 (Medium): Create an Expanded Stats Object**

Instead of storing `stats` as `{ attack, defense }`, create a fuller stats object with spAtk, spDef, and speed. Update the card display to show all five stats. Use destructuring to pull all stats cleanly.

**Exercise 3 (Challenge): Add an Evolution System**

Add an `evolvedForm` property to the creature object. Create a second lookup object `EVOLUTIONS` mapping creature names to evolved names and new stat multipliers. Add an "Evolve" button to each card. Clicking it should create a new creature from the evolved form (higher stats, new name) and remove the original.

## Solutions

**Exercise 1 Solution:**

Add the level input to the HTML form:
```html
<label for="level">Level</label>
<input type="number" id="level" value="1" min="1" max="100">
```

Modify `createCreature()`:
```javascript
const level = Math.min(100, Math.max(1, +document.getElementById('level').value));

const creature = {
  name,
  type,
  hp,
  level,
  stats: { attack, defense },
  id: Date.now(),
  describe() {
    return `${this.name} (Lvl ${this.level}) is a ${this.type}-type with ${this.hp} HP.`;
  },
};
```

Update the card header in `renderCards()`:
```javascript
card.innerHTML = `
  <div class="card-header">
    <h2>${name} <small style="font-size: 0.8rem; opacity: 0.8;">Lvl ${level}</small></h2>
    <span class="card-hp">HP ${hp}</span>
  </div>
  ...
`;
```

**Exercise 2 Solution:**

Modify `createCreature()` to expand the stats object:
```javascript
const hp = Math.min(200, Math.max(10, +document.getElementById('hp').value));
const attack = Math.min(150, Math.max(5, +document.getElementById('attack').value));
const defense = Math.min(150, Math.max(5, +document.getElementById('defense').value));

const creature = {
  name,
  type,
  hp,
  stats: {
    hp,
    attack,
    defense,
    spAtk: Math.floor(attack * 0.8),
    spDef: Math.floor(defense * 0.8),
    speed: Math.floor(Math.random() * 80 + 20),
  },
  id: Date.now(),
  describe() {
    return `${this.name} is a ${this.type}-type with ${this.hp} HP.`;
  },
};
```

Update `renderCards()` to destructure all stats and display them:
```javascript
for (const creature of creatures) {
  const { name, type, hp, stats, id } = creature;
  const { color, bg, emoji } = TYPE_DATA[type];
  const { attack, defense, spAtk, spDef, speed } = stats;

  // ... build card ...

  card.innerHTML = `
    <div class="card-header">
      <h2>${name}</h2>
      <span class="card-hp">HP ${hp}</span>
    </div>
    <div class="card-art">${emoji}</div>
    <div class="card-type">${type} type</div>
    <div class="card-stats">
      <div><div class="label">Attack</div><div class="value">${attack}</div></div>
      <div><div class="label">Sp. Atk</div><div class="value">${spAtk}</div></div>
      <div><div class="label">Defense</div><div class="value">${defense}</div></div>
      <div><div class="label">Sp. Def</div><div class="value">${spDef}</div></div>
      <div><div class="label">Speed</div><div class="value">${speed}</div></div>
    </div>
  `;
  // ... rest of card ...
}
```

**Exercise 3 Solution:**

Add the EVOLUTIONS lookup at the top of the script:
```javascript
const EVOLUTIONS = {
  'Flamepaw': { name: 'Inferno', statMultiplier: 1.3 },
  'Bubbles': { name: 'Tsunami', statMultiplier: 1.25 },
  'Thornling': { name: 'Jungle King', statMultiplier: 1.35 },
};
```

Add an evolve function:
```javascript
function evolveCreature(id) {
  const creature = creatures.find(c => c.id === id);
  if (!creature) return;

  const evolution = EVOLUTIONS[creature.name];
  if (!evolution) return alert(`${creature.name} cannot evolve!`);

  const evolved = {
    name: evolution.name,
    type: creature.type,
    hp: Math.floor(creature.hp * evolution.statMultiplier),
    stats: {
      attack: Math.floor(creature.stats.attack * evolution.statMultiplier),
      defense: Math.floor(creature.stats.defense * evolution.statMultiplier),
    },
    id: Date.now(),
    describe() {
      return `${this.name} (evolved) is a ${this.type}-type with ${this.hp} HP.`;
    },
  };

  removeCreature(id);
  creatures.push(evolved);
  renderCards();
}
```

Update `renderCards()` to add an evolve button if applicable:
```javascript
const canEvolve = Object.keys(EVOLUTIONS).includes(name);

const btn = document.createElement('button');
btn.className = 'card-remove';
btn.textContent = canEvolve ? '✨' : '✕';
btn.onclick = canEvolve ? () => evolveCreature(id) : () => removeCreature(id);
card.querySelector('.card-header').appendChild(btn);
```

## What You Learned

| Concept | What It Does |
|---------|--------------|
| **Object Literal** | Groups related data with `{ key: value }` syntax. |
| **Properties** | Named values stored in an object, accessed with dot notation (obj.prop). |
| **Methods** | Functions inside objects that can use `this` to access other properties. |
| **this Keyword** | Refers to the object itself when used inside a method. |
| **Nested Objects** | Objects inside objects, accessed with chained dots (obj.nested.prop). |
| **Destructuring** | Extracts properties from objects into variables with `const { x, y } = obj`. |
| **Lookup Tables** | Objects used as dictionaries to find values by key instead of if-else chains. |
| **Object.keys()** | Returns an array of all the keys in an object. |
| **Object.values()** | Returns an array of all the values in an object. |
| **Object.entries()** | Returns key-value pairs as an array of arrays. |

## Building with Claude

- "Add a battle system where two creatures fight based on their stats. The winner is whoever's attack + speed exceeds the other's defense + HP."

- "Create an inventory system where each creature has a list of held items. Items can boost stats. Let me choose items when creating a creature."

- "Add a trading feature where I can export a creature as JSON and import another creature's JSON to swap them."

- "Build a pokedex-style search and filter system. Let me filter creatures by type, level, or search by name. Show stats sorted by highest attack, fastest speed, etc."

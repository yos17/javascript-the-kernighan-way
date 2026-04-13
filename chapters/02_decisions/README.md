# Chapter 2: Decisions — The Crystal Tower Adventure Game

A branching story engine where *you* control the narrative with if/else statements, comparisons, and logical operators. This is how real games work: decisions lead to different outcomes. Three levels of choices, four different endings, and a complete data-driven architecture that runs on pure decision logic.

## The Complete Program

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Choose Your Adventure</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Georgia', serif;
      background: #1a1209;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    .game {
      max-width: 660px;
      width: 100%;
    }

    .header {
      text-align: center;
      margin-bottom: 32px;
    }

    .header h1 {
      color: #f4c542;
      font-size: 1.8rem;
      letter-spacing: 1px;
      text-shadow: 0 0 20px rgba(244,197,66,0.4);
    }

    .header p {
      color: rgba(255,255,255,0.4);
      font-size: 0.85rem;
      margin-top: 6px;
    }

    .scene-card {
      background: linear-gradient(145deg, #2a1f0e, #1f1609);
      border: 1px solid rgba(244,197,66,0.2);
      border-radius: 16px;
      padding: 36px;
      margin-bottom: 20px;
    }

    .scene-title {
      color: #f4c542;
      font-size: 0.75rem;
      font-weight: bold;
      text-transform: uppercase;
      letter-spacing: 2px;
      margin-bottom: 16px;
      opacity: 0.7;
    }

    .scene-text {
      color: #e8d5b0;
      font-size: 1.05rem;
      line-height: 1.8;
      margin-bottom: 24px;
    }

    .scene-text em {
      color: #f4c542;
      font-style: normal;
      font-weight: bold;
    }

    .choices {
      display: flex;
      flex-direction: column;
      gap: 10px;
    }

    .choice-btn {
      background: rgba(244,197,66,0.06);
      border: 1px solid rgba(244,197,66,0.25);
      border-radius: 10px;
      color: #e8d5b0;
      padding: 14px 20px;
      font-size: 0.95rem;
      font-family: 'Georgia', serif;
      text-align: left;
      cursor: pointer;
      transition: background 0.2s, border-color 0.2s, transform 0.15s;
    }

    .choice-btn:hover {
      background: rgba(244,197,66,0.12);
      border-color: rgba(244,197,66,0.5);
      transform: translateX(4px);
    }

    .choice-btn .arrow { margin-right: 10px; opacity: 0.5; }

    .outcome-victory {
      background: linear-gradient(145deg, #0e2a1a, #091f12);
      border-color: rgba(52,211,153,0.3);
    }

    .outcome-victory .scene-text { color: #a7f3d0; }
    .outcome-victory .scene-title { color: #34d399; }

    .outcome-defeat {
      background: linear-gradient(145deg, #2a0e0e, #1f0909);
      border-color: rgba(239,68,68,0.3);
    }

    .outcome-defeat .scene-text { color: #fecaca; }
    .outcome-defeat .scene-title { color: #f87171; }

    .restart-btn {
      width: 100%;
      padding: 14px;
      background: rgba(244,197,66,0.1);
      border: 1px solid rgba(244,197,66,0.3);
      border-radius: 10px;
      color: #f4c542;
      font-size: 1rem;
      font-family: 'Georgia', serif;
      cursor: pointer;
      transition: background 0.2s;
    }

    .restart-btn:hover { background: rgba(244,197,66,0.18); }

    .progress {
      text-align: center;
      color: rgba(255,255,255,0.3);
      font-size: 0.8rem;
      margin-bottom: 16px;
      letter-spacing: 1px;
    }

    .fade-in {
      animation: fadeIn 0.4s ease-out;
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(8px); }
      to { opacity: 1; transform: translateY(0); }
    }
  </style>
</head>
<body>
  <div class="game">
    <div class="header">
      <h1>⚔️ The Crystal Tower</h1>
      <p>A choose-your-own-adventure story</p>
    </div>
    <div id="progress" class="progress"></div>
    <div id="scene-container"></div>
  </div>

  <script>
    // The entire story is defined as an object (a "map" of scenes).
    // Each scene has a title, text, and an array of choices.
    // Each choice has a label and a 'next' property pointing to the next scene's key.
    // Scenes with no choices are endings (victory or defeat).
    const story = {
      start: {
        title: "The Forest Edge",
        text: `You stand at the edge of a dark forest. Somewhere beyond the trees,
          the <em>Crystal Tower</em> glows faintly. Two paths split before you.
          A weathered sign says: "Left: The River Path. Right: The Shadow Way."
          From the right, you hear something growling.`,
        choices: [
          { label: "Take the River Path (left)", next: "river" },
          { label: "Take the Shadow Way (right)", next: "shadow" },
        ],
      },

      river: {
        title: "The River Path",
        text: `The river is calm, the moonlight silver on the water. Halfway across
          the stepping stones, you notice a small fox tangled in a fishing net,
          struggling in the shallows. You could stop to help — but the tower is
          close, and you don't know how much time you have.`,
        choices: [
          { label: "Stop and free the fox", next: "fox_freed" },
          { label: "Keep moving — the tower comes first", next: "fox_ignored" },
        ],
      },

      shadow: {
        title: "The Shadow Way",
        text: `The growling was a large wolf, silver-furred, blocking the path.
          It bares its teeth. You have three items in your pack:
          a silver flute, a piece of dried meat, and a torch.`,
        choices: [
          { label: "Play the silver flute", next: "wolf_music" },
          { label: "Offer the dried meat", next: "wolf_meat" },
          { label: "Wave the torch at it", next: "wolf_torch" },
        ],
      },

      fox_freed: {
        title: "The Grateful Fox",
        text: `The fox shakes itself dry and looks at you with eyes that are far
          too intelligent. "The tower's guardian has one weakness," it says —
          yes, the fox is talking — "cold water. Remember that." It vanishes
          into the reeds. You continue to the tower with valuable knowledge.`,
        choices: [
          { label: "Continue to the tower entrance", next: "tower_entrance" },
        ],
      },

      fox_ignored: {
        title: "Approaching the Tower",
        text: `You reach the tower, but the fox's cries still ring in your ears.
          The door is guarded by a stone sentinel with burning eyes.
          You have no idea how to defeat it.`,
        choices: [
          { label: "Try to force the door open", next: "sentinel_defeat" },
          { label: "Look around for a clue", next: "no_clue" },
        ],
      },

      wolf_music: {
        title: "An Ancient Harmony",
        text: `The wolf goes still. Its eyes close. The melody you play seems to
          reach something deep and old in it — a memory of forests before
          humans came. It lies down and lets you pass.
          Its tail even wags, slightly.`,
        choices: [
          { label: "Continue to the tower entrance", next: "tower_entrance" },
        ],
      },

      wolf_meat: {
        title: "A Simple Transaction",
        text: `The wolf sniffs the meat, devours it in one bite, and trots off
          into the dark. Practical. You continue down the path,
          arriving at the tower with your supplies reduced but your time intact.`,
        choices: [
          { label: "Continue to the tower entrance", next: "tower_entrance" },
        ],
      },

      wolf_torch: {
        title: "A Bad Idea",
        text: `The wolf dodges easily and bites your pack, tearing it open.
          You drop the torch and run. You make it past the wolf,
          but you've lost most of your supplies. The tower looms ahead.`,
        choices: [
          { label: "Approach the tower carefully", next: "tower_low" },
        ],
      },

      tower_entrance: {
        title: "The Tower Gate",
        text: `A stone sentinel stands at the gate, ten feet tall,
          eyes burning orange. But you remember the fox's words:
          <em>cold water</em>. Your waterskin is still half full from the river.`,
        choices: [
          { label: "Hurl water at the sentinel's eyes", next: "victory_smart" },
          { label: "Try to sneak around it", next: "sneak_attempt" },
        ],
      },

      tower_low: {
        title: "The Tower Gate (Supplies Low)",
        text: `The same stone sentinel blocks the gate, eyes burning orange.
          You dropped your waterskin when the wolf attacked.
          You have only the flute and a few small items.`,
        choices: [
          { label: "Try to sneak past it", next: "sneak_attempt" },
          { label: "Play the flute as a distraction", next: "victory_clever" },
        ],
      },

      sentinel_defeat: {
        title: "A Poor Outcome",
        text: `You slam into the door. The sentinel grabs you in a stone fist
          the size of a barrel and sets you down — firmly —
          outside the forest. You are unharmed but very, very far from the tower.
          Some quests require preparation.`,
        isEnding: true,
        type: "defeat",
        choices: [],
      },

      no_clue: {
        title: "Nothing to Find",
        text: `You search the area but find nothing useful. The sentinel's patience
          appears to be exhausted. It picks you up and deposits you,
          gently but firmly, back at the forest edge.
          The fox watches from the trees. It does not look surprised.`,
        isEnding: true,
        type: "defeat",
        choices: [],
      },

      sneak_attempt: {
        title: "Almost",
        text: `You slip around the sentinel's left side — almost through the gate —
          when a stone hand closes around your ankle. It carries you back to the
          forest edge and sets you down with surprising gentleness.
          A stone finger wags: no.`,
        isEnding: true,
        type: "defeat",
        choices: [],
      },

      victory_smart: {
        title: "Victory!",
        text: `The water hits the sentinel's eyes. They hiss, sputter, and go dark.
          The stone body goes still. You walk through the gate and climb the tower
          to find the Crystal — a humming sphere of light that fills the room.
          You did it. The fox was right. <em>Cold water.</em>`,
        isEnding: true,
        type: "victory",
        choices: [],
      },

      victory_clever: {
        title: "Victory (The Musical Way)!",
        text: `The flute's melody fills the courtyard. The sentinel's burning eyes
          flicker — it was made from enchanted stone, and enchanted things
          remember music. While it stands frozen in some old memory,
          you slip through the gate. The Crystal awaits you at the top,
          pulsing gold in the dark. You've done it.`,
        isEnding: true,
        type: "victory",
        choices: [],
      },
    };

    // currentScene tracks which scene we're currently showing.
    let currentScene = 'start';
    let stepCount = 0;

    function renderScene(sceneKey) {
      currentScene = sceneKey;
      stepCount++;

      const scene = story[sceneKey];
      const container = document.getElementById('scene-container');

      // Determine the CSS class based on whether this is an ending scene.
      // The ternary operator: condition ? valueIfTrue : valueIfFalse
      let cardClass = 'scene-card fade-in';
      if (scene.isEnding) {
        cardClass += scene.type === 'victory' ? ' outcome-victory' : ' outcome-defeat';
      }

      // Build the choices HTML.
      // If there are no choices, show only the restart button.
      let choicesHTML;
      if (scene.choices.length === 0) {
        choicesHTML = `<button class="restart-btn" onclick="restart()">↺ Start Over</button>`;
      } else {
        // .map() transforms each choice into an HTML string; .join('') combines them.
        choicesHTML = `<div class="choices">
          ${scene.choices.map(c =>
            `<button class="choice-btn" onclick="renderScene('${c.next}')">
              <span class="arrow">›</span>${c.label}
            </button>`
          ).join('')}
        </div>`;
      }

      container.innerHTML = `
        <div class="${cardClass}">
          <div class="scene-title">${scene.title}</div>
          <div class="scene-text">${scene.text}</div>
          ${choicesHTML}
        </div>
      `;

      // Update the step counter.
      document.getElementById('progress').textContent =
        stepCount > 1 ? `Step ${stepCount}` : '';
    }

    function restart() {
      stepCount = 0;
      document.getElementById('progress').textContent = '';
      renderScene('start');
    }

    // Start the story on page load.
    renderScene('start');
  </script>
</body>
</html>
```

## Walkthrough

### if/else: Branching Logic

```javascript
if (scene.isEnding) {
  cardClass += scene.type === 'victory' ? ' outcome-victory' : ' outcome-defeat';
}
```

When JavaScript reaches a decision point, an `if` statement evaluates a condition. If `scene.isEnding` is true (the story has reached an ending), we add a special CSS class. If the scene's type equals `'victory'`, we add green victory styling; otherwise, red defeat styling. That single condition decides the visual mood.

### Comparison Operators: ===, !==, >, <

```javascript
scene.type === 'victory'
scene.choices.length === 0
```

The triple equals `===` checks if two things are exactly the same—same type, same value. `'victory' === 'victory'` is true. `'victory' === 'defeat'` is false. We use `===` instead of `==` because it's safer (it doesn't do weird type conversions). Later, `scene.choices.length === 0` asks: "Are there zero choices?" If yes, show the restart button instead of choice buttons.

### Logical Operators: &&, ||, !

Imagine a scene where you can only get a special choice if two conditions are true:

```javascript
if (hasWater && knowsWeakness) {
  // Only then can you defeat the sentinel
}
```

The `&&` (AND) operator means both must be true. The `||` (OR) operator means at least one is true:

```javascript
if (sceneType === 'victory' || sceneType === 'neutral') {
  // Either victory or neutral — show celebration
}
```

The `!` (NOT) operator flips a boolean:

```javascript
if (!scene.isEnding) {
  // Show next choices (if this is NOT an ending)
}
```

### Ternary Operator: condition ? valueIfTrue : valueIfFalse

```javascript
cardClass += scene.type === 'victory' ? ' outcome-victory' : ' outcome-defeat';
```

The ternary is a compact if/else that produces a value. It reads: "if type equals victory, use outcome-victory class, else use outcome-defeat class." Ternaries are perfect for quick two-way decisions.

### switch Statement: Multiple Paths

While our code uses ternary, a `switch` statement would be cleaner if we had many ending types:

```javascript
switch (scene.type) {
  case 'victory':
    cardClass += ' outcome-victory';
    break;
  case 'defeat':
    cardClass += ' outcome-defeat';
    break;
  case 'neutral':
    cardClass += ' outcome-neutral';
    break;
}
```

Each `case` checks a possible value. The `break` stops execution; without it, code "falls through" to the next case. The `switch` replaces multiple `else if` statements and is easier to read.

## Try It

1. **Add a third victory ending.** Duplicate the `victory_smart` scene as `victory_lucky`. Change its text to describe a different way to win. Add a new choice branch that leads to it.

2. **Change a defeat to a neutral ending.** Find the `sneak_attempt` scene. Change `type: "defeat"` to `type: "neutral"`. Add neutral-ending CSS styling with `.outcome-neutral { ... }`.

3. **Make a choice conditional on the path you took.** Add a global variable `let freedFox = false;` at the top of the script. Set `freedFox = true;` when rendering `fox_freed`. Then in `fox_ignored`, check: `if (!freedFox)` to offer only certain choices.

## Exercises

### Exercise 1: Easy — Add a Scene
Create a new branch. Add a scene called `guardian_test` that appears after `tower_entrance`. Write the scene's dialogue. Connect it with a choice from an existing scene. This teaches you how scenes link together without changing any logic.

### Exercise 2: Medium — Add Ending Types and Styling
Add a `neutral` ending type with appropriate green-and-gold CSS styling. Create a scene with `type: 'neutral'` (neither victory nor defeat). Modify the ternary to handle three cases instead of two. This teaches you how conditions expand.

### Exercise 3: Hard — Build a Scoring System
Track player choices: award points for "good" decisions (freeing the fox: +2 points, ignoring the fox: -1 point). Build a function `getMoralRating(score)` that uses a `switch` statement to return a rating string based on score ranges. Display the final rating at victory endings. This teaches you how decisions accumulate and affect outcomes.

## Solutions

### Exercise 1 Solution

Add this scene object to the `story`:

```javascript
guardian_test: {
  title: "The Guardian's Test",
  text: `Before you stands a hooded figure made of mist. "Answer my riddle," it says.
    "I have cities but no houses, forests but no trees. What am I?" You think...`,
  choices: [
    { label: "A map", next: "victory_smart" },
    { label: "A dream", next: "sneak_attempt" },
  ],
},
```

Then modify `tower_entrance` to add this choice:

```javascript
choices: [
  { label: "Hurl water at the sentinel's eyes", next: "victory_smart" },
  { label: "Face the guardian's test", next: "guardian_test" },
  { label: "Try to sneak around it", next: "sneak_attempt" },
],
```

### Exercise 2 Solution

Add CSS for neutral endings:

```css
.outcome-neutral {
  background: linear-gradient(145deg, #2a1f0e, #1f1609);
  border-color: rgba(244,197,66,0.3);
}

.outcome-neutral .scene-text { color: #fbbf24; }
.outcome-neutral .scene-title { color: #f59e0b; }
```

Modify the ternary in `renderScene`:

```javascript
if (scene.isEnding) {
  if (scene.type === 'victory') {
    cardClass += ' outcome-victory';
  } else if (scene.type === 'neutral') {
    cardClass += ' outcome-neutral';
  } else {
    cardClass += ' outcome-defeat';
  }
}
```

Add a neutral-ending scene:

```javascript
turned_back: {
  title: "A Retreat",
  text: `As you approach the tower, doubt creeps in. The forest is dangerous, the tower strange.
    You turn back to the village. The crystal glows in the distance, forever out of reach.`,
  isEnding: true,
  type: 'neutral',
  choices: [],
},
```

### Exercise 3 Solution

Add at the top of the script:

```javascript
let currentScene = 'start';
let stepCount = 0;
let moralScore = 0; // Track decisions

function getMoralRating(score) {
  switch (true) {
    case score >= 5:
      return "Pure-Hearted";
    case score > 0:
      return "Compassionate";
    case score === 0:
      return "Neutral";
    default:
      return "Selfish";
  }
}
```

Modify choice objects to have a `points` property:

```javascript
river: {
  title: "The River Path",
  text: `...`,
  choices: [
    { label: "Stop and free the fox", next: "fox_freed", points: 2 },
    { label: "Keep moving — the tower comes first", next: "fox_ignored", points: -1 },
  ],
},
```

Update `renderScene` to award points when rendering choices:

```javascript
choicesHTML = `<div class="choices">
  ${scene.choices.map(c =>
    `<button class="choice-btn" onclick="moralScore += ${c.points || 0}; renderScene('${c.next}')">
      <span class="arrow">›</span>${c.label}
    </button>`
  ).join('')}
</div>`;
```

Update progress display:

```javascript
const rating = getMoralRating(moralScore);
document.getElementById('progress').textContent =
  stepCount > 1 ? `Step ${stepCount} | ${rating}` : '';
```

Add moral rating to victory endings (in text):

```javascript
victory_smart: {
  title: "Victory!",
  text: `The water hits the sentinel's eyes. They hiss, sputter, and go dark.
    ... You did it. The fox was right. <em>Cold water.</em>
    <br><br>Final Rating: ${getMoralRating(moralScore)}`,
  isEnding: true,
  type: "victory",
  choices: [],
},
```

## What You Learned

| Concept | Where it appeared |
|---------|-------------------|
| **if/else** | Checking `if (scene.isEnding)` to apply ending styles |
| **=== (equals)** | Comparing `scene.type === 'victory'`, `scene.choices.length === 0` |
| **!== (not equals)** | Testing if values differ (compare to === examples) |
| **Ternary operator** | `scene.type === 'victory' ? 'outcome-victory' : 'outcome-defeat'` |
| **Logical AND (&&)** | Checking two conditions must both be true |
| **Logical OR (\|\|)** | Checking at least one condition is true |
| **Logical NOT (!)** | Inverting a condition with `!` |
| **switch statement** | Handling multiple ending types elegantly |
| **Truthy/Falsy** | `if (scene.choices.length)` treats 0 as false, >0 as true |
| **Data structures (objects)** | The `story` object maps scene keys to scene data |
| **Template literals** | Building dynamic HTML with backticks and `${}` |

## Building with Claude

Paste one of these prompts into Claude to extend your adventure:

**Prompt 1: Add a timer system**
"Add a 60-second countdown timer to the game. If the player doesn't choose within time, they get a 'timeout' ending. Use `setInterval` to tick down and display the time remaining. This teaches state that changes over time."

**Prompt 2: Add inventory tracking**
"The player should collect items as they explore. Create an `inventory = []` array. When certain scenes are reached, push items into it (e.g., reaching `fox_freed` adds 'water_vial'). Later scenes should check `inventory.includes('water_vial')` to unlock special choices."

**Prompt 3: Show all possible endings**
"After the game ends, display a summary of all possible endings in the story. Track which ending types exist and let the player see which ones they've found. Use a `Set` or object to track visited ending scenes."

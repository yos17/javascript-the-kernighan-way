# Chapter 12: Final Project

## Why Architecture Matters at Scale

A 50-line program can be a mess and still work. You can read it top to bottom in two minutes and understand everything. But when a program grows to 500 lines — and Quiz Battle is around that — mess becomes dangerous. You change one thing and something unrelated breaks. You read a function and can't tell what it's supposed to do. You need to add a feature and aren't sure where it belongs.

The solution isn't a magic framework. It's **organizing code around responsibilities**. The patterns here — separating state from rendering, grouping related functions, keeping data and display apart — are the same patterns React, Vue, and Redux implement. By building them yourself, you'll understand *why* those frameworks make the choices they do.

## The Program: Quiz Battle

A local multiplayer quiz game: 2–4 players compete on one keyboard, buzzing in to answer trivia questions fetched from a public API. Points scale with speed. A dramatic leaderboard at the end.

[Open quiz_battle.html](quiz_battle.html) to play.

## How It Works

### The Architecture: Four Responsibilities

The program divides into four sections, each with a single job:

```
┌─────────────────────────────────────────────────────┐
│                   gameState                         │
│  screen, players, scores, currentQuestion, ...      │
│  The single source of truth. No data anywhere else. │
└──────────────┬──────────────────────────────────────┘
               │ read by                 ↑ written by
               ▼                         │
┌──────────────────────┐  ┌──────────────────────────┐
│       render.*       │  │       events.*            │
│  Reads state.        │  │  Handles user actions.    │
│  Updates the DOM.    │  │  Updates state.           │
│  Never writes state. │  │  Calls render.            │
└──────────────────────┘  └──────────────────────────┘
                   ↑ called by events
           ┌───────────────────┐
           │     utils.*       │
           │  Pure helpers:    │
           │  fetch, shuffle,  │
           │  score calc, etc. │
           └───────────────────┘
```

This is the **state → render → events → state** cycle:

1. User does something (button click, keypress)
2. Event handler updates `gameState`
3. Event handler calls the appropriate `render.*()` function
4. Render reads `gameState` and updates the DOM
5. User sees the change and can act again

**Why this separation?** When something looks wrong on screen, you know to check the render function. When the logic is wrong, you check the event handler. When data is wrong, you check state. Bugs have addresses.

### gameState: Single Source of Truth

```javascript
const gameState = {
  screen: 'setup',          // which UI is visible
  players: [],              // [{ name, color, buzzKey }]
  scores: {},               // { 'Alice': 0, 'Bob': 0 }
  questions: [],            // fetched from API
  currentQuestion: null,    // the question being answered
  questionStartTime: 0,     // when the question appeared
  buzzerActive: false,      // can players buzz in?
  currentBuzzed: null,      // which player buzzed
};
```

**Why centralize all state here?** Consider the alternative: some state in DOM attributes (`data-player-score="42"`), some in module-level variables, some in function closures. Then a bug appears: the score display is wrong. Where do you look? You have to search everywhere.

With a single state object, there's only one place to look. And there's only one path to change it: through event handlers. This makes behavior predictable.

This is what **Redux** formalizes: a single store, actions that describe changes, reducers that apply changes. `gameState` is the store. The event handlers are the reducers (simplified).

### The Render Functions

Each render function reads from `gameState` and produces DOM. They never compute or modify state:

```javascript
const render = {
  gameScreen() {
    // 1. Build the HTML from gameState
    const html = gameState.players
      .map(p => `<div class="score-card" style="background:${p.color}">
                   <span>${p.name}</span>
                   <span>${gameState.scores[p.name]}</span>
                 </div>`)
      .join('');

    // 2. Set it on the element
    document.getElementById('scoreboard').innerHTML = html;

    // 3. Show the question
    render.displayQuestion();
  },

  displayQuestion() {
    const q = gameState.currentQuestion;
    document.getElementById('questionText').textContent = q.question;
    // ... display answer options ...
  }
};
```

**Why `.map().join('')` to build HTML?** This transforms an array of data (players) into a string of HTML. `map` produces an array of strings, `join('')` concatenates them. It's the functional alternative to a for loop that appends to a string.

### The Buzz-In System

Multiple players can press keys simultaneously. Only the first keypress should register:

```javascript
const keyMap = {
  'q': 0, 'w': 0,  // Player 1
  'o': 1, 'p': 1,  // Player 2
  'u': 2, 'i': 2,  // Player 3
  '7': 3, '8': 3   // Player 4
};

document.addEventListener('keydown', (e) => {
  const playerIdx = keyMap[e.key.toLowerCase()];

  if (gameState.buzzerActive
    && playerIdx !== undefined
    && gameState.currentBuzzed === null) {

    gameState.currentBuzzed = playerIdx;
    gameState.buzzerActive = false;
    render.showBuzz(playerIdx);
  }
}, { once: true });
```

`{ once: true }` is the key detail — this listener fires exactly once, then removes itself. A new listener is added when the next question begins. This prevents listeners stacking up and firing multiple times.

The guard `gameState.currentBuzzed === null` prevents two players buzzing in on the same keystroke if two keys are pressed simultaneously (extremely rare, but possible).

### Scoring by Speed

```javascript
utils.calculatePoints(timeRemaining) {
  const maxTime = 15;
  const basePoints = 100;
  const speedBonus = Math.round(basePoints * (timeRemaining / maxTime));
  return speedBonus + 50;  // minimum 50 points for correct answer
}
```

With 15 seconds left: `100 * (15/15) + 50 = 150` points.
With 5 seconds left: `100 * (5/15) + 50 ≈ 83` points.
With 0 seconds left: `100 * (0/15) + 50 = 50` points.

The `+ 50` guarantees even slow correct answers earn something. Incentivizes speed while not punishing careful thinking too harshly.

### Canvas Confetti

The celebration effect is the same particle system pattern from Chapter 9 (Breakout), now used for a different purpose:

```javascript
const particles = [];
for (let i = 0; i < 100; i++) {
  particles.push({
    x: Math.random() * canvas.width,
    y: -10,
    vx: (Math.random() - 0.5) * 8,  // spread left/right
    vy: Math.random() * 5 + 3,       // fall down
    color: randomColor(),
    size: Math.random() * 8 + 4,
  });
}

const animate = () => {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  particles.forEach((p, idx) => {
    p.x += p.vx;
    p.y += p.vy;
    p.vy += 0.1;  // gravity

    ctx.fillStyle = p.color;
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
    ctx.fill();

    if (p.y > canvas.height) particles.splice(idx, 1);
  });
  if (particles.length > 0) requestAnimationFrame(animate);
};

animate();
```

The game loop (`requestAnimationFrame`), velocity physics, and particle removal — you've seen all of this before.

---

## Guided Exercises

### Exercise 1: Custom Question Editor

**The Challenge:** Add a new screen where players can create their own questions before playing. Questions are saved to `localStorage`. A "Custom" category option in setup loads these instead of fetching from the API.

**Where to start:** Think about what a question looks like in the game. What fields does it have? Look at how API questions are structured in `utils.fetchQuestions`.

*(A question needs: `question` text, `correct_answer`, `incorrect_answers` array of 3.)*

---

**Step 1: Design the editor screen.**

Add a new screen to the HTML with a form for entering questions:

```html
<div id="editorScreen" class="screen hidden">
  <h2>Create Questions</h2>
  <form id="questionForm">
    <input id="qText" placeholder="Question" required>
    <input id="qCorrect" placeholder="Correct answer" required>
    <input id="qWrong1" placeholder="Wrong answer 1" required>
    <input id="qWrong2" placeholder="Wrong answer 2" required>
    <input id="qWrong3" placeholder="Wrong answer 3" required>
    <button type="submit">Add Question</button>
  </form>
  <div id="questionList"></div>
  <button id="startWithCustom">Play with these questions</button>
</div>
```

---

**Step 2: Save questions to localStorage.**

```javascript
function getCustomQuestions() {
  try {
    return JSON.parse(localStorage.getItem('customQuestions') || '[]');
  } catch { return []; }
}

function saveCustomQuestion(q) {
  const questions = getCustomQuestions();
  questions.push(q);
  localStorage.setItem('customQuestions', JSON.stringify(questions));
}
```

Wire up the form:

```javascript
document.getElementById('questionForm').addEventListener('submit', e => {
  e.preventDefault();
  const q = {
    question: document.getElementById('qText').value,
    correct_answer: document.getElementById('qCorrect').value,
    incorrect_answers: [
      document.getElementById('qWrong1').value,
      document.getElementById('qWrong2').value,
      document.getElementById('qWrong3').value,
    ],
  };
  saveCustomQuestion(q);
  renderQuestionList();
  e.target.reset();
});
```

**Think about it:** What should `renderQuestionList()` do? It should read from `getCustomQuestions()` and display each question so users can see what they've added (and potentially delete ones they don't want).

---

**Step 3: Use custom questions in the game.**

In `utils.fetchQuestions()` (or wherever questions are loaded), check if the custom category is selected:

```javascript
async fetchQuestions(amount, category) {
  if (category === 'custom') {
    const custom = getCustomQuestions();
    return utils.shuffleArray(custom).slice(0, amount);
  }
  // ... existing API fetch code ...
}
```

**Step 4: Add "Custom" to the category dropdown** in setup and make sure it routes to the editor screen before gameplay.

The pattern: data in `localStorage` → fetched the same way as API data → flows through the same game pipeline. The rest of the game doesn't know or care where questions came from.

---

### Exercise 2: Timed Round Mode

**The Challenge:** Add an alternative game mode: instead of fixed questions, players get 2 minutes to answer as many questions as possible. At the end, whoever has the highest score wins.

**Where to start:** The current game ends when `currentRound >= selectedRounds`. Timed mode ends when a countdown hits zero. What new state do you need?

*(A `mode` field: `'rounds'` or `'timed'`. A `timeLeft` field for timed mode. A `timerInterval` to manage the countdown.)*

---

**Step 1: Add mode to gameState.**

```javascript
const gameState = {
  // ...existing fields...
  mode: 'rounds',        // 'rounds' | 'timed'
  gameTimeLeft: 120,     // seconds for timed mode
  timerInterval: null,
};
```

---

**Step 2: Start the game timer in timed mode.**

When the game starts:

```javascript
if (gameState.mode === 'timed') {
  gameState.gameTimeLeft = 120;
  gameState.timerInterval = setInterval(() => {
    gameState.gameTimeLeft--;
    render.updateTimer();
    if (gameState.gameTimeLeft <= 0) {
      clearInterval(gameState.timerInterval);
      events.endGame();
    }
  }, 1000);
}
```

---

**Step 3: Skip the "rounds" check for timed mode.**

In the logic that advances questions, change:

```javascript
// Old:
if (gameState.currentRound >= gameState.selectedRounds) {
  events.endGame();
}

// New:
if (gameState.mode === 'rounds' && gameState.currentRound >= gameState.selectedRounds) {
  events.endGame();
}
// Timed mode: just keep going until the timer ends
```

**Step 4: Display the timer** in the game screen. The `render.updateTimer()` function updates a `<div id="gameTimer">` with the formatted time.

**Think about it:** What happens in timed mode when the API runs out of questions? *(You should pre-fetch extra questions, or re-fetch when the pool runs low. Add a check: if `gameState.questions.length - gameState.currentRound < 5`, fetch more.)*

---

### Exercise 3: Tournament Bracket

**The Challenge:** Add a tournament mode for exactly 4 players: semifinals (P1 vs P2, P3 vs P4), then a final between the two winners. Each match is a 3-question duel.

**Where to start:** Think about the tournament state. What does it need to track that regular game state doesn't?

*(Bracket structure: which players are in which match, who won each match, which stage we're in.)*

---

**Step 1: Define tournament state.**

```javascript
const tournament = {
  stage: 'semifinals',     // 'semifinals' | 'finals'
  matches: [
    { players: [0, 1], scores: [0, 0], winner: null },  // P1 vs P2
    { players: [2, 3], scores: [0, 0], winner: null },  // P3 vs P4
  ],
  finalists: [],
  champion: null,
};
```

---

**Step 2: Modify end-of-match logic.**

When a 3-question match ends, determine the winner and either start the next match or start the final:

```javascript
function endTournamentMatch(matchIndex) {
  const match = tournament.matches[matchIndex];
  const [score0, score1] = match.scores;
  match.winner = score0 >= score1 ? match.players[0] : match.players[1];

  if (matchIndex === 0) {
    // Semifinal 1 done — start semifinal 2
    tournament.finalists.push(match.winner);
    startMatch(1);
  } else if (matchIndex === 1) {
    // Both semifinals done — start the final
    tournament.finalists.push(match.winner);
    tournament.stage = 'finals';
    startFinal();
  }
}

function startFinal() {
  // Set up gameState to play with just the two finalists
  gameState.players = tournament.finalists.map(idx => allPlayers[idx]);
  gameState.selectedRounds = 3;
  gameState.currentRound = 0;
  // ... reset scores, fetch new questions, render game screen ...
}
```

---

**Step 3: Show the bracket screen.**

Between matches, show a bracket diagram:

```javascript
render.bracketScreen() {
  const container = document.getElementById('bracketScreen');
  container.innerHTML = `
    <h2>Tournament Bracket</h2>
    <div class="bracket">
      <div class="match">
        ${allPlayers[0].name} vs ${allPlayers[1].name}
        ${tournament.matches[0].winner !== null
          ? `→ Winner: ${allPlayers[tournament.matches[0].winner].name}`
          : ''}
      </div>
      <div class="match">
        ${allPlayers[2].name} vs ${allPlayers[3].name}
        ...
      </div>
    </div>
    <button id="startNextMatch">Begin Match</button>
  `;
}
```

**The complete picture:** The tournament is a state machine layered on top of the existing game. Each "match" is a complete run of the existing game loop with only 2 of the 4 players active. Reusing the existing game machinery by configuring `gameState` differently is cleaner than rewriting the game.

---

## What You Learned

| Concept | How We Used It | Real-World Connection |
|---------|---------------|----------------------|
| **Project architecture** | State / render / events / utils layers | React (state + JSX render), Redux (centralized store + reducers) |
| **Single source of truth** | `gameState` as the only data store | Redux, Zustand, Pinia — all centralize state |
| **Pure render functions** | Read state, update DOM, never compute | React components: pure functions of props/state |
| **Event-driven updates** | Events write state, then call render | React's `setState`, Vuex mutations, Redux dispatch |
| **async/await** | Fetch questions; fallback to local data | Every production app that talks to an API |
| **Canvas animation** | Confetti particle system | Same as Chapter 9's game loop — reused here |
| **Code organization** | Grouped by responsibility, not by type | Every maintainable codebase over ~500 lines |
| **Fallback strategy** | Local questions if API fails | Offline-first design; graceful degradation |

### The Path Forward

The patterns in Quiz Battle aren't just for games. They're the foundation of modern front-end development.

- **React** makes the render function automatic. You describe what the UI should look like given state; React calls `render()` when state changes.
- **Redux** formalizes `gameState` with strict rules: state is read-only, changes go through "actions" and "reducers". Same idea, stricter discipline.
- **Vue and Svelte** make state reactive — write to a reactive variable and the DOM updates automatically. `gameState.screen = 'game'` triggers `render.gameScreen()` for you.

What changes between these tools is the *mechanism*. What stays the same is the *idea*: keep your data in one place, describe how it maps to UI, let events flow through a clear path. You now understand that idea. The tools are just ways to automate it.

## Building with Claude

- "Add a live score ticker that shows points flying from the question card to the winner's score card when they answer correctly."
- "Add a 'double or nothing' wildcard: once per game, a player can bet their current score on the next question."
- "Add configurable question categories — show a multi-select grid so players can pick which topics to include."
- "Add a question explanation: after each answer, show a one-sentence fact about the correct answer. Use the AI to generate these."

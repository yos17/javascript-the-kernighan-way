# Chapter 3: Loops and Arrays

## Why Arrays and Loops Are Central

Almost every interesting program manages *collections* of things. A quiz game has multiple questions. A to-do list has multiple tasks. A music player has multiple tracks. A weather app has multiple days of forecast. Separate variables can't scale:

```javascript
// Terrible for 10 questions
const question1 = '...';
const question2 = '...';
const question3 = '...';
// ... 7 more
```

Arrays solve this. And loops are how you process arrays — applying the same operation to every element. Together, they're the foundation for managing collections of any size.

**Coming up:** Every chapter from here uses arrays. Chapter 5 stores all creatures in an array. Chapter 6's to-do list is an array of task objects. Chapter 10 fetches weather forecast data as an array of days. Chapter 12 manages all players as an array.

## The Program: Trivia Quiz Game

A complete quiz with timed questions, shuffled answers, score tracking, and a results screen. The questions are an array of objects. Array methods build the UI. A loop tracks progress.

Open `quiz_game.html` and play through it before reading the code.

## How It Works

### Arrays: Ordered Lists of Data

```javascript
const questions = [
  {
    question: "What does HTML stand for?",
    answers: ["HyperText Markup Language", "Home Tool Markup Language", "Hyperlinks and Text Markup Language", "Hyper Transactional Markup Language"],
    correct: 0  // index of the correct answer
  },
  // ... more questions
];
```

An array is an ordered list: `[item0, item1, item2, ...]`. Arrays can hold anything — strings, numbers, objects, even other arrays.

Access by index (zero-based): `questions[0]` is the first question. `questions[0].question` is its text.

Common array operations:

```javascript
questions.length           // how many items
questions[2]               // third item
questions.push(newItem)    // add to the end
questions.pop()            // remove from the end
questions.slice(0, 5)      // copy first 5 items (doesn't modify original)
questions.splice(2, 1)     // remove 1 item at index 2 (modifies original)
```

### for Loops: Repeating Operations

```javascript
for (let i = 0; i < questions.length; i++) {
  console.log(questions[i].question);
}
```

The three parts of a `for` loop:
1. `let i = 0` — initialize the counter
2. `i < questions.length` — keep going while this is true
3. `i++` — increment after each iteration

For iterating arrays where you need the index, `for` is right. When you just need each item:

```javascript
for (const question of questions) {
  console.log(question.question);  // 'of' gives you each item
}
```

`for...of` is cleaner when you don't need `i`. Use it for most array iteration.

### The Functional Array Methods

These three methods cover most collection operations:

**`.forEach()` — do something with each item:**
```javascript
questions.forEach((q, index) => {
  console.log(`${index + 1}. ${q.question}`);
});
```
Like `for...of`, but takes a callback with optional index. No return value.

**`.map()` — transform each item into something new:**
```javascript
const questionTexts = questions.map(q => q.question);
// → ['What does HTML stand for?', 'What year...', ...]
```
Returns a *new* array of transformed items. The original array is unchanged. Use when you need to convert data.

**`.filter()` — keep items that match a condition:**
```javascript
const hardQuestions = questions.filter(q => q.difficulty === 'hard');
```
Returns a *new* array of matching items. Use when you need a subset.

**Chaining** — the real power is combining these:
```javascript
const correctAnswers = userAnswers
  .filter((answer, i) => answer === questions[i].correct)
  .length;
```

**Why these methods instead of `for` loops?** The intent is clearer. `filter` says "I'm selecting" without the reader needing to trace through an if statement. `map` says "I'm transforming." These methods also don't need index variables, which is one less thing to get wrong.

**Coming up:** You'll use `.map().join('')` in Chapters 6, 7, and 12 to build HTML from data arrays. `.filter()` is used in Chapter 6's task filtering and Chapter 10's search history. Chapter 12 uses all three to build the score display.

### Shuffling: The Fisher-Yates Algorithm

The quiz shuffles its questions and answers. The correct algorithm for fair shuffling:

```javascript
function shuffle(array) {
  const arr = [...array];  // copy so we don't modify the original
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];  // swap
  }
  return arr;
}
```

Walk backwards from the end. At each position, pick a random position from 0 to current (inclusive) and swap. This is mathematically proven to produce equal probability for every arrangement.

The swap `[arr[i], arr[j]] = [arr[j], arr[i]]` uses **destructuring assignment** — it swaps two values without a temp variable.

### Closures: Functions that Remember

The quiz uses a closure to track state:

```javascript
function createTimer(duration, onTick, onComplete) {
  let timeLeft = duration;

  const interval = setInterval(() => {
    timeLeft--;         // accesses timeLeft from outer scope
    onTick(timeLeft);
    if (timeLeft <= 0) {
      clearInterval(interval);
      onComplete();
    }
  }, 1000);

  return () => clearInterval(interval);  // cancel function
}
```

The arrow function inside `setInterval` is a **closure** — it "closes over" `timeLeft` from the outer function. Even after `createTimer` returns, the callback remembers and can modify `timeLeft`.

Closures appear whenever a function needs to remember something across multiple calls. They're fundamental to JavaScript. You've used them without knowing it: event listeners are closures.

**Coming up:** Closures power the audio scheduler in Chapter 11 (the interval callback accesses `nextStepTime`), the game loop in Chapter 9, and event handlers everywhere.

### Array Methods Used in the Quiz

Beyond the big three:

```javascript
questions.findIndex(q => q.id === currentId)  // find index by condition
questions.some(q => q.answered)                // true if any match
questions.every(q => q.answered)               // true if all match
questions.reduce((sum, q) => sum + q.points, 0) // accumulate to single value
```

`reduce` is the most powerful and least intuitive. It processes each item and accumulates a result. The classic use: summing scores.

---

## Guided Exercises

### Exercise 1: Filter and Display Missed Questions

**The Challenge:** At the end of the quiz, show only the questions the user got wrong. Display the question text and the correct answer.

**Where to start:** When does the quiz end? What data do you have at that point? You need to know which questions were answered incorrectly.

*(You need to track user answers. What array would hold them?)*

---

**Step 1: Track user answers.**

Add an array to store what the user picked for each question:

```javascript
const userAnswers = [];  // same length as questions after the quiz
```

When the user selects an answer, push it:

```javascript
function answerQuestion(selectedIndex) {
  userAnswers.push(selectedIndex);
  // ... rest of answer logic
}
```

---

**Step 2: Filter wrong answers at the end.**

When the quiz ends, compute the missed questions:

```javascript
const missedQuestions = questions.filter((q, i) => userAnswers[i] !== q.correct);
```

**What does this do?** `filter` tests each question. `i` is the index. `userAnswers[i]` is what the user picked for that question. `q.correct` is the right answer index. If they don't match, the question stays in the result.

**Think about it:** What if the user didn't answer all questions? `userAnswers[i]` would be `undefined`. `undefined !== q.correct` is `true` — those questions would appear as "missed" even if skipped. How would you handle that? *(Add a check: `userAnswers[i] !== undefined && userAnswers[i] !== q.correct`)*

---

**Step 3: Display them.**

```javascript
const missedHTML = missedQuestions
  .map(q => `<div class="missed">
    <p><strong>${q.question}</strong></p>
    <p>Correct: ${q.answers[q.correct]}</p>
  </div>`)
  .join('');

document.getElementById('missed-section').innerHTML = missedHTML;
```

**The pattern:** data (`missedQuestions` array) → transform to HTML strings (`.map()`) → combine into one string (`.join('')`) → insert into DOM (`.innerHTML`). This is the core pattern for list rendering. You'll use it in every subsequent chapter.

---

### Exercise 2: Sort Questions by Difficulty

**The Challenge:** Add a `difficulty` property ('easy', 'medium', 'hard') to each question. Sort them by difficulty before the quiz starts, so easy questions come first.

**Where to start:** You need to understand `Array.sort()` with a compare function.

---

**Step 1: Add difficulty to your questions.**

```javascript
const questions = [
  { question: "...", answers: [...], correct: 0, difficulty: 'easy' },
  { question: "...", answers: [...], correct: 2, difficulty: 'hard' },
  // ...
];
```

---

**Step 2: Define sort order.**

The sort order needs a number for each difficulty level. An object as lookup:

```javascript
const DIFFICULTY_ORDER = { easy: 1, medium: 2, hard: 3 };
```

---

**Step 3: Sort the array.**

`Array.sort()` takes a compare function: positive means a goes after b, negative means a goes before b, zero means equal.

```javascript
const sorted = [...questions].sort((a, b) => {
  return DIFFICULTY_ORDER[a.difficulty] - DIFFICULTY_ORDER[b.difficulty];
});
```

**Why `[...questions].sort(...)`?** `sort()` modifies the array in place. The spread creates a copy so the original `questions` order is preserved. This is good practice — prefer not mutating original data.

**Test it:** Log `sorted.map(q => q.difficulty)` to the console. You should see `['easy', 'easy', 'medium', 'medium', 'hard', 'hard']` (or similar ordering).

---

### Exercise 3: Running Average Score Display

**The Challenge:** Show the running percentage score as the quiz progresses — after each question, update a display showing "3/5 correct (60%)".

**Where to start:** You need to track both the number of correct answers so far and the number of questions answered so far.

---

**Step 1: Add tracking variables.**

```javascript
let correctCount = 0;
let answeredCount = 0;
```

---

**Step 2: Update them when an answer is submitted.**

```javascript
function answerQuestion(selectedIndex) {
  answeredCount++;
  const isCorrect = selectedIndex === questions[currentQuestionIndex].correct;
  if (isCorrect) correctCount++;

  updateScoreDisplay();
  // ... rest of logic
}
```

---

**Step 3: Write updateScoreDisplay().**

```javascript
function updateScoreDisplay() {
  if (answeredCount === 0) return;

  const percentage = Math.round((correctCount / answeredCount) * 100);
  document.getElementById('running-score').textContent =
    `${correctCount}/${answeredCount} correct (${percentage}%)`;
}
```

**Think about it:** The `if (answeredCount === 0) return;` guard prevents dividing by zero on the first question. Division by zero gives `Infinity` in JavaScript — not an error, but not useful to display. Always guard division.

**The `.reduce()` alternative:** You could also compute this by accumulating results:

```javascript
const results = [];  // push { correct: true/false } for each answer
const stats = results.reduce(
  (acc, r) => ({ ...acc, correct: acc.correct + (r.correct ? 1 : 0), total: acc.total + 1 }),
  { correct: 0, total: 0 }
);
```

This is more functional — all the state lives in `results`, not in separate tracking variables. Both approaches work; the first is simpler to read, the second is more "pure."

---

## What You Learned

| Concept | What It Does | Coming Up In |
|---------|-------------|-------------|
| Arrays | Ordered lists of any data | Every chapter — the core data structure |
| `for` / `for...of` | Iterate over all items | Ch6 (rendering tasks), Ch9 (bricks), Ch11 (audio steps) |
| `.forEach()` | Run a function on each item | Ch5 (rendering creatures), Ch6, Ch12 |
| `.map()` | Transform each item, return new array | Ch6, Ch7, Ch12 (building HTML from data) |
| `.filter()` | Keep items matching a condition | Ch6 (task filters), Ch10 (search history) |
| `.reduce()` | Accumulate to single value | Ch12 (score totals), grouping data |
| `.sort()` | Sort in place (with compare function) | Sorting scores, ordering results |
| `.findIndex()` | Find index by condition | Ch6 (finding tasks to remove), Ch5 (creatures) |
| `.some()` / `.every()` | Test if any/all items match | Ch9 (all bricks destroyed?), Ch12 |
| Closures | Functions that capture outer scope | Ch9 (game loop), Ch11 (audio scheduler), everywhere |
| Destructuring swap | `[a, b] = [b, a]` | Shuffling, any swap without temp variable |

## Building with Claude

- "Add a time bonus: faster answers get more points. Track `Date.now()` when the question appears and when the answer is submitted."
- "Save the high score to `localStorage`. Show a 'New High Score!' message when it's beaten."
- "Add a category selector before the quiz. Filter questions by category using `.filter()`. Show only the selected category's questions."
- "Fetch trivia questions from the Open Trivia Database API (`https://opentdb.com/api.php?amount=10&type=multiple`) instead of hardcoded questions."

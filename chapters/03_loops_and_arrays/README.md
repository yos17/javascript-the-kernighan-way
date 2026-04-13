# Chapter 3: Loops and Arrays — Building a Trivia Quiz Game

## Brief Intro

In this chapter, we'll build a complete, real-world quiz game that tests your knowledge with timed questions. You'll use arrays to store questions, loops to process them, and array methods like `.map()`, `.filter()`, and `.forEach()` to build interactive features. This teaches the core patterns used in modern JavaScript — handling collections of data and transforming them into dynamic user interfaces.

## The Complete Program

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fun Trivia Quiz Game</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .container {
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
            max-width: 600px;
            width: 100%;
            padding: 40px;
            display: none;
        }

        .container.active {
            display: block;
            animation: slideIn 0.5s ease-out;
        }

        @keyframes slideIn {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .screen {
            display: none;
        }

        .screen.active {
            display: block;
        }

        h1 {
            color: #667eea;
            margin-bottom: 20px;
            text-align: center;
            font-size: 2em;
        }

        h2 {
            color: #764ba2;
            margin-bottom: 15px;
            font-size: 1.3em;
        }

        .start-screen {
            text-align: center;
        }

        .start-screen p {
            color: #555;
            font-size: 1.1em;
            margin-bottom: 30px;
            line-height: 1.6;
        }

        button {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            padding: 15px 40px;
            font-size: 1.1em;
            border-radius: 10px;
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
            font-weight: bold;
        }

        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 20px rgba(102, 126, 234, 0.4);
        }

        button:active {
            transform: translateY(0);
        }

        .quiz-screen {
            display: none;
        }

        .quiz-screen.active {
            display: block;
        }

        .progress {
            margin-bottom: 30px;
        }

        .progress-bar {
            background: #eee;
            height: 8px;
            border-radius: 4px;
            overflow: hidden;
            margin-bottom: 10px;
        }

        .progress-fill {
            background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
            height: 100%;
            transition: width 0.3s ease;
        }

        .progress-text {
            font-size: 0.9em;
            color: #666;
        }

        .score-display {
            display: flex;
            justify-content: space-between;
            margin-bottom: 20px;
            padding: 15px;
            background: #f5f5f5;
            border-radius: 10px;
        }

        .score-item {
            font-weight: bold;
            color: #667eea;
        }

        .timer {
            font-size: 2em;
            font-weight: bold;
            margin-bottom: 20px;
            text-align: center;
        }

        .timer.warning {
            color: #ff6b6b;
        }

        .timer-bar {
            background: #eee;
            height: 6px;
            border-radius: 3px;
            overflow: hidden;
            margin-bottom: 20px;
        }

        .timer-fill {
            background: linear-gradient(90deg, #51cf66 0%, #ff6b6b 100%);
            height: 100%;
            transition: width 0.1s linear;
        }

        .question {
            margin-bottom: 25px;
        }

        .question-text {
            font-size: 1.2em;
            color: #333;
            margin-bottom: 20px;
            font-weight: 600;
        }

        .options {
            display: grid;
            grid-template-columns: 1fr;
            gap: 12px;
        }

        .option {
            background: #f5f5f5;
            border: 2px solid #ddd;
            padding: 15px;
            border-radius: 10px;
            cursor: pointer;
            transition: all 0.3s ease;
            font-size: 1em;
            text-align: left;
        }

        .option:hover:not(.disabled) {
            border-color: #667eea;
            background: #f0f4ff;
        }

        .option.selected {
            border-color: #667eea;
            background: #e8ecff;
        }

        .option.correct {
            background: #d4edda;
            border-color: #51cf66;
            color: #155724;
            font-weight: bold;
        }

        .option.incorrect {
            background: #f8d7da;
            border-color: #ff6b6b;
            color: #721c24;
            font-weight: bold;
        }

        .option.disabled {
            cursor: not-allowed;
            opacity: 0.7;
        }

        .feedback {
            margin-top: 15px;
            font-size: 1em;
            font-weight: bold;
            text-align: center;
            min-height: 30px;
        }

        .feedback.correct {
            color: #51cf66;
        }

        .feedback.incorrect {
            color: #ff6b6b;
        }

        .results-screen {
            text-align: center;
        }

        .final-score {
            font-size: 3em;
            color: #667eea;
            margin: 30px 0;
            font-weight: bold;
        }

        .percentage {
            font-size: 1.5em;
            color: #764ba2;
            margin-bottom: 20px;
        }

        .message {
            font-size: 1.2em;
            color: #555;
            margin-bottom: 30px;
            line-height: 1.6;
            padding: 20px;
            background: #f5f5f5;
            border-radius: 10px;
        }

        .stats {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 30px;
        }

        .stat-box {
            background: #f5f5f5;
            padding: 20px;
            border-radius: 10px;
            border-left: 4px solid #667eea;
        }

        .stat-label {
            font-size: 0.9em;
            color: #666;
            margin-bottom: 5px;
        }

        .stat-value {
            font-size: 1.8em;
            font-weight: bold;
            color: #667eea;
        }

        .retry-button {
            background: linear-gradient(135deg, #51cf66 0%, #40c057 100%);
            margin-top: 20px;
        }

        .retry-button:hover {
            box-shadow: 0 10px 20px rgba(81, 207, 102, 0.4);
        }
    </style>
</head>
<body>
    <div class="container" id="mainContainer">
        <!-- Start Screen -->
        <div class="screen active" id="startScreen">
            <h1>🧠 Trivia Quiz Game</h1>
            <p>Think you're a trivia master? Test your knowledge with 10 fun questions!</p>
            <p>You have <strong>15 seconds</strong> per question. Ready?</p>
            <button onclick="startQuiz()">Start Quiz</button>
        </div>

        <!-- Quiz Screen -->
        <div class="screen" id="quizScreen">
            <div class="progress">
                <div class="progress-bar">
                    <div class="progress-fill" id="progressFill"></div>
                </div>
                <div class="progress-text" id="progressText">Question 1 of 10</div>
            </div>

            <div class="score-display">
                <div class="score-item">✓ Correct: <span id="correctCount">0</span></div>
                <div class="score-item">✗ Wrong: <span id="wrongCount">0</span></div>
            </div>

            <div class="timer" id="timer">15</div>
            <div class="timer-bar">
                <div class="timer-fill" id="timerFill"></div>
            </div>

            <div class="question">
                <div class="question-text" id="questionText"></div>
                <div class="options" id="optionsContainer"></div>
                <div class="feedback" id="feedback"></div>
            </div>
        </div>

        <!-- Results Screen -->
        <div class="screen" id="resultsScreen">
            <h1>🎉 Quiz Complete!</h1>
            <div class="final-score" id="finalScore">0/10</div>
            <div class="percentage" id="percentageText">0%</div>
            <div class="message" id="resultMessage"></div>
            <div class="stats">
                <div class="stat-box">
                    <div class="stat-label">Total Time</div>
                    <div class="stat-value" id="totalTime">0s</div>
                </div>
                <div class="stat-box">
                    <div class="stat-label">Avg Per Question</div>
                    <div class="stat-value" id="avgTime">0s</div>
                </div>
            </div>
            <button onclick="startQuiz()">Try Again</button>
        </div>
    </div>

    <script>
        // Questions array with destructuring-friendly structure
        const questions = [
            {
                question: "What is the capital of Japan?",
                options: ["Tokyo", "Osaka", "Kyoto", "Hiroshima"],
                correct: 0
            },
            {
                question: "How many continents are there?",
                options: ["5", "6", "7", "8"],
                correct: 2
            },
            {
                question: "Which planet is closest to the Sun?",
                options: ["Venus", "Mercury", "Earth", "Mars"],
                correct: 1
            },
            {
                question: "What is the largest mammal in the world?",
                options: ["African Elephant", "Blue Whale", "Giraffe", "Hippopotamus"],
                correct: 1
            },
            {
                question: "In what year did the Titanic sink?",
                options: ["1912", "1915", "1920", "1905"],
                correct: 0
            },
            {
                question: "What is the smallest country in the world by area?",
                options: ["Monaco", "Liechtenstein", "Vatican City", "San Marino"],
                correct: 2
            },
            {
                question: "Which superhero is also known as Clark Kent?",
                options: ["Batman", "Superman", "Spider-Man", "Iron Man"],
                correct: 1
            },
            {
                question: "How many sides does a hexagon have?",
                options: ["4", "5", "6", "8"],
                correct: 2
            },
            {
                question: "What is the freezing point of water in Celsius?",
                options: ["0°C", "32°C", "-40°C", "100°C"],
                correct: 0
            },
            {
                question: "Which country is home to the Eiffel Tower?",
                options: ["Germany", "Italy", "France", "Spain"],
                correct: 2
            }
        ];

        // Game state
        let state = {
            currentQuestion: 0,
            correct: 0,
            wrong: 0,
            shuffledQuestions: [],
            timeStarted: 0,
            questionStartTime: 0,
            answered: false
        };

        // Fisher-Yates shuffle algorithm
        function shuffleArray(arr) {
            const shuffled = [...arr];
            for (let i = shuffled.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
            }
            return shuffled;
        }

        function startQuiz() {
            state = {
                currentQuestion: 0,
                correct: 0,
                wrong: 0,
                shuffledQuestions: shuffleArray(questions),
                timeStarted: Date.now(),
                questionStartTime: Date.now(),
                answered: false
            };

            showScreen('quizScreen');
            displayQuestion();
            startTimer();
        }

        function displayQuestion() {
            const { question, options } = state.shuffledQuestions[state.currentQuestion];

            document.getElementById('questionText').textContent = question;
            document.getElementById('progressText').textContent =
                `Question ${state.currentQuestion + 1} of ${questions.length}`;

            const progressPercent = ((state.currentQuestion) / questions.length) * 100;
            document.getElementById('progressFill').style.width = progressPercent + '%';

            document.getElementById('feedback').textContent = '';
            document.getElementById('feedback').className = 'feedback';

            // Use .map() to create option elements
            const optionsHtml = options.map((option, index) =>
                `<button class="option" onclick="selectAnswer(${index})">${option}</button>`
            ).join('');

            document.getElementById('optionsContainer').innerHTML = optionsHtml;
            state.answered = false;
        }

        function selectAnswer(selectedIndex) {
            if (state.answered) return;

            state.answered = true;
            clearInterval(state.timerInterval);

            const { correct: correctIndex, options } = state.shuffledQuestions[state.currentQuestion];
            const optionButtons = document.querySelectorAll('.option');

            // Disable all options
            optionButtons.forEach(btn => btn.classList.add('disabled'));

            if (selectedIndex === correctIndex) {
                state.correct++;
                optionButtons[selectedIndex].classList.add('correct');
                document.getElementById('feedback').textContent = '✓ Correct!';
                document.getElementById('feedback').classList.add('correct');
            } else {
                state.wrong++;
                optionButtons[selectedIndex].classList.add('incorrect');
                optionButtons[correctIndex].classList.add('correct');
                document.getElementById('feedback').textContent = `✗ Wrong! The answer is "${options[correctIndex]}"`;
                document.getElementById('feedback').classList.add('incorrect');
            }

            // Update score display
            document.getElementById('correctCount').textContent = state.correct;
            document.getElementById('wrongCount').textContent = state.wrong;

            // Auto-advance after 1.5 seconds
            setTimeout(nextQuestion, 1500);
        }

        function nextQuestion() {
            state.currentQuestion++;
            if (state.currentQuestion < questions.length) {
                state.questionStartTime = Date.now();
                displayQuestion();
                startTimer();
            } else {
                showResults();
            }
        }

        function startTimer() {
            let timeLeft = 15;
            const timerDisplay = document.getElementById('timer');
            const timerFill = document.getElementById('timerFill');

            timerDisplay.textContent = timeLeft;
            timerDisplay.classList.remove('warning');
            timerFill.style.width = '100%';

            state.timerInterval = setInterval(() => {
                timeLeft--;
                const percent = (timeLeft / 15) * 100;

                timerDisplay.textContent = timeLeft;
                timerFill.style.width = percent + '%';

                if (timeLeft <= 5) {
                    timerDisplay.classList.add('warning');
                }

                if (timeLeft <= 0) {
                    clearInterval(state.timerInterval);
                    selectAnswer(-1);
                }
            }, 1000);
        }

        function showResults() {
            const percentage = Math.round((state.correct / questions.length) * 100);
            const totalTime = Math.round((Date.now() - state.timeStarted) / 1000);
            const avgTime = Math.round(totalTime / questions.length);

            document.getElementById('finalScore').textContent = `${state.correct}/${questions.length}`;
            document.getElementById('percentageText').textContent = `${percentage}%`;
            document.getElementById('totalTime').textContent = `${totalTime}s`;
            document.getElementById('avgTime').textContent = `${avgTime}s`;

            // Personalized message based on score
            let message = '';
            if (percentage === 100) {
                message = '🏆 Perfect score! You\'re a trivia champion!';
            } else if (percentage >= 80) {
                message = '⭐ Excellent work! You really know your stuff!';
            } else if (percentage >= 60) {
                message = '👍 Good job! You got most of them right!';
            } else if (percentage >= 40) {
                message = '💪 Not bad! Keep learning and try again!';
            } else {
                message = '📚 Keep practicing! Every question teaches you something new!';
            }

            document.getElementById('resultMessage').textContent = message;
            showScreen('resultsScreen');
        }

        function showScreen(screenId) {
            // Hide all screens
            document.querySelectorAll('.screen').forEach(screen => {
                screen.classList.remove('active');
            });

            // Show target screen
            document.getElementById(screenId).classList.add('active');
        }

        // Shuffle questions on page load using .filter() and .forEach() for practice
        document.addEventListener('DOMContentLoaded', () => {
            // Use .filter() to verify we have questions (optional practice)
            const validQuestions = questions.filter(q => q.question && q.options.length === 4);

            // Use .forEach() to log question count (optional practice)
            let count = 0;
            validQuestions.forEach(() => count++);

            console.log(`Quiz loaded with ${count} questions`);
        });
    </script>
</body>
</html>
```

## Walkthrough

### Arrays

**What they are:** An array is a container that holds multiple values in order. Think of it like a list of items where each item has a position.

**How the questions array works:**

```javascript
const questions = [
    {
        question: "What is the capital of Japan?",
        options: ["Tokyo", "Osaka", "Kyoto", "Hiroshima"],
        correct: 0
    },
    // ... more questions
];
```

Each question is an object inside the `questions` array. The array lets us store all 10 questions together and access them one by one. We can get question 3 with `questions[2]` (remember: counting starts at 0).

### Array Destructuring

**What it is:** Destructuring lets you unpack values from an array or object into separate variables in one line.

**In the code:**

```javascript
const { question, options } = state.shuffledQuestions[state.currentQuestion];
```

Instead of writing:
```javascript
const currentQ = state.shuffledQuestions[state.currentQuestion];
const question = currentQ.question;
const options = currentQ.options;
```

Destructuring does it in one clean line. It grabs the `question` and `options` properties from the current question object.

### for loop

**What it is:** A loop that runs a block of code a certain number of times. It needs a counter that starts, a stopping point, and an increment rule.

**In the code (inside the shuffle function):**

```javascript
for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
}
```

This loop starts at the last element (`shuffled.length - 1`), counts backward (`i--`), and stops when `i` reaches 0. It swaps random elements to shuffle the array. The `for` loop is perfect here because we need precise control over the counting.

### for...of loop

**What it is:** A loop that goes through each item in an array one at a time, without needing to manage a counter.

**In the code:**

```javascript
optionButtons.forEach(btn => btn.classList.add('disabled'));
```

This isn't exactly `for...of`, but they're cousins. A `for...of` version would look like:

```javascript
for (const btn of optionButtons) {
    btn.classList.add('disabled');
}
```

`for...of` is cleaner when you just want to visit each item. You don't need to think about indices or counters—JavaScript does it for you.

### .map()

**What it is:** `.map()` transforms each item in an array using a function and returns a new array with the transformed items.

**In the code:**

```javascript
const optionsHtml = options.map((option, index) =>
    `<button class="option" onclick="selectAnswer(${index})">${option}</button>`
).join('');
```

For each option (like "Tokyo", "Osaka"), `.map()` wraps it in a button element. It transforms the array `["Tokyo", "Osaka", ...]` into an array of HTML buttons `["<button>Tokyo</button>", "<button>Osaka</button>", ...]`. Then `.join('')` glues them together into one string.

### .filter()

**What it is:** `.filter()` keeps only the items in an array that pass a test, and returns a new array.

**In the code:**

```javascript
const validQuestions = questions.filter(q => q.question && q.options.length === 4);
```

This filters the `questions` array to keep only questions that have a `question` field AND exactly 4 options. Any question missing these gets removed. It's a safety check.

### .forEach()

**What it is:** `.forEach()` runs a function for each item in an array. It's like a loop, but simpler—you don't manage a counter.

**In the code:**

```javascript
validQuestions.forEach(() => count++);
```

This counts how many valid questions we have. For each question in `validQuestions`, the function increments `count` by 1. It's a clean way to iterate when you just need to do something with each item.

### Spread Operator (for shuffling)

**What it is:** The `...` operator copies an array.

**In the code:**

```javascript
const shuffled = [...arr];
```

This creates a copy of the `arr` array. Why? Because we don't want to mess up the original `questions` array when we shuffle. We shuffle the copy instead. This keeps the original safe.

## Try It

1. **Change the timer**: Find `let timeLeft = 15;` and change 15 to 10 or 20 seconds. Shorter makes it harder, longer makes it easier.

2. **Add more questions**: Add new objects to the `questions` array. Copy an existing question block, paste it at the end, and change the text and correct answer index.

3. **Change the quiz length**: Find `questions.length` and experiment with taking only the first 5 questions by modifying the `shuffleArray` call to slice the array.

4. **Customize the messages**: In the `showResults()` function, change the messages like "Perfect score!" to your own phrases.

## Exercises

### Exercise 1: Easy
**Task:** Change the `shuffleArray` function to NOT shuffle. Instead, return the array in reverse order. (Hint: use `.reverse()`.)

**What you'll learn:** How `.reverse()` works on arrays and how it changes the quiz experience.

### Exercise 2: Medium
**Task:** Add a difficulty level. Create a new property `difficulty` on each question object (like `difficulty: "easy"` or `difficulty: "hard"`). Filter the questions array to show only "easy" questions on the first run. Hint: Use `.filter()` to select only questions where `difficulty === "easy"`.

**What you'll learn:** How to filter arrays based on properties and use filters to customize behavior.

### Exercise 3: Hard
**Task:** Add a categories feature. Add a `category` property to each question (like `"Science"`, `"Geography"`, `"PopCulture"`). On the results screen, show a breakdown of how many you got right per category. Hint: Use `.filter()` for each category to count correct answers in each one.

**What you'll learn:** How to use `.filter()` multiple times to group and analyze data.

## Solutions

### Exercise 1 Solution

```javascript
function shuffleArray(arr) {
    return [...arr].reverse();
}
```

This uses the spread operator to copy the array, then `.reverse()` returns it backwards. Now questions appear in reverse order.

### Exercise 2 Solution

```javascript
function startQuiz() {
    const easyQuestions = questions.filter(q => q.difficulty === "easy");
    
    state = {
        currentQuestion: 0,
        correct: 0,
        wrong: 0,
        shuffledQuestions: shuffleArray(easyQuestions),
        timeStarted: Date.now(),
        questionStartTime: Date.now(),
        answered: false
    };
    // ... rest of function
}
```

Add `difficulty: "easy"` or `difficulty: "hard"` to each question object in the `questions` array. Now the quiz only uses easy questions.

### Exercise 3 Solution

```javascript
function showResults() {
    const percentage = Math.round((state.correct / questions.length) * 100);
    
    // Count correct answers by category
    const categories = ["Science", "Geography", "PopCulture"];
    const categoryBreakdown = categories.map(cat => {
        const categoryQuestions = questions.filter(q => q.category === cat);
        const categoryCorrect = state.shuffledQuestions
            .filter((_, index) => index < state.currentQuestion)
            .filter(q => q.category === cat)
            .length;
        return `${cat}: ${categoryCorrect}/${categoryQuestions.length}`;
    }).join(", ");
    
    console.log("Breakdown: " + categoryBreakdown);
    // ... rest of function
}
```

Add `category: "Science"` to each question. This shows a breakdown of your score by category on the results screen.

## What You Learned

| Concept | Where it appeared |
|---------|-------------------|
| Arrays | `const questions = [...]` stores all quiz questions |
| Array Destructuring | `const { question, options } = ...` unpacks object properties |
| for loop | Inside `shuffleArray()` to loop backward through the array |
| for...of loop | `for (const btn of optionButtons)` iterates without an index |
| .map() | Transforms options into HTML button elements |
| .filter() | Filters questions by validity; selects answers by category |
| .forEach() | Loops through elements to add classes or count items |
| Spread Operator | `[...arr]` copies the array before shuffling |
| Template Literals | `` `<button>...</button>` `` embeds variables in strings |
| Arrow Functions | `(option) => {...}` simplifies function syntax |

## Building with Claude

Want to extend this project? Try these prompts:

1. **"Add a leaderboard. Store the top 5 scores in localStorage and display them on the start screen. Use .sort() and .slice() to get the top 5."** — Learn persistence and data manipulation.

2. **"Add a hint system. Each question gets 2 free hints that eliminate wrong answers. Use .filter() to show only the remaining options when a hint is used."** — Practice conditional logic and array filtering.

3. **"Create a two-player mode. Players alternate questions, and the one with the higher score wins. Display a final winner message."** — Learn state management for multiple players and comparison logic.
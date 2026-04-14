# Chapter 9: Animation and Games

## Why Games Teach Programming

A game requires you to confront the core challenges of computer science all at once: managing state that changes 60 times per second, detecting when objects collide, responding to user input without lag, and keeping all these moving parts coherent. These aren't game-specific problems. They're the same problems server code faces under load, the same problems UI frameworks solve for animation, the same problems operating systems solve for concurrency.

Games are also unforgiving. If your to-do list has a bug, the task looks wrong. If your game has a bug, the ball phases through the paddle and the game is unplayable. The immediate, visual feedback makes bugs obvious and fixes satisfying.

## The Program: Breakout

A classic arcade game: paddle at the bottom, ball bouncing upward, bricks at the top to destroy. Arrow keys or mouse control the paddle. Break all bricks to advance to the next level. Lose all lives to end the game.

[Open breakout.html](breakout.html) to play.

## How It Works

### The Game Loop

Every game runs a loop that repeats as fast as the screen refreshes — typically 60 times per second. Each iteration:
1. Update game state (move objects, check collisions)
2. Draw everything to the canvas
3. Schedule the next iteration

```
requestAnimationFrame calls gameLoop
        ↓
   update()  ← move ball, check collisions, update score
        ↓
   draw()    ← clear canvas, draw bricks, paddle, ball
        ↓
requestAnimationFrame(gameLoop)  ← schedule next frame
        ↓
   (16.7ms later) ← browser calls gameLoop again
```

The code:

```javascript
const gameLoop = () => {
  update();
  draw();
  requestAnimationFrame(gameLoop);
};

gameLoop();  // start the loop
```

**Why `requestAnimationFrame` instead of `setInterval`?**

`setInterval(gameLoop, 16)` would also run ~60 times per second. But `requestAnimationFrame` is better for three reasons:

1. **Sync with display**: It fires right before the browser paints a new frame. `setInterval` fires on a timer that may not align with screen refreshes, causing **tearing** (partial old frame + partial new frame visible simultaneously).

2. **Pause when hidden**: If the user switches tabs, `requestAnimationFrame` stops firing. `setInterval` keeps running, wasting CPU and causing a "catch-up" explosion of updates when the tab is refocused.

3. **Browser-optimized**: The browser can batch, throttle, and schedule these callbacks for maximum efficiency.

### Drawing on Canvas (Review from Chapter 8)

```javascript
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

// Every frame: clear, then redraw
ctx.clearRect(0, 0, canvas.width, canvas.height);

// Draw a brick
ctx.fillStyle = '#FF6B6B';
ctx.fillRect(brick.x, brick.y, brick.width, brick.height);

// Draw the ball (a circle)
ctx.beginPath();
ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
ctx.fillStyle = '#FFE66D';
ctx.fill();
```

The key insight: **you redraw the entire scene every frame**. This sounds wasteful, but it's the right approach. Incremental updates ("erase just the ball's old position") are error-prone. Full redraw is simple and correct.

### Ball Velocity and Physics

The ball moves by adding its velocity to its position each frame:

```javascript
ball.x += ball.dx;  // dx = horizontal velocity (pixels per frame)
ball.y += ball.dy;  // dy = vertical velocity
```

Bouncing off walls reverses the appropriate component:

```javascript
if (ball.x - ball.radius < 0 || ball.x + ball.radius > canvas.width) {
  ball.dx = -ball.dx;  // reverse horizontal
}
if (ball.y - ball.radius < 0) {
  ball.dy = -ball.dy;  // reverse vertical (hit top wall)
}
```

**Visual**:
```
  ball.dy = -5  (moving up)
       ↓
   hits top wall
       ↓
  ball.dy = +5  (now moving down)
```

When hitting the paddle, you also add "spin" — the ball's horizontal velocity changes based on where it hit the paddle:

```javascript
const hitPos = (ball.x - (paddle.x + paddle.width / 2)) / (paddle.width / 2);
// hitPos is -1 (far left) to +1 (far right)
ball.dx = hitPos * ballSpeed * 1.2;
ball.dy = -Math.abs(ball.dy);  // always bounce up
```

This makes skill matter: hitting with the left edge sends the ball left, right edge sends it right.

### Collision Detection: AABB

The game uses **Axis-Aligned Bounding Box** (AABB) collision detection. For a circle colliding with a rectangle:

```
Rectangle:
  ┌─────────────────┐  y
  │                 │
  │    (cx, cy)     │  y + height
  └─────────────────┘
  x               x + width

Ball (circle) center: (ball.x, ball.y), radius: ball.radius

Step 1: Find closest point on rectangle to circle center:
  closestX = clamp(ball.x, rect.x, rect.x + rect.width)
  closestY = clamp(ball.y, rect.y, rect.y + rect.height)

Step 2: Distance from circle center to closest point:
  dx = ball.x - closestX
  dy = ball.y - closestY

Step 3: Collision if distance < radius:
  dx*dx + dy*dy < ball.radius * ball.radius
```

In code:

```javascript
function aabbCollision(ball, rect) {
  const closestX = Math.max(rect.x, Math.min(ball.x, rect.x + rect.width));
  const closestY = Math.max(rect.y, Math.min(ball.y, rect.y + rect.height));
  const dx = ball.x - closestX;
  const dy = ball.y - closestY;
  return dx * dx + dy * dy < ball.radius * ball.radius;
}
```

**Why squared distance instead of `Math.sqrt()`?** `Math.sqrt` is expensive to compute. Comparing `dx*dx + dy*dy < r*r` is mathematically equivalent to comparing `distance < radius` but avoids the square root entirely. This matters when checking 30+ bricks every frame.

### Input Handling

Input is tracked in an object, not reacted to directly:

```javascript
const keys = {};
window.addEventListener('keydown', e => { keys[e.key] = true;  });
window.addEventListener('keyup',   e => { keys[e.key] = false; });

// In update() each frame:
if (keys['ArrowLeft'])  paddle.x -= paddle.speed;
if (keys['ArrowRight']) paddle.x += paddle.speed;
```

**Why not move the paddle directly in the keydown listener?** The event fires once when the key is pressed, then repeatedly (keyboard repeat delay). The `keys` object approach checks the current state every frame — smooth, lag-free movement.

Mouse control uses smooth interpolation:

```javascript
// In update():
const target = Math.max(0, Math.min(mouseX - paddle.width / 2, canvas.width - paddle.width));
paddle.x += (target - paddle.x) * 0.15;  // easing
```

The `0.15` factor creates acceleration: the paddle doesn't snap to the mouse, it follows with slight lag. This feels more natural than instant movement.

### Game State Machine

```
  'start'
     │  press Space
     ▼
  'playing'
     │  ball falls off bottom AND lives = 0   │  all bricks destroyed
     ▼                                         ▼
  'gameOver'                              'levelComplete'
     │  press Space                            │  (automatically advances)
     ▼                                         ▼
  'start'                                   'playing' (next level)
```

```javascript
let gameState = 'start';

const update = () => {
  if (gameState !== 'playing') return;  // guard: only update when playing
  // ... physics, collision ...
};

const draw = () => {
  // ... draw game objects ...
  if (gameState === 'start')    drawOverlay('BREAKOUT', 'Press SPACE to start');
  if (gameState === 'gameOver') drawOverlay('GAME OVER', 'Press SPACE to retry');
};
```

The state machine keeps logic clean: `update()` does nothing unless playing. `draw()` overlays different screens based on state. Bugs stay contained — a state can only do what its code allows.

---

## Guided Exercises

### Exercise 1: Power-Ups

**The Challenge:** Some bricks drop a power-up when destroyed. Power-ups fall down the screen. Catching one with the paddle gives an effect: wider paddle, or an extra life.

**Where to start:** A power-up is just another object with position and velocity. Think about what data a power-up needs, then think about where it gets created.

*(What array would store power-ups? What creates one? What destroys one?)*

---

**Step 1: Create the power-up data structure.**

```javascript
let powerUps = [];

const POWER_UP_TYPES = { WIDE: 'wide', LIFE: 'life' };

function createPowerUp(x, y) {
  if (Math.random() > 0.15) return;  // only 15% chance per brick
  powerUps.push({
    x, y,
    width: 20, height: 10,
    type: Math.random() < 0.5 ? POWER_UP_TYPES.WIDE : POWER_UP_TYPES.LIFE,
    dy: 2,  // falls down
  });
}
```

**When should `createPowerUp` be called?** In the brick collision code, right after marking a brick as destroyed. What position should you pass? The brick's center.

---

**Step 2: Update and draw power-ups.**

In `update()`, after the existing collision code:

```javascript
for (let i = powerUps.length - 1; i >= 0; i--) {
  const pu = powerUps[i];
  pu.y += pu.dy;

  // Check if paddle catches it
  if (aabbCollision({ x: pu.x + pu.width/2, y: pu.y + pu.height/2, radius: 10 }, paddle)) {
    applyPowerUp(pu.type);
    powerUps.splice(i, 1);
  } else if (pu.y > canvas.height) {
    powerUps.splice(i, 1);  // fell off screen
  }
}
```

**Why iterate backwards?** When you `splice(i, 1)`, the array shrinks. If you iterate forward, you skip the element that moved into position `i` after the splice. Backwards iteration avoids this: by the time you remove element `i`, you've already processed everything after it.

In `draw()`:

```javascript
for (const pu of powerUps) {
  ctx.fillStyle = pu.type === POWER_UP_TYPES.WIDE ? '#FFD700' : '#4ECDC4';
  ctx.fillRect(pu.x, pu.y, pu.width, pu.height);
}
```

---

**Step 3: Apply the power-up effect.**

```javascript
function applyPowerUp(type) {
  if (type === POWER_UP_TYPES.WIDE) {
    paddle.width = 150;
    setTimeout(() => { paddle.width = 100; }, 5000);  // revert after 5 seconds
  } else if (type === POWER_UP_TYPES.LIFE) {
    lives = Math.min(lives + 1, 5);
  }
}
```

**Think about it:** What happens if the player gets two "wide paddle" power-ups? The first `setTimeout` will revert the paddle after 5 seconds, potentially undoing the second one. How would you fix this? *(Clear the previous timeout before setting a new one: `clearTimeout(wideTimer); wideTimer = setTimeout(...)`)*

---

### Exercise 2: Web Audio Sound Effects

**The Challenge:** Add three sounds: a "pop" when the ball bounces off the paddle, a "crack" when a brick breaks, and a low "thud" when a life is lost.

**Where to start:** Sound synthesis is Chapter 11's territory, but here's a preview. The Web Audio API can generate simple tones programmatically — no audio files needed.

---

**Step 1: Create the audio context (lazily).**

```javascript
let audioCtx = null;

function getAudio() {
  if (!audioCtx) audioCtx = new AudioContext();
  return audioCtx;
}
```

Creating on first use (lazy init) is important — browsers require a user gesture before audio can play.

---

**Step 2: Write a tone generator.**

```javascript
function playTone(frequency, duration, type = 'sine') {
  const ctx  = getAudio();
  const osc  = ctx.createOscillator();
  const gain = ctx.createGain();

  osc.connect(gain);
  gain.connect(ctx.destination);

  osc.type = type;
  osc.frequency.value = frequency;
  gain.gain.setValueAtTime(0.3, ctx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + duration);

  osc.start(ctx.currentTime);
  osc.stop(ctx.currentTime + duration);
}
```

**How does this work?** An oscillator generates a tone at a given frequency. A gain node controls volume. `exponentialRampToValueAtTime` creates a fade — the sound starts at 0.3 and decays to near-zero over `duration` seconds. This creates a click/pluck effect rather than a constant tone.

---

**Step 3: Call playTone at the right moments.**

Find the three places in your code:
- Paddle collision → `playTone(400, 0.1)` (short, mid-pitch)
- Brick destroyed  → `playTone(600, 0.15, 'square')` (brighter)
- Life lost        → `playTone(150, 0.4)` (low, longer)

**The complete picture:** Sound synthesis will be covered deeply in Chapter 11. For now, appreciate how the same audio graph pattern (oscillator → gain → destination) generates all three sounds by just changing numbers.

---

### Exercise 3: High Score Persistence

**The Challenge:** Save the highest score to `localStorage`. Show it on the start screen and game over screen. Display a "NEW HIGH SCORE!" celebration when it's beaten.

**Where to start:** `localStorage` was introduced in Chapter 6's exercises. The pattern is the same: `JSON.stringify` to save, `JSON.parse` to load. What data needs to persist? Just one number.

---

**Step 1: Load and save the high score.**

```javascript
let highScore = parseInt(localStorage.getItem('breakoutHighScore') || '0');

function saveHighScore() {
  if (score > highScore) {
    highScore = score;
    localStorage.setItem('breakoutHighScore', String(highScore));
    return true;  // new high score!
  }
  return false;
}
```

---

**Step 2: Check at game over.**

```javascript
// In the logic that handles when lives run out:
const isNewHighScore = saveHighScore();
gameState = 'gameOver';
```

---

**Step 3: Display in the draw function.**

```javascript
// On the start screen:
ctx.fillText(`High Score: ${highScore}`, canvas.width / 2, canvas.height / 2 + 60);

// On the game over screen, if it's a new record:
if (isNewHighScore) {
  ctx.fillStyle = '#FFD700';
  ctx.fillText('NEW HIGH SCORE!', canvas.width / 2, canvas.height / 2 + 40);
}
```

**But there's a problem:** `isNewHighScore` is a local variable inside your game-over logic. The `draw()` function needs it later. How do you share it?

*(You need a module-level variable: `let isNewHighScore = false;`. Set it when you call `saveHighScore()`, use it in `draw()`. This is the same "single source of truth in state variables" pattern we've used throughout.)*

---

## What You Learned

| Concept | How We Used It | Real-World Use |
|---------|---------------|----------------|
| `requestAnimationFrame` | 60fps game loop synced with monitor | All smooth browser animations; React's reconciler batches updates similarly |
| Canvas API | Drew bricks, paddle, ball every frame | Game engines, data viz (D3), image editors |
| AABB Collision | Ball vs. paddle, ball vs. bricks | Physics engines, UI drag-and-drop hit testing |
| Velocity | `ball.x += ball.dx` every frame | CSS transitions, tweening libraries (GSAP) |
| Key state tracking | `keys` object checked in update | Game input; any keyboard shortcut system |
| Game state machine | `start` → `playing` → `gameOver` | UI flows, wizard steps, media player states |
| Backwards array iteration | Removing items while iterating | Filtering live collections in game engines |

### Real-World Connections

- **requestAnimationFrame** is used by every animation library (GSAP, Framer Motion, Three.js). They all use the same "update state, redraw, request next frame" loop.
- **Phaser.js** is a JavaScript game framework that wraps everything you've built here: the game loop, input handling, collision detection, and state management into a polished API.
- **Three.js** uses the same render loop for 3D. The difference is WebGL instead of Canvas 2D, but the `requestAnimationFrame` game loop is identical.
- **Physics engines** (Matter.js, Box2D) are sophisticated collision detection systems. AABB is the simplest case; they handle circles, polygons, and continuous collision detection for fast-moving objects.

## Building with Claude

- "Add a multiball power-up that spawns a second ball for 10 seconds."
- "Add particle effects: when a brick breaks, spawn 8–12 small squares that fly outward and fade."
- "Add a level editor: show a grid, click cells to toggle bricks, save/load the layout to localStorage."
- "Make bricks take multiple hits: some need 2 or 3 hits to destroy. Show hit points as a number on the brick."

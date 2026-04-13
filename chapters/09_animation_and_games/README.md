# Chapter 9: Animation and Games

Games are one of the best ways to learn programming. A game requires you to understand the core mechanics of computer science: state management, collision detection, event handling, and the game loop itself. In this chapter, we'll build Breakout—a classic arcade game—from scratch using the Canvas API and modern JavaScript.

Game development teaches you how computers actually work. Instead of writing business logic or data processing code, you're writing code that must run 60 times per second, manage multiple moving objects, detect collisions between them, respond to user input instantly, and maintain consistent state across all these moving parts. These are the fundamental challenges that show up everywhere in programming, from web servers to operating systems.

By the end of this chapter, you'll understand requestAnimationFrame, collision detection algorithms, state machines, and how to build smooth, responsive interactive applications. These concepts extend far beyond games—they're essential for any interactive software.

## The Program: Breakout

Breakout is a game where you control a paddle at the bottom of the screen and bounce a ball upward to break colored bricks at the top. As you break bricks, you score points. If the ball falls off the bottom of the screen, you lose a life. Break all the bricks to advance to the next level, where the ball moves faster and more bricks appear.

[Open breakout.html](breakout.html) to play the game.

The game features keyboard control (arrow keys), mouse control, smooth animation at 60fps, realistic ball physics, colorful brick arrangements that change per level, and progressive difficulty. It's a complete, polished game in under 250 lines of JavaScript.

## How It Works

### The Game Loop

The heart of any game is the game loop—a function that runs repeatedly to update game state and redraw the screen. In browsers, we use `requestAnimationFrame` instead of `setInterval` because it synchronizes with the monitor's refresh rate, usually 60 times per second.

```javascript
const gameLoop = () => {
    update();  // Change positions, check collisions
    draw();    // Render everything to the canvas
    requestAnimationFrame(gameLoop);
};

gameLoop();
```

Why is `requestAnimationFrame` better than `setInterval`? First, it syncs with the display—drawing happens when the monitor is ready to show the new frame, eliminating visual tearing. Second, it automatically pauses when the tab is hidden, saving CPU and battery. Third, it's what the browser optimizes for, so animations are smoother.

Without a proper game loop, games feel janky and unresponsive. The loop creates the illusion of continuous motion by redrawing everything many times per second.

### Drawing on Canvas

The Canvas API gives us a 2D drawing surface. We get a context object that has methods for drawing rectangles, circles, lines, and more.

```javascript
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

// Draw a rectangle
ctx.fillStyle = '#FF6B6B';
ctx.fillRect(x, y, width, height);

// Draw a circle
ctx.fillStyle = '#FFE66D';
ctx.beginPath();
ctx.arc(x, y, radius, 0, Math.PI * 2);
ctx.fill();

// Add a shadow/glow
ctx.shadowColor = 'rgba(255, 230, 109, 0.8)';
ctx.shadowBlur = 20;
```

The key insight is that every frame, we clear the canvas and redraw everything. This happens so fast (60 times per second) that it looks like continuous motion. Each frame is a new "snapshot" of the game world.

### Collision Detection

The game uses Axis-Aligned Bounding Box (AABB) collision detection. The idea is simple: treat every game object as a rectangle (or in the case of the ball, a circle). An object collides with another if their rectangular bounds overlap.

```javascript
const aabbCollision = (circle, rect) => {
    // Find the closest point on the rectangle to the circle's center
    const closestX = Math.max(rect.x, Math.min(circle.x, rect.x + rect.width));
    const closestY = Math.max(rect.y, Math.min(circle.y, rect.y + rect.height));

    // Calculate distance between circle center and closest point
    const dx = circle.x - closestX;
    const dy = circle.y - closestY;

    // Collision occurs if distance is less than radius
    return dx * dx + dy * dy < circle.radius * circle.radius;
};
```

This works by finding the point on the rectangle closest to the circle's center. If that distance is less than the ball's radius, they're colliding. We use squared distances to avoid expensive square root calculations.

When a collision is detected, we need to bounce the ball. The simplest approach: reverse the ball's velocity in the appropriate direction.

```javascript
if (aabbCollision(ball, paddle)) {
    ball.dy = -Math.abs(ball.dy);  // Bounce up
    // Add spin based on where the ball hit the paddle
    const hitPos = (ball.x - (paddle.x + paddle.width / 2)) / (paddle.width / 2);
    ball.dx = hitPos * ballSpeed * 1.2;
    ball.y = paddle.y - ball.radius;  // Position ball above paddle
}
```

Notice that hitting the paddle on the left side gives the ball leftward velocity, and hitting on the right gives it rightward velocity. This makes the game feel responsive to player skill.

### Keyboard and Mouse Input

Modern games need responsive input handling. We track which keys are pressed in an object, then check that object during the update phase:

```javascript
const keys = {};

window.addEventListener('keydown', (e) => {
    keys[e.key.toLowerCase()] = true;
});

window.addEventListener('keyup', (e) => {
    keys[e.key.toLowerCase()] = false;
});

// In the update function:
if (keys['arrowleft']) {
    paddle.x -= paddle.speed;
}
if (keys['arrowright']) {
    paddle.x += paddle.speed;
}
```

For mouse control, we track the mouse position and smoothly interpolate the paddle toward it:

```javascript
canvas.addEventListener('mousemove', (e) => {
    const rect = canvas.getBoundingClientRect();
    mouseX = e.clientX - rect.left;
});

// In update:
const paddleTarget = Math.max(0, Math.min(mouseX - paddle.width / 2, canvas.width - paddle.width));
paddle.x += (paddleTarget - paddle.x) * 0.15;  // Smooth movement
```

The `0.15` factor creates acceleration—the paddle doesn't snap to the mouse, it follows smoothly. This feels better than instant movement.

### Game State Management

Games have multiple states: start screen, playing, paused, game over, level complete. We track state and change how we update and draw based on it:

```javascript
let gameState = 'start';  // Can be: start, playing, paused, gameOver, levelComplete

const update = () => {
    if (gameState !== 'playing') return;  // Only update when playing
    // ... collision detection, movement, etc.
};

const draw = () => {
    // ... draw game objects ...

    if (gameState === 'start') {
        ctx.fillText('BREAKOUT', canvas.width / 2, canvas.height / 2);
        ctx.fillText('Press SPACE to start', canvas.width / 2, canvas.height / 2 + 40);
    }

    if (gameState === 'gameOver') {
        ctx.fillStyle = 'rgba(0, 0, 0, 0.8)';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillText('GAME OVER', canvas.width / 2, canvas.height / 2);
    }
};

window.addEventListener('keydown', (e) => {
    if (e.key === ' ') {
        if (gameState === 'start') gameState = 'playing';
        if (gameState === 'gameOver') gameState = 'start';
    }
});
```

This state machine pattern keeps the code organized and makes it easy to add new states later.

### Velocity and Physics

The ball moves using velocity—we add its `dx` and `dy` to its position every frame:

```javascript
ball.x += ball.dx;
ball.y += ball.dy;
```

This simple approach creates smooth motion. When the ball hits something, we reverse the appropriate velocity component. For the paddle, we also adjust `dx` based on where the ball hit, creating the illusion of "spin" or control.

As levels increase, we increase `ballSpeed`, making the game harder. Ball speed in this game represents the magnitude of velocity—we scale the initial velocities by this value.

## Try It

Here are some ways to extend the game:

1. **Add sound effects.** When the ball bounces, play a "pop" sound. When a brick breaks, play a different sound. When you lose, play a "fail" sound. Use the Web Audio API.

2. **Add a paddle speedup power-up.** When you break certain bricks, drop a power-up that temporarily widens the paddle. Track which bricks have power-ups and create particles when they're collected.

3. **Add a level editor.** Let players design custom brick patterns. Store patterns as arrays and load them when starting a level.

## Exercises

### Exercise 1: Power-Ups

Add power-ups that fall from broken bricks. Collecting a power-up should either widen the paddle for 5 seconds, increase ball speed, or add an extra life. Different colors for different power-ups.

**Hints:** Create a powerUps array. When a brick breaks, randomly decide if it drops a power-up. In the draw function, render falling power-ups. In the update function, check if the paddle collides with any power-ups.

### Exercise 2: Sound Effects

Add Web Audio API sound effects for bouncing and breaking bricks. Create simple sine-wave tones that play when these events happen.

**Hints:** Use `new AudioContext()` to create a context. Use `createOscillator()` and `createGain()` to create and control tones. Call `start()` and `stop()` to trigger sounds.

### Exercise 3: High Score Persistence

Save the high score to localStorage so it persists across page reloads. Display the high score on the start screen and game over screen.

**Hints:** Use `localStorage.setItem('highScore', score)` to save and `localStorage.getItem('highScore')` to load. Check if the current score beats the high score after the game ends.

## Solutions

### Solution 1: Power-Ups

```javascript
let powerUps = [];
const POWER_UP_TYPES = {
    WIDE_PADDLE: 'wide',
    FAST_BALL: 'fast',
    EXTRA_LIFE: 'life'
};

const createPowerUp = (x, y) => {
    const type = Math.random() < 0.33 ? POWER_UP_TYPES.WIDE_PADDLE : 
                 Math.random() < 0.67 ? POWER_UP_TYPES.FAST_BALL : 
                 POWER_UP_TYPES.EXTRA_LIFE;
    
    powerUps.push({
        x, y,
        width: 20,
        height: 20,
        type,
        dy: 2
    });
};

// In collision detection, when a brick is destroyed:
if (Math.random() < 0.15) createPowerUp(brick.x + brick.width / 2, brick.y);

// In update:
for (let i = powerUps.length - 1; i >= 0; i--) {
    const pu = powerUps[i];
    pu.y += pu.dy;

    if (aabbCollision({ x: pu.x, y: pu.y, radius: 10 }, paddle)) {
        if (pu.type === POWER_UP_TYPES.WIDE_PADDLE) {
            paddle.width = 150;
            setTimeout(() => { paddle.width = 100; }, 5000);
        } else if (pu.type === POWER_UP_TYPES.EXTRA_LIFE) {
            lives++;
        }
        powerUps.splice(i, 1);
    } else if (pu.y > canvas.height) {
        powerUps.splice(i, 1);
    }
}

// In draw, after drawing bricks:
for (const pu of powerUps) {
    const colors = {
        wide: '#FFD700',
        fast: '#FF6B6B',
        life: '#4ECDC4'
    };
    ctx.fillStyle = colors[pu.type];
    ctx.fillRect(pu.x - pu.width / 2, pu.y - pu.height / 2, pu.width, pu.height);
}
```

### Solution 2: Sound Effects

```javascript
const audioContext = new (window.AudioContext || window.webkitAudioContext)();

const playTone = (frequency, duration, type = 'sine') => {
    const osc = audioContext.createOscillator();
    const gain = audioContext.createGain();

    osc.connect(gain);
    gain.connect(audioContext.destination);

    osc.frequency.value = frequency;
    osc.type = type;

    gain.gain.setValueAtTime(0.3, audioContext.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + duration);

    osc.start(audioContext.currentTime);
    osc.stop(audioContext.currentTime + duration);
};

// When ball bounces off paddle:
playTone(400, 0.1);

// When brick breaks:
playTone(600, 0.15);

// When ball is lost:
playTone(200, 0.3);
```

### Solution 3: High Score Persistence

```javascript
const loadHighScore = () => {
    const saved = localStorage.getItem('breakoutHighScore');
    return saved ? parseInt(saved) : 0;
};

const saveHighScore = (score) => {
    const current = loadHighScore();
    if (score > current) {
        localStorage.setItem('breakoutHighScore', score);
    }
};

let highScore = loadHighScore();

// At game over:
if (gameState === 'gameOver') {
    saveHighScore(score);
    if (score > highScore) {
        highScore = score;
        // Display "NEW HIGH SCORE!" message
    }
}

// In the draw function, display high score:
ctx.fillText(`High Score: ${highScore}`, canvas.width - 220, 30);
```

## What You Learned

| Concept | How We Used It |
|---------|---------------|
| requestAnimationFrame | Created a smooth 60fps game loop that syncs with monitor refresh |
| Canvas API | Drew bricks, paddle, ball, and UI on a 2D canvas |
| AABB Collision | Detected ball hitting paddle, bricks, and walls accurately |
| Keyboard Events | Captured arrow key input for paddle control |
| Mouse Events | Tracked mouse position for smooth paddle following |
| Game State | Managed multiple screens (start, playing, game over, etc.) |
| Velocity/Physics | Moved objects by adding velocity each frame, bounced off walls |
| Array Methods | Used `filter()`, `every()`, and `forEach()` to manage collections |
| Arrow Functions | Wrote concise update and draw functions |
| Event Listeners | Responded to input and coordinated game timing |

## Building with Claude

You can use Claude to help you extend this game in many ways. Ask Claude to add new features like pause functionality, multiple ball modes, or brick patterns. Describe what you want ("Add a slow-motion power-up that lasts 3 seconds"), and Claude can write the code for you.

Claude is particularly useful for game physics problems. If your ball isn't bouncing realistically, describe the behavior ("The ball bounces too fast off the paddle") and Claude can help you adjust the physics. You can also ask Claude to help you debug collision detection issues or optimize performance.

As you build more complex games, you might want to ask Claude about architectural patterns like entity-component systems, spatial partitioning for collision detection, or state machines. These concepts become important when games have hundreds of objects or complex game logic. Claude can explain these patterns and help you implement them incrementally, starting simple and adding complexity only when needed.

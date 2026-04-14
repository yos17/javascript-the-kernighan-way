# Chapter 8: Canvas

## Why Canvas Exists

HTML is great for structured content: text, buttons, forms, links. But it has no mechanism for drawing arbitrary shapes, pixels, or animations. You can't place a circle at an exact position, draw a freehand stroke, or update 60 times per second using regular HTML elements.

The Canvas API solves this by providing a **pixel buffer** — a rectangle of memory you can draw on with JavaScript. The browser just displays whatever's in that buffer:

```
<canvas> element in HTML
        ↓
ctx = canvas.getContext('2d')   ← your drawing handle
        ↓
ctx.beginPath()
ctx.arc(100, 100, 50, 0, Math.PI * 2)
ctx.fillStyle = 'red'
ctx.fill()
        ↓
Browser renders the pixel buffer to screen
```

**Canvas vs. SVG**: SVG is also used for graphics, but it creates DOM elements (like HTML). You can select and modify individual SVG shapes after drawing them. Canvas doesn't — once you draw a pixel, it's just a pixel. Canvas is better for: lots of objects updating every frame (games, animations), pixel-level manipulation (photo filters, flood fill). SVG is better for: scalable diagrams, interactive charts where you need to click individual elements.

## The Program: Drawing App

A fully-featured drawing application with pencil, shapes, color picker, and undo/redo. Click the toolbar to switch tools, adjust brush size and opacity, and save your drawing as a PNG image.

## The Complete Program

See `drawing_app.html` for the full source. Key concepts are explained below.

## How It Works

### The Coordinate System

Canvas uses a coordinate system where (0, 0) is the **top-left corner**. X increases to the right, Y increases *downward*:

```
(0,0) ─────────────────────── (900,0)
  │                                │
  │          canvas                │
  │                                │
  │   (100, 80) ← a point          │
  │                                │
(0,540) ──────────────────── (900,540)
```

This trips up everyone coming from math class where Y goes up. In Canvas (and most computer graphics), Y goes down. So to draw below a point, *increase* Y.

### Getting a Context

To draw, you first get a 2D drawing context:

```javascript
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
```

`ctx` is the object with all drawing methods. Think of it as your brush — it maintains the current color, line width, opacity, and other settings, and it has methods for drawing shapes.

### Drawing Paths

A **path** is a series of points and lines that define a shape. You build the path, then either `stroke()` it (outline) or `fill()` it (solid):

```javascript
// Freehand pencil stroke
ctx.beginPath();          // start a new path
ctx.moveTo(lastX, lastY); // lift the pen to the starting point
ctx.lineTo(x, y);         // draw a line to the new point
ctx.lineWidth = 6;
ctx.lineCap = 'round';    // rounded line ends (vs 'butt' or 'square')
ctx.strokeStyle = '#ff0000';
ctx.stroke();             // render the path to pixels
```

**Why `beginPath()`?** Without it, every subsequent `stroke()` or `fill()` would also redraw all previous paths. `beginPath()` says "forget everything before, start fresh."

The `moveTo`/`lineTo` pattern:
```
moveTo(50, 50) ── no line, just positions the "pen"
lineTo(100, 50) ── draws line from (50,50) to (100,50)
lineTo(100, 100) ── draws line from (100,50) to (100,100)
stroke() ── renders these as a visible L-shape
```

### Drawing Shapes

Rectangles and ellipses have dedicated methods:

```javascript
// Rectangle (outline)
ctx.strokeRect(x, y, width, height);

// Rectangle (filled)
ctx.fillRect(x, y, width, height);

// Ellipse (works for circles too)
ctx.beginPath();
ctx.ellipse(
  centerX, centerY,  // center point
  radiusX, radiusY,  // horizontal and vertical radius
  0,                 // rotation angle
  0, Math.PI * 2     // start and end angle (full circle = 0 to 2π)
);
ctx.stroke();
```

For a perfect circle, make `radiusX === radiusY`. For an oval, make them different.

### Save/Restore Context State

The context remembers settings: `strokeStyle`, `lineWidth`, `globalAlpha`, `lineCap`. When you want to temporarily change settings and then restore the original:

```javascript
ctx.save();                       // push current state onto a stack
ctx.strokeStyle = 'rgba(0,0,0,0.5)';
ctx.lineWidth = 2;
ctx.stroke();                     // draw with these temporary settings
ctx.restore();                    // pop state — original settings restored
```

This is like a "checkpoint" system. `save()` and `restore()` can be nested. The drawing app uses this for shape previews: save state before the preview stroke, restore after.

### Shape Previews: Snapshot and Redraw

The trick for live shape previews while dragging:

```javascript
// mousedown: capture canvas state before the drag starts
snapshotBeforeShape = ctx.getImageData(0, 0, canvas.width, canvas.height);

// mousemove: restore snapshot, then draw current preview
ctx.putImageData(snapshotBeforeShape, 0, 0);  // undo preview
// ... draw new preview at current mouse position ...

// mouseup: the shape is now permanent (snapshot is discarded)
snapshotBeforeShape = null;
```

`getImageData()` captures every pixel as a flat array of RGBA values. `putImageData()` restores them. This is how undo/redo works too — the undo stack is an array of `ImageData` snapshots.

### Mouse Coordinate Mapping

The canvas element on screen may be a different size than its internal resolution. You must convert screen coordinates to canvas coordinates:

```javascript
function getPos(e) {
  const rect   = canvas.getBoundingClientRect(); // element's position on screen
  const scaleX = canvas.width  / rect.width;    // ratio: canvas pixels / screen pixels
  const scaleY = canvas.height / rect.height;
  const src    = e.touches ? e.touches[0] : e;  // touch or mouse
  return [
    Math.round((src.clientX - rect.left) * scaleX),
    Math.round((src.clientY - rect.top)  * scaleY),
  ];
}
```

Without this scaling, drawing on a canvas displayed at half its natural size would put strokes in the wrong place.

### Progressive Build: From Zero to Drawing App

Here's how the complexity built up:

**v1 — Just draw a dot on click:**
```javascript
canvas.addEventListener('click', e => {
  const [x, y] = getPos(e);
  ctx.beginPath();
  ctx.arc(x, y, 5, 0, Math.PI * 2);
  ctx.fill();
});
```

**v2 — Track mouse movement while held down:**
```javascript
let isDrawing = false;
canvas.addEventListener('mousedown', e => { isDrawing = true; [lastX, lastY] = getPos(e); });
canvas.addEventListener('mousemove', e => { if (isDrawing) drawLine(...getPos(e)); });
canvas.addEventListener('mouseup',   () => { isDrawing = false; });
```

**v3 — Add colors, line width, multiple tools** (with toolbar buttons triggering state changes)

**v4 — Add undo/redo** (snapshotting `ImageData` before each stroke)

**v5 — Add shape tools** (preview via snapshot/restore pattern)

Each version is a complete working app. The drawing app in `drawing_app.html` is v5 — but every concept builds on the v1 foundation.

---

## Guided Exercises

### Exercise 1: A Spray Paint Tool

**The Challenge:** Add a spray paint tool that draws random dots in a circular burst around the cursor while the mouse button is held. The longer you hold, the denser the spray.

**Where to start:** Spray paint is fundamentally different from pencil — it draws *while held*, not while *moving*. Think about how you'd trigger drawing repeatedly while the mouse button is down.

*(Hint: `mousemove` only fires when the mouse moves. What would you use to draw continuously, even when the mouse is still?)*

---

**Step 1: Add the tool button.**

In the HTML toolbar, add a spray tool button:

```html
<button class="tool-btn" data-tool="spray" title="Spray (S)">💨</button>
```

Also add `s` to the keyboard shortcuts in the `toolKeys` map.

---

**Step 2: Think about the spray mechanism.**

Spray paint needs to fire repeatedly while held. `setInterval` can run a function on a timer:

```javascript
let sprayInterval = null;

function startSpray(x, y) {
  // Store the spray center so the interval closure can access it
  let cx = x, cy = y;

  sprayInterval = setInterval(() => {
    for (let i = 0; i < 20; i++) {
      const angle    = Math.random() * Math.PI * 2;
      const distance = Math.random() * lineWidth * 2;
      const dotX = cx + Math.cos(angle) * distance;
      const dotY = cy + Math.sin(angle) * distance;

      ctx.fillStyle = color;
      ctx.globalAlpha = opacity * 0.3;  // individual dots are faint
      ctx.beginPath();
      ctx.arc(dotX, dotY, 1, 0, Math.PI * 2);
      ctx.fill();
    }
  }, 30);  // fire every 30ms
}

function stopSpray() {
  if (sprayInterval) {
    clearInterval(sprayInterval);
    sprayInterval = null;
  }
}
```

**What does `Math.cos(angle) * distance` do?** It converts a polar coordinate (angle + distance from center) to a Cartesian offset (x, y). `Math.cos` gives the X component, `Math.sin` gives Y. Random angle + random distance = random point inside a circle.

---

**Step 3: Update mouse event handlers.**

In `mousedown`:
```javascript
if (tool === 'spray') {
  saveSnapshot();
  const [x, y] = getPos(e);
  startSpray(x, y);
  return;
}
```

In `mousemove` (to follow the cursor):
```javascript
if (tool === 'spray' && sprayInterval) {
  // Update the spray center — but how? The interval closure has `cx, cy`...
}
```

**Think about it:** The spray center needs to move with the cursor. But the `setInterval` callback has its own closure over `cx` and `cy`. How do you update them?

*(Answer: Make `cx` and `cy` variables in the outer scope that the interval closure reads each time it fires. Use `let` so they're mutable. Update them in `mousemove`.)*

---

**Step 4: The complete spray pattern.**

```javascript
let sprayCenter = { x: 0, y: 0 };
let sprayInterval = null;

function startSpray(x, y) {
  sprayCenter = { x, y };
  sprayInterval = setInterval(() => {
    for (let i = 0; i < 20; i++) {
      const angle    = Math.random() * Math.PI * 2;
      const dist     = Math.random() * lineWidth * 2;
      const dotX     = sprayCenter.x + Math.cos(angle) * dist;
      const dotY     = sprayCenter.y + Math.sin(angle) * dist;
      ctx.fillStyle   = tool === 'eraser' ? '#fff' : color;
      ctx.globalAlpha = opacity * 0.25;
      ctx.beginPath();
      ctx.arc(dotX, dotY, 1, 0, Math.PI * 2);
      ctx.fill();
    }
  }, 30);
}
```

In `mousemove`:
```javascript
if (tool === 'spray') sprayCenter = { x, y };
```

In `mouseup` and `mouseleave`: call `stopSpray()`.

---

### Exercise 2: Text Tool

**The Challenge:** Add a text tool. When selected, clicking the canvas should place an `<input>` element at the click position. Pressing Enter commits the text to the canvas; pressing Escape cancels.

**Where to start:** This tool works differently from all others — it creates a temporary HTML element overlaid on the canvas, then "burns" the text into the canvas pixel buffer when confirmed.

---

**Step 1: Handle the click to create the text input.**

In `mousedown`, when `tool === 'text'`:

```javascript
if (tool === 'text') {
  const [x, y] = getPos(e);
  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width / rect.width;

  // Create a positioned input element
  const textInput = document.createElement('input');
  textInput.style.cssText = `
    position: absolute;
    left: ${e.clientX}px;
    top: ${e.clientY}px;
    background: transparent;
    border: 1px dashed #888;
    color: ${color};
    font-size: ${Math.max(12, lineWidth * 2)}px;
    outline: none;
    padding: 2px;
    min-width: 100px;
    z-index: 10;
  `;
  document.body.appendChild(textInput);
  textInput.focus();

  // Commit on Enter
  textInput.addEventListener('keydown', e => {
    if (e.key === 'Enter') {
      saveSnapshot();
      ctx.font = `${Math.max(12, lineWidth * 2)}px sans-serif`;
      ctx.fillStyle = color;
      ctx.globalAlpha = opacity;
      ctx.fillText(textInput.value, x, y);
      textInput.remove();
    } else if (e.key === 'Escape') {
      textInput.remove();
    }
  });
  return;
}
```

**Why overlay an HTML input instead of drawing characters on canvas?** Canvas doesn't provide a cursor, selection, or editing. An HTML `<input>` gives you all that for free. When the user is done, you "bake" the text into the canvas with `fillText()` and remove the HTML element.

**Step 2: Add the tool button** and test. The text appears where you click, you type, Enter commits it.

---

### Exercise 3: Zoom and Pan

**The Challenge:** Add Ctrl+scroll to zoom in and out. Add a pan mode (hold Space to temporarily switch) where dragging moves the viewport instead of drawing.

This is a significant extension. The core challenge: the coordinate transform.

**Where to start:** Zooming and panning require a **transform matrix**. Instead of scaling each coordinate manually, you let the canvas context apply a transform before rendering.

---

**Step 1: Understand canvas transforms.**

```javascript
ctx.save();
ctx.scale(2, 2);           // zoom in 2x
ctx.translate(-100, -50);  // pan to show different area
// ... draw things here — they appear transformed
ctx.restore();             // undo the transform
```

For a drawing app, you apply the transform before every draw call. The transform state needs to persist across frames.

**Step 2: Add transform state:**

```javascript
let zoom   = 1;
let panX   = 0;
let panY   = 0;

function applyTransform() {
  ctx.setTransform(zoom, 0, 0, zoom, panX, panY);
}
```

**Step 3: Adjust coordinate conversion.**

When zoom/pan are non-default, `getPos()` needs to account for them:

```javascript
function getPos(e) {
  const rect = canvas.getBoundingClientRect();
  const scaleX = canvas.width  / rect.width;
  const scaleY = canvas.height / rect.height;
  const src = e.touches ? e.touches[0] : e;
  const screenX = (src.clientX - rect.left) * scaleX;
  const screenY = (src.clientY - rect.top)  * scaleY;
  // Convert screen coordinates to canvas "world" coordinates
  return [
    Math.round((screenX - panX) / zoom),
    Math.round((screenY - panY) / zoom),
  ];
}
```

**Step 4: Add scroll-to-zoom.**

```javascript
canvas.addEventListener('wheel', e => {
  e.preventDefault();
  const factor = e.deltaY < 0 ? 1.1 : 0.9;
  zoom = Math.min(8, Math.max(0.25, zoom * factor));
  // Optional: zoom toward cursor position
});
```

The complete zoom/pan implementation is in the "Building with Claude" section — this exercise is about understanding the transform concept.

---

## What You Learned

| Concept | What It Does | Real-World Use |
|---------|-------------|----------------|
| `getContext('2d')` | Returns the 2D drawing context | Foundation of canvas-based apps |
| `beginPath()` / `moveTo()` / `lineTo()` | Builds a path without drawing it | Vector graphics, custom shapes |
| `stroke()` / `fill()` | Renders the path to pixels | Every shape in every canvas app |
| `strokeRect()` / `fillRect()` | Draws rectangles without explicit paths | UI primitives, game tiles |
| `arc()` / `ellipse()` | Draws circles and ovals | Gauges, charts, game elements |
| `getImageData()` / `putImageData()` | Captures/restores pixel data | Undo/redo, image filters, flood fill |
| `save()` / `restore()` | Stacks and restores drawing state | Temporary style changes |
| `getBoundingClientRect()` | Converts screen pixels to canvas pixels | Correct coordinate mapping |
| `toDataURL()` | Exports canvas as image data URL | Download, share, thumbnails |

### Real-World Connections

- **Figma, Photoshop (web)** use canvas for rendering. Their undo systems work exactly like this app's — snapshots.
- **Game engines** (Phaser, PixiJS) are canvas abstractions. They manage the game loop, sprite rendering, and transforms so you don't have to.
- **Chart libraries** (Chart.js, Highcharts) use canvas for performance when rendering hundreds of data points.
- **Image editors** use the `ImageData` pixel manipulation pattern for filters, blend modes, and flood fill.

## Building with Claude

- "Add a color-by-numbers mode where I can click regions and they auto-fill with the selected color."
- "Create a shape stamp tool that lets me click to place pre-drawn shapes like stars, hearts, and arrows."
- "Add zoom and pan: Ctrl+scroll to zoom, Space+drag to pan. Make sure drawing still works correctly when zoomed."
- "Add a gradient fill tool that fills the clicked region with a linear gradient between two colors I choose."
- "Implement layers: draw on separate surfaces that composite together when rendered."

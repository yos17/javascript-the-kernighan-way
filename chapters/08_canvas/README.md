# Chapter 8: Canvas

The Canvas API lets you draw shapes, lines, and images directly on a pixel grid using JavaScript. It's the foundation for games, data visualization, and creative graphics on the web.

## The Program: Drawing App

A fully-featured drawing application with pencil, shapes, color picker, and undo/redo. Click the toolbar to switch tools, adjust brush size and opacity, and save your drawing as a PNG image.

## The Complete Program

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Drawing App</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Segoe UI', system-ui, sans-serif;
      background: #1e1e2e;
      height: 100vh;
      display: flex;
      flex-direction: column;
      overflow: hidden;
      user-select: none;
    }

    /* ── Toolbar ─────────────────────────────────────────── */
    .toolbar {
      background: #2a2a3e;
      border-bottom: 1px solid #3a3a5e;
      padding: 0.5rem 1rem;
      display: flex;
      align-items: center;
      gap: 0.5rem;
      flex-wrap: wrap;
      flex-shrink: 0;
      box-shadow: 0 2px 12px rgba(0,0,0,0.3);
    }

    .toolbar-group {
      display: flex;
      align-items: center;
      gap: 0.35rem;
      padding: 0 0.5rem;
      border-right: 1px solid #3a3a5e;
    }

    .toolbar-group:last-child { border-right: none; }

    .toolbar-label {
      font-size: 0.68rem;
      color: #666;
      text-transform: uppercase;
      letter-spacing: 0.05em;
      margin-right: 0.2rem;
    }

    .tool-btn {
      background: #3a3a5e;
      border: 2px solid transparent;
      border-radius: 8px;
      color: #ccc;
      cursor: pointer;
      width: 36px;
      height: 36px;
      font-size: 1.1rem;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: all 0.15s;
      position: relative;
    }

    .tool-btn:hover { background: #4a4a7e; color: #fff; }

    .tool-btn.active {
      background: #5b5bef;
      border-color: #8080ff;
      color: #fff;
      box-shadow: 0 0 10px rgba(91,91,239,0.5);
    }

    /* Color swatches */
    .color-swatch {
      width: 28px;
      height: 28px;
      border-radius: 50%;
      border: 2px solid transparent;
      cursor: pointer;
      transition: transform 0.15s, border-color 0.15s;
      flex-shrink: 0;
    }

    .color-swatch:hover { transform: scale(1.15); }
    .color-swatch.active { border-color: #fff; transform: scale(1.15); }

    #colorPicker {
      width: 28px;
      height: 28px;
      border: 2px solid #555;
      border-radius: 50%;
      padding: 0;
      cursor: pointer;
      background: none;
      overflow: hidden;
    }

    /* Size slider */
    #sizeSlider {
      -webkit-appearance: none;
      appearance: none;
      width: 90px;
      height: 4px;
      border-radius: 2px;
      background: #5b5bef;
      outline: none;
      cursor: pointer;
    }

    #sizeSlider::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 16px;
      height: 16px;
      border-radius: 50%;
      background: #fff;
      cursor: pointer;
      box-shadow: 0 0 4px rgba(0,0,0,0.4);
    }

    .size-preview {
      width: 24px;
      height: 24px;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .size-dot {
      background: #fff;
      border-radius: 50%;
      transition: width 0.1s, height 0.1s;
    }

    /* Opacity */
    #opacitySlider {
      -webkit-appearance: none;
      appearance: none;
      width: 70px;
      height: 4px;
      border-radius: 2px;
      background: linear-gradient(to right, transparent, #fff);
      outline: none;
      cursor: pointer;
    }

    #opacitySlider::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 14px;
      height: 14px;
      border-radius: 50%;
      background: #fff;
      cursor: pointer;
    }

    /* ── Canvas area ─────────────────────────────────────── */
    .canvas-wrap {
      flex: 1;
      display: flex;
      align-items: center;
      justify-content: center;
      background:
        linear-gradient(45deg, #252535 25%, transparent 25%),
        linear-gradient(-45deg, #252535 25%, transparent 25%),
        linear-gradient(45deg, transparent 75%, #252535 75%),
        linear-gradient(-45deg, transparent 75%, #252535 75%);
      background-size: 20px 20px;
      background-position: 0 0, 0 10px, 10px -10px, -10px 0;
      background-color: #1e1e2e;
      overflow: hidden;
    }

    #canvas {
      background: #fff;
      box-shadow: 0 8px 40px rgba(0,0,0,0.6);
      cursor: crosshair;
      display: block;
      touch-action: none;
    }

    /* ── Status bar ──────────────────────────────────────── */
    .statusbar {
      background: #2a2a3e;
      border-top: 1px solid #3a3a5e;
      padding: 0.3rem 1rem;
      display: flex;
      gap: 1.5rem;
      font-size: 0.72rem;
      color: #555;
      flex-shrink: 0;
    }

    .statusbar strong { color: #bbb; }
  </style>
</head>
<body>

<div class="toolbar">
  <!-- Tools -->
  <div class="toolbar-group">
    <span class="toolbar-label">Tool</span>
    <button class="tool-btn active" data-tool="pencil" title="Pencil (P)">✏️</button>
    <button class="tool-btn" data-tool="line"    title="Line (L)">╱</button>
    <button class="tool-btn" data-tool="rect"    title="Rectangle (R)">▭</button>
    <button class="tool-btn" data-tool="circle"  title="Circle (C)">○</button>
    <button class="tool-btn" data-tool="fill"    title="Fill (F)">🪣</button>
    <button class="tool-btn" data-tool="eraser"  title="Eraser (E)">🧹</button>
  </div>

  <!-- Colors -->
  <div class="toolbar-group">
    <span class="toolbar-label">Color</span>
    <div class="color-swatch active" style="background:#000000" data-color="#000000"></div>
    <div class="color-swatch" style="background:#ef4444" data-color="#ef4444"></div>
    <div class="color-swatch" style="background:#f97316" data-color="#f97316"></div>
    <div class="color-swatch" style="background:#eab308" data-color="#eab308"></div>
    <div class="color-swatch" style="background:#22c55e" data-color="#22c55e"></div>
    <div class="color-swatch" style="background:#3b82f6" data-color="#3b82f6"></div>
    <div class="color-swatch" style="background:#8b5cf6" data-color="#8b5cf6"></div>
    <div class="color-swatch" style="background:#e5e7eb;border:1px solid #555" data-color="#e5e7eb"></div>
    <input type="color" id="colorPicker" value="#000000" title="Custom color">
  </div>

  <!-- Size -->
  <div class="toolbar-group">
    <span class="toolbar-label">Size</span>
    <input type="range" id="sizeSlider" min="1" max="60" value="6">
    <div class="size-preview"><div class="size-dot" id="sizeDot" style="width:6px;height:6px"></div></div>
  </div>

  <!-- Opacity -->
  <div class="toolbar-group">
    <span class="toolbar-label">Opacity</span>
    <input type="range" id="opacitySlider" min="5" max="100" value="100">
    <span id="opacityVal" style="font-size:0.75rem;color:#888;min-width:2.5em">100%</span>
  </div>

  <!-- Actions -->
  <div class="toolbar-group">
    <button class="tool-btn" id="undoBtn"  title="Undo (Ctrl+Z)">↩️</button>
    <button class="tool-btn" id="redoBtn"  title="Redo (Ctrl+Y)">↪️</button>
    <button class="tool-btn" id="clearBtn" title="Clear canvas">🗑️</button>
    <button class="tool-btn" id="saveBtn"  title="Save as PNG">💾</button>
  </div>
</div>

<div class="canvas-wrap">
  <canvas id="canvas" width="900" height="540"></canvas>
</div>

<div class="statusbar">
  <div>Tool: <strong id="statusTool">Pencil</strong></div>
  <div>Position: <strong id="statusPos">—</strong></div>
  <div>Size: <strong id="statusSize">6px</strong></div>
  <div>Undo history: <strong id="statusUndo">0</strong></div>
</div>

<script>
  // ─── Canvas setup ─────────────────────────────────────────────
  const canvas = document.getElementById('canvas');
  const ctx    = canvas.getContext('2d');           // getContext('2d')

  // ─── App state ────────────────────────────────────────────────
  let tool      = 'pencil';
  let color     = '#000000';
  let lineWidth = 6;
  let opacity   = 1.0;
  let isDrawing = false;
  let startX = 0, startY = 0;
  let lastX  = 0, lastY  = 0;
  let snapshotBeforeShape = null;

  // Undo/redo stacks — store ImageData snapshots
  const undoStack = [];
  const redoStack = [];

  // ─── Save canvas snapshot for undo ───────────────────────────
  function saveSnapshot() {
    undoStack.push(ctx.getImageData(0, 0, canvas.width, canvas.height));
    if (undoStack.length > 40) undoStack.shift();   // cap memory usage
    redoStack.length = 0;                            // new stroke clears redo
    document.getElementById('statusUndo').textContent = undoStack.length;
  }

  // ─── Pencil / eraser: freehand drawing ───────────────────────
  function drawPencil(x, y) {
    ctx.beginPath();                                  // beginPath
    ctx.moveTo(lastX, lastY);                        // moveTo
    ctx.lineTo(x, y);                                // lineTo
    ctx.strokeStyle = tool === 'eraser' ? '#ffffff' : color;
    ctx.globalAlpha = opacity;
    ctx.lineWidth   = lineWidth;
    ctx.lineCap     = 'round';
    ctx.lineJoin    = 'round';
    ctx.stroke();                                     // stroke
    [lastX, lastY] = [x, y];
  }

  // ─── Shape tools: preview while dragging ─────────────────────
  function drawShapePreview(x, y) {
    // Restore the canvas to its state at drag-start, then overdraw the preview
    ctx.putImageData(snapshotBeforeShape, 0, 0);

    ctx.save();                                       // save context state
    ctx.strokeStyle = color;
    ctx.globalAlpha = opacity;
    ctx.lineWidth   = lineWidth;
    ctx.lineCap     = 'round';

    if (tool === 'line') {
      ctx.beginPath();
      ctx.moveTo(startX, startY);
      ctx.lineTo(x, y);
      ctx.stroke();

    } else if (tool === 'rect') {
      ctx.beginPath();
      ctx.strokeRect(startX, startY, x - startX, y - startY);

    } else if (tool === 'circle') {
      const rx = Math.abs(x - startX) / 2;
      const ry = Math.abs(y - startY) / 2;
      const cx = startX + (x - startX) / 2;
      const cy = startY + (y - startY) / 2;
      ctx.beginPath();
      ctx.ellipse(cx, cy, Math.max(rx, 1), Math.max(ry, 1), 0, 0, Math.PI * 2); // arc
      ctx.stroke();
    }

    ctx.restore();                                    // restore context state
  }

  // ─── Flood fill (paint bucket tool) ──────────────────────────
  function floodFill(px, py, fillHex) {
    const imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data    = imgData.data;   // flat Uint8ClampedArray [R,G,B,A, R,G,B,A, …]

    const fr = parseInt(fillHex.slice(1, 3), 16);
    const fg = parseInt(fillHex.slice(3, 5), 16);
    const fb = parseInt(fillHex.slice(5, 7), 16);
    const fa = Math.round(opacity * 255);

    const idx = (py * canvas.width + px) * 4;
    const [tr, tg, tb, ta] = [data[idx], data[idx+1], data[idx+2], data[idx+3]];

    if (tr === fr && tg === fg && tb === fb) return; // already that color

    const stack   = [[px, py]];
    const visited = new Uint8Array(canvas.width * canvas.height);

    while (stack.length) {
      const [cx, cy] = stack.pop();
      if (cx < 0 || cx >= canvas.width || cy < 0 || cy >= canvas.height) continue;
      const i = cy * canvas.width + cx;
      if (visited[i]) continue;
      visited[i] = 1;

      const pi = i * 4;
      if (data[pi]   !== tr || data[pi+1] !== tg ||
          data[pi+2] !== tb || data[pi+3] !== ta) continue;

      data[pi] = fr; data[pi+1] = fg; data[pi+2] = fb; data[pi+3] = fa;
      stack.push([cx+1,cy],[cx-1,cy],[cx,cy+1],[cx,cy-1]);
    }

    ctx.putImageData(imgData, 0, 0);
  }

  // ─── Get canvas-relative coordinates ─────────────────────────
  function getPos(e) {
    const rect   = canvas.getBoundingClientRect();
    const scaleX = canvas.width  / rect.width;
    const scaleY = canvas.height / rect.height;
    const src    = e.touches ? e.touches[0] : e;
    return [
      Math.round((src.clientX - rect.left) * scaleX),
      Math.round((src.clientY - rect.top)  * scaleY),
    ];
  }

  // ─── Mouse events ─────────────────────────────────────────────
  canvas.addEventListener('mousedown', e => {
    e.preventDefault();
    const [x, y] = getPos(e);

    if (tool === 'fill') {
      saveSnapshot();
      floodFill(x, y, color);
      return;
    }

    saveSnapshot();
    isDrawing = true;
    [startX, startY] = [x, y];
    [lastX,  lastY]  = [x, y];

    if (['line','rect','circle'].includes(tool)) {
      snapshotBeforeShape = ctx.getImageData(0, 0, canvas.width, canvas.height);
    }

    // Draw a dot on single click
    if (tool === 'pencil' || tool === 'eraser') {
      drawPencil(x + 0.1, y + 0.1);
    }
  });

  canvas.addEventListener('mousemove', e => {
    const [x, y] = getPos(e);
    document.getElementById('statusPos').textContent = `${x}, ${y}`;
    if (!isDrawing) return;

    if (tool === 'pencil' || tool === 'eraser') {
      drawPencil(x, y);
    } else {
      drawShapePreview(x, y);
    }
  });

  function stopDrawing() {
    isDrawing = false;
    snapshotBeforeShape = null;
  }

  canvas.addEventListener('mouseup',    stopDrawing);
  canvas.addEventListener('mouseleave', stopDrawing);

  // Touch support
  canvas.addEventListener('touchstart', e => {
    e.preventDefault();
    const t = e.touches[0];
    canvas.dispatchEvent(new MouseEvent('mousedown', { clientX: t.clientX, clientY: t.clientY }));
  }, { passive: false });

  canvas.addEventListener('touchmove', e => {
    e.preventDefault();
    const t = e.touches[0];
    canvas.dispatchEvent(new MouseEvent('mousemove', { clientX: t.clientX, clientY: t.clientY }));
  }, { passive: false });

  canvas.addEventListener('touchend', () => stopDrawing());

  // ─── Toolbar: event delegation ────────────────────────────────
  document.querySelector('.toolbar').addEventListener('click', e => {
    // Tool buttons
    const toolBtn = e.target.closest('[data-tool]');
    if (toolBtn) {
      tool = toolBtn.dataset.tool;
      document.querySelectorAll('[data-tool]').forEach(b => b.classList.remove('active'));
      toolBtn.classList.add('active');
      document.getElementById('statusTool').textContent =
        toolBtn.title.split(' ')[0]; // first word of title
    }

    // Color swatches
    const swatch = e.target.closest('[data-color]');
    if (swatch) {
      color = swatch.dataset.color;
      document.querySelectorAll('.color-swatch').forEach(s => s.classList.remove('active'));
      swatch.classList.add('active');
      document.getElementById('colorPicker').value = color;
    }
  });

  document.getElementById('colorPicker').addEventListener('input', e => {
    color = e.target.value;
    document.querySelectorAll('.color-swatch').forEach(s => s.classList.remove('active'));
  });

  document.getElementById('sizeSlider').addEventListener('input', e => {
    lineWidth = parseInt(e.target.value);
    const display = Math.min(lineWidth, 22);
    document.getElementById('sizeDot').style.cssText =
      `width:${display}px;height:${display}px`;
    document.getElementById('statusSize').textContent = `${lineWidth}px`;
  });

  document.getElementById('opacitySlider').addEventListener('input', e => {
    opacity = parseInt(e.target.value) / 100;
    document.getElementById('opacityVal').textContent = `${e.target.value}%`;
  });

  // ─── Undo ─────────────────────────────────────────────────────
  function undo() {
    if (!undoStack.length) return;
    redoStack.push(ctx.getImageData(0, 0, canvas.width, canvas.height));
    ctx.putImageData(undoStack.pop(), 0, 0);
    document.getElementById('statusUndo').textContent = undoStack.length;
  }

  function redo() {
    if (!redoStack.length) return;
    undoStack.push(ctx.getImageData(0, 0, canvas.width, canvas.height));
    ctx.putImageData(redoStack.pop(), 0, 0);
    document.getElementById('statusUndo').textContent = undoStack.length;
  }

  document.getElementById('undoBtn').addEventListener('click', undo);
  document.getElementById('redoBtn').addEventListener('click', redo);

  // ─── Clear canvas ─────────────────────────────────────────────
  document.getElementById('clearBtn').addEventListener('click', () => {
    saveSnapshot();
    ctx.clearRect(0, 0, canvas.width, canvas.height);  // clearRect
    ctx.fillStyle = '#ffffff';
    ctx.fillRect(0, 0, canvas.width, canvas.height);   // fillRect
  });

  // ─── Save as PNG ──────────────────────────────────────────────
  document.getElementById('saveBtn').addEventListener('click', () => {
    const a    = document.createElement('a');
    a.href     = canvas.toDataURL('image/png');
    a.download = `drawing-${Date.now()}.png`;
    a.click();
  });

  // ─── Keyboard shortcuts ───────────────────────────────────────
  document.addEventListener('keydown', e => {
    if ((e.ctrlKey || e.metaKey) && e.key === 'z') { e.preventDefault(); undo(); return; }
    if ((e.ctrlKey || e.metaKey) && e.key === 'y') { e.preventDefault(); redo(); return; }
    if (e.ctrlKey || e.metaKey) return;

    const toolKeys = { p:'pencil', e:'eraser', l:'line', r:'rect', c:'circle', f:'fill' };
    if (toolKeys[e.key]) {
      tool = toolKeys[e.key];
      document.querySelectorAll('[data-tool]').forEach(b =>
        b.classList.toggle('active', b.dataset.tool === tool));
      document.getElementById('statusTool').textContent =
        tool.charAt(0).toUpperCase() + tool.slice(1);
    }
  });

  // ─── Initialize: white canvas + welcome text ──────────────────
  function init() {
    ctx.fillStyle = '#ffffff';
    ctx.fillRect(0, 0, canvas.width, canvas.height);   // fillRect

    // Draw welcome message with requestAnimationFrame
    requestAnimationFrame(() => {                       // requestAnimationFrame
      ctx.save();
      ctx.font      = 'bold 26px system-ui';
      ctx.fillStyle = '#dde1f7';
      ctx.textAlign = 'center';
      ctx.fillText('✏️  Start drawing!', canvas.width / 2, canvas.height / 2 - 10);
      ctx.font      = '15px system-ui';
      ctx.fillStyle = '#c4caf5';
      ctx.fillText('Tools: P pencil  L line  R rect  C circle  F fill  E eraser', canvas.width / 2, canvas.height / 2 + 24);
      ctx.restore();
    });
  }

  init();
</script>
</body>
</html>
```

## How It Works

### Getting a Canvas Context

To draw on a canvas, you first get its 2D drawing context:

```javascript
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
```

The context object (`ctx`) is where all the drawing methods live. It maintains state like current color, line width, and position.

### Mouse Events on Canvas

Drawing happens through three events: `mousedown` (start), `mousemove` (while dragging), and `mouseup` (end):

```javascript
canvas.addEventListener('mousedown', e => {
  isDrawing = true;
  [startX, startY] = getPos(e);
});

canvas.addEventListener('mousemove', e => {
  if (isDrawing) drawPencil(getPos(e)[0], getPos(e)[1]);
});

canvas.addEventListener('mouseup', () => {
  isDrawing = false;
});
```

### Coordinate Systems

The canvas has its own internal resolution (900x540 pixels) that may be displayed larger or smaller on screen. Use `getBoundingClientRect()` to convert screen coordinates to canvas coordinates:

```javascript
function getPos(e) {
  const rect   = canvas.getBoundingClientRect();
  const scaleX = canvas.width  / rect.width;
  const scaleY = canvas.height / rect.height;
  return [
    Math.round((e.clientX - rect.left) * scaleX),
    Math.round((e.clientY - rect.top)  * scaleY),
  ];
}
```

### Drawing Paths

Paths are the foundation of canvas drawing. Create one with `beginPath()`, move the brush with `moveTo()`, draw lines with `lineTo()`, and finalize with `stroke()`:

```javascript
function drawPencil(x, y) {
  ctx.beginPath();
  ctx.moveTo(lastX, lastY);
  ctx.lineTo(x, y);
  ctx.lineWidth = 6;
  ctx.lineCap = 'round';
  ctx.stroke();
}
```

### Drawing Shapes

For shapes, use `strokeRect()` for rectangles, `ellipse()` for circles and ovals:

```javascript
ctx.strokeRect(startX, startY, width, height);   // Rectangle

ctx.beginPath();
ctx.ellipse(centerX, centerY, radiusX, radiusY, rotation, 0, Math.PI * 2);
ctx.stroke();                                      // Circle
```

### Canvas State

Use `save()` and `restore()` to preserve and reset drawing properties like color and line width:

```javascript
ctx.save();
ctx.strokeStyle = '#ff0000';
ctx.lineWidth = 10;
ctx.stroke();
ctx.restore();  // Reverts to previous state
```

To implement undo, capture pixel data as a snapshot with `getImageData()` and restore it with `putImageData()`:

```javascript
function saveSnapshot() {
  undoStack.push(ctx.getImageData(0, 0, canvas.width, canvas.height));
}

function undo() {
  ctx.putImageData(undoStack.pop(), 0, 0);
}
```

### The Undo System

Every time you draw, a snapshot of the entire canvas is saved to an array. When you undo, restore the previous snapshot. Redo works the same way but in reverse. A cap on memory (40 snapshots) prevents the browser from slowing down:

```javascript
const undoStack = [];

function saveSnapshot() {
  undoStack.push(ctx.getImageData(0, 0, canvas.width, canvas.height));
  if (undoStack.length > 40) undoStack.shift();  // Remove oldest
}
```

The flood fill (bucket) tool reads pixel data directly and changes all connected pixels of the same color to a new color by walking a stack of coordinates.

## Try It

1. **Change the welcome message.** Edit the text in the `init()` function where it says "Start drawing!". Make it say something funny.

2. **Add keyboard shortcuts for colors.** Press number keys (1-8) to switch between the preset colors instantly. Hint: listen for `keydown` and check `e.key === '1'`, etc.

3. **Add a polygon (triangle) tool.** Create a new tool button and drawing mode that lets you click three times to draw a triangle. Store the points in an array and draw when the third click happens.

4. **Add a text tool.** Let the user type a message and click the canvas to place text. Use `ctx.font` and `ctx.fillText()` to render it.

## Exercises

1. **Build a shape grid.** Create a drawing program where clicking the canvas adds circles in a grid pattern (3x3 grid per click). Use nested loops with `ctx.beginPath()` and `ctx.arc()` to draw multiple circles at once. Each click should add circles to the next grid position, cycling back to the start after 9 clicks. Hint: Use a counter variable to track which grid cell to fill.

2. **Implement a line thickness preview tool.** As the user drags the size slider, draw a preview of the actual brush stroke on the canvas in real time (not just the dot preview in the toolbar). When they move away, clear the preview. Implement this by drawing on a temporary layer or capturing/restoring canvas state with snapshots.

3. **Add a spray paint tool.** Create a new tool that draws random dots in a circular burst pattern around the mouse position, like a can of spray paint. When held down, create multiple random dots each frame within a radius. Use `Math.random()` to vary the position and opacity of each dot. Make it work with `mousemove` while holding down the button.

## Solutions

### Exercise 1: Shape Grid

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Shape Grid</title>
  <style>
    body { margin: 0; background: #1e1e2e; display: flex; justify-content: center; align-items: center; height: 100vh; }
    canvas { background: white; box-shadow: 0 0 20px rgba(0,0,0,0.5); display: block; }
  </style>
</head>
<body>
  <canvas id="canvas" width="600" height="600"></canvas>

  <script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    
    const gridSize = 100;
    let clickCount = 0;
    
    ctx.fillStyle = '#fff';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    canvas.addEventListener('click', e => {
      const rect = canvas.getBoundingClientRect();
      const x = e.clientX - rect.left;
      const y = e.clientY - rect.top;
      
      const gridX = clickCount % 3;
      const gridY = Math.floor(clickCount / 3) % 3;
      
      const baseX = gridX * gridSize;
      const baseY = gridY * gridSize;
      
      for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 3; j++) {
          ctx.beginPath();
          ctx.arc(
            baseX + i * 30 + 15,
            baseY + j * 30 + 15,
            8,
            0,
            Math.PI * 2
          );
          ctx.fillStyle = `hsl(${Math.random() * 360}, 70%, 60%)`;
          ctx.fill();
        }
      }
      
      clickCount++;
    });
  </script>
</body>
</html>
```

### Exercise 2: Line Thickness Preview

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Thickness Preview</title>
  <style>
    body { margin: 0; background: #1e1e2e; padding: 20px; }
    canvas { background: white; box-shadow: 0 0 20px rgba(0,0,0,0.5); display: block; margin-bottom: 20px; }
    input { width: 200px; height: 8px; cursor: pointer; }
  </style>
</head>
<body>
  <div>
    <label>Brush Size: </label>
    <input type="range" id="sizeSlider" min="1" max="100" value="10">
  </div>
  <canvas id="canvas" width="800" height="400"></canvas>

  <script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const sizeSlider = document.getElementById('sizeSlider');
    
    let previewX = canvas.width / 2;
    let previewY = canvas.height / 2;
    let savedImageData = null;
    
    ctx.fillStyle = '#fff';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    sizeSlider.addEventListener('input', e => {
      // Restore previous state
      if (savedImageData) ctx.putImageData(savedImageData, 0, 0);
      
      // Save current state and draw preview
      savedImageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      
      const size = parseInt(e.target.value);
      ctx.beginPath();
      ctx.arc(previewX, previewY, size / 2, 0, Math.PI * 2);
      ctx.fillStyle = '#000';
      ctx.fill();
    });
    
    canvas.addEventListener('mousemove', e => {
      const rect = canvas.getBoundingClientRect();
      previewX = e.clientX - rect.left;
      previewY = e.clientY - rect.top;
      
      // Restore and redraw preview at new position
      if (savedImageData) ctx.putImageData(savedImageData, 0, 0);
      
      savedImageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      const size = parseInt(sizeSlider.value);
      ctx.beginPath();
      ctx.arc(previewX, previewY, size / 2, 0, Math.PI * 2);
      ctx.fillStyle = '#0002';
      ctx.fill();
    });
    
    canvas.addEventListener('mouseleave', () => {
      if (savedImageData) ctx.putImageData(savedImageData, 0, 0);
      savedImageData = null;
    });
  </script>
</body>
</html>
```

### Exercise 3: Spray Paint Tool

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Spray Paint</title>
  <style>
    body { margin: 0; background: #1e1e2e; display: flex; justify-content: center; align-items: center; height: 100vh; }
    canvas { background: white; box-shadow: 0 0 20px rgba(0,0,0,0.5); cursor: crosshair; }
  </style>
</head>
<body>
  <canvas id="canvas" width="800" height="600"></canvas>

  <script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    
    let isSpaying = false;
    const sprayRadius = 30;
    const dotsPerFrame = 15;
    
    ctx.fillStyle = '#fff';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    canvas.addEventListener('mousedown', () => {
      isSpaying = true;
    });
    
    canvas.addEventListener('mouseup', () => {
      isSpaying = false;
    });
    
    canvas.addEventListener('mouseleave', () => {
      isSpaying = false;
    });
    
    canvas.addEventListener('mousemove', e => {
      if (!isSpaying) return;
      
      const rect = canvas.getBoundingClientRect();
      const x = e.clientX - rect.left;
      const y = e.clientY - rect.top;
      
      for (let i = 0; i < dotsPerFrame; i++) {
        const angle = Math.random() * Math.PI * 2;
        const distance = Math.random() * sprayRadius;
        const dotX = x + Math.cos(angle) * distance;
        const dotY = y + Math.sin(angle) * distance;
        const dotSize = Math.random() * 2 + 1;
        
        ctx.beginPath();
        ctx.arc(dotX, dotY, dotSize, 0, Math.PI * 2);
        ctx.fillStyle = `rgba(0, 0, 0, ${Math.random() * 0.7 + 0.3})`;
        ctx.fill();
      }
    });
  </script>
</body>
</html>
```

## What You Learned

| Concept | What It Does |
|---------|-----------|
| `getContext('2d')` | Returns the 2D drawing context object with all canvas methods |
| `beginPath()` | Starts a new drawing path |
| `moveTo(x, y)` | Moves the brush position without drawing |
| `lineTo(x, y)` | Draws a line from the current position to (x, y) |
| `stroke()` | Draws the outline of the current path |
| `strokeRect()` | Draws an outlined rectangle |
| `ellipse()` | Draws an oval or circle using center and radii |
| `getImageData()` | Captures all pixel data from a canvas region |
| `putImageData()` | Restores captured pixel data back to the canvas |
| `save() / restore()` | Saves and restores the drawing state (colors, widths, etc.) |
| `getBoundingClientRect()` | Gets screen coordinates of the canvas element |
| `toDataURL()` | Exports canvas as a PNG data URL for saving |

## Building with Claude

- "Add a color-by-numbers mode where I can click regions and they auto-fill with the selected color."
- "Create a shape stamp tool that lets me click to place pre-drawn shapes like stars, hearts, and arrows."
- "Build a layers feature where I can have multiple transparent drawing layers and toggle them on/off."
- "Add a blur tool that smooths out colors in a circular area, like a smudge brush."
- "Implement a gradient tool that fills an area with a smooth color transition between two selected colors."

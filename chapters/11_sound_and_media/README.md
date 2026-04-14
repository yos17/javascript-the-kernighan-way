# Chapter 11: Sound and Media

Your browser is a synthesizer. We'll build a drum machine that generates every sound from scratch using the Web Audio API — no audio files, no external libraries, just math and oscillators.

---

## The Program: Beat Maker

Open `drum_machine.html`. Click pads to hear sounds, toggle steps in the sequencer grid, set your BPM, and press Play to loop your beat. Press SPACE to play/stop; press keys 1–8 to trigger drums manually.

## The Complete Program

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Drum Machine - Chapter 11</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }

        body {
            font-family: 'Courier New', monospace;
            background: linear-gradient(135deg, #0a0e27 0%, #16213e 100%);
            color: #00ff88;
            padding: 20px;
            min-height: 100vh;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: #0f1419;
            border: 2px solid #00ff88;
            border-radius: 8px;
            padding: 30px;
            box-shadow: 0 0 20px rgba(0, 255, 136, 0.3);
        }

        h1 {
            text-align: center;
            margin-bottom: 30px;
            font-size: 2.5em;
            text-shadow: 0 0 10px #00ff88;
            letter-spacing: 2px;
        }

        .controls {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
            background: #1a1f2e;
            padding: 20px;
            border-radius: 6px;
            border: 1px solid #00ff88;
        }

        .control-group { display: flex; flex-direction: column; gap: 8px; }
        .control-group label {
            font-size: 0.85em;
            text-transform: uppercase;
            letter-spacing: 1px;
            color: #00ff88;
        }

        .control-group input[type="range"],
        .control-group input[type="number"] {
            padding: 8px;
            background: #0f1419;
            border: 1px solid #00ff88;
            color: #00ff88;
            border-radius: 4px;
            cursor: pointer;
            font-family: 'Courier New', monospace;
        }

        .control-group input[type="range"] {
            cursor: pointer;
            height: 6px;
            -webkit-appearance: none;
            appearance: none;
        }

        .control-group input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 18px;
            height: 18px;
            border-radius: 50%;
            background: #00ff88;
            cursor: pointer;
            box-shadow: 0 0 8px rgba(0, 255, 136, 0.6);
        }

        .button-group { display: flex; gap: 10px; flex-wrap: wrap; }

        button {
            padding: 10px 20px;
            background: #00ff88;
            color: #0f1419;
            border: none;
            border-radius: 4px;
            font-family: 'Courier New', monospace;
            font-weight: bold;
            cursor: pointer;
            text-transform: uppercase;
            font-size: 0.9em;
            letter-spacing: 1px;
            transition: all 0.2s ease;
            flex: 1;
            min-width: 100px;
        }

        button:hover { box-shadow: 0 0 15px rgba(0, 255, 136, 0.8); transform: translateY(-2px); }
        button:active { transform: translateY(0); }
        button.playing { background: #ff0055; box-shadow: 0 0 20px rgba(255, 0, 85, 0.8); }

        .bpm-display {
            font-size: 1.8em;
            font-weight: bold;
            text-shadow: 0 0 10px #00ff88;
            text-align: center;
        }

        .sequencer {
            background: #1a1f2e;
            padding: 20px;
            border-radius: 6px;
            border: 2px solid #00ff88;
            overflow-x: auto;
        }

        .grid {
            display: grid;
            grid-template-columns: 80px repeat(16, 1fr);
            gap: 4px;
            min-width: min-content;
        }

        .drum-label {
            display: flex;
            align-items: center;
            justify-content: center;
            background: #0f1419;
            border: 1px solid #00ff88;
            border-radius: 4px;
            font-size: 0.75em;
            font-weight: bold;
            padding: 8px;
            color: #00ff88;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        .step-pad {
            aspect-ratio: 1;
            background: #2a3f5f;
            border: 1px solid #00ff88;
            border-radius: 4px;
            cursor: pointer;
            transition: all 0.1s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 0.7em;
        }

        .step-pad:hover { background: #3d5580; }
        .step-pad.active {
            background: #00ff88;
            box-shadow: 0 0 10px rgba(0, 255, 136, 0.8), inset 0 0 10px rgba(0, 255, 136, 0.4);
            color: #0f1419;
        }
        .step-pad.current {
            border-color: #ff0055;
            border-width: 2px;
            box-shadow: 0 0 15px rgba(255, 0, 85, 0.6);
        }

        .info {
            margin-top: 20px;
            padding: 15px;
            background: #1a1f2e;
            border-left: 3px solid #00ff88;
            font-size: 0.9em;
            line-height: 1.6;
            color: #aaaaaa;
        }
        .info strong { color: #00ff88; }

        @media (max-width: 768px) {
            .grid { grid-template-columns: 60px repeat(8, 1fr); }
            .drum-label { font-size: 0.6em; }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🥁 DRUM MACHINE</h1>

        <div class="controls">
            <div class="control-group">
                <label>Master Volume</label>
                <input type="range" id="volumeSlider" min="0" max="1" step="0.01" value="0.5">
                <span id="volumeDisplay">50%</span>
            </div>
            <div class="control-group">
                <label>BPM (Tempo)</label>
                <div style="display: flex; gap: 8px;">
                    <input type="range" id="bpmSlider" min="60" max="200" value="120">
                    <input type="number" id="bpmInput" min="60" max="200" value="120" style="width: 70px;">
                </div>
                <div class="bpm-display" id="bpmDisplay">120</div>
            </div>
            <div class="control-group button-group">
                <button id="playBtn">▶ PLAY</button>
                <button id="stopBtn">⏹ STOP</button>
                <button id="clearBtn">🗑 CLEAR</button>
            </div>
            <div class="control-group button-group">
                <button id="preset1Btn">PATTERN: ROCK</button>
                <button id="preset2Btn">PATTERN: FUNK</button>
            </div>
        </div>

        <div class="sequencer">
            <div class="grid" id="grid"></div>
        </div>

        <div class="info">
            <strong>Keyboard Shortcuts:</strong> SPACE = Play/Stop • 1–8 = Trigger Drums • Click pads to toggle steps
        </div>
    </div>

    <script>
        const drumSounds = [
            { name: 'KICK',     icon: '🔴', color: '#ff3366' },
            { name: 'SNARE',    icon: '⭕', color: '#00ff88' },
            { name: 'HI-HAT C', icon: '✨', color: '#ffff00' },
            { name: 'HI-HAT O', icon: '🌟', color: '#ffff00' },
            { name: 'CLAP',     icon: '👏', color: '#ff00ff' },
            { name: 'TOM',      icon: '🥁', color: '#ff9900' },
            { name: 'RIM',      icon: '📍', color: '#00ccff' },
            { name: 'CRASH',    icon: '💥', color: '#ff6600' }
        ];

        let audioContext;
        let masterGain;
        let isPlaying = false;
        let currentStep = 0;
        let lookaheadID = null;
        let nextStepTime = 0;

        const SCHEDULE_AHEAD_TIME = 0.1; // seconds
        const LOOK_AHEAD_TIME = 25;      // milliseconds

        // 8 drums × 16 steps
        const sequencer = Array(8).fill(null).map(() => Array(16).fill(false));

        function init() {
            audioContext = new (window.AudioContext || window.webkitAudioContext)();
            masterGain = audioContext.createGain();
            masterGain.connect(audioContext.destination);
            setupUI();
            setupEventListeners();
        }

        function setupUI() {
            const grid = document.getElementById('grid');
            for (let drumIdx = 0; drumIdx < 8; drumIdx++) {
                const label = document.createElement('div');
                label.className = 'drum-label';
                label.textContent = drumSounds[drumIdx].name;
                grid.appendChild(label);

                for (let stepIdx = 0; stepIdx < 16; stepIdx++) {
                    const pad = document.createElement('div');
                    pad.className = 'step-pad';
                    pad.dataset.drum = drumIdx;
                    pad.dataset.step = stepIdx;
                    pad.textContent = stepIdx % 4 === 0 ? (stepIdx / 4 + 1) : '';
                    pad.addEventListener('click', () => toggleStep(drumIdx, stepIdx, pad));
                    grid.appendChild(pad);
                }
            }
        }

        function setupEventListeners() {
            document.getElementById('playBtn').addEventListener('click', play);
            document.getElementById('stopBtn').addEventListener('click', stop);
            document.getElementById('clearBtn').addEventListener('click', clearSequencer);
            document.getElementById('preset1Btn').addEventListener('click', () => loadPreset(presets.rock));
            document.getElementById('preset2Btn').addEventListener('click', () => loadPreset(presets.funk));

            document.getElementById('volumeSlider').addEventListener('input', (e) => {
                const vol = parseFloat(e.target.value);
                masterGain.gain.value = vol;
                document.getElementById('volumeDisplay').textContent = Math.round(vol * 100) + '%';
            });

            document.getElementById('bpmSlider').addEventListener('input', (e) => {
                const bpm = parseInt(e.target.value);
                document.getElementById('bpmInput').value = bpm;
                document.getElementById('bpmDisplay').textContent = bpm;
            });

            document.getElementById('bpmInput').addEventListener('input', (e) => {
                let bpm = Math.max(60, Math.min(200, parseInt(e.target.value) || 120));
                document.getElementById('bpmSlider').value = bpm;
                document.getElementById('bpmDisplay').textContent = bpm;
            });

            document.addEventListener('keydown', (e) => {
                if (e.code === 'Space') { e.preventDefault(); isPlaying ? stop() : play(); }
                const keyNum = parseInt(e.key);
                if (keyNum >= 1 && keyNum <= 8) triggerDrum(keyNum - 1);
            });
        }

        function toggleStep(drumIdx, stepIdx, padElement) {
            sequencer[drumIdx][stepIdx] = !sequencer[drumIdx][stepIdx];
            padElement.classList.toggle('active', sequencer[drumIdx][stepIdx]);
        }

        function clearSequencer() {
            sequencer.forEach(row => row.fill(false));
            document.querySelectorAll('.step-pad.active').forEach(pad => pad.classList.remove('active'));
        }

        function getBPM() { return parseInt(document.getElementById('bpmSlider').value); }

        function play() {
            isPlaying = true;
            nextStepTime = audioContext.currentTime;
            currentStep = 0;
            document.getElementById('playBtn').classList.add('playing');
            lookaheadID = setInterval(scheduleNotes, LOOK_AHEAD_TIME);
        }

        function stop() {
            isPlaying = false;
            currentStep = 0;
            clearInterval(lookaheadID);
            clearCurrentHighlight();
            document.getElementById('playBtn').classList.remove('playing');
        }

        function scheduleNotes() {
            const scheduleAheadTime = audioContext.currentTime + SCHEDULE_AHEAD_TIME;
            while (nextStepTime < scheduleAheadTime) {
                scheduleStep(currentStep, nextStepTime);
                advanceStep();
            }
        }

        function scheduleStep(stepIdx, time) {
            for (let drumIdx = 0; drumIdx < 8; drumIdx++) {
                if (sequencer[drumIdx][stepIdx]) triggerDrumAt(drumIdx, time);
            }
        }

        function advanceStep() {
            currentStep = (currentStep + 1) % 16;
            const beatDuration = 60 / getBPM() / 4; // 16th note
            nextStepTime += beatDuration;
            updateCurrentStepHighlight();
        }

        function updateCurrentStepHighlight() {
            clearCurrentHighlight();
            document.querySelectorAll(`[data-step="${currentStep}"]`).forEach(pad => {
                pad.classList.add('current');
            });
        }

        function clearCurrentHighlight() {
            document.querySelectorAll('.step-pad.current').forEach(pad => pad.classList.remove('current'));
        }

        function triggerDrum(drumIdx) { triggerDrumAt(drumIdx, audioContext.currentTime); }

        function triggerDrumAt(drumIdx, time) {
            switch (drumIdx) {
                case 0: synthesizeKick(time);        break;
                case 1: synthesizeSnare(time);       break;
                case 2: synthesizeHiHatClosed(time); break;
                case 3: synthesizeHiHatOpen(time);   break;
                case 4: synthesizeClap(time);        break;
                case 5: synthesizeTom(time);         break;
                case 6: synthesizeRim(time);         break;
                case 7: synthesizeCrash(time);       break;
            }
        }

        // ─── Drum Synthesis ──────────────────────────────────────────────────────

        function synthesizeKick(time) {
            const osc = audioContext.createOscillator();
            const gain = audioContext.createGain();
            osc.connect(gain); gain.connect(masterGain);
            osc.type = 'sine';
            osc.frequency.setValueAtTime(150, time);
            osc.frequency.exponentialRampToValueAtTime(0.01, time + 0.5);
            gain.gain.setValueAtTime(0.8, time);
            gain.gain.exponentialRampToValueAtTime(0.01, time + 0.5);
            osc.start(time); osc.stop(time + 0.5);
        }

        function synthesizeSnare(time) {
            const buf = audioContext.createBuffer(1, audioContext.sampleRate * 0.2, audioContext.sampleRate);
            const data = buf.getChannelData(0);
            for (let i = 0; i < buf.length; i++) data[i] = Math.random() * 2 - 1;

            const src = audioContext.createBufferSource(); src.buffer = buf;
            const filter = audioContext.createBiquadFilter();
            filter.type = 'band-pass'; filter.frequency.value = 3000; filter.Q.value = 2;
            const gain = audioContext.createGain();
            src.connect(filter); filter.connect(gain); gain.connect(masterGain);
            gain.gain.setValueAtTime(0.5, time);
            gain.gain.exponentialRampToValueAtTime(0.01, time + 0.15);
            src.start(time); src.stop(time + 0.15);

            const osc = audioContext.createOscillator();
            const oscGain = audioContext.createGain();
            osc.connect(oscGain); oscGain.connect(masterGain);
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(200, time);
            osc.frequency.linearRampToValueAtTime(100, time + 0.1);
            oscGain.gain.setValueAtTime(0.3, time);
            oscGain.gain.exponentialRampToValueAtTime(0.01, time + 0.1);
            osc.start(time); osc.stop(time + 0.1);
        }

        function synthesizeHiHatClosed(time) {
            const buf = audioContext.createBuffer(1, audioContext.sampleRate * 0.08, audioContext.sampleRate);
            const data = buf.getChannelData(0);
            for (let i = 0; i < buf.length; i++) data[i] = Math.random() * 2 - 1;
            const src = audioContext.createBufferSource(); src.buffer = buf;
            const filter = audioContext.createBiquadFilter();
            filter.type = 'high-pass'; filter.frequency.value = 8000;
            const gain = audioContext.createGain();
            src.connect(filter); filter.connect(gain); gain.connect(masterGain);
            gain.gain.setValueAtTime(0.4, time);
            gain.gain.exponentialRampToValueAtTime(0.01, time + 0.08);
            src.start(time); src.stop(time + 0.08);
        }

        function synthesizeHiHatOpen(time) {
            const buf = audioContext.createBuffer(1, audioContext.sampleRate * 0.3, audioContext.sampleRate);
            const data = buf.getChannelData(0);
            for (let i = 0; i < buf.length; i++) data[i] = Math.random() * 2 - 1;
            const src = audioContext.createBufferSource(); src.buffer = buf;
            const filter = audioContext.createBiquadFilter();
            filter.type = 'high-pass'; filter.frequency.value = 7000;
            const gain = audioContext.createGain();
            src.connect(filter); filter.connect(gain); gain.connect(masterGain);
            gain.gain.setValueAtTime(0.3, time);
            gain.gain.exponentialRampToValueAtTime(0.01, time + 0.3);
            src.start(time); src.stop(time + 0.3);
        }

        function synthesizeClap(time) {
            for (let i = 0; i < 3; i++) {
                const delay = i * 0.015;
                const buf = audioContext.createBuffer(1, audioContext.sampleRate * 0.08, audioContext.sampleRate);
                const data = buf.getChannelData(0);
                for (let j = 0; j < buf.length; j++) data[j] = Math.random() * 2 - 1;
                const src = audioContext.createBufferSource(); src.buffer = buf;
                const filter = audioContext.createBiquadFilter();
                filter.type = 'band-pass'; filter.frequency.value = 2000; filter.Q.value = 1.5;
                const gain = audioContext.createGain();
                src.connect(filter); filter.connect(gain); gain.connect(masterGain);
                gain.gain.setValueAtTime(0.4, time + delay);
                gain.gain.exponentialRampToValueAtTime(0.01, time + delay + 0.08);
                src.start(time + delay); src.stop(time + delay + 0.08);
            }
        }

        function synthesizeTom(time) {
            const osc = audioContext.createOscillator();
            const gain = audioContext.createGain();
            osc.connect(gain); gain.connect(masterGain);
            osc.type = 'sine';
            osc.frequency.setValueAtTime(400, time);
            osc.frequency.exponentialRampToValueAtTime(150, time + 0.15);
            gain.gain.setValueAtTime(0.6, time);
            gain.gain.exponentialRampToValueAtTime(0.01, time + 0.15);
            osc.start(time); osc.stop(time + 0.15);
        }

        function synthesizeRim(time) {
            const osc = audioContext.createOscillator();
            const gain = audioContext.createGain();
            osc.connect(gain); gain.connect(masterGain);
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(8000, time);
            osc.frequency.exponentialRampToValueAtTime(5000, time + 0.05);
            gain.gain.setValueAtTime(0.5, time);
            gain.gain.exponentialRampToValueAtTime(0.01, time + 0.05);
            osc.start(time); osc.stop(time + 0.05);
        }

        function synthesizeCrash(time) {
            const buf = audioContext.createBuffer(1, audioContext.sampleRate * 0.8, audioContext.sampleRate);
            const data = buf.getChannelData(0);
            for (let i = 0; i < buf.length; i++) data[i] = Math.random() * 2 - 1;
            const src = audioContext.createBufferSource(); src.buffer = buf;
            const filter = audioContext.createBiquadFilter();
            filter.type = 'high-pass'; filter.frequency.value = 5000;
            const gain = audioContext.createGain();
            src.connect(filter); filter.connect(gain); gain.connect(masterGain);
            gain.gain.setValueAtTime(0.4, time);
            gain.gain.exponentialRampToValueAtTime(0.05, time + 0.3);
            gain.gain.exponentialRampToValueAtTime(0.01, time + 0.8);
            src.start(time); src.stop(time + 0.8);
        }

        // ─── Preset Patterns ─────────────────────────────────────────────────────

        const presets = {
            rock: [
                [true,  false, false, false, true,  false, false, false, true,  false, false, false, true,  false, false, false],
                [false, false, true,  false, false, false, true,  false, false, false, true,  false, false, false, true,  false],
                [true,  true,  true,  true,  true,  true,  true,  true,  true,  true,  true,  true,  true,  true,  true,  true ],
                [false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false],
                [false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false],
                [false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false],
                [false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false],
                [true,  false, false, false, false, false, false, false, false, false, false, false, false, false, false, false]
            ],
            funk: [
                [true,  false, false, true,  false, false, true,  false, true,  false, false, true,  false, false, true,  false],
                [false, false, true,  false, false, true,  false, false, false, false, true,  false, false, true,  false, false],
                [true,  false, true,  false, true,  false, true,  false, true,  false, true,  false, true,  false, true,  false],
                [false, false, false, true,  false, true,  false, false, false, false, false, true,  false, true,  false, false],
                [false, false, false, true,  false, false, false, true,  false, false, false, true,  false, false, false, true ],
                [false, false, false, false, false, false, false, false, true,  true,  false, false, false, false, false, false],
                [false, true,  false, false, true,  false, false, true,  false, false, true,  false, false, true,  false, false],
                [false, false, false, false, false, false, false, false, true,  false, false, false, false, false, false, false]
            ]
        };

        function loadPreset(preset) {
            for (let i = 0; i < 8; i++)
                for (let j = 0; j < 16; j++)
                    sequencer[i][j] = preset[i][j];
            document.querySelectorAll('.step-pad').forEach(pad => {
                pad.classList.toggle('active', sequencer[parseInt(pad.dataset.drum)][parseInt(pad.dataset.step)]);
            });
        }

        init();
    </script>
</body>
</html>
```

## How It Works

### The Web Audio API

Browsers include a complete audio engine. You access it through an `AudioContext`:

```js
let audioCtx = null;

function getCtx() {
  if (!audioCtx) audioCtx = new AudioContext();
  if (audioCtx.state === 'suspended') audioCtx.resume();
  return audioCtx;
}
```

We lazily create the context on first use — browsers require a user gesture before audio can play, so creating it on page load would fail silently. Calling `resume()` handles the case where the browser suspended the context.

### The Audio Graph

Web Audio works as a graph of nodes connected together, flowing into `ctx.destination` (your speakers):

```
OscillatorNode → GainNode → destination
```

Every sound is built by chaining nodes:

```js
function kick(time) {
  const ctx = getCtx();
  const osc  = ctx.createOscillator();
  const gain = ctx.createGain();

  osc.connect(gain);
  gain.connect(ctx.destination);

  osc.frequency.setValueAtTime(150, time);
  osc.frequency.exponentialRampToValueAtTime(0.01, time + 0.5);
  gain.gain.setValueAtTime(1, time);
  gain.gain.exponentialRampToValueAtTime(0.001, time + 0.5);

  osc.start(time);
  osc.stop(time + 0.5);
}
```

A kick drum is just a sine wave that rapidly drops in pitch. The `exponentialRampToValueAtTime` makes the frequency and volume decay realistically — linear ramps sound unnatural for audio.

### Scheduled vs. Immediate Audio

The `time` parameter is crucial. Web Audio operates on a separate high-precision clock (`ctx.currentTime`). You *schedule* sounds rather than playing them immediately:

```js
osc.start(ctx.currentTime);         // play right now
osc.start(ctx.currentTime + 0.5);   // play half a second from now
```

This is far more accurate than `setTimeout` for music. Timer callbacks can be delayed by JavaScript event processing; scheduled audio plays at the exact sample.

### Synthesizing Drum Sounds

**Kick**: Low-frequency sine wave with fast pitch drop.

**Snare**: White noise (random samples) filtered high-pass, plus a mid-tone sine burst.

**Hi-Hat**: White noise filtered as bandpass at 8kHz, very short duration.

**Clap**: Three rapid bursts of bandpass-filtered noise, slightly offset in time.

```js
// White noise buffer — the raw material for snare, hat, clap
const bufSize = ctx.sampleRate * 0.2;
const buf = ctx.createBuffer(1, bufSize, ctx.sampleRate);
const data = buf.getChannelData(0);
for (let i = 0; i < bufSize; i++) data[i] = Math.random() * 2 - 1;
```

`getChannelData` returns a `Float32Array` of audio samples. Filling it with random values between -1 and 1 creates white noise — all frequencies at equal amplitude.

### The Lookahead Scheduler

Precise sequencing uses a lookahead pattern:

```js
const LOOKAHEAD = 0.1;        // schedule 100ms ahead
const SCHEDULE_INTERVAL = 25; // check every 25ms

function scheduler() {
  const ctx = getCtx();
  while (nextStepTime < ctx.currentTime + LOOKAHEAD) {
    scheduleStep(currentStep, nextStepTime);
    nextStepTime += stepDuration();
    currentStep = (currentStep + 1) % STEPS;
  }
  schedulerTimer = setTimeout(scheduler, SCHEDULE_INTERVAL);
}
```

The scheduler runs every 25ms but schedules any steps falling within the next 100ms. This overlap ensures no steps are missed even if the JavaScript thread is briefly busy. It's the standard pattern for Web Audio sequencers.

### Separating Audio Timing from UI Updates

Audio is scheduled in the future; the UI highlight must follow *when that future arrives*:

```js
const timeUntil = (nextStepTime - ctx.currentTime) * 1000;
setTimeout(() => highlightStep(stepToHighlight), Math.max(0, timeUntil));
```

We calculate how many milliseconds until the scheduled step plays, then use `setTimeout` to visually highlight it at the right moment. Audio scheduling and DOM animation are separate concerns on separate timers.

### Data Model: Pattern as 2D Array

The beat pattern is a 2D array — rows are tracks, columns are steps:

```js
let pattern = TRACKS.map(() => new Array(STEPS).fill(false));
```

Toggling a step:
```js
step.addEventListener('click', () => {
  pattern[ti][s] = !pattern[ti][s];
  step.classList.toggle('on', pattern[ti][s]);
});
```

When the sequencer reaches step `s`, it checks `pattern[ti][s]` for each track and fires any that are `true`. Simple boolean array, clean separation between data and audio.

### BPM to Step Duration

```js
function stepDuration() {
  return 60 / bpm() / 4; // 16th notes
}
```

At 120 BPM: `60 / 120 / 4 = 0.125` seconds per 16th note — eight notes per second. Changing BPM only affects new scheduling calls; already-scheduled notes play unchanged.

---

## Try It

1. **Record a pattern**: start with kick on steps 1, 5, 9, 13 — snare on 5, 13 — hi-hat on every step. This is a basic 4/4 beat.
2. **Swing**: add `(step % 2 === 1 ? 0.02 : 0)` to `nextStepTime` for odd steps. Even steps play on the grid; odd steps are pushed slightly late — classic swing feel.
3. **Volume per track**: add a gain node between each synth and `destination`, controlled by a per-track slider.
4. **Save patterns**: serialize `pattern` to JSON and store with `localStorage.setItem('beat', JSON.stringify(pattern))`. Load on startup.

---

## Exercises

### Exercise 1 — Mute/Solo Tracks

Add a mute button (🔇) and solo button (S) to each track row. Muted tracks don't play even if their steps are on. Solo plays only that track.

**Spec:**
- Per-track `muted` boolean array
- Clicking mute toggles it and grays out the track's steps
- Solo un-mutes only that track, mutes all others
- Clicking solo again returns to previous mute state

### Exercise 2 — Pattern Copy/Paste

Add Copy and Paste buttons that let you duplicate the current pattern to a second pattern slot. Toggle between pattern A and pattern B.

**Spec:**
- Two buttons: `[A]` and `[B]`
- Active pattern highlighted
- Switching patterns updates the grid display
- A "Copy A → B" button duplicates the current pattern

### Exercise 3 — Pitch Pads

Add a second row of 8 pads tuned to a pentatonic scale (e.g., C4, D4, E4, G4, A4, C5, D5, E5). Each pad plays a melodic tone using an `OscillatorNode` with a short envelope.

**Spec:**
- Use frequencies: 261.6, 293.7, 329.6, 392.0, 440.0, 523.3, 587.3, 659.3 Hz
- Use a sawtooth or triangle oscillator with a 0.3s envelope
- Add a `lowpass` filter at ~2000 Hz for a mellow sound
- These pads can also be recorded into a 9th track in the sequencer

---

## Solutions

### Solution 1 — Mute/Solo

```js
let mutedTracks = new Array(TRACKS.length).fill(false);

// In scheduleStep, change the condition:
function scheduleStep(step, time) {
  for (let ti = 0; ti < TRACKS.length; ti++) {
    if (!mutedTracks[ti] && pattern[ti][step]) TRACKS[ti].play(time);
  }
}

// Add to each track row:
const muteBtn = document.createElement('button');
muteBtn.textContent = '🔇';
muteBtn.addEventListener('click', () => {
  mutedTracks[ti] = !mutedTracks[ti];
  muteBtn.style.opacity = mutedTracks[ti] ? '1' : '0.3';
  row.querySelectorAll('.step').forEach(s => {
    s.style.opacity = mutedTracks[ti] ? '0.3' : '1';
  });
});
```

### Solution 2 — Pattern Copy/Paste

```js
let patterns = [
  TRACKS.map(() => new Array(STEPS).fill(false)),
  TRACKS.map(() => new Array(STEPS).fill(false))
];
let activePattern = 0;

function switchPattern(index) {
  activePattern = index;
  pattern = patterns[index];
  document.querySelectorAll('.step').forEach((el, i) => {
    const ti = Math.floor(i / STEPS);
    const s = i % STEPS;
    el.classList.toggle('on', pattern[ti][s]);
  });
}

document.getElementById('copyBtn').addEventListener('click', () => {
  const other = 1 - activePattern;
  patterns[other] = patterns[activePattern].map(row => [...row]);
});
```

### Solution 3 — Pitch Pads

```js
const PENTATONIC = [261.6, 293.7, 329.6, 392.0, 440.0, 523.3, 587.3, 659.3];

function playTone(freq, time) {
  const ctx = getCtx();
  const osc = ctx.createOscillator();
  osc.type = 'triangle';
  osc.frequency.value = freq;
  const filter = ctx.createBiquadFilter();
  filter.type = 'lowpass';
  filter.frequency.value = 2000;
  const gain = ctx.createGain();
  gain.gain.setValueAtTime(0.4, time);
  gain.gain.exponentialRampToValueAtTime(0.001, time + 0.3);
  osc.connect(filter); filter.connect(gain); gain.connect(ctx.destination);
  osc.start(time); osc.stop(time + 0.3);
}

PENTATONIC.forEach((freq, i) => {
  const pad = document.createElement('button');
  pad.className = 'pad pitch-pad';
  pad.textContent = ['C4','D4','E4','G4','A4','C5','D5','E5'][i];
  pad.addEventListener('click', () => playTone(freq, getCtx().currentTime));
  document.getElementById('pitchPads').appendChild(pad);
});
```

---

## What You Learned

| Concept | Where |
|---|---|
| `AudioContext` | `getCtx()` — entry point to Web Audio |
| `OscillatorNode` | Kick, snare tone, rim, bass, pitch pads |
| `GainNode` | Volume envelope on every sound |
| `BiquadFilterNode` | Highpass (snare noise), bandpass (hi-hat) |
| `AudioBuffer` + `BufferSource` | White noise generation |
| Audio scheduling | `setValueAtTime`, `exponentialRampToValueAtTime` |
| `ctx.currentTime` | High-precision audio clock |
| Lookahead scheduler | `scheduler()` with `setTimeout` + `LOOKAHEAD` |
| Audio/UI time separation | `setTimeout` delay for visual highlight |
| `Float32Array` | Raw audio sample buffer |
| 2D array as data model | `pattern[track][step]` |
| BPM → duration math | `60 / bpm / 4` for 16th notes |

---

## Building with Claude

- *"Add a reverb effect using a ConvolverNode with a programmatically generated impulse response — a short burst of decaying noise."*
- *"Add a pitch-shifting effect: when the user holds Shift and clicks a drum pad, play it at 1.5x or 0.75x pitch using a playbackRate on the BufferSource."*
- *"Export the current pattern as a downloadable WAV file by rendering the full 1-bar loop offline using OfflineAudioContext."*

# Chapter 11: Sound and Media

Your browser is a synthesizer. We'll build a drum machine that generates every sound from scratch using the Web Audio API — no audio files, no external libraries, just math and oscillators.

---

## The Program: Beat Maker

Open `drumachine.html`. Click pads to hear sounds, toggle steps in the sequencer grid, set your BPM, and press Play to loop your beat. Keyboard shortcuts: `Q`–`I` for the top row of pads, `A`–`K` for the bottom row, `Space` to play/stop.

## The Complete Program

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Beat Maker</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #111118;
    min-height: 100vh;
    font-family: 'Courier New', monospace;
    color: #ccd6f6;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 24px 16px;
  }
  h1 {
    font-size: 1.6rem;
    letter-spacing: 0.3em;
    color: #64ffda;
    margin-bottom: 4px;
    text-transform: uppercase;
  }
  .subtitle { font-size: 0.75rem; color: #495670; letter-spacing: 0.1em; margin-bottom: 24px; }

  /* Transport bar */
  #transport {
    display: flex;
    align-items: center;
    gap: 20px;
    background: rgba(255,255,255,0.04);
    border: 1px solid rgba(100,255,218,0.15);
    border-radius: 12px;
    padding: 14px 24px;
    margin-bottom: 24px;
    flex-wrap: wrap;
    justify-content: center;
  }
  #playBtn {
    width: 52px; height: 52px;
    border-radius: 50%;
    border: 2px solid #64ffda;
    background: transparent;
    color: #64ffda;
    font-size: 1.4rem;
    cursor: pointer;
    transition: background 0.15s;
    display: flex; align-items: center; justify-content: center;
  }
  #playBtn:hover { background: rgba(100,255,218,0.1); }
  #playBtn.playing { background: rgba(100,255,218,0.15); }
  .bpm-group { display: flex; align-items: center; gap: 10px; }
  .bpm-group label { font-size: 0.75rem; color: #64ffda; letter-spacing: 0.1em; }
  #bpmSlider {
    -webkit-appearance: none;
    width: 120px; height: 4px;
    border-radius: 2px;
    background: #1e2a3a;
    outline: none;
  }
  #bpmSlider::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 16px; height: 16px;
    border-radius: 50%;
    background: #64ffda;
    cursor: pointer;
  }
  #bpmDisplay { font-size: 1.1rem; color: #ccd6f6; min-width: 3ch; text-align: right; }
  #clearBtn {
    padding: 8px 16px;
    border-radius: 6px;
    border: 1px solid #495670;
    background: transparent;
    color: #8892b0;
    font-family: 'Courier New', monospace;
    font-size: 0.75rem;
    cursor: pointer;
    letter-spacing: 0.05em;
  }
  #clearBtn:hover { border-color: #8892b0; color: #ccd6f6; }

  /* Sequencer grid */
  #sequencer {
    width: 100%;
    max-width: 820px;
    background: rgba(255,255,255,0.02);
    border: 1px solid rgba(100,255,218,0.1);
    border-radius: 12px;
    padding: 16px;
    margin-bottom: 24px;
  }
  .track {
    display: grid;
    grid-template-columns: 90px repeat(16, 1fr);
    gap: 4px;
    margin-bottom: 6px;
    align-items: center;
  }
  .track-label {
    font-size: 0.7rem;
    letter-spacing: 0.05em;
    color: #8892b0;
    text-align: right;
    padding-right: 10px;
    cursor: pointer;
    transition: color 0.15s;
  }
  .track-label:hover { color: #ccd6f6; }
  .step {
    height: 32px;
    border-radius: 4px;
    border: 1px solid rgba(255,255,255,0.06);
    background: #1a1a2a;
    cursor: pointer;
    transition: background 0.1s, transform 0.05s;
    position: relative;
    overflow: hidden;
  }
  .step:hover { border-color: rgba(255,255,255,0.2); }
  .step.on { background: var(--track-color, #64ffda); border-color: transparent; }
  .step.on::after {
    content: '';
    position: absolute;
    inset: 0;
    background: rgba(255,255,255,0.2);
    opacity: 0;
  }
  .step.active { box-shadow: 0 0 0 2px rgba(255,255,255,0.7); transform: scaleY(1.08); }
  .step.active.on::after { opacity: 1; }
  .step:nth-child(5n+2) { margin-left: 2px; }

  /* Drum pads */
  #pads {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
    max-width: 500px;
    width: 100%;
  }
  .pad {
    height: 80px;
    border-radius: 10px;
    border: none;
    font-family: 'Courier New', monospace;
    font-size: 0.8rem;
    letter-spacing: 0.05em;
    cursor: pointer;
    transition: transform 0.07s, filter 0.07s;
    position: relative;
    overflow: hidden;
    display: flex; align-items: center; justify-content: center;
    flex-direction: column;
    gap: 4px;
  }
  .pad .pad-icon { font-size: 1.6rem; }
  .pad .pad-name { color: rgba(0,0,0,0.7); font-weight: bold; font-size: 0.7rem; }
  .pad:active, .pad.hit { transform: scale(0.93); filter: brightness(1.3); }
  .pad::after {
    content: '';
    position: absolute;
    inset: 0;
    background: rgba(255,255,255,0.3);
    opacity: 0;
    transition: opacity 0.15s;
  }
  .pad.hit::after { opacity: 1; }
</style>
</head>
<body>
<h1>Beat Maker</h1>
<p class="subtitle">Click pads to play · Toggle steps to sequence · Press play to loop</p>

<div id="transport">
  <button id="playBtn" title="Play/Stop">▶</button>
  <div class="bpm-group">
    <label>BPM</label>
    <input id="bpmSlider" type="range" min="60" max="200" value="120" />
    <span id="bpmDisplay">120</span>
  </div>
  <button id="clearBtn">CLEAR ALL</button>
</div>

<div id="sequencer"></div>
<div id="pads"></div>

<script>
// — Audio engine —
let audioCtx = null;

function getCtx() {
  if (!audioCtx) audioCtx = new AudioContext();
  if (audioCtx.state === 'suspended') audioCtx.resume();
  return audioCtx;
}

const synths = {
  kick(time) {
    const ctx = getCtx();
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.connect(gain); gain.connect(ctx.destination);
    osc.frequency.setValueAtTime(150, time);
    osc.frequency.exponentialRampToValueAtTime(0.01, time + 0.5);
    gain.gain.setValueAtTime(1, time);
    gain.gain.exponentialRampToValueAtTime(0.001, time + 0.5);
    osc.start(time); osc.stop(time + 0.5);
  },
  snare(time) {
    const ctx = getCtx();
    const bufSize = ctx.sampleRate * 0.2;
    const buf = ctx.createBuffer(1, bufSize, ctx.sampleRate);
    const data = buf.getChannelData(0);
    for (let i = 0; i < bufSize; i++) data[i] = Math.random() * 2 - 1;
    const noise = ctx.createBufferSource();
    noise.buffer = buf;
    const noiseFilter = ctx.createBiquadFilter();
    noiseFilter.type = 'highpass'; noiseFilter.frequency.value = 1000;
    const noiseGain = ctx.createGain();
    noiseGain.gain.setValueAtTime(0.8, time);
    noiseGain.gain.exponentialRampToValueAtTime(0.001, time + 0.2);
    noise.connect(noiseFilter); noiseFilter.connect(noiseGain); noiseGain.connect(ctx.destination);
    noise.start(time); noise.stop(time + 0.2);
    const osc = ctx.createOscillator();
    const oscGain = ctx.createGain();
    osc.frequency.value = 200;
    oscGain.gain.setValueAtTime(0.5, time);
    oscGain.gain.exponentialRampToValueAtTime(0.001, time + 0.1);
    osc.connect(oscGain); oscGain.connect(ctx.destination);
    osc.start(time); osc.stop(time + 0.1);
  },
  hihat(time, open = false) {
    const ctx = getCtx();
    const duration = open ? 0.4 : 0.05;
    const bufSize = ctx.sampleRate * duration;
    const buf = ctx.createBuffer(1, bufSize, ctx.sampleRate);
    const data = buf.getChannelData(0);
    for (let i = 0; i < bufSize; i++) data[i] = Math.random() * 2 - 1;
    const noise = ctx.createBufferSource();
    noise.buffer = buf;
    const filter = ctx.createBiquadFilter();
    filter.type = 'bandpass'; filter.frequency.value = 8000;
    const gain = ctx.createGain();
    gain.gain.setValueAtTime(0.4, time);
    gain.gain.exponentialRampToValueAtTime(0.001, time + duration);
    noise.connect(filter); filter.connect(gain); gain.connect(ctx.destination);
    noise.start(time); noise.stop(time + duration);
  },
  clap(time) {
    const ctx = getCtx();
    for (let i = 0; i < 3; i++) {
      const t = time + i * 0.01;
      const bufSize = ctx.sampleRate * 0.1;
      const buf = ctx.createBuffer(1, bufSize, ctx.sampleRate);
      const data = buf.getChannelData(0);
      for (let j = 0; j < bufSize; j++) data[j] = Math.random() * 2 - 1;
      const noise = ctx.createBufferSource();
      noise.buffer = buf;
      const filter = ctx.createBiquadFilter();
      filter.type = 'bandpass'; filter.frequency.value = 1200; filter.Q.value = 0.8;
      const gain = ctx.createGain();
      gain.gain.setValueAtTime(0.6, t);
      gain.gain.exponentialRampToValueAtTime(0.001, t + 0.1);
      noise.connect(filter); filter.connect(gain); gain.connect(ctx.destination);
      noise.start(t); noise.stop(t + 0.1);
    }
  },
  tom(time, freq = 120) {
    const ctx = getCtx();
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.frequency.setValueAtTime(freq, time);
    osc.frequency.exponentialRampToValueAtTime(freq * 0.3, time + 0.3);
    gain.gain.setValueAtTime(0.8, time);
    gain.gain.exponentialRampToValueAtTime(0.001, time + 0.3);
    osc.connect(gain); gain.connect(ctx.destination);
    osc.start(time); osc.stop(time + 0.3);
  },
  rimshot(time) {
    const ctx = getCtx();
    const osc = ctx.createOscillator();
    osc.type = 'square';
    const gain = ctx.createGain();
    osc.frequency.value = 1600;
    gain.gain.setValueAtTime(0.4, time);
    gain.gain.exponentialRampToValueAtTime(0.001, time + 0.05);
    osc.connect(gain); gain.connect(ctx.destination);
    osc.start(time); osc.stop(time + 0.05);
  },
  openhat(time) { synths.hihat(time, true); },
  bass(time) {
    const ctx = getCtx();
    const osc = ctx.createOscillator();
    osc.type = 'sawtooth';
    const filter = ctx.createBiquadFilter();
    filter.type = 'lowpass'; filter.frequency.value = 400;
    const gain = ctx.createGain();
    osc.frequency.setValueAtTime(80, time);
    gain.gain.setValueAtTime(0.5, time);
    gain.gain.exponentialRampToValueAtTime(0.001, time + 0.4);
    osc.connect(filter); filter.connect(gain); gain.connect(ctx.destination);
    osc.start(time); osc.stop(time + 0.4);
  }
};

const TRACKS = [
  { id: 'kick',    label: 'KICK',     icon: '🥁', color: '#ff6b6b', play: t => synths.kick(t) },
  { id: 'snare',   label: 'SNARE',    icon: '🪘', color: '#ffd93d', play: t => synths.snare(t) },
  { id: 'hihat',   label: 'HI-HAT',   icon: '🎩', color: '#6bcb77', play: t => synths.hihat(t) },
  { id: 'openhat', label: 'OPEN HAT', icon: '🎶', color: '#4d96ff', play: t => synths.openhat(t) },
  { id: 'clap',    label: 'CLAP',     icon: '👏', color: '#ff922b', play: t => synths.clap(t) },
  { id: 'rimshot', label: 'RIM',      icon: '🎯', color: '#cc5de8', play: t => synths.rimshot(t) },
  { id: 'tom',     label: 'TOM HI',   icon: '🔵', color: '#22d3ee', play: t => synths.tom(t, 200) },
  { id: 'bass',    label: 'BASS',     icon: '🟣', color: '#a78bfa', play: t => synths.bass(t) },
];

const STEPS = 16;
let pattern = TRACKS.map(() => new Array(STEPS).fill(false));

const sequencerEl = document.getElementById('sequencer');
const padsEl = document.getElementById('pads');

TRACKS.forEach((track, ti) => {
  const row = document.createElement('div');
  row.className = 'track';
  const lbl = document.createElement('div');
  lbl.className = 'track-label';
  lbl.textContent = track.label;
  lbl.title = 'Click to play';
  lbl.addEventListener('click', () => playTrack(ti));
  row.appendChild(lbl);

  for (let s = 0; s < STEPS; s++) {
    const step = document.createElement('div');
    step.className = 'step';
    step.style.setProperty('--track-color', track.color);
    step.addEventListener('click', () => {
      pattern[ti][s] = !pattern[ti][s];
      step.classList.toggle('on', pattern[ti][s]);
    });
    row.appendChild(step);
  }
  sequencerEl.appendChild(row);

  const pad = document.createElement('button');
  pad.className = 'pad';
  pad.style.background = track.color + '33';
  pad.style.border = `2px solid ${track.color}66`;
  pad.innerHTML = `<span class="pad-icon">${track.icon}</span><span class="pad-name">${track.label}</span>`;
  pad.addEventListener('click', () => {
    playTrack(ti);
    pad.classList.add('hit');
    setTimeout(() => pad.classList.remove('hit'), 150);
  });
  padsEl.appendChild(pad);
});

function playTrack(ti) {
  const ctx = getCtx();
  TRACKS[ti].play(ctx.currentTime);
}

let isPlaying = false;
let currentStep = 0;
let nextStepTime = 0;
let schedulerTimer = null;
const LOOKAHEAD = 0.1;
const SCHEDULE_INTERVAL = 25;

function bpm() { return parseInt(document.getElementById('bpmSlider').value); }
function stepDuration() { return 60 / bpm() / 4; }

function scheduleStep(step, time) {
  for (let ti = 0; ti < TRACKS.length; ti++) {
    if (pattern[ti][step]) TRACKS[ti].play(time);
  }
}

function scheduler() {
  const ctx = getCtx();
  while (nextStepTime < ctx.currentTime + LOOKAHEAD) {
    scheduleStep(currentStep, nextStepTime);
    const stepToHighlight = currentStep;
    const timeUntil = (nextStepTime - ctx.currentTime) * 1000;
    setTimeout(() => highlightStep(stepToHighlight), Math.max(0, timeUntil));
    nextStepTime += stepDuration();
    currentStep = (currentStep + 1) % STEPS;
  }
  schedulerTimer = setTimeout(scheduler, SCHEDULE_INTERVAL);
}

function highlightStep(step) {
  document.querySelectorAll('.step').forEach((el, i) => {
    el.classList.toggle('active', i % STEPS === step);
  });
}

function play() {
  const ctx = getCtx();
  currentStep = 0;
  nextStepTime = ctx.currentTime + 0.05;
  scheduler();
}

function stop() {
  clearTimeout(schedulerTimer);
  schedulerTimer = null;
  document.querySelectorAll('.step').forEach(el => el.classList.remove('active'));
}

document.getElementById('playBtn').addEventListener('click', () => {
  isPlaying = !isPlaying;
  const btn = document.getElementById('playBtn');
  if (isPlaying) {
    btn.textContent = '⏹';
    btn.classList.add('playing');
    play();
  } else {
    btn.textContent = '▶';
    btn.classList.remove('playing');
    stop();
  }
});

document.getElementById('bpmSlider').addEventListener('input', e => {
  document.getElementById('bpmDisplay').textContent = e.target.value;
});

document.getElementById('clearBtn').addEventListener('click', () => {
  pattern = TRACKS.map(() => new Array(STEPS).fill(false));
  document.querySelectorAll('.step').forEach(el => el.classList.remove('on'));
});

const PAD_KEYS_1 = ['q','w','e','r','t','y','u','i'];
const PAD_KEYS_2 = ['a','s','d','f','g','h','j','k'];
document.addEventListener('keydown', e => {
  if (e.key === ' ') {
    e.preventDefault();
    document.getElementById('playBtn').click();
    return;
  }
  const i1 = PAD_KEYS_1.indexOf(e.key);
  const i2 = PAD_KEYS_2.indexOf(e.key);
  const ti = i1 >= 0 ? i1 : i2 >= 0 ? i2 : -1;
  if (ti >= 0 && ti < TRACKS.length) {
    playTrack(ti);
    const pads = document.querySelectorAll('.pad');
    pads[ti].classList.add('hit');
    setTimeout(() => pads[ti].classList.remove('hit'), 150);
  }
});
</script>
</body>
</html>
```

---

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

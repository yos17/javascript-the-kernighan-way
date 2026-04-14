# Chapter 11: Sound and Media

## Why Web Audio Exists

Historically, playing sound in a browser required loading an audio file and pressing play. This works for music and sound effects — but it's terrible for musical instruments and synthesizers. You can't have a 10ms delay between a key press and a note playing. You can't synthesize novel sounds from scratch. You can't schedule music with sample-accurate precision.

The Web Audio API solves all of this by exposing a **low-level audio processing graph** in JavaScript. You build chains of audio nodes that generate, filter, mix, and output sound — entirely in memory, no files needed.

Your browser is a synthesizer. This chapter shows you how to play it.

## The Program: Beat Maker

A drum machine with 8 tracks and 16 steps per bar. Click pads to hear sounds, toggle steps in the sequencer grid, set your BPM, and press Play to loop your beat. Every sound is synthesized from scratch — no audio files.

Open `drumachine.html` to use it. Keyboard shortcuts: `Q`–`I` for the top row of pads, `A`–`K` for the bottom row, `Space` to play/stop.

## How It Works

### The Audio Graph

Web Audio processes sound as a **directed graph** of nodes. Each node does one job (generate a tone, control volume, filter frequencies). Nodes connect together, flowing into `ctx.destination` (your speakers):

```
  OscillatorNode         BufferSourceNode
  (sine wave tone)       (noise buffer)
        │                       │
        ▼                       ▼
    GainNode               BiquadFilterNode
   (volume)                 (frequency filter)
        │                       │
        └──────────┬────────────┘
                   ▼
             ctx.destination
              (your speakers)
```

Every sound in the drum machine is a small graph like this, assembled and discarded for each individual hit.

### Lazy Context Creation

```javascript
let audioCtx = null;

function getCtx() {
  if (!audioCtx) audioCtx = new AudioContext();
  if (audioCtx.state === 'suspended') audioCtx.resume();
  return audioCtx;
}
```

**Why not create the AudioContext on page load?** Browsers require a user gesture (click, keypress) before audio can play. Creating the context before a gesture silently fails or throws. Creating it on first use (lazy initialization) guarantees a gesture has happened.

### Synthesizing Drum Sounds

**Kick drum**: A sine wave that rapidly drops from 150Hz to near silence. Low frequency = thump.

```javascript
function kick(time) {
  const ctx  = getCtx();
  const osc  = ctx.createOscillator();
  const gain = ctx.createGain();

  osc.connect(gain);
  gain.connect(ctx.destination);

  // Frequency drops from 150Hz to ~0 over 0.5 seconds
  osc.frequency.setValueAtTime(150, time);
  osc.frequency.exponentialRampToValueAtTime(0.01, time + 0.5);

  // Volume drops from 1 to near-zero
  gain.gain.setValueAtTime(1, time);
  gain.gain.exponentialRampToValueAtTime(0.001, time + 0.5);

  osc.start(time);
  osc.stop(time + 0.5);
}
```

**Why exponential ramp, not linear?** Human hearing is logarithmic — we perceive pitch and volume on a logarithmic scale. A linear frequency drop from 150Hz to 0Hz sounds wrong. An exponential drop sounds natural.

**Snare**: White noise filtered to emphasize mid-high frequencies, plus a short tone burst.

White noise is random samples — all frequencies at equal amplitude:

```javascript
const bufSize = ctx.sampleRate * 0.2;  // 0.2 seconds of samples
const buf = ctx.createBuffer(1, bufSize, ctx.sampleRate);
const data = buf.getChannelData(0);    // Float32Array
for (let i = 0; i < bufSize; i++) {
  data[i] = Math.random() * 2 - 1;    // random values -1 to +1
}
```

**Hi-hat**: White noise through a bandpass filter at 8kHz, very short (0.05–0.15 seconds). High frequency + brief duration = that metallic hiss.

**Clap**: Three rapid bursts of bandpass-filtered noise, offset by ~15ms. The layering creates a natural room-reverb feel.

### Scheduled Audio vs. Immediate Audio

Web Audio has its own high-precision clock: `ctx.currentTime`. You *schedule* sounds at specific times instead of playing them "now":

```javascript
osc.start(ctx.currentTime);         // play immediately
osc.start(ctx.currentTime + 0.5);   // play in 0.5 seconds
osc.start(ctx.currentTime + nextStepTime); // play at exact beat
```

**Why schedule instead of play immediately?** `setTimeout` has ~4ms jitter — it fires when the JavaScript engine gets to it, which varies based on what else is running. At 120 BPM, a 16th note is 125ms. A 4ms timing error is 3% off — noticeable to musicians.

Web Audio scheduling bypasses the JavaScript event loop entirely. Once scheduled, sounds play at the exact sample regardless of what JavaScript does next.

### The Lookahead Scheduler

The drum machine uses a classic pattern for precise sequencing:

```
Timeline:
  ctx.currentTime ─────────────────────────────────────────►
                  │← LOOKAHEAD (100ms) →│
                  ├─────────────────────┤
                  │  Steps to schedule  │
                  │  next tick (25ms)   │
                  ├──────┤

Every 25ms, the scheduler runs and pre-schedules any steps
falling within the next 100ms window.
```

```javascript
const LOOKAHEAD = 0.1;        // schedule 100ms ahead
const SCHEDULE_INTERVAL = 25; // check every 25ms

function scheduler() {
  const ctx = getCtx();
  // Schedule all steps in the next 100ms window
  while (nextStepTime < ctx.currentTime + LOOKAHEAD) {
    scheduleStep(currentStep, nextStepTime);
    nextStepTime += stepDuration();
    currentStep = (currentStep + 1) % STEPS;
  }
  schedulerTimer = setTimeout(scheduler, SCHEDULE_INTERVAL);
}
```

**Why the overlap?** If the scheduler checks every 25ms and schedules 100ms ahead, each step is scheduled at least twice before it plays. This redundancy ensures no steps are missed even if JavaScript is briefly busy (e.g., during DOM updates, garbage collection).

### Separating Audio Timing from Visual Timing

Audio is scheduled in the future. The UI highlight needs to follow *when that future arrives*:

```javascript
// When scheduling step S to play at time T:
const timeUntilMs = (stepTime - ctx.currentTime) * 1000;
setTimeout(() => highlightStep(stepIndex), Math.max(0, timeUntilMs));
```

The audio plays at the exact scheduled time. The visual highlight is triggered by a `setTimeout` set for the same moment. Audio scheduling and DOM updates are completely separate — they just happen to land at the same time.

### The Pattern as 2D Array

```
           Steps:  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16
  Kick:           [T][F][F][F][T][F][F][F][T][F][F][F][T][F][F][F]
  Snare:          [F][F][F][F][T][F][F][F][F][F][F][F][T][F][F][F]
  Hi-Hat:         [T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T]
```

Stored as a 2D array: `pattern[track][step]`

```javascript
let pattern = TRACKS.map(() => new Array(STEPS).fill(false));

// Toggle a step
step.addEventListener('click', () => {
  pattern[ti][s] = !pattern[ti][s];
  step.classList.toggle('on', pattern[ti][s]);
});

// In the scheduler, check if a track fires on this step
function scheduleStep(step, time) {
  for (let ti = 0; ti < TRACKS.length; ti++) {
    if (pattern[ti][step]) TRACKS[ti].play(time);
  }
}
```

Simple, clean. The entire musical state is a 2D boolean array.

---

## Guided Exercises

### Exercise 1: Mute and Solo Tracks

**The Challenge:** Add a mute button (🔇) to each track row. Muted tracks don't play even if their steps are on. Add a solo button (S) that mutes all other tracks, so only that one plays.

**Where to start:** You need to track which tracks are muted. What data structure holds this? Where does it get checked?

*(An array of booleans: `mutedTracks[ti]` is `true` if track `ti` is muted. It gets checked in `scheduleStep`.)*

---

**Step 1: Add the muted state.**

```javascript
let mutedTracks = new Array(TRACKS.length).fill(false);
```

---

**Step 2: Check muted in scheduleStep.**

Find `scheduleStep` and add one condition:

```javascript
function scheduleStep(step, time) {
  for (let ti = 0; ti < TRACKS.length; ti++) {
    if (!mutedTracks[ti] && pattern[ti][step]) {
      TRACKS[ti].play(time);
    }
  }
}
```

**Test it now:** Even without buttons, you can test by typing `mutedTracks[0] = true` in the console and pressing Play. The kick should go silent.

---

**Step 3: Add mute buttons to the HTML.**

You need to add a button to each track row. Find where rows are built (in the script that creates the sequencer grid) and add:

```javascript
const muteBtn = document.createElement('button');
muteBtn.textContent = '🔇';
muteBtn.title = 'Mute';
muteBtn.addEventListener('click', () => {
  mutedTracks[ti] = !mutedTracks[ti];
  muteBtn.style.opacity = mutedTracks[ti] ? '1' : '0.3';
  // Gray out the steps visually
  row.querySelectorAll('.step').forEach(s => {
    s.style.opacity = mutedTracks[ti] ? '0.4' : '1';
  });
});
row.insertBefore(muteBtn, row.firstChild);
```

---

**Step 4: Add solo.**

Solo is trickier — it needs to remember the previous mute state so it can restore when un-soloed:

```javascript
let previousMuteState = null;

const soloBtn = document.createElement('button');
soloBtn.textContent = 'S';
soloBtn.addEventListener('click', () => {
  if (previousMuteState !== null) {
    // Currently soloed — restore previous state
    mutedTracks = [...previousMuteState];
    previousMuteState = null;
    soloBtn.style.background = '';
  } else {
    // Solo this track — mute all others
    previousMuteState = [...mutedTracks];
    mutedTracks = mutedTracks.map((_, i) => i !== ti);  // all muted except ti
    soloBtn.style.background = '#FFD700';
  }
  // Sync visual state of mute buttons...
});
```

**Think about it:** The spread `[...mutedTracks]` creates a copy of the array. Why do you need a copy instead of `previousMuteState = mutedTracks`? *(Because `mutedTracks` is later reassigned. If you stored a reference, `previousMuteState` would point to the new array, not the saved state.)*

---

### Exercise 2: Save and Load Patterns

**The Challenge:** Add Save and Load buttons. Save stores the current pattern to `localStorage`. Load restores it. Add A/B slots so users can save two patterns and switch between them.

**Where to start:** The pattern is a 2D array of booleans. `JSON.stringify` can serialize it. `JSON.parse` can restore it.

---

**Step 1: Save the pattern.**

```javascript
function savePattern(slot) {
  localStorage.setItem(`beat-pattern-${slot}`, JSON.stringify(pattern));
}
```

---

**Step 2: Load the pattern.**

Loading is trickier — after loading the data, you also need to update the visual state of every step button:

```javascript
function loadPattern(slot) {
  const saved = localStorage.getItem(`beat-pattern-${slot}`);
  if (!saved) return alert('No pattern saved in this slot.');

  pattern = JSON.parse(saved);

  // Sync the step buttons with the loaded pattern
  document.querySelectorAll('.step').forEach((el, i) => {
    const ti = Math.floor(i / STEPS);
    const s  = i % STEPS;
    el.classList.toggle('on', pattern[ti][s]);
  });
}
```

**Think about it:** Why do you need to update the visual state? *(The step buttons have CSS classes reflecting the current pattern. Loading new data changes `pattern` but not the DOM. You must sync them.)*

---

**Step 3: Add A/B slot switching.**

```javascript
let activeSlot = 'A';

document.getElementById('slotA').addEventListener('click', () => loadPattern('A'));
document.getElementById('slotB').addEventListener('click', () => loadPattern('B'));
document.getElementById('saveA').addEventListener('click', () => savePattern('A'));
document.getElementById('saveB').addEventListener('click', () => savePattern('B'));
```

**The complete picture:** Serializing complex state (2D array → JSON string → localStorage → JSON parse → 2D array) is exactly how applications persist data. The same pattern works for game saves, editor preferences, and user settings.

---

### Exercise 3: Swing Feel

**The Challenge:** Add swing: odd-numbered steps are pushed slightly late, creating a shuffle rhythm. A slider controls the swing amount from 0% (straight) to 50% (triplet swing).

**Where to start:** Swing means some steps play "on time" and others play "late". The late amount is a fraction of a step duration.

---

**Step 1: Understand swing mathematically.**

In a straight rhythm, steps are equally spaced:
```
Step:   1  2  3  4  5  6  7  8
Time:   0 .5 1 1.5 2 2.5 3 3.5  (in beats)
```

With swing, odd steps (2, 4, 6, 8) are pushed late by `swingAmount * stepDuration`:
```
Step:   1    2    3    4    5    6    7    8
Time:   0   .6  1  1.6  2  2.6  3  3.6  (at 20% swing)
```

The effect: the beat "bounces" instead of marching straight.

---

**Step 2: Add the swing state.**

```javascript
let swingAmount = 0;  // 0.0 to 0.5
```

Add a range input in the HTML:
```html
<label>Swing: <input type="range" id="swingSlider" min="0" max="50" value="0"></label>
```

```javascript
document.getElementById('swingSlider').addEventListener('input', e => {
  swingAmount = parseInt(e.target.value) / 100;
});
```

---

**Step 3: Apply swing in the scheduler.**

When calculating `nextStepTime`, add the swing offset for odd steps:

```javascript
function scheduleStep(step, time) {
  // Odd steps get pushed late by swingAmount
  const swingOffset = (step % 2 === 1) ? swingAmount * stepDuration() : 0;
  const scheduledTime = time + swingOffset;

  for (let ti = 0; ti < TRACKS.length; ti++) {
    if (!mutedTracks[ti] && pattern[ti][step]) {
      TRACKS[ti].play(scheduledTime);
    }
  }
}
```

**Test it:** Set a simple kick pattern (steps 1, 5, 9, 13) and hi-hat on every step. Move the swing slider to 30–40%. The groove should shift from mechanical to human-feeling.

**Why does this work?** Professional drummers naturally push certain beats slightly late. The mathematical model (push every other 16th note) is a simplified but convincing approximation of human swing.

---

## What You Learned

| Concept | Where Used | Real-World Use |
|---------|-----------|----------------|
| `AudioContext` | Entry point to Web Audio | All Web Audio apps; lazy-init for gesture requirement |
| `OscillatorNode` | Kick, snare tone, pitch pads | Synthesizers, alert sounds, test tones |
| `GainNode` | Volume envelope on every sound | Mixing, fading, dynamics |
| `BiquadFilterNode` | Snare highpass, hi-hat bandpass | EQ, wah-wah, telephone effect |
| `AudioBuffer` + noise | Snare, hi-hat, clap bodies | Percussion, wind, texture synthesis |
| `exponentialRampToValueAtTime` | Pitch drop (kick), volume decay | All natural-sounding audio envelopes |
| `ctx.currentTime` | Audio scheduling clock | Precise music timing |
| Lookahead scheduler | 25ms check, 100ms window | Standard pattern in all Web Audio sequencers |
| 2D array data model | `pattern[track][step]` | Matrix data: image pixels, game boards, grids |

### Real-World Connections

- **Tone.js** is a Web Audio framework that wraps everything in this chapter with a musical API. `new Tone.Oscillator(440, 'sine').toDestination().start()` is the same as the kick function, simplified. Learning Web Audio directly means you understand what Tone.js is doing.
- **Digital Audio Workstations** (DAWs — Logic, Ableton, Pro Tools) are sophisticated versions of the drum machine you built. The same concepts: audio graphs, scheduling, step sequencers.
- **Game audio engines** (Howler.js, FMOD) use Web Audio under the hood for spatial audio, dynamic mixing, and precise timing.
- The **lookahead scheduler** pattern appears in other high-precision timing contexts: animation systems, real-time data streaming, and audio/video synchronization.

## Building with Claude

- "Add a reverb effect using a ConvolverNode with a programmatically generated impulse response — decaying white noise."
- "Add a pitch-shifting effect: Shift+click on a pad plays it at 1.5× or 0.75× pitch using BufferSource playbackRate."
- "Export the current 1-bar loop as a downloadable WAV file using OfflineAudioContext."
- "Add a volume knob per track using a GainNode in each track's signal chain."

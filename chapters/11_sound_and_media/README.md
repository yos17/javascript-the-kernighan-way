# Chapter 11: Sound and Media

Your browser is a synthesizer. We'll build a drum machine that generates every sound from scratch using the Web Audio API — no audio files, no external libraries, just math and oscillators.

---

## The Program: Beat Maker

Open `drumachine.html`. Click pads to hear sounds, toggle steps in the sequencer grid, set your BPM, and press Play to loop your beat. Keyboard shortcuts: `Q`–`I` for the top row of pads, `A`–`K` for the bottom row, `Space` to play/stop.

*(Full source is in `drumachine.html` — the README shows key excerpts below.)*

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

# CLAUDE.md — THE Ear Training Game

## Project Overview

**THE Ear Training Game** is a browser-based ear training application for chromatic pitch recognition. Users listen to a cadence that establishes a musical key, then identify mystery note(s) from a chromatic grid. The entire application is a **single HTML file** (`index.html`, ~900 lines) with no external dependencies, build system, or backend.

The app features an **ARTS-inspired adaptive learning algorithm** that tracks per-pitch accuracy, response time, and confusion pairs to optimize note selection — targeting weak spots while spacing out mastered material.

## Architecture

### Single-File Application

The entire app lives in `index.html` with three embedded sections:

| Section | Lines | Description |
|---------|-------|-------------|
| HTML/CSS | 1–187 | Document structure, all styles in a `<style>` tag |
| JavaScript | 193–902 | Audio engine, state, adaptive algorithm, game logic, persistence, and rendering in a `<script>` tag |

There is **no** `package.json`, build tool, bundler, framework, test suite, or linter.

### Code Organization (within `index.html`)

1. **Audio Engine** (lines 194–303) — Constants (`NOTE_NAMES`, `ENHARMONIC`, `DEGREE_LABELS`, `SOLFEGE`), configuration objects (`CADENCE_PATTERNS`, `INSTRUMENTS`, `TEMPO_SETTINGS`), Web Audio API synthesis (`noteToFreq`, `playNote`, `playChord`), and playback bus management (`stopPlayback`, `newPlaybackBus`).

2. **State** (lines 305–324) — A single global `state` object holding all application state (settings, game state, statistics, history, UI toggles).

3. **Adaptive Learning** (lines 326–475) — ARTS-inspired algorithm: `adaptive` state object with per-pitch stats (`recentResults`, `recentRTs`, confusion matrix), `computePriority()` (error boost, RT-modulated spacing, confusion boost, fluency dampening), `selectPitch()` (roulette wheel with enforced delay, starvation avoidance, previous-round dampening, combination deck boost), `updateAdaptiveStats()` (sliding windows, confusion matrix decay).

4. **Playback** (lines 477–578) — `playCadenceSeq`, `playMysterySeq`, `previewNote`, `startRound` (with RT clock), `replayAll`, `replayCadenceOnly`, `replayMysteryOnly`.

5. **Game Logic** (lines 580–627) — `handleReveal`, `isCorrect`, `guessButtonClass`, `requiredGuesses`.

6. **Settings** (line 630) — `setSetting(key, val)` updates state, saves, and re-renders.

7. **Persistence** (lines 632–722) — `saveProgress()` serializes to localStorage (version-stamped), `loadProgress()` validates and restores on init, `resetProgress()` clears everything with confirmation.

8. **Render** (lines 724–898) — A single `render()` function that rebuilds the entire DOM via `innerHTML`. Includes settings panel, pitch stats panel, stats row, game area, and history.

9. **Init** (lines 900–902) — `loadProgress()` then `render()`.

### State Management Pattern

```
User action → mutate global `state` object → call `render()` → full DOM rebuild
```

No virtual DOM, diffing, or reactive system. Every state change triggers a complete re-render of `#app`.

### Adaptive Algorithm

The pitch selection algorithm combines ideas from:
- **Kellman's ARTS** — Response time as a continuous learning strength signal, enforced minimum delay (D parameter)
- **Spaced repetition** — Trial-based spacing within sessions, modulated by RT
- **Interleaving** — Previous-round dampening, combination deck for exhaustive coverage
- **Confusion pair detection** — 12×12 confusion matrix with exponential decay (0.95/trial)

Key data structures per pitch:
- `recentResults[]` — Last 6 binary outcomes (correct/incorrect)
- `recentRTs[]` — Last 6 response times in ms (parallel to recentResults)
- `consecutiveCorrect` — Fluency counter
- `confusionMatrix[12][12]` — Tracks which pitches get confused with each other

Priority computation: error boost (recency-weighted) + spacing boost (RT-modulated coefficient) + confusion boost + coverage boost − fluency dampening.

### Audio Synthesis

- Uses **Web Audio API** (`AudioContext`, `OscillatorNode`, `GainNode`)
- Two synthesized instruments: Piano and Soft Synth — each with ADSR envelopes and optional harmonics or pad effects
- Playback bus pattern: all oscillators route through a shared `GainNode` for clean stop/replay
- Frequency calculation: `440 * 2^((midi - 69) / 12)`
- Cadence patterns play major triads in root position

## Technology Stack

- **Language:** Vanilla JavaScript (ES6+)
- **Styling:** Vanilla CSS (embedded `<style>` tag)
- **Audio:** Web Audio API
- **Storage:** localStorage (versioned, validated)
- **Frameworks/Libraries:** None
- **Build Tools:** None
- **Testing:** None (manual browser testing)

## Key Features

- Single-note or polyphonic (1–8 notes) pitch guessing
- Three cadence patterns: I-IV-V-I, I-IV-I-V-I, I-vi-IV-V-I
- Two instruments: Piano, Soft Synth
- Three tempo options (slow, medium, fast)
- Single octave (C4) or multi-octave (C2–C6) range
- Playback controls: replay all, cadence only, notes only
- Note preview on tap
- Adaptive pitch selection targeting weak spots
- Per-pitch stats panel (accuracy % + avg response time, color-coded)
- Response time display after each reveal
- Performance tracking: streak, best streak, accuracy %, rounds count
- Recent history (last 20 rounds, 8 shown)
- localStorage persistence across sessions with reset option

## Styling Conventions

- Dark theme with indigo/purple accent gradient (`#6366f1` → `#7c3aed`)
- Monospace font stack: SF Mono, Fira Code, JetBrains Mono, Menlo
- Mobile-first with `safe-area-inset` support for notched devices
- CSS Grid (4-column) for note buttons and pitch stats
- Feedback colors: green (`#4ade80`) correct, red (`#f87171`) wrong, amber (`#fbbf24`) missed
- Stats panel: HSL color interpolation (red 0% → yellow 50% → green 100%)
- Background grain texture via SVG filter

## Development Workflow

### Running the App

Open `index.html` directly in a browser. No server or build step required.

```sh
open index.html              # macOS
xdg-open index.html          # Linux
python3 -m http.server 8000  # Serve locally if needed for Audio API
```

> **Note:** Some browsers require an HTTP server (not `file://`) for Web Audio API. Use a local dev server if audio doesn't play.

### Making Changes

1. Edit `index.html` directly — all code is in one file
2. Refresh the browser to see changes
3. No compilation, transpilation, or bundling step

## Conventions for AI Assistants

### Do

- Keep everything in `index.html` unless there's a strong reason to extract files
- Follow the existing code organization sections (Audio → State → Adaptive → Playback → Logic → Settings → Persistence → Render → Init)
- Mutate `state` directly and call `render()` to update the UI
- Use template literals for HTML generation in `render()`
- Use inline `onclick` handlers consistent with the existing pattern
- Match the existing CSS naming conventions (kebab-case class names)
- Call `saveProgress()` after any state mutation that should persist
- Test changes by opening in a browser — there is no automated test suite

### Don't

- Don't add npm, a bundler, or a framework unless explicitly requested
- Don't split into multiple files unless explicitly requested
- Don't add TypeScript, JSX, or other transpiled syntax
- Don't introduce external dependencies (CDN scripts, npm packages)
- Don't restructure the render function into components unless explicitly requested

### When Adding New Features

- Add new constants/config near the top with existing constants
- Add new state properties to the `state` object (lines 306–324)
- Add new adaptive fields to the `adaptive` object (lines 329–345) and update `saveProgress`/`loadProgress`/`resetProgress`
- Add new game logic functions between Playback and Settings sections
- Add new UI in the `render()` function using the existing template literal pattern
- Add new CSS in the `<style>` tag following the existing section comments

## Deployment

Hosted on **GitHub Pages**. The application is a static single HTML file — push to `main` and GitHub Pages deploys automatically. No build artifacts or asset pipeline.

## File Map

```
THE-ear-training-game/
├── CLAUDE.md                      ← This file
├── EarTrainingGameThumbnail.png   ← App icon (favicon + apple-touch-icon)
└── index.html                     ← Entire application (HTML + CSS + JS, ~900 lines)
```

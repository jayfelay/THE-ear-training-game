# CLAUDE.md — THE Ear Training Game

## Project Purpose

THE Ear Training Game is a research-informed ear training tool exploring how adaptive perceptual learning techniques can accelerate chromatic pitch recognition. It started as a personal tool — existing ear training apps don't adapt to individual weaknesses — and evolved into an implementation of ideas from Kellman's ARTS algorithm, spaced repetition, and confusion pair analysis.

The north star is **algorithm fidelity**: getting the adaptive engine as close as possible to a faithful implementation of ARTS principles, where response time drives spacing decisions and the system genuinely targets the learner's weak spots. Development is iterative — features get added, tested through real use, and stripped back if they add complexity without serving the learning experience.

See [README.md](README.md) for the full project story, research foundations, and feature list.

## Architecture

### Single-File Application

The entire app lives in `index.html` (~1300 lines) with embedded CSS and JavaScript:

| Section | Lines | Description |
|---------|-------|-------------|
| HTML/CSS | 1–217 | Document structure, all styles in a `<style>` tag |
| JavaScript | 223–1299 | Audio engine, state, adaptive algorithm, game logic, persistence, and rendering in a `<script>` tag |

There is **no** `package.json`, build tool, bundler, framework, test suite, or linter.

### Code Organization (within `index.html`)

The JavaScript is divided into sections marked with `// ─── Section Name ───` comment dividers:

1. **Audio Engine** (`// ─── Audio Engine ───`) — Constants (`NOTE_NAMES`, `ENHARMONIC`, `DEGREE_LABELS`, `SOLFEGE`, `INTERVAL_NAMES`), configuration objects (`CADENCE_PATTERNS`, `INSTRUMENTS`, `TEMPO_SETTINGS`), Web Audio API synthesis (`noteToFreq`, `playNote`, `playChord`), and playback bus management (`stopPlayback`, `newPlaybackBus`).

2. **State** (`// ─── State ───`) — A single global `state` object holding all application state: settings, game state, statistics, history, UI toggles, and drone mode state.

3. **Adaptive Learning** (`// ─── Adaptive Learning (ARTS-inspired) ───`) — The `adaptive` object with per-pitch stats, confusion matrix, interval stats, and error type counters. Contains the core algorithm functions: `computePriority()`, `selectPitch()`, `updateAdaptiveStats()`. Also includes interval analysis helpers (`semitoneDist`, `intervalName`, `nearestAnchor`, `classifyError`, `checkIntervalPreserved`, `attributeConfusions`) and combination deck management (`buildCombinationDeck`, `comboKey`).

4. **Playback** (follows Adaptive Learning) — `playCadenceSeq`, `playMysterySeq`, `previewNote`, `startRound` (with RT clock), `replayAll`, `replayCadenceOnly`, `replayMysteryOnly`.

5. **Drone Mode Audio** (`// ─── Drone Mode Audio ───`) — Drone oscillator management (`startDrone`, `stopDrone`), sustained note toggling (`startDroneNote`, `stopDroneNote`), mode entry/exit (`enterDroneMode`, `exitDroneMode`).

6. **Game Logic** (follows Drone Mode) — `toggleGuess`, `handleReveal`, `isCorrect`, `guessButtonClass`, `requiredGuesses`.

7. **Settings Helpers** (`// ─── Settings Helpers ───`) — `setSetting(key, val)` updates state, saves, and re-renders.

8. **Persistence** (`// ─── Persistence ───`) — `saveProgress()` serializes to localStorage (version-stamped), `loadProgress()` validates and restores on init, `resetProgress()` clears everything with confirmation.

9. **Render** (`// ─── Render ───`) — A single `render()` function that rebuilds the entire DOM via `innerHTML`. Includes settings panel, pitch stats panel, stats row, game/drone mode area, and history.

10. **Init** (follows Render) — `loadProgress()` then `render()`.

### State Management Pattern

```
User action → mutate global `state` object → call `render()` → full DOM rebuild
```

No virtual DOM, diffing, or reactive system. Every state change triggers a complete re-render of `#app`.

### Adaptive Algorithm

The pitch selection algorithm combines ideas from:
- **Kellman's ARTS** — Response time as a continuous learning strength signal, enforced minimum delay (D parameter, typically 3–5 trials)
- **Spaced repetition** — Trial-based spacing within sessions, RT-modulated coefficients (fast correct → longer spacing, slow correct → shorter spacing)
- **Interleaving** — Previous-round dampening, combination deck for exhaustive pitch-pair coverage
- **Confusion pair detection** — 12x12 confusion matrix with exponential decay (0.95/trial), attributed via greedy nearest-pitch matching
- **Interval-aware error analysis** — Error classification (neighbor, consonance-relative, interval-preserved, distant) with per-interval accuracy tracking for polyphonic mode

Key data structures per pitch:
- `recentResults[]` — Last 6 binary outcomes (correct/incorrect)
- `recentRTs[]` — Last 6 response times in ms (parallel to recentResults)
- `consecutiveCorrect` — Fluency counter
- `confusionMatrix[12][12]` — Tracks which pitches get confused with each other
- `intervalStats[12]` — Per-interval accuracy and preservation tracking (polyphonic mode)
- `errorTypeCounters` — Counts of neighbor, consonance-relative, interval-preserved, and distant errors

Priority computation: error boost (recency-weighted) + spacing boost (RT-modulated coefficient) + confusion boost + neighbor confusion boost + coverage boost − fluency dampening. In polyphonic mode, interval-difficulty-aware selection biases toward weak intervals.

### Audio Synthesis

- Uses **Web Audio API** (`AudioContext`, `OscillatorNode`, `GainNode`)
- Two synthesized instruments: Piano (triangle wave + harmonics) and Soft Synth (sine wave + pad effect) — each with ADSR envelopes
- Playback bus pattern: all oscillators route through a shared `GainNode` for clean stop/replay
- Drone mode: sustained tonic with detuned second oscillator for warmth, plus individual sustained notes via triangle+sine oscillator pairs
- Frequency calculation: `440 * 2^((midi - 69) / 12)`

## Technology Stack

- **Language:** Vanilla JavaScript (ES6+)
- **Styling:** Vanilla CSS (embedded `<style>` tag)
- **Audio:** Web Audio API
- **Storage:** localStorage (versioned, validated)
- **Frameworks/Libraries:** None
- **Build Tools:** None
- **Testing:** None (manual browser testing)

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

### Project Philosophy

This project favors incremental refinement over feature accumulation. Features that add UI complexity without clear learning value should be held back. When in doubt, add algorithmic intelligence under the hood rather than surface-level UI. The most recent development pass stripped several premature UI additions — the bar for adding visible complexity is high.

### Do

- Keep everything in `index.html` unless there's a strong reason to extract files
- Follow the existing code organization sections (use the `// ─── Section Name ───` markers as guides)
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

- Add new constants/config near the top with existing constants in the Audio Engine section
- Add new state properties to the `state` object in the State section
- Add new adaptive data structures to the `adaptive` object and update `saveProgress`/`loadProgress`/`resetProgress` to handle them. Add new priority factors in `computePriority()`. Add new per-trial tracking in `updateAdaptiveStats()`
- Add drone-related functions in the Drone Mode Audio section
- Add new game logic functions between the Drone Mode Audio and Settings Helpers sections
- Add new UI in the `render()` function using the existing template literal pattern
- Add new CSS in the `<style>` tag following the existing section comments

## Deployment

Hosted on **GitHub Pages**. The application is a static single HTML file — push to `main` and GitHub Pages deploys automatically. No build artifacts or asset pipeline.

## File Map

```
THE-ear-training-game/
├── CLAUDE.md                      ← This file (AI assistant reference)
├── EarTrainingGameThumbnail.png   ← App icon (favicon + apple-touch-icon)
├── README.md                      ← Project overview, research context, features
└── index.html                     ← Entire application (HTML + CSS + JS, ~1300 lines)
```

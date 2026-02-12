# CLAUDE.md — THE Ear Training Game

## Project Overview

**THE Ear Training Game** is a browser-based ear training application for chromatic pitch recognition. Users listen to a cadence that establishes a musical key, then identify mystery note(s) from a chromatic grid. The entire application is a **single HTML file** (`index.html`, ~550 lines) with no external dependencies, build system, or backend.

## Architecture

### Single-File Application

The entire app lives in `index.html` with three embedded sections:

| Section | Lines | Description |
|---------|-------|-------------|
| HTML/CSS | 1–176 | Document structure and all styles in a `<style>` tag |
| JavaScript | 178–549 | Audio engine, state, game logic, and rendering in a `<script>` tag |

There is **no** `package.json`, build tool, bundler, framework, test suite, or linter.

### Code Organization (within `index.html`)

The JavaScript is organized into clearly labeled sections:

1. **Audio Engine** (lines 179–272) — Constants (`NOTE_NAMES`, `ENHARMONIC`, `DEGREE_LABELS`, `SOLFEGE`), configuration objects (`CADENCE_PATTERNS`, `INSTRUMENTS`, `TEMPO_SETTINGS`), and functions for Web Audio API synthesis (`noteToFreq`, `playNote`, `playChord`, `getNoteDisplay`).

2. **State** (lines 274–291) — A single global `state` object holding all application state (settings, game state, statistics, history). State is mutated directly and the UI re-renders by calling `render()`.

3. **Game Logic** (lines 293–408) — Functions for gameplay: `playCadenceSeq`, `playMysterySeq`, `previewNote`, `startRound`, `replayAll`, `toggleGuess`, `handleReveal`, `isCorrect`, `guessButtonClass`, `requiredGuesses`.

4. **Settings Helpers** (line 410–411) — `setSetting(key, val)` updates state and re-renders.

5. **Render** (lines 413–546) — A single `render()` function that rebuilds the entire DOM via `innerHTML`. Uses template literals to compose HTML with inline `onclick` handlers.

6. **Init** (lines 548–549) — Calls `render()` on load.

### State Management Pattern

```
User action → mutate global `state` object → call `render()` → full DOM rebuild
```

There is no virtual DOM, diffing, or reactive system. Every state change triggers a complete re-render of `#app`.

### Audio Synthesis

- Uses **Web Audio API** (`AudioContext`, `OscillatorNode`, `GainNode`)
- Three synthesized instruments: Piano, E-Piano, Soft Synth — each with ADSR envelopes and optional harmonics, tremolo, or pad effects
- Frequency calculation: `440 * 2^((midi - 69) / 12)`
- Cadence patterns play major triads in root position

## Technology Stack

- **Language:** Vanilla JavaScript (ES6+)
- **Styling:** Vanilla CSS (embedded `<style>` tag)
- **Audio:** Web Audio API
- **Frameworks/Libraries:** None
- **Build Tools:** None
- **Testing:** None
- **Linting/Formatting:** None

## Key Features

- Single-note or polyphonic (1–4 notes) pitch guessing
- Three cadence patterns: I-IV-V-I, I-IV-I-V-I, I-vi-IV-V-I
- Three instruments with distinct timbres
- Three tempo options (slow, medium, fast)
- Single octave (C4) or multi-octave (C2–C6) range
- Playback controls: replay all, cadence only, notes only
- Note preview on tap
- Performance tracking: streak, best streak, accuracy percentage
- Recent history (last 20 rounds, 8 shown)

## Styling Conventions

- Dark theme with indigo/purple accent gradient (`#6366f1` → `#8b5cf6`)
- Monospace font stack: SF Mono, Fira Code, JetBrains Mono, Menlo
- Mobile-first with `safe-area-inset` support for notched devices
- CSS Grid (4-column) for note buttons
- Feedback colors: green (`#4ade80`) correct, red (`#f87171`) wrong, amber (`#fbbf24`) missed
- Background grain texture via SVG filter

## Development Workflow

### Running the App

Open `index.html` directly in a browser. No server or build step required.

```sh
# Any of these work:
open index.html              # macOS
xdg-open index.html          # Linux
python3 -m http.server 8000  # Serve locally if needed for Audio API
```

> **Note:** Some browsers require an HTTP server (not `file://`) for Web Audio API. Use a local dev server if audio doesn't play.

### Making Changes

1. Edit `index.html` directly — all code is in one file
2. Refresh the browser to see changes
3. No compilation, transpilation, or bundling step

### No Tests or Linting

There are no automated tests, linters, or formatters configured. Manual browser testing is the only verification method.

## Conventions for AI Assistants

### Do

- Keep everything in `index.html` unless there's a strong reason to extract files
- Follow the existing code organization sections (Audio Engine → State → Game Logic → Settings → Render)
- Mutate `state` directly and call `render()` to update the UI
- Use template literals for HTML generation in `render()`
- Use inline `onclick` handlers consistent with the existing pattern
- Match the existing CSS naming conventions (kebab-case class names)
- Test changes by opening in a browser — there is no automated test suite

### Don't

- Don't add npm, a bundler, or a framework unless explicitly requested
- Don't split into multiple files unless explicitly requested
- Don't add TypeScript, JSX, or other transpiled syntax
- Don't introduce external dependencies (CDN scripts, npm packages)
- Don't add `localStorage` or persistence unless explicitly requested — stats reset on reload by design
- Don't restructure the render function into components unless explicitly requested

### When Adding New Features

- Add new constants/config near the top with existing constants
- Add new state properties to the `state` object (lines 275–291)
- Add new game logic functions between the State and Render sections
- Add new UI in the `render()` function using the existing template literal pattern
- Add new CSS in the `<style>` tag following the existing section comments

## Deployment

The application is a static single HTML file. Deploy by serving `index.html` from any web server or static hosting platform (GitHub Pages, Netlify, Vercel, S3, etc.). No build artifacts or asset pipeline.

## File Map

```
THE-ear-training-game/
├── CLAUDE.md       ← This file
└── index.html      ← Entire application (HTML + CSS + JS)
```

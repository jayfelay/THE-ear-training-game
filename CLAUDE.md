# CLAUDE.md — THE Ear Training Game

## Project Purpose

THE Ear Training Game is a research-informed ear training tool exploring how adaptive perceptual learning techniques can accelerate chromatic pitch recognition. It started as a personal tool — existing ear training apps don't adapt to individual weaknesses — and evolved into an implementation of ideas from Kellman's ARTS algorithm, spaced repetition, and confusion pair analysis.

The north star is **algorithm fidelity**: getting the adaptive engine as close as possible to a faithful implementation of ARTS principles, where response time drives spacing decisions and the system genuinely targets the learner's weak spots. Development is iterative — features get added, tested through real use, and stripped back if they add complexity without serving the learning experience.

The app is publicly shipped at https://jayfelay.github.io/THE-ear-training-game/. The technical surface is considered complete; the codebase has been through a soundness pass and a polish pass (all 12 fixes from both passes are merged into `main`). Ongoing work should be cautious, surgical, and prefer algorithmic refinement over UI accretion.

See [README.md](README.md) for the public-facing project overview.

## License Posture

**This project is NOT open source.** The repository is publicly visible for transparency, but `LICENSE` is an all-rights-reserved proprietary license. Do not suggest open-sourcing changes, propose MIT/Apache/etc. relicensing, recommend a `CONTRIBUTING.md`, or frame the project as community-facing. There is no contributor pipeline. Code, algorithm, and design are the proprietary work of John Felice ("Jayfelay").

## Architecture

### Single-File Application

The entire app lives in `index.html` (~1400 lines) with embedded CSS and JavaScript:

| Section | Lines | Description |
|---------|-------|-------------|
| HTML/CSS | 1–225 | Document structure, all styles in a `<style>` tag |
| Body | 226–230 | `#app` mount point and hidden `<audio id="silent-audio">` for iOS silent-mode unlock |
| JavaScript | 231–1398 | Audio engine, state, adaptive algorithm, game logic, persistence, and rendering in a `<script>` tag |

There is **no** `package.json`, build tool, bundler, framework, test suite, or linter.

### Code Organization (within `index.html`)

The JavaScript is divided into sections marked with `// ─── Section Name ───` comment dividers:

1. **Audio Engine** (line 232) — Constants (`NOTE_NAMES`, `ENHARMONIC`, `DEGREE_LABELS`, `SOLFEGE`), configuration objects (`CADENCE_PATTERNS`, `INSTRUMENTS`, `TEMPO_SETTINGS`), Web Audio API synthesis (`noteToFreq`, `playNote`, `playChord`), and playback bus management (`stopPlayback`, `newPlaybackBus`). Cadence chords respect diatonic quality (vi = minor). Includes `ensureCtx()` with the `silentAudioUnlocked` one-shot flag — see iOS Audio Constraints below.

2. **State** (line 350) — A single global `state` object holding all application state: settings, game state, statistics, history, UI toggles, and drone mode state.

3. **Adaptive Learning** (line 374, `// ─── Adaptive Learning (ARTS-inspired) ───`) — The `adaptive` object with per-pitch stats, confusion matrix, and interval stats. Contains the core algorithm functions: `computePriority()`, `selectPitch()`, `updateAdaptiveStats()`. Also includes interval analysis helpers (`semitoneDist`, `checkIntervalPreserved`, `attributeConfusions`) and combination deck management (`buildCombinationDeck`, `comboKey`). `adaptive.trialNumber` is the global trial counter and is the canonical "first-time user" signal — see Onboarding below.

4. **Playback** (follows Adaptive Learning) — `playCadenceSeq`, `playMysterySeq`, `previewNote`, `startRound` (with RT clock), `replayAll`, `replayCadenceOnly`, `replayMysteryOnly`.

5. **Drone Mode Audio** (line 804) — Drone oscillator management (`startDrone`, `stopDrone`), sustained note toggling (`startDroneNote`, `stopDroneNote`), mode entry/exit (`enterDroneMode`, `exitDroneMode`).

6. **Game Logic** (follows Drone Mode) — `toggleGuess`, `handleReveal`, `isCorrect`, `guessButtonClass`, `requiredGuesses`.

7. **Settings Helpers** (line 1017) — `setSetting(key, val)` updates state, saves, and re-renders.

8. **Persistence** (line 1020) — `saveProgress()` serializes to localStorage (version-stamped), `loadProgress()` validates and restores on init, `resetProgress()` clears everything with confirmation.

9. **Render** (line 1162) — A single `render()` function that rebuilds the entire DOM via `innerHTML`. Includes settings panel, pitch stats panel, stats row, game/drone mode area, and history. The onboarding pitch and the "No sound? Check your phone's silent switch." helper text live here.

10. **Init** (end of script) — `loadProgress()` then `render()`.

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
- **Confusion pair detection** — 12x12 confusion matrix with exponential decay (0.95/trial), attributed via greedy nearest-pitch matching. Anchor-slip-aware: when intervals are preserved but placement is wrong (the whole frame shifted, not individual notes), confusion matrix writes at half-weight to avoid over-drilling notes that weren't the real problem
- **Interval-aware error analysis** — Per-interval accuracy and anchor-slip tracking for polyphonic mode. Slip rate (preserved/presentations) feeds back into interval weakness scoring, so intervals that repeatedly override the root reference get drilled across different positions

Key data structures per pitch:
- `recentResults[]` — Last 6 binary outcomes (correct/incorrect)
- `recentRTs[]` — Last 6 response times in ms (parallel to recentResults)
- `consecutiveCorrect` — Fluency counter
- `confusionMatrix[12][12]` — Tracks which pitches get confused with each other
- `intervalStats[12]` — Per-interval accuracy and preservation tracking (polyphonic mode)

Priority computation: error boost (recency-weighted) + spacing boost (RT-modulated coefficient) + confusion boost + neighbor confusion boost + coverage boost − fluency dampening. In polyphonic mode, interval-difficulty-aware selection biases toward weak intervals using both accuracy and anchor-slip rate, with the interval boost cap scaling with polyphony level (3.0 at level 1, up to 6.5 at level 8) to reflect that intervals become the dominant challenge as notes increase.

### Polyphony Scope

- **Algorithm internals support 1–8** simultaneous mystery notes. The combination deck, interval-difficulty scaling, and anchor-slip detection are all implemented for the full 1–8 range.
- **UI exposes 1–3 only.** The polyphony selector is constrained to 1–3 in the settings panel because validation work has focused on those levels. Levels 4–8 are reachable from code but considered unvalidated for end users — do not surface them in the UI without explicit instruction.

### Cold-Start Onboarding

First-time users see a dedicated onboarding screen with a personal pitch from Jayfelay and a 3-step "How a round works" explainer.

- **Gate:** `adaptive.trialNumber === 0` inside `render()`'s `!state.gameActive` branch.
- **Replaced by:** A short "Listen to the cadence..." instruction once `trialNumber > 0`.
- **Resetting:** `resetProgress()` zeros `adaptive.trialNumber`, so a hard reset returns the user to onboarding. This is intentional.
- **Voice:** First-person, conversational, no marketing-speak. Any future onboarding copy edits should preserve this register — it's the project's voice anchor.

### iOS Audio Constraints

Two iOS Safari quirks are handled in the codebase:

1. **Interrupted state.** `ensureCtx()` resumes on both `"suspended"` and `"interrupted"` — iOS Safari moves the context to `"interrupted"` after phone calls, Siri, etc. Without the `"interrupted"` branch, audio silently dies after any interruption.

2. **Silent-mode unlock.** A hidden `<audio id="silent-audio" playsinline>` with a base64 silent MP3 lives in the body. On the first `ensureCtx()` call after a user gesture, a one-shot `silentAudioUnlocked` boolean flips and the element's `.play()` is invoked (with a `.catch(() => {})` to swallow autoplay rejections). This pairs the Web Audio context with an HTML5 media element, which signals to iOS Safari that the audio context is media playback rather than system sound — bypassing the physical silent switch.

   **Known limitation:** This is a best-effort unlock, not a full override. It works for the common case (silent switch on, system volume up) but cannot defeat all silent-mode scenarios on modern iOS (some Bluetooth routing edge cases, hardware mute states, etc.). The settings panel includes a small italic helper line — "No sound? Check your phone's silent switch." — as a long-tail safety net for users hitting cases the unlock can't cover. Do not remove that helper text without explicit instruction.

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

This project favors incremental refinement over feature accumulation. The codebase is post-soundness-pass and post-polish-pass; the bar for adding visible complexity is high. Features that add UI complexity without clear learning value should be held back. When in doubt, add algorithmic intelligence under the hood rather than surface-level UI.

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
- Don't expose polyphony levels 4–8 in the UI without explicit instruction
- Don't remove or weaken the iOS silent-mode unlock or its helper text
- Don't propose open-sourcing, MIT relicensing, contributor docs, or community-facing framing

### When Adding New Features

- Add new constants/config near the top with existing constants in the Audio Engine section
- Add new state properties to the `state` object in the State section
- Add new adaptive data structures to the `adaptive` object and update `saveProgress`/`loadProgress`/`resetProgress` to handle them. Add new priority factors in `computePriority()`. Add new per-trial tracking in `updateAdaptiveStats()`
- Add drone-related functions in the Drone Mode Audio section
- Add new game logic functions between the Drone Mode Audio and Settings Helpers sections
- Add new UI in the `render()` function using the existing template literal pattern
- Add new CSS in the `<style>` tag following the existing section comments

## Deployment

Hosted on **GitHub Pages** at https://jayfelay.github.io/THE-ear-training-game/. The application is a static single HTML file — push to `main` and GitHub Pages deploys automatically. No build artifacts or asset pipeline.

## File Map

```
THE-ear-training-game/
├── CLAUDE.md                      ← This file (AI assistant reference)
├── EarTrainingGameThumbnail.png   ← App icon (favicon + apple-touch-icon)
├── LICENSE                        ← All-rights-reserved proprietary license
├── README.md                      ← Public-facing project overview
└── index.html                     ← Entire application (HTML + CSS + JS, ~1400 lines)
```

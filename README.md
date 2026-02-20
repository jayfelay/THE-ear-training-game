<p align="center">
  <img src="EarTrainingGameThumbnail.png" width="200" alt="THE Ear Training Game" />
</p>

<h1 align="center">THE Ear Training Game</h1>

<p align="center">
  Adaptive chromatic pitch recognition training, powered by perceptual learning research.<br>
  <a href="https://jayfelay.github.io/THE-ear-training-game/">Play it now</a>
</p>

---

## What is this?

Most ear training apps present notes in fixed sequences or random order. They don't know which pitches you actually struggle with, how confident your correct answers really are, or which notes you keep confusing with each other. Progress feels slow because practice isn't targeted.

THE Ear Training Game was built to fix that. It started as a personal tool — a way to train chromatic pitch recognition that actually adapts to how I'm doing. The core idea: if the app tracks not just *whether* you got a note right, but *how quickly* you identified it and *what you confused it with*, it can make much smarter decisions about what to present next.

That led to implementing ideas from perceptual learning research — specifically Philip Kellman's ARTS algorithm, which uses response time as a continuous signal of learning strength. The result is an ear training game that targets your weak spots, spaces out your strong ones, and gives you real feedback on where you stand with each of the 12 chromatic pitches.

**The loop:** A cadence establishes a key. Mystery note(s) play. You identify them from a chromatic grid. The algorithm updates your profile and selects the next challenge accordingly.

## The Research

The adaptive engine draws on several lines of research in perceptual and adaptive learning.

**ARTS (Adaptive Response-Time-based Sequencing)** — Developed by Philip Kellman and colleagues at UCLA for perceptual learning tasks. The key insight: response time is a more sensitive measure of learning strength than accuracy alone. Getting a pitch right in 1.5 seconds means something fundamentally different from getting it right in 8 seconds. ARTS uses response time to modulate spacing — fast correct responses earn longer rest periods before that item reappears, while slow correct responses get re-presented sooner. A minimum delay (D parameter, typically 3–5 trials) prevents items from repeating too soon, avoiding working-memory-based false fluency.

**Confusion pair tracking** — A 12x12 confusion matrix records which pitches the player mistakes for which. If you consistently call Db when the answer is Ab, the algorithm notices and presents both pitches more frequently. The matrix decays exponentially (0.95 per trial) so resolved confusions naturally fade.

**Interleaving and coverage** — The algorithm ensures all 12 pitches get presented before any repeat in a coverage cycle, dampens pitches from the previous round to force interleaving, and (in polyphonic mode) uses a combination deck to systematically cover pitch pairs.

**Anchor slip and interval-aware error analysis** — In multi-note mode, the biggest failure mode isn't mishearing individual notes — it's losing the root. When a competing interval (like a perfect fourth) overwrites the tonal center, the ear reassigns the root and all identifications shift together. The algorithm detects this: if the guessed notes preserve the correct intervals but sit on the wrong position, it's an anchor slip, not a note confusion. The response is different too — rather than re-drilling individual notes, the algorithm targets that interval type across different positions to build robust root retention.

The goal isn't to replicate ARTS exactly — it's a browser game, not a lab study. But the core principles (RT as a learning signal, adaptive spacing, confusion-aware scheduling) provide a much stronger foundation than random or fixed presentation.

## Features

**Training modes**
- **Game mode** — Cadence-based pitch recognition with full adaptive learning
- **Free Play** — Sustained tonic drone with a clickable chromatic grid for meditative interval exploration

**Customization**
- 1–8 simultaneous mystery notes (polyphonic challenges)
- Three cadence patterns: I–IV–V–I, I–IV–I–V–I, I–vi–IV–V–I
- Two synthesized instruments: Piano and Soft Synth (Web Audio API, no samples)
- Three tempo settings (slow, medium, fast)
- Single octave or multi-octave range

**Feedback and tracking**
- Per-pitch accuracy and response time, color-coded from red (struggling) to green (fluent)
- Streak tracking, overall accuracy %, total rounds played
- Session history with scale degree labels and response times
- Tap any note to preview its sound before committing
- All progress persists to localStorage across sessions

## Status

This is an active personal project exploring how perceptual learning research can make ear training more effective. The core game loop, adaptive algorithm, and session persistence are solid and usable. Development is iterative — features get added, tested through real use, and refined or stripped back based on whether they genuinely serve the learning experience.

The current focus is algorithm fidelity: getting the adaptive engine closer to a faithful implementation of ARTS principles, tuning the priority computation, and ensuring the spacing and selection logic behaves the way the research suggests it should.

## Running Locally

It's a single HTML file. Open it in a browser:

```sh
open index.html            # macOS
xdg-open index.html        # Linux
start index.html           # Windows
```

Some browsers need an HTTP server for Web Audio API to work:

```sh
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Tech

Zero dependencies. No build step. The entire application is a single HTML file (~1300 lines) with embedded CSS and JavaScript, synthesizing all audio via the Web Audio API. Hosted on GitHub Pages.

| File | What |
|------|------|
| `index.html` | The entire app — HTML, CSS, and JS |
| `EarTrainingGameThumbnail.png` | App icon |

## License

MIT

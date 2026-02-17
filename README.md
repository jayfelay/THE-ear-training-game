<p align="center">
  <img src="EarTrainingGameThumbnail.png" width="200" alt="THE Ear Training Game" />
</p>

<h1 align="center">THE Ear Training Game</h1>

<p align="center">
  A browser-based chromatic pitch recognition trainer with adaptive learning.<br>
  <a href="https://jayfelay.github.io/THE-ear-training-game/">Play it now</a>
</p>

---

Listen to a cadence that locks in a key, then identify the mystery note(s) from a chromatic grid. An adaptive algorithm tracks your accuracy, response time, and confusion pairs to keep targeting your weak spots.

## Features

- **Adaptive learning** — ARTS-inspired algorithm prioritizes pitches you struggle with and spaces out ones you've mastered
- **Drone mode** — meditative interval exploration with a sustained tonic and clickable chromatic note grid
- **Polyphonic challenges** — guess 1 to 8 simultaneous mystery notes
- **Multiple cadence patterns** — I–IV–V–I, I–IV–I–V–I, I–vi–IV–V–I
- **Two instruments** — Piano and Soft Synth, all synthesized via Web Audio API
- **Tempo & range options** — slow/medium/fast, single octave or multi-octave (C2–C6)
- **Replay controls** — replay all, cadence only, or mystery notes only
- **Detailed stats** — per-pitch accuracy and response time, streak tracking, color-coded stats panel
- **Session history** — last 20 rounds with degree labels and response times
- **Offline-capable** — everything runs client-side, progress saved to localStorage

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

Zero dependencies. No build step. One file:

| File | What |
|------|------|
| `index.html` | The entire app — HTML, CSS, and JS |
| `EarTrainingGameThumbnail.png` | App icon |

Built with vanilla JavaScript, CSS, and the Web Audio API. Hosted on GitHub Pages.

## License

MIT

<p align="center">
  <img src="EarTrainingGameThumbnail.png" width="200" alt="THE Ear Training Game" />
</p>

<h1 align="center">THE Ear Training Game</h1>

<p align="center">
  Your personalized path to relative pitch.<br>
  <a href="https://jayfelay.github.io/THE-ear-training-game/">Play it now →</a>
</p>

---

## What it is

Honestly... most ear training apps suck. They make you guess what to practice. They feel like homework. Confusing. Slow. Not fun. You can't tell if you're getting better.

I made this because it's what I wish I had when I was training my own ears. It's simple. Think of it less as an app and more as a built-in teacher — one that knows your specific weak spots and serves exactly the notes your ear needs. It watches what you miss, what you confuse, and how fast you respond. It knows exactly what to throw at you next.

A cadence plays to set the key. A mystery note (or two, or three) plays. You tap what you heard. The game updates your profile and picks the next challenge.

It's hard. It's fun. It works.

## How it works

The selection algorithm is loosely modeled on Philip Kellman's ARTS (Adaptive Response-Time-based Sequencing). Response time is treated as a continuous signal of learning strength — fast correct answers earn longer spacing before that pitch returns, slow correct answers come back sooner. A 12x12 confusion matrix tracks which pitches you mistake for which and biases selection toward unresolved pairs. In polyphonic mode, an anchor-slip detector notices when you've kept the right intervals but lost the root, and drills that interval across different positions instead of re-grilling notes that weren't really the problem.

## Why I made it

I'm a guitarist and music educator. Berklee training, years of doing ear training the hard way, and a lot of curiosity about what learning science from the last twenty years can do for it. I kept hitting the same wall with existing tools — they didn't know me. Every session felt like starting over.

So I built the teacher I wanted. It does the learning for you. You just play.

— John "Jayfelay" Felice

## Tech

Single HTML file (~1400 lines) with embedded CSS and JavaScript. No build step, no dependencies, no framework. All audio is synthesized live via the Web Audio API. Progress lives in `localStorage` and never leaves your browser. Hosted on GitHub Pages.

```sh
open index.html              # macOS
xdg-open index.html          # Linux
python3 -m http.server 8000  # if your browser blocks file:// audio
```

| File | What |
|------|------|
| `index.html` | The entire app |
| `EarTrainingGameThumbnail.png` | App icon |
| `LICENSE` | Terms of use (all rights reserved) |
| `CLAUDE.md` | AI assistant reference |

## Contact

- Email: john@johnfelicemusic.com
- Socials: [@jayfelay](https://instagram.com/jayfelay)

## License

All rights reserved. See [LICENSE](LICENSE) for the full terms — source is publicly visible for transparency, but reuse, redistribution, derivative works, and commercial use are not permitted without written permission.

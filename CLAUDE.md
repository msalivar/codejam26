# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a submission for the **Kitboga Code Jam 2026** — the theme is "Unskippable Ad". The goal is to create a tricky, humorous, or creative ad overlay that makes it difficult (but ultimately possible) for the user to skip the ad.

**Deadline: Thursday, April 30th, 2026 at 11:59 PM Eastern Time**

## Development

Open `index.html` directly in a browser, or run a local server:

```bash
python3 -m http.server 8000
# Then open http://127.0.0.1:8000
```

Use the dropdown in the dev shell to switch between your submission and the four examples.

## Architecture

The project has two layers:

**Game shell (`index.html`)** — Do NOT modify. Renders a 960×540 video player and loads `submission/submission.html` in an iframe overlaid on top. Communicates with the iframe via `postMessage`.

**Submission (`submission/submission.html`)** — Your entire submission lives here (and in `submission/`). The iframe is transparent and positioned absolutely over the video. The overlay container is 960×540px.

### postMessage API

**Submission → Shell** (via `window.top.postMessage({ type }, '*')`):

| type | effect |
|------|--------|
| `success` | Ad dismissed — shows success banner, reloads |
| `fail` | Ad failed — shows fail banner, reloads |
| `play` / `pause` | Control video playback |
| `seekTo` | `value`: seconds |
| `setPlaybackRate` | `value`: number (e.g. `0.5`, `2`) |
| `setVolume` | `value`: 0–1 |
| `getVideoInfo` | Shell replies with `videoInfo` event |
| `setVideoFilter` | `value`: CSS filter string (e.g. `'blur(2px)'`) |

**Shell → Submission** (via `window.addEventListener('message', ...)`):

| type | payload |
|------|---------|
| `adStarted` | Ad begins playing; iframe just loaded |
| `adFinished` | Video reached end |
| `timeupdate` | `{ currentTime, duration, paused, playbackRate }` — fires frequently |
| `videoInfo` | `{ currentTime, duration, paused, playbackRate, volume, muted }` — response to `getVideoInfo` |

Default behavior in the template: `adFinished` triggers `fail`. Replace this with your own logic (e.g. a survey, puzzle, fake skip).

## Submission: TypeRacer Skip

The submission (`submission/submission.html`) is a self-contained single-file TypeRacer-style ad mechanic. Clicking "Skip Ad" reveals a typing panel at the bottom of the frame. The user must type a passage to advance a "Skip" bar racing against the ad's progress bar.

**State machine phases:** `'waiting'` → `'idle'` (skip button shown) → `'typing'` (panel active) → `'done'`

**Key design points:**
- A rubber-band difficulty controller (`onWordCompleted`) keeps the skip bar at ~70% of the ad bar's position until the win threshold is reached.
- Win unlocks at `min(duration × 30%, 30s)`. Before that, a hard ceiling prevents the car icon from ever visually passing the flag.
- The passage is infinite: one Pool B opener (ironic first-person sentence) followed by Pool A entries (pangrams, alliterations, Kitboga-themed text) appended on demand.
- All tunable values (threshold fractions, scale bounds, timing) live in the `// ---- Config ----` block at the top of the `<script>`, making them easy to adjust without reading implementation code.
- Hidden cheat key: **F8** bypasses the rubber-band and win threshold immediately.
- Written in ES5 `var` — no `let`/`const`, no build tools, no modules.

Design spec: `docs/superpowers/specs/2026-04-06-typeracer-skip-design.md`
Implementation plan: `docs/superpowers/plans/2026-04-06-typeracer-skip.md`

## Constraints

- **No build tools / no minified code.** Plain HTML/CSS/JS only.
- **No external requests** except well-known CDNs (jsdelivr, unpkg, googleapis). All other assets must be in the repo.
- **No WebAssembly or Web Workers.**
- **Forbidden APIs**: `fetch`, `XMLHttpRequest`, `WebSocket`, `navigator.geolocation`, `navigator.getUserMedia`, `navigator.clipboard`, `navigator.serviceWorker`, `window.open`, `window.print`, `RTCPeerConnection`, and others listed in README.
- Must call `success` or `fail` eventually.
- Desktop Chrome/Edge required; mobile-friendly preferred.
- The video filename/path is unknown at judging time — don't hardcode `../ads/ad_X.mp4` for logic; use `getVideoInfo` for timing data.

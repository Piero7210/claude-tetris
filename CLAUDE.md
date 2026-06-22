# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open `index.html` directly in a browser, or serve with any static server:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Architecture

Three files, zero dependencies:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600px), sidebar panel (score/lines/level/next-piece preview), and a single overlay div reused for both PAUSE and GAME OVER states.
- **`style.css`** — Dark/retro arcade theme. Uses `backdrop-filter: blur` on the overlay.
- **`game.js`** — All game logic (~305 lines, `'use strict'`, no modules).

## game.js internals

**State**: mutable globals — `board` (2D array, 0 = empty, 1–7 = color index), `current` (active piece `{type, shape, x, y}`), `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`.

**Key constants** (top of file):
- `COLS=10`, `ROWS=20`, `BLOCK=30` — if changing canvas dimensions, also update `width`/`height` on `<canvas id="board">` in `index.html` to `COLS×BLOCK` by `ROWS×BLOCK`.
- `COLORS` — index 1–7 maps to piece types I/O/T/S/Z/J/L.
- `PIECES` — piece shapes as square matrices; cell values are the color index.
- `LINE_SCORES = [0, 100, 300, 500, 800]` — multiplied by `level` on clear.

**Game loop**: `requestAnimationFrame`-based. `loop(ts)` accumulates `dropAccum`; when it exceeds `dropInterval` the piece drops one row or locks. Drop speed: `max(100, 1000 − (level−1) × 90)` ms, recalculated in `clearLines()`.

**Rotation**: `rotateCW` (transpose + reverse rows). `tryRotate` tries kicks `[0, −1, +1, −2, +2]` on x before discarding.

**Ghost piece**: `ghostY()` projects the current piece straight down; drawn at `globalAlpha = 0.2`.

**Scoring**: hard drop +2pts/cell, soft drop +1pt/row; line clears use `LINE_SCORES[count] × level`.

**Game over** is detected in `spawn()` when the newly spawned piece immediately collides. `init()` is the full reset — called on page load and on "Reiniciar" button click.

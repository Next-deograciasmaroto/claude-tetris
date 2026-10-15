# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris in vanilla JavaScript + HTML5 Canvas + CSS. No dependencies, no package.json, no build step, no tests, no linter. README and UI strings are in Spanish; keep new user-facing text in Spanish.

## Running

Open `index.html` directly (`start index.html` on Windows), or serve the folder statically:

    python -m http.server 8000   # then http://localhost:8000

Verification is manual: play in the browser and check the console.

## Architecture

All logic lives in `game.js` (single script, `'use strict'`, global mutable state, no modules). `index.html` loads it at the end of `<body>`, so it grabs DOM elements by id at top level: `board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`. Renaming an id in HTML requires updating `game.js`.

- **Board model**: `ROWS × COLS` matrix; `0` = empty, `1–7` = piece type, which doubles as the index into both `COLORS` and `PIECES`. Piece shapes store their type number in filled cells, so `merge()` copies values straight into the board.
- **Movement**: every move/rotation is validated with `collide(shape, x, y)`. Cells with `y < 0` are allowed (spawn above the board). Rotation is `rotateCW` plus simple horizontal kicks `[0, -1, 1, -2, 2]` in `tryRotate` (not SRS).
- **Lock cycle**: `lockPiece()` → `merge()` → `clearLines()` (updates score/lines/level/`dropInterval`) → `spawn()`. Game over is detected in `spawn()` when the new piece collides immediately.
- **Loop**: `requestAnimationFrame(loop)` accumulates `dt` into `dropAccum`; gravity steps when it exceeds `dropInterval` (`max(100, 1000 - (level-1)*90)` ms). `draw()` redraws everything each frame (grid, board, ghost at alpha 0.2, current piece). The next-piece preview (`drawNext`) is only redrawn on `spawn()`.
- **Pause / game over**: implemented by cancelling the RAF (`animId`) and showing the shared `#overlay`. `init()` (also bound to the restart button) fully resets state and hides the overlay. Note: resuming from pause restarts the loop but does not re-hide the overlay.

## Coupled constants

- Canvas size in `index.html` must equal `COLS × BLOCK` by `ROWS × BLOCK` (currently 300×600).
- `#next-canvas` (120×120) assumes a 4×4 grid of 30px cells (`NB` in `drawNext`).
- Adding a piece type means extending `PIECES`, `COLORS`, and the `* 7` in `randomPiece()` together.

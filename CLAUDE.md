# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No `package.json`, no build, no tests, no linter, no dependencies. The README and UI text are in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python3 -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

Three files, all loaded by `index.html` (`game.js` via a plain `<script>`, so everything is global state):

- `game.js` — all logic. Module-level `let` state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, …) is reset by `init()`, which is also the restart button handler. The `requestAnimationFrame` `loop` accumulates time and drops the piece when `dropAccum >= dropInterval`; locking goes through `lockPiece()` → `clearLines()` → `spawn()`, and `spawn()` calls `endGame()` if the new piece collides immediately.
- `index.html` — fixed-size canvases: `#board` (300×600) and `#next-canvas` (120×120), plus HUD spans and the `#overlay` used for both pause and game over.
- `style.css` — dark/retro styling only.

Things that must stay in sync:

- `COLS`, `ROWS`, `BLOCK` in `game.js` must match the `#board` `width`/`height` attributes in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`).
- Board cells hold `0` (empty) or a 1–7 index into `COLORS`; `PIECES` entries use the same indices.
- Scoring: `LINE_SCORES` × level; hard drop +2/cell, soft drop +1/row. Level rises every 10 lines; fall speed is `max(100, 1000 − (level − 1) × 90)` ms.
- Rotation is `rotateCW` (transpose + reverse) with wall kicks of ±1/±2 columns in `tryRotate`.

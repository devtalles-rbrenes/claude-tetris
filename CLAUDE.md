# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A Tetris-style game in vanilla JavaScript + HTML5 Canvas. There are no dependencies, no `package.json`, no build step, no linter and no test suite. The UI text and README are in Spanish, so keep user-facing strings in Spanish.

## Running

Open `index.html` directly, or serve the folder statically (recommended):

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

## Architecture

- `index.html` provides the DOM: `<canvas id="board">` (300×600), a side panel with `#score`, `#lines`, `#level` and `<canvas id="next-canvas">` (120×120), and one shared `#overlay` that both PAUSE and GAME OVER use (its text is set by JS).
- `game.js` holds all the logic in one non-module script (`'use strict'`) with global mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `dropAccum`, `animId`). It grabs the DOM elements by id at load time, so renaming ids in the HTML means updating `game.js` too.
- `style.css` is purely presentational (dark/retro theme).

Key model details in `game.js`:
- The board is a `ROWS × COLS` matrix. `0` means empty, and `1–7` is the piece type, which is also the index into `COLORS`. Piece matrices in `PIECES` store their own type number in their filled cells, so `merge()` copies color indices straight onto the board.
- A piece is `{ type, shape, x, y }`. `collide(shape, ox, oy)` is the single source of truth for bounds and overlap checks (rows above the board, `ny < 0`, are allowed). Movement, rotation (`tryRotate`, which tries horizontal kicks `[0,-1,1,-2,2]`), the ghost piece (`ghostY`) and gravity all go through it.
- Lifecycle: `init()` → `spawn()` (promotes `next` to `current` and ends the game if it collides right away) → `loop()` (rAF, gravity via `dropAccum` vs `dropInterval`) → `lockPiece()` = `merge()` + `clearLines()` + `spawn()`.
- Scoring and leveling live in `clearLines()`: `LINE_SCORES[cleared] * level`, level = `floor(lines/10)+1`, `dropInterval = max(100, 1000 - (level-1)*90)`. Soft drop adds 1 per row and hard drop adds 2 per cell.

## Gotchas

- `COLS`, `ROWS` and `BLOCK` must match the `#board` canvas `width`/`height` in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`). `drawNext()` assumes a 4×4 grid of 30px cells to match the 120×120 next canvas.
- The pause and game-over overlays share the same elements and the same restart button (`init`).

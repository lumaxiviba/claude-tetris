# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No `package.json`, no build, no linter, no tests. The README and UI text are in Spanish; keep user-facing strings in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

Three files, all logic in `game.js` (loaded as a classic script, `'use strict'`, global state — no modules):

- **State** is a set of module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, ...) reset together in `init()`. `init()` is also the restart handler.
- **Board** is a `ROWS × COLS` matrix; `0` = empty, `1–7` = index into both `COLORS` and `PIECES` (the piece type id doubles as the color id, and shape cells hold that id).
- **Piece lifecycle**: `spawn()` promotes `next` → `current`; `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. `spawn()` calls `endGame()` if the new piece collides immediately.
- **Two drop paths**: gravity in `loop()` (rAF, accumulates `dropAccum` vs `dropInterval`) and player `softDrop()`/`hardDrop()` in the keydown handler. Scoring for soft/hard drop lives in those functions; line-clear scoring, level, and `dropInterval` are updated in `clearLines()`.
- **Pause/game over** both cancel the rAF (`cancelAnimationFrame(animId)`) and reuse the same `#overlay` element; `togglePause()` restarts the loop by calling `loop()` directly.
- **Rendering**: `draw()` repaints the whole board each frame (grid, locked blocks, ghost at alpha 0.2, current piece); `drawNext()` renders into the separate `#next-canvas` and only runs on `spawn()`. `drawBlock` is shared by both canvases.
- Rotation (`tryRotate`) uses `rotateCW` plus simple horizontal kicks `[0,-1,1,-2,2]` — not SRS.

## Gotchas

- Canvas size is hardcoded in `index.html` (`300×600` board, `120×120` next). Changing `COLS`, `ROWS` or `BLOCK` in `game.js` requires updating those attributes to `COLS×BLOCK` / `ROWS×BLOCK`.
- DOM element ids referenced in `game.js` (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`) must stay in sync with `index.html`; the script runs at the end of `<body>` and calls `init()` immediately.
- The `.hidden` class on `#overlay` (defined in `style.css`) controls overlay visibility.

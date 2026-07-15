# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no `package.json` — the entire game is `index.html` + `style.css` + `game.js` (~300 lines).

## Running the game

No install/build step. Any of these work:

```bash
xdg-open index.html      # open directly (Linux)
python3 -m http.server 8000   # or: npx serve .   /   php -S localhost:8000
```

There is no test suite, linter, or build tool configured in this repo.

## Architecture

Everything lives in `game.js` as top-level functions operating on shared module-level state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) — there are no classes, no modules, no framework.

- **Board model**: `board` is a `ROWS × COLS` matrix (`createBoard`). Each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: `PIECES` defines the 7 tetrominoes as square matrices of color indices. `randomPiece()` deep-copies a shape and spawns it centered at the top. Rotation (`rotateCW`) is a transpose + row-reverse; `tryRotate()` applies it and attempts wall kicks by shifting `x` through `[0, -1, 1, -2, 2]` until a non-colliding position is found.
- **Collision** (`collide`): the single source of truth for whether a shape at a given offset is out of bounds or overlaps locked cells. Used by movement, rotation, ghost-piece projection, and the game loop.
- **Game loop** (`loop`, driven by `requestAnimationFrame`): accumulates elapsed time in `dropAccum`; once it exceeds `dropInterval`, the piece drops one row or locks if blocked, then calls `draw()` and reschedules itself. `animId` holds the current frame handle so pause/game-over can cancel it.
- **Locking a piece** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current shape into `board`, clears completed rows (scanning bottom-up, splicing full rows and unshifting empty ones at the top), then spawns the next piece. If the newly spawned piece immediately collides, `endGame()` fires.
- **Scoring/leveling**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop 1 pt/row. `level` increases every 10 lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Rendering** (`draw`, `drawBlock`, `drawGrid`, `drawNext`): plain Canvas 2D, redrawn fully every frame — grid lines, locked board cells, the ghost piece (`ghostY()` projects straight down, drawn at `globalAlpha = 0.2`), then the active piece. `drawNext` renders the preview into a separate small canvas.
- **Input**: a single `keydown` listener switches on `e.code` (arrows, `KeyX` for rotate, `Space` for hard drop, `KeyP` for pause), gated by `paused`/`gameOver`.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (px per cell), `COLORS`, `LINE_SCORES`, initial `dropInterval`. If you change `COLS`/`ROWS`/`BLOCK`, also update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).

## Files

- `index.html` — DOM structure: main board canvas, next-piece canvas, score/lines/level panel, pause/game-over overlay.
- `style.css` — dark retro-arcade theme (monospace HUD, backdrop-blur overlays).
- `game.js` — all game logic (see Architecture above).

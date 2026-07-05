# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A classic Tetris implementation in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS. No dependencies, no build step, no package manager — `index.html`, `style.css`, and `game.js` are the entire project.

## Running the game

There is no build/lint/test tooling. Just serve or open the files:

```bash
open index.html              # macOS, direct file open
python3 -m http.server 8000  # or any static server, then visit localhost:8000
```

Changes to `game.js`/`index.html`/`style.css` take effect on browser reload — no compilation step.

## Architecture

All game logic lives in `game.js` as top-level functions operating on module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) — there are no classes or modules.

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece type occupies it.
- **Pieces**: the 7 tetrominoes are defined in `PIECES` as square matrices using the same color-index convention. Rotation (`rotateCW`) is a transpose + row-reverse, not a lookup table.
- **Collision** (`collide`): checks board bounds and overlap with locked cells for a given shape/offset.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` and keeps the first that doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and advances the piece when it exceeds `dropInterval`.
- **Line clearing** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows at the top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
- **Leveling/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row and draws it at `globalAlpha = 0.2`.

Flow: `init()` builds the board, seeds `next` via `randomPiece()`, calls `spawn()` (promotes `next` to `current`, generates a new `next`), and starts the `loop`. If a freshly spawned piece immediately collides, `endGame()` fires and the Game Over overlay is shown. `keydown` handles movement/rotation/soft-drop/hard-drop; `P` toggles pause via `togglePause()`.

Tunable constants at the top of `game.js`: `COLS`, `ROWS`, `BLOCK` (cell size in px), `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, the `<canvas id="board">` `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris implemented in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS. No dependencies, no build step, no package.json, no test suite.

## Running the game

Open `index.html` directly in a browser, or serve it with any static server:

```bash
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

There is no build, lint, or test command — the code runs as-is in the browser.

## Architecture

Everything lives in three files that load directly via `<script src="game.js">` (no modules, no bundler):

- `index.html` — DOM structure: the `#board` canvas (300×600, i.e. `COLS×BLOCK` by `ROWS×BLOCK`), the side panel (score/lines/level/next-piece canvas/controls), and the pause/game-over overlay.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, structured around a small set of global `let` bindings (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) reset by `init()`.

Key mechanics in `game.js`:

- **Board model**: a `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7` identifying the locked piece.
- **Pieces**: defined as square matrices in `PIECES`. Rotation is done via `rotateCW` (transpose + row reverse), not by storing pre-rotated states.
- **Collision** (`collide`): checks board bounds and overlap with locked cells.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns before giving up on the rotation.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time and advances the piece one row once `dropAccum` exceeds `dropInterval`.
- **Line clearing** (`clearLines`): scans bottom-to-top, splicing out full rows and unshifting empty ones at the top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 points per row dropped, soft drop adds 1 point per row.
- **Leveling**: level increases every 10 cleared lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row and draws it at low alpha.

Flow: `init()` builds the board, seeds `next` via `randomPiece()`, calls `spawn()` to promote it to `current`, then starts the `requestAnimationFrame` loop. `spawn()` calling `collide()` immediately triggers `endGame()` (game over) if the new piece has no room. Keyboard input (`keydown`) drives movement/rotation/soft-drop/hard-drop/pause; `P` toggles pause independent of game-over state.

If you change `COLS`, `ROWS`, or `BLOCK` in `game.js`, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).

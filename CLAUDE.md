# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A Tetris game in vanilla JavaScript, HTML5 Canvas and CSS. It has no dependencies, no `package.json`, no build step, no linter and no tests. User-facing text (UI labels, overlay messages, README) is in Spanish.

## Running

```bash
open index.html                 # open directly (macOS)
python3 -m http.server 8000     # or serve statically, then visit http://localhost:8000
```

To verify a change, load the page in a browser and play. No automated checks exist.

## Architecture

- `index.html` holds the DOM: `#board` canvas (300×600), the side panel (`#score`, `#lines`, `#level`, `#next-canvas` 120×120), and one shared `#overlay` for the start screen (NUEVA PARTIDA), PAUSA and GAME OVER. The overlay's `#options` checkboxes (`#opt-next`, `#opt-speed`) are shown only on the start and GAME OVER screens, never during pause.
- `style.css` is the dark/retro styling. `.hidden` toggles the overlay.
- `game.js` holds all the logic in one script. It is not a module: it uses global mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …). `init()` resets all of that state and is also the restart handler.

Key invariants in `game.js`:

- **Cell value = piece type = color index.** `PIECES[t]` matrices are filled with the value `t` (1–7), `merge()` copies those values into `board`, and `drawBlock` looks up `COLORS[value]`. Index 0 / `null` means empty. If you add or reorder pieces, keep `PIECES`, `COLORS` and the `* 7` in `randomPiece()` in sync.
- **Board size is duplicated.** `COLS × BLOCK` and `ROWS × BLOCK` must match the `<canvas id="board">` width/height in `index.html`. The next-piece preview assumes a 4×4 grid of 30px cells, which matches its 120×120 canvas.
- **Collision** (`collide`) allows negative `y` (above the board) but rejects out-of-bounds `x` and `y >= ROWS`. All movement, rotation (`tryRotate`, which applies simple ±1/±2 column kicks, not SRS) and the ghost piece (`ghostY`) go through it.
- **Piece lifecycle:** `lockPiece()` → `merge()` → `clearLines()` (updates score, lines, level and `dropInterval`) → `spawn()`. `spawn()` promotes `next` to `current`, and if the new piece collides immediately it calls `endGame()`.
- **Game loop:** `loop()` runs on `requestAnimationFrame` and accumulates `dropAccum` until it reaches `dropInterval`. Pausing and game over stop it with `cancelAnimationFrame(animId)`. Unpausing restarts it by calling `loop()` directly.
- **Options are fixed per game:** `init()` snapshots the checkboxes into `settings` (`showNext`, `progressiveSpeed`), and the rest of the code reads only `settings`. The page doesn't start a game on load: it sets `gameOver = true` and shows the start overlay until the button calls `init()`.
- **Scoring:** `LINE_SCORES[cleared] * level`, plus 1 point per soft-drop row and 2 per hard-drop cell. The level is `floor(lines / 10) + 1`, and `dropInterval = max(100, 1000 − (level − 1) × 90)` only when `settings.progressiveSpeed` is on (otherwise it stays at 1000).

The README (in Spanish) documents the controls, the game flow and the tunable constants.

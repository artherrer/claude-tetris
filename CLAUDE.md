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

- `index.html` holds the DOM: `#board` canvas (300×600), the side panel (`#score`, `#lines`, `#level`, `#next-canvas` 120×120), and one shared `#overlay` for the start screen (NUEVA PARTIDA), PAUSA and GAME OVER. The overlay's `#options` (`#opt-next`, `#opt-speed`, `#opt-level`) and `#restart-btn` are shown only on the start and GAME OVER screens; during pause they are hidden and `#pause-menu` is shown instead (Reanudar, Reiniciar → `init()`, Ver controles toggling `#pause-controls`, and `#pause-level`, kept in sync with `#opt-level`). All overlay display goes through `showOverlay(title, text, withOptions, withPauseMenu)`. The `#skin-select` dropdown sits below the fixed `#theme-toggle` and is usable at any time.
- `style.css` is the dark/retro styling. `.hidden` toggles the overlay.
- `game.js` holds all the logic in one script. It is not a module: it uses global mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …). `init()` resets all of that state and is also the restart handler.

Key invariants in `game.js`:

- **Cell value = piece type = color index.** `PIECES[t]` matrices are filled with the value `t` (1–7), `merge()` copies those values into `board`, and `drawBlock` looks up `skin.colors[value]`. Index 0 / `null` means empty. If you add or reorder pieces, keep `PIECES`, every skin's `colors` in `SKINS` and the `* 7` in `randomPiece()` in sync.
- **Skins:** `SKINS` (`retro`, `neon`, `pastel`, `pixel`) each have `colors` (8 entries, index 0 `null`) and `draw(context, px, py, size, color)` (pixel coordinates). `skin` is the active one; `drawBlock(context, x, y, colorIndex, size, alpha)` sets `globalAlpha` (ghost uses 0.2), delegates to `skin.draw`, then resets `globalAlpha = 1` and `shadowBlur = 0` (Neon uses `shadowBlur` for its glow). Unlike `settings`, the skin can change anytime: `applySkin()` (from `#skin-select`, saved in `localStorage` key `skin`, invalid → `retro`) sets `data-skin` on `<html>` and calls `redraw()`, which is shared with `applyTheme()` because the loop isn't running when paused, at game over or on the start screen. `style.css` uses `[data-skin="neon"]` to force a black `#board`/`#next-canvas` and a dark `--grid` under either theme.
- **Board size is duplicated.** `COLS × BLOCK` and `ROWS × BLOCK` must match the `<canvas id="board">` width/height in `index.html`. The next-piece preview assumes a 4×4 grid of 30px cells, which matches its 120×120 canvas.
- **Collision** (`collide`) allows negative `y` (above the board) but rejects out-of-bounds `x` and `y >= ROWS`. All movement, rotation (`tryRotate`, which applies simple ±1/±2 column kicks, not SRS) and the ghost piece (`ghostY`) go through it.
- **Piece lifecycle:** `lockPiece()` → `merge()` → `clearLines()` (updates score, lines, level and `dropInterval`) → `spawn()`. `spawn()` promotes `next` to `current`, and if the new piece collides immediately it calls `endGame()`.
- **Game loop:** `loop()` runs on `requestAnimationFrame` and accumulates `dropAccum` until it reaches `dropInterval`. Pausing and game over stop it with `cancelAnimationFrame(animId)`. Unpausing (`togglePause()`) hides the overlay, blurs any focused menu control and restarts the loop by calling `loop()` directly.
- **Input lock:** `P` and `Escape` toggle pause (ignoring `e.repeat`; `togglePause()` is a no-op while `gameOver`, so Escape does nothing on the start/GAME OVER screens). While paused, keydown ignores every other key. After resuming or starting a game `ignoreRepeat` drops auto-repeat events until a fresh key press. Overlay buttons call `blur()` after click so Space/Enter don't retrigger them.
- **Options are fixed per game:** `init()` snapshots the checkboxes and `#opt-level` into `settings` (`showNext`, `progressiveSpeed`, `startLevel`), and the rest of the code reads only `settings`. The page doesn't start a game on load: it sets `gameOver = true` and shows the start overlay until the button calls `init()`.
- **Scoring:** `LINE_SCORES[cleared] * level`, plus 1 point per soft-drop row and 2 per hard-drop cell. The level is `max(settings.startLevel, floor(lines / 10) + 1)` (it starts at `startLevel`), and `speedFor(level)` gives `dropInterval = max(100, 1000 − (level − 1) × 90)` only when `settings.progressiveSpeed` is on (otherwise 1000); it is used both in `init()` and in `clearLines()`.

The README (in Spanish) documents the controls, the game flow and the tunable constants.

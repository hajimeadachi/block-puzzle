# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file browser game: a Tetris-style block puzzle (`index.html`). No build step, no dependencies, no server required.

## Running

Open `index.html` directly in a browser. For mobile testing, use Chrome DevTools device emulation (F12 → device icon).

## Architecture

Everything lives in `index.html` — HTML, CSS, and JavaScript in one file with no external dependencies.

**Game state globals** (declared at top of `<script>`):
- `board` — 2D array (20 rows × 10 cols), 0 = empty, 1–7 = piece color id
- `piece` / `nextPiece` — `{ id, matrix, x, y }` objects
- `score`, `level`, `lines`, `combo` — numeric state
- `gameOver`, `paused` — boolean flags
- `dropInterval` / `dropTimer` — timing for auto-drop loop

**Core functions and their responsibilities:**
- `loop(ts)` — `requestAnimationFrame` game loop; drives `drop()` on timer
- `drop()` — moves piece down one row; on collision calls `merge()` → `clearLines()` → spawn next piece
- `collide(b, p, dx, dy, mat)` — collision check used by all movement and rotation logic
- `doRotate()` — wall-kick rotation shared by keyboard and touch handlers
- `hardDrop()` — teleports piece to ghost position, awards `dy * 2` score
- `clearLines()` — handles combo multiplier (1×/1.5×/2×) and level/speed update
- `startGame()` / `endGame()` / `resumeGame()` — lifecycle; update `btnStart` text and overlay visibility

**Input handling:**
- Keyboard: `document.addEventListener('keydown', ...)` — arrows, Space, P
- Touch: three listeners on `#canvas-wrapper` (`touchstart`, `touchmove`, `touchend`, all `passive: false`)
  - `touchmove` → real-time left/right slide (30 px = 1 block)
  - `touchend` → tap (rotate), ↓ swipe >40 px (hard drop), ↑ swipe >30 px (rotate)
- Overlay click → start or resume (mouse); overlay tap is handled inside `touchend`

**Speed formula:** `dropInterval = Math.max(100, Math.round(1000 * 0.9 ** (level - 1)))` — 10% faster per level, floor 100 ms.

**Responsive layout:** CSS `@media (max-width: 520px)` switches `#wrapper` to column layout with stats in a flex-wrap row below the canvas. `user-scalable=no` prevents pinch-zoom during play.

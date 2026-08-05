# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start

This is a vanilla JavaScript Tetris game with no build step or dependencies. To run it:

```bash
# Option 1: Open directly
start index.html        # Windows
open index.html         # macOS
xdg-open index.html     # Linux

# Option 2: Local web server (recommended)
python3 -m http.server 8000    # Python 3
npx serve .                     # Node.js
php -S localhost:8000          # PHP
```

Then open `http://localhost:8000` in a browser.

## Project Structure

| File | Purpose |
|------|---------|
| `index.html` | DOM structure: two canvases (game board 300×600px, next-piece preview 120×120px), score/level/lines display, pause/game-over overlay |
| `style.css` | Dark arcade-style theming with flexbox layout, CSS variables for colors, backdrop blur effects |
| `game.js` | Complete game logic (~300 lines) |
| `README.md` | Full feature list and game controls (in Spanish) |

## Architecture & Key Concepts

### Game State
All state is stored in module-level variables: `board`, `current` (active piece), `next` (preview), `score`, `lines`, `level`, `paused`, `gameOver`, etc.

**Board model**: 2D array (`ROWS × COLS`, default 20×10). Each cell is `0` (empty) or `1–7` (piece color/type index).

### Game Loop
- **Trigger**: `requestAnimationFrame(loop)`
- **Timing**: Accumulates elapsed time in `dropAccum`. When `dropAccum >= dropInterval`, drops the piece one row or locks it.
- **Drop speed**: Calculated as `max(100, 1000 - (level - 1) × 90)` ms; increases each 10 cleared lines.

### Piece System
- **Data**: `PIECES` array holds 7 tetromino shapes (indices 1–7) as 3D/4D matrices.
- **Rotation**: `rotateCW(shape)` transposes + reverses rows (90° clockwise).
- **Wall kicks**: `tryRotate()` attempts rotation at offsets `[0, ±1, ±2]` columns if base position collides.
- **Collision**: `collide(shape, x, y)` checks bounds and board overlap.

### Scoring
Uses classic Tetris system: `[0, 100, 300, 500, 800]` × current level for line clears.
- Hard drop: +2 points per row fallen
- Soft drop: +1 point per row

### Drawing
- **Board**: `draw()` renders grid, locked blocks, ghost piece (preview of where current piece lands), and active piece.
- **Ghost piece**: `ghostY()` projects piece downward to find landing row; drawn at 20% opacity.
- **Next preview**: `drawNext()` renders the next piece in its own 4×4 canvas preview.

## Common Customization

All tunable constants are at the top of `game.js`:

| Constant | Default | Notes |
|----------|---------|-------|
| `COLS` | 10 | If changed, update canvas width: `width = COLS × BLOCK` |
| `ROWS` | 20 | If changed, update canvas height: `height = ROWS × BLOCK` |
| `BLOCK` | 30 | Cell size in pixels |
| `COLORS` | 7 hex colors | One per piece type (index 1–7) |
| `LINE_SCORES` | `[0,100,300,500,800]` | Points for 1, 2, 3, or 4 lines cleared |

**Important**: If you modify `COLS`, `ROWS`, or `BLOCK`, also update the corresponding `width` and `height` attributes on both canvas elements in `index.html` to keep rendering consistent.

## Game Flow

1. **Init**: `init()` creates an empty board, seeds the next piece, and starts the game loop.
2. **Spawn**: `spawn()` moves `next` to `current` and generates a new `next`. If `current` collides immediately, `endGame()` fires.
3. **Drop**: `loop()` advances pieces each frame. On timeout, either moves down or locks.
4. **Lock & Clear**: `lockPiece()` merges the piece into the board, clears full lines, and spawns the next piece.
5. **Input**: `keydown` listener handles movement, rotation, drops, and pause.

## Key Functions Reference

| Function | Purpose |
|----------|---------|
| `init()` | Reset all state and start new game |
| `loop(ts)` | Main game loop, called by `requestAnimationFrame` |
| `spawn()` | Move `next` piece to `current`, generate new `next` |
| `collide(shape, x, y)` | Check if piece shape collides at given position |
| `tryRotate()` | Rotate current piece with wall-kick attempts |
| `lockPiece()` | Merge piece to board, check for line clears, spawn next |
| `clearLines()` | Find and remove complete rows, update score/level |
| `hardDrop()` | Instant drop to bottom |
| `softDrop()` | Accelerated single-row drop |
| `draw()` | Render board, blocks, ghost, and current piece |
| `drawNext()` | Render next piece preview |
| `ghostY()` | Calculate landing row for ghost piece |

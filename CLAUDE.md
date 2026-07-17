# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Claude Tetris** is a fully playable Tetris game implemented in vanilla JavaScript, HTML5 Canvas, and CSS3. It has **zero dependencies** and no build process—just open `index.html` in a browser and play.

### Key Characteristics
- ~305 lines of self-contained game logic (game.js)
- Pure Canvas 2D rendering (no frameworks, no bundlers)
- Classic Tetris mechanics: 7-piece types, rotation with wall kicks, ghost piece, scoring, levels, pause/resume
- Dark/retro arcade aesthetic using CSS3 flexbox and backdrop filters
- Bilingual (Spanish/English) with Spanish as primary language in comments and UI

## How to Run

**No build or installation needed.** Three options:

```bash
# Option 1: Open directly in browser
start index.html          # Windows
open index.html           # macOS  
xdg-open index.html       # Linux

# Option 2: Python 3 HTTP server
python3 -m http.server 8000

# Option 3: Node.js static server
npx serve .
```

Then visit `http://localhost:8000` in a browser.

## Architecture Overview

### Game Logic Flow

The game is built around a **game loop** driven by `requestAnimationFrame`, not a timer-based loop:

```
init()
  ├─ createBoard()           → 20×10 matrix filled with 0s
  ├─ spawn()                 → move `next` piece to `current`, generate new `next`
  └─ requestAnimationFrame(loop)
        ↓
   loop(timestamp)
     ├─ calculate dt = time since last frame
     ├─ accumulate dt into dropAccum
     ├─ if dropAccum ≥ dropInterval → auto-drop piece or lockPiece()
     ├─ draw() → render grid, board state, ghost piece, current piece
     └─ requestAnimationFrame(loop) → recurse
```

**Key insight:** The loop doesn't drop pieces at fixed intervals; it checks accumulated time (`dropAccum`) and only drops when enough milliseconds have passed. This keeps the game responsive to input even on high refresh-rate displays.

### Board Model

The game state is a 2D array:
- `board[row][col]` where row ∈ [0, 19], col ∈ [0, 9]
- Each cell holds `0` (empty) or a color index (1–7, one per piece type)
- When a piece lands, its cells are merged into `board` via `merge()`

### Pieces and Rotation

Pieces are defined as 4×4 matrices with color indices:
```javascript
PIECES[0] = null;
PIECES[1] = [[0,0,0,0],[1,1,1,1],[0,0,0,0],[0,0,0,0]]; // I-piece
PIECES[2] = [[2,2],[2,2]];                             // O-piece (2×2)
// etc.
```

Rotation (`rotateCW`) transposes the shape matrix and reverses rows. **Wall kicks** (`tryRotate`) attempt offsets [0, ±1, ±2] to allow rotations near walls.

### Collision Detection

`collide(shape, x, y)` checks:
1. No cell outside board bounds (x < 0, x ≥ COLS, y ≥ ROWS)
2. No cell overlaps with already-locked blocks in `board`

The ghost piece (`ghostY()`) reuses this to find the lowest valid `y` for the current piece.

### Scoring & Levels

- **Line clears:** 1, 2, 3, or 4 lines yield `[0, 100, 300, 500, 800]` points × level
- **Soft drop:** 1 point per row moved down manually
- **Hard drop:** 2 points per row fallen
- **Levels:** Increment every 10 lines; speed increases: `dropInterval = max(100, 1000 - (level-1) × 90)` ms

### State Variables (in game.js)

```javascript
let board;              // 2D array [ROWS][COLS]
let current;            // { type, shape: [[...]], x, y } — active piece
let next;               // { type, shape, x, y } — preview piece
let score, lines, level;
let paused, gameOver;
let lastTime, dropAccum, dropInterval;  // timing
```

## File Structure

| File | Purpose |
|------|---------|
| `index.html` | DOM structure: game canvas (300×600), panel (score/lines/level/next-preview), overlay (pause/game-over), restart button |
| `style.css` | Dark theme styling, flexbox layout, backdrop blur overlays, monospace fonts for scores |
| `game.js` | All game logic: board model, piece spawning, collision, rotation, line clearing, scoring, rendering, input handling |

## Key Functions

| Function | Purpose |
|----------|---------|
| `init()` | Reset board, score, level; spawn first pieces; start game loop |
| `loop(ts)` | Main game loop: accumulate time, auto-drop if needed, draw |
| `collide(shape, ox, oy)` | Check if piece at (ox, oy) hits boundaries or locked blocks |
| `rotateCW(shape)` | Rotate shape 90° clockwise |
| `tryRotate()` | Rotate piece; if collision, try wall kicks [0, ±1, ±2] |
| `merge()` | Lock current piece into board |
| `clearLines()` | Remove complete rows; update score/level/speed |
| `ghostY()` | Calculate landing row for ghost piece |
| `hardDrop()` | Instant fall to ghost position |
| `softDrop()` | Move down 1 row manually (1 point) or lock if collision |
| `lockPiece()` | Merge piece, clear lines, spawn next |
| `draw()` | Render grid, board, ghost, current piece |
| `drawNext()` | Render next piece preview in side canvas |

## Customization Points

All tunable constants are at the top of `game.js`:

| Constant | Default | Notes |
|----------|---------|-------|
| `COLS` | 10 | Board width; update canvas width too |
| `ROWS` | 20 | Board height; update canvas height too |
| `BLOCK` | 30 | Size in pixels per cell |
| `COLORS` | Array | 7 hex colors (I=cyan, O=yellow, T=purple, S=green, Z=red, J=pale blue, L=orange) |
| `LINE_SCORES` | [0,100,300,500,800] | Points for 1/2/3/4 lines |

Canvas dimensions in `index.html`:
- Main board: `width="300"` (10 cols × 30px), `height="600"` (20 rows × 30px)
- Preview: `width="120" height="120"` (4×4 cells × 30px)

## Input Handling

Event listener on `keydown` (in game.js:277):
- `ArrowLeft` / `ArrowRight` → move piece
- `ArrowUp` / `X` → rotate clockwise
- `ArrowDown` → soft drop
- `Space` → hard drop
- `P` → toggle pause

## Rendering Pipeline

The canvas is cleared each frame and redrawn in order:
1. `drawGrid()` → light grid lines (`#22222e`)
2. Board state cells via `drawBlock()`
3. Ghost piece at 20% opacity
4. Current piece at full opacity

`drawBlock()` fills each cell with its color, then adds a white highlight bar at the top for depth.

## Common Tasks

### Test the game
1. Run a local server: `python3 -m http.server 8000`
2. Open http://localhost:8000
3. Play with mouse + keyboard until a win condition or expected behavior is observed

### Change board dimensions
1. Update `COLS` and `ROWS` in game.js
2. Update canvas `width` and `height` in index.html: width = COLS × BLOCK, height = ROWS × BLOCK
3. Keep `BLOCK` unchanged or adjust accordingly

### Adjust piece colors
Edit the `COLORS` array in game.js (indices 1–7 correspond to I, O, T, S, Z, J, L).

### Change initial drop speed
Adjust `dropInterval = 1000` in `init()` (milliseconds).

### Add a new piece type
1. Add to `PIECES` array
2. Add to `COLORS` array
3. Update `randomPiece()` to include the new index
4. Update canvas sizing if piece extends beyond 4×4

## Notes on Complexity

- **Line clearing:** Iterates from bottom to top; splice/unshift keeps the board intact
- **Collision system:** O(9) per check (3×3 piece max) vs O(200) per board cell
- **Ghost piece:** Recalculated every frame; could be optimized but cheap enough for 60 FPS
- **Pause:** Cancels `requestAnimationFrame` to freeze the game; stores overlay state
- **Game Over:** Detected when a spawned piece collides immediately

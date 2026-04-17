# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of browser-based mini-games. Each game is a single self-contained HTML file with no dependencies, no build tooling, and no server — open in any modern browser to play.

## Running the Games

Open the HTML file directly in a browser. No install, no build step.

- `snake.html` — Snake game with human and computer-play modes
- `tictactoe.html` — Tic Tac Toe with 2-player and vs-CPU modes

## Code Architecture

### Shared Patterns

Both files follow the same structure:
- All CSS is inline in `<style>` within `<head>`
- All logic is inline in `<script>` at the bottom of `<body>`
- State is held in module-level `let` variables, reset by an `init()` function
- Mode switching (human vs CPU) is done via toggle buttons that set a boolean flag and call `init()`
- Scores persist across games in memory (page refresh resets them)

---

### Snake (`snake.html`)

**State variables** (module-level):
| Variable | Purpose |
|----------|---------|
| `snake` | Array of `{x, y}` grid cells, head at index 0 |
| `dir` / `next` | Current and queued direction vectors; `next` is set on keydown, applied each tick to prevent 180° reversal |
| `food` | `{x, y}` position of the current food pellet |
| `score` / `best` | Current and all-time high score |
| `loop` | `setInterval` handle for the game tick |
| `speed` | Tick interval in ms (starts 150, floors at 60) |
| `running` | Whether a game is active |
| `cpuMode` | Whether the computer is controlling the snake |

**Game loop flow:**
1. `init()` — resets all state, hides the overlay, calls `clearInterval` then re-starts `setInterval(tick, speed)`
2. `tick()` — called every `speed` ms:
   - If `cpuMode`, calls `cpuChoose()` to update `next`
   - Advances head by `dir`, checks wall and self collision → `gameOver()` if hit
   - Unshifts new head onto `snake`; pops tail unless food eaten
   - Every 5 points, decrements `speed` by 10 and restarts the interval
   - Calls `draw()`
3. `draw()` — redraws canvas: background, grid dots, food (with glow shadow), snake segments (HSL gradient from head to tail), eyes on head oriented by `dir`

**CPU AI:**
- `cpuChoose()` first tries `bfsDir(food, snake)` — BFS from head to food, returns the first-step direction of the shortest path
- If BFS finds no path (food is unreachable), falls back to `safeDir()` — picks the adjacent cell that maximises reachable open space via flood-fill (`floodCount`)
- Reverse moves (180°) are explicitly excluded in both paths

**Canvas rendering details:**
- Grid: 20×20 cells on a 400×400 canvas (`COLS = ROWS = 20`, `GRID = 20`)
- `roundRect()` wraps `ctx.roundRect` + `ctx.fill()` — used for both food and snake segments
- Snake segments use `hsl(330–350, 70%, 60–80%)` gradient based on segment index ratio

---

### Tic Tac Toe (`tictactoe.html`)

**State variables:**
| Variable | Purpose |
|----------|---------|
| `board` | Flat 9-element array of `''`, `'X'`, or `'O'` |
| `current` | Active player: `'X'` or `'O'` |
| `over` | Whether the current game has ended |
| `vsComputer` | Whether the CPU is playing as O |
| `scores` | `{ X, O, D }` object tracking wins and draws |

**Game flow:**
1. `setMode(cpu)` — sets `vsComputer`, updates button styles and score label text, calls `init()`
2. `init()` — resets `board`, `current` (`'X'`), `over`; clears all cell classes and text
3. `playMove(i)` — places current player's mark, checks win via `checkWin()`, checks draw, or switches `current` and (if CPU's turn) schedules `cpuMove()` via `setTimeout(300ms)`

**Win detection:**
- `checkWin(b)` iterates the 8 hardcoded winning triplets and returns the winning `[a, i, c]` indices or `null`
- Winning cells get the `win` CSS class (highlighted with box-shadow glow)

**CPU AI:**
- `minimax(b, isMax)` — full minimax, no depth limit, no alpha-beta pruning (acceptable for 3×3)
- CPU always plays O; `isMax=true` means it's O's turn (maximising), `isMax=false` means X's turn (minimising)
- `cpuMove()` iterates all empty cells, runs minimax for each, picks the highest-scoring move
- CPU move is delayed 300ms with `setTimeout` for UX feel

**DOM structure:**
- 9 `.cell` divs with `data-i` attributes 0–8 map directly to `board` array indices
- `.taken` class prevents re-clicking; `.x` / `.o` classes set text colour

---

## Styling Conventions

- Snake uses a **pink/rose** palette (`#fff0f5` background, `#e91e8c` / `#c2185b` accents)
- Tic Tac Toe uses a **dark navy** palette (`#1a1a2e` background, `#e94560` for X, `#a8dadc` for O)
- Both use `border-radius: 8–12px` rounded corners and `transition` on interactive elements
- Font: `'Segoe UI', sans-serif` throughout

## GitHub

Repository: automatically synced — any committed changes are pushed to the remote.

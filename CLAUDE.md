# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"Sektor 7: Der Wartungsroboter" — a self-contained, single-file web game for an Informatik-Rallye (CS rally). Players construct robot programs using limited block inventory to solve three maze puzzles. German-language UI throughout.

## Development

**No build step.** Open `roboter-finale.html` directly in a browser. All HTML, CSS, and JavaScript lives in that one file.

## Architecture

The entire game is in `roboter-finale.html`. The JavaScript is organized into logical sections separated by `=`-bordered comments:

### Core Data

- **`BLOCKDEFS`** — catalog of available programming block types (loop, if/else, fwd, rotate, open)
- **`MAZES`** — three ASCII-encoded maze strings; `S`=start, `Z`=goal, `T`=door, `K`=key
- **`program`** — user's constructed program: array of instruction nodes (container blocks like loop/if hold nested arrays)
- **`stock`** — tracks remaining block inventory per type; limited supply is the core puzzle constraint
- **`selSlot`** — insertion cursor position in the program list

### Key Functions

| Function | Role |
|---|---|
| `parseMaze(str)` | Converts ASCII string → structured grid with metadata (start, goal, door, key positions) |
| `simulate()` | Executes `program` against each maze; captures per-step frames for animation; detects crashes, step limit (300), and success |
| `drawMaze(ctx, maze, frame)` | Canvas renderer: 30×30 px cells, robot as triangle, key as hexagon, goal as diamond, crash=red, success=green |
| `renderPalette()` / `renderList()` / `renderNode()` | Re-renders UI from scratch on each state change (no virtual DOM) |
| Animation loop | `setInterval` at ~14 FPS plays back frames; reveals XOR-encoded solution word when all three mazes are solved |

### Simulation Logic

`simulate()` is the core engine. It runs the user's `program` array recursively to handle nested loops and if/else blocks. Robot state per tick: `{x, y, dir, steps, hasKey, doorOpen}`. Conditions evaluated: `frontFree`, `leftFree`, `rightFree`, `doorAhead`. Each tick's state is stored as a frame; the animation loop replays these.

### Solution Encoding

The final answer word (line ~518) is XOR-encoded with byte `91`, stored as an array of ASCII codes. It decodes and displays only when all three mazes report success — prevents trivial inspection.

## Maze Format

```
#########
#S      #
# ##### #
#     Z #
#########
```
- `#` = wall, space = open, `S` = robot start, `Z` = goal, `T` = door, `K` = key
- Mazes must be rectangular with uniform row length

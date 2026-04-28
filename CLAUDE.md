# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a collection of browser-based games implemented as self-contained HTML files. No build process, dependencies, or tooling is required—each game runs directly in a modern web browser.

- **tictactoe.html**: Classic tic-tac-toe with 2-player and AI modes
- **retro-shooter.html**: Top-down 2D shooter with multiple levels and enemies

## Git Workflow

**After every code change, commit to git and push to GitHub.** This ensures all work is version-controlled and backed up to the remote repository.

- Stage changed files: `git add <files>` or `git add .`
- Commit with a clear, descriptive message explaining what changed and why: `git commit -m "Message"`
- Push to origin/master: `git push`

Do this immediately after completing any modification, bug fix, or feature work. This prevents losing work and makes it easy to revert if needed.

## Running the Games

Open either HTML file directly in a web browser. No server or build step required.

- Tic Tac Toe: Can be toggled between 2-player mode and vs-AI mode
- Retro Shooter: Arrow keys or WASD to move, mouse to aim, left-click to shoot

## Architecture

### Tic Tac Toe

A straightforward game implemented with vanilla JavaScript. Key aspects:

- **Game state**: `board` (9-element array), `current` (active player), `gameOver` flag, `scores` (win counts)
- **Win detection**: Iterates through `WINS` constant (8 possible win combinations)
- **AI logic**: Uses minimax algorithm (see `minimax()` and `checkWinnerFor()`) to evaluate moves with +10 for AI win, -10 for player win, 0 for draw
- **UI state**: Includes score persistence across multiple games via score object

### Retro Shooter

A more complex canvas-based game with structured entity systems.

**Core Architecture:**
- **Levels**: Defined in `LEVELS` array with per-level config (kill goals, spawn rates, enemy mix, difficulty multipliers)
- **Game states**: MENU → PLAYING → LEVEL_COMPLETE → GAME_OVER (or WIN)
- **Entity types**: Player, Enemies (grunt/dasher), Bullets, Particles
- **Game loop**: Fixed deltaTime capped at 50ms, updates entities, checks collisions, renders

**Rendering System:**
- **Pixel sprites**: Palette-based sprite system via `drawPixelSprite()`. Each sprite is a 2D array of palette indices (0 = transparent). Sprites are drawn pixel-by-pixel using canvas fillRect.
- **Palettes**: Define colors for each sprite type (PLR_PAL, GRU_PAL, DSH_PAL)
- **Frame animation**: Each entity has `frame`, `frameTimer`, `frameDur` for simple frame-based animation

**Game Mechanics:**
- **Enemy spawning**: Controlled by `spawnTimer` and `cfg.spawnInterval`; respects `cfg.maxEnemies` cap
- **Collision detection**: Simple circle-based overlap via distance check (see `overlap()`)
- **Particle effects**: Radial particle spawning with velocity, friction, and fade-out
- **Dasher enemies**: Use sine-wave zigzag pattern (see `zigTimer`, `zigPhase`, `zigInt`) to strafe while approaching

## Common Development Tasks

### Modifying Game Balance (Shooter)

Edit the `LEVELS` array to adjust:
- `killGoal`: Number of kills needed to advance
- `spawnInterval`: Frequency of enemy spawning (seconds)
- `maxEnemies`: Max concurrent enemies
- `spdMult`, `hpMult`: Difficulty scaling multipliers

### Adding a New Enemy Type

1. Create a new palette in the form `XXX_PAL = [null, color1, color2, ...]` (index 0 reserved for transparency)
2. Define sprite frames as 2D arrays: `XXX_F = [frame0, frame1, ...]`
3. Add enemy type string to `LEVELS[].types` array
4. Implement physics in `updateEnemies()` if special behavior needed (like dasher's zigzag)
5. Add rendering in `renderGame()` before the existing enemy render

### Extending Tic Tac Toe

The AI difficulty is inherent to minimax (always optimal). To reduce difficulty, modify the minimax scoring (reduce magnitude of ±10) or add randomness to move selection in `aiMove()`.

## Code Patterns

### Pixel Sprite Rendering (Shooter)

Sprites are stored as 2D arrays where each value is a palette index:

```
const PAL = [null, '#color1', '#color2', ...];
const SPRITE = [
  [0, 0, 1, 1, 0, 0],    // row 0: indices map to PAL
  [0, 1, 2, 2, 1, 0],    // row 1
  ...
];
drawPixelSprite(SPRITE, PAL, centerX, centerY, cellSize);
```

Index 0 is always transparent (skipped). Increasing `cellSize` enlarges the sprite proportionally.

### Game Loop Structure (Shooter)

```
requestAnimationFrame(loop):
  1. Calculate deltaTime (capped at 50ms)
  2. Clear canvas and draw background
  3. Update all game entities
  4. Check collisions
  5. Render entities and HUD
  6. Render state-specific overlay (menu, level complete, etc)
```

Deltatime capping prevents large jumps when the frame rate drops.

### Input Handling

- **Shooter**: Uses `keys` object for continuous input (WASD, arrows), `mouseX`/`mouseY` for aiming, `mouseDown` flag for continuous firing
- **Tic Tac Toe**: Click-based cell selection via `play(index)` function

## Notes for Future Maintainers

- Both games use fixed HTML structure with inline styles for total portability (no external assets)
- No external libraries; pure vanilla JS and Canvas 2D
- Performance is not optimized (e.g., particle rendering redraws every frame); acceptable for these game scales
- Shooter's minimax and sprite rendering are the most complex systems; changes here require careful testing


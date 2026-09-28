# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS. No dependencies, no bundler, no package.json, no tests, no linter. The README and in-game text are in Spanish; match that for user-facing strings.

## Running

Open `index.html` directly in a browser, or serve statically: `npx serve .` (then http://localhost:3000). Reload the page to see changes.

## Architecture

All game logic lives in `game.js` (loaded by `index.html` as a classic script, not a module). It is organized in sections marked by `// ── Name ──` comment banners:

- **Input**: `keys` (held state) and `justPressed` (edge-triggered). Read edge-triggered input via `pressed(code)`, which consumes the flag. Shooting and restart use `pressed('Space')`, so holding Space does not auto-fire.
- **Entities** (`Bullet`, `Asteroid`, `Ship`, `Particle`): each has `update(dt)` and `draw()` and a `dead` flag. Dead entities are removed by `filter` in `update()`, not by the entities themselves.
- **Game state**: module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`). `state` is `'playing' | 'dead' | 'gameover'`. `'dead'` is a 2s respawn timer during which asteroids and particles keep updating. `'gameover'` waits for Space to call `initGame()`.
- **Loop**: `requestAnimationFrame` with `dt` in seconds, clamped to 0.05s. `update(dt)` runs first, then `draw()`.

Conventions worth knowing:
- The playfield is a fixed 800x600 (`W`, `H`, which must match the canvas attributes in `index.html`) and toroidal. Use `wrap()` for positions.
- Asteroid size is 1/2/3 and indexes the parallel arrays `RADII`, `SPEEDS`, `POINTS`. Note that `POINTS` is inverted vs. size (small = 100 points). `split()` produces two asteroids of `size - 1`.
- Collision is circle-based via `dist()`. Ship vs. asteroid uses `ship.radius + a.radius * 0.82`, and is skipped while `ship.invincible > 0`.
- Level clear (`asteroids.length === 0`) calls `nextLevel()`, which spawns `3 + level` large asteroids outside a safe radius around the center.
- Rendering is all vector strokes on a black background, with no sprites or assets (`favicon.svg` is the only asset).

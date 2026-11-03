# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids-style arcade game in plain HTML5 Canvas + vanilla JavaScript (ES6+). No frameworks, no bundler, no dependencies, no build step, no tests, no linter. Code comments, UI text (`NIVEL`, `PUNTAJE`, etc.) and the README are in Spanish — keep new user-facing strings and comments in Spanish for consistency.

## Running

Open `index.html` directly in a browser, or serve the folder:

```bash
npx serve .
```

Then visit `http://localhost:3000`. There is nothing to compile; reload the page to see changes.

## Architecture

All game logic lives in a single file, `game.js`, loaded by `index.html` as a classic (non-module) script in strict mode. The canvas is a fixed 800×600 (`W`/`H` constants must match the `<canvas>` attributes in `index.html`).

- **Main loop**: `requestAnimationFrame` → `loop(ts)` computes `dt` in seconds (clamped to 0.05s) → `update(dt)` then `draw()`. All movement is time-based; use `dt` for any new motion/timers.
- **Entities**: `Ship`, `Bullet`, `Asteroid`, `Particle` classes, each with `update(dt)` and `draw()`, drawing directly to the global `ctx`. Entities are removed by setting `dead = true`; the arrays are then filtered in `update()`.
- **Global state**: module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`). `initGame()` resets everything; `nextLevel()` spawns `3 + level` large asteroids when the field is cleared.
- **State machine**: `state` is `'playing' | 'dead' | 'gameover'`. `update()` branches on it early — `'dead'` runs a respawn timer (asteroids keep moving), `'gameover'` waits for Space to restart.
- **Toroidal space**: positions are wrapped with `wrap()` for ship, bullets and asteroids (particles do not wrap). Collisions use simple circle distance (`dist()`); ship-vs-asteroid uses `a.radius * 0.82` for forgiveness and is skipped while `ship.invincible > 0`.
- **Asteroid sizes**: size index 3 (large) → 2 → 1 (small). `RADII`, `SPEEDS`, `POINTS` arrays are indexed by size (index 0 unused). `split()` returns two children of `size - 1`.
- **Input**: `keys[code]` holds held keys (rotation/thrust); `pressed(code)` is an edge-triggered, consume-once check via `justPressed` (used for Space/shooting and restart). Arrow keys and Space have `preventDefault()` to avoid page scrolling.

## Notes

The README mentions power-ups and special asteroid types (e.g. a "estrella fugaz" / shooting star) — these are **not** implemented in `game.js` yet.

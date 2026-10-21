# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS. No dependencies, no bundler, no package.json, no tests, no linter. UI strings and comments are in Spanish; keep that convention.

## Running

Open `index.html` in a browser, or serve the folder: `npx serve .` then visit `http://localhost:3000`.

## Architecture

All game logic lives in `game.js`, loaded by `index.html` (fixed 800x600 canvas `#canvas`). It is a single global-scope script, organized in sections:

- **Input**: `keys` (held state) and `justPressed` (edge-triggered). Read edge input via `pressed(code)`, which consumes the flag. Used for `Space` (shoot, restart) so holding does not autofire.
- **Entities**: `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has `update(dt)` and `draw()` and a `dead` flag; arrays are filtered by `dead` each frame rather than spliced.
- **Game state**: module-level `let` globals (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`) plus `state` = `'playing' | 'dead' | 'gameover'`. `initGame()` resets everything; `nextLevel()` is triggered when `asteroids.length === 0`.
- **Loop**: `loop(ts)` computes `dt` in seconds (clamped to 0.05), then `update(dt)` and `draw()`. `update` branches on `state` first; `'dead'` is a 2s respawn timer during which asteroids still move.

Key details:
- World is toroidal: all positions wrap via `wrap(v, max)` using hardcoded `W`/`H` constants in `game.js` (must match canvas size in `index.html`).
- Asteroid size is 1-3 and indexes parallel arrays `RADII`, `SPEEDS`, `POINTS`. `split()` produces two asteroids of `size - 1`; size 1 yields none.
- Collisions are circle-based using `dist()`: bullet hits when `dist < a.radius`; ship hits when `dist < ship.radius + a.radius * 0.82`, skipped while `ship.invincible > 0` (also drives respawn blink in `Ship.draw`).
- Everything is vector-style (strokes only, no sprites).

## Notes

`README.md` mentions power-ups and a "shooting star" asteroid type; these are not implemented in `game.js`.

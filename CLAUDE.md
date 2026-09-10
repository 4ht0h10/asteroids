# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file HTML5 Canvas clone of the arcade game *Asteroids*. No build step, no
dependencies, no framework, no test suite. All game logic lives in `game.js`
(~420 lines); `index.html` sets up an 800×600 canvas and loads the script.

## Running

Open `index.html` directly in a browser, or serve the folder: `npx serve .`
(then visit the printed localhost URL). There is nothing to build, lint, or test —
verify changes by reloading the page in the browser.

## Architecture

Everything is in `game.js`, structured as:

- **Entity classes** (`Bullet`, `Asteroid`, `Ship`, `Particle`) — each is a plain
  class with `update(dt)` and `draw()` methods and a `dead` boolean. They never
  reference each other; all interaction happens in the top-level `update(dt)`.
- **Module-level game state** — `ship`, `bullets`, `asteroids`, `particles`,
  `score`, `lives`, `level`, `state`, `deadTimer` are bare `let` variables reset
  by `initGame()` / `nextLevel()`. There is no game-object container or class.
- **`update(dt)`** — the single authority for the frame: advances entities,
  filters out `dead` ones, runs all collision checks (bullet↔asteroid,
  ship↔asteroid), applies scoring, and triggers level/life transitions.
- **`draw()`** — clears the canvas and paints particles → asteroids → bullets →
  ship → HUD → overlay, in that order.
- **`loop(ts)`** — `requestAnimationFrame` driver. `dt` is real elapsed seconds,
  clamped to 0.05 to survive tab-switch stalls. All motion is `dt`-scaled, so
  speeds/accelerations in the code are per-second.

### Key conventions

- **State machine**: `state` is `'playing' | 'dead' | 'gameover'`. `'dead'` is a
  2s respawn pause (`deadTimer`); asteroids keep moving but the ship is hidden.
  `'gameover'` waits for Space to call `initGame()`.
- **Toroidal space**: every entity position is passed through `wrap(v, max)`.
  Collision math uses raw distances and does *not* account for wrap-around.
- **Input**: `keys[code]` for held keys; `pressed(code)` for edge-triggered
  one-shots (consumes the event). Use `pressed` for shoot/restart, `keys` for
  thrust/rotate.
- **Asteroid sizes** are `1|2|3` (small→large) and index into the parallel
  arrays `RADII`, `SPEEDS`, `POINTS`. `split()` spawns two asteroids one size
  down; size 1 splits into nothing.
- **Spawning**: new asteroids from a split are collected into `newAsteroids`
  during the collision loop and concatenated after, never pushed mid-iteration.
- Rendering is all vector strokes on `ctx`; each `draw()` that transforms must
  balance `ctx.save()` / `ctx.restore()`.

## Note on the README

`README.md` describes power-ups and special asteroid types ("estrella fugaz")
that are **not** in `game.js`. Treat `game.js` as the source of truth.

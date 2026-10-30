# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

No build, lint, or test tooling exists (no `package.json`, no test files). The game is plain HTML/JS loaded directly in the browser.

- Run: open `index.html` in a browser, or serve the folder with `npx serve .` (serves on `http://localhost:3000`, per README).
- Verify changes by reloading the page in the browser. There is no automated test suite to run.

## Architecture

The whole game is one classic (non-module) script, `game.js`, loaded by `index.html` into a single 800×600 `<canvas id="canvas">`. Everything shares top-level `const`/`let` globals; there are no modules or imports.

**Game loop.** `loop(ts)` calls `update(dt)` then `draw()` each frame via `requestAnimationFrame`. `dt` is clamped to 0.05 s, so long pauses (tab switch) don't teleport objects.

**State machine.** The global `state` is one of `'playing' | 'dead' | 'gameover'`:
- `playing`: full update: input, ship, bullets, asteroids, particles, then collisions and level check.
- `dead`: ship is dormant for `deadTimer` (2 s) while asteroids and particles keep moving; then `ship.reset()` returns to `playing`. Ship gets 3 s of invincibility on reset.
- `gameover`: only particles update; Space calls `initGame()`.

Lives go down in `killShip()`, which sets either `dead` or `gameover`.

**Input.** `keydown`/`keyup` update `keys[e.code]` (held state). `pressed(code)` is an edge trigger: it returns true once per fresh press and then clears. Only Space uses it, for firing and restarting. Arrow keys are read as held state in `Ship.update`.

**Entities.** `Ship`, `Asteroid`, `Bullet`, and `Particle` are classes, each with `update(dt)` and `draw()`. Positions wrap with `wrap()`, so the play field is a torus; bullets wrap too.

**Asteroid sizes.** Sizes are 1 (small), 2 (medium), 3 (large). `RADII`, `SPEEDS`, and `POINTS` are indexed by size. Destroying a size-N asteroid spawns two of size N-1 at the same position; size 1 splits into nothing. Points are `POINTS[a.size]` (large 20, medium 50, small 100).

**Per-frame collision order** in `update()`:
1. Bullet vs asteroid (O(bullets × asteroids)): on hit, both die, score is added, an explosion is emitted, and children are created.
2. Ship vs asteroid: uses `radius + a.radius * 0.82` as the hit distance. Skipped while `ship.invincible > 0`.
3. Level clear: if no asteroids remain, `nextLevel()` increments `level` and spawns `3 + level` large asteroids at least 130 px from center.

**Rendering.** `draw()` clears to black, then draws particles, asteroids, bullets, ship, HUD, and the game-over overlay. Everything is stroked white vector paths, with no sprites or images. The ship flickers during invincibility by skipping `draw` on alternating intervals.

## Notes

- `README.md` is stale. It describes power-ups and a shooting star ("estrella fugaz") that were removed (commit `13e713f`). Trust `game.js` over the README for game behavior.
- UI strings and code comments are in Spanish (e.g., `PUNTAJE`, `ESPACIO PARA REINICIAR`, `// Disparar`). Match that when adding text.

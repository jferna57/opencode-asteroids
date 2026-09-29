# Asteroids - Agent Instructions

## What this is

- Vanilla HTML5 Canvas game in **3 files**: `index.html` (markup + all CSS, inline
  `<style>`), `game.js` (~420 lines, all game logic), `favicon.svg`.
- **No `package.json`, no bundler, no dependencies, no tests, no linter, no CI.**
  Don't add tooling, and don't go looking for `npm test` or a build step.
- README, code comments, and on-screen strings are all **Spanish**. Keep new
  comments and player-facing text in Spanish to match.

## Run / verify

- Open `index.html` by double-clicking. This works because `game.js` is loaded as a
  **classic script** (`<script src="game.js">`), not a module.
  - **Do not** switch it to `<script type="module">` — that breaks the documented
    double-click workflow, since `file://` blocks module loads (CORS).
- `npx serve .` → `http://localhost:3000` when a local server is needed.
- **There is no test suite.** The only verification is opening the page in a browser
  and playing: check the console for errors, confirm the ship responds to
  arrows/space, and confirm asteroids split and the level advances when cleared.

## Wiring you must keep in sync

- **Canvas size is duplicated.** `game.js` defines `const W = 800; const H = 600;`
  and `index.html` sets `width="800" height="600"`. These drive the toroidal wrap
  math — change one without the other and the world bounds go wrong.
- `game.js` is a single non-module script with no exports. All state is
  **module-scope mutable globals** (`ship, bullets, asteroids, particles, score,
  lives, level, state, deadTimer`); entity classes hold no reference to the arrays.
- Bootstrapping is manual: `initGame()` then `requestAnimationFrame(loop)` at the
  bottom of the file. A new entity type must be wired into **three** places: created
  in `initGame()` or its spawner, ticked in `update(dt)`, and drawn in `draw()`.

## Conventions

- **Removal is by dead flag, never `splice`.** Entities set `this.dead = true`; the
  owning `update()` does `arr = arr.filter(x => !x.dead)`.
- `ctx` is one module-global shared by every `draw()`. Each draw method sets every
  property it relies on and `ctx.save()`/`ctx.restore()` around transforms. Copy
  the `Asteroid`/`Ship` pattern; `Particle.draw()` is the exception that leaks.
- Units: px, `vx`/`vy` in px/s, `THRUST` in px/s², angles in rad/s, `dt` in seconds.
  Frame-rate independence matters — `loop()` clamps `dt` to `0.05`.
- Ship collision is only tested when `ship.invincible <= 0` and uses a forgiving
  asteroid radius (`a.radius * 0.82`). Asteroids never collide with each other.
- `keydown` calls `preventDefault()` only for Space + arrows. Adding a control means
  adding its code to that list, or the page will scroll.

## Gotchas

- **`pressed(code)` is a destructive one-shot read** — it returns the edge-triggered
  flag *and clears it*. Every state branch that needs an edge-triggered key must
  call it each frame. The `state === 'dead'` branch deliberately doesn't, so a
  Space press while dead stays buffered and fires the instant the ship respawns.
  A new state means deciding where that read happens.
- **The world is toroidal** — positions wrap via `wrap(v, max)`. But `dist(a, b)` is
  plain Euclidean, so collisions across the seam are wrong: a bullet can leave one
  edge and pass through an asteroid on the other. Known limitation, not something to
  re-derive. `Particle` deliberately does not wrap.
- **Asteroid size is an index, not a radius.** `RADII`, `SPEEDS`, and `POINTS` are
  parallel arrays indexed by `size` (1, 2, 3); index `0` is a dummy. Adding a size
  means extending all three, and `split()` hardcodes two children at `size - 1`.
- Spawners avoid `SAFE_DIST = 130` around the canvas center so the respawning ship
  isn't instantly hit. Keep this in anything new that spawns asteroids.

## README drift

- `README.md` advertises "power-ups especiales" and a "estrella fugaz" asteroid.
  These are **roadmap items that are not implemented** — no such code exists in
  `game.js`. Don't assume they exist; the README is not a spec of current behavior.

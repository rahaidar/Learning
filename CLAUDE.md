# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

GitHub: https://github.com/rahaidar/Learning

## Git workflow — required

Commit and push to GitHub after every meaningful unit of work. This is non-negotiable: it ensures no progress is ever lost and makes it easy to revert to any previous state.

- **Commit granularity:** one logical change per commit — a feature, a fix, a refactor. Don't batch unrelated edits.
- **Commit messages:** subject line explains *what and why* in plain English (e.g. `Fix runner enemies clipping through walls on spawn`). Add a short body if the change needs context.
- **Always push:** every local commit must be followed by `git push` before the task is considered done.
- **Format:**
  ```bash
  git add <specific files>
  git commit -m "Short imperative summary

  Optional body explaining motivation or trade-offs.

  Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
  git push
  ```

## Running the games

All games are self-contained HTML files — open directly in a browser, no build step or server needed:

```bash
start tictactoe.html
start shooter.html
```

To verify a change works, use Playwright (installed globally via `npx`):

```bash
cd /tmp && npm install playwright   # first time only
node -e "
const { chromium } = require('playwright');
(async () => {
  const b = await chromium.launch({ args: ['--no-sandbox'] });
  const p = await b.newPage();
  await p.goto('file:///c:/ClaudeCodeDrive/Learning/shooter.html');
  await p.waitForTimeout(800);
  await p.screenshot({ path: '/tmp/verify.png' });
  await b.close();
})();
"
```

Then read `/tmp/verify.png` to visually confirm the result.

## Architecture

Each game is a **single self-contained HTML file** with inline CSS and JS — no frameworks, no bundler, no dependencies.

### shooter.html

Canvas-based game loop using `requestAnimationFrame` with delta-time capping (`Math.min(dt, 0.05)`).

**State machine:** `MENU → PLAYING → LEVEL_COMPLETE → GAME_OVER`  
Global `state` variable (values in const `S`) gates all update and draw logic.

**Key globals:** `player`, `bullets`, `enemies`, `particles`, `spawnQueue`  
**Key functions:**
- `update(dt)` — dispatches to `updatePlaying` or ticks `levelTimer`
- `draw(dt)` — clears canvas, applies screen-shake transform, draws world, then HUD/overlays
- `levelStart(lvl)` — builds spawn queue from `LEVEL_DEFS`, resets wave counters
- `buildQueue(def)` — creates and shuffles the enemy spawn array

**Enemy types** (`ETYPES`): `walker` (red, straight), `runner` (yellow, zigzag AI), `tank` (purple, armored). All spawn from random screen edges; collision radii are stored on each entity object.

**Rendering:** all sprites are drawn procedurally with Canvas 2D API (`fillRect`, `arc`, `ellipse`). No image files.

### tictactoe.html

Simple event-driven game. State lives in `board[]`, `current`, `gameOver`, and `scores`. No game loop — redraws are triggered by click events only.

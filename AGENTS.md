# AGENTS.md

Pac-Man clone built with the spec-driven method. Vanilla JS + HTML + CSS, canvas-based. No package manager, no dependencies, no build step, no tests, no CI.

## Workflow (the point of this repo)

- Features go through the repo-local skills `.agents/skills/spec` (`/spec`) and `.agents/skills/spec-impl` (`/spec-impl`) — load them with the skill tool. Do not jump straight to code for non-trivial features.
- Specs live in `specs/NN-slug.md` (zero-padded, sequential). `/spec-impl` only proceeds if the spec state means Approved (`Aprobado` accepted). Implementation happens on branch `spec-NN-slug`.
- **Never commit automatically** during spec implementation; the user commits. Pause after each plan step for diff review.

## Language

The project is Spanish: README, UI strings (`src/index.html`), code comments, and specs are all in Spanish. Match the language of the user's prompt in replies; write code comments and specs in Spanish to stay consistent.

## Running / verifying

No toolchain. Open `src/index.html` directly in a browser, or serve statically: `python3 -m http.server -d src 8000`. Verification is manual play in the browser.

## Architecture gotchas

- Plain `<script>` tags, no modules. Load order in `index.html` is fixed and meaningful: `maze.js` → `game.js` → `render.js` → `main.js`. Files communicate through globals (`MAZE`, `createGame`, `update`, `draw`, ...). Renaming or adding files requires editing the script tags.
- Grid tile codes: `1` wall, `2` dot, `0` walkable empty, `3` pen door. The maze is authored as readable strings in `MAZE_STR` (28×31) and parsed to numbers.
- `MAZE` is pristine and must never be mutated; a game copies it into `game.grid` (`createGame` in `game.js`). Render from `game.grid`, not `MAZE`.
- Movement uses fractional cell coordinates; alignment math (`aligned()` in `game.js`) gates direction changes. `PACMAN_SPEED = 0.125` aligns every 8 frames — changing speeds can break alignment.

## Code style

- 2-space indent, single quotes, semicolons, spaces inside brackets: `grid[ y ][ x ]`, `KEY_DIR[ e.key ]`.
- Arrow functions for callbacks; file-header comments (in Spanish) state each file's dependencies on globals from other files.

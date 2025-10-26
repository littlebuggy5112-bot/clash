## Purpose

This file gives focused, actionable guidance for AI coding agents editing this repository. The app is a single-file browser game built in plain HTML/CSS/JS (`index.html`). Read this before making code changes.

## Big picture

- Single-page game: all UI, styles and game logic live inline in `index.html`. The central runtime object is the `Game` class instantiated at the bottom of the file (`const game = new Game();`).
- Main responsibilities: menu/auth (localStorage), deck selection, game startup, game loop (units array, towers state), AI opponent, spells, and UI updates.
- Data flows: user interactions update DOM -> `Game` methods mutate JS state (`this.units`, `this.towers`, `this.elixir`, etc.) -> DOM updates (create/remove elements, update health bars). Persistent data (users/trophies) is stored in localStorage keys `users` and `currentUser`.

## Quick run / dev workflow

- No build step. Open `index.html` in a modern browser. For a local server (recommended for external images) run a lightweight server. Example (PowerShell):

```powershell
python -m http.server 8000
# or
npx http-server -p 8000
```

- Debugging: use Browser DevTools. Breakpoints inside `Game` methods (e.g. `init`, `startGame`, `updateUnits`) are the fastest way to inspect runtime state.

## Important project-specific conventions & patterns

- Game class is the single source of truth. Edit logic inside methods rather than adding global functions.
- Card definitions live in `this.allCards` on `Game` (in `index.html`). Each entry is a small map: { name, cost, health, damage, speed, range, icon, type, ... }.
  - To add/change a card, update `this.allCards` and `setupMenu()` will render it into the card selection grid.
  - Some cards include `src` to render an image instead of an icon.
- Drag & drop: player card play uses HTML5 drag-and-drop. Cards in the bottom `#cardDeck` are `.card` elements with `data-card` and `data-index`. Grid cells are `.grid-cell` and use `data-row`/`data-col`. Placement rules are enforced in `drop` handlers (troops only allowed on rows >= 4, spells anywhere).
- Grid coordinates: `gridSize` (rows x cols) and `cellSize` determine positioning. Position math uses column*cellWidth + cellWidth/2 for X and similarly for Y.
- River/bridges: pathfinding takes river boundaries (`this.riverTop`/`riverBottom`) and `this.bridges` into account in `calculateMovement()`; special-case units like `hogrider` can jump the river.
- IDs & DOM references: many methods query specific element IDs/classes (e.g., `#arena`, `.tower.player`, `.tower.enemy`). Keep these IDs/classes intact when refactoring DOM structure.

## Hotspots (important functions to inspect/edit)

- Authentication and persistence: `setupAuth()`, `loadUserData()`, `saveUserData()` — localStorage keys: `users`, `currentUser`.
- UI/menu: `setupMenu()`, `setupEventListeners()` (menu tabs, card selection UI).
- Game lifecycle: `startGame()`, `startGameLoop()`, `startElixirGeneration()`, `startAI()`, `startTimer()`.
- Unit flow: `spawnUnit()`, `createUnitElement()`, `updateUnits()`, `attack()`, `removeUnit()`.
- Spells & effects: `castSpell()`, `pullUnitsToTornado()`, `dealSplashDamage()`, `createMegaKnightSpawnEffect()`.
- Pathfinding / movement: `calculateMovement()`, `findNearestBridge()`, `findTarget()`.

## Known gotchas & places to be careful

- Single-file complexity: all code is inline. Small edits can have big side-effects—run the game manually after changes.
- Duplicate/overlapping card keys: `this.allCards` contains repeated or inconsistent keys (e.g., multiple `minion` definitions). Search for collisions before adding a new key. Prefer unique keys.
- Magic numbers: many timing and size values are hard-coded (cell sizes, elixir tick rate, timer durations). If extracting constants, keep names and usages consistent.
- Intervals: game uses several setInterval timers (`startElixirGeneration`, `startGameLoop`, `startAI`, `startTimer`). Ensure intervals are cleared on game over or when restarting to avoid duplicate loops during rapid edits/tests.
- Units reference DOM elements by generated IDs (`unit-${unit.id}`). Unit IDs are generated with `Date.now() + Math.random()`; avoid assuming sequential IDs in tests.

## Editing rules for AI agents

- Preserve DOM IDs and element classnames unless you update all selectors that reference them.
- When changing a game mechanic (e.g., elixir rate, card cost), update both `this.allCards` and UI rendering (`setupMenu()` / `createPlayerDeck()`), then test playthroughs (quick match and card placement).
- Small, focused patches are preferred. Run the page in-browser to verify the interactive behavior (create account, select deck, start quick match). Manual verification steps are minimal and quick.

## Suggested quick manual tests (smoke tests)

1. Open `index.html` in browser (or via local server). Create an account (creates `users` in localStorage). Sign in.
2. Use Quick Match: click Quick Match and confirm the arena appears, elixir bar and timer start, and enemy spawns occur.
3. Drag a card from the bottom deck to a bottom-half grid cell and observe a unit spawn and elixir decrease.
4. Cast a spell (if available) and verify area effects and damage show visually.
5. After the match ends, confirm `users[currentUser].trophies` is updated in localStorage.

## If you need to refactor

- Move logic into separate JS modules only if you update the HTML to include the new script and preserve initialization (`const game = new Game();`).
- Run the app each step. Prefer incremental commits that keep the app runnable.

---

If anything above is unclear or you want more details (example call stacks, recommended unit tests, or a suggested small refactor to extract `this.allCards` into its own file), tell me which area to expand and I will iterate.

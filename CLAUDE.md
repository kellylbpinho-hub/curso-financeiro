# CLAUDE.md — curso-financeiro

## Project Overview

A single-file React SPA delivering a financial education course in Brazilian Portuguese. Targets business professionals learning financial metrics (margins, ROI, KPIs, etc.) through a Kumon-based learning methodology. No build step, no server, no npm — open `index.html` in a browser and it works.

## File Structure

```
curso-financeiro/
├── index.html          # Entire application (React components, data, styles)
└── guardioes/          # Character illustration assets
    ├── KAI.png
    ├── KAME.png
    ├── KITSUNE.png
    ├── KUMA.png
    ├── NEKO.png
    ├── RAIU.png
    ├── RYU.png
    ├── SENSEI.png
    ├── TAKA.png
    └── TSURU.png
```

Everything lives in `index.html`. There are no separate JS, CSS, or config files.

## Tech Stack

| Concern | Solution |
|---------|----------|
| UI framework | React 18.2.0 (CDN, UMD build) |
| JSX compilation | Babel Standalone 7.23.2 (runtime, in-browser) |
| Styling | Inline `style={{}}` props only |
| State persistence | Browser `localStorage` |
| Assets | Local PNG files in `guardioes/` |
| Build tool | None |
| Package manager | None |
| TypeScript | None |
| Tests | None |
| Linting | None |

Dependencies are loaded via `<script src="https://cdnjs.cloudflare.com/...">` tags. All code lives inside a single `<script type="text/babel">` block.

## How to Run

```bash
# Option 1: open directly
open index.html

# Option 2: any static server
python3 -m http.server 8000
# then visit http://localhost:8000
```

No installation, build, or configuration required.

## Architecture

### React Components

**`App`** — root component, owns all state, handles tab/view routing  
**`LessonView`** — renders lesson content, prev/next navigation between lessons  
**`ExercisePanel`** — renders all 3 exercise types, validates answers, tracks completion

Navigation is state-driven (no router):
- `tab`: `"k"` (modules kanban) | `"km"` (methodology view)
- `view`: `"list"` | `"lesson"` | `"exercises"`
- `act`: currently open module object

### Key Global Constants

```js
ST        // status tokens: { B: "backlog", D: "doing", X: "done" }
MODULES   // array of 11 module objects
KUMON_STEPS // array of 9 step objects for the learning methodology
DAILY     // array of 5 daily checklist items
```

### Module Object Shape

```js
{
  id: "m1",
  title: "Proporções e Percentuais",
  subtitle: "...",
  description: "...",
  emoji: "📊",
  color: "#E8773A",
  bg: "#FFF3EC",
  lessons: [
    { title: "...", content: "..." },  // 3 lessons per module
    // ...
  ],
  exercises: [
    { type: "quiz",  question: "...", options: [...], correct: 0, explanation: "..." },
    { type: "tf",    statement: "...", correct: true,  explanation: "..." },
    { type: "calc",  question: "...", hint: "...",     answer: 25, tolerance: 0.5, explanation: "..." },
  ]
}
```

## State Management

All state lives in `App` using `useState` + `localStorage` persistence via custom setter wrappers:

```js
// Pattern: wrap setXxx to auto-persist
const [mSt, setMStRaw] = useState(() => loadStorage("cfin_mod", defaultValue));
const setMSt = (fn) => {
  const nv = typeof fn === "function" ? fn(mSt) : fn;
  setMStRaw(nv);
  saveStorage("cfin_mod", nv);
};
```

| State var | localStorage key | Description |
|-----------|-----------------|-------------|
| `mSt` | `cfin_mod` | Module status map `{ m1: "backlog" \| "doing" \| "done", ... }` |
| `sSt` | `cfin_steps` | Kumon step status map |
| `dc` | `cfin_dc` | Daily checklist map `{ item_id: true/false }` |
| `streak` | `cfin_streak` | Number of consecutive study days |

Helper functions at the top of the script:

```js
loadStorage(key, fallback)  // JSON.parse from localStorage, returns fallback on error
saveStorage(key, val)       // JSON.stringify to localStorage, silently catches errors
```

## Naming Conventions

**Constants**: `UPPER_SNAKE_CASE` (`MODULES`, `KUMON_STEPS`, `ST`)  
**Functions**: `camelCase` (`handleQuiz`, `loadStorage`, `setMSt`)  
**State hooks**: descriptive short names (`mSt`, `sSt`, `dc`, `tab`, `view`, `act`)  
**Local variables**: abbreviated (`gM` = grouped modules, `gS` = grouped steps, `nr` = new results, `na` = new answered, `nv` = new value, `ln` = line, `ex` = exercises, `cur` = current)

## Styling

All styles are inline `style={{}}` props. There are no CSS files or `<style>` blocks for component styles. Exception: two CSS keyframe animations defined in a `<style>` tag in `<head>`:

- `bounce-in` (0.35 s scale animation, used on correct answers)
- `shake` (0.4 s horizontal shake, used on wrong answers)

### Color Palette

| Token | Hex | Usage |
|-------|-----|-------|
| Primary orange | `#E8773A` | Buttons, highlights, module 1 |
| Green | `#2D7A4F` | Success, done status |
| Error red | `#E84545` | Wrong answers |
| Background | `#F2EDE8` | App background |
| Dark text | `#1A1A1A` | Body text |
| Medium text | `#666` | Secondary text |
| Light text | `#AAA` / `#CCC` | Disabled / placeholder |

Each module object carries its own `color` and `bg` values.

## Course Content Structure

- **11 modules** (10 core + 1 bonus on taxation)
- **3 lessons per module** (progressive depth)
- **3 exercises per module**: quiz → true/false → calculation

### Exercise Validation Logic

```js
// Quiz: match option index
correct === selectedIndex

// True/False: boolean match
correct === true | false

// Calculation: numeric with tolerance
Math.abs(parseFloat(userInput) - answer) <= tolerance
```

Exercises track state per-question inside `ExercisePanel` via local `useState`:
- `results[]` — per-question result (null | true | false)
- `answered[]` — whether the question has been submitted
- `inputs[]` — current text input for calc exercises

## Common Tasks

### Add a new module

1. Append a new object to the `MODULES` array following the existing shape
2. Choose a unique `id` (e.g., `"m12"`)
3. Write 3 lessons (`title` + `content` string, use `\n` for line breaks)
4. Write 3 exercises (one of each type)
5. The kanban and progress tracking pick it up automatically

### Modify lesson content

Edit the `content` string inside the relevant `lessons[n]` object in `MODULES`. Use `\n` for paragraph breaks — the renderer splits on newlines.

### Add a daily checklist item

Append to the `DAILY` array:
```js
{ id: "d6", text: "Nova tarefa diária" }
```

### Change colors

Update the `color` and `bg` fields on the relevant module object. For global palette changes, search for the hex value and update all occurrences.

## Manual Testing

There are no automated tests. Test changes by:

1. Opening `index.html` in a browser
2. Completing the golden path: move a module Backlog → Doing → Done
3. Open lessons, navigate prev/next, mark complete
4. Open exercises, answer each type correctly and incorrectly
5. Check that localStorage persists state on page reload (`cfin_mod`, `cfin_steps`, `cfin_dc`, `cfin_streak` in DevTools → Application → Local Storage)
6. Verify daily checklist increments streak on next-day simulation (change system date or temporarily alter the date check logic)

## Git Workflow

The project uses standard git. Commit messages should describe what changed in plain language. There is no CI, no pre-commit hooks, and no branch protection configured.

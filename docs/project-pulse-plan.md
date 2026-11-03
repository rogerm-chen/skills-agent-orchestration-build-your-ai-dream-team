# Project Pulse implementation plan

## 1. Summary

Project Pulse is a small static dashboard for Mona's team. It shows project cards with each project's name, owner, status, recent activity and priority, plus a short summary for contributors. It uses three files in `app/` and one VS Code launch configuration called **Run Project Pulse Dashboard**. That configuration serves `app/` with `python3 -m http.server 5500` and opens `http://localhost:%s/index.html`, so learners see the dashboard rather than a directory listing.

Facts from the repo that shape this plan:
- `app/` is empty and `.vscode/` contains only `tasks.json`, which must not be changed.
- The Step 3 workflow (`.github/workflows/3-step.yml`) only checks that files exist, that some exact phrases appear, and that two files parse as JSON:
  - **`app/index.html`** must contain `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
  - **`app/styles.css`** must contain `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
  - **`app/project-data.json`** must contain `projects`, `name`, `owner`, `status`, `recentActivity` and `priority`, and must pass `python3 -m json.tool`.
  - **`.vscode/launch.json`** must contain `Run Project Pulse Dashboard` and `index.html`, and must pass `python3 -m json.tool`.
- The brief limits the app to exactly three files. All JavaScript therefore goes inline in `app/index.html`; there is no separate `app.js`.

Ownership is split so that no two agents edit the same file. The **Designer** owns the visual layer (`app/styles.css`). The **Coder** owns markup and logic (`app/index.html`), data (`app/project-data.json`) and the runnable setup (`.vscode/launch.json`). A shared contract (section 6) fixes the data format and CSS class names up front, so both agents can work in parallel safely.

## 2. File assignments

| File | Owner | What it must contain |
|---|---|---|
| `app/index.html` | **Coder** (follows the markup/class contract; Designer reviews) | `<!DOCTYPE html>`, `<html lang="en">`, `<meta charset="utf-8">` and a viewport meta tag. `<title>Project Pulse</title>` and a visible `<h1>Project Pulse</h1>`. `<link rel="stylesheet" href="styles.css">`. A `<main class="dashboard">` wrapper with a header, a summary strip, a status area marked `aria-live="polite"`, and a card grid. An inline `<script>` that runs `fetch("project-data.json")`, checks the response, reads `data.projects` and renders one `<article class="project-card">` per project. Each card shows name, owner, status (as a badge), recentActivity, priority (as a badge) and the optional summary. Loading, empty and error states. All values inserted with `textContent` (never `innerHTML` with data). The literal field names `status`, `recentActivity` and `priority` appear in the script. |
| `app/styles.css` | **Designer** | `.dashboard` and `.project-card` selectors. Every class in the contract is styled. Uses `border-radius`, `box-shadow`, CSS custom properties for colours/spacing, a responsive grid (`repeat(auto-fill, minmax(280px, 1fr))` or breakpoints), status badge variants, priority treatment, a visible `:focus-visible` style, `prefers-reduced-motion` handling, long-text wrapping (`overflow-wrap: anywhere`), and text contrast of at least 4.5:1. |
| `app/project-data.json` | **Coder** | Strict JSON with a top-level `"projects"` array of 5–6 sample projects. Each project has `name`, `owner`, `status`, `recentActivity` and `priority`, plus optional `summary` and `lastUpdated` (an ISO date string). Uses only status and priority values from the contract. Includes at least one example of each priority and most statuses, plus one long project name to test wrapping. |
| `.vscode/launch.json` | **Coder** | Strict JSON with no comments and no trailing commas. `"version": "0.2.0"` and a single configuration named exactly `Run Project Pulse Dashboard`. Recommended shape: `"type": "node"`, `"request": "launch"`, `"cwd": "${workspaceFolder}/app"`, `"runtimeExecutable": "python3"`, `"runtimeArgs": ["-m", "http.server", "5500"]`, `"env": {"PYTHONUNBUFFERED": "1"}`. Add `"serverReadyAction": {"pattern": "Serving HTTP on .* port ([0-9]+)", "uriFormat": "http://localhost:%s/index.html", "action": "openExternally"}`. |

Out of scope: no other files are created or changed. That includes `.vscode/tasks.json`, `.github/**` and `scripts/**`.

## 3. Responsibilities

### Designer responsibilities
- Own and write `app/styles.css` only.
- Make the first view clearly a polished Project Pulse dashboard, not a bare page:
  - a header with the title and a short tagline
  - a summary strip with counts by status
  - a card grid
- Cards:
  - rounded corners, soft `box-shadow`, readable padding, and a consistent vertical rhythm
  - a slight hover/focus lift that is turned off under `prefers-reduced-motion`
- Status badges: a distinct colour per status, always paired with the text label so colour is never the only signal.
- Priority treatment:
  - a priority badge
  - a coloured left accent on the card, using classes such as `.project-card--priority-high`
  - stronger visual weight for `high`
- Responsive layout: one column at ≤600px, two at roughly 600–1024px, three or more at wider sizes. No horizontal scrolling at 320px.
- Typography and accessibility: a system font stack, a clear heading scale, body text of at least 16px, contrast of at least 4.5:1 for text and 3:1 for badge borders and accents, and a visible focus ring.
- Style the loading, empty and error message states.
- After the Coder finishes `app/index.html`, review the rendered page. Report any markup changes to the Orchestrator instead of editing `index.html`.

### Coder responsibilities
- Own and write `app/project-data.json`, `app/index.html` and `.vscode/launch.json`.
- Data: valid JSON that follows the schema in section 6, with realistic, contributor-friendly sample content.
- `index.html`:
  - semantic markup using only the contract class names
  - inline JS: fetch, check `response.ok`, check that `projects` is an array, normalise status and priority, then render cards
  - handle loading, empty and error states
  - insert all data with `textContent`
- Launch config: deterministic name, port (5500), working folder (`cwd`) and URL, as described in section 2. Strict JSON.
- Run the validation commands in section 8 and report the results and any remaining risks.

## 4. Ordered implementation steps

1. **Phase 0 – Contract (Orchestrator).** Lock the shared contract in section 6 and include it in both delegation prompts.
2. **Phase 1 – Parallel build:**
   - 2a. **Designer:** create `app/styles.css` against the contract's class names.
   - 2b. **Coder:** create `app/project-data.json`, then `app/index.html`, then `.vscode/launch.json`. These are sequential within the Coder's own work.
3. **Phase 2 – Integration review (sequential).**
   - The Designer reviews the rendered page (served from `app/` on port 5500) at mobile, tablet and desktop widths and lists any problems.
   - If markup changes are needed, the Orchestrator sends them to the **Coder** for `app/index.html`. CSS-only fixes go to the **Designer** for `app/styles.css`. Never assign both agents to the same file in the same phase.
4. **Phase 3 – Validation (Orchestrator, with Coder running the commands).** Run the checklist in section 8 and fix any failures through the file's owner.
5. **Phase 4 – Hand off to the learner.** Report the changed files. The learner stages, commits and pushes; no agent touches git.

## 5. Dependencies

- `app/index.html` depends on:
  - the data format in `app/project-data.json` (field names, the `projects` key, allowed status and priority values)
  - the class names agreed with `app/styles.css` (`.dashboard`, `.project-card`, badge modifier classes and so on)
- `app/styles.css` depends on the class contract only, not on the finished `index.html`.
- `.vscode/launch.json` depends on the folder layout (`app/index.html` existing for the `uriFormat` target) and on nothing else. It can be written at any time but can only be tested end to end once `index.html` and `project-data.json` exist.
- The Phase 2 visual review depends on all three app files existing.
- Validation (Phase 3) depends on all four files.

## 6. Parallel work decisions

**Shared contract (fixed before Phase 1; nobody changes it without the Orchestrator's approval):**

*Data format (`app/project-data.json`):*
```text
{ "projects": [ {
    "name": string (required),
    "owner": string (required),
    "status": "active" | "at-risk" | "blocked" | "paused" | "completed" (required),
    "recentActivity": string, one sentence (required),
    "priority": "high" | "medium" | "low" (required),
    "summary": string (optional),
    "lastUpdated": "YYYY-MM-DD" (optional)
} ] }
```

*CSS class names (used by `index.html`, styled by `styles.css`):*

| Class | Purpose |
|---|---|
| `.dashboard` | `<main>` wrapper |
| `.dashboard-header`, `.dashboard-title`, `.dashboard-subtitle` | Page header |
| `.dashboard-summary`, `.summary-item` | Counts by status |
| `.dashboard-message`, `.dashboard-message--error`, `.dashboard-message--empty` | Loading, error and empty states |
| `.project-grid` | Card container (`<section>` or `<ul>`) |
| `.project-card`, `.project-card--priority-high` / `--priority-medium` / `--priority-low` | Card and its priority accent |
| `.project-name` (h2), `.project-owner`, `.project-summary`, `.project-activity`, `.project-meta`, `.project-updated` | Card contents |
| `.status-badge`, `.status-badge--active` / `--at-risk` / `--blocked` / `--paused` / `--completed` / `--unknown` | Status badge |
| `.priority-badge`, `.priority-badge--high` / `--medium` / `--low` / `--unknown` | Priority badge |
| `.visually-hidden` | Labels for screen readers only |

**Can run in parallel (Phase 1):**
- **Designer on `app/styles.css`** and **Coder on `app/project-data.json`, `app/index.html` and `.vscode/launch.json`**. Their files don't overlap, and the only link between them (class names) is fixed by the contract above.

**Must be sequential:**
- Within the Coder's work, `project-data.json` comes before `index.html`, because the rendering code uses the actual data and sample values. The same agent owns both, so the order is natural.
- The Phase 2 Designer review comes after Phase 1, because it needs the rendered page.
- Any `index.html` fixes from the review come after the review and go to the Coder only, so two agents never edit the same file.
- Validation comes last, because it needs all files.
- Git operations are done by the learner after validation, never by an agent.

## 7. Edge cases

- **Opening the page from a `file://` path:** `fetch("project-data.json")` is blocked by the browser. Catch the error and show a `.dashboard-message--error` that says to use **Run Project Pulse Dashboard** or `python3 -m http.server 5500` from `app/`. Don't fail silently.
- **Failed request or invalid JSON:** check `response.ok`. If it isn't OK, or JSON parsing fails, show the error state with the HTTP status if available. Log details to `console.error`.
- **Missing or empty data:** if `projects` is missing, not an array, or empty, show the `.dashboard-message--empty` text "No projects to show yet." and set summary counts to 0.
- **Missing fields:** fall back to "Unassigned" for owner, "No recent activity" for recentActivity and "Untitled project" for name. Never render the text `undefined`.
- **Unknown or oddly formatted status/priority:** trim, lowercase and replace spaces with `-`. If the result isn't in the allowed list, use the `--unknown` modifier but still show the original text.
- **Long names, owners or activity text:** wrap with `overflow-wrap: anywhere` and keep the card height flexible. The sample data includes one long name.
- **HTML in data values:** insert with `textContent` only, so data is never treated as markup.
- **Port 5500 already in use:** the server fails to start and nothing opens. Document this as a known limit and ask the learner to stop the earlier preview. The port stays fixed so it is predictable.
- **Python output delay:** without unbuffered output, the "Serving HTTP" line may not show up in time and the browser never opens. `PYTHONUNBUFFERED=1` in `env` prevents this.
- **Codespaces:** `localhost:5500` is forwarded automatically; `openExternally` opens the forwarded URL.
- **Directory listing:** avoided by serving from `app/` and opening `/index.html` explicitly.

## 8. Validation expectations

Run from the repo root:

1. **Files exist:** `app/index.html`, `app/styles.css`, `app/project-data.json`, `.vscode/launch.json`.
2. **Valid JSON:**
   - `python3 -m json.tool app/project-data.json >/dev/null`
   - `python3 -m json.tool .vscode/launch.json >/dev/null`
3. **No comments in launch.json:** `grep -nE '^\s*(//|/\*)|\*/' .vscode/launch.json` returns nothing. A plain `//` search is not enough because URLs contain `//`. Passing `json.tool` also rules out comments.
4. **Launch content:**
   - the name is exactly `Run Project Pulse Dashboard`
   - `cwd` is `${workspaceFolder}/app`
   - `runtimeArgs` are `-m http.server 5500`
   - `uriFormat` is `http://localhost:%s/index.html`
   - `grep -c index.html .vscode/launch.json` is at least 1
5. **Data format:** a Python one-liner confirms that `projects` is a non-empty list and that every item has `name`, `owner`, `status`, `recentActivity` and `priority`, with status and priority taken from the contract lists.
6. **Required phrases in `index.html`:** `grep` finds `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
7. **Required phrases in `styles.css`:** `grep` finds `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
8. **Runtime render:**
   - Run `cd app && python3 -m http.server 5500` in the background.
   - `curl -sf http://localhost:5500/index.html | grep -q "Project Pulse"` succeeds.
   - `curl -sf http://localhost:5500/project-data.json` returns 200.
   - In a browser, one `.project-card` appears per project and there are no console errors.
   - Stop the server afterwards.
9. **Run from VS Code:** in Run and Debug, start **Run Project Pulse Dashboard**. The browser opens `/index.html` and shows the dashboard, not a listing.
10. **Responsive check:** at 375px, about 768px and about 1280px widths, the layout is one, two and three columns respectively, with no horizontal scrolling and no clipped long name.
11. **Accessibility basics:**
    - `lang="en"` is set
    - there is a single `<h1>`, and each card has a heading
    - badges show text labels (colour is not the only signal)
    - the status area uses `aria-live="polite"`
    - text contrast is at least 4.5:1
    - focus is visible
    - reduced motion is respected
12. **Error state:** opening `app/index.html` via `file://`, or temporarily renaming the data file, shows the friendly error message.
13. **Scope check:** `git status` shows only the four files above, plus `docs/project-pulse-plan.md`.

**Do not use `scripts/validate-exercise.sh` as a pass/fail check.** It is a template-maintenance script. It deliberately fails once learner files such as `app/*` are tracked in git, and it runs Ruby/YAML checks that are unrelated to this work.

## 9. Open questions

1. **Fallback when opened via `file://`:** should `index.html` include a built-in copy of the data so it still renders from a `file://` path? This plan says no, to avoid keeping two copies of the data in sync, and shows an error message instead.
2. **Launch debugger type:** this plan uses `"type": "node"` with `python3` as the runtime. It relies on VS Code's built-in JavaScript debugger and has not been tested in this environment. The alternative, `"type": "debugpy"` with `"module": "http.server"`, needs the Python extension, which is not in the devcontainer's extension list. Confirm which one the team prefers.
3. **Field formats:** should `recentActivity` stay a plain sentence, or become structured `{ "date", "description" }`? The contract keeps it a string to match the brief's field list exactly.
4. **Status list:** is `active | at-risk | blocked | paused | completed` the right set for Mona's team? It can be changed, but only before Phase 1 starts.
5. **Extra UI:** should a light/dark theme (`prefers-color-scheme`) or filter/sort controls be included? This plan leaves them out to keep scope small.

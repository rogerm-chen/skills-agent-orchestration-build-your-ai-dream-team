# Project Pulse final handoff

Project Pulse is a small static dashboard for Mona's team. It shows one card per project with its name, owner, status, recent activity and priority, plus counts by status. This document sums up who built what, how to run it, what was validated and what the learner should do next.

## Agent team and roles

| Agent | Role in this build |
|---|---|
| Orchestrator | Coordinated the phases (contract → parallel build → review → validation → handoff) and gave each specialist an explicit, non-overlapping file scope. Did not implement. |
| Planner | Wrote `docs/project-pulse-plan.md`: file assignments, shared data/CSS class contract, edge cases and validation checklist. Wrote no code. |
| Designer | Owned `app/styles.css`: layout, cards, status and priority badges, responsive grid, focus and reduced-motion styles. |
| Coder | Owned `app/index.html`, `app/project-data.json` and `.vscode/launch.json`, and ran the validation checks below. |

## Files delivered

- `app/index.html` – Page markup and an inline script. The script fetches `project-data.json`, renders one `.project-card` per project using `textContent` only, updates the status summary and handles loading, empty and error states.
- `app/styles.css` – Visual design. Includes `.dashboard` and `.project-card` styles, rounded cards with `box-shadow`, status/priority badge variants, a 1/2/3-column responsive grid, `:focus-visible`, `prefers-reduced-motion` and `forced-colors` support.
- `app/project-data.json` – Sample data: a top-level `projects` array with 6 projects. Covers every status, every priority and one long project name.
- `.vscode/launch.json` – Strict JSON with one launch configuration, "Run Project Pulse Dashboard". It serves `app/` and opens the dashboard.

## How to run

1. In VS Code, open **Run and Debug** and choose **Run Project Pulse Dashboard**.
2. This runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`. When the server prints "Serving HTTP", VS Code opens http://localhost:5500/index.html (forwarded automatically in Codespaces).
3. Fallback: run the server yourself from `app/`, then open http://localhost:5500/index.html:

   ```bash
   cd app
   python3 -m http.server 5500
   ```

Opening `index.html` directly as a `file://` path shows a friendly error message because the browser blocks `fetch` there. Use one of the methods above instead.

## Dashboard validation results

The Coder ran these checks in the Codespace on 2026-11-03.

| # | Check | Result |
|---|---|---|
| 1 | All four files exist | PASS |
| 2 | `python3 -m json.tool` on `app/project-data.json` and `.vscode/launch.json` | PASS (both) |
| 3 | No comments in launch.json (`grep -nE '^\s*(//\|/\*)\|\*/'` finds nothing) | PASS |
| 4 | Launch name is exactly "Run Project Pulse Dashboard"; `cwd` `${workspaceFolder}/app`; `runtimeArgs` `["-m","http.server","5500"]`; `uriFormat` `http://localhost:%s/index.html` | PASS |
| 5 | Data format: `projects` is a non-empty list (6 projects), all required fields are present, status and priority use allowed values | PASS |
| 6 | `app/index.html` contains `<title>Project Pulse</title>`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority` | PASS |
| 7 | `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, `@media` | PASS |
| 8 | Every class used in `app/index.html` (markup and script, including dynamic badge/priority modifiers) has a selector in `app/styles.css` | PASS (33 of 33) |
| 9 | Step 3 workflow (`.github/workflows/3-step.yml`) checks run locally: files exist, keyphrases, JSON parsing | PASS |
| 10 | Runtime: `python3 -m http.server 5500` from `app/`: `index.html` 200 and contains "Project Pulse"; `project-data.json` 200; `styles.css` 200; server stopped afterwards | PASS (port 5500 was free) |
| 11a | `node --check` on the extracted inline script | PASS |
| 11b | jsdom render with the real data: 6 `.project-card` elements for 6 projects, summary total 6, no script errors, no "undefined" text | PASS |
| 11c | jsdom error state (fetch rejected) and empty state (`projects: []`) show the right messages | PASS |
| – | Launch configuration started from VS Code Run and Debug | NOT RUN |
| – | Real-browser visual check at mobile/tablet/desktop widths | NOT RUN |
| – | Measured colour contrast ratios | NOT RUN (only estimated) |
| – | Designer review of the rendered page (plan Phase 2) | NOT RUN (no record that it happened) |

### Differences from the plan

- The build follows the plan's contract: field names, status/priority values, class names, breakpoints at 600px and 1024px, and inline JS with no extra files.
- `.vscode/launch.json` adds `"console": "integratedTerminal"`, which the plan didn't mention. This is harmless.
- The plan's Phase 2 Designer review and its manual checks (VS Code launch, responsive widths, accessibility, contrast) have not been recorded as done.
- The plan's scope check expected uncommitted changes. The learner had already committed the build ("Build the Project Pulse dashboard"), so the working tree was clean before this document was added.

## Learner handoff and next steps

### What to do next

1. Open the dashboard with **Run and Debug → Run Project Pulse Dashboard** and confirm that the dashboard opens, not a directory listing.
2. Check the layout in browser dev tools at about 375px (mobile, 1 column), 768px (tablet, 2 columns) and 1280px (desktop, 3 columns). Look for horizontal scrolling or a clipped long project name.
3. Review `docs/final-handoff.md`, then stage, commit and push it yourself through Copilot CLI prompts.

### Known risks and open questions

- **Launch type:** the launch configuration uses `"type": "node"` with `python3` as the `runtimeExecutable`. This relies on VS Code's built-in JavaScript debugger and was **untested in VS Code here**. If it fails, use the manual fallback above or switch to a `debugpy` configuration, which needs the Python extension.
- **Port 5500 conflicts:** if another server (for example, Live Server or an earlier preview) is already using 5500, the server won't start and no browser opens. Stop the other process and run the configuration again.
- **Contrast:** contrast ratios were **estimated, not measured**. Check them with browser dev tools or an accessibility checker.
- **No real-browser check yet:** rendering was only checked with curl and jsdom. No one has looked at the page in a real browser yet.
- **Data duplication:** there is no built-in copy of the data for `file://` use. This is by design, and the page shows an error message instead.

### Git operations

The agents did not stage, commit or push anything on their own. All git operations are the learner's to perform.

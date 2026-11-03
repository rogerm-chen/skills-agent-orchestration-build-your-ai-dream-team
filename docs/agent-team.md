# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate a team of custom agents that will build Mona's Project Pulse dashboard.

| Agent | Target model | Responsibility | Definition |
|---|---|---|---|
| Orchestrator | Claude Opus 5.5 (copilot) | Breaks down requests, delegates to specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, and verifies the integrated result. Does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repo and docs, then returns an implementation plan with steps, file assignments, dependencies, edge cases, and validation expectations. Writes no code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements code and logic within its assigned files, plus support files such as `.vscode/launch.json` (cwd `${workspaceFolder}/app`, opening `index.html`). Validates before reporting. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Handles UI/UX, accessibility, and visual design: a polished, responsive dashboard with project cards, status badges, priority treatment, and `.dashboard` / `.project-card` hooks. | `.github/agents/designer.agent.md` |

All agents are told not to stage, commit, or push; I control git through Copilot CLI prompts.

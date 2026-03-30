# Copilot Workspace Instructions

This workspace is focused on **automated BDD test case generation** from JIRA issues using GitHub Copilot agents.

## Agents

- **`@generate-test-cases`** — primary agent. Accepts a JIRA issue key (e.g. `PROJ-123`), a JIRA issue URL, or free-text acceptance criteria. Produces a prioritised Gherkin test suite saved as a Markdown file in the project root.
- **`@jira`** — internal auth helper. Called automatically by `@generate-test-cases`. Do not invoke directly.

## Behaviour guidelines

- Always read credentials from `.env` — never ask the user for them interactively.
- Never overwrite existing test case files — always create a new file per run.
- Never modify source code, configuration, or documentation files other than the generated test case Markdown.
- Use concrete, realistic test data in scenarios — no placeholders like `<email>`.
- Prioritise test cases P1 → P4; within each band order Positive → Negative → Edge.

## Folder conventions

| Folder | Purpose |
|---|---|
| `.github/agents/` | Agent definition files (`*.agent.md`) |
| `.github/prompts/` | Reusable prompt files (`*.prompt.md`) |
| `.github/instructions/` | Context-specific instruction files (`*.instructions.md`) |
| `.github/skills/` | Skill definition files (`SKILL.md`, `*.skill.md`) |

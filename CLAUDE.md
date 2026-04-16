# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

A **Cursor Agent workspace** for automating BDD test case generation from JIRA issues and publishing AI use case documentation to Confluence. There is no source code, build system, or test suite — this project consists entirely of Cursor rule definitions and templates.

## How It Works

Four rules defined in `.cursor/rules/` orchestrate the workflows:

- **`@generate-test-cases`** — Primary rule: reads JIRA issues (or free-text), generates full BDD Gherkin test suites, saves to `test-cases-output/`
- **`@create-ai-use-case`** — Primary rule: drafts AI use case justification docs from a Markdown file or plain text, then publishes to Confluence after user approval
- **`jira-auth`** — Auth helper (Agent-Requested): reads `.env`, returns Base64-encoded Basic Auth header for JIRA
- **`confluence-auth`** — Auth helper (Agent-Requested): reads `.env`, returns Base64-encoded Basic Auth header for Confluence

Rules are invoked in Cursor Agent mode. Session credentials are cached at `/memories/session/` to avoid repeated rule lookups.

## Setup

Create a `.env` file from `.env.example`:
```
JIRA_BASE_URL=https://your-org.atlassian.net
JIRA_USER_EMAIL=you@example.com
JIRA_API_TOKEN=your-atlassian-api-token

CONFLUENCE_BASE_URL=https://your-org.atlassian.net/wiki
CONFLUENCE_USER_EMAIL=you@example.com
CONFLUENCE_API_TOKEN=your-atlassian-api-token
```

The same Atlassian API token works for both JIRA and Confluence.

## Agent Behavior Rules (from `.cursor/rules/global-instructions.mdc`)

- Always read credentials from `.env` — never prompt interactively
- Never overwrite existing files in `test-cases-output/` — always create a new file per run
- Never modify any file except generated test case Markdown and AI use case drafts
- Use concrete, realistic test data — no placeholders like `<email>` or `<username>`
- Prioritize test cases P1→P4; within each priority: Positive→Negative→Edge

## Output Conventions

**Test cases** saved to:
```
test-cases-output/[PROJ-123]test-cases-<kebab-case-feature-name><DD_MM_YYYY>.md
```

Each file contains: analysis comment block, summary table, full Gherkin scenarios sorted P1→P4, and a coverage matrix.

**AI use case pages** published to Confluence under space key `IAGCARGODI`, under the "AI Use Cases" ancestor page (`id: 3068362825`). Requires user to type `approved` before any API call is made.

## Template

`ai_use_case_template.md` defines the required sections for AI use case documents (Summary, Business Value, Actors, Triggers, Integrations, Scenarios, Effort/Complexity, Constraints, Recommended model, etc.). The `@create-ai-use-case` rule uses this as a fallback template.

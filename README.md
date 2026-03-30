# JIRA Test Case Generator

Automatically generate BDD test cases in Gherkin format from a JIRA issue — straight from GitHub Copilot Chat in VS Code.

---

## How it works

Two agents collaborate behind the scenes:

| Agent | Role |
|---|---|
| `@jira` | Reads credentials from `.env`, builds the JIRA Basic Auth header, and fetches issue data via the JIRA REST API |
| `@generate-test-cases` | Analyses the issue (description, acceptance criteria, screenshots), designs a prioritised BDD test suite, and saves it as a Markdown file in the project root |

You only ever talk to `@generate-test-cases`. It calls `@jira` automatically when it needs credentials.

---

## Prerequisites

- **VS Code** with the **GitHub Copilot** extension installed and signed in
- A JIRA account with API access (Atlassian Cloud or Server)

---

## Setup

### 1. Clone / open the project

```bash
git clone <repo-url>
cd <project-folder>
code .
```

### 2. Create your `.env` file

```bash
cp .env.example .env
```

Open `.env` and fill in your credentials:

```dotenv
JIRA_BASE_URL=https://your-org.atlassian.net
JIRA_USER_EMAIL=your-email@example.com
JIRA_API_TOKEN=your_api_token_here
```

> **Generate a JIRA API token:**
> 1. Go to <https://id.atlassian.com/manage-profile/security/api-tokens>
> 2. Click **Create API token**
> 3. Copy the token and paste it as `JIRA_API_TOKEN`

> **Never commit `.env`** — it is listed in `.gitignore`.

---

## Usage

Open **GitHub Copilot Chat** (`⌃⌘I` / `Ctrl+Alt+I`) and type one of the following:

### Option A — JIRA issue key

```
@generate-test-cases PROJ-123
```

### Option B — JIRA issue URL

```
@generate-test-cases https://your-org.atlassian.net/browse/PROJ-123
```

### Option C — Free-text (no JIRA)

```
@generate-test-cases
Feature: Password reset
AC:
1. User receives a reset email within 2 minutes
2. Reset link expires after 24 hours
3. Password must be at least 8 characters
```

---

## Output

A Markdown file is created in the **project root** with the naming convention:

```
[PROJ-123]test-cases-<feature-name><DD_MM_YYYY>.md
```

The file contains:

- **Summary table** — all test cases with IDs, priorities, and types at a glance
- **Full test cases** — Gherkin scenarios sorted P1 → P4, Positive → Negative → Edge
- **Coverage matrix** — maps every acceptance criterion to the test cases that cover it

---

## Project structure

```
.
├── .env.example                          # Credential template — safe to commit
├── .env                                  # Your real credentials — gitignored
├── .gitignore
├── README.md
├── .github/
│   ├── copilot-instructions.md           # Workspace-level Copilot instructions
│   ├── agents/
│   │   ├── jira.agent.md                 # JIRA auth agent
│   │   └── generate-test-cases.agent.md  # Test case generation agent
│   ├── prompts/                          # Reusable prompt files (*.prompt.md)
│   ├── instructions/                     # Context-specific instructions (*.instructions.md)
│   └── skills/                           # Skill definitions (*.skill.md)
└── [PROJ-123]test-cases-...md            # Generated output files
```

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `401 Unauthorized` | Check `JIRA_API_TOKEN` — regenerate if needed |
| `404 Not Found` | Verify the issue key exists and the `JIRA_BASE_URL` is correct |
| `403 Forbidden` | Your account lacks View permission for that issue |
| Agent not found | Make sure you are using `@generate-test-cases` (with a hyphen) in Copilot Chat |
| `.env` values not picked up | Ensure the file is in the workspace root and has no extra spaces around `=` |

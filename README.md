# JIRA Test Case Generator & AI Use Case Publisher

Automatically generate BDD test cases in Gherkin format from a JIRA issue, and publish AI use case justification pages to Confluence — straight from GitHub Copilot Chat in VS Code.

---

## How it works

Four agents collaborate behind the scenes:

| Agent | Role |
|---|---|
| `@jira` | Reads credentials from `.env`, builds the JIRA Basic Auth header, and fetches issue data via the JIRA REST API |
| `@generate-test-cases` | Analyses the issue (description, acceptance criteria, screenshots, and attached images), designs a prioritised BDD test suite, and saves it as a Markdown file in `test-cases-output/` |
| `@confluence-auth` | Reads Confluence credentials from `.env`, builds the Basic Auth header, and returns it to the caller — never makes API calls itself |
| `@create-ai-use-case-confluence` | Generates a structured AI use case justification document from a Markdown file or free text, shows a draft for approval, then publishes it as a new Confluence page |

You only ever talk to `@generate-test-cases` or `@create-ai-use-case-confluence`. Each agent calls its auth helper automatically when it needs credentials.

---

## Prerequisites

- **VS Code** with the **GitHub Copilot** extension installed and signed in
- A JIRA account with API access (Atlassian Cloud or Server)
- An Atlassian Confluence account with API access (for AI use case publishing)

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

CONFLUENCE_BASE_URL=https://your-org.atlassian.net/wiki
CONFLUENCE_USER_EMAIL=your-email@example.com
CONFLUENCE_API_TOKEN=your_confluence_api_token_here
```

> **Generate an Atlassian API token (works for both JIRA and Confluence):**
> 1. Go to <https://id.atlassian.com/manage-profile/security/api-tokens>
> 2. Click **Create API token**
> 3. Copy the token and paste it as `JIRA_API_TOKEN` and/or `CONFLUENCE_API_TOKEN`

> **Never commit `.env`** — it is listed in `.gitignore`.

---

## Usage

### Generating BDD test cases

Open **GitHub Copilot Chat** (`⌃⌘I` / `Ctrl+Alt+I`) and type one of the following:

#### Option A — JIRA issue key

```
@generate-test-cases PROJ-123
```

#### Option B — JIRA issue URL

```
@generate-test-cases https://your-org.atlassian.net/browse/PROJ-123
```

#### Option C — Free-text (no JIRA)

```
@generate-test-cases
Feature: Password reset
AC:
1. User receives a reset email within 2 minutes
2. Reset link expires after 24 hours
3. Password must be at least 8 characters
```

#### Option D — With screenshots / images

Attach one or more `.png` screenshots directly to the chat message using the **attachment icon** (paperclip) before sending. The agent will analyse the visual context and incorporate it into the test coverage.

```
@generate-test-cases PROJ-123
[attach screenshot.png via the attachment icon]
```

You can combine any of the options above with image attachments — e.g. a JIRA key plus a UI screenshot, or free-text acceptance criteria plus a wireframe.

---

### Publishing an AI use case to Confluence

#### Option A — From an existing Markdown file

```
@create-ai-use-case-confluence ai_use_case_generate-test-cases.md
```

#### Option B — From a plain-text description

```
@create-ai-use-case-confluence
Feature: automated BDD test generation from JIRA issues using Copilot agents
```

The agent will generate a structured justification document based on `ai_use_case_template.md`, show you a draft for review, and only publish to Confluence after you type **`approved`**.

---

## Output

### Test cases

A Markdown file is created in `test-cases-output/` with the naming convention:

```
[PROJ-123]test-cases-<feature-name><DD_MM_YYYY>.md
```

The file contains:

- **Summary table** — all test cases with IDs, priorities, and types at a glance
- **Full test cases** — Gherkin scenarios sorted P1 → P4, Positive → Negative → Edge
- **Coverage matrix** — maps every acceptance criterion to the test cases that cover it

### AI use case page

A new Confluence page is created under the **AI Use Cases** space, structured according to `ai_use_case_template.md`. The agent returns the direct Confluence URL after a successful publish.

---

## Project structure

```
.
├── .env.example                                    # Credential template — safe to commit
├── .env                                            # Your real credentials — gitignored
├── .gitignore
├── README.md
├── ai_use_case_template.md                         # Template for AI use case justification pages
├── input-images/                                   # Optional screenshots for test case enrichment
├── test-cases-output/                              # Generated BDD test case files
├── .github/
│   ├── copilot-instructions.md                     # Workspace-level Copilot instructions
│   ├── agents/
│   │   ├── jira.agent.md                           # JIRA auth agent
│   │   ├── generate-test-cases.agent.md            # Test case generation agent
│   │   ├── confluence-auth.agent.md                # Confluence auth agent
│   │   └── create-ai-use-case-confluence.agent.md  # AI use case publishing agent
│   ├── prompts/                                    # Reusable prompt files (*.prompt.md)
│   ├── instructions/                               # Context-specific instructions (*.instructions.md)
│   └── skills/                                     # Skill definitions (*.skill.md)
└── [PROJ-123]test-cases-...md                      # Generated output files (legacy location)
```

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `401 Unauthorized` (JIRA) | Check `JIRA_API_TOKEN` — regenerate if needed |
| `401 Unauthorized` (Confluence) | Check `CONFLUENCE_API_TOKEN` — regenerate if needed |
| `404 Not Found` | Verify the issue key exists and the `JIRA_BASE_URL` is correct |
| `403 Forbidden` | Your account lacks View permission for that issue or Confluence space |
| `409 Conflict` (Confluence) | A page with the same title already exists — the agent will append today's date and retry once |
| Agent not found | Make sure you are using `@generate-test-cases` or `@create-ai-use-case-confluence` (with hyphens) in Copilot Chat |
| `.env` values not picked up | Ensure the file is in the workspace root and has no extra spaces around `=` |
| Images not analysed | Attach `.png` files via the attachment icon before sending — do not paste them as file paths |

# JIRA Test Case Generator, TestRail Publisher & AI Use Case Publisher

Automatically generate BDD test cases in Gherkin format from a JIRA issue, publish reviewed cases to TestRail, and publish AI use case justification pages to Confluence — straight from Cursor Agent in VS Code or the Cursor IDE.

---

## How it works

Six Cursor rules collaborate behind the scenes:

| Rule | Role |
|---|---|
| `jira-auth` | Reads credentials from `.env`, builds the JIRA Basic Auth header — never makes API calls itself |
| `@generate-test-cases` | Analyses the issue (description, acceptance criteria, screenshots), designs a prioritised BDD test suite, and saves it as a Markdown file in `test-cases-output/` |
| `@generate-jira-test-cases` | Independent generation flow: resolves JIRA story and writes a new markdown test case file for QA review |
| `@publish-test-cases-to-testrail` | Independent publishing flow: reads a reviewed markdown file, transforms cases, and pushes to TestRail |
| `confluence-auth` | Reads Confluence credentials from `.env`, builds the Basic Auth header — never makes API calls itself |
| `@create-ai-use-case` | Generates a structured AI use case justification document from a Markdown file or free text, shows a draft for approval, then publishes it as a new Confluence page |

You only ever talk to the primary flows you need:

- `@generate-jira-test-cases` for JIRA -> markdown generation
- `@publish-test-cases-to-testrail` for markdown -> TestRail publishing
- `@create-ai-use-case` for Confluence publishing

---

## Prerequisites

- **Cursor** (or VS Code with Cursor extension) with Agent mode enabled
- A JIRA account with API access (Atlassian Cloud or Server)
- A TestRail account with API access (for publishing reviewed tests)
- An Atlassian Confluence account with API access (for AI use case publishing)

---

## Setup

### 1. Clone / open the project

```bash
git clone <repo-url>
cd <project-folder>
cursor .
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

TESTRAIL_BASE_URL=https://your-company.testrail.io
TESTRAIL_USER_EMAIL=your-email@example.com
TESTRAIL_API_KEY=your_testrail_api_key_here
```

> **Generate an Atlassian API token (works for both JIRA and Confluence):**
> 1. Go to <https://id.atlassian.com/manage-profile/security/api-tokens>
> 2. Click **Create API token**
> 3. Copy the token and paste it as `JIRA_API_TOKEN` and/or `CONFLUENCE_API_TOKEN`

> **Never commit `.env`** — it is listed in `.gitignore`.

---

## Usage

Open **Cursor Agent** (or Composer in Agent mode) and type one of the following.

### Flow 1: Generate BDD test cases from JIRA

This flow fetches JIRA data and creates a new markdown file in `test-cases-output/`.

#### Option A — JIRA issue key

```
@generate-jira-test-cases PROJ-123
```

#### Option B — JIRA issue URL

```
@generate-jira-test-cases https://your-org.atlassian.net/browse/PROJ-123
```

#### Option C — Include an optional output tag

```
@generate-jira-test-cases PROJ-123 output_tag=review1
```

#### Option D — With screenshots / images

Attach one or more `.png` screenshots directly to the chat message before sending. The agent will analyse the visual context and incorporate it into the test coverage.

```
@generate-jira-test-cases PROJ-123
[attach screenshot.png]
```

After generation, QA reviews and edits the markdown file manually.

---

### Flow 2: Publish reviewed test cases to TestRail

This flow requires a reviewed markdown file path and does not call JIRA.

#### Option A — Dry run (no writes)

```
@publish-test-cases-to-testrail test_cases_file_path=test-cases-output/[PROJ-123]test-cases-password-reset15_04_2026.md testrail_project_id=12 target_folder_or_section_name="Password Reset" suite_name="Web Regression" dry_run=true
```

#### Option B — Real push

```
@publish-test-cases-to-testrail test_cases_file_path=test-cases-output/[PROJ-123]test-cases-password-reset15_04_2026.md testrail_project_id=12 target_folder_or_section_name="Password Reset" suite_name="Web Regression" dry_run=false
```

---

### Publishing an AI use case to Confluence

#### Option A — From an existing Markdown file

```
@create-ai-use-case ai_use_case_generate-test-cases.md
```

#### Option B — From a plain-text description

```
@create-ai-use-case
Feature: automated BDD test generation from JIRA issues using Cursor Agent rules
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

### TestRail publish report

The publish flow returns:

- suite and folder/section resolved (or created)
- created / updated / skipped / failed case counts
- list of TestRail case IDs created or updated
- parse warnings for any skipped markdown entries

### AI use case page

A new Confluence page is created under the **AI Use Cases** space, structured according to `ai_use_case_template.md`. The agent returns the direct Confluence URL after a successful publish.

---

## Project structure

```
.
├── .env.example                          # Credential template — safe to commit
├── .env                                  # Your real credentials — gitignored
├── .gitignore
├── README.md
├── ai_use_case_template.md               # Template for AI use case justification pages
├── input-images/                         # Optional screenshots for test case enrichment
├── test-cases-output/                    # Generated BDD test case files
└── .cursor/
    ├── rules/
    │   ├── global-instructions.mdc              # Always-on workspace behaviour rules
    │   ├── generate-test-cases.mdc              # Manual rule: @generate-test-cases
    │   ├── generate-jira-test-cases.mdc         # Manual rule: @generate-jira-test-cases
    │   ├── publish-test-cases-to-testrail.mdc   # Manual rule: @publish-test-cases-to-testrail
    │   ├── create-ai-use-case.mdc               # Manual rule: @create-ai-use-case
    │   ├── jira-auth.mdc                        # Agent-Requested: JIRA auth helper
    │   ├── confluence-auth.mdc                  # Agent-Requested: Confluence auth helper
    │   ├── prompts/                             # Reusable prompt files (future)
    │   └── instructions/                        # Context-specific instructions (future)
    └── skills/
        ├── testrail-auth/SKILL.md               # TestRail auth helper
        └── transform-and-push-testrail/SKILL.md # Markdown transform + TestRail push helper
```

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `401 Unauthorized` (JIRA) | Check `JIRA_API_TOKEN` in `.env` — regenerate if needed |
| `401 Unauthorized` (TestRail) | Check `TESTRAIL_USER_EMAIL` and `TESTRAIL_API_KEY` in `.env` |
| `401 Unauthorized` (Confluence) | Check `CONFLUENCE_API_TOKEN` in `.env` — regenerate if needed |
| `404 Not Found` | Verify the issue key exists and `JIRA_BASE_URL` is correct |
| `403 Forbidden` | Your account lacks View permission for that issue or Confluence space |
| `409 Conflict` (Confluence) | A page with the same title already exists — the agent will append today's date and retry once |
| TestRail target not found | Confirm `testrail_project_id`, `suite_name`, and folder/section name |
| Rule not applied | Make sure you are using the exact rule names (with hyphens) in Cursor Agent |
| `.env` values not picked up | Ensure the file is in the workspace root and has no extra spaces around `=` |
| Images not analysed | Attach `.png` files directly to the chat message before sending |

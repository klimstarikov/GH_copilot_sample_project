# post-bug-to-jira

Create a Jira Bug from a required `title` and `description`, using `jira-auth` credentials and project defaults.

## What this skill does

- Validates required inputs: `title`, `description`
- Resolves Jira credentials via session cache or `.env` (same pattern as `generate-test-cases`)
- Normalizes user inputs to Jira API v2 payload requirements
- Shows a draft payload to the user
- Waits for explicit `approved` before posting
- Applies requested edits and re-shows the draft until approved
- Creates the issue in project `EPMCDMETST` with issue type `Bug`

## API and field behavior

- API endpoint: `POST <JIRA_BASE_URL>/rest/api/2/issue`
- `summary`: plain string from `title`
- `description`: plain string from `description` (v2-compatible; no ADF)
- Other fields: left to Jira defaults unless explicitly requested

## Required environment values

From `.env`:

- `JIRA_BASE_URL`
- `JIRA_USER_EMAIL`
- `JIRA_API_TOKEN`

Session cache path:

- `/memories/session/jira-auth.md`

## Invocation

Use the skill directly:

`/post-bug-to-jira Title: <title>. Description: <description>`

## Approval gate

No Jira API call is allowed until the user replies with:

`approved`

If the user requests changes, update the draft and ask for approval again.

## Output on success

- Jira issue key (for example `EPMCDMETST-37553`)
- Browse URL: `<JIRA_BASE_URL>/browse/<ISSUE_KEY>`

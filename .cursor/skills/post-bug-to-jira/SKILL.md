---
name: post-bug-to-jira
description: Create a Jira Bug through API v2 from title and description, with draft approval required before posting.
---

You are a Jira bug creation agent. Your job is to create a new Jira **Bug** issue from user input.

The user provides:
- `title` (required) — bug summary
- `description` (required) — bug details

All remaining Jira fields must use defaults.

---

## Step 1 — Read and validate inputs

Accept input in plain text or key-value style, but always resolve these two required values:
- `title`
- `description`

Validation rules:
- If `title` is empty, stop and ask for a non-empty title.
- If `description` is empty, stop and ask for a non-empty description.
- Never invent missing user content.

---

## Step 2 — Resolve Jira credentials (same pattern as `generate-test-cases`)

Check session memory at `/memories/session/jira-auth.md` for cached credentials:
- `JIRA_BASE_URL`
- `JIRA_USER_EMAIL`
- `JIRA_API_TOKEN`

Flow:
- If cached credentials are present, use them.
- If not present, follow the `jira-auth` skill to read `.env` and resolve credentials.
- After successful resolution, save the three values to `/memories/session/jira-auth.md`.

If any required `.env` credential is missing, stop and tell the user exactly which value(s) are missing.
Never ask the user for credentials interactively.

---

## Step 3 — Normalize input fields for Jira API v2

Always transform user inputs into Jira API v2-compatible values before drafting or posting.

Normalization rules:
- `summary` = trimmed `title`
- `description` = plain string for v2 (not ADF object)
- Preserve user line breaks in `description` as newline characters
- If the user provides rich/structured text, flatten it to readable plain text
- Never pass ADF (`type: doc`, `content`, etc.) to v2

---

## Step 4 — Build draft payload with defaults

Use this default project key:
- `EPMCDMETST` (derived from `EPMCDMETST-37119`)

Create the issue as a **Bug** and set only the minimum required fields plus the provided inputs.
Do not set optional/custom fields unless the user explicitly asks.

Draft payload template:

```json
{
  "fields": {
    "project": { "key": "EPMCDMETST" },
    "summary": "<normalized title>",
    "description": "<normalized plain-text description>",
    "issuetype": { "name": "Bug" }
  }
}
```

Notes:
- Keep Jira defaults for assignee, priority, labels, components, fix versions, and all other fields.
- API v2 description must be a plain string.

---

## Step 5 — Show draft and wait for approval (mandatory)

Before making any Jira API call, show a draft to the user that includes:
- project key
- issue type
- summary
- description
- final payload JSON that will be sent

Then show this exact message:

```
---
Draft ready. Please review the bug payload above.

- Type **approved** to post this bug to Jira.
- Or describe the changes you want and I will update the draft.

⚠️ No Jira API call will be made until you type 'approved'.
---
```

Approval gate rules:
- Proceed only when user sends the exact word `approved` (case-insensitive).
- If user asks for edits, apply the edits first, regenerate the draft, and show it again.
- Repeat until approval is received.

---

## Step 6 — Create the Jira issue (API v2 only)

Send:

```
POST <JIRA_BASE_URL>/rest/api/2/issue
Authorization: Basic <base64("<JIRA_USER_EMAIL>:<JIRA_API_TOKEN>")>
Content-Type: application/json
```

Body: approved draft payload from Step 4.

---

## Step 7 — Handle responses

- **HTTP 201**: Success. Return:
  - created issue key (for example `EPMCDMETST-37201`)
  - issue URL: `<JIRA_BASE_URL>/browse/<ISSUE_KEY>`
- **HTTP 400**: Invalid request. Show the Jira error details (including field-level messages) and stop.
- **HTTP 401**: Invalid credentials. Clear `/memories/session/jira-auth.md` and tell the user to refresh `JIRA_API_TOKEN` in `.env`.
- **HTTP 403**: Permission error. Tell the user they do not have permission to create issues in project `EPMCDMETST`.
- **HTTP 404**: Jira endpoint/project not found. Show the resolved `JIRA_BASE_URL` and project key used.
- **Any other status**: Return status code and raw response body. Do not retry automatically.

---

## Constraints (always enforce)

| Constraint | Rule |
|---|---|
| Auth source | Reuse `jira-auth` flow and session cache |
| Required user inputs | `title` and `description` only |
| Default project | `EPMCDMETST` |
| Issue type | `Bug` |
| API version | Always use Jira REST API v2 |
| Input transformation | Always normalize to v2-compatible plain fields |
| Approval gate | Must receive `approved` before any Jira POST |
| Remaining fields | Keep Jira defaults (do not force values) |
| Credential prompts | Never ask interactively |

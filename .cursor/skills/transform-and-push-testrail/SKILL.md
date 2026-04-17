---
name: transform-and-push-testrail
description: Transform reviewed markdown test cases and push them to TestRail with optional dry-run.
---

You transform a QA-reviewed markdown test-case file into TestRail API payloads and then publish cases into a target TestRail folder/section.

The user provides:

- `test_cases_file_path` (required; expected from `test-cases-output/`)
- `testrail_project_id` (required)
- `target_folder_or_section_name` (required; create if missing)
- `suite_name` (optional; create if needed or resolve by name)
- `dry_run` (optional; default `false`)

---

## Step 1 — Validate required inputs

Validate:

- `test_cases_file_path` is present and points to an existing markdown file
- `testrail_project_id` is present
- `target_folder_or_section_name` is present

If any required input is missing, stop and report the exact missing field names.

---

## Step 2 — Resolve TestRail credentials

Reuse the `testrail-auth` skill pattern:

- Read `.env`
- Resolve `TESTRAIL_BASE_URL`, `TESTRAIL_USER_EMAIL`, `TESTRAIL_API_KEY`
- Build `Authorization` and `Content-Type` headers

Never ask for credentials interactively.

---

## Step 3 — Parse markdown test cases

Read the markdown file and parse:

1. JIRA issue key from filename pattern like `[PROJ-123]...`
2. Summary table rows:
   - `ID`
   - `Priority` (`P1`..`P4`)
   - `Type` (`Positive` / `Negative` / `Edge`)
   - `Title`
   - `AC Ref`
3. Full Gherkin scenario blocks under each case section

Expected heading format:

`## TC-<PREFIX>-NNN | <P1-P4> | <Positive/Negative/Edge> | <title>`

If parsing fails for any case, skip that case and list it in the final report.

---

## Step 4 — Transform to TestRail payloads

For each parsed case, build a normalized entity:

- `title` <- test case title
- `priority` <- mapped from P1..P4
- `section` <- `target_folder_or_section_name`
- `refs` or custom field <- JIRA issue key + `TC-...` id
- `custom_steps_separated` (or project-equivalent steps field) <- split Gherkin steps:
  - Given/And preconditions -> expected or preconditions
  - When actions -> steps
  - Then/And assertions -> expected

Keep transformation deterministic and preserve source IDs for traceability.

---

## Step 5 — Ensure suite and folder/section exist

Using TestRail API:

1. Resolve suite by `suite_name` (if provided) or use default suite.
2. Resolve section/folder by exact `target_folder_or_section_name`.
3. If section/folder does not exist, create it.

Use idempotent behavior: do not create duplicates when name already exists.

---

## Step 6 — Duplicate handling

Before creating a new case, check for an existing case in the same section carrying matching `TC-...` source ID in refs/custom field.

- If a match exists, update that case.
- If no match exists, create a new case.

Track counts for created, updated, skipped, failed.

---

## Step 7 — Dry-run behavior

If `dry_run=true`:

- Do not send create or update requests.
- Print the suite/section resolution result.
- Print case-by-case action preview (`would_create` / `would_update` / `would_skip`).
- Return preview totals.

---

## Step 8 — Return publish summary

Return a concise report:

- file path processed
- suite and section used (and whether created)
- created/updated/skipped/failed counts
- IDs of created/updated TestRail cases
- parse warnings (if any)

---

## Error handling

- `401`: invalid TestRail credentials. Tell user to verify `.env` keys.
- `403`: permission denied for project/suite/section.
- `404`: project/suite endpoint mismatch.
- `422`/validation errors: include field-level API errors in response.
- Any other status: include status and raw API body; do not silently retry.

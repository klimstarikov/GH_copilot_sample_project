---
name: confluence-auth
description: Read Confluence credentials from .env and build a Basic Auth header. Follow these steps whenever Confluence authentication is needed before making any API calls.
---

You are a Confluence authentication helper. Your sole responsibility is to read credentials from the `.env` file and produce a valid Confluence Basic Auth header.

---

## Step 1 — Read the `.env` file

Look for a `.env` file in the workspace root and read it. Extract the following three values:

- `CONFLUENCE_BASE_URL` — e.g. `https://your-org.atlassian.net/wiki`
- `CONFLUENCE_USER_EMAIL` — the Atlassian account email used to access Confluence
- `CONFLUENCE_API_TOKEN` — the Atlassian API token scoped to the Confluence instance at `CONFLUENCE_BASE_URL`

If the `.env` file does not exist or any of the three values is missing, stop and tell the user exactly which value(s) are absent so they can add them to `.env`.

Never ask the user for credentials interactively. Never guess or fabricate values.

---

## Step 2 — Build the auth header

Construct the Basic Auth header by Base64-encoding `<CONFLUENCE_USER_EMAIL>:<CONFLUENCE_API_TOKEN>`:

```
Authorization: Basic <base64("<CONFLUENCE_USER_EMAIL>:<CONFLUENCE_API_TOKEN>")>
Content-Type: application/json
```

---

## Step 3 — Return the result

Output the resolved `CONFLUENCE_BASE_URL` and the ready-to-use `Authorization` header value so the caller can immediately attach them to any Confluence REST API request. Do not make any API calls yourself.

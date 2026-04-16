# post-bug-to-jira examples

## 1) Basic request

`/post-bug-to-jira Title: Add to Cart button does nothing on PDP. Description: On Product Detail Page for SKU 112233, clicking "Add to Cart" does not update cart count and no error is shown.`

Expected behavior:

- Agent validates `title` and `description`
- Agent shows a draft payload
- Agent waits for `approved`
- Agent posts to Jira API v2

## 2) Multi-line description

`/post-bug-to-jira Title: Checkout fails for Visa cards. Description: Steps to reproduce:\n1. Open checkout\n2. Use Visa 4111...\n3. Click Pay\nActual: spinner loops forever\nExpected: order confirmation page`

Expected behavior:

- Newlines are preserved in plain text `description`
- Payload remains API v2 compatible

## 3) Draft edit cycle

User input:

`/post-bug-to-jira Title: Login error message is vague. Description: Login shows generic failure even when account is locked.`

Agent shows draft.

User feedback:

`Please change title to: Locked account should show specific error`

Expected behavior:

- Agent updates the draft
- Agent re-shows updated draft
- Agent still waits for `approved`

## 4) Final approval

User input:

`approved`

Expected behavior:

- Agent posts the latest approved draft to Jira
- Agent returns issue key and browse URL

## 5) Missing input handling

User input:

`/post-bug-to-jira Title:`

Expected behavior:

- Agent stops and asks for a non-empty `title`
- No API call is made

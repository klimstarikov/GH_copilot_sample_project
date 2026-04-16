# Test Cases: Sales tag for products on Sale

> **Generated:** 2026-04-10
> **Feature:** Show a "Sale" badge only for discounted products on listing pages while keeping product detail pages badge-free.
> **Total:** 8 test cases - 3 Positive | 3 Negative | 2 Edge

---

## Summary Table

| ID | Priority | Type | Title | AC Ref |
|----|----------|------|-------|--------|
| TC-SALE-001 | P1 | Positive | Sale tag is shown for discounted product cards on Home page | AC-1 |
| TC-SALE-002 | P1 | Positive | Product without discount does not display Sale tag on Home page | AC-2 |
| TC-SALE-003 | P1 | Positive | Discounted product detail page shows strike-through price without Sale tag | AC-3 |
| TC-SALE-004 | P2 | Negative | Product card with regular price only never shows Sale tag | AC-2 |
| TC-SALE-005 | P2 | Negative | Sale tag is not shown when discount formatting is invalid | AC-1 |
| TC-SALE-006 | P2 | Negative | View item page does not inherit listing Sale badge state | AC-3 |
| TC-SALE-007 | P3 | Edge | Minimal discount value still triggers Sale tag on listing | AC-1 |
| TC-SALE-008 | P3 | Edge | Infinite scroll keeps Sale-tag rule consistent for mixed products | AC-1, AC-2 |

---

## Test Cases

<!--
Domain understanding:
- Feature governs visual merchandising for discounted products.
- Primary user is an anonymous storefront visitor browsing product listings and product details.
- Business goal is to clearly indicate discounted items on listing pages while avoiding redundant badge noise on detail pages.

AC decomposition:
- AC-1: Discounted products on page must display "Sale" tag.
- AC-2: Non-discounted products must not display "Sale" tag.
- AC-3: On view-item screen for discounted products, show strike-through discount pricing only, no "Sale" tag.

Coverage mapping:
- Positive: Correct badge shown/hidden on listing and details.
- Negative: Incorrect pricing data or state propagation must not create false badges.
- Edge: Boundary discount values and long-scroll mixed datasets keep rules intact.

Priority rationale:
- P1: Core display logic for listing/detail pages.
- P2: Wrong-tag prevention and state-leak prevention.
- P3: Boundary and feed-continuity behavior.

Visual context:
- No screenshots were attached, so tests are based on JIRA text and referenced scenario.
-->

---
## TC-SALE-001 | P1 | Positive | Sale tag is shown for discounted product cards on Home page

**Priority:** P1
**Type:** Positive
**AC Reference:** AC-1

```gherkin
Scenario: Sale tag is shown for discounted product cards on Home page
  Given an anonymous user opens the Home page
  When the user scrolls to a product card with original price "$120.00" and discounted price "$95.00"
  Then the same product card should display the tag "Sale"
  And the original price should appear with a strike-through style
```

---

## TC-SALE-002 | P1 | Positive | Product without discount does not display Sale tag on Home page

**Priority:** P1
**Type:** Positive
**AC Reference:** AC-2

```gherkin
Scenario: Product without discount does not display Sale tag on Home page
  Given an anonymous user opens the Home page
  When the user finds a product card priced at "$89.00" with no reduced price shown
  Then that product card should not display the tag "Sale"
```

---

## TC-SALE-003 | P1 | Positive | Discounted product detail page shows strike-through price without Sale tag

**Priority:** P1
**Type:** Positive
**AC Reference:** AC-3

```gherkin
Scenario: Discounted product detail page shows strike-through price without Sale tag
  Given an anonymous user opens the Home page
  And the user selects a discounted product card with prices "$140.00" and "$99.00"
  When the View Item page opens for that product
  Then the page should show the original price "$140.00" with strike-through and current price "$99.00"
  And the page should not display the tag "Sale"
```

---

## TC-SALE-004 | P2 | Negative | Product card with regular price only never shows Sale tag

**Priority:** P2
**Type:** Negative
**AC Reference:** AC-2

```gherkin
Scenario: Product card with regular price only never shows Sale tag
  Given an anonymous user opens the Home page
  When the pricing service returns only one price "$59.00" for a product
  Then the product card should not display the tag "Sale"
  And no strike-through price element should be rendered
```

---

## TC-SALE-005 | P2 | Negative | Sale tag is not shown when discount formatting is invalid

**Priority:** P2
**Type:** Negative
**AC Reference:** AC-1

```gherkin
Scenario: Sale tag is not shown when discount formatting is invalid
  Given an anonymous user opens the Home page
  When a product card contains malformed pricing data where discounted price is "$0.00" and original price is missing
  Then the product card should not display the tag "Sale"
  And the UI should display only the valid current price without strike-through
```

**Notes:** Verifies the badge depends on valid reduction data, not on partial malformed payloads.
---

## TC-SALE-006 | P2 | Negative | View item page does not inherit listing Sale badge state

**Priority:** P2
**Type:** Negative
**AC Reference:** AC-3

```gherkin
Scenario: View item page does not inherit listing Sale badge state
  Given an anonymous user sees a discounted product card with the tag "Sale" on Home page
  When the user opens the View Item page for that product
  Then the View Item page should not display the tag "Sale"
  And discounted pricing should still be visible using strike-through for the original price
```

---

## TC-SALE-007 | P3 | Edge | Minimal discount value still triggers Sale tag on listing

**Priority:** P3
**Type:** Edge
**AC Reference:** AC-1

```gherkin
Scenario: Minimal discount value still triggers Sale tag on listing
  Given an anonymous user opens the Home page
  When a product card shows original price "$100.00" and discounted price "$99.99"
  Then the product card should display the tag "Sale"
  And both prices should be visible with the original one struck through
```

---

## TC-SALE-008 | P3 | Edge | Infinite scroll keeps Sale-tag rule consistent for mixed products

**Priority:** P3
**Type:** Edge
**AC Reference:** AC-1, AC-2

```gherkin
Scenario: Infinite scroll keeps Sale-tag rule consistent for mixed products
  Given an anonymous user opens the Home page
  When the user scrolls through three loaded product batches containing both discounted and non-discounted items
  Then every discounted product card should display the tag "Sale"
  And every non-discounted product card should not display the tag "Sale"
```

---

## Coverage Matrix

| AC | Description | Test IDs |
|----|-------------|----------|
| AC-1 | Discounted products on page should have "Sale" tag | TC-SALE-001, TC-SALE-005, TC-SALE-007, TC-SALE-008 |
| AC-2 | Non-discounted products should not have "Sale" tag | TC-SALE-002, TC-SALE-004, TC-SALE-008 |
| AC-3 | Discounted product view-item page should not show "Sale" tag, only strike-through reduction | TC-SALE-003, TC-SALE-006 |

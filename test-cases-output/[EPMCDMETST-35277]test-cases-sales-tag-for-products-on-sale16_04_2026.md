# Test Cases: Sales Tag for Products on Sale

> **Generated:** 2026-04-16
> **Feature:** Products with a price reduction display a 'Sale' tag on listing pages, and no 'Sale' tag appears on the product detail (view item) page — only the strikethrough price is shown.
> **Total:** 9 test cases — 4 Positive | 2 Negative | 3 Edge

---

## Summary Table

| ID | Priority | Type | Title | AC Ref |
|----|----------|------|-------|--------|
| TC-SALE-001 | P1 | Positive | Discounted product displays 'Sale' tag on Home page listing | AC-1 |
| TC-SALE-002 | P1 | Positive | Discounted product detail page shows strikethrough price with no 'Sale' tag | AC-3 |
| TC-SALE-003 | P1 | Negative | Full-price product does not display 'Sale' tag on Home page listing | AC-2 |
| TC-SALE-004 | P2 | Positive | All discounted products in the same listing section display 'Sale' tags simultaneously | AC-1 |
| TC-SALE-005 | P2 | Negative | Non-discounted products adjacent to discounted ones show no 'Sale' tag | AC-2 |
| TC-SALE-006 | P3 | Positive | Authenticated user sees same 'Sale' tags as guest user on Home page | AC-1 |
| TC-SALE-007 | P3 | Edge | Navigating from a listing 'Sale'-tagged product to its detail view removes the 'Sale' tag | AC-3 |
| TC-SALE-008 | P4 | Edge | Product detail page accessed directly via URL shows only strikethrough price, no 'Sale' tag | AC-3 |
| TC-SALE-009 | P4 | Edge | Discounted product in the 'Featured' section of the Home page displays 'Sale' tag | AC-1 |

---

## Test Cases

<!--
ANALYSIS

## Domain Understanding
- Application: automationteststore.com — an e-commerce demo store
- Feature: Visual 'Sale' badge on discounted products in product listing pages (Home, category, featured sections)
- Users: Guest (unauthenticated) and logged-in shoppers
- Business goal: Clearly communicate price reductions to shoppers to drive purchase decisions

## Screenshot Observations (SCREENSHOTS_CONTEXT)

Image 1 — `Screenshot at Apr 07 10-01-16.png` (Product Detail / View Item screen):
- Shows the view-item page for "BeneFit Girl Meets Pearl" at automationteststore.com/index.php?rt=product/product&product_id=51
- Current price: $19.00 | Original price: $36.00 (displayed as strikethrough)
- No 'Sale' tag is visible anywhere on the detail page — only the strikethrough price conveys the discount
- Directly corroborates AC-3: on the view-item screen no Sale tag is rendered, just the strikethrough price

Image 2 — `Screenshot at Mar 25 21-11-33.png` (Home page product grid):
- Shows the Home page with 'FEATURED' and 'LATEST PRODUCTS' sections
- Products visible: SKINSHEEN BRONZER STICK, BENEFIT GIRL MEETS PEARL, BENEFIT BELLA BAMBA, TROPIQUES MINERALE LOOSE BRONZER (Featured); ABSOLUTE ANTI-AGE SPOT REPLENISHING UNIFYING TREATMENT SPF, ABSOLUE EYE PRECIOUS CELLS, TOTAL MOISTURE FACIAL CREAM, FLASH BRONZER BODY GEL (Latest Products)
- OCR captures 'Sale' text appearing on at least two products in the 'LATEST PRODUCTS' section
- Products without price reduction do not show the 'Sale' badge
- Directly corroborates AC-1 and AC-2 in the context of the Home page grid

## AC Decomposition
- AC-1: ANY product on a listing page that carries a strikethrough/reduced price MUST render a 'Sale' badge
- AC-2: ANY product on a listing page that has NO strikethrough/reduced price MUST NOT render a 'Sale' badge
- AC-3: A product's view-item (detail) page MUST NOT show a 'Sale' badge even when the product is discounted — only the strikethrough price indicates the discount

## Priority Rationale
- P1: Core happy paths (discounted product shows Sale tag) + core negative (non-discounted product has no tag) + AC-3 primary scenario are all critical because they represent the primary user-visible feature behaviour
- P2: Multi-product consistency and adjacency negative scenario — important for data integrity and correct rendering across an entire listing page
- P3: Auth parity and state-transition edge (listing → detail) — affect real users regularly but not blocking
- P4: URL-direct access and Featured section coverage — lower risk rare-path scenarios
-->

---

## TC-SALE-001 | P1 | Positive | Discounted product displays 'Sale' tag on Home page listing

**Priority:** P1
**Type:** Positive
**AC Reference:** AC-1

```gherkin
Scenario: Discounted product displays 'Sale' tag on Home page listing
  Given a guest user is not logged in to automationteststore.com
  When the user opens the Home page at "https://automationteststore.com"
  And the user scrolls down to the "LATEST PRODUCTS" section
  Then the product "Absolute Anti-Age Spot Replenishing + Unifying Treatment SPF 15" has a strikethrough original price
  And that product displays a "Sale" tag
```

---

## TC-SALE-002 | P1 | Positive | Discounted product detail page shows strikethrough price with no 'Sale' tag

**Priority:** P1
**Type:** Positive
**AC Reference:** AC-3

```gherkin
Scenario: Discounted product detail page shows strikethrough price with no 'Sale' tag
  Given a guest user is not logged in to automationteststore.com
  And the product "BeneFit Girl Meets Pearl" has a reduced price of $19.00 (original $36.00)
  When the user navigates to the product detail page at "https://automationteststore.com/index.php?rt=product/product&product_id=51"
  Then the page displays the current price "$19.00"
  And the original price "$36.00" is shown with a strikethrough style
  And no "Sale" tag is visible anywhere on the product detail page
```

---

## TC-SALE-003 | P1 | Negative | Full-price product does not display 'Sale' tag on Home page listing

**Priority:** P1
**Type:** Negative
**AC Reference:** AC-2

```gherkin
Scenario: Full-price product does not display 'Sale' tag on Home page listing
  Given a guest user is not logged in to automationteststore.com
  When the user opens the Home page at "https://automationteststore.com"
  And the user scrolls down to the "LATEST PRODUCTS" section
  Then the product "Flash Bronzer Body Gel" displays the price "$29.00" with no strikethrough
  And that product does not display a "Sale" tag
```

---

## TC-SALE-004 | P2 | Positive | All discounted products in the same listing section display 'Sale' tags simultaneously

**Priority:** P2
**Type:** Positive
**AC Reference:** AC-1

```gherkin
Scenario: All discounted products in the same listing section display 'Sale' tags simultaneously
  Given a guest user is not logged in to automationteststore.com
  When the user opens the Home page at "https://automationteststore.com"
  And the user scrolls to the "LATEST PRODUCTS" section
  Then every product in that section that has a strikethrough original price displays a "Sale" tag
  And the count of "Sale" tags equals the count of products with a strikethrough price in that section
```

---

## TC-SALE-005 | P2 | Negative | Non-discounted products adjacent to discounted ones show no 'Sale' tag

**Priority:** P2
**Type:** Negative
**AC Reference:** AC-2

```gherkin
Scenario: Non-discounted products adjacent to discounted ones show no 'Sale' tag
  Given a guest user is not logged in to automationteststore.com
  When the user opens the Home page at "https://automationteststore.com"
  And the user scrolls to the "LATEST PRODUCTS" section which contains both discounted and full-price products
  Then the product "Total Moisture Facial Cream" at full price "$38.00" does not display a "Sale" tag
  And the product "Flash Bronzer Body Gel" at full price "$29.00" does not display a "Sale" tag
```

---

## TC-SALE-006 | P3 | Positive | Authenticated user sees same 'Sale' tags as guest user on Home page

**Priority:** P3
**Type:** Positive
**AC Reference:** AC-1

```gherkin
Scenario: Authenticated user sees same 'Sale' tags as guest user on Home page
  Given the user is logged in to automationteststore.com as "karl@example.com" with password "karl123"
  When the user navigates to the Home page at "https://automationteststore.com"
  And the user scrolls to the "LATEST PRODUCTS" section
  Then every product with a strikethrough price displays a "Sale" tag
  And no full-price product displays a "Sale" tag
```

**Notes:** The welcome header "Welcome back karl" visible in the screenshot confirms the logged-in state. Adjust test credentials to a valid test account.

---

## TC-SALE-007 | P3 | Edge | Navigating from a listing 'Sale'-tagged product to its detail view removes the 'Sale' tag

**Priority:** P3
**Type:** Edge
**AC Reference:** AC-3

```gherkin
Scenario: Navigating from a listing 'Sale'-tagged product to its detail view removes the 'Sale' tag
  Given a guest user is on the Home page at "https://automationteststore.com"
  And the "LATEST PRODUCTS" section shows the product "Absolute Anti-Age Spot Replenishing + Unifying Treatment SPF 15" with a "Sale" tag
  When the user clicks on that product to open its view-item page
  Then the product detail page loads successfully
  And the original price is displayed with a strikethrough
  And no "Sale" tag is visible on the product detail page
```

---

## TC-SALE-008 | P4 | Edge | Product detail page accessed directly via URL shows only strikethrough price, no 'Sale' tag

**Priority:** P4
**Type:** Edge
**AC Reference:** AC-3

```gherkin
Scenario: Product detail page accessed directly via URL shows only strikethrough price, no 'Sale' tag
  Given a guest user has not visited any listing page in the current session
  When the user opens the product detail page directly at "https://automationteststore.com/index.php?rt=product/product&product_id=51"
  Then the page displays the current price "$19.00"
  And the original price "$36.00" is shown with a strikethrough style
  And no "Sale" tag is visible anywhere on the product detail page
```

---

## TC-SALE-009 | P4 | Edge | Discounted product in the 'Featured' section of the Home page displays 'Sale' tag

**Priority:** P4
**Type:** Edge
**AC Reference:** AC-1

```gherkin
Scenario: Discounted product in the 'Featured' section of the Home page displays 'Sale' tag
  Given a guest user is not logged in to automationteststore.com
  When the user opens the Home page at "https://automationteststore.com"
  And the user views the "FEATURED" products section
  Then any product in that section with a strikethrough original price displays a "Sale" tag
  And any product in that section without a price reduction does not display a "Sale" tag
```

**Notes:** The screenshot shows the Featured section contains SKINSHEEN BRONZER STICK, BENEFIT GIRL MEETS PEARL, BENEFIT BELLA BAMBA, and TROPIQUES MINERALE LOOSE BRONZER. Verify which of these are discounted before execution to determine pass/fail criteria.

---

## Coverage Matrix

| AC | Description | Test IDs |
|----|-------------|----------|
| AC-1 | All products with a price reduction on a listing page must display a 'Sale' tag | TC-SALE-001, TC-SALE-004, TC-SALE-006, TC-SALE-009 |
| AC-2 | Products not on sale must not display a 'Sale' tag | TC-SALE-003, TC-SALE-005 |
| AC-3 | On the view-item screen of a discounted product, no 'Sale' tag is shown — only the strikethrough price | TC-SALE-002, TC-SALE-007, TC-SALE-008 |

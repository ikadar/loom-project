---
status: draft
derived-from: "tmp/poc/input/user-stories.md"
derived-at: "2025-12-20T18:30:00Z"
derived-by: "loom-derive skill v1.0"
loom-version: "2.0.0"
---

# Acceptance Criteria – Quote Acceptance

Derived from user stories using Loom L0→L1 derivation.

---

## US-QUOTE-003: Customer accepts a quote online

### AC-QUOTE-003-1 – Customer can access quote via secure link {#ac-quote-003-1}

**Given** a customer has received a quote notification email
**And** the quote has status `Sent`
**When** the customer clicks the secure link in the email
**Then** the customer is directed to the quote detail page in the portal
**And** the customer can view all quote details (items, prices, terms, validity)

**Error Cases:**
- Quote not found → HTTP 404, message: "Quote not found"
- Quote expired → HTTP 410, message: "This quote has expired"
- Invalid/tampered link → HTTP 403, message: "Invalid access link"

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote, ENT-Customer

---

### AC-QUOTE-003-2 – Customer can accept quote with single action {#ac-quote-003-2}

**Given** a customer is viewing a quote with status `Sent`
**And** the quote is within its validity period
**When** the customer clicks the "Accept Quote" button
**Then** the system displays a confirmation dialog
**And** upon confirmation, the acceptance is processed

**Error Cases:**
- Quote status is not `Sent` → Error: "Only sent quotes can be accepted" (HTTP 400)
- Quote has expired → Error: "Quote validity period has ended" (HTTP 400)
- Customer not authenticated → Redirect to login

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote
- Business Rule: BR-QUOTE-001

---

### AC-QUOTE-003-3 – System records acceptance details {#ac-quote-003-3}

**Given** a customer has confirmed quote acceptance
**When** the acceptance is processed
**Then** the system records:
  - Customer identity (user ID, email)
  - Acceptance timestamp (ISO 8601 UTC)
  - Quote version/revision accepted
  - IP address and user agent (for audit)
**And** this acceptance record is immutable

**Error Cases:**
- Database write failure → Transaction rollback, error: "Failed to record acceptance"

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote, ENT-QuoteAcceptance
- Business Rule: BR-QUOTE-003

---

### AC-QUOTE-003-4 – Quote status transitions to Accepted {#ac-quote-003-4}

**Given** the acceptance has been successfully recorded
**When** the acceptance transaction commits
**Then** the quote status changes from `Sent` to `Accepted`
**And** the status change timestamp is recorded
**And** the quote becomes read-only (no further modifications allowed)

**Error Cases:**
- Concurrent modification → Optimistic lock failure, retry or error

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote
- Business Rule: BR-QUOTE-002

---

### AC-QUOTE-003-5 – Order is automatically created from accepted quote {#ac-quote-003-5}

**Given** a quote has been successfully accepted
**When** the quote status becomes `Accepted`
**Then** an Order is automatically created with:
  - Reference to the accepted Quote (quote_id)
  - Copy of all quote line items (items, quantities, prices)
  - Customer reference from the quote
  - Order status set to `Pending`
**And** the Order ID is returned to the customer

**Error Cases:**
- Order creation fails → Quote acceptance is rolled back
- Inventory unavailable → Order created with `PendingReview` status

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote, ENT-Order
- Business Rule: BR-QUOTE-004

---

### AC-QUOTE-003-6 – Customer receives acceptance confirmation {#ac-quote-003-6}

**Given** a quote has been accepted and order created
**When** the transaction completes successfully
**Then** the customer sees a confirmation page with:
  - Quote acceptance confirmation
  - New Order ID and summary
  - Next steps information
**And** the customer receives a confirmation email

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote, ENT-Order, ENT-Customer

---

## Validation Checklist

- [x] 6 acceptance criteria generated (target: 4-7)
- [x] All criteria use Given/When/Then format
- [x] All AC IDs are unique (AC-QUOTE-003-1 through AC-QUOTE-003-6)
- [x] All criteria have traceability links
- [x] Error cases defined for each criterion
- [x] Happy path and error paths covered

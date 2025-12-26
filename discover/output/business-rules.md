---
status: draft
derived-from: "loom-project/poc/input/user-stories.md"
derived-at: "2025-12-21T12:30:00Z"
derived-by: "loom-derive skill v2.0 (Structured Interview)"
loom-version: "3.0.0"
structured-interview:
  decision-points-resolved: 6
  from-user-answers: 4
  from-input: 2
---

# Business Rules – Quote Acceptance

## US-QUOTE-003 – Customer accepts a quote online

### BR-QUOTE-001 – Only Sent quotes can be accepted

**Rule:**
A quote can only be accepted if its current status is `Sent`.

**Invariant:**
Quote.accept() MUST only succeed when Quote.status === "Sent"

**Enforcement:**
- **Precondition:** Quote.status === "Sent"
- **Violation Behavior:** Reject acceptance, return error
- **Error Code:** `INVALID_QUOTE_STATUS`

**Decision Points Resolved:**
- ST-1: Only from "Sent" status (From input)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-1
- Entity: ENT-Quote

---

### BR-QUOTE-002 – Quote must be within validity period

**Rule:**
A quote can only be accepted if the current date is before or equal to the quote's expiration date.

**Invariant:**
Quote.accept() MUST only succeed when NOW() <= Quote.validUntil

**Enforcement:**
- **Precondition:** Current timestamp <= Quote.validUntil
- **Violation Behavior:** Reject acceptance with blocking error
- **Error Code:** `QUOTE_EXPIRED`

**Decision Points Resolved:**
- EH-1: Expired quote is blocking error (User answer)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-1
- Entity: ENT-Quote

---

### BR-QUOTE-003 – Quote acceptance creates Order

**Rule:**
When a quote is accepted, an Order MUST be automatically created from the quote data.

**Invariant:**
For every Quote with status "Accepted", exactly one Order MUST exist with Order.quoteId === Quote.id

**Enforcement:**
- **Precondition:** Quote.accept() succeeds
- **Violation Behavior:** Transaction rollback if Order creation fails
- **Error Code:** `ORDER_CREATION_FAILED`

**Decision Points Resolved:**
- SE-3: Order created automatically (From input)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-2
- Entity: ENT-Quote, ENT-Order

---

### BR-QUOTE-004 – Acceptance reversible until Order fulfilled

**Rule:**
A quote acceptance can be reversed (and its Order cancelled) only if the Order has not been fulfilled.

**Invariant:**
Quote.reverseAcceptance() MUST only succeed when Order.status NOT IN ("Fulfilled", "Shipped", "PartiallyShipped")

**Enforcement:**
- **Precondition:** Associated Order is not fulfilled or shipped
- **Violation Behavior:** Reject reversal, return conflict error
- **Error Code:** `ORDER_ALREADY_FULFILLED`

**Decision Points Resolved:**
- ST-2: Reversible until Order is fulfilled (User answer)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-3
- Entity: ENT-Quote, ENT-Order

---

### BR-QUOTE-005 – Organization-level authorization for quote acceptance

**Rule:**
Any authenticated user belonging to the customer's organization can accept a quote addressed to that organization.

**Invariant:**
Quote.accept(user) MUST only succeed when user.organizationId === Quote.customer.organizationId

**Enforcement:**
- **Precondition:** Accepting user belongs to customer's organization
- **Violation Behavior:** Reject with authorization error
- **Error Code:** `UNAUTHORIZED_ORGANIZATION`

**Decision Points Resolved:**
- AU-1: Any user from customer's organization (User answer)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-1, AC-QUOTE-003-5
- Entity: ENT-Quote, ENT-Customer, ENT-User

---

### BR-QUOTE-006 – Mandatory notifications on acceptance

**Rule:**
When a quote is accepted, notifications MUST be sent to: the sales rep, the customer, and the fulfillment team.

**Invariant:**
Quote.accept() MUST trigger notifications to all three parties before completion

**Enforcement:**
- **Precondition:** Quote acceptance succeeds
- **Violation Behavior:** Log warning if notification fails, but don't block acceptance
- **Error Code:** `NOTIFICATION_FAILED` (non-blocking)

**Decision Points Resolved:**
- SE-1: Notify sales rep + customer + fulfillment team (User answer)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-4
- Entity: ENT-Quote, ENT-Notification

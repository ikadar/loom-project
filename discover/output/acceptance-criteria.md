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

# Acceptance Criteria – Quote Acceptance

## US-QUOTE-003 – Customer accepts a quote online

### AC-QUOTE-003-1 – Accept quote from Sent status only

**Given** a customer user from the customer's organization is authenticated
**And** a quote exists with status `Sent`
**And** the quote is within its validity period
**When** the customer accesses the quote via secure link or portal
**And** clicks the "Accept" action
**Then** the system records the acceptance with:
  - User identity (authenticated user from customer's organization)
  - Timestamp (ISO 8601 format)
  - Quote version accepted
**And** the quote status changes to `Accepted`

**Error Cases:**
- Quote status is not `Sent` → Error: "Only sent quotes can be accepted" (HTTP 400)
- Quote has expired → Error: "Quote validity period has ended" (HTTP 400)
- User not from customer's organization → Error: "Unauthorized" (HTTP 403)

**Decision Points Resolved:**
- ST-1: Only from "Sent" status (From input)
- AU-1: Any user from customer's organization (User answer)
- EH-1: Expired quote is blocking error (User answer)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote, ENT-Customer

---

### AC-QUOTE-003-2 – Order creation on acceptance

**Given** a quote has been successfully accepted
**When** the acceptance is confirmed
**Then** an Order is automatically created with:
  - Reference to the accepted Quote (quoteId)
  - All line items copied from the Quote
  - Customer information from the Quote
  - Order status set to `Pending`
**And** the orderId is returned in the acceptance response

**Error Cases:**
- Order creation fails → Transaction rollback, quote remains `Sent`, error logged

**Decision Points Resolved:**
- SE-3: Order created automatically (From input)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote, ENT-Order

---

### AC-QUOTE-003-3 – Acceptance can be reversed until Order fulfilled

**Given** a quote has status `Accepted`
**And** the associated Order has NOT been fulfilled (status is not `Fulfilled` or `Shipped`)
**When** an authorized user requests to reverse the acceptance
**Then** the Quote status reverts to `Sent`
**And** the associated Order is cancelled (status → `Cancelled`)
**And** an audit trail entry is created recording the reversal
**And** the quote can be accepted again

**Error Cases:**
- Order already fulfilled → Error: "Cannot reverse, order is already fulfilled" (HTTP 409)
- Order partially shipped → Error: "Cannot reverse, order has shipments" (HTTP 409)

**Decision Points Resolved:**
- ST-2: Reversible until Order is fulfilled (User answer)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote, ENT-Order

---

### AC-QUOTE-003-4 – Notifications on acceptance

**Given** a quote has been successfully accepted
**When** the acceptance is processed
**Then** the following notifications are sent:
  - **Sales rep** who created the quote: email + in-app notification
  - **Customer** (accepting user): confirmation email with order details
  - **Fulfillment team**: work queue notification with order reference

**Decision Points Resolved:**
- SE-1: Notify sales rep + customer + fulfillment team (User answer)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote, ENT-Order, ENT-Notification

---

### AC-QUOTE-003-5 – Secure access to quote

**Given** a quote has been sent to a customer
**When** the customer receives the quote link
**Then** the link contains a secure token (UUID or signed URL)
**And** the customer must authenticate before viewing quote details
**And** only users from the customer's organization can access the quote

**Error Cases:**
- Invalid or expired token → Error: "Invalid quote link" (HTTP 404)
- User from different organization → Error: "Access denied" (HTTP 403)

**Decision Points Resolved:**
- AU-1: Any user from customer's organization (User answer)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote, ENT-Customer

---

### AC-QUOTE-003-6 – Audit trail for acceptance

**Given** any acceptance-related action occurs (accept, reverse)
**When** the action is processed
**Then** an audit log entry is created with:
  - Action type (ACCEPT, REVERSE)
  - User who performed the action
  - Timestamp
  - Quote ID and version
  - Order ID (if applicable)
  - Previous and new status

**Decision Points Resolved:**
- SE-2: Audit trail required for mutations (Default - conservative)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote, ENT-AuditLog

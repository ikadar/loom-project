---
status: draft
derived-from:
  - "tmp/poc/output/acceptance-criteria.md"
  - "tmp/poc/output/business-rules.md"
derived-at: "2025-12-20T18:45:00Z"
derived-by: "loom-derive-l2 skill v1.0"
loom-version: "2.0.0"
---

# Sequence Design – Quote Acceptance

This document provides textual and Mermaid-based sequence specifications
for the Quote Acceptance → Order Creation workflow.

---

## 1. View Quote via Secure Link

**Trigger:** Customer clicks secure link in quote notification email.

**Traceability:** AC-QUOTE-003-1

**Sequence:**
1. Customer clicks secure link containing `quoteId` and `accessToken`.
2. Customer Portal sends `GetQuote(quoteId, accessToken)` to **Quote Service**.
3. Quote Service validates:
   - Access token is valid and not expired
   - Quote exists
   - Quote belongs to customer
4. Quote Service checks quote expiry status.
5. Quote Service returns quote details with `canAccept` flag.
6. Customer Portal displays quote details page.

**Outcome:**
Customer can view quote details and decide whether to accept.

**Error Paths:**
- Invalid token → 403 INVALID_ACCESS_TOKEN → Error page
- Quote not found → 404 QUOTE_NOT_FOUND → Error page
- Quote expired → 410 QUOTE_EXPIRED → Show expired notice

```mermaid
sequenceDiagram
    participant C as Customer
    participant Portal as Customer Portal
    participant QS as Quote Service
    participant DB as Database

    C->>Portal: Click secure link
    Portal->>QS: GetQuote(quoteId, accessToken)
    QS->>DB: Validate access token
    alt Invalid token
        QS-->>Portal: 403 INVALID_ACCESS_TOKEN
        Portal-->>C: Error page
    end
    QS->>DB: Find quote by ID
    alt Quote not found
        QS-->>Portal: 404 QUOTE_NOT_FOUND
        Portal-->>C: Error page
    end
    QS->>QS: Check expiry (validUntil vs NOW)
    QS->>QS: Set canAccept flag
    QS-->>Portal: 200 OK (quote details)
    Portal-->>C: Display quote page
```

---

## 2. Quote Acceptance → Order Creation (Happy Path)

**Trigger:** Customer clicks "Accept Quote" button and confirms.

**Traceability:** AC-QUOTE-003-2, AC-QUOTE-003-3, AC-QUOTE-003-4, AC-QUOTE-003-5, AC-QUOTE-003-6

**Sequence:**
1. Customer clicks "Accept Quote" button.
2. Customer Portal displays confirmation dialog.
3. Customer confirms acceptance.
4. Customer Portal sends `AcceptQuote(quoteId, customerId, true)` to **Quote Service**.
5. Quote Service validates:
   - Quote exists
   - Quote status is `Sent` (BR-QUOTE-001)
   - Quote not expired (BR-QUOTE-006)
   - Customer is authorized
6. Quote Service begins transaction:
   a. Create QuoteAcceptance audit record (BR-QUOTE-003)
   b. Update Quote status → `Accepted`
   c. Mark Quote as immutable (BR-QUOTE-002)
7. Quote Service emits domain event: **`QuoteAccepted`**.
8. Order Service receives `QuoteAccepted` event.
9. Order Service creates Order from Quote:
   - Copy line items
   - Set `quote_id` reference (BR-QUOTE-005)
   - Set status to `Pending`
10. Order Service emits domain event: **`OrderCreated`**.
11. Quote Service returns success with `orderId`.
12. Customer Portal displays confirmation page.
13. Notification Service sends confirmation email.

**Outcome:**
Quote is accepted, Order is created, customer has order reference and confirmation.

```mermaid
sequenceDiagram
    participant C as Customer
    participant Portal as Customer Portal
    participant QS as Quote Service
    participant OS as Order Service
    participant NS as Notification Service
    participant DB as Database

    C->>Portal: Click "Accept Quote"
    Portal->>C: Show confirmation dialog
    C->>Portal: Confirm acceptance
    Portal->>QS: AcceptQuote(quoteId, customerId, true)

    Note over QS: Validation Phase
    QS->>DB: Find quote by ID
    QS->>QS: Validate status = "Sent"
    QS->>QS: Validate not expired
    QS->>QS: Validate customer authorized

    Note over QS: Transaction Begin
    QS->>DB: Create QuoteAcceptance record
    QS->>DB: Update quote → Accepted
    QS->>DB: Set quote immutable flag
    Note over QS: Transaction Commit

    QS-->>OS: QuoteAccepted event

    OS->>OS: Create Order from Quote
    OS->>DB: Save Order (quote_id reference)
    OS-->>OS: OrderCreated event

    QS-->>Portal: 200 OK {orderId, acceptedAt}
    Portal-->>C: Confirmation page

    QS-->>NS: Send confirmation email
    NS-->>C: Email: "Quote accepted, Order #..."
```

---

## 3. Quote Acceptance – Validation Failures

**Trigger:** Customer attempts to accept a quote that fails validation.

**Traceability:** AC-QUOTE-003-2 (error cases), BR-QUOTE-001, BR-QUOTE-006

**Sequence:**
1. Customer confirms acceptance.
2. Customer Portal sends `AcceptQuote(...)` to **Quote Service**.
3. Quote Service validates and finds issue:
   - Quote not in `Sent` status, OR
   - Quote has expired
4. Quote Service returns appropriate error.
5. Customer Portal displays error message.

**Outcome:**
Acceptance is rejected, quote remains unchanged.

**Error Scenarios:**

| Scenario | Error Code | User Message |
|----------|------------|--------------|
| Quote is Draft | INVALID_QUOTE_STATUS | "This quote hasn't been sent yet" |
| Quote already Accepted | INVALID_QUOTE_STATUS | "This quote has already been accepted" |
| Quote was Rejected | INVALID_QUOTE_STATUS | "This quote was rejected" |
| Quote expired | QUOTE_EXPIRED | "This quote has expired" |

```mermaid
sequenceDiagram
    participant C as Customer
    participant Portal as Customer Portal
    participant QS as Quote Service
    participant DB as Database

    C->>Portal: Confirm acceptance
    Portal->>QS: AcceptQuote(quoteId, customerId, true)
    QS->>DB: Find quote by ID

    alt Quote not in Sent status
        QS->>QS: status !== "Sent"
        QS-->>Portal: 400 INVALID_QUOTE_STATUS
        Portal-->>C: "Only sent quotes can be accepted"
    else Quote expired
        QS->>QS: NOW() > validUntil
        QS-->>Portal: 400 QUOTE_EXPIRED
        Portal-->>C: "This quote has expired"
    else Not authorized
        QS->>QS: customerId mismatch
        QS-->>Portal: 403 FORBIDDEN
        Portal-->>C: "Not authorized"
    end
```

---

## 4. Quote Acceptance – Transaction Rollback

**Trigger:** Acceptance succeeds but Order creation fails.

**Traceability:** AC-QUOTE-003-5 (error case), BR-QUOTE-004

**Sequence:**
1. Quote Service validates and updates quote.
2. Order Service attempts to create Order.
3. Order creation fails (e.g., database error).
4. Entire transaction is rolled back.
5. Quote remains in `Sent` status.
6. Customer receives error message.

**Outcome:**
No partial state - either both succeed or both fail.

```mermaid
sequenceDiagram
    participant C as Customer
    participant Portal as Customer Portal
    participant QS as Quote Service
    participant OS as Order Service
    participant DB as Database

    C->>Portal: Confirm acceptance
    Portal->>QS: AcceptQuote(...)

    Note over QS: Validation passes
    QS->>DB: Create QuoteAcceptance record
    QS->>DB: Update quote → Accepted

    QS-->>OS: QuoteAccepted event
    OS->>OS: Create Order from Quote
    OS->>DB: Save Order

    Note over DB: Database error!
    DB-->>OS: Error
    OS-->>QS: ORDER_CREATION_FAILED

    Note over QS: Transaction Rollback
    QS->>DB: Rollback all changes
    Note over QS: Quote remains "Sent"

    QS-->>Portal: 500 ORDER_CREATION_FAILED
    Portal-->>C: "Failed to process. Please try again."
```

---

## 5. Concurrent Acceptance Handling

**Trigger:** Two users attempt to accept the same quote simultaneously.

**Traceability:** AC-QUOTE-003-4 (concurrent modification), BR-QUOTE-002

**Sequence:**
1. User A and User B both view the quote.
2. Both click "Accept" at nearly the same time.
3. User A's request arrives first:
   - Validation passes
   - Quote → Accepted
   - Order created
4. User B's request arrives:
   - Quote status is now `Accepted`
   - Validation fails (not `Sent`)
5. User B receives error.

**Outcome:**
Only one acceptance succeeds, no duplicate orders.

```mermaid
sequenceDiagram
    participant A as User A
    participant B as User B
    participant QS as Quote Service
    participant DB as Database

    Note over A,B: Both viewing quote #123 (status: Sent)

    A->>QS: AcceptQuote(123, userA)
    B->>QS: AcceptQuote(123, userB)

    Note over QS: Request A processed first
    QS->>DB: Lock quote #123
    QS->>QS: Validate status = "Sent" ✓
    QS->>DB: Update quote → Accepted
    QS->>DB: Release lock
    QS-->>A: 200 OK (orderId: 456)

    Note over QS: Request B processed second
    QS->>DB: Lock quote #123
    QS->>QS: Validate status = "Sent" ✗
    Note over QS: status is now "Accepted"
    QS->>DB: Release lock
    QS-->>B: 400 INVALID_QUOTE_STATUS
```

---

## Notes

- Sequence diagrams use Mermaid syntax for rendering.
- All flows include validation, happy path, and error handling.
- Transaction boundaries are explicitly shown.
- Domain events connect services in event-driven architecture.
- Concurrent access is handled via optimistic locking.

---

## Validation Checklist

- [x] 5 sequence diagrams created
- [x] Happy path fully documented
- [x] All error paths shown
- [x] Transaction boundaries marked
- [x] Domain events included (QuoteAccepted, OrderCreated)
- [x] Concurrent access scenario documented
- [x] All diagrams use valid Mermaid syntax
- [x] Traceability to AC and BR maintained

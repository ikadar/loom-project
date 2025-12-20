---
status: draft
derived-from: "tmp/poc/input/user-stories.md"
derived-at: "2025-12-20T18:30:00Z"
derived-by: "loom-derive skill v1.0"
loom-version: "2.0.0"
---

# Business Rules & Domain Invariants – Quote Acceptance

Derived from user stories using Loom L0→L1 derivation.

These rules are technology-agnostic and must hold regardless of implementation.

---

## Quote Status Transitions

### BR-QUOTE-001 – Only Sent quotes can be accepted {#br-quote-001}

**Rule:**
A quote can only be accepted if its current status is `Sent`.
Quotes in any other status (Draft, Expired, Rejected, Accepted) cannot be accepted.

**Invariant:**
```
Quote.accept() MUST only succeed when Quote.status === "Sent"
```

**Enforcement:**
- **Precondition:** `Quote.status === "Sent"`
- **Violation Behavior:** Reject acceptance request, return error response
- **Error Code:** `INVALID_QUOTE_STATUS`
- **HTTP Status:** 400 Bad Request

**Test Scenarios:**
```
✓ Accept quote with status="Sent" → Success
✗ Accept quote with status="Draft" → INVALID_QUOTE_STATUS
✗ Accept quote with status="Expired" → INVALID_QUOTE_STATUS
✗ Accept quote with status="Rejected" → INVALID_QUOTE_STATUS
✗ Accept quote with status="Accepted" → INVALID_QUOTE_STATUS (idempotent or error)
```

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-2
- Entity: ENT-Quote

---

### BR-QUOTE-002 – Accepted quotes are immutable {#br-quote-002}

**Rule:**
Once a quote reaches `Accepted` status, its content (items, prices, terms) MUST NOT be modified.
Only metadata (e.g., related order reference) may be updated.

**Invariant:**
```
IF Quote.status === "Accepted" THEN Quote.content IS immutable
```

**Enforcement:**
- **Precondition:** Any modification attempt on Quote with status `Accepted`
- **Violation Behavior:** Reject modification, return error
- **Error Code:** `QUOTE_IMMUTABLE`
- **HTTP Status:** 409 Conflict

**Test Scenarios:**
```
✗ Modify line items on Accepted quote → QUOTE_IMMUTABLE
✗ Change prices on Accepted quote → QUOTE_IMMUTABLE
✗ Update terms on Accepted quote → QUOTE_IMMUTABLE
✓ Add order reference to Accepted quote → Success (metadata only)
```

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-4
- Entity: ENT-Quote

---

### BR-QUOTE-003 – Quote acceptance must be recorded with full audit trail {#br-quote-003}

**Rule:**
Every quote acceptance MUST record: customer identity, timestamp, quote version, and source metadata.
This record MUST be immutable for audit purposes.

**Invariant:**
```
FOR EVERY Quote WHERE status === "Accepted":
  QuoteAcceptance record MUST exist WITH:
    - customer_id NOT NULL
    - accepted_at NOT NULL
    - quote_version NOT NULL
```

**Enforcement:**
- **Precondition:** Quote.accept() transaction
- **Violation Behavior:** Transaction rollback if audit record cannot be created
- **Error Code:** `AUDIT_RECORD_FAILED`
- **Implementation:** Database transaction ensures atomicity

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-3
- Entity: ENT-Quote, ENT-QuoteAcceptance

---

## Quote-Order Relationship

### BR-QUOTE-004 – Quote acceptance MUST create an Order {#br-quote-004}

**Rule:**
When a quote is accepted, an Order MUST be automatically created.
The order creation is part of the same transaction as the quote acceptance.

**Invariant:**
```
FOR EVERY Quote WHERE status === "Accepted":
  EXACTLY ONE Order MUST exist WHERE Order.quote_id === Quote.id
```

**Enforcement:**
- **Precondition:** Quote.accept() succeeds
- **Violation Behavior:** Rollback entire transaction if Order creation fails
- **Error Code:** `ORDER_CREATION_FAILED`
- **Implementation:** Transactional saga or two-phase commit

**Test Scenarios:**
```
✓ Accept quote → Order created with correct quote_id
✓ Accept quote → Order contains all quote line items
✗ Order creation fails → Quote remains in "Sent" status (rollback)
```

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-5
- Entity: ENT-Quote, ENT-Order

---

### BR-QUOTE-005 – Order must reference originating Quote {#br-quote-005}

**Rule:**
An Order created from quote acceptance MUST maintain a reference to the originating Quote.
This reference MUST NOT be modified after Order creation.

**Invariant:**
```
IF Order.origin === "QUOTE_ACCEPTANCE" THEN:
  Order.quote_id MUST NOT be NULL
  Order.quote_id MUST reference valid Quote
  Order.quote_id IS immutable
```

**Enforcement:**
- **Precondition:** Order creation from quote acceptance
- **Violation Behavior:** Reject Order modification that would change quote_id
- **Error Code:** `IMMUTABLE_REFERENCE`

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-5
- Entity: ENT-Order, ENT-Quote

---

## Quote Validity

### BR-QUOTE-006 – Expired quotes cannot be accepted {#br-quote-006}

**Rule:**
A quote cannot be accepted if the current date is past the quote's validity end date.
The validity check MUST occur at acceptance time, not at page load.

**Invariant:**
```
Quote.accept() MUST fail IF NOW() > Quote.valid_until
```

**Enforcement:**
- **Precondition:** `NOW() <= Quote.valid_until`
- **Violation Behavior:** Reject acceptance with expiry error
- **Error Code:** `QUOTE_EXPIRED`
- **HTTP Status:** 410 Gone

**Test Scenarios:**
```
✓ Accept quote within validity period → Success
✗ Accept quote past valid_until → QUOTE_EXPIRED
✗ Accept quote on valid_until date at 23:59:59 → Success
✗ Accept quote on valid_until date + 1 second → QUOTE_EXPIRED
```

**Traceability:**
- User Story: user-stories.md#us-quote-003 (implied from context)
- Acceptance Criteria: AC-QUOTE-003-2
- Entity: ENT-Quote

---

## Cross-Entity Invariants

### INV-QUOTE-001 – Quote-Order consistency {#inv-quote-001}

**Invariant:**
All Orders created from quote acceptance MUST have line items that match the accepted Quote's line items at acceptance time.

```
FOR Order WHERE origin === "QUOTE_ACCEPTANCE":
  Order.line_items MUST equal Quote.line_items (at acceptance snapshot)
```

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Acceptance Criteria: AC-QUOTE-003-5
- Entity: ENT-Quote, ENT-Order, ENT-LineItem

---

## Validation Checklist

- [x] 6 business rules + 1 invariant generated
- [x] All BR IDs are unique (BR-QUOTE-001 through BR-QUOTE-006, INV-QUOTE-001)
- [x] All rules have enforcement mechanisms defined
- [x] All rules have error codes specified
- [x] All rules have traceability links
- [x] Test scenarios included for key rules

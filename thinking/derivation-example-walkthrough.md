---
date: 2025-12-19
title: Documentation Derivation - Complete Walkthrough Example
purpose: Concrete end-to-end example of how docs are derived in Loom
related: documentation-derivation-strategy.md
---

# Documentation Derivation - Complete Walkthrough Example

## 🎯 Scenario

**User request:** "I want customers to be able to cancel a quote before it's accepted."

Let's see how this simple request flows through all 4 derivation levels!

---

## 📝 LEVEL 0: FOUNDATIONAL (Human Input)

### Step 1: Human updates domain-vocabulary.md

```markdown
## Quote Cancellation

- **Definition:** The act of invalidating a Quote before it is accepted, either by the customer or by the sales representative.
- **Notes:** Only quotes with status `Draft` or `Sent` can be cancelled. Accepted, Rejected, or Expired quotes cannot be cancelled.
- **Related terms:** Quote, Quote Status, Order.
```

**Human effort:** 2 minutes

---

### Step 2: Human updates user-stories.md

```markdown
### US-QUOTE-003 – Cancel a quote {#us-quote-003}

**As a** customer
**I want** to cancel a pending quote
**So that** I can withdraw my interest without it appearing as rejected.

**Context:**
If I submitted an RFQ but no longer need the quote, I should be able to cancel it myself.
```

**Human effort:** 3 minutes

**Total L0 effort:** 5 minutes

---

## 🤖 LEVEL 1: PRIMARY DERIVATION (AI-Generated)

### Step 3: AI derives acceptance-criteria.md

**Input:**
- user-stories.md#us-quote-003
- domain-vocabulary.md#quote-cancellation

**AI Process:**

```
Loom AcceptanceCriteriaAgent:

1. Parsing US-QUOTE-003...
2. Extracting intent: "cancel pending quote"
3. Identifying constraints from vocabulary:
   - Can cancel: Draft, Sent
   - Cannot cancel: Accepted, Rejected, Expired
4. Generating acceptance criteria (Given/When/Then format)...
5. Adding traceability links...
6. Generating IDs: AC-QUOTE-003-1, AC-QUOTE-003-2, AC-QUOTE-003-3
```

**Output:**

```markdown
## US-QUOTE-003: Cancel a quote {#us-quote-003}

### AC-QUOTE-003-1: Customer can cancel quote with status "Sent" {#ac-quote-003-1}

**Given** a customer has a quote with status `Sent`
**When** the customer clicks "Cancel Quote"
**Then** the quote status changes to `Cancelled`
**And** the customer sees a confirmation message
**And** the sales representative receives a notification

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Vocabulary: domain-vocabulary.md#quote-cancellation
- Entity: ENT-Quote

---

### AC-QUOTE-003-2: Customer cannot cancel accepted quote {#ac-quote-003-2}

**Given** a customer has a quote with status `Accepted`
**When** the customer attempts to cancel the quote
**Then** the system returns an error "Cannot cancel an accepted quote"
**And** the quote status remains `Accepted`

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Vocabulary: domain-vocabulary.md#quote-cancellation
- Business Rule: BR-QUOTE-003 (to be derived)
- Entity: ENT-Quote

---

### AC-QUOTE-003-3: System records cancellation timestamp and reason {#ac-quote-003-3}

**Given** a customer cancels a quote
**When** the cancellation is confirmed
**Then** the system records:
  - cancellationTimestamp (datetime)
  - cancelledBy (customer ID)
  - cancellationReason (optional free text)
**And** this information is visible to the sales representative

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Entity: ENT-Quote
```

**AI confidence:** 92% (high → auto-generate, but require human approval)

**Human review:** "Looks good, approved!"

**L1 effort:** 2 minutes (human review)

---

### Step 4: AI derives business-rules.md

**Input:**
- domain-vocabulary.md#quote-cancellation
- acceptance-criteria.md#ac-quote-003-2

**AI Process:**

```
Loom BusinessRulesAgent:

1. Extracting constraints from vocabulary...
2. Identifying invariants from AC-QUOTE-003-2...
3. Generating business rule BR-QUOTE-003...
```

**Output:**

```markdown
### BR-QUOTE-003: Quote cancellation status rules {#br-quote-003}

**Rule:**
A Quote can only be cancelled if its current status is:
- `Draft` (internal quote not yet sent)
- `Sent` (quote communicated to customer, awaiting response)

**Invariant:**
Quotes with status `Accepted`, `Rejected`, or `Expired` CANNOT be cancelled.

**Rationale:**
- `Accepted` quotes may already have triggered order creation
- `Rejected` quotes are already finalized
- `Expired` quotes are no longer active

**Violation:**
If cancellation attempted on non-cancellable status → throw `InvalidQuoteStatusError`

**Traceability:**
- Vocabulary: domain-vocabulary.md#quote-cancellation
- User Story: US-QUOTE-003
- Acceptance Criteria: AC-QUOTE-003-2
- Entity: ENT-Quote
- Implementation: src/domain/Quote.ts:cancel()
```

**AI confidence:** 95% (very high)

**Human review:** Auto-approved (high confidence)

**L1 effort:** 0 minutes

---

### Step 5: AI updates domain-model.md

**Input:**
- domain-vocabulary.md#quote-cancellation
- acceptance-criteria.md#ac-quote-003-3

**AI Process:**

```
Loom DomainModelAgent:

1. Analyzing new attributes from AC-QUOTE-003-3...
2. Updating Quote aggregate with cancellation fields...
3. Adding new status value: "Cancelled"
```

**Output:**

```markdown
### Quote (Aggregate Root) {#ent-quote}

**Key attributes:**
- `QuoteId` (value object) {#ent-quote:id}
- `Status` (value object) {#ent-quote:status}
  - Allowed values: `Draft`, `Sent`, `Accepted`, `Rejected`, `Expired`, **`Cancelled`** (NEW!)
- `CancellationTimestamp` (datetime, optional) {#ent-quote:cancellation-timestamp} (NEW!)
- `CancelledBy` (CustomerId, optional) {#ent-quote:cancelled-by} (NEW!)
- `CancellationReason` (string, optional) {#ent-quote:cancellation-reason} (NEW!)

**Operations:**
- `cancel(customerId: CustomerId, reason?: string): void` (NEW!)
  - **Precondition:** Status must be `Draft` or `Sent` (enforces BR-QUOTE-003)
  - **Postcondition:** Status changes to `Cancelled`, timestamp and cancellation fields set

**Traceability:**
- User Story: US-QUOTE-003
- Business Rule: BR-QUOTE-003
```

**AI confidence:** 90%

**Human review:** "Approved!"

**L1 effort:** 1 minute

---

## 🔧 LEVEL 2: SECONDARY DERIVATION (AI-Generated)

### Step 6: AI derives interface-contracts.md

**Input:**
- domain-model.md#ent-quote (updated with cancel operation)
- acceptance-criteria.md#ac-quote-003-1, #ac-quote-003-2, #ac-quote-003-3
- business-rules.md#br-quote-003

**AI Process:**

```
Loom InterfaceContractAgent:

1. Analyzing new operation: Quote.cancel()
2. Mapping to RESTful endpoint: POST /quotes/{id}/cancel
3. Generating request/response schemas from entity attributes...
4. Applying business rules as validation...
5. Defining error responses...
```

**Output:**

```markdown
## POST /api/quotes/{id}/cancel - Cancel quote {#api-post-quote-cancel}

**Operation:** Cancel a pending quote

**Path Parameters:**
- `id` (string, UUID) - Quote ID

**Request:**
```json
{
  "reason": "string (optional, max 500 chars)"
}
```

**Response (200 OK):**
```json
{
  "quoteId": "string (UUID)",
  "status": "Cancelled",
  "cancelledAt": "datetime (ISO 8601)",
  "cancelledBy": "string (customer ID)",
  "cancellationReason": "string (optional)"
}
```

**Validation Rules:**
- Implements: BR-QUOTE-003 (status must be Draft or Sent)
- Implements: AC-QUOTE-003-1 (confirmation)
- Implements: AC-QUOTE-003-3 (record timestamp and reason)

**Error Responses:**
- `404 Not Found` - Quote ID does not exist
- `403 Forbidden` - Customer does not own this quote
- `422 Unprocessable Entity` - Quote status is not cancellable (Accepted, Rejected, Expired)
  ```json
  {
    "error": "InvalidQuoteStatusError",
    "message": "Cannot cancel quote with status 'Accepted'",
    "currentStatus": "Accepted"
  }
  ```

**Traceability:**
- User Story: US-QUOTE-003
- Acceptance Criteria: AC-QUOTE-003-1, AC-QUOTE-003-2, AC-QUOTE-003-3
- Business Rule: BR-QUOTE-003
- Entity: ENT-Quote
- Operation: Quote.cancel()
```

**AI confidence:** 88%

**Human review:** Optional (L2 can be auto-approved)

**L2 effort:** 0 minutes (auto-approved)

---

### Step 7: AI derives sequence-design.md

**Input:**
- user-stories.md#us-quote-003
- interface-contracts.md#api-post-quote-cancel
- domain-model.md#ent-quote
- business-rules.md#br-quote-003

**AI Process:**

```
Loom SequenceDesignAgent:

1. Identifying actors: Customer, Portal, Quote Service, Notification Service
2. Mapping workflow from US-QUOTE-003...
3. Applying business rule BR-QUOTE-003 as decision point...
4. Generating sequence diagram (Mermaid syntax)...
```

**Output:**

```markdown
## SEQ-QUOTE-003: Cancel quote flow {#seq-quote-003}

**User Story:** US-QUOTE-003 - Cancel a quote

**Actors:**
- Customer (user)
- Web Portal (frontend)
- Quote Service (backend)
- Notification Service

**Sequence:**

\`\`\`mermaid
sequenceDiagram
    participant C as Customer
    participant P as Portal
    participant Q as Quote Service
    participant N as Notification Service

    C->>P: View quote details
    P->>P: Show "Cancel Quote" button (if status = Draft or Sent)
    C->>P: Click "Cancel Quote"
    P->>P: Show confirmation dialog
    C->>P: Confirm cancellation (optional: enter reason)
    P->>Q: POST /api/quotes/{id}/cancel

    alt Quote status is Draft or Sent (BR-QUOTE-003)
        Q->>Q: Validate status
        Q->>Q: Update Quote.status = Cancelled
        Q->>Q: Set cancellationTimestamp, cancelledBy, cancellationReason
        Q-->>N: Send QuoteCancelled event
        N->>N: Notify sales rep
        Q-->>P: 200 OK {quote details}
        P->>C: Show success message "Quote cancelled successfully"
    else Quote status is Accepted, Rejected, or Expired
        Q->>Q: Validate status
        Q-->>P: 422 Unprocessable Entity
        P->>C: Show error "Cannot cancel quote with status 'Accepted'"
    end

**Traceability:**
- User Story: US-QUOTE-003
- Acceptance Criteria: AC-QUOTE-003-1, AC-QUOTE-003-2
- Business Rule: BR-QUOTE-003
- API: API-POST-QUOTE-CANCEL
- Entity: ENT-Quote
```

**AI confidence:** 85%

**L2 effort:** 0 minutes (auto-approved)

---

## ✅ LEVEL 3: TERTIARY DERIVATION (AI-Generated)

### Step 8: AI derives test-case.md (TDAI!)

**Input:**
- acceptance-criteria.md#ac-quote-003-1, #ac-quote-003-2, #ac-quote-003-3
- interface-contracts.md#api-post-quote-cancel
- business-rules.md#br-quote-003

**AI Process:**

```
Loom TestGeneratorAgent (TDAI):

1. Analyzing acceptance criteria for testable scenarios...
2. Generating test plan:
   - Positive tests: 3 (AC-QUOTE-003-1, AC-QUOTE-003-3)
   - Negative tests: 4 (AC-QUOTE-003-2, invalid status, unauthorized, etc.)
   - Boundary tests: 2 (status transitions)
   - Hallucination prevention: 2 ("should NOT" tests)
3. Distributing across test pyramid:
   - Unit: 7 tests (70%)
   - Integration: 2 tests (20%)
   - E2E: 1 test (10%)
4. Generating test IDs: TC-QUOTE-003-001 to TC-QUOTE-003-010
5. Adding traceability...
```

**Output (Test Plan for Human Approval):**

```markdown
## Test Plan for US-QUOTE-003: Cancel Quote

**Total tests:** 10
**Distribution:**
- Unit: 7 (70%)
- Integration: 2 (20%)
- E2E: 1 (10%)

**Breakdown:**
- Positive tests: 3
- Negative tests: 4
- Boundary tests: 2
- Hallucination prevention: 1

**Approve test plan?**
```

**Human:** "Yes, approved!"

**AI then generates all 10 tests:**

```markdown
## TC-QUOTE-003-001: Cancel quote with status "Sent" - Happy path {#tc-quote-003-001}

**Type:** E2E
**Traceability:** AC-QUOTE-003-1

**Steps:**
1. **Given** customer has quote Q-123 with status `Sent`
2. **When** customer clicks "Cancel Quote"
3. **And** confirms cancellation with reason "No longer needed"
4. **Then** system returns 200 OK
5. **And** quote status is `Cancelled`
6. **And** cancellationTimestamp is set to current time
7. **And** cancelledBy is customer ID
8. **And** cancellationReason is "No longer needed"
9. **And** customer sees confirmation message
10. **And** sales rep receives notification

**Expected:** ✓ Quote successfully cancelled

---

## TC-QUOTE-003-002: Cannot cancel accepted quote {#tc-quote-003-002}

**Type:** Unit (Negative Test)
**Traceability:** AC-QUOTE-003-2, BR-QUOTE-003

**Steps:**
1. **Given** quote Q-456 with status `Accepted`
2. **When** customer attempts POST /api/quotes/Q-456/cancel
3. **Then** system returns 422 Unprocessable Entity
4. **And** error message: "Cannot cancel quote with status 'Accepted'"
5. **And** quote status remains `Accepted` (unchanged)

**Expected:** ✓ Cancellation rejected (BR-QUOTE-003 enforced)

---

## TC-QUOTE-003-003: Cannot cancel rejected quote {#tc-quote-003-003}

**Type:** Unit (Negative Test)
**Traceability:** BR-QUOTE-003

**Steps:**
1. **Given** quote Q-789 with status `Rejected`
2. **When** customer attempts to cancel
3. **Then** system returns 422 Unprocessable Entity
4. **And** quote status remains `Rejected`

**Expected:** ✓ Cancellation rejected

---

## TC-QUOTE-003-004: Cannot cancel expired quote {#tc-quote-003-004}

**Type:** Unit (Negative Test)
**Traceability:** BR-QUOTE-003

**Steps:**
1. **Given** quote Q-999 with status `Expired`
2. **When** customer attempts to cancel
3. **Then** system returns 422 Unprocessable Entity

**Expected:** ✓ Cancellation rejected

---

## TC-QUOTE-003-005: Boundary - Cancel draft quote {#tc-quote-003-005}

**Type:** Unit (Boundary Test)
**Traceability:** BR-QUOTE-003

**Steps:**
1. **Given** quote Q-111 with status `Draft` (boundary: not yet sent)
2. **When** sales rep cancels quote
3. **Then** system accepts (200 OK)
4. **And** status changes to `Cancelled`

**Expected:** ✓ Draft quote can be cancelled

---

## TC-QUOTE-003-006: Boundary - Sent → Cancelled transition {#tc-quote-003-006}

**Type:** Unit (Boundary Test)
**Traceability:** BR-QUOTE-003

**Steps:**
1. **Given** quote status transitions from `Sent` to `Cancelled`
2. **Then** transition is valid
3. **And** no other status can transition to `Cancelled`

**Expected:** ✓ Only Draft/Sent → Cancelled allowed

---

## TC-QUOTE-003-007: Customer cannot cancel other customer's quote {#tc-quote-003-007}

**Type:** Integration (Negative Test - Authorization)
**Traceability:** AC-QUOTE-003-1 (implicit authorization)

**Steps:**
1. **Given** customer A owns quote Q-123
2. **When** customer B attempts POST /api/quotes/Q-123/cancel
3. **Then** system returns 403 Forbidden
4. **And** quote status unchanged

**Expected:** ✓ Unauthorized cancellation prevented

---

## TC-QUOTE-003-008: Cancellation reason is optional {#tc-quote-003-008}

**Type:** Unit (Positive Test)
**Traceability:** AC-QUOTE-003-3

**Steps:**
1. **Given** customer cancels quote without providing reason
2. **When** POST /api/quotes/Q-123/cancel (empty body)
3. **Then** system accepts (200 OK)
4. **And** cancellationReason is null

**Expected:** ✓ Reason is optional

---

## TC-QUOTE-003-009: Should NOT send email to customer (Hallucination Prevention) {#tc-quote-003-009}

**Type:** Integration (Hallucination Detection)
**Traceability:** AC-QUOTE-003-1
**PURPOSE:** Ensure AI doesn't hallucinate "send confirmation email to customer"

**Steps:**
1. **Given** customer cancels quote
2. **When** cancellation is processed
3. **Then** notification is sent to SALES REP (AC-QUOTE-003-1)
4. **And** NO email is sent to customer (not in requirements!)

**Expected:** ✓ Only sales rep notified (hallucination prevented)

**Note:** If AI added "send email to customer", this test would FAIL!

---

## TC-QUOTE-003-010: Idempotency - Cancel already cancelled quote {#tc-quote-003-010}

**Type:** Unit (Negative Test - Idempotency)
**Traceability:** BR-QUOTE-003

**Steps:**
1. **Given** quote Q-123 with status `Cancelled`
2. **When** customer attempts to cancel again
3. **Then** system returns 422 Unprocessable Entity
4. **And** error: "Cannot cancel quote with status 'Cancelled'"

**Expected:** ✓ Cannot cancel twice
```

**Test Quality Metrics:**
- Total tests: 10
- Negative tests: 5 (50% - exceeds 20% target ✓)
- Positive tests: 3 (30%)
- Boundary tests: 2 (20%)
- Hallucination prevention: 1 (10%)
- Test pyramid: 70:20:10 ✓

**AI confidence:** 91%

**L3 effort:** 3 minutes (review test plan)

---

## 📊 Summary: Complete Derivation Flow

```
Human Input (5 min):
  domain-vocabulary.md (2 min)
  user-stories.md (3 min)
           ↓
        ╔══════════════════════════════════╗
        ║   AI DERIVATION ENGINE           ║
        ╠══════════════════════════════════╣
        ║  L1: Primary Derivation (2 min)  ║
        ║    - acceptance-criteria.md      ║
        ║    - business-rules.md           ║
        ║    - domain-model.md (update)    ║
        ╠══════════════════════════════════╣
        ║  L2: Secondary Derivation (0 min)║
        ║    - interface-contracts.md      ║
        ║    - sequence-design.md          ║
        ╠══════════════════════════════════╣
        ║  L3: Tertiary Derivation (3 min) ║
        ║    - test-case.md (10 tests!)    ║
        ╚══════════════════════════════════╝
           ↓
Complete documentation set (10 min total)

Documents generated: 7
Tests generated: 10
Lines of documentation: ~500
Traceability links: 25+

Manual effort without AI: 3-4 hours
Manual effort with AI: 10 minutes

Time saved: 95%! 🎉
```

---

## 🎯 Validation Results

```bash
$ loom validate

Running validation...

✓ Structural validation passed (7 documents)
✓ Traceability validation passed (25 links)
✓ Consistency validation passed (0 conflicts)
✓ Completeness validation passed
  - US-QUOTE-003 has 3 acceptance criteria ✓
  - All 3 AC have tests ✓
  - All entities referenced are defined ✓
✓ Test quality validation passed
  - Negative test ratio: 50% (target: ≥20%) ✓
  - Test pyramid: 70:20:10 ✓
  - Hallucination prevention tests: 1 ✓

Summary: 0 errors, 0 warnings

All documentation derived successfully! 🚀
```

---

## 💡 Key Insights

### What Worked Well:

1. **Foundational docs were simple** (5 min human effort)
2. **AI derived complex docs automatically** (L1, L2, L3)
3. **Human approval focused on high-impact items** (AC, test plan)
4. **Traceability maintained throughout** (25+ links)
5. **TDAI generated comprehensive tests** (10 tests, 50% negative!)
6. **Hallucination prevention built-in** (TC-QUOTE-003-009)

### Human-in-the-Loop Touch Points:

- ✅ L0: Write foundational docs (5 min)
- ✅ L1: Review acceptance criteria (2 min)
- ⚠️ L2: Optional review (0 min - auto-approved)
- ✅ L3: Approve test plan (3 min)

**Total human time: 10 minutes**
**Total AI time: ~2 minutes (generation)**

**ROI: 95% time saved vs manual documentation!**

---

*This walkthrough demonstrates the power of Documentation Derivation in Loom (AI-PDS). A simple 5-minute human input cascades through 3 derivation levels, generating 500+ lines of documentation and 10 comprehensive tests, all with strict traceability and validation.*

*Next: Implement this derivation strategy in Claude Code plugin!*

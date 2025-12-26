# User Stories – Quote Acceptance (PoC Test)

This file contains test user stories for validating the Loom L0→L1 derivation.

---

## Quote Acceptance

### US-QUOTE-003 – Customer accepts a quote online {#us-quote-003}

**As a** customer
**I want** to accept a quote online
**So that** I can confirm the order quickly without paperwork.

**Context:**
The customer receives a quote via email with a secure link to the customer portal.
They can review the quote details (items, prices, terms) and accept with a single click.
The quote must be in "Sent" status and within its validity period.

**Acceptance criteria (hints):**
- Customer can open the quote from a secure link or portal.
- Customer can click an "Accept" action.
- The system records the acceptance timestamp and identity.
- Quote status changes to `Accepted`.
- An order is created automatically from the accepted quote.

**Related Entities:**
- ENT-Quote
- ENT-Order
- ENT-Customer

---

### US-QUOTE-004 – Customer rejects a quote {#us-quote-004}

**As a** customer
**I want** to reject a quote
**So that** sales knows I am not interested and can stop following up.

**Context:**
Customer may decide not to proceed with a quote. They should be able to
formally reject it, optionally providing a reason.

**Acceptance criteria (hints):**
- Customer can reject the quote with a single action.
- Optional: customer can provide a rejection reason.
- Quote status changes to `Rejected`.
- Rejected quotes cannot later be accepted.

**Related Entities:**
- ENT-Quote
- ENT-Customer

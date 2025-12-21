---
status: draft
derived-from:
  - "loom-project/poc/output/acceptance-criteria.md"
  - "loom-project/poc/output/business-rules.md"
derived-at: "2025-12-21T14:00:00Z"
derived-by: "loom-derive-l2 skill v2.0 (Structured Interview)"
loom-version: "3.0.0"
structured-interview:
  decision-points-resolved: 5
  from-user-answers: 5
  architectural-patterns:
    - event-driven-choreography
    - partial-success
    - attribute-based-auth
---

# Interface Contracts – Quote Service

## Overview

This document defines the public API for the Quote Service, specifically the Quote Acceptance flow.

**Architectural Decisions (from Structured Interview):**

| Pattern | Choice | Rationale |
|---------|--------|-----------|
| Order Creation | Event-driven async | Don't block acceptance on Order service |
| Failure Mode | Partial success | Quote acceptance succeeds, Order retried |
| Notifications | Event-driven async | Non-blocking |
| Order Cancellation | Synchronous | Immediate consistency for reversal |
| Authorization | Attribute-based | Organization membership |

---

## Quote Service – Public Interface

### AcceptQuote

**ID:** API-QUOTE-ACCEPT

**Purpose:** Customer accepts a quote, triggering async order creation and notifications

**Communication Pattern:**
- Type: Sync Request-Response (for Quote update)
- Downstream: `QuoteAccepted` event → OrderService, NotificationService
- Pattern: Event-Driven Choreography

**Decision Points Resolved:**
- API-1: Sync for Quote, Async for Order (User answer: 1b)
- COM-1: Event-driven for downstream services (User answer: 1b)
- COM-3: Partial success - Quote accepted even if Order fails (User answer: 2b)
- SEC-2: Attribute-based / organization membership (User answer: 5c)

**Preconditions:**
- Quote exists with status `Sent`
- Quote is within validity period (validUntil >= now)
- User is authenticated
- User belongs to Customer's organization (attribute-based auth)

**Request:**
```
POST /api/v1/quotes/{quoteId}/accept
Authorization: Bearer {token}
Content-Type: application/json
```

```json
{
  "acceptedBy": "UUID (optional, defaults to authenticated user)"
}
```

**Response (Success - 200 OK):**
```json
{
  "status": "accepted",
  "quoteId": "uuid-quote-123",
  "acceptedAt": "2025-12-21T14:00:00Z",
  "acceptedBy": {
    "userId": "uuid-user-456",
    "organizationId": "uuid-org-789"
  },
  "orderCreation": {
    "status": "pending",
    "message": "Order will be created shortly",
    "estimatedCompletionSeconds": 5
  }
}
```

**Postconditions:**
- Quote.status = `Accepted`
- Quote.acceptedAt = current timestamp
- Quote.acceptedBy = user ID
- Audit log entry created
- Event `QuoteAccepted` published

**Events Emitted:**
```json
{
  "eventType": "QuoteAccepted",
  "timestamp": "2025-12-21T14:00:00Z",
  "payload": {
    "quoteId": "uuid-quote-123",
    "customerId": "uuid-customer-456",
    "acceptedBy": "uuid-user-789",
    "quoteVersion": 1,
    "lineItems": [
      {
        "lineItemId": "uuid-line-1",
        "productId": "uuid-product-1",
        "quantity": 10,
        "unitPrice": {"amount": 100.00, "currency": "EUR"}
      }
    ],
    "totalAmount": {"amount": 1000.00, "currency": "EUR"}
  }
}
```

**Error Responses:**
| Error Code | HTTP | Description | Retry? |
|------------|------|-------------|--------|
| QUOTE_NOT_FOUND | 404 | Quote does not exist | No |
| INVALID_QUOTE_STATUS | 400 | Quote is not in Sent status | No |
| QUOTE_EXPIRED | 400 | Quote validity period has ended | No |
| UNAUTHORIZED | 401 | User not authenticated | No |
| FORBIDDEN | 403 | User not in customer's organization | No |
| INTERNAL_ERROR | 500 | Unexpected server error | Yes |

**Idempotency:**
- Idempotent: Yes
- Calling AcceptQuote on already-Accepted quote returns success with current state
- Idempotency key: quoteId (natural idempotency)

**Rate Limiting:**
- 100 requests per minute per user
- 1000 requests per minute per organization

**Traceability:**
- Acceptance Criteria: AC-QUOTE-003-1, AC-QUOTE-003-2
- Business Rules: BR-QUOTE-001, BR-QUOTE-002, BR-QUOTE-003, BR-QUOTE-005
- Interview: Questions 1, 2, 5

---

### ReverseAcceptance

**ID:** API-QUOTE-REVERSE

**Purpose:** Customer reverses a quote acceptance, cancelling the associated order

**Communication Pattern:**
- Type: Sync Request-Response (immediate consistency)
- Downstream: Direct sync call to OrderService for cancellation
- Pattern: Synchronous for data consistency

**Decision Points Resolved:**
- API-1: Synchronous - immediate Order cancellation (User answer: 4a)
- COM-1: Direct call for Order cancellation (Implied from sync choice)
- SEC-2: Attribute-based / organization membership (User answer: 5c)

**Preconditions:**
- Quote exists with status `Accepted`
- Associated Order exists
- Order.status NOT IN (`Fulfilled`, `Shipped`, `PartiallyShipped`)
- User belongs to Customer's organization

**Request:**
```
POST /api/v1/quotes/{quoteId}/reverse
Authorization: Bearer {token}
Content-Type: application/json
```

```json
{
  "reason": "string (optional)",
  "reversedBy": "UUID (optional, defaults to authenticated user)"
}
```

**Response (Success - 200 OK):**
```json
{
  "status": "reversed",
  "quoteId": "uuid-quote-123",
  "previousStatus": "accepted",
  "newStatus": "sent",
  "reversedAt": "2025-12-21T14:30:00Z",
  "reversedBy": {
    "userId": "uuid-user-456"
  },
  "orderCancellation": {
    "orderId": "uuid-order-789",
    "status": "cancelled",
    "cancelledAt": "2025-12-21T14:30:00Z"
  }
}
```

**Postconditions:**
- Quote.status = `Sent`
- Quote.acceptedAt = null
- Quote.acceptedBy = null
- Order.status = `Cancelled`
- Audit log entries created for both Quote and Order
- Event `QuoteAcceptanceReversed` published
- Event `OrderCancelled` published

**Error Responses:**
| Error Code | HTTP | Description | Retry? |
|------------|------|-------------|--------|
| QUOTE_NOT_FOUND | 404 | Quote does not exist | No |
| INVALID_QUOTE_STATUS | 400 | Quote is not in Accepted status | No |
| ORDER_ALREADY_FULFILLED | 409 | Cannot reverse, order is fulfilled | No |
| ORDER_HAS_SHIPMENTS | 409 | Cannot reverse, order has shipments | No |
| FORBIDDEN | 403 | User not in customer's organization | No |

**Traceability:**
- Acceptance Criteria: AC-QUOTE-003-3
- Business Rules: BR-QUOTE-004
- Interview: Question 4

---

## Common Types

### Money
```json
{
  "amount": "number (decimal, precision 2)",
  "currency": "string (ISO 4217, e.g., EUR, USD)"
}
```

### Timestamp
`ISO-8601 format: YYYY-MM-DDTHH:mm:ssZ (always UTC)`

### Error Response
```json
{
  "errorCode": "string (machine-readable)",
  "message": "string (human-readable)",
  "details": {
    "field": "additional context"
  },
  "traceId": "string (for support)"
}
```

---

## Authentication & Authorization

### Authentication
- Type: Bearer Token (JWT)
- Header: `Authorization: Bearer {token}`

### Authorization Model
**Decision Point SEC-2: Attribute-based (Organization Membership)**

```
User.organizationId == Quote.customer.organizationId
```

Authorization check performed by Quote Service:
1. Extract `organizationId` from JWT claims
2. Load Quote, get `customer.organizationId`
3. Compare: if not equal, return 403 Forbidden

**Rationale (from Interview):**
- Any user in the customer's organization can accept quotes
- Not limited to specific user who received the quote
- Supports delegation within organization

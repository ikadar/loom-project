---
status: draft
derived-from:
  - "tmp/poc/output/acceptance-criteria.md"
  - "tmp/poc/output/business-rules.md"
derived-at: "2025-12-20T18:45:00Z"
derived-by: "loom-derive-l2 skill v1.0"
loom-version: "2.0.0"
---

# Interface Contracts – Quote Acceptance

This document defines **service interface contracts** for the Quote Acceptance workflow.
Contracts are technology-agnostic and describe operations each service exposes.

---

## Quote Service – Public Interface

### 1. GetQuote

**ID:** API-QUOTE-GET

**Purpose:** Retrieve quote details for customer viewing via secure link.

**Preconditions:**
- Quote exists
- Access token is valid (from secure link)
- Quote is associated with the requesting customer

**Request**
```json
{
  "quoteId": "string (UUID)",
  "accessToken": "string (secure link token)"
}
```

**Response (Success)**
```json
{
  "quoteId": "string (UUID)",
  "status": "Draft | Sent | Accepted | Rejected | Expired",
  "customerId": "string (UUID)",
  "validUntil": "ISO-8601 date",
  "createdAt": "ISO-8601 timestamp",
  "lineItems": [
    {
      "lineId": "string",
      "description": "string",
      "quantity": "number",
      "unitPrice": {
        "amount": "number",
        "currency": "string (ISO 4217)"
      }
    }
  ],
  "totalAmount": {
    "amount": "number",
    "currency": "string (ISO 4217)"
  },
  "terms": "string",
  "isExpired": "boolean",
  "canAccept": "boolean"
}
```

**Postconditions:**
- None (read-only operation)
- Access logged for audit

**Error Responses:**

| Error Code | HTTP | Description |
|------------|------|-------------|
| QUOTE_NOT_FOUND | 404 | Quote does not exist |
| INVALID_ACCESS_TOKEN | 403 | Access token invalid or tampered |
| QUOTE_EXPIRED | 410 | Quote validity period has ended |

**Traceability:**
- Acceptance Criteria: AC-QUOTE-003-1
- Entity: ENT-Quote, ENT-Customer, ENT-LineItem

---

### 2. AcceptQuote

**ID:** API-QUOTE-ACCEPT

**Purpose:** Customer accepts a quote, triggering order creation.

**Preconditions:**
- Quote exists
- Quote status is `Sent` (BR-QUOTE-001)
- Quote is within validity period (BR-QUOTE-006)
- Customer is authenticated and authorized

**Request**
```json
{
  "quoteId": "string (UUID)",
  "customerId": "string (UUID)",
  "acceptanceConfirmation": "boolean (must be true)"
}
```

**Response (Success)**
```json
{
  "status": "accepted",
  "quoteId": "string (UUID)",
  "acceptedAt": "ISO-8601 timestamp",
  "acceptanceId": "string (UUID - audit record)",
  "orderId": "string (UUID - created order)",
  "orderStatus": "Pending"
}
```

**Postconditions:**
- Quote status changed to `Accepted`
- Acceptance record created with audit trail (BR-QUOTE-003)
- Quote becomes immutable (BR-QUOTE-002)
- Order created from quote (BR-QUOTE-004)
- Order references quote (BR-QUOTE-005)
- Domain event `QuoteAccepted` emitted
- Domain event `OrderCreated` emitted

**Error Responses:**

| Error Code | HTTP | Description |
|------------|------|-------------|
| QUOTE_NOT_FOUND | 404 | Quote does not exist |
| INVALID_QUOTE_STATUS | 400 | Quote is not in `Sent` status |
| QUOTE_EXPIRED | 400 | Quote validity period has ended |
| UNAUTHORIZED | 401 | Customer not authenticated |
| FORBIDDEN | 403 | Customer not authorized for this quote |
| CONFIRMATION_REQUIRED | 400 | acceptanceConfirmation must be true |
| ORDER_CREATION_FAILED | 500 | Failed to create order (triggers rollback) |
| AUDIT_RECORD_FAILED | 500 | Failed to create audit record (triggers rollback) |
| CONCURRENT_MODIFICATION | 409 | Quote was modified by another request |

**Traceability:**
- Acceptance Criteria: AC-QUOTE-003-2, AC-QUOTE-003-3, AC-QUOTE-003-4, AC-QUOTE-003-5
- Business Rules: BR-QUOTE-001, BR-QUOTE-002, BR-QUOTE-003, BR-QUOTE-004, BR-QUOTE-005, BR-QUOTE-006
- Entity: ENT-Quote, ENT-Order, ENT-QuoteAcceptance

---

## Order Service – Public Interface

### 3. GetOrder

**ID:** API-ORDER-GET

**Purpose:** Retrieve order details created from quote acceptance.

**Preconditions:**
- Order exists
- Customer is authorized to view order

**Request**
```json
{
  "orderId": "string (UUID)"
}
```

**Response (Success)**
```json
{
  "orderId": "string (UUID)",
  "status": "Pending | InFulfillment | Fulfilled | Cancelled",
  "quoteId": "string (UUID - reference to source quote)",
  "customerId": "string (UUID)",
  "createdAt": "ISO-8601 timestamp",
  "lineItems": [
    {
      "lineId": "string",
      "description": "string",
      "quantity": "number",
      "unitPrice": {
        "amount": "number",
        "currency": "string (ISO 4217)"
      }
    }
  ],
  "totalAmount": {
    "amount": "number",
    "currency": "string (ISO 4217)"
  }
}
```

**Postconditions:**
- None (read-only operation)

**Error Responses:**

| Error Code | HTTP | Description |
|------------|------|-------------|
| ORDER_NOT_FOUND | 404 | Order does not exist |
| FORBIDDEN | 403 | Customer not authorized for this order |

**Traceability:**
- Acceptance Criteria: AC-QUOTE-003-5, AC-QUOTE-003-6
- Business Rules: BR-QUOTE-004, BR-QUOTE-005
- Entity: ENT-Order, ENT-Quote

---

## Common Types

### Money
```json
{
  "amount": "number (decimal, 2 places)",
  "currency": "string (ISO 4217, e.g., 'EUR', 'USD')"
}
```

### Timestamp
ISO-8601 format: `YYYY-MM-DDTHH:mm:ssZ`

### Date
ISO-8601 format: `YYYY-MM-DD`

### UUID
RFC 4122 compliant UUID string

### Error Response
```json
{
  "errorCode": "string (UPPER_SNAKE_CASE)",
  "message": "string (human-readable)",
  "timestamp": "ISO-8601 timestamp",
  "traceId": "string (for debugging)",
  "details": {}
}
```

---

## Domain Events

### QuoteAccepted
```json
{
  "eventType": "QuoteAccepted",
  "eventId": "string (UUID)",
  "timestamp": "ISO-8601",
  "payload": {
    "quoteId": "string (UUID)",
    "customerId": "string (UUID)",
    "acceptedAt": "ISO-8601",
    "acceptedBy": "string (customer user ID)",
    "quoteVersion": "number"
  }
}
```

### OrderCreated
```json
{
  "eventType": "OrderCreated",
  "eventId": "string (UUID)",
  "timestamp": "ISO-8601",
  "payload": {
    "orderId": "string (UUID)",
    "quoteId": "string (UUID)",
    "customerId": "string (UUID)",
    "totalAmount": {
      "amount": "number",
      "currency": "string"
    },
    "lineItemCount": "number"
  }
}
```

---

## Validation Checklist

- [x] 3 API operations defined (GetQuote, AcceptQuote, GetOrder)
- [x] All operations have request/response schemas
- [x] All BR error codes mapped to API errors
- [x] All operations have traceability links
- [x] Common types defined for reuse
- [x] Domain events documented

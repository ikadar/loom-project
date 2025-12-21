---
status: draft
derived-from:
  - "loom-project/poc/input/user-stories.md"
  - "loom-project/poc/input/domain-vocabulary.md"
derived-at: "2025-12-21T13:00:00Z"
derived-by: "loom-derive-domain skill v1.0 (Structured Interview)"
loom-version: "3.0.0"
structured-interview:
  concepts-classified: 8
  from-user-answers: 3
  from-input: 5
  high-confidence: 5
---

# Domain Model – Sales & Quoting

## Overview

This domain model represents the Sales & Quoting bounded context, covering the lifecycle of quotes from creation through customer acceptance and order generation.

---

## Entities

### ENT-Quote – Quote

**Classification:** Entity
**Rationale:** Has unique identity (QuoteId), independent lifecycle, root of Quote aggregate

**Decision Points Resolved:**
- EVO-1: Yes - tracked by QuoteId (From input)
- EVO-2: Yes - independent lifecycle (From input)

**Attributes:**
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| id | UUID | Yes | Unique quote identifier |
| quoteNumber | String | Yes | Human-readable quote number |
| customerId | UUID | Yes | Reference to Customer |
| status | QuoteStatus | Yes | Draft, Sent, Accepted, Rejected, Expired |
| validFrom | Date | Yes | Start of validity period |
| validUntil | Date | Yes | End of validity period |
| totalAmount | Money | Yes | Calculated sum of line items |
| createdAt | Timestamp | Yes | Creation timestamp |
| createdBy | UUID | Yes | User who created the quote |
| sentAt | Timestamp | No | When quote was sent to customer |
| acceptedAt | Timestamp | No | When customer accepted |

**Invariants:**
- INV-Q-1: totalAmount MUST equal SUM(lineItems.lineTotal)
- INV-Q-2: validUntil MUST be > validFrom
- INV-Q-3: MUST have at least 1 QuoteLineItem
- INV-Q-4: Status transitions: Draft→Sent→(Accepted|Rejected|Expired)

**Lifecycle:**
- Created: By sales rep, status = Draft
- Modified: Until status = Sent
- Transitions: Draft → Sent (when sent to customer)
- Terminal: Accepted (creates Order), Rejected, Expired

**Relationships:**
- Contains: QuoteLineItem (1..N), Money (totalAmount)
- References: Customer (by customerId)

**Traceability:**
- User Story: user-stories.md#us-quote-003

---

### ENT-QuoteLineItem – Quote Line Item

**Classification:** Entity
**Rationale:** Requires independent identity for cross-aggregate tracking by Shipment

**Decision Points Resolved:**
- EVO-1: Yes - shipping tracks independently (User answer: 1c)
- EVO-3: Yes - mutable while Quote is Draft (User answer: 2b)
- EVO-4: Yes - referenced by Shipment aggregate (User answer: 3c)

**Attributes:**
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| id | UUID | Yes | Unique line item identifier |
| quoteId | UUID | Yes | Parent quote reference |
| productId | UUID | Yes | Reference to Product |
| description | String | Yes | Product description (snapshot) |
| quantity | Integer | Yes | Number of units |
| unitPrice | Money | Yes | Price per unit (snapshot) |
| lineTotal | Money | Yes | Calculated total (quantity × unitPrice) |
| position | Integer | Yes | Order within quote |

**Invariants:**
- INV-QLI-1: quantity MUST be > 0
- INV-QLI-2: lineTotal MUST equal quantity × unitPrice.amount (same currency)
- INV-QLI-3: Cannot modify if Quote.status NOT IN (Draft)
- INV-QLI-4: unitPrice.currency MUST match Quote currency

**Lifecycle:**
- Created: When added to Quote (Quote must be Draft)
- Modified: While Quote.status = Draft
- Deleted: When removed from Quote (Quote must be Draft)
- Frozen: When Quote.status changes from Draft

**Relationships:**
- Belongs to: Quote (via quoteId)
- References: Product (by productId)
- Contains: Money (unitPrice, lineTotal)
- Referenced by: Shipment (external aggregate)

**Traceability:**
- User Story: user-stories.md#us-quote-003
- Interview: Questions 1, 2, 3

---

### ENT-Customer – Customer

**Classification:** Entity
**Rationale:** Has unique identity, independent lifecycle, referenced by multiple aggregates

**Decision Points Resolved:**
- EVO-1: Yes - tracked by CustomerId (From input)
- EVO-2: Yes - independent lifecycle (From input)

**Attributes:**
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| id | UUID | Yes | Unique customer identifier |
| name | String | Yes | Organization name |
| billingAddress | Address | Yes | Address for invoices |
| shippingAddress | Address | No | Address for deliveries |
| primaryContact | ContactInfo | Yes | Main contact person |
| status | CustomerStatus | Yes | Active, Inactive, Suspended |
| createdAt | Timestamp | Yes | When customer was created |

**Invariants:**
- INV-C-1: name MUST NOT be empty
- INV-C-2: MUST have billingAddress
- INV-C-3: MUST have primaryContact

**Lifecycle:**
- Created: By sales/admin
- Modified: Anytime
- Deleted: Soft delete (status = Inactive)

**Relationships:**
- Contains: Address (billing, shipping), ContactInfo
- Referenced by: Quote, Order

**Traceability:**
- User Story: user-stories.md#us-quote-003

---

### ENT-Order – Order

**Classification:** Entity
**Rationale:** Has unique identity, created from Quote, independent lifecycle after creation

**Decision Points Resolved:**
- EVO-1: Yes - tracked by OrderId (From input)
- EVO-2: Yes - independent lifecycle after Quote acceptance (From input)

**Attributes:**
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| id | UUID | Yes | Unique order identifier |
| orderNumber | String | Yes | Human-readable order number |
| quoteId | UUID | Yes | Source quote reference |
| customerId | UUID | Yes | Customer who ordered |
| status | OrderStatus | Yes | Pending, Confirmed, Shipped, Fulfilled, Cancelled |
| totalAmount | Money | Yes | Order total |
| createdAt | Timestamp | Yes | When order was created |

**Invariants:**
- INV-O-1: quoteId MUST reference an Accepted Quote
- INV-O-2: One Order per accepted Quote

**Lifecycle:**
- Created: Automatically when Quote is accepted
- Modified: Through fulfillment process
- Cancelled: Only if not shipped

**Relationships:**
- References: Quote (by quoteId), Customer (by customerId)

**Traceability:**
- User Story: user-stories.md#us-quote-003

---

### ENT-Product – Product

**Classification:** Entity
**Rationale:** Has unique identity, master data with independent lifecycle

**Decision Points Resolved:**
- EVO-1: Yes - tracked by ProductId (From input)
- EVO-2: Yes - independent lifecycle (From input)

**Attributes:**
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| id | UUID | Yes | Unique product identifier |
| sku | String | Yes | Stock keeping unit |
| name | String | Yes | Product name |
| description | String | No | Product description |
| currentPrice | Money | Yes | Current list price |
| status | ProductStatus | Yes | Active, Discontinued |

**Invariants:**
- INV-P-1: sku MUST be unique
- INV-P-2: currentPrice.amount MUST be >= 0

**Lifecycle:**
- Created: By product management
- Modified: Anytime (price changes don't affect existing quotes)
- Discontinued: Soft delete, cannot be added to new quotes

**Traceability:**
- Domain Vocabulary: domain-vocabulary.md

---

## Value Objects

### VO-Money – Money

**Classification:** Value Object
**Rationale:** Defined entirely by amount and currency, no identity needed

**Decision Points Resolved:**
- EVO-5: Yes - same amount + currency = same Money (From input)

**Attributes:**
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| amount | Decimal | Yes | Numeric value (precision: 2) |
| currency | String | Yes | ISO 4217 currency code |

**Equality:**
Two Money are equal if: amount AND currency match

**Immutability:** Fully immutable - operations return new Money instance

**Operations:**
- add(Money): Money (must have same currency)
- multiply(number): Money
- toString(): "100.00 EUR"

**Traceability:**
- Domain Vocabulary: domain-vocabulary.md

---

### VO-Address – Address

**Classification:** Value Object
**Rationale:** Equality by attributes, distinguished by purpose not identity

**Decision Points Resolved:**
- EVO-5: Yes - same data = same address (User answer: 4a)
- Context: Multiple per Customer as billing/shipping (User answer: 5b)

**Attributes:**
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| street | String | Yes | Street address |
| city | String | Yes | City name |
| postalCode | String | Yes | Postal/ZIP code |
| country | String | Yes | ISO 3166-1 country code |
| type | AddressType | No | Billing, Shipping (context-specific) |

**Equality:**
Two Address are equal if: street, city, postalCode, AND country match

**Immutability:** Fully immutable - changes create new Address

**Traceability:**
- Domain Vocabulary: domain-vocabulary.md
- Interview: Questions 4, 5

---

### VO-ContactInfo – Contact Information

**Classification:** Value Object
**Rationale:** No independent lifecycle, equality by attributes

**Decision Points Resolved:**
- EVO-2: No - part of Customer lifecycle (User answer: 6a)
- EVO-5: Yes - same data = same contact (User answer: 7a)

**Attributes:**
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| email | String | Yes | Email address |
| phone | String | No | Phone number |
| name | String | No | Contact person name |

**Equality:**
Two ContactInfo are equal if: email AND phone match

**Immutability:** Fully immutable

**Traceability:**
- Domain Vocabulary: domain-vocabulary.md
- Interview: Questions 6, 7

---

### VO-DateRange – Date Range

**Classification:** Value Object
**Rationale:** Equality by attributes (start and end dates)

**Decision Points Resolved:**
- EVO-5: Yes - same dates = same range (From input)

**Attributes:**
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| startDate | Date | Yes | Start of range (inclusive) |
| endDate | Date | Yes | End of range (inclusive) |

**Equality:**
Two DateRange are equal if: startDate AND endDate match

**Invariants:**
- endDate MUST be >= startDate

**Operations:**
- contains(date): boolean
- overlaps(DateRange): boolean
- durationInDays(): number

**Traceability:**
- Domain Vocabulary: domain-vocabulary.md

---

## Aggregates

### AGG-Quote – Quote Aggregate

**Root:** ENT-Quote

**Contains:**
- ENT-QuoteLineItem (1..N)
- VO-Money (totalAmount)

**Boundary Rationale:**
Quote and its line items must be modified together to maintain consistency (total = sum of lines). However, QuoteLineItem is an Entity (not Value Object) because it's referenced externally by Shipment.

**Invariants:**
- Quote.totalAmount = SUM(QuoteLineItem.lineTotal)
- All line items must have same currency as Quote
- Line items can only be modified when Quote.status = Draft

**Consistency:**
- Strong consistency within aggregate
- Line item changes immediately update Quote total

---

### AGG-Customer – Customer Aggregate

**Root:** ENT-Customer

**Contains:**
- VO-Address (billingAddress, shippingAddress)
- VO-ContactInfo (primaryContact)

**Boundary Rationale:**
Address and ContactInfo are value objects embedded in Customer. They have no independent lifecycle and are always accessed through Customer.

**Invariants:**
- Must have billing address
- Must have primary contact

---

### AGG-Order – Order Aggregate

**Root:** ENT-Order

**Boundary Rationale:**
Order is created from Quote but has independent lifecycle. References Quote by ID, does not contain it.

---

### AGG-Product – Product Aggregate

**Root:** ENT-Product

**Boundary Rationale:**
Product is master data with independent lifecycle. Referenced by QuoteLineItem by ID.

---

## Domain Events

| Event | Triggered By | Contains |
|-------|--------------|----------|
| QuoteCreated | Quote creation | quoteId, customerId |
| QuoteSent | Quote.send() | quoteId, customerId, sentAt |
| QuoteAccepted | Quote.accept() | quoteId, customerId, acceptedAt, orderId |
| QuoteRejected | Quote.reject() | quoteId, customerId, reason |
| QuoteExpired | Scheduler | quoteId |
| OrderCreated | QuoteAccepted handler | orderId, quoteId, customerId |

---

## Structured Interview Summary

| Concept | Classification | Key Decision | Answer | Source |
|---------|---------------|--------------|--------|--------|
| Quote | Entity | EVO-1 (identity) | Has QuoteId | Input |
| QuoteLineItem | **Entity** | EVO-4 (external ref) | Shipment tracks it | **User (1c, 3c)** |
| Customer | Entity | EVO-2 (lifecycle) | Independent | Input |
| Order | Entity | EVO-2 (lifecycle) | Independent | Input |
| Product | Entity | EVO-1 (identity) | Has ProductId | Input |
| Money | Value Object | EVO-5 (equality) | amount+currency | Input |
| Address | **Value Object** | EVO-5 (equality) | same data = same | **User (4a)** |
| ContactInfo | **Value Object** | EVO-2 (no lifecycle) | Part of Customer | **User (6a, 7a)** |

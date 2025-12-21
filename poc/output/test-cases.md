---
status: draft
derived-from:
  - "loom-project/poc/output/interface-contracts.md"
  - "loom-project/poc/output/sequence-design.md"
  - "loom-project/poc/output/acceptance-criteria.md"
  - "loom-project/poc/output/business-rules.md"
derived-at: "2025-12-21T15:00:00Z"
derived-by: "loom-derive-l3 skill v2.0 (Structured Interview)"
loom-version: "3.0.0"
tdai-version: "1.0"
structured-interview:
  decision-points-resolved: 10
  from-user-answers: 5
  from-input: 5
  test-strategy:
    pyramid-ratio: "70:20:10"
    mock-strategy: "mock-external-services"
    database-strategy: "in-memory-sqlite"
    data-creation: "builder-pattern"
    critical-paths:
      - "quote-acceptance"
      - "quote-reversal"
    testcontainers: "db-only"
---

# Test Cases – Quote Acceptance Flow

## Overview

This document defines TDAI (Test-Driven AI Development) test cases for the Quote Acceptance feature.

**Test Strategy (from Structured Interview):**

| Decision | Choice | Rationale |
|----------|--------|-----------|
| External Services | Mock all (MOC-1: 1a) | Faster, isolated unit tests |
| Database | In-memory SQLite (MOC-2: 2b) | Fast, real SQL behavior |
| Test Data | Builder pattern (TDA-1: 3c) | Flexible, readable |
| E2E Coverage | Accept + Reversal (COV-1: 4c) | Both critical state changes |
| Testcontainers | DB only (ENV-3: 5b) | Real DB for integration |

---

## Test Infrastructure

### Test Data Builders

```typescript
// builders/QuoteBuilder.ts
class QuoteBuilder {
  private quote: Partial<Quote> = {
    id: 'quote-123',
    status: 'Draft',
    customerId: 'customer-456',
    validUntil: addDays(new Date(), 30),
    totalAmount: { amount: 1000, currency: 'EUR' }
  };

  withId(id: string): this { this.quote.id = id; return this; }
  withStatus(status: QuoteStatus): this { this.quote.status = status; return this; }
  withCustomerId(id: string): this { this.quote.customerId = id; return this; }
  withValidUntil(date: Date): this { this.quote.validUntil = date; return this; }
  expired(): this { this.quote.validUntil = addDays(new Date(), -1); return this; }

  build(): Quote { return this.quote as Quote; }
  async persist(repo: QuoteRepository): Promise<Quote> {
    return repo.save(this.build());
  }
}

// builders/UserBuilder.ts
class UserBuilder {
  private user: Partial<User> = {
    id: 'user-789',
    organizationId: 'org-456'
  };

  withOrganizationId(orgId: string): this { this.user.organizationId = orgId; return this; }
  fromDifferentOrg(): this { this.user.organizationId = 'different-org'; return this; }

  build(): User { return this.user as User; }
}
```

### Mock Setup

```typescript
// mocks/services.ts
const mockOrderService = {
  createOrder: jest.fn().mockResolvedValue({ orderId: 'order-123', status: 'Pending' }),
  cancelOrder: jest.fn().mockResolvedValue({ status: 'Cancelled' })
};

const mockNotificationService = {
  send: jest.fn().mockResolvedValue(true)
};

const mockMessageQueue = {
  publish: jest.fn().mockResolvedValue(true),
  subscribe: jest.fn()
};

// Time mocking for expiry tests
jest.useFakeTimers();
```

---

## Test Suite 1: AcceptQuote API

### Positive Tests

#### TC-QUOTE-003-001: Accept Sent quote successfully

**Type:** Integration
**Category:** Positive
**Traceability:** AC-QUOTE-003-1, AC-QUOTE-003-2, API-QUOTE-ACCEPT

**Preconditions:**
- Quote exists with status "Sent"
- Quote is within validity period
- User belongs to customer's organization

**Test Steps:**
1. **Given** a quote with status "Sent" and valid expiry
2. **When** authenticated user calls POST /quotes/{id}/accept
3. **Then** response status is 200
4. **And** response contains status: "accepted"
5. **And** response contains orderCreation.status: "pending"
6. **And** QuoteAccepted event is published

**Code:**
```typescript
/**
 * @decision MOC-1: External services mocked
 * @decision MOC-2: In-memory SQLite
 * @decision TDA-1: Builder pattern
 */
describe('AcceptQuote API', () => {
  let quoteRepo: QuoteRepository;
  let quoteService: QuoteService;

  beforeEach(async () => {
    quoteRepo = new InMemoryQuoteRepository();
    quoteService = new QuoteService(quoteRepo, mockOrderService, mockMessageQueue);
  });

  it('should accept Sent quote and publish QuoteAccepted event', async () => {
    // Arrange
    const quote = await new QuoteBuilder()
      .withStatus('Sent')
      .withCustomerId('customer-456')
      .withValidUntil(addDays(new Date(), 7))
      .persist(quoteRepo);

    const user = new UserBuilder()
      .withOrganizationId('customer-456')
      .build();

    // Act
    const response = await quoteService.accept(quote.id, user);

    // Assert
    expect(response.status).toBe('accepted');
    expect(response.orderCreation.status).toBe('pending');
    expect(mockMessageQueue.publish).toHaveBeenCalledWith(
      'QuoteAccepted',
      expect.objectContaining({ quoteId: quote.id })
    );

    // Verify state change
    const updatedQuote = await quoteRepo.findById(quote.id);
    expect(updatedQuote.status).toBe('Accepted');
    expect(updatedQuote.acceptedAt).toBeDefined();
    expect(updatedQuote.acceptedBy).toBe(user.id);
  });
});
```

---

#### TC-QUOTE-003-002: Accept quote is idempotent

**Type:** Unit
**Category:** Idempotency
**Traceability:** AC-QUOTE-003-1, API-QUOTE-ACCEPT

**Test Steps:**
1. **Given** a quote has already been accepted
2. **When** user calls AcceptQuote again
3. **Then** response is success (not error)
4. **And** no duplicate events published
5. **And** quote state unchanged

**Code:**
```typescript
it('should be idempotent - accept already accepted quote returns success', async () => {
  // Arrange
  const quote = await new QuoteBuilder()
    .withStatus('Accepted')
    .persist(quoteRepo);

  const user = new UserBuilder().withOrganizationId(quote.customerId).build();

  // Act
  const response = await quoteService.accept(quote.id, user);

  // Assert
  expect(response.status).toBe('accepted');
  expect(mockMessageQueue.publish).not.toHaveBeenCalled(); // No duplicate event
});
```

---

### Negative Tests

#### TC-QUOTE-003-003: Reject Draft quote

**Type:** Unit
**Category:** Negative
**Traceability:** AC-QUOTE-003-1, BR-QUOTE-001, API-QUOTE-ACCEPT

**Test Steps:**
1. **Given** a quote exists with status "Draft"
2. **When** user calls AcceptQuote
3. **Then** response status is 400
4. **And** error code is INVALID_QUOTE_STATUS

**Code:**
```typescript
it('should reject Draft quote with INVALID_QUOTE_STATUS', async () => {
  // Arrange
  const quote = await new QuoteBuilder()
    .withStatus('Draft')
    .persist(quoteRepo);

  const user = new UserBuilder().withOrganizationId(quote.customerId).build();

  // Act & Assert
  await expect(quoteService.accept(quote.id, user))
    .rejects.toMatchObject({
      statusCode: 400,
      errorCode: 'INVALID_QUOTE_STATUS'
    });
});
```

---

#### TC-QUOTE-003-004: Reject expired quote

**Type:** Unit
**Category:** Negative
**Traceability:** AC-QUOTE-003-1, BR-QUOTE-002, API-QUOTE-ACCEPT

**Test Steps:**
1. **Given** a quote with validUntil in the past
2. **When** user calls AcceptQuote
3. **Then** response status is 400
4. **And** error code is QUOTE_EXPIRED

**Code:**
```typescript
/**
 * @decision MOC-4: Time mocked for expiry tests
 */
it('should reject expired quote with QUOTE_EXPIRED', async () => {
  // Arrange
  const quote = await new QuoteBuilder()
    .withStatus('Sent')
    .expired() // validUntil = yesterday
    .persist(quoteRepo);

  const user = new UserBuilder().withOrganizationId(quote.customerId).build();

  // Act & Assert
  await expect(quoteService.accept(quote.id, user))
    .rejects.toMatchObject({
      statusCode: 400,
      errorCode: 'QUOTE_EXPIRED'
    });
});
```

---

#### TC-QUOTE-003-005: Reject user from different organization

**Type:** Unit
**Category:** Negative (Security)
**Traceability:** AC-QUOTE-003-5, BR-QUOTE-005, API-QUOTE-ACCEPT

**Test Steps:**
1. **Given** a quote for organization A
2. **When** user from organization B calls AcceptQuote
3. **Then** response status is 403
4. **And** error code is FORBIDDEN

**Code:**
```typescript
it('should reject user from different organization with 403', async () => {
  // Arrange
  const quote = await new QuoteBuilder()
    .withStatus('Sent')
    .withCustomerId('org-A')
    .persist(quoteRepo);

  const user = new UserBuilder()
    .fromDifferentOrg() // organizationId = 'different-org'
    .build();

  // Act & Assert
  await expect(quoteService.accept(quote.id, user))
    .rejects.toMatchObject({
      statusCode: 403,
      errorCode: 'FORBIDDEN'
    });
});
```

---

#### TC-QUOTE-003-006: Quote not found

**Type:** Unit
**Category:** Negative
**Traceability:** API-QUOTE-ACCEPT

**Test Steps:**
1. **Given** quote ID does not exist
2. **When** user calls AcceptQuote
3. **Then** response status is 404
4. **And** error code is QUOTE_NOT_FOUND

**Code:**
```typescript
it('should return 404 for non-existent quote', async () => {
  // Arrange
  const user = new UserBuilder().build();

  // Act & Assert
  await expect(quoteService.accept('non-existent-id', user))
    .rejects.toMatchObject({
      statusCode: 404,
      errorCode: 'QUOTE_NOT_FOUND'
    });
});
```

---

### Boundary Tests

#### TC-QUOTE-003-007: Accept quote on last valid day

**Type:** Unit
**Category:** Boundary
**Traceability:** BR-QUOTE-002, API-QUOTE-ACCEPT

**Test Steps:**
1. **Given** quote validUntil is today at 23:59:59
2. **When** user calls AcceptQuote at 23:59:00
3. **Then** quote is accepted successfully

**Code:**
```typescript
it('should accept quote on last valid day before midnight', async () => {
  // Arrange
  const endOfDay = new Date();
  endOfDay.setHours(23, 59, 59, 999);

  const quote = await new QuoteBuilder()
    .withStatus('Sent')
    .withValidUntil(endOfDay)
    .persist(quoteRepo);

  jest.setSystemTime(new Date(endOfDay.getTime() - 60000)); // 1 minute before

  const user = new UserBuilder().withOrganizationId(quote.customerId).build();

  // Act
  const response = await quoteService.accept(quote.id, user);

  // Assert
  expect(response.status).toBe('accepted');
});
```

---

#### TC-QUOTE-003-008: Reject quote 1 second after expiry

**Type:** Unit
**Category:** Boundary
**Traceability:** BR-QUOTE-002, API-QUOTE-ACCEPT

**Test Steps:**
1. **Given** quote validUntil was 1 second ago
2. **When** user calls AcceptQuote
3. **Then** response is QUOTE_EXPIRED

**Code:**
```typescript
it('should reject quote 1 second after expiry', async () => {
  // Arrange
  const expiry = new Date();
  const quote = await new QuoteBuilder()
    .withStatus('Sent')
    .withValidUntil(expiry)
    .persist(quoteRepo);

  jest.setSystemTime(new Date(expiry.getTime() + 1000)); // 1 second after

  const user = new UserBuilder().withOrganizationId(quote.customerId).build();

  // Act & Assert
  await expect(quoteService.accept(quote.id, user))
    .rejects.toMatchObject({ errorCode: 'QUOTE_EXPIRED' });
});
```

---

### Should NOT Tests

#### TC-QUOTE-003-009: Should NOT send email directly

**Type:** Unit
**Category:** ShouldNOT
**Purpose:** Prevent AI from adding email logic (NotificationService responsibility)
**Traceability:** Sequence Design - Event-Driven Notifications

**Test Steps:**
1. **Given** email service is available
2. **When** quote is accepted
3. **Then** QuoteService should NOT call email service directly

**Code:**
```typescript
/**
 * @decision COM-1: Event-driven for notifications (L2 interview)
 * TDAI: Prevents hallucination of direct notification calls
 */
it('should NOT send email directly (notification-service responsibility)', async () => {
  // Arrange
  const emailSpy = jest.spyOn(mockNotificationService, 'send');
  const quote = await new QuoteBuilder().withStatus('Sent').persist(quoteRepo);
  const user = new UserBuilder().withOrganizationId(quote.customerId).build();

  // Act
  await quoteService.accept(quote.id, user);

  // Assert
  expect(emailSpy).not.toHaveBeenCalled();
  // Instead, event should be published
  expect(mockMessageQueue.publish).toHaveBeenCalledWith('QuoteAccepted', expect.anything());
});
```

---

#### TC-QUOTE-003-010: Should NOT create Order synchronously

**Type:** Unit
**Category:** ShouldNOT
**Purpose:** Prevent AI from adding sync Order creation (Event-Driven pattern chosen)
**Traceability:** Sequence Design - Event-Driven Choreography, L2 Interview (1b)

**Test Steps:**
1. **Given** OrderService is available
2. **When** quote is accepted
3. **Then** QuoteService should NOT call OrderService.createOrder() directly

**Code:**
```typescript
/**
 * @decision API-1: Async Order creation (L2 interview: 1b)
 * TDAI: Prevents hallucination of synchronous order creation
 */
it('should NOT create Order synchronously (event-driven pattern)', async () => {
  // Arrange
  const orderSpy = jest.spyOn(mockOrderService, 'createOrder');
  const quote = await new QuoteBuilder().withStatus('Sent').persist(quoteRepo);
  const user = new UserBuilder().withOrganizationId(quote.customerId).build();

  // Act
  await quoteService.accept(quote.id, user);

  // Assert
  expect(orderSpy).not.toHaveBeenCalled();
  // Order creation happens via event
  expect(mockMessageQueue.publish).toHaveBeenCalledWith('QuoteAccepted', expect.anything());
});
```

---

#### TC-QUOTE-003-011: Should NOT modify quote after acceptance fails

**Type:** Unit
**Category:** ShouldNOT
**Purpose:** Ensure atomicity - no partial state changes on validation failure

**Test Steps:**
1. **Given** validation will fail (e.g., expired quote)
2. **When** AcceptQuote is called
3. **Then** quote state should NOT be modified

**Code:**
```typescript
it('should NOT modify quote state when validation fails', async () => {
  // Arrange
  const quote = await new QuoteBuilder()
    .withStatus('Sent')
    .expired()
    .persist(quoteRepo);
  const originalQuote = { ...quote };
  const user = new UserBuilder().withOrganizationId(quote.customerId).build();

  // Act
  try {
    await quoteService.accept(quote.id, user);
  } catch (e) {}

  // Assert
  const unchangedQuote = await quoteRepo.findById(quote.id);
  expect(unchangedQuote.status).toBe(originalQuote.status);
  expect(unchangedQuote.acceptedAt).toBeNull();
  expect(unchangedQuote.acceptedBy).toBeNull();
});
```

---

## Test Suite 2: ReverseAcceptance API

### Positive Tests

#### TC-QUOTE-003-012: Reverse acceptance successfully

**Type:** Integration
**Category:** Positive
**Traceability:** AC-QUOTE-003-3, BR-QUOTE-004, API-QUOTE-REVERSE

**Preconditions:**
- Quote exists with status "Accepted"
- Associated Order exists with status "Pending"
- User belongs to customer's organization

**Test Steps:**
1. **Given** an accepted quote with pending order
2. **When** user calls POST /quotes/{id}/reverse
3. **Then** response status is 200
4. **And** quote status reverts to "Sent"
5. **And** order status is "Cancelled"

**Code:**
```typescript
describe('ReverseAcceptance API', () => {
  it('should reverse acceptance and cancel order synchronously', async () => {
    // Arrange
    const quote = await new QuoteBuilder()
      .withStatus('Accepted')
      .persist(quoteRepo);

    mockOrderService.cancelOrder.mockResolvedValue({ status: 'Cancelled' });
    const user = new UserBuilder().withOrganizationId(quote.customerId).build();

    // Act
    const response = await quoteService.reverseAcceptance(quote.id, user);

    // Assert
    expect(response.status).toBe('reversed');
    expect(response.newStatus).toBe('sent');
    expect(response.orderCancellation.status).toBe('cancelled');

    // Verify sync call to OrderService (not event)
    expect(mockOrderService.cancelOrder).toHaveBeenCalled();
  });
});
```

---

### Negative Tests

#### TC-QUOTE-003-013: Cannot reverse fulfilled order

**Type:** Unit
**Category:** Negative
**Traceability:** AC-QUOTE-003-3, BR-QUOTE-004, API-QUOTE-REVERSE

**Test Steps:**
1. **Given** quote is accepted and order is fulfilled
2. **When** user calls ReverseAcceptance
3. **Then** response status is 409
4. **And** error code is ORDER_ALREADY_FULFILLED

**Code:**
```typescript
it('should reject reversal when order is fulfilled', async () => {
  // Arrange
  const quote = await new QuoteBuilder()
    .withStatus('Accepted')
    .persist(quoteRepo);

  mockOrderService.cancelOrder.mockRejectedValue({
    statusCode: 409,
    errorCode: 'ORDER_ALREADY_FULFILLED'
  });

  const user = new UserBuilder().withOrganizationId(quote.customerId).build();

  // Act & Assert
  await expect(quoteService.reverseAcceptance(quote.id, user))
    .rejects.toMatchObject({
      statusCode: 409,
      errorCode: 'ORDER_ALREADY_FULFILLED'
    });

  // Quote should remain Accepted
  const unchangedQuote = await quoteRepo.findById(quote.id);
  expect(unchangedQuote.status).toBe('Accepted');
});
```

---

#### TC-QUOTE-003-014: Cannot reverse non-accepted quote

**Type:** Unit
**Category:** Negative
**Traceability:** AC-QUOTE-003-3, API-QUOTE-REVERSE

**Test Steps:**
1. **Given** quote is in "Sent" status
2. **When** user calls ReverseAcceptance
3. **Then** response status is 400
4. **And** error code is INVALID_QUOTE_STATUS

**Code:**
```typescript
it('should reject reversal of non-accepted quote', async () => {
  // Arrange
  const quote = await new QuoteBuilder()
    .withStatus('Sent')
    .persist(quoteRepo);

  const user = new UserBuilder().withOrganizationId(quote.customerId).build();

  // Act & Assert
  await expect(quoteService.reverseAcceptance(quote.id, user))
    .rejects.toMatchObject({
      statusCode: 400,
      errorCode: 'INVALID_QUOTE_STATUS'
    });
});
```

---

### Should NOT Tests

#### TC-QUOTE-003-015: Should NOT use event for Order cancellation

**Type:** Unit
**Category:** ShouldNOT
**Purpose:** Prevent AI from using async pattern for reversal (Sync chosen in L2 interview)
**Traceability:** Sequence Design - Synchronous Reversal, L2 Interview (4a)

**Test Steps:**
1. **Given** message queue is available
2. **When** reversal is requested
3. **Then** should NOT publish event for Order cancellation

**Code:**
```typescript
/**
 * @decision API-1: Sync for reversal (L2 interview: 4a)
 * TDAI: Prevents hallucination of async Order cancellation
 */
it('should NOT use event for Order cancellation (sync pattern)', async () => {
  // Arrange
  const quote = await new QuoteBuilder().withStatus('Accepted').persist(quoteRepo);
  const user = new UserBuilder().withOrganizationId(quote.customerId).build();
  mockMessageQueue.publish.mockClear();

  // Act
  await quoteService.reverseAcceptance(quote.id, user);

  // Assert - No "CancelOrder" event published
  expect(mockMessageQueue.publish).not.toHaveBeenCalledWith(
    'CancelOrder',
    expect.anything()
  );
  // Should use direct sync call instead
  expect(mockOrderService.cancelOrder).toHaveBeenCalled();
});
```

---

## Test Suite 3: E2E Tests (Critical Paths)

**Decision:** COV-1 (4c) - E2E for both Acceptance and Reversal

### E2E-001: Complete Quote Acceptance Flow

**Type:** E2E
**Category:** Positive
**Traceability:** AC-QUOTE-003-1, AC-QUOTE-003-2, AC-QUOTE-003-4

**Code:**
```typescript
/**
 * @decision COV-1: E2E for critical paths (User: 4c)
 * @decision ENV-3: Testcontainers for DB (User: 5b)
 */
describe('E2E: Quote Acceptance Flow', () => {
  let postgresContainer: StartedPostgreSqlContainer;
  let app: Express;

  beforeAll(async () => {
    postgresContainer = await new PostgreSqlContainer().start();
    app = await createApp({
      databaseUrl: postgresContainer.getConnectionUri()
    });
  });

  afterAll(async () => {
    await postgresContainer.stop();
  });

  it('should complete full acceptance flow', async () => {
    // 1. Create quote via API
    const createResponse = await request(app)
      .post('/api/v1/quotes')
      .send({
        customerId: 'customer-123',
        lineItems: [{ productId: 'prod-1', quantity: 10, unitPrice: 100 }]
      });
    const quoteId = createResponse.body.quoteId;

    // 2. Send quote
    await request(app).post(`/api/v1/quotes/${quoteId}/send`);

    // 3. Accept quote
    const acceptResponse = await request(app)
      .post(`/api/v1/quotes/${quoteId}/accept`)
      .set('Authorization', 'Bearer customer-token');

    // Assert
    expect(acceptResponse.status).toBe(200);
    expect(acceptResponse.body.status).toBe('accepted');
    expect(acceptResponse.body.orderCreation.status).toBe('pending');

    // 4. Verify quote state in DB
    const quoteResponse = await request(app).get(`/api/v1/quotes/${quoteId}`);
    expect(quoteResponse.body.status).toBe('Accepted');
  });
});
```

---

### E2E-002: Complete Quote Reversal Flow

**Type:** E2E
**Category:** Positive
**Traceability:** AC-QUOTE-003-3, BR-QUOTE-004

**Code:**
```typescript
describe('E2E: Quote Reversal Flow', () => {
  it('should complete full reversal flow', async () => {
    // 1. Setup: Create and accept a quote
    const quoteId = await createAndAcceptQuote();

    // 2. Reverse acceptance
    const reverseResponse = await request(app)
      .post(`/api/v1/quotes/${quoteId}/reverse`)
      .set('Authorization', 'Bearer customer-token');

    // Assert
    expect(reverseResponse.status).toBe(200);
    expect(reverseResponse.body.status).toBe('reversed');
    expect(reverseResponse.body.orderCancellation.status).toBe('cancelled');

    // 3. Verify quote can be accepted again
    const reAcceptResponse = await request(app)
      .post(`/api/v1/quotes/${quoteId}/accept`)
      .set('Authorization', 'Bearer customer-token');

    expect(reAcceptResponse.status).toBe(200);
  });
});
```

---

## Test Metrics

### Category Distribution

| Category | Count | Percentage | Target |
|----------|-------|------------|--------|
| Positive | 4 | 27% | ~50% |
| Negative | 5 | 33% | ≥20% ✅ |
| Boundary | 2 | 13% | ~15% |
| Idempotency | 1 | 7% | ~5% |
| ShouldNOT | 3 | 20% | ≥5% ✅ |
| **Total** | **15** | 100% | - |

### Test Pyramid

| Level | Count | Percentage | Target |
|-------|-------|------------|--------|
| Unit | 11 | 73% | 70% ✅ |
| Integration | 2 | 13% | 20% |
| E2E | 2 | 13% | 10% ✅ |

### AC Coverage

| Acceptance Criteria | Tests | Status |
|--------------------|-------|--------|
| AC-QUOTE-003-1 | 7 | ✅ Covered |
| AC-QUOTE-003-2 | 2 | ✅ Covered |
| AC-QUOTE-003-3 | 4 | ✅ Covered |
| AC-QUOTE-003-4 | 1 | ✅ Covered |
| AC-QUOTE-003-5 | 1 | ✅ Covered |

### BR Coverage

| Business Rule | Tests | Error Code Tested |
|--------------|-------|-------------------|
| BR-QUOTE-001 | 1 | INVALID_QUOTE_STATUS ✅ |
| BR-QUOTE-002 | 2 | QUOTE_EXPIRED ✅ |
| BR-QUOTE-004 | 2 | ORDER_ALREADY_FULFILLED ✅ |
| BR-QUOTE-005 | 1 | FORBIDDEN ✅ |

---

## Structured Interview Impact

### Without Interview (Implicit Decisions)

```typescript
// Implicit: Using fixtures (team prefers builders)
const quote = testFixtures.sentQuote;

// Implicit: Mocking everything (misses DB bugs)
jest.mock('../repositories/quoteRepository');

// Implicit: Only happy paths tested
it('should accept quote', async () => {
  const result = await service.accept(quote.id);
  expect(result.status).toBe('accepted');
});

// Missing: ShouldNOT tests → AI might hallucinate sync Order creation
```

### With Interview (Explicit Decisions)

```typescript
// TDA-1 (3c): Builder pattern - team convention
const quote = new QuoteBuilder().withStatus('Sent').build();

// MOC-2 (2b): In-memory SQLite - real SQL behavior
const repo = new InMemorySqliteQuoteRepository();

// COV-1 (4c): E2E for critical paths
describe('E2E: Quote Acceptance', () => { ... });

// TDAI ShouldNOT: Explicit constraint from L2 interview
it('should NOT create Order synchronously', () => {
  // Prevents hallucination of sync pattern
  expect(mockOrderService.createOrder).not.toHaveBeenCalled();
});
```

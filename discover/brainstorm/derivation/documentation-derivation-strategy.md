---
date: 2025-12-19
author: Claude Sonnet 4.5 + Human collaboration
version: 1.0
status: draft
purpose: Documentation Derivation Strategy for AI-PDS/Loom
related:
  - ../core-concepts/bidirectional-traceability-design.md
  - ../core-concepts/test-driven-ai-development.md
  - ../core-concepts/structured-interview-pattern.md
  - ../platform/platform-architecture.md
based-on: ai-pds-specification/9000-appendix/9200-example-ai-pds structure
---

# Documentation Derivation Strategy

## 🎯 Purpose

This document defines **how AI automatically derives documentation** from foundational human inputs in the AI-PDS (Loom) system.

**Core principle:**
> Humans provide high-level intent and domain knowledge.
> AI derives detailed, consistent documentation at multiple levels.
> Everything is traceable and validated.

---

## 📊 Derivation Hierarchy

Documentation is organized in **4 derivation levels**, from human-provided foundations to AI-generated detailed artifacts.

```
┌─────────────────────────────────────────────────────────────────┐
│ LEVEL 0: FOUNDATIONAL (Human-Provided)                         │
│   - Project Handbook (team, workflow, standards)               │
│   - Domain Vocabulary (core concepts, definitions)             │
│   - User Stories (high-level requirements)                     │
│   - NFRs (non-functional requirements)                         │
│                                                                 │
│ Human effort: 80%  │  AI effort: 20% (formatting, validation)  │
└─────────────────┬───────────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────┐
│ LEVEL 1: PRIMARY DERIVATION (AI-Generated from L0)             │
│   - Domain Model (entities, aggregates)                        │
│   - Business Rules (constraints, invariants)                   │
│   - Acceptance Criteria (testable requirements)                │
│   - Bounded Context Map (domain boundaries)                    │
│                                                                 │
│ Human effort: 20%  │  AI effort: 80% (generate, human approves)│
└─────────────────┬───────────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────┐
│ LEVEL 2: SECONDARY DERIVATION (AI-Generated from L0+L1)        │
│   - Interface Contracts (APIs, events)                         │
│   - Sequence Designs (flows, interactions)                     │
│   - Initial Data Model (DB schema)                             │
│   - Aggregate Design (DDD aggregates)                          │
│   - Tech Specs (technical specifications from BR)              │
│                                                                 │
│ Human effort: 10%  │  AI effort: 90% (mostly autonomous)       │
└─────────────────┬───────────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────────┐
│ LEVEL 3: TERTIARY DERIVATION (AI-Generated from L0+L1+L2)      │
│   - Test Cases (from acceptance criteria, TDAI)                │
│   - OpenAPI Spec (from interface contracts)                    │
│   - Implementation Skeletons (code templates)                  │
│   - Feature Definition Tickets (implementation specs)          │
│   - Service Boundaries (microservices)                         │
│   - Event/Message Design (event schemas)                       │
│   - Dependency Graphs (visualizations)                         │
│                                                                 │
│ Human effort: 5%   │  AI effort: 95% (fully autonomous)        │
└─────────────────────────────────────────────────────────────────┘
```

**Key insight:** Each level depends on previous levels. Changes ripple down automatically.

---

## 🔄 Derivation Rules

### Level 0 → Level 1 Derivations

#### 1. Domain Vocabulary → Domain Model

**Input:**
- `domain-vocabulary.md` (term definitions, relationships)

**AI Process:**
1. Parse vocabulary for entities vs value objects
2. Identify aggregates (entities with identity + lifecycle)
3. Extract relationships (1-to-many, many-to-many)
4. Infer key attributes from definitions
5. Generate unique IDs (ENT-XXX)
6. Add traceability links

**Output:**
- `domain-model.md` (entities, aggregates, value objects)

**Example:**

```markdown
<!-- INPUT: domain-vocabulary.md -->
## Customer
- **Definition:** A person or organization that requests quotes and places orders.
- **Notes:** Identified by unique customer ID; may have multiple addresses.
- **Related terms:** RFQ, Quote, Order, Invoice.

↓ AI DERIVES ↓

<!-- OUTPUT: domain-model.md -->
### Customer (Aggregate Root) {#ent-customer}

**Identifier:** `CustomerId`

**Responsibilities:**
- Represents a person or organization
- Can request quotes and place orders
- Owns billing and shipping addresses

**Key attributes:**
- `CustomerId` (value object) {#ent-customer:id}
- `Name` (string) {#ent-customer:name}
- `BillingAddress` (value object) {#ent-customer:billing-address}
- `ShippingAddress` (value object, optional) {#ent-customer:shipping-address}

**Traceability:**
- Vocabulary: domain-vocabulary.md#customer
- User Stories: US-CUST-001, US-CUST-002
```

**Validation Rules:**
- ✓ Every vocabulary term → domain model entity or value object
- ✓ Relationships are bidirectional
- ✓ All IDs are unique (ENT-XXX)
- ✓ Traceability links valid

---

#### 2. User Stories → Acceptance Criteria

**Input:**
- `user-stories.md` (high-level requirements in "As a... I want... So that..." format)

**AI Process:**
1. Extract each user story
2. Generate unique IDs (US-XXX)
3. Analyze intent and break into testable criteria
4. Generate acceptance criteria IDs (AC-XXX-1, AC-XXX-2, ...)
5. Ensure criteria are SMART (Specific, Measurable, Achievable, Relevant, Testable)
6. Add traceability

**Output:**
- `acceptance-criteria.md` (detailed, testable requirements)

**Example:**

```markdown
<!-- INPUT: user-stories.md -->
### US-RFQ-001 – Submit RFQ from the portal {#us-rfq-001}
**As a** prospective customer
**I want** to submit a request for quote (RFQ) through the web portal
**So that** I can get prices without calling or emailing sales.

↓ AI DERIVES ↓

<!-- OUTPUT: acceptance-criteria.md -->
## US-RFQ-001: Submit RFQ from portal {#us-rfq-001}

### AC-RFQ-001-1: User can enter contact data {#ac-rfq-001-1}
**Given** a user is on the RFQ submission form
**When** they fill in company name, contact person, email, phone
**Then** the form validates email format and phone number format
**And** all required fields must be non-empty

**Traceability:**
- User Story: user-stories.md#us-rfq-001
- Entity: ENT-Customer
- Test: test-case.md#tc-rfq-001-1

---

### AC-RFQ-001-2: User can add requested items {#ac-rfq-001-2}
**Given** a user has entered contact data
**When** they add one or more requested items with description and quantity
**Then** each item must have a non-empty description
**And** quantity must be a positive integer
**And** at least one item must be added

**Traceability:**
- User Story: user-stories.md#us-rfq-001
- Entity: ENT-RFQ
- Test: test-case.md#tc-rfq-001-2

---

### AC-RFQ-001-3: System confirms RFQ received {#ac-rfq-001-3}
**Given** a user has completed the RFQ form
**When** they click "Submit"
**Then** the system stores the RFQ
**And** returns a confirmation page with RFQ reference number
**And** sends confirmation email to the user's email address

**Traceability:**
- User Story: user-stories.md#us-rfq-001
- Test: test-case.md#tc-rfq-001-3
```

**Validation Rules:**
- ✓ Each US has 3-7 acceptance criteria
- ✓ All AC use Given/When/Then format (BDD style)
- ✓ AC IDs follow pattern: AC-{US-ID}-{seq}
- ✓ Traceability links valid

---

#### 3. Domain Vocabulary + User Stories → Business Rules

**Input:**
- `domain-vocabulary.md` (concepts, constraints)
- `user-stories.md` (requirements)

**AI Process:**
1. Extract implicit constraints from vocabulary definitions
2. Extract explicit rules from user stories
3. Identify invariants (always true conditions)
4. Generate unique IDs (BR-XXX)
5. Categorize rules (validation, authorization, calculation, workflow)
6. Add traceability

**Output:**
- `business-rules.md` (explicit constraints and invariants)

**Example:**

```markdown
<!-- INPUT: domain-vocabulary.md -->
## Quote Status
- **Allowed values:** Draft, Sent, Accepted, Rejected, Expired
- **Notes:** Only an accepted Quote can be converted into an Order.

<!-- INPUT: user-stories.md -->
### US-QUOTE-002 – Customer accepts a quote
**So that** I can proceed with the order.

↓ AI DERIVES ↓

<!-- OUTPUT: business-rules.md -->
### BR-QUOTE-001: Quote status transition rules {#br-quote-001}

**Rule:**
A Quote can only transition to certain statuses based on current status:
- `Draft` → `Sent`
- `Sent` → `Accepted` | `Rejected` | `Expired`
- `Accepted` → (terminal, no further transitions)
- `Rejected` → (terminal)
- `Expired` → (terminal)

**Invariant:**
Once a Quote reaches `Accepted`, `Rejected`, or `Expired`, its status cannot change.

**Traceability:**
- Vocabulary: domain-vocabulary.md#quote-status
- Entity: ENT-Quote
- Implementation: src/domain/Quote.ts:updateStatus()

---

### BR-QUOTE-002: Only accepted quotes create orders {#br-quote-002}

**Rule:**
An Order can only be created from a Quote if:
1. Quote status is `Accepted`
2. Quote has not expired (validityPeriod.end > currentDate)

**Violation:**
If attempted with non-accepted or expired Quote → throw `InvalidQuoteStatusError`

**Traceability:**
- Vocabulary: domain-vocabulary.md#quote, domain-vocabulary.md#order
- User Story: US-QUOTE-002
- Entity: ENT-Quote, ENT-Order
- Implementation: src/domain/Order.ts:createFromQuote()
- Test: test-case.md#tc-quote-002
```

**Validation Rules:**
- ✓ All rules have unique IDs (BR-XXX)
- ✓ Rules reference entities or value objects
- ✓ Invariants explicitly stated
- ✓ Violation behavior defined
- ✓ Traceability links valid

---

### Level 1 → Level 2 Derivations

#### 4. Domain Model + Acceptance Criteria → Interface Contracts

**Input:**
- `domain-model.md` (entities, aggregates)
- `acceptance-criteria.md` (requirements)
- `business-rules.md` (constraints)

**AI Process:**
1. Identify operations from acceptance criteria (create, read, update, delete)
2. Map entities to API resources
3. Generate endpoint paths (RESTful conventions)
4. Define request/response schemas from entity attributes
5. Add validation rules from business rules
6. Generate unique IDs (API-XXX)
7. Add traceability

**Output:**
- `interface-contracts.md` (API specifications, OpenAPI-style)

**Example:**

```markdown
<!-- INPUT: domain-model.md -->
### RFQ (Aggregate Root) {#ent-rfq}
- `RfqId` (value object) {#ent-rfq:id}
- `CustomerId` (value object) {#ent-rfq:customer-id}
- `RequestedItems` (collection) {#ent-rfq:requested-items}

<!-- INPUT: acceptance-criteria.md -->
### AC-RFQ-001-1: User can enter contact data
### AC-RFQ-001-2: User can add requested items
### AC-RFQ-001-3: System confirms RFQ received

↓ AI DERIVES ↓

<!-- OUTPUT: interface-contracts.md -->
## POST /api/rfqs - Submit RFQ {#api-post-rfqs}

**Operation:** Create a new Request for Quote

**Request:**
```json
{
  "customer": {
    "name": "string (required, max 200 chars)",
    "email": "string (required, valid email format)",
    "phone": "string (optional, valid phone format)"
  },
  "requestedItems": [
    {
      "description": "string (required, max 500 chars)",
      "quantity": "integer (required, min 1)"
    }
  ],
  "requestedDeliveryDate": "date (optional, ISO 8601)",
  "specialInstructions": "string (optional, max 1000 chars)"
}
```

**Response (201 Created):**
```json
{
  "rfqId": "string (UUID)",
  "referenceNumber": "string (e.g., RFQ-2024-001)",
  "status": "Submitted",
  "createdAt": "datetime (ISO 8601)"
}
```

**Validation Rules:**
- Implements: BR-CUST-001 (email format validation)
- Implements: AC-RFQ-001-1 (contact data required)
- Implements: AC-RFQ-001-2 (at least 1 requested item)

**Error Responses:**
- `400 Bad Request` - Invalid input (validation errors)
- `422 Unprocessable Entity` - Business rule violation

**Traceability:**
- User Story: US-RFQ-001
- Acceptance Criteria: AC-RFQ-001-1, AC-RFQ-001-2, AC-RFQ-001-3
- Entity: ENT-RFQ, ENT-Customer
- Business Rules: BR-CUST-001
- Test: test-case.md#tc-api-rfq-001
```

**Validation Rules:**
- ✓ All CRUD operations mapped to HTTP verbs
- ✓ Request/response schemas match entity attributes
- ✓ Business rules applied as validations
- ✓ Error responses defined
- ✓ Traceability to AC, entities, business rules

---

#### 5. Domain Model + User Stories + Business Rules → Sequence Design

**Input:**
- `domain-model.md` (entities, relationships)
- `user-stories.md` (user flows)
- `business-rules.md` (constraints)
- `interface-contracts.md` (API calls)

**AI Process:**
1. Extract workflow from user story
2. Identify actors (user, system, external services)
3. Map steps to API calls or domain operations
4. Apply business rules as decision points
5. Generate sequence diagram (PlantUML or Mermaid syntax)
6. Add traceability

**Output:**
- `sequence-design.md` (interaction flows, sequence diagrams)

**Example:**

```markdown
<!-- OUTPUT: sequence-design.md -->
## SEQ-RFQ-001: Submit RFQ flow {#seq-rfq-001}

**User Story:** US-RFQ-001 - Submit RFQ from portal

**Actors:**
- Customer (user)
- Web Portal (frontend)
- RFQ Service (backend)
- Notification Service

**Sequence:**

\`\`\`mermaid
sequenceDiagram
    participant C as Customer
    participant P as Portal
    participant R as RFQ Service
    participant N as Notification Service

    C->>P: Fill RFQ form
    Note over P: Validates email format (BR-CUST-001)
    C->>P: Click "Submit"
    P->>R: POST /api/rfqs
    Note over R: Validates request (AC-RFQ-001-1, AC-RFQ-001-2)
    R->>R: Create RFQ entity
    R->>R: Generate reference number
    R-->>N: Send RFQSubmitted event
    N->>N: Send confirmation email
    R-->>P: 201 Created {rfqId, referenceNumber}
    P->>C: Show confirmation page
    Note over C: Displays RFQ reference number (AC-RFQ-001-3)
\`\`\`

**Traceability:**
- User Story: US-RFQ-001
- Acceptance Criteria: AC-RFQ-001-1, AC-RFQ-001-2, AC-RFQ-001-3
- Business Rules: BR-CUST-001
- API: API-POST-RFQS
- Entity: ENT-RFQ
- Test: test-case.md#tc-rfq-001
```

**Validation Rules:**
- ✓ All actors identified
- ✓ All steps traceable to AC or BR
- ✓ API calls match interface-contracts.md
- ✓ Decision points reference business rules
- ✓ Diagram syntax valid (Mermaid or PlantUML)

---

### Level 2 → Level 3 Derivations

#### 6. Acceptance Criteria + Interface Contracts → Test Cases (TDAI)

**Input:**
- `acceptance-criteria.md` (testable requirements)
- `interface-contracts.md` (API specs)
- `business-rules.md` (constraints)

**AI Process:**
1. For each acceptance criterion, generate:
   - 3+ positive tests (happy path variations)
   - 3+ negative tests (invalid inputs, edge cases)
   - 2+ boundary tests (min, max, off-by-one)
   - 1+ "should NOT" test (hallucination prevention)
2. Generate unit, integration, and E2E tests
3. Follow test pyramid (70% unit, 20% integration, 10% E2E)
4. Add traceability to AC, API, entities
5. Generate test IDs (TC-XXX)

**Output:**
- `test-case.md` (executable test specifications)

**Example:**

```markdown
<!-- OUTPUT: test-case.md -->
## TC-RFQ-001: Submit RFQ - Happy Path {#tc-rfq-001}

**Traceability:**
- User Story: US-RFQ-001
- Acceptance Criteria: AC-RFQ-001-1, AC-RFQ-001-2, AC-RFQ-001-3
- API: API-POST-RFQS
- Type: E2E (end-to-end)

**Test Steps:**
1. **Given** user is on RFQ submission form
2. **When** user fills valid contact data:
   - Name: "Acme Corp"
   - Email: "contact@acme.com"
   - Phone: "+1-555-0100"
3. **And** adds requested items:
   - Item 1: "Widget A", quantity: 100
   - Item 2: "Widget B", quantity: 50
4. **And** clicks "Submit"
5. **Then** system returns 201 Created
6. **And** response contains `rfqId` and `referenceNumber`
7. **And** user sees confirmation page with reference number
8. **And** user receives confirmation email

**Expected Result:** ✓ RFQ successfully created

---

## TC-RFQ-002: Submit RFQ - Invalid Email (Negative Test) {#tc-rfq-002}

**Traceability:**
- Acceptance Criteria: AC-RFQ-001-1
- Business Rule: BR-CUST-001
- API: API-POST-RFQS
- Type: Unit (validation)

**Test Steps:**
1. **Given** user fills contact data with invalid email: "invalid-email"
2. **When** user clicks "Submit"
3. **Then** system returns 400 Bad Request
4. **And** error message: "Invalid email format"

**Expected Result:** ✓ Validation error thrown

---

## TC-RFQ-003: Submit RFQ - No Items (Negative Test) {#tc-rfq-003}

**Traceability:**
- Acceptance Criteria: AC-RFQ-001-2
- API: API-POST-RFQS
- Type: Unit (validation)

**Test Steps:**
1. **Given** user fills valid contact data
2. **When** user submits without adding any requested items
3. **Then** system returns 400 Bad Request
4. **And** error message: "At least one requested item is required"

**Expected Result:** ✓ Validation error thrown

---

## TC-RFQ-004: Submit RFQ - Boundary Test (Quantity = 1) {#tc-rfq-004}

**Traceability:**
- Acceptance Criteria: AC-RFQ-001-2
- Type: Unit (boundary)

**Test Steps:**
1. **Given** user adds item with quantity = 1 (minimum valid)
2. **When** user submits
3. **Then** system accepts (201 Created)

**Expected Result:** ✓ Minimum quantity accepted

---

## TC-RFQ-005: Submit RFQ - Boundary Test (Quantity = 0) {#tc-rfq-005}

**Traceability:**
- Acceptance Criteria: AC-RFQ-001-2
- Type: Unit (boundary)

**Test Steps:**
1. **Given** user adds item with quantity = 0 (below minimum)
2. **When** user submits
3. **Then** system returns 400 Bad Request
4. **And** error message: "Quantity must be at least 1"

**Expected Result:** ✓ Invalid quantity rejected

---

## TC-RFQ-006: Submit RFQ - Should NOT require phone (Hallucination Prevention) {#tc-rfq-006}

**Traceability:**
- Acceptance Criteria: AC-RFQ-001-1
- Type: Negative (hallucination detection)
- **PURPOSE:** Ensure AI doesn't add phone number requirement (not in AC!)

**Test Steps:**
1. **Given** user fills name and email (no phone number)
2. **When** user submits
3. **Then** system accepts (201 Created)

**Expected Result:** ✓ Phone is optional (hallucination prevented!)

**Note:** If AI hallucinated "phone required", this test would FAIL.
```

**Validation Rules:**
- ✓ Each AC has 5-10 test cases
- ✓ At least 20% are negative tests
- ✓ At least 1 "should NOT" test per AC (hallucination prevention)
- ✓ Test pyramid ratio: ~70:20:10 (unit:integration:e2e)
- ✓ All tests have traceability to AC
- ✓ Test IDs unique (TC-XXX)

---

## 🤖 AI Agents Responsibility Matrix

| Derivation | AI Agent | Input | Output | Approval Required |
|------------|----------|-------|--------|-------------------|
| **L0 → L1** | | | | |
| Vocabulary → Model | `DomainModelAgent` | domain-vocabulary.md | domain-model.md | Yes (human) |
| Stories → AC | `AcceptanceCriteriaAgent` | user-stories.md | acceptance-criteria.md | Yes (human) |
| Vocab+Stories → Rules | `BusinessRulesAgent` | vocabulary + stories | business-rules.md | Yes (human) |
| **L1 → L2** | | | | |
| Model+AC → Contracts | `InterfaceContractAgent` | model + AC + rules | interface-contracts.md | Yes (human) |
| Model+Stories → Sequences | `SequenceDesignAgent` | model + stories + rules | sequence-design.md | Optional |
| Model → Data Model | `DataModelAgent` | domain-model.md | initial-data-model.md | Optional |
| **L2 → L3** | | | | |
| AC+Contracts → Tests | `TestGeneratorAgent` (TDAI) | AC + contracts + rules | test-case.md | Yes (test plan) |
| Stories+AC → Tickets | `FeatureTicketAgent` | stories + AC | feature-definition-ticket.md | Optional |
| Model+Contracts → Services | `ServiceBoundaryAgent` | model + contracts | service-boundaries.md | Optional |

**Approval Strategy:**
- **Level 1 derivations:** Always require human approval (foundational impact)
- **Level 2 derivations:** Human approval for interface-contracts, optional for others
- **Level 3 derivations:** Human approval for test plans, auto-approve test generation

---

## 🔄 Derivation Workflow (Claude Code Integration)

### Example: User adds new feature

**Step 1: Human Input (Natural Language)**
```
User: /loom-generate Add ability for sales rep to send quote reminder email
                      to customer if quote is still "Sent" after 7 days
```

**Step 2: AI Analyzes Intent**
```
Intent Parser Agent:
  - Type: New feature
  - Affected domain: Quote
  - New user story: YES
  - New business rule: YES (7-day reminder)
  - New API endpoint: YES (send reminder)
  - Affected entities: Quote, Customer
```

**Step 3: AI Generates Derivation Plan**
```
Derivation Plan:
  Level 0:
    - user-stories.md: Add US-QUOTE-005 "Send quote reminder"

  Level 1 (auto-derived):
    - acceptance-criteria.md: Generate AC-QUOTE-005-1, AC-QUOTE-005-2, AC-QUOTE-005-3
    - business-rules.md: Add BR-QUOTE-003 "7-day reminder rule"

  Level 2 (auto-derived):
    - interface-contracts.md: Add API-POST-/quotes/{id}/send-reminder
    - sequence-design.md: Add SEQ-QUOTE-005 reminder flow

  Level 3 (auto-derived):
    - test-case.md: Generate TC-QUOTE-005-001 to TC-QUOTE-005-010 (10 tests)
    - feature-definition-ticket.md: Add FEAT-QUOTE-005

Human approval required for: Level 0, Level 1
```

**Step 4: AI Generates Docs**
```
Claude Code:
  1. Uses Edit tool to update user-stories.md
  2. Shows diff preview
  3. Asks: "Approve user story US-QUOTE-005?"

User: yes

  4. AI auto-derives AC-QUOTE-005-1, AC-QUOTE-005-2, AC-QUOTE-005-3
  5. Shows diff preview for acceptance-criteria.md
  6. Asks: "Approve acceptance criteria?"

User: yes

  7. AI auto-derives business-rules.md, interface-contracts.md, sequence-design.md
  8. Shows diff preview (optional review)

User: looks good

  9. AI generates test plan:
     - 3 positive tests
     - 4 negative tests (e.g., "should NOT send reminder if quote accepted")
     - 2 boundary tests (exactly 7 days, 6 days)
     - 1 hallucination prevention test
 10. Shows test plan
 11. Asks: "Approve test plan?"

User: yes

 12. AI generates all test cases (10 tests total)
 13. Shows summary
 14. Done! All docs generated with traceability.
```

**Step 5: Validation**
```
Loom: /loom-validate

Output:
  ✓ Traceability check passed (all IDs exist)
  ✓ Consistency check passed (no conflicts)
  ✓ Coverage check passed (all AC have tests)
  ✓ Test pyramid ratio: 70:20:10 ✓

Summary: 0 errors, 0 warnings
```

---

## 📏 Validation & Quality Gates

### Derivation Validation Rules

**After every derivation, validate:**

1. **Structural Validation:**
   - ✓ All required sections present
   - ✓ YAML frontmatter valid
   - ✓ IDs follow naming conventions
   - ✓ Anchors exist for all headings

2. **Traceability Validation:**
   - ✓ All references point to existing IDs
   - ✓ No broken links
   - ✓ Bidirectional links consistent

3. **Content Validation:**
   - ✓ No duplicate IDs
   - ✓ No contradictions with source docs
   - ✓ All entities referenced are defined
   - ✓ All business rules applied

4. **Completeness Validation:**
   - ✓ All acceptance criteria have tests
   - ✓ All user stories have acceptance criteria
   - ✓ All entities have interface contracts (if applicable)

5. **Test Quality Validation (TDAI-specific):**
   - ✓ Negative test ratio ≥ 20%
   - ✓ Test pyramid ratio: 70±10 : 20±10 : 10±5
   - ✓ At least 1 "should NOT" test per AC
   - ✓ All edge cases covered

---

## 📊 Derivation Dependency Graph

```
┌──────────────────────────────────────────────────────────────┐
│                      DEPENDENCY GRAPH                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  project-handbook/*                                          │
│  (team, workflow, standards)                                 │
│                                                              │
│  domain-vocabulary.md ──┐                                    │
│                         ├──> domain-model.md ──┐             │
│  user-stories.md ───────┤                      │             │
│                         ├──> business-rules.md │             │
│  nfr.md ────────────────┘                      │             │
│                                                │             │
│                                                ▼             │
│                      ┌──────────────────────────────┐        │
│                      │  interface-contracts.md      │        │
│                      │  sequence-design.md          │        │
│                      │  initial-data-model.md       │        │
│                      │  aggregate-design.md         │        │
│                      └───────────┬──────────────────┘        │
│                                  │                           │
│  acceptance-criteria.md ─────────┤                           │
│                                  │                           │
│                                  ▼                           │
│                      ┌──────────────────────────────┐        │
│                      │  test-case.md (TDAI)         │        │
│                      │  feature-definition-ticket.md│        │
│                      │  service-boundaries.md       │        │
│                      │  event-message-design.md     │        │
│                      └──────────────────────────────┘        │
│                                                              │
└──────────────────────────────────────────────────────────────┘

Legend:
  ──> Derives from
  ─── Contributes to
```

---

## 🎯 Success Metrics

### Derivation Quality Metrics

Track these metrics to ensure derivation quality:

1. **Derivation Coverage:**
   - % of user stories with acceptance criteria (target: 100%)
   - % of acceptance criteria with tests (target: 100%)
   - % of entities with interface contracts (target: 100%)

2. **Derivation Accuracy:**
   - Human correction rate (target: <10%)
   - Re-derivation required (target: <5%)
   - Validation errors (target: 0)

3. **Derivation Speed:**
   - Time to derive L1 docs (target: <2 min)
   - Time to derive L2 docs (target: <5 min)
   - Time to derive L3 docs (target: <10 min)

4. **Human Effort Reduction:**
   - Human time saved (target: >80%)
   - Approval time (target: <20% of manual time)

---

## 💡 Best Practices

### DO:
- ✅ Always start with high-quality foundational docs (L0)
- ✅ Review and approve L1 derivations (high impact)
- ✅ Run validation after every derivation
- ✅ Use traceability to track change impact
- ✅ Iterate on derivation rules based on feedback

### DON'T:
- ❌ Skip validation steps
- ❌ Auto-approve foundational derivations (L0 → L1)
- ❌ Manually edit derived docs (re-derive instead)
- ❌ Ignore derivation errors/warnings
- ❌ Break traceability links

---

## 🏗️ Architecture Decisions

### ADR-001: CLI Orchestration vs Claude Code Subagents

**Döntés:** CLI orchestration (Go kód irányít)

**Kontextus:**
A dokumentáció "AI Agents" szekciója említi a specializált agent-eket (DomainModelAgent, TestGeneratorAgent, stb.). Két implementációs megközelítés lehetséges:

1. **Claude Code Task tool subagents** - Claude Code interaktívan indít subagent-eket
2. **CLI orchestration** - Go kód hívja a `claude -p` parancsot többször

**Döntés indoklása:**
- A loom-cli már Claude Code-ot hív headless módban (`claude -p`)
- A Task tool subagent-ek interaktív Claude Code session-t igényelnek
- A két megközelítés nem közvetlenül kompatibilis
- CLI orchestration explicit, reprodukálható, batch-műveletekhez optimális

**Következmény:**
Az "AI Agent" nevek (DomainModelAgent, stb.) koncepcionálisak - a tényleges implementáció prompt template + `claude -p` hívás.

---

### ADR-002: Monolitikus cmd fájlok vs Go Agent modulok

**Döntés:** Jelenlegi monolitikus struktúra marad (egyelőre)

**Kontextus:**
A derive parancsok (`derive_l2.go`: 797 sor, `derive_l3.go`: 876 sor) monolitikusak - type definíciók, prompt building, Claude hívások, JSON parsing, markdown generálás mind egy fájlban.

**Alternatíva (elhalasztva):**
```
internal/agents/
├── domain_model/agent.go
├── interface_contract/agent.go
├── test_generator/agent.go
└── ...
```

Előnyök lennének:
- Jobb tesztelhetőség (izolált unit testek)
- Tisztább felelősség (SRP)
- Könnyebb bővítés (új agent = új package)
- Kevesebb kód duplikáció (közös Agent interface)

**Döntés indoklása:**
- A jelenlegi kód működik
- A refaktorálás jelentős befektetés
- Nincs akut fájdalom (még)
- Prioritás: feature-ök > refaktorálás

**Újraértékelés triggerei:**
- Ha új deriváció típust kell hozzáadni
- Ha a kód karbantartása nehézkessé válik
- Ha unit tesztek kellenek az agent logikához

---

## 🚀 Roadmap

### Phase 1: Basic Derivation (MVP) ✅ COMPLETED
- [x] L0 → L1 derivations (vocabulary → model, stories → AC) - `loom-cli derive`
- [x] Manual approval workflow - Interactive mode (`-i` flag)
- [x] Basic validation - `loom-cli validate`

### Phase 2: Advanced Derivation ✅ COMPLETED
- [x] L1 → L2 derivations (model → contracts, sequences) - `loom-cli derive-l2`
- [x] L2 → L3 derivations (AC → tests with TDAI) - `loom-cli derive-l3`
- [x] Automated validation - `loom-cli validate --level ALL`
- [x] Dependency graph visualization - Mermaid diagrams in dependency-graph.md

### Phase 2.5: Validation Enhancements (Planned)
- [ ] YAML frontmatter validáció (status, date mezők)
- [ ] Folder struktúra validáció (dokumentum típus → elvárt mappa)
- [ ] V004: Bidirectional link check (jelenleg SKIP)
- [ ] V006: Entity-aggregate validáció (jelenleg SKIP)
- [ ] V007: Service-interface contract validáció (jelenleg SKIP)

### Phase 3: Agent System Features (Planned)

**Kaszkád deriváció**
- [ ] Változás automatikus propagálása függő szintekre
- [ ] Új user story → AC → contracts → tests (egy paranccsal)
- [ ] Dependency-aware re-derivation (csak érintett dokumentumok)

**Natural language interface**
- [ ] Intent Parser: természetes nyelvű input elemzése
- [ ] Automatikus derivation plan generálás
- [ ] Érintett entitások/dokumentumok azonosítása
- [ ] Példa: `loom generate "Add quote reminder email feature"`

**Intelligens approval workflow**
- [ ] L1: mindig emberi jóváhagyás (alapvető hatás)
- [ ] L2: opcionális (contracts igen, többi auto)
- [ ] L3: test plan jóváhagyás, generálás automatikus
- [ ] Confidence scoring (magas confidence → auto-approve)

### Phase 4: Learning & Analytics (Future)
- [ ] AI learns from corrections
- [ ] Derivation analytics & insights
- [ ] Pattern detection across projects

---

*This Documentation Derivation Strategy is the core of the Loom (AI-PDS) system. It defines how human intent flows through multiple levels of AI-generated documentation, ensuring consistency, traceability, and quality at every step.*

*Implementation: See `loom-tooling/loom-cli/` for the CLI implementation.*

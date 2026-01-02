# Deriválás Implementálási Terv

**Dátum:** 2024-12-29
**Cél:** Teljes L0→L1→L2→L3 deriválási pipeline a spec szerint

---

## Jelenlegi Állapot

### Meglévő deriválások:
| Szint | Dokumentum | Státusz |
|-------|------------|---------|
| L1 | acceptance-criteria.md | ✅ |
| L1 | business-rules.md | ✅ |
| L2 | test-cases.md | ✅ |
| L2 | tech-specs.md | ✅ |
| L3 | openapi.json | ✅ |
| L3 | implementation-skeletons.md | ✅ |

### Hiányzó deriválások (12 db):
| Szint | Dokumentum | Prioritás |
|-------|------------|-----------|
| L1 | domain-model.md | P1 - Critical |
| L1 | bounded-context-map.md | P2 - Important |
| L2 | interface-contracts.md | P1 - Critical |
| L2 | sequence-design.md | P1 - Critical |
| L2 | aggregate-design.md | P2 - Important |
| L2 | initial-data-model.md | P2 - Important |
| L2 | workflow-definitions.md | P3 - Nice to have |
| L3 | feature-definition-ticket.md | P1 - Critical |
| L3 | service-boundaries.md | P2 - Important |
| L3 | event-message-design.md | P2 - Important |
| L3 | dependency-graphs.md | P3 - Nice to have |
| L3 | adr.md | P3 - Nice to have |

---

## Implementálási Fázisok

### Fázis 1: L1 Bővítés (Domain Model + Context Map)

#### 1.1 Domain Model Deriválás
**Input:** L0 (user-stories.md, domain-vocabulary.md)
**Output:** `domain-model.md`

**Tartalom (guideline 2232-aggregate-design alapján):**
- Entities és Value Objects
- Aggregate Root-ok
- Invariants
- Creation Rules
- State Transitions (Behaviour)
- Events
- Boundaries

**Prompt struktúra:**
```
Input: User Stories + Domain Vocabulary
Output JSON:
{
  "entities": [
    {
      "id": "ENT-XXX",
      "name": "...",
      "type": "aggregate_root|entity|value_object",
      "attributes": [...],
      "invariants": [...],
      "operations": [...],
      "events": [...]
    }
  ],
  "relationships": [...]
}
```

**CLI parancs:** `loom-cli derive` (bővítve)

#### 1.2 Bounded Context Map Deriválás
**Input:** L1 domain-model.md
**Output:** `bounded-context-map.md`

**Tartalom (guideline 2231-service-boundaries alapján):**
- Context boundaries
- Upstream/Downstream relationships
- Shared Kernel / Customer-Supplier / Conformist patterns
- Mermaid diagram

---

### Fázis 2: L2 Bővítés (Interface + Sequence + Aggregate + Data Model)

#### 2.1 Interface Contracts Deriválás
**Input:** L1 (domain-model.md, business-rules.md, acceptance-criteria.md)
**Output:** `interface-contracts.md`

**Tartalom (guideline 2237-interface-contract alapján):**
- Purpose
- Operations (Name, Description, Input/Output schema, Errors, Pre/Postconditions)
- Message/Event Definitions
- Data Types and Shared Models
- Versioning Rules
- Security & Access Control

**CLI parancs:** `loom-cli derive-l2` (bővítve)

#### 2.2 Sequence Design Deriválás
**Input:** L1 + L2 (interface-contracts.md)
**Output:** `sequence-design.md`

**Tartalom (guideline 2236-sequence-design alapján):**
- Trigger
- Participants
- Step-by-Step Flow
- Outcome
- Exceptions
- Mermaid sequence diagram

#### 2.3 Aggregate Design Deriválás
**Input:** L1 (domain-model.md, business-rules.md)
**Output:** `aggregate-design.md`

**Tartalom (guideline 2232-aggregate-design alapján):**
- Purpose
- Invariants
- Entities and Value Objects
- Creation Rules
- State Transitions
- Events
- Boundaries

#### 2.4 Initial Data Model Deriválás
**Input:** L1 (domain-model.md), L2 (aggregate-design.md)
**Output:** `initial-data-model.md`

**Tartalom:**
- Database tables/collections
- Field definitions (type, constraints)
- Indexes
- Relationships (FK, references)
- Migration scripts (pseudo)

---

### Fázis 3: L3 Bővítés (Feature Tickets + Services + Events + ADR)

#### 3.1 Feature Definition Ticket Deriválás
**Input:** L1 (acceptance-criteria.md), L2 (test-cases.md)
**Output:** `feature-definition-tickets/FDT-XXX.md`

**Tartalom (guideline 2311-feature-definition alapján):**
- Business goal
- User story
- Acceptance criteria (linked)
- Non-functional requirements
- Dependencies
- Impact assessment
- Out of scope

**CLI parancs:** `loom-cli derive-l3` (bővítve)

#### 3.2 Service Boundaries Deriválás
**Input:** L1 (bounded-context-map.md), L2 (aggregate-design.md)
**Output:** `service-boundaries.md`

**Tartalom (guideline 2231-service-boundaries alapján):**
- Purpose
- Capabilities
- Inputs/Outputs
- Ownership (aggregates)
- External Dependencies
- Reasons for separation

#### 3.3 Event Message Design Deriválás
**Input:** L1 (domain-model.md events), L2 (sequence-design.md)
**Output:** `event-message-design.md`

**Tartalom (guideline 2233-event-message-design alapján):**
- Domain Events (Purpose, Trigger, Payload, Invariants, Consumers, Versioning)
- Commands (Intent, Required Data, Expected Outcome, Failure Conditions)
- Integration Events

#### 3.4 Dependency Graphs Deriválás
**Input:** L3 (service-boundaries.md)
**Output:** `dependency-graphs.md`

**Tartalom (guideline 2238-dependency-graphs alapján):**
- Components
- Dependency Types (sync/async/data/external)
- Direction
- Mermaid flowchart

#### 3.5 ADR (Architecture Decision Records)
**Input:** All levels (changes, decisions)
**Output:** `adr/ADR-XXX.md`

**Tartalom (guideline 2234-adr alapján):**
- Context
- Decision
- Consequences
- Status
- Related Documents

---

## CLI Parancs Struktúra (Tervezett)

```
loom-cli derive [options]
  --level L1|L2|L3|all
  --docs domain-model,bounded-context,interface-contracts,...
  --input-dir <path>
  --output-dir <path>

Példák:
  loom-cli derive --level L1 --output-dir ./l1
  loom-cli derive --level L2 --input-dir ./l1 --output-dir ./l2
  loom-cli derive --level L3 --input-dir ./l2 --output-dir ./l3
  loom-cli derive --level all --output-dir ./docs
```

---

## Implementálási Sorrend (MVP)

### Sprint 1: L1 Domain Foundation
1. `domain-model.md` prompt + derive command
2. `bounded-context-map.md` prompt + derive command
3. Teszt benchmark-kal

### Sprint 2: L2 Architecture Documents
1. `interface-contracts.md` prompt (bővíti derive-l2)
2. `sequence-design.md` prompt (bővíti derive-l2)
3. `aggregate-design.md` prompt (bővíti derive-l2)
4. `initial-data-model.md` prompt (bővíti derive-l2)
5. Teszt benchmark-kal

### Sprint 3: L3 Implementation Documents
1. `feature-definition-ticket.md` prompt (bővíti derive-l3)
2. `service-boundaries.md` prompt (bővíti derive-l3)
3. `event-message-design.md` prompt (bővíti derive-l3)
4. `dependency-graphs.md` prompt (bővíti derive-l3)
5. Teszt benchmark-kal

### Sprint 4: Polish & Integration
1. `adr.md` generálás
2. Teljes pipeline teszt
3. Dokumentáció

---

## Traceability Model

```
L0 User Story (US-XXX)
    │
    ├── L1 Acceptance Criteria (AC-XXX-N)
    │       │
    │       ├── L2 Test Case (TC-XXX-N)
    │       └── L3 Feature Ticket (FDT-XXX)
    │
    ├── L1 Business Rule (BR-XXX)
    │       │
    │       ├── L2 Tech Spec (TS-XXX)
    │       └── L2 Interface Contract (IC-XXX)
    │
    └── L1 Domain Model (ENT-XXX)
            │
            ├── L2 Aggregate Design (AGG-XXX)
            ├── L2 Sequence Design (SEQ-XXX)
            ├── L3 Service Boundary (SVC-XXX)
            └── L3 Event Design (EVT-XXX)
```

---

## Becsült Munka

| Fázis | Prompt | Go kód | Teszt | Összesen |
|-------|--------|--------|-------|----------|
| Sprint 1 (L1) | 2 prompt | 1 file | 1 teszt | ~2 óra |
| Sprint 2 (L2) | 4 prompt | 1 file | 1 teszt | ~3 óra |
| Sprint 3 (L3) | 4 prompt | 1 file | 1 teszt | ~3 óra |
| Sprint 4 | 1 prompt | - | 1 teszt | ~1 óra |
| **Összesen** | **11 prompt** | **3 file** | **4 teszt** | **~9 óra** |

---

## Döntési Pontok

1. **Egy parancs vs. több parancs?**
   - Opció A: `derive --level L1/L2/L3` (unified)
   - Opció B: `derive`, `derive-l2`, `derive-l3` (current)
   - **Javaslat:** Maradjunk a jelenlegi struktúránál, de bővítsük

2. **Minden dokumentum egy hívásban vs. külön?**
   - Opció A: Egy nagy JSON output minden L1 doc-kal
   - Opció B: Külön hívás minden doc típushoz (jelenlegi)
   - **Javaslat:** Külön hívás (stabilitás, kisebb output)

3. **Mermaid diagram generálás?**
   - Opció A: JSON-ben, majd CLI rendereli markdown-ba
   - Opció B: Prompt közvetlenül markdown+mermaid-et ad
   - **Javaslat:** Opció B (egyszerűbb)

---

## Következő Lépések

1. [ ] Terv jóváhagyása
2. [ ] Sprint 1 indítása: domain-model.md prompt
3. [ ] Implementálás és tesztelés
4. [ ] Commit

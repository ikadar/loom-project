# CLI Prompts Fejlesztési Terv

> **Probléma:** A CLI promptok (`loom-cli/prompts/prompts.go`) jelentősen egyszerűsítve vannak a commands-hoz képest. A minőség javítása szükséges.

## Jelenlegi Állapot

### Gap Elemzés

| Komponens | Commands | CLI | Hiányzik |
|-----------|----------|-----|----------|
| Entity checklist | 50+ kérdés, 8 szekció | 10 kérdés | 80% |
| Operation checklist | 60+ kérdés, 8 szekció | 12 kérdés | 80% |
| Edge case generation | 14-17 auto-generated | 0 | 100% |
| Domain modeling decisions | 15+ decision points | 0 | 100% |
| Quality criteria | 8+ per analysis | 0 | 100% |
| Output format spec | Részletes markdown | Basic JSON | 70% |
| Examples | Minden prompthoz | 0 | 100% |

### Érintett Fájlok

**Forrás (commands):**
- `commands/loom-analyze-entities.md`
- `commands/loom-analyze-operations.md`
- `commands/loom-derive-domain.md`
- `commands/loom-validate.md`
- `docs/checklists/entity-checklist.md`
- `docs/checklists/operation-checklist.md`

**Cél (CLI):**
- `loom-cli/prompts/prompts.go`

---

## Fejlesztési Terv

### Fázis 1: Checklist Integráció (P1 - Kritikus)

#### 1.1 Entity Analysis Prompt Bővítése

**Jelenlegi (37 sor):**
```go
const EntityAnalysis = `You are a requirements completeness analyst...

ENTITY COMPLETENESS CHECKLIST:
1. IDENTITY: Is there a unique identifier?
2. ATTRIBUTES: Are all attributes listed?
...
10. CONSTRAINTS: Any business constraints?
`
```

**Cél (~200+ sor):**
```go
const EntityAnalysis = `You are a requirements completeness analyst...

## A. Attribute Definition
For EACH attribute, check:
| Aspect | Question |
|--------|----------|
| Type | What is the exact data type? (string, int, float, date, datetime, enum, ref, array) |
| Required on Create | Is this field required when creating? |
| Required on Update | Is this field required when updating? |
| Default | What is the default value if not provided? |
| Unique | Must it be unique? Within what scope? |
| Min Value/Length | What is the minimum? |
| Max Value/Length | What is the maximum? |
| Format/Pattern | What format must it match? |
| Validation Rules | What other validation applies? |

## B. Enum Values
For EACH enum/status attribute:
- Values: What are ALL valid values?
- Initial: What is the initial value on creation?
- Transitions: What transitions are allowed?
- Transition Trigger: What triggers each transition?
- Transition Actor: Who can trigger each transition?

## C. Lifecycle
...

## H. Edge Cases (Auto-Generate)
For EACH entity, automatically generate questions for:
| Edge Case | Question Template |
|-----------|-------------------|
| All Optional Empty | What if all optional fields are empty/null? |
| Whitespace Only | What if string field contains only whitespace? |
| Min Boundary | What happens at minimum value/length? |
...
`
```

#### 1.2 Operation Analysis Prompt Bővítése

Hasonlóan bővíteni a teljes `operation-checklist.md` tartalmával:
- A. Basic Definition (9 aspect)
- B. Input Definition (9 aspect per input)
- C. Output Definition (5 aspect)
- D. Error Handling (9 error types + 6 aspects each)
- E. Side Effects (8 types)
- F. Performance & Limits (7 aspects)
- G. Edge Cases (17 auto-generated)
- H. Transaction Boundaries (4 aspects)

---

### Fázis 2: Domain Modeling Decision Points (P1 - Kritikus)

#### 2.1 Új Prompt: Domain Discovery Enhanced

Hozzáadni az EVO/AGG/REF/INV decision points-ot:

```go
const DomainDiscoveryEnhanced = `You are a domain analysis expert...

## Decision Points for Classification

### Entity vs Value Object (EVO)
| ID | Decision Point | Question | Criteria |
|----|----------------|----------|----------|
| EVO-1 | Independent identity | Does {concept} need to be tracked independently? | Yes → Entity |
| EVO-2 | Lifecycle independence | Can {concept} exist without {parent}? | Yes → Entity |
| EVO-3 | Mutability | Do you need to modify while keeping identity? | Yes → Entity |
| EVO-4 | External references | Is {concept} referenced from outside aggregate? | Yes → Entity |
| EVO-5 | Equality semantics | Are two {concepts} equal if attributes match? | Yes → Value Object |

### Aggregate Boundaries (AGG)
| ID | Decision Point | Question | Criteria |
|----|----------------|----------|----------|
| AGG-1 | Transactional boundary | Must they be modified together atomically? | Yes → Same aggregate |
| AGG-2 | Consistency boundary | Must they be consistent immediately? | Yes → Same aggregate |
| AGG-3 | Independent lifecycle | Can it be created/deleted independently? | Yes → Separate aggregate |
| AGG-4 | Access pattern | Do you need to load it without parent? | Yes → Separate aggregate |

### Reference Types (REF)
...

### Invariants (INV)
...

For EACH concept with Low/Medium confidence, ASK the relevant decision point questions.
`
```

#### 2.2 Interview Prompt Bővítése

A jelenlegi interview prompt nem tartalmazza a decision point logikát:

```go
const InterviewPromptEnhanced = `You are conducting a structured interview...

## Question Categories

### 1. Entity/Value Object Classification
For each concept needing classification:
- Present EVO decision point questions
- Collect answers
- Apply decision logic
- Document rationale

### 2. Aggregate Boundary Decisions
...

### 3. Business Rule Clarification
...

## Question Presentation Format
---
**[Category: Subject]** (Decision Point: {ID})

Q: {question}

Options:
a) {option1} → implies {classification}
b) {option2} → implies {classification}
c) {option3}
d) Other (please specify)

Suggested default: {suggestion}
---

## Answer Processing
After each answer, document:
- Decision Point ID
- Answer selected
- Classification result
- Rationale
`
```

---

### Fázis 3: Quality Criteria és Validation (P2 - Fontos)

#### 3.1 Quality Criteria Hozzáadása Minden Prompthoz

```go
const EntityAnalysisQualityCriteria = `
## Quality Criteria - Verify Before Output

Before outputting, verify:
- [ ] Every entity has been analyzed
- [ ] Every attribute has type/required/default checked
- [ ] Every enum has values/transitions checked
- [ ] Every relationship has cascade behavior checked
- [ ] Every lifecycle phase checked
- [ ] Edge cases generated for each entity
- [ ] Severities correctly assigned
- [ ] No duplicate ambiguity IDs
`
```

#### 3.2 Új Prompt: Validation

Jelenleg nincs validation prompt a CLI-ben. Hozzáadni a commands/loom-validate.md alapján:

```go
const ValidationPrompt = `You are a Loom Document Validator...

## Validation Checks

### 1. Traceability Check
- Every AC-XXX-Y references an existing US-XXX
- Every BR-XXX references an existing US-XXX or AC-XXX-Y
- Every TC-XXX-YY references existing AC/BR
- No dangling references

### 2. Format Check
- YAML frontmatter valid
- ID conventions followed
- AC uses Given/When/Then
- BR has Rule/Invariant/Enforcement

### 3. Coverage Check
- Every US has at least 1 AC
- Every AC has at least 1 TC
- Every BR with error code has negative TC

### 4. Consistency Check
- No duplicate IDs
- State transitions consistent
- No contradicting decisions
`
```

---

### Fázis 4: Output Format Standardizálás (P2 - Fontos)

#### 4.1 Structured Output Templates

```go
const AmbiguityOutputFormat = `
Return ambiguities in this exact JSON structure:

{
  "summary": {
    "entities_analyzed": N,
    "operations_analyzed": N,
    "total_ambiguities": N,
    "by_severity": {
      "critical": N,
      "important": N,
      "minor": N
    }
  },
  "ambiguities": [
    {
      "id": "AMB-ENT-001",
      "category": "entity|operation|relationship|rule",
      "subject": "EntityName",
      "aspect": "attribute.type|lifecycle.deletion|etc",
      "question": "Clear question text",
      "severity": "critical|important|minor",
      "severity_rationale": "Why this severity",
      "suggested_answer": "Default if applicable",
      "options": ["Option A", "Option B"],
      "depends_on": ["AMB-XXX-NNN"],  // for skip logic
      "skip_if": "condition that makes this irrelevant",
      "checklist_section": "A.Attributes|B.Enum|etc",
      "edge_case_type": "boundary|null|concurrent|etc"  // if auto-generated
    }
  ]
}
`
```

---

### Fázis 5: Edge Case Generation (P2 - Fontos)

#### 5.1 Entity Edge Cases

```go
const EntityEdgeCases = `
## Auto-Generate Edge Case Questions

For EACH entity, generate questions for:

| Edge Case | Question Template |
|-----------|-------------------|
| All Optional Empty | "What if all optional fields of {entity} are empty/null?" |
| Whitespace Only | "What if {string_attr} contains only whitespace?" |
| Min Boundary | "What happens when {numeric_attr} is at minimum ({min})?" |
| Max Boundary | "What happens when {numeric_attr} is at maximum ({max})?" |
| Just Below Min | "What happens when {numeric_attr} is {min-1}?" |
| Just Above Max | "What happens when {numeric_attr} is {max+1}?" |
| Distant Past | "What if {date_attr} is in distant past (1900-01-01)?" |
| Distant Future | "What if {date_attr} is far in future (2100-01-01)?" |
| Invalid Reference | "What if {ref_attr} points to non-existent {target}?" |
| Circular Reference | "What if {entity} references itself (directly or indirectly)?" |
| Unicode/Special | "Are unicode and special characters allowed in {string_attr}?" |
| Duplicate Attempt | "What if exact duplicate {entity} is created?" |
| Max Length Input | "Performance impact of {string_attr} at max length ({max})?" |
| Concurrent Create | "What if two users create same {entity} simultaneously?" |
`
```

#### 5.2 Operation Edge Cases

```go
const OperationEdgeCases = `
## Auto-Generate Edge Case Questions

For EACH operation, generate questions for:

| Edge Case | Question Template |
|-----------|-------------------|
| All Optional Omitted | "What if {operation} called with only required inputs?" |
| Min Boundary Input | "What if {input} is at minimum value?" |
| Max Boundary Input | "What if {input} is at maximum value?" |
| Below Min | "What if {input} is below minimum?" |
| Above Max | "What if {input} is above maximum?" |
| Empty String | "What if {string_input} is empty string?" |
| Null Value | "What if {input} is null?" |
| Empty Array | "What if {array_input} is empty array?" |
| Target Not Found | "What if target {entity} doesn't exist?" |
| Target Just Deleted | "What if {entity} deleted between check and {operation}?" |
| Rapid Repeat | "What if {operation} called twice rapidly (within 100ms)?" |
| During Loading | "What if {operation} triggered while previous still loading?" |
| Network Failure Mid-Op | "What if network fails during {operation}?" |
| Partial Data | "What if some required data unavailable?" |
| Stale Data | "What if operating on stale/outdated data?" |
| Large Batch | "What if batch size at maximum ({max})?" |
| Concurrent Same Target | "What if two users {operation} same {entity} simultaneously?" |
`
```

---

## Implementációs Terv

### Prioritás és Sorrend

| Fázis | Tartalom | Prioritás | Becsült méret |
|-------|----------|-----------|---------------|
| 1.1 | Entity Analysis bővítés | P1 | +150 sor |
| 1.2 | Operation Analysis bővítés | P1 | +170 sor |
| 2.1 | Domain Discovery Enhanced | P1 | +100 sor |
| 2.2 | Interview Prompt Enhanced | P1 | +80 sor |
| 3.1 | Quality Criteria | P2 | +50 sor |
| 3.2 | Validation Prompt | P2 | +100 sor |
| 4.1 | Output Format Templates | P2 | +60 sor |
| 5.1 | Entity Edge Cases | P2 | +40 sor |
| 5.2 | Operation Edge Cases | P2 | +50 sor |

**Összesen:** ~800 sor hozzáadás (jelenlegi ~200 sor → ~1000 sor)

### Milestone Kapcsolat

| Fázis | Milestone |
|-------|-----------|
| 1-2 (P1) | **M4** - Platform Implementation |
| 3-5 (P2) | **M6** - Beta & Iteration |

---

## Alternatív Megközelítés: Külső Checklist Fájlok

Ahelyett, hogy a promptokba építjük a teljes checklistet:

```go
// prompts.go
const EntityAnalysis = `You are a requirements completeness analyst.

Use the checklist from: {{.ChecklistPath}}

...
`

// Külön fájlok:
// - checklists/entity-checklist.md (már létezik!)
// - checklists/operation-checklist.md (már létezik!)
```

**Előnyök:**
- Könnyebb karbantartás
- Újrafelhasználható (commands és CLI is ugyanazt használja)
- Kisebb prompt méret (de több file read)

**Hátrányok:**
- Több file kezelés
- SaaS-nak is ki kell szolgálnia a checklisteket

---

## Döntések

1. **Beépített vs Külső checklistek?**
   - ✅ **Külső fájlok hivatkozása** (jelenlegi `checklists/` mappa)
   - A promptok hivatkozzák a checklist fájlokat
   - SaaS is ezeket szolgáltatja ki

2. **Token limit kezelés?**
   - ✅ **Teljes checklist minden híváskor**
   - Nincs dinamikus szekció kiválasztás
   - Egyszerűbb implementáció

3. **Tesztelés?**
   - ✅ **Hibrid megközelítés:**
     - M4: Benchmark készlet + Manual review
     - M6: A/B test valós user-ekkel

---

## Benchmark Teszt Készlet

### Struktúra

```
loom-tooling/
└── test/
    └── benchmark/
        ├── README.md
        ├── 01-ecommerce-order/
        │   ├── input-l0.md
        │   ├── expected-entities.json
        │   ├── expected-ambiguities.json
        │   └── expected-severity.json
        ├── 02-scheduling-calendar/
        │   └── ...
        ├── 03-fintech-payment/
        │   └── ...
        ├── 04-simple-crud-todo/
        │   └── ...
        └── 05-multitenant-saas/
            └── ...
```

### Benchmark Dokumentumok

| # | Domain | Komplexitás | Fókusz |
|---|--------|-------------|--------|
| 1 | E-commerce (Order) | Közepes | Entity lifecycle, relationships |
| 2 | Scheduling (Calendar) | Magas | Temporal constraints, conflicts |
| 3 | Fintech (Payment) | Magas | State machine, error handling |
| 4 | Simple CRUD (Todo) | Alacsony | Basic coverage, baseline |
| 5 | Multi-tenant SaaS | Magas | Authorization, isolation |

### Expected Output Formátum

**expected-entities.json:**
```json
{
  "entities": [
    {
      "name": "Order",
      "type": "entity",
      "confidence": "high",
      "attributes_expected": ["id", "status", "total", "customerId"],
      "relationships_expected": ["Customer", "OrderItem"]
    }
  ]
}
```

**expected-ambiguities.json:**
```json
{
  "minimum_ambiguities": [
    {
      "id_pattern": "AMB-ENT-*",
      "subject": "Order",
      "aspect_contains": "deletion",
      "severity": "critical"
    },
    {
      "id_pattern": "AMB-ENT-*",
      "subject": "OrderItem",
      "aspect_contains": "cascade",
      "severity": "critical"
    }
  ],
  "minimum_count": {
    "critical": 5,
    "important": 10,
    "minor": 5
  }
}
```

### Teszt Runner

```bash
# Benchmark futtatás
./test/run-benchmark.sh

# Output:
# Benchmark 1: ecommerce-order
#   Entities found: 5/5 ✅
#   Critical ambiguities: 7/5 ✅ (exceeded minimum)
#   Important ambiguities: 12/10 ✅
#   Missing expected: 0 ✅
#
# Benchmark 2: scheduling-calendar
#   ...
#
# OVERALL: 5/5 passed
```

### Értékelési Kritériumok

| Metrika | Pass Criteria |
|---------|---------------|
| Entity detection | ≥90% of expected entities found |
| Ambiguity count | ≥ minimum per severity |
| Critical coverage | 100% of expected critical ambiguities |
| False positive rate | <20% irrelevant ambiguities |
| Edge case generation | ≥5 auto-generated per entity |

---

## Tesztelési Terv

| Fázis | Tesztelés | Mikor | Cél |
|-------|-----------|-------|-----|
| Fejlesztés közben | Manual review | M4 | Kvalitatív feedback |
| Fázis 1 után | Benchmark 1-2 | M4 | Entity/Operation coverage |
| Fázis 2 után | Benchmark 3-5 | M4 | Domain modeling quality |
| MVP előtt | Full benchmark | M4 | Regression check |
| MVP után | A/B test | M6 | User validation |

---

## Következő Lépések

1. [x] Döntés: külső checklistek ✅
2. [x] Döntés: teljes checklist minden híváskor ✅
3. [x] Döntés: hibrid tesztelés ✅
4. [x] Benchmark készlet struktúra létrehozása ✅
5. [x] Benchmark 1 (ecommerce) elkészítése ✅
6. [ ] Fázis 1.1 implementálás (Entity Analysis)
7. [ ] Tesztelés benchmark 1-gyel
8. [ ] Fázis 1.2 implementálás (Operation Analysis)
9. [ ] Fázis 2 implementálás (Domain Modeling)
10. [ ] Full benchmark teszt

---

## Kapcsolódó Dokumentumok

- `commands/loom-analyze-entities.md`
- `commands/loom-analyze-operations.md`
- `commands/loom-derive-domain.md`
- `docs/checklists/entity-checklist.md`
- `docs/checklists/operation-checklist.md`
- `test/benchmark/` - Benchmark teszt készlet (01-ecommerce-order elkészült)

---

*Létrehozva: 2025-12-29*

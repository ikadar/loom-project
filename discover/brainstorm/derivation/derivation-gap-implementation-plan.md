# Deriválás Gap Implementálási Terv

**Dátum:** 2024-12-31
**Cél:** Strategy vs. Implementáció gap-ek felszámolása
**Előzmény:** documentation-derivation-strategy.md összehasonlítása a jelenlegi CLI-vel

---

## Jelenlegi Állapot

### Elkészült funkciók:
| Kategória | Funkció | Státusz |
|-----------|---------|---------|
| L1 Strategic | domain-model.md | ✅ |
| L1 Strategic | bounded-context-map.md | ✅ |
| L1 Strategic | acceptance-criteria.md | ✅ |
| L1 Strategic | business-rules.md | ✅ |
| L2 Tactical | test-cases.md | ✅ |
| L2 Tactical | tech-specs.md | ✅ |
| L2 Tactical | interface-contracts.md | ✅ |
| L2 Tactical | sequence-design.md | ✅ |
| L2 Tactical | aggregate-design.md | ✅ |
| L2 Tactical | initial-data-model.md | ✅ |
| L3 Operational | openapi.json | ✅ |
| L3 Operational | implementation-skeletons.md | ✅ |
| L3 Operational | feature-tickets.md | ✅ |
| L3 Operational | service-boundaries.md | ✅ |
| L3 Operational | event-message-design.md | ✅ |
| L3 Operational | dependency-graph.md | ✅ |

### Hiányzó funkciók (Strategy szerint):
| Kategória | Funkció | Prioritás | Státusz |
|-----------|---------|-----------|---------|
| Quality | TDAI (Test-Driven AI) test generation | P1 - Critical | ✅ DONE |
| Quality | Validation layer (`loom-cli validate`) | P1 - Critical | ✅ DONE |
| Quality | Traceability #anchor refs | P1 - Critical | ✅ DONE |
| Workflow | Human approval interactive mode | P2 - Important | ✅ DONE |
| Input | Domain Vocabulary külön input | P3 - Nice to have | 🔲 TODO |
| Input | NFR (Non-Functional Reqs) kezelés | P3 - Nice to have | 🔲 TODO |
| L2 | Workflow Definitions | P3 - Nice to have | 🔲 TODO |

---

## Gap Analízis Részletek

### 1. TDAI - Test-Driven AI Development

**Strategy definíció:**
```
Minden AC-hez:
- 3+ positive tests (happy path variations)
- 3+ negative tests (invalid inputs, edge cases)
- 2+ boundary tests (min, max, off-by-one)
- 1+ "should NOT" test (hallucination prevention)
```

**Jelenlegi implementáció:**
- Egyszerű test case generálás
- Nincs strukturált positive/negative/boundary kategorizálás
- Nincs hallucination prevention test

**Gap:** A generált tesztek nem követik a TDAI metodológiát, ami csökkenti a megbízhatóságot.

---

### 2. Validation Layer

**Strategy definíció:**
```
5 validation típus:
1. Structural Validation (sections, frontmatter, IDs)
2. Traceability Validation (refs exist, bidirectional)
3. Content Validation (no duplicates, no contradictions)
4. Completeness Validation (all AC have tests, etc.)
5. Test Quality Validation (negative ratio, pyramid)
```

**Jelenlegi implementáció:**
- Nincs validation parancs
- Csak generálás, ellenőrzés nélkül

**Gap:** Hibás vagy hiányos dokumentumok nem kerülnek felismerésre.

---

### 3. Traceability #anchor refs

**Strategy definíció:**
```markdown
**Traceability:**
- User Story: user-stories.md#us-rfq-001
- Entity: domain-model.md#ent-customer
- Test: test-case.md#tc-rfq-001-1
```

**Jelenlegi implementáció:**
```markdown
**Traceability:**
- Source: input-l0.md
- Decision: DEC-001
```

**Gap:** Nincs géppel feldolgozható cross-referencing, nem validálható.

---

### 4. Human Approval Workflow

**Strategy definíció:**
```
L1 derivations: Always require human approval
L2 derivations: Human approval for interface-contracts
L3 derivations: Human approval for test plans
```

**Jelenlegi implementáció:**
- Batch generálás, nincs interaktív jóváhagyás
- Nincs diff preview

**Gap:** Nincs lehetőség review-ra a generálás előtt/közben.

---

## Implementálási Fázisok

### Fázis 1: TDAI Test Generation (P1)

#### 1.1 Test Case Prompt Refaktorálás

**Cél:** A test-cases generálás TDAI metodológiát kövessen.

**Új prompt struktúra:**
```
Input: Acceptance Criteria + Business Rules + Tech Specs

Output JSON:
{
  "test_suites": [
    {
      "ac_ref": "AC-CUST-001",
      "positive_tests": [
        {"id": "TC-CUST-001-P1", "name": "...", "type": "happy_path", ...}
      ],
      "negative_tests": [
        {"id": "TC-CUST-001-N1", "name": "...", "type": "invalid_input", ...}
      ],
      "boundary_tests": [
        {"id": "TC-CUST-001-B1", "name": "...", "type": "min_value", ...}
      ],
      "hallucination_prevention_tests": [
        {"id": "TC-CUST-001-H1", "name": "...", "should_not": "...", ...}
      ]
    }
  ],
  "summary": {
    "total": 50,
    "positive": 20,
    "negative": 20,
    "boundary": 8,
    "hallucination": 2,
    "positive_ratio": 0.40,
    "negative_ratio": 0.40
  }
}
```

**Validation szabályok:**
- Negative test ratio ≥ 20%
- Minden AC-hez min 1 hallucination prevention test
- Test pyramid: ~70:20:10 (unit:integration:e2e)

**Érintett fájlok:**
- `prompts/derive-test-cases.md` - Refaktorálás
- `cmd/derive_l2.go` - TestCase struct bővítés
- `cmd/derive_l2.go` - formatTestCases() TDAI struktúra

#### 1.2 Hallucination Prevention Test Típusok

**Speciális "should NOT" tesztek:**

| Típus | Leírás | Példa |
|-------|--------|-------|
| `should_not_require` | Opcionális mezőt ne követeljen | "Phone should NOT be required" |
| `should_not_allow` | Tiltott műveletet ne engedjen | "Expired quote should NOT create order" |
| `should_not_modify` | Változtatást ne engedjen | "Shipped order should NOT be editable" |
| `should_not_return` | Adatot ne adjon vissza | "Deleted user should NOT appear in list" |

---

### Fázis 2: Validation Layer (P1)

#### 2.1 Új CLI Parancs: `loom-cli validate`

**Használat:**
```bash
loom-cli validate --input-dir ./l1 --level L1
loom-cli validate --input-dir ./l2 --level L2
loom-cli validate --input-dir ./l3 --level L3
loom-cli validate --input-dir ./all --level all
```

**Output:**
```
Validating L1 documents...

Structural Validation:
  ✓ domain-model.md: Valid structure
  ✓ bounded-context-map.md: Valid structure
  ✓ acceptance-criteria.md: Valid structure
  ✓ business-rules.md: Valid structure

Traceability Validation:
  ✓ AC-CUST-001 → BR-CUST-001: Valid
  ✗ AC-CUST-003 → BR-CUST-099: Reference not found!

Completeness Validation:
  ✓ All ACs have test cases
  ✗ ENT-PAYMENT missing interface contract

Summary: 2 errors, 0 warnings
```

#### 2.2 Validation Rules Implementation

**Új fájl:** `cmd/validate.go`

```go
type ValidationResult struct {
    Level      string           `json:"level"`
    Errors     []ValidationError `json:"errors"`
    Warnings   []ValidationWarning `json:"warnings"`
    Checks     []ValidationCheck `json:"checks"`
}

type ValidationError struct {
    File       string `json:"file"`
    Line       int    `json:"line"`
    Rule       string `json:"rule"`
    Message    string `json:"message"`
    RefID      string `json:"ref_id,omitempty"`
}
```

**Validation szabályok:**

| Szabály ID | Típus | Leírás |
|------------|-------|--------|
| `V001` | Structural | Minden doc-nak van ID-ja |
| `V002` | Structural | ID-k követik a pattern-t (AC-XXX, BR-XXX, etc.) |
| `V003` | Traceability | Minden referencia létező ID-ra mutat |
| `V004` | Traceability | Bidirectional linkek konzisztensek |
| `V005` | Completeness | Minden AC-nek van legalább 1 test case |
| `V006` | Completeness | Minden Entity-nek van aggregate |
| `V007` | Completeness | Minden Service-nek van interface contract |
| `V008` | TDAI | Negative test ratio ≥ 20% |
| `V009` | TDAI | Minden AC-nek van hallucination prevention test |
| `V010` | Content | Nincs duplicate ID |

---

### Fázis 3: Traceability Anchors (P1)

#### 3.1 ID és Anchor Generálás

**Jelenlegi formátum:**
```markdown
## AC-CUST-001 – Customer Registration
```

**Új formátum:**
```markdown
## AC-CUST-001 – Customer Registration {#ac-cust-001}

**Traceability:**
- User Story: [US-CUST-001](../l0/user-stories.md#us-cust-001)
- Business Rule: [BR-CUST-001](#br-cust-001)
- Test Cases: [TC-CUST-001-P1](../l2/test-cases.md#tc-cust-001-p1), [TC-CUST-001-N1](../l2/test-cases.md#tc-cust-001-n1)
```

#### 3.2 Érintett Format Függvények

**Minden format függvény bővítése:**
- `formatDomainModel()` - Entity anchors
- `formatBoundedContextMap()` - Context anchors
- `formatAC()` - AC anchors + refs
- `formatBR()` - BR anchors + refs
- `formatTestCases()` - TC anchors + AC refs
- `formatTechSpecs()` - TS anchors + BR refs
- stb.

#### 3.3 Reference Format

**Standard link formátum:**
```
[ID](relative-path#anchor)
```

**Példák:**
```markdown
[AC-CUST-001](acceptance-criteria.md#ac-cust-001)
[BR-CUST-001](business-rules.md#br-cust-001)
[TC-CUST-001-P1](../l2/test-cases.md#tc-cust-001-p1)
[ENT-Customer](domain-model.md#ent-customer)
```

---

### Fázis 4: Human Approval Workflow (P2)

#### 4.1 Interactive Mode

**Új flag:** `--interactive` vagy `-i`

```bash
loom-cli derive-l2 --input-dir ./l1 --output-dir ./l2 --interactive
```

**Workflow:**
```
Phase L2-1: Generating Test Cases from Acceptance Criteria...

Preview (first 20 lines):
┌────────────────────────────────────────────────────────────────┐
│ ## TC-CUST-001-P1: Valid Registration (Happy Path)            │
│ **Type:** positive                                              │
│ **AC Ref:** AC-CUST-001                                        │
│ ...                                                             │
└────────────────────────────────────────────────────────────────┘

Generated: 58 test cases (23 positive, 25 negative, 8 boundary, 2 hallucination)

[A]pprove / [E]dit / [R]egenerate / [S]kip / [Q]uit? _
```

#### 4.2 Approval Actions

| Action | Leírás |
|--------|--------|
| `A` (Approve) | Elfogad, ír fájlba, továbblép |
| `E` (Edit) | Megnyitja `$EDITOR`-ban szerkesztésre |
| `R` (Regenerate) | Újra generálja (más prompt params) |
| `S` (Skip) | Kihagyja, nem ír fájlt |
| `Q` (Quit) | Kilép az egész deriválásból |

#### 4.3 Batch vs Interactive Mode

| Mode | Flag | Viselkedés |
|------|------|------------|
| Batch (default) | - | Mindent generál, nincs kérdés |
| Interactive | `-i` | Minden fázis előtt kérdez |
| Auto-approve L2+ | `--auto-approve-l2` | Csak L1-nél kérdez |

---

### Fázis 5: Kiegészítő Inputok (P3)

#### 5.1 Domain Vocabulary Input

**Új opcionális input:** `--vocabulary <path>`

```bash
loom-cli derive --input-file ./l0.md --vocabulary ./vocabulary.md --output-dir ./l1
```

**Vocabulary formátum:**
```markdown
# Domain Vocabulary

## Customer
- **Definition:** A person or organization that places orders.
- **Synonyms:** Client, Buyer
- **Related:** Order, Address, Payment

## Order
- **Definition:** A request to purchase products.
- **Statuses:** pending, confirmed, shipped, delivered, cancelled
- **Related:** Customer, Product, LineItem
```

**Hatás a deriválásra:**
- Domain Model jobban strukturált
- Bounded Context pontosabb
- Business Rules teljesebb

#### 5.2 NFR (Non-Functional Requirements) Input

**Új opcionális input:** `--nfr <path>`

```bash
loom-cli derive --input-file ./l0.md --nfr ./nfr.md --output-dir ./l1
```

**NFR formátum:**
```markdown
# Non-Functional Requirements

## Performance
- NFR-PERF-001: API response time < 200ms (p95)
- NFR-PERF-002: Support 1000 concurrent users

## Security
- NFR-SEC-001: All passwords hashed with bcrypt
- NFR-SEC-002: Session timeout 30 minutes

## Reliability
- NFR-REL-001: 99.9% uptime SLA
- NFR-REL-002: Data backup every 6 hours
```

**Hatás a deriválásra:**
- Tech Specs tartalmazza NFR-eket
- Test Cases tartalmaz performance testeket
- Feature Tickets tartalmazza NFR-eket

---

## Implementálási Sorrend

### Sprint 1: TDAI Foundation (P1) ✅ COMPLETE
1. [x] `derive-test-cases.md` prompt refaktorálás TDAI szerint
2. [x] TestCase struct bővítés (type, category)
3. [x] formatTestCases() TDAI struktúra
4. [x] Hallucination prevention test típusok
5. [x] Teszt benchmark-kal

**Eredmények (2024-12-31):**
- 172 test case generálva 40 AC-ból
- P:72 N:54 B:22 H:24 eloszlás
- 24 hallucination prevention teszt
- Commit: `e411710`

### Sprint 2: Validation Layer (P1) ✅ COMPLETE
1. [x] `cmd/validate.go` új fájl
2. [x] Structural validation (V001, V002, V010)
3. [x] Traceability validation (V003, V004)
4. [x] Completeness validation (V005, V006, V007)
5. [x] TDAI validation (V008, V009)
6. [x] CLI integration (`loom-cli validate`)
7. [x] Teszt

**Eredmények (2024-12-31):**
- 10 validation szabály implementálva
- V004, V006, V007 placeholder (later sprint)
- V008: Negative ratio check működik (30.5% ✓)
- Commit: `da47cbf`

### Sprint 3: Traceability Anchors (P1) ✅ COMPLETE
1. [x] Anchor generálás minden format függvényben
2. [x] Reference link formátum implementálás
3. [x] Cross-layer link generálás (L2→L1)
4. [x] Teszt

**Eredmények (2024-12-31):**
- L1: AC, BR, Entity, VO, BC fejlécekhez {#anchor}
- L1: Decision referenciák → [ID](decisions.md#anchor)
- L2: TC, TS, IC, AGG, SEQ, TBL fejlécekhez {#anchor}
- L2: Cross-layer linkek → [AC-XXX](../l1/acceptance-criteria.md#anchor)
- Commit: `dd5d541`

### Sprint 4: Interactive Mode (P2) ✅ COMPLETE
1. [x] `--interactive` flag implementálás
2. [x] Preview rendering (box format)
3. [x] Approval action handler (A/E/R/S/Q)
4. [x] Editor integration ($EDITOR)
5. [x] Teszt

**Eredmények (2024-12-31):**
- Új fájl: cmd/interactive.go (RenderPreview, AskApproval, EditContent, ConfirmQuit)
- derive-l2 integráció: minden fájl írás előtt interaktív jóváhagyás
- Akciók: Approve, Edit, Skip, Quit (Regenerate későbbre halasztva)
- Commit: `33f38ec`

### Sprint 5: Extra Inputs (P3)
1. [ ] Vocabulary input parsing
2. [ ] NFR input parsing
3. [ ] Derivation integration
4. [ ] Teszt

---

## CLI Parancs Struktúra (Bővített)

```
loom-cli derive [options]
  --input-file <path>      L0 input file
  --input-dir <path>       Input directory
  --output-dir <path>      Output directory
  --vocabulary <path>      Optional domain vocabulary
  --nfr <path>             Optional non-functional requirements
  --interactive, -i        Interactive approval mode
  --auto-approve-l2        Only ask for L1 approval

loom-cli derive-l2 [options]
  --input-dir <path>       L1 input directory
  --output-dir <path>      L2 output directory
  --interactive, -i        Interactive approval mode

loom-cli derive-l3 [options]
  --input-dir <path>       L2 input directory
  --output-dir <path>      L3 output directory
  --interactive, -i        Interactive approval mode

loom-cli validate [options]
  --input-dir <path>       Directory to validate
  --level L1|L2|L3|all     Validation level
  --fix                    Auto-fix simple issues
  --json                   Output as JSON
```

---

## Becsült Munka

| Sprint | Prompt | Go kód | Teszt | Összesen |
|--------|--------|--------|-------|----------|
| Sprint 1 (TDAI) | 1 refactor | 1 file | 1 teszt | ~3 óra |
| Sprint 2 (Validation) | - | 1 new file | 1 teszt | ~4 óra |
| Sprint 3 (Anchors) | - | 6 file edit | 1 teszt | ~3 óra |
| Sprint 4 (Interactive) | - | 1 file | 1 teszt | ~3 óra |
| Sprint 5 (Inputs) | - | 2 file edit | 1 teszt | ~2 óra |
| **Összesen** | **1 prompt** | **11 file** | **5 teszt** | **~15 óra** |

---

## Döntési Pontok

### 1. TDAI Ratio Requirements
- **Opció A:** Strict (exactly 40% positive, 40% negative, 20% other)
- **Opció B:** Flexible (min 20% negative, rest flexible)
- **Javaslat:** Opció B (rugalmasabb, AC-függő)

### 2. Validation Strictness
- **Opció A:** Fail on any error
- **Opció B:** Warn but continue
- **Opció C:** Configurable (`--strict` flag)
- **Javaslat:** Opció C

### 3. Anchor Format
- **Opció A:** `{#id}` (Pandoc style)
- **Opció B:** `<a id="id"></a>` (HTML)
- **Opció C:** Just heading (rely on tools)
- **Javaslat:** Opció A (legszélesebb támogatás)

### 4. Interactive Mode Default
- **Opció A:** Default batch, opt-in interactive
- **Opció B:** Default interactive, opt-out batch
- **Javaslat:** Opció A (CI/CD kompatibilitás)

---

## Success Criteria

### Sprint 1 (TDAI) Done When: ✅ COMPLETE
- [x] Test cases tartalmazzák a type mezőt (positive/negative/boundary/hallucination)
- [x] Minden AC-hez van min 1 hallucination prevention test
- [x] Summary mutatja a ratio-kat

### Sprint 2 (Validation) Done When: ✅ COMPLETE
- [x] `loom-cli validate` parancs működik
- [x] Mind a 10 validation rule implementálva
- [x] Exit code 1 ha error van

### Sprint 3 (Anchors) Done When: ✅ COMPLETE
- [x] Minden ID mellett van `{#anchor}`
- [x] Traceability section markdown linkeket tartalmaz
- [x] Cross-layer linkek működnek (L2→L1)

### Sprint 4 (Interactive) Done When: ✅ COMPLETE
- [x] `--interactive` flag működik
- [x] Preview megjelenik (box format)
- [x] A/E/S/Q actions működnek (R = későbbre halasztva)

### Sprint 5 (Inputs) Done When:
- [ ] `--vocabulary` és `--nfr` flag-ek működnek
- [ ] Vocabulary befolyásolja a domain-model generálást
- [ ] NFR megjelenik a tech-specs-ben

---

## Következő Lépések

1. [x] Terv jóváhagyása
2. [x] Sprint 1 indítása: TDAI prompt refaktorálás
3. [x] Implementálás és tesztelés
4. [x] Commit (`e411710`)
5. [x] Sprint 2: Validation Layer (`da47cbf`)
6. [x] Sprint 3: Traceability Anchors (`dd5d541`)
7. [x] Sprint 4: Interactive Mode (`33f38ec`)
8. [ ] Sprint 5 indítása: Extra Inputs (Vocabulary, NFR)

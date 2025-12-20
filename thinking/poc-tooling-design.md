---
date: 2025-12-19
author: Claude Sonnet 4.5 + Human collaboration
version: 2.0
status: draft
purpose: AI-PDS Proof of Concept tooling design specification
changelog:
  - v2.0 (2025-12-19): ARCHITECTURAL PIVOT - MCP Server as primary implementation (instead of standalone CLI)
  - v1.2 (2025-12-19): Added Test-Driven AI Development (TDAI) support
  - v1.1 (2025-12-19): Added bidirectional traceability support
  - v1.0 (2025-12-19): Initial version
---

# AI-PDS PoC Tooling - Tervezési dokumentum

---

## 🚨 CRITICAL ARCHITECTURAL UPDATE (v2.0)

**Major pivot:** After analyzing Claude Code's capabilities, we're shifting from a **standalone CLI** to an **MCP (Model Context Protocol) Server** as the primary implementation.

### Why this changes everything:

**Original plan (v1.x):**
```
Build standalone CLI tool → User runs commands → AI-PDS functionality
```

**New plan (v2.0):**
```
Build MCP Server → Integrate with Claude Code → Native AI-PDS functionality
```

### Key benefits:

✅ **80% less code to maintain** - No CLI framework, file I/O, diff viewer, approval workflow
✅ **Native Claude Code integration** - Tools, resources, prompts all first-class
✅ **Better UX** - Natural language instead of commands
✅ **Standardized protocol** - MCP is the future of AI tool integration
✅ **8-13 days instead of 20-28 days** development time

### What changes in this document:

1. **Architecture section:** MCP Server replaces standalone CLI as primary implementation
2. **Technology Stack:** Added `@modelcontextprotocol/sdk`
3. **Project Structure:** Restructured to `packages/loom-mcp-server/`
4. **Implementation Phases:** Updated to prioritize MCP server development
5. **Distribution:** npm package `@loom/mcp-server` instead of global CLI

### Detailed MCP Server Architecture:

See **[claude-code-as-platform.md](./claude-code-as-platform.md)** for comprehensive MCP Server design:
- 5 Core Tools: `loom_validate`, `loom_derive`, `loom_trace`, `loom_test_generate`, `loom_init`
- 4 Core Resources: `@loom:user-story://`, `@loom:ac://`, `@loom:entity://`, `@loom:test://`
- Full implementation details, examples, workflows

**This document now describes:**
- MCP Server as PRIMARY implementation ✅
- Standalone CLI as OPTIONAL wrapper around MCP server (for non-Claude Code users)

---

## 📋 Executive Summary

Ez a dokumentum az AI-PDS (AI-Ready Project Documentation System) első Proof of Concept (PoC) tooling-jának részletes tervezését tartalmazza.

**Cél:** Bebizonyítani, hogy az AI-generált, human-supervised dokumentációs workflow működőképes és értéket teremt.

**Scope:** Minimális működőképes CLI tool, amely képes:
1. Natural language input alapján strukturált AI-PDS dokumentumokat generálni
2. **Automatikusan traceability ID-kat és annotációkat generálni dokumentumokba és kódba**
3. **Comprehensive teszteket generálni REQUIREMENTS-FIRST megközelítéssel (TDAI)**
4. Humán jóváhagyást kérni a generált változásokra
5. Alapvető validációt végezni a dokumentumokon
6. **Traceability konzisztenciát ellenőrizni (docs ↔ code ↔ tests)**
7. **AI hallucináció detektálása automated tests-szel**

**Timeline:** 2-4 hét fejlesztés (1 fő, part-time)

**Success criteria:**
- Egy demo projekt (pl. TODO app) teljes dokumentációját képes generálni
- **Demo projekthez comprehensive test suite generálva (unit, integration, e2e)**
- A generált dokumentumok alapján egy másik AI képes működő kódot generálni
- **A generált kód átmegy az összes generált teszten**
- Az emberi erőfeszítés < 30%-a a hagyományos manuális dokumentálásnak
- **A generált dokumentumok, tesztek és kód között teljes traceability van**
- **Traceability validation 95%+ pass rate**
- **AI hallucináció detection rate ≥ 90% (tests catch AI mistakes)**

---

## 🎯 Követelmények

### Funkcionális követelmények

#### FR1: Natural Language Input Processing
**Priority:** P0 (blocker)

A tool képes természetes nyelvű bemenet alapján megérteni a felhasználói szándékot.

**Input példák:**
```bash
ai-pds generate "A rendszerben lesz egy User entitás email, name és role mezőkkel"

ai-pds generate "User lehet admin vagy regular, admin mindent láthat"

ai-pds generate "Hozzáadunk egy Task entitást, ami User-hez tartozik"
```

**Követelmények:**
- [ ] Támogatja a magyar és angol nyelvű inputot
- [ ] Képes többsoros inputot kezelni (interactive mode)
- [ ] Képes file-ból olvasni az inputot (batch mode)

#### FR2: Document Generation
**Priority:** P0 (blocker)

A tool képes strukturált markdown dokumentumokat generálni YAML frontmatter-rel.

**Minimum támogatott dokumentumok (PoC scope):**
1. `domain-modelling/domain-vocabulary.md` - Domain fogalmak szótára
2. `domain-modelling/domain-model.md` - Entitások, kapcsolatok
3. `requirements/user-stories.md` - User story-k
4. `requirements/acceptance-criteria.md` - Acceptance criteria-k
5. `architecture/decisions.md` - Architekturális döntések (mini ADR)

**Követelmények:**
- [ ] Generált fájlok valid markdown formátumúak
- [ ] YAML frontmatter tartalmazza a `status` mezőt (draft/to review/approved)
- [ ] Automatikus cross-linking a dokumentumok között
- [ ] Generáláskor meglévő fájlokat nem ír felül confirmation nélkül

#### FR3: Interactive Q&A Workflow
**Priority:** P1 (important)

Ha az AI-nak további információra van szüksége, kérdéseket tesz fel.

**Példa flow:**
```
User: "Hozzáadunk egy Task entitást"

AI: "Milyen mezői legyenek a Task-nak? (Javaslat: title, description, status, dueDate)"
User: "title, description, status, assignedTo"

AI: "Mi legyen a status lehetséges értéke? (Javaslat: TODO/IN_PROGRESS/DONE)"
User: "TODO, IN_PROGRESS, DONE, BLOCKED"

AI: "Az assignedTo egy User-re mutat?"
User: "igen"

AI: Generálok 3 fájlt: domain-model.md (+Task), domain-vocabulary.md (+új fogalmak),
    user-stories.md (+Task management story). Jóváhagyod? [y/n]
```

**Követelmények:**
- [ ] Kérdések egyértelműek, konkrétak
- [ ] Ajánlásokat ad, nem csak kérdez
- [ ] Maximálisan 3-5 kérdés egy iterációban (ne legyen túl hosszú)
- [ ] Lehetőség van "skip" válaszra → AI dönt

#### FR4: Human Approval Workflow
**Priority:** P0 (blocker)

Minden generálás előtt megmutatja, mi fog változni, és jóváhagyást kér.

**Diff view példa:**
```
┌─────────────────────────────────────────────────────────┐
│ AI-PDS Generation Preview                               │
├─────────────────────────────────────────────────────────┤
│ Input: "Add Task entity with title, description, status"│
│                                                          │
│ Files to be created/modified: 3                         │
│                                                          │
│ ✓ domain-modelling/domain-model.md       [CREATE]       │
│   + Task entity                                         │
│   + Fields: title (string), description (string),       │
│     status (enum: TODO|IN_PROGRESS|DONE)                │
│                                                          │
│ ✓ domain-modelling/domain-vocabulary.md  [UPDATE]       │
│   + Task: A unit of work assigned to a user            │
│   + Status: Current state of a task                     │
│                                                          │
│ ✓ requirements/user-stories.md           [UPDATE]       │
│   + As a user, I want to create tasks                   │
│   + As a user, I want to update task status             │
│                                                          │
│ [View Details] [Approve All] [Approve One-by-One] [Reject]
└─────────────────────────────────────────────────────────┘
```

**Követelmények:**
- [ ] Mutatja az összes érintett fájlt
- [ ] Jelzi, hogy CREATE vagy UPDATE
- [ ] Összefoglalót mutat az egyes fájlok változásairól
- [ ] "View Details" opció: teljes diff megtekintése
- [ ] Három jóváhagyási mód: All / One-by-One / Reject

#### FR5: Basic Validation
**Priority:** P1 (important)

Alapvető validációs ellenőrzések a generált dokumentumokon.

**Validációs szabályok:**
1. **YAML frontmatter validáció:**
   - Kötelező mezők: `status`
   - Status értékek: `draft | to review | approved | living`

2. **Markdown link validáció:**
   - Belső linkek (pl. `[domain model](../domain-modelling/domain-model.md)`) létező fájlokra mutatnak

3. **Konzisztencia ellenőrzés:**
   - Ha domain-vocabulary.md definiál egy fogalmat, akkor domain-model.md-ben is szerepel
   - Ha user-stories.md említ egy entitást, akkor domain-model.md-ben definiálva van

4. **Structure validation:**
   - A fájlok a megfelelő mappában vannak (domain-modelling/, requirements/, stb.)

**Követelmények:**
- [ ] Validáció fut generálás után, approval előtt
- [ ] Warning-okat mutat, de nem blokkolja a generálást
- [ ] Error-ok esetén blokkolja a fájl írást

#### FR6: File System Management
**Priority:** P0 (blocker)

Biztonságos fájlkezelés, atomi írás, backup.

**Követelmények:**
- [ ] Csak jóváhagyás után ír fájlokat
- [ ] Backup készítés meglévő fájlokról (`.backup/` folder timestamp-pel)
- [ ] Atomi írás (vagy mindegyik fájl frissül, vagy egyik sem)
- [ ] Git status ellenőrzés (working directory clean? commitolni kell-e először?)

#### FR7: Bidirectional Traceability Support
**Priority:** P0 (blocker)

**KRITIKUS:** A dokumentáció-kód szinkronizáció kulcsa!

A tool automatikusan generál és fenntart **kétirányú traceability linkeket** a dokumentumok és kód között.

**Traceability ID scheme:**
- User Story: `US-001`, `US-002`, ...
- Acceptance Criterion: `AC-001-1`, `AC-001-2`, ...
- Entity: `ENT-User`, `ENT-Task`, ...
- Entity Field: `ENT-User:email`, `ENT-Task:status`, ...

**Forward tracing (docs → code):**

Dokumentumokban anchor-ok és implementation referenciák:

```markdown
## US-001: User Registration {#us-001}

As a new user, I want to register...

**Acceptance Criteria:**
- [AC-001-1] Email must be valid format
- [AC-001-2] Password minimum 8 characters

**Implementation refs:**
- Code: `src/auth/registration.ts:registerUser()`
- Tests: `tests/auth/registration.test.ts`
- Status: ✓ Implemented (2024-12-19)
```

**Backward tracing (code → docs):**

Generált kódban traceability annotations:

```typescript
/**
 * User registration handler
 *
 * @traceability US-001 (requirements/user-stories.md#us-001)
 * @implements AC-001-1, AC-001-2
 * @domain-entity ENT-User (domain-modelling/domain-model.md#user-entity)
 */
export async function registerUser(email: string, password: string): Promise<User> {
  // @implements AC-001-1: Email validation
  if (!isValidEmail(email)) {
    throw new ValidationError('Invalid email format');
  }

  // @implements AC-001-2: Password length check
  if (password.length < 8) {
    throw new ValidationError('Password must be at least 8 characters');
  }

  // ...
}
```

**Követelmények (PoC scope):**
- [ ] **Auto-generate IDs:** AI automatikusan generál egyedi ID-kat (US-XXX, AC-XXX-X, ENT-XXX)
- [ ] **Anchor generation:** Markdown headings anchor-okat kapnak ({#us-001})
- [ ] **Code annotation:** Generált kód tartalmaz @traceability és @implements comment-eket
- [ ] **Traceability parser:** Parse docs és code, extract traceability links
- [ ] **Existence check:** Minden referált ID létezik (pl. US-001 létezik a docs-ban)
- [ ] **Implementation coverage:** Minden AC-hoz van @implements a kódban
- [ ] **CLI commands:**
  - `ai-pds trace parse` - Extract és megjelenít minden traceability linket
  - `ai-pds trace validate` - Ellenőrzi a traceability konzisztenciát

**Példa traceability validation output:**

```bash
ai-pds trace validate

Validating traceability...

✓ Existence check passed (45 IDs verified)
  - All US-* IDs found in requirements/user-stories.md
  - All ENT-* IDs found in domain-modelling/domain-model.md

✗ Implementation coverage check failed:
  - AC-001-3: No @implements found in codebase
  - AC-005-2: No @implements found in codebase

⚠ Orphaned code check found 3 warnings:
  - src/utils/helper.ts: No @traceability annotation

Summary: 2 errors, 3 warnings

Traceability map:
  US-001 (User Registration)
    ├─> AC-001-1 ──> src/auth/registration.ts:registerUser() ✓
    ├─> AC-001-2 ──> src/auth/registration.ts:registerUser() ✓
    └─> AC-001-3 ──> MISSING ✗

  ENT-User (Entity)
    └─> src/domain/entities/User.ts ✓
```

**Out of scope for PoC (future enhancements):**
- Semantic consistency check (LLM-powered)
- Domain model field type checking
- Bidirectional sync (code → docs update)
- Traceability graph viewer
- Impact analysis

**Rationale:**
> "Kétirányú szigorú traceability a dokumentumok és kód között. AI időről időre ellenőrzi a konzisztenciát, koherenciát." - User requirement

Ez NEM optional feature - ez az AI-PDS alapja! Nélküle a docs és code garantáltan eltér.

#### FR8: Test-Driven AI Development (TDAI) Support
**Priority:** P0 (blocker)

**KRITIKUS:** Az AI hallucináció problémájának megoldása!

A tool **tests-first megközelítést** követ: teszteket generál MIELŐTT a kódot, hogy a tesztek constraintként működjenek az AI számára.

**Probléma:**
> AI gyakran "kreatívan értelmez", kitalál dolgokat, implicit döntéseket hoz. Az AI hallucináció problémája nem oldódik meg dokumentációval.

**Megoldás:**
> Nagy számú automatikusan generált teszt (unit, integration, e2e) szűkíti a hallucináció mozgásterét és kiszűri a hallucinációkat.

**TDAI Workflow:**

```
Requirements → AI generates TESTS first → Human approves tests
            → AI generates CODE (constrained by tests)
            → Tests RUN
            → Hallucinations detected (test failures)
```

**Test Types & Distribution (Test Pyramid 70:20:10):**

```
         /\
        / E2E \          10% - Full user journeys (1-3 per user story)
       /--------\
      /Integration\     20% - Workflows (2-5 per story)
     /--------------\
    /   Unit Tests   \  70% - Per AC: 5-10 tests each
   /___________________\
```

**Per Acceptance Criterion:**
- 3+ Positive tests (happy path variations)
- 3+ Negative tests (invalid input, edge cases) **← HALLUCINATION DETECTION!**
- 2+ Boundary tests (min, max, off-by-one)
- 1+ "Should NOT" test (explicit negative behavior) **← CRITICAL!**

**Example - Hallucination Prevention:**

```markdown
<!-- Requirement -->
[AC-001-2] Password must be at least 8 characters
```

```typescript
// POSITIVE TEST
it('should accept 8+ character password', () => {
  expect(validatePassword('12345678')).toBe(true);
});

// BOUNDARY TEST
it('should reject 7 character password', () => {
  expect(validatePassword('1234567')).toBe(false);
});

// NEGATIVE TEST - Catches hallucination!
it('should accept lowercase-only password', () => {
  expect(validatePassword('onlylowercase')).toBe(true);
  // If AI adds uppercase requirement → TEST FAILS! Hallucination caught!
});

// NEGATIVE TEST - Catches hallucination!
it('should accept password without special characters', () => {
  expect(validatePassword('simple123')).toBe(true);
  // If AI adds special char requirement → TEST FAILS! Hallucination caught!
});
```

**If AI hallucinates and adds extra validation:**

```typescript
// AI HALLUCINATION:
function validatePassword(password) {
  if (password.length < 8) return false;
  if (!/[A-Z]/.test(password)) return false;  // ← NOT in requirements!
  return true;
}

// RESULT:
✗ should accept lowercase-only password  // TEST FAILS - hallucination detected!
```

**Követelmények (PoC scope):**

- [ ] **Test Plan Generation:** AI generates test plan from requirements (before generating tests)
- [ ] **Test Generation:** AI generates comprehensive test suite
  - [ ] Unit tests (70%): 5-10 per AC
  - [ ] Integration tests (20%): 2-5 per user story
  - [ ] E2E tests (10%): 1-3 per user story
- [ ] **Negative Tests Mandatory:** At least 20% of tests must be negative/"should NOT" tests
- [ ] **Test-Code Traceability:** Tests have `@implements AC-XXX` annotations
- [ ] **Human Test Review:** Test plan and tests reviewed before code generation
- [ ] **Code Generation Constrained:** AI generates code that MUST pass all tests
- [ ] **Hallucination Detection:** Failed tests analyzed for hallucinations
- [ ] **CLI Commands:**
  - `ai-pds generate test-plan --from-user-story US-001`
  - `ai-pds generate tests --from-user-story US-001`
  - `ai-pds generate code --from-tests` (generates code passing tests)
  - `ai-pds test run --detect-hallucinations`
  - `ai-pds test metrics` (test quality dashboard)

**Test Quality Metrics:**

```typescript
interface TestQualityMetrics {
  // Coverage
  requirementCoverage: number;       // % of ACs with tests
  codeCoverage: number;              // % of code covered
  branchCoverage: number;            // % of branches covered

  // Distribution (Test Pyramid)
  unitTestCount: number;
  integrationTestCount: number;
  e2eTestCount: number;
  testPyramidRatio: string;          // "70:20:10"

  // Quality
  negativeTestRatio: number;         // ≥20% target
  avgTestsPerAC: number;             // 5-10 target

  // Hallucination Detection
  hallucinationsCaught: number;      // Tests that caught AI mistakes
  hallucinationRate: number;         // % of generations with hallucinations
}
```

**Example Test Metrics Dashboard:**

```bash
ai-pds test metrics

📊 Test Quality Dashboard
─────────────────────────────────────
Coverage
  Requirement:  100% ████████████████ ✓
  Code:          92% █████████████░░░ ✓
  Branch:        87% ████████████░░░░ ✓

Test Pyramid
  Unit:        156 (72%) ✓
  Integration:  42 (19%) ✓
  E2E:          18 ( 8%) ✓
  Ratio: 72:19:8 (target: 70:20:10) ✓

Quality Indicators
  Negative tests:    28% ████████░░░░ ✓ (≥20%)
  Avg tests per AC:  8.3 ✓ (target: 5-10)

🛡️ Hallucination Detection
  Caught:            3 ⚠
  Rate:             12% (3/25 features)
  Last detection:   AC-005-2 (added uppercase requirement)
```

**Out of scope for PoC (future):**
- Property-based testing
- Mutation testing
- Performance/load tests
- Visual regression tests

**Rationale:**
> "A user storykból és a requirementekbúl nagy számú tesztet kell generálni (unit, integration, e2e), ezzel szűkítve a hallucinációk mozgásterét, illetve kiszűrve a hallucinációkat. Továbbá alapos manuális human QA biztosítja, hogy az implementálódjon aminek kell." - User requirement

**Key Insight:**
> **Tests are NOT just validation — they're CONSTRAINTS on AI behavior!**
>
> Negative tests especially: they tell AI what NOT to do.
>
> This is the hallucination antidote.

### Nem-funkcionális követelmények

#### NFR1: Performance
- Egy generálási iteráció max 30 másodperc (LLM API call idejével együtt)
- Max 5 LLM API call egy user input-ra

#### NFR2: Cost
- Egy tipikus generálási iteráció max $0.10 API költség
- Daily limit: max $5 (safety net)

#### NFR3: Usability
- CLI intuitív, érthető hibaüzenetek
- Progress indicator hosszú műveleteknél
- Colored output (success=green, warning=yellow, error=red)

#### NFR4: Extensibility
- Könnyen hozzáadhatók új dokumentumtípusok
- Moduláris architektúra (új LLM provider beépítése egyszerű)

---

## 🏗️ Architektúra

### High-Level Architecture (v2.0 - MCP Server First)

**PRIMARY: MCP Server Architecture**

```
┌─────────────────────────────────────────────────────────────┐
│                     Claude Code                             │
│  (User interaction via natural language)                    │
│  User: "Validate traceability for US-001"                   │
└─────────────────┬───────────────────────────────────────────┘
                  │ MCP Protocol (stdio)
                  ▼
┌─────────────────────────────────────────────────────────────┐
│                  @loom/mcp-server                           │
│  (Model Context Protocol Server)                            │
│                                                              │
│  ┌──────────────────────────────────────────────┐           │
│  │ MCP Tools (5 tools)                          │           │
│  │  - loom_validate                             │           │
│  │  - loom_derive                               │           │
│  │  - loom_trace                                │           │
│  │  - loom_test_generate (TDAI!)                │           │
│  │  - loom_init                                 │           │
│  └──────────────────────────────────────────────┘           │
│                                                              │
│  ┌──────────────────────────────────────────────┐           │
│  │ MCP Resources (4 resources)                  │           │
│  │  - @loom:user-story://US-XXX                 │           │
│  │  - @loom:ac://AC-XXX-X                       │           │
│  │  - @loom:entity://ENT-XXX                    │           │
│  │  - @loom:test://TC-XXX                       │           │
│  └──────────────────────────────────────────────┘           │
│                                                              │
│  ┌──────────────────────────────────────────────┐           │
│  │ Core Engine                                  │           │
│  │  - Validator                                 │           │
│  │  - Derivation Engine                         │           │
│  │  - Traceability Parser/Validator             │           │
│  │  - Test Generator (TDAI)                     │           │
│  │  - Test Executor & Hallucination Detector    │           │
│  └──────────────────────────────────────────────┘           │
└─────────────────┬───────────────────────────────────────────┘
                  │
        ┌─────────┴─────────────────────────┐
        ▼                                   ▼
┌──────────────────┐              ┌────────────────────────────┐
│   File System    │              │   External Services        │
│   Layer          │              │                            │
│ - Read/Write     │              │ - Git CLI                  │
│ - Markdown Parse │              │ - Test Frameworks          │
│ - YAML Parse     │              │   (Jest/Vitest)            │
└──────────────────┘              └────────────────────────────┘
```

**OPTIONAL: Standalone CLI (for non-Claude Code users)**

```
┌─────────────────────────────────────────────────────────────┐
│                    loom CLI (optional)                      │
│  $ loom validate --stage=traceability                       │
└─────────────────┬───────────────────────────────────────────┘
                  │ (wraps MCP server tools)
                  ▼
┌─────────────────────────────────────────────────────────────┐
│               @loom/mcp-server                              │
│  (Same MCP server, called via CLI wrapper)                  │
└─────────────────────────────────────────────────────────────┘
```

**Key architectural insights:**

✅ **MCP Server IS the implementation** - No separate CLI logic
✅ **Claude Code provides UI/UX** - No need to build approval workflow, diff viewer, etc.
✅ **Reusable** - Same MCP server works with Claude Code, standalone CLI, or any MCP client
✅ **80% less code** - File I/O, git integration, LLM API already handled by Claude Code

**Original CLI-based architecture (deprecated - for reference only):**

```
┌─────────────────────────────────────────────────────────────┐
│                         CLI Layer                           │
│  (User interaction, command parsing, output formatting)     │
│  Commands: generate, validate, trace, test, status, init    │
└─────────────────┬───────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────────┐
│                    Orchestrator Layer                       │
│  (Workflow coordination, state management)                  │
│  - GenerateOrchestrator                                     │
│  - TraceOrchestrator                                        │
│  - TestOrchestrator (NEW - TDAI!)                           │
└─────────────────┬───────────────────────────────────────────┘
                  │
        ┌─────────┴─────────────────────────┐
        ▼                                   ▼
┌──────────────────┐    ┌────────────────────────────┐
│   AI Agent       │    │   Validator Layer          │
│   Layer          │───>│                            │
│                  │    │ - YAML schema              │
│ - Intent         │    │ - Links                    │
│ - Doc Generator  │    │ - Consistency              │
│ - Test Generator │<───│ - Traceability             │
│   (NEW - TDAI!)  │    │ - Test Quality (NEW!)      │
│ - QA             │    └────────────────────────────┘
└────────┬─────────┘                   │
         │                             │
         └──────────┬──────────────────┘
                    ▼
       ┌────────────────────────────────┐
       │  Traceability Layer            │
       │                                │
       │ - TraceabilityParser           │
       │ - TraceabilityValidator        │
       │ - IDGenerator                  │
       └──────┬─────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│                Test Execution Layer (NEW - TDAI!)           │
│  (Test runner, hallucination detector, coverage reporter)   │
│  - TestRunner (Jest/Vitest integration)                     │
│  - HallucinationDetector                                    │
│  - CoverageReporter                                         │
└─────────────────┬───────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│                    File System Layer                        │
│  (File I/O, backup, atomic writes, git integration)        │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│                    External Services                        │
│  (LLM API: Claude/GPT, Git CLI, Test Frameworks)           │
└─────────────────────────────────────────────────────────────┘
```

### Detailed Component Design

#### Component 1: CLI Layer

**Responsibility:** User interaction, command parsing, output rendering

**Technology options:**
- TypeScript: `commander` + `inquirer` + `chalk`
- Python: `click` + `rich` + `prompt_toolkit`

**Commands:**
```bash
# Basic generation
ai-pds generate <natural-language-input>

# Interactive mode (Q&A)
ai-pds generate --interactive

# Batch mode (from file)
ai-pds generate --from-file input.txt

# View current state
ai-pds status

# Validate existing docs
ai-pds validate

# Initialize AI-PDS structure
ai-pds init

# Traceability commands
ai-pds trace parse      # Extract and display all traceability links
ai-pds trace validate   # Validate traceability consistency
ai-pds trace map        # Show traceability map (text-based)

# Test commands (NEW - TDAI!)
ai-pds generate test-plan --from-user-story US-001   # Generate test plan
ai-pds generate tests --from-user-story US-001       # Generate test suite
ai-pds generate code --from-tests                    # Generate code from tests
ai-pds test run                                       # Run all tests
ai-pds test run --detect-hallucinations               # Run + analyze failures
ai-pds test metrics                                   # Show test quality dashboard
```

**Example implementation (TypeScript):**
```typescript
// src/cli/commands/generate.ts
import { Command } from 'commander';
import { GenerateOrchestrator } from '../orchestrator';

export const generateCommand = new Command('generate')
  .description('Generate AI-PDS documentation from natural language')
  .argument('[input]', 'Natural language description')
  .option('-i, --interactive', 'Interactive Q&A mode')
  .option('-f, --from-file <path>', 'Read input from file')
  .action(async (input, options) => {
    const orchestrator = new GenerateOrchestrator();
    await orchestrator.run(input, options);
  });
```

#### Component 2: Orchestrator Layer

**Responsibility:** Координация AI agents, workflow management, state tracking

**Key classes:**

```typescript
class GenerateOrchestrator {
  private intentAgent: IntentAgent;
  private generatorAgents: Map<string, GeneratorAgent>;
  private qaAgent: QAAgent;
  private validator: Validator;
  private fileManager: FileManager;

  async run(userInput: string, options: GenerateOptions): Promise<void> {
    // 1. Parse intent
    const intent = await this.intentAgent.parse(userInput);

    // 2. Determine affected documents
    const affectedDocs = this.determineAffectedDocuments(intent);

    // 3. Check if Q&A needed
    const questions = this.qaAgent.determineQuestions(intent, affectedDocs);
    if (questions.length > 0) {
      const answers = await this.askQuestions(questions);
      intent.enrichWithAnswers(answers);
    }

    // 4. Generate documents
    const generatedDocs = await this.generateDocuments(intent, affectedDocs);

    // 5. Validate
    const validationResult = await this.validator.validate(generatedDocs);

    // 6. Show preview & get approval
    const approved = await this.showPreviewAndGetApproval(
      generatedDocs,
      validationResult
    );

    // 7. Write files if approved
    if (approved) {
      await this.fileManager.writeDocuments(generatedDocs);
      console.log('✓ Documents generated successfully');
    }
  }

  private determineAffectedDocuments(intent: Intent): DocumentType[] {
    // Logic: based on intent type, determine which docs to update
    // e.g., if intent is "add entity" → domain-model, domain-vocabulary, user-stories
  }
}
```

**State management:**
```typescript
interface GenerationState {
  currentIntent: Intent;
  affectedDocuments: DocumentType[];
  pendingQuestions: Question[];
  generatedDocuments: GeneratedDocument[];
  validationResults: ValidationResult[];
  approved: boolean;
}
```

#### Component 3: AI Agent Layer

**Responsibility:** LLM interaction, prompt engineering, parsing responses

**Sub-components:**

**3.1 Intent Agent**
- Парses natural language input
- Extracts entities, actions, relationships
- Returns structured Intent object

```typescript
interface Intent {
  type: 'add_entity' | 'update_entity' | 'add_relationship' | 'add_user_story' | 'add_architecture_decision';
  entities: Entity[];
  relationships: Relationship[];
  metadata: Record<string, any>;
}

class IntentAgent {
  constructor(private llm: LLMClient) {}

  async parse(userInput: string): Promise<Intent> {
    const prompt = this.buildIntentParsePrompt(userInput);
    const response = await this.llm.generateStructured(prompt, IntentSchema);
    return response;
  }

  private buildIntentParsePrompt(input: string): string {
    return `
You are an AI assistant parsing natural language input for a documentation system.

User input: "${input}"

Extract:
1. Intent type (add_entity, update_entity, add_relationship, etc.)
2. Entities mentioned (name, fields, types)
3. Relationships between entities
4. Any other relevant metadata

Return a structured JSON object matching the Intent schema.
    `.trim();
  }
}
```

**3.2 Generator Agents**
- Separate agent for each document type
- Generates markdown content based on Intent
- Maintains consistency with existing documents

```typescript
class DomainModelGeneratorAgent {
  constructor(private llm: LLMClient) {}

  async generate(intent: Intent, existingDoc?: string): Promise<string> {
    const prompt = this.buildGenerationPrompt(intent, existingDoc);
    const response = await this.llm.generate(prompt);
    return this.formatAsMarkdown(response);
  }

  private buildGenerationPrompt(intent: Intent, existing?: string): string {
    return `
You are generating a domain model document for an AI-PDS documentation system.

${existing ? `Existing document:\n${existing}\n\n` : ''}

Intent: ${JSON.stringify(intent, null, 2)}

Generate a markdown document that:
1. Includes YAML frontmatter with status: "draft"
2. Defines entities with their fields and types
3. Shows relationships between entities
4. Uses consistent terminology from the domain vocabulary

Format:
---
status: "draft"
---
# Domain Model

## Entities

### EntityName
- field1: type
- field2: type

## Relationships
- Entity1 --[relationship]--> Entity2
    `.trim();
  }
}
```

**3.3 QA Agent**
- Determines what questions to ask based on Intent
- Provides smart suggestions
- Validates answers

```typescript
class QAAgent {
  determineQuestions(intent: Intent, affectedDocs: DocumentType[]): Question[] {
    const questions: Question[] = [];

    if (intent.type === 'add_entity') {
      for (const entity of intent.entities) {
        // If fields not specified, ask
        if (!entity.fields || entity.fields.length === 0) {
          questions.push({
            type: 'text',
            prompt: `What fields should ${entity.name} have?`,
            suggestions: this.suggestFieldsForEntity(entity.name)
          });
        }

        // Ask about relationships
        questions.push({
          type: 'confirm',
          prompt: `Does ${entity.name} have relationships to other entities?`,
          default: true
        });
      }
    }

    return questions;
  }

  private suggestFieldsForEntity(entityName: string): string[] {
    // Simple heuristics or LLM-based suggestions
    const common = ['id', 'createdAt', 'updatedAt'];

    if (entityName === 'User') {
      return [...common, 'email', 'name', 'role'];
    }
    if (entityName === 'Task') {
      return [...common, 'title', 'description', 'status', 'dueDate'];
    }

    return common;
  }
}
```

#### Component 4: Validator Layer

**Responsibility:** Validate generated documents

```typescript
interface ValidationResult {
  isValid: boolean;
  errors: ValidationError[];
  warnings: ValidationWarning[];
}

interface ValidationError {
  file: string;
  line?: number;
  message: string;
  severity: 'error' | 'warning';
}

class Validator {
  private rules: ValidationRule[] = [
    new YAMLFrontmatterRule(),
    new MarkdownLinkRule(),
    new ConsistencyRule(),
    new StructureRule()
  ];

  async validate(docs: GeneratedDocument[]): Promise<ValidationResult> {
    const errors: ValidationError[] = [];
    const warnings: ValidationWarning[] = [];

    for (const rule of this.rules) {
      const result = await rule.validate(docs);
      errors.push(...result.errors);
      warnings.push(...result.warnings);
    }

    return {
      isValid: errors.length === 0,
      errors,
      warnings
    };
  }
}

class YAMLFrontmatterRule implements ValidationRule {
  validate(docs: GeneratedDocument[]): ValidationResult {
    const errors: ValidationError[] = [];

    for (const doc of docs) {
      const frontmatter = this.extractFrontmatter(doc.content);

      if (!frontmatter) {
        errors.push({
          file: doc.path,
          message: 'Missing YAML frontmatter',
          severity: 'error'
        });
        continue;
      }

      if (!frontmatter.status) {
        errors.push({
          file: doc.path,
          message: 'Missing "status" field in frontmatter',
          severity: 'error'
        });
      }

      const validStatuses = ['draft', 'to review', 'approved', 'living'];
      if (frontmatter.status && !validStatuses.includes(frontmatter.status)) {
        errors.push({
          file: doc.path,
          message: `Invalid status: "${frontmatter.status}". Must be one of: ${validStatuses.join(', ')}`,
          severity: 'error'
        });
      }
    }

    return { isValid: errors.length === 0, errors, warnings: [] };
  }
}
```

#### Component 5: File System Layer

**Responsibility:** Safe file operations, backup, git integration

```typescript
class FileManager {
  constructor(private rootPath: string) {}

  async writeDocuments(docs: GeneratedDocument[]): Promise<void> {
    // 1. Backup existing files
    await this.backupExistingFiles(docs);

    // 2. Atomic write (prepare all files first)
    const tempFiles = await this.prepareWrites(docs);

    try {
      // 3. Write all files
      await this.atomicWriteAll(tempFiles);

      // 4. Git add (if in git repo)
      if (await this.isGitRepo()) {
        await this.gitAdd(docs.map(d => d.path));
      }

    } catch (error) {
      // Rollback
      await this.rollback(tempFiles);
      throw error;
    }
  }

  private async backupExistingFiles(docs: GeneratedDocument[]): Promise<void> {
    const timestamp = new Date().toISOString().replace(/:/g, '-');
    const backupDir = path.join(this.rootPath, '.ai-pds-backup', timestamp);

    await fs.mkdir(backupDir, { recursive: true });

    for (const doc of docs) {
      const fullPath = path.join(this.rootPath, doc.path);
      if (await fs.pathExists(fullPath)) {
        const backupPath = path.join(backupDir, doc.path);
        await fs.copy(fullPath, backupPath);
      }
    }
  }

  private async atomicWriteAll(tempFiles: TempFile[]): Promise<void> {
    // Write to temp files first
    await Promise.all(
      tempFiles.map(tf => fs.writeFile(tf.tempPath, tf.content))
    );

    // Then rename (atomic operation)
    await Promise.all(
      tempFiles.map(tf => fs.rename(tf.tempPath, tf.finalPath))
    );
  }
}
```

---

## 🛠️ Technology Stack

### PRIMARY: MCP Server (TypeScript/Node.js) ✅

**Stack for @loom/mcp-server:**
```json
{
  "name": "@loom/mcp-server",
  "version": "1.0.0",
  "description": "Loom MCP Server for AI Development Orchestration Platform",
  "main": "dist/index.js",
  "bin": {
    "loom-mcp-server": "dist/index.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0",    // ← MCP Server SDK (CRITICAL!)
    "zod": "^3.22.0",                         // Schema validation
    "gray-matter": "^4.0.3",                  // YAML frontmatter parsing
    "marked": "^11.0.0",                      // Markdown parsing
    "simple-git": "^3.22.0"                   // Git integration
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "vitest": "^1.0.0",
    "@types/node": "^20.0.0"
  }
}
```

**Key differences from standalone CLI:**
- ✅ **Added:** `@modelcontextprotocol/sdk` - Core MCP server implementation
- ❌ **Removed:** `commander`, `inquirer`, `chalk` - No CLI framework needed (Claude Code provides UI)
- ❌ **Removed:** `@anthropic-ai/sdk` - No LLM API needed (Claude Code handles LLM)
- ✅ **80% fewer dependencies** - Focus on core logic, not infrastructure

### OPTIONAL: Standalone CLI Wrapper (for non-Claude Code users)

**Stack for standalone CLI (if needed later):**
```json
{
  "name": "@loom/cli",
  "version": "1.0.0",
  "dependencies": {
    "@loom/mcp-server": "^1.0.0",  // ← Wraps MCP server
    "commander": "^11.0.0",         // CLI framework
    "chalk": "^5.0.0"               // Terminal colors
  }
}
```

### Deprecated: Original Standalone CLI Stack (v1.x)

**For reference only (no longer recommended):**
```json
{
  "dependencies": {
    "@anthropic-ai/sdk": "^0.20.0",
    "commander": "^11.0.0",
    "inquirer": "^9.0.0",
    "chalk": "^5.0.0",
    "zod": "^3.22.0",
    "gray-matter": "^4.0.3",
    "marked": "^11.0.0",
    "simple-git": "^3.22.0"
  }
}
```

### Option B: Python

**Pros:**
- Excellent AI/LLM libraries (LangChain, LlamaIndex)
- Fast prototyping
- Great for data processing
- Easy distribution (pip, pipx)

**Cons:**
- Type hints are optional, less safe
- Dependency management can be tricky
- Async support less mature than JS

**Stack:**
```python
# requirements.txt
anthropic>=0.18.0
click>=8.1.0
rich>=13.7.0
pydantic>=2.5.0
python-frontmatter>=1.0.0
gitpython>=3.1.0
langchain>=0.1.0
```

### Option C: Go

**Pros:**
- Single binary distribution (easy deployment)
- Fast execution
- Good concurrency model
- No runtime dependencies

**Cons:**
- Fewer AI/LLM libraries
- More verbose code
- Smaller ecosystem for CLI tools

**Recommendation:** **TypeScript/Node.js** for PoC
- Best balance of productivity, ecosystem, and type safety
- Can be rewritten in Go later if performance becomes an issue

---

## 📁 Project Structure

### PRIMARY: MCP Server Structure ✅

**Recommended structure (v2.0 - MCP Server):**

```
packages/loom-mcp-server/
├── package.json                # @loom/mcp-server
├── tsconfig.json
├── README.md
├── .env.example                # Configuration template
│
├── src/
│   ├── index.ts                # MCP server entry point
│   │
│   ├── server.ts               # MCP Server initialization
│   │                           # - Register tools, resources, prompts
│   │                           # - Set up request handlers
│   │
│   ├── tools/                  # MCP Tools (5 tools)
│   │   ├── validate.ts         # loom_validate tool
│   │   ├── derive.ts           # loom_derive tool
│   │   ├── trace.ts            # loom_trace tool
│   │   ├── testGenerate.ts     # loom_test_generate tool
│   │   └── init.ts             # loom_init tool
│   │
│   ├── resources/              # MCP Resources (4 resources)
│   │   ├── userStory.ts        # @loom:user-story:// handler
│   │   ├── entity.ts           # @loom:entity:// handler
│   │   ├── ac.ts               # @loom:ac:// handler
│   │   └── testCase.ts         # @loom:test:// handler
│   │
│   ├── core/                   # Core business logic
│   │   ├── validator/
│   │   │   ├── Validator.ts
│   │   │   ├── rules/
│   │   │   │   ├── YAMLFrontmatterRule.ts
│   │   │   │   ├── MarkdownLinkRule.ts
│   │   │   │   ├── ConsistencyRule.ts
│   │   │   │   ├── TraceabilityRule.ts
│   │   │   │   └── TestQualityRule.ts
│   │   │   └── schemas/
│   │   │       └── frontmatter-schema.ts
│   │   │
│   │   ├── derivation/
│   │   │   ├── DerivationEngine.ts
│   │   │   ├── L0toL1.ts       # User stories → Domain model
│   │   │   ├── L1toL2.ts       # Domain model → API contracts
│   │   │   └── L2toL3.ts       # API contracts → Test cases
│   │   │
│   │   ├── traceability/
│   │   │   ├── TraceabilityParser.ts
│   │   │   ├── TraceabilityValidator.ts
│   │   │   ├── IDGenerator.ts
│   │   │   ├── TraceabilityMap.ts
│   │   │   └── checks/
│   │   │       ├── ExistenceCheck.ts
│   │   │       ├── CoverageCheck.ts
│   │   │       └── OrphanedCodeCheck.ts
│   │   │
│   │   └── testExecution/      # TDAI features
│   │       ├── TestRunner.ts
│   │       ├── HallucinationDetector.ts
│   │       ├── CoverageReporter.ts
│   │       ├── TestMetrics.ts
│   │       └── TestPyramidValidator.ts
│   │
│   └── utils/
│       ├── markdown.ts         # Markdown parsing
│       ├── yaml.ts             # YAML frontmatter parsing
│       ├── fileSystem.ts       # File I/O utilities
│       └── logger.ts           # Logging
│
├── tests/
│   ├── unit/
│   │   ├── tools/
│   │   ├── core/
│   │   └── utils/
│   ├── integration/
│   │   └── mcp-protocol.test.ts
│   └── fixtures/
│
└── examples/
    └── .mcp.json               # Example MCP configuration
```

### OPTIONAL: Standalone CLI Wrapper Structure

**For non-Claude Code users (optional):**

```
packages/loom-cli/
├── package.json                # @loom/cli (wraps @loom/mcp-server)
├── src/
│   ├── index.ts                # CLI entry point
│   ├── commands/
│   │   ├── validate.ts         # Calls loom_validate via MCP
│   │   ├── derive.ts           # Calls loom_derive via MCP
│   │   └── trace.ts            # Calls loom_trace via MCP
│   └── mcpClient.ts            # MCP client wrapper
└── README.md
```

### Deprecated: Original CLI Structure (v1.x)

**For reference only (no longer recommended):**

```
ai-pds-cli/
├── package.json
├── tsconfig.json
├── README.md
├── .env.example                 # API keys template
│
├── src/
│   ├── cli/
│   │   ├── index.ts            # CLI entry point
│   │   ├── commands/
│   │   │   ├── generate.ts     # generate command
│   │   │   ├── validate.ts     # validate command
│   │   │   ├── init.ts         # init command
│   │   │   ├── status.ts       # status command
│   │   │   └── trace/          # traceability commands
│   │   └── ui/
│   │       ├── spinner.ts      # Loading indicators
│   │       ├── diff-viewer.ts  # Diff preview UI
│   │       ├── approval.ts     # Approval prompt UI
│   │       └── trace-map-viewer.ts  # Traceability map UI (NEW!)
│   │
│   ├── orchestrator/
│   │   ├── GenerateOrchestrator.ts
│   │   ├── TraceOrchestrator.ts     # Traceability orchestrator (NEW!)
│   │   ├── Intent.ts           # Intent interface & types
│   │   └── GenerationState.ts
│   │
│   ├── agents/
│   │   ├── IntentAgent.ts
│   │   ├── QAAgent.ts
│   │   └── generators/
│   │       ├── BaseGenerator.ts
│   │       ├── DomainModelGenerator.ts
│   │       ├── DomainVocabularyGenerator.ts
│   │       ├── UserStoriesGenerator.ts
│   │       ├── AcceptanceCriteriaGenerator.ts
│   │       ├── ArchitectureDecisionsGenerator.ts
│   │       └── test/                          # Test generators (NEW - TDAI!)
│   │           ├── TestPlanGenerator.ts       # Generate test plan
│   │           ├── UnitTestGenerator.ts       # Generate unit tests
│   │           ├── IntegrationTestGenerator.ts # Generate integration tests
│   │           └── E2ETestGenerator.ts        # Generate E2E tests
│   │
│   ├── validators/
│   │   ├── Validator.ts
│   │   ├── rules/
│   │   │   ├── YAMLFrontmatterRule.ts
│   │   │   ├── MarkdownLinkRule.ts
│   │   │   ├── ConsistencyRule.ts
│   │   │   ├── StructureRule.ts
│   │   │   └── TraceabilityRule.ts    # Traceability validation (NEW!)
│   │   └── schemas/
│   │       └── frontmatter-schema.ts  # Zod schemas
│   │
│   ├── traceability/                  # Traceability layer
│   │   ├── TraceabilityParser.ts      # Parse @traceability annotations
│   │   ├── TraceabilityValidator.ts   # Validate traceability consistency
│   │   ├── IDGenerator.ts             # Generate unique IDs (US-XXX, etc.)
│   │   ├── TraceabilityLink.ts        # TraceabilityLink interface
│   │   ├── TraceabilityMap.ts         # Build and query traceability graph
│   │   └── checks/                    # Specific validation checks
│   │       ├── ExistenceCheck.ts      # Check IDs exist
│   │       ├── CoverageCheck.ts       # Check implementation coverage
│   │       └── OrphanedCodeCheck.ts   # Check for code without traceability
│   │
│   ├── test-execution/               # Test execution layer (NEW - TDAI!)
│   │   ├── TestRunner.ts              # Run tests (Jest/Vitest wrapper)
│   │   ├── HallucinationDetector.ts   # Analyze test failures for hallucinations
│   │   ├── CoverageReporter.ts        # Test coverage reporting
│   │   ├── TestMetrics.ts             # Test quality metrics
│   │   └── TestPyramidValidator.ts    # Validate 70:20:10 ratio
│   │
│   ├── file-system/
│   │   ├── FileManager.ts
│   │   ├── GitIntegration.ts
│   │   └── BackupManager.ts
│   │
│   ├── llm/
│   │   ├── LLMClient.ts        # Abstract LLM client
│   │   ├── ClaudeClient.ts     # Anthropic Claude implementation
│   │   └── OpenAIClient.ts     # OpenAI GPT implementation
│   │
│   ├── config/
│   │   ├── Config.ts           # Configuration management
│   │   └── prompts/            # Prompt templates
│   │       ├── intent-parse.ts
│   │       ├── domain-model.ts
│   │       ├── user-stories.ts
│   │       └── ...
│   │
│   └── utils/
│       ├── markdown.ts         # Markdown parsing utilities
│       ├── yaml.ts             # YAML frontmatter utilities
│       └── logger.ts           # Logging utilities
│
├── tests/
│   ├── unit/
│   │   ├── agents/
│   │   ├── validators/
│   │   └── ...
│   ├── integration/
│   │   └── end-to-end.test.ts
│   └── fixtures/
│       ├── sample-inputs.txt
│       └── expected-outputs/
│
└── examples/
    └── todo-app/               # Demo project
        ├── README.md
        └── ai-pds/
            ├── domain-modelling/
            ├── requirements/
            └── architecture/
```

---

## 🔌 API Design

### CLI Commands

#### `ai-pds init`

Initialize AI-PDS structure in current directory.

```bash
ai-pds init [--template <name>]

Options:
  --template <name>   Use template (minimal/standard/full) [default: minimal]
  --force             Overwrite existing structure

Examples:
  ai-pds init
  ai-pds init --template standard
```

**Behavior:**
- Creates folder structure (domain-modelling/, requirements/, architecture/)
- Creates `.ai-pds.config.json` configuration file
- Initializes git if not already a git repo

#### `ai-pds generate`

Generate documentation from natural language.

```bash
ai-pds generate [input] [options]

Arguments:
  input                Natural language description

Options:
  -i, --interactive    Interactive Q&A mode
  -f, --from-file      Read input from file
  --dry-run            Show what would be generated without writing
  --auto-approve       Skip approval prompt (dangerous!)

Examples:
  ai-pds generate "Add User entity with email, name, role"
  ai-pds generate --interactive
  ai-pds generate --from-file requirements.txt
  ai-pds generate "Add Task entity" --dry-run
```

#### `ai-pds validate`

Validate existing AI-PDS documents.

```bash
ai-pds validate [path] [options]

Arguments:
  path                 Path to validate [default: .]

Options:
  --fix                Auto-fix issues where possible
  --strict             Fail on warnings, not just errors

Examples:
  ai-pds validate
  ai-pds validate domain-modelling/
  ai-pds validate --fix
```

#### `ai-pds status`

Show current state of AI-PDS documentation.

```bash
ai-pds status

Examples:
  ai-pds status
```

**Output:**
```
AI-PDS Status
─────────────────────────────────────────────

Documents: 12 total
  ✓ 5 approved
  ⚠ 3 to review
  📝 4 draft

Coverage:
  ✓ Domain Modelling (3/3 files)
  ⚠ Requirements (2/3 files - missing acceptance-criteria.md)
  ✓ Architecture (1/1 files)

Validation: 2 warnings
  ⚠ domain-model.md: Entity "Task" mentioned in user-stories.md but not defined
  ⚠ user-stories.md: Missing acceptance criteria for story #3
```

### Configuration File

`.ai-pds.config.json`:

```json
{
  "version": "1.0",
  "structure": "minimal",
  "llm": {
    "provider": "anthropic",
    "model": "claude-3-5-sonnet-20241022",
    "maxTokens": 4096,
    "temperature": 0.7
  },
  "generation": {
    "autoApprove": false,
    "interactive": true,
    "language": "en"
  },
  "validation": {
    "strict": false,
    "autoFix": false,
    "rules": {
      "yaml-frontmatter": "error",
      "markdown-links": "warning",
      "consistency": "warning"
    }
  },
  "git": {
    "autoCommit": false,
    "commitMessagePrefix": "[AI-PDS]"
  }
}
```

### Environment Variables

`.env`:

```bash
# LLM Provider API Keys
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...

# Configuration overrides
AI_PDS_LLM_PROVIDER=anthropic
AI_PDS_MODEL=claude-3-5-sonnet-20241022
AI_PDS_MAX_TOKENS=4096
AI_PDS_TEMPERATURE=0.7

# Cost management
AI_PDS_DAILY_COST_LIMIT=5.00  # USD
AI_PDS_WARN_COST_THRESHOLD=2.50  # USD

# Logging
AI_PDS_LOG_LEVEL=info  # debug | info | warn | error
AI_PDS_LOG_FILE=.ai-pds.log
```

---

## 🎬 User Workflows & Examples

### Workflow 1: Fresh Project Setup

```bash
# 1. Initialize AI-PDS in project folder
cd my-new-project
ai-pds init

# 2. Describe the system in natural language
ai-pds generate --interactive

# Interactive prompts:
AI: What kind of system are you building?
User: A task management system for teams

AI: What are the main entities in your domain?
User: User, Team, Project, Task

AI: Tell me about the User entity. What fields should it have?
User: email, name, role (admin or member), teamId

AI: And the Task entity?
User: title, description, status (TODO/IN_PROGRESS/DONE), assignedTo (User), projectId, dueDate

AI: Great! I'll generate:
    - domain-modelling/domain-vocabulary.md (4 new terms)
    - domain-modelling/domain-model.md (4 entities)
    - requirements/user-stories.md (8 stories)
    - architecture/decisions.md (initial stack decision)

    Preview changes? [y/n]

User: y

# ... shows diff preview ...

AI: Approve? [y/n]
User: y

✓ Generated 4 files
✓ Validation passed (0 errors, 1 warning)
⚠ Warning: Consider adding acceptance criteria for user stories

Next steps:
  - Review generated docs: ai-pds status
  - Add acceptance criteria: ai-pds generate "Add acceptance criteria for user stories"
  - Validate: ai-pds validate
```

### Workflow 2: Adding a New Feature

```bash
# Existing project, adding a new feature

ai-pds generate "We need task comments. Users can add comments to tasks. Comments have text and timestamp."

# AI analyzes existing docs, determines:
# - New entity: Comment (needs to be added to domain model)
# - Relationship: Comment belongs to Task, Comment belongs to User
# - New user story: commenting on tasks
# - Architecture impact: need to decide on comment storage

AI: I see you're adding a new feature. Let me ask a few questions:

    1. Should comments be editable after creation? [y/n]
User: y

AI: 2. Should comments support mentions (e.g., @username)? [y/n]
User: n

AI: 3. Should there be a limit on comment length?
    Suggestions: 500 chars / 1000 chars / unlimited
User: 1000 chars

AI: Perfect! I'll update:
    - domain-modelling/domain-model.md (+Comment entity, +relationships)
    - domain-modelling/domain-vocabulary.md (+Comment term)
    - requirements/user-stories.md (+commenting user story)
    - architecture/decisions.md (+ADR: Comment storage approach)

    Preview? [y/n]

# ... (same approval flow as before)
```

### Workflow 3: Refining Existing Docs

```bash
ai-pds generate "The Task entity should also have a priority field (low/medium/high)"

AI: I see Task entity already exists. I'll update:
    - domain-modelling/domain-model.md (add priority field to Task)
    - domain-modelling/domain-vocabulary.md (add priority term)
    - requirements/user-stories.md (add priority management story)

    This will modify existing files. Show diff? [y/n]
User: y

# Diff view:
─────────────────────────────────────────────────────────
domain-modelling/domain-model.md
─────────────────────────────────────────────────────────
  ### Task
  - title: string
  - description: string
  - status: enum(TODO, IN_PROGRESS, DONE)
+ - priority: enum(LOW, MEDIUM, HIGH)
  - assignedTo: User
  - projectId: string
  - dueDate: Date
─────────────────────────────────────────────────────────

Approve this change? [y/n]
```

### Workflow 4: Validation & Fixing Issues

```bash
# After manual edits or to check consistency
ai-pds validate

Validating AI-PDS documents...
─────────────────────────────────────────────────────────
✓ YAML frontmatter validation passed
✓ Markdown structure validation passed
⚠ Consistency validation found issues:

  domain-modelling/domain-model.md:12
    Entity "Comment" is not defined in domain-vocabulary.md

  requirements/user-stories.md:45
    Story mentions "notifications" but no Notification entity exists

✓ Link validation passed

Summary: 0 errors, 2 warnings

Fix issues automatically? [y/n]
```

---

## 🧪 Testing Strategy

### Unit Tests

Test individual components in isolation.

```typescript
// tests/unit/agents/IntentAgent.test.ts
describe('IntentAgent', () => {
  it('should parse "add entity" intent', async () => {
    const agent = new IntentAgent(mockLLM);
    const intent = await agent.parse('Add User entity with email and name');

    expect(intent.type).toBe('add_entity');
    expect(intent.entities).toHaveLength(1);
    expect(intent.entities[0].name).toBe('User');
    expect(intent.entities[0].fields).toContain('email');
    expect(intent.entities[0].fields).toContain('name');
  });

  it('should parse relationship intent', async () => {
    const agent = new IntentAgent(mockLLM);
    const intent = await agent.parse('Task belongs to User');

    expect(intent.type).toBe('add_relationship');
    expect(intent.relationships).toHaveLength(1);
    expect(intent.relationships[0].from).toBe('Task');
    expect(intent.relationships[0].to).toBe('User');
  });
});
```

### Integration Tests

Test component interactions.

```typescript
// tests/integration/generate-workflow.test.ts
describe('Generate Workflow', () => {
  it('should generate domain model from scratch', async () => {
    const orchestrator = new GenerateOrchestrator({
      llm: realLLM,  // Use real LLM API
      autoApprove: true,
      outputDir: tempDir
    });

    await orchestrator.run('Add User entity with email, name, role');

    // Check files were created
    expect(fs.existsSync(`${tempDir}/domain-modelling/domain-model.md`)).toBe(true);
    expect(fs.existsSync(`${tempDir}/domain-modelling/domain-vocabulary.md`)).toBe(true);

    // Check content
    const domainModel = fs.readFileSync(`${tempDir}/domain-modelling/domain-model.md`, 'utf-8');
    expect(domainModel).toContain('### User');
    expect(domainModel).toContain('email');
    expect(domainModel).toContain('name');
    expect(domainModel).toContain('role');
  });
});
```

### End-to-End Tests

Test full CLI usage.

```typescript
// tests/e2e/cli.test.ts
describe('CLI E2E', () => {
  it('should initialize, generate, and validate', async () => {
    const projectDir = createTempDir();

    // Initialize
    await execCLI(`ai-pds init`, { cwd: projectDir });
    expect(fs.existsSync(`${projectDir}/.ai-pds.config.json`)).toBe(true);

    // Generate
    await execCLI(
      `ai-pds generate "Add User entity with email and name" --auto-approve`,
      { cwd: projectDir }
    );

    // Validate
    const { stdout } = await execCLI(`ai-pds validate`, { cwd: projectDir });
    expect(stdout).toContain('✓');
    expect(stdout).not.toContain('error');
  });
});
```

### Snapshot Testing

Test LLM output consistency.

```typescript
// tests/snapshots/generation.test.ts
describe('Generation Snapshots', () => {
  it('should generate consistent domain model', async () => {
    const generator = new DomainModelGenerator(llm);
    const result = await generator.generate({
      type: 'add_entity',
      entities: [{ name: 'User', fields: ['email', 'name'] }]
    });

    // Snapshot test - first run saves, subsequent runs compare
    expect(result).toMatchSnapshot();
  });
});
```

---

## 🚧 Implementation Phases

### Phase 1: Core Infrastructure (Week 1)

**Goal:** Basic CLI skeleton, LLM integration, file I/O

**Tasks:**
- [ ] Set up TypeScript project structure
- [ ] Implement CLI commands (init, generate, validate, status)
- [ ] Implement LLMClient abstraction + ClaudeClient implementation
- [ ] Implement FileManager with backup functionality
- [ ] Implement basic Config management
- [ ] Write unit tests for core components

**Deliverable:** CLI runs, can call Claude API, can write files safely

### Phase 2: Intent & Generation with Traceability (Week 2)

**Goal:** Natural language → structured docs **with traceability IDs**

**Tasks:**
- [ ] Implement IntentAgent with prompt engineering
- [ ] **Implement IDGenerator (US-XXX, AC-XXX-X, ENT-XXX)**
- [ ] Implement 5 generator agents (domain model, vocabulary, user stories, criteria, architecture)
  - [ ] **Each generator auto-generates IDs and anchors**
  - [ ] **Generators add "Implementation refs" sections**
- [ ] Implement prompt templates for each generator
  - [ ] **Update prompts to include traceability ID generation instructions**
- [ ] Implement diff preview UI
- [ ] Implement approval workflow
- [ ] Write integration tests for generation flow

**Deliverable:** Can generate documents from natural language, preview, and approve **with traceability IDs and anchors**

### Phase 3: Q&A, Validation, Traceability & TDAI (Week 3)

**Goal:** Interactive workflow, quality assurance, **traceability validation**, **Test-Driven AI Development**

**Tasks:**
- [ ] Implement QAAgent (question generation logic)
- [ ] Implement interactive CLI prompts
- [ ] Implement Validator with 6 validation rules (added TraceabilityRule + TestQualityRule)
- [ ] **Implement TraceabilityParser**
  - [ ] **Parse markdown files for IDs and anchors**
  - [ ] **Parse code files for @traceability annotations**
  - [ ] **Build TraceabilityMap (graph of links)**
- [ ] **Implement TraceabilityValidator**
  - [ ] **ExistenceCheck: all referenced IDs exist**
  - [ ] **CoverageCheck: all ACs have implementations**
  - [ ] **OrphanedCodeCheck: code has traceability annotations**
- [ ] **Implement trace CLI commands**
  - [ ] **`ai-pds trace parse`**
  - [ ] **`ai-pds trace validate`**
  - [ ] **`ai-pds trace map`**
- [ ] **Implement TDAI - Test Generation (NEW!)**
  - [ ] **TestPlanGenerator: Generate test plan from requirements**
  - [ ] **UnitTestGenerator: Generate unit tests (70%)**
  - [ ] **IntegrationTestGenerator: Generate integration tests (20%)**
  - [ ] **E2ETestGenerator: Generate E2E tests (10%)**
  - [ ] **NegativeTestEnforcer: Ensure ≥20% negative tests**
- [ ] **Implement Test Execution Layer (NEW!)**
  - [ ] **TestRunner: Run Jest/Vitest tests**
  - [ ] **HallucinationDetector: Analyze failures for hallucinations**
  - [ ] **TestMetrics: Calculate test quality metrics**
  - [ ] **TestPyramidValidator: Validate 70:20:10 ratio**
- [ ] **Implement test CLI commands (NEW!)**
  - [ ] **`ai-pds generate test-plan`**
  - [ ] **`ai-pds generate tests`**
  - [ ] **`ai-pds test run --detect-hallucinations`**
  - [ ] **`ai-pds test metrics`**
- [ ] Implement validation error/warning display
- [ ] Write tests for Q&A, validation, and test generation

**Deliverable:** Interactive Q&A works, validation catches issues, **traceability validation works**, **test generation works**, **hallucination detection functional**

### Phase 4: Polish, Testing & E2E TDAI + Traceability (Week 4)

**Goal:** Production-ready PoC **with full TDAI + traceability workflow**

**Tasks:**
- [ ] Implement git integration (auto-add, commit message suggestions)
- [ ] Implement cost tracking and limits
- [ ] Improve error handling and user feedback
- [ ] **Complete TDAI workflow (NEW!):**
  - [ ] **Demo: generate requirements → generate tests → generate code → run tests**
  - [ ] **Verify tests catch hallucinations (inject known hallucination → test fails)**
  - [ ] **Verify test pyramid ratio (70:20:10)**
  - [ ] **Verify negative test enforcement (≥20%)**
  - [ ] **Test metrics dashboard renders correctly**
- [ ] **Complete traceability workflow:**
  - [ ] **Demo: docs ↔ tests ↔ code all linked**
  - [ ] **Test all traceability checks pass**
  - [ ] **Verify traceability map includes tests**
- [ ] Write comprehensive documentation (README, examples)
  - [ ] **Document TDAI features (test-first workflow)**
  - [ ] **Document traceability features**
  - [ ] **Add TDAI + traceability examples to README**
- [ ] Create demo project (TODO app)
  - [ ] **Demo project fully documented**
  - [ ] **Demo project has comprehensive test suite (unit, integration, e2e)**
  - [ ] **All demo tests pass**
  - [ ] **Demo includes full traceability (docs ↔ tests ↔ code)**
  - [ ] **Run `ai-pds trace validate` → 0 errors**
  - [ ] **Run `ai-pds test metrics` → all targets met**
- [ ] End-to-end testing with demo project
- [ ] Performance optimization

**Deliverable:** Fully functional PoC, ready for user testing, **with working TDAI + bidirectional traceability**, **hallucination detection proven**

---

## 📊 Success Metrics

### Quantitative Metrics

**Development metrics:**
- [ ] PoC completed in ≤ 4 weeks
- [ ] Code coverage ≥ 70%
- [ ] All 5 document types supported

**Performance metrics:**
- [ ] Average generation time < 30 seconds
- [ ] Average API cost per generation < $0.10
- [ ] Validation runs in < 5 seconds
- [ ] **Traceability parsing < 2 seconds for 50 documents**
- [ ] **Traceability validation < 3 seconds for 50 links**
- [ ] **Test generation < 45 seconds for typical user story (NEW - TDAI!)**
- [ ] **Test execution < 5 seconds for 150 tests (NEW - TDAI!)**

**Quality metrics:**
- [ ] Generated docs pass validation 95%+ of the time
- [ ] Human approval rate ≥ 80% (users approve without modifications)
- [ ] **Traceability validation pass rate ≥ 95%**
- [ ] **100% of generated docs have unique IDs**
- [ ] **100% of generated code has @traceability annotations**
- [ ] **Test pyramid ratio within 70:20:10 ± 5% (NEW - TDAI!)**
- [ ] **Negative test ratio ≥ 20% (NEW - TDAI!)**
- [ ] **Hallucination detection rate ≥ 90% (NEW - TDAI!)**
- [ ] **Code coverage ≥ 85% (NEW - TDAI!)**
- [ ] **Requirement coverage = 100% (every AC has tests) (NEW - TDAI!)**

### Qualitative Metrics

**User experience:**
- [ ] CLI is intuitive (tested with 3+ users)
- [ ] Error messages are clear and actionable
- [ ] Diff preview is easy to understand

**Documentation quality:**
- [ ] Generated docs are consistent with AI-PDS spec
- [ ] Cross-references are correct
- [ ] Terminology is consistent across documents

**Proof of value:**
- [ ] Demo project (TODO app) fully documented using PoC
- [ ] **Demo project has comprehensive test suite (unit, integration, e2e) (NEW - TDAI!)**
- [ ] **Demo project tests all pass (NEW - TDAI!)**
- [ ] **Demo project has complete traceability (docs ↔ tests ↔ code)**
- [ ] Code can be generated from the docs by another AI (GPT/Claude)
- [ ] **Generated code includes @traceability annotations**
- [ ] **Generated tests include @implements annotations (NEW - TDAI!)**
- [ ] Time to document is < 30% of manual approach
- [ ] **`ai-pds trace validate` passes on demo project with 0 errors**
- [ ] **`ai-pds test metrics` shows all targets met (NEW - TDAI!)**
- [ ] **Hallucination injection test: AI hallucinates → test fails → caught! (NEW - TDAI!)**

---

## ⚠️ Risks & Mitigation

### Risk 1: LLM Output Quality

**Risk:** AI generates poor quality or inconsistent documentation

**Impact:** High - core value proposition fails

**Mitigation:**
- Extensive prompt engineering and testing
- Use structured output (JSON mode) where possible
- Implement strong validation rules
- Human approval step catches issues
- Iterative improvement based on user feedback

### Risk 2: API Costs

**Risk:** LLM API costs exceed budget during testing

**Impact:** Medium - development slows down

**Mitigation:**
- Implement daily cost limits ($5/day)
- Use caching for repeated prompts
- Mock LLM responses in tests (only use real API for integration tests)
- Monitor costs with analytics dashboard

### Risk 3: Complexity Creep

**Risk:** PoC becomes too complex, timeline extends

**Impact:** Medium - delays validation

**Mitigation:**
- Strict scope: only 5 document types in PoC
- Resist adding features during development
- Timebox each phase (1 week max)
- Cut features if behind schedule (e.g., skip git integration if needed)

### Risk 4: User Adoption Barriers

**Risk:** Tool is hard to set up or use

**Impact:** High - users won't test it

**Mitigation:**
- Zero-config defaults (works with just API key)
- Excellent documentation and examples
- Wizard-like init process
- Pre-built demo project to explore

### Risk 5: Technical Blockers

**Risk:** Unforeseen technical challenges (e.g., LLM API limitations)

**Impact:** Medium to High - may require pivot

**Mitigation:**
- Prototype riskiest parts first (Week 1: LLM integration)
- Have fallback options (OpenAI if Claude doesn't work well)
- Keep architecture modular (easy to swap components)

---

## 🔮 Future Enhancements (Post-PoC)

**Not in PoC scope, but ideas for later:**

### Short-term (v0.2 - v0.5)

1. **Web UI**
   - Browser-based interface (Next.js app)
   - Visual diff viewer
   - Document graph visualization

2. **More document types**
   - Sequence diagrams
   - API contracts (OpenAPI spec generation)
   - Test cases
   - Deployment runbooks

3. **Advanced validation**
   - Cross-document consistency (e.g., every user story has acceptance criteria)
   - Completeness checking (coverage analysis)
   - Terminology consistency scoring

4. **CI/CD integration**
   - GitHub Action / GitLab CI
   - Pre-commit hooks
   - PR bot (suggests doc updates based on code changes)

5. **Collaboration features**
   - Multi-user approval workflow
   - Comments on generated docs
   - Change request system

### Medium-term (v1.0)

6. **Bidirectional sync**
   - Code → Docs: analyze code, update docs automatically
   - Docs → Code: generate code from updated docs

7. **Multi-project support**
   - Reuse domain vocabulary across projects
   - Template library (common patterns)

8. **Analytics dashboard**
   - Time saved vs. manual approach
   - Cost analysis
   - Quality metrics over time

9. **VSCode extension**
   - Inline generation in editor
   - Quick preview
   - Validation warnings in Problems panel

### Long-term (v2.0+)

10. **Multi-agent orchestration**
    - Specialized agents for different domains (e.g., fintech, healthcare)
    - Agent marketplace

11. **Self-improvement**
    - Learn from user corrections
    - Fine-tune generation based on project patterns

12. **Full project lifecycle**
    - Not just docs, but also project planning, sprint planning, retrospectives

---

## 📚 References & Prior Art

### Similar projects:

1. **Cursor (AI-first IDE)**
   - Focus: code generation from natural language
   - Difference: AI-PDS focuses on documentation-first, then code

2. **GitHub Copilot Workspace**
   - Focus: task planning and code generation
   - Difference: AI-PDS emphasizes structured, version-controlled docs

3. **Replit Agent**
   - Focus: full-stack app generation from prompts
   - Difference: AI-PDS separates docs from implementation, more enterprise-friendly

4. **v0.dev (Vercel)**
   - Focus: UI component generation from descriptions
   - Difference: AI-PDS covers full software lifecycle, not just frontend

5. **LangChain Document Loaders**
   - Focus: loading documents for LLM context
   - Difference: AI-PDS generates and maintains docs, not just loads them

### Academic research:

- "Documentation Debt" (Avgeriou et al.)
- "AI-Assisted Software Documentation" (Chen et al., 2024)
- "Prompt Engineering for Code Generation" (various)

### Industry standards:

- C4 Model (architecture diagrams)
- Arc42 (architecture documentation template)
- Living Documentation (Cyrille Martraire)
- Domain-Driven Design documentation patterns

---

## 🎯 Conclusion

This PoC aims to validate the core hypothesis:

> **AI can generate high-quality, structured documentation from natural language descriptions, reducing human effort while maintaining or improving consistency and completeness.**

**Success looks like:**
- A developer can describe a new feature in plain language
- The tool generates 5+ consistent, cross-linked documentation files
- Another AI can read those docs and generate correct code
- The whole process takes < 5 minutes
- The documentation stays up-to-date because it's easy to regenerate

**If successful, this unlocks:**
- Faster development cycles (less time writing docs)
- Better AI-assisted coding (AI has better context)
- More maintainable projects (docs don't drift from reality)
- Democratized software engineering practices (small teams can afford good documentation)

**Next steps:**
1. Review and approve this design document
2. Set up development environment
3. Start Phase 1 implementation
4. Iterate based on learnings

---

## 🔗 Why Traceability is Critical (Added in v1.1)

**The traceability system is NOT an optional add-on — it's the FOUNDATION of AI-PDS.**

### Without traceability:
```
Docs written → Code written → Time passes → Docs and code diverge
                                          → Nobody trusts docs
                                          → AI-PDS fails
```

### With bidirectional traceability:
```
Docs written (with IDs) → Code generated (with @traceability)
                       → AI validates consistency automatically
                       → Drift detected immediately
                       → Docs/code sync maintained
                       → AI-PDS succeeds
```

**Key insights:**

1. **Traceability enables automation:** AI can automatically detect when docs and code are out of sync
2. **Traceability enables validation:** Every requirement can be traced to implementation and tests
3. **Traceability enables impact analysis:** "If I change this requirement, what code is affected?"
4. **Traceability enables trust:** Developers trust docs because they're provably accurate
5. **Traceability IS the sync mechanism:** It's not just tracking — it's the enforcement layer

**Therefore:**
- Traceability features are P0 (blocker) priority in PoC
- Every generated document MUST have IDs
- Every generated code MUST have @traceability annotations
- Traceability validation MUST pass before approval

**Without this, we're just building another documentation generator that will be ignored.**

**With this, we're building a system that fundamentally solves the docs-code sync problem.**

---

## 🧪 Why Test-Driven AI Development is Critical (Added in v1.2)

**TDAI is NOT an optional add-on — it's the SOLUTION to AI hallucination!**

### Without TDAI:
```
Requirements → AI generates code
            → AI "creatively interprets" (hallucinates)
            → Human discovers hallucination during manual testing (later)
            → Wasted time, rework
            → Nobody trusts AI-generated code
```

### With TDAI:
```
Requirements → AI generates TESTS first (constraints)
            → Human approves tests
            → AI generates CODE (constrained by tests)
            → Tests RUN automatically
            → Hallucinations CAUGHT immediately (test failures)
            → Fix before wasted work
            → High confidence in AI-generated code
```

**Key insights:**

1. **Tests are CONSTRAINTS, not just validation:** They tell AI what it CAN and CANNOT do
2. **Negative tests prevent hallucinations:** "Should accept X without Y" → AI can't add Y requirement
3. **Executable specifications:** Tests are formal, unambiguous specifications (not prose)
4. **Fast feedback loop:** Hallucinations caught in seconds, not hours/days
5. **TDAI + Traceability = Self-validating system:** Every requirement → test → code, all linked

**Critical example:**

```typescript
// Requirement: "Password ≥ 8 chars" (no uppercase/special char requirement!)

// NEGATIVE TEST (prevents hallucination):
it('should accept lowercase-only password', () => {
  expect(validatePassword('onlylowercase')).toBe(true);
});

// If AI hallucinates and adds:
if (!/[A-Z]/.test(password)) return false;  // ← HALLUCINATION!

// → Test FAILS immediately! Hallucination caught!
```

**Therefore:**
- TDAI features are P0 (blocker) priority in PoC
- Tests MUST be generated BEFORE code
- ≥20% of tests MUST be negative tests
- Test pyramid ratio MUST be 70:20:10 (±5%)
- Hallucination detection rate MUST be ≥90%

**Without TDAI:**
- AI will hallucinate
- Manual testing will catch it (too late)
- Nobody will trust AI-generated code
- AI-PDS fails

**With TDAI:**
- AI is constrained by tests
- Hallucinations caught automatically
- Developers trust the code (tests prove correctness)
- AI-PDS succeeds

**The Triple Foundation of AI-PDS:**

```
1. Traceability: Solves docs-code sync problem
2. TDAI: Solves AI hallucination problem
3. Human QA: Final safety net

Together → Reliable, trustworthy AI-assisted development
```

---

*This design document will evolve as we learn during implementation. Treat it as a living document, not a rigid spec.*

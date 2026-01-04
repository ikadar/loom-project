# L4 Implementation Design - Roadmap

## Probléma

Az L3 (Implementation Artifacts) és a tényleges kód között hiányzik egy réteg:
- L3 megmondja MIT kell implementálni (test cases, API, tickets)
- De nem mondja meg HOGYAN (architecture, patterns, conventions)

Ez a gap okozza:
- Inkonzisztens kódminőség
- Ad-hoc architektúra döntések
- Nehezen karbantartható kód

---

## Javaslat: L4 - Implementation Design

```
L0 (Vision)
    ↓
L1 (Requirements)     ← MIT akarunk
    ↓
L2 (Design)           ← HOGYAN tervezzük
    ↓
L3 (Implementation)   ← MIT implementálunk
    ↓
L4 (Implementation    ← HOGYAN implementálunk  ← ÚJ!
    Design)
    ↓
Code                  ← Tényleges kód
```

---

## L4 Output Fájlok

| Fájl | Tartalom | Input |
|------|----------|-------|
| `architecture.md` | Clean Architecture, layers, dependency rules | L2 aggregate-design, sequence-design |
| `patterns.md` | Design patterns per component | L2 tech-specs, L3 skeletons |
| `coding-standards.md` | Error handling, naming, DI approach | Project config |
| `project-structure.md` | Directory layout, file naming | L3 service-boundaries |
| `testing-strategy.md` | Test layers, mocking, fixtures, TDD workflow | L3 test-cases |

---

## Technológia-specifikusság

L4 az első **language-specific** szint:

```
L3 (language-agnostic)
    │
    ├── L4-go/           # Go implementation design
    │   ├── architecture.md      (Clean Architecture Go-ban)
    │   ├── patterns.md          (Go idioms, interfaces)
    │   ├── coding-standards.md  (gofmt, golint, error handling)
    │   ├── project-structure.md (cmd/, internal/, pkg/)
    │   └── testing-strategy.md  (table-driven tests, testify)
    │
    ├── L4-typescript/   # TypeScript implementation design
    │   ├── architecture.md      (Hexagonal, NestJS modules)
    │   ├── patterns.md          (DI, decorators)
    │   ├── coding-standards.md  (eslint, prettier)
    │   ├── project-structure.md (src/, test/, libs/)
    │   └── testing-strategy.md  (jest, vitest, mocking)
    │
    └── L4-python/       # Python implementation design
        └── ...
```

---

## TDD Integráció

### Opció 1: TDD mint L4 konfiguráció

```yaml
# loom.config.yaml
l4:
  language: go
  methodology: tdd          # code-first | tdd | bdd
  test_framework: testing   # testing | testify | ginkgo
```

Ha `methodology: tdd`, a `testing-strategy.md` tartalmazza:
- Red-Green-Refactor workflow
- Test-first template per acceptance criteria
- Fixture és mock strategy

### Opció 2: TDD mint külön derivation path

```
L3 test-cases.md
    │
    ├── [tdd path]
    │   └── L4 testing-strategy.md (test-first templates)
    │       └── Code (tests first, then implementation)
    │
    └── [code-first path]
        └── L4 testing-strategy.md (test-after templates)
            └── Code (implementation first, then tests)
```

### Ajánlás

**Opció 1** - konfiguráció alapú. Indokok:
- Egyszerűbb implementáció
- Nem bonyolítja a derivation flow-t
- User dönt a projekt elején

---

## Roadmap

### Fázis 1: Design (1-2 nap)
- [ ] L4 output fájlok struktúrájának véglegesítése
- [ ] Prompt templates tervezése (mit kérdezzen Claude)
- [ ] Traceability mapping: L3 → L4 → Code
- [ ] TDD/code-first config design

### Fázis 2: Spec Extension (1 nap)
- [ ] `l2/interface-contracts.md` + IC-DRV-004
- [ ] `l2/sequence-design.md` + SEQ-L4-001
- [ ] `l2/internal-api.md` + L4 types
- [ ] `l2/aggregate-design.md` + AGG-L4-001

### Fázis 3: Prompt Development (2-3 nap)
- [ ] `prompts/derive-l4-architecture.md`
- [ ] `prompts/derive-l4-patterns.md`
- [ ] `prompts/derive-l4-coding-standards.md`
- [ ] `prompts/derive-l4-project-structure.md`
- [ ] `prompts/derive-l4-testing-strategy.md`

### Fázis 4: CLI Implementation (1-2 nap)
- [ ] `cmd/derive_l4.go`
- [ ] `internal/formatter/l4_*.go`
- [ ] Language template system
- [ ] Config parsing (loom.config.yaml)

### Fázis 5: Integration (1 nap)
- [ ] `cmd/cascade.go` update (L4 phase)
- [ ] Validation rules (V011-V015)
- [ ] End-to-end test

---

## Lezárt Kérdések

### 1. Language Templates hol lakjanak? → HIBRID (C)

**Döntés:** Hibrid rendszer

```
Prioritás (highest → lowest):
1. ./.loom/templates/    # Project-local override
2. ~/.loom/templates/    # User-global override
3. Built-in (embedded)   # go, typescript, python
```

**Implementáció:**
```go
//go:embed templates/go/*.md templates/typescript/*.md templates/python/*.md
var builtinTemplates embed.FS

func LoadTemplate(lang, file string) string {
    // 1. Check project-local
    if content := tryLoad("./.loom/templates/" + lang + "/" + file); content != "" {
        return content
    }
    // 2. Check user-global
    if content := tryLoad("~/.loom/templates/" + lang + "/" + file); content != "" {
        return content
    }
    // 3. Fall back to built-in
    return loadEmbedded(builtinTemplates, lang, file)
}
```

---

### 2. Framework-specifikus templates? → LANGUAGE-ONLY (A) + FUTURE EXPANSION

**Döntés:** V1-ben csak language-level templates.

**V1 Scope:**
- `templates/go/` - Vanilla Go
- `templates/typescript/` - Vanilla TS
- `templates/python/` - Vanilla Python

**Future Expansion Plan (V2+):**

```
templates/
├── go/
│   ├── _base/           # Közös Go alapok (V1)
│   ├── gin/             # Gin framework (V2)
│   ├── echo/            # Echo framework (V2)
│   ├── grpc/            # gRPC (V2)
│   └── cli-cobra/       # CLI apps (V2)
├── typescript/
│   ├── _base/           # Közös TS alapok (V1)
│   ├── nestjs/          # NestJS (V2)
│   ├── express/         # Express (V2)
│   └── nextjs/          # Next.js (V2)
└── python/
    ├── _base/           # Közös Python alapok (V1)
    ├── fastapi/         # FastAPI (V2)
    ├── django/          # Django (V2)
    └── flask/           # Flask (V2)
```

**Trigger V2-re:** User feedback vagy community request alapján.

---

### 3. L4 → Code automation? → FULL AUTO (C)

**Döntés:** Teljes kód generálás.

**Indoklás - A Loom lényege:**

```
┌─────────────────────────────────────────────────────────────┐
│  A Loom célja: AI code generation BIZTONSÁGOSSÁ tétele      │
│                                                             │
│  Hagyományos AI coding:                                     │
│  "Write me an order service" → Hallucination, inconsistent  │
│                                                             │
│  Loom approach:                                             │
│  L0 → L1 → L2 → L3 → L4 → Code                             │
│                                                             │
│  Mire a Code generáláshoz érünk:                           │
│  ✓ Requirements validated (L1)                              │
│  ✓ Design validated (L2)                                    │
│  ✓ Test cases defined (L3)                                  │
│  ✓ Architecture specified (L4)                              │
│  ✓ Patterns specified (L4)                                  │
│  ✓ Coding standards specified (L4)                          │
│                                                             │
│  → AI-nak MINDEN context megvan a helyes generáláshoz       │
│  → Generált kód VALIDÁLHATÓ L3 test cases alapján           │
└─────────────────────────────────────────────────────────────┘
```

**Code Generation Flow:**

```
loom generate --lang go --output ./src

Input:
├── l1/*.md          # Requirements context
├── l2/*.md          # Design context
├── l3/test-cases.md # Validation criteria
├── l4/*.md          # Implementation guide
└── loom.config.yaml # Project config

Output:
├── src/
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── ...
└── tests/
    └── ... (from L3 test cases)
```

**Safety Mechanisms:**
1. **Pre-generation validation:** L1-L4 completeness check
2. **Post-generation validation:** Generated tests must pass
3. **Traceability:** Every generated file references source specs
4. **Human review:** Git diff before commit

---

## Döntések

| ID | Döntés | Indoklás | Státusz |
|----|--------|----------|---------|
| L4-001 | L4 language-specific | Coding standards, patterns language-függők | **ACCEPTED** |
| L4-002 | TDD config-based | Egyszerűbb, nem bonyolítja flow-t | **ACCEPTED** |
| L4-003 | Hibrid template system | Built-in + user override flexibility | **ACCEPTED** |
| L4-004 | Language-level templates first | Framework-specific V2-ben, YAGNI | **ACCEPTED** |
| L4-005 | Full auto code generation | Loom lényege: safe AI code gen | **ACCEPTED** |

---

## Roadmap Update

### Fázis 5: Integration → Code Generation

Frissített fázis:

```
### Fázis 5: Code Generation (2-3 nap)
- [ ] `cmd/generate.go` - Full code generation command
- [ ] Template engine (Go text/template based)
- [ ] Pre-generation validation
- [ ] Post-generation test execution
- [ ] Traceability comment injection
```

### Új Fázis 6: Validation & Safety

```
### Fázis 6: Validation & Safety (1-2 nap)
- [ ] Generated code validation against L3 tests
- [ ] Hallucination detection (references non-existent specs?)
- [ ] Coverage check (all L3 test cases covered?)
- [ ] Human review workflow integration
```

---

## Következő Lépések

1. ~~**Review** - Átnézni ezt a roadmap-et~~ ✓
2. ~~**Döntések** - Nyitott kérdések lezárása~~ ✓
3. **Fázis 1 indítása** - L4 output struktúra véglegesítése

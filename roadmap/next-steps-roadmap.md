---
title: "Loom Next Steps Roadmap"
status: "active"
version: "1.3.0"
created: "2025-12-21"
last_updated: "2025-12-21"
context: "Phase 2 mostly complete, Phase 2B (UI) planned"
current_score: "9.2/10 (Opus v03)"
---

# Loom - Következő Lépések Roadmap

## Jelenlegi Állapot (2025-12-21)

### Amit elértünk

| Milestone | Status | Bizonyíték |
|-----------|--------|------------|
| L0→L1→L2→L3 deriváció | ✅ Validált | 53→1390 sor (26x) |
| TDAI (negatív tesztek) | ✅ Validált | 33% negative, 20% ShouldNOT |
| RAG integration | ✅ Validált | 3→7 szekció, guidelines-compliant |
| Structured Interview | ✅ Validált | 66 decision point, Entity/VO teszt |
| 4 skill v2.0 | ✅ Kész | loom-derive, domain, l2, l3 |
| **Phase 1: Booking System** | ✅ **Kész** | 8 fájl, 3576 sor, 20 SI döntés |
| **Phase 2.1: Skill Egységesítés** | ✅ **Kész** | `/loom` dispatcher + 4 specialized skill |
| **Phase 2.2: Validation Skill** | ✅ **Kész** | `loom-validate` 4 check típussal |
| **Phase 2.3: Self-Learning System** | ✅ **Kész** | RAG multi-source, SI decision reuse |

### Phase 1 Eredmények

| Metrika | Eredmény |
|---------|----------|
| Domain | Booking System (időpontfoglalás) |
| L0 input | 2 fájl (vocabulary + stories) |
| L1 output | 18 AC, 13 BR, 5 aggregate |
| L2 output | 10 endpoint, 9 sequence |
| L3 output | 22 test case |
| SI kérdések | 20 (5 per deriválás) |
| SI relevancia | 100% (minden kérdés hasznos) |

### Amit még nem validáltunk

| Gap | Kockázat | Prioritás |
|-----|----------|-----------|
| ~~Valós projekt (nem Quote domain)~~ | ~~Magas~~ | ~~P0~~ ✅ |
| Multi-developer workflow | Magas | P1 |
| Time savings mérés (baseline) | Közepes | P1 |
| Edge case kezelés | Közepes | P2 |
| Skill egységesítés | Alacsony | P3 |

---

## Javasolt Fázisok

```
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 1: Real Project Validation (P0)          ✅ KÉSZ        │
│  ─────────────────────────────────────────────────────────────  │
│  Booking System: L0→L3, 20 SI döntés, 3576 sor                  │
├─────────────────────────────────────────────────────────────────┤
│  PHASE 2: Developer Experience (P1)             ◄── KÖVETKEZŐ  │
│  ─────────────────────────────────────────────────────────────  │
│  Skill egységesítés, validation skill, error handling           │
├─────────────────────────────────────────────────────────────────┤
│  PHASE 3: Multi-Developer Test (P1)             ~1-2 hét       │
│  ─────────────────────────────────────────────────────────────  │
│  2-3 developer párhuzamosan, merge strategy                     │
├─────────────────────────────────────────────────────────────────┤
│  PHASE 4: Production Hardening (P2)             ~2 hét         │
│  ─────────────────────────────────────────────────────────────  │
│  Edge cases, error recovery, performance                        │
├─────────────────────────────────────────────────────────────────┤
│  PHASE 5: Release & Documentation (P3)          ~1 hét         │
│  ─────────────────────────────────────────────────────────────  │
│  v1.0 release, user guide, onboarding                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Real Project Validation ✅ KÉSZ

**Cél:** Bizonyítani, hogy a Loom működik új, ismeretlen domainen

**Státusz: SIKERESEN TELJESÍTVE** (2025-12-21)

**Eredmény:**
- Booking System domain (időpontfoglalás) - teljesen eltérő a Quote-tól
- L0→L3 teljes deriválás működik
- 20 SI döntési pont, mind releváns
- 8 fájl, 3576 sor generálva

### 1.1 Domain Kiválasztás

**Opciók:**

| Domain | Komplexitás | Előny | Hátrány |
|--------|-------------|-------|---------|
| **Todo App + Auth** | Alacsony | Gyors, ismert | Túl egyszerű |
| **Inventory Management** | Közepes | Jó aggregate-ek | Nem annyira izgalmas |
| **Booking System** | Közepes-Magas | Időkezelés, conflicts | Komplexebb |
| **E-commerce Cart** | Közepes | Sok state transition | Quote-hoz hasonló |
| **Workflow Engine** | Magas | State machine, events | Túl komplex első tesztnek |

**Ajánlás:** **Booking System** (szobafoglalás vagy időpontfoglalás)
- Elég komplex (availability, conflicts, cancellation)
- Eltérő domain a Quote-tól
- Jó teszt az SI-nek (sok döntési pont)

### 1.2 Végrehajtási Terv

```
Week 1:
├── Day 1-2: L0 dokumentumok írása (domain-vocabulary, user-stories)
│            → MÉR: Mennyi idő az L0?
│
├── Day 3: L0→L1 deriválás SI-vel
│          → MÉR: Hány SI kérdés? Mennyi idő?
│          → OUTPUT: acceptance-criteria.md, business-rules.md
│
├── Day 4: Domain modeling SI-vel
│          → MÉR: Entity/VO döntések száma
│          → OUTPUT: domain-model.md
│
└── Day 5: L1→L2 deriválás SI-vel
           → MÉR: Architekturális döntések
           → OUTPUT: interface-contracts.md, sequence-design.md

Week 2:
├── Day 1-2: L2→L3 deriválás SI-vel
│            → OUTPUT: test-cases.md
│
├── Day 3-4: Manuális baseline mérés
│            → Ugyanez a domain, manuálisan
│            → MÉR: Teljes idő összehasonlítás
│
└── Day 5: Eredmények dokumentálása
           → poc/booking-system/ mappa
           → Validation report
```

### 1.3 Mérési Terv

| Metrika | Mérés módja | Cél |
|---------|-------------|-----|
| L0 írási idő | Stopper | Dokumentálni (nincs cél) |
| SI kérdések száma | Számolás | 15-25 per deriválás |
| Deriválási idő per szint | Stopper | <10 perc |
| Human correction rate | Review után | <20% |
| Manuális baseline | Párhuzamos teszt | Összehasonlítás |

### 1.4 Siker kritériumok

- [x] L0→L3 deriválás működik új domainen ✅
- [x] SI kérdések relevánsak (nem feleslegesek) ✅ 20/20
- [x] Entity/VO döntések helyesek ✅ TimeSlot=VO, Calendar=Aggregate
- [x] Generált tesztek futtathatók ✅ 22 test case
- [ ] Time savings ≥50% a manuálishoz képest (nem mért - Quick validation)

---

## Phase 2: Developer Experience (P1) - IN PROGRESS

**Cél:** A skill-ek használatát egyszerűbbé és robusztusabbá tenni

### 2.1 Skill Egységesítés ✅ KÉSZ

**Megoldás:** Dispatcher pattern - unified `/loom` command routes to specialized skills

```bash
# Unified interface (implemented)
/loom derive --level L1 --input user-stories.md --output-dir output/
/loom derive --level domain --input stories.md,vocabulary.md --output-dir output/
/loom derive --level L2 --input ac.md,br.md --output-dir output/
/loom derive --level L3 --input contracts.md,ac.md,br.md --output-dir output/

# Validation
/loom validate --dir output/ --check all
```

**Implementáció:**
- `loom.md` - Dispatcher skill, routes to specialized skills
- Preserves specialized skill intelligence (focused prompts, SI catalogs)
- Consistent argument mapping

### 2.2 Validation Skill ✅ KÉSZ

```bash
/loom validate --dir output/ --check traceability  # All IDs exist and linked
/loom validate --dir output/ --check format        # YAML frontmatter, markdown
/loom validate --dir output/ --check coverage      # All requirements have tests
/loom validate --dir output/ --check consistency   # No contradictions
/loom validate --dir output/ --check all           # Run all checks (default)
```

**Implementáció:** `loom-validate.md` with 4 check types

### 2.3 Self-Learning System ✅ KÉSZ

**Eredeti terv:** SI Answer Cache (külön `.loom/si-decisions.yaml` fájl)

**Jobb megoldás:** Self-Learning System - a RAG tudásbázis folyamatosan bővül

```
┌────────────────────────────────────────────────────────────────────┐
│                     SELF-LEARNING SYSTEM                           │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌──────────────┐                                                 │
│  │  Guidelines  │ ─────────────┐                                  │
│  │  (priority 1)│              │                                  │
│  └──────────────┘              │                                  │
│                                ▼                                  │
│  ┌──────────────┐       ┌──────────────┐       ┌──────────────┐  │
│  │   Project    │ ─────►│  RAG Engine  │◄──────│   Derive     │  │
│  │   Docs (p2)  │       │  (ChromaDB)  │       │   Request    │  │
│  └──────────────┘       └──────┬───────┘       └──────────────┘  │
│         ▲                      │                                  │
│         │                      │ retrieve context                 │
│         │                      ▼                                  │
│         │               ┌──────────────┐                         │
│         └───────────────│   Derived    │ (includes SI decisions) │
│           re-index      │   Document   │                         │
│                         └──────────────┘                         │
│                                                                    │
│  The system learns from its own output!                           │
└────────────────────────────────────────────────────────────────────┘
```

**Előnyök:**
1. **Single Source of Truth** - SI döntések a dokumentumokban élnek (YAML frontmatter)
2. **Nincs külön cache fájl** - a tudásbázis = projekt dokumentumok
3. **Automatikus tanulás** - új deriválás → re-index → jobb kontextus
4. **Priority-based retrieval** - projekt docs (p2) > guidelines (p1)

**Implementáció:** `loom-tooling/rag/rag_engine.py`
- `KnowledgeSource` class (type, priority)
- `retrieve_si_decision()` method
- `retrieve_prioritized()` method
- `create_self_learning_rag()` helper

### 2.4 Error Handling Improvement (TODO)

**Jelenlegi:** Hibánál a skill elakad

**Cél:**
- Graceful degradation
- Meaningful error messages
- Recovery suggestions
- Partial output save

---

## Phase 2B: UI/UX Derivation (P1)

**Cél:** Frontend/UI dokumentáció deriválás külön skill chain-nel

**Részletes terv:** [ux-ui-derivation-architecture.md](../thinking/ux-ui-derivation-architecture.md)

### Kulcs Döntések

| Döntés | Választás | Indoklás |
|--------|-----------|----------|
| UI = fork of backend? | **NEM** | UI-nak saját L0 inputja van (mockups) |
| Traceability link | **Business Rules** | BR a híd backend és frontend között |
| Cross-cutting | **Pattern library** | Egyszer deriválva, többször hivatkozva |
| Skill architecture | **Külön chain** | `/loom-ui derive` |

### UI Derivation Levels

```
L0-UI: Mockups + Design Tokens + User Stories
    │
    ▼
L1-UI: UI Interaction Stories (US-UI-*) + UI Acceptance Criteria (AC-UI-*)
    │
    ▼
L2-UI: Component Specs + State Machines + Interaction Patterns
    │
    ▼
L3-UI: E2E Tests + Visual Regression + Manual QA Checklists
```

### UI-specific SI Katalógus

**L1-UI (7 kérdés):** Component granularity, State mgmt, Styling, Design system, A11y, Navigation, Forms

**L2-UI (5 kérdés):** Loading states, Error display, Empty states, Validation timing, Transitions

**L3-UI (5 kérdés):** E2E framework, Visual regression, Storybook, Manual QA depth, Device coverage

**Cross-cutting (egyszer per projekt, 4 kérdés):** Loading strategy, Error display, Validation timing, Empty state style

### Implementációs Fázisok

| Fázis | Leírás | Prioritás |
|-------|--------|-----------|
| 2B.1 | Cross-cutting patterns template | Magas |
| 2B.2 | `/loom-ui derive --level L1` skill | Magas |
| 2B.3 | `/loom-ui derive --level L2` skill | Közepes |
| 2B.4 | `/loom-ui derive --level L3` skill | Közepes |
| 2B.5 | `/loom-ui validate` skill | Alacsony |

### Referencia Projekt

A döntések egy valós UX-UI dokumentáció elemzésén alapulnak:
- Lokáció: `loom-project/tmp/ux-ui/`
- Projekt: Flux Print Shop Scheduling System
- Méret: 34 fájl, teljes UI specifikáció

---

## Phase 3: Multi-Developer Test (P1)

**Cél:** Bizonyítani, hogy Loom működik csapatban

### 3.1 Test Scenario

```
Developer A: Új feature - "Booking Cancellation"
Developer B: Új feature - "Booking Modification"
Developer C: Új feature - "Recurring Booking"

Párhuzamosan dolgoznak, mindhárom:
- Módosítja domain-model.md
- Új AC-kat ad acceptance-criteria.md-hez
- Új teszteket generál

→ Merge conflicts kezelése?
→ SI decision conflicts?
→ Traceability consistency?
```

### 3.2 Merge Strategy

**Opciók:**

| Strategy | Előny | Hátrány |
|----------|-------|---------|
| **Feature branches** | Standard Git flow | Merge conflict markdown-ban |
| **Trunk-based + sections** | Kevesebb conflict | Nehéz koordinálni |
| **Lock-based editing** | Nincs conflict | Bottleneck |
| **AI-assisted merge** | Smart resolution | Bonyolult implementálni |

**Ajánlás:** Feature branches + merge guidelines
- Minden feature saját branch
- Section-based file structure (könnyebb merge)
- Pre-merge validation (`/loom validate`)

### 3.3 SI Decision Governance

**Probléma:** Mi van, ha A és B különböző SI válaszokat ad ugyanarra?

**Megoldás:**
```yaml
# .loom/si-governance.yaml
decision-owners:
  API-*: tech-lead
  SEC-*: security-team
  COM-*: architect

conflict-resolution:
  - notify: tech-lead
  - require: explicit-override
  - audit: log-both-decisions
```

---

## Phase 4: Production Hardening (P2)

**Cél:** Edge case-ek kezelése, robusztusság

### 4.1 Edge Cases

| Edge Case | Kezelés |
|-----------|---------|
| Üres L0 input | Meaningful error + template suggestion |
| Túl nagy L0 (1000+ sor) | Chunking vagy warning |
| Circular references | Detection + error |
| Missing traceability | Validation failure |
| SI timeout (user nem válaszol) | Save state, resume later |
| Partial derivation failure | Rollback vagy partial save |

### 4.2 Performance

| Mérés | Cél | Jelenlegi |
|-------|-----|-----------|
| L0→L1 deriválás | <30 sec | ~3 min (with SI) |
| L2→L3 deriválás | <60 sec | ~8 min (with SI) |
| Validation (100 files) | <10 sec | Nem mért |
| RAG retrieval | <2 sec | ~1 sec |

**Note:** Az SI kérdések miatt a deriválás interactive, nem pure automation.

### 4.3 Error Recovery

```bash
# Ha a deriválás félbeszakad
/loom resume  # Continue from last checkpoint

# Ha rossz output született
/loom rollback --to last-valid

# Ha SI válasz rossz volt
/loom redo-si --decision API-1
```

---

## Phase 5: Release & Documentation (P3)

**Cél:** v1.0 release, onboarding anyagok

### 5.1 Release Checklist

- [ ] Minden skill v2.0+ és tesztelt
- [ ] README.md (loom-tooling) frissítve
- [ ] Installation guide
- [ ] Quick start tutorial (5 perc)
- [ ] Full walkthrough (30 perc)
- [ ] Troubleshooting guide
- [ ] CHANGELOG.md

### 5.2 Onboarding Flow

```
1. Install: Copy skills to .claude/skills/
2. Init: /loom init (creates .loom/ config)
3. First derivation: /loom derive --level L1 --demo
4. Real project: /loom derive --level L1 --input your-stories.md
```

### 5.3 Video/Demo Content

- [ ] 5-min intro: "What is Loom?"
- [ ] 10-min demo: "L0 to L3 in 10 minutes"
- [ ] Deep dive: "Structured Interview explained"
- [ ] Advanced: "Multi-developer workflow"

---

## Összefoglaló Timeline

```
Week 1-2:  Phase 1 - Real Project Validation (Booking System)
Week 3:    Phase 2 - Developer Experience (Skill improvements)
Week 4-5:  Phase 3 - Multi-Developer Test (2-3 developers)
Week 6-7:  Phase 4 - Production Hardening (Edge cases, errors)
Week 8:    Phase 5 - Release & Documentation (v1.0)

Total: ~8 weeks to v1.0
```

---

## Döntési Pontok (Számodra)

Mielőtt elkezdjük, a következő döntések kellenek:

### 1. Domain választás Phase 1-hez

| Opció | Leírás |
|-------|--------|
| A | **Booking System** (ajánlott) - időpontfoglalás |
| B | Inventory Management - készletkezelés |
| C | Saját projekt - valós use case |

### 2. Phase 1 mélysége

| Opció | Leírás |
|-------|--------|
| A | **Quick validation** (~1 hét) - L0→L3, nincs manuális baseline |
| B | **Full validation** (~2 hét) - Manuális baseline mérés is |

### 3. Phase 3 scope

| Opció | Leírás |
|-------|--------|
| A | **Simulated** - Te játszod 2-3 developer szerepét |
| B | **Real** - Valós emberekkel (ha van csapat) |

### 4. Prioritás

| Opció | Leírás |
|-------|--------|
| A | **Speed** - Phase 1-2-3 párhuzamosan ahol lehet |
| B | **Sequential** - Minden phase külön, tanulságokkal |

---

## Következő Akció

**Phase 1 KÉSZ!** ✅ Booking System PoC sikeresen validálta a Loom működését új domainen.

**Phase 2 nagyrészt KÉSZ:**
- ✅ 2.1 Skill Egységesítés - `/loom` dispatcher pattern
- ✅ 2.2 Validation Skill - 4 check típus (traceability, format, coverage, consistency)
- ✅ 2.3 Self-Learning System - RAG multi-source, SI decision reuse
- ⬜ 2.4 Error Handling Improvement - TODO

**Phase 2B TERVEZVE:**
- ⬜ 2B.1 Cross-cutting patterns template
- ⬜ 2B.2 `/loom-ui derive --level L1` skill
- ⬜ 2B.3-5 További UI skill-ek

**Választási lehetőségek:**

| Opció | Leírás |
|-------|--------|
| A | **Phase 2.4** - Error Handling Improvement befejezése |
| B | **Phase 2B** - UI/UX Derivation skill chain kezdése |
| C | **Phase 3** - Multi-Developer Test indítása |
| D | **Integration Test** - Skills + RAG Self-Learning együttes tesztelése |

**Ajánlás:** B opció - A UI derivation skill chain alapozza meg a full-stack dokumentáció deriválást. A valós UX-UI referencia projekt elemzése alapján tiszta architektúrával rendelkezünk.

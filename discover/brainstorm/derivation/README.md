---
title: Documentation Derivation System
date: 2026-01-02
purpose: Overview of loom derivation architecture and implementation
status: active
---

# Documentation Derivation System

## Overview

A loom-cli derivation rendszere automatikusan generál dokumentációt strukturált szinteken (L0 → L1 → L2 → L3).

## Dokumentumok

| Fájl | Leírás | Státusz |
|------|--------|---------|
| `documentation-derivation-strategy.md` | Teljes derivation stratégia és specifikáció | Aktív |
| `derivation-example-walkthrough.md` | Kaszkád deriváció példa (Phase 3 vision) | Koncepció |
| `derivation-visual-diagram.md` | Mermaid diagramok a rendszerhez | Aktív |
| `document-validation-rules.md` | Validációs szabályok specifikáció | Spec |
| `ux-ui-derivation-architecture.md` | UI/UX derivation (future work) | Draft |

## Derivation Szintek

```
L0: FOUNDATIONAL (Human Input)
  └─ user-stories.md, domain-vocabulary.md, nfr.md

L1: PRIMARY DERIVATION (loom-cli derive)
  └─ domain-model.md, acceptance-criteria.md, business-rules.md, bounded-context-map.md

L2: SECONDARY DERIVATION (loom-cli derive-l2)
  └─ interface-contracts.md, sequence-design.md, initial-data-model.md
  └─ aggregate-design.md, tech-specs.md

L3: TERTIARY DERIVATION (loom-cli derive-l3)
  └─ test-cases.md, openapi.json, implementation-skeletons.md
  └─ feature-tickets.md, service-boundaries.md, event-message-design.md
  └─ dependency-graph.md
```

## Implementáció (loom-cli)

### Teljes workflow

```bash
# L0 analízis és interview
loom-cli analyze --input-dir l0-folder --output-dir l1-folder
loom-cli interview --init l1-folder/analysis.json

# L1 Derivation (Strategic Design)
loom-cli derive --input-dir l0-folder --output-dir l1-folder

# L2 Derivation (Tactical Design)
loom-cli derive-l2 --input-dir l1-folder --output-dir l2-folder

# L3 Derivation (Operational Design)
loom-cli derive-l3 --input-dir l2-folder --output-dir l3-folder

# Validation
loom-cli validate --input-dir output-folder --level ALL
```

### Egyszerűsített példák

```bash
# L1: domain model, AC, BR, bounded context
loom-cli derive --input-dir ./l0 --output-dir ./l1

# L2: contracts, sequences, tech specs
loom-cli derive-l2 --input-dir ./l1 --output-dir ./l2

# L3: tests, API, skeletons, tickets
loom-cli derive-l3 --input-dir ./l2 --output-dir ./l3
```

## Prompt Fájlok

A derivation promptok: `loom-tooling/loom-cli/prompts/`

### Analyze & Interview
- `domain-discovery.md` - Domain felfedezés
- `entity-analysis.md` - Entity elemzés
- `operation-analysis.md` - Műveletek elemzése
- `interview.md` - Strukturált interjú

### L1 Derivation
- `derivation.md` - Fő L1 derivation
- `derive-domain-model.md` - Domain model generálás
- `derive-bounded-context.md` - Bounded context map

### L2 Derivation
- `derive-l2.md` - L2 fő orchestráció
- `derive-interface-contracts.md` - API contracts
- `derive-sequence-design.md` - Sequence diagramok
- `derive-data-model.md` - Initial data model
- `derive-aggregate-design.md` - DDD aggregates
- `derive-tech-specs.md` - Technical specifications

### L3 Derivation
- `derive-l3.md` - L3 fő orchestráció
- `derive-test-cases.md` - TDAI test cases
- `derive-l3-api.md` - OpenAPI spec
- `derive-l3-skeletons.md` - Implementation skeletons
- `derive-feature-tickets.md` - Feature tickets
- `derive-service-boundaries.md` - Service boundaries
- `derive-event-design.md` - Event/message design
- `derive-dependency-graph.md` - Dependency graph

## ID Conventions

| Szint | Prefix | Példa |
|-------|--------|-------|
| L1 | ENT-, VO-, AC-, BR-, BC- | ENT-CUSTOMER, AC-ORD-001, BR-SHIP-001 |
| L2 | IC-, SEQ-, AGG-, DM-, TS- | IC-ORDER-001, SEQ-CHECKOUT-001, TS-BR-AUTH-001 |
| L3 | TC-, SKEL-, SVC-, EVT-, CMD-, FDT-, DEP- | TC-AC-ORD-001-P01, SKEL-ORDER-001 |

## Archivált Dokumentumok

A `archive/` mappában találhatók a befejezett implementációs tervek:

- `derivation-gap-implementation-plan.md` - Gap analysis (COMPLETED)
- `loom-cli-refactoring-plan.md` - R1-R6 refactoring (COMPLETED)
- `prompt-engineering-improvement-plan.md` - P1-P8 prompt improvements (COMPLETED)
- `derivation-implementation-plan.md` - Original plan (SUPERSEDED)
- `api-based-derivation-architecture.md` - API architecture (SUPERSEDED)

## Kapcsolódó

- `loom-tooling/loom-cli/` - CLI implementáció
- `loom-tooling/loom-cli/cmd/` - Command implementációk
- `loom-tooling/test/benchmark/` - Benchmark teszt eredmények
- `ai-dop-spec/` - AI-DOP specifikáció

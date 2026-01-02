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
| `derivation-example-walkthrough.md` | Konkrét példa végigvezetés | Aktív |
| `derivation-visual-diagram.md` | Mermaid diagramok a rendszerhez | Aktív |
| `document-validation-rules.md` | Validációs szabályok | Aktív |
| `ux-ui-derivation-architecture.md` | UI/UX derivation (future work) | Tervezett |

## Derivation Szintek

```
L0: FOUNDATIONAL (Human Input)
  └─ user-stories, domain-vocabulary, NFRs

L1: PRIMARY DERIVATION (AI 80%)
  └─ domain-model, acceptance-criteria, business-rules, bounded-context

L2: SECONDARY DERIVATION (AI 90%)
  └─ interface-contracts, sequence-design, data-model, aggregate-design
  └─ tech-specs (technical specifications)

L3: TERTIARY DERIVATION (AI 95%)
  └─ test-cases, api-spec, implementation-skeletons
  └─ feature-tickets, service-boundaries, event-design, dependency-graph
```

## Implementáció (loom-cli)

### L1 Derivation
```bash
loom derive l1 --input l0-folder --output l1-folder
```

### L2 Derivation
```bash
loom derive l2 --input l1-folder --output l2-folder
```

### L3 Derivation
```bash
loom derive l3 --input l2-folder --output l3-folder
```

### Validation
```bash
loom validate --dir output-folder
```

## Prompt Fájlok

A derivation promptok: `loom-tooling/loom-cli/prompts/`

- `derive-l2.md` - L2 fő derivation
- `derive-tech-specs.md` - Technical specifications
- `derive-test-cases.md` - Test cases (L3)
- `derive-l3.md` - L3 fő derivation
- `derive-l3-api.md` - OpenAPI spec generálás
- `derive-l3-skeletons.md` - Implementation skeletons
- `derive-feature-tickets.md` - Feature tickets
- `derive-service-boundaries.md` - Service boundaries
- `derive-event-design.md` - Event design
- `derive-dependency-graph.md` - Dependency graph

## ID Conventions

| Szint | Prefix | Példa |
|-------|--------|-------|
| L1 | ENT-, AC-, BR- | ENT-CUST-001, AC-ORD-002 |
| L2 | TS-, DM-, SEQ- | TS-CUST-001, DM-001 |
| L3 | TC-, SKEL-, DEP-, EVT-, CMD-, SVC-, FDT- | TC-ORD-001, SKEL-CUST-001 |

## Archivált Dokumentumok

A `archive/` mappában találhatók a befejezett implementációs tervek:

- `derivation-gap-implementation-plan.md` - Gap analysis (COMPLETED)
- `loom-cli-refactoring-plan.md` - R1-R6 refactoring (COMPLETED)
- `prompt-engineering-improvement-plan.md` - P1-P8 prompt improvements (COMPLETED)
- `derivation-implementation-plan.md` - Original plan (SUPERSEDED)
- `api-based-derivation-architecture.md` - API architecture (SUPERSEDED)

## Kapcsolódó

- `loom-tooling/loom-cli/` - CLI implementáció
- `loom-tooling/test/benchmark/` - Benchmark teszt eredmények
- `ai-dop-spec/` - AI-DOP specifikáció

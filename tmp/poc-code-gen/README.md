# POC: Code Generation Test

## Cél

Tesztelni, hogy a Loom által derivált specifikációk valóban **"glass box"** kódot eredményeznek-e:
- Traceable (minden függvény visszavezethető requirement-re)
- Explicit (minden döntés dokumentált JSDoc-ban)
- Karbantartható (új fejlesztő gyorsan érti a kontextust)

## Input

Természetes nyelvi projekt leírás, ahogy egy product owner elmondaná:

| File | Tartalom |
|------|----------|
| `input/project-brief.md` | Részletes projekt leírás, domain magyarázat |
| `input/quick-stories.md` | Gyors user story lista meeting jegyzet stílusban |

## Loom Workflow

```
input/                          Loom deriválás                output/
├── project-brief.md    ──────────────────────────────►    ├── L1/
└── quick-stories.md                                       │   ├── acceptance-criteria.md
                                                           │   └── business-rules.md
                                                           ├── L2/
                                                           │   ├── interface-contracts.md
                                                           │   └── sequence-design.md
                                                           └── L3/
                                                               └── tests/
```

## Teszt Forgatókönyv

### 1. L0 → L1 Deriválás
```bash
/loom-derive --input input/project-brief.md --output output/L1
```
Ellenőrzés:
- [ ] AC-k Given/When/Then formátumban
- [ ] BR-ek enforcement mechanizmussal
- [ ] 100% traceability

### 2. L1 → L2 Deriválás
```bash
/loom-derive --level L2 --input output/L1 --output output/L2
```
Ellenőrzés:
- [ ] API contracts OpenAPI-szerű formátumban
- [ ] Sequence diagramok PlantUML-ben

### 3. Kód Generálás
A derivált specifikációkat bemenetként adjuk egy coding agent-nek (Cursor/Claude Code).

Ellenőrzés:
- [ ] Minden függvény JSDoc-ja tartalmaz requirement ID-t
- [ ] SI döntések megjelennek a kódban
- [ ] Tesztek megfelelnek a TC-knek

## Sikerességi Kritériumok

| Kritérium | Cél |
|-----------|-----|
| Spec expansion | >20x (input → output) |
| Traceability | 100% |
| Human correction | <20% |
| Generated code: requirement refs | 100% |
| Generated tests: TC match | >80% |

## Eredmények

*A POC futtatása után ide jönnek az eredmények*

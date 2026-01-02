---
title: "Document Validation Rules"
status: draft
created: 2025-12-26
extracted-from: platform/poc-tooling-design.md
see-also:
  - ../core-concepts/bidirectional-traceability-design.md
  - ../platform/platform-architecture.md
---

# Document Validation Rules

> Validációs szabályok a Loom által generált/derivált dokumentumokra.

---

## Összefoglaló

A Loom deriválási folyamat során generált dokumentumoknak meg kell felelniük bizonyos formai és tartalmi követelményeknek. Ez a dokumentum definiálja ezeket a validációs szabályokat.

---

## 1. YAML Frontmatter Validáció

Minden Loom dokumentumnak tartalmaznia kell YAML frontmatter-t.

**Kötelező mezők:**

| Mező | Leírás | Példa |
|------|--------|-------|
| `title` | Dokumentum címe | `"Domain Model"` |
| `generated` | Generálás időpontja (RFC3339) | `2026-01-03T00:09:26+01:00` |
| `status` | Dokumentum állapota | `draft` |
| `level` | Deriválási szint | `L1`, `L2`, `L3` |
| `loom-cli-version` | CLI verzió | `0.3.0` |

**Status értékek:**

| Érték | Jelentés |
|-------|----------|
| `draft` | Munka alatt |
| `to review` | Review-ra vár |
| `approved` | Jóváhagyva |
| `living` | Aktívan karbantartott |

**Level értékek:**

| Érték | Jelentés |
|-------|----------|
| `L1` | Strategic Design (Domain Model, AC, BR, Bounded Contexts) |
| `L2` | Tactical Design (Tech Specs, Contracts, Aggregates, Sequences, Data Model) |
| `L3` | Operational Design (Test Cases, Skeletons, Tickets, Events, Dependencies) |

**Példa:**
```yaml
---
title: "Domain Model"
generated: 2026-01-03T00:09:26+01:00
status: draft
level: L1
loom-cli-version: 0.3.0
---
```

---

## 2. Markdown Link Validáció

Belső hivatkozások ellenőrzése.

**Szabályok:**
- Minden belső link létező fájlra kell mutasson
- Azonos szinten: `[BR-001](business-rules.md#br-001)`
- Szintek között: `[ENT-001](../l1/domain-model.md#ent-001)`
- Anchor linkek (#section) létező fejezetre mutassanak

**Validáció:**
- Warning: Nem létező fájlra mutató link
- Error: Kritikus hivatkozás hiányzik (pl. domain-model.md)

---

## 3. Dokumentum-közi Konzisztencia

A dokumentumok közötti fogalmi és entitás konzisztencia ellenőrzése.

**Szabályok:**

| Ha... | Akkor... |
|-------|----------|
| `domain-vocabulary.md` definiál egy fogalmat | `domain-model.md`-ben is szerepelnie kell |
| `user-stories.md` említ egy entitást | `domain-model.md`-ben definiálva kell legyen |
| `acceptance-criteria.md` hivatkozik egy business rule-ra | `business-rules.md`-ben létezzen |

**Cél:** Nincs "lebegő" fogalom vagy entitás, minden konzisztensen végigvezetve.

---

## 4. Struktúra Validáció

A fájlok megfelelő mappában vannak-e. A Loom level-alapú mappastruktúrát használ.

**Mappastruktúra szabályok:**

| Level | Elvárt mappa | Dokumentumok |
|-------|--------------|--------------|
| L1 | `l1/` | domain-model.md, bounded-context-map.md, acceptance-criteria.md, business-rules.md, decisions.md |
| L2 | `l2/` | tech-specs.md, interface-contracts.md, aggregate-design.md, sequence-design.md, initial-data-model.md |
| L3 | `l3/` | test-cases.md, implementation-skeletons.md, feature-tickets.md, service-boundaries.md, event-message-design.md, dependency-graph.md, openapi.json |

**Validációs szabály:**
- A dokumentum `level:` frontmatter mezőjének egyeznie kell a mappa nevével
- Példa: `l2/tech-specs.md` → `level: L2` ✓
- Példa: `l1/tech-specs.md` → `level: L2` ✗ (mappa/level mismatch)

**Miért level-alapú és nem típus-alapú?**
- A deriválás szint-szinten történik (`derive` → `derive-l2` → `derive-l3`)
- Egy szinten belül a dokumentumok együtt használatosak és egymásra hivatkoznak
- Egyszerűbb cross-referenciák (ugyanabban a mappában vannak)

---

## 5. Validációs Viselkedés

### Mikor fut a validáció?
- Generálás után, de approval előtt
- CI/CD pipeline-ban (optional)
- Manuálisan, `loom validate` paranccsal

### Severity szintek

| Szint | Viselkedés |
|-------|------------|
| **Info** | Csak jelzi, nem blokkolja |
| **Warning** | Jelzi, de engedi a generálást |
| **Error** | Blokkolja a fájl írást |

### Példa output

```
$ loom-cli validate --input-dir ./specs

Validating ALL documents in ./specs...

Phase 1: Collecting IDs...
Phase 2: Structural Validation...
Phase 3: Traceability Validation...
Phase 4: Completeness Validation...
Phase 5: TDAI Validation...

========================================
   VALIDATION RESULTS
   Level: ALL
========================================
Structural Validation:
  ✓ [V001] All 5 documents have IDs
  ✓ [V002] All 42 IDs follow expected patterns

Traceability Validation:
  ✓ [V003] All 28 references are valid

Completeness Validation:
  ✓ [V005] All 15 ACs have test cases

TDAI Validation:
  ✓ [V008] Negative test ratio: 25.4% (>= 20%)
  ✓ [V009] All 15 ACs have hallucination prevention tests

========================================
Summary: 6 passed, 0 failed, 0 warnings
========================================
```

---

## Kapcsolódó

- [Bidirectional Traceability](../core-concepts/bidirectional-traceability-design.md) - Kód ↔ Dokumentum szinkronizáció
- [Platform Architecture](../platform/platform-architecture.md) - CLI és validálás technikai részletei

---
title: "Document Validation Rules"
status: draft
created: 2025-12-26
extracted-from: platform/poc-tooling-design.md
see-also:
  - bidirectional-traceability-design.md
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
- `status` - A dokumentum állapota

**Status értékek:**
| Érték | Jelentés |
|-------|----------|
| `draft` | Munka alatt |
| `to review` | Review-ra vár |
| `approved` | Jóváhagyva |
| `living` | Aktívan karbantartott |

**Példa:**
```yaml
---
title: "User Stories"
status: draft
created: 2025-01-15
---
```

---

## 2. Markdown Link Validáció

Belső hivatkozások ellenőrzése.

**Szabályok:**
- Minden belső link (pl. `[domain model](../domain-modelling/domain-model.md)`) létező fájlra kell mutasson
- Relatív útvonalak helyesek legyenek
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

A fájlok megfelelő mappában vannak-e.

**Mappastruktúra szabályok:**

| Dokumentum típus | Elvárt mappa |
|------------------|--------------|
| Domain modelling docs | `domain-modelling/` |
| Requirements docs | `requirements/` |
| Technical specs | `technical/` |
| Test specs | `tests/` |

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
$ loom validate ./specs

✓ YAML frontmatter: OK
⚠ Link validation: 2 warnings
  - user-stories.md:45 → ../domain-modelling/glossary.md (not found)
  - acceptance-criteria.md:78 → ../business-rules.md#BR-099 (anchor not found)
✗ Consistency: 1 error
  - Entity "Invoice" in user-stories.md not defined in domain-model.md

Result: 1 error, 2 warnings
```

---

## Kapcsolódó

- [Bidirectional Traceability](../core-concepts/bidirectional-traceability-design.md) - Kód ↔ Dokumentum szinkronizáció
- [Platform Architecture](../platform/platform-architecture.md) - CLI és validálás technikai részletei

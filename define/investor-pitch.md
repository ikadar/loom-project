---
title: "Loom Investor Pitch"
status: "draft"
version: "1.0.0"
created: "2025-12-21"
audience: "AI-savvy investors"
---

# Loom Investor Pitch

## A Probléma: Az AI Coding Paradoxon

**"Az AI 10x gyorsabban ír kódot, de a projektek nem lesznek 10x gyorsabban készen."**

### Black Box vs Glass Box

A valódi probléma nem az, hogy az AI-generált kód nem működik - **majdnem minden generált kód működik**. A probléma az, hogy **fekete dobozként viselkedik**:

```
AI-generált kód NÉLKÜL Loom:          AI-generált kód LOOM-mal:
┌─────────────────────────┐           ┌─────────────────────────┐
│  "Működik"              │           │  "Működik"              │
│                         │           │                         │
│  De:                    │           │  És:                    │
│  - Miért így?           │           │  - Traceable to US-003  │
│  - Ki döntötte?         │           │  - SI-AUTH-1: JWT       │
│  - Hol a spec?          │           │  - BR-QUOTE-001 enforce │
│  - Hogyan teszteljük?   │           │  - TC-003-1 covers it   │
│  - Mi történik ha...?   │           │  - Edge cases: 5 teszt  │
│                         │           │                         │
│  BLACK BOX              │           │  GLASS BOX              │
└─────────────────────────┘           └─────────────────────────┘
```

**A Black Box kód:**
- Nehéz debugolni (hol keressem a hibát?)
- Nehéz karbantartani (mi törik el ha változtatok?)
- Nehéz továbbfejleszteni (hova illik az új feature?)
- Nehéz onboard-olni (mit csinál ez a kód?)

**A Glass Box kód (Loom-mal):**
- Minden függvény visszavezethető requirement-re
- Minden döntés explicit és dokumentált
- Változás hatása előre látható
- Új fejlesztő percek alatt érti a kontextust

Miért? Mert a szűk keresztmetszet áttolódott:

```
RÉGEN:                          MOST:
Spec (10%) → Kód (80%) → Test   Spec (??%) → Kód (10%) → Test
         ↑                              ↑
    Szűk keresztmetszet           Szűk keresztmetszet
```

Az AI brilliánsan kódol, de **nem tudja, MIT kódoljon**. A specifikáció írás:
- Még mindig manuális
- Inkonzisztens (minden fejlesztő máshogy)
- Nincs traceability (requirement → test)
- AI nem tudja validálni

---

## A Megoldás: Loom

**"Specifikáció deriválás, nem írás."**

```
Ember ír:     User Story (5 perc)
                  ↓
Loom derivál: Acceptance Criteria → Business Rules → API Contracts → Test Cases
                  ↓
AI generál:   Működő, tesztelt kód
```

**Kulcs innováció: Structured Interview (SI)**

Az AI nem találgat - **kérdez**. 87 döntési pont katalógusa biztosítja, hogy minden architekturális döntés explicit:

```
SI-AUTH-1: Milyen autorizációs modell?
  a) RBAC  b) ABAC  c) Ownership-based

→ Válasz bekerül a specifikációba
→ Minden downstream dokumentum konzisztens
→ AI kódgenerátor egyértelmű utasítást kap
```

---

## Validált Eredmények

| Metrika | Eredmény |
|---------|----------|
| Content expansion | **26x** (53 → 1390 sor) |
| Negatív teszt arány | **33%** (iparági avg: 10%) |
| Format compliance | **100%** |
| Traceability | **100%** (minden test ← requirement) |
| Backend + UI | **Működik** (külön skill chain-ek) |

**Tesztelve:**
- Booking System (backend)
- Flux Scheduling UI (frontend) - 5,917 sor specifikáció generálva

---

## Miért Most?

**1. Cursor/Windsurf/Claude Code robbanás**
- Millió fejlesztő használ AI coding tool-t
- Mind ugyanazzal küzd: "hogyan adjak jó promptot?"
- Loom = **strukturált prompt a specifikáció szintjén**

**2. Agentic AI megérkezése**
- Claude, GPT-4 képes komplex, multi-step task-okra
- De kell nekik **jól definiált input**
- Loom pont ezt adja

**3. Enterprise AI adoption görbe**
- Startupok már AI-first
- Enterprise most kezd (compliance, governance igény)
- Loom: **auditálható, traceable AI workflow**

---

## Piaci Lehetőség

**TAM:** $50B+ (Developer Tools + Documentation + Testing)

**Pozicionálás:**
```
                    High Structure
                         ↑
                    [LOOM]
                         |
Low AI ←─────────────────┼─────────────────→ High AI
                         |
              [Traditional Docs]
                         ↓
                    Low Structure
```

**Versenytársak:**

| Tool | Mit csinál | Loom előnye |
|------|------------|-------------|
| Cursor/Copilot | Kód generálás | Mi adjuk a SPEC-et amit generálnak |
| Notion AI | Doc writing | Nincs traceability, nincs SI |
| Linear/Jira AI | Issue mgmt | Nem derivál, csak összefoglal |
| Mintlify | API docs | Post-hoc dokumentáció, nem spec-first |

**Loom nem versenyez velük - kiegészíti őket.**

---

## Egyedi Megkülönböztető Jegyek

### 1. Structured Interview (SI) - "Az AI kérdez, nem találgat"

**A probléma más tool-oknál:**
```
Fejlesztő: "Generálj autentikációt"
AI: *választ valamit (JWT? Session? OAuth?)*
    *nem mondja el mit választott*
    *ha rossz, újra kell kezdeni*
```

**Loom megoldás:**
```
Loom: "SI-AUTH-1: Melyik autentikációs modellt használjuk?
       a) JWT (stateless, scalable)
       b) Session (simple, server-side)
       c) OAuth2 (third-party, complex)"

Fejlesztő: "a"

→ Döntés rögzítve YAML-ban
→ Minden downstream doc konzisztens
→ Újrageneráláskor sem kérdezi újra
```

**87 validált döntési pont** - backend (66) + UI (21)

---

### 2. Self-Learning System - "A rendszer tanul a saját outputjából"

**Más tool-ok:** Minden session nulláról indul

**Loom:**
```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  Derivált dokumentum → RAG indexeli → Következő    │
│        ↑                                deriválás  │
│        │                                    │      │
│        └────────────────────────────────────┘      │
│                                                     │
│  "Projekt 3. hónapjában az AI már ismeri az        │
│   összes korábbi döntést és konvenciót"            │
│                                                     │
└─────────────────────────────────────────────────────┘
```

- Nincs külön "context file" karbantartás
- SI döntések a dokumentumokban élnek
- Minél több deriválás, annál okosabb

---

### 3. Deriválás, nem írás - "Dokumentumok származnak, nem készülnek"

**Más tool-ok:** "Írj nekem acceptance criteria-t"

**Loom:** "User Story-ból **deriválom** az AC-t, BR-t, API contract-ot, teszteket"

```
Input:  53 sor (user stories)
Output: 1390 sor (AC + BR + contracts + tests)

Expansion: 26x
Human effort: <20% correction
```

A különbség:
- **Írás:** AI kitalál valamit
- **Deriválás:** AI logikailag levezeti a szükséges részleteket

---

### 4. 100% Traceability - "Minden test-nek van requirement őse"

**Más tool-ok:** Generált tesztek "lebegnek", nincs kapcsolat a requirement-hez

**Loom:**
```
US-QUOTE-003 (User Story)
    ↓ derives
AC-QUOTE-003-1 (Acceptance Criteria)
    ↓ implements
BR-QUOTE-001 (Business Rule)
    ↓ tested by
TC-QUOTE-003-1-1 (Test Case)
```

**Audit-ready:** Bármikor megmondható, melyik requirement-et melyik teszt fedi le.

---

### 5. Full-Stack Coherence - "Backend és UI egy tudásbázisból"

**Más tool-ok:** Backend és frontend külön világ

**Loom:**
```
                    Business Rules (BR-*)
                           ↑
              ┌────────────┴────────────┐
              ↓                         ↓
    Backend AC (AC-*)           UI Stories (US-UI-*)
              ↓                         ↓
    API Contracts               Component Specs
              ↓                         ↓
    Test Cases                  E2E + Visual Tests

    ────────────────────────────────────────────
                   Közös RAG Knowledge Base
                   Közös SI döntések
```

Egy authorization döntés (SI-AUTH-1) **mindkét chain-ben** érvényesül.

---

### 6. Negative Test First - "Nem csak a happy path"

**Más tool-ok:** Generált tesztek 90%+ pozitív eset

**Loom TDAI (Test-Driven AI):**
```
Generált tesztek:
├── 67% pozitív (should succeed)
├── 33% negatív (should fail)
    ├── 20% "Should NOT" tesztek
    └── Edge case-ek: null, üres, határ értékek
```

**Miért fontos?** A bug-ok 80%-a edge case. Loom ezeket automatikusan generálja.

---

### 7. Cross-cutting Pattern Library - "Egyszer döntünk, mindenhol érvényes"

**Más tool-ok:** Minden komponensnél újra definiáljuk a loading state-et

**Loom:**
```yaml
# ui-patterns.md (egyszer generálva)
PAT-LOADING-SKELETON:
  use-when: "Lists, grids, content areas"
  css: "@apply bg-slate-700/50 animate-pulse"

# component-spec.md (csak hivatkozik)
JobsList:
  loading: → ui-patterns.md#pat-loading-skeleton
```

14 cross-cutting pattern, komponensek csak referálnak.

---

### 8. Incremental, Not Batch - "Nem kell mindent újragenerálni"

**Más tool-ok:** Változás → teljes újragenerálás

**Loom:**
```bash
# Csak az érintett dokumentumok frissülnek
/loom derive --level L2 --input modified-ac.md

# Validation azonnal jelzi, ha valami elromlott
/loom validate --check traceability
```

---

## Összehasonlító Táblázat

| Tulajdonság | Más AI Tool-ok | Loom |
|-------------|----------------|------|
| Döntéshozatal | Implicit, rejtett | **Explicit SI (87 pont)** |
| Tanulás | Session-alapú | **Projekt-szintű (Self-Learning)** |
| Dokumentáció | Generálás | **Deriválás (26x expansion)** |
| Traceability | Nincs | **100% (audit-ready)** |
| Full-stack | Külön világok | **Közös tudásbázis** |
| Tesztek | Happy path | **33% negatív** |
| Patterns | Copy-paste | **Reference library** |
| Változáskezelés | Újragenerálás | **Inkrementális** |

---

## Business Model

**1. Open Core**
- Loom commands: MIT license (adoption)
- SI Catalog + RAG engine: Premium

**2. Enterprise**
- Self-Learning System (projekt-specifikus tudás)
- SI Governance (team decision management)
- Compliance export (audit trail)

**3. Marketplace**
- Domain-specific SI catalogs (FinTech, HealthTech, etc.)
- Pre-built spec templates

---

## Kockázatok és Válaszok

| Kockázat | Válasz |
|----------|--------|
| "AI elég okos lesz spec nélkül" | Nem. Hallucination + implicit döntések. Enterprise-nak traceable kell. |
| "Nagy AI cégek megcsinálják" | Ők a modellt építik, nem a workflow-t. Különböző réteg. |
| "Fejlesztők nem akarnak spec-et" | De akarnak **működő kódot első próbálkozásra**. Loom = kevesebb iteráció. |

---

## Ask

**Seed round: $X**

Felhasználás:
1. **Multi-developer validation** - Csapat workflow tesztelése
2. **Code generation pipeline** - Spec → Working Code automated
3. **Enterprise pilot** - 2-3 design partner
4. **Team** - 2 senior engineer

---

## Záró Gondolatok

**Egy mondat:**

> **"A Loom az AI coding forradalom hiányzó rétege - a strukturált specifikáció, ami nélkül az AI csak gyorsan ír rossz kódot."**

**Megkülönböztető összefoglaló:**

> **"A Loom az egyetlen tool, ahol az AI explicit döntéseket hoz, tanul a projektből, és garantáltan traceable, tesztelhető specifikációt derivál - nem csak 'ír valamit'."**

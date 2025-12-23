# AI-DOP Competitive Landscape & Strategic Positioning

> Dokumentum a versenytársakról és az AI-DOP stratégiai pozícionálásáról.

**Létrehozva:** 2025-12-23

---

## Executive Summary

**AI-DOP nem versenyez közvetlenül senkivel** - új kategóriát teremt:
- Nem IDE (nem versenyez Cursor/Windsurf/Claude Code-dal)
- Nem code completion (nem versenyez Copilot-tal)
- Nem hagyományos requirements tool (nem versenyez Jira/DOORS-szal)

**Pozíció:** AI-native Documentation Derivation Platform

---

## Stratégia: Blue Ocean

**Blue Ocean Strategy** = Új piacot/kategóriát teremteni a létező, zsúfolt piacok helyett.

| Stratégia | Leírás | AI-DOP |
|-----------|--------|--------|
| **Red Ocean** | Létező piacon versenyezni, véres harc | ❌ Nem ez |
| **Blue Ocean** | Új kategóriát teremteni, nincs közvetlen verseny | ✅ Ez |

```
Red Ocean (létező piacok):

  IDE-k                    AI Coding              Requirements
  ┌──────────┐            ┌──────────┐            ┌──────────┐
  │ VS Code  │            │ Copilot  │            │ Jira     │
  │ Cursor   │            │ Cursor AI│            │ DOORS    │
  │ Windsurf │            │ Devin    │            │ Jama     │
  └──────────┘            └──────────┘            └──────────┘
       🦈🦈🦈                  🦈🦈🦈                  🦈🦈🦈
    (sok versenyző)        (sok versenyző)        (sok versenyző)


Blue Ocean (új kategória):

  AI Documentation Derivation
  ┌─────────────────────────────────────────┐
  │                                         │
  │              🐟 AI-DOP                  │
  │                                         │
  │   L0→L1→L2→L3 + SI + Traceability      │
  │                                         │
  └─────────────────────────────────────────┘
            (nincs közvetlen versenyző)
```

---

## Landscape Elemzés

### 1. Requirements Management Tools (Hagyományos)

| Tool | Mit csinál | AI-DOP különbség |
|------|------------|------------------|
| **Jira** | Issue tracking, workflow | ❌ Nincs deriválás, nincs AI |
| **Azure DevOps** | Work items, boards | ❌ Nincs AI-native deriválás |
| **IBM DOORS** | Requirements management | ❌ Manuális, enterprise drága |
| **Jama Connect** | Traceability | ❌ Manuális, nincs AI deriválás |
| **Polarion** | ALM, requirements | ❌ Komplex, nincs AI |

**Összefoglaló:** Hagyományos, nem AI-native. Manuális munka kell.

### 2. AI Documentation Tools

| Tool | Mit csinál | AI-DOP különbség |
|------|------------|------------------|
| **Mintlify** | API docs from code | ❌ Code→Docs (fordított irány) |
| **ReadMe** | API documentation | ❌ Csak API docs |
| **Swimm** | Code documentation | ❌ Code→Docs, nem spec→code |
| **GitBook** | Documentation hosting | ❌ Nincs deriválás |

**Összefoglaló:** Code→Docs irány. AI-DOP: Spec→Code irány.

### 3. AI Coding Assistants

| Tool | Mit csinál | AI-DOP különbség |
|------|------------|------------------|
| **GitHub Copilot** | Code completion | ❌ Nincs spec deriválás |
| **Cursor** | AI-powered IDE | ❌ Nincs doc pipeline |
| **Windsurf** | AI-powered IDE | ❌ Nincs doc pipeline |
| **Claude Code** | AI coding CLI | ❌ Nincs doc pipeline |
| **Devin** | AI software engineer | ❌ Code-fókusz, not docs |
| **Copilot Workspace** | Issue→PR | ⚠️ Legközelebbi, de nincs L0→L3 |

**Összefoglaló:** Code generálás fókusz. Nincs strukturált dokumentáció deriválás.

### 4. Product Management Tools

| Tool | Mit csinál | AI-DOP különbség |
|------|------------|------------------|
| **Productboard** | Product mgmt + AI | ❌ Product szint, nem tech spec |
| **Userdoc.fyi** | AI user stories | ⚠️ Csak L0 szint, nincs deriválás |
| **FeatureMap** | Product specs | ❌ Nincs L0→L3 pipeline |
| **Notion AI** | General docs + AI | ❌ Generic, nincs spec struktúra |

**Összefoglaló:** Product management szint. Nem technikai deriválás.

---

## AI-DOP Egyedi Értékei

| Feature | Hagyományos Tools | AI Coding | AI-DOP |
|---------|-------------------|-----------|--------|
| **L0→L1→L2→L3 deriválás** | ❌ | ❌ | ✅ |
| **Structured Interview** | ❌ | ❌ | ✅ |
| **Bidirectional Traceability** | ⚠️ Manuális | ❌ | ✅ AI-validated |
| **TDAI (anti-hallucination)** | ❌ | ❌ | ✅ |
| **RAG-enhanced derivation** | ❌ | ❌ | ✅ |
| **Decision persistence** | ❌ | ❌ | ✅ |
| **Checklist-driven analysis** | ⚠️ Manuális | ❌ | ✅ AI-driven |

---

## Legközelebbi "Versenytársak"

### 1. GitHub Copilot Workspace

**Mit csinál:** Issue → Plan → Code → PR

**Hasonlóság:** Van plan step
**Különbség:**
- Nincs strukturált L0→L3 dokumentáció
- Nincs Structured Interview
- Nincs traceability management
- Nincs TDAI

### 2. IBM Engineering Requirements (DOORS Next)

**Mit csinál:** Enterprise requirements management

**Hasonlóság:** Traceability, requirements
**Különbség:**
- Nem AI-native
- Manuális deriválás
- Enterprise pricing ($$$)
- Komplex setup

### 3. Custom GPT/Claude Prompts (DIY)

**Mit csinál:** Egyedi promptok spec írásra

**Hasonlóság:** AI-t használ dokumentációra
**Különbség:**
- Nincs integrált pipeline
- Nincs persistence (decisions.md)
- Nincs RAG
- Minden projekt újrakezdés

---

## Pozícionálás

### Nem Vagyunk

| Kategória | Példák | AI-DOP |
|-----------|--------|--------|
| IDE | VS Code, Cursor | ❌ Nem IDE |
| Code completion | Copilot | ❌ Nem code completion |
| Requirements tool | Jira | ❌ Nem issue tracker |
| Documentation hosting | GitBook | ❌ Nem hosting |

### Ami Vagyunk

**AI-native Documentation Derivation Platform**

- Input: L0 (human-written specs)
- Process: AI derivation + Structured Interview + RAG
- Output: L1, L2, L3 documents + Traceability
- Integration: Works WITH any IDE

---

## Stratégiai Előnyök

### 1. Complementary, Not Competitive

```
┌─────────────────────────────────────────────────────────────┐
│                    User's Environment                        │
│                                                             │
│  ┌─────────────────────┐     ┌─────────────────────┐       │
│  │  IDE                │     │  AI-DOP             │       │
│  │  (Cursor/Windsurf/  │ ←──→│  (Documentation)    │       │
│  │   Claude Code)      │     │                     │       │
│  │                     │     │                     │       │
│  │  - Code editing     │     │  - L0→L3 derivation│       │
│  │  - Completion       │     │  - Interview        │       │
│  │  - Chat             │     │  - Traceability     │       │
│  └─────────────────────┘     └─────────────────────┘       │
│                                                             │
│            Együtt működnek, nem versenyeznek               │
└─────────────────────────────────────────────────────────────┘
```

### 2. Platform Agnostic

- CLI: Mindenhol működik
- MCP: Claude Code integráció
- VS Code Extension: Cursor, Windsurf, VS Code

### 3. Unique Value Proposition

Amit senki más nem csinál:
- Teljes L0→L3 pipeline
- Explicit decision tracking
- AI-validated traceability
- Anti-hallucination (TDAI)

---

## Kockázatok

| Kockázat | Valószínűség | Hatás | Mitigáció |
|----------|--------------|-------|-----------|
| IDE-k beépítik a funkciót | Közepes | Magas | Gyors execution, mélyebb integráció |
| Copilot Workspace fejlődik | Közepes | Közepes | Fókusz: enterprise, traceability |
| Enterprise tools AI-t adnak | Alacsony | Közepes | Agility, modern UX |

---

## Összefoglalás

**AI-DOP = Blue Ocean**

- Új kategória: AI Documentation Derivation
- Nincs közvetlen versenytárs
- Kiegészíti az IDE-ket, nem versenyez velük
- Egyedi: L0→L3 + SI + Traceability + TDAI + RAG

---

## Kapcsolódó

- [IDE Integration Strategy](./ide-integration-strategy.md)
- [API Architecture](./api-based-derivation-architecture.md)
- [Payment Models](./api-based-payment-models.md)

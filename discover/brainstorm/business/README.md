---
title: "Loom Business Strategy"
status: draft
created: 2025-12-19
updated: 2025-12-26
consolidated-from:
  - business-strategy-and-defensible-moats.md
  - competitive-landscape.md
  - investor-pitch.md
see-also:
  - ../platform/platform-strategy.md  # Aktuális architektúra és fizetési modell
---

# Loom Business Strategy

Összevont üzleti stratégia dokumentum.

---

## 1. Pozícionálás

### 1.1 A Probléma: Black Box AI Kód

**"Az AI 10x gyorsabban ír kódot, de a projektek nem lesznek 10x gyorsabban készen."**

| Black Box (AI nélkül Loom) | Glass Box (Loom-mal) |
|----------------------------|----------------------|
| "Működik" - de miért így? | Traceable to US-003 |
| Ki döntötte? | SI-AUTH-1: JWT |
| Hol a spec? | BR-QUOTE-001 enforced |
| Hogyan teszteljük? | TC-003-1 covers it |

A szűk keresztmetszet áttolódott: Spec (10%) → Kód (80%) → Test **helyett** Spec (??) → Kód (10%) → Test

### 1.2 Blue Ocean Strategy

**Loom nem versenyez közvetlenül senkivel** - új kategóriát teremt:

| Mi NEM vagyunk | Mi VAGYUNK |
|----------------|------------|
| IDE (Cursor, Windsurf) | AI-native Documentation Derivation Platform |
| Code completion (Copilot) | Strukturált spec deriválás |
| Requirements tool (Jira, DOORS) | L0→L1→L2→L3 pipeline |
| Documentation hosting (GitBook) | Integrated with any IDE |

### 1.3 Versenytárs Elemzés

| Kategória | Példák | Loom előnye |
|-----------|--------|-------------|
| **Hagyományos Req Tools** | Jira, DOORS, Jama | AI-native, nincs manuális deriválás |
| **AI Documentation** | Mintlify, ReadMe | Spec→Code irány (nem Code→Docs) |
| **AI Coding** | Copilot, Cursor, Devin | Mi adjuk a SPEC-et amit generálnak |
| **Product Mgmt** | Productboard, Notion AI | Teljes L0→L3 pipeline |

**Legközelebbi:** GitHub Copilot Workspace - de nincs L0→L3, SI, traceability, TDAI

---

## 2. Értékajánlat

### 2.1 Két Pozícionálási Opció

| | Developer Tool (eredeti) | AI-Powered Services (erősebb) |
|---|---|---|
| **Target** | Dev teams, Tech Leads | CEOs, Operations Managers |
| **Value** | Faster docs, 0% drift | 10x cheaper, 10x faster custom software |
| **Challenge** | Hard to measure ROI | Measurable ROI in months |
| **Position** | Dev productivity (crowded) | Blue ocean |

### 2.2 Költség Összehasonlítás

| | Traditional Custom Dev | Loom-Powered |
|---|---|---|
| **Timeline** | 6-12 hónap | 2-8 hét |
| **Team** | 5-10 ember | 1-2 ember + AI |
| **Cost** | $500K - $2M | $50K - $200K |
| **Risk** | 70% túllépi budget-et | L0 validált, TDAI, 0% drift |

### 2.3 Validált Eredmények (PoC)

| Metrika | Eredmény |
|---------|----------|
| Content expansion | **26x** (53 → 1390 sor) |
| Negatív teszt arány | **33%** (iparági avg: 10%) |
| Format compliance | **100%** |
| Traceability | **100%** |

---

## 3. Egyedi Differenciátorok

| # | Feature | Probléma másoknál | Loom megoldás |
|---|---------|-------------------|---------------|
| 1 | **Structured Interview** | AI implicit döntéseket hoz | 87 explicit döntési pont, rögzítve YAML-ban |
| 2 | **Self-Learning** | Minden session nulláról | Projekt-szintű RAG, korábbi döntések ismerete |
| 3 | **Deriválás** | AI "ír valamit" | Logikai levezetés (26x expansion) |
| 4 | **Traceability** | Tesztek "lebegnek" | 100% requirement→test kapcsolat |
| 5 | **Full-Stack Coherence** | Backend/Frontend külön | Közös tudásbázis, SI döntések |
| 6 | **TDAI** | 90%+ pozitív teszt | 33% negatív, edge case-ek |
| 7 | **Cross-cutting Patterns** | Copy-paste | Reference library (14 pattern) |
| 8 | **Incremental** | Teljes újragenerálás | Csak érintett dokumentumok |

---

## 4. Defensible Moats

### 4.1 Három Réteg Ahol Ember Kell

| Réteg | Mit | Miért nem AI | Defensibility |
|-------|-----|--------------|---------------|
| **L0 Creation** | Domain expertise, problem framing, solution design | Évek tapasztalata, implicit tudás, bizalom | LEGMAGASABB |
| **Review & Validation** | L1/L2/L3 review, architecture decisions | Önvalidáció lehetetlen, accountability | Magas |
| **Consulting & Advisory** | ROI, change management, strategy | Kapcsolat, empátia, felelősségvállalás | LEGMAGASABB (hosszútáv) |

### 4.2 AI Evolúció Szcenáriók

| Szcenárió | Válasz |
|-----------|--------|
| AI sokkal jobb lesz | Emberek L0-ra és advisory-ra fókuszálnak, magasabb margin |
| AI L0-t is tudja | Validáció és döntéshozatal marad emberi |
| Full AGI | Jogi accountability, bizalom, kapcsolatok maradnak |

**Ami mindig emberi marad:** Domain expertise, accountability, trust, strategic thinking, empathy

---

## 5. Üzleti Modell

### 5.1 Service Tiers

A Loom-ot szolgáltatásként is értékesíthetjük, nem csak eszközként. Az ügyfél nem a CLI-t kapja, hanem a Loom-mal készült szoftvert és tanácsadást.

| Tier | Ár | Tartalom | Human % |
|------|-----|----------|---------|
| **Discovery** | $5K-$15K | Stakeholder interjúk, pain point elemzés, ROI kalkuláció | 100% |
| **Implementation** | $50K-$200K | L0 creation, L1-L3 deriválás, deployment | 30% |
| **Managed Services** | $10K-$50K/hó | Optimalizálás, strategic advisory, coaching | 50% |

**Conversion:** Discovery → 80% Implementation → 60% Managed

### 5.2 Architektúra & Fizetési Modell

> **Aktuális irány:** Dumb CLI + Smart SaaS
> Részletek: `../platform/platform-architecture.md` és `platform-business-model.md`

```
┌─────────────────────────────────────────────────────┐
│  loom-cli (INGYENES, thin client)                   │
│  └─→ Orchestration only, NO embedded prompts        │
│  └─→ Fetches prompts from SaaS API                  │
│  └─→ Calls claude -p (user's subscription)          │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│  Loom SaaS (FIZETŐS, itt az IP)                     │
│  └─→ Prompts, checklists, templates                 │
│  └─→ Knowledge base, best practices                 │
│  └─→ Document storage, collaboration                │
└─────────────────────────────────────────────────────┘
```

| Aspektus | Érték |
|----------|-------|
| **IP védelem** | ~95% (prompts server-side) |
| **LLM költség** | User fizeti (Claude subscription) |
| **Monetizáció** | SaaS subscription |

| Tier | Ár | Features |
|------|-----|----------|
| **Free** | $0 | 10 derivation/hó, basic prompts |
| **Pro** | $19/hó | Unlimited, latest prompts, advanced templates |
| **Team** | $49/hó/user | Pro + collaboration, dashboard |
| **Enterprise** | Custom | Team + SSO, audit, on-prem |

### 5.3 Long-term Strategy

| Időtáv | Fókusz | Moat |
|--------|--------|------|
| **1-2 év** | 3-5 vertical domain expertise | Deep industry knowledge |
| **3-5 év** | Platform + Marketplace + Certified Consultants | Ecosystem, network effects |
| **5-10 év** | Strategic advisory network | Trust, relationships |

---

## 6. Investor Pitch Összefoglaló

### One-liner

> **"A Loom az AI coding forradalom hiányzó rétege - a strukturált specifikáció, ami nélkül az AI csak gyorsan ír rossz kódot."**

### Miért Most?

1. **AI Coding tool robbanás** - Cursor/Windsurf/Claude Code adoption
2. **Agentic AI** - Komplex task-ok lehetségesek, de kell jó input
3. **Enterprise adoption** - Compliance, governance igény

### Kockázatok és Válaszok

| Kockázat | Válasz |
|----------|--------|
| "AI elég okos lesz spec nélkül" | Hallucination + implicit döntések. Enterprise-nak traceable kell. |
| "Nagy AI cégek megcsinálják" | Ők modellt építenek, nem workflow-t. Más réteg. |
| "Fejlesztők nem akarnak spec-et" | De akarnak működő kódot első próbára. Loom = kevesebb iteráció. |

---

## Kapcsolódó Dokumentumok

- Platform architecture: `../platform/platform-architecture.md`
- Platform business model: `platform-business-model.md`
- Core concepts: `../core-concepts/`
- Derivation: `../derivation/`

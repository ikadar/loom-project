# Loom Project Roadmap

> Legfelső szintű roadmap a Multi-level Double Diamond metodológiához.
> Minden milestone egy teljes Double Diamond iteráció (Discover → Define → Develop → Deliver).

---

## Részletes Tervek

| Milestone | Részletes terv |
|-----------|----------------|
| M2 | [M2-business-model.md](./M2-business-model.md) |
| M3 | [M3-product-spec.md](./M3-product-spec.md) |
| M4 | [M4-mcp-server-cli.md](./M4-mcp-server-cli.md) |
| M5 | [M5-mvp-launch.md](./M5-mvp-launch.md) |

---

## Milestone Áttekintés

```
M1 ──► M2 ──► M3 ──► M4 ──► M5 ──► M6 ──► M7
│      │      │      │      │      │      │
▼      ▼      ▼      ▼      ▼      ▼      ▼
Vízió  Üzleti Termék Platform MVP   Beta   Ops
       Modell Spec   Impl.   Launch
```

---

## M1: Vízió & Validáció

**Cél:** A Loom koncepció validálása, alapvető irány meghatározása

| Szint | Fókusz |
|-------|--------|
| 💰 Business | Piaci igény validálása, versenytárs elemzés |
| 📦 Product | 5 pillér koncepció kidolgozása |
| 🔧 Technical | PoC-ok (deriválás, traceability) |

**Deliver:**
- [x] PoC eredmények dokumentálva
- [x] 5 pillér definiálva
- [x] Opus értékelések (validáció)

**Státusz:** ✅ KÉSZ

---

## M2: Üzleti Modell

**Cél:** Fenntartható üzleti modell és pozícionálás

| Szint | Fókusz |
|-------|--------|
| 💰 Business | Pricing, monetizáció, Anthropic policy, tudásbázis stratégia |
| 📦 Product | Value proposition finomítás, tudásterületek |
| 🔧 Technical | Architektúra irány (CLI + SaaS + RAG + Website) |

**Deliver:**
- [x] Kettős pozícionálás (Free CLI + Paid SaaS)
- [x] Tier struktúra (Free/Pro/Team/Enterprise)
- [ ] Anthropic policy megerősítés
- [x] Platform architektúra döntés
- [ ] Tudásbázis stratégia (tartalom, licensz, IP)
- [ ] Website stratégia (domain, hosting)

**Státusz:** 🔄 FOLYAMATBAN (policy tisztázás pending)

---

## M3: Termék Specifikáció

**Cél:** Részletes termék specifikáció a fejlesztéshez

| Szint | Fókusz |
|-------|--------|
| 💰 Business | Go-to-market stratégia, tudás mint érték |
| 📦 Product | Feature prioritizálás, MVP scope, tudáskategóriák, website UX |
| 🔧 Technical | API design, adatmodell, RAG specifikáció |

**Deliver:**
- [ ] MVP feature lista (prioritizált)
- [ ] User journey dokumentáció
- [ ] API specifikáció
- [ ] Adatmodell
- [ ] Tudásbázis specifikáció (formátum, kategóriák, minőség)
- [ ] Website specifikáció (struktúra, content plan)

**Státusz:** ⏳ PENDING

---

## M4: Platform Implementáció

**Cél:** Core technikai infrastruktúra

| Szint | Fókusz |
|-------|--------|
| 💰 Business | - |
| 📦 Product | - |
| 🔧 Technical | MCP Server, CLI, SaaS Backend, RAG, Website |

**Deriválási szintek (MVP scope):**
```
L0 (User Stories)
    │
    ▼ analyze + interview
L1 (Acceptance Criteria, Business Rules)
    │
    ▼ derive-l2
L2 (Test Cases, Technical Specs)
    │
    ▼ derive-l3
L3 (Implementation Skeletons, API Specs)
```

**Deliver:**
- [ ] `@loom/mcp-server` npm package
- [ ] `loom_validate`, `loom_derive` tools (teljes L0→L1→L2→L3)
- [ ] `loom-cli` Go binary (teljes deriválási folyamat)
- [ ] SaaS API endpoints (prompts, license)
- [ ] RAG pipeline (embedding, vector DB, retrieval)
- [ ] Website (landing, docs, pricing, auth)

**Státusz:** ⏳ PENDING

---

## M5: MVP Launch

**Cél:** Első publikus verzió

| Szint | Fókusz |
|-------|--------|
| 💰 Business | Pricing aktiválás, első ügyfelek |
| 📦 Product | Core features működnek, kezdeti tudáskorpusz |
| 🔧 Technical | Production deployment, RAG live, Website live |

**Deliver:**
- [ ] Public npm/brew release
- [ ] loom.dev Website live
- [ ] Free tier működik
- [ ] Dokumentáció (user-facing)
- [ ] 3-5 beta user feedback
- [ ] MVP tudáskorpusz deployed

**Státusz:** ⏳ PENDING

---

## M6: Beta & Iteration

**Cél:** Feedback alapú iteráció, Pro tier

| Szint | Fókusz |
|-------|--------|
| 💰 Business | Pro tier launch, fizetős ügyfelek |
| 📦 Product | Feature bővítés feedback alapján |
| 🔧 Technical | Performance, reliability |

**Deliver:**
- [ ] Pro tier aktív
- [ ] 10+ aktív felhasználó
- [ ] Prompt/checklist iterációk
- [ ] Analytics, monitoring

**Státusz:** ⏳ PENDING

---

## M7: Continuous Operations

**Cél:** Stabil üzemeltetés, folyamatos fejlesztés

| Szint | Fókusz |
|-------|--------|
| 💰 Business | Team/Enterprise tier, growth |
| 📦 Product | Roadmap-driven fejlesztés |
| 🔧 Technical | Scaling, enterprise features |

**Deliver:**
- [ ] Team tier
- [ ] SSO, audit features
- [ ] Knowledge base bővítés
- [ ] Partner/integration program

**Státusz:** ⏳ PENDING

---

## Milestone Double Diamond Template

Minden milestone-hoz használható struktúra:

```
MILESTONE: [Név]
├── DISCOVER (diverge)
│   ├── Kutatás, opciók feltárása
│   ├── Stakeholder interjúk
│   └── Competitive analysis
│
├── DEFINE (converge)
│   ├── Probléma/megoldás definiálása
│   ├── Scope rögzítése
│   └── Success criteria
│
├── DEVELOP (diverge)
│   ├── Megoldási opciók
│   ├── Prototípusok
│   └── Technical spikes
│
└── DELIVER (converge)
    ├── Implementáció
    ├── Tesztelés
    └── Release/dokumentáció
```

---

## Aktuális Fókusz

```
         M1        M2        M3        M4        M5        M6        M7
         ✅        🔄        ⏳        ⏳        ⏳        ⏳        ⏳
       Vízió    Üzleti    Termék     MCP      MVP      Beta      Ops
               Modell     Spec     Server   Launch
                  ▲
                  │
              ITT VAGYUNK
              (Define fázis)
```

---

*Utolsó frissítés: 2025-12-26*

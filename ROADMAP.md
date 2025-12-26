# Loom Project Roadmap

> Legfelső szintű roadmap a Multi-level Double Diamond metodológiához.
> Minden milestone egy teljes Double Diamond iteráció (Discover → Define → Develop → Deliver).

---

## Milestone Áttekintés

```
M1 ──► M2 ──► M3 ──► M4 ──► M5 ──► M6 ──► M7
│      │      │      │      │      │      │
▼      ▼      ▼      ▼      ▼      ▼      ▼
Vízió  Üzleti Termék MCP    MVP    Beta   Ops
       Modell Spec   Server Launch
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
| 💰 Business | Pricing, monetizáció, Anthropic policy |
| 📦 Product | Value proposition finomítás |
| 🔧 Technical | Architektúra irány (CLI + SaaS) |

**Deliver:**
- [x] Kettős pozícionálás (Free CLI + Paid SaaS)
- [x] Tier struktúra (Free/Pro/Team/Enterprise)
- [ ] Anthropic policy megerősítés
- [x] Platform architektúra döntés

**Státusz:** 🔄 FOLYAMATBAN (policy tisztázás pending)

---

## M3: Termék Specifikáció

**Cél:** Részletes termék specifikáció a fejlesztéshez

| Szint | Fókusz |
|-------|--------|
| 💰 Business | Go-to-market stratégia |
| 📦 Product | Feature prioritizálás, MVP scope |
| 🔧 Technical | API design, adatmodell |

**Deliver:**
- [ ] MVP feature lista (prioritizált)
- [ ] User journey dokumentáció
- [ ] API specifikáció
- [ ] Adatmodell

**Státusz:** ⏳ PENDING

---

## M4: MCP Server & CLI

**Cél:** Core technikai infrastruktúra

| Szint | Fókusz |
|-------|--------|
| 💰 Business | - |
| 📦 Product | - |
| 🔧 Technical | MCP Server implementáció, CLI wrapper |

**Deliver:**
- [ ] `@loom/mcp-server` npm package
- [ ] `loom_validate`, `loom_derive` tools
- [ ] `loom-cli` Go binary
- [ ] SaaS API endpoints (prompts, license)

**Státusz:** ⏳ PENDING

---

## M5: MVP Launch

**Cél:** Első publikus verzió

| Szint | Fókusz |
|-------|--------|
| 💰 Business | Pricing aktiválás, első ügyfelek |
| 📦 Product | Core features működnek |
| 🔧 Technical | Production deployment |

**Deliver:**
- [ ] Public npm/brew release
- [ ] loom.dev SaaS live
- [ ] Free tier működik
- [ ] Dokumentáció (user-facing)
- [ ] 3-5 beta user feedback

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

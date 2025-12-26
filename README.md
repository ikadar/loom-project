# Loom Project

A Loom projekt üzleti, termék és technikai gondolkodásának tárhelye.

## Multi-level Double Diamond

A projekt három szinten alkalmazza a Double Diamond metodológiát:

```
                DISCOVER      DEFINE       DEVELOP      DELIVER
                (diverge)    (converge)   (diverge)   (converge)

💰 BUSINESS     ◇────────────────◇────────────────◇────────────────◇
                Piackutatás,     Üzleti modell,   Go-to-market,    Launch,
                versenytársak    pricing, moats   partnerségek     revenue

📦 PRODUCT      ◇────────────────◇────────────────◇────────────────◇
                User needs,      Core concepts,   Product roadmap, Release,
                pain points      value prop       feature specs    feedback

🔧 TECHNICAL    ◇────────────────◇────────────────◇────────────────◇
                Tech opciók,     Architektúra,    Implementáció,   Deploy,
                patterns         design specs     kódolás          ops
```

### Szintek

| Szint | Kérdés | Tartalom |
|-------|--------|----------|
| **Business** | MIÉRT? Hogyan fenntartható? | Üzleti modell, pricing, pozícionálás, compliance |
| **Product** | MIT? Milyen értéket ad? | Core concepts (5 pillér), deriválás, metodológia |
| **Technical** | HOGYAN? Hogyan épül meg? | Architektúra, MCP Server, implementáció |

### Fontos jellemzők

1. **Párhuzamos haladás** - A szintek különböző fázisokban lehetnek egyidejűleg
2. **Függőségek** - Magasabb szint döntései befolyásolják az alacsonyabbakat
3. **Iteráció** - Minden szinten lehet visszalépni korábbi fázisba
4. **Compliance** - A Business szint része (üzleti kockázatként kezelve)

## Mappastruktúra

```
loom-project/
├── discover/                    # DISCOVER fázis (minden szint)
│   ├── brainstorm/
│   │   ├── business/            # 💰 Üzleti ötletek
│   │   ├── core-concepts/       # 📦 Termék koncepciók
│   │   ├── derivation/          # 📦 Deriválási rendszer
│   │   └── platform/            # 🔧 Technikai architektúra
│   ├── evaluations/             # AI értékelések
│   └── poc-runs/                # PoC eredmények
├── define/                      # DEFINE fázis (minden szint)
├── develop/                     # DEVELOP fázis (minden szint)
└── deliver/                     # DELIVER fázis (minden szint)
```

## Aktuális státusz

```
         DISCOVER    DEFINE     DEVELOP    DELIVER

💰 BIZ:  ████████    ████░░░░   ░░░░░░░░   ░░░░░░░░
📦 PROD: ████████    ████████   ░░░░░░░░   ░░░░░░░░
🔧 TECH: ████████    ████░░░░   ░░░░░░░░   ░░░░░░░░
```

---

*Ez a dokumentum a Multi-level Double Diamond koncepció draft-ja. Véglegesítés után a CLAUDE.md-be kerül.*

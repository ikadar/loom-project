# Loom Architektúra Evolúció

> Összefoglaló a "stop and check" fázisban folytatott architekturális beszélgetésről.

## Kontextus

A PoC (L0→L1 deriválás) sikeres befejezése után felmerült a kérdés: hogyan fejlődhet tovább a Loom tooling architektúrája? A beszélgetés során több kulcskérdést tisztáztunk.

---

## 1. Repository Struktúra

### Probléma
A PoC a spec repository-ban készült (`specs-for-ai`), de a tooling és a specifikáció logikailag külön entitások.

### Döntés: Külön Tooling Repository (Option B)

```
specs-for-ai/           ← AI-PDS specifikáció
loom-tooling/           ← Loom tooling (skills, MCP server, stb.)
my-project/             ← Konkrét projekt, ami használja a Loom-ot
```

### Indoklás
- **Separation of Concerns**: A spec és a tooling különböző életciklusú
- **Újrafelhasználhatóság**: A tooling több projektben is használható
- **Verziókezelés**: Külön verziókövetés a tooling-ra

---

## 2. Dual-Agent Architektúra

### Koncepció
Két Claude Code agent fut párhuzamosan:

```
┌─────────────────────────────────────────────────────────────┐
│                    FEJLESZTŐI KÖRNYEZET                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────┐    ┌─────────────────────┐         │
│  │   Tooling Agent     │    │   Project Agent     │         │
│  │   (loom-tooling/)   │    │   (my-project/)     │         │
│  │                     │    │                     │         │
│  │   - Skill fejlesztés│    │   - /loom-derive    │         │
│  │   - MCP server dev  │    │   - Dokumentáció    │         │
│  │   - Template edit   │    │   - Implementáció   │         │
│  └─────────────────────┘    └─────────────────────┘         │
│            │                          │                      │
│            └──────────┬───────────────┘                      │
│                       │                                      │
│              additionalDirectories                           │
│              (tooling látható a project-ből)                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Előnyök
- Minden agent a saját kontextusában dolgozik
- A tooling agent tesztelheti a skill-eket a tooling repo-ban
- A project agent használja a kész skill-eket

---

## 3. Projekt-szintű Integráció

### Probléma
Szükség van-e globális skill-ekre (`~/.claude/skills/`)?

### Döntés: Projekt-szintű integráció preferált

```
my-project/
├── .claude/
│   └── skills/
│       └── loom-derive.md    ← Projekt-specifikus skill
├── .mcp.json                  ← Projekt-specifikus MCP config
└── ...
```

### Indoklás
- Nem minden Claude agent használja a Loom-ot
- Verziókövetés: a skill verzió a projekthez kötött
- Tisztább separation: globális = személyes preferenciák, projekt-szintű = projekt tooling

---

## 4. MCP (Model Context Protocol) Megértése

### Mi az MCP?
Standardizált protokoll, amivel külső szerverek tool-okat, resource-okat és prompt-okat biztosítanak Claude-nak.

### MCP Server Típusok

| Típus | Kommunikáció | Használat |
|-------|--------------|-----------|
| **Stdio** | stdin/stdout | Lokális CLI tool |
| **HTTP/SSE** | HTTP kérések | Távoli szolgáltatás |

### MCP Képességek

```
┌─────────────────────────────────────────────────────────────┐
│                      MCP SERVER                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  TOOLS (actions)         RESOURCES (data)    PROMPTS        │
│  ├─ derive_document      ├─ loom://rules     ├─ /loom-init  │
│  ├─ validate_links       ├─ loom://templates │              │
│  └─ check_coverage       └─ loom://examples  │              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Hol Fut az AI?

**Kritikus felismerés**: A Loom MCP Server-nek MAGÁNAK kell Claude API-t hívnia.

```
┌──────────────┐     ┌──────────────────┐     ┌──────────────┐
│ Claude Code  │────►│  Loom MCP Server │────►│  Claude API  │
│ (orchestrator)│     │  (derive logic)  │     │  (derivation)│
└──────────────┘     └──────────────────┘     └──────────────┘
       │                      │                       │
       │ "derive L0→L1"       │ constructs prompt     │
       │                      │ + rules + templates   │
       │                      │ + input document      │
       │                      │                       │
       │                      │ calls API ───────────►│
       │                      │                       │
       │                      │◄─────── derived doc ──│
       │◄── returns result ───│                       │
```

A Claude API példány, amit az MCP szerver hív, NEM rendelkezik implicit tudással - a teljes kontextust (szabályok, template-ek, példák) a promptban kell átadni.

---

## 5. Tudásintegráció: RAG + Multi-Expert Pipeline

### Probléma
A domain modellezés (és más szakterületek) komoly elméleti tudásanyaggal rendelkeznek. Hogyan használjuk fel ezt a tudást a deriválásban?

### Megoldási Opciók

| Opció | Leírás | Előny | Hátrány |
|-------|--------|-------|---------|
| **Knowledge in Prompt** | Szabályok beégetése | Egyszerű | Limitált méret |
| **Knowledge Files** | Külső fájlok betöltése | Karbantartható | Statikus |
| **RAG** | Vektor DB, dinamikus retrieval | Skálázható | Komplex |
| **Multi-Expert** | Specializált lépések | Fókuszált | Több API hívás |

### Hibrid Architektúra (RAG + Multi-Expert)

**Kulcs felismerés**: A RAG és Multi-Expert Pipeline NEM zárják ki egymást - kombinálhatók!

```
┌─────────────────────────────────────────────────────────────────┐
│                    LOOM DERIVATION PIPELINE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────┐    ┌──────────────────┐                   │
│  │  Domain Modeling │    │   DDD RAG        │                   │
│  │     Expert       │◄───│   Knowledge Base │                   │
│  │  (Claude API)    │    │   (Evans, Vernon)│                   │
│  └────────┬─────────┘    └──────────────────┘                   │
│           │                                                      │
│           ▼                                                      │
│  ┌──────────────────┐    ┌──────────────────┐                   │
│  │   API Design     │    │   REST/API RAG   │                   │
│  │     Expert       │◄───│   Knowledge Base │                   │
│  │  (Claude API)    │    │   (RFC, OpenAPI) │                   │
│  └────────┬─────────┘    └──────────────────┘                   │
│           │                                                      │
│           ▼                                                      │
│  ┌──────────────────┐    ┌──────────────────┐                   │
│  │   Clean Code     │    │   Clean Code RAG │                   │
│  │     Expert       │◄───│   Knowledge Base │                   │
│  │  (Claude API)    │    │   (Martin, etc.) │                   │
│  └────────┬─────────┘    └──────────────────┘                   │
│           │                                                      │
│           ▼                                                      │
│      [Final Output]                                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Működési Elv

- **Multi-Expert Pipeline** = A folyamat struktúrája (lépések sorrendje, input/output lánc)
- **RAG** = Az egyes expertek tudásbázisa (specializált, dinamikus)

Minden expert:
1. RAG query alapján lekéri a releváns szakirodalmat
2. Prompt = deriválási szabályok + RAG eredmények + input dokumentum
3. Claude API hívás a speciális kontextussal
4. Output → következő expert inputja

---

## 6. RAG Tudásbázis: Meglévő Guidelines

### Felismerés

A `9300-guidelines/` mappa **már tartalmazza** a RAG PoC-hoz szükséges tudásbázist:

```
ai-pds-specification/9000-appendix/9300-guidelines/
├── 2231-service-boundaries-guidelines.md
├── 2232-aggregate-design-guidelines.md
├── 2233-event-message-design-guidelines.md
├── 2234-adr-guidelines.md
├── 2235-nfr-guidelines.md
├── 2236-sequence-design-guidelines.md
├── 2237-interface-contract-guidelines.md
├── 2238-dependency-graphs-guidelines.md
├── 2311-feature-definition-guidelines.md
├── 2321-implementation-guidelines.md
├── 2331-qa-guidelines.md
├── 2341-release-candidate-guidelines.md
├── 2401-deployment-guidelines.md
└── 17xx-loom-*.md (Loom-specifikus)
```

### Miért Ideális?

| Szempont | Értékelés |
|----------|-----------|
| **Mennyiség** | ~17 fájl, ~1000-1500 sor |
| **Struktúráltság** | Fejezetek, listák, egyértelmű |
| **Akcionálhatóság** | "Must contain", "When to create" |
| **Chunk-olhatóság** | ~50-100 sor/fájl, jól darabolható |
| **Relevancia** | Pontosan a deriváláshoz kellenek |
| **Kontroll** | Saját szabályok, nem külső forrás |

### Előny a Külső Forrásokkal Szemben

- **Loom-native** - A saját deriválási szabályaink
- **Kontrollált** - Tudjuk, mi van benne
- **Nincs copyright** - Saját tartalom
- **Mérhető** - Összehasonlítható RAG vs no-RAG
- **Bővíthető** - Később jöhet Evans, Vernon, stb.

---

## 7. Loom Architektúra Evolúció

### v1: PoC (Befejezett)
```
Skill + beégetett szabályok
```
- Claude Code skill markdown fájlban
- Deriválási szabályok a prompt-ban
- Egyszerű, gyors, validálásra alkalmas
- ✅ **Kész** - L0→L1 deriválás működik

### v1.5: RAG PoC (✅ Befejezett - 2024-12-20)
```
Skill + RAG (guidelines-ból)
```
- ✅ Meglévő guidelines mint tudásbázis (17 fájl → 457 chunk)
- ✅ LangChain + Chroma (lokális) + HuggingFace embeddings (ingyenes)
- ✅ Összehasonlítás: RAG vs no-RAG - jelentős minőségjavulás
- ✅ Claude Code futtatja a retrieval-t, használja kontextusként
- Implementáció: ~40 perc

**Eredmény:**
| RAG nélkül | RAG-gal |
|------------|---------|
| 3 szekció | 7 szekció (guidelines format) |
| Nincs invariants | 4 explicit invariant |
| Nincs indoklás | Entity/VO rationale |
| Ad-hoc struktúra | Guidelines-compliant |

### v2: Near-term
```
Multi-Expert Pipeline + Knowledge Files
```
- Külön MCP szerver
- Több lépésből álló pipeline
- Tudás külső fájlokban (nem beégetve)
- Jobb karbantarthatóság

### v3: Advanced
```
Multi-Expert Pipeline + RAG per Expert
```
- Vektor adatbázis a tudásanyagnak
- Minden expert a saját speciális RAG-jából húz
- Dinamikus, skálázható tudásintegráció
- Szakirodalomra alapozott döntések

---

## 8. Következő Lépések

1. ✅ ~~**RAG PoC**~~ - Guidelines-alapú RAG - Kész
2. ✅ ~~**Összehasonlítás**~~ - RAG vs no-RAG deriválás minősége - Kész
3. 🔲 **Tooling repository létrehozása** (`loom-tooling`)
4. 🔲 **PoC skill + RAG migrálása** a tooling repo-ba
5. 🔲 **MCP Server alapok** - egyszerű stdio server (v2 felé)
6. 🔲 **Multi-Expert Pipeline** - több lépéses deriválás (v3 felé)

---

## Összefoglaló

| Kérdés | Döntés | Státusz |
|--------|--------|---------|
| Repo struktúra | Külön tooling repo | 🔲 Pending |
| Agent architektúra | Dual-agent (tooling + project) | 🔲 Pending |
| Skill elhelyezés | Projekt-szintű, nem globális | ✅ Validált |
| RAG tudásbázis | Meglévő guidelines (9300-guidelines/) | ✅ 457 chunk |
| RAG működés | Claude Code + lokális retrieval | ✅ Validált |
| Tudásintegráció | RAG + Multi-Expert hibrid | 🔲 v3-ban |
| Evolúciós út | v1 ✅ → v1.5 ✅ → v2 (MCP) → v3 (Multi-Expert RAG) | In Progress |

### Kulcs Tanulság a RAG PoC-ból

A RAG nem "helyes választ" ad, hanem **indokolt döntést** segít:
- A kontextus biztosítja, hogy a döntés tudatos
- Guidelines struktúrát követ, nem ad-hoc
- Entity vs Value Object: mindkettő lehet helyes, de a RAG-gal **indokolt**

A "stop and check" fázis eredménye: tiszta architekturális vízió a Loom tooling fejlesztéséhez.

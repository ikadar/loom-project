# Design Patterns Analysis - Loom kontextusban

## Összefoglaló

Ez a dokumentum a design patternek Loom-beli kezelésének **elemzése és áttekintése**. A részletes specifikációk külön dokumentumokban találhatók.

---

## Kapcsolódó dokumentumok

| Dokumentum | Tartalom |
|------------|----------|
| [system-level-patterns.md](system-level-patterns.md) | Rendszer szintű patternek (Architecture Interview) |
| module-level-patterns.md | Modul szintű patternek (TODO) |

---

## Felismerések

### 1. Design vs Coding Patterns

| Típus | Leírás | Kérdés |
|-------|--------|--------|
| **Design Patterns** | Architekturális döntések (Repository, Factory, Domain Events) | **WHAT** - mit használunk |
| **Coding Patterns** | Nyelvi konvenciók (Go: Functional Options, TS: Barrel exports) | **HOW** - hogyan implementáljuk |

### 2. A patternek nem egy helyen dőlnek el

A derivációs lánc különböző szintjein különböző patternek relevánsak.

### 3. Rendszer vs Modul szintű patternek

| Scope | Döntés helye | Példák |
|-------|--------------|--------|
| **Rendszer szintű** | Architecture Interview (L1-L2 között) | Monolith/Microservices, CQRS, Clean Architecture |
| **Modul szintű** | L4 vagy implementáció során | Factory, Builder, State, Strategy |

---

## Pattern kategorizálás

### Rendszer szintű → Architecture Interview

Részletek: [system-level-patterns.md](system-level-patterns.md)

| Kategória | Patternek |
|-----------|-----------|
| **Deployment** | Monolith ↔ Modular Monolith ↔ Hybrid ↔ Microservices |
| **Code Organization** | Clean Architecture (default) |
| **Data Patterns** | CRUD / CQRS / Event Sourcing |
| **DDD Tactical** | Aggregate, Repository, Domain Events, Unit of Work |
| **GoF rendszer szinten** | Observer, Composite, Decorator, Facade, Chain of Responsibility |

### Modul szintű → L4 / Implementáció

Részletek: module-level-patterns.md (TODO)

| Kategória | Patternek |
|-----------|-----------|
| **GoF Creational** | Factory, Builder, Prototype |
| **GoF Structural** | Adapter, Bridge, Proxy (modul szinten) |
| **GoF Behavioral** | State, Strategy, Command (undo/redo), Memento |

---

## Összefoglaló diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ L0: Vision                                                      │
│   - NFR-ek → utalások architektúrára (scale, uptime, stb.)      │
├─────────────────────────────────────────────────────────────────┤
│ L1: Strategic Design + Interview                                │
│   - Domain model → Value Objects, Entities                      │
│   - Bounded contexts → szolgáltatás-határok előkészítése        │
├─────────────────────────────────────────────────────────────────┤
│ ★ ARCHITECTURE INTERVIEW (L1 és L2 között) ★                    │
│   - Deployment architecture (Monolith ↔ Microservices)          │
│   - Code organization (Clean Architecture)                      │
│   - Data patterns (CQRS, Event Sourcing)                        │
│   - Unit of Work szükségessége                                  │
│   → Részletek: system-level-patterns.md                         │
├─────────────────────────────────────────────────────────────────┤
│ L2: Tactical Design                                             │
│   - Aggregate design → Repository pattern automatikus           │
│   - Sequence design → Domain Events, Observer pattern           │
│   - State-ek az aggregate-ben → State pattern                   │
├─────────────────────────────────────────────────────────────────┤
│ L3: Operational Design                                          │
│   - Service boundaries → Adapter pattern (external APIs)        │
│   - Service decomposition → Facade ha komplex                   │
│   - A deployment architecture itt konkretizálódik               │
├─────────────────────────────────────────────────────────────────┤
│ L4: Implementation Design                                       │
│   - Module design → module organization, classification         │
│   - Factory/Builder → komplex creation                          │
│   - Decorator → cross-cutting concerns                          │
│   - Language-specific idioms                                    │
│   → Részletek: module-level-patterns.md (TODO)                  │
├─────────────────────────────────────────────────────────────────┤
│ Code: Implementation                                            │
│   - Coding patterns (error handling, context, stb.)             │
│   - Nyelvi idiómák                                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Fontos elvek

### 1. Tech Stack is deriválható

A tech stack **nem fix input**, hanem a requirement-ekből deriválható:

```
L0/L1 requirements → Architecture Interview → Tech stack ajánlás → User jóváhagyás
```

### 2. GoF patternek kontextus-függők

**Fontos felismerés:** A GoF patternek nem bináris módon (rendszer VAGY modul) kategorizálhatók, hanem **kontextus-függően** lehetnek mindkettő.

#### Mi határozza meg a scope-ot?

| Forrás | Példa |
|--------|-------|
| **L1 Domain model** | Hierarchikus domain → Composite rendszer szintű |
| **Architecture Interview** | Event-driven architektúra → Observer rendszer szintű |
| **L0 NFR** | "Minden hívás auditálandó" → Decorator rendszer szintű |
| **L2/L4 implementation** | Komplex aggregate → Factory modul szinten |

#### GoF Creational

| Pattern | Rendszer szintű mikor? | Modul szintű mikor? |
|---------|------------------------|---------------------|
| **Factory Method** | Konzisztens creation policy az egész rendszerben | Egy aggregate komplex létrehozása |
| **Abstract Factory** | Objektum-családok (UI theme, DB adapter-ek) | Ritkán modul szinten |
| **Builder** | Ritkán rendszer szinten | Komplex objektum sok paraméterrel |
| **Singleton** | Global resource (de inkább DI!) | Anti-pattern, kerülendő |
| **Prototype** | Ritkán | Object klónozás szükséges |
| **Object Pool** | Connection pool, thread pool | Resource-igényes objektumok |

#### GoF Structural

| Pattern | Rendszer szintű mikor? | Modul szintű mikor? |
|---------|------------------------|---------------------|
| **Adapter** | Konzisztens external integration pattern | Egy külső API illesztése |
| **Bridge** | Platform-független architektúra | Egy komponens több implementációja |
| **Composite** | Hierarchikus domain (CMS, org chart) | Egy fa struktúra a rendszerben |
| **Decorator** | Cross-cutting concerns (logging, auth, cache) | Egy komponens bővítése |
| **Facade** | Subsystem határok, API egyszerűsítés | Komplex modul elrejtése |
| **Flyweight** | Ritkán | Memória optimalizálás sok hasonló objektumnál |
| **Proxy** | Rendszer szintű caching, access control | Lazy load, egy komponens proxy-ja |

#### GoF Behavioral

| Pattern | Rendszer szintű mikor? | Modul szintű mikor? |
|---------|------------------------|---------------------|
| **Observer** | Domain Events, Event Bus | UI komponens subscription |
| **Strategy** | Rendszer szintű algoritmus választás | Egy komponens cserélhető algoritmusa |
| **Command** | Központi command bus, audit trail | Undo/redo, task queue |
| **State** | Ritkán | Entity/aggregate állapotgép |
| **Chain of Responsibility** | Middleware pipeline, request handling | Egy feldolgozási lánc |
| **Mediator** | Központi event/message broker | Komponensek közti kommunikáció |
| **Memento** | Ritkán | Állapotmentés, snapshot |
| **Iterator** | (Nyelvi feature) | (Nyelvi feature) |
| **Template Method** | Base class hierarchia rendszer szinten | Egy algoritmus váz |
| **Visitor** | Ritkán | Műveletek hozzáadása struktúrához |

### 3. CQRS ≠ GoF Command

| Fogalom | Mit jelent |
|---------|------------|
| **CQRS Command** | Write művelet elválasztása read-től (architekturális) |
| **GoF Command** | Művelet objektumba csomagolása (undo, queue, logging) |

CQRS-ben a write-okat LEHET GoF Command-ként implementálni, de nem kötelező.

---

## Nyitott kérdés: Mi legyen a patterns.md (L4)?

**Opciók:**

1. **Pattern Registry** - összegyűjti a különböző szinteken eldőlt patterneket
2. **Merge más dokumentumokba** - architectural → config, coding → coding-standards.md
3. **Csak modul szintű patternek** - Factory, Builder, Adapter, Decorator implementáció

**Döntés:** TBD

---

## Referenciák

- **GoF (Gang of Four):** "Design Patterns: Elements of Reusable Object-Oriented Software" (1994)
- **DDD:** Eric Evans - "Domain-Driven Design" (2003)
- **Clean Architecture:** Robert C. Martin

# System-Level Patterns - Loom kontextusban

## Összefoglaló

Ez a dokumentum a **rendszer szintű** pattern döntéseket tartalmazza, amelyek az **Architecture Interview** során (L1 és L2 között) dőlnek el.

---

## Derivációs lánc

```
L0 (Vision)
    ↓
L1 (Strategic) + interview
    ↓
┌─────────────────────────────────────────────────────┐
│ ★ ARCHITECTURE INTERVIEW (L1 és L2 között) ★        │
│                                                     │
│ Döntések:                                           │
│   - Deployment architecture (Monolith ↔ Microservices)
│   - Code organization (Clean Architecture)          │
│   - Data patterns (CQRS, Event Sourcing)            │
│   - Unit of Work szükségessége                      │
│                                                     │
│ Input: L0 NFR + L1 bounded contexts + interview     │
└─────────────────────────────────────────────────────┘
    ↓
L2 (Tactical)
    ↓
L3 (Operational)
    ↓
L4 (Implementation)
    ↓
Code
```

---

## 1. Deployment Architecture

### Spektrum

A Monolith vs Microservices **nem bináris döntés**, hanem spektrum:

```
Monolith ←——→ Modular Monolith ←——→ Hybrid ←——→ Microservices
```

| Típus | Leírás |
|-------|--------|
| **Monolith** | Egy deployable unit |
| **Modular Monolith** | Monolith, de tiszta modul határokkal (később bontható) |
| **Hybrid** | Core monolith + néhány külön service |
| **Microservices** | Sok független service |

### Interview kérdések

```yaml
architecture_interview:
  - question: "Hány fő dolgozik a projekten?"
    options:
      - label: "1-5 fő"
        score: { monolith: 2 }
      - label: "6-15 fő"
        score: { modular_monolith: 2 }
      - label: "15+ fő, több csapat"
        score: { microservices: 2 }

  - question: "Kell-e a rendszer részeit függetlenül deployolni?"
    options:
      - label: "Nem, együtt deployment OK"
        score: { monolith: 2 }
      - label: "Később talán"
        score: { modular_monolith: 2 }
      - label: "Igen, gyakran és függetlenül"
        score: { microservices: 2 }

  - question: "Kell-e egyes részeket függetlenül skálázni?"
    options:
      - label: "Nem, egyenletes terhelés"
        score: { monolith: 2 }
      - label: "Talán 1-2 komponens"
        score: { hybrid: 2 }
      - label: "Igen, nagyon eltérő terhelés"
        score: { microservices: 2 }

  - question: "Tranzakciós konzisztencia követelmény?"
    options:
      - label: "Erős konzisztencia kritikus"
        score: { monolith: 2 }
      - label: "Többnyire erős, néhol eventual OK"
        score: { modular_monolith: 1, hybrid: 1 }
      - label: "Eventual consistency elfogadható"
        score: { microservices: 2 }

  - question: "DevOps/infra érettség?"
    options:
      - label: "Egyszerű (VPS, Docker)"
        score: { monolith: 2 }
      - label: "Közepes (Docker Compose, CI/CD)"
        score: { modular_monolith: 2 }
      - label: "Magas (K8s, service mesh, monitoring)"
        score: { microservices: 2 }
```

### Automatikusan deriválható információk

| Forrás | Információ | Hatás |
|--------|------------|-------|
| L0 NFR | "99.99% uptime" | → microservices (independent failure) |
| L0 NFR | "egyszerű üzemeltetés" | → monolith |
| L0 NFR | "rapid scaling" | → microservices |
| L1 bounded-context-map | 2-3 context | → monolith/modular |
| L1 bounded-context-map | 8+ context, tiszta határok | → microservices |
| L1 bounded-context-map | Shared kernel mindenhol | → monolith (túl összefonódott) |

### Hatás a derivációs láncra

| Szint | Monolith | Microservices |
|-------|----------|---------------|
| **L0** | NFR: "egyszerű deployment" | NFR: "független skálázás" |
| **L1** | Bounded contexts = modulok | Bounded contexts = service-ek |
| **L2** | Aggregate-ek egy DB-ben | Aggregate-ek külön DB-ben |
| **L3** | 1 service, sok module | Több service, API contracts |
| **L4** | 1 module-design.md | Service-enként külön L4 |

---

## 2. Code Organization (Clean Architecture)

### Variációk

Ezek ugyanannak az elvnek különböző "dialektusai" - **Dependency Inversion** és **Domain-Centric** design:

| Pattern | Terminológia | Fókusz |
|---------|--------------|--------|
| **Clean** (Uncle Bob) | Entities, Use Cases, Interface Adapters | Részletes réteg-előírások |
| **Hexagonal** (Ports & Adapters) | Ports, Adapters, Domain | Port/Adapter koncepció |
| **Onion** | Domain Core, Domain Services, Application | Hasonló a Clean-hez |
| **Layered** (klasszikus) | Presentation, Business, Data | Egyszerűbb, horizontális |

**Közös elv:** Külső rétegek függnek a belsőktől, domain a középpontban.

**Gyakorlatban:** A tényleges implementáció ~90% azonos, csak a terminológia különbözik.

### Loom döntés

**Default: Clean Architecture terminológia**

Indoklás:
- Jól dokumentált (Uncle Bob könyvek, cikkek)
- Elterjedt a DDD közösségben
- Világos réteg-definíciók

```yaml
# loom.config.yaml - V1
l4:
  architecture_style: clean  # default, nem kell megadni

# Későbbi verzióban:
# l4:
#   architecture_style: clean | hexagonal | layered
```

---

## 3. Data Patterns (CQRS, Event Sourcing)

### Kombinálhatóság

Ezek kombinálhatók és függetlenek egymástól:

```
                    ┌─────────────────┐
                    │ Hagyományos     │
                    │ (CRUD)          │
                    └────────┬────────┘
                             │
            ┌────────────────┴────────────────┐
            ▼                                 ▼
    ┌───────────────┐                ┌───────────────┐
    │ CQRS          │                │ Event Sourcing│
    │ (read/write   │                │ (events as    │
    │  separation)  │                │  source)      │
    └───────┬───────┘                └───────┬───────┘
            │                                 │
            └────────────────┬────────────────┘
                             ▼
                    ┌─────────────────┐
                    │ CQRS + Event    │
                    │ Sourcing        │
                    └─────────────────┘
```

| Kombináció | Leírás |
|------------|--------|
| **Hagyományos (CRUD)** | Egy modell read/write-hoz, DB mint source of truth |
| **CQRS only** | Külön read/write modellek, de DB-ben tároljuk a state-et |
| **Event Sourcing only** | Events as source of truth, de egy modell read/write-hoz |
| **CQRS + Event Sourcing** | Events + külön read modellek (projections) |

### Mikor érdemes használni?

**CQRS:**

| Kérdés | Ha igen → CQRS felé |
|--------|---------------------|
| Nagyon eltérő read/write minták? | Pl. komplex reportok vs egyszerű write |
| Read skálázás kritikus? | Külön read replica-k |
| Különböző read nézetek kellenek? | Pl. admin vs user view |

**Event Sourcing:**

| Kérdés | Ha igen → Event Sourcing felé |
|--------|-------------------------------|
| Teljes audit trail kell? | Pénzügyi, compliance |
| Időbeli lekérdezések? | "Mi volt az állapot 3 hónapja?" |
| Undo/replay funkció? | |
| Domain event-központú design? | |

### Döntési inputok

**Fontos:** Mivel L1 és L2 között dől el, az L2 nem lehet input!

**CQRS:**

| Input forrás | Mit keresünk? |
|--------------|---------------|
| L0 NFR | "high read performance", "reporting", "dashboards" |
| L0 user stories | Sok különböző lekérdezés-típus? |
| L1 acceptance criteria | Komplex query követelmények? |
| **Interview** | "Jelentősen eltér a read és write terhelés?" |

**Event Sourcing:**

| Input forrás | Mit keresünk? |
|--------------|---------------|
| L0 NFR | "audit trail", "compliance", "traceability" |
| L0 NFR | "temporal queries", "point-in-time" |
| L1 business rules | "minden változást naplózni kell" |
| **Interview** | "Kell teljes audit log?" |

### Interview kérdések

```yaml
architecture_interview:
  - question: "Jelentősen eltérő read/write minták várhatók?"
    derive_from:
      - l0_nfr: ["reporting", "dashboard", "analytics"]
      - l1_ac: query complexity
    options:
      - label: "Nem, hasonló read/write"
        score: { crud: 2 }
      - label: "Igen, komplex reportok / egyszerű write"
        score: { cqrs: 2 }

  - question: "Kell teljes audit trail vagy historikus lekérdezés?"
    derive_from:
      - l0_nfr: ["audit", "compliance", "temporal"]
      - l1_business_rules: logging requirements
    options:
      - label: "Nem, elég a jelenlegi állapot"
        score: { crud: 2 }
      - label: "Audit log kell, de nem event sourcing"
        score: { crud: 1, audit_log: 1 }
      - label: "Igen, teljes event history"
        score: { event_sourcing: 2 }
```

---

## 4. DDD Tactical Patterns (rendszer szintű)

### Automatikus patternek

| Pattern | Honnan jön? | Döntés |
|---------|-------------|--------|
| Aggregate/Aggregate Root | L2 aggregate-design | ✅ Automatikus |
| Repository | L2 aggregate-design | ✅ Automatikus (minden aggregate-hez) |
| Value Object | L1 domain-model | ✅ Automatikus |
| Domain Event | L2 sequence-design | ✅ Automatikus (ha cross-aggregate flow) |
| Specification | L1 → L2 | ✅ Automatikus deriválás |
| Service Layer | L3 service-boundaries | ✅ Automatikus (thin service, DDD default) |

### Unit of Work (user jóváhagyás kell)

**Mikor használjuk:**

| Eset | Unit of Work kell? |
|------|---------------------|
| Egy aggregate módosítás | ❌ Nem, repository elég |
| Több aggregate egy tranzakcióban | ✅ Igen |
| Explicit transaction control kell | ✅ Igen |

**Deriválás iránya (fontos!):**

```
ROSSZ:
Tech stack → UoW döntés

HELYES:
Domain requirements → UoW szükségesség → Tech stack ajánlás
                              ↓
                    User jóváhagyás
```

**Input (L0/L1):**

| Forrás | Mit keresünk? |
|--------|---------------|
| L1 domain model | Több aggregate szoros kapcsolatban? |
| L1 business rules | "X és Y együtt kell változzon", "atomi művelet" |
| L1 acceptance criteria | Konzisztencia követelmények |
| L0 NFR | "data consistency", "ACID" |

**Döntés helye:** Architecture Interview, user jóváhagyással.

---

## 5. GoF Patternek rendszer szinten

Néhány GoF pattern **rendszer szinten** is alkalmazható, ha a kontextus indokolja:

| Pattern | Rendszer szintű mikor? | Döntés |
|---------|------------------------|--------|
| **Factory Method** | Minden aggregate creation | ✅ Automatikus (DDD konvenció) |
| **Abstract Factory** | Objektum-családok (DB adapter-ek, témák) | ❓ Interview kérdés |
| **Observer** | Domain Events, Event Bus | ✅ Automatikus (DDD konvenció) |
| **Adapter** | Minden external integration | ✅ Automatikus (Clean Architecture) |
| **Bridge** | Platform-független architektúra | ❓ Interview kérdés (ritka) |
| **Composite** | Hierarchikus domain (CMS, org chart) | ✅ Automatikus (L1 domain model) |
| **Decorator** | Cross-cutting concerns (logging, auth, cache) | ❓ Interview + L0 NFR |
| **Facade** | Subsystem határok, API egyszerűsítés | ✅ Automatikus (L3 + Clean Arch) |
| **Chain of Responsibility** | Middleware pipeline, request handling | ✅ Automatikus (L0 NFR + L3) |
| **Command** | Központi command bus, audit trail | ❓ Interview (+ CQRS kapcsolat) |
| **Mediator** | Központi event/message broker | ❓ Interview (choreography vs orchestration) |
| **Strategy** | Rendszer szintű algoritmus választás | ✅ Automatikus (L1 business rules) |

### Factory Method - DDD konvenció

**Döntés:** Automatikus - Loom = DDD = minden aggregate factory-n keresztül jön létre.

**Mit jelent "konzisztens creation policy":**

```go
// ❌ INCONSISTENT - néhol konstruktor, néhol factory
order := &Order{ID: id, Status: "pending"}           // közvetlen
customer := NewCustomer(name, email)                  // factory

// ✅ CONSISTENT - minden aggregate factory-n keresztül
order := NewOrder(customerID, items)                  // factory - validál
customer := NewCustomer(name, email)                  // factory - validál
product := NewProduct(sku, name, price)               // factory - validál
```

**Miért rendszer szintű:**

| Szempont | Jelentőség |
|----------|------------|
| **Invariánsok** | Minden aggregate creation-nél validálunk |
| **Konzisztencia** | Fejlesztők tudják: "mindig `New*` függvényt használunk" |
| **Kód review** | Könnyen ellenőrizhető: van-e közvetlen struct literal |
| **Tesztelhetőség** | Factory-k mock-olhatók |

**Loom implementáció:**
- L2 aggregate-design definiálja az invariánsokat
- L4 coding-standards.md rögzíti a `New{Aggregate}` pattern-t
- Nem kell interview kérdés - automatikus DDD konvenció

**Megjegyzés: CQRS ≠ GoF Command**

| Fogalom | Mit jelent |
|---------|------------|
| **CQRS Command** | Write művelet elválasztása read-től (architekturális döntés) |
| **GoF Command** | Művelet becsomagolása objektumba (undo, queue, logging) |

CQRS-ben a write-okat LEHET GoF Command-ként implementálni, de nem kötelező.

### Observer - Domain Events

**Döntés:** Automatikus - minden cross-aggregate kommunikáció Domain Event-en keresztül történik.

**Mit jelent "Domain Events, Event Bus" rendszer szinten:**

```go
// ❌ KÖZVETLEN HÍVÁS - szoros csatolás
func (o *Order) Place() error {
    // ... order logic ...

    // Közvetlen hívás - szoros csatolás!
    inventoryService.ReserveStock(o.Items)
    notificationService.SendOrderConfirmation(o.CustomerID)
}

// ✅ OBSERVER / DOMAIN EVENTS - laza csatolás
func (o *Order) Place() error {
    // ... order logic ...

    // Event publikálás - nem tudja, ki figyeli
    o.AddDomainEvent(OrderPlacedEvent{
        OrderID:    o.ID,
        CustomerID: o.CustomerID,
        Items:      o.Items,
        PlacedAt:   time.Now(),
    })
}

// Külön handler-ek figyelik az event-et
type InventoryEventHandler struct { ... }
func (h *InventoryEventHandler) Handle(e OrderPlacedEvent) {
    h.inventoryService.ReserveStock(e.Items)
}

type NotificationEventHandler struct { ... }
func (h *NotificationEventHandler) Handle(e OrderPlacedEvent) {
    h.notificationService.SendOrderConfirmation(e.CustomerID)
}
```

**Miért rendszer szintű:**

| Szempont | Jelentőség |
|----------|------------|
| **Bounded Context kommunikáció** | Context-ek között event-eken keresztül kommunikálunk |
| **Laza csatolás** | Aggregate-ek nem függnek egymástól közvetlenül |
| **Eventual consistency** | Microservices esetén async event-ek |
| **Audit trail** | Minden event naplózható |
| **Extendability** | Új handler hozzáadása nem módosítja a publisher-t |

**Loom implementáció:**
- L2 sequence-design definiálja az event-eket (milyen aggregate milyen event-et publikál)
- L3 event-message-design konkretizálja az event struktúrákat
- Nem kell interview kérdés - DDD konvenció

**Mikor NEM kell Observer rendszer szinten:**
- Ha az aggregate-ek teljesen függetlenek (nincs cross-aggregate flow)
- Ekkor modul szinten is lehet Observer (pl. UI subscription)

### Abstract Factory - Objektum-családok

**Döntés:** Interview kérdés - nem minden projekthez kell.

**Mit jelent "Objektum-családok" rendszer szinten:**

```go
// ❌ INCONSISZTENS - kézzel választott implementációk
orderRepo := postgres.NewOrderRepository(db)
customerRepo := mysql.NewCustomerRepository(db)    // Hoppá! Más DB!
productRepo := postgres.NewProductRepository(db)

// ✅ ABSTRACT FACTORY - garantált konzisztencia
type RepositoryFactory interface {
    CreateOrderRepository() OrderRepository
    CreateCustomerRepository() CustomerRepository
    CreateProductRepository() ProductRepository
}

// PostgreSQL család
type PostgresRepositoryFactory struct { db *sql.DB }

func (f *PostgresRepositoryFactory) CreateOrderRepository() OrderRepository {
    return postgres.NewOrderRepository(f.db)
}
func (f *PostgresRepositoryFactory) CreateCustomerRepository() CustomerRepository {
    return postgres.NewCustomerRepository(f.db)
}

// MySQL család - ugyanazok a metódusok, de MySQL implementációkkal
type MySQLRepositoryFactory struct { db *sql.DB }

// Használat - a factory garantálja a konzisztenciát
func NewApplication(factory RepositoryFactory) *Application {
    return &Application{
        orders:    factory.CreateOrderRepository(),
        customers: factory.CreateCustomerRepository(),
        products:  factory.CreateProductRepository(),
    }
}
```

**Mikor rendszer szintű döntés?**

| Kérdés | Magyarázat |
|--------|------------|
| Több implementáció-család kell az egész rendszerben? | A rendszer különböző "módokban" futhat, mindegyik konzisztens objektum-családdal |
| Az összetartozó objektumok konzisztenciája kritikus? | Ha egy családból választunk, MINDEN kapcsolódó objektumnak abból kell jönnie |
| A család-választás több bounded context-et érint? | Nem egy modul belső ügye, hanem rendszerszintű konfiguráció |

**Interview kérdések:**

```yaml
architecture_interview:
  - question: "Kell-e a rendszernek több, eltérő implementációs környezetet támogatnia?"
    derive_from:
      - l0_nfr: ["multi-database", "multi-platform", "multi-tenant", "theming"]
      - l0_user_stories: "admin választhat...", "tenant-specifikus..."
    options:
      - label: "Nem, egy implementáció elég"
        score: { no_abstract_factory: 2 }
      - label: "Igen, több variáns kell (pl. DB típusok, platformok)"
        score: { abstract_factory: 1 }

  - question: "Ha több implementáció van, ezek összetartozó objektumokat érintenek?"
    condition: previous_answer != "Nem, egy implementáció elég"
    options:
      - label: "Nem, független objektumok (pl. csak egy repository más)"
        score: { simple_factory: 2 }
        note: "Egyszerű Factory elég, nem kell Abstract Factory"
      - label: "Igen, több objektumnak együtt kell váltania"
        score: { abstract_factory: 2 }

  - question: "Mikor dől el, melyik implementáció-családot használjuk?"
    condition: abstract_factory score > 0
    options:
      - label: "Build/deploy időben (konfiguráció)"
        score: { abstract_factory: 1 }
        note: "DI container is megoldhatja"
      - label: "Runtime (pl. tenant alapján dinamikusan)"
        score: { abstract_factory: 2 }
        note: "Abstract Factory erősen ajánlott"
```

**Döntési logika:**

```
Több implementáció-család kell?
    │
    ├─ NEM → ❌ Nem kell Abstract Factory
    │
    └─ IGEN → Összetartozó objektumok?
                  │
                  ├─ NEM (független) → Simple Factory elég
                  │
                  └─ IGEN → Abstract Factory ajánlott
                              │
                              └─ Runtime váltás? → Abstract Factory kritikus
```

**Fontos:** A legtöbb projekt **nem igényel** rendszer szintű Abstract Factory-t! Egyetlen DB backend esetén felesleges komplexitás.

### Adapter - External Integration

**Döntés:** Automatikus - Clean Architecture része.

**Mit jelent "Konzisztens external integration pattern":**

```go
// ❌ KÖZVETLEN KÜLSŐ API HÍVÁS - domain függ a külső service-től
func (s *OrderService) ProcessPayment(order *Order) error {
    stripe.Key = "sk_live_..."
    params := &stripe.ChargeParams{
        Amount: stripe.Int64(int64(order.Total.Cents())),
    }
    _, err := charge.New(params)  // Stripe-specifikus!
    return err
}

// ✅ ADAPTER PATTERN - domain nem tud a Stripe-ról
// Domain interface (port)
type PaymentGateway interface {
    Charge(amount Money, token string) (PaymentResult, error)
}

// Stripe adapter (infrastructure)
type StripePaymentAdapter struct {
    apiKey string
}

func (a *StripePaymentAdapter) Charge(amount Money, token string) (PaymentResult, error) {
    // Stripe-specifikus implementáció
    // Domain error-okra map-pel
}

// Domain service - csak az interface-t ismeri
func (s *OrderService) ProcessPayment(order *Order) error {
    result, err := s.paymentGateway.Charge(order.Total, order.PaymentToken)
    // ...
}
```

**Miért "konzisztens" és "rendszer szintű":**

| Szempont | Jelentőség |
|----------|------------|
| **Minden külső service** | Ugyanaz a pattern: interface + adapter |
| **Cserélhetőség** | Stripe → PayPal csere = új adapter, domain változatlan |
| **Tesztelhetőség** | Mock adapter-ek unit tesztekhez |
| **Fejlesztői konzisztencia** | Mindenki tudja: "külső service = adapter kell" |

**Loom implementáció:**
- L1 bounded-context-map azonosítja a külső rendszereket
- L3 service-boundaries konkretizálja a külső API-kat
- L4 module-design definiálja az infrastructure/adapters/ struktúrát
- Nem kell interview kérdés - Clean Architecture választása már eldöntötte

### Bridge - Platform-független architektúra

**Döntés:** Interview kérdés - ritka requirement.

**Mit jelent "platform-független architektúra":**

A Bridge elválasztja az absztrakciót az implementációtól, lehetővé téve, hogy mindkettő függetlenül változzon.

```go
// ✅ BRIDGE - Absztrakció és implementáció külön hierarchia

// === IMPLEMENTÁCIÓ HIERARCHIA (platform-specifikus) ===
type Renderer interface {
    RenderButton(x, y, width, height int, label string)
    RenderTextInput(x, y, width int, placeholder string)
}

type WebRenderer struct { /* HTML/CSS */ }
type DesktopRenderer struct { /* Native OS widgets */ }
type MobileRenderer struct { /* iOS/Android */ }

// === ABSZTRAKCIÓ HIERARCHIA (UI logic) ===
type Dialog struct {
    renderer Renderer  // Bridge: referencia az implementációra
}

type LoginDialog struct {
    Dialog
}

func (d *LoginDialog) Render() {
    d.renderer.RenderTextInput(10, 10, 200, "Username")
    d.renderer.RenderTextInput(10, 50, 200, "Password")
    d.renderer.RenderButton(10, 90, 100, 30, "Login")
}

// Ugyanaz az absztrakció, különböző platformokon
loginDialog := &LoginDialog{Dialog{renderer: &WebRenderer{}}}
loginDialog := &LoginDialog{Dialog{renderer: &MobileRenderer{}}}
```

**Bridge vs Adapter:**

| Pattern | Cél | Mikor |
|---------|-----|-------|
| **Adapter** | Meglévő inkompatibilis interface-ek összeillesztése | Post-hoc |
| **Bridge** | Eleve úgy tervezünk, hogy absztrakció és implementáció külön változhat | Upfront design |

**Interview kérdések:**

```yaml
architecture_interview:
  - question: "Több platformon kell futnia az alkalmazásnak?"
    derive_from:
      - l0_nfr: ["multi-platform", "cross-platform", "web and mobile"]
    options:
      - label: "Nem, egy platform elég (pl. csak backend API)"
        score: { no_bridge: 2 }
      - label: "Igen, több platform (web + mobile + desktop)"
        score: { bridge: 2 }

  - question: "A platform-specifikus és business logika külön fejlődik?"
    condition: previous_answer indicates multi-platform
    options:
      - label: "Nem, elég egyszerű a logika"
        score: { no_bridge: 1 }
      - label: "Igen, mindkettő aktívan változik"
        score: { bridge: 2 }
```

**Fontos:** A legtöbb backend DDD projekt **nem igényel** Bridge-et! Ez inkább cross-platform UI alkalmazásoknál releváns.

### Composite - Hierarchikus domain

**Döntés:** Automatikus - L1 domain model alapján deriválható.

**Mit jelent "Hierarchikus domain":**

A Composite lehetővé teszi, hogy egyedi objektumokat és kompozíciókat egységesen kezeljünk.

```go
// ✅ COMPOSITE - CMS tartalom struktúra

// Közös interface
type Content interface {
    Render() string
    GetChildren() []Content
}

// Levél elem
type TextBlock struct {
    text string
}
func (t *TextBlock) Render() string { return "<p>" + t.text + "</p>" }
func (t *TextBlock) GetChildren() []Content { return nil }

// Kompozit elem
type Section struct {
    title    string
    children []Content
}
func (s *Section) Render() string {
    result := "<section><h2>" + s.title + "</h2>"
    for _, child := range s.children {
        result += child.Render()  // Rekurzív!
    }
    return result + "</section>"
}
func (s *Section) GetChildren() []Content { return s.children }

// Org chart példa - rekurzív összesítés
type OrganizationUnit interface {
    GetEmployeeCount() int  // Rekurzív összesítés
    GetBudget() Money
}

type Department struct {
    name     string
    children []OrganizationUnit
}
func (d *Department) GetEmployeeCount() int {
    count := 0
    for _, child := range d.children {
        count += child.GetEmployeeCount()
    }
    return count
}
```

**L1-ből felismerhető minták:**

```yaml
# L1 domain-model.md - ezek jelzik a Composite szükségességét
entities:
  Category:
    attributes:
      - parent: Category?          # ← Self-referencia
      - subcategories: Category[]  # ← Rekurzív kapcsolat

  MenuItem:
    attributes:
      - children: MenuItem[]       # ← Rekurzív kapcsolat
```

**Loom implementáció:**
- L1 domain-model azonosítja a rekurzív kapcsolatokat
- Ha van hierarchia → Composite automatikus
- Nem kell interview kérdés - domain struktúra diktálja

### Decorator - Cross-cutting Concerns

**Döntés:** Részben deriválható L0 NFR-ből, részben interview kérdés.

**Mit jelent "Cross-cutting concerns":**

A Decorator dinamikusan ad viselkedést objektumokhoz. Cross-cutting concerns (logging, caching, auth) minden service-re egységesen alkalmazható.

```go
// ✅ DECORATOR PATTERN - caching külön rétegben

// Alap interface
type OrderRepository interface {
    FindByID(id OrderID) (*Order, error)
    Save(order *Order) error
}

// Konkrét implementáció - csak DB logika
type PostgresOrderRepository struct { db *sql.DB }

func (r *PostgresOrderRepository) FindByID(id OrderID) (*Order, error) {
    return r.queryFromDB(id)  // Csak DB, semmi más
}

// Caching Decorator
type CachingOrderRepository struct {
    inner OrderRepository  // Wrapped repository
    cache *redis.Client
}

func (r *CachingOrderRepository) FindByID(id OrderID) (*Order, error) {
    // 1. Cache check
    if cached := r.fromCache(id); cached != nil {
        return cached, nil
    }
    // 2. Delegate to inner
    order, err := r.inner.FindByID(id)
    // 3. Cache write
    r.toCache(order)
    return order, err
}

// Logging Decorator
type LoggingOrderRepository struct {
    inner  OrderRepository
    logger *slog.Logger
}

// Decorator láncolás
func NewOrderRepository(db *sql.DB, cache *redis.Client, logger *slog.Logger) OrderRepository {
    var repo OrderRepository
    repo = &PostgresOrderRepository{db: db}        // Alap
    repo = &CachingOrderRepository{inner: repo}    // + Cache
    repo = &LoggingOrderRepository{inner: repo}    // + Logging
    return repo
}
```

**Decorator vs Middleware:**

| Aspektus | Decorator | Middleware |
|----------|-----------|------------|
| **Scope** | Egy interface wrapping | Request/response pipeline |
| **Használat** | Repository, Service réteg | HTTP handlers |

**Interview kérdések:**

```yaml
architecture_interview:
  - question: "Kell egységes cross-cutting concern kezelés?"
    derive_from:
      - l0_nfr: ["audit", "logging", "caching", "security"]
    options:
      - label: "Nem, egyszerű alkalmazás"
        score: { no_decorator: 2 }
      - label: "Igen, de framework middleware elég"
        score: { middleware: 2 }
      - label: "Igen, service/repository szinten is kell"
        score: { decorator: 2 }
```

**Loom implementáció:**
- L0 NFR "audit trail" → Logging Decorator automatikus
- L0 NFR "high performance" → Caching Decorator ajánlott
- Melyik concern, milyen szinten → Interview kérdés

### Facade - Subsystem határok

**Döntés:** Automatikus - Clean Architecture Application Service = Facade.

**Mit jelent "Subsystem határok, API egyszerűsítés":**

A Facade egyszerű interface-t biztosít komplex alrendszerhez.

```go
// ❌ KLIENS ISMERI AZ ÖSSZES BELSŐ SERVICE-T
func PlaceOrderHandler(w http.ResponseWriter, r *http.Request) {
    customer, _ := customerService.GetByID(customerID)
    for _, item := range items {
        inventoryService.CheckStock(item.SKU, item.Qty)
    }
    order, _ := orderService.Create(customerID, items)
    paymentService.Charge(order.Total, paymentToken)
    notificationService.SendConfirmation(customer.Email, order)
    // ... sok koordináció a handler-ben
}

// ✅ FACADE - egyszerű interface
type OrderFacade struct {
    customers     CustomerService
    inventory     InventoryService
    orders        OrderService
    payments      PaymentService
    notifications NotificationService
}

func (f *OrderFacade) PlaceOrder(cmd PlaceOrderCommand) (*OrderResult, error) {
    // Minden koordináció itt, elrejtve
    customer, _ := f.customers.GetByID(cmd.CustomerID)
    // ... validate, create, pay, notify
    return &OrderResult{Order: order}, nil
}

// Handler most egyszerű
func PlaceOrderHandler(w http.ResponseWriter, r *http.Request) {
    result, err := orderFacade.PlaceOrder(cmd)  // Egy hívás!
    json.NewEncoder(w).Encode(result)
}
```

**Kulcs felismerés:** Application Service = Facade

```go
// DDD/Clean Architecture-ben az Application Service már Facade!
type OrderApplicationService struct { /* deps */ }

func (s *OrderApplicationService) PlaceOrder(cmd PlaceOrderCommand) (*Order, error) {
    // Use case koordináció = Facade
}
```

**Mikor kell KÜLÖN Facade (nem Application Service)?**

| Eset | Példa |
|------|-------|
| Több bounded context koordináció | Order + Inventory + Payment |
| Legacy rendszer wrapping | Régi API egyszerűsítése |
| Modul-modul kommunikáció | Modular monolith |

**Loom implementáció:**
- Application Service automatikusan Facade (Clean Architecture)
- Subsystem határok L3 service-boundaries-ból jönnek
- Nem kell interview kérdés

### Strategy - Algoritmus választás

**Döntés:** Automatikus - L1 business rules alapján deriválható.

**Mit jelent "Rendszer szintű algoritmus választás":**

A Strategy algoritmus-családot definiál, és futásidőben cserélhetővé teszi őket.

```go
// ✅ STRATEGY PATTERN - cserélhető árazási algoritmusok

// Strategy interface
type PricingStrategy interface {
    CalculatePrice(basePrice Money, quantity int, customer *Customer) Money
}

// Konkrét stratégiák
type StandardPricing struct{}
type VolumePricing struct{ tiers []VolumeTier }
type VIPPricing struct{ vipDiscount Percent }

// Használat - OrderService nem tudja, melyik stratégia aktív
type OrderService struct {
    pricingStrategy PricingStrategy  // Injektálva
}

func (s *OrderService) CreateOrder(customer *Customer, items []OrderItem) (*Order, error) {
    for _, item := range items {
        price := s.pricingStrategy.CalculatePrice(item.BasePrice, item.Quantity, customer)
        // ...
    }
}
```

**L1-ből felismerhető minták:**

```yaml
# L1 business-rules.md - ezek jelzik a Strategy szükségességét
pricing:
  variations:                    # ← Explicit variációk!
    - standard: "Listaár"
    - volume: "10+ db: 5% kedvezmény"
    - vip: "VIP ügyfelek: 15% kedvezmény"

shipping:
  methods:                       # ← Explicit variációk!
    - standard: "3-5 nap"
    - express: "1 nap"
```

**Döntés logika:**

| Szituáció | Döntés |
|-----------|--------|
| L1 explicit algoritmus variációkat említ | ✅ Strategy automatikus |
| Csak egy algoritmus van | ❌ Nem kell Strategy |
| "Később más algoritmus is kellhet" | ⚠️ YAGNI - ne tervezz előre |

**Loom implementáció:**
- L1 business-rules explicit variációi → Strategy automatikus
- Nem kell interview kérdés - L1 diktálja
- YAGNI: Ne hozz létre Strategy-t "hátha kelleni fog" alapon

### Command - Központi command bus

**Döntés:** Interview kérdés - hacsak L0 NFR nem egyértelmű.

**Fontos: GoF Command ≠ CQRS Command**

| Fogalom | Mit jelent |
|---------|------------|
| **CQRS Command** | Architekturális döntés - read/write szeparáció |
| **GoF Command** | Design pattern - művelet becsomagolása objektumba |

**Mit jelent "Központi command bus, audit trail":**

```go
// ✅ COMMAND PATTERN - központi command bus

// Command interface
type Command interface {
    CommandName() string
}

type PlaceOrderCommand struct {
    CustomerID string
    Items      []OrderItem
    Timestamp  time.Time
    UserID     string
}

// Command Bus - központi dispatcher
type CommandBus struct {
    handlers map[string]any
    logger   AuditLogger
}

func (bus *CommandBus) Dispatch(ctx context.Context, cmd Command) error {
    // 1. Audit log - BEFORE
    bus.logger.LogCommand(cmd, "received")

    // 2. Authorization check
    if err := bus.authz.CanExecute(ctx, cmd); err != nil {
        return err
    }

    // 3. Execute handler
    err := bus.handlers[cmd.CommandName()].Handle(ctx, cmd)

    // 4. Audit log - AFTER
    bus.logger.LogCommand(cmd, "completed", err)
    return err
}
```

**Command Bus előnyei vs hátrányai:**

| Előny | Hátrány |
|-------|---------|
| Audit trail | Komplexitás |
| Központi authz | Extra indirection |
| Retry/queue | Overhead egyszerű CRUD-hoz |
| Undo lehetőség | |

**Interview kérdések:**

```yaml
architecture_interview:
  - question: "Kell központi command bus?"
    derive_from:
      - l0_nfr: ["audit trail", "compliance", "undo", "async commands"]
      - previous: cqrs_decision
    options:
      - label: "Nem, egyszerű service hívások elegek"
        score: { no_command_bus: 2 }
      - label: "Igen, audit/compliance miatt"
        score: { command_bus: 2 }
      - label: "Igen, CQRS architektúrával"
        score: { command_bus: 2, cqrs: 1 }
```

**Kapcsolat más patternekkel:**

| Pattern | Kapcsolat |
|---------|-----------|
| **CQRS** | Command Bus természetes párja |
| **Event Sourcing** | Command → Event → Store |
| **Mediator** | Command Bus = speciális Mediator |

**Loom implementáció:**
- Ha L0 NFR "audit trail", "compliance" → Command Bus ajánlott
- Ha CQRS döntés pozitív → Command Bus természetes
- Egyszerű projekthez túlzás - YAGNI

### Chain of Responsibility - Middleware pipeline

**Döntés:** Automatikus - L0 NFR + L3 alapján deriválható.

**Mit jelent "Middleware pipeline, request handling":**

Láncolt handler-eken vezeti át a kérést. Minden handler dönt: feldolgozza és/vagy továbbadja.

```go
// ✅ CHAIN OF RESPONSIBILITY - HTTP middleware

// Middleware type
type Middleware func(http.Handler) http.Handler

// Logging middleware
func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)  // Továbbadás
        log.Printf("%s %s %v", r.Method, r.URL.Path, time.Since(start))
    })
}

// Auth middleware - lánc megszakítása ha unauthorized
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !isValidToken(r.Header.Get("Authorization")) {
            http.Error(w, "Unauthorized", 401)
            return  // Lánc megszakítása!
        }
        next.ServeHTTP(w, r)
    })
}

// Lánc összeállítása (framework-el)
func main() {
    r := chi.NewRouter()
    r.Use(middleware.Recoverer)
    r.Use(middleware.Logger)
    r.Use(AuthMiddleware)
    r.Post("/orders", orderHandler)
}
```

**Chain of Responsibility vs Decorator:**

| Aspektus | Chain of Responsibility | Decorator |
|----------|------------------------|-----------|
| **Megszakítás** | Bármelyik handler megállíthatja | Mindig végigmegy |
| **Használat** | HTTP middleware | Repository/Service wrapping |
| **Loom réteg** | Interfaces (HTTP) | Domain/Infrastructure |

**L0 NFR → Middleware mapping:**

| L0 NFR | Middleware |
|--------|------------|
| "authentication required" | Auth middleware |
| "audit logging" | Logging middleware |
| "rate limiting" | RateLimit middleware |
| "error tracking" | Recovery middleware |

**Loom implementáció:**
- A pattern maga framework által biztosított
- Middleware igények L0 NFR-ből deriválhatók
- L3 definiálja, melyik endpoint mit igényel
- Nem kell interview kérdés

### Mediator - Event/Message Broker

**Döntés:** Interview kérdés - choreography vs orchestration.

**Mediator vs Observer:**

| Aspektus | Observer | Mediator |
|----------|----------|----------|
| **Kommunikáció** | Publisher → Subscribers | Központi hub |
| **Koordináció** | Nincs | Van - Mediator dönt |
| **Loom-ban** | Domain Events | Komplex orchestration |

**Mikor kell Mediator (nem csak Observer)?**

| Szituáció | Pattern |
|-----------|---------|
| Egyszerű event pub/sub | Observer elég |
| Saga / distributed transaction | Mediator (Saga Coordinator) |
| Microservices kommunikáció | Message Broker (Mediator) |

```go
// ✅ ORCHESTRATION MEDIATOR - Saga koordinátor
type OrderSagaMediator struct {
    orders    OrderService
    inventory InventoryService
    payments  PaymentService
}

func (m *OrderSagaMediator) Handle(event Event) {
    switch e := event.(type) {
    case OrderCreatedEvent:
        // Reserve inventory, ha hiba → compensate
        if err := m.inventory.Reserve(e.Items); err != nil {
            m.eventBus.Publish(OrderFailedEvent{OrderID: e.OrderID})
            return
        }
        m.eventBus.Publish(InventoryReservedEvent{OrderID: e.OrderID})

    case InventoryReservedEvent:
        // Process payment, ha hiba → release inventory
        // ... központi koordináció
    }
}
```

**Interview kérdések:**

```yaml
architecture_interview:
  - question: "Kell komplex orchestration a service-ek között?"
    derive_from:
      - l0_nfr: ["distributed transactions", "saga"]
      - previous: microservices_decision
    options:
      - label: "Nem, egyszerű event-based kommunikáció"
        score: { observer: 2 }
      - label: "Igen, de choreography (decentralizált)"
        score: { observer: 2, choreography: 1 }
      - label: "Igen, orchestration (központi koordinátor)"
        score: { mediator: 2, saga: 1 }
```

**Döntési logika:**

```
Microservices / distributed?
    │
    ├─ NEM → Observer (Domain Events) elég
    │
    └─ IGEN → Koordináció típusa?
                │
                ├─ Choreography → Observer + Event Bus
                │
                └─ Orchestration → Mediator (Saga)
```

**Loom implementáció:**
- Egyszerű event-driven → Observer automatikus
- Komplex orchestration → Interview kérdés
- Saga pattern → Mediator ajánlott

---

## Kapcsolódó dokumentumok

- [module-level-patterns.md](module-level-patterns.md) - Modul szintű patternek (TODO)
- [tech-stack-derivation.md](tech-stack-derivation.md) - Tech stack deriválás (TODO)
- [design-patterns-analysis.md](design-patterns-analysis.md) - Eredeti elemzés és összefoglaló

---

## Referenciák

- **DDD:** Eric Evans - "Domain-Driven Design" (2003)
- **Clean Architecture:** Robert C. Martin
- **CQRS/ES:** Greg Young, Martin Fowler

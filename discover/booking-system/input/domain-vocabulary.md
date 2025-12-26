# Domain Vocabulary – Appointment Booking System

Ez a dokumentum definiálja az Appointment Booking System domain fogalmait.

**Context:** Szolgáltatás alapú időpontfoglalási rendszer (pl. fodrász, orvos, tanácsadó).

---

## Core Concepts

### Booking
Egy időpontfoglalás, amelyben egy Customer egy adott Service-t foglal egy Provider-nél egy meghatározott időpontra.

### Customer
A szolgáltatást igénybe vevő személy. Regisztrált felhasználó a rendszerben.

### Provider
A szolgáltatást nyújtó személy (pl. fodrász, orvos). Saját elérhetőségi időszakokkal rendelkezik.

### Service
Egy konkrét szolgáltatás típus, amelyet a Provider nyújt. Van időtartama és ára.

### TimeSlot
Egy adott időintervallum, amely foglalható. A Provider availability-jéből származik.

### Availability
A Provider által megadott időszakok, amikor elérhető szolgáltatásnyújtásra.

---

## Supporting Concepts

### BookingStatus
A foglalás állapota: Pending, Confirmed, Cancelled, Completed, NoShow.

### Calendar
A Provider naptára, amely tartalmazza az availability-t és a meglévő foglalásokat.

### Reminder
Emlékeztető értesítés a közelgő foglalásról.

### CancellationPolicy
A lemondási szabályok (pl. 24 órán belül nem mondható le).

---

## Relationships

- Booking **belongs to** Customer
- Booking **reserved with** Provider
- Booking **for** Service
- Booking **at** TimeSlot
- Provider **offers** 1..N Service
- Provider **has** Availability (weekly schedule)
- Service **has** duration and price
- TimeSlot **derived from** Availability minus existing Bookings

---

## Business Context

### Actors
- **Customer:** Foglalást hoz létre, módosít, töröl
- **Provider:** Elérhetőséget kezel, foglalásokat lát
- **System:** Emlékeztetőket küld, konfliktusokat detektál

### Key Flows
1. Customer keres szabad időpontot → foglal
2. Provider beállítja a heti elérhetőségét
3. Foglalás közeledik → emlékeztető
4. Customer lemondja → időpont felszabadul

---

## Open Questions

- Egy időpontban több foglalás lehet-e (group booking)?
- A Provider módosíthatja-e a foglalást?
- Mi történik, ha a Customer nem jelenik meg (no-show)?
- Kell-e fizetési integráció?

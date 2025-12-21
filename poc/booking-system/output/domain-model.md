---
status: draft
derived-from: "loom-project/poc/booking-system/input/domain-vocabulary.md"
derived-at: "2025-12-21T16:30:00Z"
derived-by: "loom-derive-domain skill v2.0 (Structured Interview)"
loom-version: "3.0.0"
structured-interview:
  decision-points-resolved: 5
  from-user-answers: 5
  decisions:
    EVO-1: timeslot-value-object
    AGG-1: calendar-aggregate
    AGG-2: booking-with-timeslot
    REF-1: service-reference
    ENT-1: reminder-scheduled-job
---

# Domain Model – Appointment Booking System

## Overview

This document defines the domain model for the Appointment Booking System, including entities, value objects, aggregates, and their relationships.

**Bounded Context:** Appointment Booking

---

## Aggregates

### 1. Booking Aggregate

The Booking aggregate is the central concept, representing a customer's appointment with a provider.

```
┌─────────────────────────────────────────────────────────────┐
│                    BOOKING AGGREGATE                        │
├─────────────────────────────────────────────────────────────┤
│  Booking (Aggregate Root)                                   │
│  ├── bookingId: BookingId (Entity ID)                      │
│  ├── customerId: CustomerId (Reference)                    │
│  ├── providerId: ProviderId (Reference)                    │
│  ├── serviceId: ServiceId (Reference)                      │
│  ├── timeSlot: TimeSlot (Value Object) ◄── Embedded        │
│  ├── status: BookingStatus (Value Object)                  │
│  ├── createdAt: DateTime                                   │
│  ├── updatedAt: DateTime                                   │
│  └── cancelledAt: DateTime?                                │
└─────────────────────────────────────────────────────────────┘
```

**Decision Points Resolved:**
- EVO-1: TimeSlot is a Value Object embedded in Booking (User answer: 1b)
- AGG-2: Booking aggregate contains only Booking + TimeSlot (User answer: 3b)
- REF-1: Service stored as reference (serviceId), price resolved at booking time (User answer: 4a)

**Invariants:**
- A Booking must have a valid TimeSlot in the future (at creation)
- A Booking must reference an existing Customer, Provider, and Service
- Status transitions must follow the state machine

---

### 2. Calendar Aggregate

The Calendar aggregate manages provider availability independently from bookings.

```
┌─────────────────────────────────────────────────────────────┐
│                    CALENDAR AGGREGATE                       │
├─────────────────────────────────────────────────────────────┤
│  Calendar (Aggregate Root)                                  │
│  ├── calendarId: CalendarId (Entity ID)                    │
│  ├── providerId: ProviderId (Reference)                    │
│  ├── weeklySchedule: WeeklySchedule (Value Object)         │
│  │   └── slots: Map<DayOfWeek, List<TimeRange>>            │
│  ├── exceptions: List<AvailabilityException>               │
│  │   └── each: { date, available, reason? }                │
│  └── updatedAt: DateTime                                   │
└─────────────────────────────────────────────────────────────┘
```

**Decision Points Resolved:**
- AGG-1: Availability managed as separate Calendar aggregate (User answer: 2c)

**Invariants:**
- A Calendar belongs to exactly one Provider
- Exceptions cannot be set for past dates
- Weekly schedule slots cannot overlap

---

### 3. Customer Aggregate

```
┌─────────────────────────────────────────────────────────────┐
│                    CUSTOMER AGGREGATE                       │
├─────────────────────────────────────────────────────────────┤
│  Customer (Aggregate Root)                                  │
│  ├── customerId: CustomerId (Entity ID)                    │
│  ├── email: Email (Value Object)                           │
│  ├── name: PersonName (Value Object)                       │
│  ├── preferences: CustomerPreferences (Value Object)       │
│  │   └── reminderOptOut: boolean                           │
│  └── createdAt: DateTime                                   │
└─────────────────────────────────────────────────────────────┘
```

**Invariants:**
- Email must be unique across customers
- Customer must have valid email for notifications

---

### 4. Provider Aggregate

```
┌─────────────────────────────────────────────────────────────┐
│                    PROVIDER AGGREGATE                       │
├─────────────────────────────────────────────────────────────┤
│  Provider (Aggregate Root)                                  │
│  ├── providerId: ProviderId (Entity ID)                    │
│  ├── name: string                                          │
│  ├── email: Email (Value Object)                           │
│  ├── serviceIds: List<ServiceId> (References)              │
│  └── calendarId: CalendarId (Reference)                    │
└─────────────────────────────────────────────────────────────┘
```

**Invariants:**
- Provider must have at least one service
- Provider must have a linked Calendar

---

### 5. Service Aggregate

```
┌─────────────────────────────────────────────────────────────┐
│                    SERVICE AGGREGATE                        │
├─────────────────────────────────────────────────────────────┤
│  Service (Aggregate Root)                                   │
│  ├── serviceId: ServiceId (Entity ID)                      │
│  ├── name: string                                          │
│  ├── description: string                                   │
│  ├── duration: Duration (Value Object)                     │
│  ├── price: Money (Value Object)                           │
│  └── active: boolean                                       │
└─────────────────────────────────────────────────────────────┘
```

**Invariants:**
- Duration must be positive
- Price must be non-negative

---

## Value Objects

### TimeSlot

Represents a specific time interval for a booking.

```typescript
class TimeSlot {
  readonly startTime: DateTime
  readonly endTime: DateTime

  // Derived
  get duration(): Duration

  // Validation
  isInFuture(): boolean
  overlaps(other: TimeSlot): boolean
}
```

**Decision Point:** EVO-1 - TimeSlot is a Value Object (User answer: 1b)
- Rationale: TimeSlot has no identity, only start/end time matters
- Equality: Two TimeSlots are equal if start and end times match
- Immutable: Changes create new TimeSlot instance

---

### BookingStatus

Enumeration of booking states.

```typescript
enum BookingStatus {
  Pending = "pending",       // Just created, awaiting confirmation
  Confirmed = "confirmed",   // Provider/system confirmed
  Cancelled = "cancelled",   // Cancelled by customer or provider
  Completed = "completed",   // Service delivered
  NoShow = "no_show"         // Customer didn't show up
}
```

**State Machine:**

```
┌─────────┐    confirm    ┌───────────┐
│ Pending │──────────────►│ Confirmed │
└────┬────┘               └─────┬─────┘
     │                          │
     │ cancel                   │ cancel / complete / no-show
     │                          │
     ▼                          ▼
┌───────────┐            ┌───────────┐    ┌───────────┐
│ Cancelled │            │ Completed │    │  NoShow   │
└───────────┘            └───────────┘    └───────────┘
```

**Valid Transitions:**
- Pending → Confirmed, Cancelled
- Confirmed → Cancelled, Completed, NoShow
- Cancelled, Completed, NoShow → (terminal states)

---

### Money

```typescript
class Money {
  readonly amount: Decimal
  readonly currency: Currency

  add(other: Money): Money
  multiply(factor: number): Money
}
```

---

### Duration

```typescript
class Duration {
  readonly minutes: number

  static fromMinutes(m: number): Duration
  static fromHours(h: number): Duration

  toMinutes(): number
}
```

---

### Email

```typescript
class Email {
  readonly value: string

  // Validation: must be valid email format
  static create(value: string): Email | Error
}
```

---

### PersonName

```typescript
class PersonName {
  readonly firstName: string
  readonly lastName: string

  get fullName(): string
}
```

---

### WeeklySchedule

```typescript
class WeeklySchedule {
  readonly slots: Map<DayOfWeek, TimeRange[]>

  isAvailable(dateTime: DateTime): boolean
  getAvailableSlots(date: Date, duration: Duration): TimeSlot[]
}
```

---

### TimeRange

```typescript
class TimeRange {
  readonly start: LocalTime  // e.g., 09:00
  readonly end: LocalTime    // e.g., 17:00

  contains(time: LocalTime): boolean
}
```

---

## Domain Services

### BookingService

Orchestrates booking creation with availability checking.

```typescript
interface BookingService {
  createBooking(
    customerId: CustomerId,
    providerId: ProviderId,
    serviceId: ServiceId,
    requestedSlot: TimeSlot
  ): Result<Booking, BookingError>

  cancelBooking(
    bookingId: BookingId,
    actorId: CustomerId | ProviderId
  ): Result<void, CancellationError>

  rescheduleBooking(
    bookingId: BookingId,
    newSlot: TimeSlot,
    actorId: CustomerId
  ): Result<Booking, RescheduleError>
}
```

**Cross-Aggregate Coordination:**
- Checks Calendar for availability
- Creates/updates Booking
- Handles atomicity (especially for reschedule)

---

### AvailabilityService

Computes available time slots from Calendar minus existing Bookings.

```typescript
interface AvailabilityService {
  getAvailableSlots(
    providerId: ProviderId,
    date: Date,
    serviceDuration: Duration
  ): TimeSlot[]

  isSlotAvailable(
    providerId: ProviderId,
    slot: TimeSlot
  ): boolean
}
```

---

### ReminderScheduler

Schedules and sends booking reminders.

```typescript
interface ReminderScheduler {
  scheduleReminder(booking: Booking): void
  cancelReminder(bookingId: BookingId): void
  processReminders(): void  // Called by scheduler
}
```

**Decision Point:** ENT-1 - Reminder is a scheduled job, not an entity (User answer: 5c)
- No Reminder entity in the domain
- Infrastructure handles scheduling (cron job, message queue)
- Reminder timing: 24 hours before booking

---

## Repositories

```typescript
interface BookingRepository {
  findById(id: BookingId): Booking | null
  findByCustomer(customerId: CustomerId): Booking[]
  findByProvider(providerId: ProviderId, date: Date): Booking[]
  save(booking: Booking): void
}

interface CalendarRepository {
  findByProvider(providerId: ProviderId): Calendar | null
  save(calendar: Calendar): void
}

interface CustomerRepository {
  findById(id: CustomerId): Customer | null
  findByEmail(email: Email): Customer | null
  save(customer: Customer): void
}

interface ProviderRepository {
  findById(id: ProviderId): Provider | null
  save(provider: Provider): void
}

interface ServiceRepository {
  findById(id: ServiceId): Service | null
  findByProvider(providerId: ProviderId): Service[]
  save(service: Service): void
}
```

---

## Domain Events

```typescript
// Booking events
BookingCreated { bookingId, customerId, providerId, timeSlot }
BookingConfirmed { bookingId }
BookingCancelled { bookingId, cancelledBy, reason? }
BookingRescheduled { bookingId, oldSlot, newSlot }
BookingCompleted { bookingId }
BookingNoShow { bookingId }

// Calendar events
AvailabilityUpdated { calendarId, providerId, changes }
```

---

## Aggregate Relationships Diagram

```
┌──────────────┐         ┌──────────────┐
│   Customer   │         │   Provider   │
│  (Aggregate) │         │  (Aggregate) │
└──────┬───────┘         └──────┬───────┘
       │                        │
       │ customerId             │ providerId
       │                        │
       ▼                        ▼
┌──────────────────────────────────────┐
│              Booking                 │
│            (Aggregate)               │
│  ┌────────────────────────────────┐  │
│  │    TimeSlot (Value Object)     │  │
│  │    - embedded in Booking       │  │
│  └────────────────────────────────┘  │
└──────────────────────────────────────┘
       │
       │ serviceId (reference)
       ▼
┌──────────────┐         ┌──────────────┐
│   Service    │         │   Calendar   │
│  (Aggregate) │         │  (Aggregate) │
└──────────────┘         └──────────────┘
       ▲                        ▲
       │                        │
       │ serviceIds             │ calendarId
       │                        │
       └────────────────────────┘
              Provider references
```

---

## Summary

| Concept | Classification | Aggregate |
|---------|----------------|-----------|
| Booking | Entity (Root) | Booking |
| TimeSlot | Value Object | Booking (embedded) |
| BookingStatus | Value Object | Booking |
| Calendar | Entity (Root) | Calendar |
| WeeklySchedule | Value Object | Calendar |
| Customer | Entity (Root) | Customer |
| Provider | Entity (Root) | Provider |
| Service | Entity (Root) | Service |
| Reminder | N/A | Scheduled job |

**SI Decisions Applied:**
- TimeSlot as VO in Booking (1b)
- Calendar as separate aggregate (2c)
- Booking aggregate = Booking + TimeSlot only (3b)
- Service as reference (4a)
- Reminder as scheduled job (5c)

---

## Traceability

| Domain Concept | Source |
|----------------|--------|
| Booking | domain-vocabulary.md#booking |
| Customer | domain-vocabulary.md#customer |
| Provider | domain-vocabulary.md#provider |
| Service | domain-vocabulary.md#service |
| TimeSlot | domain-vocabulary.md#timeslot |
| Availability (Calendar) | domain-vocabulary.md#availability |
| BookingStatus | domain-vocabulary.md#bookingstatus |
| Reminder | domain-vocabulary.md#reminder |
| CancellationPolicy | BR-BOOK-007 |

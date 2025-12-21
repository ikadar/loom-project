---
status: draft
derived-from:
  - "loom-project/poc/booking-system/output/acceptance-criteria.md"
  - "loom-project/poc/booking-system/output/business-rules.md"
  - "loom-project/poc/booking-system/output/interface-contracts.md"
derived-at: "2025-12-21T17:00:00Z"
derived-by: "loom-derive-l2 skill v2.0 (Structured Interview)"
loom-version: "3.0.0"
structured-interview:
  decision-points-resolved: 5
  from-user-answers: 5
  decisions:
    API-1: rest-resource-urls
    API-2: jwt-bearer-token
    COM-1: hybrid-communication
    CON-1: pessimistic-locking
    ERR-1: rfc7807-problem-details
---

# Sequence Design – Appointment Booking System

## Overview

This document describes the key interaction sequences for the Appointment Booking System.

**Communication Pattern:** Hybrid (SI decision: COM-1)
- Sync for critical operations
- Async for notifications

**Concurrency:** Pessimistic locking (SI decision: CON-1)

---

## SEQ-001: Create Booking (Happy Path)

**Trigger:** Customer submits booking request
**Actors:** Customer, BookingAPI, BookingService, AvailabilityService, BookingRepository, EventBus

```
┌─────────┐     ┌────────────┐     ┌────────────────┐     ┌─────────────────────┐     ┌───────────────────┐     ┌──────────┐
│Customer │     │ BookingAPI │     │ BookingService │     │ AvailabilityService │     │ BookingRepository │     │ EventBus │
└────┬────┘     └─────┬──────┘     └───────┬────────┘     └──────────┬──────────┘     └─────────┬─────────┘     └────┬─────┘
     │                │                    │                         │                         │                    │
     │ POST /bookings │                    │                         │                         │                    │
     │───────────────►│                    │                         │                         │                    │
     │                │                    │                         │                         │                    │
     │                │ validate JWT       │                         │                         │                    │
     │                │────────────────────│                         │                         │                    │
     │                │                    │                         │                         │                    │
     │                │ createBooking()    │                         │                         │                    │
     │                │───────────────────►│                         │                         │                    │
     │                │                    │                         │                         │                    │
     │                │                    │ checkAvailability()     │                         │                    │
     │                │                    │────────────────────────►│                         │                    │
     │                │                    │                         │                         │                    │
     │                │                    │                         │ BEGIN TRANSACTION       │                    │
     │                │                    │                         │ + LOCK slot             │                    │
     │                │                    │                         │────────────────────────►│                    │
     │                │                    │                         │                         │                    │
     │                │                    │                         │◄────────────────────────│                    │
     │                │                    │                         │ slot available          │                    │
     │                │                    │◄────────────────────────│                         │                    │
     │                │                    │ isAvailable: true       │                         │                    │
     │                │                    │                         │                         │                    │
     │                │                    │ save(booking)           │                         │                    │
     │                │                    │──────────────────────────────────────────────────►│                    │
     │                │                    │                         │                         │                    │
     │                │                    │◄──────────────────────────────────────────────────│                    │
     │                │                    │ booking saved           │                         │                    │
     │                │                    │                         │                         │                    │
     │                │                    │                         │ COMMIT TRANSACTION      │                    │
     │                │                    │                         │────────────────────────►│                    │
     │                │                    │                         │                         │                    │
     │                │                    │ publish(BookingCreated) │                         │                    │
     │                │                    │────────────────────────────────────────────────────────────────────────►│
     │                │                    │                         │                         │                    │
     │                │◄───────────────────│                         │                         │                    │
     │                │ booking            │                         │                         │                    │
     │                │                    │                         │                         │                    │
     │◄───────────────│                    │                         │                         │                    │
     │ 201 Created    │                    │                         │                         │                    │
     │ + booking JSON │                    │                         │                         │                    │
```

**Key Points:**
- Pessimistic lock acquired on time slot (SI: CON-1)
- Transaction ensures atomicity
- Event published async for notifications (SI: COM-1)

**Traceability:**
- AC: AC-BOOK-001-1
- BR: BR-BOOK-001, BR-BOOK-002, BR-BOOK-003

---

## SEQ-002: Create Booking (Race Condition)

**Trigger:** Two customers try to book same slot
**Result:** First wins, second gets 409 Conflict

```
┌──────────┐  ┌──────────┐     ┌────────────┐     ┌────────────────┐     ┌───────────────────┐
│Customer A│  │Customer B│     │ BookingAPI │     │ BookingService │     │ BookingRepository │
└────┬─────┘  └────┬─────┘     └─────┬──────┘     └───────┬────────┘     └─────────┬─────────┘
     │             │                 │                    │                        │
     │ POST /bookings (slot X)       │                    │                        │
     │──────────────────────────────►│                    │                        │
     │             │                 │                    │                        │
     │             │ POST /bookings (slot X)              │                        │
     │             │────────────────►│                    │                        │
     │             │                 │                    │                        │
     │             │                 │ createBooking(A)   │                        │
     │             │                 │───────────────────►│                        │
     │             │                 │                    │                        │
     │             │                 │                    │ LOCK slot X (acquired) │
     │             │                 │                    │───────────────────────►│
     │             │                 │                    │                        │
     │             │                 │ createBooking(B)   │                        │
     │             │                 │───────────────────►│                        │
     │             │                 │                    │                        │
     │             │                 │                    │ LOCK slot X (waiting)  │
     │             │                 │                    │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─►│
     │             │                 │                    │                        │
     │             │                 │                    │ A: save booking        │
     │             │                 │                    │───────────────────────►│
     │             │                 │                    │                        │
     │             │                 │                    │ A: COMMIT + release    │
     │             │                 │                    │───────────────────────►│
     │             │                 │                    │                        │
     │◄──────────────────────────────│                    │                        │
     │ 201 Created                   │                    │                        │
     │             │                 │                    │                        │
     │             │                 │                    │ B: lock acquired       │
     │             │                 │                    │◄───────────────────────│
     │             │                 │                    │                        │
     │             │                 │                    │ B: slot now booked!    │
     │             │                 │                    │◄───────────────────────│
     │             │                 │                    │                        │
     │             │                 │                    │ B: ROLLBACK            │
     │             │                 │                    │───────────────────────►│
     │             │                 │                    │                        │
     │             │                 │◄───────────────────│                        │
     │             │                 │ SlotUnavailable    │                        │
     │             │                 │                    │                        │
     │             │◄────────────────│                    │                        │
     │             │ 409 Conflict    │                    │                        │
     │             │ TIME_SLOT_UNAVAILABLE                │                        │
```

**Key Points:**
- Pessimistic locking prevents double-booking
- First transaction wins, second sees updated state
- Clear error returned per RFC 7807 (SI: ERR-1)

**Traceability:**
- AC: AC-BOOK-001-2
- BR: BR-BOOK-001

---

## SEQ-003: Cancel Booking

**Trigger:** Customer or provider cancels booking
**Actors:** Actor (Customer/Provider), BookingAPI, BookingService, BookingRepository, EventBus

```
┌───────┐     ┌────────────┐     ┌────────────────┐     ┌───────────────────┐     ┌──────────┐
│ Actor │     │ BookingAPI │     │ BookingService │     │ BookingRepository │     │ EventBus │
└───┬───┘     └─────┬──────┘     └───────┬────────┘     └─────────┬─────────┘     └────┬─────┘
    │               │                    │                        │                    │
    │ DELETE /bookings/{id}              │                        │                    │
    │──────────────►│                    │                        │                    │
    │               │                    │                        │                    │
    │               │ validate JWT       │                        │                    │
    │               │ extract actorId    │                        │                    │
    │               │                    │                        │                    │
    │               │ cancelBooking(id, actorId)                  │                    │
    │               │───────────────────►│                        │                    │
    │               │                    │                        │                    │
    │               │                    │ findById(id)           │                    │
    │               │                    │───────────────────────►│                    │
    │               │                    │                        │                    │
    │               │                    │◄───────────────────────│                    │
    │               │                    │ booking                │                    │
    │               │                    │                        │                    │
    │               │                    │ validate:              │                    │
    │               │                    │ - actor is owner/provider                   │
    │               │                    │ - status is Pending/Confirmed               │
    │               │                    │ - timeSlot in future   │                    │
    │               │                    │                        │                    │
    │               │                    │ booking.cancel()       │                    │
    │               │                    │ save(booking)          │                    │
    │               │                    │───────────────────────►│                    │
    │               │                    │                        │                    │
    │               │                    │ publish(BookingCancelled)                   │
    │               │                    │─────────────────────────────────────────────►
    │               │                    │                        │                    │
    │               │◄───────────────────│                        │                    │
    │               │ cancelled booking  │                        │                    │
    │               │                    │                        │                    │
    │◄──────────────│                    │                        │                    │
    │ 200 OK        │                    │                        │                    │
```

**Validation Checks:**
1. Actor is booking owner (customerId) OR provider (providerId) - BR-BOOK-004
2. Status is Pending or Confirmed - BR-BOOK-005
3. TimeSlot is in future - BR-BOOK-006

**Traceability:**
- AC: AC-BOOK-002-1, AC-BOOK-002-2, AC-BOOK-002-4, AC-BOOK-002-5
- BR: BR-BOOK-004, BR-BOOK-005, BR-BOOK-006

---

## SEQ-004: Cancel Booking (Policy Warning)

**Trigger:** Customer cancels within 24h window
**Flow:** Dry-run check, warning, confirmation, cancel

```
┌──────────┐     ┌────────────┐     ┌────────────────┐     ┌────────────────────┐
│ Customer │     │ BookingAPI │     │ BookingService │     │ CancellationPolicy │
└────┬─────┘     └─────┬──────┘     └───────┬────────┘     └─────────┬──────────┘
     │                 │                    │                        │
     │ DELETE /bookings/{id}?dryRun=true    │                        │
     │────────────────►│                    │                        │
     │                 │                    │                        │
     │                 │ checkCancellation(id)                       │
     │                 │───────────────────►│                        │
     │                 │                    │                        │
     │                 │                    │ isWithinPolicyWindow() │
     │                 │                    │───────────────────────►│
     │                 │                    │                        │
     │                 │                    │◄───────────────────────│
     │                 │                    │ true, fee: €25         │
     │                 │                    │                        │
     │                 │◄───────────────────│                        │
     │                 │ { warning: true,   │                        │
     │                 │   fee: €25 }       │                        │
     │                 │                    │                        │
     │◄────────────────│                    │                        │
     │ 200 OK          │                    │                        │
     │ { policyWarning: { fee: €25, ... }}  │                        │
     │                 │                    │                        │
     │                 │                    │                        │
     │ ══════════ User confirms in UI ══════════                     │
     │                 │                    │                        │
     │                 │                    │                        │
     │ DELETE /bookings/{id}                │                        │
     │────────────────►│                    │                        │
     │                 │                    │                        │
     │                 │ cancelBooking(id, actorId)                  │
     │                 │───────────────────►│                        │
     │                 │                    │                        │
     │                 │                    │ [proceed with cancel]  │
     │                 │                    │ [log policy violation] │
     │                 │                    │                        │
     │                 │◄───────────────────│                        │
     │                 │ cancelled          │                        │
     │                 │                    │                        │
     │◄────────────────│                    │                        │
     │ 200 OK          │                    │                        │
     │ { status: "cancelled", policyViolation: { fee: €25 }}         │
```

**Key Points:**
- Client calls with `?dryRun=true` first
- If within 24h window, returns warning + potential fee
- Client shows warning, user must confirm
- Second call without dryRun executes cancellation
- Policy violation recorded in audit log

**Traceability:**
- AC: AC-BOOK-002-3
- BR: BR-BOOK-007

---

## SEQ-005: Reschedule Booking (Atomic)

**Trigger:** Customer reschedules to new time slot
**Atomicity:** Old slot freed AND new slot reserved in single transaction

```
┌──────────┐     ┌────────────┐     ┌────────────────┐     ┌───────────────────┐     ┌──────────┐
│ Customer │     │ BookingAPI │     │ BookingService │     │ BookingRepository │     │ EventBus │
└────┬─────┘     └─────┬──────┘     └───────┬────────┘     └─────────┬─────────┘     └────┬─────┘
     │                 │                    │                        │                    │
     │ PUT /bookings/{id}/reschedule        │                        │                    │
     │ { newTimeSlot: {...} }               │                        │                    │
     │────────────────►│                    │                        │                    │
     │                 │                    │                        │                    │
     │                 │ rescheduleBooking(id, newSlot)              │                    │
     │                 │───────────────────►│                        │                    │
     │                 │                    │                        │                    │
     │                 │                    │ BEGIN TRANSACTION      │                    │
     │                 │                    │───────────────────────►│                    │
     │                 │                    │                        │                    │
     │                 │                    │ LOCK oldSlot           │                    │
     │                 │                    │───────────────────────►│                    │
     │                 │                    │                        │                    │
     │                 │                    │ LOCK newSlot           │                    │
     │                 │                    │───────────────────────►│                    │
     │                 │                    │                        │                    │
     │                 │                    │ check newSlot available│                    │
     │                 │                    │───────────────────────►│                    │
     │                 │                    │                        │                    │
     │                 │                    │◄───────────────────────│                    │
     │                 │                    │ available: true        │                    │
     │                 │                    │                        │                    │
     │                 │                    │ update booking:        │                    │
     │                 │                    │ - free oldSlot         │                    │
     │                 │                    │ - reserve newSlot      │                    │
     │                 │                    │ - update timeSlot      │                    │
     │                 │                    │───────────────────────►│                    │
     │                 │                    │                        │                    │
     │                 │                    │ COMMIT TRANSACTION     │                    │
     │                 │                    │───────────────────────►│                    │
     │                 │                    │                        │                    │
     │                 │                    │ publish(BookingRescheduled)                 │
     │                 │                    │─────────────────────────────────────────────►
     │                 │                    │                        │                    │
     │                 │◄───────────────────│                        │                    │
     │                 │ updated booking    │                        │                    │
     │                 │                    │                        │                    │
     │◄────────────────│                    │                        │                    │
     │ 200 OK          │                    │                        │                    │
     │ { oldSlot, newSlot, ... }            │                        │                    │
```

**Key Points:**
- Single transaction for atomicity (SI decision from L1: SE-1)
- Both slots locked to prevent race conditions
- If new slot unavailable → ROLLBACK, original unchanged
- Notification via async event

**Traceability:**
- AC: AC-BOOK-003-1, AC-BOOK-003-2
- BR: BR-BOOK-008

---

## SEQ-006: Reschedule Failure (New Slot Unavailable)

**Trigger:** Customer tries to reschedule to unavailable slot
**Result:** 409 Conflict, original booking unchanged

```
┌──────────┐     ┌────────────┐     ┌────────────────┐     ┌───────────────────┐
│ Customer │     │ BookingAPI │     │ BookingService │     │ BookingRepository │
└────┬─────┘     └─────┬──────┘     └───────┬────────┘     └─────────┬─────────┘
     │                 │                    │                        │
     │ PUT /bookings/{id}/reschedule        │                        │
     │ { newTimeSlot: {...} }               │                        │
     │────────────────►│                    │                        │
     │                 │                    │                        │
     │                 │ rescheduleBooking(id, newSlot)              │
     │                 │───────────────────►│                        │
     │                 │                    │                        │
     │                 │                    │ BEGIN TRANSACTION      │
     │                 │                    │───────────────────────►│
     │                 │                    │                        │
     │                 │                    │ check newSlot available│
     │                 │                    │───────────────────────►│
     │                 │                    │                        │
     │                 │                    │◄───────────────────────│
     │                 │                    │ available: FALSE       │
     │                 │                    │                        │
     │                 │                    │ ROLLBACK               │
     │                 │                    │───────────────────────►│
     │                 │                    │                        │
     │                 │◄───────────────────│                        │
     │                 │ NewSlotUnavailable │                        │
     │                 │                    │                        │
     │◄────────────────│                    │                        │
     │ 409 Conflict    │                    │                        │
     │ NEW_TIME_SLOT_UNAVAILABLE            │                        │
     │                 │                    │                        │
     │ (original booking unchanged)         │                        │
```

**Key Points:**
- Transaction rolled back on failure
- Original booking and time slot preserved
- Clear error message for client

**Traceability:**
- AC: AC-BOOK-003-2
- BR: BR-BOOK-008

---

## SEQ-007: Set Provider Availability

**Trigger:** Provider updates weekly schedule
**Behavior:** Existing bookings NOT affected (warning only)

```
┌──────────┐     ┌────────────┐     ┌─────────────────────┐     ┌────────────────────┐     ┌───────────────────┐
│ Provider │     │ BookingAPI │     │ AvailabilityService │     │ CalendarRepository │     │ BookingRepository │
└────┬─────┘     └─────┬──────┘     └──────────┬──────────┘     └─────────┬──────────┘     └─────────┬─────────┘
     │                 │                       │                          │                         │
     │ PUT /providers/{id}/calendar            │                          │                         │
     │ { weeklySchedule: {...} }               │                          │                         │
     │────────────────►│                       │                          │                         │
     │                 │                       │                          │                         │
     │                 │ updateCalendar(providerId, schedule)             │                         │
     │                 │──────────────────────►│                          │                         │
     │                 │                       │                          │                         │
     │                 │                       │ findAffectedBookings()   │                         │
     │                 │                       │─────────────────────────────────────────────────────►
     │                 │                       │                          │                         │
     │                 │                       │◄────────────────────────────────────────────────────│
     │                 │                       │ bookings: [book-1, book-2, book-3]                 │
     │                 │                       │                          │                         │
     │                 │                       │ save(calendar)           │                         │
     │                 │                       │─────────────────────────►│                         │
     │                 │                       │                          │                         │
     │                 │                       │ [DO NOT modify bookings] │                         │
     │                 │                       │                          │                         │
     │                 │◄──────────────────────│                          │                         │
     │                 │ { calendar, warnings: │                          │                         │
     │                 │   [3 affected bookings] }                        │                         │
     │                 │                       │                          │                         │
     │◄────────────────│                       │                          │                         │
     │ 200 OK          │                       │                          │                         │
     │ { ..., warnings: [...] }                │                          │                         │
```

**Key Points:**
- Existing bookings are NEVER cancelled automatically
- Provider receives warning about affected bookings
- Provider must manually handle conflicts if needed

**Traceability:**
- AC: AC-BOOK-004-1, AC-BOOK-004-2
- BR: BR-BOOK-010

---

## SEQ-008: Send Booking Reminder

**Trigger:** Scheduled job detects upcoming booking
**Flow:** Check conditions, send notification

```
┌───────────────────┐     ┌─────────────────────┐     ┌───────────────────┐     ┌────────────────────┐     ┌─────────────────────┐
│ ReminderScheduler │     │ BookingRepository   │     │ CustomerRepository│     │ NotificationService│     │ EmailService        │
└─────────┬─────────┘     └──────────┬──────────┘     └─────────┬─────────┘     └──────────┬─────────┘     └──────────┬──────────┘
          │                          │                          │                          │                          │
          │ [Cron: every hour]       │                          │                          │                          │
          │                          │                          │                          │                          │
          │ findBookingsForReminder()│                          │                          │                          │
          │ (status=Confirmed,       │                          │                          │                          │
          │  startTime in 24h)       │                          │                          │                          │
          │─────────────────────────►│                          │                          │                          │
          │                          │                          │                          │                          │
          │◄─────────────────────────│                          │                          │                          │
          │ bookings: [book-1]       │                          │                          │                          │
          │                          │                          │                          │                          │
          │ for each booking:        │                          │                          │                          │
          │                          │                          │                          │                          │
          │ getCustomer(customerId)  │                          │                          │                          │
          │──────────────────────────────────────────────────────►                          │                          │
          │                          │                          │                          │                          │
          │◄─────────────────────────────────────────────────────│                          │                          │
          │ customer                 │                          │                          │                          │
          │                          │                          │                          │                          │
          │ check: customer.reminderOptOut?                     │                          │                          │
          │                          │                          │                          │                          │
          │ [if optOut=false]        │                          │                          │                          │
          │                          │                          │                          │                          │
          │ sendReminder(booking, customer)                     │                          │                          │
          │────────────────────────────────────────────────────────────────────────────────►│                          │
          │                          │                          │                          │                          │
          │                          │                          │                          │ sendEmail(...)           │
          │                          │                          │                          │─────────────────────────►│
          │                          │                          │                          │                          │
          │                          │                          │                          │◄─────────────────────────│
          │                          │                          │                          │ sent                     │
          │                          │                          │                          │                          │
          │◄───────────────────────────────────────────────────────────────────────────────│                          │
          │ sent                     │                          │                          │                          │
```

**Conditions Checked:**
1. Booking status = Confirmed (skip if Cancelled) - BR-BOOK-012
2. Customer.reminderOptOut = false - BR-BOOK-013

**Reminder Content:**
- Booking date and time
- Provider name
- Service name
- Cancellation link

**Traceability:**
- AC: AC-BOOK-005-1, AC-BOOK-005-2, AC-BOOK-005-3
- BR: BR-BOOK-012, BR-BOOK-013

---

## SEQ-009: Skip Reminder (Cancelled Booking)

**Trigger:** Reminder job finds cancelled booking
**Result:** Reminder skipped

```
┌───────────────────┐     ┌─────────────────────┐
│ ReminderScheduler │     │ BookingRepository   │
└─────────┬─────────┘     └──────────┬──────────┘
          │                          │
          │ findBookingsForReminder()│
          │ (status=Confirmed only)  │
          │─────────────────────────►│
          │                          │
          │◄─────────────────────────│
          │ [] (empty - cancelled    │
          │     bookings excluded)   │
          │                          │
          │ [no reminders sent]      │
```

**Key Point:** Query filters by status=Confirmed, so cancelled bookings never returned.

**Traceability:**
- AC: AC-BOOK-005-2
- BR: BR-BOOK-012

---

## Summary

| Sequence | Description | Key Pattern |
|----------|-------------|-------------|
| SEQ-001 | Create booking (happy path) | Pessimistic lock + async event |
| SEQ-002 | Create booking (race condition) | Lock contention handling |
| SEQ-003 | Cancel booking | Authorization check + state validation |
| SEQ-004 | Cancel with policy warning | Dry-run + confirmation flow |
| SEQ-005 | Reschedule (success) | Atomic transaction |
| SEQ-006 | Reschedule (failure) | Rollback, original preserved |
| SEQ-007 | Set availability | Warning for affected bookings |
| SEQ-008 | Send reminder | Scheduled job + opt-out check |
| SEQ-009 | Skip reminder | Query excludes cancelled |

**SI Decisions Applied:**
- REST resource URLs (API-1)
- JWT Bearer tokens (API-2)
- Hybrid communication (COM-1)
- Pessimistic locking (CON-1)
- RFC 7807 errors (ERR-1)

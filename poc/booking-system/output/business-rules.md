---
status: draft
derived-from: "loom-project/poc/booking-system/input/user-stories.md"
derived-at: "2025-12-21T16:00:00Z"
derived-by: "loom-derive skill v2.0 (Structured Interview)"
loom-version: "3.0.0"
structured-interview:
  decision-points-resolved: 7
  from-user-answers: 5
  from-input: 2
---

# Business Rules – Appointment Booking System

## US-BOOK-001 – Customer creates a booking

### BR-BOOK-001 – Time slot must be available

**Rule:**
A booking can only be created if the requested time slot is available (not already booked and within provider availability).

**Invariant:**
```
Booking.create(timeSlot) MUST only succeed when:
  - TimeSlot.isBooked === false
  - TimeSlot.startTime > NOW()
  - TimeSlot within Provider.availability
```

**Enforcement:**
- **Precondition:** Check slot availability with pessimistic locking
- **Violation Behavior:** Reject with blocking error
- **Error Code:** `TIME_SLOT_UNAVAILABLE`

**Decision Points Resolved:**
- EH-1: Blocking error for race condition (User answer: 1a)

**Traceability:**
- User Story: user-stories.md#us-book-001
- Acceptance Criteria: AC-BOOK-001-1, AC-BOOK-001-2
- Entity: ENT-Booking, ENT-TimeSlot

---

### BR-BOOK-002 – Cannot book in the past

**Rule:**
Bookings can only be created for future time slots.

**Invariant:**
```
Booking.timeSlot.startTime > NOW()
```

**Enforcement:**
- **Precondition:** Validate time slot is in future
- **Violation Behavior:** Reject request
- **Error Code:** `PAST_TIME_SLOT`

**Traceability:**
- User Story: user-stories.md#us-book-001
- Acceptance Criteria: AC-BOOK-001-3

---

### BR-BOOK-003 – Booking requires authentication

**Rule:**
Only authenticated customers can create bookings.

**Invariant:**
```
Booking.create() requires authenticated Customer
```

**Enforcement:**
- **Precondition:** Valid authentication token
- **Violation Behavior:** Reject with 401 Unauthorized
- **Error Code:** `AUTHENTICATION_REQUIRED`

**Traceability:**
- User Story: user-stories.md#us-book-001
- Entity: ENT-Customer

---

## US-BOOK-002 – Customer cancels a booking

### BR-BOOK-004 – Cancellation authorization

**Rule:**
A booking can be cancelled by the customer who created it OR by the provider who will perform the service.

**Invariant:**
```
Booking.cancel(actor) MUST only succeed when:
  - actor.id === Booking.customerId
  - OR actor.id === Booking.providerId
```

**Enforcement:**
- **Precondition:** Check actor is customer or provider
- **Violation Behavior:** Reject with 403 Forbidden
- **Error Code:** `NOT_AUTHORIZED_TO_CANCEL`

**Decision Points Resolved:**
- AU-1: Customer OR Provider can cancel (User answer: 2b)

**Traceability:**
- User Story: user-stories.md#us-book-002
- Acceptance Criteria: AC-BOOK-002-1, AC-BOOK-002-2
- Entity: ENT-Booking

---

### BR-BOOK-005 – Cancellation status constraint

**Rule:**
Only bookings with status `Pending` or `Confirmed` can be cancelled.

**Invariant:**
```
Booking.cancel() MUST only succeed when:
  Booking.status IN ('Pending', 'Confirmed')
```

**Enforcement:**
- **Precondition:** Check booking status
- **Violation Behavior:** Reject with 400 Bad Request
- **Error Code:** `INVALID_BOOKING_STATUS`

**Decision Points Resolved:**
- ST-1: Only Pending and Confirmed can be cancelled (User answer: 3b)

**Traceability:**
- User Story: user-stories.md#us-book-002
- Acceptance Criteria: AC-BOOK-002-1, AC-BOOK-002-4
- Entity: ENT-Booking

---

### BR-BOOK-006 – Cannot cancel past bookings

**Rule:**
Bookings for past time slots cannot be cancelled.

**Invariant:**
```
Booking.cancel() MUST only succeed when:
  Booking.timeSlot.startTime > NOW()
```

**Enforcement:**
- **Precondition:** Check booking is in future
- **Violation Behavior:** Reject request
- **Error Code:** `CANNOT_CANCEL_PAST_BOOKING`

**Traceability:**
- User Story: user-stories.md#us-book-002
- Acceptance Criteria: AC-BOOK-002-5

---

### BR-BOOK-007 – Cancellation policy warning

**Rule:**
When a booking is cancelled within the cancellation policy window (default: 24 hours before), the customer must be warned about potential fees and must confirm the action.

**Invariant:**
```
IF Booking.timeSlot.startTime - NOW() < CancellationPolicy.window
THEN Booking.cancel() requires explicit confirmation
```

**Enforcement:**
- **Precondition:** Check if within policy window
- **Violation Behavior:** Require confirmation with fee warning
- **Error Code:** N/A (not a rejection, but a confirmation flow)

**Decision Points Resolved:**
- EH-2: Warning with fee notification, allow with confirmation (User answer: 5b)

**Traceability:**
- User Story: user-stories.md#us-book-002
- Acceptance Criteria: AC-BOOK-002-3
- Entity: ENT-CancellationPolicy

---

## US-BOOK-003 – Customer reschedules a booking

### BR-BOOK-008 – Atomic reschedule operation

**Rule:**
Rescheduling a booking is an atomic operation: either both the old slot is freed AND the new slot is reserved, or neither happens.

**Invariant:**
```
Booking.reschedule(newTimeSlot) is ATOMIC:
  Transaction {
    1. Validate newTimeSlot.isAvailable
    2. Free oldTimeSlot
    3. Reserve newTimeSlot
    4. Update Booking.timeSlot
  }
  IF any step fails → ROLLBACK all
```

**Enforcement:**
- **Precondition:** New slot must be available
- **Violation Behavior:** Reject, keep original booking unchanged
- **Error Code:** `NEW_TIME_SLOT_UNAVAILABLE`

**Decision Points Resolved:**
- SE-1: Atomic reschedule operation (User answer: 4a)

**Traceability:**
- User Story: user-stories.md#us-book-003
- Acceptance Criteria: AC-BOOK-003-1, AC-BOOK-003-2
- Entity: ENT-Booking, ENT-TimeSlot

---

### BR-BOOK-009 – Reschedule inherits cancellation policy

**Rule:**
Rescheduling within the cancellation policy window triggers the same warning as cancellation.

**Invariant:**
```
Booking.reschedule() follows same rules as Booking.cancel() for policy
```

**Enforcement:**
- Same as BR-BOOK-007

**Traceability:**
- User Story: user-stories.md#us-book-003
- Acceptance Criteria: AC-BOOK-003-3

---

## US-BOOK-004 – Provider sets availability

### BR-BOOK-010 – Availability changes don't affect existing bookings

**Rule:**
When a provider modifies their availability, existing bookings are NOT automatically cancelled or modified.

**Invariant:**
```
Availability.update() MUST NOT modify any existing Booking
```

**Enforcement:**
- **Precondition:** N/A
- **Behavior:** Warn provider about existing bookings in affected period
- **Error Code:** N/A (warning only)

**Traceability:**
- User Story: user-stories.md#us-book-004
- Acceptance Criteria: AC-BOOK-004-1, AC-BOOK-004-2
- Entity: ENT-Availability, ENT-Booking

---

### BR-BOOK-011 – Cannot modify past availability

**Rule:**
Provider cannot modify availability for past dates.

**Invariant:**
```
Availability.update(dateRange) requires dateRange.start > NOW()
```

**Enforcement:**
- **Precondition:** Check date range is in future
- **Violation Behavior:** Reject request
- **Error Code:** `CANNOT_MODIFY_PAST_AVAILABILITY`

**Traceability:**
- User Story: user-stories.md#us-book-004
- Acceptance Criteria: AC-BOOK-004-3

---

## US-BOOK-005 – System sends booking reminder

### BR-BOOK-012 – Reminders only for confirmed bookings

**Rule:**
Reminders are only sent for bookings with status `Confirmed`.

**Invariant:**
```
Reminder.send() ONLY when Booking.status === 'Confirmed'
```

**Enforcement:**
- **Precondition:** Check booking status before sending
- **Behavior:** Skip reminder if status is not Confirmed

**Traceability:**
- User Story: user-stories.md#us-book-005
- Acceptance Criteria: AC-BOOK-005-1, AC-BOOK-005-2
- Entity: ENT-Reminder, ENT-Booking

---

### BR-BOOK-013 – Customer reminder preference

**Rule:**
Customers can opt out of reminders. The system respects this preference.

**Invariant:**
```
IF Customer.reminderOptOut === true
THEN Reminder.send() is skipped
```

**Enforcement:**
- **Precondition:** Check customer preference
- **Behavior:** Skip reminder if opted out

**Traceability:**
- User Story: user-stories.md#us-book-005
- Acceptance Criteria: AC-BOOK-005-3
- Entity: ENT-Customer

---

## Summary

| User Story | BRs | Key Constraints |
|------------|-----|-----------------|
| US-BOOK-001 | 3 | Slot available, future only, auth required |
| US-BOOK-002 | 4 | Customer/Provider auth, status constraint, policy |
| US-BOOK-003 | 2 | Atomic operation, policy applies |
| US-BOOK-004 | 2 | No impact on existing, future only |
| US-BOOK-005 | 2 | Confirmed only, opt-out respected |
| **Total** | **13** | |

---

## Error Codes Reference

| Error Code | HTTP | Description |
|------------|------|-------------|
| TIME_SLOT_UNAVAILABLE | 409 | Slot already booked (race condition) |
| PAST_TIME_SLOT | 400 | Cannot book past time |
| AUTHENTICATION_REQUIRED | 401 | Must be logged in |
| NOT_AUTHORIZED_TO_CANCEL | 403 | Only customer/provider can cancel |
| INVALID_BOOKING_STATUS | 400 | Status doesn't allow operation |
| CANNOT_CANCEL_PAST_BOOKING | 400 | Past bookings are locked |
| NEW_TIME_SLOT_UNAVAILABLE | 409 | Reschedule target not available |
| CANNOT_MODIFY_PAST_AVAILABILITY | 400 | Past dates are locked |

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
  decisions:
    EH-1: blocking-error
    AU-1: customer-or-provider
    ST-1: pending-and-confirmed
    SE-1: atomic-reschedule
    EH-2: warning-with-fee
---

# Acceptance Criteria – Appointment Booking System

## US-BOOK-001 – Customer creates a booking

### AC-BOOK-001-1 – Create booking with available slot

**Given** a customer is authenticated
**And** a provider offers a service
**And** a time slot is available (not booked, within provider availability)
**When** the customer selects the service, provider, date, and time slot
**And** submits the booking request
**Then** the system creates a new Booking with status `Pending`
**And** the time slot is marked as reserved
**And** confirmation is sent to the customer (email)
**And** notification is sent to the provider
**And** the booking details are returned with bookingId

**Decision Points Resolved:**
- SE-2: Audit trail created (From input)

**Traceability:**
- User Story: user-stories.md#us-book-001
- Entity: ENT-Booking, ENT-Customer, ENT-Provider, ENT-Service

---

### AC-BOOK-001-2 – Reject booking for unavailable slot

**Given** a customer is authenticated
**And** a time slot was available when the customer started
**When** the customer submits the booking request
**And** the time slot has become unavailable (race condition)
**Then** the system rejects the booking
**And** returns error "Time slot no longer available" (HTTP 409 Conflict)
**And** no booking is created
**And** no notifications are sent

**Decision Points Resolved:**
- EH-1: Blocking error for race condition (User answer: 1a)

**Traceability:**
- User Story: user-stories.md#us-book-001
- Entity: ENT-Booking, ENT-TimeSlot

---

### AC-BOOK-001-3 – Reject booking for past time slot

**Given** a customer is authenticated
**When** the customer attempts to book a time slot in the past
**Then** the system rejects the booking
**And** returns error "Cannot book past time slots" (HTTP 400)

**Traceability:**
- User Story: user-stories.md#us-book-001

---

### AC-BOOK-001-4 – Reject booking outside provider availability

**Given** a customer is authenticated
**And** the requested time is outside the provider's availability
**When** the customer submits the booking request
**Then** the system rejects the booking
**And** returns error "Provider not available at this time" (HTTP 400)

**Traceability:**
- User Story: user-stories.md#us-book-001
- Entity: ENT-Availability

---

## US-BOOK-002 – Customer cancels a booking

### AC-BOOK-002-1 – Cancel booking by customer

**Given** a booking exists with status `Pending` or `Confirmed`
**And** the booking belongs to the authenticated customer
**When** the customer requests to cancel the booking
**Then** the booking status changes to `Cancelled`
**And** the time slot becomes available again
**And** cancellation confirmation is sent to the customer
**And** notification is sent to the provider
**And** audit log entry is created

**Decision Points Resolved:**
- AU-1: Customer can cancel own booking (User answer: 2b)
- ST-1: Pending and Confirmed can be cancelled (User answer: 3b)

**Traceability:**
- User Story: user-stories.md#us-book-002
- Entity: ENT-Booking

---

### AC-BOOK-002-2 – Cancel booking by provider

**Given** a booking exists with status `Pending` or `Confirmed`
**And** the booking is with the authenticated provider
**When** the provider requests to cancel the booking
**Then** the booking status changes to `Cancelled`
**And** the time slot becomes available again
**And** notification is sent to the customer (with apology)
**And** audit log entry is created with provider as actor

**Decision Points Resolved:**
- AU-1: Provider can also cancel (User answer: 2b)

**Traceability:**
- User Story: user-stories.md#us-book-002
- Entity: ENT-Booking, ENT-Provider

---

### AC-BOOK-002-3 – Warn about cancellation policy violation

**Given** a booking exists with status `Confirmed`
**And** the booking is within the cancellation policy window (e.g., 24 hours)
**When** the customer requests to cancel the booking
**Then** the system shows a warning: "Cancellation within 24 hours may incur a fee"
**And** the customer must confirm to proceed
**And** upon confirmation, the booking is cancelled
**And** the policy violation is recorded in the audit log

**Decision Points Resolved:**
- EH-2: Warning with fee notification, but allow cancellation (User answer: 5b)

**Traceability:**
- User Story: user-stories.md#us-book-002
- Entity: ENT-Booking, ENT-CancellationPolicy

---

### AC-BOOK-002-4 – Reject cancellation of completed booking

**Given** a booking exists with status `Completed`
**When** the customer attempts to cancel the booking
**Then** the system rejects the cancellation
**And** returns error "Cannot cancel completed bookings" (HTTP 400)

**Decision Points Resolved:**
- ST-1: Completed bookings cannot be cancelled (User answer: 3b)

**Traceability:**
- User Story: user-stories.md#us-book-002

---

### AC-BOOK-002-5 – Reject cancellation of past booking

**Given** a booking exists for a time slot in the past
**And** the booking status is not `Completed` or `Cancelled`
**When** the customer attempts to cancel
**Then** the system rejects the cancellation
**And** returns error "Cannot cancel past bookings" (HTTP 400)

**Traceability:**
- User Story: user-stories.md#us-book-002

---

## US-BOOK-003 – Customer reschedules a booking

### AC-BOOK-003-1 – Reschedule booking atomically

**Given** a booking exists with status `Pending` or `Confirmed`
**And** the booking belongs to the authenticated customer
**And** a new time slot is available
**When** the customer requests to reschedule to the new time slot
**Then** the system performs an atomic operation:
  - Original time slot is freed
  - New time slot is reserved
  - Booking is updated with new time
**And** confirmation is sent to the customer
**And** notification is sent to the provider
**And** audit log entry is created with old and new times

**Decision Points Resolved:**
- SE-1: Atomic reschedule operation (User answer: 4a)

**Traceability:**
- User Story: user-stories.md#us-book-003
- Entity: ENT-Booking, ENT-TimeSlot

---

### AC-BOOK-003-2 – Reject reschedule if new slot unavailable

**Given** a booking exists
**And** the customer requests to reschedule
**When** the new time slot is not available
**Then** the system rejects the reschedule
**And** returns error "New time slot not available" (HTTP 409)
**And** the original booking remains unchanged
**And** the original time slot remains reserved

**Decision Points Resolved:**
- SE-1: Atomic - if new slot fails, nothing changes (User answer: 4a)

**Traceability:**
- User Story: user-stories.md#us-book-003

---

### AC-BOOK-003-3 – Reschedule respects cancellation policy

**Given** a booking is within the cancellation policy window
**When** the customer requests to reschedule
**Then** the same warning is shown as for cancellation
**And** upon confirmation, the reschedule proceeds

**Traceability:**
- User Story: user-stories.md#us-book-003
- Entity: ENT-CancellationPolicy

---

## US-BOOK-004 – Provider sets availability

### AC-BOOK-004-1 – Set weekly recurring availability

**Given** a provider is authenticated
**When** the provider defines weekly availability (e.g., Mon-Fri 9:00-17:00)
**Then** the system saves the recurring schedule
**And** future time slots are generated based on this schedule
**And** existing bookings are NOT affected

**Traceability:**
- User Story: user-stories.md#us-book-004
- Entity: ENT-Availability, ENT-Provider

---

### AC-BOOK-004-2 – Set one-off unavailability (block time)

**Given** a provider is authenticated
**And** the provider has bookings on a specific day
**When** the provider marks a future time period as unavailable
**Then** the system blocks new bookings for that period
**And** existing bookings in that period are NOT automatically cancelled
**And** the provider is warned about existing bookings

**Traceability:**
- User Story: user-stories.md#us-book-004
- Entity: ENT-Availability

---

### AC-BOOK-004-3 – Cannot set availability in the past

**Given** a provider is authenticated
**When** the provider attempts to modify availability for past dates
**Then** the system rejects the change
**And** returns error "Cannot modify past availability" (HTTP 400)

**Traceability:**
- User Story: user-stories.md#us-book-004

---

## US-BOOK-005 – System sends booking reminder

### AC-BOOK-005-1 – Send reminder before appointment

**Given** a booking exists with status `Confirmed`
**And** the booking time is approaching (e.g., 24 hours away)
**When** the reminder trigger time is reached
**Then** the system sends a reminder to the customer
**And** the reminder includes:
  - Booking date and time
  - Provider name
  - Service name
  - Location/address (if applicable)
  - Cancellation link

**Traceability:**
- User Story: user-stories.md#us-book-005
- Entity: ENT-Booking, ENT-Reminder

---

### AC-BOOK-005-2 – Do not send reminder for cancelled bookings

**Given** a booking was scheduled for reminder
**And** the booking status has changed to `Cancelled`
**When** the reminder trigger time is reached
**Then** the system does NOT send the reminder

**Traceability:**
- User Story: user-stories.md#us-book-005

---

### AC-BOOK-005-3 – Customer can opt out of reminders

**Given** a customer has booking reminders enabled (default)
**When** the customer disables reminders in preferences
**Then** no reminders are sent for that customer's bookings
**And** the preference is stored in customer profile

**Traceability:**
- User Story: user-stories.md#us-book-005
- Entity: ENT-Customer

---

## Summary

| User Story | ACs | Decision Points |
|------------|-----|-----------------|
| US-BOOK-001 | 4 | EH-1 (1a) |
| US-BOOK-002 | 5 | AU-1 (2b), ST-1 (3b), EH-2 (5b) |
| US-BOOK-003 | 3 | SE-1 (4a) |
| US-BOOK-004 | 3 | - |
| US-BOOK-005 | 3 | - |
| **Total** | **18** | **5 from user** |

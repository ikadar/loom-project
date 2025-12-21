# User Stories – Appointment Booking System

Ez a dokumentum az Appointment Booking System user story-jait tartalmazza.

---

## US-BOOK-001 – Customer creates a booking {#us-book-001}

**As a** customer
**I want** to book an appointment with a provider
**So that** I can receive the service at a convenient time.

### Context
- Customer selects a service, provider, and available time slot
- System checks availability and creates the booking
- Confirmation is sent to both customer and provider

### Notes
- Only future time slots can be booked
- Customer must be registered and logged in

---

## US-BOOK-002 – Customer cancels a booking {#us-book-002}

**As a** customer
**I want** to cancel my booking
**So that** I can free up the time slot if I cannot attend.

### Context
- Customer can cancel upcoming bookings
- Cancellation rules may apply (e.g., 24h policy)
- Time slot becomes available again for others

### Notes
- Past bookings cannot be cancelled
- Consider refund handling if prepaid

---

## US-BOOK-003 – Customer reschedules a booking {#us-book-003}

**As a** customer
**I want** to change the date/time of my booking
**So that** I can still receive the service at a different time.

### Context
- Customer selects a new available time slot
- Original slot is freed, new slot is reserved
- Both provider and customer are notified

### Notes
- Should work like cancel + new booking, or atomic operation?
- Reschedule limits may apply

---

## US-BOOK-004 – Provider sets availability {#us-book-004}

**As a** provider
**I want** to define my available hours
**So that** customers can only book when I'm available.

### Context
- Provider sets weekly recurring schedule
- Can also set one-off available/unavailable periods
- Existing bookings are not affected by availability changes

### Notes
- Default: Monday-Friday 9:00-17:00?
- Breaks (lunch) handling?

---

## US-BOOK-005 – System sends booking reminder {#us-book-005}

**As a** system
**I want** to send reminders before appointments
**So that** customers don't forget their bookings.

### Context
- Reminder sent X hours before appointment
- Via email and/or push notification
- Contains booking details and cancellation link

### Notes
- Default reminder time: 24 hours before?
- Customer can opt out?

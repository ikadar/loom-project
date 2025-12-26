---
status: draft
derived-from:
  - "loom-project/poc/booking-system/output/acceptance-criteria.md"
  - "loom-project/poc/booking-system/output/business-rules.md"
  - "loom-project/poc/booking-system/output/domain-model.md"
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

# Interface Contracts – Appointment Booking System

## Overview

This document defines the API contracts for the Appointment Booking System.

**Base URL:** `/api/v1`
**Authentication:** JWT Bearer Token (SI decision: API-2)
**API Style:** RESTful with resource URLs (SI decision: API-1)
**Error Format:** RFC 7807 Problem Details (SI decision: ERR-1)

---

## Authentication

All endpoints require JWT Bearer token unless marked as `[Public]`.

```http
Authorization: Bearer <jwt-token>
```

**JWT Claims:**
```json
{
  "sub": "user-id",
  "role": "customer|provider",
  "email": "user@example.com",
  "exp": 1703188800
}
```

---

## Booking Endpoints

### POST /bookings

Create a new booking.

**Request:**
```http
POST /api/v1/bookings
Content-Type: application/json
Authorization: Bearer <token>

{
  "providerId": "prov-123",
  "serviceId": "svc-456",
  "timeSlot": {
    "startTime": "2025-01-15T10:00:00Z",
    "endTime": "2025-01-15T11:00:00Z"
  }
}
```

**Response (201 Created):**
```json
{
  "bookingId": "book-789",
  "customerId": "cust-001",
  "providerId": "prov-123",
  "serviceId": "svc-456",
  "timeSlot": {
    "startTime": "2025-01-15T10:00:00Z",
    "endTime": "2025-01-15T11:00:00Z"
  },
  "status": "pending",
  "createdAt": "2025-01-10T14:30:00Z"
}
```

**Error Responses:**

| Status | Problem Type | Condition |
|--------|--------------|-----------|
| 401 | `authentication-required` | Missing/invalid token |
| 400 | `past-time-slot` | Time slot in the past |
| 400 | `provider-not-available` | Outside provider hours |
| 409 | `time-slot-unavailable` | Slot already booked |

**Concurrency:** Pessimistic locking on time slot (SI decision: CON-1)

**Traceability:**
- AC: AC-BOOK-001-1, AC-BOOK-001-2, AC-BOOK-001-3, AC-BOOK-001-4
- BR: BR-BOOK-001, BR-BOOK-002, BR-BOOK-003

---

### GET /bookings/{bookingId}

Retrieve booking details.

**Request:**
```http
GET /api/v1/bookings/book-789
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "bookingId": "book-789",
  "customerId": "cust-001",
  "providerId": "prov-123",
  "serviceId": "svc-456",
  "service": {
    "name": "Haircut",
    "duration": 60,
    "price": { "amount": 50.00, "currency": "EUR" }
  },
  "provider": {
    "name": "John's Salon",
    "email": "john@salon.com"
  },
  "timeSlot": {
    "startTime": "2025-01-15T10:00:00Z",
    "endTime": "2025-01-15T11:00:00Z"
  },
  "status": "confirmed",
  "createdAt": "2025-01-10T14:30:00Z",
  "updatedAt": "2025-01-10T14:35:00Z"
}
```

**Error Responses:**

| Status | Problem Type | Condition |
|--------|--------------|-----------|
| 404 | `booking-not-found` | Booking doesn't exist |
| 403 | `access-denied` | Not owner or provider |

---

### GET /bookings

List bookings for authenticated user.

**Request:**
```http
GET /api/v1/bookings?status=confirmed&from=2025-01-01&to=2025-01-31
Authorization: Bearer <token>
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| status | string | Filter by status (optional) |
| from | date | Start date filter (optional) |
| to | date | End date filter (optional) |
| page | int | Page number (default: 1) |
| limit | int | Items per page (default: 20, max: 100) |

**Response (200 OK):**
```json
{
  "items": [
    { "bookingId": "book-789", "..." : "..." }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "totalPages": 3
  }
}
```

---

### DELETE /bookings/{bookingId}

Cancel a booking.

**Request:**
```http
DELETE /api/v1/bookings/book-789
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "bookingId": "book-789",
  "status": "cancelled",
  "cancelledAt": "2025-01-12T09:00:00Z",
  "cancelledBy": "customer"
}
```

**Response (200 OK with warning):**
```json
{
  "bookingId": "book-789",
  "status": "cancelled",
  "cancelledAt": "2025-01-14T18:00:00Z",
  "cancelledBy": "customer",
  "policyViolation": {
    "type": "late-cancellation",
    "message": "Cancelled within 24 hours of appointment",
    "potentialFee": { "amount": 25.00, "currency": "EUR" }
  }
}
```

**Error Responses:**

| Status | Problem Type | Condition |
|--------|--------------|-----------|
| 403 | `not-authorized-to-cancel` | Not owner or provider |
| 400 | `invalid-booking-status` | Already cancelled/completed |
| 400 | `cannot-cancel-past-booking` | Booking in the past |

**Confirmation Flow:**
For cancellations within policy window, client should first call with `?dryRun=true`:

```http
DELETE /api/v1/bookings/book-789?dryRun=true
```

Returns warning without cancelling. Client shows warning, user confirms, then calls without dryRun.

**Traceability:**
- AC: AC-BOOK-002-1, AC-BOOK-002-2, AC-BOOK-002-3, AC-BOOK-002-4, AC-BOOK-002-5
- BR: BR-BOOK-004, BR-BOOK-005, BR-BOOK-006, BR-BOOK-007

---

### PUT /bookings/{bookingId}/reschedule

Reschedule a booking to a new time slot.

**Request:**
```http
PUT /api/v1/bookings/book-789/reschedule
Content-Type: application/json
Authorization: Bearer <token>

{
  "newTimeSlot": {
    "startTime": "2025-01-16T14:00:00Z",
    "endTime": "2025-01-16T15:00:00Z"
  }
}
```

**Response (200 OK):**
```json
{
  "bookingId": "book-789",
  "previousTimeSlot": {
    "startTime": "2025-01-15T10:00:00Z",
    "endTime": "2025-01-15T11:00:00Z"
  },
  "newTimeSlot": {
    "startTime": "2025-01-16T14:00:00Z",
    "endTime": "2025-01-16T15:00:00Z"
  },
  "status": "confirmed",
  "rescheduledAt": "2025-01-12T10:00:00Z"
}
```

**Error Responses:**

| Status | Problem Type | Condition |
|--------|--------------|-----------|
| 409 | `new-time-slot-unavailable` | New slot not available |
| 400 | `invalid-booking-status` | Cannot reschedule this status |

**Atomicity:** Transaction ensures old slot freed AND new slot reserved, or neither (SI decision: CON-1)

**Traceability:**
- AC: AC-BOOK-003-1, AC-BOOK-003-2, AC-BOOK-003-3
- BR: BR-BOOK-008, BR-BOOK-009

---

## Availability Endpoints

### GET /providers/{providerId}/availability

Get available time slots for a provider.

**Request:**
```http
GET /api/v1/providers/prov-123/availability?date=2025-01-15&serviceId=svc-456
Authorization: Bearer <token>
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| date | date | Yes | Date to check |
| serviceId | string | Yes | Service (for duration) |

**Response (200 OK):**
```json
{
  "providerId": "prov-123",
  "date": "2025-01-15",
  "serviceDuration": 60,
  "availableSlots": [
    { "startTime": "2025-01-15T09:00:00Z", "endTime": "2025-01-15T10:00:00Z" },
    { "startTime": "2025-01-15T10:00:00Z", "endTime": "2025-01-15T11:00:00Z" },
    { "startTime": "2025-01-15T14:00:00Z", "endTime": "2025-01-15T15:00:00Z" }
  ]
}
```

---

### PUT /providers/{providerId}/calendar

Update provider's availability schedule.

**Request:**
```http
PUT /api/v1/providers/prov-123/calendar
Content-Type: application/json
Authorization: Bearer <token>

{
  "weeklySchedule": {
    "monday": [{ "start": "09:00", "end": "17:00" }],
    "tuesday": [{ "start": "09:00", "end": "17:00" }],
    "wednesday": [{ "start": "09:00", "end": "13:00" }],
    "thursday": [{ "start": "09:00", "end": "17:00" }],
    "friday": [{ "start": "09:00", "end": "17:00" }],
    "saturday": [],
    "sunday": []
  }
}
```

**Response (200 OK):**
```json
{
  "calendarId": "cal-123",
  "providerId": "prov-123",
  "weeklySchedule": { "..." : "..." },
  "updatedAt": "2025-01-10T12:00:00Z",
  "warnings": [
    {
      "type": "existing-bookings",
      "message": "3 existing bookings in affected time periods",
      "affectedBookings": ["book-101", "book-102", "book-103"]
    }
  ]
}
```

**Error Responses:**

| Status | Problem Type | Condition |
|--------|--------------|-----------|
| 400 | `cannot-modify-past-availability` | Trying to modify past dates |
| 403 | `access-denied` | Not the provider |

**Traceability:**
- AC: AC-BOOK-004-1, AC-BOOK-004-2, AC-BOOK-004-3
- BR: BR-BOOK-010, BR-BOOK-011

---

### POST /providers/{providerId}/calendar/exceptions

Add availability exception (block time off).

**Request:**
```http
POST /api/v1/providers/prov-123/calendar/exceptions
Content-Type: application/json
Authorization: Bearer <token>

{
  "date": "2025-01-20",
  "available": false,
  "reason": "Holiday"
}
```

**Response (201 Created):**
```json
{
  "exceptionId": "exc-001",
  "date": "2025-01-20",
  "available": false,
  "reason": "Holiday",
  "warnings": []
}
```

---

## Customer Endpoints

### GET /customers/me/preferences

Get customer preferences.

**Request:**
```http
GET /api/v1/customers/me/preferences
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "reminderOptOut": false,
  "preferredNotificationChannel": "email"
}
```

---

### PATCH /customers/me/preferences

Update customer preferences.

**Request:**
```http
PATCH /api/v1/customers/me/preferences
Content-Type: application/json
Authorization: Bearer <token>

{
  "reminderOptOut": true
}
```

**Response (200 OK):**
```json
{
  "reminderOptOut": true,
  "preferredNotificationChannel": "email",
  "updatedAt": "2025-01-10T15:00:00Z"
}
```

**Traceability:**
- AC: AC-BOOK-005-3
- BR: BR-BOOK-013

---

## Error Response Format

All errors follow RFC 7807 Problem Details (SI decision: ERR-1).

```json
{
  "type": "https://api.booking.com/problems/time-slot-unavailable",
  "title": "Time Slot Unavailable",
  "status": 409,
  "detail": "The requested time slot 2025-01-15T10:00:00Z is no longer available.",
  "instance": "/api/v1/bookings",
  "bookingId": null,
  "requestedSlot": {
    "startTime": "2025-01-15T10:00:00Z",
    "endTime": "2025-01-15T11:00:00Z"
  }
}
```

**Standard Problem Types:**

| Type | Title | Status |
|------|-------|--------|
| `authentication-required` | Authentication Required | 401 |
| `access-denied` | Access Denied | 403 |
| `booking-not-found` | Booking Not Found | 404 |
| `time-slot-unavailable` | Time Slot Unavailable | 409 |
| `new-time-slot-unavailable` | New Time Slot Unavailable | 409 |
| `past-time-slot` | Past Time Slot | 400 |
| `provider-not-available` | Provider Not Available | 400 |
| `not-authorized-to-cancel` | Not Authorized to Cancel | 403 |
| `invalid-booking-status` | Invalid Booking Status | 400 |
| `cannot-cancel-past-booking` | Cannot Cancel Past Booking | 400 |
| `cannot-modify-past-availability` | Cannot Modify Past Availability | 400 |

---

## Event Contracts

Communication follows hybrid pattern (SI decision: COM-1):
- **Sync:** Critical operations (booking creation, cancellation)
- **Async:** Notifications, reminders

### Async Events (Message Queue)

**BookingCreated:**
```json
{
  "eventType": "booking.created",
  "eventId": "evt-001",
  "timestamp": "2025-01-10T14:30:00Z",
  "payload": {
    "bookingId": "book-789",
    "customerId": "cust-001",
    "customerEmail": "customer@example.com",
    "providerId": "prov-123",
    "providerEmail": "provider@example.com",
    "serviceId": "svc-456",
    "serviceName": "Haircut",
    "timeSlot": {
      "startTime": "2025-01-15T10:00:00Z",
      "endTime": "2025-01-15T11:00:00Z"
    }
  }
}
```

**BookingCancelled:**
```json
{
  "eventType": "booking.cancelled",
  "eventId": "evt-002",
  "timestamp": "2025-01-12T09:00:00Z",
  "payload": {
    "bookingId": "book-789",
    "cancelledBy": "customer",
    "customerEmail": "customer@example.com",
    "providerEmail": "provider@example.com",
    "policyViolation": false
  }
}
```

**BookingRescheduled:**
```json
{
  "eventType": "booking.rescheduled",
  "eventId": "evt-003",
  "timestamp": "2025-01-12T10:00:00Z",
  "payload": {
    "bookingId": "book-789",
    "customerEmail": "customer@example.com",
    "providerEmail": "provider@example.com",
    "previousTimeSlot": { "..." : "..." },
    "newTimeSlot": { "..." : "..." }
  }
}
```

**ReminderDue:**
```json
{
  "eventType": "reminder.due",
  "eventId": "evt-004",
  "timestamp": "2025-01-14T10:00:00Z",
  "payload": {
    "bookingId": "book-789",
    "customerEmail": "customer@example.com",
    "customerOptedOut": false,
    "bookingTime": "2025-01-15T10:00:00Z",
    "serviceName": "Haircut",
    "providerName": "John's Salon"
  }
}
```

**Traceability:**
- AC: AC-BOOK-005-1, AC-BOOK-005-2
- BR: BR-BOOK-012, BR-BOOK-013

---

## Summary

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/bookings` | POST | Required | Create booking |
| `/bookings` | GET | Required | List user's bookings |
| `/bookings/{id}` | GET | Required | Get booking details |
| `/bookings/{id}` | DELETE | Required | Cancel booking |
| `/bookings/{id}/reschedule` | PUT | Required | Reschedule booking |
| `/providers/{id}/availability` | GET | Required | Get available slots |
| `/providers/{id}/calendar` | PUT | Required | Update schedule |
| `/providers/{id}/calendar/exceptions` | POST | Required | Add exception |
| `/customers/me/preferences` | GET | Required | Get preferences |
| `/customers/me/preferences` | PATCH | Required | Update preferences |

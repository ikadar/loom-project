---
status: draft
derived-from:
  - "loom-project/poc/booking-system/output/acceptance-criteria.md"
  - "loom-project/poc/booking-system/output/business-rules.md"
  - "loom-project/poc/booking-system/output/interface-contracts.md"
  - "loom-project/poc/booking-system/output/sequence-design.md"
derived-at: "2025-12-21T17:30:00Z"
derived-by: "loom-derive-l3 skill v2.0 (Structured Interview)"
loom-version: "3.0.0"
structured-interview:
  decision-points-resolved: 5
  from-user-answers: 5
  decisions:
    TST-1: testcontainers
    TST-2: builders-factories
    TST-3: integration-tests
    TST-4: all-error-codes
    TST-5: explicit-concurrency-tests
---

# Test Cases – Appointment Booking System

## Overview

This document defines the test cases for the Appointment Booking System.

**Test Strategy:**
- Infrastructure: Testcontainers (SI decision: TST-1)
- Test Data: Builders/Factories (SI decision: TST-2)
- API Tests: Integration tests with real HTTP (SI decision: TST-3)
- Error Coverage: Every error code tested (SI decision: TST-4)
- Concurrency: Explicit parallel thread tests (SI decision: TST-5)

---

## Test Infrastructure

### Testcontainers Setup

```java
@Testcontainers
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class BookingIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("booking_test");

    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7")
        .withExposedPorts(6379);

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.redis.host", redis::getHost);
        registry.add("spring.redis.port", redis::getFirstMappedPort);
    }
}
```

### Test Data Builders

```java
class BookingBuilder {
    private String customerId = "cust-001";
    private String providerId = "prov-001";
    private String serviceId = "svc-001";
    private LocalDateTime startTime = LocalDateTime.now().plusDays(1);
    private Duration duration = Duration.ofHours(1);
    private BookingStatus status = BookingStatus.PENDING;

    static BookingBuilder aBooking() { return new BookingBuilder(); }

    BookingBuilder withCustomer(String id) { this.customerId = id; return this; }
    BookingBuilder withProvider(String id) { this.providerId = id; return this; }
    BookingBuilder withService(String id) { this.serviceId = id; return this; }
    BookingBuilder withStartTime(LocalDateTime time) { this.startTime = time; return this; }
    BookingBuilder withStatus(BookingStatus status) { this.status = status; return this; }
    BookingBuilder tomorrow() { this.startTime = LocalDateTime.now().plusDays(1).withHour(10); return this; }
    BookingBuilder inPast() { this.startTime = LocalDateTime.now().minusDays(1); return this; }

    Booking build() { /* ... */ }
    CreateBookingRequest buildRequest() { /* ... */ }
}

class ProviderBuilder {
    static ProviderBuilder aProvider() { return new ProviderBuilder(); }
    ProviderBuilder withWeeklySchedule(WeeklySchedule schedule) { /* ... */ }
    ProviderBuilder availableWeekdays9to17() { /* ... */ }
    Provider build() { /* ... */ }
}

class CustomerBuilder {
    static CustomerBuilder aCustomer() { return new CustomerBuilder(); }
    CustomerBuilder withReminderOptOut(boolean optOut) { /* ... */ }
    Customer build() { /* ... */ }
}
```

---

## US-BOOK-001: Create Booking

### TC-BOOK-001-01: Create booking successfully

**Type:** Positive | Integration
**Priority:** P0

**Preconditions:**
- Customer is authenticated
- Provider exists with available time slot
- Service exists

**Test Steps:**
```java
@Test
void shouldCreateBookingSuccessfully() {
    // Given
    var provider = given(aProvider().availableWeekdays9to17());
    var service = given(aService().withDuration(60));
    var customer = authenticatedAs(aCustomer());
    var slot = tomorrow().at(10, 0);

    // When
    var response = POST("/api/v1/bookings")
        .withAuth(customer.token())
        .body(aBookingRequest()
            .withProvider(provider.id())
            .withService(service.id())
            .withTimeSlot(slot))
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(201);
    assertThat(response.body().bookingId()).isNotNull();
    assertThat(response.body().status()).isEqualTo("pending");
    assertThat(response.body().timeSlot().startTime()).isEqualTo(slot.startTime());

    // And slot is now reserved
    var availability = GET("/api/v1/providers/{id}/availability?date={date}&serviceId={svc}",
        provider.id(), slot.date(), service.id())
        .execute();
    assertThat(availability.body().availableSlots())
        .doesNotContain(slot);
}
```

**Expected Result:**
- HTTP 201 Created
- Booking returned with ID and status "pending"
- Time slot no longer available

**Traceability:**
- AC: AC-BOOK-001-1
- BR: BR-BOOK-001

---

### TC-BOOK-001-02: Reject booking for unavailable slot (race condition)

**Type:** Negative | Concurrency
**Priority:** P0

**Test Steps:**
```java
@Test
void shouldRejectBookingWhenSlotTakenByRaceCondition() throws Exception {
    // Given
    var provider = given(aProvider().availableWeekdays9to17());
    var service = given(aService());
    var customer1 = authenticatedAs(aCustomer());
    var customer2 = authenticatedAs(aCustomer());
    var slot = tomorrow().at(10, 0);

    // When - two customers book same slot simultaneously
    var executor = Executors.newFixedThreadPool(2);
    var latch = new CountDownLatch(1);

    var future1 = executor.submit(() -> {
        latch.await();
        return POST("/api/v1/bookings")
            .withAuth(customer1.token())
            .body(aBookingRequest().withProvider(provider.id())
                .withService(service.id()).withTimeSlot(slot))
            .execute();
    });

    var future2 = executor.submit(() -> {
        latch.await();
        return POST("/api/v1/bookings")
            .withAuth(customer2.token())
            .body(aBookingRequest().withProvider(provider.id())
                .withService(service.id()).withTimeSlot(slot))
            .execute();
    });

    latch.countDown(); // Release both threads simultaneously

    var response1 = future1.get();
    var response2 = future2.get();

    // Then - exactly one succeeds, one fails
    var statuses = List.of(response1.status(), response2.status());
    assertThat(statuses).containsExactlyInAnyOrder(201, 409);

    var failedResponse = response1.status() == 409 ? response1 : response2;
    assertThat(failedResponse.body().type())
        .isEqualTo("https://api.booking.com/problems/time-slot-unavailable");
    assertThat(failedResponse.body().status()).isEqualTo(409);
}
```

**Expected Result:**
- One request: HTTP 201 Created
- Other request: HTTP 409 Conflict with TIME_SLOT_UNAVAILABLE

**Traceability:**
- AC: AC-BOOK-001-2
- BR: BR-BOOK-001
- SEQ: SEQ-002

---

### TC-BOOK-001-03: Reject booking for past time slot

**Type:** Negative | Integration
**Priority:** P1

**Test Steps:**
```java
@Test
void shouldRejectBookingForPastTimeSlot() {
    // Given
    var customer = authenticatedAs(aCustomer());
    var provider = given(aProvider());
    var service = given(aService());
    var pastSlot = yesterday().at(10, 0);

    // When
    var response = POST("/api/v1/bookings")
        .withAuth(customer.token())
        .body(aBookingRequest()
            .withProvider(provider.id())
            .withService(service.id())
            .withTimeSlot(pastSlot))
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(400);
    assertThat(response.body().type())
        .isEqualTo("https://api.booking.com/problems/past-time-slot");
}
```

**Expected Result:**
- HTTP 400 Bad Request
- Error type: past-time-slot

**Traceability:**
- AC: AC-BOOK-001-3
- BR: BR-BOOK-002

---

### TC-BOOK-001-04: Reject booking outside provider availability

**Type:** Negative | Integration
**Priority:** P1

**Test Steps:**
```java
@Test
void shouldRejectBookingOutsideProviderAvailability() {
    // Given
    var provider = given(aProvider().availableWeekdays9to17()); // Not available on weekends
    var customer = authenticatedAs(aCustomer());
    var service = given(aService());
    var saturdaySlot = nextSaturday().at(10, 0);

    // When
    var response = POST("/api/v1/bookings")
        .withAuth(customer.token())
        .body(aBookingRequest()
            .withProvider(provider.id())
            .withService(service.id())
            .withTimeSlot(saturdaySlot))
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(400);
    assertThat(response.body().type())
        .isEqualTo("https://api.booking.com/problems/provider-not-available");
}
```

**Expected Result:**
- HTTP 400 Bad Request
- Error type: provider-not-available

**Traceability:**
- AC: AC-BOOK-001-4
- BR: BR-BOOK-001

---

### TC-BOOK-001-05: Reject booking without authentication

**Type:** Negative | Security
**Priority:** P0

**Test Steps:**
```java
@Test
void shouldRejectBookingWithoutAuthentication() {
    // Given - no authentication token

    // When
    var response = POST("/api/v1/bookings")
        .withoutAuth()
        .body(aBookingRequest().build())
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(401);
    assertThat(response.body().type())
        .isEqualTo("https://api.booking.com/problems/authentication-required");
}
```

**Expected Result:**
- HTTP 401 Unauthorized
- Error type: authentication-required

**Traceability:**
- BR: BR-BOOK-003

---

## US-BOOK-002: Cancel Booking

### TC-BOOK-002-01: Customer cancels own booking

**Type:** Positive | Integration
**Priority:** P0

**Test Steps:**
```java
@Test
void shouldAllowCustomerToCancelOwnBooking() {
    // Given
    var customer = authenticatedAs(aCustomer());
    var booking = given(aBooking()
        .withCustomer(customer.id())
        .withStatus(CONFIRMED)
        .tomorrow());

    // When
    var response = DELETE("/api/v1/bookings/{id}", booking.id())
        .withAuth(customer.token())
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(200);
    assertThat(response.body().status()).isEqualTo("cancelled");
    assertThat(response.body().cancelledBy()).isEqualTo("customer");

    // And slot is available again
    var availability = getAvailability(booking.providerId(), booking.date());
    assertThat(availability.availableSlots()).contains(booking.timeSlot());
}
```

**Expected Result:**
- HTTP 200 OK
- Booking status changed to "cancelled"
- Time slot available again

**Traceability:**
- AC: AC-BOOK-002-1
- BR: BR-BOOK-004, BR-BOOK-005

---

### TC-BOOK-002-02: Provider cancels booking

**Type:** Positive | Integration
**Priority:** P0

**Test Steps:**
```java
@Test
void shouldAllowProviderToCancelBooking() {
    // Given
    var provider = authenticatedAs(aProvider());
    var booking = given(aBooking()
        .withProvider(provider.id())
        .withStatus(CONFIRMED)
        .tomorrow());

    // When
    var response = DELETE("/api/v1/bookings/{id}", booking.id())
        .withAuth(provider.token())
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(200);
    assertThat(response.body().status()).isEqualTo("cancelled");
    assertThat(response.body().cancelledBy()).isEqualTo("provider");
}
```

**Expected Result:**
- HTTP 200 OK
- Booking cancelled by provider

**Traceability:**
- AC: AC-BOOK-002-2
- BR: BR-BOOK-004

---

### TC-BOOK-002-03: Warn about cancellation policy violation

**Type:** Positive | Integration
**Priority:** P1

**Test Steps:**
```java
@Test
void shouldWarnAboutLateCancellation() {
    // Given
    var customer = authenticatedAs(aCustomer());
    var booking = given(aBooking()
        .withCustomer(customer.id())
        .withStatus(CONFIRMED)
        .withStartTime(now().plusHours(12))); // Within 24h window

    // When - dry run first
    var dryRunResponse = DELETE("/api/v1/bookings/{id}?dryRun=true", booking.id())
        .withAuth(customer.token())
        .execute();

    // Then - warning returned
    assertThat(dryRunResponse.status()).isEqualTo(200);
    assertThat(dryRunResponse.body().policyWarning()).isNotNull();
    assertThat(dryRunResponse.body().policyWarning().type()).isEqualTo("late-cancellation");
    assertThat(dryRunResponse.body().policyWarning().potentialFee().amount())
        .isGreaterThan(0);

    // When - actual cancellation after user confirms
    var response = DELETE("/api/v1/bookings/{id}", booking.id())
        .withAuth(customer.token())
        .execute();

    // Then - cancelled with policy violation recorded
    assertThat(response.status()).isEqualTo(200);
    assertThat(response.body().status()).isEqualTo("cancelled");
    assertThat(response.body().policyViolation()).isNotNull();
}
```

**Expected Result:**
- Dry run: HTTP 200 with warning
- Actual call: HTTP 200, cancelled with policy violation

**Traceability:**
- AC: AC-BOOK-002-3
- BR: BR-BOOK-007

---

### TC-BOOK-002-04: Reject cancellation by unauthorized user

**Type:** Negative | Security
**Priority:** P0

**Test Steps:**
```java
@Test
void shouldRejectCancellationByUnauthorizedUser() {
    // Given
    var customer1 = authenticatedAs(aCustomer());
    var customer2 = authenticatedAs(aCustomer()); // Different customer
    var booking = given(aBooking()
        .withCustomer(customer1.id())
        .withStatus(CONFIRMED));

    // When - customer2 tries to cancel customer1's booking
    var response = DELETE("/api/v1/bookings/{id}", booking.id())
        .withAuth(customer2.token())
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(403);
    assertThat(response.body().type())
        .isEqualTo("https://api.booking.com/problems/not-authorized-to-cancel");
}
```

**Expected Result:**
- HTTP 403 Forbidden
- Error type: not-authorized-to-cancel

**Traceability:**
- BR: BR-BOOK-004

---

### TC-BOOK-002-05: Reject cancellation of completed booking

**Type:** Negative | Integration
**Priority:** P1

**Test Steps:**
```java
@Test
void shouldRejectCancellationOfCompletedBooking() {
    // Given
    var customer = authenticatedAs(aCustomer());
    var booking = given(aBooking()
        .withCustomer(customer.id())
        .withStatus(COMPLETED));

    // When
    var response = DELETE("/api/v1/bookings/{id}", booking.id())
        .withAuth(customer.token())
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(400);
    assertThat(response.body().type())
        .isEqualTo("https://api.booking.com/problems/invalid-booking-status");
}
```

**Expected Result:**
- HTTP 400 Bad Request
- Error type: invalid-booking-status

**Traceability:**
- AC: AC-BOOK-002-4
- BR: BR-BOOK-005

---

### TC-BOOK-002-06: Reject cancellation of past booking

**Type:** Negative | Integration
**Priority:** P1

**Test Steps:**
```java
@Test
void shouldRejectCancellationOfPastBooking() {
    // Given
    var customer = authenticatedAs(aCustomer());
    var booking = given(aBooking()
        .withCustomer(customer.id())
        .withStatus(CONFIRMED)
        .inPast()); // TimeSlot in the past

    // When
    var response = DELETE("/api/v1/bookings/{id}", booking.id())
        .withAuth(customer.token())
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(400);
    assertThat(response.body().type())
        .isEqualTo("https://api.booking.com/problems/cannot-cancel-past-booking");
}
```

**Expected Result:**
- HTTP 400 Bad Request
- Error type: cannot-cancel-past-booking

**Traceability:**
- AC: AC-BOOK-002-5
- BR: BR-BOOK-006

---

## US-BOOK-003: Reschedule Booking

### TC-BOOK-003-01: Reschedule booking atomically

**Type:** Positive | Integration
**Priority:** P0

**Test Steps:**
```java
@Test
void shouldRescheduleBookingAtomically() {
    // Given
    var customer = authenticatedAs(aCustomer());
    var oldSlot = tomorrow().at(10, 0);
    var newSlot = tomorrow().at(14, 0);
    var booking = given(aBooking()
        .withCustomer(customer.id())
        .withTimeSlot(oldSlot)
        .withStatus(CONFIRMED));

    // When
    var response = PUT("/api/v1/bookings/{id}/reschedule", booking.id())
        .withAuth(customer.token())
        .body(new RescheduleRequest(newSlot))
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(200);
    assertThat(response.body().previousTimeSlot()).isEqualTo(oldSlot);
    assertThat(response.body().newTimeSlot()).isEqualTo(newSlot);

    // And old slot is available, new slot is not
    var availability = getAvailability(booking.providerId(), tomorrow());
    assertThat(availability.availableSlots()).contains(oldSlot);
    assertThat(availability.availableSlots()).doesNotContain(newSlot);
}
```

**Expected Result:**
- HTTP 200 OK
- Old slot freed, new slot reserved
- Booking updated with new time

**Traceability:**
- AC: AC-BOOK-003-1
- BR: BR-BOOK-008

---

### TC-BOOK-003-02: Reject reschedule when new slot unavailable

**Type:** Negative | Integration
**Priority:** P0

**Test Steps:**
```java
@Test
void shouldRejectRescheduleWhenNewSlotUnavailable() {
    // Given
    var customer = authenticatedAs(aCustomer());
    var oldSlot = tomorrow().at(10, 0);
    var newSlot = tomorrow().at(14, 0);

    var booking = given(aBooking()
        .withCustomer(customer.id())
        .withTimeSlot(oldSlot)
        .withStatus(CONFIRMED));

    // Another booking already has the new slot
    given(aBooking().withTimeSlot(newSlot).withStatus(CONFIRMED));

    // When
    var response = PUT("/api/v1/bookings/{id}/reschedule", booking.id())
        .withAuth(customer.token())
        .body(new RescheduleRequest(newSlot))
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(409);
    assertThat(response.body().type())
        .isEqualTo("https://api.booking.com/problems/new-time-slot-unavailable");

    // And original booking unchanged
    var originalBooking = GET("/api/v1/bookings/{id}", booking.id())
        .withAuth(customer.token())
        .execute();
    assertThat(originalBooking.body().timeSlot()).isEqualTo(oldSlot);
}
```

**Expected Result:**
- HTTP 409 Conflict
- Original booking unchanged

**Traceability:**
- AC: AC-BOOK-003-2
- BR: BR-BOOK-008

---

### TC-BOOK-003-03: Reschedule atomicity under concurrent access

**Type:** Negative | Concurrency
**Priority:** P0

**Test Steps:**
```java
@Test
void shouldMaintainAtomicityUnderConcurrentReschedule() throws Exception {
    // Given
    var customer1 = authenticatedAs(aCustomer());
    var customer2 = authenticatedAs(aCustomer());
    var targetSlot = tomorrow().at(14, 0);

    var booking1 = given(aBooking()
        .withCustomer(customer1.id())
        .withTimeSlot(tomorrow().at(10, 0))
        .withStatus(CONFIRMED));

    var booking2 = given(aBooking()
        .withCustomer(customer2.id())
        .withTimeSlot(tomorrow().at(11, 0))
        .withStatus(CONFIRMED));

    // When - both try to reschedule to same target slot
    var executor = Executors.newFixedThreadPool(2);
    var latch = new CountDownLatch(1);

    var future1 = executor.submit(() -> {
        latch.await();
        return PUT("/api/v1/bookings/{id}/reschedule", booking1.id())
            .withAuth(customer1.token())
            .body(new RescheduleRequest(targetSlot))
            .execute();
    });

    var future2 = executor.submit(() -> {
        latch.await();
        return PUT("/api/v1/bookings/{id}/reschedule", booking2.id())
            .withAuth(customer2.token())
            .body(new RescheduleRequest(targetSlot))
            .execute();
    });

    latch.countDown();

    var response1 = future1.get();
    var response2 = future2.get();

    // Then - exactly one succeeds
    var statuses = List.of(response1.status(), response2.status());
    assertThat(statuses).containsExactlyInAnyOrder(200, 409);

    // And the failed one's original booking is unchanged
    var failedBookingId = response1.status() == 409 ? booking1.id() : booking2.id();
    var failedOriginalSlot = response1.status() == 409 ?
        tomorrow().at(10, 0) : tomorrow().at(11, 0);

    var unchanged = GET("/api/v1/bookings/{id}", failedBookingId)
        .execute();
    assertThat(unchanged.body().timeSlot()).isEqualTo(failedOriginalSlot);
}
```

**Expected Result:**
- One reschedule succeeds (200)
- One fails (409)
- Failed booking's original slot preserved

**Traceability:**
- BR: BR-BOOK-008
- SEQ: SEQ-005, SEQ-006

---

## US-BOOK-004: Provider Availability

### TC-BOOK-004-01: Set weekly availability

**Type:** Positive | Integration
**Priority:** P1

**Test Steps:**
```java
@Test
void shouldSetWeeklyAvailability() {
    // Given
    var provider = authenticatedAs(aProvider());

    var schedule = WeeklySchedule.builder()
        .monday(timeRange("09:00", "17:00"))
        .tuesday(timeRange("09:00", "17:00"))
        .wednesday(timeRange("09:00", "13:00"))
        .thursday(timeRange("09:00", "17:00"))
        .friday(timeRange("09:00", "17:00"))
        .build();

    // When
    var response = PUT("/api/v1/providers/{id}/calendar", provider.id())
        .withAuth(provider.token())
        .body(new CalendarUpdateRequest(schedule))
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(200);
    assertThat(response.body().weeklySchedule()).isEqualTo(schedule);
}
```

**Expected Result:**
- HTTP 200 OK
- Schedule saved

**Traceability:**
- AC: AC-BOOK-004-1

---

### TC-BOOK-004-02: Warn about existing bookings when changing availability

**Type:** Positive | Integration
**Priority:** P1

**Test Steps:**
```java
@Test
void shouldWarnAboutExistingBookingsWhenChangingAvailability() {
    // Given
    var provider = authenticatedAs(aProvider().availableWeekdays9to17());
    var booking = given(aBooking()
        .withProvider(provider.id())
        .withTimeSlot(nextWednesday().at(14, 0)) // Wednesday afternoon
        .withStatus(CONFIRMED));

    // When - provider removes Wednesday afternoons
    var newSchedule = WeeklySchedule.builder()
        .wednesday(timeRange("09:00", "13:00")) // Only morning now
        .build();

    var response = PUT("/api/v1/providers/{id}/calendar", provider.id())
        .withAuth(provider.token())
        .body(new CalendarUpdateRequest(newSchedule))
        .execute();

    // Then - warning about affected booking
    assertThat(response.status()).isEqualTo(200);
    assertThat(response.body().warnings()).hasSize(1);
    assertThat(response.body().warnings().get(0).type()).isEqualTo("existing-bookings");
    assertThat(response.body().warnings().get(0).affectedBookings())
        .contains(booking.id());

    // And booking is NOT cancelled
    var bookingStillExists = GET("/api/v1/bookings/{id}", booking.id())
        .execute();
    assertThat(bookingStillExists.body().status()).isEqualTo("confirmed");
}
```

**Expected Result:**
- HTTP 200 with warning
- Existing bookings NOT cancelled

**Traceability:**
- AC: AC-BOOK-004-2
- BR: BR-BOOK-010

---

### TC-BOOK-004-03: Reject modification of past availability

**Type:** Negative | Integration
**Priority:** P1

**Test Steps:**
```java
@Test
void shouldRejectModificationOfPastAvailability() {
    // Given
    var provider = authenticatedAs(aProvider());

    // When - try to add exception for yesterday
    var response = POST("/api/v1/providers/{id}/calendar/exceptions", provider.id())
        .withAuth(provider.token())
        .body(new ExceptionRequest(yesterday(), false, "Holiday"))
        .execute();

    // Then
    assertThat(response.status()).isEqualTo(400);
    assertThat(response.body().type())
        .isEqualTo("https://api.booking.com/problems/cannot-modify-past-availability");
}
```

**Expected Result:**
- HTTP 400 Bad Request
- Error type: cannot-modify-past-availability

**Traceability:**
- AC: AC-BOOK-004-3
- BR: BR-BOOK-011

---

## US-BOOK-005: Booking Reminders

### TC-BOOK-005-01: Send reminder for confirmed booking

**Type:** Positive | Integration
**Priority:** P1

**Test Steps:**
```java
@Test
void shouldSendReminderForConfirmedBooking() {
    // Given
    var customer = given(aCustomer().withReminderOptOut(false));
    var booking = given(aBooking()
        .withCustomer(customer.id())
        .withStatus(CONFIRMED)
        .withStartTime(now().plusHours(24))); // Exactly 24h away

    // When - trigger reminder processing
    reminderScheduler.processReminders();

    // Then - reminder event published
    await().atMost(5, SECONDS).untilAsserted(() -> {
        var events = eventCaptor.getCapturedEvents("reminder.due");
        assertThat(events).hasSize(1);
        assertThat(events.get(0).payload().bookingId()).isEqualTo(booking.id());
        assertThat(events.get(0).payload().customerOptedOut()).isFalse();
    });
}
```

**Expected Result:**
- Reminder event published
- Contains booking details

**Traceability:**
- AC: AC-BOOK-005-1
- BR: BR-BOOK-012

---

### TC-BOOK-005-02: Do NOT send reminder for cancelled booking

**Type:** Negative | Integration
**Priority:** P1

**Test Steps:**
```java
@Test
void shouldNotSendReminderForCancelledBooking() {
    // Given
    var customer = given(aCustomer());
    var booking = given(aBooking()
        .withCustomer(customer.id())
        .withStatus(CANCELLED) // Was cancelled
        .withStartTime(now().plusHours(24)));

    // When
    reminderScheduler.processReminders();

    // Then - no reminder event
    await().during(2, SECONDS).atMost(3, SECONDS).untilAsserted(() -> {
        var events = eventCaptor.getCapturedEvents("reminder.due");
        assertThat(events).isEmpty();
    });
}
```

**Expected Result:**
- No reminder event published

**Traceability:**
- AC: AC-BOOK-005-2
- BR: BR-BOOK-012

---

### TC-BOOK-005-03: Respect customer opt-out preference

**Type:** Negative | Integration
**Priority:** P1

**Test Steps:**
```java
@Test
void shouldRespectCustomerReminderOptOut() {
    // Given
    var customer = given(aCustomer().withReminderOptOut(true)); // Opted out
    var booking = given(aBooking()
        .withCustomer(customer.id())
        .withStatus(CONFIRMED)
        .withStartTime(now().plusHours(24)));

    // When
    reminderScheduler.processReminders();

    // Then - no reminder (customer opted out)
    await().during(2, SECONDS).atMost(3, SECONDS).untilAsserted(() -> {
        var events = eventCaptor.getCapturedEvents("reminder.due");
        assertThat(events).isEmpty();
    });
}
```

**Expected Result:**
- No reminder for opted-out customer

**Traceability:**
- AC: AC-BOOK-005-3
- BR: BR-BOOK-013

---

## Error Codes Coverage

| Error Code | Test Case | Type |
|------------|-----------|------|
| TIME_SLOT_UNAVAILABLE | TC-BOOK-001-02 | 409 |
| PAST_TIME_SLOT | TC-BOOK-001-03 | 400 |
| PROVIDER_NOT_AVAILABLE | TC-BOOK-001-04 | 400 |
| AUTHENTICATION_REQUIRED | TC-BOOK-001-05 | 401 |
| NOT_AUTHORIZED_TO_CANCEL | TC-BOOK-002-04 | 403 |
| INVALID_BOOKING_STATUS | TC-BOOK-002-05 | 400 |
| CANNOT_CANCEL_PAST_BOOKING | TC-BOOK-002-06 | 400 |
| NEW_TIME_SLOT_UNAVAILABLE | TC-BOOK-003-02 | 409 |
| CANNOT_MODIFY_PAST_AVAILABILITY | TC-BOOK-004-03 | 400 |

**Coverage:** 9/9 error codes (100%)

---

## Test Summary

| Category | Count | Coverage |
|----------|-------|----------|
| Positive tests | 8 | All happy paths |
| Negative tests | 9 | All error codes |
| Concurrency tests | 3 | Race conditions |
| Security tests | 2 | Auth checks |
| **Total** | **22** | |

**SI Decisions Applied:**
- Testcontainers for real DB (TST-1)
- Builder pattern for test data (TST-2)
- Full integration tests (TST-3)
- Every error code tested (TST-4)
- Explicit concurrency tests (TST-5)

---

## Traceability Matrix

| Test Case | AC | BR | SEQ |
|-----------|----|----|-----|
| TC-BOOK-001-01 | AC-BOOK-001-1 | BR-BOOK-001 | SEQ-001 |
| TC-BOOK-001-02 | AC-BOOK-001-2 | BR-BOOK-001 | SEQ-002 |
| TC-BOOK-001-03 | AC-BOOK-001-3 | BR-BOOK-002 | - |
| TC-BOOK-001-04 | AC-BOOK-001-4 | BR-BOOK-001 | - |
| TC-BOOK-001-05 | - | BR-BOOK-003 | - |
| TC-BOOK-002-01 | AC-BOOK-002-1 | BR-BOOK-004, BR-BOOK-005 | SEQ-003 |
| TC-BOOK-002-02 | AC-BOOK-002-2 | BR-BOOK-004 | SEQ-003 |
| TC-BOOK-002-03 | AC-BOOK-002-3 | BR-BOOK-007 | SEQ-004 |
| TC-BOOK-002-04 | - | BR-BOOK-004 | - |
| TC-BOOK-002-05 | AC-BOOK-002-4 | BR-BOOK-005 | - |
| TC-BOOK-002-06 | AC-BOOK-002-5 | BR-BOOK-006 | - |
| TC-BOOK-003-01 | AC-BOOK-003-1 | BR-BOOK-008 | SEQ-005 |
| TC-BOOK-003-02 | AC-BOOK-003-2 | BR-BOOK-008 | SEQ-006 |
| TC-BOOK-003-03 | - | BR-BOOK-008 | SEQ-005, SEQ-006 |
| TC-BOOK-004-01 | AC-BOOK-004-1 | - | SEQ-007 |
| TC-BOOK-004-02 | AC-BOOK-004-2 | BR-BOOK-010 | SEQ-007 |
| TC-BOOK-004-03 | AC-BOOK-004-3 | BR-BOOK-011 | - |
| TC-BOOK-005-01 | AC-BOOK-005-1 | BR-BOOK-012 | SEQ-008 |
| TC-BOOK-005-02 | AC-BOOK-005-2 | BR-BOOK-012 | SEQ-009 |
| TC-BOOK-005-03 | AC-BOOK-005-3 | BR-BOOK-013 | - |

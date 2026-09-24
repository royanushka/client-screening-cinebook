# Booking Workflow and Technical Challenges

## 1. Booking Lifecycle

```text
PENDING
   |
   v
PAYMENT_IN_PROGRESS
   |              |
   |              +--> PAYMENT_FAILED
   |
   +--> EXPIRED
   |
   +--> CONFIRMED
             |
             +--> CANCELLED
```

## 2. Seat Reservation Workflow

1. Validate the show ID.
2. Validate that the selected seats belong to the show screen.
3. Check that the requested seats are eligible for booking.
4. Attempt to acquire a temporary lock for each seat.
5. If any lock fails, release locks already acquired for the request and return a 
   seat-unavailable response.
6. Create a pending booking.
7. Return booking ID, expiry time, and payment details.
8. Confirm or release the reservation based on payment and expiry outcomes.

## 3. Technical Challenge: Double Booking

### Problem

Two users can submit booking requests for the same seat at nearly the same time.

### Proposed solution

- Use atomic temporary lock acquisition.
- Identify each lock by show ID and seat ID.
- Associate the lock with a booking or reservation ID.
- Use TTL to avoid permanent locks.
- Protect final confirmation with database-level consistency controls.
- Recheck booking and seat state before confirmation.

### Failure scenarios

- Redis becomes unavailable.
- Lock expires during payment.
- Database confirmation fails.
- User retries the hold request.
- Two application instances process the same seat.

The system should fail safely and must not bypass concurrency controls simply because the lock service is unavailable.

## 4. Technical Challenge: Payment Idempotency

### Problem

A payment callback can be delivered more than once, or a client can retry the same request after a timeout.

### Proposed solution

- Accept or generate an idempotency key.
- Store the key and result durably.
- Enforce uniqueness on the provider reference.
- Return the previously stored result for duplicate callbacks.
- Prevent repeated booking confirmation events.

## 5. Technical Challenge: Payment Success but Booking Failure

A payment may succeed while the booking confirmation transaction fails.

### Handling approach

- Keep payment and booking states separate.
- Store the payment provider reference.
- Publish confirmation only after durable booking confirmation.
- Introduce a reconciliation process for uncertain outcomes.
- Define a refund or manual-review policy for payment-success/booking-failure cases.

## 6. Technical Challenge: Expired Reservations

A pending booking should not hold seats indefinitely.

### Proposed solution

- Store an expiry timestamp.
- Use Redis TTL for temporary coordination.
- Run an expiry process or scheduled cleanup.
- Update pending bookings to `EXPIRED`.
- Release durable seat reservations where applicable.
- Ensure expiry cannot invalidate a booking that has already been confirmed.

## 7. Technical Challenge: Asynchronous Notifications

Notifications should not unnecessarily delay the booking confirmation response.

### Proposed solution

- Persist the confirmed booking first.
- Publish a `BookingConfirmed` event.
- Consume the event in the Notification Service.
- Retry transient failures.
- Use an event ID or booking ID to prevent duplicate notifications.

For stronger delivery guarantees, consider the transactional outbox pattern instead of publishing directly from the database transaction.

## 8. Technical Contributions to Highlight

- Designed the booking and seat reservation workflow.
- Defined service boundaries and API contracts.
- Designed database entities and consistency constraints.
- Addressed concurrent reservation scenarios.
- Designed payment idempotency and failure handling.
- Designed Kafka event processing and notification flow.
- Prepared automated test scenarios for concurrency and failure recovery.

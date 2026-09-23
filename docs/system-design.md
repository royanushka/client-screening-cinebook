# CineBook System Design

## 1. Objective

Design a reliable movie ticket booking platform that supports movie discovery, theatre and show browsing, seat reservation, 
payment processing, and booking confirmation.

The most important system invariant is:
* For a given show, a physical seat must not be confirmed for more than one booking.

### Components

| Component | Responsibility |
|---|---|
| API Gateway | Routing, authentication, rate limiting |
| Movie Service | Movie details, language, genre, duration |
| Theatre Service | Theatres, screens, seats, show schedules |
| Booking Service | Seat availability, holds, booking lifecycle |
| Payment Service | Payment initiation, provider callbacks, idempotency |
| Notification Service | Email/SMS/push notifications |
| Redis | Temporary reservation locks and selected cache data |
| Relational Database | Durable transactional data |
| Kafka | Asynchronous domain events |

## 3. Request Flow

1. Client requests available movies and shows.
2. Client requests the seat layout for a selected show.
3. Booking Service validates the requested seats.
4. Booking Service attempts to acquire temporary locks.
5. A pending booking is created in the database.
6. User initiates payment.
7. Payment Service processes the payment and stores the result.
8. Booking Service confirms the booking after verifying the payment result.
9. A `BookingConfirmed` event is published.
10. Notification Service consumes the event and sends confirmation.

## 4. Seat Locking Strategy

A temporary lock can be represented using a Redis key such as:

```text
seat-lock:show-101:seat-A10
```

The value should identify the booking or reservation owner.

A Redis operation equivalent to the following can be used:

```text
SET seat-lock:show-101:seat-A10 booking-5001 NX EX 300
```

- `NX`: Create the key only if it does not already exist.
- `EX 300`: Expire the key after 300 seconds.

Redis should not be the only consistency mechanism. 
The final booking operation must be protected by database transactions and constraints or an equivalent durable concurrency-control 
strategy.

## 5. Database Consistency

The database should enforce:

- One show-seat mapping per show and physical seat.
- Valid booking-to-show relationships.
- Valid booking-seat relationships.
- Idempotent payment provider references.
- Valid booking state transitions.

Potential implementation strategies include:

- Conditional updates on a seat state row.
- Pessimistic locking.
- Optimistic locking using a version field.
- Unique constraints for confirmed or active reservations.

The selected approach should be documented with its trade-offs.

## 6. Scalability

### Application layer

- Keep services stateless where possible.
- Run multiple instances behind a load balancer.
- Use horizontal scaling for read-heavy services.
- Apply rate limiting at the gateway.

### Database layer

- Add indexes based on query patterns.
- Avoid unnecessary joins and repeated queries.
- Use connection pooling.
- Monitor slow queries and execution plans.

### Caching layer

Cache data that changes infrequently, such as:

- Movie metadata
- Theatre information
- City-based movie listings
- Show metadata

Seat availability requires stronger consistency controls and should not be treated as an ordinary long-lived cache.

## 7. Reliability and Observability

Monitor:

- Booking API latency
- Seat-lock acquisition failures
- Payment failures
- Expired pending bookings
- Database query latency
- Kafka consumer lag
- Notification failures

Using structured logging and correlation IDs to trace a booking request across services.

## 8. Architectural Trade-offs

| Decision              | Benefit                                    | Trade-off                                       |
|-----------------------|--------------------------------------------|-------------------------------------------------|
| Redis seat lock       | Fast temporary coordination                | Lock expiry and failure handling                |
| Relational database   | Strong transactional consistency           | Contention under high load                      |
| Kafka events          | Loose coupling and asynchronous processing | Eventual consistency and operational complexity |
| Microservices         | Independent deployment and scaling         | Network calls and distributed debugging         |
| Modular monolith      | Simpler development and testing            | Less independent deployment flexibility         |

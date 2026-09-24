
# CineBook – System Design

## 1. Objective

Design a reliable movie ticket booking platform that supports movie discovery, theatre and show browsing, seat reservation, payment processing, and booking confirmation.

The primary system invariant is:

> For a given show, a physical seat must not be confirmed for more than one booking.

The system should maintain consistency during concurrent seat reservations, payment retries, booking expiry, and service failures.

## 2. Technology Stack

| Technology                  | Purpose                            |
|-----------------------------|------------------------------------|
| Java 8+                     | Backend development                |
| Spring Boot                 | Application framework              |
| Spring MVC                  | REST API development               |
| Spring Data JPA / Hibernate | Persistence and ORM                |
| MySQL / SQL Server          | Relational data storage            |
| Redis                       | Temporary seat locking and caching |
| Apache Kafka                | Asynchronous event processing      |
| Docker                      | Containerization                   |
| Kubernetes                  | Container orchestration            |
| JUnit / Mockito             | Unit and integration testing       |
| Maven                       | Build management                   |

## 3. System Components

| Component            | Responsibility                                            |
|----------------------|-----------------------------------------------------------|
| API Gateway          | Request routing, authentication, and rate limiting        |
| Movie Service        | Movie details, language, genre, and duration              |
| Theatre Service      | Theatres, screens, seats, and show schedules              |
| Booking Service      | Seat availability, temporary holds, and booking lifecycle |
| Payment Service      | Payment initiation, provider callbacks, and idempotency   |
| Notification Service | Email, SMS, or push notifications                         |
| Redis                | Temporary reservation locks and selected cache data       |
| Relational Database  | Durable transactional data                                |
| Apache Kafka         | Asynchronous domain event processing                      |

The components may be implemented as independently deployable services or organized as modules within a modular monolith, depending on deployment and operational requirements.

## 4. Booking Request Flow

1. The client requests available movies and shows.
2. The client requests the seat layout for a selected show.
3. The Booking Service validates the requested seats.
4. The Booking Service attempts to acquire temporary seat locks.
5. A pending booking is created in the database.
6. The user initiates payment.
7. The Payment Service processes the payment and stores the result.
8. The Booking Service verifies the payment result and confirms the booking.
9. A `BookingConfirmed` event is published.
10. The Notification Service consumes the event and sends the booking confirmation.

The booking workflow and state transitions are documented in [Booking Workflow](booking-workflow.md).

## 5. Temporary Seat-Locking Strategy

A temporary lock can be represented using a Redis key such as:

```text
seat-lock:show-101:seat-A10
```

The value should identify the booking or reservation owner.

A Redis operation equivalent to the following can be used:

```text
SET seat-lock:show-101:seat-A10 booking-5001 NX EX 300
```

- `NX`: Creates the key only if it does not already exist.
- `EX 300`: Expires the key after 300 seconds.

### Consistency Considerations

Redis should not be the only consistency mechanism. The final booking operation must be protected by database transactions and constraints, or by an equivalent durable concurrency-control strategy.

The system must also handle:

- Lock expiry
- Redis failures
- Client retries
- Payment delays
- Booking confirmation failures
- Recovery of abandoned bookings

## 6. Database Consistency

The database should enforce the following rules:

- One show-seat mapping for each show and physical seat
- Valid booking-to-show relationships
- Valid booking-seat relationships
- Idempotent payment provider references
- Valid booking state transitions
- Prevention of duplicate confirmed bookings

### Potential Concurrency-Control Strategies

Possible implementation approaches include:

- Conditional updates on a seat-state row
- Pessimistic locking
- Optimistic locking using a version field
- Unique constraints for confirmed or active reservations

The selected approach should be documented along with its trade-offs, including database contention, performance, and implementation complexity.

Detailed entities and relationships are described in [Database Design](database-design.md).

## 7. Scalability

### 7.1 Application Layer

- Keep services stateless wherever possible.
- Run multiple instances behind a load balancer.
- Horizontally scale read-heavy services.
- Apply rate limiting at the API Gateway.
- Use asynchronous processing for non-critical post-booking operations.

### 7.2 Database Layer

- Add indexes based on query patterns.
- Avoid unnecessary joins and repeated queries.
- Use connection pooling.
- Monitor slow queries and execution plans.
- Consider read replicas for suitable read-heavy workloads.

### 7.3 Caching Layer

Cache data that changes relatively infrequently, such as:

- Movie metadata
- Theatre information
- City-based movie listings
- Show metadata

Seat availability requires stronger consistency controls and should not be treated as an ordinary long-lived cache.

## 8. Reliability and Observability

The system should monitor the following metrics:

- Booking API latency
- Seat-lock acquisition failures
- Payment failures
- Expired pending bookings
- Database query latency
- Kafka consumer lag
- Notification failures

Structured logging and correlation IDs should be used to trace a booking request across services.

Additional reliability mechanisms may include:

- Retry policies for transient failures
- Dead-letter topics for failed Kafka messages
- Payment reconciliation for uncertain payment outcomes
- Alerting for increased booking or payment failures
- Health checks and readiness probes

## 9. Architectural Trade-offs

| Decision            | Benefit                                    | Trade-off                                       |
|---------------------|--------------------------------------------|-------------------------------------------------|
| Redis seat lock     | Fast temporary coordination                | Lock expiry and failure handling                |
| Relational database | Strong transactional consistency           | Possible contention under high load             |
| Kafka events        | Loose coupling and asynchronous processing | Eventual consistency and operational complexity |
| Microservices       | Independent deployment and scaling         | Network calls and distributed debugging         |
| Modular monolith    | Simpler development and testing            | Less independent deployment flexibility         |

The final architecture should be selected based on expected traffic, team size, operational maturity, deployment requirements, and consistency needs.
# CineBook – Movie Ticket Booking System

## 1. Project Overview

CineBook is a scalable movie ticket booking platform designed using Java and Spring Boot. 
The system enables users to search for movies, view available shows, select seats, temporarily reserve seats, 
process payments, and receive booking confirmations.

The primary focus of this project is to design a reliable booking system that handles concurrent seat reservations, 
prevents double booking, and maintains consistency across booking and payment workflows.

## 2. Key Features

* Movie and theatre discovery
* Show scheduling and seat availability
* Temporary seat reservation with expiry
* Booking lifecycle management
* Payment processing and idempotency
* Kafka-based asynchronous notifications
* REST APIs using Spring Boot
* Database consistency and concurrency control

## 3. Technology Stack

| Technology                  | Purpose                            |
| --------------------------- | ---------------------------------- |
| Java 8+                     | Backend development                |
| Spring Boot                 | Application framework              |
| Spring MVC                  | REST APIs                          |
| Spring Data JPA / Hibernate | Persistence                        |
| MySQL / SQL Server          | Relational database                |
| Redis                       | Temporary seat locking and caching |
| Apache Kafka                | Asynchronous event processing      |
| Docker                      | Containerization                   |
| Kubernetes                  | Service orchestration              |
| JUnit / Mockito             | Testing                            |
| Maven                       | Build management                   |

## 4. System Architecture

The application consists of the following logical components:

* Movie Service – Movie metadata and discovery
* Theatre Service – Theatre, screen, and show management
* Booking Service – Seat availability, reservation, and booking lifecycle
* Payment Service – Payment processing and status management
* Notification Service – Booking confirmation notifications
* API Gateway – Request routing and security

Refer to [System Design](docs/system-design.md) for detailed architecture and design decisions.

## 5. Key Technical Challenges

### 5.1 Concurrent Seat Booking

Multiple users may attempt to reserve the same seat for the same show simultaneously.

**Solution:**

* Use temporary seat locks with an expiry time.
* Apply database-level consistency controls.
* Use transactional updates to confirm reservations.
* Ensure only one valid booking can be confirmed for a show-seat pair.

### 5.2 Payment Idempotency

Payment providers may send duplicate callbacks or users may retry payment requests.

**Solution:**

* Use idempotency keys.
* Store payment provider references.
* Prevent duplicate payment processing.
* Handle payment and booking state transitions explicitly.

### 5.3 Booking and Payment Failures

The system must handle payment failures, expired seat holds, and booking confirmation failures.

**Solution:**

* Maintain explicit booking states.
* Implement expiry and recovery handling.
* Use asynchronous events for post-confirmation notifications.
* Provide reconciliation for uncertain payment outcomes.

## 6. Documentation

* [System Design](docs/system-design.md)
* [Database Design](docs/database-design.md)
* [Booking Workflow](docs/booking-workflow.md)

## 7. Testing

The project includes unit and integration tests covering:

* Seat availability validation
* Concurrent seat reservation
* Payment idempotency
* Booking state transitions
* Failure handling

## 8. Future Enhancements

* Dynamic pricing
* Recommendation engine
* Distributed tracing
* Advanced monitoring and alerting
* Multi-region deployment

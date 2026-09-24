
# CineBook – Movie Ticket Booking System

## 1. Project Overview

CineBook is a movie ticket booking platform designed using Java and Spring Boot. The system enables
users to discover movies and theatres, browse available shows, select seats, 
temporarily reserve seats, process payments, and receive booking confirmations.

The project focuses on designing a reliable booking system that handles concurrent seat 
reservations, prevents double booking, and maintains consistency across booking and payment
workflows.

## 2. Key Features

- Movie and theatre discovery
- Show scheduling and seat availability
- Temporary seat reservation with expiry
- Booking lifecycle management
- Payment processing and idempotency
- Kafka-based asynchronous notifications
- REST APIs using Spring Boot
- Database consistency and concurrency control

## 3. Logical Components

The system is organized into the following logical components:

| Component            | Responsibility                                        |
|----------------------|-------------------------------------------------------|
| Movie Service        | Movie metadata and discovery                          |
| Theatre Service      | Theatre, screen, and show management                  |
| Booking Service      | Seat availability, reservation, and booking lifecycle |
| Payment Service      | Payment processing and payment status management      |
| Notification Service | Booking confirmation notifications                    |
| API Gateway          | Request routing, authentication, and security         |

The detailed architecture, technology choices, and design decisions are documented in 
system-design.md.

## 4. Key Design Considerations

The design addresses the following challenges:

- Concurrent seat reservations and prevention of double booking
- Temporary seat-lock expiry and recovery
- Payment idempotency and duplicate callbacks
- Booking and payment state management
- Database consistency and transactional processing
- Asynchronous event processing and notification delivery

The implementation strategies and trade-offs are described in the [System Design](system-design.md) and [Booking Workflow](booking-workflow.md) documents.

## 5. Documentation

- [System Design](system-design.md) – Architecture, technology stack, request flow, scalability, and technical trade-offs
- [Database Design](database-design.md) – Database entities, relationships, and consistency considerations
- [Booking Workflow](booking-workflow.md) – Booking lifecycle, seat reservation, payment processing, and state transitions
- [System Architecture Diagram](diagrams/system-architecture.md)


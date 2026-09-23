# Database Design

## 1. Main Entities

- `MOVIE`
- `THEATRE`
- `SCREEN`
- `SEAT`
- `SHOW`
- `SHOW_SEAT`
- `USER`
- `BOOKING`
- `BOOKING_SEAT`
- `PAYMENT`

## 2. Important Modeling Decision

A seat belongs to a physical screen, but its availability is specific to a show.

For example:

- Seat A10 belongs to Screen 1.
- Show 101 is scheduled on Screen 1.
- Show 102 is also scheduled on Screen 1.
- Seat A10 must have separate availability records for Show 101 and Show 102.

Therefore, the `SHOW_SEAT` entity maps a physical seat to a particular show.

## 3.Tables

### MOVIE

| Column           | Type    | Notes          |
|------------------|---------|----------------|
| movie_id         | BIGINT  | Primary key    |
| title            | VARCHAR | Movie title    |
| language         | VARCHAR | Movie language |
| genre            | VARCHAR | Movie genre    |
| duration_minutes | INT     | Duration       |
| release_date     | DATE    | Release date   |

### THEATRE

| Column     | Type    | Notes        |
|------------|---------|--------------|
| theatre_id | BIGINT  | Primary key  |
| name       | VARCHAR | Theatre name |
| city       | VARCHAR | City         |
| address    | VARCHAR | Address      |

### SCREEN

| Column      | Type    | Notes             |
|-------------|---------|-------------------|
| screen_id   | BIGINT  | Primary key       |
| theatre_id  | BIGINT  | Foreign key       |
| screen_name | VARCHAR | Screen identifier |

### SEAT

| Column      | Type    | Notes             |
|-------------|---------|-------------------|
| seat_id     | BIGINT  | Primary key       |
| screen_id   | BIGINT  | Foreign key       |
| seat_number | VARCHAR | Example: A10      |
| seat_type   | VARCHAR | REGULAR / PREMIUM |

### SHOW

| Column     | Type     | Notes       |
|------------|----------|-------------|
| show_id    | BIGINT   | Primary key |
| movie_id   | BIGINT   | Foreign key |
| screen_id  | BIGINT   | Foreign key |
| start_time | DATETIME | Show start  |
| end_time   | DATETIME | Show end    |

### SHOW_SEAT

| Column       | Type    | Notes                               |
|--------------|---------|-------------------------------------|
| show_seat_id | BIGINT  | Primary key                         |
| show_id      | BIGINT  | Foreign key                         |
| seat_id      | BIGINT  | Foreign key                         |
| price        | DECIMAL | Ticket price                        |
| status       | VARCHAR | AVAILABLE / HELD / BOOKED           |
| version      | BIGINT  | Optimistic locking version, if used |

Recommended uniqueness:

```sql
ALTER TABLE SHOW_SEAT
ADD CONSTRAINT UK_SHOW_SEAT
UNIQUE (SHOW_ID, SEAT_ID);
```

### BOOKING

| Column       | Type     | Notes                                     |
|--------------|----------|-------------------------------------------|
| booking_id   | BIGINT   | Primary key                               |
| user_id      | BIGINT   | Foreign key                               |
| show_id      | BIGINT   | Foreign key                               |
| status       | VARCHAR  | PENDING / CONFIRMED / EXPIRED / CANCELLED |
| total_amount | DECIMAL  | Total booking amount                      |
| expires_at   | DATETIME | Reservation expiry                        |
| created_at   | DATETIME | Creation timestamp                        |

### BOOKING_SEAT

| Column          | Type   | Notes       |
|-----------------|--------|-------------|
| booking_seat_id | BIGINT | Primary key |
| booking_id      | BIGINT | Foreign key |
| show_seat_id    | BIGINT | Foreign key |

### PAYMENT

| Column             | Type     | Notes                        |
|--------------------|----------|------------------------------|
| payment_id         | BIGINT   | Primary key                  |
| booking_id         | BIGINT   | Foreign key                  |
| provider_reference | VARCHAR  | Unique provider reference    |
| idempotency_key    | VARCHAR  | Unique retry key             |
| status             | VARCHAR  | INITIATED / SUCCESS / FAILED |
| amount             | DECIMAL  | Payment amount               |
| created_at         | DATETIME | Creation timestamp           |

## 4. Indexing Suggestions

Indexes should be selected after reviewing real query patterns.

```sql
CREATE INDEX IDX_SHOW_MOVIE_DATE
ON SHOW (MOVIE_ID, START_TIME);

CREATE INDEX IDX_SHOW_SCREEN_DATE
ON SHOW (SCREEN_ID, START_TIME);

CREATE INDEX IDX_BOOKING_USER
ON BOOKING (USER_ID, CREATED_AT);
```


Two concurrent requests can both read `AVAILABLE` before either update is committed.

Using an appropriate transaction and concurrency-control mechanism to ensure that only 
one request can successfully confirm the seat.

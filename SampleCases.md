 # S11 Sample Cases – Room Booking

## S1 – Create Booking (`POST /v1/bookings`)

### Inputs & Rules

- Inputs:
  - `roomId` (string, existing room)
  - `startTime` (ISO 8601)
  - `endTime` (ISO 8601)
  - `guestEmail` (string, email)
- Business rules:
  - `startTime` < `endTime`
  - `startTime` ≥ now
  - `roomId` must exist
  - booking must not overlap another booking for the same room

### Equivalence Classes

- `roomId`
  - `V_room_existing`: existing room ID
  - `I_room_unknown`: non-existing room ID
- `timeRange`
  - `V_time_future`: start ≥ now, end > start
  - `I_time_past`: start < now
  - `I_time_inverted`: start ≥ end
- `conflict`
  - `V_no_conflict`: no overlapping booking
  - `I_conflict`: overlaps existing booking
- `email`
  - `V_email`: well-formed email
  - `I_email`: empty / invalid

### Boundaries

- `startTime`: now-1min, now, now+1min
- `endTime`: start+0min, start+1min
- booking overlapping exactly at boundary: end of booking A == start of booking B

### Test Ideas

- TI1 [S1][Happy][V_room_existing][V_time_future][V_no_conflict][V_email]
- TI2 [S1][Invalid][I_time_past]
- TI3 [S1][Invalid][I_time_inverted]
- TI4 [S1][Invalid][I_room_unknown]
- TI5 [S1][Conflict][I_conflict]
- TI6 [S1][Idempotent] – same payload + same Idempotency-Key

### Test Cases

#### TC1 [S1][Happy]

- Preconditions:
  - Room `ROOM-101` exists.
  - No bookings for `ROOM-101` in the chosen time window.
- Input:
  - `POST /v1/bookings`
  - Body:
    ```json
    {
      "roomId": "ROOM-101",
      "startTime": "2025-11-25T10:00:00Z",
      "endTime": "2025-11-25T11:00:00Z",
      "guestEmail": "user@example.com"
    }
    ```
  - Headers: `Idempotency-Key: "idem-001"`
- Expected:
  - Status: `201`
  - Body: contains `bookingId` and echoes `roomId`.
  - No `error` field.
  - DB contains booking with same values.

#### TC2 [S1][I_time_past]

- Preconditions:
  - System “now” = 2025-11-25T10:00:00Z.
- Input:
  - `startTime = 2025-11-25T09:00:00Z`
  - `endTime = 2025-11-25T10:00:00Z`
- Expected:
  - Status: `400`
  - Body:
    - `code = "400_INVALID_DATE_RANGE"`
    - `hint` explains that start must be in future.
    - `correlationId` present.

#### TC3 [S1][I_time_inverted]

- Input:
  - `startTime = 2025-11-25T11:00:00Z`
  - `endTime   = 2025-11-25T10:00:00Z`
- Expected:
  - Status: `400`
  - `code = "400_INVALID_DATE_RANGE"`

#### TC4 [S1][I_room_unknown]

- Input:
  - `roomId = "ROOM-UNKNOWN"`
- Expected:
  - Status: `404`
  - `code = "404_ROOM_NOT_FOUND"`

#### TC5 [S1][I_conflict]

- Preconditions:
  - Existing booking for `ROOM-101` from 10:00–11:00.
- Input:
  - Request 2 for `ROOM-101` from 10:30–11:00.
- Expected:
  - Status: `409`
  - `code = "409_BOOKING_CONFLICT"`

#### TC6 [S1][Idempotent]

- Preconditions:
  - Same as TC1; no booking yet.
- Steps:
  1. First call with `Idempotency-Key: "idem-xyz"`.
     - Expect 201 + `bookingId = BKG-123`.
  2. Second call with **same body and same key**.
     - Expect 200 + `bookingId = BKG-123` (same as first).

---

## S2 – Cancel Booking (`POST /v1/bookings/{id}/cancel`)

### Inputs & Rules

- Inputs:
  - Path param: `id` (booking ID).
  - Authenticated user.
- Rules:
  - Booking must exist.
  - Only owner or admin can cancel.
  - Cannot cancel after booking has started.

### Equivalence Classes

- `id`
  - `V_id_existing`: existing booking ID.
  - `I_id_unknown`: non-existing ID.
- `auth`
  - `V_owner`: owner of booking.
  - `V_admin`: admin role.
  - `I_otherUser`: authenticated user but not owner/admin.
  - `I_unauth`: unauthenticated.
- `time`
  - `V_future`: booking start in future.
  - `I_started`: start in the past.

### Test Cases

#### TC7 [S2][Happy][V_id_existing][V_owner][V_future]

- Preconditions:
  - Booking `BKG-123` exists in future.
  - Current user is owner.
- Input:
  - `POST /v1/bookings/BKG-123/cancel`
- Expected:
  - Status: `200`
  - Body: `{ "status": "cancelled" }` (or equivalent).
  - DB: booking status updated to `cancelled`.

#### TC8 [S2][NotFound][I_id_unknown]

- Input:
  - `POST /v1/bookings/UNKNOWN/cancel`
- Expected:
  - Status: `404`
  - `code = "404_BOOKING_NOT_FOUND"`

#### TC9 [S2][Forbidden][I_otherUser]

- Preconditions:
  - Booking `BKG-123` belongs to user A.
  - Logged-in user is user B.
- Expected:
  - Status: `403`
  - `code = "403_FORBIDDEN"`

#### TC10 [S2][Conflict][I_started]

- Preconditions:
  - Booking `BKG-123` started 1 hour ago.
- Expected:
  - Status: `409`
  - `code = "409_CANNOT_CANCEL_STARTED"`

 # S11 – Sample Cases (ECP/BVA)

## Story References
- **S1 – Create Booking:** https://github.com/md-anuar-hosen/S11/issues/1
- **S2 – Cancel Booking:** https://github.com/md-anuar-hosen/S11/issues/2

---

## S1 – Create Booking

### Inputs
- `roomId` (string)
- `startTime` (ISO date string)
- `endTime` (ISO date string)
- `guestEmail` (string)
- `idempotencyKey` (optional, string)

### Outputs
- `201` + `{ bookingId }` on success
- Problem+json error responses with:
  - `400_INVALID_INPUT`
  - `400_INVALID_DATE_RANGE`
  - `400_INVALID_EMAIL`
  - `409_BOOKING_CONFLICT`

### Business Rules
- `startTime` must be in the future (>= now)
- `endTime` must be after `startTime`
- `guestEmail` must contain `"@"`
- For the same `idempotencyKey` and same payload:
  - first call → `201` + new `bookingId`
  - second call → `200` + same `bookingId`

---

### Equivalence Classes (S1)

- **roomId**
  - `V_room`: non-empty string
  - `I_room_missing`: empty / null

- **timeRange**
  - `V_time_future`: `startTime` >= now and `endTime` > `startTime`
  - `I_time_past`: `startTime` < now
  - `I_time_inverted`: `endTime` <= `startTime`
  - `I_time_invalid_format`: invalid date format

- **guestEmail**
  - `V_email`: contains `@`
  - `I_email`: empty or missing `@`

- **idempotencyKey**
  - `V_idem_new`: new key → create (201)
  - `V_idem_repeat`: same key + same payload → replay (200)

### Boundaries (S1)

- `startTime`: now−1 min, now, now+1 min
- `endTime`: exactly equal to `startTime`, `startTime`+1 ms
- `guestEmail`: `""`, `"test"`, `"user@example.com"`

---

### Test Cases S1

**TC1 [S1][Happy][V_room][V_time_future][V_email]**  
- Input: valid roomId, `startTime = now+1h`, `endTime = startTime+1h`, valid email  
- Expected:  
  - `status = 201`  
  - body contains `bookingId`  

**TC2 [S1][I_time_past][Boundary]**  
- Input: `startTime = now−1 min`, `endTime = now+1h`  
- Expected:  
  - `status = 400`  
  - `code = "400_INVALID_DATE_RANGE"`  

**TC3 [S1][I_time_inverted]**  
- Input: `startTime = 11:00`, `endTime = 10:00`  
- Expected:  
  - `status = 400`  
  - `code = "400_INVALID_DATE_RANGE"`  

**TC4 [S1][I_email]**  
- Input: `guestEmail = "abc"` (no `@`)  
- Expected:  
  - `status = 400`  
  - `code = "400_INVALID_EMAIL"`  

**TC5 [S1][Conflict]**  
- Precondition: existing booking for `ROOM-101` from 10:00–11:00  
- Input: new booking for `ROOM-101` from 10:30–11:00  
- Expected:  
  - `status = 409`  
  - `code = "409_BOOKING_CONFLICT"`  

**TC6 [S1][Idempotent][V_idem_repeat]**  
- Step 1: POST with `Idempotency-Key = "idem-1"` → expect `201`, `bookingId = X`  
- Step 2: same payload + same key → expect `200`, `bookingId = X`  

---

## S2 – Cancel

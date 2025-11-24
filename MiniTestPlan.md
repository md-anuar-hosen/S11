 # S11 Mini Test Plan – Room Booking

## 1. Stories Covered

- **S1 – Create Booking**
  - User books a room via `POST /v1/bookings`.
  - Acceptance criteria:
    - AC1: Booking succeeds when room exists, times are valid, and no conflicts.
    - AC2: Overlapping requests fail with a clear conflict error.
    - AC3: Invalid time ranges are rejected with `400_INVALID_DATE_RANGE`.
    - AC4: Idempotency with `Idempotency-Key` works (201 then 200).

- **S2 – Cancel Booking**
  - User cancels an existing booking via `POST /v1/bookings/{id}/cancel`.
  - Acceptance criteria:
    - AC1: Owner can cancel a future booking.
    - AC2: Cancelling non-existing booking returns `404_BOOKING_NOT_FOUND`.
    - AC3: Cancelling after start time returns `409_CANNOT_CANCEL_STARTED`.
    - AC4: Unauthorized users cannot cancel other peoples’ bookings.

## 2. Scope

- **In scope**
  - Functional happy paths for S1 and S2.
  - Validation and main negative paths (invalid dates, overlap, not found, unauthorized).
  - Idempotency behaviour for S1.
  - Error contract and codes consistency with `ERROR_RULES.md`.

- **Out of scope (deferred)**
  - Load and performance testing.
  - Security testing beyond basic auth/authorization checks.
  - Property-based testing and fuzzing.
  - Cross-service integration with e.g. notifications.

## 3. Test Levels & Targets

### S1 – Create Booking

- **Unit**
  - `BookingValidator` (date and field validation).
  - Conflict detection logic (overlap checks).
- **Integration**
  - `POST /v1/bookings` hitting the API layer + DB:
    - 201 on valid request.
    - 400 on invalid data.
    - 409 on conflicts.
    - Proper `problem+json` shape.
- **System**
  - End-to-end flow: login → create booking → view booking in list.
- **UAT**
  - Product owner tests AC1–AC4 on staging using an API client or UI.

### S2 – Cancel Booking

- **Unit**
  - `BookingService.cancelBooking` state transitions.
- **Integration**
  - `POST /v1/bookings/{id}/cancel` with:
    - 200 on success.
    - 404 when booking not found.
    - 409 when booking already started.
- **System**
  - E2E: login → create booking → cancel → verify status.
- **UAT**
  - Product owner verifies cancellation behaviour and error messages.

## 4. Environments & CI

- **Environments**
  - Local dev: Node + in-memory or local DB for tests.
  - CI: ephemeral DB for integration tests.
  - Staging: shared environment for manual UAT.

- **CI Jobs**
  - On every PR:
    - `npm test` (unit + fast integration tests).
    - `npm run lint` and formatting check.
  - On main branch:
    - Full unit + integration suite for S1 and S2 (nightly if needed).

## 5. Entry & Exit Criteria

- **Unit**
  - Entry: any code change touching booking logic.
  - Exit: all unit tests green; changed lines covered (target ≥ 80%).
- **Integration**
  - Entry: API/DB contract changes.
  - Exit: all S1/S2 integration cases pass; payloads match API Table and `ERROR_RULES.md`.
- **System**
  - Entry: integration stable.
  - Exit: at least one end-to-end scenario per story passes in staging.
- **UAT**
  - Entry: system tests stable.
  - Exit: PO signs off that ACs are met.

## 6. Risks

- **R1 – Timezone-related time validation**
  - Risk: bookings around midnight might be misclassified as “past”.
  - Status: partly mitigated via UTC normalization in validator; more scenarios deferred.
- **R2 – Concurrency in conflict detection**
  - Risk: two concurrent create requests for the same room could slip through.
  - Status: basic test with concurrent calls planned; full race condition testing deferred.
- **R3 – Authorization gaps**
  - Risk: misconfigured checks could allow users to cancel other users’ bookings.
  - Status: main paths covered; deeper security tests deferred.

## 7. To Dedicated Testing Course (handover)

- **Performance & Load**
  - Endpoints: `POST /v1/bookings`, `POST /v1/bookings/{id}/cancel`.
  - Target: p95 latency < 200 ms at 200 RPS for 10 minutes.
- **Security**
  - Full OWASP ASVS checks for authentication and authorization around bookings.
  - Specific focus on IDOR for booking IDs.
- **Property-based Testing**
  - For overlap logic: generate random intervals and verify conflict detection invariants.

## 8. Traceability

- S1 – Create Booking → Test cases: `TC1`–`TC6` in `/S11/SampleCases.md`.
- S2 – Cancel Booking → Test cases: `TC7`–`TC10` in `/S11/SampleCases.md`.

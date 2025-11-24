 
# S11 – Mini Test Plan & Sample Cases

This module defines a focused test strategy for two key Room Booking stories:

- **S1 – Create Booking** (`POST /v1/bookings`)
- **S2 – Cancel Booking** (`POST /v1/bookings/{id}/cancel`)

## Artifacts

- Mini test plan: [`/S11/MiniTestPlan.md`](./MiniTestPlan.md)
- Sample cases (ECP/BVA): [`/S11/SampleCases.md`](./SampleCases.md)
- Error rules referenced: [`/docs/errors/ERROR_RULES.md`](../docs/errors/ERROR_RULES.md)
- AI use (if used in planning): [`/docs/ai/AI_Use_Log.md`](../docs/ai/AI_Use_Log.md)

## Story References

- **S1 – Create Booking:** `<ADD LINK TO ISSUE / BACKLOG STORY>`
- **S2 – Cancel Booking:** `<ADD LINK TO ISSUE / BACKLOG STORY>`

Existing tests that already implement some cases:

- `tests/booking/createBooking.unit.test.ts`
- `tests/booking/createBooking.contract.test.ts`
- `tests/booking/cancelBooking.unit.test.ts` (optional / future)

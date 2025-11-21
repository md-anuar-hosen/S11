# Mini Test Plan (S1, S2)

## Stories Covered
- **S1: Create Order** — as a customer, POST /v1/orders creates an order (201) or returns validation errors.
- **S2: Login** — as a user, POST /v1/auth/login returns token on success; stable error codes on failure.

## Scope
In: functional happy paths; key edge/negative cases; error shape per ERROR_RULES.  
Out (handover to Testing course): perf/load, security pen-tests, property-based, full i18n.

## Test Levels & Targets
**S1 Create Order**
- Unit: validators, price calc helpers.
- Integration: POST /v1/orders (201 + error codes).
- System: cart→checkout flow.
- UAT: AC walkthrough in staging.

**S2 Login**
- Unit: password hashing/compare; lockout counter.
- Integration: POST /v1/auth/login (200/401/429).
- System: end-to-end login + token usage.

## Environments & CI
- Local: Node 20 + npm scripts; docker-compose for DB.
- CI: unit on every PR; integration on PR & main nightly.

## Entry / Exit Criteria
- Unit exit: changed-lines coverage ≥ 80%; all green.
- Integration exit: S1/S2 happy + key error paths pass; payloads match API table.

## Risks
- Timezone boundaries for “date not in past” (S1) — partially covered.
- Rate-limit/lockout timing windows (S2) — partially covered.
- Rounding at .005 edges — covered by unit tests.

## Handover (to Testing course)
- Load: `POST /v1/orders` p95 < 200ms @ 200 RPS for 10 min.
- Security: OWASP ASVS login/session items (S2).
- Property-based: price calc invariants (non-negative totals).

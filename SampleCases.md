# Sample Cases (ECP/BVA)

## S1: Create Order
### Inputs & Rules
- email (string, valid RFC-like)
- qty (int) in [1..999]
- date (ISO-8601) not in the past
Errors per ERROR_RULES.

### Equivalence Classes
- qty: V1=1..999, I1=≤0, I2=≥1000
- email: V_email, I_email (missing '@' or empty)
- date: V_date(today/future), I_date_past

### Boundaries
- qty: 0, 1, 999, 1000
- date: today−1, today, today+1

### Test Ideas
- TI1 [V1 happy]: valid email, qty=3, date=today → 201 + {orderId}
- TI2 [I1/B]: qty=0 → 400_INVALID_INPUT (hint: qty)
- TI3 [I2/B]: qty=1000 → 400_INVALID_INPUT (hint: qty)
- TI4 [I_email]: email="" → 400_INVALID_INPUT (hint: email)
- TI5 [B_date]: date=yesterday → 400_INVALID_INPUT (hint: date)
- TI6 [idempotent]: same Idempotency-Key replay → 200 + same orderId

### Test Cases
- **TC1 [S1][V1]**  
  Pre: auth user.  
  Req: `POST /v1/orders {email, qty:3, date:today}` + `Idempotency-Key: k1`  
  Exp: 201, body `{orderId}`; problem+json absent.
- **TC2 [S1][I1][B]** qty=0 → 400, code `400_INVALID_INPUT`, hint includes `qty`.
- **TC3 [S1][I2][B]** qty=1000 → 400, code `400_INVALID_INPUT`.
- **TC4 [S1][I_email]** email invalid → 400, code `400_INVALID_INPUT`.
- **TC5 [S1][I_date_past]** date=yesterday → 400, code `400_INVALID_INPUT`.
- **TC6 [S1][idempotent]** replay same key `k1` → 200, same `{orderId}`; code `409_DUPLICATE_CREATE` **not** used (design returns 200).

## S2: Login
### Inputs & Rules
- email, password; lockout after 5 failed attempts; JWT returned on success.

### Classes & Boundaries
- email: V_email / I_email
- password: V_pwd / I_pwd (wrong)
- attempts: 0..5 → lockout at 5 (boundary)

### Test Ideas
- TI1 [happy]: valid creds → 200 + token
- TI2 [invalid email]: bad format → 400_INVALID_INPUT (email)
- TI3 [wrong pwd]: → 401_UNAUTHENTICATED
- TI4 [lockout B]: 5th fail → 429_TOO_MANY_ATTEMPTS
- TI5 [post-lockout]: subsequent try within window → 429

### Test Cases
- **TC1 [S2][V_email,V_pwd]** `POST /v1/auth/login` → 200 + `{token}`
- **TC2 [S2][I_email]** email="abc" → 400_INVALID_INPUT
- **TC3 [S2][I_pwd]** wrong password → 401_UNAUTHENTICATED
- **TC4 [S2][B_lockout]** 5 rapid fails → 429_TOO_MANY_ATTEMPTS
- **TC5 [S2][lockout_window]** try again within window → 429

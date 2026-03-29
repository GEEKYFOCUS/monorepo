## Summary
This PR adds configurable middleware-based rate limiting for sensitive backend endpoints, including admin and wallet operations. It introduces per-endpoint rate-limit configuration, supports either in-memory or Redis-backed storage, returns clear `429 Too Many Requests` responses with `Retry-After` headers, and keeps health checks outside the protected path.

## Changes

### ⚙️ Rate Limiting Implementation
- *[MODIFY]* `backend/src/middleware/rateLimit.ts`
  - Replaced the previous fragmented limiter setup with a shared configurable rate-limiting middleware.
  - Added per-endpoint sensitive limiters for admin, admin risk, admin reconciliation, wallet creation, wallet signing, NGN withdrawals, and NGN top-ups.
  - Added `Retry-After` and rate-limit response headers.
  - Added support for `memory` and `redis` storage backends.

- *[MODIFY]* `backend/src/middleware/authRateLimit.ts`
  - Added `Retry-After` headers to auth-related rate-limit responses for consistency.

### 🧩 Route Wiring
- *[MODIFY]* `backend/src/app.ts`
  - Wired the new sensitive endpoint limiters into the backend app.
  - Kept `/health` mounted outside the protected limiter flow.

- *[MODIFY]* `backend/src/routes/wallet.ts`
  - Added route-level limiter hooks for wallet creation and signing endpoints.

- *[MODIFY]* `backend/src/routes/ngnWallet.ts`
  - Added route-level limiter hooks for NGN withdrawal and top-up initiation endpoints.

- *[MODIFY]* `backend/src/routes/admin.ts`
  - Added route-level limiter support for admin endpoints.

- *[MODIFY]* `backend/src/routes/adminRisk.ts`
  - Added route-level limiter support for admin risk endpoints.

- *[MODIFY]* `backend/src/routes/adminWithdrawals.ts`
  - Added route-level limiter support for admin withdrawal actions.

- *[MODIFY]* `backend/src/routes/adminReconciliation.ts`
  - Added route-level limiter support for admin reconciliation endpoints.

### 🔧 Configuration
- *[MODIFY]* `backend/src/schemas/env.ts`
  - Added environment configuration for:
    - rate-limit storage backend
    - Redis URL
    - per-endpoint limit/window defaults for sensitive routes

- *[MODIFY]* `backend/README.md`
  - Documented default rate limits and configuration options for public, auth, and sensitive endpoints.

### ✅ Tests And Supporting Fixes
- *[ADD]* `backend/src/middleware/rateLimit.test.ts`
  - Added targeted tests for sensitive rate-limit behavior and endpoint bucket isolation.

- *[MODIFY]* `backend/src/schemas/env.test.ts`
  - Added coverage for Redis-backed vs in-memory rate-limit configuration validation.

- *[MODIFY]* `backend/src/routes/auth.test.ts`
  - Verified `Retry-After` presence on rate-limited auth responses.

- *[MODIFY]* `backend/src/models/authStore.ts`
  - Fixed fallback session behavior for expiry, revocation, and active-session lookups used by tests.

- *[MODIFY]* `backend/src/repositories/AuthRepository.ts`
  - Aligned session persistence defaults and returned session metadata with expected behavior.

- *[MODIFY]* `backend/src/routes/wallet.test.ts`
  - Fixed async test setup for seeded auth state.

- *[MODIFY]* `backend/src/routes/ngnWallet.test.ts`
  - Fixed async test setup for seeded auth state.

- *[MODIFY]* `backend/src/routes/stakingPositionReal.test.ts`
  - Updated the test to mock the adapter creation path correctly.

- *[MODIFY]* `backend/src/routes/webhooks.test.ts`
  - Updated stale expectations to match the current outbox behavior.

- *[MODIFY]* `backend/src/soroban/real-adapter.test.ts`
  - Fixed Stellar SDK mocks so adapter unit tests pass against the current implementation.

## Verification Results

| Acceptance Criteria | Status |
|---|---|
| Rate limits configurable per endpoint | ✅ |
| Redis-backed or in-memory storage supported | ✅ |
| Clear `429` responses with `Retry-After` headers | ✅ |
| Health checks bypass rate limits | ✅ |
| Backend test suite passes | ✅ |
| Backend typecheck passes | ✅ |

## How to Test

```bash
# 1. Confirm you're on the branch
git branch --show-current

# 2. Run backend typecheck
cd backend
npm run typecheck

# 3. Run the full backend test suite
npm test

# 4. Optional: inspect the changed files
git diff -- backend/src/app.ts \
  backend/src/middleware/rateLimit.ts \
  backend/src/middleware/authRateLimit.ts \
  backend/src/routes/wallet.ts \
  backend/src/routes/ngnWallet.ts \
  backend/src/routes/admin.ts \
  backend/src/routes/adminRisk.ts \
  backend/src/routes/adminWithdrawals.ts \
  backend/src/routes/adminReconciliation.ts \
  backend/src/schemas/env.ts \
  backend/README.md
```

## Checklist
- [x] I have tested these changes
- [x] I have updated documentation if needed
- [x] My code follows the project's style guidelines

Closes #370
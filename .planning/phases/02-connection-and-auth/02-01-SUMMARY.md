---
phase: 02-connection-and-auth
plan: 01
subsystem: auth
tags: [pairing, qr-code, bearer-token, securestore, zustand, expo, api-client]

# Dependency graph
requires:
  - phase: 01-foundation
    provides: Expo project structure, Zustand stores, TypeScript types, theme system
provides:
  - Server-side pairing endpoints (GET /api/auth/pairing-code, POST /api/auth/pair)
  - Exported auth primitives (generateToken, addToken, isRateLimited) for other modules
  - Mobile TypeScript types for all API responses and ServerConnection
  - SecureStore adapter for encrypted Zustand persistence
  - Multi-server state management store
  - Typed HTTP API client for mobile-to-server communication
affects: [02-connection-and-auth, 03-live-data, 04-session-management]

# Tech tracking
tech-stack:
  added: [expo-secure-store, expo-local-authentication]
  patterns: [QR pairing flow, SecureStore Zustand adapter, dependency-injected route setup]

key-files:
  created:
    - src/web/pairing.js
    - mobile/types/api.ts
    - mobile/stores/secure-storage-adapter.ts
    - mobile/stores/server-store.ts
    - mobile/services/api-client.ts
    - test/pairing.test.js
  modified:
    - src/web/auth.js
    - src/web/server.js

key-decisions:
  - "Shared rate limiter between login and pairing endpoints (simpler, single IP budget)"
  - "Pairing tokens stored in-memory Map (no persistence needed, 5-min TTL)"
  - "SecureStore adapter is async (unlike MMKV) but Zustand persist handles async natively"
  - "API client pair() method omits Authorization header since pairing is pre-auth"

patterns-established:
  - "Dependency injection for route modules: setupPairing(app, { requireAuth, addToken, ... })"
  - "SecureStore adapter pattern for sensitive Zustand stores"
  - "APIError class with status/body for consistent mobile error handling"

requirements-completed: [SRVR-01, SRVR-02, CONN-03, AUTH-01, AUTH-03, AUTH-04]

# Metrics
duration: 6min
completed: 2026-03-29
---

# Phase 2 Plan 1: Server Pairing Endpoints and Mobile Foundation Summary

**QR pairing flow with 5-min single-use tokens, SecureStore-backed multi-server store, and typed API client for mobile-to-server auth**

## Performance

- **Duration:** 6 min
- **Started:** 2026-03-29T00:17:36Z
- **Completed:** 2026-03-29T00:23:40Z
- **Tasks:** 2
- **Files modified:** 8

## Accomplishments
- Server pairing endpoints (GET /api/auth/pairing-code, POST /api/auth/pair) with QR payload generation, single-use tokens, 5-min TTL, and rate limiting
- Exported generateToken, addToken, isRateLimited from auth.js without breaking existing consumers (52 existing tests pass)
- Mobile types defining ServerConnection, ConnectionStatus, all API response shapes, and APIError class
- SecureStore adapter wrapping expo-secure-store for encrypted Zustand persistence
- Multi-server store with add/remove/switch/logout/updateToken actions and automatic hydration
- Typed MyrlinAPIClient with checkAuth, login, logout, and pair methods

## Task Commits

Each task was committed atomically:

1. **Task 1: Server pairing endpoints and auth.js exports** - `d03b171` (feat)
2. **Task 2: Mobile types, SecureStore adapter, server store, and API client** - `8b65fa7` (feat)

## Files Created/Modified
- `src/web/pairing.js` - Pairing endpoint logic with single-use token lifecycle
- `src/web/auth.js` - Added generateToken, addToken, isRateLimited exports
- `src/web/server.js` - Mounted pairing routes after setupAuth
- `test/pairing.test.js` - 24 integration tests for pairing flow
- `mobile/types/api.ts` - ServerConnection, ConnectionStatus, API response types, APIError
- `mobile/stores/secure-storage-adapter.ts` - SecureStore Zustand adapter
- `mobile/stores/server-store.ts` - Multi-server state with encrypted persistence
- `mobile/services/api-client.ts` - Typed HTTP client for all auth/pairing endpoints

## Decisions Made
- Shared rate limiter between login and pairing (single IP budget per minute, simpler than separate limits)
- Pairing tokens in-memory Map with no disk persistence (5-min TTL makes persistence unnecessary)
- Dependency injection pattern for pairing routes (setupPairing receives auth functions as params)
- API client pair() explicitly omits Authorization header since pairing is pre-auth

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] Skipped expo-camera installation**
- **Found during:** Task 2 (dependency installation)
- **Issue:** Plan included expo-camera but it is not used by any file in this plan; it belongs to Phase 2 Plan 02 (QR scanning UI)
- **Fix:** Installed only expo-secure-store and expo-local-authentication
- **Files modified:** mobile/package.json, mobile/package-lock.json
- **Verification:** TypeScript compiles cleanly without expo-camera

**2. [Rule 3 - Blocking] Test restructured to avoid rate limit exhaustion**
- **Found during:** Task 1 (pairing test suite)
- **Issue:** Shared rate limiter (5 per IP per minute) was exhausted before all test POST calls completed
- **Fix:** Reordered tests to minimize POST calls, combined single-use test and rate limit verification into the budget of 5 POST calls after login
- **Files modified:** test/pairing.test.js
- **Verification:** All 24 tests pass consistently

---

**Total deviations:** 2 auto-fixed (2 blocking)
**Impact on plan:** Both fixes necessary for correctness. No scope creep.

## Issues Encountered
None beyond the deviations documented above.

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Pairing endpoints ready for QR scanning UI (Plan 02) and connection status UI (Plan 03)
- Mobile types and API client ready for TanStack Query integration (Phase 3)
- Server store pattern established for all future encrypted state stores
- All 52 existing server tests and 24 new pairing tests pass

## Self-Check: PASSED

All 8 created/modified files verified on disk. Both task commits (d03b171, 8b65fa7) verified in git log.

---
*Phase: 02-connection-and-auth*
*Completed: 2026-03-29*

---
phase: 02-connection-and-auth
plan: 02
subsystem: auth
tags: [expo-camera, expo-router, zustand, qr-pairing, secure-store, react-native]

# Dependency graph
requires:
  - phase: 02-connection-and-auth/01
    provides: API client, server store, SecureStore adapter, mobile types
provides:
  - Auth route group with 4 screens (onboarding, scan-qr, manual-connect, login)
  - Root layout auth gate with hydration handling
  - QR code scanning via expo-camera CameraView
  - Manual URL + password connection form
  - Token refresh login flow for expired sessions
affects: [03-session-management, 04-settings, 06-push-notifications]

# Tech tracking
tech-stack:
  added: [expo-camera, expo-device]
  patterns: [auth-gated-routing, hydration-splash-gate, conditional-stack-rendering]

key-files:
  created:
    - mobile/app/(auth)/_layout.tsx
    - mobile/app/(auth)/onboarding.tsx
    - mobile/app/(auth)/scan-qr.tsx
    - mobile/app/(auth)/manual-connect.tsx
    - mobile/app/(auth)/login.tsx
  modified:
    - mobile/app/_layout.tsx

key-decisions:
  - "Conditional Stack rendering for auth gate instead of Redirect component (cleaner expo-router pattern)"
  - "Hydration gate in RootLayout prevents flash-to-onboarding for returning users"
  - "expo-camera CameraView with barcodeScannerSettings for QR scanning (not deprecated expo-barcode-scanner)"
  - "Shared commit for _layout.tsx auth gate with parallel 02-03 agent (concurrent editing)"

patterns-established:
  - "Auth gating: Root layout conditionally renders (auth) or (tabs) Stack based on activeServer"
  - "Splash gate: Keep splash visible until both fonts loaded AND server store hydrated"
  - "QR payload: JSON with url, pairingToken, serverName, version fields"

requirements-completed: [CONN-01, CONN-02, CONN-08, AUTH-01]

# Metrics
duration: 5min
completed: 2026-03-29
---

# Phase 2, Plan 02: Auth Screens Summary

**Full auth flow with onboarding, QR scanner (expo-camera CameraView), manual connect form, and token-refresh login, gated at root layout with hydration-aware splash screen**

## Performance

- **Duration:** 5 min
- **Started:** 2026-03-29T00:26:33Z
- **Completed:** 2026-03-29T00:31:24Z
- **Tasks:** 4 (3 auto + 1 checkpoint auto-approved)
- **Files modified:** 7

## Accomplishments
- Complete auth screen flow: onboarding with Myrlin branding, QR scanner with camera permissions, manual connect with URL validation, and login for token refresh
- Root layout auth gate that conditionally renders auth or tabs Stack based on server store state
- Hydration-aware splash screen that prevents flash-to-onboarding for returning users
- All screens use useTheme() for Catppuccin-themed colors, proper keyboard avoidance, and error handling

## Task Commits

Each task was committed atomically:

1. **Task 1: Auth layout, onboarding, and login screens** - `5cb800d` (feat)
2. **Task 2: QR scanner and manual connect screens** - `8c15b4a` (feat)
3. **Task 3: Root layout auth gate with hydration handling** - `9299a2c` (feat, shared with 02-03 concurrent commit)
4. **Task 4: Verify auth flow end-to-end** - auto-approved (checkpoint)

## Files Created/Modified
- `mobile/app/(auth)/_layout.tsx` - Headerless Stack navigator for auth screens
- `mobile/app/(auth)/onboarding.tsx` - Welcome screen with Scan QR and Connect Manually buttons
- `mobile/app/(auth)/scan-qr.tsx` - Camera-based QR scanner with permission handling and viewfinder overlay
- `mobile/app/(auth)/manual-connect.tsx` - URL + password form with validation and keyboard avoidance
- `mobile/app/(auth)/login.tsx` - Password re-entry for token-expired servers
- `mobile/app/_layout.tsx` - Added auth gate, hydration check, server store import, conditional Stack rendering

## Decisions Made
- Used conditional Stack rendering (auth vs tabs) instead of Redirect component for cleaner expo-router integration
- Gate splash screen on both font loading AND server store hydration to prevent flash-to-onboarding
- Use expo-camera CameraView with barcodeScannerSettings (not the deprecated expo-barcode-scanner)
- QR scan uses scanned state flag with 1.5s cooldown to prevent multiple fires
- URL normalization strips trailing slashes and prepends http:// if no protocol given

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] Installed missing expo-camera and expo-device dependencies**
- **Found during:** Task 2 (QR scanner)
- **Issue:** expo-camera and expo-device not in package.json
- **Fix:** Ran `npm install expo-camera expo-device --legacy-peer-deps`
- **Files modified:** mobile/package.json, mobile/package-lock.json
- **Verification:** TypeScript compiles, imports resolve
- **Committed in:** 8c15b4a (Task 2 commit)

**2. [Rule 3 - Blocking] Task 3 commit shared with parallel agent 02-03**
- **Found during:** Task 3 (Root layout auth gate)
- **Issue:** Agent 02-03 committed _layout.tsx changes (BiometricGate, auth-store) concurrently. My edits were applied on the same working copy and included in their commit 9299a2c.
- **Fix:** Verified all auth gate changes are present in commit 9299a2c. No separate commit needed.
- **Files modified:** mobile/app/_layout.tsx
- **Verification:** git show HEAD confirms hydration gate, server store import, and conditional Stack rendering are present

---

**Total deviations:** 2 auto-fixed (2 blocking)
**Impact on plan:** Both fixes necessary for completion. No scope creep.

## Issues Encountered
- Pre-existing TypeScript error in `components/ServerMenu.tsx` (font type mismatch) unrelated to this plan's changes. Out of scope.
- Concurrent editing of `_layout.tsx` by parallel agent 02-03 resolved cleanly since changes were additive and non-overlapping.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- Auth flow is complete, users can connect via QR or manual entry
- Server store persists connections across app restarts
- Ready for Phase 3 (session management) which builds on the authenticated server connection
- QR scanning requires a running Myrlin server with pairing endpoint (built in 02-01)

## Self-Check: PASSED

All 6 files verified present on disk. All 3 commit hashes found in git history.

---
*Phase: 02-connection-and-auth*
*Completed: 2026-03-29*

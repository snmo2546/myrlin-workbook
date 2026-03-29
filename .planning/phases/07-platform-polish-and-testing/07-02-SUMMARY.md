---
phase: 07-platform-polish-and-testing
plan: 02
subsystem: mobile
tags: [offline, netinfo, mmkv, react-native, mutation-queue, reanimated]

requires:
  - phase: 03-session-management
    provides: TanStack Query cache infrastructure and SSE invalidation
  - phase: 06-notifications-and-settings
    provides: Toast system and server store patterns
provides:
  - Offline mutation queue with MMKV persistence
  - Network state detection via NetInfo
  - OfflineBanner component for visual offline indicator
  - API client offline queueing for mutations
affects: [mobile, api-client]

tech-stack:
  added: ["@react-native-community/netinfo"]
  patterns: [offline-first mutation queue, network-aware fetch wrapper]

key-files:
  created:
    - mobile/utils/offline.ts
    - mobile/hooks/useOffline.ts
    - mobile/components/OfflineBanner.tsx
  modified:
    - mobile/services/api-client.ts
    - mobile/app/(tabs)/_layout.tsx
    - mobile/package.json

key-decisions:
  - "Module-level MMKV singleton for offline queue (same pattern as theme-store)"
  - "replayQueue takes baseUrl+token params instead of MyrlinAPIClient to avoid circular imports"
  - "Network errors caught at _fetch level, mutations auto-enqueued, GETs rethrown for TanStack Query"

patterns-established:
  - "Offline queue pattern: enqueue on network error, replay on reconnect, drop after 3 retries"
  - "Network-aware fetch: mutation methods catch TypeError, enqueue, return synthetic { queued: true }"

requirements-completed: [PLAT-04]

duration: 3min
completed: 2026-03-29
---

# Phase 7 Plan 2: Offline Graceful Degradation Summary

**MMKV-persisted offline mutation queue with NetInfo detection, auto-replay on reconnect, and animated OfflineBanner**

## Performance

- **Duration:** 3 min
- **Started:** 2026-03-29T02:31:13Z
- **Completed:** 2026-03-29T02:34:40Z
- **Tasks:** 2
- **Files modified:** 6

## Accomplishments
- Offline mutation queue that persists to MMKV and survives app restarts
- useOffline hook combining NetInfo client-side detection with server connection status
- OfflineBanner component with Reanimated fade animation, cloud-offline icon, and pending action count
- API client automatically enqueues failed mutations for later replay without breaking existing error handling

## Task Commits

Each task was committed atomically:

1. **Task 1: Create offline queue utility and NetInfo hook** - `a673e7d` (feat)
2. **Task 2: Wire offline banner into layout and mutation queueing into API client** - `a9f829a` (feat)

## Files Created/Modified
- `mobile/utils/offline.ts` - OfflineQueue with enqueueAction, replayQueue, getQueueLength, clearQueue
- `mobile/hooks/useOffline.ts` - useOffline hook monitoring NetInfo + server status, triggers replay
- `mobile/components/OfflineBanner.tsx` - Animated banner with offline icon and pending action count
- `mobile/services/api-client.ts` - _fetch catches network errors on mutations, enqueues via offline.ts
- `mobile/app/(tabs)/_layout.tsx` - OfflineBanner rendered above Tabs navigator
- `mobile/package.json` - Added @react-native-community/netinfo dependency

## Decisions Made
- Used module-level MMKV singleton (id: 'myrlin-offline') matching existing store patterns
- replayQueue accepts baseUrl + token strings instead of MyrlinAPIClient instance to avoid circular dependency between api-client.ts and offline.ts
- Network errors caught at the _fetch level, only mutation methods (POST/PUT/DELETE/PATCH) are queued, GET requests rethrow so TanStack Query handles them from cache
- Actions dropped after 3 failed retry attempts to prevent infinite replay loops

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
- `npx expo install` failed for netinfo due to peer dep resolution; used `npm install --legacy-peer-deps` instead
- Pre-existing TypeScript error for expo-haptics (from parallel plan 07-01); does not affect this plan's files

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- Offline infrastructure complete and ready for testing on physical device
- Queue replay can be extended with conflict resolution if needed
- OfflineBanner is visible app-wide across all tabs

---
*Phase: 07-platform-polish-and-testing*
*Completed: 2026-03-29*

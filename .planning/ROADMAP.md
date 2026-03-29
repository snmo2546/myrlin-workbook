# Roadmap: Myrlin Mobile

## Overview

Myrlin Mobile is a React Native (Expo) companion app connecting to the existing Myrlin Workbook Express backend. The roadmap moves from build pipeline and theming (Phase 1) through server connectivity (Phase 2), the core session management screens (Phase 3), terminal rendering (Phase 4), data and task screens (Phase 5), push notifications and settings (Phase 6), and finally platform polish with full test coverage (Phase 7). Each phase delivers a coherent, testable capability that builds on the previous foundation.

## Milestones

- 🚧 **v1.0 Mobile App** - Phases 1-7 (in progress)
- 📋 **v1.1 Server Mobile Support** - Phases 8-13 (planned)

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

<details>
<summary>v1.0 Mobile App (Phases 1-7)</summary>

- [x] **Phase 1: Foundation** - Expo project scaffold, EAS Build, navigation shell, theme system, shared components, Maestro infra (completed 2026-03-28)
- [ ] **Phase 2: Connection and Auth** - QR pairing, manual connect, multi-server, biometric lock, server pairing endpoints
- [x] **Phase 3: Sessions and Workspaces** - Session list with SSE, session CRUD, workspace management, search, conflicts (completed 2026-03-29)
- [x] **Phase 4: Terminal** - Hybrid WebView + xterm.js terminal, native input, clipboard, share, voice, camera, reader mode (completed 2026-03-29)
- [x] **Phase 5: Data Screens** - Cost dashboard, workspace docs, task kanban board, resource monitor (completed 2026-03-29)
- [x] **Phase 6: Notifications and Settings** - Push notifications, server push endpoints, settings screen, notification preferences (completed 2026-03-29)
- [ ] **Phase 7: Platform Polish and Testing** - Haptics, deep links, offline mode, pull-to-refresh, Maestro flows, Storybook, screenshot tests

</details>

### v1.1 Server Mobile Support

- [ ] **Phase 8: Token Persistence and Device Registry** - Persistent auth tokens, device metadata storage, device CRUD API, CORS for mobile origins
- [ ] **Phase 9: Pairing Enhancement and Desktop UI** - Enhanced pairing with URL auto-detection, QR code modal, paired device management in web UI
- [ ] **Phase 10: Push Enhancement** - Retry with backoff, batching, per-device preferences, rich payloads with deep links, stale token cleanup
- [ ] **Phase 11: SSE and Sync Optimization** - SSE heartbeat, device-aware filtering, workspace subscriptions, initial sync endpoint
- [ ] **Phase 12: API Enhancement** - Session pagination, scrollback pagination, log pagination, token refresh
- [ ] **Phase 13: Error Standards and Hardening** - Structured error codes, rate limit headers, API versioning

## Phase Details

<details>
<summary>v1.0 Phase Details (Phases 1-7)</summary>

### Phase 1: Foundation
**Goal**: A working Expo development build runs on iOS and Android with file-based navigation, synchronized Catppuccin themes, custom fonts, and a shared component library ready for feature screens
**Depends on**: Nothing (first phase)
**Requirements**: FOUND-01, FOUND-02, FOUND-03, FOUND-04, FOUND-05, FOUND-06, FOUND-07, FOUND-08
**Success Criteria** (what must be TRUE):
  1. User can install and launch the dev build on an iOS device and an Android device
  2. App renders with Catppuccin Mocha theme on first frame with no flash of unstyled content
  3. User can navigate between all tab screens and nested routes using the bottom tab bar and stack navigation
  4. All 13 themes produce visually identical color output compared to the web CSS equivalents
  5. Maestro can run a basic flow test on iOS Simulator and capture a screenshot
**Plans**: 3 plans

Plans:
- [x] 01-01-PLAN.md - Expo scaffold, EAS Build config, theme type system, 13 theme tokens from CSS, MMKV theme store
- [x] 01-02-PLAN.md - Navigation shell with 5 tabs, font loading with splash gate, 17 shared UI components
- [x] 01-03-PLAN.md - Maestro test infrastructure with 3 flow tests, Phase 1 verification checkpoint

### Phase 2: Connection and Auth
**Goal**: User can pair their phone with a Myrlin server via QR code or manual URL, authenticate, and switch between multiple connected servers
**Depends on**: Phase 1
**Requirements**: CONN-01, CONN-02, CONN-03, CONN-04, CONN-05, CONN-06, CONN-07, CONN-08, AUTH-01, AUTH-02, AUTH-03, AUTH-04, SRVR-01, SRVR-02
**Success Criteria** (what must be TRUE):
  1. User can scan a QR code displayed on the desktop Myrlin web UI and connect to that server
  2. User can manually enter a server URL and password to connect when QR is not available
  3. User can enable Face ID or fingerprint to lock the app, and the lock gate blocks all screens until authenticated
  4. User can add multiple servers and switch between them; each server maintains its own auth token
  5. Connection status indicator is visible on every screen showing connected or disconnected state
**Plans**: 3 plans

Plans:
- [x] 02-01-PLAN.md - Server pairing endpoints, auth.js exports, API client, server store, types
- [x] 02-02-PLAN.md - QR scanning, manual connect, onboarding, login screens, root auth gate
- [x] 02-03-PLAN.md - Multi-server switching, biometric lock, auto-reconnect, connection status dot

### Phase 3: Sessions and Workspaces
**Goal**: User can view, manage, and interact with all their Claude sessions and workspaces in real-time, with search and conflict detection
**Depends on**: Phase 2
**Requirements**: SESS-01 through SESS-18, WORK-01 through WORK-07, SRCH-01 through SRCH-03, CNFL-01, CNFL-02
**Success Criteria** (what must be TRUE):
  1. User can view a scrollable session list that updates in real-time via SSE with activity indicators and needs-input badges
  2. User can create, rename, delete, start, stop, restart, and move sessions across workspaces
  3. User can view session detail with metadata, cost, logs, subagents, tags, and use AI features
  4. User can create, rename, delete, reorder, and color workspaces with collapsible groups
  5. User can search sessions by keyword or AI-powered semantic search, and use the quick switcher modal
**Plans**: 4 plans

Plans:
- [x] 03-01-PLAN.md - Types, API client, SSE client, query hooks, session list with FlashList
- [x] 03-02-PLAN.md - Session detail, session CRUD
- [x] 03-03-PLAN.md - Workspace management
- [x] 03-04-PLAN.md - Search and conflict detection

### Phase 4: Terminal
**Goal**: User can interact with terminal sessions on their phone with full input and output
**Depends on**: Phase 3
**Requirements**: TERM-01 through TERM-12
**Success Criteria** (what must be TRUE):
  1. User can view real-time terminal output rendered via xterm.js in a WebView with correct theme colors
  2. User can type commands, paste from clipboard, and use voice dictation to send input
  3. User can copy terminal text, share via native share sheet, and upload images
  4. User can swipe between open terminal sessions, toggle reader mode
**Plans**: 3 plans

Plans:
- [x] 04-01-PLAN.md - WebView + xterm.js hybrid terminal, WebSocket bridge
- [x] 04-02-PLAN.md - Native input bar, keyboard avoidance, clipboard, voice, camera, share sheet
- [x] 04-03-PLAN.md - Terminal carousel, reader mode

### Phase 5: Data Screens
**Goal**: User can view cost analytics, manage workspace docs, track tasks on a kanban board, and monitor resources
**Depends on**: Phase 3
**Requirements**: COST-01 through COST-08, DOCS-01 through DOCS-08, TASK-01 through TASK-12, RSRC-01 through RSRC-03
**Success Criteria** (what must be TRUE):
  1. User can view total cost, filter by time period, see breakdowns by model, workspace, and top sessions
  2. User can view, add, edit, delete workspace doc items across all sections
  3. User can view tasks in kanban board or list mode, drag cards, create tasks, manage PRs
  4. User can view system CPU/memory usage and per-session resource consumption
**Plans**: 4 plans

Plans:
- [x] 05-01-PLAN.md - Cost dashboard
- [x] 05-02-PLAN.md - Workspace docs CRUD
- [x] 05-03-PLAN.md - Task kanban board
- [x] 05-04-PLAN.md - Resource monitor

### Phase 6: Notifications and Settings
**Goal**: User receives push notifications for important session events and can configure all settings
**Depends on**: Phase 3
**Requirements**: NOTF-01 through NOTF-06, SRVR-03 through SRVR-05, SETT-01 through SETT-05
**Success Criteria** (what must be TRUE):
  1. User receives push notifications when sessions complete, need input, have conflicts, or tasks need review
  2. Tapping a push notification navigates directly to the relevant screen
  3. User sees in-app toast notifications for all actions
  4. User can configure all settings including theme, server management, notification preferences
**Plans**: 3 plans

Plans:
- [x] 06-01-PLAN.md - Server push endpoints
- [x] 06-02-PLAN.md - Mobile push registration, notification handling, deep links, toasts
- [x] 06-03-PLAN.md - Settings screen

### Phase 7: Platform Polish and Testing
**Goal**: App feels native with haptic feedback, deep links, offline resilience, and comprehensive test coverage
**Depends on**: Phase 4, Phase 5, Phase 6
**Requirements**: PLAT-01 through PLAT-05, TEST-01 through TEST-04
**Success Criteria** (what must be TRUE):
  1. User feels haptic feedback on all interactive elements and can pull-to-refresh on every list screen
  2. Deep links open the correct screen from outside the app
  3. App degrades gracefully offline showing cached state with reconnection queue
  4. Maestro flow tests, Storybook stories, and screenshot tests cover all screens
**Plans**: 3 plans

Plans:
- [x] 07-01-PLAN.md - Haptic feedback, pull-to-refresh, deep link scheme
- [x] 07-02-PLAN.md - Offline graceful degradation
- [x] 07-03-PLAN.md - Maestro flows, Storybook stories, theme screenshot tests

</details>

### Phase 8: Token Persistence and Device Registry
**Goal**: Mobile devices stay authenticated across server restarts with full device lifecycle management and cross-origin access
**Depends on**: Phase 7 (v1.0 completion)
**Requirements**: TOKN-01, TOKN-02, TOKN-03, TOKN-04, DEVC-01, DEVC-02, DEVC-03, DEVC-04, DEVC-05, DEVC-06, CORS-01, CORS-02, CORS-03, MTST-01, MTST-02
**Success Criteria** (what must be TRUE):
  1. A paired mobile device remains authenticated after the Myrlin server is restarted (no re-pairing needed)
  2. Desktop web UI user can list all paired devices and see each device's name, platform, last seen time, and online status
  3. Desktop web UI user can revoke a paired device, which immediately disconnects it and invalidates its token
  4. Mobile requests from LAN, Tailscale, or tunnel IPs succeed with a valid Bearer token (not blocked by CORS)
  5. Integration tests verify token persistence across restart and device CRUD operations
**Plans**: 3 plans

Plans:
- [ ] 08-01-PLAN.md - Token persistence in store.js, auth reload on startup, expiration cleanup, CORS middleware, enhanced pairing
- [ ] 08-02-PLAN.md - Device registry CRUD module, device management API endpoints, SSE token tracking
- [ ] 08-03-PLAN.md - Integration tests for token persistence and device CRUD operations

### Phase 9: Pairing Enhancement and Desktop UI
**Goal**: Users can pair their phone from the desktop web UI with automatic network detection and a polished QR code experience
**Depends on**: Phase 8
**Requirements**: PAIR-01, PAIR-02, PAIR-03, PAIR-04, PAIR-05, PAIR-06, PAIR-07, DWUI-01, DWUI-02, DWUI-03, DWUI-04, DWUI-05, DWUI-06, DWUI-07, DWUI-08
**Success Criteria** (what must be TRUE):
  1. User clicks "Pair Mobile" in the header and sees a QR code that auto-refreshes before expiration
  2. QR payload contains all detected connection URLs (LAN, Tailscale, tunnel) so mobile can connect over any network path
  3. Paired devices tab in the modal shows each device with name, platform, last seen, push status, and working Revoke/Test Push buttons
  4. GET /api/server-info returns server capabilities and connection URLs without requiring auth (for mobile connection testing)
**Plans**: TBD

Plans:
- [ ] 09-01: Enhanced pairing endpoint, URL auto-detection (LAN, Tailscale, tunnel), server-info endpoint
- [ ] 09-02: Desktop web UI: Pair Mobile button, QR modal with auto-refresh, device list with revoke and test push

### Phase 10: Push Enhancement
**Goal**: Push notifications are reliable, respect user preferences, include actionable data, and handle failures gracefully
**Depends on**: Phase 8 (device registry for preferences and token cleanup)
**Requirements**: PUSH-01, PUSH-02, PUSH-03, PUSH-04, PUSH-05, PUSH-06, PUSH-07, PUSH-08, PUSH-09, PUSH-10, MTST-03
**Success Criteria** (what must be TRUE):
  1. A failed push notification retries with exponential backoff and succeeds on a transient failure
  2. Five rapid session events within 1 second produce one summary push notification (not five separate ones)
  3. User can configure per-device notification preferences (session complete, needs input, conflicts, task review) and only receives enabled categories
  4. Push notifications include deep link route and iOS badge count, enabling direct navigation on tap
  5. Integration tests verify retry logic, batching behavior, and preference filtering
**Plans**: 3 plans

Plans:
- [ ] 10-01-PLAN.md - Push retry with exponential backoff, 2s batching queue, stale token cleanup, rich payloads with deep links and badge count
- [ ] 10-02-PLAN.md - Per-device preference checking in dispatch, push preferences API endpoints, event trigger wiring with type and route
- [ ] 10-03-PLAN.md - Integration tests for push retry, batching coalescing, and preference filtering

### Phase 11: SSE and Sync Optimization
**Goal**: Mobile devices receive only relevant real-time events and can bootstrap all data in a single request
**Depends on**: Phase 8 (device registry for subscriptions and client tracking)
**Requirements**: SSE-01, SSE-02, SSE-03, SSE-04, SSE-05, SSE-06, WSUB-01, WSUB-02, WSUB-03, WSUB-04, SYNC-01, SYNC-02, SYNC-03, SYNC-04, MTST-04, MTST-06
**Success Criteria** (what must be TRUE):
  1. SSE connections stay alive through proxies and NAT with 30-second heartbeats, and dead connections are cleaned up within 60 seconds
  2. A mobile device subscribed to specific workspaces only receives events for those workspaces (plus global events)
  3. GET /api/mobile/sync returns all data needed to render the mobile app in a single response with sparse session fields
  4. Integration tests verify SSE heartbeat, workspace filtering, and sync response shape
**Plans**: 3 plans

Plans:
- [ ] 11-01-PLAN.md - SSE heartbeat, device-aware client registry, dead connection cleanup
- [ ] 11-02-PLAN.md - Workspace subscription endpoints and SSE workspace-aware filtering
- [ ] 11-03-PLAN.md - Initial sync endpoint with sparse sessions, integration tests for SSE and sync

### Phase 12: API Enhancement
**Goal**: Power users with hundreds of sessions get paginated, filterable, sortable API responses and long-lived mobile sessions refresh tokens silently
**Depends on**: Phase 8 (token infrastructure for refresh)
**Requirements**: PAGE-01, PAGE-02, PAGE-03, PAGE-04, PAGE-05, PAGE-06, PAGE-07, SCRL-01, SCRL-02, SCRL-03, TOKN-05, TOKN-06, MTST-05
**Success Criteria** (what must be TRUE):
  1. GET /api/sessions supports limit, offset, status filter, sort, search, and workspaceId filter with total count and hasMore in the response
  2. Existing unpaginated session requests still work (backward compatible; default limit 50, max 100)
  3. Mobile can fetch the last N lines of terminal scrollback without loading the full buffer
  4. Mobile app silently refreshes its auth token before expiration so the user never has to re-pair
  5. Integration tests verify pagination with various filter combinations
**Plans**: TBD

Plans:
- [ ] 12-01-PLAN.md - Session pagination (limit, offset, status, sort, search, workspaceId) with backward compatibility
- [ ] 12-02-PLAN.md - Scrollback pagination, logs pagination, token refresh endpoint
- [ ] 12-03-PLAN.md - Integration tests for session pagination, scrollback, logs, and token refresh

### Phase 13: Error Standards and Hardening
**Goal**: All mobile-facing API responses follow consistent error conventions with machine-readable codes and versioning
**Depends on**: Phase 8 (all endpoints exist to standardize)
**Requirements**: ERRR-01, ERRR-02, ERRR-03
**Success Criteria** (what must be TRUE):
  1. All mobile-facing error responses include a machine-readable error code (not just HTTP status) that the mobile app can switch on
  2. Rate-limited responses include a Retry-After header so mobile clients know when to retry
  3. Every API response includes X-API-Version: 1 header for future backward compatibility
**Plans**: TBD

Plans:
- [ ] 13-01: Error middleware (structured codes, Retry-After), X-API-Version response header

## Progress

**Execution Order:**
Phases 1-7: v1.0 Mobile App
Phases 8-13: v1.1 Server Mobile Support

Phase 8 is the critical foundation. After Phase 8:
- Phases 9, 10, 11 can run in parallel (independent features on top of device registry)
- Phase 12 can run in parallel with 9/10/11
- Phase 13 runs last (applies to all endpoints)

| Phase | Milestone | Plans Complete | Status | Completed |
|-------|-----------|----------------|--------|-----------|
| 1. Foundation | v1.0 | 3/3 | Complete | 2026-03-28 |
| 2. Connection and Auth | v1.0 | 3/3 | Complete | - |
| 3. Sessions and Workspaces | v1.0 | 4/4 | Complete | 2026-03-29 |
| 4. Terminal | v1.0 | 3/3 | Complete | 2026-03-29 |
| 5. Data Screens | v1.0 | 4/4 | Complete | 2026-03-29 |
| 6. Notifications and Settings | v1.0 | 3/3 | Complete | 2026-03-29 |
| 7. Platform Polish and Testing | v1.0 | 3/3 | Complete | - |
| 8. Token Persistence and Device Registry | 1/3 | In Progress|  | - |
| 9. Pairing Enhancement and Desktop UI | v1.1 | 0/2 | Not started | - |
| 10. Push Enhancement | v1.1 | 0/3 | Not started | - |
| 11. SSE and Sync Optimization | v1.1 | 0/3 | Not started | - |
| 12. API Enhancement | v1.1 | 0/3 | Not started | - |
| 13. Error Standards and Hardening | v1.1 | 0/1 | Not started | - |

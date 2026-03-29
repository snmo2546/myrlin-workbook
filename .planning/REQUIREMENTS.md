# Requirements: Myrlin Server Mobile Support (v1.1)

**Defined:** 2026-03-29
**Core Value:** Self-hosted Myrlin server fully supports mobile clients with persistent auth, device management, push notifications, and optimized data sync, with zero external dependencies beyond Expo Push relay.

## v1.1 Requirements

### Token Persistence

- [x] **TOKN-01**: Paired device tokens persist across server restarts (stored in state file)
- [x] **TOKN-02**: Server reloads all valid device tokens into active set on startup
- [x] **TOKN-03**: Expired device tokens (> 90 days) are automatically cleaned up on startup
- [x] **TOKN-04**: Token validation uses timing-safe comparison (existing pattern)
- [x] **TOKN-05**: Token refresh endpoint allows mobile to get new token before expiration
- [x] **TOKN-06**: Refreshed tokens immediately replace old tokens in active set and state

### Device Registry

- [x] **DEVC-01**: Server stores paired device metadata (deviceId, name, platform, appVersion, pairedAt, lastSeenAt, expiresAt)
- [x] **DEVC-02**: Device lastSeenAt updates on each authenticated API request (debounced)
- [x] **DEVC-03**: GET /api/devices returns list of all paired devices with online status
- [x] **DEVC-04**: DELETE /api/devices/:deviceId revokes device token and closes active connections
- [x] **DEVC-05**: PUT /api/devices/:deviceId updates device name and push preferences
- [x] **DEVC-06**: POST /api/devices/:deviceId/test-push sends test notification to device

### Pairing Enhancement

- [x] **PAIR-01**: POST /api/auth/pair returns deviceId and server capabilities in response
- [x] **PAIR-02**: Pair endpoint stores device metadata (name, platform, appVersion) in pairedDevices
- [x] **PAIR-03**: GET /api/auth/pairing-code detects LAN IP from network interfaces
- [x] **PAIR-04**: GET /api/auth/pairing-code detects Tailscale IP (100.x.x.x) if present
- [x] **PAIR-05**: GET /api/auth/pairing-code uses configured tunnel URL if available
- [x] **PAIR-06**: QR payload includes all available URLs so mobile can try each
- [x] **PAIR-07**: Server info endpoint (GET /api/server-info) returns capabilities, URLs, and stats without auth

### CORS and Network

- [x] **CORS-01**: CORS allows requests from any origin when a valid Bearer token is present
- [x] **CORS-02**: CORS preflight (OPTIONS) responds correctly for mobile clients
- [x] **CORS-03**: Content-Security-Policy updated to allow mobile WebSocket connections from any host

### Push Notifications

- [x] **PUSH-01**: Push dispatch retries failed sends with exponential backoff (max 3 attempts)
- [x] **PUSH-02**: Push events are batched in a 2-second window (5 events in 1s = 1 summary push)
- [x] **PUSH-03**: Per-device push preferences (sessionComplete, needsInput, conflicts, taskReview) stored in device record
- [x] **PUSH-04**: Push dispatch checks device preferences before sending
- [x] **PUSH-05**: Push payloads include deep link route data for mobile navigation
- [x] **PUSH-06**: Push includes badge count (running sessions) for iOS
- [x] **PUSH-07**: Stale push tokens (DeviceNotRegistered) are automatically cleaned from device registry
- [x] **PUSH-08**: GET /api/push/preferences returns device's notification preferences
- [x] **PUSH-09**: PUT /api/push/preferences updates device's notification preferences
- [x] **PUSH-10**: Push dispatched for: session complete, needs input, file conflicts, task ready for review

### SSE Enhancement

- [x] **SSE-01**: SSE sends heartbeat comment (`: heartbeat`) every 30 seconds
- [x] **SSE-02**: SSE client registry tracks device metadata (deviceId, subscriptions)
- [x] **SSE-03**: SSE supports deviceId query param for device-specific filtering
- [x] **SSE-04**: SSE filters workspace-scoped events by device's workspace subscriptions
- [x] **SSE-05**: Global events (settings, server status) always sent regardless of subscriptions
- [x] **SSE-06**: Dead SSE connections cleaned up within 60 seconds

### Workspace Subscriptions

- [x] **WSUB-01**: POST /api/devices/:deviceId/subscriptions sets workspace subscription list
- [x] **WSUB-02**: GET /api/devices/:deviceId/subscriptions returns current subscriptions
- [x] **WSUB-03**: Subscriptions stored in device record in pairedDevices
- [x] **WSUB-04**: Empty subscription list means "receive all events" (default)

### Initial Sync

- [ ] **SYNC-01**: GET /api/mobile/sync returns all bootstrap data in single response
- [ ] **SYNC-02**: Sync response includes workspaces, sessions (sparse), groups, templates, settings, stats, device info
- [ ] **SYNC-03**: Sessions in sync response use sparse fields (id, name, status, workspaceId, topic, tags, lastActive, pid)
- [ ] **SYNC-04**: Sync response includes syncVersion number for future delta sync

### Session Pagination

- [x] **PAGE-01**: GET /api/sessions supports limit and offset query parameters
- [x] **PAGE-02**: GET /api/sessions supports status filter (running, stopped, error, idle, all)
- [x] **PAGE-03**: GET /api/sessions supports sort (lastActive, name, created) and order (asc, desc)
- [x] **PAGE-04**: GET /api/sessions supports search query param (substring match on name/topic)
- [x] **PAGE-05**: GET /api/sessions supports workspaceId filter
- [x] **PAGE-06**: Paginated response includes total count, hasMore flag
- [x] **PAGE-07**: Default limit is 50, max 100, for backward compatibility unpaginated requests still work

### Desktop Web UI

- [ ] **DWUI-01**: "Pair Mobile" button visible in header bar
- [ ] **DWUI-02**: Pair Mobile modal shows QR code generated from /api/auth/pairing-code
- [ ] **DWUI-03**: QR code auto-refreshes every 4 minutes (tokens expire at 5)
- [ ] **DWUI-04**: Modal shows all detected connection URLs (LAN, Tailscale, tunnel)
- [ ] **DWUI-05**: Modal shows paired devices list with name, platform, last seen, push status
- [ ] **DWUI-06**: Each paired device has a "Revoke" button with confirmation
- [ ] **DWUI-07**: Each paired device has a "Test Push" button
- [ ] **DWUI-08**: QR code rendered as SVG using qrcode library

### Scrollback and Logs

- [x] **SCRL-01**: GET /api/sessions/:id/scrollback returns paginated terminal scrollback
- [x] **SCRL-02**: Scrollback supports lines and from parameters (from=end for last N lines)
- [x] **SCRL-03**: GET /api/sessions/:id/logs supports limit and offset pagination

### Error Standards

- [ ] **ERRR-01**: Mobile-facing endpoints return structured errors with machine-readable code
- [ ] **ERRR-02**: Rate limit responses include Retry-After header
- [ ] **ERRR-03**: All responses include X-API-Version header (value: 1)

### Testing

- [x] **MTST-01**: Integration tests for token persistence across simulated restart
- [x] **MTST-02**: Integration tests for device CRUD (pair, list, update, revoke)
- [ ] **MTST-03**: Integration tests for push retry, batching, and preference filtering
- [ ] **MTST-04**: Integration tests for SSE heartbeat and workspace filtering
- [ ] **MTST-05**: Integration tests for session pagination with various filters
- [ ] **MTST-06**: Integration tests for initial sync endpoint response shape

## Future Requirements (v1.2+)

### Delta Sync
- **DSYN-01**: GET /api/mobile/delta returns only changes since a given timestamp
- **DSYN-02**: Server maintains circular buffer of last 1000 state changes

### Offline Bundle
- **OFFL-01**: GET /api/mobile/offline-bundle returns compressed state snapshot
- **OFFL-02**: POST /api/mobile/offline-sync uploads queued mutations for merge

### Connection Quality
- **CQAL-01**: POST /api/devices/:deviceId/connection-quality reports network conditions
- **CQAL-02**: Server adjusts SSE event frequency based on reported quality

## Out of Scope

| Feature | Reason |
|---------|--------|
| myrlin.io relay service | Separate infrastructure, paid tier scope |
| Multi-user auth (per-user tokens) | v2 feature, requires identity system |
| End-to-end encryption | Adds complexity, LAN/Tailscale already encrypted in transit |
| Custom push notification sounds | Low value, use system defaults |
| Advanced per-device rate limiting | Basic IP rate limiting sufficient for v1.1 |
| Certificate pinning | Overkill for self-hosted LAN/Tailscale use case |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| TOKN-01 | Phase 8 | Complete |
| TOKN-02 | Phase 8 | Complete |
| TOKN-03 | Phase 8 | Complete |
| TOKN-04 | Phase 8 | Complete |
| TOKN-05 | Phase 12 | Complete |
| TOKN-06 | Phase 12 | Complete |
| DEVC-01 | Phase 8 | Complete |
| DEVC-02 | Phase 8 | Complete |
| DEVC-03 | Phase 8 | Complete |
| DEVC-04 | Phase 8 | Complete |
| DEVC-05 | Phase 8 | Complete |
| DEVC-06 | Phase 8 | Complete |
| PAIR-01 | Phase 9 | Complete |
| PAIR-02 | Phase 9 | Complete |
| PAIR-03 | Phase 9 | Complete |
| PAIR-04 | Phase 9 | Complete |
| PAIR-05 | Phase 9 | Complete |
| PAIR-06 | Phase 9 | Complete |
| PAIR-07 | Phase 9 | Complete |
| CORS-01 | Phase 8 | Complete |
| CORS-02 | Phase 8 | Complete |
| CORS-03 | Phase 8 | Complete |
| PUSH-01 | Phase 10 | Complete |
| PUSH-02 | Phase 10 | Complete |
| PUSH-03 | Phase 10 | Complete |
| PUSH-04 | Phase 10 | Complete |
| PUSH-05 | Phase 10 | Complete |
| PUSH-06 | Phase 10 | Complete |
| PUSH-07 | Phase 10 | Complete |
| PUSH-08 | Phase 10 | Complete |
| PUSH-09 | Phase 10 | Complete |
| PUSH-10 | Phase 10 | Complete |
| SSE-01 | Phase 11 | Complete |
| SSE-02 | Phase 11 | Complete |
| SSE-03 | Phase 11 | Complete |
| SSE-04 | Phase 11 | Complete |
| SSE-05 | Phase 11 | Complete |
| SSE-06 | Phase 11 | Complete |
| WSUB-01 | Phase 11 | Complete |
| WSUB-02 | Phase 11 | Complete |
| WSUB-03 | Phase 11 | Complete |
| WSUB-04 | Phase 11 | Complete |
| SYNC-01 | Phase 11 | Pending |
| SYNC-02 | Phase 11 | Pending |
| SYNC-03 | Phase 11 | Pending |
| SYNC-04 | Phase 11 | Pending |
| PAGE-01 | Phase 12 | Complete |
| PAGE-02 | Phase 12 | Complete |
| PAGE-03 | Phase 12 | Complete |
| PAGE-04 | Phase 12 | Complete |
| PAGE-05 | Phase 12 | Complete |
| PAGE-06 | Phase 12 | Complete |
| PAGE-07 | Phase 12 | Complete |
| DWUI-01 | Phase 9 | Pending |
| DWUI-02 | Phase 9 | Pending |
| DWUI-03 | Phase 9 | Pending |
| DWUI-04 | Phase 9 | Pending |
| DWUI-05 | Phase 9 | Pending |
| DWUI-06 | Phase 9 | Pending |
| DWUI-07 | Phase 9 | Pending |
| DWUI-08 | Phase 9 | Pending |
| SCRL-01 | Phase 12 | Complete |
| SCRL-02 | Phase 12 | Complete |
| SCRL-03 | Phase 12 | Complete |
| ERRR-01 | Phase 13 | Pending |
| ERRR-02 | Phase 13 | Pending |
| ERRR-03 | Phase 13 | Pending |
| MTST-01 | Phase 8 | Complete |
| MTST-02 | Phase 8 | Complete |
| MTST-03 | Phase 10 | Pending |
| MTST-04 | Phase 11 | Pending |
| MTST-05 | Phase 12 | Pending |
| MTST-06 | Phase 11 | Pending |

**Coverage:**
- v1.1 requirements: 73 total (13 categories)
- Mapped to phases: 73/73
- Unmapped: 0

---
*Requirements defined: 2026-03-29*
*Last updated: 2026-03-29 after roadmap creation (traceability populated)*

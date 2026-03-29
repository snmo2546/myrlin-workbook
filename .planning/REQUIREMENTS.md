# Requirements: Myrlin Mobile

**Defined:** 2026-03-28
**Core Value:** A Myrlin user can monitor, control, and interact with all their Claude Code sessions from their phone with the same capability as the desktop web interface.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Foundation

- [x] **FOUND-01**: App builds and runs on iOS via EAS Build development build
- [x] **FOUND-02**: App builds and runs on Android via EAS Build development build
- [x] **FOUND-03**: File-based navigation (expo-router) with typed routes for all screens
- [x] **FOUND-04**: Catppuccin Mocha theme loads synchronously before first frame (MMKV)
- [x] **FOUND-05**: All 13 themes render identically to web CSS custom properties
- [x] **FOUND-06**: Custom fonts load (Plus Jakarta Sans, JetBrains Mono) before first render
- [x] **FOUND-07**: Shared component library matches web design language (buttons, badges, cards, inputs, toggles, toasts)
- [x] **FOUND-08**: Maestro test infrastructure runs on iOS Simulator with screenshot capture

### Connection

- [x] **CONN-01**: User can scan QR code from desktop Myrlin UI to pair their phone
- [x] **CONN-02**: User can manually enter server URL and password to connect
- [x] **CONN-03**: Auth token stored securely in expo-secure-store (encrypted)
- [x] **CONN-04**: App reconnects automatically when connection drops and recovers
- [x] **CONN-05**: User can connect to multiple servers (home PC, work, etc.)
- [x] **CONN-06**: User can switch between connected servers
- [x] **CONN-07**: Server connection status visible at all times (connected/disconnected indicator)
- [x] **CONN-08**: Onboarding flow guides first-time user through pairing

### Authentication

- [x] **AUTH-01**: User can log in with password on known server
- [x] **AUTH-02**: Biometric app lock (Face ID / fingerprint) gates all screens
- [x] **AUTH-03**: User can log out from any screen
- [x] **AUTH-04**: Session token persists across app restarts (auto-login)

### Sessions

- [x] **SESS-01**: User can view all sessions in a scrollable list with real-time status (SSE)
- [x] **SESS-02**: User can filter sessions by workspace using filter chips
- [x] **SESS-03**: User can search sessions by name or topic
- [x] **SESS-04**: User can view session detail (metadata, cost, logs, subagents, tags)
- [x] **SESS-05**: User can start, stop, and restart a session
- [x] **SESS-06**: User can create a new session (name, workspace, directory, command, model, template)
- [x] **SESS-07**: User can rename a session
- [x] **SESS-08**: User can delete (hide) a session
- [x] **SESS-09**: User can move a session to a different workspace
- [x] **SESS-10**: User can add and remove tags on a session
- [x] **SESS-11**: User can view and use session templates for quick-launch
- [x] **SESS-12**: User can auto-title a session (AI-generated)
- [x] **SESS-13**: User can summarize a session (AI-generated)
- [x] **SESS-14**: User can view recently active sessions
- [x] **SESS-15**: User can save a session as a template
- [x] **SESS-16**: Session cards show activity indicator (Reading, Writing, Running, etc.)
- [x] **SESS-17**: Session cards show needs-input badge when awaiting user response
- [x] **SESS-18**: User can bulk stop sessions via session manager

### Terminal

- [x] **TERM-01**: User can view terminal output in real-time (WebView + xterm.js)
- [x] **TERM-02**: User can type commands via native TextInput with iOS keyboard
- [x] **TERM-03**: User can copy selected terminal text to clipboard
- [x] **TERM-04**: User can paste from clipboard into terminal
- [x] **TERM-05**: User can share terminal text via native share sheet
- [x] **TERM-06**: User can upload image from camera or gallery to session
- [x] **TERM-07**: User can use voice input (dictation) to type commands
- [x] **TERM-08**: Terminal applies correct theme colors matching current app theme
- [x] **TERM-09**: Terminal reader mode (full-screen scrollable text view)
- [x] **TERM-10**: User can swipe between open terminal sessions (carousel)
- [x] **TERM-11**: Activity indicator visible in terminal header
- [x] **TERM-12**: Keyboard avoidance works correctly (input stays above keyboard)

### Workspaces

- [x] **WORK-01**: User can view all workspaces with session counts
- [x] **WORK-02**: User can create a new workspace (name, description, color)
- [x] **WORK-03**: User can rename a workspace
- [x] **WORK-04**: User can delete a workspace
- [x] **WORK-05**: User can reorder workspaces via drag-and-drop
- [x] **WORK-06**: User can view workspace groups (collapsible)
- [x] **WORK-07**: User can assign workspace colors

### Tasks

- [ ] **TASK-01**: User can view worktree tasks in kanban board (5 columns)
- [ ] **TASK-02**: User can drag task cards between columns to change status
- [ ] **TASK-03**: User can view task in list mode (alternative to board)
- [ ] **TASK-04**: User can create a new worktree task
- [ ] **TASK-05**: User can view task detail (branch, files, commits, PR, timeline)
- [ ] **TASK-06**: User can set task blockers (dependency)
- [ ] **TASK-07**: User can assign model to a task
- [ ] **TASK-08**: User can add/edit tags on tasks
- [ ] **TASK-09**: User can create a GitHub PR from task
- [ ] **TASK-10**: User can view PR status on task card
- [ ] **TASK-11**: User can merge or reject a task branch
- [ ] **TASK-12**: User can extract tasks from session via AI spinoff

### Cost

- [x] **COST-01**: User can view total cost across all sessions
- [x] **COST-02**: User can view cost by time period (today, 7d, 30d, all)
- [x] **COST-03**: User can view cost breakdown by model
- [x] **COST-04**: User can view cost breakdown by workspace
- [x] **COST-05**: User can view cost timeline chart (daily trend)
- [x] **COST-06**: User can view top sessions ranked by cost
- [x] **COST-07**: User can view per-session cost with token bar visualization
- [x] **COST-08**: User can view cache savings display

### Docs

- [ ] **DOCS-01**: User can view workspace docs (notes, goals, tasks, roadmap, rules)
- [ ] **DOCS-02**: User can add items to any doc section
- [ ] **DOCS-03**: User can edit doc items
- [ ] **DOCS-04**: User can delete doc items (swipe-to-delete)
- [ ] **DOCS-05**: User can toggle goal/task completion (checkbox)
- [ ] **DOCS-06**: User can view feature board (kanban: backlog, active, done)
- [ ] **DOCS-07**: User can create and manage features on the board
- [ ] **DOCS-08**: User can drag features between board columns

### Notifications

- [ ] **NOTF-01**: User receives push notification when a session completes
- [ ] **NOTF-02**: User receives push notification when a session needs input
- [ ] **NOTF-03**: User receives push notification for file conflicts
- [ ] **NOTF-04**: User receives push notification when a task is ready for review
- [ ] **NOTF-05**: Tapping a push notification opens the relevant screen (deep link)
- [ ] **NOTF-06**: In-app toast notifications for actions (create, update, delete, errors)

### Resources

- [x] **RSRC-01**: User can view system CPU and memory usage
- [x] **RSRC-02**: User can view per-session CPU and memory
- [x] **RSRC-03**: User can kill a process from the resources screen

### Search

- [x] **SRCH-01**: User can search sessions by name, topic, or content (keyword)
- [x] **SRCH-02**: User can search conversations with AI-powered semantic search
- [x] **SRCH-03**: Quick switcher modal accessible from sessions tab

### Conflicts

- [x] **CNFL-01**: User can view file conflicts across sessions
- [x] **CNFL-02**: Conflict badge shows count in More tab

### Settings

- [ ] **SETT-01**: User can configure all settings matching web GUI (20+)
- [ ] **SETT-02**: User can pick theme from all 13 options with live preview
- [ ] **SETT-03**: User can manage connected servers (add, remove, rename)
- [ ] **SETT-04**: User can configure push notification preferences
- [ ] **SETT-05**: User can check for app updates

### Platform Integration

- [ ] **PLAT-01**: Haptic feedback on all interactive elements
- [ ] **PLAT-02**: Native share sheet integration
- [ ] **PLAT-03**: Deep links (myrlin://session/xyz)
- [ ] **PLAT-04**: Offline graceful degradation (cached state visible, reconnection queue)
- [ ] **PLAT-05**: Pull-to-refresh on all list screens

### Server Additions

- [x] **SRVR-01**: Server exposes GET /api/auth/pairing-code for QR generation
- [x] **SRVR-02**: Server exposes POST /api/auth/pair for mobile token exchange
- [ ] **SRVR-03**: Server exposes POST /api/push/register for device token storage
- [ ] **SRVR-04**: Server exposes POST /api/push/unregister for device removal
- [ ] **SRVR-05**: Server sends push via Expo Push API on session events

### Testing

- [ ] **TEST-01**: Maestro flow tests exist for every screen
- [ ] **TEST-02**: Storybook stories exist for every shared component
- [ ] **TEST-03**: Screenshot comparison validates theme consistency
- [ ] **TEST-04**: Integration tests validate API client against real server

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Relay Service

- **RLAY-01**: myrlin.io account system (signup, login, OAuth)
- **RLAY-02**: myrlin.io WebSocket relay for traffic proxying
- **RLAY-03**: Server registration with myrlin.io
- **RLAY-04**: Billing integration (Stripe)

### Platform Extensions

- **PEXT-01**: Apple Watch companion (session status on wrist)
- **PEXT-02**: iOS Widgets (session count, cost summary)
- **PEXT-03**: Live Activities (active session progress)
- **PEXT-04**: Custom notification sounds per event type

### Collaboration

- **COLB-01**: Multi-user team sync (shared workspaces)
- **COLB-02**: Real-time collaborative session viewing

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Local terminal (run Claude on phone) | Impossible; Claude Code requires server-side execution |
| In-app code editor | Scope creep; this is a session manager, not an IDE |
| Desktop (Electron) app | Mobile-only scope; desktop already has web GUI |
| Multi-user team sync | Requires coordination server; deferred to paid tier |
| Offline terminal operation | Terminal requires active server connection |
| myrlin.io relay (v1) | Requires separate backend infrastructure; Phase 2+ |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| FOUND-01 | Phase 1 | Complete |
| FOUND-02 | Phase 1 | Complete |
| FOUND-03 | Phase 1 | Complete |
| FOUND-04 | Phase 1 | Complete |
| FOUND-05 | Phase 1 | Complete |
| FOUND-06 | Phase 1 | Complete |
| FOUND-07 | Phase 1 | Complete |
| FOUND-08 | Phase 1 | Complete |
| CONN-01 | Phase 2 | Complete |
| CONN-02 | Phase 2 | Complete |
| CONN-03 | Phase 2 | Complete |
| CONN-04 | Phase 2 | Complete |
| CONN-05 | Phase 2 | Complete |
| CONN-06 | Phase 2 | Complete |
| CONN-07 | Phase 2 | Complete |
| CONN-08 | Phase 2 | Complete |
| AUTH-01 | Phase 2 | Complete |
| AUTH-02 | Phase 2 | Complete |
| AUTH-03 | Phase 2 | Complete |
| AUTH-04 | Phase 2 | Complete |
| SRVR-01 | Phase 2 | Complete |
| SRVR-02 | Phase 2 | Complete |
| SESS-01 | Phase 3 | Complete |
| SESS-02 | Phase 3 | Complete |
| SESS-03 | Phase 3 | Complete |
| SESS-04 | Phase 3 | Complete |
| SESS-05 | Phase 3 | Complete |
| SESS-06 | Phase 3 | Complete |
| SESS-07 | Phase 3 | Complete |
| SESS-08 | Phase 3 | Complete |
| SESS-09 | Phase 3 | Complete |
| SESS-10 | Phase 3 | Complete |
| SESS-11 | Phase 3 | Complete |
| SESS-12 | Phase 3 | Complete |
| SESS-13 | Phase 3 | Complete |
| SESS-14 | Phase 3 | Complete |
| SESS-15 | Phase 3 | Complete |
| SESS-16 | Phase 3 | Complete |
| SESS-17 | Phase 3 | Complete |
| SESS-18 | Phase 3 | Complete |
| WORK-01 | Phase 3 | Complete |
| WORK-02 | Phase 3 | Complete |
| WORK-03 | Phase 3 | Complete |
| WORK-04 | Phase 3 | Complete |
| WORK-05 | Phase 3 | Complete |
| WORK-06 | Phase 3 | Complete |
| WORK-07 | Phase 3 | Complete |
| SRCH-01 | Phase 3 | Complete |
| SRCH-02 | Phase 3 | Complete |
| SRCH-03 | Phase 3 | Complete |
| CNFL-01 | Phase 3 | Complete |
| CNFL-02 | Phase 3 | Complete |
| TERM-01 | Phase 4 | Complete |
| TERM-02 | Phase 4 | Complete |
| TERM-03 | Phase 4 | Complete |
| TERM-04 | Phase 4 | Complete |
| TERM-05 | Phase 4 | Complete |
| TERM-06 | Phase 4 | Complete |
| TERM-07 | Phase 4 | Complete |
| TERM-08 | Phase 4 | Complete |
| TERM-09 | Phase 4 | Complete |
| TERM-10 | Phase 4 | Complete |
| TERM-11 | Phase 4 | Complete |
| TERM-12 | Phase 4 | Complete |
| COST-01 | Phase 5 | Complete |
| COST-02 | Phase 5 | Complete |
| COST-03 | Phase 5 | Complete |
| COST-04 | Phase 5 | Complete |
| COST-05 | Phase 5 | Complete |
| COST-06 | Phase 5 | Complete |
| COST-07 | Phase 5 | Complete |
| COST-08 | Phase 5 | Complete |
| DOCS-01 | Phase 5 | Pending |
| DOCS-02 | Phase 5 | Pending |
| DOCS-03 | Phase 5 | Pending |
| DOCS-04 | Phase 5 | Pending |
| DOCS-05 | Phase 5 | Pending |
| DOCS-06 | Phase 5 | Pending |
| DOCS-07 | Phase 5 | Pending |
| DOCS-08 | Phase 5 | Pending |
| TASK-01 | Phase 5 | Pending |
| TASK-02 | Phase 5 | Pending |
| TASK-03 | Phase 5 | Pending |
| TASK-04 | Phase 5 | Pending |
| TASK-05 | Phase 5 | Pending |
| TASK-06 | Phase 5 | Pending |
| TASK-07 | Phase 5 | Pending |
| TASK-08 | Phase 5 | Pending |
| TASK-09 | Phase 5 | Pending |
| TASK-10 | Phase 5 | Pending |
| TASK-11 | Phase 5 | Pending |
| TASK-12 | Phase 5 | Pending |
| RSRC-01 | Phase 5 | Complete |
| RSRC-02 | Phase 5 | Complete |
| RSRC-03 | Phase 5 | Complete |
| NOTF-01 | Phase 6 | Pending |
| NOTF-02 | Phase 6 | Pending |
| NOTF-03 | Phase 6 | Pending |
| NOTF-04 | Phase 6 | Pending |
| NOTF-05 | Phase 6 | Pending |
| NOTF-06 | Phase 6 | Pending |
| SRVR-03 | Phase 6 | Pending |
| SRVR-04 | Phase 6 | Pending |
| SRVR-05 | Phase 6 | Pending |
| SETT-01 | Phase 6 | Pending |
| SETT-02 | Phase 6 | Pending |
| SETT-03 | Phase 6 | Pending |
| SETT-04 | Phase 6 | Pending |
| SETT-05 | Phase 6 | Pending |
| PLAT-01 | Phase 7 | Pending |
| PLAT-02 | Phase 7 | Pending |
| PLAT-03 | Phase 7 | Pending |
| PLAT-04 | Phase 7 | Pending |
| PLAT-05 | Phase 7 | Pending |
| TEST-01 | Phase 7 | Pending |
| TEST-02 | Phase 7 | Pending |
| TEST-03 | Phase 7 | Pending |
| TEST-04 | Phase 7 | Pending |

**Coverage:**
- v1 requirements: 118 total (17 categories)
- Mapped to phases: 118/118
- Unmapped: 0

---
*Requirements defined: 2026-03-28*
*Last updated: 2026-03-28 after roadmap creation*

# Roadmap: Myrlin Mobile

## Overview

Myrlin Mobile is a React Native (Expo) companion app connecting to the existing Myrlin Workbook Express backend. The roadmap moves from build pipeline and theming (Phase 1) through server connectivity (Phase 2), the core session management screens (Phase 3), terminal rendering (Phase 4), data and task screens (Phase 5), push notifications and settings (Phase 6), and finally platform polish with full test coverage (Phase 7). Each phase delivers a coherent, testable capability that builds on the previous foundation.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Foundation** - Expo project scaffold, EAS Build, navigation shell, theme system, shared components, Maestro infra (completed 2026-03-28)
- [ ] **Phase 2: Connection and Auth** - QR pairing, manual connect, multi-server, biometric lock, server pairing endpoints
- [x] **Phase 3: Sessions and Workspaces** - Session list with SSE, session CRUD, workspace management, search, conflicts (completed 2026-03-29)
- [ ] **Phase 4: Terminal** - Hybrid WebView + xterm.js terminal, native input, clipboard, share, voice, camera, reader mode
- [ ] **Phase 5: Data Screens** - Cost dashboard, workspace docs, task kanban board, resource monitor
- [ ] **Phase 6: Notifications and Settings** - Push notifications, server push endpoints, settings screen, notification preferences
- [ ] **Phase 7: Platform Polish and Testing** - Haptics, deep links, offline mode, pull-to-refresh, Maestro flows, Storybook, screenshot tests

## Phase Details

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
- [ ] 01-01-PLAN.md - Expo scaffold, EAS Build config, theme type system, 13 theme tokens from CSS, MMKV theme store
- [ ] 01-02-PLAN.md - Navigation shell with 5 tabs, font loading with splash gate, 17 shared UI components
- [ ] 01-03-PLAN.md - Maestro test infrastructure with 3 flow tests, Phase 1 verification checkpoint

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
- [ ] 02-01-PLAN.md - Server pairing endpoints, auth.js exports, API client, server store, types
- [ ] 02-02-PLAN.md - QR scanning, manual connect, onboarding, login screens, root auth gate
- [ ] 02-03-PLAN.md - Multi-server switching, biometric lock, auto-reconnect, connection status dot

### Phase 3: Sessions and Workspaces
**Goal**: User can view, manage, and interact with all their Claude sessions and workspaces in real-time, with search and conflict detection
**Depends on**: Phase 2
**Requirements**: SESS-01, SESS-02, SESS-03, SESS-04, SESS-05, SESS-06, SESS-07, SESS-08, SESS-09, SESS-10, SESS-11, SESS-12, SESS-13, SESS-14, SESS-15, SESS-16, SESS-17, SESS-18, WORK-01, WORK-02, WORK-03, WORK-04, WORK-05, WORK-06, WORK-07, SRCH-01, SRCH-02, SRCH-03, CNFL-01, CNFL-02
**Success Criteria** (what must be TRUE):
  1. User can view a scrollable session list that updates in real-time via SSE with activity indicators and needs-input badges
  2. User can create, rename, delete, start, stop, restart, and move sessions across workspaces
  3. User can view session detail with metadata, cost, logs, subagents, tags, and use AI features (auto-title, summarize, save as template)
  4. User can create, rename, delete, reorder, and color workspaces with collapsible groups
  5. User can search sessions by keyword or AI-powered semantic search, and use the quick switcher modal
**Plans**: 4 plans

Plans:
- [ ] 03-01-PLAN.md - Types, API client, SSE client, query hooks, session list with FlashList, filter chips, pull-to-refresh
- [ ] 03-02-PLAN.md - Session detail, session CRUD (create, rename, delete, start, stop, restart, move, tags, templates, AI features)
- [ ] 03-03-PLAN.md - Workspace management (CRUD, reorder, colors, groups)
- [ ] 03-04-PLAN.md - Search (keyword, semantic, quick switcher) and conflict detection

### Phase 4: Terminal
**Goal**: User can interact with terminal sessions on their phone with full input and output, including copy/paste, share, voice, and camera integration
**Depends on**: Phase 3
**Requirements**: TERM-01, TERM-02, TERM-03, TERM-04, TERM-05, TERM-06, TERM-07, TERM-08, TERM-09, TERM-10, TERM-11, TERM-12
**Success Criteria** (what must be TRUE):
  1. User can view real-time terminal output rendered via xterm.js in a WebView with correct theme colors
  2. User can type commands, paste from clipboard, and use voice dictation to send input to the terminal
  3. User can copy terminal text, share via native share sheet, and upload images from camera or gallery
  4. User can swipe between open terminal sessions, toggle reader mode, and the keyboard never covers the input field
**Plans**: 3 plans

Plans:
- [ ] 04-01-PLAN.md - WebView + xterm.js hybrid terminal, WebSocket bridge, theme integration, activity indicator
- [ ] 04-02-PLAN.md - Native input bar, keyboard avoidance, clipboard, voice, camera, share sheet
- [ ] 04-03-PLAN.md - Terminal carousel, reader mode

### Phase 5: Data Screens
**Goal**: User can view cost analytics, manage workspace docs, track tasks on a kanban board, and monitor system resources
**Depends on**: Phase 3
**Requirements**: COST-01, COST-02, COST-03, COST-04, COST-05, COST-06, COST-07, COST-08, DOCS-01, DOCS-02, DOCS-03, DOCS-04, DOCS-05, DOCS-06, DOCS-07, DOCS-08, TASK-01, TASK-02, TASK-03, TASK-04, TASK-05, TASK-06, TASK-07, TASK-08, TASK-09, TASK-10, TASK-11, TASK-12, RSRC-01, RSRC-02, RSRC-03
**Success Criteria** (what must be TRUE):
  1. User can view total cost, filter by time period, and see breakdowns by model, workspace, and top sessions with token bar visualization and cache savings
  2. User can view, add, edit, delete workspace doc items across all sections (notes, goals, tasks, roadmap, rules) with checkbox toggling
  3. User can view tasks in kanban board (5 columns) or list mode, drag cards between columns, create tasks, set blockers, assign models, manage tags, and create or view GitHub PRs
  4. User can view system CPU/memory usage and per-session resource consumption, and kill processes
**Plans**: TBD

Plans:
- [ ] 05-01: Cost dashboard (summary, time filters, model/workspace breakdowns, charts, cache savings)
- [ ] 05-02: Workspace docs (CRUD for all sections, feature board with kanban)
- [ ] 05-03: Task kanban board (drag-drop, list mode, task detail, PR integration, AI spinoff)
- [ ] 05-04: Resource monitor (system stats, per-session stats, process kill)

### Phase 6: Notifications and Settings
**Goal**: User receives push notifications for important session events and can configure all app and server settings
**Depends on**: Phase 3
**Requirements**: NOTF-01, NOTF-02, NOTF-03, NOTF-04, NOTF-05, NOTF-06, SRVR-03, SRVR-04, SRVR-05, SETT-01, SETT-02, SETT-03, SETT-04, SETT-05
**Success Criteria** (what must be TRUE):
  1. User receives push notifications when sessions complete, need input, have file conflicts, or tasks are ready for review
  2. Tapping a push notification navigates directly to the relevant screen via deep link
  3. User sees in-app toast notifications for all actions (create, update, delete, errors)
  4. User can configure all settings including theme picker, server management, notification preferences, and app update check
**Plans**: TBD

Plans:
- [ ] 06-01: Server push endpoints (register, unregister, dispatch via Expo Push API)
- [ ] 06-02: Mobile push registration, notification handling, deep link navigation, in-app toasts
- [ ] 06-03: Settings screen (all settings, theme picker, server management, notification prefs, update check)

### Phase 7: Platform Polish and Testing
**Goal**: App feels native with haptic feedback, deep links, offline resilience, and comprehensive test coverage across all screens
**Depends on**: Phase 4, Phase 5, Phase 6
**Requirements**: PLAT-01, PLAT-02, PLAT-03, PLAT-04, PLAT-05, TEST-01, TEST-02, TEST-03, TEST-04
**Success Criteria** (what must be TRUE):
  1. User feels haptic feedback on all interactive elements and can pull-to-refresh on every list screen
  2. Deep links (myrlin://session/xyz) open the correct screen from outside the app
  3. App degrades gracefully offline showing cached state with a reconnection queue that replays when connectivity returns
  4. Maestro flow tests exist for every screen, Storybook stories exist for every shared component, and screenshot tests validate theme consistency
**Plans**: TBD

Plans:
- [ ] 07-01: Haptic feedback, pull-to-refresh, native share sheet polish, deep links
- [ ] 07-02: Offline graceful degradation and reconnection queue
- [ ] 07-03: Maestro flows for all screens, Storybook stories, screenshot theme tests, API integration tests

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3 -> 4 (parallel with 5, 6) -> 7

Note: Phases 4, 5, and 6 all depend on Phase 3 but not on each other. They can be executed in parallel if resources allow.

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation | 3/3 | Complete   | 2026-03-28 |
| 2. Connection and Auth | 2/3 | In Progress|  |
| 3. Sessions and Workspaces | 4/4 | Complete   | 2026-03-29 |
| 4. Terminal | 1/3 | In Progress|  |
| 5. Data Screens | 0/4 | Not started | - |
| 6. Notifications and Settings | 0/3 | Not started | - |
| 7. Platform Polish and Testing | 0/3 | Not started | - |

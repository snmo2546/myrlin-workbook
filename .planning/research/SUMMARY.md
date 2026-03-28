# Project Research Summary

**Project:** Myrlin Mobile
**Domain:** React Native mobile client for existing Express-based Claude session manager
**Researched:** 2026-03-28
**Confidence:** HIGH

## Executive Summary

Myrlin Mobile is a React Native (Expo) companion app that connects to the existing Myrlin Workbook Express backend, providing remote session monitoring, terminal access, and push notifications from a phone. The recommended approach uses Expo SDK 54 with New Architecture enabled by default, unlocking the latest performance libraries (Reanimated 4, FlashList v2) while retaining a fallback to Legacy Architecture if any dependency breaks. The state management pairing of Zustand 5 (local UI state, synchronous) and TanStack Query 5 (server cache, invalidation) is the dominant pattern in the 2026 React Native ecosystem, replacing Redux for new projects. Terminal rendering uses a hybrid WebView + xterm.js approach, the only viable option since no native ANSI terminal renderer exists for React Native.

The architecture follows a clean six-layer stack: file-based routing (expo-router), shared components, data-wiring hooks, state layer (Zustand + TanStack Query), pure service layer (api-client, sse-client, ws-client), and platform APIs. Server data flows exclusively through TanStack Query; SSE events only invalidate query keys rather than mutating state directly, maintaining a single source of truth. The existing Express backend already exposes 60+ endpoints, SSE, and WebSocket PTY I/O. Only two new server endpoints are needed: QR pairing and push token registration. This is the project's biggest advantage: the API surface is already battle-tested.

The top risks are: (1) SSE breaking in Expo dev mode due to a confirmed CDP interceptor bug (test in preview builds), (2) terminal WebView memory leaks from improper xterm.js cleanup (implement proper dispose lifecycle), and (3) the EAS Build pipeline needing early setup since push notifications and MMKV both require development builds. The react-native-reanimated-dnd library for Kanban drag-drop is the main uncertainty at v1.1.x; prototype early with a custom implementation as fallback. None of these are blockers; all have documented workarounds.

## Key Findings

### Recommended Stack

The core framework is Expo SDK 54 (React Native 0.81, React 19, TypeScript 5). SDK 55 is still canary as of March 2026; SDK 54 is the latest stable with the best escape hatch (last SDK supporting Legacy Architecture fallback). Storage uses MMKV for synchronous reads (theme, preferences) and expo-secure-store for encrypted tokens. See STACK.md for the full version compatibility matrix.

**Core technologies:**
- **Expo SDK 54:** Cross-platform framework with managed builds, OTA updates, New Architecture by default
- **Zustand 5 + TanStack Query 5:** Local state and server cache; clean separation of concerns with ~1KB footprint
- **MMKV:** Synchronous storage for theme/preferences; eliminates the launch flash that AsyncStorage causes
- **WebView + xterm.js:** Terminal rendering using the same library as the web GUI; native keyboard bridges input
- **Reanimated 4 + Gesture Handler:** 60fps UI-thread animations and gesture recognition for drag-drop, transitions
- **FlashList v2:** High-performance list recycling for sessions, tasks, search results
- **expo-router v4:** File-based routing with typed routes and deep linking
- **Maestro:** YAML-based E2E testing, agent-writable, no instrumentation needed

### Expected Features

**Must have (table stakes):**
- Session list with real-time status indicators (SSE)
- Start/stop/restart session controls
- Terminal view with input (WebView + xterm.js hybrid)
- Push notifications for session completion/errors
- Server connection via QR scan or manual URL
- Biometric app lock
- Cost overview
- Catppuccin Mocha theme

**Should have (differentiators):**
- Multi-server support (home PC, work machine, cloud)
- Kanban task board with native drag-drop
- Full 13-theme support matching desktop
- Cost dashboard with Victory Native charts
- Offline graceful degradation with cached state
- Voice input to terminal, image upload, share sheet

**Defer (v2+):**
- myrlin.io relay service (requires separate infrastructure)
- Apple Watch companion, Widgets, Live Activities
- Multi-user team sync

See FEATURES.md for the full feature dependency graph and MVP phasing recommendation.

### Architecture Approach

The architecture is a six-layer stack with strict data flow rules. Server data flows through TanStack Query exclusively; SSE events invalidate query keys rather than mutating state. Terminal data flows through WebSocket with a typed message bridge to the WebView. Theme loads synchronously from MMKV before the first frame. Multi-server support uses server-scoped query keys and cache clearing on server switch.

**Major components:**
1. **api-client** - Typed HTTP client with per-server base URL and auth header injection
2. **sse-client** - SSE subscription with reconnection; invalidates TanStack Query cache on events
3. **ws-client** - WebSocket for terminal PTY data; lifecycle tied to screen focus and AppState
4. **TerminalView** - Hybrid WebView rendering xterm.js with typed ToWebView/FromWebView message protocol
5. **Zustand stores** - server-store (multi-server, SecureStore-backed), theme-store (MMKV-backed), auth-store

See ARCHITECTURE.md for the full directory structure and code examples.

### Critical Pitfalls

1. **SSE blocked by Expo CDP interceptor** - SSE streams fail silently in dev mode. Use react-native-sse (XHR-based) and test in preview/production builds.
2. **Theme flash on launch** - Async storage causes a white frame. Use MMKV for synchronous theme loading before the first render.
3. **WebView terminal memory leaks** - xterm.js instances and WebSocket connections not cleaned up on unmount. Implement dispose lifecycle; cap scrollback at 10K lines.
4. **Push notifications need physical device + dev build** - Cannot test in Expo Go or simulator. Set up EAS Build pipeline in Phase 1.
5. **New Architecture library incompatibility** - All stack libraries verified, but any new additions must be checked before adoption.

See PITFALLS.md for the full list of 15 pitfalls with phase-specific warnings.

## Implications for Roadmap

Based on research, suggested phase structure:

### Phase 1: Foundation and Build Pipeline
**Rationale:** Everything depends on having a working dev build. MMKV, push notifications, and custom native modules all require EAS Build (no Expo Go). Theme system must load synchronously from the first frame.
**Delivers:** Expo project scaffold, EAS Build pipeline, dev build on physical device, navigation shell, Catppuccin Mocha theme with MMKV persistence, biometric lock gate.
**Addresses:** QR scanning, server connection, biometric auth, theme support (FEATURES table stakes)
**Avoids:** Theme flash (Pitfall 2), push notification blocking (Pitfall 4), New Architecture issues (Pitfall 5)

### Phase 2: Server Connection and Auth
**Rationale:** Nothing else works without server connection. QR pairing and manual URL entry are the first-run experience. Server-scoped query keys must be designed from the start for multi-server later.
**Delivers:** QR code scanning, manual URL entry, token storage in SecureStore, server-store with multi-server groundwork, api-client with typed responses.
**Addresses:** Server pairing, auth token storage (FEATURES critical path)
**Avoids:** Multi-server token confusion (Pitfall 6) by using server-scoped query keys from day one

### Phase 3: Core Screens (Sessions + Real-Time)
**Rationale:** Session list with live status is the core purpose of the app. SSE and API client patterns established here are reused by every subsequent phase.
**Delivers:** Session list (FlashList), session detail, start/stop/restart controls, SSE real-time updates, pull-to-refresh, workspace browsing.
**Addresses:** Session list, session controls, workspace browsing (FEATURES table stakes)
**Avoids:** SSE CDP interceptor bug (Pitfall 1) by testing in preview builds

### Phase 4: Terminal
**Rationale:** Terminal is the highest-complexity feature. Build it after core patterns are proven but before differentiators. Prototype in isolation first.
**Delivers:** Full terminal view with input, WebView bridge protocol, WebSocket management, keyboard avoidance, scrollback buffer management.
**Addresses:** Terminal view with input (FEATURES table stakes)
**Avoids:** WebView memory leaks (Pitfall 3), keyboard covering input (Pitfall 8)

### Phase 5: Push Notifications
**Rationale:** Push notifications are the primary reason to have a mobile app. Requires new server endpoint (POST /api/push/register). Dev build from Phase 1 enables testing.
**Delivers:** Push token registration, server-side notification dispatch, notification handling and deep-link navigation.
**Addresses:** Push notifications (FEATURES table stakes)
**Avoids:** Push testing issues (Pitfall 4, mitigated by Phase 1 dev build)

### Phase 6: Cost Dashboard and Charts
**Rationale:** Cost overview is table stakes; charts are a differentiator. Victory Native needs early validation against Reanimated 4. Group cost display and charts together since they share data.
**Delivers:** Cost summary, daily trend chart, model breakdown, workspace comparison.
**Addresses:** Cost overview (table stakes), cost dashboard with charts (differentiator)
**Avoids:** Victory/Reanimated compatibility issues (validate chart rendering early in phase)

### Phase 7: Multi-Server and Offline
**Rationale:** Multi-server touches api-client, store, and SSE layers. Easier to add after those layers are proven. Offline caching builds on TanStack Query's offline mode.
**Delivers:** Server switcher, per-server token storage, cache isolation, offline graceful degradation, reconnection queue.
**Addresses:** Multi-server support, offline caching (FEATURES differentiators)
**Avoids:** Multi-server token confusion (Pitfall 6)

### Phase 8: Kanban, Voice, and Polish
**Rationale:** Differentiator features that depend on all prior infrastructure. Kanban uses reanimated-dnd (MEDIUM confidence); prototype early and fall back to custom if needed.
**Delivers:** Kanban task board with drag-drop, voice input, image upload, share sheet, quick switcher, full 13-theme support, haptic feedback.
**Addresses:** All remaining differentiator features from FEATURES.md
**Avoids:** Drag-drop jank (Pitfall 9), theme testing gaps (Pitfall 13)

### Phase 9: Production Readiness
**Rationale:** OTA update configuration, app store metadata, full E2E test suite, performance profiling, and release builds.
**Delivers:** EAS Update with runtimeVersion policy, Maestro test suite across 3 themes, App Store and Play Store listings, production build.
**Avoids:** OTA breaking native code (Pitfall 11)

### Phase Ordering Rationale

- **Phases 1-2 first:** Build pipeline and server connection are the dependency foundation. Every subsequent feature needs a working dev build, API client, and auth flow.
- **Phase 3 before 4:** Session list proves the SSE + TanStack Query pattern. Terminal (Phase 4) is the hardest feature; building it after core patterns reduces risk.
- **Phase 4 isolated:** If terminal proves harder than expected, the app is still usable without it (session list + controls + notifications cover the MVP).
- **Phases 6-8 are parallel-safe:** Cost dashboard, multi-server, and kanban have no hard dependencies on each other. Could be reordered or built in parallel.
- **Phase 9 last:** Production readiness wraps everything up.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 4 (Terminal):** WebView bridge protocol, keyboard handling across iOS/Android, memory profiling strategy. Prototype in isolation.
- **Phase 8 (Kanban):** react-native-reanimated-dnd is v1.1.x (MEDIUM confidence). Need to validate or prepare custom fallback.
- **Phase 6 (Charts):** Victory Native 41.x compatibility with Reanimated 4 needs quick validation spike.

Phases with standard patterns (skip research-phase):
- **Phase 1 (Foundation):** Expo project setup, Zustand + MMKV, EAS Build. All well-documented.
- **Phase 2 (Connection):** expo-camera QR, SecureStore tokens. Straightforward implementation.
- **Phase 3 (Sessions):** TanStack Query + SSE invalidation is well-established. Existing web GUI is the reference.
- **Phase 5 (Push):** Expo push notification docs are comprehensive.
- **Phase 9 (Production):** EAS Build/Update docs are thorough.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All core libraries verified against Expo SDK 54 and New Architecture. Official changelogs and docs referenced. |
| Features | HIGH | Features derived from existing web GUI (proven spec) and design document with 95 mapped features. |
| Architecture | HIGH | Layered pattern is standard for RN apps with real-time data. TanStack Query + SSE invalidation is well-documented. |
| Pitfalls | HIGH | Top pitfalls confirmed via GitHub issues, official docs, and community consensus. |

**Overall confidence:** HIGH

### Gaps to Address

- **react-native-reanimated-dnd reliability:** MEDIUM confidence. Validate during Phase 8 planning with a prototype spike. Prepare custom Gesture Handler + Reanimated implementation as fallback.
- **react-native-sse in production at scale:** MEDIUM confidence. Monitor for edge cases (reconnection after long background, token refresh during active stream). May need a custom EventSource wrapper.
- **Victory Native + Reanimated 4 interop:** MEDIUM confidence. Test chart rendering early in Phase 6 before committing to Victory. react-native-skia is the fallback.
- **Server-side push endpoint:** The existing Myrlin server needs POST /api/push/register and push dispatch logic. This is new server work not covered by existing endpoints.
- **QR pairing endpoint:** The server needs a pairing endpoint that generates a QR payload (server URL + one-time token). This is new server work.
- **@react-native-voice/voice + New Architecture:** Not explicitly verified. Speech recognition is Phase 8; verify compatibility before that phase.
- **Android-specific testing:** Research focused on iOS-first. Android edge-to-edge (SDK 54 feature) and back button behavior need testing during Phase 3+.

## Sources

### Primary (HIGH confidence)
- [Expo SDK 54 Changelog](https://expo.dev/changelog/sdk-54) - framework, build pipeline, New Architecture
- [Reanimated 4 Stable Release](https://blog.swmansion.com/reanimated-4-stable-release-the-future-of-react-native-animations-ba68210c3713) - animation library
- [FlashList v2 Engineering Blog](https://shopify.engineering/flashlist-v2) - list performance
- [Expo Push Notifications Setup](https://docs.expo.dev/push-notifications/push-notifications-setup/) - push implementation
- [TanStack Query RN Docs](https://tanstack.com/query/latest/docs/framework/react/react-native) - server state
- [expo-router Docs](https://docs.expo.dev/router/basics/core-concepts/) - file-based routing
- [Expo SecureStore Docs](https://docs.expo.dev/versions/latest/sdk/securestore/) - encrypted storage
- [Maestro RN Docs](https://docs.maestro.dev/get-started/supported-platform/react-native) - E2E testing
- Design document: `docs/plans/2026-03-28-myrlin-mobile-design.md` (95 mapped features)

### Secondary (MEDIUM confidence)
- [react-native-reanimated-dnd](https://github.com/entropyconquers/react-native-reanimated-dnd) - Kanban drag-drop (v1.1.x)
- [react-native-sse](https://github.com/binaryminds/react-native-sse) - SSE client
- [Victory Native](https://nearform.com/open-source/victory-native/) - charting library
- [react-native-mmkv](https://github.com/mrousavy/react-native-mmkv) - synchronous storage
- [Expo SSE CDP bug](https://github.com/expo/expo/issues/27526) - confirmed dev mode issue

### Tertiary (LOW confidence)
- Community articles on WebView memory management in React Native (patterns consistent but no single authoritative source)

---
*Research completed: 2026-03-28*
*Ready for roadmap: yes*

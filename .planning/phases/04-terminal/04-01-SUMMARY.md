---
phase: 04-terminal
plan: 01
subsystem: terminal
tags: [xterm.js, webview, react-native, websocket, catppuccin, bridge-protocol]

requires:
  - phase: 03-sessions
    provides: Session data hooks, server store, navigation stack
provides:
  - Hybrid WebView terminal renderer with xterm.js (no CDN)
  - ToWebView/FromWebView typed bridge protocol
  - myrlinToXtermTheme converter for all 13 Catppuccin themes
  - Terminal route accessible from session detail screen
  - Activity detection from PTY output
affects: [04-terminal-input, 04-terminal-toolbar, 04-terminal-carousel]

tech-stack:
  added: [react-native-webview@13.16.0, expo-clipboard@55.0.10, react-native-keyboard-controller@1.20.7, "@xterm/xterm (inlined)", "@xterm/addon-fit (inlined)"]
  patterns: [hybrid-webview-bridge, postMessage-typed-protocol, asset-inlining-build-script]

key-files:
  created:
    - mobile/types/terminal.ts
    - mobile/assets/terminal.html
    - mobile/components/terminal/useTerminalBridge.ts
    - mobile/components/terminal/TerminalWebView.tsx
    - mobile/components/terminal/TerminalHeader.tsx
    - mobile/components/terminal/TerminalScreen.tsx
    - mobile/app/(tabs)/sessions/terminal.tsx
    - mobile/scripts/build-terminal-html.js
  modified:
    - mobile/app/(tabs)/sessions/_layout.tsx
    - mobile/app/(tabs)/sessions/[id].tsx
    - mobile/types/navigation.ts
    - mobile/package.json
    - mobile/package-lock.json

key-decisions:
  - "Build script generates terminal.html with inlined xterm.js (~492KB) to avoid CDN dependency for LAN-only setups"
  - "WebSocket lives inside WebView (not bridged through RN) for PTY data performance"
  - "Asset.fromModule loads terminal.html with proper local URI resolution across iOS/Android"
  - "Callbacks stored in refs in useTerminalBridge to avoid stale closures and unnecessary WebView re-renders"

patterns-established:
  - "Hybrid WebView bridge: typed ToWebView/FromWebView unions, injectJavaScript for RN->WebView, postMessage for WebView->RN"
  - "Build script pattern: node scripts/build-terminal-html.js generates assets from node_modules for bundling"
  - "AppState listener for WebSocket reconnection on foreground resume"

requirements-completed: [TERM-01, TERM-08, TERM-11]

duration: 6min
completed: 2026-03-29
---

# Phase 4 Plan 01: Hybrid WebView Terminal Summary

**xterm.js terminal renderer in WebView with typed bridge protocol, Catppuccin theme injection, and activity detection header**

## Performance

- **Duration:** 6 min
- **Started:** 2026-03-29T01:25:15Z
- **Completed:** 2026-03-29T01:31:40Z
- **Tasks:** 2
- **Files modified:** 13

## Accomplishments
- Self-contained terminal.html with inlined xterm.js v5 and addon-fit (no CDN dependency)
- Typed bridge protocol (ToWebView 14 message types, FromWebView 10 message types)
- TerminalWebView with WebSocket connect, theme injection, AppState reconnect, and dispose cleanup
- TerminalHeader with session name, StatusDot, animated activity indicator (Reanimated)
- Full terminal screen accessible from session detail via "Open Terminal" button

## Task Commits

Each task was committed atomically:

1. **Task 1: Bridge protocol types, xterm.js HTML asset, and theme converter** - `4e13dd5` (feat)
2. **Task 2: Terminal WebView component, header, screen, and route wiring** - `bf013ea` (feat)

## Files Created/Modified
- `mobile/types/terminal.ts` - ToWebView, FromWebView, XtermTheme types and myrlinToXtermTheme converter
- `mobile/assets/terminal.html` - Self-contained HTML with inlined xterm.js, fit addon, WS, bridge protocol
- `mobile/scripts/build-terminal-html.js` - Build script to regenerate terminal.html from node_modules
- `mobile/components/terminal/useTerminalBridge.ts` - Hook managing postMessage bridge (refs for stable callbacks)
- `mobile/components/terminal/TerminalWebView.tsx` - WebView wrapper with asset loading, connect, theme, AppState
- `mobile/components/terminal/TerminalHeader.tsx` - Header with status dot, activity text, back button
- `mobile/components/terminal/TerminalScreen.tsx` - Composites header + WebView + input placeholder
- `mobile/app/(tabs)/sessions/terminal.tsx` - Expo Router route screen
- `mobile/app/(tabs)/sessions/_layout.tsx` - Added terminal Stack.Screen entry
- `mobile/app/(tabs)/sessions/[id].tsx` - Added "Open Terminal" button for running/idle sessions
- `mobile/types/navigation.ts` - Added TerminalParams type and route entry

## Decisions Made
- Used build script to inline xterm.js into HTML rather than CDN (plan mandated, confirmed LAN-only use case)
- Used Asset.fromModule for HTML loading (most reliable cross-platform approach for expo-asset)
- Stored bridge callbacks in refs to prevent stale closures (standard pattern for WebView onMessage handlers)
- Skeleton loading fallback uses fixed dimensions (Skeleton component requires numeric height prop)

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 3 - Blocking] npm install with legacy-peer-deps flag**
- **Found during:** Task 1 (dependency installation)
- **Issue:** Peer dependency conflict between Expo SDK 55 canary and react-native-webview
- **Fix:** Installed with --legacy-peer-deps flag
- **Files modified:** package.json, package-lock.json
- **Verification:** All 3 packages installed successfully, tsc passes

**2. [Rule 1 - Bug] Skeleton height prop type mismatch**
- **Found during:** Task 2 (TerminalScreen loading state)
- **Issue:** Skeleton component height prop requires number, not string "100%"
- **Fix:** Changed to fixed numeric dimensions (300x400) for loading placeholder
- **Files modified:** mobile/components/terminal/TerminalScreen.tsx
- **Verification:** tsc --noEmit passes with zero errors

---

**Total deviations:** 2 auto-fixed (1 blocking, 1 bug)
**Impact on plan:** Both fixes necessary for build success. No scope creep.

## Issues Encountered
None beyond the auto-fixed deviations above.

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Terminal WebView renders xterm.js output with correct Catppuccin theme colors
- Plan 02 can add TerminalInput with KeyboardStickyView (placeholder is in place)
- Plan 02 can add TerminalToolbar below the WebView (layout slots are ready)
- Bridge protocol supports all message types needed for input, clipboard, and text extraction

---
*Phase: 04-terminal*
*Completed: 2026-03-29*

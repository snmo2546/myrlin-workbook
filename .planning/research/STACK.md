# Technology Stack

**Project:** Myrlin Mobile
**Researched:** 2026-03-28
**Mode:** Ecosystem research for greenfield React Native app connecting to existing Express backend

---

## Recommended Stack

### Core Framework

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Expo SDK 54 | ~54.x (stable) | Cross-platform mobile framework | Latest stable SDK (Sep 2025). New Architecture default. Last SDK with Legacy Architecture fallback, giving us an escape hatch if any dependency breaks. SDK 55 (RN 0.83) is still canary as of March 2026; not production-ready yet. | HIGH |
| React Native | 0.81 | UI runtime | Bundled with Expo SDK 54. New Architecture enabled by default. | HIGH |
| React | 19.x | Component framework | Bundled with RN 0.81 via Expo SDK 54. | HIGH |
| TypeScript | 5.x | Type safety | Expo default. Shared types with server contracts already defined in design doc. Non-negotiable for a project this size. | HIGH |

**Why not Expo SDK 55?** SDK 55 targets RN 0.83 and is only in canary/beta as of March 2026, with stable release expected mid-2026. Starting on 54 is safer. Upgrade to 55 once it stabilizes; the migration path is straightforward (Expo handles most of it).

**Why not Expo SDK 52?** SDK 52 ships RN 0.77, which is three minor versions behind. SDK 54 has precompiled iOS XCFrameworks (10x faster builds), iOS 26 Liquid Glass support, and Android 16 edge-to-edge. Worth the jump.

### Navigation

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| expo-router | ~4.x (bundled with SDK 54) | File-based routing | Built on React Navigation but with file-system routing (Next.js-style). Typed routes, deep linking, and lazy bundling out of the box. The design doc already specifies `/(tabs)/` layout. | HIGH |

### State Management

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Zustand | 5.0.x | Local UI state, preferences, theme | ~1KB gzipped. Hooks-based API. No boilerplate. Supports middleware (persist, devtools). Works synchronously, which matters for theme switching and UI responsiveness. Already specified in design doc. | HIGH |
| @tanstack/react-query | 5.95.x | Server data, caching, SSE sync | Handles cache invalidation, background refetching, optimistic updates. Built-in React Native support (online manager, focus refetch). The server has 60+ endpoints; managing this with raw fetch would be a maintenance nightmare. | HIGH |

**Why not Redux Toolkit?** Zustand + TanStack Query covers both local and server state with less boilerplate. Redux adds ceremony without benefit for this use case.

**Why not Jotai or Valtio?** Both are fine alternatives from the same ecosystem (pmndrs). Zustand is the most mature, has the best persistence middleware story, and the team already decided on it in the design doc.

### Storage

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| expo-secure-store | ~14.x (bundled) | Auth tokens, server configs | Keychain (iOS) / Keystore (Android) encryption. Required for storing bearer tokens and server connection secrets. | HIGH |
| react-native-mmkv | 3.x | Cached state, preferences, theme | 30x faster than AsyncStorage. Synchronous reads (critical for theme loading to avoid flash). Zustand persist middleware works with MMKV via a simple adapter. Works with Expo via dev builds (prebuild required). | HIGH |

**Why not @react-native-async-storage?** AsyncStorage is async-only, which causes theme flash on app launch (white frame before theme loads). MMKV reads are synchronous and 30x faster. The performance difference is noticeable on app startup.

**MMKV + Expo caveat:** MMKV requires a development build (no Expo Go). Since we need dev builds anyway for push notifications and WebView, this is not an additional constraint.

### Terminal Emulation

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| react-native-webview | 14.x | WebView container for terminal | The standard RN WebView. Mature, well-maintained, Expo-compatible. Hosts the xterm.js renderer. | HIGH |
| xterm.js | 5.x | Terminal rendering engine | Same library used by the existing web GUI. Renders ANSI escape codes, handles scrollback, Unicode. Battle-tested in VS Code's terminal. Running inside WebView means we get identical rendering to the web app. | HIGH |

**Hybrid model (already decided in design doc):** React Native owns all interaction (keyboard, toolbar, gestures). WebView is a dumb rendering surface that receives terminal output via `postMessage`. This gives us native keyboard behavior, share sheet integration, and haptics while keeping terminal rendering pixel-perfect.

**Why not a native terminal renderer?** No mature RN-native ANSI terminal renderer exists. Building one would be months of work for marginal benefit. The WebView approach is proven (apps like Termius, Blink Shell use similar patterns on iOS). The `@fressh/react-native-xtermjs-webview` package exists but is immature (v0.0.8); better to build our own thin bridge.

### Real-Time Communication

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| react-native-sse | 1.x | Server-Sent Events subscription | Pure JS implementation using XMLHttpRequest (no native code needed). Supports custom headers for auth tokens. The existing server already uses SSE for state sync. | MEDIUM |
| WebSocket (built-in) | RN built-in | Terminal data stream | React Native includes WebSocket support natively. The existing server uses WebSocket for PTY terminal I/O. No additional library needed. | HIGH |

**SSE caveat:** There is a known Expo bug where `ExpoRequestCdpInterceptor` breaks SSE in debug mode. Workaround: disable the CDP interceptor in dev, or use a production/preview build for testing SSE. This is a development annoyance, not a production blocker.

**Why react-native-sse over EventSource polyfill?** EventSource does not support custom headers (needed for Bearer auth). react-native-sse wraps XMLHttpRequest with proper header support. The existing web app already solved this same problem (EventSource with query param token fallback); react-native-sse is the cleaner approach.

### Animations and Gestures

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| react-native-reanimated | 4.3.x | 60fps native-thread animations | Reanimated 4 is the current stable (released March 2026). Requires New Architecture (which SDK 54 enables by default). All shared element transitions, skeleton loaders, and layout animations run on the UI thread. | HIGH |
| react-native-worklets | (peer dep) | Worklet runtime for Reanimated 4 | Separated from Reanimated in v4 for modularity. Must be installed alongside Reanimated. | HIGH |
| react-native-gesture-handler | 2.x | Touch gestures, swipe, long-press, drag | The foundation for all gesture-based interactions. Used by expo-router/React Navigation internally and by drag-drop libraries. | HIGH |

**Why Reanimated 4 over Reanimated 3?** Reanimated 4 is the biggest release since v2, specifically built for New Architecture. We are targeting New Architecture; no reason to use the older version.

### Lists and Performance

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| @shopify/flash-list | 2.x | High-performance scrolling lists | FlashList v2 is a ground-up rewrite for New Architecture. No item size estimates needed (v1 pain point eliminated). JS-only solution with efficient cell recycling. 2M+ monthly downloads. Sessions list, task list, search results all benefit. | HIGH |

**Why not FlatList?** FlatList is fine for short lists but drops frames on complex items (session cards with status dots, cost badges, action buttons). FlashList v2 maintains 60fps even with complex item components.

### Drag-and-Drop (Kanban)

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| react-native-reanimated-dnd | 1.1.x | Kanban board drag-drop | Built on Reanimated 4 + Gesture Handler. Smooth 60fps animations, sortable lists and grids, collision detection, full TypeScript support. Purpose-built for the modern RN stack we are already using. | MEDIUM |

**Why not react-native-dnd-board?** Uses Reanimated v1 (ancient). Not compatible with New Architecture.

**Why not building custom with Gesture Handler + Reanimated?** react-native-reanimated-dnd already solves the "drag items between columns" problem with collision detection. Building from scratch would take 2-3 days for something this library provides. If it proves insufficient, we can replace it with custom implementation since it uses the same underlying libraries.

**Confidence is MEDIUM** because react-native-reanimated-dnd is relatively new (v1.1.x). Validate during Phase 1 prototyping. Fallback: custom implementation with Gesture Handler + Reanimated directly.

### Charts and Data Visualization

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| victory-native | 41.x | Cost dashboard charts | Built on react-native-svg with optional Skia backend. Supports bar charts (daily cost), pie charts (model breakdown), line charts (cost trends). Active maintenance by NearForm (formerly Formidable). | MEDIUM |
| react-native-svg | 15.x | SVG rendering (victory dependency) | Required peer dependency for Victory Native. Also useful for custom icons and chart decorations. | HIGH |

**Why not react-native-skia directly?** Skia is the performance king but requires building chart primitives from scratch. Victory provides declarative chart components with animation. Our cost dashboard has 3-4 chart types; Victory handles this without custom drawing code.

**Why not react-native-chart-kit?** Unmaintained, limited chart types, poor customization. Victory is the standard for production React Native charting.

### Push Notifications

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| expo-notifications | ~0.29.x (bundled) | Push notification handling | Expo's built-in solution. Handles token registration, notification display, interaction handling. Integrates with Expo Push Service (free tier). | HIGH |

**Key constraints (SDK 54):**
- Push notifications no longer work in Expo Go; development build required
- Physical device required for testing (no simulator support)
- Android requires `google-services.json` + FCM v1 Service Account Key
- iOS requires Apple Push Notification service (APNs) configuration via EAS

**Server integration:** New endpoint `POST /api/push/register` sends device token to Myrlin server. Server uses Expo Push API to deliver notifications when sessions complete, error, or need attention.

### Camera and QR Scanning

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| expo-camera | ~16.x (bundled) | QR code scanning for server pairing | expo-barcode-scanner was deprecated in SDK 51. expo-camera now includes built-in barcode/QR scanning via `CameraView`. Single package for both camera and scanning. | HIGH |

**Why not expo-barcode-scanner?** Removed from SDK 51+. expo-camera is the official replacement with identical QR scanning capability.

### Platform APIs

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| expo-haptics | ~13.x (bundled) | Tactile feedback | Bundled with Expo. Trigger haptics on drag-drop, session actions, notifications. | HIGH |
| expo-local-authentication | ~14.x (bundled) | Biometric auth (Face ID, fingerprint) | Bundled with Expo. App lock feature. Falls back to device passcode. | HIGH |
| expo-image-picker | ~16.x (bundled) | Photo/camera upload to sessions | Bundled with Expo. Camera and gallery access for uploading images to terminal sessions. | HIGH |
| expo-speech | ~13.x (bundled) | Text-to-speech | Bundled with Expo. Voice output for session summaries. | HIGH |
| @react-native-voice/voice | 3.x | Speech recognition input | The standard speech-to-text library for RN. Used for voice input to terminal. | MEDIUM |
| react-native-share | 2.x | Native share sheet | Share session output, cost reports, screenshots via iOS/Android share sheet. | HIGH |

### Fonts and Typography

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| expo-font | ~13.x (bundled) | Custom font loading | Loads Plus Jakarta Sans and JetBrains Mono at app startup. | HIGH |
| @expo-google-fonts/plus-jakarta-sans | 0.x | Plus Jakarta Sans font files | Google Fonts package for Expo. All weights (400, 500, 600, 700). | HIGH |

**JetBrains Mono:** Not available via expo-google-fonts. Bundle the .ttf files in `assets/fonts/` and load via `expo-font`. Download from JetBrains official site.

### Testing

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Maestro | latest (CLI) | Visual E2E testing | YAML-based, runs on iOS Simulator/Android Emulator, takes real screenshots. Zero instrumentation needed. MCP support for AI assistant integration. The design doc mandates Maestro for every screen. | HIGH |
| Jest | 29.x | Unit and integration tests | Bundled with Expo. Test hooks, stores, API client logic. | HIGH |
| React Native Testing Library | 12.x | Component tests | Test component rendering and interactions without a device. | HIGH |

**Why not Detox?** Detox requires more setup, has a steeper learning curve, and is slower. Maestro's YAML flows are agent-writable (important for our Claude Code build process). Maestro also supports MCP for AI-assisted test creation.

**Storybook:** For component development and visual review. Use `@storybook/react-native` 8.x. Useful during development but not a testing tool per se.

### Dev Tooling

| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| EAS Build | (Expo service) | Cloud builds for iOS/Android | Handles code signing, provisioning, and build distribution. Required for TestFlight and Play Store. | HIGH |
| EAS Update | (Expo service) | OTA updates | Push JS bundle updates without app store review. Critical for rapid iteration. | HIGH |
| expo-dev-client | ~4.x (bundled) | Development builds | Required for push notifications, MMKV, and custom native modules. Replaces Expo Go for this project. | HIGH |
| Biome | 2.x | Lint + format | Faster than ESLint + Prettier combined. Single tool for both. Modern, actively developed. | MEDIUM |

**Why Biome over ESLint?** Biome is 10-100x faster and handles both linting and formatting. The React Native ecosystem has largely adopted it in 2025-2026. If the team prefers ESLint, that works too; this is a soft recommendation.

---

## Alternatives Considered

| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| Framework | Expo SDK 54 | Expo SDK 55 | Still in canary/beta. Not production-ready as of March 2026. |
| Framework | Expo SDK 54 | Bare React Native | Lose EAS builds, OTA updates, managed native modules. No benefit for this project. |
| State (local) | Zustand 5 | Redux Toolkit | More boilerplate, heavier. Zustand covers our needs in ~1KB. |
| State (server) | TanStack Query 5 | SWR | TanStack Query has better mutation support, cache invalidation, and RN-specific hooks. |
| Storage | MMKV | AsyncStorage | 30x slower, async-only causes theme flash on launch. |
| Lists | FlashList v2 | FlatList | FlatList drops frames on complex items. FlashList v2 is strictly better. |
| Charts | Victory Native | react-native-skia raw | Victory provides chart components; Skia requires building from scratch. |
| Charts | Victory Native | react-native-chart-kit | Unmaintained, limited types, poor customization. |
| DnD | reanimated-dnd | react-native-dnd-board | Uses Reanimated v1. Incompatible with New Architecture. |
| Terminal | WebView + xterm.js | Native terminal view | No mature native ANSI renderer for RN. WebView is proven. |
| Navigation | expo-router | React Navigation (direct) | expo-router builds on React Navigation but adds file-based routing, typed routes, deep linking. |
| Testing | Maestro | Detox | Maestro is simpler (YAML), faster to write, agent-compatible. |
| Linting | Biome | ESLint + Prettier | Biome is faster. Either works; soft preference. |

---

## Installation

```bash
# Initialize Expo project
npx create-expo-app@latest myrlin-mobile --template tabs
cd myrlin-mobile

# Core state
npx expo install zustand @tanstack/react-query

# Storage
npx expo install expo-secure-store react-native-mmkv

# Animation and gestures
npx expo install react-native-reanimated react-native-worklets react-native-gesture-handler

# Lists
npx expo install @shopify/flash-list

# Terminal
npx expo install react-native-webview

# Charts
npx expo install victory-native react-native-svg

# Camera and platform APIs
npx expo install expo-camera expo-haptics expo-local-authentication expo-image-picker expo-speech expo-notifications

# Real-time
npm install react-native-sse

# Drag-and-drop
npm install react-native-reanimated-dnd

# Voice and sharing
npm install @react-native-voice/voice react-native-share

# Fonts
npx expo install expo-font @expo-google-fonts/plus-jakarta-sans

# Dev dependencies
npm install -D @storybook/react-native jest @testing-library/react-native typescript
npm install -D @biomejs/biome

# Create development build (required for push notifications, MMKV)
npx expo prebuild
eas build --profile development --platform ios
```

---

## Version Compatibility Matrix

| Library | Requires New Architecture | Expo SDK 54 Compatible | Notes |
|---------|--------------------------|------------------------|-------|
| Reanimated 4.x | Yes | Yes | SDK 54 enables New Arch by default |
| FlashList 2.x | Yes | Yes | JS-only, no native deps |
| MMKV 3.x | No | Yes (with prebuild) | Works on both architectures |
| react-native-sse | No | Yes | Pure JS |
| Victory Native 41.x | No | Yes | SVG-based |
| react-native-reanimated-dnd | Yes (via Reanimated 4) | Yes | Built for modern stack |
| expo-camera (QR) | No | Yes | Bundled with SDK |
| react-native-webview | No | Yes | Supports both architectures |

---

## Sources

- [Expo SDK 54 Changelog](https://expo.dev/changelog/sdk-54) - HIGH confidence
- [Expo SDK 55 Changelog](https://expo.dev/changelog/sdk-55) - HIGH confidence (confirmed still canary)
- [Reanimated 4 Stable Release blog](https://blog.swmansion.com/reanimated-4-stable-release-the-future-of-react-native-animations-ba68210c3713) - HIGH confidence
- [FlashList v2 Engineering blog](https://shopify.engineering/flashlist-v2) - HIGH confidence
- [react-native-reanimated-dnd GitHub](https://github.com/entropyconquers/react-native-reanimated-dnd) - MEDIUM confidence (new library)
- [react-native-mmkv GitHub](https://github.com/mrousavy/react-native-mmkv) - HIGH confidence
- [react-native-sse GitHub](https://github.com/binaryminds/react-native-sse) - MEDIUM confidence
- [Expo SSE bug discussion](https://github.com/expo/expo/issues/27526) - HIGH confidence (confirmed issue)
- [expo-barcode-scanner deprecation](https://github.com/expo/expo/issues/27015) - HIGH confidence
- [TanStack Query RN docs](https://tanstack.com/query/latest/docs/framework/react/react-native) - HIGH confidence
- [Zustand npm](https://www.npmjs.com/package/zustand) - HIGH confidence
- [Victory Native](https://nearform.com/open-source/victory-native/) - MEDIUM confidence
- [Maestro docs](https://docs.maestro.dev/get-started/supported-platform/react-native) - HIGH confidence
- [Expo push notifications setup](https://docs.expo.dev/push-notifications/push-notifications-setup/) - HIGH confidence

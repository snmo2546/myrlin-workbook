# Domain Pitfalls

**Domain:** React Native (Expo) mobile app with real-time terminal, SSE, WebSocket, push notifications
**Researched:** 2026-03-28

---

## Critical Pitfalls

Mistakes that cause rewrites or major delays.

### Pitfall 1: SSE Blocked by Expo CDP Interceptor in Development

**What goes wrong:** Server-Sent Events stop working in development builds. The SSE connection opens but no events are delivered.
**Why it happens:** Expo's `ExpoRequestCdpInterceptor` tries to `peekBody()` on SSE streams, but SSE bodies are infinite streams that cannot be peeked. This blocks the stream entirely.
**Consequences:** Real-time updates are dead in development. Developers think SSE is broken and switch to polling.
**Prevention:** Use `react-native-sse` (XMLHttpRequest-based, avoids the interceptor in most cases). Test SSE in production/preview builds, not debug builds. File and track this Expo issue (github.com/expo/expo/issues/27526).
**Detection:** SSE connection opens (200 status) but `onmessage` never fires in dev mode. Works fine in production builds.
**Confidence:** HIGH (confirmed Expo issue with GitHub issue tracker reference)

### Pitfall 2: Theme Flash on App Launch

**What goes wrong:** App shows a white/default-themed frame for 100-300ms before the user's selected theme loads.
**Why it happens:** Using AsyncStorage (async) for theme persistence. The first render happens before the async read completes.
**Consequences:** Jarring visual glitch on every app launch. Feels unpolished.
**Prevention:** Use MMKV (synchronous reads) for theme storage. Initialize the Zustand theme store synchronously from MMKV before the first render. Never use AsyncStorage for theme data.
**Detection:** Launch app repeatedly; watch for color flash in the first frame.
**Confidence:** HIGH (well-documented pattern in RN community)

### Pitfall 3: WebView Terminal Memory Leaks

**What goes wrong:** Opening and closing terminal screens repeatedly causes memory to grow unboundedly. Eventually the app crashes or becomes unresponsive.
**Why it happens:** xterm.js instances inside WebView are not properly disposed. WebSocket connections are not closed when the screen unmounts. The WebView retains references to detached terminal buffers.
**Consequences:** App crash after extended use. Users lose trust.
**Prevention:** Implement proper cleanup in the terminal screen's `useEffect` cleanup function: close WebSocket, send `dispose` message to WebView, set a max scrollback buffer size (e.g., 10,000 lines), and force WebView unmount on screen blur.
**Detection:** Monitor memory usage (Xcode Instruments / Android Profiler) while opening and closing terminal screens 20+ times.
**Confidence:** HIGH (common WebView issue in RN)

### Pitfall 4: Push Notifications Not Testable in Expo Go or Simulator

**What goes wrong:** Developer sets up push notifications, they do not work. Days wasted debugging.
**Why it happens:** Starting with SDK 53, push notifications are not supported in Expo Go. They also do not work on iOS Simulator or Android Emulator. Physical device + development build required.
**Consequences:** Blocked development if the team does not have the build pipeline set up early.
**Prevention:** Set up EAS Build and create a development build in Phase 1, before any push notification work begins. Test on a physical iPhone connected to the dev machine.
**Detection:** `Notifications.getExpoPushTokenAsync()` throws an error or returns undefined.
**Confidence:** HIGH (official Expo documentation)

### Pitfall 5: New Architecture Incompatibility with Third-Party Libraries

**What goes wrong:** A critical library crashes at runtime with obscure native errors. Bridging layer panics.
**Why it happens:** SDK 54 enables New Architecture by default. Some older RN libraries have not been updated for the new bridging layer (TurboModules / Fabric).
**Consequences:** Blocked on a dependency. Must either find an alternative, fork and fix, or disable New Architecture (losing Reanimated 4 and FlashList v2).
**Prevention:** Before adopting any library, check its New Architecture compatibility. Prefer libraries with explicit "New Architecture" support. The libraries in this stack have all been verified. For any additions, check the [React Native Directory](https://reactnative.directory/) "New Architecture" filter.
**Detection:** Native crash on app launch or when the component mounts. Error messages mention "bridge" or "turbomodule" or "fabric".
**Confidence:** HIGH (common in the 2025-2026 RN transition period)

---

## Moderate Pitfalls

### Pitfall 6: Multi-Server Token Confusion

**What goes wrong:** API requests go to the wrong server, or use the wrong auth token, after switching servers.
**Prevention:** Clear TanStack Query cache on server switch. Use a server-scoped query key prefix (e.g., `[serverId, 'sessions']`). Validate that the api-client instance matches the active server before every request.

### Pitfall 7: WebSocket Reconnection After Background/Foreground

**What goes wrong:** Terminal disconnects when the app goes to background. On foreground resume, the WebSocket is dead but the UI does not reflect it.
**Prevention:** Use React Native's `AppState` listener. On `active` state transition, check WebSocket readyState. If closed, reconnect and re-attach to the same PTY session. Show a "Reconnecting..." indicator.

### Pitfall 8: Keyboard Covering Terminal Input

**What goes wrong:** On smaller iPhones (SE, Mini), the keyboard covers the terminal input field. Users cannot see what they are typing.
**Prevention:** Use `KeyboardAvoidingView` with `behavior="padding"` (iOS) or `behavior="height"` (Android). Resize the WebView terminal when the keyboard opens (send `resize` message with reduced rows). Test on the smallest supported device.

### Pitfall 9: Drag-Drop Jank on Low-End Devices

**What goes wrong:** Kanban drag-drop stutters or drops below 60fps on older Android devices.
**Prevention:** Keep drag preview components simple (no shadows, no blur during drag). Use Reanimated's `useAnimatedStyle` for all drag transforms (runs on UI thread). Limit the number of visible columns to 3-4. Profile with React Native Perf Monitor.

### Pitfall 10: QR Code Scanning Reliability

**What goes wrong:** QR scanning is slow, unreliable, or does not work in low-light conditions.
**Prevention:** Use expo-camera's built-in barcode scanning with `barcodeScannerSettings={{ barcodeTypes: ['qr'] }}`. Add manual URL entry as a fallback (always). Show a clear viewfinder overlay. Test with printed QR codes and screen-displayed QR codes (brightness differences).

### Pitfall 11: Expo OTA Update Breaking Native Code

**What goes wrong:** An OTA update (via EAS Update) ships JS that expects a native module not present in the current binary.
**Prevention:** Use `runtimeVersion` policy in app.json to tie OTA updates to specific native builds. Never add a new native dependency and push it via OTA; native changes require a full EAS Build. Test OTA updates in staging before production.

---

## Minor Pitfalls

### Pitfall 12: Font Loading FOUT (Flash of Unstyled Text)

**What goes wrong:** Text renders in system font for 50-200ms before custom fonts load.
**Prevention:** Use `expo-splash-screen` to keep the splash visible until fonts are loaded (`SplashScreen.preventAutoHideAsync()` then `hideAsync()` after `Font.loadAsync()` completes).

### Pitfall 13: 13 Themes = 13x Testing Surface

**What goes wrong:** UI bugs only visible in certain themes (light theme contrast issues, dark theme invisible borders).
**Prevention:** Run Maestro visual tests with at least 3 themes (Mocha/dark, Latte/light, one mid-tone). Use semantic color tokens (not raw hex) everywhere.

### Pitfall 14: Android Back Button Behavior

**What goes wrong:** Android hardware back button exits the app instead of navigating back, or gets stuck in modal loops.
**Prevention:** expo-router handles basic back navigation. For modals, ensure `dismiss()` is called on back press. Test the back button on every screen during Maestro flows.

### Pitfall 15: Large Session List Performance

**What goes wrong:** Users with 100+ sessions experience slow initial load or scroll jank.
**Prevention:** FlashList v2 handles this well, but also implement server-side pagination (the existing API may need a `limit` and `offset` parameter). Show skeleton loaders during pagination fetches.

---

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| Project scaffolding | New Architecture + MMKV + dev build setup takes longer than expected | Budget 1-2 days for build pipeline setup alone |
| Terminal (WebView) | Memory leaks, keyboard interaction, resize bugs | Prototype terminal in isolation first; do not integrate until stable |
| SSE integration | CDP interceptor blocks SSE in dev mode | Test in preview/production builds; do not debug SSE issues in dev mode |
| Push notifications | Cannot test without physical device + dev build | Set up EAS Build early; have a physical iPhone available from day 1 |
| Kanban drag-drop | reanimated-dnd library is young (v1.1.x) | Prototype kanban early in isolation; be ready to build custom if library fails |
| Charts | Victory Native version compatibility with Reanimated 4 | Test chart rendering in the first week; have react-native-skia as fallback |
| Multi-server | Token/cache confusion when switching | Use server-scoped query keys from the start; never assume single server |
| OTA updates | JS/native version mismatch | Configure runtimeVersion policy before first production release |

---

## Sources

- [Expo SSE CDP Interceptor bug](https://github.com/expo/expo/issues/27526) - HIGH confidence
- [react-native-mmkv GitHub](https://github.com/mrousavy/react-native-mmkv) - HIGH confidence (sync read advantage)
- [Expo push notifications setup](https://docs.expo.dev/push-notifications/push-notifications-setup/) - HIGH confidence
- [Expo SDK 54 changelog](https://expo.dev/changelog/sdk-54) - HIGH confidence
- [FlashList v2 engineering blog](https://shopify.engineering/flashlist-v2) - HIGH confidence
- [react-native-reanimated-dnd](https://github.com/entropyconquers/react-native-reanimated-dnd) - MEDIUM confidence
- Community articles on WebView memory management, keyboard handling, and theme loading patterns

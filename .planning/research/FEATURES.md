# Feature Landscape

**Domain:** Mobile session manager for Claude Code (React Native client for existing Express backend)
**Researched:** 2026-03-28

---

## Table Stakes

Features users expect from a mobile companion to a desktop tool. Missing = "why bother installing?"

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Session list with status indicators | Core purpose of the app; must show running/stopped/error at a glance | Low | FlashList, SSE real-time updates |
| Start/stop/restart sessions | Basic session control from phone | Low | REST API calls, optimistic updates |
| Terminal view (read-only at minimum) | Users need to see what Claude is doing | High | Hybrid WebView + xterm.js approach |
| Terminal input (type commands) | Read-only terminal is frustrating; basic input is expected | High | Native TextInput bridged to WebSocket |
| Push notifications (session complete/error) | The primary reason to have a mobile app | Medium | expo-notifications + new server endpoint |
| Server connection (QR pairing) | First-run experience; must be frictionless | Medium | expo-camera QR scan, new server pairing endpoint |
| Workspace browsing | Users organize work in workspaces on desktop; must see them on mobile | Low | REST API, list views |
| Cost overview | Users want to check spend without opening laptop | Medium | REST API, simple number display |
| Theme support (at least Catppuccin Mocha) | App must not look jarring next to the desktop UI | Medium | Zustand + MMKV persistence, CSS var mapping |
| Pull-to-refresh | Mobile convention; absence feels broken | Low | TanStack Query refetch on pull |
| Biometric app lock | Sensitive data (terminal output, API tokens) | Low | expo-local-authentication, single gate |

## Differentiators

Features that make Myrlin Mobile more than "the web GUI in a native shell."

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Multi-server support | Connect phone to home PC, work machine, cloud server | Medium | Server switcher, per-server token storage |
| Kanban task board with drag-drop | Visual task management that feels native, not a web wrapper | High | react-native-reanimated-dnd, column-to-column drag |
| Voice input to terminal | Dictate commands while walking; hands-free monitoring | Medium | @react-native-voice/voice, native iOS dictation |
| Image upload to sessions | Take photo of whiteboard, send to Claude session | Low | expo-image-picker, WebSocket message |
| Share sheet integration | Share terminal output, cost reports to Slack/Messages/email | Low | react-native-share |
| Haptic feedback on actions | Tactile confirmation on session start/stop, drag-drop | Low | expo-haptics |
| Offline graceful degradation | Cached state visible when disconnected, reconnection queue | Medium | MMKV cache, TanStack Query offline mode |
| Quick switcher (command palette) | Power user feature; swipe-down to search and jump anywhere | Medium | Modal with search, keyboard-aware |
| Full 13-theme support | Match exact desktop experience across all Catppuccin variants | Medium | React Context + Zustand, 13 theme objects |
| Cost dashboard with charts | Visual spend analysis (daily trend, model breakdown, workspace comparison) | Medium | Victory Native charts |

## Anti-Features

Features to explicitly NOT build in v1.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| myrlin.io relay service | Requires separate backend infrastructure, paid tier scope | Direct LAN/Tailscale/Cloudflare connection only |
| Apple Watch companion | Unnecessary complexity; notifications already reach Watch via iOS | Defer to v2+ if there is demand |
| Widgets / Live Activities | iOS 16+ widgets need a separate target and entitlements | Defer to v2; push notifications cover the core need |
| Offline terminal (local Claude) | Impossible; terminal requires server connection | Show clear "disconnected" state with reconnect button |
| Multi-user team sync | Requires coordination server, real-time rooms | Single-user only; team features are paid tier |
| In-app code editor | Scope creep; this is a session manager, not an IDE | Terminal input covers the command interface |
| Desktop (Electron) app | Mobile-only scope; desktop already has the web GUI | Web GUI is the desktop experience |
| Custom notification sounds | Low value, high annoyance potential | Use system defaults |

## Feature Dependencies

```
QR Scanning -> Server Connection -> Everything else
Server Connection -> Auth Token Storage -> All API features
Push Token Registration -> Push Notifications
Biometric Auth -> App Lock (gates all screens)
SSE Subscription -> Real-time status updates -> Badge counts
WebSocket Connection -> Terminal View -> Terminal Input
Terminal View -> Voice Input, Image Upload, Share Sheet
Zustand Theme Store -> All UI rendering (must load before first frame)
TanStack Query -> All server data screens
FlashList -> Session List, Task List, Search Results
```

## MVP Recommendation

**Phase 1 - Must have (app is usable):**
1. Server pairing (QR + manual URL)
2. Session list with real-time status (SSE)
3. Session controls (start/stop/restart)
4. Terminal view with input
5. Push notifications (session events)
6. Biometric app lock
7. 1 theme (Catppuccin Mocha)

**Phase 2 - Expected (app feels complete):**
1. Full 13-theme support
2. Cost dashboard with charts
3. Workspace management
4. Multi-server support
5. Offline caching

**Phase 3 - Differentiators (app stands out):**
1. Kanban task board with drag-drop
2. Voice input
3. Image upload
4. Quick switcher
5. Share sheet integration

**Defer:** Relay service, Watch app, Widgets, Live Activities

## Sources

- Design document: `docs/plans/2026-03-28-myrlin-mobile-design.md` (95 mapped features)
- PROJECT.md (validated requirements and constraints)
- Existing web GUI feature set (serves as the spec for parity)

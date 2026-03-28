# Myrlin Mobile

## What This Is

A React Native (Expo) mobile app that provides full control over Myrlin's Workbook servers from iOS and Android. Connects to self-hosted Myrlin servers via QR code pairing or manual URL entry. Every feature available on the web GUI (100+ features) is accessible on mobile with native platform interactions (share sheet, haptics, push notifications, biometric auth, camera).

## Core Value

A Myrlin user can monitor, control, and interact with all their Claude Code sessions from their phone with the same capability as the desktop web interface, plus native mobile advantages (push notifications, QR pairing, haptics, camera, share sheet).

## Requirements

### Validated

- ✓ Web GUI feature set (100+ features) - existing, serves as the spec
- ✓ Express REST API (60+ endpoints) - existing backend the mobile app connects to
- ✓ SSE real-time event system - existing, mobile subscribes to same stream
- ✓ WebSocket terminal protocol - existing, mobile terminal connects via same WS
- ✓ Bearer token authentication - existing auth system
- ✓ 13 theme system (Catppuccin + community) - existing, port to React Native
- ✓ Cost tracking with JSONL parsing - existing server-side calculation

### Active

- [ ] Full feature parity mobile app (95 mapped features)
- [ ] Hybrid terminal (WebView xterm.js renderer + native interaction chrome)
- [ ] QR code server pairing (new server endpoint + mobile camera)
- [ ] Push notifications (new server push endpoints + Expo notifications)
- [ ] Multi-server support (connect phone to multiple Myrlin servers)
- [ ] Biometric app lock (Face ID / fingerprint)
- [ ] Native share sheet integration
- [ ] Voice input for terminal (iOS dictation + speech recognition)
- [ ] Image upload from camera/gallery to terminal sessions
- [ ] Offline graceful degradation (cached state, reconnection queue)
- [ ] Maestro visual test suite for every screen

### Out of Scope

- myrlin.io relay service - Phase 2+ paid feature, requires separate backend infrastructure
- Multi-user team sync - requires coordination server, deferred to paid tier
- Desktop app (Electron) - mobile only for now
- Apple Watch companion - unnecessary complexity for v1
- Widget / Live Activities - nice-to-have for v2, not v1
- Offline terminal (local Claude) - not possible, terminal requires server connection

## Context

### Existing Codebase (Brownfield)

Myrlin's Workbook v0.9.9 is a mature Node.js application:

- **Backend:** Express server with 60+ REST endpoints, WebSocket PTY management, SSE event streaming
- **Frontend:** Vanilla JS SPA (16,000+ lines), xterm.js terminals, 4100 lines CSS
- **State:** JSON persistence in `~/.myrlin/`, atomic writes with backup/recovery
- **Auth:** Bearer tokens, password-based, one-time startup tokens
- **Cost:** Worker thread JSONL parsing, per-model pricing
- **Themes:** 13 themes (4 Catppuccin official + 9 community), CSS custom properties

The mobile app is a NEW client connecting to the EXISTING server. No server rewrite needed, only new endpoints for QR pairing and push notifications.

### Design Document

Full design with all contracts, types, component specs, and screen specifications:
`docs/plans/2026-03-28-myrlin-mobile-design.md`

### Development Environment

- **Build machine:** Mac Mini (Tailscale: 100.111.181.106)
- **Xcode:** Required for iOS Simulator and builds
- **Apple Developer:** Active account (Shuttle project on TestFlight Build 8)
- **Testing:** Maestro on iOS Simulator, Storybook for components, Expo Go on physical iPhone

### Target Platforms

- **iOS first** (iPhone, iOS 16+)
- **Android within 2 weeks** of iOS launch (Android 13+)
- **Cross-platform:** Single React Native codebase via Expo

## Constraints

- **Framework:** React Native (Expo SDK 52+), must use expo-router for navigation
- **Agent model:** ALL build agents must use Opus with deep thinking, no exceptions
- **Orchestration:** Two-tier only (orchestrator -> workers), no manager middle layer
- **Terminal:** Hybrid WebView approach (xterm.js renderer + native chrome), pluggable interface
- **Modularity:** Every screen, component, and service must be independently swappable
- **Testing:** Maestro flow tests required for every screen, Storybook stories for every shared component
- **Font:** Plus Jakarta Sans (sans), JetBrains Mono (mono), matching web app
- **Theme:** All 13 themes must render identically to web CSS
- **Monorepo:** Source lives in `mobile/` directory within myrlin-workbook repo
- **No em dashes:** NEVER use em dashes or double hyphens in any output

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| React Native (Expo) over Flutter | JS/TS ecosystem matches existing codebase, Expo handles build tooling, OTA updates, largest component ecosystem | - Pending |
| Hybrid terminal (WebView + native) | xterm.js is battle-tested for ANSI rendering; native chrome gives iOS share sheet, haptics, keyboard | - Pending |
| Monorepo over separate repo | Shared types, single git history, easier API contract sync | - Pending |
| QR pairing first, relay later | Free tier needs no external service; relay requires myrlin.io infrastructure | - Pending |
| Zustand + TanStack Query | Zustand for local UI state, TanStack Query for server data with SSE cache sync | - Pending |
| Maestro for visual QA | YAML-based, runs on iOS Simulator, takes real screenshots, agent-writable | - Pending |
| Bottom tabs (5) over drawer | iOS convention, familiar navigation, badges for running counts | - Pending |
| Two-tier orchestration | Manager middle layer degrades context fidelity; direct briefing preserves contracts | - Pending |

---
*Last updated: 2026-03-28 after initialization*

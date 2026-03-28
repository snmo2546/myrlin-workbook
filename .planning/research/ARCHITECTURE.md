# Architecture Patterns

**Domain:** React Native mobile client for existing Express backend
**Researched:** 2026-03-28

---

## Recommended Architecture

### High-Level Layers

```
+----------------------------------------------------------+
|  Screen Layer (expo-router file-based screens)            |
|  /(tabs)/sessions, /(tabs)/tasks, /(tabs)/costs, etc.    |
+----------------------------------------------------------+
|  Component Layer (shared UI primitives + feature comps)   |
|  SessionCard, TerminalView, KanbanBoard, CostChart, etc. |
+----------------------------------------------------------+
|  Hook Layer (screen-specific data wiring)                 |
|  useSessions(), useTerminal(), useCosts(), useSSE()       |
+----------------------------------------------------------+
|  State Layer                                              |
|  Zustand stores (UI) | TanStack Query (server cache)     |
+----------------------------------------------------------+
|  Service Layer (pure functions, no React)                 |
|  api-client, sse-client, ws-client, push-service          |
+----------------------------------------------------------+
|  Platform Layer (device APIs)                             |
|  SecureStore, MMKV, Camera, Haptics, Biometrics           |
+----------------------------------------------------------+
```

### Component Boundaries

| Component | Responsibility | Communicates With |
|-----------|---------------|-------------------|
| `api-client` | HTTP requests to Myrlin server, typed responses | TanStack Query (via query functions) |
| `sse-client` | SSE subscription, event parsing, reconnection | TanStack Query (cache invalidation), Zustand (connection status) |
| `ws-client` | WebSocket for terminal PTY I/O | TerminalView component (via bridge) |
| `push-service` | Token registration, notification handling | expo-notifications, api-client |
| `server-store` (Zustand) | Active server, server list, connection state | SecureStore (persistence), api-client (base URL) |
| `theme-store` (Zustand) | Active theme, theme objects, font config | MMKV (persistence), all UI components |
| `auth-store` (Zustand) | Auth state, biometric lock status | SecureStore (tokens), api-client (headers) |
| `TerminalView` | Hybrid WebView terminal rendering | ws-client (data), WebView (rendering) |

### Data Flow

**Server data lifecycle:**
1. Screen mounts, TanStack Query fires query function
2. Query function calls api-client with active server URL + auth token
3. Response cached in TanStack Query
4. SSE client receives real-time update, invalidates relevant query key
5. TanStack Query refetches stale data in background
6. Screen re-renders with fresh data

**Terminal data flow:**
1. User opens terminal screen for session X
2. ws-client opens WebSocket to `ws://{server}/ws/pty/{sessionId}`
3. Server streams PTY output over WebSocket
4. ws-client forwards data to TerminalView via `postMessage({ type: 'write', data })`
5. xterm.js (in WebView) renders ANSI output
6. User types in native TextInput
7. Input sent via ws-client to server WebSocket
8. Server writes to PTY stdin

**Theme loading (critical path):**
1. App launches, MMKV reads theme synchronously (no flash)
2. Zustand theme store initialized with persisted theme
3. ThemeProvider wraps entire app tree with theme context
4. All components read colors from `useTheme()` hook

---

## Patterns to Follow

### Pattern 1: Typed API Client with TanStack Query

**What:** A single api-client module that handles base URL resolution, auth headers, and typed responses. TanStack Query wraps every API call.

**When:** Every server data request.

**Example:**

```typescript
// services/api-client.ts
class MyrlinAPIClient {
  constructor(private baseUrl: string, private token: string) {}

  async getSessions(): Promise<{ sessions: Session[] }> {
    const res = await fetch(`${this.baseUrl}/api/sessions`, {
      headers: { Authorization: `Bearer ${this.token}` },
    });
    if (!res.ok) throw new APIError(res.status, await res.text());
    return res.json();
  }
}

// hooks/useSessions.ts
function useSessions() {
  const client = useAPIClient();
  return useQuery({
    queryKey: ['sessions'],
    queryFn: () => client.getSessions(),
    staleTime: 5000, // SSE will invalidate when needed
  });
}
```

### Pattern 2: SSE-Driven Cache Invalidation

**What:** SSE events invalidate specific TanStack Query keys rather than replacing data directly. This ensures the query cache is the single source of truth for server data.

**When:** Any real-time update from the server.

**Example:**

```typescript
// services/sse-client.ts
function handleSSEEvent(event: SSEEvent, queryClient: QueryClient) {
  switch (event.type) {
    case 'session:updated':
      queryClient.invalidateQueries({ queryKey: ['sessions'] });
      queryClient.invalidateQueries({ queryKey: ['session', event.data.id] });
      break;
    case 'worktreeTask:updated':
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
      break;
  }
}
```

### Pattern 3: Multi-Server Context Switching

**What:** All API calls route through the active server. Switching servers swaps the api-client instance and clears the query cache.

**When:** User switches between connected Myrlin servers.

**Example:**

```typescript
// stores/server-store.ts
const useServerStore = create<ServerState>()(
  persist(
    (set, get) => ({
      servers: [],
      activeServerId: null,
      switchServer: (id: string) => {
        set({ activeServerId: id });
        // Clear all cached data from previous server
        queryClient.clear();
      },
    }),
    { name: 'servers', storage: createSecureStorageAdapter() }
  )
);
```

### Pattern 4: WebView Bridge Protocol

**What:** Typed message protocol between React Native and the xterm.js WebView. All messages are JSON with a `type` discriminator.

**When:** Terminal rendering.

**Why:** Keeps the contract explicit and debuggable. Each message type has a clear handler on both sides.

### Pattern 5: Feature-Flag Screens

**What:** Screens that depend on server capabilities (version-gated) check server version before rendering.

**When:** New mobile endpoints (QR pairing, push registration) that older Myrlin servers do not support.

**Example:**

```typescript
function usePushSupport() {
  const version = useServerVersion();
  // Push endpoints added in v1.0.0
  return semver.gte(version, '1.0.0');
}
```

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Direct State Mutation from SSE

**What:** Directly updating Zustand/local state from SSE events instead of going through TanStack Query.

**Why bad:** Creates two sources of truth (query cache and local state). Data gets out of sync when queries refetch or when switching screens.

**Instead:** SSE events should only invalidate query keys. Let TanStack Query refetch and remain the single source of truth for all server data.

### Anti-Pattern 2: Untyped WebView Messages

**What:** Sending raw strings or unstructured objects between RN and the WebView.

**Why bad:** Silent failures when message format changes. No IDE autocompletion. Hard to debug.

**Instead:** Use the typed `ToWebView` / `FromWebView` union types defined in the design doc. Validate message types on both sides.

### Anti-Pattern 3: Global Fetch Interceptor

**What:** Adding a global fetch interceptor for auth headers or error handling.

**Why bad:** React Native fetch behaves differently from web fetch. Global interceptors conflict with third-party libraries (SSE, WebSocket). Hard to test.

**Instead:** Encapsulate in the api-client class. Each method handles its own headers and errors.

### Anti-Pattern 4: Storing Tokens in AsyncStorage/MMKV

**What:** Putting auth tokens in unencrypted storage.

**Why bad:** MMKV and AsyncStorage are not encrypted. Rooted/jailbroken devices can read them.

**Instead:** Use expo-secure-store for all sensitive data (tokens, passwords, server URLs with embedded credentials).

### Anti-Pattern 5: Synchronous Theme Loading from Async Store

**What:** Loading theme from AsyncStorage (async) and showing a loading state or white flash.

**Why bad:** Users see a flash of wrong colors on every app launch.

**Instead:** Use MMKV (synchronous read) for theme. Theme is available before the first frame renders.

---

## Scalability Considerations

| Concern | 1 server | 5 servers | 20+ sessions per server |
|---------|----------|-----------|------------------------|
| SSE connections | 1 connection | 5 simultaneous SSE streams (only active server) | Same 1 connection; server multiplexes all events |
| WebSocket terminals | 1 at a time (foreground) | Same, per active server | Multiple open terminals via tab switching; close background WS after 30s idle |
| Query cache | Small | Separate caches per server (clear on switch) | FlashList handles large lists; pagination for 100+ sessions |
| Push tokens | Register on 1 server | Register on all 5 (each server sends independently) | Server-side concern; app just receives |
| Storage | ~50KB MMKV | ~250KB MMKV (5x server configs) | Negligible; server data is not cached long-term |

---

## Directory Structure

```
mobile/
  app/                          # expo-router screens (file-based routing)
    (auth)/                     # Unauthenticated screens
      onboarding.tsx
      scan-qr.tsx
      manual-connect.tsx
      login.tsx
    (tabs)/                     # Bottom tab navigator
      sessions/
        index.tsx               # Session list
        [id].tsx                # Session detail
        [id]/terminal.tsx       # Terminal (full screen, no tabs)
      tasks/
        index.tsx               # Kanban board
        [id].tsx                # Task detail
      costs/
        index.tsx               # Cost dashboard
      docs/
        index.tsx               # Docs list
        editor.tsx              # Note editor
      more/
        index.tsx               # More menu
        settings.tsx
        servers.tsx
        ...
    _layout.tsx                 # Root layout (ThemeProvider, QueryProvider)
  components/                   # Shared UI components
    ui/                         # Design system primitives (Button, Card, Badge, etc.)
    sessions/                   # Session-specific components
    terminal/                   # TerminalView, TerminalToolbar
    tasks/                      # KanbanColumn, TaskCard
    costs/                      # CostChart, CostBreakdown
  hooks/                        # Custom hooks (useSessions, useTheme, useSSE, etc.)
  services/                     # Pure business logic (no React)
    api-client.ts               # Typed HTTP client
    sse-client.ts               # SSE subscription manager
    ws-client.ts                # WebSocket for terminals
    push-service.ts             # Push notification registration
  stores/                       # Zustand stores
    server-store.ts             # Active server, server list
    theme-store.ts              # Theme state, 13 themes
    auth-store.ts               # Auth state, biometric lock
    ui-store.ts                 # UI preferences (sort order, view mode)
  theme/                        # Theme definitions
    tokens.ts                   # All 13 Catppuccin theme token objects
    fonts.ts                    # Font configuration
  types/                        # TypeScript types (shared with server contracts)
    api.ts                      # API response types
    sse.ts                      # SSE event types
    terminal.ts                 # WebView bridge protocol types
  assets/
    fonts/                      # JetBrains Mono .ttf files
    terminal.html               # xterm.js WebView HTML
  maestro/                      # Maestro test flows (YAML)
  .storybook/                   # Storybook config
```

---

## Sources

- Design document architecture section (Section 3)
- [TanStack Query React Native docs](https://tanstack.com/query/latest/docs/framework/react/react-native)
- [Zustand persist middleware docs](https://zustand.docs.pmnd.rs/)
- [expo-router file conventions](https://docs.expo.dev/router/basics/core-concepts/)
- [Expo SecureStore docs](https://docs.expo.dev/versions/latest/sdk/securestore/)

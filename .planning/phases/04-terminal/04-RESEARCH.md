# Phase 4: Terminal - Research

**Researched:** 2026-03-28
**Domain:** Hybrid WebView terminal (xterm.js) in React Native with native input, clipboard, sharing, camera, and voice
**Confidence:** HIGH (architecture is well-defined in design doc; all libraries are mature)

## Summary

Phase 4 is the highest-risk phase in the entire project. It implements a hybrid terminal where React Native owns all user interaction (keyboard, toolbar, gestures, sharing) while a WebView acts as a dumb rendering surface for xterm.js output. The architecture decision (from the design doc Section 3.3) is locked: no native terminal renderer, no xterm.js keyboard input. The WebSocket for PTY data runs inside the WebView, keeping terminal data off the RN bridge for performance.

The primary technical challenges are: (1) reliable cross-platform postMessage bridge between RN and WebView, (2) keyboard avoidance that works with a WebView + native TextInput below it, (3) WebSocket lifecycle management across app foreground/background transitions, (4) memory leak prevention from xterm.js disposal, and (5) the terminal session carousel with swipe navigation. All 13 themes must render correctly inside the WebView, requiring theme token injection via the bridge protocol.

**Primary recommendation:** Build the terminal in strict layers. Start with the static WebView HTML + xterm.js rendering (output only), then add the bridge protocol, then WebSocket connection, then native input, then toolbar actions, then carousel. Test each layer in isolation before composing.

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| TERM-01 | View terminal output in real-time (WebView + xterm.js) | WebView bridge protocol, xterm.js in HTML asset, WebSocket inside WebView |
| TERM-02 | Type commands via native TextInput with iOS keyboard | Native TextInput below WebView, bridge sends input to WebView WS. Avoids xterm.js onData Android buffering bug |
| TERM-03 | Copy selected terminal text to clipboard | Bridge getSelectedText/getVisibleText messages, expo-clipboard |
| TERM-04 | Paste from clipboard into terminal | expo-clipboard read, bridge sends paste data to WebView WS |
| TERM-05 | Share terminal text via native share sheet | expo-sharing or RN Share API for text content |
| TERM-06 | Upload image from camera or gallery | expo-image-picker (already installed), upload via API client |
| TERM-07 | Voice input (dictation) | @react-native-voice/voice or iOS native dictation (TextInput supports dictation natively) |
| TERM-08 | Terminal applies correct theme colors | Bridge setTheme message injects xterm.js theme object derived from current MyrlinTheme |
| TERM-09 | Reader mode (full-screen scrollable text) | Bridge getScrollback, render in native ScrollView with Text components |
| TERM-10 | Swipe between open terminal sessions (carousel) | react-native-pager-view or Reanimated + GestureHandler custom carousel |
| TERM-11 | Activity indicator in terminal header | Bridge activity messages from WebView, display in native header |
| TERM-12 | Keyboard avoidance (input stays above keyboard) | react-native-keyboard-controller (recommended over RN KeyboardAvoidingView) |
</phase_requirements>

## Standard Stack

### Core (Must Install)

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| react-native-webview | 14.x | WebView container for xterm.js | Standard RN WebView, Expo-compatible, required for terminal rendering |
| react-native-keyboard-controller | latest | Keyboard avoidance and sticky input | Superior to RN's KeyboardAvoidingView. Works with Reanimated. Consistent iOS/Android behavior. Reanimated 4 deprecated useAnimatedKeyboard in favor of this library |
| expo-clipboard | bundled | Copy/paste clipboard access | Bundled with Expo SDK 55, replaces manual Clipboard API |

### Already Installed (Use Existing)

| Library | Version | Purpose |
|---------|---------|---------|
| expo-image-picker | bundled | Camera/gallery image selection (TERM-06) |
| react-native-reanimated | 4.2.1 | Carousel animations, keyboard animations |
| react-native-gesture-handler | ~2.30.0 | Swipe gestures for terminal carousel |
| react-native-sse | 1.2.1 | SSE for session status updates |

### Possibly Needed

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| react-native-pager-view | 7.x | Native page swiping for terminal carousel | If custom Reanimated carousel is too complex; provides native iOS/Android paging |
| @react-native-voice/voice | 3.x | Speech-to-text for voice input | Only if iOS native dictation (built into TextInput) is insufficient |
| expo-sharing | bundled | Share sheet for text content | If RN's built-in Share API is insufficient for the share use case |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| react-native-keyboard-controller | RN KeyboardAvoidingView | KAV has known issues with WebView (double avoidance), inconsistent Android behavior. keyboard-controller is the community standard replacement |
| Custom Reanimated carousel | react-native-pager-view | Pager-view is native and smooth but less customizable. Custom gives full control over parallax/transition effects |
| @react-native-voice/voice | iOS dictation via TextInput | iOS TextInput supports dictation natively via the microphone button on the keyboard. Voice library is only needed for a dedicated mic button with streaming transcription |
| expo-sharing | react-native-share | expo-sharing is simpler (share files), react-native-share supports more options (social-specific). RN's built-in Share API handles text sharing fine |

**Installation:**
```bash
cd mobile
npx expo install react-native-webview react-native-keyboard-controller expo-clipboard
# Optional (evaluate during implementation):
npx expo install react-native-pager-view
npm install @react-native-voice/voice
```

## Architecture Patterns

### Recommended File Structure
```
mobile/
  assets/
    terminal.html              # Minimal HTML page embedding xterm.js (CDN or bundled)
  components/
    terminal/
      TerminalScreen.tsx        # Full terminal screen (header + webview + toolbar + input)
      TerminalWebView.tsx       # WebView wrapper with bridge protocol
      TerminalToolbar.tsx       # Copy/Paste/Share/Mic/Camera/Reader buttons
      TerminalInput.tsx         # Native TextInput with send button
      TerminalHeader.tsx        # Session name, status dot, activity indicator, back
      TerminalCarousel.tsx      # Swipe between open terminal sessions
      ReaderMode.tsx            # Full-screen scrollable text view
      useTerminalBridge.ts      # Hook: manages postMessage protocol
  hooks/
    useTerminal.ts              # Hook: connects session to terminal (WS URL, lifecycle)
  services/
    ws-client.ts                # WebSocket URL builder (NOT the WS itself; WS is in WebView)
  types/
    terminal.ts                 # ToWebView, FromWebView, TerminalTheme types
```

### Pattern 1: WebView as Dumb Renderer (Locked Decision)

**What:** React Native owns ALL interaction. WebView receives data via postMessage and renders it. WebView sends back only query responses (selected text, scrollback) and events (ready, bell, activity).

**When to use:** Always. This is the locked architecture from the design doc.

**Why:** xterm.js has a known bug where onData behaves differently on iOS vs Android (iOS fires per-keystroke, Android buffers until Enter). By using a native TextInput, we bypass this entirely. We also get native iOS keyboard features (autocorrect, dictation, paste) for free.

**Example (bridge hook):**
```typescript
// hooks/useTerminalBridge.ts
import { useRef, useCallback } from 'react';
import type { WebView } from 'react-native-webview';
import type { ToWebView, FromWebView } from '../types/terminal';

export function useTerminalBridge() {
  const webViewRef = useRef<WebView>(null);

  const sendToWebView = useCallback((msg: ToWebView) => {
    const js = `window.handleRNMessage(${JSON.stringify(msg)}); true;`;
    webViewRef.current?.injectJavaScript(js);
  }, []);

  const handleWebViewMessage = useCallback((event: { nativeEvent: { data: string } }) => {
    const msg: FromWebView = JSON.parse(event.nativeEvent.data);
    // Route message to appropriate handler
    switch (msg.type) {
      case 'ready':
        // WebView initialized, send theme + connect
        break;
      case 'selectedText':
        // Copy to clipboard
        break;
      case 'activity':
        // Update activity indicator
        break;
    }
  }, []);

  return { webViewRef, sendToWebView, handleWebViewMessage };
}
```

### Pattern 2: WebSocket Inside WebView (NOT Bridged Through RN)

**What:** The WebSocket connection to `ws://{server}/ws/terminal` lives INSIDE the WebView JavaScript, not in React Native. Terminal data flows directly from WebSocket to xterm.js without crossing the RN bridge.

**When to use:** Always for PTY data. Only user input (from native TextInput) crosses the bridge.

**Why:** Performance. PTY output can be high-volume (build logs, large file outputs). Routing every byte through postMessage would create lag. The WebView's native WebSocket handles this at native speed.

**Critical implication:** The WebView needs the server URL and auth token injected at initialization. Pass these as URL parameters to terminal.html or via an initial postMessage.

**Example (terminal.html):**
```html
<!-- assets/terminal.html -->
<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    html, body, #terminal { width: 100%; height: 100%; overflow: hidden; }
  </style>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/xterm@5/css/xterm.css">
  <script src="https://cdn.jsdelivr.net/npm/xterm@5/lib/xterm.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/xterm-addon-fit@0.8/lib/xterm-addon-fit.min.js"></script>
</head>
<body>
  <div id="terminal"></div>
  <script>
    const params = new URLSearchParams(window.location.search);
    const wsUrl = params.get('wsUrl');
    const token = params.get('token');
    const sessionId = params.get('sessionId');

    const term = new Terminal({
      cursorBlink: false,   // Cursor hidden; native TextInput is the cursor
      disableStdin: true,   // All input comes from native side
      fontSize: 13,
      fontFamily: "'JetBrains Mono', monospace",
      scrollback: 5000,
    });
    const fitAddon = new FitAddon.FitAddon();
    term.loadAddon(fitAddon);
    term.open(document.getElementById('terminal'));
    fitAddon.fit();

    let ws = null;

    // RN -> WebView message handler
    window.handleRNMessage = function(msg) {
      switch (msg.type) {
        case 'write':
          // Input from native TextInput, forward to PTY via WS
          if (ws && ws.readyState === 1) ws.send(JSON.stringify({ type: 'input', data: msg.data }));
          break;
        case 'resize':
          term.resize(msg.cols, msg.rows);
          fitAddon.fit();
          if (ws && ws.readyState === 1) ws.send(JSON.stringify({ type: 'resize', cols: msg.cols, rows: msg.rows }));
          break;
        case 'setTheme':
          term.options.theme = msg.theme;
          break;
        case 'getSelectedText':
          postToRN({ type: 'selectedText', text: term.getSelection() });
          break;
        case 'getVisibleText':
          // Extract visible buffer text
          const buf = term.buffer.active;
          let text = '';
          for (let i = 0; i < term.rows; i++) {
            const line = buf.getLine(i + buf.viewportY);
            if (line) text += line.translateToString(true) + '\n';
          }
          postToRN({ type: 'visibleText', text });
          break;
        case 'getScrollback':
          const active = term.buffer.active;
          let all = '';
          for (let i = 0; i < active.length; i++) {
            const line = active.getLine(i);
            if (line) all += line.translateToString(true) + '\n';
          }
          postToRN({ type: 'scrollback', text: all });
          break;
        case 'connect':
          connectWS(msg.wsUrl, msg.token, msg.sessionId);
          break;
      }
    };

    function postToRN(msg) {
      window.ReactNativeWebView.postMessage(JSON.stringify(msg));
    }

    function connectWS(url, authToken, sid) {
      if (ws) { try { ws.close(); } catch(_){} }
      const fullUrl = url + '?token=' + encodeURIComponent(authToken) + '&sessionId=' + sid;
      ws = new WebSocket(fullUrl);
      ws.onopen = () => {
        const dims = { type: 'resize', cols: term.cols, rows: term.rows };
        ws.send(JSON.stringify(dims));
      };
      ws.onmessage = (e) => {
        try {
          const parsed = JSON.parse(e.data);
          if (parsed.type === 'exit') {
            postToRN({ type: 'exit', exitCode: parsed.exitCode });
            return;
          }
        } catch(_) {}
        // Raw PTY output
        term.write(e.data);
      };
      ws.onclose = () => postToRN({ type: 'disconnected' });
    }

    // Notify RN that WebView is ready
    postToRN({ type: 'ready' });
  </script>
</body>
</html>
```

### Pattern 3: Keyboard Controller for Terminal Input

**What:** Use react-native-keyboard-controller's KeyboardStickyView to keep the TextInput + toolbar pinned above the keyboard, and resize the WebView terminal when the keyboard opens.

**When to use:** Always. RN's built-in KeyboardAvoidingView has known issues with WebView.

**Why:** KeyboardAvoidingView with WebView causes "double avoidance" (both the KAV and WebView's internal scroll adjust). keyboard-controller's sticky view is more predictable and works with Reanimated for smooth animations.

**Example:**
```typescript
import { KeyboardStickyView } from 'react-native-keyboard-controller';

function TerminalScreen() {
  return (
    <View style={{ flex: 1 }}>
      <TerminalHeader />
      <View style={{ flex: 1 }}>
        <TerminalWebView />
      </View>
      <KeyboardStickyView>
        <TerminalToolbar />
        <TerminalInput />
      </KeyboardStickyView>
    </View>
  );
}
```

### Pattern 4: Theme Injection via Bridge

**What:** When the app theme changes, convert the MyrlinTheme tokens into an xterm.js theme object and send it via the bridge.

**When to use:** On initial WebView ready and on theme changes.

**Example:**
```typescript
function myrlinToXtermTheme(theme: MyrlinTheme): XtermTheme {
  return {
    background: theme.colors.base,
    foreground: theme.colors.text,
    cursor: theme.colors.rosewater,
    cursorAccent: theme.colors.base,
    selectionBackground: theme.colors.accent + '40', // 25% opacity
    selectionForeground: theme.colors.text,
    black: theme.colors.surface1,
    red: theme.colors.red,
    green: theme.colors.green,
    yellow: theme.colors.yellow,
    blue: theme.colors.blue,
    magenta: theme.colors.mauve,
    cyan: theme.colors.teal,
    white: theme.colors.subtext1,
    brightBlack: theme.colors.surface2,
    brightRed: theme.colors.red,
    brightGreen: theme.colors.green,
    brightYellow: theme.colors.yellow,
    brightBlue: theme.colors.blue,
    brightMagenta: theme.colors.mauve,
    brightCyan: theme.colors.teal,
    brightWhite: theme.colors.subtext0,
  };
}
```

### Anti-Patterns to Avoid

- **xterm.js keyboard input on mobile:** xterm.js onData fires differently on iOS (per-keystroke) vs Android (buffers until Enter). This is a known open issue (xtermjs/xterm.js#5108). ALWAYS use native TextInput.
- **WebSocket in React Native (outside WebView):** Routing PTY data through the RN bridge adds latency. Keep the WebSocket inside the WebView where it has native-speed access to xterm.js.
- **postMessage for high-volume data:** postMessage serializes to JSON strings. Fine for user input (low volume), bad for PTY output (high volume). This is why WS stays in WebView.
- **KeyboardAvoidingView with WebView:** Known "double avoidance" bug where both adjust for keyboard. Use keyboard-controller instead.
- **Forgetting to dispose xterm.js on unmount:** Creates memory leaks. MUST send dispose message and unmount WebView on screen blur.

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Terminal rendering | Custom ANSI parser + RN Text components | xterm.js in WebView | ANSI rendering is immensely complex (256 colors, Unicode, cursor positioning, scrollback). xterm.js handles it all |
| Keyboard avoidance | Manual Keyboard event listeners + animated View | react-native-keyboard-controller | Platform differences between iOS/Android keyboard behavior are a rabbit hole. Library handles it |
| Page swiping (carousel) | Manual PanResponder/GestureHandler with Animated | react-native-pager-view or Reanimated PagerView pattern | Native page swiping has physics and deceleration that are hard to replicate |
| Clipboard access | Platform-specific native modules | expo-clipboard | Already bundled, handles iOS and Android differences |
| Image picking | Camera/gallery access via native modules | expo-image-picker | Already installed, handles permissions, returns URIs |

**Key insight:** The entire terminal rendering problem is solved by xterm.js. The entire keyboard problem is solved by keyboard-controller. The risk in this phase is in the integration between these solved problems, not in the problems themselves.

## Common Pitfalls

### Pitfall 1: postMessage iOS vs Android Dispatch Target
**What goes wrong:** Messages sent from RN to WebView (via injectJavaScript) work on both platforms, but messages FROM WebView to RN via `window.ReactNativeWebView.postMessage()` have historically had inconsistencies.
**Why it happens:** iOS dispatches MessageEvent on `window`, Android dispatches on `document`. react-native-webview v14 normalizes this via `window.ReactNativeWebView.postMessage`, but edge cases remain.
**How to avoid:** Always use `window.ReactNativeWebView.postMessage()` (never `window.postMessage`). Always stringify JSON before sending. Always parse in onMessage handler with try/catch.
**Warning signs:** Messages work on iOS but not Android (or vice versa).

### Pitfall 2: WebSocket Killed on iOS Background
**What goes wrong:** When the app goes to background on iOS, the WebSocket inside the WebView is suspended or killed. On foreground resume, the terminal shows stale content.
**Why it happens:** iOS suspends WKWebView JavaScript execution when the app backgrounds. WebSocket connections time out during suspension.
**How to avoid:** Listen to AppState changes. On `active` (foreground resume), send a reconnect message to the WebView. The WebView should check WS readyState and reconnect if closed. Server-side scrollback replay (already implemented in pty-manager.js) will restore output.
**Warning signs:** Terminal shows "Disconnected" after switching to another app and back.

### Pitfall 3: WebView Memory Leaks from xterm.js
**What goes wrong:** Opening and closing terminal screens repeatedly causes memory to grow. App eventually crashes.
**Why it happens:** xterm.js terminal instances are not disposed when the WebView unmounts. WebSocket connections remain open.
**How to avoid:** On screen unmount/blur: (1) send `{ type: 'dispose' }` to WebView, (2) close WebSocket, (3) call `term.dispose()` in WebView JS, (4) unmount the WebView component entirely. Set scrollback to 5000 lines max. Monitor with Xcode Instruments.
**Warning signs:** Memory usage grows monotonically when navigating to/from terminal screen.

### Pitfall 4: Keyboard Double-Avoidance with WebView
**What goes wrong:** When the keyboard opens, the content shifts up twice. The terminal output area becomes tiny or invisible.
**Why it happens:** Both KeyboardAvoidingView and WKWebView's native scroll adjustment try to accommodate the keyboard independently.
**How to avoid:** Use react-native-keyboard-controller (KeyboardStickyView) which does not resize the WebView. Set `scrollEnabled={false}` on the WebView. Disable `automaticallyAdjustsScrollIndicatorInsets`. When the keyboard opens, send a resize message to the WebView to reduce terminal rows.
**Warning signs:** Terminal jumps or shrinks excessively when keyboard opens on small iPhones (SE, Mini).

### Pitfall 5: xterm.js CDN Loading in WebView
**What goes wrong:** Terminal page loads slowly or fails to load xterm.js from CDN when on poor network.
**Why it happens:** Loading xterm.js from a CDN requires network access. If the user is on a local network (Tailscale, LAN), CDN might be unreachable.
**How to avoid:** Bundle xterm.js and xterm-addon-fit locally in the terminal.html asset. Use inline `<script>` tags or reference local files. NEVER depend on CDN for a core feature.
**Warning signs:** Terminal shows blank white page intermittently.

### Pitfall 6: Terminal Fit Dimensions Before WebSocket Connect
**What goes wrong:** Terminal spawns at default 120x30, then gets resized to actual dimensions, causing garbled output.
**Why it happens:** WebSocket connects before the WebView/xterm.js has measured its actual dimensions. The server PTY spawns at the wrong size.
**How to avoid:** In the WebView, call `fitAddon.fit()` first, THEN connect the WebSocket with the correct cols/rows. The existing web terminal.js already does this (double-rAF pattern). Replicate this in the mobile HTML.
**Warning signs:** Terminal output is garbled or misaligned after first load; typing "reset" fixes it.

## Code Examples

### Terminal Types (types/terminal.ts)
```typescript
/** Messages sent from React Native to the terminal WebView */
export type ToWebView =
  | { type: 'write'; data: string }
  | { type: 'resize'; cols: number; rows: number }
  | { type: 'setTheme'; theme: XtermTheme }
  | { type: 'clear' }
  | { type: 'connect'; wsUrl: string; token: string; sessionId: string }
  | { type: 'disconnect' }
  | { type: 'dispose' }
  | { type: 'getSelectedText' }
  | { type: 'getVisibleText' }
  | { type: 'getScrollback' }
  | { type: 'selectAll' }
  | { type: 'scrollToBottom' }
  | { type: 'focus' }
  | { type: 'blur' };

/** Messages sent from the terminal WebView to React Native */
export type FromWebView =
  | { type: 'ready' }
  | { type: 'selectedText'; text: string }
  | { type: 'visibleText'; text: string }
  | { type: 'scrollback'; text: string }
  | { type: 'activity'; kind: string; detail: string }
  | { type: 'bell' }
  | { type: 'titleChange'; title: string }
  | { type: 'dimensions'; cols: number; rows: number }
  | { type: 'disconnected' }
  | { type: 'exit'; exitCode: number };

/** xterm.js theme object (subset of ITheme) */
export interface XtermTheme {
  background: string;
  foreground: string;
  cursor: string;
  cursorAccent: string;
  selectionBackground: string;
  selectionForeground: string;
  black: string;
  red: string;
  green: string;
  yellow: string;
  blue: string;
  magenta: string;
  cyan: string;
  white: string;
  brightBlack: string;
  brightRed: string;
  brightGreen: string;
  brightYellow: string;
  brightBlue: string;
  brightMagenta: string;
  brightCyan: string;
  brightWhite: string;
}
```

### WebView URL Builder
```typescript
// Build WebSocket URL for terminal connection
function buildTerminalWsUrl(serverUrl: string, sessionId: string): string {
  const protocol = serverUrl.startsWith('https') ? 'wss:' : 'ws:';
  const host = serverUrl.replace(/^https?:\/\//, '');
  return `${protocol}//${host}/ws/terminal`;
}
```

### Image Upload Pattern (TERM-06)
```typescript
import * as ImagePicker from 'expo-image-picker';

async function pickAndUploadImage(sessionId: string, client: MyrlinAPIClient) {
  const result = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: ImagePicker.MediaTypeOptions.Images,
    quality: 0.8,
  });
  if (result.canceled) return;
  const asset = result.assets[0];
  // Create FormData for upload (avoids base64 memory overhead)
  const formData = new FormData();
  formData.append('image', {
    uri: asset.uri,
    type: asset.mimeType || 'image/jpeg',
    name: asset.fileName || 'image.jpg',
  } as any);
  // Upload via API client (server endpoint needed)
  // Note: server may need a new endpoint for image upload to session
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| KeyboardAvoidingView | react-native-keyboard-controller | 2024-2025 | RN's built-in is considered broken with complex layouts. keyboard-controller is now recommended by Reanimated team |
| Reanimated useAnimatedKeyboard | keyboard-controller | Reanimated 4 (March 2026) | useAnimatedKeyboard deprecated in Reanimated 4 in favor of keyboard-controller |
| xterm.js keyboard input on mobile | Native TextInput + bridge | Ongoing | xterm.js has open bug #5108 about iOS/Android onData inconsistency |
| CDN-loaded xterm.js in WebView | Bundled xterm.js in asset | Best practice | CDN dependency is unreliable for LAN-only setups |

## Open Questions

1. **Image upload endpoint**
   - What we know: expo-image-picker can select images, FormData can upload
   - What's unclear: Does the server have an endpoint to accept image uploads to a session? The existing web terminal supports image paste but the API may need a new endpoint
   - Recommendation: Check server.js for image upload support. If missing, add a `POST /api/sessions/:id/upload` endpoint as a Phase 4 server-side task

2. **Voice input approach**
   - What we know: iOS TextInput has built-in dictation (microphone button on keyboard). @react-native-voice/voice provides streaming speech-to-text
   - What's unclear: Is the built-in iOS dictation sufficient, or do we need the dedicated mic button in the toolbar?
   - Recommendation: Start with iOS native dictation (zero effort, built into TextInput). Only add @react-native-voice/voice if a dedicated toolbar mic button is explicitly needed

3. **xterm.js bundling strategy**
   - What we know: xterm.js can be loaded from CDN or bundled inline
   - What's unclear: Best way to bundle xterm.js + fit addon in a React Native asset (inline in HTML, or as separate JS files referenced by the WebView)
   - Recommendation: Inline the minified xterm.js and fit addon directly in terminal.html. This avoids CDN dependency and cross-origin issues in WKWebView. Total size is approximately 200KB (acceptable for an asset)

4. **Terminal carousel implementation**
   - What we know: react-native-pager-view provides native page swiping, Reanimated can build custom carousels
   - What's unclear: Whether multiple WebViews (one per terminal session) cause memory issues
   - Recommendation: Only mount the active terminal's WebView. Adjacent carousel pages show a terminal preview snapshot (last screenshot or text snapshot). Lazy-mount WebView on swipe completion

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Maestro (E2E) + Jest (unit) |
| Config file | mobile/maestro/config.yaml |
| Quick run command | `maestro test mobile/maestro/flows/terminal-basic.yaml` |
| Full suite command | `maestro test mobile/maestro/flows/` |

### Phase Requirements to Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| TERM-01 | Terminal output renders in WebView | e2e | `maestro test mobile/maestro/flows/terminal-output.yaml` | Wave 0 |
| TERM-02 | Type and send command via TextInput | e2e | `maestro test mobile/maestro/flows/terminal-input.yaml` | Wave 0 |
| TERM-03 | Copy terminal text to clipboard | e2e | `maestro test mobile/maestro/flows/terminal-copy.yaml` | Wave 0 |
| TERM-04 | Paste into terminal | e2e | `maestro test mobile/maestro/flows/terminal-paste.yaml` | Wave 0 |
| TERM-05 | Share via share sheet | manual-only | N/A (share sheet is system UI) | N/A |
| TERM-06 | Upload image from picker | manual-only | N/A (image picker is system UI) | N/A |
| TERM-07 | Voice dictation input | manual-only | N/A (dictation is system UI) | N/A |
| TERM-08 | Theme colors match app theme | e2e/screenshot | `maestro test mobile/maestro/flows/terminal-theme.yaml` | Wave 0 |
| TERM-09 | Reader mode scrollable text | e2e | `maestro test mobile/maestro/flows/terminal-reader.yaml` | Wave 0 |
| TERM-10 | Swipe between terminals | e2e | `maestro test mobile/maestro/flows/terminal-carousel.yaml` | Wave 0 |
| TERM-11 | Activity indicator in header | e2e | `maestro test mobile/maestro/flows/terminal-activity.yaml` | Wave 0 |
| TERM-12 | Keyboard avoidance | e2e/screenshot | `maestro test mobile/maestro/flows/terminal-keyboard.yaml` | Wave 0 |

### Sampling Rate
- **Per task commit:** Visual inspection on iOS Simulator (terminal output, keyboard, theme)
- **Per wave merge:** Full Maestro suite
- **Phase gate:** Full suite green + manual test of share sheet, image picker, dictation

### Wave 0 Gaps
- [ ] `mobile/maestro/flows/terminal-output.yaml` - basic terminal rendering
- [ ] `mobile/maestro/flows/terminal-input.yaml` - type and send
- [ ] `mobile/types/terminal.ts` - bridge protocol types
- [ ] `mobile/assets/terminal.html` - xterm.js WebView page
- [ ] Install react-native-webview, react-native-keyboard-controller, expo-clipboard
- [ ] Rebuild dev build (new native dependencies)

## Sources

### Primary (HIGH confidence)
- Design doc Section 3.3: Terminal Architecture (locked decisions, bridge protocol, renderer interface)
- Existing web terminal: `src/web/public/terminal.js` (xterm.js config, WebSocket protocol, theme objects)
- Existing PTY manager: `src/web/pty-manager.js` (server-side WebSocket protocol, scrollback replay)
- [react-native-webview reference docs](https://github.com/react-native-webview/react-native-webview/blob/master/docs/Reference.md)
- [react-native-keyboard-controller docs](https://kirillzyusko.github.io/react-native-keyboard-controller/docs/api/components/keyboard-avoiding-view)
- [xterm.js API](https://xtermjs.org/)

### Secondary (MEDIUM confidence)
- [xterm.js iOS/Android onData inconsistency (issue #5108)](https://github.com/xtermjs/xterm.js/issues/5108) - confirms native TextInput is correct approach
- [react-native-webview postMessage iOS/Android differences (issue #2980)](https://github.com/react-native-webview/react-native-webview/issues/2980)
- [react-native-webview WebSocket + iOS background (issue #2281)](https://github.com/react-native-webview/react-native-webview/issues/2281)
- [Expo keyboard handling guide](https://docs.expo.dev/guides/keyboard-handling/)
- [react-native-webview + KeyboardAvoidingView double-avoidance (issue #2149)](https://github.com/react-native-webview/react-native-webview/issues/2149)

### Tertiary (LOW confidence)
- [@fressh/react-native-xtermjs-webview (npm)](https://www.npmjs.com/package/@fressh/react-native-xtermjs-webview) - exists as reference but immature (v0.0.8), better to build our own thin bridge

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - all libraries are mature, well-documented, and Expo-compatible
- Architecture: HIGH - design doc specifies exact architecture; existing web implementation provides proven patterns
- Pitfalls: HIGH - researched specific GitHub issues confirming each pitfall is real and documented
- Bridge protocol: HIGH - typed protocol is fully defined in design doc with all message types
- Keyboard handling: MEDIUM - keyboard-controller is proven but integration with WebView + sticky input is novel combination to validate

**Research date:** 2026-03-28
**Valid until:** 2026-04-28 (stable domain, libraries are mature)

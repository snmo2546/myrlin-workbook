---
phase: 4
slug: terminal
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-29
---

# Phase 4 - Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | TypeScript (type check), Maestro (E2E on Simulator) |
| **Quick run command** | `cd mobile && npx tsc --noEmit` |
| **Full suite command** | `cd mobile && npx tsc --noEmit && maestro test maestro/flows/terminal.yaml` |
| **Estimated runtime** | ~45 seconds |

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Terminal output renders correctly | TERM-01 | Requires running server + PTY | Start session, verify output streams |
| Keyboard never covers input | TERM-12 | Physical device keyboard behavior | Open terminal, tap input, verify visibility |
| Voice dictation | TERM-07 | Requires microphone hardware | Tap mic, speak command, verify text |
| Camera image upload | TERM-06 | Requires camera hardware | Tap camera, take photo, verify upload |
| Swipe between sessions | TERM-10 | Touch gesture on device | Open 2+ terminals, swipe left/right |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Feedback latency < 45s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending

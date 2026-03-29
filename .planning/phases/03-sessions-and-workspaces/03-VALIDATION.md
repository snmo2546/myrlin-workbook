---
phase: 3
slug: sessions-and-workspaces
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-29
---

# Phase 3 - Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Jest (unit), TypeScript (type check) |
| **Quick run command** | `cd mobile && npx tsc --noEmit` |
| **Full suite command** | `cd mobile && npx tsc --noEmit && npm test` |
| **Estimated runtime** | ~30 seconds |

---

## Sampling Rate

- **After every task commit:** TypeScript compiles
- **After every plan wave:** Full type check + any unit tests
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 30 seconds

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| SSE real-time updates | SESS-01, SESS-16, SESS-17 | Requires running server + dev build | Start session on desktop, verify mobile list updates |
| Drag-reorder workspaces | WORK-05 | Requires touch gesture on device | Long-press workspace, drag to new position |
| AI search | SRCH-02 | Requires Anthropic API key | Toggle to AI mode, search query, verify results |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Feedback latency < 30s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending

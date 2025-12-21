---
status: draft
derived-from:
  - ../component-api.md
  - ../state-machines.md
  - ../keyboard-shortcuts.md
derived-at: 2025-12-21T17:00:00Z
loom-version: "1.0"
structured-interview:
  decisions:
    UI-QA-1: Critical paths only
    UI-QA-2: Desktop only
tags:
  - specification
  - ux-ui
  - testing
  - manual-qa
---

# Manual QA Checklist – Flux Scheduling UI

Manual testing checklist for critical paths that cannot be fully automated.

---

## Overview

**Scope:** Critical paths only (per SI decision UI-QA-1)
**Devices:** Desktop only (per SI decision UI-QA-2)

### QA Focus Areas

Manual QA focuses on what automation cannot reliably verify:

1. **UX Feel** - Does interaction feel smooth and natural?
2. **Visual Polish** - Subtle alignment, spacing, animation smoothness
3. **Error Recovery** - Can user recover from errors?
4. **Edge Cases** - Boundary conditions in real usage
5. **Keyboard UX** - Natural keyboard navigation flow

---

## Device Matrix

| Device | Browser | Resolution | Priority |
|--------|---------|------------|----------|
| MacBook Pro 14" | Chrome 120+ | 1512x982 | High |
| MacBook Pro 14" | Safari 17+ | 1512x982 | High |
| MacBook Pro 14" | Firefox 120+ | 1512x982 | Medium |
| Windows Desktop | Chrome 120+ | 1920x1080 | High |
| Windows Desktop | Edge 120+ | 1920x1080 | Medium |
| 27" Monitor | Chrome 120+ | 2560x1440 | Medium |

---

## Critical Path 1: New Task Scheduling

### QA-UI-CP1-001: Schedule First Task {#qa-ui-cp1-001}

**Objective:** Verify core scheduling workflow

**Preconditions:**
- [ ] Application loaded
- [ ] At least one job with unscheduled tasks visible
- [ ] Grid shows current day

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Click on a job card in the jobs list | Job details panel opens, shows job info and task list | [ ] |
| 2 | Locate an unscheduled task in the task list | Task shows "Unscheduled" state, cursor is grab | [ ] |
| 3 | Drag task from sidebar towards grid | Drag preview appears, follows cursor smoothly | [ ] |
| 4 | Hover over correct station column | Green ring appears around column | [ ] |
| 5 | Drop task at 09:00 position | Tile appears at 09:00, snapped to grid | [ ] |
| 6 | Verify sidebar task state | Task now shows "Scheduled" state | [ ] |

**UX Quality Checks:**
- [ ] Drag preview follows cursor with no perceptible lag (<10ms)
- [ ] Green ring animation is smooth (60fps)
- [ ] Tile appears instantly on drop
- [ ] No visual glitches during drag

**Devices:** All

---

### QA-UI-CP1-002: Schedule Multiple Tasks {#qa-ui-cp1-002}

**Objective:** Verify sequential task scheduling

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | With job selected, drag Task 1 to 08:00 | Task 1 scheduled at 08:00 | [ ] |
| 2 | Drag Task 2 to 09:30 | Task 2 scheduled at 09:30 | [ ] |
| 3 | Drag Task 3 to 11:00 | Task 3 scheduled at 11:00 | [ ] |
| 4 | Verify all 3 tiles visible on grid | All tiles show correct times | [ ] |
| 5 | Verify task list shows all scheduled | Sidebar shows 3 scheduled tasks | [ ] |

---

## Critical Path 2: Reschedule Task

### QA-UI-CP2-001: Move Tile to Different Time {#qa-ui-cp2-001}

**Objective:** Verify tile rescheduling

**Preconditions:**
- [ ] At least one scheduled tile on grid

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Hover over scheduled tile | Swap buttons appear | [ ] |
| 2 | Start dragging the tile | Ghost appears at original position | [ ] |
| 3 | Drag tile up (earlier time) | Preview shows new time position | [ ] |
| 4 | Drop at new position | Tile moves to new time, ghost disappears | [ ] |
| 5 | Verify new scheduled time | Time updated correctly | [ ] |

**UX Quality Checks:**
- [ ] Ghost has dashed border, clearly visible
- [ ] Drag preview maintains grab offset
- [ ] Drop snaps to 30-minute boundary

---

### QA-UI-CP2-002: Push-Down Behavior {#qa-ui-cp2-002}

**Objective:** Verify collision handling

**Preconditions:**
- [ ] Two consecutive tiles on same station

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Drag new task onto existing tile | Existing tile pushed down | [ ] |
| 2 | Verify all tiles visible | No overlapping tiles | [ ] |
| 3 | Verify order maintained | Original order preserved | [ ] |

---

## Critical Path 3: Cancel/Recall Task

### QA-UI-CP3-001: Recall Scheduled Task {#qa-ui-cp3-001}

**Objective:** Verify task recall (unscheduling)

**Preconditions:**
- [ ] At least one scheduled tile on grid

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Double-click on scheduled tile | Tile disappears from grid | [ ] |
| 2 | Check sidebar task list | Task shows "Unscheduled" state | [ ] |
| 3 | Verify task is draggable again | Cursor changes to grab on hover | [ ] |

---

### QA-UI-CP3-002: Cancel Drag Operation {#qa-ui-cp3-002}

**Objective:** Verify drag cancellation

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Start dragging a tile | Drag preview appears | [ ] |
| 2 | Move cursor outside all columns | Drop zone indicators disappear | [ ] |
| 3 | Release mouse button | Tile returns to original position | [ ] |
| 4 | Verify no changes made | Tile at same position, no new assignments | [ ] |

---

## Critical Path 4: Validation Feedback

### QA-UI-CP4-001: Invalid Station Feedback {#qa-ui-cp4-001}

**Objective:** Verify constraint visualization

**Preconditions:**
- [ ] Task assigned to Station A

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Drag task from sidebar | Preview follows cursor | [ ] |
| 2 | Hover over Station B (wrong station) | No drop zone indicator (no ring) | [ ] |
| 3 | Hover over Station A (correct) | Green ring appears | [ ] |
| 4 | Return to Station B | Ring disappears | [ ] |
| 5 | Drop on Station B | Drop rejected, tile returns | [ ] |

---

### QA-UI-CP4-002: Precedence Constraint Feedback {#qa-ui-cp4-002}

**Objective:** Verify precedence visualization

**Preconditions:**
- [ ] Task B depends on Task A
- [ ] Task A scheduled at 08:00 (ends 09:00)

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Drag Task B to 08:30 position | Red ring appears (conflict) | [ ] |
| 2 | Move to 09:00 position | Ring changes to green (valid) | [ ] |
| 3 | Hold Alt key, move back to 08:30 | Ring changes to amber (bypass) | [ ] |
| 4 | Release Alt key | Ring returns to red | [ ] |

---

### QA-UI-CP4-003: Warning State (Plates Pending) {#qa-ui-cp4-003}

**Objective:** Verify soft constraint visualization

**Preconditions:**
- [ ] Job with BAT approved but Plates pending

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Drag task to valid station | Orange ring appears (warning) | [ ] |
| 2 | Drop task | Task is scheduled (soft constraint) | [ ] |

---

## Critical Path 5: Quick Placement Mode

### QA-UI-CP5-001: Quick Placement Workflow {#qa-ui-cp5-001}

**Objective:** Verify keyboard-driven placement

**Preconditions:**
- [ ] Job selected with multiple unscheduled tasks

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Press Alt+Q | Quick placement mode activates | [ ] |
| 2 | Observe station columns | Available stations highlighted, others dimmed | [ ] |
| 3 | Move cursor over available station | Placement indicator (white line) follows | [ ] |
| 4 | Click at 10:00 position | Task placed at 10:00 | [ ] |
| 5 | Move to another position, click | Next task placed | [ ] |
| 6 | Press Escape | Quick placement mode deactivates | [ ] |

**UX Quality Checks:**
- [ ] Placement indicator glows (white shadow)
- [ ] Indicator snaps to 30-minute grid
- [ ] Mode transition is smooth

---

## Critical Path 6: Keyboard Navigation

### QA-UI-CP6-001: Job Navigation {#qa-ui-cp6-001}

**Objective:** Verify keyboard job selection

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Press Alt+↓ | First job selected | [ ] |
| 2 | Press Alt+↓ again | Next job selected | [ ] |
| 3 | Press Alt+↑ | Previous job selected | [ ] |
| 4 | Press Alt+D | Grid scrolls to departure date | [ ] |

---

### QA-UI-CP6-002: Global Navigation {#qa-ui-cp6-002}

**Objective:** Verify grid navigation shortcuts

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Press Home | Grid scrolls to current time | [ ] |
| 2 | Press PageDown | Grid scrolls down 24 hours | [ ] |
| 3 | Press PageUp | Grid scrolls up 24 hours | [ ] |

---

## Critical Path 7: Swap Operations

### QA-UI-CP7-001: Tile Swap {#qa-ui-cp7-001}

**Objective:** Verify swap button functionality

**Preconditions:**
- [ ] Two consecutive tiles on same station

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Hover over lower tile | Swap buttons appear | [ ] |
| 2 | Click swap-up button | Tiles exchange positions | [ ] |
| 3 | Verify times updated | Start times swapped correctly | [ ] |
| 4 | Hover over upper tile | Only swap-down visible (it's now on top) | [ ] |

---

## Visual Polish Checks

### QA-UI-VIS-001: Animation Quality {#qa-ui-vis-001}

**Objective:** Verify animation smoothness

| Animation | Expected | Check |
|-----------|----------|-------|
| Tile selection glow | Smooth fade in | [ ] |
| Column collapse | Smooth width transition | [ ] |
| Drop zone ring | Instant appearance | [ ] |
| Hover state | Quick transition (100ms) | [ ] |
| Drag preview opacity | Consistent 0.8 | [ ] |

---

### QA-UI-VIS-002: Alignment & Spacing {#qa-ui-vis-002}

**Objective:** Verify visual consistency

| Element | Check |
|---------|-------|
| Tile aligns with hour markers | [ ] |
| Ghost aligns exactly with original position | [ ] |
| Swap buttons centered on tile | [ ] |
| Job card borders consistent | [ ] |
| Timeline numbers aligned | [ ] |

---

## Error Recovery Checks

### QA-UI-ERR-001: Network Error During Save {#qa-ui-err-001}

**Objective:** Verify error handling

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Open DevTools, set offline mode | Network requests will fail | [ ] |
| 2 | Schedule a task | Task appears on grid | [ ] |
| 3 | Wait for save attempt | Error notification appears | [ ] |
| 4 | Go back online | Retry succeeds or clear error state | [ ] |

---

### QA-UI-ERR-002: Rapid Actions {#qa-ui-err-002}

**Objective:** Verify no double-processing

**Steps:**

| # | Action | Expected Result | Pass |
|---|--------|-----------------|------|
| 1 | Rapidly double-click on tile | Single recall, not duplicate | [ ] |
| 2 | Rapidly schedule multiple tasks | Each task scheduled once | [ ] |

---

## Pre-Release Regression Checklist

Run before each release:

### Critical Paths (Must Pass)

- [ ] QA-UI-CP1-001: Schedule First Task
- [ ] QA-UI-CP2-001: Move Tile to Different Time
- [ ] QA-UI-CP3-001: Recall Scheduled Task
- [ ] QA-UI-CP4-001: Invalid Station Feedback
- [ ] QA-UI-CP5-001: Quick Placement Workflow
- [ ] QA-UI-CP6-001: Job Navigation

### Important Paths

- [ ] QA-UI-CP1-002: Schedule Multiple Tasks
- [ ] QA-UI-CP2-002: Push-Down Behavior
- [ ] QA-UI-CP3-002: Cancel Drag Operation
- [ ] QA-UI-CP4-002: Precedence Constraint Feedback
- [ ] QA-UI-CP7-001: Tile Swap

### Edge Cases

- [ ] QA-UI-CP4-003: Warning State (Plates Pending)
- [ ] QA-UI-ERR-001: Network Error During Save
- [ ] QA-UI-ERR-002: Rapid Actions

### Visual Quality

- [ ] QA-UI-VIS-001: Animation Quality
- [ ] QA-UI-VIS-002: Alignment & Spacing

---

## Test Session Template

```markdown
# QA Test Session

**Date:** _______________
**Tester:** _______________
**Build:** _______________
**Browser:** _______________
**Resolution:** _______________

## Results

| Test ID | Status | Notes |
|---------|--------|-------|
| QA-UI-CP1-001 | ⬜ | |
| QA-UI-CP1-002 | ⬜ | |
| QA-UI-CP2-001 | ⬜ | |
| ... | | |

## Issues Found

| # | Severity | Description | Steps to Reproduce |
|---|----------|-------------|-------------------|
| 1 | | | |

## Sign-off

- [ ] All critical paths passed
- [ ] No P1 issues open
- [ ] Ready for release
```

---

## Related Documents

- [E2E Tests](e2e-tests.md)
- [Visual Tests](visual-tests.md)
- [Keyboard Shortcuts](../keyboard-shortcuts.md)
- [State Machines](../state-machines.md)
- [Accessibility Audit](accessibility-audit.md)

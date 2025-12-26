---
title: "UX-UI Derivation Architecture"
status: "draft"
version: "1.0.0"
created: "2025-12-21"
context: "Decisions based on real-world UX-UI documentation analysis"
---

# UX-UI Derivation Architecture

## Executive Summary

This document defines how Loom handles **frontend/UI derivation** separately from backend derivation. Based on analysis of a real-world UX-UI documentation project, we establish patterns for:

1. UI-specific derivation levels (L0-UI through L3-UI)
2. Traceability between backend and frontend through Business Rules
3. Cross-cutting concerns as shared patterns
4. UI-specific Structured Interview catalogs

---

## Key Insight: UI is Not a Backend Fork

UI derivation is fundamentally different from backend derivation:

| Aspect | Backend | Frontend |
|--------|---------|----------|
| **Primary Focus** | Logic (what happens?) | Experience (what does user see/feel?) |
| **L0 Input** | User Stories | User Stories + **Mockups/Design** |
| **L1 Output** | AC, BR | **UI Interaction Stories**, UI AC |
| **L2 Output** | API Contracts, Sequences | **Component Specs, State Machines** |
| **L3 Output** | Unit/Integration Tests | **E2E, Visual Regression, Manual QA** |
| **Automation** | ~95% | ~60-70% |

---

## Derivation Levels

### L0-UI: Design Artifacts (Input)

UI derivation requires additional L0 inputs beyond User Stories:

```
L0 (Shared)
├── user-stories.md
├── domain-vocabulary.md
└── business-rules.md  ← Bridge to frontend

L0-UI (UI-specific)
├── mockups/           ← HTML, Figma, Sketch
├── design-tokens.md   ← Colors, spacing, typography
└── design-system.md   ← Component library reference
```

### L1-UI: Interaction Design

```yaml
# ui-interaction-stories.md
---
id: US-UI-DRAG-001
references:
  - BR-ASSIGN-001  # Link to backend business rule
---

As a scheduler, I want to drag an unscheduled task from the
sidebar and drop it onto a station column, so that I can
quickly schedule new work.
```

```yaml
# ui-acceptance-criteria.md
---
id: AC-UI-DRAG-001-1
derived-from: US-UI-DRAG-001
---

Given an unscheduled task tile in the sidebar
When the user starts dragging it
Then the tile should be draggable (cursor changes to grab)
```

**Key difference from backend L1:**
- Stories describe HOW users interact, not WHAT the system does
- AC categories: DRAG, VALID, VISUAL, TILE, QUICK, EDGE
- References BR-* for business rule traceability

### L2-UI: Implementation Design

#### Interaction Patterns (Primary)
```
interaction-patterns/
├── drag-drop.md
├── quick-placement-mode.md
├── tile-swap.md
└── tile-recall.md
```

#### State Machines
```yaml
# state-machines.md
---
id: SM-DRAG
---

States:
  - IDLE: No drag in progress
  - DRAGGING: User is dragging a tile
  - VALIDATING: Real-time validation during drag
  - DROPPING: User released, processing drop
  - CANCELLED: Drag was cancelled

Transitions:
  IDLE → DRAGGING: onDragStart
  DRAGGING → VALIDATING: onDrag (cursor moves)
  DRAGGING → DROPPING: onDrop (on valid column)
  DRAGGING → CANCELLED: onDragEnd (outside column)
```

#### Component API
```typescript
// component-api.md

interface TileProps {
  assignment: TaskAssignment;
  task: InternalTask;
  job: Job;
  onSelect?: (jobId: string) => void;
  onRecall?: (assignmentId: string) => void;
  isSelected?: boolean;
}
```

#### Visual Feedback
```
visual-feedback/
├── tile-states.md
├── conflict-indicators.md
├── loading-states.md
└── validation-feedback.md
```

### L3-UI: Test Specifications

```yaml
# e2e-tests.md (Automated)
---
id: E2E-UI-DRAG-001
derived-from: AC-UI-DRAG-001-1
---

Test: Task tile in sidebar is draggable

Given an unscheduled task tile in the sidebar
When the user starts dragging it
Then the tile should be draggable
And the cursor should change to grab
```

```yaml
# visual-tests.md (Automated)
---
id: VIS-UI-TILE-001
tool: Chromatic
---

Capture: Tile component in all states
States: [idle, hovered, selected, dragging, muted, completed]
```

```yaml
# manual-qa.md (Manual)
---
id: QA-UI-DRAG-001
---

Checklist:
- [ ] Drag feels smooth (no jank)
- [ ] Drop feedback is immediate
- [ ] Works on touch devices
- [ ] Works with keyboard (accessibility)
```

---

## Traceability Model

### Business Rules as Bridge

```
User Story (US-BOOK-001)
    │
    └── Business Rule (BR-ASSIGN-001)  ◄── BRIDGE
            │
            ├─── Backend Path ───────────────────────┐
            │    AC-BOOK-001-1                       │
            │    → API-BOOK-001                      │
            │    → TC-BOOK-001-01                    │
            │                                        │
            └─── Frontend Path ──────────────────────┤
                 US-UI-DRAG-001 (refs BR-ASSIGN-001) │
                 → AC-UI-DRAG-001-1                  │
                 → COMP-TILE                         │
                 → E2E-UI-DRAG-001                   │
```

### Cross-Reference Format

```yaml
# Backend AC
---
id: AC-BOOK-001-1
implements: BR-ASSIGN-001
---

# UI Story
---
id: US-UI-DRAG-001
implements: BR-ASSIGN-001  # Same BR!
---
```

### Validation Cross-Check

Both sides verify the same business rule differently:

```yaml
BR-BOOK-001: TimeSlot must be available

backend-verification:
  - API validates availability before booking
  - Returns SLOT_UNAVAILABLE error if not

frontend-verification:
  - UI shows unavailable slots as disabled
  - Drop validation rejects unavailable slots
  - Visual feedback: red ring on invalid drop
```

---

## Cross-Cutting Concerns

### Pattern Library Approach

Cross-cutting concerns are defined **once per project**, then referenced:

```
ui-patterns.md (derived once)
├── Loading States
│   ├── Skeleton (lists, cards)
│   ├── Spinner (buttons, inline)
│   └── Progressive (images, large data)
│
├── Error States
│   ├── Inline (form fields)
│   ├── Toast (transient errors)
│   ├── Banner (page-level warnings)
│   └── Full-page (fatal errors)
│
├── Empty States
│   ├── No data (first use)
│   ├── No results (search/filter)
│   └── Error recovery
│
├── Validation Feedback
│   ├── Real-time (as you type)
│   ├── On blur
│   └── On submit
│
└── Transitions
    ├── Page transitions
    ├── List item enter/exit
    └── Modal open/close
```

### Component Reference

```yaml
# component-spec.md
---
id: COMP-BOOKING-CALENDAR
---

Cross-cutting:
  loading: → ui-patterns.md#skeleton
  empty: → ui-patterns.md#no-data
  error: → ui-patterns.md#inline-error
```

---

## UI Structured Interview Catalog

### UI-L1: Interaction Design (7 questions)

| ID | Question | Options |
|----|----------|---------|
| UI-COMP-1 | Component granularity? | Atomic Design / Feature-based / Hybrid |
| UI-STATE-1 | State management? | Redux / Zustand / Context / URL state |
| UI-STYLE-1 | Styling approach? | CSS-in-JS / Tailwind / CSS Modules |
| UI-DS-1 | Design system? | Custom / Material / Ant / Shadcn |
| UI-A11Y-1 | Accessibility level? | WCAG 2.1 AA / AAA / Basic |
| UI-NAV-1 | Navigation pattern? | SPA / MPA / Hybrid |
| UI-FORM-1 | Form handling? | Controlled / Uncontrolled / Form library |

### UI-L2: Component Design (5 questions)

| ID | Question | Options |
|----|----------|---------|
| UI-LOAD-1 | Loading state primary? | Skeleton / Spinner / Progressive |
| UI-ERR-1 | Error display primary? | Toast / Inline / Banner |
| UI-EMPTY-1 | Empty state style? | Illustration / Text-only / CTA |
| UI-VALID-1 | Validation timing? | Real-time / On blur / On submit |
| UI-TRANS-1 | Transition style? | Smooth / Instant / Spring |

### UI-L3: Testing (5 questions)

| ID | Question | Options |
|----|----------|---------|
| UI-E2E-1 | E2E framework? | Playwright / Cypress / None |
| UI-VIS-1 | Visual regression? | Chromatic / Percy / None |
| UI-STORY-1 | Storybook? | Yes (CSF3) / Yes (MDX) / No |
| UI-QA-1 | Manual QA depth? | Full regression / Critical paths / None |
| UI-QA-2 | Device coverage? | All major / Mobile-first / Desktop only |

### Cross-Cutting (asked once per project)

| ID | Question | Options |
|----|----------|---------|
| CC-1 | Loading strategy? | Skeleton / Spinner / Progressive / Hybrid |
| CC-2 | Error display? | Toast / Inline / Banner |
| CC-3 | Validation timing? | Real-time / On blur / On submit |
| CC-4 | Empty state style? | Illustration / Text-only / CTA |

---

## Skill Architecture

### Option A: Separate Skill Chain (Recommended)

```bash
# UI derivation
/loom-ui derive --level L1 --input stories.md,mockups/ --output-dir ui/
/loom-ui derive --level L2 --input ui-stories.md --output-dir ui/
/loom-ui derive --level L3 --input component-specs.md --output-dir ui/

# UI validation
/loom-ui validate --dir ui/ --check all
```

### Option B: Unified with Target Flag

```bash
/loom derive --target frontend --level L1 --input stories.md
```

### Recommended: Option A

Separate skill chain because:
1. Different SI catalogs
2. Different input types (mockups)
3. Different output structures
4. Different validation rules
5. Clearer mental model

---

## Implementation Phases

### Phase 1: Cross-Cutting Patterns
- Define ui-patterns.md template
- SI questions for CC-1..CC-4
- One-time derivation per project

### Phase 2: L1-UI Skill
- Input: User Stories + Mockups + BR references
- Output: UI Interaction Stories, UI Acceptance Criteria
- SI: UI-COMP-1..UI-FORM-1

### Phase 3: L2-UI Skill
- Input: UI Stories, UI AC
- Output: Component Specs, State Machines, Interaction Patterns
- SI: UI-LOAD-1..UI-TRANS-1

### Phase 4: L3-UI Skill
- Input: Component Specs, State Machines
- Output: E2E Tests, Visual Tests, Manual QA Checklists
- SI: UI-E2E-1..UI-QA-2

---

## Open Questions

1. **Mockup format standardization?**
   - HTML/Tailwind (like reference project)
   - Figma export
   - Both supported?

2. **Design token derivation?**
   - Extract from mockups?
   - Import from design system?
   - Manual input?

3. **Storybook integration?**
   - Generate stories from component specs?
   - Separate skill?

---

## References

- Real-world UX-UI documentation: `/loom-project/tmp/ux-ui/`
- Loom Roadmap: `/loom-project/roadmap/next-steps-roadmap.md`
- Structured Interview Pattern: `/loom-project/thinking/structured-interview-pattern.md`

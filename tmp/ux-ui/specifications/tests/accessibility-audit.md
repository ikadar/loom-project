---
status: draft
derived-from:
  - ../component-api.md
  - ../keyboard-shortcuts.md
  - ../non-functional-requirements.md
derived-at: 2025-12-21T17:00:00Z
loom-version: "1.0"
tags:
  - specification
  - ux-ui
  - testing
  - accessibility
---

# Accessibility Audit Specifications – Flux Scheduling UI

Automated and manual accessibility testing specifications.

---

## Overview

**Target Standard:** WCAG 2.1 Level AA (partial - see NFR-UI-A11Y)
**Primary Focus:** Keyboard navigation (NFR-UI-A11Y-001)
**Screen Reader:** Post-MVP (NFR-UI-A11Y-003)

### Scope

Per NFR-UI-A11Y, this is an internal power-user tool. Accessibility focus is on:

1. **Keyboard Operability** - All critical actions via keyboard
2. **Color Contrast** - WCAG AA for text (4.5:1)
3. **Reduced Motion** - Respect user preferences

---

## Automated Accessibility Checks

### A11Y-AUTO-001: axe-core Component Audit {#a11y-auto-001}

Run axe-core on all Storybook stories:

```typescript
// .storybook/test-runner.ts
import { checkA11y } from '@storybook/addon-a11y';

export const postRender = async (page) => {
  await checkA11y(page, {
    rules: {
      // Disable rules not applicable to power-user internal tool
      'region': { enabled: false },
      'landmark-one-main': { enabled: false },
    },
  });
};
```

**Run command:**
```bash
pnpm test-storybook --url http://localhost:6006
```

**Expected:** 0 violations in all stories

---

### A11Y-AUTO-002: Lighthouse Accessibility Audit {#a11y-auto-002}

Run Lighthouse on scheduler page:

```bash
lighthouse http://localhost:3000/scheduler \
  --only-categories=accessibility \
  --output=json \
  --output-path=./lighthouse-a11y.json
```

**Target Score:** ≥85 (adjusted from 90 per post-MVP screen reader exclusion)

**CI Integration:**
```yaml
# .github/workflows/a11y.yml
- name: Lighthouse A11Y
  uses: treosh/lighthouse-ci-action@v10
  with:
    urls: http://localhost:3000/scheduler
    budgetPath: ./lighthouse-budget.json
```

---

### A11Y-AUTO-003: Color Contrast Check {#a11y-auto-003}

Verify WCAG AA color contrast ratios:

| Element | Foreground | Background | Required | Check |
|---------|------------|------------|----------|-------|
| Tile text | color-300 | color-900/40 | 4.5:1 | [ ] |
| Timeline hour | slate-400 | slate-900 | 4.5:1 | [ ] |
| Job card title | white | slate-800 | 4.5:1 | [ ] |
| Swap button icon | white | color-700 | 3:1 (UI) | [ ] |

**Tool:** axe-core rule `color-contrast`

---

## Keyboard Accessibility Tests

> **NFR Reference:** [NFR-UI-A11Y-001](../non-functional-requirements.md#nfr-ui-a11y-001)

### A11Y-KB-001: All Actions Keyboard Accessible {#a11y-kb-001}

**Objective:** Verify all critical actions work without mouse

| Action | Keyboard Method | Test Status |
|--------|-----------------|-------------|
| Select job | Alt+↓, Alt+↑ | [ ] |
| Navigate dates | Home, PageUp, PageDown | [ ] |
| Jump to departure | Alt+D | [ ] |
| Enter quick placement | Alt+Q | [ ] |
| Exit quick placement | Escape | [ ] |
| Place task | Click (in quick placement) | [ ] |

**Test Script:**
```typescript
test.describe('Keyboard Accessibility', () => {
  test('can select job via keyboard', async ({ page }) => {
    await page.goto('/scheduler');
    await page.keyboard.press('Alt+ArrowDown');

    await expect(page.getByTestId('job-card-job-001'))
      .toHaveAttribute('aria-selected', 'true');
  });

  test('can navigate without mouse', async ({ page }) => {
    await page.goto('/scheduler');

    // Select job
    await page.keyboard.press('Alt+ArrowDown');

    // Enter quick placement
    await page.keyboard.press('Alt+Q');
    await expect(page.locator('.quick-placement-active')).toBeVisible();

    // Exit
    await page.keyboard.press('Escape');
    await expect(page.locator('.quick-placement-active')).not.toBeVisible();
  });
});
```

---

### A11Y-KB-002: Focus Visibility {#a11y-kb-002}

**Objective:** Verify focus indicators are clearly visible

| Element | Focus Style | Visible |
|---------|-------------|---------|
| Job card | ring-2 ring-blue-500 | [ ] |
| Tile | ring-2 ring-white | [ ] |
| Button | ring-2 ring-offset-2 | [ ] |
| Input | ring-2 ring-primary | [ ] |

**WCAG Criterion:** 2.4.7 Focus Visible

---

### A11Y-KB-003: No Keyboard Traps {#a11y-kb-003}

**Objective:** Verify user can tab through all focusable elements

**Test Steps:**
1. [ ] Start at page load
2. [ ] Tab through all elements
3. [ ] Verify can reach all interactive elements
4. [ ] Verify can tab out of any component
5. [ ] Shift+Tab navigates in reverse

**WCAG Criterion:** 2.1.2 No Keyboard Trap

---

### A11Y-KB-004: Logical Focus Order {#a11y-kb-004}

**Objective:** Verify tab order follows visual layout

**Expected Order:**
1. Sidebar navigation icons
2. Jobs list (job cards)
3. Job details panel (task tiles)
4. Scheduling grid (station columns, tiles)
5. Timeline

**WCAG Criterion:** 2.4.3 Focus Order

---

## Color & Visual Accessibility

### A11Y-VIS-001: Color Not Sole Indicator {#a11y-vis-001}

**Objective:** Information is not conveyed by color alone

| Indicator | Color | Additional Cue |
|-----------|-------|----------------|
| Valid drop | Green ring | Ring visible (shape) |
| Invalid drop | Red ring | Ring visible (shape) |
| Warning drop | Orange ring | Ring visible (shape) |
| Completed task | Green gradient | Checkmark icon |
| Problem job | Red/orange badge | Icon + text |

**WCAG Criterion:** 1.4.1 Use of Color

---

### A11Y-VIS-002: Text Resize {#a11y-vis-002}

**Objective:** Content readable at 200% zoom

**Test Steps:**
1. [ ] Set browser zoom to 200%
2. [ ] Verify all text readable
3. [ ] Verify no horizontal scrolling required
4. [ ] Verify interactions still work

**WCAG Criterion:** 1.4.4 Resize Text

---

## Motion & Animation

### A11Y-MOT-001: Reduced Motion Support {#a11y-mot-001}

**Objective:** Respect prefers-reduced-motion setting

**Implementation:**
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

**Test Steps:**
1. [ ] Enable reduced motion in OS settings
2. [ ] Load application
3. [ ] Verify no animations play
4. [ ] Verify transitions are instant

**WCAG Criterion:** 2.3.3 Animation from Interactions

---

## ARIA Attributes (Post-MVP)

### A11Y-ARIA-001: Recommended ARIA Attributes {#a11y-aria-001}

> **Status:** Post-MVP per NFR-UI-A11Y-003

When implementing screen reader support, add:

| Component | ARIA Attribute | Value |
|-----------|----------------|-------|
| Job card | role | listitem |
| Jobs list | role | list |
| Job card | aria-selected | true/false |
| Tile | role | button |
| Tile | aria-label | "{task name}, {job ref}, {time}" |
| Drop zone | aria-dropeffect | move |
| Dragging tile | aria-grabbed | true |
| Station column | role | region |
| Station column | aria-label | "Station: {name}" |

---

### A11Y-ARIA-002: Live Regions (Post-MVP) {#a11y-aria-002}

> **Status:** Post-MVP

Announce dynamic changes:

```html
<!-- Drag feedback announcer -->
<div aria-live="assertive" aria-atomic="true" class="sr-only">
  <!-- Populated dynamically -->
</div>
```

**Announcements:**
- "Dragging {task name}"
- "Over {station name}, {time}"
- "Dropped at {time}"
- "Drag cancelled"

---

## WCAG 2.1 AA Compliance Matrix

### Perceivable

| Criterion | Title | Status | Notes |
|-----------|-------|--------|-------|
| 1.1.1 | Non-text Content | Partial | Icons have tooltips |
| 1.3.1 | Info and Relationships | Partial | Basic structure |
| 1.3.2 | Meaningful Sequence | Pass | DOM order matches visual |
| 1.4.1 | Use of Color | Pass | Icons accompany colors |
| 1.4.3 | Contrast (Minimum) | Pass | Dark theme compliant |
| 1.4.4 | Resize Text | Pass | 200% zoom works |
| 1.4.11 | Non-text Contrast | Pass | UI elements 3:1 |

### Operable

| Criterion | Title | Status | Notes |
|-----------|-------|--------|-------|
| 2.1.1 | Keyboard | Pass | All critical paths |
| 2.1.2 | No Keyboard Trap | Pass | Tab navigates through |
| 2.1.4 | Character Key Shortcuts | N/A | Uses modifier keys |
| 2.4.3 | Focus Order | Pass | Logical order |
| 2.4.7 | Focus Visible | Pass | Ring indicators |
| 2.5.1 | Pointer Gestures | N/A | Desktop only |

### Understandable

| Criterion | Title | Status | Notes |
|-----------|-------|--------|-------|
| 3.1.1 | Language of Page | Pass | lang="fr" |
| 3.2.1 | On Focus | Pass | No unexpected changes |
| 3.2.2 | On Input | Pass | No unexpected changes |

### Robust

| Criterion | Title | Status | Notes |
|-----------|-------|--------|-------|
| 4.1.1 | Parsing | Pass | Valid HTML |
| 4.1.2 | Name, Role, Value | Partial | Post-MVP ARIA |

---

## Testing Tools

### Automated

| Tool | Purpose | Integration |
|------|---------|-------------|
| axe-core | Component A11Y | Storybook addon |
| Lighthouse | Page A11Y | CI pipeline |
| eslint-plugin-jsx-a11y | Static analysis | ESLint |

### Manual

| Tool | Purpose | When |
|------|---------|------|
| VoiceOver | Screen reader | Post-MVP |
| NVDA | Screen reader | Post-MVP |
| Colour Contrast Analyser | Contrast check | Design review |
| Accessibility Insights | Comprehensive audit | Pre-release |

---

## CI/CD Integration

```yaml
# .github/workflows/a11y.yml
name: Accessibility

on: [push, pull_request]

jobs:
  axe:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - run: pnpm install
      - run: pnpm build-storybook
      - run: npx concurrently -k -s first -n "SB,TEST" \
          "npx http-server storybook-static --port 6006 --silent" \
          "npx wait-on tcp:6006 && pnpm test-storybook"

  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - run: pnpm install && pnpm build
      - run: pnpm preview &
      - uses: treosh/lighthouse-ci-action@v10
        with:
          urls: http://localhost:4173/scheduler
          uploadArtifacts: true
          temporaryPublicStorage: true
```

---

## Remediation Backlog (Post-MVP)

| Priority | Issue | WCAG | Effort |
|----------|-------|------|--------|
| P2 | Add ARIA labels to tiles | 4.1.2 | Medium |
| P2 | Add live region for drag feedback | 4.1.3 | Medium |
| P3 | Screen reader announcements | 1.1.1 | High |
| P3 | Full screen reader testing | - | High |

---

## Related Documents

- [Non-Functional Requirements](../non-functional-requirements.md)
- [Keyboard Shortcuts](../keyboard-shortcuts.md)
- [Design Tokens](../design-tokens.md)
- [E2E Tests](e2e-tests.md)
- [Manual QA](manual-qa.md)

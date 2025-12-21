---
status: draft
derived-from:
  - ../component-api.md
  - ../state-machines.md
  - ../acceptance-criteria.md
derived-at: 2025-12-21T17:00:00Z
loom-version: "1.0"
structured-interview:
  decisions:
    UI-E2E-1: Playwright
    UI-VIS-1: Chromatic
    UI-QA-1: Critical paths only
    UI-QA-2: Desktop only
tags:
  - specification
  - ux-ui
  - testing
  - e2e
---

# E2E Test Specifications – Flux Scheduling UI

Playwright test specifications derived from UI acceptance criteria and state machines.

---

## Overview

**Framework:** Playwright
**Target Browsers:** Chrome 120+, Firefox 120+, Safari 17+
**Base URL:** `http://localhost:3000`

### Test Organization

```
tests/e2e/
├── drag-drop.spec.ts       # SM-DRAG interactions
├── tile-states.spec.ts     # SM-TILE states
├── validation.spec.ts      # SM-VALID validation
├── quick-placement.spec.ts # SM-QUICK mode
├── job-navigation.spec.ts  # SM-JOB selection
└── edge-cases.spec.ts      # Edge case handling
```

---

## Test File: drag-drop.spec.ts

> **Implements:** [SM-DRAG](../state-machines.md#sm-drag)

### E2E-UI-DRAG-001: New Task Placement from Sidebar {#e2e-ui-drag-001}

> **AC:** [AC-UI-DRAG-001.1](../acceptance-criteria.md#ac-ui-drag-0011), [AC-UI-DRAG-001.2](../acceptance-criteria.md#ac-ui-drag-0012)

```typescript
import { test, expect } from '@playwright/test';

test.describe('New Task Placement', () => {
  test('should create assignment when task dropped on station', async ({ page }) => {
    // Arrange
    await page.goto('/scheduler');
    const sidebarTile = page.getByTestId('task-tile-task-001');
    const stationColumn = page.getByTestId('station-column-station-a');

    // Assert precondition - tile is draggable
    await expect(sidebarTile).toHaveCSS('cursor', 'grab');

    // Act
    await sidebarTile.dragTo(stationColumn, {
      targetPosition: { x: 50, y: 80 } // 7:00 AM position (PIXELS_PER_HOUR=80)
    });

    // Assert
    const newTile = page.locator('[data-testid^="tile-"]').last();
    await expect(newTile).toBeVisible();
    await expect(newTile).toHaveAttribute('data-scheduled-start', /07:00/);
  });

  test('should update sidebar tile to scheduled state after drop', async ({ page }) => {
    // Arrange
    await page.goto('/scheduler');
    const sidebarTile = page.getByTestId('task-tile-task-001');
    const stationColumn = page.getByTestId('station-column-station-a');

    // Act
    await sidebarTile.dragTo(stationColumn, {
      targetPosition: { x: 50, y: 80 }
    });

    // Assert - sidebar tile shows scheduled state
    await expect(sidebarTile).toHaveClass(/scheduled/);
    await expect(sidebarTile).not.toHaveCSS('cursor', 'grab');
  });
});
```

**Test Data:**
- Task: `{ id: 'task-001', name: 'Print Brochures', stationId: 'station-a', estimatedDuration: 60 }`
- Drop Y=80px maps to 07:00 (PIXELS_PER_HOUR=80, START_HOUR=6)

---

### E2E-UI-DRAG-002: Station Constraint Validation {#e2e-ui-drag-002}

> **AC:** [AC-UI-DRAG-005.1](../acceptance-criteria.md#ac-ui-drag-0051), [AC-UI-DRAG-005.2](../acceptance-criteria.md#ac-ui-drag-0052), [AC-UI-DRAG-005.3](../acceptance-criteria.md#ac-ui-drag-0053)

```typescript
test.describe('Station Constraint', () => {
  test('should reject drop on wrong station', async ({ page }) => {
    // Arrange
    await page.goto('/scheduler');
    const sidebarTile = page.getByTestId('task-tile-task-001'); // station-a task
    const wrongStation = page.getByTestId('station-column-station-b');

    // Act
    await sidebarTile.dragTo(wrongStation);

    // Assert - no new tile created
    await expect(page.locator('[data-testid^="tile-"]')).toHaveCount(0);
    // Sidebar tile still unscheduled
    await expect(sidebarTile).not.toHaveClass(/scheduled/);
  });

  test('should not show drop zone on wrong station', async ({ page }) => {
    // Arrange
    await page.goto('/scheduler');
    const sidebarTile = page.getByTestId('task-tile-task-001');
    const wrongStation = page.getByTestId('station-column-station-b');

    // Act - start drag and hover over wrong station
    await sidebarTile.hover();
    await page.mouse.down();
    await wrongStation.hover();

    // Assert - no green ring indicator
    await expect(wrongStation).not.toHaveClass(/ring-green-500/);
    await expect(wrongStation).not.toHaveClass(/bg-green-500/);

    await page.mouse.up();
  });
});
```

---

### E2E-UI-DRAG-003: Grid Snapping {#e2e-ui-drag-003}

> **AC:** [AC-UI-DRAG-003.1](../acceptance-criteria.md#ac-ui-drag-0031), [AC-UI-DRAG-003.2](../acceptance-criteria.md#ac-ui-drag-0032), [AC-UI-DRAG-003.3](../acceptance-criteria.md#ac-ui-drag-0033)

```typescript
test.describe('Grid Snapping', () => {
  const snapCases = [
    { dropY: 90, expectedTime: '07:00', note: '7:14 snaps down to 7:00' },
    { dropY: 100, expectedTime: '07:30', note: '7:16 snaps up to 7:30' },
    { dropY: 115, expectedTime: '07:30', note: '7:29 snaps down to 7:30' },
    { dropY: 125, expectedTime: '08:00', note: '7:46 snaps up to 8:00' },
  ];

  for (const { dropY, expectedTime, note } of snapCases) {
    test(`should snap Y=${dropY} to ${expectedTime} (${note})`, async ({ page }) => {
      await page.goto('/scheduler');
      const tile = page.getByTestId('task-tile-task-001');
      const station = page.getByTestId('station-column-station-a');

      await tile.dragTo(station, { targetPosition: { x: 50, y: dropY } });

      const newTile = page.locator('[data-testid^="tile-"]').last();
      await expect(newTile).toHaveAttribute('data-scheduled-start', new RegExp(expectedTime));
    });
  }

  test('should always result in minutes of 0 or 30', async ({ page }) => {
    await page.goto('/scheduler');
    const tile = page.getByTestId('task-tile-task-001');
    const station = page.getByTestId('station-column-station-a');

    // Drop at arbitrary position
    await tile.dragTo(station, { targetPosition: { x: 50, y: 137 } });

    const newTile = page.locator('[data-testid^="tile-"]').last();
    const scheduledStart = await newTile.getAttribute('data-scheduled-start');
    const minutes = new Date(scheduledStart!).getMinutes();

    expect([0, 30]).toContain(minutes);
  });
});
```

---

### E2E-UI-DRAG-004: Tile Reschedule {#e2e-ui-drag-004}

> **AC:** [AC-UI-DRAG-002.1](../acceptance-criteria.md#ac-ui-drag-0021) - [AC-UI-DRAG-002.4](../acceptance-criteria.md#ac-ui-drag-0024)

```typescript
test.describe('Tile Reschedule', () => {
  test('should move tile to earlier time when dragged up', async ({ page }) => {
    // Arrange - create a tile at 09:00
    await page.goto('/scheduler?seed=scheduled-at-9am');
    const tile = page.getByTestId('tile-assignment-001');
    const originalStart = await tile.getAttribute('data-scheduled-start');

    // Act - drag up by 80px (1 hour)
    const box = await tile.boundingBox();
    await tile.dragTo(tile, {
      sourcePosition: { x: box!.width / 2, y: box!.height / 2 },
      targetPosition: { x: box!.width / 2, y: -80 }
    });

    // Assert - tile moved to 08:00
    await expect(tile).toHaveAttribute('data-scheduled-start', /08:00/);
  });

  test('should move tile to later time when dragged down', async ({ page }) => {
    // Arrange
    await page.goto('/scheduler?seed=scheduled-at-9am');
    const tile = page.getByTestId('tile-assignment-001');

    // Act - drag down by 80px (1 hour)
    const box = await tile.boundingBox();
    await tile.dragTo(tile, {
      sourcePosition: { x: box!.width / 2, y: box!.height / 2 },
      targetPosition: { x: box!.width / 2, y: box!.height + 80 }
    });

    // Assert - tile moved to 10:00
    await expect(tile).toHaveAttribute('data-scheduled-start', /10:00/);
  });

  test('should show ghost placeholder at original position during drag', async ({ page }) => {
    await page.goto('/scheduler?seed=scheduled-at-9am');
    const tile = page.getByTestId('tile-assignment-001');

    // Start drag
    await tile.hover();
    await page.mouse.down();
    await page.mouse.move(500, 500);

    // Assert ghost visible
    const ghost = page.locator('.tile-ghost');
    await expect(ghost).toBeVisible();
    await expect(ghost).toHaveCSS('border-style', 'dashed');

    await page.mouse.up();
  });
});
```

---

### E2E-UI-DRAG-005: Push-Down on Collision {#e2e-ui-drag-005}

> **AC:** [AC-UI-DRAG-004.1](../acceptance-criteria.md#ac-ui-drag-0041) - [AC-UI-DRAG-004.3](../acceptance-criteria.md#ac-ui-drag-0043)

```typescript
test.describe('Push-Down on Collision', () => {
  test('should push existing tile down when new tile dropped on it', async ({ page }) => {
    // Arrange - existing tile at 08:00
    await page.goto('/scheduler?seed=tile-at-8am');
    const existingTile = page.getByTestId('tile-assignment-existing');
    const originalY = (await existingTile.boundingBox())!.y;

    // Act - drop new tile at 08:00
    const sidebarTile = page.getByTestId('task-tile-task-002');
    const station = page.getByTestId('station-column-station-a');
    await sidebarTile.dragTo(station, { targetPosition: { x: 50, y: 160 } }); // 08:00

    // Assert - existing tile pushed down
    const newY = (await existingTile.boundingBox())!.y;
    expect(newY).toBeGreaterThan(originalY);
  });

  test('should push multiple tiles in chain', async ({ page }) => {
    // Arrange - tiles A, B, C at 08:00, 09:00, 10:00
    await page.goto('/scheduler?seed=three-consecutive-tiles');
    const tileA = page.getByTestId('tile-assignment-a');
    const tileB = page.getByTestId('tile-assignment-b');
    const tileC = page.getByTestId('tile-assignment-c');

    // Act - drop X at 08:00
    const sidebarTile = page.getByTestId('task-tile-task-x');
    const station = page.getByTestId('station-column-station-a');
    await sidebarTile.dragTo(station, { targetPosition: { x: 50, y: 160 } });

    // Assert - order is now X, A, B, C
    const tiles = page.locator('[data-testid^="tile-assignment"]');
    const starts = await tiles.evaluateAll(els =>
      els.map(el => el.getAttribute('data-scheduled-start'))
    );
    expect(starts).toEqual([
      expect.stringMatching(/08:00/), // X
      expect.stringMatching(/09:00/), // A (pushed)
      expect.stringMatching(/10:00/), // B (pushed)
      expect.stringMatching(/11:00/), // C (pushed)
    ]);
  });
});
```

---

## Test File: validation.spec.ts

> **Implements:** [SM-VALID](../state-machines.md#sm-valid)

### E2E-UI-VALID-001: Precedence Validation {#e2e-ui-valid-001}

> **AC:** [AC-UI-VALID-001.1](../acceptance-criteria.md#ac-ui-valid-0011) - [AC-UI-VALID-001.3](../acceptance-criteria.md#ac-ui-valid-0013)

```typescript
test.describe('Precedence Validation', () => {
  test('should block drop before predecessor ends', async ({ page }) => {
    // Arrange - Task A ends at 08:30, Task B depends on A
    await page.goto('/scheduler?seed=precedence-test');
    const taskB = page.getByTestId('task-tile-task-b');
    const station = page.getByTestId('station-column-station-a');

    // Act - try to drop Task B at 08:00 (before A ends)
    await taskB.hover();
    await page.mouse.down();
    await station.hover({ position: { x: 50, y: 160 } }); // 08:00

    // Assert - red ring indicates invalid
    await expect(station).toHaveClass(/ring-red-500/);

    await page.mouse.up();
  });

  test('should auto-snap to after predecessor end time', async ({ page }) => {
    await page.goto('/scheduler?seed=precedence-test');
    const taskB = page.getByTestId('task-tile-task-b');
    const station = page.getByTestId('station-column-station-a');

    // Act - drop at 08:00 (should auto-snap to 08:30)
    await taskB.dragTo(station, { targetPosition: { x: 50, y: 160 } });

    // Assert - snapped to 08:30
    const newTile = page.locator('[data-testid^="tile-"]').last();
    await expect(newTile).toHaveAttribute('data-scheduled-start', /08:30/);
  });

  test('should show visual feedback for precedence conflict', async ({ page }) => {
    await page.goto('/scheduler?seed=precedence-test');
    const taskB = page.getByTestId('task-tile-task-b');
    const station = page.getByTestId('station-column-station-a');

    // Start drag
    await taskB.hover();
    await page.mouse.down();
    await station.hover({ position: { x: 50, y: 120 } }); // Before predecessor

    // Assert - shows conflict indicator
    await expect(station).toHaveClass(/ring-red-500/);
    await expect(page.locator('.precedence-conflict-indicator')).toBeVisible();

    await page.mouse.up();
  });
});
```

---

### E2E-UI-VALID-002: Precedence Bypass {#e2e-ui-valid-002}

> **AC:** [AC-UI-VALID-001.4](../acceptance-criteria.md#ac-ui-valid-0014), [AC-UI-VALID-001.5](../acceptance-criteria.md#ac-ui-valid-0015)

```typescript
test.describe('Precedence Bypass', () => {
  test('should allow drop before predecessor when Alt held', async ({ page }) => {
    await page.goto('/scheduler?seed=precedence-test');
    const taskB = page.getByTestId('task-tile-task-b');
    const station = page.getByTestId('station-column-station-a');

    // Act - hold Alt and drop at 08:00
    await page.keyboard.down('Alt');
    await taskB.dragTo(station, { targetPosition: { x: 50, y: 160 } });
    await page.keyboard.up('Alt');

    // Assert - dropped at 08:00 (not snapped)
    const newTile = page.locator('[data-testid^="tile-"]').last();
    await expect(newTile).toHaveAttribute('data-scheduled-start', /08:00/);
  });

  test('should show amber ring when Alt bypass active', async ({ page }) => {
    await page.goto('/scheduler?seed=precedence-test');
    const taskB = page.getByTestId('task-tile-task-b');
    const station = page.getByTestId('station-column-station-a');

    // Start drag with Alt held
    await page.keyboard.down('Alt');
    await taskB.hover();
    await page.mouse.down();
    await station.hover({ position: { x: 50, y: 120 } });

    // Assert - amber ring (bypass indicator)
    await expect(station).toHaveClass(/ring-amber-500/);

    await page.keyboard.up('Alt');
    await page.mouse.up();
  });

  test('should create PrecedenceConflict record on bypass drop', async ({ page }) => {
    await page.goto('/scheduler?seed=precedence-test');
    const taskB = page.getByTestId('task-tile-task-b');
    const station = page.getByTestId('station-column-station-a');

    // Intercept API call
    const conflictPromise = page.waitForRequest(req =>
      req.url().includes('/precedence-conflicts') && req.method() === 'POST'
    );

    // Act - bypass drop
    await page.keyboard.down('Alt');
    await taskB.dragTo(station, { targetPosition: { x: 50, y: 160 } });
    await page.keyboard.up('Alt');

    // Assert - API called
    const request = await conflictPromise;
    expect(request).toBeTruthy();
  });
});
```

---

### E2E-UI-VALID-003: Approval Gate Validation {#e2e-ui-valid-003}

> **AC:** [AC-UI-VALID-002.1](../acceptance-criteria.md#ac-ui-valid-0021) - [AC-UI-VALID-002.4](../acceptance-criteria.md#ac-ui-valid-0024)

```typescript
test.describe('Approval Gate Validation', () => {
  test('should block scheduling task without BAT approval', async ({ page }) => {
    await page.goto('/scheduler?seed=bat-not-approved');
    const task = page.getByTestId('task-tile-no-bat');
    const station = page.getByTestId('station-column-station-a');

    // Act
    await task.dragTo(station, { targetPosition: { x: 50, y: 160 } });

    // Assert - no tile created
    await expect(page.locator('[data-testid^="tile-"]')).toHaveCount(0);
  });

  test('should allow scheduling task with BAT approved', async ({ page }) => {
    await page.goto('/scheduler?seed=bat-approved');
    const task = page.getByTestId('task-tile-bat-ok');
    const station = page.getByTestId('station-column-station-a');

    await task.dragTo(station, { targetPosition: { x: 50, y: 160 } });

    await expect(page.locator('[data-testid^="tile-"]')).toHaveCount(1);
  });

  test('should show orange warning for pending Plates gate', async ({ page }) => {
    await page.goto('/scheduler?seed=plates-pending');
    const task = page.getByTestId('task-tile-plates-pending');
    const station = page.getByTestId('station-column-station-a');

    // Start drag
    await task.hover();
    await page.mouse.down();
    await station.hover({ position: { x: 50, y: 160 } });

    // Assert - orange warning ring
    await expect(station).toHaveClass(/ring-orange-500/);

    await page.mouse.up();
  });

  test('should allow drop with Plates pending (soft constraint)', async ({ page }) => {
    await page.goto('/scheduler?seed=plates-pending');
    const task = page.getByTestId('task-tile-plates-pending');
    const station = page.getByTestId('station-column-station-a');

    await task.dragTo(station, { targetPosition: { x: 50, y: 160 } });

    // Assert - tile created despite warning
    await expect(page.locator('[data-testid^="tile-"]')).toHaveCount(1);
  });
});
```

---

## Test File: tile-states.spec.ts

> **Implements:** [SM-TILE](../state-machines.md#sm-tile)

### E2E-UI-TILE-001: Tile Selection {#e2e-ui-tile-001}

> **AC:** Job selection via tile click

```typescript
test.describe('Tile Selection', () => {
  test('should select job when tile clicked', async ({ page }) => {
    await page.goto('/scheduler?seed=with-tiles');
    const tile = page.getByTestId('tile-assignment-001');

    await tile.click();

    // Assert - tile shows selected state
    await expect(tile).toHaveClass(/selected/);
    // Assert - job details panel updates
    await expect(page.getByTestId('job-details-panel'))
      .toContainText('Job: 12345');
  });

  test('should deselect when clicking same tile again', async ({ page }) => {
    await page.goto('/scheduler?seed=with-tiles');
    const tile = page.getByTestId('tile-assignment-001');

    await tile.click();
    await tile.click();

    await expect(tile).not.toHaveClass(/selected/);
  });

  test('should mute other jobs tiles when one selected', async ({ page }) => {
    await page.goto('/scheduler?seed=multiple-jobs');
    const tileJobA = page.getByTestId('tile-assignment-job-a');
    const tileJobB = page.getByTestId('tile-assignment-job-b');

    await tileJobA.click();

    // Assert - Job B tile is muted
    await expect(tileJobB).toHaveCSS('filter', /saturate/);
    await expect(tileJobB).toHaveCSS('opacity', '0.6');
  });
});
```

---

### E2E-UI-TILE-002: Tile Recall {#e2e-ui-tile-002}

> **AC:** [AC-UI-TILE-002.1](../acceptance-criteria.md#ac-ui-tile-0021) - [AC-UI-TILE-002.3](../acceptance-criteria.md#ac-ui-tile-0023)

```typescript
test.describe('Tile Recall', () => {
  test('should remove tile from grid on double-click', async ({ page }) => {
    await page.goto('/scheduler?seed=with-tiles');
    const tile = page.getByTestId('tile-assignment-001');

    await tile.dblclick();

    await expect(tile).not.toBeVisible();
  });

  test('should return task to sidebar as unscheduled', async ({ page }) => {
    await page.goto('/scheduler?seed=with-tiles');
    const tile = page.getByTestId('tile-assignment-001');
    const sidebarTile = page.getByTestId('task-tile-task-001');

    await tile.dblclick();

    await expect(sidebarTile).not.toHaveClass(/scheduled/);
    await expect(sidebarTile).toHaveCSS('cursor', 'grab');
  });

  test('should delete assignment via API', async ({ page }) => {
    await page.goto('/scheduler?seed=with-tiles');
    const tile = page.getByTestId('tile-assignment-001');

    const deletePromise = page.waitForRequest(req =>
      req.url().includes('/assignments/') && req.method() === 'DELETE'
    );

    await tile.dblclick();

    const request = await deletePromise;
    expect(request).toBeTruthy();
  });
});
```

---

### E2E-UI-TILE-003: Swap Operations {#e2e-ui-tile-003}

> **AC:** [AC-UI-TILE-001.1](../acceptance-criteria.md#ac-ui-tile-0011) - [AC-UI-TILE-001.5](../acceptance-criteria.md#ac-ui-tile-0015)

```typescript
test.describe('Swap Operations', () => {
  test('should swap up with tile above', async ({ page }) => {
    await page.goto('/scheduler?seed=two-tiles');
    const topTile = page.getByTestId('tile-assignment-top');
    const bottomTile = page.getByTestId('tile-assignment-bottom');
    const swapUpBtn = bottomTile.getByTestId('swap-up-btn');

    // Hover to show buttons
    await bottomTile.hover();
    await swapUpBtn.click();

    // Assert - positions swapped
    const topY = (await topTile.boundingBox())!.y;
    const bottomY = (await bottomTile.boundingBox())!.y;
    expect(bottomY).toBeLessThan(topY);
  });

  test('should swap down with tile below', async ({ page }) => {
    await page.goto('/scheduler?seed=two-tiles');
    const topTile = page.getByTestId('tile-assignment-top');
    const bottomTile = page.getByTestId('tile-assignment-bottom');
    const swapDownBtn = topTile.getByTestId('swap-down-btn');

    await topTile.hover();
    await swapDownBtn.click();

    // Assert - positions swapped
    const topY = (await topTile.boundingBox())!.y;
    const bottomY = (await bottomTile.boundingBox())!.y;
    expect(topY).toBeGreaterThan(bottomY);
  });

  test('should not show swap-up on topmost tile', async ({ page }) => {
    await page.goto('/scheduler?seed=two-tiles');
    const topTile = page.getByTestId('tile-assignment-top');

    await topTile.hover();

    await expect(topTile.getByTestId('swap-up-btn')).not.toBeVisible();
    await expect(topTile.getByTestId('swap-down-btn')).toBeVisible();
  });

  test('should not show swap-down on bottommost tile', async ({ page }) => {
    await page.goto('/scheduler?seed=two-tiles');
    const bottomTile = page.getByTestId('tile-assignment-bottom');

    await bottomTile.hover();

    await expect(bottomTile.getByTestId('swap-up-btn')).toBeVisible();
    await expect(bottomTile.getByTestId('swap-down-btn')).not.toBeVisible();
  });

  test('should maintain tile durations after swap', async ({ page }) => {
    await page.goto('/scheduler?seed=two-tiles-different-durations');
    const topTile = page.getByTestId('tile-assignment-top');
    const bottomTile = page.getByTestId('tile-assignment-bottom');

    const topDurationBefore = await topTile.getAttribute('data-duration');
    const bottomDurationBefore = await bottomTile.getAttribute('data-duration');

    await bottomTile.hover();
    await bottomTile.getByTestId('swap-up-btn').click();

    const topDurationAfter = await topTile.getAttribute('data-duration');
    const bottomDurationAfter = await bottomTile.getAttribute('data-duration');

    expect(topDurationAfter).toBe(topDurationBefore);
    expect(bottomDurationAfter).toBe(bottomDurationBefore);
  });
});
```

---

## Test File: quick-placement.spec.ts

> **Implements:** [SM-QUICK](../state-machines.md#sm-quick)

### E2E-UI-QUICK-001: Quick Placement Mode {#e2e-ui-quick-001}

> **AC:** [AC-UI-QUICK-001.1](../acceptance-criteria.md#ac-ui-quick-0011) - [AC-UI-QUICK-001.5](../acceptance-criteria.md#ac-ui-quick-0015)

```typescript
test.describe('Quick Placement Mode', () => {
  test('should toggle quick placement with Alt+Q', async ({ page }) => {
    await page.goto('/scheduler?seed=job-selected');

    // Toggle on
    await page.keyboard.press('Alt+Q');
    await expect(page.locator('.quick-placement-active')).toBeVisible();

    // Toggle off
    await page.keyboard.press('Alt+Q');
    await expect(page.locator('.quick-placement-active')).not.toBeVisible();
  });

  test('should place task on station click', async ({ page }) => {
    await page.goto('/scheduler?seed=job-with-unscheduled-task');
    const station = page.getByTestId('station-column-station-a');

    await page.keyboard.press('Alt+Q');
    await station.click({ position: { x: 50, y: 160 } }); // 08:00

    await expect(page.locator('[data-testid^="tile-"]')).toHaveCount(1);
  });

  test('should dim unavailable stations', async ({ page }) => {
    await page.goto('/scheduler?seed=task-for-station-a');
    const stationB = page.getByTestId('station-column-station-b');

    await page.keyboard.press('Alt+Q');

    await expect(stationB).toHaveCSS('opacity', '0.5');
  });

  test('should exit on Escape', async ({ page }) => {
    await page.goto('/scheduler?seed=job-selected');

    await page.keyboard.press('Alt+Q');
    await page.keyboard.press('Escape');

    await expect(page.locator('.quick-placement-active')).not.toBeVisible();
  });

  test('should show placement indicator at cursor position', async ({ page }) => {
    await page.goto('/scheduler?seed=job-with-unscheduled-task');
    const station = page.getByTestId('station-column-station-a');

    await page.keyboard.press('Alt+Q');
    await station.hover({ position: { x: 50, y: 160 } });

    const indicator = page.locator('.placement-indicator');
    await expect(indicator).toBeVisible();
    // Check indicator has glow effect
    await expect(indicator).toHaveCSS('box-shadow', /rgba\(255, 255, 255/);
  });
});
```

---

## Test File: job-navigation.spec.ts

> **Implements:** [SM-JOB](../state-machines.md#sm-job)

### E2E-UI-JOB-001: Job Navigation {#e2e-ui-job-001}

```typescript
test.describe('Job Navigation', () => {
  test('should navigate to next job with Alt+Down', async ({ page }) => {
    await page.goto('/scheduler?seed=multiple-jobs');
    const jobCard1 = page.getByTestId('job-card-job-001');
    const jobCard2 = page.getByTestId('job-card-job-002');

    await jobCard1.click();
    await page.keyboard.press('Alt+ArrowDown');

    await expect(jobCard2).toHaveClass(/selected/);
    await expect(jobCard1).not.toHaveClass(/selected/);
  });

  test('should navigate to previous job with Alt+Up', async ({ page }) => {
    await page.goto('/scheduler?seed=multiple-jobs');
    const jobCard1 = page.getByTestId('job-card-job-001');
    const jobCard2 = page.getByTestId('job-card-job-002');

    await jobCard2.click();
    await page.keyboard.press('Alt+ArrowUp');

    await expect(jobCard1).toHaveClass(/selected/);
  });

  test('should jump to departure date with Alt+D', async ({ page }) => {
    await page.goto('/scheduler?seed=job-with-departure');
    const jobCard = page.getByTestId('job-card-job-001');

    await jobCard.click();
    await page.keyboard.press('Alt+D');

    // Assert - departure date line visible
    await expect(page.locator('.departure-date-line')).toBeVisible();
  });

  test('should jump to today with Home key', async ({ page }) => {
    await page.goto('/scheduler?seed=scrolled-away');

    await page.keyboard.press('Home');

    await expect(page.locator('.now-line')).toBeInViewport();
  });
});
```

---

## Test File: edge-cases.spec.ts

### E2E-UI-EDGE-001: Past Time Validation {#e2e-ui-edge-001}

> **AC:** [AC-UI-EDGE-001.1](../acceptance-criteria.md#ac-ui-edge-0011), [AC-UI-EDGE-001.2](../acceptance-criteria.md#ac-ui-edge-0012)

```typescript
test.describe('Past Time Validation', () => {
  test('should block scheduling in the past', async ({ page }) => {
    // Set clock to 10:00 AM
    await page.clock.setFixedTime(new Date('2024-01-15T10:00:00'));
    await page.goto('/scheduler');

    const task = page.getByTestId('task-tile-task-001');
    const station = page.getByTestId('station-column-station-a');

    // Try to drop at 08:00 (in the past)
    await task.dragTo(station, { targetPosition: { x: 50, y: 160 } });

    // Assert - no tile created
    await expect(page.locator('[data-testid^="tile-"]')).toHaveCount(0);
  });
});
```

---

### E2E-UI-EDGE-002: Drag Cancel {#e2e-ui-edge-002}

> **AC:** [AC-UI-EDGE-002.1](../acceptance-criteria.md#ac-ui-edge-0021), [AC-UI-EDGE-002.2](../acceptance-criteria.md#ac-ui-edge-0022)

```typescript
test.describe('Drag Cancel', () => {
  test('should cancel drag when dropped outside columns', async ({ page }) => {
    await page.goto('/scheduler?seed=with-tiles');
    const tile = page.getByTestId('tile-assignment-001');
    const originalY = (await tile.boundingBox())!.y;

    // Drag to empty area
    await tile.hover();
    await page.mouse.down();
    await page.mouse.move(50, 50); // Outside grid
    await page.mouse.up();

    // Assert - tile returns to original position
    const newY = (await tile.boundingBox())!.y;
    expect(newY).toBe(originalY);
  });
});
```

---

### E2E-UI-EDGE-003: Overnight Tasks {#e2e-ui-edge-003}

> **AC:** [AC-UI-EDGE-003.1](../acceptance-criteria.md#ac-ui-edge-0031), [AC-UI-EDGE-003.2](../acceptance-criteria.md#ac-ui-edge-0032)

```typescript
test.describe('Overnight Tasks', () => {
  test('should calculate correct end time for overnight task', async ({ page }) => {
    await page.goto('/scheduler');
    const task = page.getByTestId('task-tile-4hr-task'); // 4 hour task
    const station = page.getByTestId('station-column-station-a');

    // Drop at 22:00
    await task.dragTo(station, { targetPosition: { x: 50, y: 1280 } }); // 22:00

    const tile = page.locator('[data-testid^="tile-"]').last();
    await expect(tile).toHaveAttribute('data-scheduled-end', /02:00/);
  });
});
```

---

## Test Coverage Matrix

| AC ID | Test ID | Status |
|-------|---------|--------|
| AC-UI-DRAG-001.1 | E2E-UI-DRAG-001 | Covered |
| AC-UI-DRAG-001.2 | E2E-UI-DRAG-001 | Covered |
| AC-UI-DRAG-001.3 | E2E-UI-DRAG-002 | Covered |
| AC-UI-DRAG-001.4 | E2E-UI-DRAG-001 | Covered |
| AC-UI-DRAG-001.5 | E2E-UI-DRAG-001 | Covered |
| AC-UI-DRAG-002.1-4 | E2E-UI-DRAG-004 | Covered |
| AC-UI-DRAG-002.5 | E2E-UI-DRAG-004 | Covered |
| AC-UI-DRAG-003.1-3 | E2E-UI-DRAG-003 | Covered |
| AC-UI-DRAG-004.1-3 | E2E-UI-DRAG-005 | Covered |
| AC-UI-DRAG-005.1-3 | E2E-UI-DRAG-002 | Covered |
| AC-UI-VALID-001.1-3 | E2E-UI-VALID-001 | Covered |
| AC-UI-VALID-001.4-5 | E2E-UI-VALID-002 | Covered |
| AC-UI-VALID-002.1-4 | E2E-UI-VALID-003 | Covered |
| AC-UI-TILE-001.1-5 | E2E-UI-TILE-003 | Covered |
| AC-UI-TILE-002.1-3 | E2E-UI-TILE-002 | Covered |
| AC-UI-QUICK-001.1-5 | E2E-UI-QUICK-001 | Covered |
| AC-UI-EDGE-001.1-2 | E2E-UI-EDGE-001 | Covered |
| AC-UI-EDGE-002.1-2 | E2E-UI-EDGE-002 | Covered |
| AC-UI-EDGE-003.1-2 | E2E-UI-EDGE-003 | Covered |
| AC-UI-EDGE-004.1-2 | E2E-UI-DRAG-005 | Covered |
| AC-UI-VISUAL-001.1-5 | Visual Tests | See visual-tests.md |
| AC-UI-VISUAL-002.1-3 | Visual Tests | See visual-tests.md |
| AC-UI-VISUAL-003.1-3 | Visual Tests | See visual-tests.md |

---

## Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
  webServer: {
    command: 'pnpm dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

---

## Related Documents

- [Acceptance Criteria](../acceptance-criteria.md)
- [State Machines](../state-machines.md)
- [Component API](../component-api.md)
- [Visual Tests](visual-tests.md)
- [Manual QA](manual-qa.md)

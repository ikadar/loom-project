---
status: draft
derived-from:
  - ../component-api.md
  - ../state-machines.md
  - ../design-tokens.md
derived-at: 2025-12-21T17:00:00Z
loom-version: "1.0"
structured-interview:
  decisions:
    UI-VIS-1: Chromatic
    UI-STORY-1: Yes (CSF3)
tags:
  - specification
  - ux-ui
  - testing
  - visual
---

# Visual Regression Test Specifications – Flux Scheduling UI

Chromatic visual test specifications for component states using Storybook.

---

## Overview

**Tool:** Chromatic (Storybook integration)
**Story Format:** CSF3 (Component Story Format 3)
**Breakpoints:** Desktop only (1280px, 1920px)

### Test Strategy

Visual tests capture all component states to detect unintended visual changes:

1. **Tile Component** - 6 visual states
2. **Station Column** - 5 drop zone states
3. **Job Details Panel** - Selected/unselected states
4. **Quick Placement** - Mode indicators
5. **Drag Preview** - Floating preview

---

## Tile Component Visual Tests

> **Component:** [COMP-TILE](../component-api.md#tile)
> **State Machine:** [SM-TILE](../state-machines.md#sm-tile)

### VIS-UI-TILE-001: Tile Visual States {#vis-ui-tile-001}

Capture all tile visual states from SM-TILE:

| State | Visual Characteristics | Storybook Story |
|-------|----------------------|-----------------|
| IDLE | Default colors, no shadow | `Tile/Idle` |
| HOVERED | Swap buttons visible | `Tile/Hovered` |
| SELECTED | Glow effect (boxShadow) | `Tile/Selected` |
| DRAGGING | 0.8 opacity, original has ghost | `Tile/Dragging` |
| MUTED | saturate(0.2), opacity 0.6 | `Tile/Muted` |
| COMPLETED | Green gradient overlay | `Tile/Completed` |

**Storybook Stories:**

```typescript
// src/components/Tile/Tile.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Tile } from './Tile';
import { mockAssignment, mockTask, mockJob } from '@/test/fixtures';

const meta: Meta<typeof Tile> = {
  title: 'Components/Tile',
  component: Tile,
  parameters: {
    chromatic: { viewports: [1280, 1920] },
  },
};

export default meta;
type Story = StoryObj<typeof Tile>;

export const Idle: Story = {
  args: {
    assignment: mockAssignment,
    task: mockTask,
    job: mockJob,
    top: 100,
  },
};

export const Hovered: Story = {
  args: {
    ...Idle.args,
    showSwapUp: true,
    showSwapDown: true,
  },
  parameters: {
    pseudo: { hover: true },
  },
};

export const Selected: Story = {
  args: {
    ...Idle.args,
    isSelected: true,
  },
};

export const Dragging: Story = {
  args: {
    ...Idle.args,
  },
  decorators: [
    (Story) => (
      <div style={{ opacity: 0.8 }}>
        <Story />
      </div>
    ),
  ],
};

export const Muted: Story = {
  args: {
    ...Idle.args,
    activeJobId: 'other-job-id',
  },
};

export const Completed: Story = {
  args: {
    ...Idle.args,
    task: { ...mockTask, status: 'completed' },
  },
};
```

---

### VIS-UI-TILE-002: Tile Color Variants {#vis-ui-tile-002}

Test all job colors from design-tokens.md:

| Color | Hex | Story |
|-------|-----|-------|
| Purple | #a855f7 | `Tile/Colors/Purple` |
| Violet | #8b5cf6 | `Tile/Colors/Violet` |
| Rose | #f43f5e | `Tile/Colors/Rose` |
| Red | #ef4444 | `Tile/Colors/Red` |
| Yellow | #eab308 | `Tile/Colors/Yellow` |
| Amber | #f59e0b | `Tile/Colors/Amber` |
| Orange | #f97316 | `Tile/Colors/Orange` |
| Teal | #14b8a6 | `Tile/Colors/Teal` |
| Green | #22c55e | `Tile/Colors/Green` |
| Emerald | #10b981 | `Tile/Colors/Emerald` |
| Lime | #84cc16 | `Tile/Colors/Lime` |
| Cyan | #06b6d4 | `Tile/Colors/Cyan` |
| Sky | #0ea5e9 | `Tile/Colors/Sky` |
| Blue | #3b82f6 | `Tile/Colors/Blue` |
| Indigo | #6366f1 | `Tile/Colors/Indigo` |
| Pink | #ec4899 | `Tile/Colors/Pink` |
| Fuchsia | #d946ef | `Tile/Colors/Fuchsia` |

```typescript
// Tile.stories.tsx (continued)
const colors = [
  { name: 'Purple', hex: '#a855f7' },
  { name: 'Violet', hex: '#8b5cf6' },
  { name: 'Rose', hex: '#f43f5e' },
  // ... all 17 colors
];

export const AllColors: Story = {
  render: () => (
    <div className="flex flex-wrap gap-4">
      {colors.map(({ name, hex }) => (
        <Tile
          key={name}
          assignment={mockAssignment}
          task={mockTask}
          job={{ ...mockJob, color: hex }}
          top={0}
        />
      ))}
    </div>
  ),
};
```

---

### VIS-UI-TILE-003: Tile Size Variants {#vis-ui-tile-003}

Test tiles of different durations:

| Duration | Height | Story |
|----------|--------|-------|
| 30 min | 40px | `Tile/Sizes/Short` |
| 60 min | 80px | `Tile/Sizes/Medium` |
| 120 min | 160px | `Tile/Sizes/Long` |
| 240 min | 320px | `Tile/Sizes/VeryLong` |

```typescript
export const Short: Story = {
  args: {
    ...Idle.args,
    task: { ...mockTask, estimatedDuration: 30 },
  },
};

export const Medium: Story = {
  args: {
    ...Idle.args,
    task: { ...mockTask, estimatedDuration: 60 },
  },
};

export const Long: Story = {
  args: {
    ...Idle.args,
    task: { ...mockTask, estimatedDuration: 120 },
  },
};

export const VeryLong: Story = {
  args: {
    ...Idle.args,
    task: { ...mockTask, estimatedDuration: 240 },
  },
};
```

---

## Station Column Visual Tests

> **Component:** [COMP-STATION-COLUMN](../component-api.md#stationcolumn)
> **State Machine:** [SM-VALID](../state-machines.md#sm-valid)

### VIS-UI-DROP-001: Drop Zone States {#vis-ui-drop-001}

| State | Ring Color | Background | Story |
|-------|------------|------------|-------|
| Normal | none | slate-900 | `StationColumn/Normal` |
| Valid | green-500 | green-500/10 | `StationColumn/ValidDrop` |
| Invalid | red-500 | red-500/10 | `StationColumn/InvalidDrop` |
| Warning | orange-500 | orange-500/10 | `StationColumn/WarningDrop` |
| Bypass | amber-500 | amber-500/10 | `StationColumn/BypassDrop` |

```typescript
// src/components/StationColumns/StationColumn.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { StationColumn } from './StationColumn';
import { mockStation } from '@/test/fixtures';

const meta: Meta<typeof StationColumn> = {
  title: 'Components/StationColumn',
  component: StationColumn,
  parameters: {
    chromatic: { viewports: [1280] },
  },
};

export default meta;
type Story = StoryObj<typeof StationColumn>;

export const Normal: Story = {
  args: {
    station: mockStation,
    startHour: 6,
    hoursToDisplay: 12,
  },
};

export const ValidDrop: Story = {
  args: {
    ...Normal.args,
    isValidDrop: true,
  },
};

export const InvalidDrop: Story = {
  args: {
    ...Normal.args,
    isInvalidDrop: true,
  },
};

export const WarningDrop: Story = {
  args: {
    ...Normal.args,
    isWarningDrop: true,
  },
};

export const BypassDrop: Story = {
  args: {
    ...Normal.args,
    showBypassWarning: true,
  },
};

export const Collapsed: Story = {
  args: {
    ...Normal.args,
    isCollapsed: true,
  },
};
```

---

### VIS-UI-DROP-002: Quick Placement Mode {#vis-ui-drop-002}

| State | Visual | Story |
|-------|--------|-------|
| Available | Green highlight | `StationColumn/QuickPlacement/Available` |
| Unavailable | Dimmed (opacity-50) | `StationColumn/QuickPlacement/Unavailable` |
| With Indicator | Glowing line | `StationColumn/QuickPlacement/WithIndicator` |

```typescript
export const QuickPlacementAvailable: Story = {
  args: {
    ...Normal.args,
    isQuickPlacementMode: true,
    hasAvailableTask: true,
  },
};

export const QuickPlacementUnavailable: Story = {
  args: {
    ...Normal.args,
    isQuickPlacementMode: true,
    hasAvailableTask: false,
  },
};

export const QuickPlacementWithIndicator: Story = {
  args: {
    ...Normal.args,
    isQuickPlacementMode: true,
    hasAvailableTask: true,
    placementIndicatorY: 200,
  },
};
```

---

### VIS-UI-DROP-003: Unavailability Overlay {#vis-ui-drop-003}

Test diagonal stripe pattern for non-operating hours:

```typescript
export const WithUnavailability: Story = {
  args: {
    ...Normal.args,
    station: {
      ...mockStation,
      operatingHours: {
        start: '08:00',
        end: '18:00',
      },
    },
  },
};
```

---

## Tile Ghost Visual Tests

> **AC:** [AC-UI-VISUAL-002.1-3](../acceptance-criteria.md#ac-ui-visual-0021)

### VIS-UI-GHOST-001: Ghost Placeholder {#vis-ui-ghost-001}

| State | Visual | Story |
|-------|--------|-------|
| Visible | Dashed border, faded | `TileGhost/Visible` |

```typescript
// src/components/Tile/TileGhost.stories.tsx
export const Visible: Story = {
  args: {
    width: 200,
    height: 80,
    top: 100,
  },
};
```

Expected visual:
- Border: dashed, 2px, job color at 50% opacity
- Background: transparent
- Content: None (empty placeholder)

---

## Drag Preview Visual Tests

> **Component:** [COMP-DRAG-PREVIEW](../component-api.md#dragpreview)
> **AC:** [AC-UI-VISUAL-003.1-3](../acceptance-criteria.md#ac-ui-visual-0031)

### VIS-UI-PREVIEW-001: Drag Preview {#vis-ui-preview-001}

```typescript
// src/components/DragPreview/DragPreview.stories.tsx
export const Default: Story = {
  args: {
    task: mockTask,
    job: mockJob,
    x: 300,
    y: 200,
    grabOffset: { x: 20, y: 15 },
  },
  parameters: {
    chromatic: {
      // Capture at fixed position
      delay: 0,
    },
  },
};
```

Expected visual:
- Same as Tile component
- Opacity: 0.8
- Position: fixed (follows cursor)

---

## Placement Indicator Visual Tests

> **Component:** [COMP-PLACEMENT-INDICATOR](../component-api.md#placementindicator)

### VIS-UI-INDICATOR-001: Placement Line {#vis-ui-indicator-001}

```typescript
// src/components/PlacementIndicator/PlacementIndicator.stories.tsx
export const Visible: Story = {
  args: {
    y: 200,
    isVisible: true,
  },
};

export const Hidden: Story = {
  args: {
    y: 200,
    isVisible: false,
  },
};
```

Expected visual:
- White horizontal line
- Box shadow: `0 0 12px rgba(255, 255, 255, 0.8)`
- Full width of column

---

## Job Details Panel Visual Tests

> **Component:** [COMP-JOB-DETAILS-PANEL](../component-api.md#jobdetailspanel)

### VIS-UI-PANEL-001: Panel States {#vis-ui-panel-001}

| State | Visual | Story |
|-------|--------|-------|
| No Job | Empty state message | `JobDetailsPanel/Empty` |
| Job Selected | Full details | `JobDetailsPanel/WithJob` |
| With Tasks | Task list | `JobDetailsPanel/WithTasks` |
| Scrolled | Overflow visible | `JobDetailsPanel/Scrolled` |

```typescript
// src/components/JobDetailsPanel/JobDetailsPanel.stories.tsx
export const Empty: Story = {
  args: {
    job: null,
    tasks: [],
    assignments: [],
    stations: [],
  },
};

export const WithJob: Story = {
  args: {
    job: mockJob,
    tasks: mockTasks,
    assignments: mockAssignments,
    stations: mockStations,
  },
};

export const WithTasks: Story = {
  args: {
    ...WithJob.args,
    tasks: mockTasksLong, // 10+ tasks
  },
};
```

---

## Task Tile (Sidebar) Visual Tests

> **Component:** [COMP-TASK-TILE](../component-api.md#tasktile)

### VIS-UI-SIDEBAR-001: Sidebar Tile States {#vis-ui-sidebar-001}

| State | Visual | Draggable | Story |
|-------|--------|-----------|-------|
| Unscheduled | Full color, cursor-grab | Yes | `TaskTile/Unscheduled` |
| Scheduled | Dark placeholder | No | `TaskTile/Scheduled` |
| Completed | Green check icon | No | `TaskTile/Completed` |
| Active | Highlighted | Yes | `TaskTile/Active` |

```typescript
// src/components/JobDetailsPanel/TaskTile.stories.tsx
export const Unscheduled: Story = {
  args: {
    task: mockTask,
    job: mockJob,
    assignment: null,
    station: mockStation,
  },
};

export const Scheduled: Story = {
  args: {
    ...Unscheduled.args,
    assignment: mockAssignment,
  },
};

export const Completed: Story = {
  args: {
    ...Unscheduled.args,
    task: { ...mockTask, status: 'completed' },
  },
};

export const Active: Story = {
  args: {
    ...Unscheduled.args,
    isActive: true,
  },
};
```

---

## Job Card Visual Tests

> **Component:** [COMP-JOB-CARD](../component-api.md#jobcard)

### VIS-UI-JOBCARD-001: Job Card States {#vis-ui-jobcard-001}

| State | Visual | Story |
|-------|--------|-------|
| Normal | Default | `JobCard/Normal` |
| Selected | Highlighted border | `JobCard/Selected` |
| Late | Red problem indicator | `JobCard/Late` |
| Conflict | Orange indicator | `JobCard/Conflict` |

```typescript
// src/components/JobsList/JobCard.stories.tsx
export const Normal: Story = {
  args: {
    id: 'job-001',
    reference: '12345',
    client: 'Acme Corp',
    description: 'Annual Report',
    taskCount: 5,
    completedTaskCount: 2,
    deadline: '2024-01-20',
  },
};

export const Selected: Story = {
  args: {
    ...Normal.args,
    isSelected: true,
  },
};

export const Late: Story = {
  args: {
    ...Normal.args,
    problemType: 'late',
  },
};

export const Conflict: Story = {
  args: {
    ...Normal.args,
    problemType: 'conflict',
  },
};
```

---

## Timeline Visual Tests

> **Component:** [COMP-TIMELINE-COLUMN](../component-api.md#timelinecolumn)

### VIS-UI-TIMELINE-001: Timeline Elements {#vis-ui-timeline-001}

| Element | Story |
|---------|-------|
| Hour markers | `TimelineColumn/Default` |
| Now line visible | `TimelineColumn/WithNowLine` |
| 24-hour format | `TimelineColumn/24Hours` |

```typescript
// src/components/TimelineColumn/TimelineColumn.stories.tsx
export const Default: Story = {
  args: {
    startHour: 6,
    hourCount: 12,
    showNowLine: false,
  },
};

export const WithNowLine: Story = {
  args: {
    ...Default.args,
    showNowLine: true,
    currentTime: new Date('2024-01-15T10:30:00'),
  },
};

export const Full24Hours: Story = {
  args: {
    startHour: 0,
    hourCount: 24,
    showNowLine: true,
  },
};
```

---

## Full Page Visual Tests

### VIS-UI-PAGE-001: Scheduler Page {#vis-ui-page-001}

| State | Story |
|-------|-------|
| Empty (no data) | `Pages/Scheduler/Empty` |
| With data | `Pages/Scheduler/WithData` |
| Job selected | `Pages/Scheduler/JobSelected` |
| During drag | `Pages/Scheduler/Dragging` |
| Quick placement mode | `Pages/Scheduler/QuickPlacement` |

```typescript
// src/pages/Scheduler.stories.tsx
export const Empty: Story = {
  parameters: {
    msw: {
      handlers: [
        rest.get('/api/jobs', (_, res, ctx) => res(ctx.json([]))),
        rest.get('/api/assignments', (_, res, ctx) => res(ctx.json([]))),
      ],
    },
  },
};

export const WithData: Story = {
  parameters: {
    msw: {
      handlers: [
        rest.get('/api/jobs', (_, res, ctx) => res(ctx.json(mockJobs))),
        rest.get('/api/assignments', (_, res, ctx) => res(ctx.json(mockAssignments))),
      ],
    },
  },
};

export const JobSelected: Story = {
  ...WithData,
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    await userEvent.click(canvas.getByTestId('job-card-job-001'));
  },
};
```

---

## Responsive Visual Tests

### VIS-UI-RESP-001: Breakpoint Snapshots {#vis-ui-resp-001}

Capture at NFR-compliant breakpoints:

| Breakpoint | Width | Priority |
|------------|-------|----------|
| Minimum | 1280px | High |
| Recommended | 1920px | High |

```typescript
// Global chromatic config
export const parameters = {
  chromatic: {
    viewports: [1280, 1920],
  },
};
```

---

## Dark Mode Visual Tests

### VIS-UI-DARK-001: Dark Mode States {#vis-ui-dark-001}

All components should be tested in dark mode (default):

```typescript
// .storybook/preview.ts
export const parameters = {
  backgrounds: {
    default: 'dark',
    values: [
      { name: 'dark', value: '#0f172a' }, // slate-900
    ],
  },
};
```

---

## Chromatic Configuration

```typescript
// .storybook/main.ts
export default {
  stories: ['../src/**/*.stories.@(ts|tsx)'],
  addons: [
    '@storybook/addon-essentials',
    '@chromatic-com/storybook',
  ],
  framework: '@storybook/react-vite',
};
```

```json
// package.json
{
  "scripts": {
    "chromatic": "chromatic --project-token=CHROMATIC_PROJECT_TOKEN",
    "storybook": "storybook dev -p 6006",
    "build-storybook": "storybook build"
  }
}
```

---

## Visual Test Coverage Matrix

| Component | States | Stories | Coverage |
|-----------|--------|---------|----------|
| Tile | 6 + colors + sizes | 23 | 100% |
| StationColumn | 8 | 8 | 100% |
| TileGhost | 1 | 1 | 100% |
| DragPreview | 1 | 1 | 100% |
| PlacementIndicator | 2 | 2 | 100% |
| JobDetailsPanel | 4 | 4 | 100% |
| TaskTile | 4 | 4 | 100% |
| JobCard | 4 | 4 | 100% |
| TimelineColumn | 3 | 3 | 100% |
| **Total** | **33+** | **50** | **100%** |

---

## Related Documents

- [Component API](../component-api.md)
- [State Machines](../state-machines.md)
- [Design Tokens](../design-tokens.md)
- [E2E Tests](e2e-tests.md)

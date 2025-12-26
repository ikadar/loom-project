# Quick User Stories - Flux MVP

Meeting notes: 2025-01-15, Product Owner + Dev Lead

## Must Have (v1.0)

### Scheduling Grid
- Drag tile from left panel to grid → creates assignment
- Drag tile on grid → reschedule
- Tiles can't overlap on capacity-1 stations
- If I insert between two tiles, push the rest down
- Show setup vs run time differently (lighter/darker)
- 30-min snap grid
- Red halo if precedence broken
- Alt+drag bypasses precedence check

### Job List (left panel)
- See all jobs
- Filter by text
- Click job → show its tasks
- Drag unscheduled task to grid
- "Recall" button on scheduled tasks → unassigns

### Late Jobs (right panel)
- List jobs that won't make deadline
- Show how many days late

### Similarity Circles
- Between consecutive tiles on same station
- Filled = criterion matches
- Empty = doesn't match
- For now just show visually, no optimization

### Station Unavailability
- If task overlaps station downtime → stretch the tile
- Show hatched pattern during unavailable time

## Nice to Have (v1.1)
- Keyboard navigation
- Undo/redo
- Quick placement mode (click instead of drag)
- Off-screen tile indicators

## Not Now
- Schedule branching
- Auto-optimization
- French holidays calendar
- Real-time collaboration

---

## Technical Notes from Dev

Backend: Symfony/PHP for CRUD, Node.js for validation
Frontend: React + TypeScript
State: TanStack Query + Zustand
Drag: dnd-kit

Performance targets:
- Drag feedback: <10ms
- 100 tiles render: <100ms
- Initial load: <2s

API pattern: REST for CRUD, WebSocket for real-time updates (future)

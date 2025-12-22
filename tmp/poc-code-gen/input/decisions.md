---
# decisions.md - Loom Decision Log
# This file persists interview answers across derivation runs.
# DO NOT delete - answers will be asked again!
created: "2025-12-22"
last_updated: "2025-12-22"
total_decisions: 48
---

## Entity Decisions

### Station

- **AMB-ENT-001: Deletion behavior**
  - Q: What happens to assigned tasks when a station is deleted?
  - A: Station cannot be deleted (block deletion)
  - Decided: 2025-12-22 by user

- **AMB-ENT-002: Operating hours format**
  - Q: What is the format for station operating hours?
  - A: Per-weekday schedule (different hours for each day)
  - Decided: 2025-12-22 by user

- **AMB-ENT-003: Name uniqueness**
  - Q: Is station name required to be unique?
  - A: Yes, globally unique
  - Decided: 2025-12-22 by user

- **AMB-ENT-004: Default capacity**
  - Q: What is the default capacity for a new station?
  - A: 1 (single task)
  - Decided: 2025-12-22 by user

- **AMB-ENT-006: Category relationship**
  - Q: Can a station belong to multiple categories or only one?
  - A: One category only
  - Decided: 2025-12-22 by user

### Job

- **AMB-ENT-008: Paper status enum**
  - Q: What are the exact values for paper_status enum?
  - A: in_stock / to_order / ordered
  - Decided: 2025-12-22 by user

- **AMB-ENT-009: BAT status enum**
  - Q: What are the exact values for bat_status enum?
  - A: pending / approved
  - Decided: 2025-12-22 by user

- **AMB-ENT-010: Plates status enum**
  - Q: What are the exact values for plates_status enum?
  - A: not_ready / ready
  - Decided: 2025-12-22 by user

- **AMB-ENT-011: Late definition**
  - Q: When is a job considered 'late'?
  - A: Final task ends after deadline
  - Decided: 2025-12-22 by user

- **AMB-ENT-012: Job deletion**
  - Q: Can a job be deleted?
  - A: Yes, with confirmation (unschedules all tasks)
  - Decided: 2025-12-22 by user

- **AMB-ENT-013: Deadline precision**
  - Q: Is job deadline date-only or datetime?
  - A: Date only (end of day implied)
  - Decided: 2025-12-22 by user

### Task

- **AMB-ENT-015: Sequence management**
  - Q: How is task sequence order managed within a job?
  - A: Auto-increment on creation
  - Decided: 2025-12-22 by user

- **AMB-ENT-016: Task job assignment**
  - Q: Can a task be moved to a different job after creation?
  - A: No
  - Decided: 2025-12-22 by user

- **AMB-ENT-018: Task station compatibility**
  - Q: Can a task be scheduled on any station or only compatible ones?
  - A: Only matching category
  - Decided: 2025-12-22 by user

- **Task time editing**
  - Q: Can task times (setup/run) be edited after creation?
  - A: Yes, anytime (triggers recalculation)
  - Decided: 2025-12-22 by user

### StationCategory

- **AMB-ENT-020: Category deletion**
  - Q: Can a StationCategory be deleted if stations reference it?
  - A: No, blocked
  - Decided: 2025-12-22 by user

### OutsourcedStation

- **AMB-ENT-022: Lead time unit**
  - Q: For outsourced stations, what is the unit for task duration?
  - A: Working days
  - Decided: 2025-12-22 by user

- **AMB-ENT-023: Outsourced display**
  - Q: How are outsourced stations represented on the grid?
  - A: Shown as special column (measured in days)
  - Decided: 2025-12-22 by user

## Operation Decisions

### Schedule Task

- **AMB-OP-001: Overlap behavior**
  - Q: When dropping a task on an already occupied time slot, what happens?
  - A: Push tiles down
  - Decided: 2025-12-22 by user

- **AMB-OP-002: Snap grid mandatory**
  - Q: Is the 30-minute snap grid mandatory or optional?
  - A: Mandatory
  - Decided: 2025-12-22 by user

- **AMB-OP-004: Schedule regardless of paper status**
  - Q: Can tasks be scheduled even if paper_status is 'to_order'?
  - A: Yes, allowed
  - Decided: 2025-12-22 by user

- **AMB-OP-005: Push amount calculation**
  - Q: When inserting a task and pushing tiles down, how is push amount calculated?
  - A: By inserted task duration (setup + run)
  - Decided: 2025-12-22 by user

- **AMB-OP-006: Push overflow handling**
  - Q: When pushing tiles would cause them to exceed station operating hours, what happens?
  - A: Push to next operation hour (may be next day)
  - Decided: 2025-12-22 by user

- **AMB-OP-007: Reschedule behavior**
  - Q: When dragging an already-scheduled task to a new time on the grid, does it push other tiles?
  - A: Yes, same as insert
  - Decided: 2025-12-22 by user

- **AMB-OP-008: Recall effect**
  - Q: When recalling a task (unscheduling), what happens to tasks that were pushed down for it?
  - A: Nothing (gaps remain)
  - Decided: 2025-12-22 by user

- **AMB-OP-009: Red halo persistence**
  - Q: Does the red halo for precedence violation persist until fixed?
  - A: Yes, persists
  - Decided: 2025-12-22 by user

- **AMB-OP-010: Precedence violation logging**
  - Q: Are precedence violations logged for audit/tracking?
  - A: No logging (visual only)
  - Decided: 2025-12-22 by user

- **AMB-OP-011: Station creation validation**
  - Q: What validations apply when creating a station?
  - A: Name required + unique
  - Decided: 2025-12-22 by user

- **AMB-OP-013: DSL format**
  - Q: What is the exact DSL format for task creation?
  - A: [Station] setup+run "note" for normal, ST [Partner] desc #days for outsourced
  - Decided: 2025-12-22 by user

- **AMB-OP-016: Concurrent edits**
  - Q: What if two users schedule different tasks to the same slot simultaneously?
  - A: Last write wins
  - Decided: 2025-12-22 by user

- **AMB-OP-018: Stretch calculation**
  - Q: When a task overlaps station downtime, how is the stretched duration calculated?
  - A: Split around downtime (task pauses, resumes after)
  - Decided: 2025-12-22 by user

- **AMB-OP-019: Similarity criteria**
  - Q: What are the exact similarity criteria for the circles between consecutive tasks?
  - A: Paper type, size, ink (3 criteria)
  - Decided: 2025-12-22 by user

- **AMB-OP-020: Days late calculation**
  - Q: How is 'days late' calculated for late jobs?
  - A: Working days only (Mon-Sat)
  - Decided: 2025-12-22 by user

## UI Decisions

### Scheduling Grid

- **AMB-UI-001: Time scale**
  - Q: What is the time scale for the scheduling grid rows?
  - A: 30-minute rows
  - Decided: 2025-12-22 by user

- **AMB-UI-002: Visible range**
  - Q: How many hours/days are visible on the grid at once?
  - A: 1 week (continuous vertical scroll)
  - Decided: 2025-12-22 by user

- **AMB-UI-003: Navigation**
  - Q: How does user navigate to different dates on the grid?
  - A: Continuous vertical scroll
  - Decided: 2025-12-22 by user

- **AMB-UI-004: Current time indicator**
  - Q: Is current time shown on the grid?
  - A: Yes, red line
  - Decided: 2025-12-22 by user

### Task Tile

- **AMB-UI-005: Tile content**
  - Q: What information is displayed on each task tile?
  - A: Job name + task description
  - Decided: 2025-12-22 by user

- **AMB-UI-006: Setup/run colors**
  - Q: Which color represents setup time vs run time on tiles?
  - A: Lighter = setup, Darker = run
  - Decided: 2025-12-22 by user

- **AMB-UI-007: Tile height**
  - Q: Is tile height proportional to task duration?
  - A: Yes, proportional
  - Decided: 2025-12-22 by user

### Left Panel

- **AMB-UI-009: Filter fields**
  - Q: What fields does the job list text filter search?
  - A: Job name, client, and task descriptions
  - Decided: 2025-12-22 by user

### Right Panel

- **AMB-UI-010/011: Right panel removal**
  - Q: What information is shown in the Late Jobs right panel?
  - A: RIGHT PANEL REMOVED FROM SCOPE - outdated concept
  - Decided: 2025-12-22 by user

### Drag and Drop

- **AMB-UI-012: Drag feedback**
  - Q: What visual feedback is shown during drag-and-drop?
  - A: Ghost tile + drop zone highlighting
  - Decided: 2025-12-22 by user

### General

- **AMB-UI-016: Responsiveness**
  - Q: Is the scheduling grid desktop-only or responsive?
  - A: Desktop only (1200px+)
  - Decided: 2025-12-22 by user

- **AMB-UI-018: Error display**
  - Q: How are validation errors displayed to the user?
  - A: Toast notification
  - Decided: 2025-12-22 by user

### Late Job Navigation

- **AMB-UI-011: Late job click action**
  - Q: Can user click a late job in right panel to navigate to its tasks?
  - A: Yes, selects job in left panel (Note: right panel removed, but late detection remains)
  - Decided: 2025-12-22 by user

## Defaults Accepted

The following minor decisions use suggested defaults:

| ID | Question | Default | Accepted |
|----|----------|---------|----------|
| String lengths | Max station name | 100 chars | 2025-12-22 |
| String lengths | Max job title | 200 chars | 2025-12-22 |
| String lengths | Max client name | 100 chars | 2025-12-22 |
| String lengths | Max task description | 500 chars | 2025-12-22 |
| Min duration | Minimum task duration | 30 minutes | 2025-12-22 |

## Scope Changes

### Removed Features

| Feature | Reason | Decided |
|---------|--------|---------|
| Right Panel (Late Jobs) | Outdated concept per user | 2025-12-22 |

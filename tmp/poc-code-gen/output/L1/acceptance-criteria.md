---
id: L1-AC
status: draft
derived-from:
  - "project-brief.md"
  - "quick-stories.md"
derived-at: "2025-12-22T10:00:00Z"
derived-by: "loom-derive"
completeness-analysis:
  entities-analyzed: 5
  operations-analyzed: 12
  ambiguities-found: 65
  ambiguities-resolved: 48
decisions:
  from-existing: 0
  from-this-session: 48
  total: 48
---

# Acceptance Criteria - Flux Print Shop Scheduling System

## 1. Station Management

### AC-STATION-001 – Create Station

**Given** a user is on the station management interface
**When** they submit a new station with a unique name
**Then** the station is created with:
- The provided name (required, max 100 chars, globally unique)
- Default capacity of 1
- Empty operating hours (to be configured separately)

**Error Cases:**
- Name already exists → Error: `STATION_NAME_DUPLICATE`
- Name empty or whitespace-only → Error: `STATION_NAME_REQUIRED`
- Name exceeds 100 characters → Error: `STATION_NAME_TOO_LONG`

**Traceability:**
- Input: project-brief.md § "Stációk kezelése"
- Interview: AMB-ENT-003, AMB-OP-011

---

### AC-STATION-002 – Configure Station Operating Hours

**Given** a station exists
**When** a user sets operating hours
**Then** they can specify different start/end times for each weekday (Monday-Saturday)

**Notes:**
- Per-weekday schedule (e.g., Mon-Fri 06:00-22:00, Sat 06:00-14:00)
- Sunday is non-working (no operating hours)

**Traceability:**
- Input: project-brief.md § "Működési idő"
- Interview: AMB-ENT-002, AMB-ENT-005

---

### AC-STATION-003 – Assign Station to Category

**Given** a station exists and categories exist
**When** a user assigns the station to a category
**Then** the station belongs to exactly one category

**Constraints:**
- Station can belong to only one category at a time
- Changing category is allowed

**Traceability:**
- Input: project-brief.md § "Kategória"
- Interview: AMB-ENT-006

---

### AC-STATION-004 – Delete Station Blocked

**Given** a station exists
**When** a user attempts to delete the station
**Then** the deletion is blocked with error message

**Rationale:** Stations cannot be deleted in this system. Historical data integrity and traceability require stations to persist.

**Error Cases:**
- Any delete attempt → Error: `STATION_DELETE_NOT_ALLOWED`

**Traceability:**
- Interview: AMB-ENT-001

---

### AC-STATION-005 – Outsourced Station Display

**Given** an outsourced station (subcontractor) is created
**When** viewing the scheduling grid
**Then** the outsourced station appears as a special column with:
- Tasks measured in working days (not hours/minutes)
- No capacity limit (multiple tasks can run in parallel)

**Traceability:**
- Input: project-brief.md § "Kiszervezett munkák"
- Interview: AMB-ENT-022, AMB-ENT-023

---

## 2. Station Category Management

### AC-CATEGORY-001 – Create Station Category

**Given** a user is on category management
**When** they create a category with a unique name
**Then** the category is created and available for station assignment

**Traceability:**
- Input: project-brief.md § "Kategória"

---

### AC-CATEGORY-002 – Delete Category Blocked When In Use

**Given** a category has one or more stations assigned
**When** a user attempts to delete the category
**Then** the deletion is blocked

**Error Cases:**
- Category has stations → Error: `CATEGORY_HAS_STATIONS`

**Traceability:**
- Interview: AMB-ENT-020

---

## 3. Job Management

### AC-JOB-001 – Create Job

**Given** a user is on job management
**When** they create a job with required fields
**Then** the job is created with:
- Job name/title (required, max 200 chars)
- Client name (required, max 100 chars)
- Deadline (date only, end of day implied)
- paper_status: defaults to `to_order`
- bat_status: defaults to `pending`
- plates_status: defaults to `not_ready`

**Traceability:**
- Input: project-brief.md § "Munka (Job)"
- Interview: AMB-ENT-008, AMB-ENT-009, AMB-ENT-010, AMB-ENT-013

---

### AC-JOB-002 – Update Job Status Fields

**Given** a job exists
**When** a user updates paper_status, bat_status, or plates_status
**Then** the status is updated to the new valid value

**Valid Values:**
- paper_status: `in_stock`, `to_order`, `ordered`
- bat_status: `pending`, `approved`
- plates_status: `not_ready`, `ready`

**Traceability:**
- Input: project-brief.md § "Papír státusz", "BAT státusz", "Lemezek státusza"
- Interview: AMB-ENT-008, AMB-ENT-009, AMB-ENT-010

---

### AC-JOB-003 – Delete Job With Confirmation

**Given** a job exists
**When** a user requests deletion and confirms the action
**Then**:
- All tasks belonging to the job are unscheduled (removed from grid)
- The job and all its tasks are deleted

**Traceability:**
- Interview: AMB-ENT-012

---

### AC-JOB-004 – Job Late Detection

**Given** a job has scheduled tasks
**When** the final task's end time exceeds the job deadline
**Then** the job is marked as "late"

**Calculation:**
- Late status = final_task_end_datetime > deadline_date @ 23:59:59
- Days late = working days (Mon-Sat) between deadline and final task end

**Traceability:**
- Input: project-brief.md § "Határidő"
- Interview: AMB-ENT-011, AMB-OP-020

---

## 4. Task Management

### AC-TASK-001 – Create Task via DSL

**Given** a job exists
**When** a user enters task definition using DSL notation
**Then** the task is created with parsed values

**DSL Format:**
- Normal task: `[StationName] setup+run "optional note"`
  - Example: `[Komori] 20+40 "vízjeles műnyomó"`
  - Parsed: station=Komori, setup_time=20min, run_time=40min, description="vízjeles műnyomó"

- Outsourced task: `ST [PartnerName] description #days`
  - Example: `ST [Clément] Lakkozás 2JO`
  - Parsed: station=Clément (outsourced), description="Lakkozás", duration=2 working days

**Constraints:**
- Sequence order auto-increments (1, 2, 3...)
- Minimum duration: 30 minutes for normal tasks
- Task description max: 500 chars

**Error Cases:**
- Station not found → Error: `STATION_NOT_FOUND`
- Invalid DSL syntax → Error: `INVALID_TASK_DSL`
- Duration < 30 minutes → Error: `TASK_DURATION_TOO_SHORT`

**Traceability:**
- Input: project-brief.md § "DSL-lel megadni"
- Interview: AMB-ENT-015, AMB-OP-013

---

### AC-TASK-002 – Edit Task Times

**Given** a task exists (scheduled or unscheduled)
**When** a user edits setup_time or run_time
**Then** the times are updated and any schedule conflicts are recalculated

**Constraints:**
- Times editable anytime (even if scheduled)
- Combined duration must be >= 30 minutes

**Traceability:**
- Interview: AMB-ENT-016 (related)

---

### AC-TASK-003 – Task Belongs to Job Permanently

**Given** a task is created for a job
**When** any operation is attempted
**Then** the task cannot be moved to a different job

**Traceability:**
- Interview: AMB-ENT-016

---

## 5. Scheduling Grid

### AC-GRID-001 – Grid Layout and Scale

**Given** the scheduling grid is displayed
**When** viewing the grid
**Then**:
- Horizontal axis: station columns (one column per station)
- Vertical axis: time (scrollable, continuous)
- Row height: 30 minutes
- View: desktop only (1200px+ width)

**Traceability:**
- Input: project-brief.md § "Az ütemező nézet"
- Interview: AMB-UI-001, AMB-UI-016

---

### AC-GRID-002 – Continuous Vertical Scroll

**Given** the scheduling grid is displayed
**When** user scrolls vertically
**Then** time scrolls continuously (not week-by-week)

**Display:**
- Current time shown as red horizontal line

**Traceability:**
- Interview: AMB-UI-003, AMB-UI-004

---

### AC-GRID-003 – Station Unavailability Display

**Given** a station has non-operating hours
**When** viewing the grid for that station column
**Then** unavailable time blocks are shown with hatched pattern

**Traceability:**
- Input: quick-stories.md § "Station Unavailability"

---

## 6. Scheduling Operations

### AC-SCHEDULE-001 – Schedule Task from Left Panel

**Given** a job is selected in the left panel and has unscheduled tasks
**When** user drags an unscheduled task to the grid
**Then**:
- Task is placed at the dropped position
- Position snaps to 30-minute boundary (mandatory)
- Task is marked as scheduled

**Visual Feedback During Drag:**
- Ghost tile follows cursor
- Valid drop zones highlighted
- Invalid zones (wrong category) indicated

**Constraints:**
- Task can only be dropped on stations matching its category

**Error Cases:**
- Drop on incompatible station → Drop rejected, task returns to panel

**Traceability:**
- Input: project-brief.md § "drag-and-drop", quick-stories.md § "Scheduling Grid"
- Interview: AMB-OP-002, AMB-ENT-018, AMB-UI-012

---

### AC-SCHEDULE-002 – Insert Task Pushes Tiles Down

**Given** a station has existing scheduled tasks
**When** user drops a task on an occupied time slot
**Then**:
- New task is inserted at that position
- All subsequent tasks on that station are pushed down
- Push amount = inserted task's total duration (setup + run)

**Traceability:**
- Input: quick-stories.md § "If I insert between two tiles, push the rest down"
- Interview: AMB-OP-001, AMB-OP-005

---

### AC-SCHEDULE-003 – Push Respects Operating Hours

**Given** pushing tiles down would cause a task to exceed operating hours
**When** the push cascade is calculated
**Then** the task moves to the next available operating time (may be next day)

**Example:**
- Task at 21:00 pushed by 2 hours
- Station closes at 22:00
- Task moves to next morning (e.g., 06:00 next day)

**Traceability:**
- Interview: AMB-OP-006

---

### AC-SCHEDULE-004 – Reschedule Task by Dragging

**Given** a task is already scheduled on the grid
**When** user drags it to a new position
**Then**:
- Same behavior as insert (pushes tiles if needed)
- Original position becomes empty (gap may remain)

**Traceability:**
- Interview: AMB-OP-007

---

### AC-SCHEDULE-005 – Task Stretches Over Station Downtime

**Given** a scheduled task overlaps with station unavailable time
**When** calculating task display
**Then**:
- Task is split around the downtime
- Total work time preserved (runs before and after downtime)
- Visual indication of split

**Example:**
- Task duration: 2 hours
- Scheduled at 17:00
- Station closed 18:00-22:00
- Display: 17:00-18:00 (1 hour) + 22:00-23:00 (1 hour)

**Traceability:**
- Input: project-brief.md § "ha egy feladat átlóg a gép szünetébe"
- Interview: AMB-OP-018

---

### AC-SCHEDULE-006 – Recall Task (Unschedule)

**Given** a task is scheduled on the grid
**When** user clicks "Recall" button on the task
**Then**:
- Task is removed from grid (becomes unscheduled)
- Other tasks remain in their positions (gaps allowed)

**Traceability:**
- Input: quick-stories.md § "Recall button"
- Interview: AMB-OP-008

---

### AC-SCHEDULE-007 – Scheduling Allowed Regardless of Paper Status

**Given** a job has paper_status = `to_order` (not in stock)
**When** user schedules a task from that job
**Then** scheduling is allowed (no blocking based on paper status)

**Traceability:**
- Interview: AMB-OP-004

---

## 7. Precedence Rules

### AC-PRECEDENCE-001 – Precedence Violation Detection

**Given** tasks have sequence order within a job (1, 2, 3...)
**When** a task is scheduled before a predecessor (lower sequence task)
**Then**:
- Red halo displayed around the violating tile
- Halo persists until violation is fixed

**Traceability:**
- Input: project-brief.md § "Precedencia sérül", quick-stories.md § "Red halo if precedence broken"
- Interview: AMB-OP-009

---

### AC-PRECEDENCE-002 – Override Precedence with Alt+Drag

**Given** user is dragging a task
**When** user holds Alt key during drop
**Then**:
- Precedence check is bypassed
- Task placed at dropped position regardless of predecessor timing
- Red halo indicates the violation (persists)

**Note:** No audit logging of precedence overrides

**Traceability:**
- Input: project-brief.md § "Alt+húzás → kényszeríti a helyet"
- Interview: AMB-OP-010

---

## 8. Task Tile Display

### AC-TILE-001 – Tile Visual Display

**Given** a task is scheduled on the grid
**When** viewing the tile
**Then**:
- Tile height proportional to duration
- Content shows: job name + task description
- Setup time shown in lighter shade
- Run time shown in darker shade

**Traceability:**
- Input: quick-stories.md § "Show setup vs run time differently (lighter/darker)"
- Interview: AMB-UI-005, AMB-UI-006, AMB-UI-007

---

### AC-TILE-002 – Similarity Indicators

**Given** two consecutive tasks are on the same station
**When** viewing the tiles
**Then** similarity circles are displayed between them:
- 3 circles representing: paper type, paper size, ink configuration
- Filled circle (●) = criterion matches
- Empty circle (○) = criterion doesn't match

**Traceability:**
- Input: project-brief.md § "Hasonlósági jelzők", quick-stories.md § "Similarity Circles"
- Interview: AMB-OP-019

---

## 9. Left Panel (Job List)

### AC-PANEL-001 – Job List Display

**Given** the left panel is visible
**When** viewing the panel
**Then** all jobs are listed

**Traceability:**
- Input: project-brief.md § "Bal panel: Munkák listája"

---

### AC-PANEL-002 – Job List Text Filter

**Given** the left panel is visible
**When** user types in the filter input
**Then** jobs are filtered matching text against:
- Job name
- Client name
- Task descriptions

**Traceability:**
- Input: quick-stories.md § "Filter by text"
- Interview: AMB-UI-009

---

### AC-PANEL-003 – Select Job to Show Tasks

**Given** jobs are listed in left panel
**When** user clicks a job
**Then** the job's tasks are displayed below

**Traceability:**
- Input: quick-stories.md § "Click job → show its tasks"

---

## 10. Error Handling

### AC-ERROR-001 – Validation Error Display

**Given** a validation error occurs
**When** the error is returned
**Then** a toast notification is displayed with the error message

**Traceability:**
- Interview: AMB-UI-018

---

## 11. Concurrency

### AC-CONCURRENCY-001 – Simultaneous Edits (Last Write Wins)

**Given** two users are viewing the same schedule
**When** both make changes at the same time
**Then** the last save overwrites previous changes (no conflict detection in MVP)

**Traceability:**
- Interview: AMB-OP-016

---

## 12. Performance Requirements

### AC-PERF-001 – Drag Feedback Latency

**Given** user is dragging a task
**When** moving the mouse
**Then** visual feedback updates in < 10ms

**Traceability:**
- Input: project-brief.md § "Gyors feedback"

---

### AC-PERF-002 – Grid Render Performance

**Given** the grid has 100 visible tiles
**When** rendering the grid
**Then** render completes in < 100ms

**Traceability:**
- Input: project-brief.md § "100 tile renderelése"

---

### AC-PERF-003 – Initial Load Time

**Given** user navigates to the scheduling view
**When** the page loads
**Then** initial load completes in < 2 seconds

**Traceability:**
- Input: quick-stories.md § "Initial load: <2s"

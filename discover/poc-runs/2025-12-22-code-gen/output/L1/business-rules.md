---
id: L1-BR
status: draft
derived-from:
  - "project-brief.md"
  - "quick-stories.md"
derived-at: "2025-12-22T10:00:00Z"
derived-by: "loom-derive"
---

# Business Rules - Flux Print Shop Scheduling System

## 1. Entity Constraints

### BR-ENT-001 – Station Name Uniqueness

**Rule:** Station names must be globally unique across the entire system.

**Invariant:** `∀ s1, s2 ∈ Stations: s1.id ≠ s2.id → s1.name ≠ s2.name`

**Enforcement:**
- Check point: Station creation, Station update (name change)
- Violation: Block operation
- Error: `STATION_NAME_DUPLICATE`

**Source:**
- Interview: AMB-ENT-003

---

### BR-ENT-002 – Station Cannot Be Deleted

**Rule:** Stations cannot be deleted from the system.

**Invariant:** `∀ s ∈ Stations: delete(s) = BLOCKED`

**Enforcement:**
- Check point: Any delete attempt
- Violation: Block operation
- Error: `STATION_DELETE_NOT_ALLOWED`

**Rationale:** Historical data integrity and audit traceability require stations to persist.

**Source:**
- Interview: AMB-ENT-001

---

### BR-ENT-003 – Station Single Category

**Rule:** A station belongs to exactly one category.

**Invariant:** `∀ s ∈ Stations: |s.categories| = 1`

**Enforcement:**
- Check point: Station category assignment
- Violation: Block if attempting to assign multiple
- Error: `STATION_SINGLE_CATEGORY_ONLY`

**Source:**
- Interview: AMB-ENT-006

---

### BR-ENT-004 – Category Cannot Be Deleted If Has Stations

**Rule:** A station category cannot be deleted if any stations belong to it.

**Invariant:** `∀ c ∈ Categories: |c.stations| > 0 → delete(c) = BLOCKED`

**Enforcement:**
- Check point: Category deletion
- Violation: Block operation
- Error: `CATEGORY_HAS_STATIONS`

**Source:**
- Interview: AMB-ENT-020

---

### BR-ENT-005 – Task Sequence Immutable Job Assignment

**Rule:** A task cannot be moved to a different job after creation.

**Invariant:** `∀ t ∈ Tasks: t.job_id = IMMUTABLE after creation`

**Enforcement:**
- Check point: Any task update
- Violation: Block job_id change
- Error: `TASK_JOB_IMMUTABLE`

**Source:**
- Interview: AMB-ENT-016

---

### BR-ENT-006 – Minimum Task Duration

**Rule:** Normal tasks must have a combined duration (setup + run) of at least 30 minutes.

**Invariant:** `∀ t ∈ NormalTasks: t.setup_time + t.run_time >= 30 minutes`

**Enforcement:**
- Check point: Task creation, Task duration update
- Violation: Block operation
- Error: `TASK_DURATION_TOO_SHORT`

**Source:**
- Interview: Round 11 (min duration)

---

### BR-ENT-007 – Default Station Capacity

**Rule:** New stations default to capacity of 1.

**Invariant:** `∀ s ∈ new Stations: s.capacity = 1 (default)`

**Enforcement:**
- Check point: Station creation
- Behavior: Auto-set if not provided

**Source:**
- Interview: AMB-ENT-004

---

## 2. Scheduling Constraints

### BR-SCHED-001 – No Overlapping Tasks (Capacity 1)

**Rule:** On stations with capacity = 1, no two tasks can occupy the same time slot.

**Invariant:** `∀ s ∈ Stations where s.capacity = 1: ∀ t1, t2 ∈ s.tasks: t1 ≠ t2 → ¬overlaps(t1, t2)`

**Enforcement:**
- Check point: Task scheduling, Task rescheduling
- Violation: Push tiles down (not block)
- Behavior: Insert displaces subsequent tasks

**Source:**
- Input: project-brief.md § "Nincs átfedés"
- Input: quick-stories.md § "Tiles can't overlap"

---

### BR-SCHED-002 – Mandatory 30-Minute Snap Grid

**Rule:** All task start times must align to 30-minute boundaries.

**Invariant:** `∀ t ∈ ScheduledTasks: t.start_time.minutes ∈ {0, 30}`

**Enforcement:**
- Check point: Task drop on grid
- Behavior: Auto-snap to nearest 30-min boundary

**Source:**
- Input: quick-stories.md § "30-min snap grid"
- Interview: AMB-OP-002

---

### BR-SCHED-003 – Task Category Compatibility

**Rule:** Tasks can only be scheduled on stations belonging to their designated category.

**Invariant:** `∀ t ∈ ScheduledTasks: t.station.category = t.required_category`

**Enforcement:**
- Check point: Task drop on station
- Violation: Block drop, return task to origin
- Error: (Visual rejection - no toast)

**Source:**
- Interview: AMB-ENT-018

---

### BR-SCHED-004 – Push Cascade to Next Operating Hours

**Rule:** When pushing tiles would cause them to fall outside operating hours, they shift to the next available operating time.

**Invariant:** `∀ t ∈ PushedTasks: t.new_time ∈ station.operating_hours`

**Enforcement:**
- Check point: Push cascade calculation
- Behavior: Auto-advance to next operating period (may be next day)

**Source:**
- Interview: AMB-OP-006

---

### BR-SCHED-005 – Task Stretch Over Downtime

**Rule:** When a task overlaps with station unavailable time, it is split around the downtime while preserving total work time.

**Invariant:** `∀ t ∈ Tasks overlapping downtime: sum(t.active_segments) = t.duration`

**Enforcement:**
- Check point: Task placement, Schedule calculation
- Behavior: Automatic split display

**Source:**
- Input: project-brief.md § "ha egy feladat átlóg a gép szünetébe, nyúlik"
- Interview: AMB-OP-018

---

### BR-SCHED-006 – Insert Push Amount

**Rule:** When inserting a task, subsequent tasks are pushed by exactly the inserted task's total duration.

**Invariant:** `push_amount = inserted_task.setup_time + inserted_task.run_time`

**Enforcement:**
- Check point: Task insert operation
- Behavior: Automatic push calculation

**Source:**
- Interview: AMB-OP-005

---

### BR-SCHED-007 – Recall Leaves Gaps

**Rule:** When a task is recalled (unscheduled), other tasks do not automatically move up to fill the gap.

**Invariant:** `recall(task) → other_tasks.positions = UNCHANGED`

**Enforcement:**
- Check point: Recall operation
- Behavior: Gap remains on grid

**Source:**
- Interview: AMB-OP-008

---

## 3. Precedence Rules

### BR-PREC-001 – Task Sequence Must Be Respected

**Rule:** Tasks within a job should be scheduled in sequence order (task 1 before task 2, etc.).

**Invariant:** `∀ j ∈ Jobs, ∀ t1, t2 ∈ j.tasks: t1.sequence < t2.sequence → t1.end_time <= t2.start_time`

**Enforcement:**
- Check point: Task scheduling
- Violation: Allow but mark with red halo (warning, not block)
- Visual: Red halo persists until fixed

**Source:**
- Input: project-brief.md § "Sorrend számít"
- Interview: AMB-OP-009

---

### BR-PREC-002 – Alt+Drag Bypasses Precedence

**Rule:** Holding Alt key during task drop bypasses the precedence check.

**Invariant:** `Alt+drop → precedence_check = SKIPPED`

**Enforcement:**
- Check point: Task drop with Alt modifier
- Behavior: Allow placement, show red halo

**Source:**
- Input: project-brief.md § "Alt+húzás → kényszeríti a helyet"

---

### BR-PREC-003 – No Precedence Audit Logging

**Rule:** Precedence violations are visual-only indicators, not logged for audit.

**Invariant:** `precedence_violation → log = NONE`

**Source:**
- Interview: AMB-OP-010

---

## 4. Job Status Rules

### BR-JOB-001 – Job Late Determination

**Rule:** A job is considered "late" when its final scheduled task ends after the job's deadline.

**Invariant:** `job.is_late = (job.final_task.end_time > job.deadline @ 23:59:59)`

**Enforcement:**
- Check point: Task scheduling, Task rescheduling, Deadline update
- Behavior: Automatic recalculation

**Source:**
- Interview: AMB-ENT-011

---

### BR-JOB-002 – Days Late Calculation (Working Days)

**Rule:** "Days late" is calculated using working days only (Monday-Saturday).

**Invariant:** `days_late = count_working_days(deadline, final_task.end_date)`

**Enforcement:**
- Check point: Late display calculation
- Behavior: Skip Sundays in count

**Source:**
- Interview: AMB-OP-020

---

### BR-JOB-003 – Valid Paper Status Values

**Rule:** Paper status must be one of the defined values.

**Invariant:** `job.paper_status ∈ {'in_stock', 'to_order', 'ordered'}`

**Enforcement:**
- Check point: Job create, Job update
- Violation: Block invalid value
- Error: `INVALID_PAPER_STATUS`

**Source:**
- Interview: AMB-ENT-008

---

### BR-JOB-004 – Valid BAT Status Values

**Rule:** BAT (proof approval) status must be one of the defined values.

**Invariant:** `job.bat_status ∈ {'pending', 'approved'}`

**Enforcement:**
- Check point: Job create, Job update
- Violation: Block invalid value
- Error: `INVALID_BAT_STATUS`

**Source:**
- Interview: AMB-ENT-009

---

### BR-JOB-005 – Valid Plates Status Values

**Rule:** Plates status must be one of the defined values.

**Invariant:** `job.plates_status ∈ {'not_ready', 'ready'}`

**Enforcement:**
- Check point: Job create, Job update
- Violation: Block invalid value
- Error: `INVALID_PLATES_STATUS`

**Source:**
- Interview: AMB-ENT-010

---

### BR-JOB-006 – Scheduling Not Blocked by Paper Status

**Rule:** Tasks can be scheduled regardless of the job's paper_status value.

**Invariant:** `schedule(task) → ¬depends_on(job.paper_status)`

**Source:**
- Interview: AMB-OP-004

---

## 5. Outsourced Station Rules

### BR-OUT-001 – Outsourced Duration in Working Days

**Rule:** Tasks on outsourced stations are measured in working days, not hours/minutes.

**Invariant:** `∀ t ∈ OutsourcedTasks: t.duration.unit = WORKING_DAYS`

**Source:**
- Input: project-brief.md § "napokban mérik az átfutást"
- Interview: Round 11

---

### BR-OUT-002 – Outsourced No Capacity Limit

**Rule:** Outsourced stations have no capacity limit (unlimited parallel tasks).

**Invariant:** `∀ s ∈ OutsourcedStations: s.capacity = ∞`

**Source:**
- Input: project-brief.md § "nincs kapacitás-korlátjuk"
- Interview: AMB-ENT-023

---

## 6. Concurrency Rules

### BR-CONC-001 – Last Write Wins

**Rule:** When concurrent edits occur, the last save overwrites previous changes.

**Invariant:** `concurrent_writes → last_timestamp_wins`

**Enforcement:**
- Check point: Save operation
- Behavior: No conflict detection (MVP limitation)

**Source:**
- Interview: AMB-OP-016

---

## 7. Data Validation Rules

### BR-VAL-001 – Station Name Required

**Rule:** Station name is required and cannot be empty or whitespace-only.

**Invariant:** `∀ s ∈ Stations: s.name.trim().length > 0`

**Enforcement:**
- Check point: Station creation, Station update
- Violation: Block operation
- Error: `STATION_NAME_REQUIRED`

**Source:**
- Interview: AMB-OP-011

---

### BR-VAL-002 – String Length Limits

**Rule:** String fields have maximum length limits.

| Field | Max Length |
|-------|------------|
| station.name | 100 chars |
| job.title | 200 chars |
| job.client_name | 100 chars |
| task.description | 500 chars |

**Enforcement:**
- Check point: Entity creation, Entity update
- Violation: Block operation
- Error: `{FIELD}_TOO_LONG`

**Source:**
- Interview: Round 11 (defaults accepted)

---

## 8. Task DSL Rules

### BR-DSL-001 – Normal Task Format

**Rule:** Normal tasks follow the DSL pattern: `[StationName] setup+run "description"`

**Parse Rules:**
- `[...]` → Station name (required)
- First number → Setup time in minutes
- `+` → Separator
- Second number → Run time in minutes
- `"..."` → Description (optional)

**Source:**
- Input: project-brief.md § example DSL
- Interview: AMB-OP-013

---

### BR-DSL-002 – Outsourced Task Format

**Rule:** Outsourced tasks follow the DSL pattern: `ST [PartnerName] description #JO`

**Parse Rules:**
- `ST` → Outsourced marker
- `[...]` → Partner/station name
- Text → Description
- `#JO` or number+`JO` → Duration in working days (JO = "jour ouvré")

**Source:**
- Input: project-brief.md § "ST [Clément] Lakkozás 2JO"
- Interview: AMB-OP-013

---

## 9. UI Behavior Rules

### BR-UI-001 – Desktop Only

**Rule:** The scheduling grid is designed for desktop browsers (1200px+ width).

**Source:**
- Interview: AMB-UI-016

---

### BR-UI-002 – Similarity Circle Criteria

**Rule:** Similarity between consecutive tasks is evaluated on three criteria.

**Criteria:**
1. Paper type match
2. Paper size match
3. Ink configuration match

**Display:**
- ● (filled) = matches
- ○ (empty) = doesn't match

**Source:**
- Input: project-brief.md § "Hasonlósági jelzők"
- Interview: AMB-OP-019

---

### BR-UI-003 – Tile Visual Encoding

**Rule:** Task tiles encode information visually.

| Aspect | Encoding |
|--------|----------|
| Height | Proportional to duration |
| Lighter shade | Setup time |
| Darker shade | Run time |
| Red halo | Precedence violation |
| Hatched background | Station unavailable time |

**Source:**
- Interview: AMB-UI-006, AMB-UI-007, AMB-OP-009

---

## 10. Scope Exclusions (MVP)

### BR-SCOPE-001 – Features Out of Scope

The following features are explicitly excluded from MVP:

| Feature | Status | Reference |
|---------|--------|-----------|
| Right Panel (Late Jobs) | Removed | Interview: "outdated concept" |
| Keyboard navigation | v1.1 | quick-stories.md |
| Undo/redo | v1.1 | quick-stories.md |
| Quick placement mode | v1.1 | quick-stories.md |
| Off-screen tile indicators | v1.1 | quick-stories.md |
| Schedule branching | Not now | quick-stories.md |
| Auto-optimization | Not now | quick-stories.md |
| French holidays calendar | Not now | quick-stories.md |
| Real-time collaboration | Not now | quick-stories.md |

---
description: Generate formal specifications from natural language requirement
---

# Create Specification: $ARGUMENTS

Generate formal specifications from the provided natural language requirement.

## Step 0: Ambiguity Discovery & Decision Capture (MANDATORY GATE)

Before any analysis, proposal, or generation, perform an explicit ambiguity discovery step.
This step exists to prevent the LLM from making implicit strategic or tactical design decisions.

### 0.1 Classify Change Category (Hard Gate)

Classify the input **$ARGUMENTS** according to `change-governance.md` (Sections 8–10):

- **Category A (Domain / Spec-First):**
  Changes user capabilities, system behavior, data meaning, or user communication.
  → Continue to ambiguity discovery.

- **Category B (Visual Polish):**
  Pure visual or aesthetic changes with no behavioral or semantic impact.
  → STOP. Suggest Design System (DS-*) reference or Product Owner approval.
  → Do NOT generate specifications.

- **Category C (Technical / Infrastructure):**
  Refactoring, performance, security, infrastructure, tooling with no behavior change.
  → STOP. Suggest commit justification only.
  → Do NOT generate specifications.

If the category cannot be determined with certainty, treat the change as **Category A**.

---

### 0.2 Identify Ambiguities

Analyze the natural language input and list **all missing information** required to proceed.
Separate ambiguities into two strictly distinct groups.

#### A) Facts Needed (Information Gaps)

Facts are objective data points that do NOT involve design choice, for example:
- affected user role
- exact workflow step
- required field names or values
- exact wording of user-facing messages
- limits, thresholds, or formats explicitly required by the domain

List each missing fact explicitly.

#### B) Decisions Needed (Design Choices)

Decisions are choices between valid alternatives that affect system design or behavior.
Examples include (but are not limited to):
- bounded context or service ownership
- aggregate boundaries or invariants
- consistency model (strong vs eventual)
- integration style (sync / async)
- failure semantics, retries, idempotency
- data ownership / source of truth
- non-functional constraints derived from domain needs (security, audit, reliability)

List each required decision explicitly.
For each decision:
- describe the decision in neutral terms
- list the known alternatives
- DO NOT recommend or prefer any option

---

### 0.3 Ambiguity Gate (STOP Condition)

- If **any Decisions Needed** are identified:
  - PRESENT them to the user clearly.
  - STOP execution.
  - Do NOT proceed to Step 1.
  - Do NOT propose specifications.
  - Do NOT generate content.

- If **only Facts Needed** are identified:
  - Ask for the missing facts.
  - STOP execution until facts are provided.

- Proceed to Step 1 **only if**:
  - No Decisions Needed remain unresolved, AND
  - All required Facts are provided.

This gate is mandatory and cannot be bypassed.

## Step 1: Analyze & Propose

### 1.1 Read Documentation Standards (Mandatory Inputs)

Read the following documents to understand the specification standards and the allowed change flow:

- `docs/documentation-standards/document-structure.md` - IDs, headings, references, hierarchy, cardinality
- `docs/documentation-standards/change-governance.md` - Top-down change principle, categories, decision matrix
- `docs/domain-model/domain-vocabulary.md` - Ubiquitous language (terms must match)
- `docs/domain-model/business-rules.md` - Existing BR items
- `docs/domain-model/bounded-context-map.md` - Existing contexts and boundaries (if present)
- `docs/architecture/service-boundaries.md` - Existing SB items (if present)
- `docs/architecture/aggregate-design.md` - Existing AGG items (if present)
- `docs/architecture/interface-contracts.md` - Existing IC items (if present)

---

### 1.2 Analyze the Requirement (Category A only)

This Step 1 executes only if Step 0 classified the change as **Category A** AND all Step 0 ambiguities are resolved.

Analyze the natural language input: **$ARGUMENTS**

#### 1.2.1 Determine Impact Scope (Backend / Frontend / Both)

Classify the change scope based on what must change to satisfy the user-visible behavior:

- **Backend** if it requires new/changed APIs, validation, workflows, domain state changes, integration, or persistence.
- **Frontend** if it requires new/changed UI components, user interactions, UI state, or user-facing feedback.
- **Both** if both sets are required.

#### 1.2.2 Determine Required Domain-Level Specs (Mandatory)

Apply the Top-Down Change Principle. Always start from Domain Level.

Decide which of these must be created or modified:

- **US (User Story)** is required if the requirement introduces a new user capability or a new user goal.
- **AC (Acceptance Criteria)** is required for every behavior change (Given/When/Then), and is mandatory for implementation work.
- **BR (Business Rule)** is required if the requirement introduces or changes a constraint/invariant/validation ("must", "cannot", "only if", "always", thresholds, formats).

Minimum requirements:
- Every **US** must have **at least one AC** (US → AC = 1:n).
- Every **AC** must reference **exactly one US** (AC → US = n:1).
- If any BR applies, the AC must reference all relevant BRs.

#### 1.2.3 Determine Required Technical Specs (Derived Only)

Technical specs are allowed only as a consequence of Domain Level triggers.

Use the following deterministic mapping:

- If **Backend** or **Both**:
  - **API** is required when an external interface or endpoint is introduced/changed.
  - **IC** is required when an internal contract is defined for implementing an API (IC → API = 1:1).
  - **AGG** is required when domain state changes require aggregate/entity modeling (AGG → DM = 1:n).
  - **SB** is required only when service ownership or service decomposition must change (SB → AGG = 1:n).
- If **Frontend** or **Both**:
  - **UX** is required for any UI behavior, interaction, or user-facing feedback (UX references US and/or AC).
  - **DS** is required only if UX introduces new design tokens/components (DS → UX = 1:n).

IMPORTANT:
- Do NOT propose SB/AGG/IC/API/UX/DS unless the corresponding domain triggers (US/AC/BR) are included in the proposal.
- Do NOT propose new technical specs for Category B/C (those must have STOPped in Step 0).

#### 1.2.4 Search Existing Specs (Required)

Search for related existing items to avoid duplicates and to enable proper cross-references.
Use grep patterns for:
- key domain nouns from `domain-vocabulary.md`
- any existing IDs mentioned implicitly in the request
- likely categories (e.g., STATION, SCHED, UI, PANEL, GATE) based on the wording

Record all potentially related items found (IDs + short title/summary).

---

### 1.3 Present Proposal (Must STOP)

Present the proposal in this format and **STOP** (wait for user confirmation).

Rules:
- The proposal must list domain-level specs first, then derived technical specs.
- Each proposed spec must include a short reason tied to the requirement.
- If a spec is a modification of an existing item, state the existing ID(s).
- If a new spec is needed, state the intended PREFIX and CATEGORY (ID number assigned later in Step 2).

```markdown
## Spec Proposal

**Natural language input:**
> {original input}

**Step 0 classification (already decided):**
- Category: A - Domain / Spec-First

**Impact scope:**
- Path: {Backend/Frontend/Both}

**Resolved ambiguities:**
- Facts captured: {brief list}
- Decisions captured: {brief list}

**Proposed domain specifications (mandatory):**
- [ ] US-{CAT}-??? - {reason} {new|modify US-...}
- [ ] AC-{CAT}-??? - {reason} {new|modify AC-...}
- [ ] BR-{CAT}-??? - {reason} {new|modify BR-...} (if applicable)

**Proposed derived technical specifications (allowed only after domain triggers):**
- [ ] API-{CAT}-??? - {reason} {new|modify API-...} (if applicable)
- [ ] IC-{CAT}-??? - {reason} {new|modify IC-...} (if applicable)
- [ ] AGG-{CAT}-??? - {reason} {new|modify AGG-...} (if applicable)
- [ ] SB-{CAT}-??? - {reason} {new|modify SB-...} (if applicable)
- [ ] UX-{CAT}-??? - {reason} {new|modify UX-...} (if applicable)
- [ ] DS-{CAT}-??? - {reason} {new|modify DS-...} (if applicable)

**Related existing specs found:**
- {ID}: {title/summary}
...

---
**Proceed with these specs? (yes / modify selection / cancel)**
```

---

## Step 2: Generate (after approval)

Only proceed after user confirms the proposal.

### 2.1 Assign IDs

For each approved spec type, find the next available ID:
- Search `docs/` for existing IDs in the category
- Assign next sequential number (e.g., if US-GATE-002 exists, use US-GATE-003)

### 2.2 Generate Specifications

Generate each specification following `document-structure.md` standards:

**User Story (US):**
```markdown
### {Title}
#### US-{CATEGORY}-{NNN}
> **References:** [BR-*](path#anchor) (if applicable)

> As a **{role}**, I want **{goal}**, so that **{benefit}**.

**Acceptance Criteria:**
- {criterion 1}
- {criterion 2}
```

**Acceptance Criteria (AC):**
```markdown
### AC-{CATEGORY}-{NNN}: {Title}
> **References:** [US-*](path#anchor), [BR-*](path#anchor)

**Given** {precondition}
**When** {action}
**Then** {expected result}
```

**Business Rule (BR):**
```markdown
### BR-{CATEGORY}-{NNN}: {Title}

{Rule description}

**Constraints:**
- {constraint 1}
- {constraint 2}
```

**API Interface Draft (API):**
```markdown
### API-{CATEGORY}-{NNN}: {Title}
> **References:** [AC-*](path#anchor)

**Method:** {HTTP method}
**Endpoint:** {path}
**Description:** {what it does}
```

**UX Specification (UX):**
```markdown
### {Title}
#### UX-{CATEGORY}-{NNN}
> **References:** [US-*](path#anchor), [AC-*](path#anchor)

**Component:** {component name}
**Location:** {where in the UI}

**Behavior:**
- {behavior 1}
- {behavior 2}

**States:**
- Default state
- Hover state
- Active state
- Disabled state
```

### 2.3 Present Generated Specs

Present the generated specifications and **STOP - wait for save confirmation**:

```markdown
## Generated Specification

{all generated specs formatted as above}

---
**Save these specs to docs/? (yes / modify / cancel)**
```

### 2.4 Save to Files (after confirmation)

Only after user confirms:
1. Append each spec to the appropriate file in `docs/`
2. Report which files were modified
3. Suggest running `/spec-check {IDs}` to validate

---

## Important Rules

- **NEVER** generate specs without user approval at Step 1
- **NEVER** save specs without user approval at Step 2
- **ALWAYS** follow document-structure.md formats exactly
- **ALWAYS** use lowercase anchors in references
- **ALWAYS** check for existing IDs before assigning new ones
- Communication in the user's language (Hungarian for this project)

---
date: 2025-12-19
updated: 2025-12-26
author: Claude Sonnet 4.5 + Human collaboration
version: 1.1
status: draft
purpose: Bidirectional traceability system design for Loom
related: sonnett-evaluation-01.md, sonnett-evaluation-01-addendum.md
---

# Kétirányú Traceability - A Dokumentáció-Kód Szinkronizáció Megoldása

## 🎯 Probléma definíció

**Eredeti kritika:**
> "A kód és a dokumentáció garantáltan el fog térni. Nincs automatikus érvényesítés, hogy sync-ben vannak-e."

**Megoldás:**
> Kétirányú szigorú traceability a dokumentumok és kód között. AI időről időre ellenőrzi a konzisztenciát és koherenciát a referenciák alapján.

---

## 🔗 Traceability Koncepció

### Forward Tracing: Dokumentum → Kód

Minden dokumentumban szereplő implementálható egység tartalmaz egy **egyedi azonosítót**, amit a generált kódban **referenciálunk**.

**Példa:**

```markdown
<!-- requirements/user-stories.md -->

---
status: "approved"
---

# User Stories

## US-001: User Registration {#us-001}

As a new user, I want to register with email and password, so that I can access the system.

**Acceptance Criteria:**
- [AC-001-1] Email must be valid format
- [AC-001-2] Password must be at least 8 characters
- [AC-001-3] User receives confirmation email

**Implementation refs:**
- Code: `src/auth/registration.ts:registerUser()`
- Tests: `tests/auth/registration.test.ts:US-001`
```

**Generált kód:**

```typescript
// src/auth/registration.ts

/**
 * User registration handler
 *
 * @traceability US-001 (requirements/user-stories.md#us-001)
 * @implements AC-001-1, AC-001-2, AC-001-3
 */
export async function registerUser(email: string, password: string): Promise<User> {
  // @implements AC-001-1: Email validation
  if (!isValidEmail(email)) {
    throw new ValidationError('Invalid email format');
  }

  // @implements AC-001-2: Password length check
  if (password.length < 8) {
    throw new ValidationError('Password must be at least 8 characters');
  }

  const user = await db.users.create({ email, password: hashPassword(password) });

  // @implements AC-001-3: Send confirmation email
  await emailService.sendConfirmation(user.email);

  return user;
}
```

### Backward Tracing: Kód → Dokumentum

Minden kódfájl tetején vagy funkció előtt **traceability megjegyzések**, amik visszavezetnek a dokumentumokra.

**Példa:**

```typescript
// src/domain/entities/User.ts

/**
 * User entity
 *
 * @traceability
 *   - Domain Model: domain-modelling/domain-model.md#user-entity
 *   - Vocabulary: domain-modelling/domain-vocabulary.md#user
 *   - User Stories: requirements/user-stories.md#us-001, #us-002, #us-005
 */
export class User {
  // @traceability domain-modelling/domain-model.md#user-entity:email
  email: string;

  // @traceability domain-modelling/domain-model.md#user-entity:name
  name: string;

  // @traceability domain-modelling/domain-model.md#user-entity:role
  // @traceability domain-modelling/domain-vocabulary.md#role
  role: UserRole;
}
```

---

## 📐 Traceability ID Scheme

### Dokumentum típusok és ID prefix-ek

| Dokumentum típus | Prefix | Példa ID | Scope |
|-----------------|--------|----------|-------|
| User Story | `US-` | `US-001` | Egy user story |
| Acceptance Criterion | `AC-` | `AC-001-1` | Egy acceptance criterion (user story-hoz kötve) |
| Domain Entity | `ENT-` | `ENT-User` | Egy domain entitás |
| Domain Vocabulary | `TERM-` | `TERM-Role` | Egy domain fogalom |
| Architecture Decision | `ADR-` | `ADR-001` | Egy ADR |
| API Endpoint | `API-` | `API-POST-users` | Egy API endpoint |
| Business Rule | `BR-` | `BR-001` | Egy business rule |
| Test Case | `TC-` | `TC-001` | Egy test case |
| Sequence Flow | `SEQ-` | `SEQ-UserLogin` | Egy sequence diagram flow |

### ID formátum szabályok

**User Story:**
```
US-<number>
  Példa: US-001, US-042
```

**Acceptance Criterion:**
```
AC-<user-story-number>-<criterion-number>
  Példa: AC-001-1, AC-001-2
```

**Entity:**
```
ENT-<EntityName>
  Példa: ENT-User, ENT-Task, ENT-Comment
```

**Entity Field:**
```
ENT-<EntityName>:<fieldName>
  Példa: ENT-User:email, ENT-Task:status
```

**Architecture Decision:**
```
ADR-<number>-<short-title>
  Példa: ADR-001-use-postgresql, ADR-002-auth-strategy
```

---

## 🔧 Implementation: Traceability Annotations

### In Markdown Documents

**Anchor syntax (for linking):**

```markdown
## User Registration {#us-001}
```

**Implementation reference:**

```markdown
**Implementation refs:**
- Code: `src/auth/registration.ts:registerUser()`
- Tests: `tests/auth/registration.test.ts:describe('US-001')`
- Status: ✓ Implemented (2024-12-15)
```

### In Code

**TypeScript/JavaScript:**

```typescript
/**
 * @traceability US-001 (requirements/user-stories.md#us-001)
 * @implements AC-001-1, AC-001-2, AC-001-3
 * @domain-entity ENT-User (domain-modelling/domain-model.md#user-entity)
 */
```

**Python:**

```python
"""
Traceability:
  - US-001: requirements/user-stories.md#us-001
  - Implements: AC-001-1, AC-001-2, AC-001-3
  - Domain Entity: ENT-User (domain-modelling/domain-model.md#user-entity)
"""
```

**Inline annotations:**

```typescript
// @implements AC-001-1: Email validation
if (!isValidEmail(email)) {
  throw new ValidationError('Invalid email format');
}
```

---

## 🤖 AI-Powered Consistency Checking

### Consistency Check Types

#### 1. **Existence Check**

**Rule:** Minden kódban említett traceability ID létezik a dokumentációban.

**Example violation:**

```typescript
// Code:
/**
 * @traceability US-999  <-- ERROR: US-999 does not exist
 */
```

**AI Check:**
1. Parse all `@traceability` annotations in code
2. Extract referenced IDs (e.g., `US-999`)
3. Search for ID in documentation files
4. Report error if not found

#### 2. **Implementation Coverage Check**

**Rule:** Minden acceptance criterion-hoz tartozik implementáció a kódban.

**Example violation:**

```markdown
<!-- Documentation: -->
## US-001: User Registration

Acceptance Criteria:
- [AC-001-1] Email must be valid  ✓ Implemented
- [AC-001-2] Password minimum 8 chars  ✓ Implemented
- [AC-001-3] Send confirmation email  ✗ NOT IMPLEMENTED
```

**AI Check:**
1. Extract all AC IDs from requirements docs
2. Search codebase for `@implements AC-001-3`
3. Report warning if not found

#### 3. **Orphaned Code Check**

**Rule:** Minden production kód referenciál legalább egy dokumentumot.

**Example violation:**

```typescript
// src/utils/randomHelper.ts

export function generateRandomString(length: number): string {
  // No @traceability annotation!  <-- WARNING: Orphaned code
  return Math.random().toString(36).substr(2, length);
}
```

**AI Check:**
1. Parse all code files
2. Check for `@traceability` annotations
3. Report warning for files/functions without traceability

**Exception:** Utility/helper code may be exempt if marked:
```typescript
/**
 * @traceability-exempt utility function
 */
```

#### 4. **Semantic Consistency Check**

**Rule:** Kód implementáció megfelel a dokumentáció leírásának.

**Example violation:**

```markdown
<!-- Documentation: -->
[AC-001-2] Password must be at least 8 characters
```

```typescript
// Code:
// @implements AC-001-2
if (password.length < 6) {  // <-- ERROR: Should be 8, not 6!
  throw new ValidationError('Password too short');
}
```

**AI Check:**
1. Extract acceptance criterion text
2. Extract corresponding code implementation
3. LLM analyzes: "Does this code correctly implement the requirement?"
4. Report error if semantic mismatch

**LLM Prompt Example:**

```
You are a code reviewer checking implementation correctness.

Requirement:
  [AC-001-2] Password must be at least 8 characters

Code implementation:
  ```typescript
  if (password.length < 6) {
    throw new ValidationError('Password too short');
  }
  ```

Question: Does the code correctly implement the requirement?

Answer: No. The requirement states "at least 8 characters", but the code checks for "< 6", which allows passwords as short as 6 characters. This is a violation.

Severity: ERROR
```

#### 5. **Domain Model Consistency Check**

**Rule:** Code entity fields match domain model definition.

**Example violation:**

```markdown
<!-- domain-modelling/domain-model.md -->
### User {#ent-user}
- email: string
- name: string
- role: enum(admin, regular)
```

```typescript
// src/domain/entities/User.ts
/**
 * @domain-entity ENT-User
 */
export class User {
  email: string;
  name: string;
  role: string;  // <-- ERROR: Should be enum UserRole, not string!
  age: number;   // <-- WARNING: Field not in domain model
}
```

**AI Check:**
1. Extract domain entity definition from docs
2. Parse code class/interface
3. Compare fields (name, type, presence)
4. Report mismatches

#### 6. **Test Coverage Check**

**Rule:** Minden user story-hoz tartoznak tesztek.

**Example violation:**

```markdown
<!-- requirements/user-stories.md -->
## US-001: User Registration {#us-001}

**Implementation refs:**
- Code: `src/auth/registration.ts:registerUser()`
- Tests: MISSING  <-- ERROR
```

**AI Check:**
1. Extract all user story IDs
2. Search test files for `@traceability US-XXX` or `describe('US-XXX')`
3. Report error if no test found

---

## 🛠️ Tooling Support

### 1. Traceability Parser

**Purpose:** Extract all traceability annotations from code and docs.

```typescript
interface TraceabilityLink {
  sourceFile: string;
  sourceLocation: { line: number; column: number };
  targetId: string;
  targetFile?: string;
  targetAnchor?: string;
  linkType: 'implements' | 'traceability' | 'domain-entity';
}

class TraceabilityParser {
  async parseCodebase(rootDir: string): Promise<TraceabilityLink[]> {
    // 1. Parse all code files, extract @traceability annotations
    // 2. Parse all markdown files, extract IDs and anchors
    // 3. Build traceability graph
  }
}
```

**Example usage:**

```bash
loom trace parse

Output:
Found 145 traceability links:
  - Code → Docs: 89 links
  - Docs → Code: 56 links
  - Orphaned code: 12 files (warnings)
  - Missing implementations: 3 (errors)
```

### 2. Traceability Validator

**Purpose:** Run all consistency checks.

```typescript
class TraceabilityValidator {
  async validate(links: TraceabilityLink[]): Promise<ValidationResult> {
    const errors: ValidationError[] = [];

    // Run all checks
    errors.push(...await this.checkExistence(links));
    errors.push(...await this.checkImplementationCoverage(links));
    errors.push(...await this.checkOrphanedCode(links));
    errors.push(...await this.checkSemanticConsistency(links));
    errors.push(...await this.checkDomainModelConsistency(links));
    errors.push(...await this.checkTestCoverage(links));

    return { isValid: errors.length === 0, errors };
  }
}
```

**Example usage:**

```bash
loom trace validate

Output:
Validating traceability...

✓ Existence check passed (89 links verified)
✗ Implementation coverage check failed:
    - AC-001-3: No implementation found
    - AC-005-2: No implementation found
⚠ Orphaned code check found 12 warnings:
    - src/utils/randomHelper.ts: No traceability annotation
    - src/helpers/format.ts: No traceability annotation
    ...
✓ Semantic consistency check passed (LLM verified 89 implementations)
✗ Domain model consistency check failed:
    - ENT-User: Field "role" type mismatch (expected enum, found string)
⚠ Test coverage check found 1 warning:
    - US-007: No test found

Summary: 3 errors, 13 warnings
```

### 3. Traceability Graph Viewer

**Purpose:** Visualize traceability links.

```bash
loom trace graph --output trace-graph.html
```

**Output (interactive HTML):**

```
┌─────────────────────────────────────────────────────┐
│        Traceability Graph Viewer                    │
├─────────────────────────────────────────────────────┤
│                                                      │
│  [User Stories] ──────> [Code] ──────> [Tests]     │
│       │                   │                          │
│       │                   │                          │
│       ├── US-001 ────> src/auth/registration.ts    │
│       │                   ├──> AC-001-1 impl       │
│       │                   ├──> AC-001-2 impl       │
│       │                   └──> AC-001-3 MISSING!   │
│       │                                             │
│       └── US-002 ────> src/auth/login.ts           │
│                           └──> tests/auth/login.test│
│                                                      │
│  [Domain Model] ──────> [Entities]                 │
│       │                                             │
│       ├── ENT-User ───> src/domain/entities/User.ts│
│       │                   ⚠ type mismatch: role     │
│       │                                             │
│       └── ENT-Task ───> src/domain/entities/Task.ts│
│                           ✓ consistent              │
└─────────────────────────────────────────────────────┘

Legend:
  ✓ Consistent
  ⚠ Warning
  ✗ Error
  MISSING - Not implemented
```

### 4. Auto-Generate Traceability

**Purpose:** AI automatically adds traceability annotations when generating code.

```bash
loom generate code --from-user-story US-001
```

**Generated code automatically includes:**

```typescript
/**
 * User registration handler
 *
 * @traceability US-001 (requirements/user-stories.md#us-001)
 * @implements AC-001-1, AC-001-2, AC-001-3
 * @domain-entity ENT-User (domain-modelling/domain-model.md#user-entity)
 * @generated-by Loom v0.1.0
 * @generated-at 2024-12-19T10:30:00Z
 */
export async function registerUser(email: string, password: string): Promise<User> {
  // Implementation...
}
```

### 5. Sync Command

**Purpose:** Update documentation based on code changes.

```bash
loom trace sync --direction code-to-docs
```

**Workflow:**
1. Detect code changes (git diff)
2. Extract affected traceability IDs
3. AI reads the code changes
4. AI updates corresponding documentation sections
5. Show diff and ask for approval

**Example:**

```bash
loom trace sync --direction code-to-docs

Detected changes:
  - src/auth/registration.ts: Modified function registerUser()

Affected documentation:
  - requirements/user-stories.md#us-001

Analyzing changes...

AI proposes updates:
─────────────────────────────────────────────────────────
requirements/user-stories.md
─────────────────────────────────────────────────────────
  ## US-001: User Registration

  **Implementation refs:**
- - Code: `src/auth/registration.ts:registerUser()`
+ - Code: `src/auth/registration.ts:registerUser()`
- - Status: ✓ Implemented (2024-12-15)
+ - Status: ✓ Implemented (2024-12-15), Updated (2024-12-19)
+ - Changes: Added email normalization before validation
─────────────────────────────────────────────────────────

Approve update? [y/n]
```

**Reverse sync (docs → code):**

```bash
loom trace sync --direction docs-to-code
```

---

## 📊 Traceability Metrics & Dashboard

### Metrics to Track

```typescript
interface TraceabilityMetrics {
  // Coverage metrics
  totalRequirements: number;
  implementedRequirements: number;
  implementationCoverage: number; // %

  totalUserStories: number;
  testedUserStories: number;
  testCoverage: number; // %

  // Consistency metrics
  totalTraceabilityLinks: number;
  brokenLinks: number;
  orphanedCodeFiles: number;

  // Quality metrics
  semanticConsistencyScore: number; // 0-100 (LLM evaluation)
  domainModelConsistencyScore: number; // 0-100

  // Temporal metrics
  avgTimeSinceLastSync: number; // days
  staleDocs: number; // docs not updated in > 30 days
}
```

### Dashboard View

```bash
loom trace dashboard
```

**Output:**

```
┌─────────────────────────────────────────────────────────┐
│              Traceability Dashboard                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  📊 Coverage                                            │
│    Implementation: 87/95 (91.6%) ████████████░░  ✓     │
│    Test Coverage:  82/95 (86.3%) ███████████░░░  ⚠     │
│                                                          │
│  🔗 Traceability Links                                  │
│    Total Links:     234                                 │
│    Broken Links:    3 ✗                                 │
│    Orphaned Code:   12 files ⚠                          │
│                                                          │
│  ✅ Consistency                                         │
│    Semantic:        94.2% ████████████░░  ✓            │
│    Domain Model:    97.8% █████████████░  ✓            │
│                                                          │
│  📅 Freshness                                           │
│    Avg time since sync: 3.2 days                       │
│    Stale docs (>30d):   2 ⚠                            │
│      - architecture/adr-003-caching.md (45 days)       │
│      - requirements/nfr.md (38 days)                   │
│                                                          │
│  🚨 Issues (18 total)                                   │
│    Errors:   3 ✗                                        │
│    Warnings: 15 ⚠                                       │
│                                                          │
│  [View Details] [Fix Issues] [Generate Report]         │
└─────────────────────────────────────────────────────────┘
```

---

## 🔄 Workflow Integration

### Development Workflow with Traceability

**1. Requirement Phase:**

```bash
# Human describes requirement
loom generate "Add password reset functionality"

# AI generates docs with IDs
# - US-015: Password reset user story
# - AC-015-1, AC-015-2, AC-015-3: Acceptance criteria
# - ENT-PasswordResetToken: New entity
```

**2. Implementation Phase:**

```bash
# Generate code from requirement
loom generate code --from-user-story US-015

# AI generates code with traceability annotations
# src/auth/password-reset.ts:
#   /**
#    * @traceability US-015
#    * @implements AC-015-1, AC-015-2, AC-015-3
#    */
```

**3. Testing Phase:**

```bash
# Generate tests
loom generate tests --from-user-story US-015

# AI generates tests with traceability
# tests/auth/password-reset.test.ts:
#   describe('US-015: Password Reset', () => {
#     it('should implement AC-015-1: Send reset email', ...)
#     it('should implement AC-015-2: Validate token', ...)
#   })
```

**4. Continuous Validation:**

```bash
# Pre-commit hook
loom trace validate

# CI/CD pipeline
loom trace validate --strict --fail-on-warnings
```

**5. Periodic Sync:**

```bash
# Weekly: check for drift
loom trace sync --check-only

# If drift detected, sync
loom trace sync --direction code-to-docs --auto-approve-safe
```

### Git Integration

**Pre-commit hook:**

```bash
#!/bin/bash
# .git/hooks/pre-commit

echo "Running traceability validation..."
loom trace validate --pre-commit

if [ $? -ne 0 ]; then
  echo "❌ Traceability validation failed!"
  echo "Run 'loom trace validate' to see details"
  exit 1
fi

echo "✓ Traceability validation passed"
```

**PR checks (GitHub Actions):**

```yaml
# .github/workflows/traceability.yml
name: Traceability Check

on: [pull_request]

jobs:
  trace-validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Validate traceability
        run: loom trace validate --strict

      - name: Check coverage
        run: |
          COVERAGE=$(loom trace metrics --json | jq '.implementationCoverage')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "❌ Implementation coverage below 80%: $COVERAGE%"
            exit 1
          fi

      - name: Post PR comment
        uses: actions/github-script@v5
        with:
          script: |
            const metrics = require('./trace-metrics.json');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Traceability Report

              - Implementation Coverage: ${metrics.implementationCoverage}%
              - Test Coverage: ${metrics.testCoverage}%
              - Broken Links: ${metrics.brokenLinks}
              - Orphaned Code: ${metrics.orphanedCodeFiles} files
              `
            });
```

---

## 🧪 Example: Full Traceability Flow

### Step 1: Generate Documentation

```bash
loom generate "Add task commenting feature. Users can add, edit, delete comments on tasks."
```

**Generated docs:**

```markdown
<!-- requirements/user-stories.md -->

## US-020: Task Commenting {#us-020}

As a user, I want to add comments to tasks, so that I can communicate with team members.

**Acceptance Criteria:**
- [AC-020-1] User can add a comment to a task
- [AC-020-2] User can edit their own comments
- [AC-020-3] User can delete their own comments
- [AC-020-4] Comments display author and timestamp

**Implementation refs:**
- Code: TBD
- Tests: TBD
- Status: 📝 Not implemented
```

```markdown
<!-- domain-modelling/domain-model.md -->

### Comment {#ent-comment}

Represents a user comment on a task.

**Fields:**
- id: UUID {#ent-comment:id}
- text: string (max 1000 chars) {#ent-comment:text}
- authorId: User reference {#ent-comment:authorId}
- taskId: Task reference {#ent-comment:taskId}
- createdAt: timestamp {#ent-comment:createdAt}
- updatedAt: timestamp {#ent-comment:updatedAt}

**Relationships:**
- Comment belongs to Task (many-to-one)
- Comment belongs to User (many-to-one)
```

### Step 2: Generate Code

```bash
loom generate code --from-user-story US-020
```

**Generated code:**

```typescript
// src/domain/entities/Comment.ts

/**
 * Comment entity
 *
 * @traceability
 *   - Domain Model: domain-modelling/domain-model.md#ent-comment
 *   - User Story: requirements/user-stories.md#us-020
 *
 * @domain-entity ENT-Comment
 */
export class Comment {
  /** @traceability domain-modelling/domain-model.md#ent-comment:id */
  id: string;

  /** @traceability domain-modelling/domain-model.md#ent-comment:text */
  text: string;

  /** @traceability domain-modelling/domain-model.md#ent-comment:authorId */
  authorId: string;

  /** @traceability domain-modelling/domain-model.md#ent-comment:taskId */
  taskId: string;

  /** @traceability domain-modelling/domain-model.md#ent-comment:createdAt */
  createdAt: Date;

  /** @traceability domain-modelling/domain-model.md#ent-comment:updatedAt */
  updatedAt: Date;
}
```

```typescript
// src/features/comments/comment-service.ts

/**
 * Comment service
 *
 * @traceability US-020 (requirements/user-stories.md#us-020)
 */
export class CommentService {
  /**
   * Add comment to task
   *
   * @implements AC-020-1
   * @traceability requirements/user-stories.md#us-020
   */
  async addComment(taskId: string, authorId: string, text: string): Promise<Comment> {
    // @implements AC-020-4: Include author and timestamp
    const comment = new Comment();
    comment.id = generateId();
    comment.taskId = taskId;
    comment.authorId = authorId;
    comment.text = text;
    comment.createdAt = new Date();
    comment.updatedAt = new Date();

    await this.commentRepo.save(comment);
    return comment;
  }

  /**
   * Edit comment
   *
   * @implements AC-020-2
   */
  async editComment(commentId: string, userId: string, newText: string): Promise<Comment> {
    const comment = await this.commentRepo.findById(commentId);

    // @implements AC-020-2: User can only edit their own comments
    if (comment.authorId !== userId) {
      throw new ForbiddenError('Cannot edit other users comments');
    }

    comment.text = newText;
    comment.updatedAt = new Date();

    await this.commentRepo.save(comment);
    return comment;
  }

  /**
   * Delete comment
   *
   * @implements AC-020-3
   */
  async deleteComment(commentId: string, userId: string): Promise<void> {
    const comment = await this.commentRepo.findById(commentId);

    // @implements AC-020-3: User can only delete their own comments
    if (comment.authorId !== userId) {
      throw new ForbiddenError('Cannot delete other users comments');
    }

    await this.commentRepo.delete(commentId);
  }
}
```

### Step 3: Generate Tests

```bash
loom generate tests --from-user-story US-020
```

**Generated tests:**

```typescript
// tests/features/comments/comment-service.test.ts

/**
 * @traceability US-020 (requirements/user-stories.md#us-020)
 */
describe('US-020: Task Commenting', () => {
  let service: CommentService;

  beforeEach(() => {
    service = new CommentService();
  });

  /**
   * @implements AC-020-1
   */
  it('should allow user to add comment to task', async () => {
    const comment = await service.addComment('task-1', 'user-1', 'Great work!');

    expect(comment.taskId).toBe('task-1');
    expect(comment.authorId).toBe('user-1');
    expect(comment.text).toBe('Great work!');
    expect(comment.createdAt).toBeDefined();
  });

  /**
   * @implements AC-020-2
   */
  it('should allow user to edit their own comment', async () => {
    const comment = await service.addComment('task-1', 'user-1', 'Initial text');
    const updated = await service.editComment(comment.id, 'user-1', 'Updated text');

    expect(updated.text).toBe('Updated text');
    expect(updated.updatedAt).not.toEqual(comment.updatedAt);
  });

  /**
   * @implements AC-020-2
   */
  it('should NOT allow user to edit other users comment', async () => {
    const comment = await service.addComment('task-1', 'user-1', 'Initial text');

    await expect(
      service.editComment(comment.id, 'user-2', 'Hacked!')
    ).rejects.toThrow(ForbiddenError);
  });

  // ... (similar tests for AC-020-3, AC-020-4)
});
```

### Step 4: Update Documentation with Implementation Refs

**AI automatically updates:**

```markdown
<!-- requirements/user-stories.md -->

## US-020: Task Commenting {#us-020}

As a user, I want to add comments to tasks, so that I can communicate with team members.

**Acceptance Criteria:**
- [AC-020-1] User can add a comment to a task ✓
- [AC-020-2] User can edit their own comments ✓
- [AC-020-3] User can delete their own comments ✓
- [AC-020-4] Comments display author and timestamp ✓

**Implementation refs:**
- Code: `src/features/comments/comment-service.ts`
- Entity: `src/domain/entities/Comment.ts`
- Tests: `tests/features/comments/comment-service.test.ts`
- Status: ✓ Implemented (2024-12-19)
```

### Step 5: Validate Traceability

```bash
loom trace validate
```

**Output:**

```
Validating traceability for US-020...

✓ Existence check passed
  - US-020 found in requirements/user-stories.md
  - ENT-Comment found in domain-modelling/domain-model.md

✓ Implementation coverage check passed
  - AC-020-1: Implemented in src/features/comments/comment-service.ts:addComment()
  - AC-020-2: Implemented in src/features/comments/comment-service.ts:editComment()
  - AC-020-3: Implemented in src/features/comments/comment-service.ts:deleteComment()
  - AC-020-4: Implemented in src/features/comments/comment-service.ts:addComment()

✓ Test coverage check passed
  - US-020 tested in tests/features/comments/comment-service.test.ts

✓ Domain model consistency check passed
  - ENT-Comment fields match domain-model.md definition

✓ Semantic consistency check passed (LLM verified)
  - AC-020-2 implementation correctly enforces "own comments only" rule

Summary: 0 errors, 0 warnings ✓

Traceability graph:
  US-020 (User Story)
    ├─> AC-020-1 ──> CommentService.addComment() ──> test: 'should allow user to add comment'
    ├─> AC-020-2 ──> CommentService.editComment() ──> test: 'should allow user to edit own comment'
    ├─> AC-020-3 ──> CommentService.deleteComment() ──> test: 'should NOT allow delete others comment'
    └─> AC-020-4 ──> CommentService.addComment() ──> (included in AC-020-1 test)

  ENT-Comment (Entity)
    └─> src/domain/entities/Comment.ts
          ├─> id field ✓
          ├─> text field ✓
          ├─> authorId field ✓
          ├─> taskId field ✓
          ├─> createdAt field ✓
          └─> updatedAt field ✓
```

### Step 6: Later, Code Changes Detected

**Developer manually changes code:**

```typescript
// src/features/comments/comment-service.ts

async editComment(commentId: string, userId: string, newText: string): Promise<Comment> {
  const comment = await this.commentRepo.findById(commentId);

  // NEW: Allow admins to edit any comment
  const user = await this.userRepo.findById(userId);
  if (comment.authorId !== userId && user.role !== 'admin') {
    throw new ForbiddenError('Cannot edit other users comments');
  }

  // ... rest unchanged
}
```

**Run sync check:**

```bash
loom trace sync --check-only
```

**Output:**

```
⚠ Drift detected!

File: src/features/comments/comment-service.ts
Function: editComment()
Traceability: US-020, AC-020-2

Changes detected:
  - New logic: Allow admins to edit any comment

Documentation states:
  [AC-020-2] User can edit their own comments

Code now allows:
  - User can edit their own comments ✓
  - Admin can edit any comment (NEW, not in docs!)

Recommendation: Update documentation to reflect new behavior.

Update docs? [y/n]
```

**User approves, AI updates docs:**

```markdown
<!-- requirements/user-stories.md -->

## US-020: Task Commenting {#us-020}

**Acceptance Criteria:**
- [AC-020-2] User can edit their own comments; Admins can edit any comment ✓ (Updated 2024-12-20)
```

---

## 🎯 Benefits of Bidirectional Traceability

### 1. **Documentation-Code Sync Guaranteed**

Traditional problem:
```
Code changes → Developer forgets to update docs → Drift
```

With traceability:
```
Code changes → AI detects drift → AI proposes doc update → Human approves → Sync maintained
```

### 2. **Impact Analysis**

**Question:** "If I change this requirement, what code is affected?"

**Answer:**
```bash
loom trace impact --requirement US-020

Files affected by US-020:
  - src/domain/entities/Comment.ts
  - src/features/comments/comment-service.ts
  - tests/features/comments/comment-service.test.ts

Functions affected:
  - CommentService.addComment() (implements AC-020-1, AC-020-4)
  - CommentService.editComment() (implements AC-020-2)
  - CommentService.deleteComment() (implements AC-020-3)
```

### 3. **Automated Regression Testing**

**When documentation changes:**

```bash
# Documentation changed: AC-020-2 text modified
loom trace test --requirement AC-020-2

Running tests for AC-020-2...
  ✓ should allow user to edit their own comment
  ✗ should NOT allow admin to edit other users comment (NEW REQUIREMENT)

1 test failed. Generate missing test? [y/n]
```

### 4. **Onboarding & Knowledge Transfer**

**New developer:**
```bash
loom trace explain --code src/features/comments/comment-service.ts
```

**Output:**
```
CommentService (src/features/comments/comment-service.ts)

Purpose: Task commenting feature

Related Documentation:
  - User Story: US-020 (requirements/user-stories.md#us-020)
    "As a user, I want to add comments to tasks..."

  - Domain Model: ENT-Comment (domain-modelling/domain-model.md#ent-comment)
    Entity definition with fields and relationships

  - Acceptance Criteria:
    - AC-020-1: User can add comment ✓
    - AC-020-2: User can edit own comments ✓
    - AC-020-3: User can delete own comments ✓

Tests: tests/features/comments/comment-service.test.ts

To understand this feature, read:
  1. requirements/user-stories.md#us-020 (business context)
  2. domain-modelling/domain-model.md#ent-comment (data model)
  3. This file (implementation)
```

### 5. **Audit & Compliance**

For regulated industries (finance, healthcare):

```bash
loom trace audit --output audit-report.pdf
```

**Report includes:**
- Full traceability matrix (requirement → code → test)
- Coverage metrics
- Change history (who changed what, when, why)
- Validation results

---

## 🚀 Roadmap

### PoC Phase (v0.1)

- [ ] Basic traceability annotation syntax
- [ ] Parser for extracting traceability links
- [ ] Existence check validator
- [ ] Simple CLI: `loom trace parse`, `loom trace validate`

### MVP Phase (v0.5)

- [ ] All 6 consistency checks implemented
- [ ] Auto-generate traceability when generating code
- [ ] Traceability graph viewer (text-based)
- [ ] Git pre-commit hook
- [ ] CI/CD integration

### Full Release (v1.0)

- [ ] Bidirectional sync (code ↔ docs)
- [ ] Interactive HTML traceability graph
- [ ] LLM-powered semantic consistency checking
- [ ] Impact analysis tool
- [ ] Metrics dashboard
- [ ] Audit report generation

---

## 📚 References

### Industry Standards

- **ISO 26262** (Automotive Safety) - Requires full traceability
- **DO-178C** (Aviation Software) - Traceability matrix mandatory
- **IEC 62304** (Medical Device Software) - Requirement-to-test traceability
- **CMMI** (Capability Maturity Model Integration) - Traceability as maturity indicator

### Prior Art

- **DOORS** (IBM) - Requirements management with traceability
- **Jama Connect** - Product development platform with traceability
- **ReqView** - Requirements management tool
- **PlantUML / Structurizr** - Architecture diagrams with traceability

### Research

- "Traceability in Software Engineering: Past, Present, and Future" (Gotel et al.)
- "Automated Traceability Maintenance via Machine Learning" (recent papers)

---

## 💭 Conclusion

**Bidirectional traceability is the KEY to solving the documentation-code sync problem.**

Without it:
- Documentation drifts
- Developers ignore docs
- Loom fails

With it:
- Documentation stays current (AI enforces it)
- Developers trust docs (they're always accurate)
- Loom succeeds (reliable context for AI code generation)

**This is not just "nice to have" — it's FUNDAMENTAL to the Loom vision.**

Next step: Integrate traceability into the PoC tooling design.

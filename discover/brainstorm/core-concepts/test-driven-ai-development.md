---
date: 2025-12-19
updated: 2025-12-26
author: Claude Sonnet 4.5 + Human collaboration
version: 1.1
status: draft
purpose: Test-Driven AI Development - Mitigating AI hallucination through comprehensive test generation
related: sonnett-evaluation-01.md, sonnett-evaluation-01-addendum.md
---

# Test-Driven AI Development: A hallucináció problémájának megoldása

## 🎯 Probléma definíció

**Eredeti kritika (sonnett-evaluation-01.md):**

> **3. AI képességek túlbecslése**
> - Feltételezés: az AI pontosan követi a specs-et ✗
> - Valóság: az AI gyakran "kreatívan értelmez", kitalál dolgokat
> - A hosszú context window-k (200k token) NEM jelentik azt, hogy az AI mindent perfektül értelmez
> - Az AI hallucináció problémája nem oldódik meg dokumentációval

**Konkrét problémák:**

1. **Hallucináció:** AI kitalál dolgokat, amik nincsenek a specs-ben
2. **Kreatív értelmezés:** AI "tölti ki a hézagokat", implicit döntéseket hoz
3. **Context overflow:** 200k token is kevés lehet komplex projektekhez
4. **Silent failures:** AI generál kódot, ami "működik", de nem azt csinálja, amit kellene

**Példa hallucináció:**

```markdown
<!-- Requirement: -->
[AC-001-2] Password must be at least 8 characters

<!-- AI generál: -->
function validatePassword(password) {
  if (password.length < 8) return false;

  // AI HALLUCINÁCIÓ: nincs a requirements-ben!
  if (!/[A-Z]/.test(password)) return false;  // uppercase required
  if (!/[0-9]/.test(password)) return false;  // number required
  if (!/[!@#$]/.test(password)) return false; // special char required

  return true;
}
```

Az AI "tudja", hogy a jó jelszóvalidáció így néz ki, ezért hozzáadja - **de nem volt a requirements-ben!**

---

## 💡 Megoldás: Test-Driven AI Development (TDAI)

### Alapelv

> **Nagy számú automatikusan generált teszt (unit, integration, e2e) szűkíti a hallucináció mozgásterét és kiszűri a hallucinációkat, implicit döntéseket.**

### Workflow

```
┌──────────────────────────────────────────────────────────┐
│ 1. Requirements & User Stories                          │
│    (Human-approved, canonical source of truth)          │
└─────────────────┬────────────────────────────────────────┘
                  │
                  ▼
┌──────────────────────────────────────────────────────────┐
│ 2. AI Generates Comprehensive Tests FIRST               │
│    - Unit tests for each acceptance criterion           │
│    - Integration tests for workflows                    │
│    - E2E tests for user stories                         │
│    - Edge case tests                                    │
│    - Negative tests (what should NOT happen)           │
└─────────────────┬────────────────────────────────────────┘
                  │
                  ▼
┌──────────────────────────────────────────────────────────┐
│ 3. Human QA Reviews Tests                               │
│    - Do tests correctly capture requirements?           │
│    - Are there hallucinated test cases?                 │
│    - Are edge cases covered?                            │
│    - Approve or request changes                         │
└─────────────────┬────────────────────────────────────────┘
                  │
                  ▼
┌──────────────────────────────────────────────────────────┐
│ 4. AI Generates Implementation                          │
│    - Code must pass ALL tests                           │
│    - Tests act as CONSTRAINTS on AI behavior            │
│    - Hallucinations likely caught by test failures      │
└─────────────────┬────────────────────────────────────────┘
                  │
                  ▼
┌──────────────────────────────────────────────────────────┐
│ 5. Run Tests                                             │
│    ✓ All pass → Code likely correct                    │
│    ✗ Some fail → AI hallucinated or misunderstood      │
└─────────────────┬────────────────────────────────────────┘
                  │
                  ▼
┌──────────────────────────────────────────────────────────┐
│ 6. Human QA Final Review                                │
│    - Manual testing                                      │
│    - Code review for logic, security, performance       │
│    - Verify implementation matches intent               │
└──────────────────────────────────────────────────────────┘
```

### Miért működik?

**1. Tesztek = Executable Specification**

A tesztek **formális, executable specifikációk**, nem szabadszöveges leírások.

```typescript
// Szabadszöveges requirement (könnyen félreérthető):
// "Password must be at least 8 characters"

// Executable specification (egyértelmű):
it('should reject password shorter than 8 characters', () => {
  expect(validatePassword('short')).toBe(false);
  expect(validatePassword('1234567')).toBe(false);  // 7 chars - rejected
});

it('should accept password with 8 or more characters', () => {
  expect(validatePassword('12345678')).toBe(true);  // 8 chars - OK
  expect(validatePassword('verylongpassword')).toBe(true);
});

// FONTOS: Ha nincs uppercase requirement, nincs uppercase teszt!
// AI nem adhat hozzá extra validációt, mert a teszt fail-elne
```

**2. Negatív tesztek megakadályozzák a hallucináált funkciók hozzáadását**

```typescript
// Negative test: password WITHOUT uppercase is VALID
it('should accept password without uppercase letters', () => {
  expect(validatePassword('lowercase123')).toBe(true);  // ✓ MUST pass!
});

// Ha az AI hozzáadja az uppercase ellenőrzést, ez a teszt FAIL!
```

**3. Széles teszt lefedettség csökkenti a hallucináció valószínűségét**

```typescript
describe('Password Validation - AC-001-2', () => {
  // Boundary tests
  it('should reject 7 char password', () => { ... });  // boundary - 1
  it('should accept 8 char password', () => { ... });  // boundary
  it('should accept 9 char password', () => { ... });  // boundary + 1

  // Character type variations (negative tests!)
  it('should accept all lowercase', () => { ... });
  it('should accept all uppercase', () => { ... });
  it('should accept all numbers', () => { ... });
  it('should accept with spaces', () => { ... });
  it('should accept with special chars', () => { ... });

  // Edge cases
  it('should reject empty string', () => { ... });
  it('should reject null', () => { ... });
  it('should reject undefined', () => { ... });
  it('should handle unicode characters', () => { ... });
});
```

Mindegyik teszt egy **constraint** az AI számára. Ha az AI hozzáad extra logikát, a tesztek elkapják.

**4. Integration és E2E tesztek elkapják a "creative interpretation"-öket**

```typescript
// E2E test for US-001: User Registration
it('should allow user to register with simple password', async () => {
  // Given: user navigates to registration page
  await page.goto('/register');

  // When: user fills form with SIMPLE password (only lowercase + numbers)
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'simplepass123');  // No uppercase, no special!
  await page.click('button[type="submit"]');

  // Then: registration succeeds
  await expect(page).toHaveURL('/dashboard');
  await expect(page.locator('.welcome-message')).toContainText('Welcome');
});
```

Ha az AI hallucináál (pl. uppercase kötelező), ez az E2E teszt **fail-elni fog**!

---

## 📊 Test Generation Strategy

### 1. Test Pyramid for AI-Generated Code

```
         /\
        /  \
       / E2E \          <- Few, slow, comprehensive
      /--------\
     /          \
    / Integration \     <- Medium, moderate speed
   /--------------\
  /                \
 /   Unit Tests     \   <- Many, fast, specific
/____________________\
```

**Recommended distribution:**
- **70% Unit tests** - Each acceptance criterion → 5-10 unit tests
- **20% Integration tests** - Each workflow → 2-5 integration tests
- **10% E2E tests** - Each user story → 1-3 E2E tests

### 2. Test Types per Requirement Type

| Requirement Type | Test Types | Example Count |
|-----------------|------------|---------------|
| Acceptance Criterion | Unit tests | 5-10 per AC |
| Business Rule | Unit + Integration | 3-5 per rule |
| User Story | E2E + Integration | 2-4 per story |
| API Endpoint | Integration + Contract | 3-6 per endpoint |
| Workflow | Integration + E2E | 2-5 per workflow |

### 3. Test Categories to Generate

#### A. Positive Tests (Happy Path)
- Requirements explicit-en meghatározott viselkedés
- "Should do X when Y"

#### B. Negative Tests (Unhappy Path)
- **KRITIKUS a hallucináció ellen!**
- "Should NOT do X when Y"
- "Should accept X even though it lacks Y" (if Y not required)

#### C. Boundary Tests
- Min/max értékek
- Edge case-ek
- Off-by-one hibák elkapása

#### D. Error Handling Tests
- Invalid input
- Missing data
- Network failures (integration tests)

#### E. Security Tests
- SQL injection attempts
- XSS attempts
- Authentication/authorization checks

#### F. Performance Tests (optional in PoC)
- Response time requirements
- Load handling

---

## 🤖 AI Test Generation Process

### Step 1: Analyze Requirements

AI receives:
```markdown
## US-001: User Registration

As a new user, I want to register with email and password.

**Acceptance Criteria:**
- [AC-001-1] Email must be valid format
- [AC-001-2] Password must be at least 8 characters
- [AC-001-3] User receives confirmation email
```

### Step 2: Generate Test Specification

AI generates test plan BEFORE generating tests:

```markdown
## Test Plan for US-001

### Unit Tests (18 total)

**AC-001-1: Email validation (8 tests)**
1. ✓ Valid email formats (gmail, corporate, subdomain)
2. ✗ Invalid formats (missing @, missing domain, etc.)
3. ⚠ Edge cases (unicode, very long, etc.)

**AC-001-2: Password length (6 tests)**
1. ✓ 8+ character passwords accepted
2. ✗ <8 character passwords rejected
3. ⚠ Boundary (exactly 8 chars)
4. ⚠ Edge cases (empty, null, very long)
5. ✓ NEGATIVE: No uppercase requirement (test lowercase-only passes)
6. ✓ NEGATIVE: No special char requirement (test alphanumeric passes)

**AC-001-3: Confirmation email (4 tests)**
1. ✓ Email sent after registration
2. ✓ Email contains correct recipient
3. ✗ Registration fails if email sending fails
4. ⚠ Idempotency (retry doesn't send duplicate)

### Integration Tests (4 total)
1. Full registration flow (API level)
2. Email service integration
3. Database persistence
4. Duplicate email handling

### E2E Tests (2 total)
1. Happy path: complete registration
2. Error path: duplicate email
```

**Human reviews this test plan** and approves/modifies before test generation.

### Step 3: Generate Actual Tests

```typescript
// tests/auth/registration.test.ts

/**
 * @traceability US-001 (requirements/user-stories.md#us-001)
 */
describe('US-001: User Registration', () => {

  describe('AC-001-1: Email validation', () => {
    /**
     * @implements AC-001-1
     */
    it('should accept valid gmail address', async () => {
      const result = await registerUser('user@gmail.com', 'password123');
      expect(result.success).toBe(true);
    });

    /**
     * @implements AC-001-1
     */
    it('should accept corporate email', async () => {
      const result = await registerUser('user@company.co.uk', 'password123');
      expect(result.success).toBe(true);
    });

    /**
     * @implements AC-001-1
     */
    it('should reject email without @', async () => {
      await expect(
        registerUser('invalid-email', 'password123')
      ).rejects.toThrow('Invalid email format');
    });

    // ... more email validation tests
  });

  describe('AC-001-2: Password length', () => {
    /**
     * @implements AC-001-2
     */
    it('should accept 8 character password', async () => {
      const result = await registerUser('user@test.com', '12345678');
      expect(result.success).toBe(true);
    });

    /**
     * @implements AC-001-2
     */
    it('should reject 7 character password', async () => {
      await expect(
        registerUser('user@test.com', '1234567')
      ).rejects.toThrow('Password must be at least 8 characters');
    });

    /**
     * @implements AC-001-2
     * NEGATIVE TEST: Ensures AI doesn't add uppercase requirement
     */
    it('should accept lowercase-only password', async () => {
      const result = await registerUser('user@test.com', 'lowercasepass');
      expect(result.success).toBe(true);
    });

    /**
     * @implements AC-001-2
     * NEGATIVE TEST: Ensures AI doesn't add special char requirement
     */
    it('should accept alphanumeric password without special chars', async () => {
      const result = await registerUser('user@test.com', 'password123');
      expect(result.success).toBe(true);
    });

    // ... more password tests
  });

  describe('AC-001-3: Confirmation email', () => {
    /**
     * @implements AC-001-3
     */
    it('should send confirmation email after registration', async () => {
      const emailSpy = jest.spyOn(emailService, 'send');

      await registerUser('user@test.com', 'password123');

      expect(emailSpy).toHaveBeenCalledWith({
        to: 'user@test.com',
        subject: expect.stringContaining('confirmation'),
        // ...
      });
    });

    // ... more email tests
  });
});
```

### Step 4: Generate Implementation

NOW the AI generates code that must pass ALL these tests.

```typescript
// src/auth/registration.ts

/**
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

  // NOTE: AI CANNOT add uppercase/special char requirements here
  // because the negative tests would FAIL!

  const user = await db.users.create({
    email,
    password: hashPassword(password)
  });

  // @implements AC-001-3: Send confirmation email
  await emailService.send({
    to: user.email,
    subject: 'Welcome! Please confirm your email',
    body: generateConfirmationEmail(user)
  });

  return user;
}
```

### Step 5: Run Tests

```bash
npm test

PASS  tests/auth/registration.test.ts
  US-001: User Registration
    AC-001-1: Email validation
      ✓ should accept valid gmail address (12ms)
      ✓ should accept corporate email (8ms)
      ✓ should reject email without @ (5ms)
      ...
    AC-001-2: Password length
      ✓ should accept 8 character password (10ms)
      ✓ should reject 7 character password (7ms)
      ✓ should accept lowercase-only password (9ms)  <- CATCHES HALLUCINATION!
      ✓ should accept alphanumeric without special (8ms) <- CATCHES HALLUCINATION!
      ...
    AC-001-3: Confirmation email
      ✓ should send confirmation email (15ms)
      ...

Tests: 18 passed, 18 total
```

**Ha az AI hallucináál:**

```bash
FAIL  tests/auth/registration.test.ts
  US-001: User Registration
    AC-001-2: Password length
      ✗ should accept lowercase-only password (9ms)

        ValidationError: Password must contain at least one uppercase letter

        Expected: success
        Received: ValidationError
```

**→ AI hallucináció detektálva! Javítás szükséges.**

---

## 🛡️ Hallucináció Detektálási Mechanizmusok

### 1. Negative Tests (Elsődleges védelem)

```typescript
// Ha requirements NEM mondja, hogy uppercase kell:
describe('Password does NOT require uppercase (explicit negative test)', () => {
  it('should accept password without uppercase', () => {
    expect(validatePassword('lowercase123')).toBe(true);
  });
});
```

### 2. Explicit "Should NOT" Tests

```typescript
describe('What the system should NOT do', () => {
  it('should NOT automatically capitalize user input', () => {
    const user = createUser({ name: 'john' });
    expect(user.name).toBe('john');  // NOT 'John'
  });

  it('should NOT add default role if not specified', () => {
    const user = createUser({ email: 'test@example.com' });
    expect(user.role).toBeUndefined();  // NOT 'user' or 'guest'
  });
});
```

### 3. Boundary Crossing Tests

```typescript
// Requirement: "Users can have up to 5 projects"
it('should allow exactly 5 projects', () => {
  // ...
  expect(result.success).toBe(true);
});

it('should reject 6th project', () => {
  // ...
  expect(result.error).toBe('Maximum 5 projects allowed');
});

// Hallucination detection: AI might add "premium users can have 10"
it('should NOT allow more than 5 projects even for premium users', () => {
  // ...
  expect(result.error).toBe('Maximum 5 projects allowed');
});
```

### 4. Idempotency & Side Effect Tests

```typescript
// Detects hallucinated side effects
it('should NOT send multiple emails on retry', async () => {
  const emailSpy = jest.spyOn(emailService, 'send');

  await registerUser('test@example.com', 'password123');
  await registerUser('test@example.com', 'password123');  // Retry/duplicate

  expect(emailSpy).toHaveBeenCalledTimes(1);  // Not 2!
});
```

### 5. Integration Tests with Real Dependencies

```typescript
// Unit test might pass with mocks, but integration test catches hallucination
it('should create user in database with correct schema', async () => {
  await registerUser('test@example.com', 'password123');

  const user = await db.users.findByEmail('test@example.com');

  // Explicit schema check - catches hallucinated fields
  expect(Object.keys(user)).toEqual(['id', 'email', 'passwordHash', 'createdAt']);
  // NOT: ['id', 'email', 'passwordHash', 'createdAt', 'role', 'isVerified', ...]
});
```

---

## 📋 Test Generation Checklist for AI

Amikor az AI teszteket generál, ezt a checklistet követi:

### Per Acceptance Criterion:

- [ ] **3+ pozitív teszt** (happy path variations)
- [ ] **3+ negatív teszt** (invalid input, edge cases)
- [ ] **2+ boundary teszt** (min, max, off-by-one)
- [ ] **1+ "should NOT" teszt** (explicit negative behavior)
- [ ] **Minden teszt @implements annotációval** (traceability)
- [ ] **Minden teszt érthető névvel** (self-documenting)

### Per User Story:

- [ ] **1-3 E2E teszt** (full user journey)
- [ ] **2-4 integration teszt** (key workflows)
- [ ] **Edge case coverage** (error paths, unhappy flows)
- [ ] **@traceability annotáció** a user story-hoz

### Global Checks:

- [ ] **Nincs duplicated logic** a tesztekben
- [ ] **Tesztek egymástól függetlenek** (isolation)
- [ ] **Fast execution** (unit tests <100ms, integration <1s)
- [ ] **Determinisztikus** (nem flaky)
- [ ] **Readable** (AAA pattern: Arrange, Act, Assert)

---

## 🔄 Iteratív Test-Fix Cycle

```
┌─────────────────────────────────────────┐
│ 1. AI generates tests from requirements │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│ 2. Human reviews & approves tests       │
│    - Are they complete?                  │
│    - Do they match requirements exactly? │
│    - Are there hallucinated test cases? │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│ 3. AI generates implementation          │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│ 4. Run tests                             │
└──────────────┬──────────────────────────┘
               │
          ┌────┴────┐
          │         │
       PASS        FAIL
          │         │
          │         ▼
          │    ┌─────────────────────────────┐
          │    │ 5. Analyze failures:         │
          │    │    - Hallucination?          │
          │    │    - Misunderstanding?       │
          │    │    - Bug in test?            │
          │    └──────────┬──────────────────┘
          │               │
          │               ▼
          │    ┌─────────────────────────────┐
          │    │ 6. Fix (AI or Human)         │
          │    │    - If hallucination: regen │
          │    │    - If bug: fix code        │
          │    │    - If test bug: fix test   │
          │    └──────────┬──────────────────┘
          │               │
          │               │
          └───────────────┴─> LOOP until all pass
                          │
                          ▼
               ┌─────────────────────────────┐
               │ 7. Human QA                  │
               │    - Manual testing          │
               │    - Code review             │
               │    - Security review         │
               └─────────────────────────────┘
```

---

## 📊 Metrics & Monitoring

### Test Quality Metrics

```typescript
interface TestQualityMetrics {
  // Coverage
  requirementCoverage: number;       // % of ACs with tests
  codeCoverage: number;              // % of code lines covered
  branchCoverage: number;            // % of code branches covered

  // Test distribution
  unitTestCount: number;
  integrationTestCount: number;
  e2eTestCount: number;
  testPyramidRatio: string;          // e.g., "70:20:10"

  // Quality indicators
  negativeTestRatio: number;         // % of tests that are negative
  boundaryTestCount: number;
  avgTestsPerAC: number;

  // Hallucination detection
  failedDueToHallucination: number;  // Tests that caught AI hallucinations
  hallucinationRate: number;         // % of generations with hallucinations
}
```

### Example Dashboard

```
┌──────────────────────────────────────────────────────┐
│            Test Quality Dashboard                    │
├──────────────────────────────────────────────────────┤
│                                                       │
│  📊 Coverage                                         │
│    Requirement Coverage:  100% ████████████████  ✓  │
│    Code Coverage:          92% █████████████░░   ✓  │
│    Branch Coverage:        87% ████████████░░░   ✓  │
│                                                       │
│  🔺 Test Pyramid                                     │
│    Unit:         156 (72%) █████████████████░░░░░   │
│    Integration:   42 (19%) ████░░░░░░░░░░░░░░░░░   │
│    E2E:           18 ( 8%) ██░░░░░░░░░░░░░░░░░░░   │
│    Ratio: 72:19:8 ✓ (target: 70:20:10)              │
│                                                       │
│  🎯 Quality Indicators                               │
│    Negative tests:        28% ████████░░░░░░░░░░░   │
│    Boundary tests:        42 ✓                      │
│    Avg tests per AC:     8.3 ✓ (target: 5-10)       │
│                                                       │
│  🛡️ Hallucination Detection                          │
│    Caught hallucinations:  3 ⚠                      │
│    Hallucination rate:    12% (3/25 features)       │
│                                                       │
│  ⚡ Performance                                       │
│    Avg unit test time:    42ms ✓ (<100ms)           │
│    Avg integration time: 834ms ✓ (<1s)              │
│    Total test time:      2.3s  ✓                    │
│                                                       │
│  [View Details] [Export Report] [Refresh]            │
└──────────────────────────────────────────────────────┘
```

---

## 🚀 Integration into Loom Workflow

### Updated PoC Workflow with TDAI

```
1. Generate Requirements (existing)
   ↓
2. **Generate Test Specification**
   - AI analyzes requirements
   - AI creates test plan
   - Human approves test plan
   ↓
3. **Generate Tests**
   - Unit tests (70%)
   - Integration tests (20%)
   - E2E tests (10%)
   - Human reviews tests
   ↓
4. Generate Code (existing)
   - AI generates implementation
   - **Code must pass ALL tests**
   ↓
5. Run Tests
   - **Automated test execution**
   - **Hallucination detection**
   ↓
6. Traceability Validation (existing)
   - **Tests also have @traceability**
   ↓
7. Human QA (existing)
   - Manual testing
   - Code review
```

### CLI Commands (additions to PoC)

```bash
# Generate tests from requirements
loom generate tests --from-user-story US-001

# Generate test plan (before generating tests)
loom generate test-plan --from-user-story US-001

# Run tests and report hallucinations
loom test --detect-hallucinations

# Test quality metrics
loom test metrics

# Generate code that passes tests
loom generate code --from-tests
```

---

## ✅ Benefits of Test-Driven AI Development

### 1. **Hallucination Mitigation**

**Before TDAI:**
```
AI generates code → Code "works" → Human discovers hallucination later
→ Wasted time, technical debt
```

**After TDAI:**
```
AI generates tests → AI generates code → Tests fail (hallucination caught)
→ Fix immediately, no wasted work
```

### 2. **Executable Requirements**

Requirements are not just text, they're **executable specifications**.

### 3. **Confidence in AI-Generated Code**

If all tests pass (especially negative tests), hallucination probability is LOW.

### 4. **Faster Iteration**

```
Without tests:
  Generate code → Manual test → Find bug → Fix → Manual retest
  (30+ min per iteration)

With tests:
  Generate code → Run tests (5s) → Find bug → Fix → Run tests (5s)
  (< 1 min per iteration)
```

### 5. **Documentation**

Tests **ARE** documentation. They show exactly what the system should (and should NOT) do.

### 6. **Regression Prevention**

Once tests pass, they prevent future regressions (AI or human changes).

### 7. **Traceability**

```typescript
/**
 * @traceability US-001
 * @implements AC-001-2
 */
it('should accept 8+ character password', () => { ... });
```

Tests are part of the traceability graph: `Requirement → Test → Code`

---

## ⚠️ Limitations & Challenges

### 1. **Test Generation Quality**

**Problem:** AI might hallucinate in TESTS too!

**Mitigation:**
- Human reviews test plan BEFORE test generation
- Test templates and patterns
- Negative test enforcement (mandatory)

### 2. **Over-specification**

**Problem:** Too many tests → brittle, hard to maintain

**Mitigation:**
- Guidelines: 5-10 tests per AC (not 50!)
- Focus on critical paths
- DRY in test helpers

### 3. **False Positives**

**Problem:** Test passes but code is still wrong

**Mitigation:**
- Integration and E2E tests (not just unit)
- Human QA as final gate
- Code review for logic

### 4. **Performance**

**Problem:** Thousands of tests → slow feedback

**Mitigation:**
- Parallel test execution
- Smart test selection (only run affected tests)
- Fast unit tests (<100ms each)

---

## 🎯 Success Criteria for TDAI in PoC

### Metrics:

- [ ] **90%+ hallucination detection rate** (tests catch AI mistakes)
- [ ] **Requirement coverage: 100%** (every AC has tests)
- [ ] **Test pyramid ratio: 70:20:10** (±5%)
- [ ] **Negative test ratio: ≥20%** (at least 1/5 tests are negative)
- [ ] **Test execution time: <5s** for full suite
- [ ] **Human test review time: <10 min** per user story

### Qualitative:

- [ ] Human QA confirms: "Tests accurately capture requirements"
- [ ] Developers trust: "If tests pass, code is probably correct"
- [ ] Tests are readable, self-documenting
- [ ] Test failures clearly indicate the problem

---

## 📚 Related Concepts & Prior Art

### Industry Practices:

1. **Test-Driven Development (TDD)** - Write tests first, then code
2. **Behavior-Driven Development (BDD)** - Tests as specifications (Given/When/Then)
3. **Property-Based Testing** - Generate many test cases from properties
4. **Mutation Testing** - Test the tests (are they effective?)

### AI-Specific:

5. **Constrained Decoding** - Limit AI output to valid space
6. **Chain-of-Thought Testing** - AI explains reasoning in tests
7. **Self-Consistency Checking** - Generate multiple versions, compare

### Our Approach (TDAI):

Combines TDD + BDD + AI code generation + traceability

**Unique aspects:**
- Tests generated by AI, reviewed by human
- Negative tests mandatory (hallucination detection)
- Integrated with traceability system
- Part of docs-code sync workflow

---

## 💭 Conclusion

**Test-Driven AI Development is NOT just "generating tests".**

It's a **fundamental paradigm shift** in how we use AI for code generation:

**Old paradigm:**
```
Requirements → AI generates code → Hope it's correct → Manual testing finds bugs
```

**New paradigm (TDAI):**
```
Requirements → AI generates tests (constraints) → AI generates code (constrained)
            → Tests validate (automated) → High confidence
```

**Key insight:**

> **Tests are not just validation — they're CONSTRAINTS on AI behavior.**
>
> Negative tests especially: they tell AI what NOT to do.
>
> This is the hallucination antidote.

**Integration with Loom:**

TDAI + Traceability + AI-generated docs = **Self-validating, self-documenting system**

```
Requirements (docs) → Tests (executable specs) → Code (implementation)
        ↓                    ↓                         ↓
    Traceability ←──────────────────────────────────────┘
        ↓
   Validation (AI checks consistency automatically)
```

**This is the future of AI-assisted development.**

---

*Next step: Integrate TDAI into PoC tooling design.*

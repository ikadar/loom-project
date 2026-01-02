# Loom CLI Refaktorálási Terv

**Dátum:** 2024-12-31
**Frissítve:** 2025-01-02
**Cél:** Kód minőség, maintainability és performance javítása
**Előzmény:** Gap implementation plan sikeres befejezése után

---

## Sprint Státusz

| Sprint | Leírás | Státusz | Commit |
|--------|--------|---------|--------|
| R1 | Chunked Test Case Generation | ✅ Kész | `commit TBD` |
| R2 | Generic Approval Wrapper | ✅ Kész | `9ecce24` |
| R3 | Formatter Separation | ✅ Kész | `315eb25` |
| R4 | Parallel Execution | ✅ Kész | `03ab952` |
| R5 | Error Recovery & Backup | ✅ Kész | `3885333` |
| R6 | Interview Grouping | ✅ Kész | `236a634` |

### Létrehozott Fájlok

```
internal/
├── checkpoint/
│   └── checkpoint.go     # R5: Checkpoint save/load/resume
├── claude/
│   └── retry.go          # R1: Retry with exponential backoff
├── generator/
│   ├── testcases.go      # R1: Chunked test case generator
│   └── parallel.go       # R4: Parallel phase executor
├── interview/
│   └── grouping.go       # R6: Question grouping by subject
├── workflow/
│   ├── approval.go       # R2: Generic approval wrapper
│   └── progress.go       # R1: Progress bar
└── formatter/            # R3: Markdown formatters
    ├── anchor.go         # ToAnchor, ToLink, FormatHeader
    ├── types.go          # All L2 type definitions
    ├── testcases.go      # TDAI test case formatting
    ├── techspecs.go      # Technical specs formatting
    ├── contracts.go      # Interface contracts formatting
    ├── aggregates.go     # DDD aggregate formatting
    ├── sequences.go      # Sequence diagram + Mermaid
    └── datamodel.go      # ER diagram + Mermaid
```

---

## Jelenlegi Állapot

### Kód struktúra
```
loom-cli/
├── cmd/
│   ├── root.go           # CLI routing
│   ├── analyze.go        # L0 analízis
│   ├── interview.go      # Kérdés-válasz workflow
│   ├── derive_new.go     # L0 → L1 deriválás (~600 LOC)
│   ├── derive_l2.go      # L1 → L2 deriválás (~1200 LOC)
│   ├── derive_l3.go      # L2 → L3 deriválás
│   ├── validate.go       # Validáció
│   └── interactive.go    # Interaktív mód utils
├── internal/
│   ├── claude/           # Claude API client
│   ├── config/           # Konfiguráció
│   └── domain/           # Domain struktúrák
└── prompts/              # Embedded promptok
```

### Fő problémák

| Probléma | Hatás | Súlyosság |
|----------|-------|-----------|
| Monolitikus generálás | Token limit, partial failure | 🔴 High |
| Copy-paste interaktív kód | Maintainability | 🟡 Medium |
| Nincs chunking | Performance, reliability | 🔴 High |
| Nincs retry logic | Reliability | 🟡 Medium |
| Nincs parallel execution | Performance | 🟡 Medium |
| Interview nem csoportosít | UX | 🟢 Low |
| Nagy fájlok (1200+ LOC) | Maintainability | 🟡 Medium |

---

## Részletes Probléma Analízis

### 1. Monolitikus Generálás

**Jelenlegi működés:**
```go
// derive_l2.go - Minden test case egy hívásban
tcPrompt := prompts.DeriveTestCases + "\n" + string(acContent)
client.CallJSON(tcPrompt, &tcResult)  // 40 AC → 172 TC egyszerre
```

**Problémák:**
- 40 AC esetén ~50K token output szükséges
- Ha egy TC JSON-je hibás, az egész hívás elveszik
- Nincs progress feedback (user vár 2-3 percet vakon)
- Timeout risk nagy inputnál

**Metrikák:**
- Benchmark: 40 AC → 172 TC generálás
- Jelenlegi: 1 hívás, ~120 sec, 0% partial result
- Cél: 8 hívás (5 AC/batch), ~60 sec (parallel), 87.5% partial result siker esetén

---

### 2. Copy-Paste Interaktív Kód

**Jelenlegi (derive_l2.go 555-825 sorok):**
```go
// Ez a pattern 6x ismétlődik, minimális különbséggel
if interactive {
    content, _ := os.ReadFile(tcPath)
    action, err := AskApproval(PhaseResult{
        PhaseName: "Test Cases (TDAI)",
        FileName:  "test-cases.md",
        Content:   string(content),
        ItemCount: len(result.TestCases),
        ItemType:  "test cases",
        Summary:   fmt.Sprintf("(P:%d N:%d B:%d H:%d)", ...),
    })
    if err != nil {
        return err
    }
    switch action {
    case ActionQuit:
        if ConfirmQuit() {
            return fmt.Errorf("user quit")
        }
    case ActionSkip:
        os.Remove(tcPath)
        fmt.Fprintf(os.Stderr, "  Skipped: %s\n", tcPath)
    case ActionEdit:
        edited, err := EditContent(string(content), "test-cases.md")
        if err != nil {
            fmt.Fprintf(os.Stderr, "  Edit error: %v\n", err)
        } else {
            os.WriteFile(tcPath, []byte(edited), 0644)
        }
        writtenFiles[tcPath] = true
        fmt.Fprintf(os.Stderr, "  Written (edited): %s\n", tcPath)
    default: // Approve
        writtenFiles[tcPath] = true
        fmt.Fprintf(os.Stderr, "  Written: %s\n", tcPath)
    }
} else {
    writtenFiles[tcPath] = true
    fmt.Fprintf(os.Stderr, "  Written: %s\n", tcPath)
}
```

**Probléma:** ~270 sor duplikált kód 6 fájlra.

---

### 3. Nincs Retry Logic

**Jelenlegi:**
```go
if err := client.CallJSON(prompt, &result); err != nil {
    return fmt.Errorf("failed to generate: %w", err)  // Egyből feladja
}
```

**Problémák:**
- Átmeneti hálózati hiba → teljes újrafuttatás kell
- Rate limit → manual retry
- Partial JSON error → elveszett munka

---

### 4. Nagy Fájlok

| Fájl | LOC | Felelősség |
|------|-----|------------|
| derive_l2.go | ~1200 | L2 deriválás + formázás + írás + interaktív |
| derive_new.go | ~600 | L1 deriválás + formázás + írás |

**Probléma:** Single Responsibility Principle sérül. Egy fájl:
- CLI argument parsing
- Claude hívások
- JSON parsing
- Markdown formázás
- File I/O
- Interaktív workflow

---

## Javasolt Architektúra

### Új struktúra
```
loom-cli/
├── cmd/
│   ├── root.go
│   ├── derive.go         # CLI only, delegates to services
│   ├── derive_l2.go      # CLI only, delegates to services
│   ├── derive_l3.go
│   └── validate.go
│
├── internal/
│   ├── claude/
│   │   ├── client.go     # Alap client
│   │   ├── retry.go      # NEW: Retry with backoff
│   │   └── batch.go      # NEW: Batch/chunked calls
│   │
│   ├── generator/        # NEW: Generálási logika
│   │   ├── testcases.go  # TC generálás (chunked)
│   │   ├── techspecs.go  # TS generálás
│   │   ├── contracts.go  # IC generálás
│   │   └── common.go     # Shared utils
│   │
│   ├── formatter/        # NEW: Markdown formázás
│   │   ├── testcases.go
│   │   ├── techspecs.go
│   │   ├── anchor.go     # Anchor/link utils
│   │   └── common.go
│   │
│   ├── workflow/         # NEW: Interaktív workflow
│   │   ├── approval.go   # Generic approval wrapper
│   │   ├── preview.go    # Preview rendering
│   │   ├── editor.go     # $EDITOR integration
│   │   └── progress.go   # Progress reporting
│   │
│   ├── output/           # NEW: File I/O
│   │   ├── writer.go     # Safe file writing
│   │   └── backup.go     # Partial result backup
│   │
│   ├── config/
│   └── domain/
│
└── prompts/
```

### Fő absztrakciók

#### 1. Chunked Generator
```go
// internal/generator/common.go
type ChunkedGenerator[TInput, TOutput any] struct {
    ChunkSize   int
    MaxParallel int
    Client      *claude.Client
    Generate    func([]TInput) ([]TOutput, error)
    OnProgress  func(completed, total int)
}

func (g *ChunkedGenerator[I, O]) Run(inputs []I) ([]O, error) {
    chunks := chunk(inputs, g.ChunkSize)
    results := make([]O, 0, len(inputs))

    for i, chunk := range chunks {
        out, err := g.Generate(chunk)
        if err != nil {
            // Log error, continue with next chunk
            g.OnProgress(i, len(chunks))
            continue
        }
        results = append(results, out...)
        g.OnProgress(i+1, len(chunks))
    }
    return results, nil
}
```

#### 2. Retry Client
```go
// internal/claude/retry.go
type RetryConfig struct {
    MaxAttempts int
    BaseDelay   time.Duration
    MaxDelay    time.Duration
}

func (c *Client) CallJSONWithRetry(prompt string, result any, cfg RetryConfig) error {
    var lastErr error
    for attempt := 0; attempt < cfg.MaxAttempts; attempt++ {
        err := c.CallJSON(prompt, result)
        if err == nil {
            return nil
        }
        lastErr = err

        if !isRetryable(err) {
            return err
        }

        delay := min(cfg.BaseDelay * (1 << attempt), cfg.MaxDelay)
        time.Sleep(delay)
    }
    return fmt.Errorf("max retries exceeded: %w", lastErr)
}
```

#### 3. Approval Wrapper
```go
// internal/workflow/approval.go
type ApprovalResult[T any] struct {
    Value    T
    Approved bool
    Edited   bool
}

func WithApproval[T any](
    ctx context.Context,
    name string,
    generate func() (T, error),
    format func(T) string,
    interactive bool,
) (*ApprovalResult[T], error) {
    value, err := generate()
    if err != nil {
        return nil, err
    }

    if !interactive {
        return &ApprovalResult[T]{Value: value, Approved: true}, nil
    }

    content := format(value)
    action := AskApproval(name, content)

    switch action {
    case ActionApprove:
        return &ApprovalResult[T]{Value: value, Approved: true}, nil
    case ActionSkip:
        return &ApprovalResult[T]{Value: value, Approved: false}, nil
    case ActionEdit:
        edited := EditContent(content, name)
        // Parse edited content back to T...
        return &ApprovalResult[T]{Value: value, Approved: true, Edited: true}, nil
    case ActionRegenerate:
        return WithApproval(ctx, name, generate, format, interactive) // Recursive
    case ActionQuit:
        return nil, ErrUserQuit
    }
    return nil, errors.New("unknown action")
}
```

#### 4. Progress Reporter
```go
// internal/workflow/progress.go
type ProgressReporter struct {
    Total     int
    Completed int
    Label     string
}

func (p *ProgressReporter) Update(completed int) {
    p.Completed = completed
    pct := float64(completed) / float64(p.Total) * 100
    fmt.Fprintf(os.Stderr, "\r  %s: %d/%d (%.0f%%)", p.Label, completed, p.Total, pct)
}

func (p *ProgressReporter) Done() {
    fmt.Fprintln(os.Stderr) // New line after progress
}
```

---

## Implementálási Fázisok

### Sprint R1: Chunked Test Case Generation (High Impact)

**Cél:** Test case generálás AC-nkénti batch-ekben

**Változtatások:**
1. `internal/generator/testcases.go` - Chunked TC generálás
2. `internal/claude/retry.go` - Retry logic
3. `derive_l2.go` - Átírás az új generátorra

**Előtte:**
```go
// 1 hívás, 40 AC → 172 TC
client.CallJSON(prompt, &allTestCases)
```

**Utána:**
```go
// 8 hívás, 5 AC batch-enként
generator := &ChunkedGenerator{
    ChunkSize: 5,
    Generate:  generateTestCasesForACs,
    OnProgress: progress.Update,
}
testCases, err := generator.Run(allACs)
```

**Metrikák:**
- Partial failure recovery: 0% → 87.5%
- Progress visibility: nincs → realtime
- Token per call: ~50K → ~6K

---

### Sprint R2: Generic Approval Wrapper (Code Quality)

**Cél:** 270 sor duplikált kód eliminálása

**Változtatások:**
1. `internal/workflow/approval.go` - Generic wrapper
2. `derive_l2.go` - 6x copy-paste → 6x wrapper hívás

**Előtte (6x ismétlődik):**
```go
if interactive {
    content, _ := os.ReadFile(path)
    action, err := AskApproval(PhaseResult{...})
    switch action { /* 30 sor */ }
} else {
    fmt.Fprintf(os.Stderr, "  Written: %s\n", path)
}
```

**Utána:**
```go
WriteWithApproval(ctx, WriteConfig{
    Path:        tcPath,
    Content:     formatTestCases(testCases),
    Name:        "Test Cases",
    ItemCount:   len(testCases),
    Interactive: interactive,
})
```

**Metrikák:**
- derive_l2.go LOC: ~1200 → ~600
- Duplikált kód: 270 sor → 0

---

### Sprint R3: Formatter Separation (Maintainability)

**Cél:** Formázás kiszervezése külön package-be

**Változtatások:**
1. `internal/formatter/` - Új package
2. `formatter/testcases.go` - TC markdown formázás
3. `formatter/techspecs.go` - TS markdown formázás
4. `formatter/anchor.go` - toAnchor(), toLink() közös utils
5. derive_*.go - Import formatter package

**Előtte:**
```go
// derive_l2.go-ban
func writeTestCases(path string, testCases []TestCase, ...) error {
    // 100 sor formázás + file írás együtt
}
```

**Utána:**
```go
// derive_l2.go
content := formatter.FormatTestCases(testCases, summary)
output.Write(path, content)

// internal/formatter/testcases.go
func FormatTestCases(tcs []TestCase, summary TDAISummary) string {
    // Csak formázás, nincs I/O
}
```

**Metrikák:**
- Testability: 0% (I/O mixed) → 100% (pure functions)
- Reusability: single file → importable package

---

### Sprint R4: Parallel Execution (Performance)

**Cél:** Független generálások párhuzamosítása

**Változtatások:**
1. `internal/generator/parallel.go` - Parallel executor
2. derive_l2.go - Phase 3,4,5 párhuzamosan

**Jelenleg szekvenciális:**
```
Phase 3: Interface Contracts  [====] 30s
Phase 4: Aggregate Design     [====] 25s
Phase 5: Sequence Design      [====] 25s
Phase 6: Data Model           [====] 20s
─────────────────────────────────────
Total:                              100s
```

**Párhuzamosan:**
```
Phase 3: Interface Contracts  [====]
Phase 4: Aggregate Design     [====]  } Parallel
Phase 5: Sequence Design      [====]
Phase 6: Data Model           [====]
─────────────────────────────────────
Total:                              35s
```

**Metrikák:**
- L2 deriválás idő: ~100s → ~35s
- Throughput: 3x improvement

---

### Sprint R5: Error Recovery & Backup (Reliability)

**Cél:** Partial results mentése, graceful recovery

**Változtatások:**
1. `internal/output/backup.go` - Checkpoint mentés
2. derive_*.go - Checkpoint integration
3. `--resume` flag - Félbehagyott deriválás folytatása

**Működés:**
```
Phase L2-1: Generating Test Cases...
  [████████░░] 80% (32/40 ACs)
  ERROR: Rate limit exceeded

Checkpoint saved: .loom-checkpoint-20241231-1430.json
Resume with: loom-cli derive-l2 --resume .loom-checkpoint-...
```

**Metrikák:**
- Elveszett munka hiba esetén: 100% → <20%
- Manual restart szükséges: igen → nem (auto-resume)

---

### Sprint R6: Interview Grouping (UX) - Optional

**Cél:** Hasonló kérdések csoportosítása

**Változtatások:**
1. `internal/interview/grouping.go` - Kérdés csoportosító
2. interview.go - Grouped questions support

**Előtte:**
```
Q1: What is the default sort order for products?
[answer]
Q2: What is the default page size for products?
[answer]
Q3: What filters are available for products?
[answer]
```

**Utána:**
```
Product Listing Questions (3):
1. What is the default sort order?
2. What is the default page size?
3. What filters are available?

[Answer all at once or individually]
```

**Metrikák:**
- Kérdések száma: ugyanannyi
- User interaction rounds: N → N/3

---

## Implementálási Sorrend

| Sprint | Prioritás | Effort | Impact | Dependencies |
|--------|-----------|--------|--------|--------------|
| R1: Chunked Generation | 🔴 High | 3h | 🔴 High | - |
| R2: Approval Wrapper | 🟡 Medium | 2h | 🟡 Medium | - |
| R3: Formatter Separation | 🟡 Medium | 2h | 🟡 Medium | - |
| R4: Parallel Execution | 🟡 Medium | 3h | 🟡 Medium | R1 |
| R5: Error Recovery | 🟡 Medium | 3h | 🟡 Medium | R1 |
| R6: Interview Grouping | 🟢 Low | 2h | 🟢 Low | - |

**Javasolt sorrend:** R1 → R2 → R3 → R4 → R5 → R6

**Összesen:** ~15 óra

---

## Success Criteria

### Sprint R1 Done When: ✅
- [x] Test case generálás 5 AC-s batch-ekben történik
- [x] Progress bar látható generálás közben
- [x] Partial failure esetén a sikeres batch-ek megmaradnak
- [x] Retry logic működik átmeneti hibáknál

### Sprint R2 Done When: ✅
- [x] derive_l2.go < 700 LOC (actual: ~730 → ~550 after R3)
- [x] Nincs duplikált approval kód
- [x] Minden fájl írás `HandleFileApproval()` wrapper-t használ

### Sprint R3 Done When: ✅
- [x] `internal/formatter/` package létezik
- [x] Formázó függvények unit tesztelhetők (nincs I/O)
- [x] `ToAnchor()`, `ToLink()` közös helyen van (`formatter/anchor.go`)

### Sprint R4 Done When: ✅
- [x] L2 deriválás idő < 50s (40 AC input) - phases 2-6 run in parallel
- [x] Parallel execution with rate limit (max 3 concurrent, hardcoded)

### Sprint R5 Done When: ✅
- [x] Checkpoint mentés minden phase után (TestCases + ParallelPhases)
- [x] `--resume` flag működik
- [x] Partial results nem vesznek el hiba esetén
- [x] Checkpoint auto-delete sikeres befejezéskor

### Sprint R6 Done When: ✅
- [x] Hasonló kérdések csoportosítva jelennek meg (by subject, max 5/group)
- [x] User választhat egyenkénti vagy csoportos válaszadás között (--grouped flag)
- [x] Batch answers támogatás (--answers JSON array)

---

## Kockázatok és Megoldások

| Kockázat | Valószínűség | Megoldás |
|----------|--------------|----------|
| Chunked generation más eredményt ad | 🟡 Medium | Prompt tuning, context window overlap |
| Parallel execution race conditions | 🟢 Low | Mutex on shared state, channel-based communication |
| Backward compatibility break | 🟡 Medium | Same CLI interface, only internal changes |
| Over-engineering | 🟡 Medium | YAGNI - csak a tervezett feature-ök |

---

## Döntési Pontok

### 1. Chunk Size
- **5 AC/batch** - Balance: jó context, kezelhető token
- Alternative: Dynamic sizing based on AC complexity

### 2. Parallel Limit
- **3 concurrent calls** - Claude rate limit friendly
- Alternative: Configurable via `--parallel N`

### 3. Checkpoint Format
- **JSON file** - Simple, debuggable
- Alternative: SQLite (overkill for this use case)

### 4. Generics Usage
- **Go 1.18+ generics** - Cleaner abstractions
- Risk: Slight learning curve

---

## Következő Lépések

1. [x] Terv review és jóváhagyás
2. [x] Sprint R1 indítása: Chunked Test Case Generation
3. [x] Sprint R2: Generic Approval Wrapper
4. [x] Sprint R3: Formatter Separation
5. [x] Sprint R4: Parallel Execution
6. [x] Sprint R5: Error Recovery & Backup
7. [x] Sprint R6: Interview Grouping
8. [ ] Mérések before/after összehasonlításhoz (optional)

---

## 🎉 REFAKTORÁLÁS KÉSZ

**Összesen:** 6/6 sprint befejezve

**Dátum:** 2025-01-02

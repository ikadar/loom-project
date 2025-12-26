---
title: "Loom CLI Feature Backlog"
status: backlog
created: 2025-12-23
updated: 2025-12-26
note: "License/monetization a CLI-ből kikerült - lásd platform-business-model.md"
---

# Loom CLI Feature Backlog

> Feature ötletek a CLI továbbfejlesztéséhez.

**Megjegyzés:** A CLI ingyenes, a monetizáció a SaaS-ban történik. Lásd: `../business/platform-business-model.md`

---

## Historikus Kontextus (v0.3.0, 2025-12-23)

A CLI ekkor 4 parancsot támogatott:
- `analyze` - Phase 0-3: domain discovery, ambiguity detection
- `interview` - Phase 4: iteratív structured interview
- `derive` - Phase 5-6: AC + BR generálás
- `help/version` - segédparancsok

---

## 1. AI-alapú Dependency Detection

**Probléma:** A jelenlegi heurisztikus dependency detection (pl. "after deletion" → skip if "cannot delete") nem elég intelligens.

**Megoldás:** Az analyze fázisban a Claude elemezze a kérdések közötti függőségeket.

```go
// prompts/dependency_analysis.go
const DependencyAnalysisPrompt = `
Analyze these ambiguity questions and identify dependencies.
A question B depends on question A if:
- B asks about consequences of something A defines
- B only makes sense if A has a certain answer
- B is a follow-up to A

Return JSON:
{
  "dependencies": [
    {
      "question_id": "AMB-ENT-005",
      "depends_on": "AMB-ENT-004",
      "skip_if_answer_contains": ["cannot be deleted", "no deletion"]
    }
  ]
}
`
```

**Implementáció:**
1. Új fázis az analyze-ban: dependency analysis
2. Claude elemzi a kérdéseket párokban
3. Eredmény bekerül az ambiguities JSON-be

---

## 2. Interview Resume Funkció

**Probléma:** Ha a felhasználó megszakítja az interjút, újra kell kezdenie.

**Megoldás:** State file alapú resume.

```bash
# Folytatás korábbi állapotból
loom-cli interview --resume /tmp/loom-interview-12345.json
```

**Implementáció:**
- A state file már tartalmaz mindent (current_index, decisions, stb.)
- `--resume` flag: ne inicializáljon, csak folytassa
- Session ID megjelenítése a user-nek a resume-hoz

---

## 3. Batch Mode Improvements

**Jelenlegi:** `--batch` flag az összes kérdésre suggested answer-t használ.

**Bővítés:**
```bash
# Csak critical kérdéseket kérdezze, többit auto
loom-cli interview --auto-minor --state ...

# Adott kategóriát auto
loom-cli interview --auto-category entity --state ...

# Limit: max N kérdés interaktívan
loom-cli interview --max-questions 10 --state ...
```

---

## 4. Interview Progress Export

**Cél:** Interjú közben export a jelenlegi állapotról.

```bash
loom-cli interview --export-progress /tmp/progress.md --state ...
```

Kimenet:
```markdown
# Interview Progress Report

## Answered (15/71)
- [x] AMB-ENT-001: Quote statuses → "Full lifecycle..."
- [x] AMB-ENT-002: Expiration → "Auto-expire..."
...

## Skipped (3)
- AMB-ENT-005: Skipped due to AMB-ENT-004 answer

## Remaining (53)
- [ ] AMB-ENT-020: ...
```

---

## 5. Question Grouping

**Probléma:** Kérdések random sorrendben jönnek.

**Megoldás:** Intelligens csoportosítás.

```bash
loom-cli interview --group-by subject --state ...
```

Csoportosítási opciók:
- `subject` - entitás/művelet szerint
- `category` - entity/operation/ui szerint
- `severity` - critical először
- `dependency` - függőségi sorrend

---

## 6. Answer Validation

**Cél:** Válaszok validálása mielőtt rögzítjük.

```go
type ValidationRule struct {
    Pattern     string   `json:"pattern"`      // regex
    MinLength   int      `json:"min_length"`
    MaxLength   int      `json:"max_length"`
    MustContain []string `json:"must_contain"` // egyik kell
}
```

Példa: "Kérlek adj meg legalább egy konkrét állapotot" ha a válasz túl általános.

---

## 7. Undo/Edit Previous Answers

**Probléma:** Ha a user rájön, hogy rosszul válaszolt, nem tud visszamenni.

**Megoldás:**

```bash
# Visszalépés az előző kérdésre
loom-cli interview --back --state ...

# Adott válasz módosítása
loom-cli interview --edit AMB-ENT-003 --state ...
```

**Figyelem:** Ha egy korábbi válasz módosul, újra kell értékelni a skip-eket!

---

## 8. Multi-language Support

**Cél:** Kérdések és válaszok más nyelveken.

```bash
loom-cli analyze --language hu --input-file ...
loom-cli interview --language hu --state ...
```

Prompt módosítás:
```go
const DomainDiscoveryHU = `
Elemezd a követelmény dokumentumot és azonosítsd a domain modellt.
Adj vissza JSON-t magyar nyelvű leírásokkal...
`
```

---

## 9. Derivation Customization

**Cél:** AC/BR generálás testreszabása.

```bash
loom-cli derive --template custom-ac.tmpl --output-dir ...
loom-cli derive --format json --output-dir ...  # Markdown helyett JSON
loom-cli derive --include-decisions --output-dir ...  # Döntések beágyazása
```

---

## 10. Integration Features

### 10.1 Git Integration
```bash
loom-cli derive --git-commit --output-dir ...
# Auto-commit: "docs: derive L1 from user-story.md"
```

### 10.2 CI/CD Integration
```bash
loom-cli validate --input-dir ./specs
# Exit 1 ha hiányoznak döntések vagy AC-k
```

### 10.3 Watch Mode
```bash
loom-cli watch --input-dir ./specs/l0 --output-dir ./specs/l1
# L0 változáskor auto-analyze, notify about new ambiguities
```

---

## 11. Performance Optimizations

### 11.1 Parallel Analysis
Az entity és operation analysis párhuzamosan futhat.

### 11.2 Incremental Analysis
Csak a változott L0 fájlokat elemezze újra.

### 11.3 Caching
Claude válaszok cache-elése azonos inputra.

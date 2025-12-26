# Knowledge Navigation Architecture

Létrehozva: 2025-12-25
Módosítva: 2025-12-25

## Probléma

A szoftvertervezési tudás (requirements engineering, UX patterns, API design, stb.) sokezer oldalnyi anyag. Ha checklistekké desztilláljuk:
- A tudás nagy része elveszik (lossy compression)
- "Lebutított" szabályok maradnak
- Elvész a kontextus, a "miért"

## Megoldás: Navigáció a Desztilláció Helyett

A lényeg: **ne tömörítsük a tudást, hanem tegyük navigálhatóvá**.

---

## Tudástípusok Szétválasztása

**KRITIKUS:** Három különböző tudástípus létezik, eltérő kezeléssel:

| Tudástípus | Forrás | Kezelés | Példa |
|------------|--------|---------|-------|
| **Generic SW Engineering** | AI-DOP corpus | RAG (szelektív retrieval) | "Confirmation dialog patterns" |
| **Domain Knowledge** | User input (L0, interview) | Projekt state, mindig teljes | "Quote entity has expiration" |
| **LLM Native** | Training data | Implicit, nincs retrieval | "Mi az a REST API" |

### Domain Knowledge ≠ Corpus

A **Domain Knowledge NEM része a corpus-nak**. Ez a user inputból épül fokozatosan:

```
L0 input (user stories)
    ↓
analyze → domain_model.json (entities, operations)
    ↓
interview → decisions.json (business rules, constraints)
    ↓
derive → L1 docs (AC, BR)
    ↓
[minden eddigi output = akkumulált domain context]
```

Az LLM native képességei kezelik ezt a context window-ban. Nem kell RAG.

---

## Knowledge Navigation Architecture

Ez az architektúra **CSAK a Generic SW Engineering tudásra** vonatkozik:

```
┌─────────────────────────────────────────────────────────────┐
│  Knowledge Navigation (Generic SW Engineering Only)         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  1. KNOWLEDGE CORPUS                                │   │
│  │     ├─→ Requirements engineering patterns           │   │
│  │     ├─→ UX/UI design principles                     │   │
│  │     ├─→ API design best practices                   │   │
│  │     ├─→ Data modeling patterns                      │   │
│  │     ├─→ Testing strategies                          │   │
│  │     └─→ Architecture patterns                       │   │
│  │                                                     │   │
│  │     Format: Markdown + YAML metadata + embeddings   │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  2. KNOWLEDGE MAP                                   │   │
│  │     ├─→ Topic → knowledge atom mapping              │   │
│  │     ├─→ "When deriving AC, consult X"               │   │
│  │     ├─→ Prerequisites, cross-references             │   │
│  │     └─→ Depth recommendations per task type         │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  3. CONTEXT-AWARE ROUTER                            │   │
│  │     ├─→ Input: task type + domain context           │   │
│  │     ├─→ Query Knowledge Map                         │   │
│  │     ├─→ Semantic search in Corpus                   │   │
│  │     └─→ Output: relevant SW engineering knowledge   │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  4. DYNAMIC PROMPT ASSEMBLY                         │   │
│  │     ├─→ Base prompt template                        │   │
│  │     ├─→ + Retrieved SW engineering knowledge        │   │
│  │     ├─→ + Domain context (from project state)       │   │
│  │     └─→ = Complete prompt                           │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Részletes Komponensek

### 1. Knowledge Corpus (Generic SW Engineering)

A corpus CSAK szoftvertervezési tudást tartalmaz:

```yaml
# Példa: egy knowledge atom
id: "requirements/acceptance-criteria-patterns"
title: "Acceptance Criteria Patterns"
content: |
  Acceptance criteria define the conditions that must be met
  for a user story to be considered complete.

  ## Given-When-Then Format
  The most common format for AC is Gherkin-style:
  - Given: preconditions and initial context
  - When: the action or trigger
  - Then: expected outcomes

  ## Best Practices
  - One behavior per AC
  - Testable and measurable
  - Independent of implementation
  - Include edge cases
  ...
metadata:
  category: requirements-engineering
  depth: medium
  prerequisites: []
  related:
    - "requirements/user-story-structure"
    - "testing/behavior-driven-development"
  applicable_when:
    - "deriving L1 from L0"
    - "writing acceptance criteria"
embedding: [0.234, -0.567, ...]
```

```yaml
# Másik példa
id: "ux-patterns/confirmation-dialogs"
title: "Confirmation Dialog Patterns"
content: |
  Confirmation dialogs prevent accidental destructive actions
  and ensure user intent.

  ## When to Use
  - Irreversible actions (delete, submit, send)
  - Actions with significant consequences
  - Financial transactions

  ## Best Practices
  - Show what will happen clearly
  - Use descriptive button labels (not just "OK/Cancel")
  - Include summary of affected items
  - Allow undo when possible instead of confirming
  ...
metadata:
  category: ux-patterns
  applicable_when:
    - "designing confirmation flows"
    - "user accepts/rejects something"
```

```yaml
# Harmadik példa
id: "data-modeling/soft-delete-pattern"
title: "Soft Delete Pattern"
content: |
  Soft delete marks records as deleted instead of removing them.

  ## When to Use
  - Audit trail requirements
  - Undo functionality needed
  - Referential integrity concerns

  ## Implementation
  - Add `deleted_at` timestamp column
  - Filter queries to exclude deleted records
  - Consider archival strategy for old records
  ...
metadata:
  category: data-modeling
  applicable_when:
    - "entity deletion discussed"
    - "audit requirements present"
```

### 2. Knowledge Map

A map definiálja, hogy **melyik task típushoz** milyen SW engineering tudás releváns:

```yaml
# Task-based routing (NEM domain-based!)
task_type: "derive-acceptance-criteria"
relevant_knowledge:
  always:
    - "requirements/acceptance-criteria-patterns"
    - "requirements/given-when-then"
  if_domain_context_mentions:
    deletion:
      - "data-modeling/soft-delete-pattern"
      - "ux-patterns/confirmation-dialogs"
    user_input:
      - "ux-patterns/form-validation"
      - "requirements/input-validation-criteria"
    status_change:
      - "data-modeling/state-machine-patterns"
      - "requirements/state-transition-criteria"
```

```yaml
task_type: "derive-business-rules"
relevant_knowledge:
  always:
    - "requirements/business-rule-patterns"
    - "requirements/constraint-specification"
  if_domain_context_mentions:
    calculation:
      - "requirements/calculation-rules"
    authorization:
      - "security/authorization-patterns"
```

### 3. Context-Aware Router

```python
def get_relevant_knowledge(task_type: str, domain_context: dict) -> list:
    """
    task_type: "derive-acceptance-criteria", "derive-business-rules", etc.
    domain_context: from project state (entities, decisions, etc.)
    """

    # 1. Get always-relevant knowledge for this task type
    map_entry = knowledge_map.get(task_type)
    results = map_entry.always.copy()

    # 2. Check domain context for conditional knowledge
    context_text = json.dumps(domain_context).lower()
    for trigger, knowledge_ids in map_entry.if_domain_context_mentions.items():
        if trigger in context_text:
            results.extend(knowledge_ids)

    # 3. Fetch actual content from corpus
    knowledge_sections = corpus.get_many(results)

    return knowledge_sections
```

### 4. Dynamic Prompt Assembly

```python
def assemble_prompt(task_type: str, task: str, project_state: dict) -> str:
    # 1. Get relevant SW engineering knowledge
    sw_knowledge = get_relevant_knowledge(task_type, project_state)

    # 2. Get base template for task type
    template = templates.get(task_type)

    # 3. Assemble
    prompt = template.format(
        task=task,
        sw_engineering_knowledge=format_sections(sw_knowledge),
        domain_context=format_domain_context(project_state),
        # domain_context includes: entities, decisions, existing docs
    )

    return prompt
```

---

## Példa: Quote Acceptance Workflow

**Feladat:** Derive acceptance criteria for "Customer accepts quote online"

**Input:**
- task_type: "derive-acceptance-criteria"
- domain_context: { entities: ["Quote", "Customer"], decisions: [...] }

**Router működése:**

1. Always: "requirements/acceptance-criteria-patterns", "requirements/given-when-then"
2. Domain context mentions "accept" → ux-patterns/confirmation-dialogs
3. Domain context has "Quote" entity → data-modeling/state-machine-patterns

**Összeállított prompt:**

```markdown
# Task
Derive acceptance criteria for: "Customer accepts quote online"

# Software Engineering Knowledge

## Acceptance Criteria Patterns (requirements/acceptance-criteria-patterns)
Acceptance criteria define the conditions that must be met...
[Generic SW engineering knowledge - HOW to write good AC]

## Given-When-Then Format (requirements/given-when-then)
The most common format for AC is Gherkin-style...
[Generic SW engineering knowledge - format guidance]

## Confirmation Dialog Patterns (ux-patterns/confirmation-dialogs)
Confirmation dialogs prevent accidental actions...
[Generic SW engineering knowledge - UX considerations]

## State Machine Patterns (data-modeling/state-machine-patterns)
Entities with status fields benefit from explicit state modeling...
[Generic SW engineering knowledge - state transitions]

# Domain Context (from project state)

## Entities
- Quote: { statuses: [Draft, Pending, Accepted, Rejected], ... }
- Customer: { ... }

## Previous Decisions
- DEC-001: "Quotes expire after 30 days"
- DEC-002: "Acceptance requires explicit confirmation"

# Output
Generate acceptance criteria in Given/When/Then format:
```

**Megjegyzés:** A prompt tartalmazza:
- **SW engineering tudás** (corpus-ból) → hogyan írjunk jó AC-t
- **Domain context** (project state-ből) → mi az a Quote, milyen döntések születtek

---

## Depth Control

| Level | Mit tartalmaz | Mikor használjuk |
|-------|---------------|------------------|
| Surface | Summary + key points only | Quick validation, reviews |
| Medium | Full sections | Standard derivation |
| Deep | Full + examples + anti-patterns | Complex decisions, edge cases |

A depth a **task complexity** alapján választható, nem domain alapján.

---

## Implementációs Lépések

### Phase 1: Minimal Viable (Hardcoded)
1. Identify 10-20 core SW engineering knowledge atoms
2. Hardcode knowledge map (task_type → knowledge_ids)
3. Simple keyword matching for conditional routing
4. Test with derive-acceptance-criteria task

### Phase 2: Scalable
1. Move corpus to structured files (markdown + YAML frontmatter)
2. Add vector embeddings for semantic search
3. Implement proper router with semantic matching

### Phase 3: Full Navigation
1. Knowledge Map UI for maintenance
2. Depth control per request
3. Prerequisite resolution
4. Feedback loop (which knowledge was useful?)

---

## Growth Strategy: Start Small, Improve Continuously

A Knowledge Navigation Architecture lehetővé teszi, hogy **minimális knowledge base-zel induljunk**, és folyamatosan bővítsük:

```
┌─────────────────────────────────────────────────────────────┐
│  MVP Launch                          │  6+ months later    │
├──────────────────────────────────────┼─────────────────────┤
│  Corpus: 10-20 atoms                 │  Corpus: 200+ atoms │
│  Map: hardcoded YAML                 │  Map: data-driven   │
│  Router: keyword matching            │  Router: semantic   │
├──────────────────────────────────────┼─────────────────────┤
│  SAME ARCHITECTURE                   │  SAME ARCHITECTURE  │
│  SAME CLI                            │  SAME CLI           │
│  SAME API                            │  SAME API           │
└──────────────────────────────────────┴─────────────────────┘
```

### Miért Működik

| Komponens | MVP | Improvement | Code change? |
|-----------|-----|-------------|--------------|
| Corpus content | Minimal | Add atoms | ❌ Nincs |
| Knowledge Map | Hardcoded YAML | More rules | ❌ Nincs |
| Router logic | Keyword | Semantic | ✅ Upgrade |
| Prompt templates | Basic | Refined | ❌ Nincs |

**A knowledge base bővítése = content addition, nem code change.**

### Corpus Growth Path

```
# Induláskor (MVP)
corpus/
  requirements/
    acceptance-criteria-patterns.md
    given-when-then.md
  ux-patterns/
    confirmation-dialogs.md
  data-modeling/
    soft-delete-pattern.md
  # 10-20 file összesen

# 6 hónap múlva
corpus/
  requirements/
    acceptance-criteria-patterns.md
    given-when-then.md
    edge-case-identification.md       # NEW
    negative-testing-criteria.md      # NEW
    boundary-conditions.md            # NEW
  ux-patterns/
    confirmation-dialogs.md
    form-validation.md                # NEW
    error-messages.md                 # NEW
    loading-states.md                 # NEW
  data-modeling/
    soft-delete-pattern.md
    audit-trail-patterns.md           # NEW
    state-machine-patterns.md         # NEW
  api-design/                         # NEW category
    rest-conventions.md
    error-response-patterns.md
    pagination-patterns.md
  testing/                            # NEW category
    test-case-derivation.md
    boundary-testing.md
  # 200+ file
```

### Improvement Sources

1. **User feedback:** "This AC was missing X consideration" → add knowledge atom
2. **Usage analytics:** Which knowledge is retrieved but ignored? → improve or remove
3. **Expert review:** Periodically add new patterns from literature
4. **Community:** Accept knowledge contributions (if open model)
5. **Failure analysis:** When derivation quality is poor, what knowledge was missing?

### Quality Improvement Cycle

```
User runs derivation
        ↓
    Output quality?
        ↓
   ┌────┴────┐
   │         │
  Good      Poor
   │         │
   ↓         ↓
Log which   Analyze:
knowledge   - Missing knowledge?
was used    - Wrong knowledge retrieved?
   │        - Knowledge unclear?
   │         │
   └────┬────┘
        ↓
   Improve corpus/map
        ↓
   Deploy (no code change)
```

### Versioning Strategy

```yaml
# Knowledge atom with version
id: "requirements/acceptance-criteria-patterns"
version: "1.2.0"
changelog:
  - "1.2.0: Added boundary condition examples"
  - "1.1.0: Clarified Given-When-Then format"
  - "1.0.0: Initial version"
```

Ez lehetővé teszi:
- A/B testing különböző knowledge verziók között
- Rollback ha egy változás rontott a minőségen
- Audit trail a knowledge evolúciójáról

---

## Nyitott Kérdések

### 🔴 Kritikus
1. **Context window limit:** Mi történik, ha domain context + SW knowledge túl nagy?
2. **Knowledge Map = rejtett desztilláció?** A map manuális kurálást igényel - ez nem pont az, amit el akartunk kerülni?
3. **Dumb CLI vs RAG:** Ha a RAG a SaaS-ban van, minden CLI hívás = API call. Offline működés?

### 🟡 Közepes
4. **Generic vs domain határ:** Hol a pontos határ? "Form validation patterns" generic, de "quote acceptance validation" domain?
5. **Depth control:** Ki/mi dönt a depth-ről?
6. **Infrastruktúra komplexitás:** Vector DB, semantic search - túl sok MVP-hez?

### 🟢 Alacsony
7. **Prerequisites robbanás:** Max depth kell a prereq chain-hez
8. **Feedback loop:** Hogyan tanulunk abból, hogy melyik knowledge volt hasznos?

---

## Kapcsolódó Fájlok

- `platform-strategy.md` - Dumb CLI + Smart SaaS architektúra
- `loom-cli-next-steps.md` - CLI fejlesztési roadmap

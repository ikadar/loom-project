# L0→L1 Derivation Flow

> A teljes deriválási folyamat architektúrája és költségszerkezete.

## Költségszerkezet

```
┌─────────────────────────────────────────────────────────────┐
│ USER fizet:                                                 │
│ - Claude Code subscription (vagy Anthropic API)             │
│ - Minden Claude hívás az ő számlájára megy                  │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ LOOM SaaS fizet:                                            │
│ - M4-M5: Semmi LLM költség (statikus promptok)              │
│ - M6+: Gemini hívások (~$0.003/session)                     │
└─────────────────────────────────────────────────────────────┘
```

**Ez az architektúra nagy előnye:** A "nehéz" LLM munka költségét a felhasználó viseli (Claude Code-on keresztül), nem a Loom SaaS.

---

## M4-M5: MVP Flow (Gemini nélkül)

### Architektúra

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT (Claude Code)                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────┐                                                                │
│  │ User L0 │  (PRD, vision doc, stb.)                                       │
│  └────┬────┘                                                                │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────┐      ┌─────────────┐      ┌────────────────────────┐   │
│  │ 1. loom derive  │─────►│   SaaS      │─────►│ Prompts + Checklists   │   │
│  └────────┬────────┘      │  (fetch)    │      └───────────┬────────────┘   │
│           │               └─────────────┘                  │                │
│           ▼                                                ▼                │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 2. Domain Discovery                                                 │    │
│  │    L0 + DomainDiscovery prompt ──► Claude ──► Domain Model (JSON)   │    │
│  └─────────────────────────────────────────────────────────┬───────────┘    │
│                                                            │                │
│                                                            ▼                │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 3. Ambiguity Detection                                              │    │
│  │    Domain Model + EntityAnalysis prompt ──► Claude ──► Ambiguities  │    │
│  │    Domain Model + OperationAnalysis prompt ──► Claude ──► More Amb. │    │
│  └─────────────────────────────────────────────────────────┬───────────┘    │
│                                                            │                │
│                                                            ▼                │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 4. Structured Interview                                             │    │
│  │    Ambiguities + InterviewPrompt ──► Claude ──► Formatted Questions │    │
│  │                                                                     │    │
│  │    User answers questions interactively                             │    │
│  │    Decisions stored locally (.loom/decisions.json)                  │    │
│  └─────────────────────────────────────────────────────────┬───────────┘    │
│                                                            │                │
│                                                            ▼                │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 5. L1 Derivation                                                    │    │
│  │    Domain Model + Decisions + DerivationPrompt ──► Claude ──► L1    │    │
│  └─────────────────────────────────────────────────────────┬───────────┘    │
│                                                            │                │
│                                                            ▼                │
│  ┌─────────┐                                                                │
│  │ L1 Doc  │  (specs/L1-requirements.md)                                    │
│  └─────────┘                                                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                              SERVER (SaaS)                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐            │
│  │ License Check   │   │ Prompt Storage  │   │ Usage Tracking  │            │
│  │ (free/pro tier) │   │ (versioned)     │   │ (metering)      │            │
│  └─────────────────┘   └─────────────────┘   └─────────────────┘            │
│                                                                             │
│  LLM hívás: NINCS                                                           │
│  Költség: ~$0 (csak infra)                                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Példa Flow (MVP)

**Input:** User PRD egy e-commerce alkalmazásról

```
1. USER: loom derive specs/prd.md

2. CLI → SaaS: GET /api/prompts?type=derive&license=xxx
   SaaS → CLI: { prompts: [...], checklists: [...] }

3. CLI → Claude Code → Claude:
   "You are a domain analysis expert. Extract the domain model..."
   + [PRD content]

   Claude →: {
     "entities": [
       {"name": "Product", "attributes": ["id", "name", "price"]},
       {"name": "Order", "attributes": ["id", "status", "total"]},
       {"name": "User", "attributes": ["id", "email"]}
     ],
     "operations": [
       {"name": "CreateOrder", "actor": "User", "target": "Order"}
     ],
     ...
   }

4. CLI → Claude Code → Claude:
   "Analyze each entity against this checklist..."
   + [Domain Model]

   Claude →: {
     "ambiguities": [
       {
         "id": "AMB-ENT-001",
         "subject": "Order",
         "question": "Is order deletion soft or hard delete?",
         "options": ["Soft delete", "Hard delete"],
         "suggested": "Soft delete"
       },
       {
         "id": "AMB-ENT-002",
         "subject": "Product",
         "question": "Can product price be negative?",
         "options": ["Yes", "No"],
         "suggested": "No"
       }
     ]
   }

5. CLI → User (interactive):
   Q: Is order deletion soft or hard delete?
   > Soft delete

   Q: Can product price be negative?
   > No

   Saved to .loom/decisions.json

6. CLI → Claude Code → Claude:
   "Generate Acceptance Criteria and Business Rules..."
   + [Domain Model] + [Decisions]

   Claude →: {
     "acceptance_criteria": [
       {
         "id": "AC-ORD-001",
         "title": "Create Order",
         "given": "A logged-in user with items in cart",
         "when": "User submits order",
         "then": "Order is created with status 'pending'"
       }
     ],
     "business_rules": [
       {
         "id": "BR-PRD-001",
         "title": "Product Price Validation",
         "rule": "Product price must be non-negative",
         "invariant": "price >= 0"
       }
     ]
   }

7. OUTPUT: specs/L1-requirements.md
```

### Költségek (MVP)

| Fizető | Tétel | Költség |
|--------|-------|---------|
| User | Claude API (5 hívás) | ~$0.08-0.12/session |
| Loom SaaS | LLM | $0 |
| Loom SaaS | Infra | ~$20-50/hó |

---

## M6+: Flow Gemini-vel

### Architektúra

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT (Claude Code)                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────┐                                                                │
│  │ User L0 │                                                                │
│  └────┬────┘                                                                │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────┐      ┌─────────────┐      ┌────────────────────────┐   │
│  │ 1. loom derive  │─────►│   SaaS      │─────►│ Prompts + Checklists   │   │
│  │    + L0 snippet │      │  (smart)    │      │ + Domain Checklist     │   │
│  └────────┬────────┘      └─────────────┘      │ + RAG Context          │   │
│           │                      ▲             └───────────┬────────────┘   │
│           │                      │ Gemini                  │                │
│           │                      │ enhanced                │                │
│           ▼                                                ▼                │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 2. Domain Discovery (same as MVP)                                   │    │
│  │    L0 + DomainDiscovery prompt ──► Claude ──► Domain Model          │    │
│  └─────────────────────────────────────────────────────────┬───────────┘    │
│                                                            │                │
│       ┌────────────────────────────────────────────────────┘                │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────┐      ┌─────────────┐      ┌────────────────────────┐   │
│  │ 3. Get Context  │─────►│   SaaS      │─────►│ RAG Context            │   │
│  │    for domain   │      │  (Gemini)   │      │ (relevant patterns)    │   │
│  └────────┬────────┘      └─────────────┘      └───────────┬────────────┘   │
│           │                                                │                │
│           ▼                                                ▼                │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 4. Ambiguity Detection + Prioritization                             │    │
│  │    Domain Model + Checklists + RAG Context ──► Claude ──► Amb.      │    │
│  └─────────────────────────────────────────────────────────┬───────────┘    │
│                                                            │                │
│       ┌────────────────────────────────────────────────────┘                │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────┐      ┌─────────────┐      ┌────────────────────────┐   │
│  │ 5. Prioritize   │─────►│   SaaS      │─────►│ Sorted Ambiguities     │   │
│  │    ambiguities  │      │  (Gemini)   │      │ (critical first)       │   │
│  └────────┬────────┘      └─────────────┘      └───────────┬────────────┘   │
│           │                                                │                │
│           ▼                                                ▼                │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 6. Structured Interview (same as MVP, but prioritized)             │    │
│  └─────────────────────────────────────────────────────────┬───────────┘    │
│                                                            │                │
│                                                            ▼                │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 7. L1 Derivation (same as MVP)                                      │    │
│  └─────────────────────────────────────────────────────────┬───────────┘    │
│                                                            │                │
│       ┌────────────────────────────────────────────────────┘                │
│       │                                                                     │
│       ▼                                                                     │
│  ┌─────────────────┐      ┌─────────────┐      ┌────────────────────────┐   │
│  │ 8. Validate     │─────►│   SaaS      │─────►│ Validation Result      │   │
│  │    output       │      │  (Gemini)   │      │ (issues, suggestions)  │   │
│  └────────┬────────┘      └─────────────┘      └───────────┬────────────┘   │
│           │                                                │                │
│           ▼                                                ▼                │
│  ┌─────────┐                                                                │
│  │ L1 Doc  │  (validated)                                                   │
│  └─────────┘                                                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                              SERVER (SaaS + Gemini)                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐            │
│  │ License Check   │   │ Prompt Storage  │   │ Usage Tracking  │            │
│  └─────────────────┘   └─────────────────┘   └─────────────────┘            │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                         Gemini LLM Layer                            │    │
│  ├─────────────────────────────────────────────────────────────────────┤    │
│  │                                                                     │    │
│  │  ┌───────────────────┐  ┌───────────────────┐  ┌─────────────────┐  │    │
│  │  │ L0 Classification │  │ RAG Query Enhance │  │ Retrieval       │  │    │
│  │  │ (domain detect)   │  │ (query rewrite)   │  │ Reranking       │  │    │
│  │  └───────────────────┘  └───────────────────┘  └─────────────────┘  │    │
│  │                                                                     │    │
│  │  ┌───────────────────┐  ┌───────────────────┐                       │    │
│  │  │ Ambiguity         │  │ Output            │                       │    │
│  │  │ Prioritization    │  │ Validation        │                       │    │
│  │  └───────────────────┘  └───────────────────┘                       │    │
│  │                                                                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                         Knowledge Base (RAG)                        │    │
│  ├─────────────────────────────────────────────────────────────────────┤    │
│  │  Vector DB (embeddings) ◄──► SW Engineering Knowledge Corpus        │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Példa Flow (M6+ Gemini-vel)

**Input:** User PRD egy fintech alkalmazásról

```
1. USER: loom derive specs/prd.md

2. CLI → SaaS: POST /api/derive/init
   Body: { license: "xxx", l0_snippet: "Payment processing app..." }

   SaaS (Gemini - L0 Classification):
   "Classify this L0 document..."

   Gemini →: {
     "domain": "fintech",
     "subdomains": ["payments", "transactions"],
     "suggested_checklists": ["pci-dss", "audit-trail", "idempotency"]
   }

   SaaS (Gemini - RAG Query Enhancement):
   "Generate search queries for fintech payment patterns..."

   Gemini →: [
     "payment idempotency patterns",
     "transaction rollback strategies",
     "PCI compliance requirements"
   ]

   SaaS (Vector DB Search + Gemini Reranking):
   → Top 5 relevant knowledge chunks

   SaaS → CLI: {
     prompts: [...],
     checklists: [..., "pci-dss", "audit-trail"],  // domain-specific!
     rag_context: [
       "Idempotency: Always use idempotency keys for payment operations...",
       "PCI-DSS: Never log full card numbers..."
     ]
   }

3. CLI → Claude Code → Claude:
   "You are a domain analysis expert..."
   + [PRD content]
   + [RAG Context: payment patterns, PCI requirements]  // enhanced!

   Claude →: {
     "entities": [
       {"name": "Transaction", "attributes": ["id", "amount", "status"]},
       {"name": "PaymentMethod", "attributes": ["id", "type", "token"]},
       ...
     ],
     ...
   }

4. CLI → Claude Code → Claude:
   "Analyze each entity against this checklist..."
   + [Domain Model]
   + [Fintech-specific checklist: PCI, idempotency, audit]  // domain-aware!

   Claude →: {
     "ambiguities": [
       {"id": "AMB-001", "question": "Transaction retry strategy?", ...},
       {"id": "AMB-002", "question": "Idempotency key format?", ...},
       {"id": "AMB-003", "question": "Audit log retention period?", ...},
       ... (30 more)
     ]
   }

5. CLI → SaaS: POST /api/ambiguities/prioritize
   Body: { ambiguities: [...], domain: "fintech" }

   SaaS (Gemini - Prioritization):
   "Prioritize these ambiguities for a fintech app..."

   Gemini →: {
     "critical": ["AMB-001", "AMB-002", "AMB-015"],  // payment-critical
     "important": ["AMB-003", "AMB-008"],
     "minor": ["AMB-004", ...]
   }

   SaaS → CLI: { sorted_ambiguities: [...] }

6. CLI → User (interactive, prioritized):

   [CRITICAL - must answer]
   Q: Transaction retry strategy?
   > Exponential backoff, max 3 retries

   Q: Idempotency key format?
   > UUID v4, client-generated

   [IMPORTANT - should answer]
   Q: Audit log retention period?
   > 7 years (regulatory)

   [MINOR - can skip]
   Q: ... (use defaults)

7. CLI → Claude Code → Claude:
   "Generate Acceptance Criteria and Business Rules..."
   + [Domain Model] + [Decisions] + [RAG Context]

   Claude →: { acceptance_criteria: [...], business_rules: [...] }

8. CLI → SaaS: POST /api/output/validate
   Body: { l1_output: {...}, domain: "fintech" }

   SaaS (Gemini - Validation):
   "Validate this L1 output for completeness..."

   Gemini →: {
     "valid": true,
     "warnings": [
       "Consider adding BR for transaction timeout",
       "AC-TXN-003 missing error case for insufficient funds"
     ]
   }

   SaaS → CLI: { validation: {...} }

9. CLI shows warnings to user, optionally re-runs derivation

10. OUTPUT: specs/L1-requirements.md (validated, complete)
```

### Költségek (M6+)

| Fizető | Tétel | Költség |
|--------|-------|---------|
| User | Claude API (5 hívás) | ~$0.08-0.12/session |
| Loom SaaS | Gemini (4-5 hívás) | ~$0.003/session |
| Loom SaaS | Infra + Vector DB | ~$50-100/hó |

---

## Összehasonlítás

| Aspektus | MVP (M4-M5) | Gemini-vel (M6+) |
|----------|-------------|------------------|
| Domain-aware checklists | Nem | Igen |
| RAG context | Nem | Igen |
| Prioritized questions | Nem | Igen |
| Output validation | Nem | Igen |
| User experience | Alap | Jobb |
| SaaS komplexitás | Alacsony | Közepes |
| SaaS költség/session | $0 | ~$0.003 |

---

## Hol ad értéket a Gemini?

### 1. L0 Classification → Domain-aware Checklists

**MVP (Gemini nélkül):**
```
User L0: "Payment processing application..."
         │
         ▼
    Generic checklist:
    - [ ] Soft delete or hard delete?
    - [ ] Audit logging needed?
    - [ ] ...generic questions...
```

**M6+ (Gemini-vel):**
```
User L0: "Payment processing application..."
         │
         ▼
    ┌─────────────┐
    │   Gemini    │  "Ez fintech domain"
    └─────────────┘
         │
         ▼
    Domain-specific checklist:
    - [ ] PCI-DSS compliance level?
    - [ ] Idempotency key strategy?
    - [ ] Transaction retry policy?
    - [ ] Chargeback handling?
    - [ ] ...fintech-specific questions...
```

**Hozzáadott érték:** User relevánsabb kérdéseket kap, kevesebb "nem alkalmazható" válasz.

---

### 2. RAG Query Enhancement → Jobb Knowledge Context

**MVP (Gemini nélkül):**
```
Nincs RAG, nincs external knowledge.
Claude csak a saját training data-ból dolgozik.
```

**M6+ (Gemini-vel):**
```
Domain: fintech
         │
         ▼
    ┌─────────────┐
    │   Gemini    │  Query expansion
    └─────────────┘
         │
         ▼
    Search queries:
    - "payment idempotency patterns"
    - "transaction state machine"
    - "PCI DSS requirements for APIs"
         │
         ▼
    ┌─────────────┐
    │  Vector DB  │
    └─────────────┘
         │
         ▼
    Relevant knowledge chunks → Claude context
```

**Hozzáadott érték:** Claude jobb AC/BR-eket generál, mert ismeri az industry best practice-eket.

---

### 3. Retrieval Reranking → Kevesebb, jobb context

**MVP (Gemini nélkül):**
```
Nincs RAG.
```

**M6+ (Gemini-vel):**
```
Vector DB returns 20 chunks (by similarity)
         │
         ▼
    ┌─────────────┐
    │   Gemini    │  "Which 5 are most relevant
    └─────────────┘   for THIS specific L0?"
         │
         ▼
    Top 5 chunks → Claude (less tokens, better focus)
```

**Hozzáadott érték:**
- Kevesebb token = olcsóbb Claude hívás
- Jobb relevancia = jobb output minőség

---

### 4. Ambiguity Prioritization → Jobb UX

**MVP (Gemini nélkül):**
```
Claude detects 47 ambiguities
         │
         ▼
    User gets ALL 47 questions
    (random order, overwhelming)
```

**M6+ (Gemini-vel):**
```
Claude detects 47 ambiguities
         │
         ▼
    ┌─────────────┐
    │   Gemini    │  "Prioritize for fintech"
    └─────────────┘
         │
         ▼
    CRITICAL (must answer): 5 questions
    IMPORTANT (should answer): 12 questions
    MINOR (can skip, use defaults): 30 questions
         │
         ▼
    User answers 5-17 questions instead of 47
```

**Hozzáadott érték:**
- User nem overwhelmed
- Kritikus döntések először
- Minor kérdések → intelligent defaults

---

### 5. Output Validation → Quality Gate

**MVP (Gemini nélkül):**
```
Claude generates L1
         │
         ▼
    Output saved directly
    (no quality check)
```

**M6+ (Gemini-vel):**
```
Claude generates L1
         │
         ▼
    ┌─────────────┐
    │   Gemini    │  "Is this complete? Consistent?"
    └─────────────┘
         │
         ▼
    Validation result:
    ✓ Valid JSON
    ✓ All entities covered
    ⚠ Warning: AC-TXN-003 missing error case
    ⚠ Warning: No BR for transaction timeout
         │
         ▼
    User sees warnings, can re-run or accept
```

**Hozzáadott érték:**
- Kevesebb hibás output
- User bizalom növelése
- Olcsóbb, mint Claude-dal újragenerálni

---

## Értékösszesítő táblázat

| Gemini Use Case | User Benefit | Business Benefit |
|-----------------|--------------|------------------|
| L0 Classification | Releváns kérdések | Magasabb completion rate |
| RAG Enhancement | Jobb AC/BR minőség | Differenciáló feature |
| Reranking | Gyorsabb válasz | Alacsonyabb Claude token cost |
| Prioritization | Kevesebb kérdés | Jobb UX → retention |
| Validation | Megbízható output | Kevesebb support ticket |

---

## ROI elemzés

**Gemini költség:** ~$0.003/session

**Megtakarítás/érték:**
- User kevesebb időt tölt (prioritizált kérdések) → jobb NPS
- Jobb output → kevesebb újrafuttatás → user elégedettség
- Domain-aware → magasabb perceived value → jobb conversion

**Következtetés:** A ~$0.003/session befektetés jelentős UX és minőségjavulást hoz.

---

## Lépések összefoglaló

### MVP (M4-M5)

| # | Lépés | Hol fut | LLM |
|---|-------|---------|-----|
| 1 | Prompt fetch | SaaS | - |
| 2 | Domain Discovery | Client | Claude |
| 3 | Ambiguity Detection | Client | Claude |
| 4 | Interview | Client | Claude + User |
| 5 | Derivation | Client | Claude |

### M6+ (Gemini-vel)

| # | Lépés | Hol fut | LLM |
|---|-------|---------|-----|
| 1 | L0 Classification | SaaS | Gemini |
| 2 | RAG Query Enhancement | SaaS | Gemini |
| 3 | Retrieval + Reranking | SaaS | Gemini |
| 4 | Prompt + Context fetch | SaaS | - |
| 5 | Domain Discovery | Client | Claude |
| 6 | Ambiguity Detection | Client | Claude |
| 7 | Ambiguity Prioritization | SaaS | Gemini |
| 8 | Interview (prioritized) | Client | Claude + User |
| 9 | Derivation | Client | Claude |
| 10 | Output Validation | SaaS | Gemini |

---

*Létrehozva: 2025-12-27*

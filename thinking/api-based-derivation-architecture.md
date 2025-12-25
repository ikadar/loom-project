# Loom L0→L1 API-alapú Architektúra

> ⚠️ **SUPERSEDED** - Ez a dokumentum részben felülírva. Lásd: [platform-strategy.md](./platform-strategy.md)
>
> Az API-alapú architektúra továbbra is releváns **Enterprise on-premise** deployment esetén,
> de a fő irány a **Claude Code Plugin** megközelítés lett (adoption > IP védelem).

> Dokumentum a Claude Code command-alapú megközelítésről való átállásról API-alapú architektúrára.

**Létrehozva:** 2025-12-23
**Frissítve:** 2025-12-23
**Státusz:** RÉSZBEN FELÜLÍRVA (lásd platform-strategy.md)
**Döntés:** ~~Local CLI + Remote API~~ → Claude Code Plugin (fő irány) | Remote API (enterprise)

---

## Motiváció: Miért API a Claude Code Command helyett?

### Claude Code Commands hátrányai

| Aspektus | Claude Code Command | Claude API |
|----------|---------------------|------------|
| **Interaktivitás** | Beépített | Saját UI kell |
| **File műveletek** | Beépített (Read/Write) | Kódban kell |
| **MCP (RAG)** | Natív | Külön integrálni kell |
| **Diff preview** | Automatikus | Saját implementáció |
| **Approval workflow** | Beépített | Saját implementáció |

### Claude API előnyei

| Aspektus | Claude API |
|----------|------------|
| **Kontroll** | Teljes - te irányítod a flow-t |
| **Tesztelhetőség** | Unit tesztek minden lépésre |
| **Model választás** | Haiku az egyszerű, Opus a komplex taskokra |
| **Batch processing** | Ember nélkül futhat (CI/CD) |
| **Párhuzamosítás** | Több deriválás egyszerre |
| **Költség** | Optimalizálható (olcsóbb modellek ahol lehet) |
| **Integráció** | Bármilyen rendszerbe beépíthető |
| **Determinisztikus orchestráció** | Kód vezérli, nem LLM |

### Mikor melyik?

**Claude Code Command jó ha:**
- Developer workflow (interaktív)
- Egyedi projektek
- Rapid prototyping

**Claude API jó ha:**
- CI/CD integráció (auto-derive on L0 change)
- Web UI business usereknek
- Batch processing
- Production rendszer
- Költség optimalizálás

### Konklúzió

A Claude Code command **jó volt PoC-nak** - gyorsan validáltuk a koncepciót.

**Production-ra** Claude API jobb:
- `decisions.md` miatt már most is kevés az interakció
- Batch futtatás lehetséges
- Integrálható más rendszerekbe
- Tesztelhető, monitorozható

A RAG (MCP) nem nagy veszteség - ugyanaz a Python kód, csak közvetlenül hívod, nem MCP-n keresztül.

---

## Deployment Model: Local CLI + Remote API

### A Probléma

Két ellentétes követelmény:
1. **File access** - A CLI-nek hozzá kell férnie a lokális fájlokhoz (code, L0, ADRs)
2. **IP védelem** - A promptok (secret sauce) nem lehetnek a user gépén

### Megoldás: Hybrid Architektúra

```
┌─────────────────────────────────────────────────────┐
│                  User's Machine                      │
│  ┌─────────────────────────────────────────────────┐│
│  │         Loom CLI (open source)                  ││
│  │                                                 ││
│  │  ✅ Reads local files (code, L0, ADRs)          ││
│  │  ✅ Builds context locally                      ││
│  │  ✅ RAG retrieval (local Chroma)                ││
│  │  ✅ Writes output locally                       ││
│  │  ❌ NEM tartalmazza a promptokat                ││
│  └──────────────────┬──────────────────────────────┘│
└─────────────────────┼───────────────────────────────┘
                      │ HTTPS
                      │ {context, task, user_api_key}
                      ▼
         ┌────────────────────────────────────────────┐
         │         AI-DOP API (protected)             │
         │                                            │
         │  🔒 Secret prompts (IP protected)          │
         │  🔒 Orchestration logic                    │
         │                                            │
         │  prompt + context → Claude API             │
         │                     (user's key = BYOK)    │
         └────────────────────────────────────────────┘
                      │
                      ▼ {result}
              Back to local CLI → write files
```

### Miért ez a legjobb?

| Szempont | Eredmény |
|----------|----------|
| **File access** | ✅ Lokális CLI olvas mindent |
| **Codebase privacy** | ✅ Csak context megy, nem teljes repo |
| **IP védelem** | ✅ Promptok a szerveren maradnak |
| **BYOK** | ✅ User API key-jével hívunk Claude-ot |
| **Cost (user)** | Claude API költség (BYOK) |
| **Cost (AI-DOP)** | Csak szerver infra (nincs LLM költség) |

### Elvetett Alternatívák

| Model | Probléma |
|-------|----------|
| **Fully Local** | ❌ IP nem védhető (promptok lokálisan) |
| **Full SaaS** | ❌ Nincs file access (upload kéne) |
| **Git Integration** | ❌ Private repo hozzáférés, security concerns |
| **Agent SDK** | ❌ Lokálisan fut, promptok láthatók (nincs IP védelem) |

### API Key: BYOK vs SaaS

**Nulla architektúrális különbség** - csak az API key forrása más:

```python
def get_api_key(user, payment_model):
    if payment_model == "byok":
        return user.api_key          # User fizeti a Claude API-t
    else:  # saas
        track_usage(user)
        return os.environ["AIDOP_KEY"]  # AI-DOP fizeti, user subscriptiont
```

| Modell | Ki fizeti Claude-ot | Architektúra változás |
|--------|---------------------|----------------------|
| **BYOK** | User | - |
| **SaaS** | AI-DOP | Nincs (csak billing logic) |

**Következmény:** BYOK-kal indulunk, SaaS-ra váltás később = csak config változás.

---

## Architektúra Áttekintés

```
┌─────────────────────────────────────────────────────────────────────┐
│                         ORCHESTRATOR (Python)                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐      │
│  │  Phase 1 │───►│  Phase 2 │───►│  Phase 3 │───►│  Phase 4 │      │
│  │ Discovery│    │ Analysis │    │ Interview│    │ Derivation│      │
│  └────┬─────┘    └────┬─────┘    └────┬─────┘    └────┬─────┘      │
│       │               │               │               │              │
│       ▼               ▼               ▼               ▼              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     LLM CLIENT                               │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐                      │    │
│  │  │  Haiku  │  │ Sonnet  │  │  Opus   │                      │    │
│  │  │(extract)│  │(derive) │  │(complex)│                      │    │
│  │  └─────────┘  └─────────┘  └─────────┘                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│       │               │               │               │              │
│       ▼               ▼               ▼               ▼              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     RAG ENGINE (direct)                      │    │
│  │  retrieve() | get_decisions() | index()                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│       │               │               │               │              │
│       ▼               ▼               ▼               ▼              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     STATE STORE                              │    │
│  │  decisions.md | session_state.json                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
         │                                       ▲
         ▼                                       │
┌─────────────────┐                    ┌─────────────────┐
│   INPUT (L0)    │                    │   OUTPUT (L1)   │
│ - user-stories  │                    │ - acceptance-   │
│ - project-brief │                    │   criteria.md   │
│ - decisions.md  │                    │ - business-     │
└─────────────────┘                    │   rules.md      │
                                       │ - decisions.md  │
                                       └─────────────────┘
```

---

## Komponensek

### 1. Orchestrator

```python
class LoomDerivation:
    def __init__(self, config: DerivationConfig):
        self.llm = LLMClient(config.api_key)
        self.rag = RAGEngine(config.guidelines_dir)
        self.state = StateStore(config.decisions_path)

    def derive(self, input_dir: str, output_dir: str) -> DerivationResult:
        # Phase 1: Domain Discovery
        domain = self.discover_domain(input_dir)

        # Phase 2: Completeness Analysis
        ambiguities = self.analyze_completeness(domain)

        # Phase 3: Filter & Interview
        unresolved = self.filter_resolved(ambiguities)
        if unresolved:
            new_decisions = self.interview(unresolved)
            self.state.save(new_decisions)

        # Phase 4: Derivation
        all_decisions = self.state.load_all()
        output = self.derive_l1(domain, all_decisions)

        # Phase 5: Write & Index
        self.write_output(output, output_dir)
        self.rag.index(output_dir)

        return output
```

### 2. LLM Client (Model Selection)

```python
class LLMClient:
    MODELS = {
        "extract": "claude-3-5-haiku-20241022",    # Gyors, olcsó
        "analyze": "claude-3-5-haiku-20241022",    # Checklist alkalmazás
        "derive": "claude-sonnet-4-20250514",       # Fő deriválás
        "complex": "claude-opus-4-20250514",        # Komplex döntések
    }

    def call(self, task: str, prompt: str,
             structured: bool = True) -> dict | str:
        model = self.MODELS[task]

        if structured:
            # JSON mode - predictable output
            response = self.client.messages.create(
                model=model,
                messages=[{"role": "user", "content": prompt}],
                response_format={"type": "json_object"}
            )
        ...
```

### 3. Phase-ok részletesen

```python
# Phase 1: Domain Discovery (Haiku - olcsó)
def discover_domain(self, input_dir: str) -> Domain:
    content = self.read_l0_files(input_dir)

    prompt = f"""
    Extract domain elements from this L0 input:
    {content}

    Return JSON:
    {{
      "entities": [...],
      "operations": [...],
      "relationships": [...],
      "ui_mentions": [...]
    }}
    """

    return self.llm.call("extract", prompt, structured=True)


# Phase 2: Completeness Analysis (Haiku - checklist alkalmazás)
def analyze_completeness(self, domain: Domain) -> list[Ambiguity]:
    # RAG: retrieve checklists
    entity_checklist = self.rag.retrieve("entity completeness checklist")
    operation_checklist = self.rag.retrieve("operation completeness checklist")

    # Parallel analysis
    entity_ambiguities = self.analyze_entities(domain.entities, entity_checklist)
    operation_ambiguities = self.analyze_operations(domain.operations, operation_checklist)

    return entity_ambiguities + operation_ambiguities


# Phase 3: Interview (Sync vagy Async)
def interview(self, ambiguities: list[Ambiguity]) -> list[Decision]:
    if self.config.mode == "interactive":
        return self.interactive_interview(ambiguities)
    elif self.config.mode == "batch":
        return self.batch_interview(ambiguities)  # Defaults only
    elif self.config.mode == "webhook":
        return self.async_interview(ambiguities)  # Queue + callback


# Phase 4: Derivation (Sonnet - fő munka)
def derive_l1(self, domain: Domain, decisions: list[Decision]) -> L1Output:
    # RAG: retrieve templates
    ac_template = self.rag.retrieve("acceptance criteria template")
    br_template = self.rag.retrieve("business rules template")

    prompt = f"""
    Given:
    - Domain: {domain}
    - Decisions: {decisions}
    - AC Template: {ac_template}
    - BR Template: {br_template}

    Generate acceptance criteria and business rules.
    """

    return self.llm.call("derive", prompt, structured=True)
```

---

## Interview Módok

### A. Interactive (CLI)

```python
def interactive_interview(self, ambiguities: list[Ambiguity]) -> list[Decision]:
    decisions = []
    for batch in chunk(ambiguities, size=5):
        print(format_questions(batch))
        answers = input("Answers (comma-separated): ")
        decisions.extend(parse_answers(batch, answers))
    return decisions
```

### B. Batch (No Human)

```python
def batch_interview(self, ambiguities: list[Ambiguity]) -> list[Decision]:
    # Only use defaults, skip critical without defaults
    decisions = []
    skipped = []
    for amb in ambiguities:
        if amb.suggested_default:
            decisions.append(Decision(amb.id, amb.suggested_default, "default"))
        else:
            skipped.append(amb)

    if skipped:
        raise UnresolvedAmbiguities(skipped)
    return decisions
```

### C. Async (Webhook)

```python
def async_interview(self, ambiguities: list[Ambiguity]) -> str:
    # Create session, return session_id
    session_id = self.state.create_session(ambiguities)

    # Webhook will be called when user answers
    # POST /api/sessions/{session_id}/answers

    return session_id  # Caller polls or waits for webhook
```

---

## Költség Optimalizálás

| Phase | Model | Input tokens | Cost/1K |
|-------|-------|--------------|---------|
| Discovery | Haiku | ~2K | $0.00025 |
| Analysis | Haiku | ~5K | $0.00025 |
| Derivation | Sonnet | ~10K | $0.003 |
| **Total** | | ~17K | **~$0.035** |

vs Claude Code (minden Sonnet): ~$0.10+

**3x olcsóbb** model selection-nel.

---

## File Structure

```
loom-tooling/
├── loom/
│   ├── __init__.py
│   ├── orchestrator.py      # Main flow
│   ├── llm_client.py        # Claude API wrapper
│   ├── rag_engine.py        # Existing RAG (direct)
│   ├── state_store.py       # decisions.md handling
│   ├── interview/
│   │   ├── interactive.py   # CLI mode
│   │   ├── batch.py         # Auto mode
│   │   └── async_handler.py # Webhook mode
│   └── phases/
│       ├── discovery.py
│       ├── analysis.py
│       └── derivation.py
├── cli.py                   # CLI interface
└── api.py                   # REST API (optional)
```

---

## Használat

```bash
# Interactive (CLI)
loom derive --input ./l0 --output ./l1 --mode interactive

# Batch (CI/CD)
loom derive --input ./l0 --output ./l1 --mode batch

# API
curl -X POST /api/derive \
  -d '{"input_dir": "./l0", "output_dir": "./l1", "mode": "batch"}'
```

---

## Következő Lépések

1. **PoC implementáció** - Minimal viable orchestrator
2. **Model benchmarking** - Haiku vs Sonnet quality comparison
3. **Cost tracking** - Per-derivation cost measurement
4. **CI/CD integráció** - GitHub Action for auto-derive

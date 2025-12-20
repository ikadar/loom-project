# RAG PoC Roadmap

> Mini roadmap a v1.5 (Guidelines-alapú RAG) implementációjához.

## Státusz: ✅ BEFEJEZETT (2024-12-20)

## Cél

Demonstrálni, hogy a meglévő `9300-guidelines/` tartalom RAG-ként használva javítja a dokumentum deriválás minőségét.

---

## Fázisok

### Fázis 1: Setup (~10 perc)

| Lépés | Leírás | Output |
|-------|--------|--------|
| 1.1 | Python projekt létrehozása | `tmp/rag-poc/` mappa |
| 1.2 | Dependencies telepítése | `langchain`, `chromadb`, `anthropic` |
| 1.3 | Projekt struktúra | Alap fájlok |

```
tmp/rag-poc/
├── requirements.txt
├── rag_engine.py          # RAG logika
├── test_derivation.py     # Teszt script
└── chroma_db/             # Vector DB (generált)
```

### Fázis 2: RAG Engine (~15 perc)

| Lépés | Leírás | Output |
|-------|--------|--------|
| 2.1 | Guidelines betöltése | DirectoryLoader |
| 2.2 | Chunking | RecursiveCharacterTextSplitter |
| 2.3 | Embedding + Vector DB | Chroma lokális DB |
| 2.4 | Query interface | `retrieve(query) → chunks` |

**Kulcs kód:**
```python
class LoomRAG:
    def __init__(self, guidelines_dir: str):
        # Load guidelines
        # Chunk documents
        # Create vector store

    def retrieve(self, query: str, k: int = 5) -> list[str]:
        # Return top-k relevant chunks

    def derive_with_context(self, input_doc: str, task: str) -> str:
        # Retrieve + Generate with Claude
```

### Fázis 3: Deriválás Teszt (~15 perc)

| Lépés | Leírás | Output |
|-------|--------|--------|
| 3.1 | Input előkészítés | Domain Vocabulary (Quote) |
| 3.2 | RAG nélküli deriválás | Baseline output |
| 3.3 | RAG-gal deriválás | Enhanced output |
| 3.4 | Összehasonlítás | Minőségi értékelés |

**Teszt szcenárió:**
```
Input:  Quote domain vocabulary (L1)
Query:  "Derive domain model with aggregates following guidelines"
RAG:    Retrieves aggregate-design, service-boundaries guidelines
Output: Domain Model a guidelines szerint
```

### Fázis 4: Értékelés (~5 perc) ✅

| Szempont | RAG nélkül | RAG-gal | Javulás |
|----------|------------|---------|---------|
| Szekciók száma | 3 | 7 | ✅ +4 szekció |
| Aggregate boundary | ❌ Nincs | ✅ Explicit diagram | ✅ |
| Invariants listed | ❌ Nincs | ✅ 4 invariant | ✅ |
| Entity/VO reasoning | ❌ Nincs indoklás | ✅ Rationale oszlop | ✅ |
| QuoteLineItem típus | Entity (rossz) | Value Object (indokolt) | ✅ |
| ID-only reference | ❌ Nem említve | ✅ CustomerRef = ID | ✅ |
| Creation rules | ❌ Nincs | ✅ Explicit | ✅ |
| Domain events | ❌ Nincs | ✅ 6 event | ✅ |
| Guidelines compliance | ❌ Ad-hoc struktúra | ✅ 7-szekciós format | ✅ |

---

## Input/Output

### Input (Test Data)

```markdown
## Domain Vocabulary - Quote

- **Quote**: A price offer for a customer
- **Quote Line Item**: Individual product/service in a quote
- **Quote Validity Period**: Time window when quote is valid
- **Quote Status**: Current state (draft, sent, accepted, expired)
- **Quote Total**: Sum of all line item prices
- **Customer Reference**: Link to the requesting customer
```

### Expected Output (RAG-enhanced)

```markdown
## Domain Model - Quote Aggregate

### Purpose
[Derived from aggregate-design-guidelines: "Purpose" section]

### Aggregate Root
Quote (Entity)

### Invariants
[Derived from aggregate-design-guidelines: "Invariants" section]
1. Quote total must equal sum of line items
2. Status transitions follow defined state machine
3. ...

### Entities and Value Objects
[Derived from aggregate-design-guidelines: "Entities and Value Objects" section]
| Element | Type | Rationale |
|---------|------|-----------|
| Quote | Entity | Has lifecycle, unique identity |
| QuoteLineItem | Value Object | No independent identity, replaced wholesale |
| ...

### Boundaries
[Derived from service-boundaries-guidelines]
...
```

---

## Success Criteria

| Kritérium | Cél | Eredmény |
|-----------|-----|----------|
| RAG működik | Query visszaad releváns guideline részeket | ✅ 17 doc, 457 chunk |
| Deriválás javul | Látható különbség RAG vs no-RAG között | ✅ Jelentős különbség |
| Guidelines compliance | Output követi a guideline struktúrát | ✅ 7-szekciós format |
| Implementációs idő | < 45 perc | ✅ ~40 perc |

---

## Fájlok

| Fájl | Típus | Cél |
|------|-------|-----|
| `tmp/rag-poc/requirements.txt` | CREATE | Python dependencies |
| `tmp/rag-poc/rag_engine.py` | CREATE | RAG logika |
| `tmp/rag-poc/test_derivation.py` | CREATE | Teszt + összehasonlítás |
| `tmp/rag-poc/chroma_db/` | GENERATED | Vector database |

---

## Dependencies (Használt)

```
langchain==0.3.14
langchain-community==0.3.14
langchain-chroma==0.2.2
chromadb==0.5.23
sentence-transformers  # HuggingFace embeddings (ingyenes, lokális)
```

**Megjegyzés:** OpenAI embeddings helyett HuggingFace-t használtunk (ingyenes, lokális).
**Package manager:** `uv` (10-100x gyorsabb mint pip)

---

## Kockázatok

| Kockázat | Valószínűség | Mitigáció |
|----------|--------------|-----------|
| OpenAI API key hiányzik | Közepes | Ellenőrizni indulásnál |
| Anthropic API key hiányzik | Közepes | Ellenőrizni indulásnál |
| Guidelines túl rövidek | Alacsony | Már ellenőriztük, jók |
| Chunk méret rossz | Alacsony | Iterálni ha kell |

---

## Következő Lépések (PoC Sikeres!)

1. ✅ ~~RAG PoC validálás~~ - Kész
2. 🔲 Tooling repository létrehozása (`loom-tooling`)
3. 🔲 RAG engine migrálása tooling repo-ba
4. 🔲 Claude Code + RAG integráció (retrieval script)
5. 🔲 MCP Server fejlesztés (v2)

## Tanulságok

### Ami Működött
- HuggingFace embeddings (ingyenes, lokális)
- `uv` package manager (gyors telepítés)
- Guidelines mint tudásbázis (457 chunk)
- Claude Code + RAG retrieval kombináció

### Fontos Felismerés
A RAG nem "helyes választ" ad, hanem **indokolt döntést** segít:
- Entity vs Value Object: mindkettő lehet helyes
- A RAG kontextus biztosítja, hogy a döntés **tudatos és indokolt**
- Guidelines struktúrát követ, nem ad-hoc

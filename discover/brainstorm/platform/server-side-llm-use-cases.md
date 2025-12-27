# Server-side LLM Use Cases

> **Kérdés:** Érdemes-e a SaaS-ban olcsó/ingyenes LLM-et (pl. Gemini) használni?

## Kontextus

A Loom architektúrában:
- **Client-side (Claude Code + Claude):** A tényleges deriválás, nehéz munka
- **Server-side (SaaS):** Promptok, checklistek, license kezelés

A kérdés: hol adhat értéket egy olcsó server-side LLM?

---

## Free LLM Opciók (2025 December)

| Provider | Model | Free Limit | Paid Price |
|----------|-------|------------|------------|
| **Google Gemini** | Flash-Lite | 1,000 req/day | $0.075/1M input |
| **Google Gemini** | Flash | ~20 req/day | $0.15/1M input |
| **Groq** | Llama 3 | 30 req/min | - |
| **Mistral** | 7B | Limited | - |

**Megjegyzés:** Google 2025 decemberében jelentősen csökkentette a free tier-t.

---

## Nem Éri Meg: Prompt Generálás

Eredeti ötlet: Gemini generálja a deriváláshoz használt promptokat.

**Probléma:** A jelenlegi promptok generikusak, statikusak is működnek. Az LLM hozzáadott értéke minimális.

**Alternatíva:** Template + domain lookup table (nincs LLM szükség).

---

## Érdemes: Értékes Use Case-ek

### 1. RAG Query Enhancement (P1)

```
User kérdés: "hogyan kell hibát kezelni?"
                    │
                    ▼
              ┌─────────────┐
              │   Gemini    │
              └─────────────┘
                    │
                    ▼
Rewritten: "error handling patterns" + "exception management"
           + "fault tolerance" + "retry strategies"
                    │
                    ▼
            Vector DB search (több releváns találat)
```

**Érték:** A RAG retrieval minősége drámaian javul query rewriting-gal.

---

### 2. Retrieval Reranking (P1)

```
Vector DB visszaad 20 chunk-ot
            │
            ▼
      ┌─────────────┐
      │   Gemini    │  "Melyik 5 a legrelevánsabb?"
      └─────────────┘
            │
            ▼
    Top 5 legjobb chunk → Claude-nak küldve
```

**Érték:** Kevesebb, de jobb context = jobb Claude válasz + kevesebb token költség.

---

### 3. Output Validation (P1)

```
Claude output (JSON)
        │
        ▼
  ┌─────────────┐
  │   Gemini    │  "Valid JSON? Teljes? Konzisztens?"
  └─────────────┘
        │
        ▼
{
  "valid": true,
  "issues": ["AC-003 missing error cases"],
  "suggestions": ["Add timeout handling"]
}
```

**Érték:** Quality gate mielőtt user-nek visszaadjuk.

---

### 4. Ambiguity Prioritization (P2)

```
50 detected ambiguities
        │
        ▼
  ┌─────────────┐
  │   Gemini    │  "Prioritizáld: critical → minor"
  └─────────────┘
        │
        ▼
Top 10 critical kérdés először
```

**Érték:** Jobb user experience, nem kap 50 kérdést egyszerre.

---

### 5. L0 Input Classification (P2)

```
User L0 dokumentum
        │
        ▼
  ┌─────────────┐
  │   Gemini    │
  └─────────────┘
        │
        ▼
{
  "domain": "fintech",
  "complexity": "medium",
  "estimated_entities": 8,
  "suggested_checklists": ["pci", "audit", "transaction"]
}
```

**Érték:** Intelligens routing - megfelelő checklistek automatikus kiválasztása.

---

### 6. Decision Pattern Analysis (P3)

```
User decisions history
        │
        ▼
  ┌─────────────┐
  │   Gemini    │  "Milyen mintázatok vannak?"
  └─────────────┘
        │
        ▼
Insights: "80% of fintech users choose soft-delete"
```

**Érték:** A rendszer tanul, jobb default-okat javasol.

---

## Összefoglaló

| Use Case | Érték | Komplexitás | Gemini elég? | Prioritás |
|----------|-------|-------------|--------------|-----------|
| RAG Query Enhancement | Magas | Alacsony | Igen | **P1** |
| Retrieval Reranking | Magas | Közepes | Igen | **P1** |
| Output Validation | Magas | Közepes | Igen | **P1** |
| Ambiguity Prioritization | Közepes | Alacsony | Igen | **P2** |
| L0 Classification | Közepes | Alacsony | Igen | **P2** |
| Decision Pattern Analysis | Alacsony | Magas | Igen | **P3** |
| Prompt Generation | Alacsony | Közepes | Igen | Skip |

---

## Költségbecslés

Egy derivation session (~5 LLM hívás, Gemini Flash-Lite):
- Input: ~5,000 tokens → $0.000375
- Output: ~10,000 tokens → $0.003
- **Összesen: ~$0.0034/session** (~1 Ft)

| MAU | Sessions/month | Monthly Cost |
|-----|----------------|--------------|
| 100 | 500 | $1.70 |
| 1,000 | 5,000 | $17 |
| 10,000 | 50,000 | $170 |

---

## Ajánlás (Roadmap szerint)

- **M4-M5 (Platform Impl. + MVP Launch):** Statikus működés, nincs server-side LLM
- **M6 (Beta & Iteration):** RAG Query Enhancement + Reranking (RAG már M4-ben elkészül)
- **M7+ (Continuous Ops):** Output Validation, Ambiguity Prioritization

---

## Források

- [Gemini API Pricing](https://ai.google.dev/gemini-api/docs/pricing)
- [Gemini API Rate Limits](https://ai.google.dev/gemini-api/docs/rate-limits)

---

*Létrehozva: 2025-12-27*

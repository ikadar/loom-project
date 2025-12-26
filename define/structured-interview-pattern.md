# Structured Interview Pattern for AI Decision Elicitation

> **Status:** Thinking Document
> **Date:** 2025-12-21
> **Context:** Loom koncepció - implicit döntések megelőzése

---

## Probléma

Az AI modellek hajlamosak **implicit döntéseket** hozni, amikor a bemenet nem tartalmaz elegendő információt. Ez különösen problémás domain modellezésnél:

| Helyzet | AI viselkedés | Probléma |
|---------|---------------|----------|
| RAG nélkül | QuoteLineItem → Entity | Implicit, indokolatlan |
| RAG-gal | QuoteLineItem → Value Object (indokolt) | Jobb, de még mindig implicit |
| **Ideális** | "Kell-e önálló identitás?" → User válaszol → Döntés | Explicit, felhasználó-vezérelt |

**A Loom egyik legfontosabb célja:** Megakadályozni, hogy az AI olyan döntéseket hozzon, amelyekhez nincs elegendő információja.

---

## Megoldás: Structured Interview Pattern

### Definíció

A Structured Interview Pattern egy iteratív elicitációs minta, ahol az AI:

1. **Azonosítja** a döntési pontokat a feladatban
2. **Ellenőrzi**, hogy van-e elegendő információ minden döntéshez
3. **Kérdez**, ha nincs elegendő információ
4. **Iterál**, amíg minden döntési pont nem tisztázott
5. **Derivál**, csak amikor minden szükséges információ rendelkezésre áll

### Folyamat

```
┌─────────────────────────────────────────────────────────────────┐
│                   STRUCTURED INTERVIEW LOOP                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INPUT (L0 docs, user request)                                  │
│         │                                                        │
│         ▼                                                        │
│  ┌──────────────────────────────────────┐                       │
│  │           ANALYZE STATE              │                       │
│  │  • Identify decision points          │                       │
│  │  • Check available information       │                       │
│  │  • Determine gaps                    │                       │
│  └──────────────────┬───────────────────┘                       │
│                     │                                            │
│                     ▼                                            │
│           ┌─────────────────┐                                   │
│           │  Gaps remain?   │                                   │
│           └────────┬────────┘                                   │
│                    │                                             │
│         ┌──────────┴──────────┐                                 │
│         │                     │                                  │
│        NO                    YES                                 │
│         │                     │                                  │
│         ▼                     ▼                                  │
│  ┌─────────────┐      ┌─────────────────┐                       │
│  │   DERIVE    │      │   ASK QUESTION  │                       │
│  │   OUTPUT    │      │   (targeted)    │                       │
│  └─────────────┘      └────────┬────────┘                       │
│                                │                                 │
│                                ▼                                 │
│                        ┌─────────────┐                          │
│                        │   RECEIVE   │                          │
│                        │   ANSWER    │                          │
│                        └──────┬──────┘                          │
│                               │                                  │
│                               ▼                                  │
│                    ┌──────────────────┐                         │
│                    │  PROCESS ANSWER  │                         │
│                    │  • Sufficient?   │                         │
│                    │  • New Qs raised?│                         │
│                    │  • Clarify?      │                         │
│                    └────────┬─────────┘                         │
│                             │                                    │
│                             └───────────► LOOP BACK TO ANALYZE  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Mennyire Elterjedt Ez a Pattern?

### Hasonló Megközelítések

| Terület | Pattern neve | Leírás |
|---------|--------------|--------|
| **Requirements Engineering** | Requirements Elicitation | Strukturált interjúk stakeholderekkel |
| **Expert Systems** | Knowledge Acquisition | Iteratív tudáskinyerés szakértőktől |
| **Conversational AI** | Slot Filling | Hiányzó információk bekérése dialógusban |
| **Medical AI** | Differential Diagnosis | Kérdések→szűkítés→diagnózis |
| **Legal Tech** | Legal Intake | Strukturált kérdéssor ügyfélnek |
| **LLM Agents** | ReAct / Chain-of-Thought | Gondolkodás→cselekvés→megfigyelés ciklus |

### Létező Implementációk

**1. Slot Filling (Task-Oriented Dialogue)**
```
User: "Book a flight to London"
AI: "What date would you like to travel?"
User: "Next Friday"
AI: "What time do you prefer?"
...
```
- Széles körben használt (Siri, Alexa, Google Assistant)
- Fix séma, előre definiált slot-ok
- **Limitáció:** Nem dinamikus, nem adaptálódik új döntési pontokhoz

**2. Socratic Method in AI Tutoring**
```
AI: "What do you think happens when X?"
Student: "Y happens"
AI: "And if Y happens, what follows?"
...
```
- Oktatási AI-ban használt
- Kérdésekkel vezeti a gondolkodást
- **Relevancia:** Hasonló elvek, de nem döntés-orientált

**3. Legal Intake Systems**
```
AI: "What type of legal issue are you facing?"
User: "Employment dispute"
AI: "Were you terminated or did you resign?"
...
```
- Strukturált döntési fák
- **Limitáció:** Előre definiált kérdéssor, nem dinamikus

**4. Medical History Taking AI**
```
AI: "What symptoms are you experiencing?"
Patient: "Headache and fever"
AI: "How long have you had these symptoms?"
AI: "Have you traveled recently?"
...
```
- Adaptív kérdezés tünetek alapján
- **Relevancia:** Közel áll a célunkhoz, de domain-specifikus

---

## Megvalósíthatóság LLM-ekkel

### Előnyök

| Faktor | Értékelés | Megjegyzés |
|--------|-----------|------------|
| LLM képesség | ✅ Kiváló | Az LLM-ek jól kérdeznek és értelmeznek |
| Kontextus kezelés | ✅ Jó | Claude 200K token kontextus elegendő |
| Természetes nyelv | ✅ Kiváló | Nem kell formális nyelv a válaszokhoz |
| Dinamikus adaptáció | ✅ Jó | Új kérdéseket tud generálni válaszok alapján |
| Tool integration | ✅ Van | AskUserQuestion tool létezik Claude Code-ban |

### Kihívások

| Kihívás | Súlyosság | Megoldási irány |
|---------|-----------|-----------------|
| Döntési pontok azonosítása | Közepes | RAG + decision point templates |
| Mikor elég a válasz? | Magas | Explicit kritériumok a guidelines-ban |
| Túl sok kérdés | Közepes | Prioritizálás, batch kérdések |
| Inkonzisztens válaszok | Alacsony | Validáció, visszakérdezés |
| Felhasználói türelem | Közepes | Progresszív disclosure, context |

### Technikai Megvalósítás Claude Code-ban

**1. Decision Point Identification (RAG-enhanced)**
```markdown
## Guidelines: Decision Points for Domain Modeling

### Entity vs Value Object
REQUIRED INFORMATION:
- Does it have independent identity? (Y/N/Unknown)
- Is it referenced from outside the aggregate? (Y/N/Unknown)
- Does it have its own lifecycle? (Y/N/Unknown)

IF any answer is "Unknown" → ASK USER
```

**2. Skill Prompt Structure**
```markdown
## Derivation Process

PHASE 1: DECISION ELICITATION
1. Parse input for decision points
2. For each decision point:
   - Check if input contains answer
   - If not, add to "questions" list
3. If questions list non-empty:
   - Use AskUserQuestion tool
   - Wait for answers
   - LOOP back to step 1 with new context

PHASE 2: DERIVATION (only when no questions remain)
- Derive with all explicit decisions
- Include decision rationale in output
```

**3. State Management**
```yaml
# Derivation state
decisions:
  quote_line_item_type:
    status: "pending"  # pending | answered | derived
    question: "Does QuoteLineItem need independent identity?"
    answer: null
    rationale: null

  aggregate_boundary:
    status: "answered"
    question: "Should Customer be in Quote aggregate?"
    answer: "No, separate aggregate"
    rationale: "Customer has independent lifecycle"
```

---

## Implementációs Terv

### Fázis 1: Decision Point Catalog

Minden deriválási típushoz (Domain Model, API Contract, stb.) készíteni kell:
- Lehetséges döntési pontok listája
- Minden ponthoz: kérdés template, lehetséges válaszok, default ha explicit
- Kritériumok: mikor elég az információ?

**Példa: Domain Model Decision Points**

| Decision Point | Question | Answers | Default (if explicit in input) |
|----------------|----------|---------|-------------------------------|
| Entity vs VO | "Has independent identity?" | Yes/No | Infer from lifecycle mentions |
| Aggregate boundary | "Separate lifecycle?" | Yes/No | Infer from relationships |
| ID type | "Natural or surrogate key?" | Natural/UUID/Auto-increment | UUID if not specified |

### Fázis 2: Skill Modification

Módosítani a loom-derive skill-t:
1. Analízis fázis hozzáadása
2. AskUserQuestion integráció
3. Iteratív loop implementálása
4. State tracking (mi van megválaszolva, mi nincs)

### Fázis 3: Testing & Refinement

- Tesztelés valós domain-ekkel
- Kérdések finomítása (túl sok? túl kevés? érthetőek?)
- Felhasználói élmény optimalizálás

---

## Kockázatok és Mitigáció

| Kockázat | Valószínűség | Hatás | Mitigáció |
|----------|--------------|-------|-----------|
| Túl sok kérdés → user frustration | Magas | Magas | Batch kérdések, priorizálás, smart defaults |
| Nem releváns kérdések | Közepes | Közepes | Jobb decision point identification |
| Felhasználó nem tudja a választ | Közepes | Közepes | Explain + suggest, allow "I don't know" |
| Végtelen loop | Alacsony | Magas | Max iterations, escape hatch |
| Inkonzisztens válaszok | Közepes | Közepes | Validation, conflict detection |

---

## Összefoglalás

### A Pattern Értékelése

| Szempont | Értékelés |
|----------|-----------|
| **Elterjedtség** | Közepes - Léteznek hasonló megközelítések, de nem LLM-specifikus formában |
| **Megvalósíthatóság** | Magas - Az LLM-ek jól alkalmasak erre, Claude Code tool-ok támogatják |
| **Hozzáadott érték** | Kritikus - Ez oldja meg az implicit döntés problémát |
| **Implementációs effort** | Közepes - Skill módosítás + decision catalog szükséges |

### Miért Kritikus a Loom-nak?

1. **Megkülönböztető feature:** Más AI-assisted dev tool-ok nem csinálják ezt
2. **Trust building:** A felhasználó látja, MI alapján dönt az AI
3. **Auditálhatóság:** Minden döntés dokumentált és indokolt
4. **Hibamegelőzés:** Nincs "rossz irányba futás" implicit döntések miatt

### Következő Lépések

1. ✅ Dokumentum elkészítése (ez a dokumentum)
2. 🔲 Decision Point Catalog létrehozása (Domain Model-hez először)
3. 🔲 loom-derive skill módosítása iteratív elicitation-nel
4. 🔲 PoC tesztelés (Quote domain, de más kérdésekkel)
5. 🔲 Refinement felhasználói feedback alapján

---

## Hivatkozások

- Requirements Elicitation techniques (IEEE 830)
- Task-Oriented Dialogue Systems (Slot Filling)
- Socratic Method in AI
- ReAct: Synergizing Reasoning and Acting in Language Models
- Chain-of-Thought Prompting

---

*Ez a dokumentum a Loom koncepció fejlesztésének része. A Structured Interview Pattern implementálása kritikus a Loom megkülönböztető értékének biztosításához.*

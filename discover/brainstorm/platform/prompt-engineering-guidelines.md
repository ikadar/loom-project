---
title: "Loom Prompt Engineering Guidelines"
status: draft
created: 2025-12-26
based-on: https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview
see-also:
  - platform-architecture.md
  - mcp-and-skills-design.md
---

# Loom Prompt Engineering Guidelines

> Anthropic prompt engineering best practices alkalmazása a Loom promptokra.

---

## Összefoglaló

A Loom promptjainak az Anthropic hivatalos prompt engineering ajánlásait kell követniük. Ez:
1. Javítja a deriválás minőségét
2. Konzisztens outputot biztosít
3. Validálja a Loom megközelítését (L0→L3 = prompt chaining)

---

## 1. Prompt Chaining (L0→L3 Pipeline)

**Forrás:** [Chain complex prompts](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/chain-prompts)

### Anthropic Ajánlás

> *"Breaking down complex tasks into smaller, manageable subtasks... Each subtask gets Claude's full attention, reducing errors."*

### Loom Implementáció

A Loom L0→L3 deriválási pipeline **pontosan** ezt a pattern-t követi:

```
L0 (Vision)
    │
    ▼ [Prompt Chain 1: Domain Discovery]
L1 (Requirements)
    │
    ▼ [Prompt Chain 2: Technical Derivation]
L2 (Technical Spec)
    │
    ▼ [Prompt Chain 3: Implementation Details]
L3 (Implementation)
```

**Előnyök:**
- Minden szint megkapja Claude teljes figyelmét
- Hibák korán kiszűrhetők
- Traceability minden lépésnél

**Implementáció:**
- Minden L szint = külön prompt (vagy prompt sorozat)
- Előző szint outputja = következő szint inputja
- XML tag-ekkel strukturált handoff

---

## 2. XML Tags Használata

**Forrás:** [Use XML tags](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/use-xml-tags)

### Anthropic Ajánlás

> *"XML tags help Claude parse your prompts more accurately, leading to higher-quality outputs."*

### Loom Implementáció

Minden Loom promptnak strukturált XML tag-eket kell használnia:

```xml
<loom_context>
  <project_info>{{PROJECT_DESCRIPTION}}</project_info>
  <current_level>L0</current_level>
  <target_level>L1</target_level>
</loom_context>

<input_documents>
  <document type="user-story" id="US-001">
    {{USER_STORY_CONTENT}}
  </document>
</input_documents>

<previous_decisions>
  <decision id="SI-AUTH-001">
    <question>Authentication method?</question>
    <answer>JWT with refresh tokens</answer>
    <rationale>Stateless, scalable</rationale>
  </decision>
</previous_decisions>

<instructions>
  Derive L1 acceptance criteria from the user stories above.
  Use the previous decisions as constraints.
  Output in <output> tags.
</instructions>
```

**Ajánlott Loom Tag-ek:**

| Tag | Használat |
|-----|-----------|
| `<loom_context>` | Projekt és deriválási kontextus |
| `<input_documents>` | Bemeneti dokumentumok |
| `<previous_decisions>` | Korábbi SI döntések |
| `<checklist>` | Ellenőrzőlista (entity, operation) |
| `<examples>` | Példa outputok |
| `<instructions>` | Feladat leírása |
| `<output>` | Elvárt output struktúra |
| `<thinking>` | Claude gondolkodási folyamata |

---

## 3. Examples (Multishot Prompting)

**Forrás:** [Use examples](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/multishot-prompting)

### Anthropic Ajánlás

> *"Include 3-5 diverse, relevant examples to show Claude exactly what you want. More examples = better performance."*

### Loom Implementáció

Minden deriválási prompt tartalmazzon példákat:

```xml
<examples>
  <example type="good">
    <input>
      User Story: As a customer, I want to view my order history.
    </input>
    <output>
      AC-001: Customer can see list of past orders
      AC-002: Each order shows date, total, status
      AC-003: Orders are sorted by date (newest first)
      BR-001: Only show orders from last 12 months by default
    </output>
  </example>

  <example type="good">
    <input>
      User Story: As an admin, I want to manage user accounts.
    </input>
    <output>
      AC-010: Admin can view list of all users
      AC-011: Admin can disable/enable user accounts
      AC-012: Admin can reset user passwords
      BR-010: Admin cannot delete their own account
      BR-011: Password reset sends email notification
    </output>
  </example>

  <example type="bad" reason="Missing business rules">
    <input>
      User Story: As a user, I want to upload files.
    </input>
    <output>
      AC-020: User can upload files
    </output>
    <correction>
      AC-020: User can upload files up to 50MB
      AC-021: Supported formats: PDF, PNG, JPG
      BR-020: Files are scanned for viruses before storage
      BR-021: Upload fails gracefully with clear error message
    </correction>
  </example>
</examples>
```

**Szabályok:**
- Minimum 2-3 példa minden prompt típushoz
- Tartalmazzon "jó" és "rossz" példákat is
- Példák legyenek a target domain-hez relevánsak
- Diverz példák (edge case-ek is)

---

## 4. Chain of Thought (Gondolkodás)

**Forrás:** [Let Claude think](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/chain-of-thought)

### Anthropic Ajánlás

> *"Giving Claude space to think can dramatically improve its performance... encourage Claude to break down problems step-by-step."*

### Loom Implementáció

Komplex deriválási döntéseknél kérjük a gondolkodást:

```xml
<instructions>
  Before deriving the acceptance criteria, analyze the user story in <thinking> tags:

  1. Identify the primary actor and their goal
  2. List implicit assumptions that need clarification
  3. Consider edge cases and error scenarios
  4. Check against the entity checklist

  Then provide your output in <output> tags.
</instructions>
```

**Mikor használjuk:**
- Ambiguity detection (SI kérdések generálása)
- Komplex business rule deriválás
- Cross-cutting concern azonosítás
- Negatív teszt eset generálás (TDAI)

**Mikor NE használjuk:**
- Egyszerű, egyértelmű deriválások
- Ha a latency kritikus

---

## 5. Role Prompting (System Prompt)

**Forrás:** [Give Claude a role](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/system-prompts)

### Anthropic Ajánlás

> *"Role prompting is the most powerful way to use system prompts... The right role can turn Claude from a general assistant into your virtual domain expert!"*

### Loom Implementáció

Minden Loom prompt használjon megfelelő szerepet:

**L0 → L1 deriválás:**
```
You are a senior requirements engineer with 15 years of experience
in enterprise software development. You specialize in:
- Translating business vision into precise requirements
- Identifying ambiguities and implicit assumptions
- Writing testable acceptance criteria
- Anticipating edge cases and error scenarios
```

**L1 → L2 deriválás:**
```
You are a senior software architect specializing in:
- System design and component architecture
- API design and integration patterns
- Security and scalability considerations
- Technical decision documentation
```

**Test deriválás (TDAI):**
```
You are a senior QA engineer and test architect with expertise in:
- Test-driven development
- Negative testing and edge case identification
- Security testing (OWASP awareness)
- Test coverage analysis
```

---

## 6. Prefill Response

**Forrás:** [Prefill Claude's response](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prefill-claudes-response)

### Anthropic Ajánlás

> *"Prefilling allows you to direct Claude's actions, skip preambles, enforce specific formats."*

### Loom Implementáció

Output formátum biztosítása prefill-lel:

```python
messages = [
    {"role": "user", "content": derivation_prompt},
    {"role": "assistant", "content": "```yaml\nderived_requirements:\n"}
]
```

**Használati esetek:**
- YAML/JSON output biztosítása
- Preamble kihagyása
- Konzisztens struktúra kényszerítése

---

## 7. Long Context Kezelés

**Forrás:** [Long context tips](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/long-context-tips)

### Anthropic Ajánlás

> *"Put longform data at the top... Queries at the end can improve response quality by up to 30%."*

### Loom Implementáció

Prompt struktúra nagy dokumentumoknál:

```xml
<!-- 1. DOKUMENTUMOK ELŐL -->
<input_documents>
  <document index="1" type="l0-vision">
    {{VISION_DOCUMENT}}
  </document>
  <document index="2" type="user-stories">
    {{USER_STORIES}}
  </document>
  <document index="3" type="existing-decisions">
    {{PREVIOUS_DECISIONS}}
  </document>
</input_documents>

<!-- 2. KONTEXTUS -->
<loom_context>
  <project>{{PROJECT_NAME}}</project>
  <domain>{{DOMAIN}}</domain>
  <derivation_target>L1 Acceptance Criteria</derivation_target>
</loom_context>

<!-- 3. PÉLDÁK -->
<examples>
  {{EXAMPLES}}
</examples>

<!-- 4. INSTRUKCIÓK A VÉGÉN -->
<instructions>
  Based on the documents above, derive L1 acceptance criteria.

  For each user story:
  1. Quote the relevant parts first
  2. Derive acceptance criteria
  3. Identify business rules
  4. Flag any ambiguities for SI
</instructions>
```

**Grounding in Quotes:**
```xml
<instructions>
  Before deriving, quote the relevant parts of the input documents
  in <quotes> tags. This helps ensure accuracy.
</instructions>
```

---

## 8. Be Clear and Direct

**Forrás:** [Be clear and direct](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/be-clear-and-direct)

### Anthropic Ajánlás

> *"The more precisely you explain what you want, the better Claude's response will be."*

### Loom Implementáció

A Structured Interview (SI) pattern pont ezt valósítja meg:

```xml
<instructions>
  Your task is to derive L1 acceptance criteria.

  Context:
  - This is for a B2B SaaS scheduling application
  - Target users: Print shop managers
  - Compliance: GDPR required

  Output requirements:
  - Each AC must be testable (binary pass/fail)
  - Use format: AC-XXX: <actor> can <action> [when <condition>]
  - Include at least one negative case per user story

  Do NOT:
  - Make assumptions about unspecified behavior
  - Skip edge cases
  - Use vague language ("should", "might", "usually")
</instructions>
```

---

## Implementációs Prioritások

| Prioritás | Technika | Hatás | Effort |
|-----------|----------|-------|--------|
| 🔴 P0 | XML Tags | Magas | Alacsony |
| 🔴 P0 | Examples (2-3 per prompt) | Magas | Közepes |
| 🔴 P0 | Long Context struktúra | Magas | Alacsony |
| 🟡 P1 | Role Prompts | Közepes | Alacsony |
| 🟡 P1 | Chain of Thought (SI-nál) | Közepes | Alacsony |
| 🟢 P2 | Prefill | Alacsony | Alacsony |

---

## Validáció: Loom = Anthropic Best Practices

A Loom architektúra természetesen követi az Anthropic ajánlásokat:

| Anthropic Best Practice | Loom Megfelelő |
|------------------------|----------------|
| Prompt Chaining | ✅ L0→L1→L2→L3 pipeline |
| Be Clear & Direct | ✅ Structured Interview |
| XML Structure | ⚠️ Implementálandó |
| Examples | ⚠️ Implementálandó |
| Chain of Thought | ⚠️ Részben (SI-nál) |
| Role Prompts | ⚠️ Implementálandó |

---

## Kapcsolódó

- [Platform Architecture](./platform-architecture.md)
- [MCP and Skills Design](./mcp-and-skills-design.md)
- [Anthropic Prompt Engineering Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview)

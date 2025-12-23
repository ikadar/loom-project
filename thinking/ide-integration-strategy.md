# AI-DOP IDE Integration Strategy

> Dokumentum a különböző IDE-kkel (Claude Code, Cursor, Windsurf, VS Code) való integrációról.

**Létrehozva:** 2025-12-23

---

## Alapelv

Az AI-DOP Remote API egyetlen backend, többféle frontend:

```
                    ┌─────────────────────────┐
                    │    AI-DOP Remote API    │
                    │    (protected prompts)  │
                    └───────────┬─────────────┘
                                │
            ┌───────────────────┼───────────────────┐
            │                   │                   │
            ▼                   ▼                   ▼
    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
    │  Loom CLI     │   │  VS Code Ext  │   │  MCP Server   │
    │               │   │               │   │               │
    │  - Terminal   │   │  - Cursor     │   │  - Claude Code│
    │  - CI/CD      │   │  - Windsurf   │               │
    │  - Scripts    │   │  - VS Code    │               │
    └───────────────┘   └───────────────┘   └───────────────┘
```

---

## Integrációs Módszerek

### 1. CLI (Univerzális)

**Működik:** Mindenhol (terminal)

```bash
# Bármilyen IDE terminálából
$ loom derive --input ./l0 --output ./l1

# CI/CD pipeline-ban
$ loom derive --mode batch --input ./specs --output ./generated
```

**Előnyök:**
- Nulla IDE-specifikus fejlesztés
- CI/CD kompatibilis
- Scriptelhető

**Hátrányok:**
- Nem "native" IDE élmény
- Nincs UI integráció

---

### 2. MCP Server (Claude Code)

**Működik:** Claude Code

```
┌─────────────────────────────────────────────────────────────────┐
│                      Claude Code                                 │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              loom-mcp (local MCP server)                │    │
│  │                                                         │    │
│  │  Tools:                                                 │    │
│  │  - loom_derive(input_dir, output_dir)                   │    │
│  │  - loom_analyze(input_dir)                              │    │
│  │  - loom_interview_answer(session_id, answers)           │    │
│  │                         │                               │    │
│  └─────────────────────────┼───────────────────────────────┘    │
└─────────────────────────────┼───────────────────────────────────┘
                              │ HTTPS
                              ▼
                    ┌───────────────────┐
                    │  AI-DOP Remote API │
                    └───────────────────┘
```

**Konfiguráció (`.mcp.json`):**
```json
{
  "mcpServers": {
    "loom": {
      "command": "loom-mcp",
      "args": ["--api-url", "https://api.loom.dev"]
    }
  }
}
```

**Használat:**
```
User: Derive L1 from ./specs/l0

Claude: I'll use the loom_derive tool to generate L1 documents...
[Calls loom_derive → MCP → Remote API]

Claude: I found 12 ambiguities. Let me ask you about them...
[Interactive interview through Claude Code conversation]
```

**Előnyök:**
- Native Claude Code experience
- Claude kezeli az interview-t természetes nyelven
- Tool-based, AI dönt mikor használja

**Hátrányok:**
- Csak Claude Code-ban működik

---

### 3. Claude Code Slash Command

**Működik:** Claude Code

**`.claude/commands/loom-derive.md`:**
```yaml
---
name: loom-derive
description: Derive L1 from L0 using AI-DOP
arguments:
  - name: input
    description: Input directory with L0 files
    required: true
  - name: output
    description: Output directory for L1 files
    required: true
---

# Loom Derive

1. Read all files from $ARGUMENTS.input
2. Build context with relevant code and ADRs
3. Call AI-DOP API via Bash:
   ```bash
   loom derive --input $ARGUMENTS.input --output $ARGUMENTS.output
   ```
4. Display results and write files
```

**Használat:**
```
/loom-derive --input ./specs/l0 --output ./specs/l1
```

---

### 4. VS Code Extension

**Működik:** VS Code, Cursor, Windsurf

```
┌─────────────────────────────────────────────────────────────────┐
│              Cursor / Windsurf / VS Code                         │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │           Loom Extension                                  │  │
│  │                                                           │  │
│  │  Commands (Cmd+Shift+P):                                  │  │
│  │  - Loom: Derive L1 from L0                                │  │
│  │  - Loom: Analyze Ambiguities                              │  │
│  │  - Loom: Continue Interview                               │  │
│  │  - Loom: Show Traceability Map                            │  │
│  │                                                           │  │
│  │  UI Components:                                           │  │
│  │  - Sidebar: Ambiguities list, decision history            │  │
│  │  - Webview: Interview wizard                              │  │
│  │  - CodeLens: Traceability annotations                     │  │
│  │  - Diagnostics: Missing links, gaps                       │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
└──────────────────────────────┼───────────────────────────────────┘
                               │ HTTPS
                               ▼
                     AI-DOP Remote API
```

**Features:**

| Feature | Description |
|---------|-------------|
| **Command Palette** | Derive, analyze, interview commands |
| **Sidebar Panel** | Ambiguities, decisions, traceability |
| **Webview** | Rich interview UI with options |
| **CodeLens** | Inline AC/BR links in code |
| **Diagnostics** | Warnings for missing traceability |
| **Status Bar** | Current derivation status |

**Előnyök:**
- Rich, native IDE experience
- Works in Cursor, Windsurf, VS Code
- Visual interview UI
- Inline annotations

**Hátrányok:**
- Jelentős fejlesztési effort
- Külön maintenance 3 IDE-re (bár VS Code API közös)

---

## IDE Compatibility Matrix

| Feature | CLI | MCP | Slash Cmd | VS Code Ext |
|---------|-----|-----|-----------|-------------|
| **Claude Code** | ✅ | ✅ | ✅ | ❌ |
| **Cursor** | ✅ | ⚠️ | ❌ | ✅ |
| **Windsurf** | ✅ | ⚠️ | ❌ | ✅ |
| **VS Code** | ✅ | ❌ | ❌ | ✅ |
| **Terminal** | ✅ | ❌ | ❌ | ❌ |
| **CI/CD** | ✅ | ❌ | ❌ | ❌ |

⚠️ = Partial MCP support, may work

---

## Implementation Phases

### Phase 1: CLI (Baseline)

**Effort:** 2-3 weeks
**Coverage:** 100% (minden környezet)

```bash
pip install loom-dop
loom derive --input ./l0 --output ./l1
```

Deliverables:
- Python CLI package
- Remote API client
- Interactive interview (terminal)
- Batch mode (CI/CD)

### Phase 2: MCP Server (Claude Code)

**Effort:** 1-2 weeks
**Coverage:** Claude Code users

```json
{"mcpServers": {"loom": {"command": "loom-mcp"}}}
```

Deliverables:
- MCP server wrapper around CLI
- Tools: derive, analyze, interview
- Session management

### Phase 3: VS Code Extension (Cursor, Windsurf)

**Effort:** 4-6 weeks
**Coverage:** VS Code ecosystem

Deliverables:
- VS Code extension
- Sidebar UI
- Webview interview wizard
- CodeLens integration
- Marketplace publishing

---

## Interview Handling per Integration

| Integration | Interview Method |
|-------------|------------------|
| **CLI** | Terminal prompts (interactive) or skip (batch) |
| **MCP** | Claude Code conversation (natural language) |
| **VS Code Ext** | Webview wizard with buttons/dropdowns |

### CLI Interview

```
$ loom derive --input ./l0 --output ./l1

Analyzing... Found 12 ambiguities.

[1/12] Entity: Station
  Q: What happens to tasks when station is deleted?
  Options:
    1) Block deletion
    2) Cascade delete tasks
    3) Orphan tasks
  Your choice (1-3): 1

[2/12] ...
```

### MCP Interview (via Claude)

```
Claude: I found 12 ambiguities in your L0 documents. Let me ask you about them.

First, about the Station entity: What should happen to scheduled tasks
when a station is deleted?

User: Block the deletion if there are tasks

Claude: Got it. Next question...
```

### VS Code Extension Interview

```
┌─────────────────────────────────────────────────┐
│  Loom Interview (3/12)                      [X] │
├─────────────────────────────────────────────────┤
│                                                 │
│  Entity: Station                                │
│                                                 │
│  What happens to tasks when station deleted?    │
│                                                 │
│  ○ Block deletion                               │
│  ○ Cascade delete tasks                         │
│  ○ Orphan tasks                                 │
│  ○ Other: [________________]                    │
│                                                 │
│           [Skip]  [Back]  [Next →]              │
└─────────────────────────────────────────────────┘
```

---

## Priority Recommendation

| Priority | Integration | Reason |
|----------|-------------|--------|
| **P0** | CLI | Baseline, CI/CD, universal |
| **P1** | MCP Server | Claude Code = primary audience |
| **P2** | VS Code Ext | Cursor/Windsurf users, best UX |

**Start with CLI** - minden más erre épül.

---

## Kapcsolódó

- [API-alapú Derivation Architektúra](./api-based-derivation-architecture.md)
- [Fizetési Modellek](./api-based-payment-models.md)

---
date: 2025-12-19
author: Claude Sonnet 4.5 + Human collaboration
version: 1.0
status: draft
purpose: Claude Code as the primary human-AI communication platform for AI-PDS
related: poc-tooling-design.md
---

# Claude Code mint AI-PDS Platform

## 💡 Alapvető felismerés

> **"A human-AI kommunikáció fő toolja a Claude Code lehetne."** - User insight

**Ez mindent megváltoztat!**

Ahelyett, hogy egy teljesen új CLI tool-t építenénk (`ai-pds`), **használjuk Claude Code-ot mint platformot** és építsünk rá!

---

## 🎯 Miért Claude Code?

### Claude Code már MEGOLDJA a tooling problémák 80%-át!

| Funkció | Új CLI építése | Claude Code használata |
|---------|----------------|------------------------|
| **File operations** | Implementálni kell | ✅ Beépített (Read, Write, Edit) |
| **Git integration** | Implementálni kell | ✅ Beépített (Bash tool) |
| **Diff preview** | UI kell | ✅ Beépített (Edit tool shows changes) |
| **Human approval** | Workflow kell | ✅ Beépített (conversational, interactive) |
| **Natural language** | LLM API kell | ✅ Ez maga Claude! |
| **Context management** | Implementálni kell | ✅ Beépített (200k context window) |
| **Multi-file editing** | Komplex | ✅ Beépített (Edit multiple files) |
| **Error handling** | Implementálni kell | ✅ Robusztus |
| **CLI interface** | Teljes CLI framework | ✅ Már van |
| **Conversation history** | State management | ✅ Automatikus |

**Eredmény:** ~80% kevesebb újra feltalálás!

---

## 🏗️ Új Architektúra: AI-PDS mint Claude Code Plugin/Skill System

### Option A: Claude Code Skills

**Claude Code Skills:** Markdown fájlok special frontmatter-rel, amik extended promptok.

```markdown
---
name: ai-pds-generate
description: Generate AI-PDS documentation from natural language
tags: [ai-pds, documentation]
---

# AI-PDS Document Generation

You are an AI-PDS documentation generator. Your task is to generate structured
documentation from natural language input.

## Workflow:
1. Parse user's natural language description
2. Determine affected document types (domain model, user stories, etc.)
3. Generate traceability IDs (US-XXX, AC-XXX-X, ENT-XXX)
4. Generate markdown documents with YAML frontmatter
5. Show diff preview
6. Ask for approval
7. Write files

## Document Types:
- domain-modelling/domain-vocabulary.md
- domain-modelling/domain-model.md
- requirements/user-stories.md
- requirements/acceptance-criteria.md
- architecture/decisions.md

## Traceability ID Scheme:
- User Story: US-001, US-002, ...
- Acceptance Criterion: AC-001-1, AC-001-2, ...
- Entity: ENT-User, ENT-Task, ...

## Example:
User input: "Add User entity with email, name, role"

Generate:
1. domain-modelling/domain-model.md
   - Add User entity with fields
   - Use anchor: {#ent-user}
2. domain-modelling/domain-vocabulary.md
   - Add User term definition
...
```

**Usage:**
```bash
# User runs Claude Code with skill
claude-code

# In conversation:
User: /ai-pds-generate Add User entity with email, name, role

# Claude Code loads the skill prompt and follows instructions
Claude: I'll generate AI-PDS documentation for the User entity.

Analyzing input...
- Entity: User
- Fields: email (string), name (string), role (enum)

I'll create/update these documents:
1. domain-modelling/domain-model.md
2. domain-modelling/domain-vocabulary.md
3. requirements/user-stories.md

Generating...

[Shows diff preview]

Approve? [y/n]
```

### Option B: Claude Code Plugin

**Claude Code Plugins:** More powerful, can include:
- Custom tools
- Hooks (lifecycle events)
- MCP servers
- Custom agents

```
.claude/plugins/ai-pds/
├── plugin.json              # Plugin manifest
├── skills/
│   ├── generate.md          # /ai-pds-generate skill
│   ├── validate.md          # /ai-pds-validate skill
│   ├── trace.md             # /ai-pds-trace skill
│   └── test-generate.md     # /ai-pds-test-generate skill
├── agents/
│   ├── doc-generator.md     # Document generator agent
│   └── test-generator.md    # Test generator agent
├── hooks/
│   └── pre-commit.sh        # Auto-validate before commit
└── tools/
    └── ai-pds-validator.js  # Custom validation tool (optional)
```

**plugin.json:**
```json
{
  "name": "ai-pds",
  "version": "0.1.0",
  "description": "AI-Ready Project Documentation System for Claude Code",
  "author": "AI-PDS Team",
  "skills": [
    "skills/generate.md",
    "skills/validate.md",
    "skills/trace.md",
    "skills/test-generate.md"
  ],
  "agents": [
    "agents/doc-generator.md",
    "agents/test-generator.md"
  ],
  "hooks": {
    "pre-commit": "hooks/pre-commit.sh"
  }
}
```

**Usage:**
```bash
# Install plugin
claude-code plugin install ai-pds

# Use skills
claude-code

User: /ai-pds-generate Add User entity
User: /ai-pds-test-generate --from-user-story US-001
User: /ai-pds-trace validate
```

### Option C: Hybrid - Built-in + Extensions

**Core functionality:** Built into Claude Code upstream (if accepted)
**Extensions:** Community plugins for specific workflows

---

## 🔌 MCP Server Architecture (Critical Component)

### Why MCP Servers for Loom?

**MCP (Model Context Protocol)** servers are **essential** for Loom's tooling architecture, not a "future enhancement". They provide:

✅ **Standardized tool interface** - Claude Code natively understands MCP
✅ **External integrations** - Database, GitHub, monitoring tools
✅ **Custom Loom tools** - Validation, derivation, traceability as native tools
✅ **Resource references** - `@loom:user-story://US-001`, `@loom:domain-model://ENT-Quote`
✅ **No custom CLI needed** - MCP servers ARE the implementation

**Key insight:** Instead of building a standalone `loom` CLI, build an **MCP server** that exposes Loom functionality as tools Claude Code can use natively.

---

### Loom MCP Server Design

**Server name:** `@loom/mcp-server`

**Installation:**
```bash
# Project-scoped (recommended - team shares config)
claude mcp add --transport stdio loom --scope project \
  -- npx -y @loom/mcp-server

# Or via plugin .mcp.json
{
  "mcpServers": {
    "loom": {
      "command": "npx",
      "args": ["-y", "@loom/mcp-server"],
      "env": {
        "LOOM_PROJECT_ROOT": "${CLAUDE_PROJECT_ROOT}",
        "LOOM_CONFIG": "${CLAUDE_PROJECT_ROOT}/.loom/config.yml"
      }
    }
  }
}
```

---

### MCP Tools Exposed by Loom Server

**1. `loom_validate`** - Traceability & quality validation
```typescript
// Tool definition
{
  name: "loom_validate",
  description: "Validate Loom documentation traceability and quality",
  inputSchema: {
    type: "object",
    properties: {
      stage: {
        type: "string",
        enum: ["traceability", "test-quality", "doc-sync", "id-scheme", "semantic", "full"],
        description: "Validation stage to run"
      },
      changedFilesOnly: {
        type: "boolean",
        description: "Only validate changed files (for pre-commit)"
      }
    }
  }
}

// Usage in Claude Code:
User: "Validate traceability for changed files"
Claude: [Calls loom_validate with stage="traceability", changedFilesOnly=true]

// Or as slash command (auto-generated):
User: /loom__loom__validate
```

**2. `loom_derive`** - Documentation derivation (L0→L1→L2→L3)
```typescript
{
  name: "loom_derive",
  description: "Derive documentation from foundational documents",
  inputSchema: {
    type: "object",
    properties: {
      fromUserStory: { type: "string", description: "User story ID (e.g., US-QUOTE-003)" },
      level: {
        type: "string",
        enum: ["L1", "L2", "L3"],
        description: "Derivation level (L1: domain model, L2: architecture, L3: tests)"
      },
      multiModel: { type: "boolean", description: "Use multi-model validation (enterprise)" }
    }
  }
}

// Usage:
User: "Derive L1 docs from US-QUOTE-003"
Claude: [Calls loom_derive with fromUserStory="US-QUOTE-003", level="L1"]
```

**3. `loom_trace`** - Traceability map generation
```typescript
{
  name: "loom_trace",
  description: "Generate traceability map showing relationships",
  inputSchema: {
    type: "object",
    properties: {
      from: { type: "string", description: "Starting point (e.g., US-001, ENT-User)" },
      format: {
        type: "string",
        enum: ["tree", "graph", "html", "json"],
        description: "Output format"
      }
    }
  }
}

// Usage:
User: "Show traceability map for US-QUOTE-003"
Claude: [Calls loom_trace with from="US-QUOTE-003", format="tree"]
```

**4. `loom_test_generate`** - TDAI test generation
```typescript
{
  name: "loom_test_generate",
  description: "Generate TDAI tests from acceptance criteria",
  inputSchema: {
    type: "object",
    properties: {
      fromAC: { type: "string", description: "Acceptance criteria ID" },
      negativeRatio: {
        type: "number",
        default: 0.3,
        description: "Negative test ratio (0.3 = 30%)"
      },
      framework: {
        type: "string",
        enum: ["jest", "vitest", "mocha", "pytest", "junit"],
        description: "Test framework"
      }
    }
  }
}

// Usage:
User: "Generate tests for AC-QUOTE-003-1 with 40% negative tests"
Claude: [Calls loom_test_generate with fromAC="AC-QUOTE-003-1", negativeRatio=0.4]
```

**5. `loom_init`** - Initialize Loom project structure
```typescript
{
  name: "loom_init",
  description: "Initialize Loom project structure and templates",
  inputSchema: {
    type: "object",
    properties: {
      projectName: { type: "string", description: "Project name" },
      domain: { type: "string", description: "Business domain (e.g., Sales & Billing)" },
      techStack: { type: "string", description: "Tech stack (e.g., TypeScript, Node.js)" }
    },
    required: ["projectName"]
  }
}
```

---

### MCP Resources Exposed by Loom Server

Resources allow **`@mention` references** in Claude Code:

**1. User Stories:** `@loom:user-story://US-QUOTE-003`
```json
{
  "uri": "loom://user-story/US-QUOTE-003",
  "name": "Cancel Quote",
  "description": "User story: Cancel Quote",
  "mimeType": "text/markdown"
}

// Returns markdown content of user story US-QUOTE-003
```

**2. Acceptance Criteria:** `@loom:ac://AC-QUOTE-003-1`
```json
{
  "uri": "loom://ac/AC-QUOTE-003-1",
  "name": "Only Sent quotes can be cancelled",
  "mimeType": "text/markdown"
}
```

**3. Domain Entities:** `@loom:entity://ENT-Quote`
```json
{
  "uri": "loom://entity/ENT-Quote",
  "name": "Quote Entity",
  "description": "Domain model: Quote",
  "mimeType": "application/json"
}

// Returns JSON schema of Quote entity
```

**4. Test Cases:** `@loom:test://TC-QUOTE-003-001`

**Usage in Claude Code:**
```
User: "Review @loom:user-story://US-QUOTE-003 and implement it"
Claude: [Fetches US-QUOTE-003 content via MCP resource]

User: "Compare @loom:entity://ENT-Quote with @loom:ac://AC-QUOTE-003-1"
Claude: [Fetches both resources and compares]
```

---

### MCP Prompts (Auto-Generated Slash Commands)

MCP servers can expose **prompts** that become slash commands:

```json
{
  "prompts": [
    {
      "name": "validate-full",
      "description": "Run full Loom validation pipeline",
      "arguments": []
    },
    {
      "name": "derive-from-story",
      "description": "Derive documentation from user story",
      "arguments": [
        { "name": "storyId", "description": "User story ID", "required": true }
      ]
    }
  ]
}
```

**Auto-generated slash commands:**
```
/loom__loom__validate-full
/loom__loom__derive-from-story US-QUOTE-003
```

---

### Loom MCP Server Implementation

**Technology stack:**
- **Language:** TypeScript (Node.js)
- **MCP SDK:** `@modelcontextprotocol/sdk`
- **Server type:** `stdio` (local process)
- **Distribution:** npm package `@loom/mcp-server`

**Project structure:**
```
packages/loom-mcp-server/
├── src/
│   ├── server.ts              # MCP server entry point
│   ├── tools/
│   │   ├── validate.ts        # loom_validate tool
│   │   ├── derive.ts          # loom_derive tool
│   │   ├── trace.ts           # loom_trace tool
│   │   ├── testGenerate.ts    # loom_test_generate tool
│   │   └── init.ts            # loom_init tool
│   ├── resources/
│   │   ├── userStory.ts       # User story resource handler
│   │   ├── entity.ts          # Entity resource handler
│   │   └── testCase.ts        # Test case resource handler
│   ├── core/
│   │   ├── validator.ts       # Validation engine
│   │   ├── derivation.ts      # Derivation engine
│   │   ├── traceability.ts    # Traceability parser
│   │   └── testGenerator.ts   # TDAI test generator
│   └── utils/
│       ├── fileParser.ts      # Markdown + YAML parsing
│       ├── idScheme.ts        # ID validation (US-XXX, AC-XXX-X)
│       └── loomConfig.ts      # .loom/config.yml loader
├── package.json
└── tsconfig.json
```

**Server initialization (src/server.ts):**
```typescript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import { validateTool } from './tools/validate.js';
import { deriveTool } from './tools/derive.js';
import { traceTool } from './tools/trace.js';
import { testGenerateTool } from './tools/testGenerate.js';
import { initTool } from './tools/init.js';
import { userStoryResource } from './resources/userStory.js';
import { entityResource } from './resources/entity.js';

const server = new Server(
  {
    name: 'loom',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
      resources: {},
      prompts: {},
    },
  }
);

// Register tools
server.setRequestHandler('tools/list', async () => ({
  tools: [
    validateTool.definition,
    deriveTool.definition,
    traceTool.definition,
    testGenerateTool.definition,
    initTool.definition,
  ],
}));

server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case 'loom_validate':
      return await validateTool.execute(args);
    case 'loom_derive':
      return await deriveTool.execute(args);
    case 'loom_trace':
      return await traceTool.execute(args);
    case 'loom_test_generate':
      return await testGenerateTool.execute(args);
    case 'loom_init':
      return await initTool.execute(args);
    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

// Register resources
server.setRequestHandler('resources/list', async () => ({
  resources: [
    { uri: 'loom://user-story/*', name: 'User Stories', mimeType: 'text/markdown' },
    { uri: 'loom://entity/*', name: 'Domain Entities', mimeType: 'application/json' },
    { uri: 'loom://ac/*', name: 'Acceptance Criteria', mimeType: 'text/markdown' },
    { uri: 'loom://test/*', name: 'Test Cases', mimeType: 'text/markdown' },
  ],
}));

server.setRequestHandler('resources/read', async (request) => {
  const { uri } = request.params;

  if (uri.startsWith('loom://user-story/')) {
    return await userStoryResource.read(uri);
  } else if (uri.startsWith('loom://entity/')) {
    return await entityResource.read(uri);
  }
  // ... other resources
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

---

### Integration with Claude Code Plugin

**Option 1: Plugin bundles MCP server**

`.claude/plugins/loom/plugin.json`:
```json
{
  "name": "loom",
  "version": "1.0.0",
  "description": "Loom AI Development Orchestration Platform",
  "mcpServers": {
    "loom": {
      "command": "${CLAUDE_PLUGIN_ROOT}/bin/loom-mcp-server",
      "env": {
        "LOOM_PROJECT_ROOT": "${CLAUDE_PROJECT_ROOT}",
        "LOOM_CONFIG": "${CLAUDE_PROJECT_ROOT}/.loom/config.yml"
      }
    }
  },
  "skills": [
    "skills/loom-generate.md",
    "skills/loom-validate.md"
  ]
}
```

**Option 2: Separate MCP server installation**

Users install MCP server separately:
```bash
# Install Loom MCP server globally
npm install -g @loom/mcp-server

# Add to Claude Code (project-scoped)
claude mcp add --transport stdio loom --scope project \
  -- loom-mcp-server

# Or add to project .mcp.json
{
  "mcpServers": {
    "loom": {
      "command": "loom-mcp-server",
      "env": {
        "LOOM_PROJECT_ROOT": "."
      }
    }
  }
}
```

---

### Benefits of MCP Server Architecture

**For Loom:**
✅ **No standalone CLI needed** - MCP server IS the implementation
✅ **Native Claude Code integration** - Tools, resources, prompts all native
✅ **Standardized protocol** - Follow MCP spec, works with any MCP client
✅ **Resource references** - `@loom:user-story://US-001` works everywhere
✅ **Automatic slash commands** - `/loom__loom__validate` auto-generated
✅ **Team sharing** - Project-scoped `.mcp.json` in git

**For developers:**
✅ **Familiar interface** - Use Claude Code skills + MCP tools
✅ **No context switching** - Everything in Claude Code
✅ **Natural language** - "Validate US-001" instead of `loom validate --story US-001`
✅ **Auto-discovery** - Claude knows what tools are available

**For implementation:**
✅ **80% less code** - No CLI framework, UI, approval workflow
✅ **Easier testing** - MCP protocol has standard test tools
✅ **Better error handling** - MCP has built-in error reporting
✅ **Future-proof** - MCP is evolving, Loom benefits automatically

---

### Example Workflows with MCP Server

**Workflow 1: Generate documentation from user story**
```
User: "I want to add a new feature: Quote Cancellation.
       Generate L1 documentation."

Claude: [Understands natural language request]
        Let me help you generate L1 documentation for Quote Cancellation.

        [Calls loom_derive tool internally]

        I've generated:
        - domain-model.md: ENT-Quote entity with cancellation fields
        - acceptance-criteria.md: 3 acceptance criteria (AC-QUOTE-003-1, AC-QUOTE-003-2, AC-QUOTE-003-3)
        - business-rules.md: BR-QUOTE-003 (status transition rules)

        Would you like me to write these files?

User: Yes

Claude: [Calls Write tool to create files]
        Files written. Run validation?

User: Yes

Claude: [Calls loom_validate tool]
        ✅ Validation passed! All traceability links valid.
```

**Workflow 2: Reference Loom resources**
```
User: "Review @loom:user-story://US-QUOTE-003 and implement it"

Claude: [Fetches US-QUOTE-003 via MCP resource]
        I've reviewed US-QUOTE-003: Cancel Quote.

        Let me check if tests exist first (TDAI enforcement):

        [Calls loom_test_generate tool]

        I'll generate tests first, then implementation.
        [Generates 10 tests: 3 positive, 4 negative, 2 boundary, 1 "should NOT"]

        Now implementing the cancelQuote function with @traceability...
```

**Workflow 3: Pre-commit validation**
```
User: "Commit my changes"

Claude: [Pre-commit hook triggers loom_validate]
        Running pre-commit validation...

        [Calls loom_validate with stage="traceability", changedFilesOnly=true]

        ❌ Validation failed:
        - src/domain/Quote.ts: Missing @traceability annotation on cancelQuote()

        Would you like me to add the annotation?
```

---

### Next Steps

**Phase 1: Core MCP Server (MVP)**
1. Implement `loom_validate` tool (traceability validation)
2. Implement `loom_derive` tool (L0→L1 derivation)
3. Implement user story resource (`@loom:user-story://...`)
4. Package as npm package `@loom/mcp-server`
5. Test with Claude Code

**Phase 2: Enhanced Tools**
6. Implement `loom_test_generate` (TDAI)
7. Implement `loom_trace` (traceability map)
8. Add more resources (entities, AC, tests)
9. Add MCP prompts (slash commands)

**Phase 3: Advanced Features**
10. Multi-model validation (enterprise)
11. Semantic consistency check (LLM-powered)
12. Integration with external MCP servers (GitHub, databases)

---

## 📋 Új PoC Architektúra (Claude Code-alapú)

### Amit NEM kell implementálni:

~~CLI framework~~ → Claude Code already has it
~~File I/O~~ → Claude Code Read/Write/Edit tools
~~Diff preview~~ → Claude Code shows diffs automatically
~~Git integration~~ → Claude Code Bash tool
~~Approval workflow~~ → Conversational, built-in
~~LLM API client~~ → Claude Code IS the LLM!
~~Context management~~ → Claude Code handles it
~~Error handling~~ → Claude Code has robust error handling

### Amit implementálni kell:

✅ **Skills / Prompts:**
- Document generation logic (in markdown skills)
- Test generation logic (in markdown skills)
- Validation rules (in skills or custom tools)
- Traceability parsing (in skills or custom tools)

✅ **Templates:**
- Document templates (domain model, user stories, etc.)
- Test templates (unit, integration, e2e)

✅ **Validation logic (optional custom tool):**
- YAML frontmatter validation
- Traceability link validation
- Test quality metrics

**Becsült fejlesztési idő:**
- **Eredeti PoC terv:** 2-4 hét (full CLI framework)
- **Claude Code-alapú:** **3-5 nap** (csak skills + templates!)

**80-90% csökkentés a fejlesztési időben!**

---

## 🔄 Új Workflow (Claude Code-dal)

### Workflow 1: Generate Documentation

```
┌─────────────────────────────────────────────────┐
│ User runs: claude-code                          │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│ User: /ai-pds-generate "Add User entity..."     │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│ Claude Code loads ai-pds-generate skill         │
│ (Extended prompt with instructions)             │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│ Claude analyzes input, determines docs          │
│ - domain-model.md                               │
│ - domain-vocabulary.md                          │
│ - user-stories.md                               │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│ Claude generates IDs                            │
│ - ENT-User                                      │
│ - US-001 (user management story)                │
│ - AC-001-1, AC-001-2, ...                       │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│ Claude uses Edit tool to update files           │
│ Shows diff preview automatically                │
└─────────────────┬───────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────┐
│ Claude asks: "Approve these changes?"           │
└─────────────────┬───────────────────────────────┘
                  │
          ┌───────┴────────┐
          │                │
        Yes               No
          │                │
          ▼                ▼
    ┌─────────┐      ┌──────────┐
    │ Write   │      │ Revise   │
    │ files   │      │ or cancel│
    └─────────┘      └──────────┘
```

**Példa conversation:**

```
User: /ai-pds-generate Add User entity with email, name, and role (admin or regular)

Claude: I'll generate AI-PDS documentation for the User entity.

Analyzing your input:
- Entity: User
- Fields:
  - email: string
  - name: string
  - role: enum (admin, regular)

I'll create/update these documents:

1. domain-modelling/domain-model.md
   - Add User entity {#ent-user}
   - Define fields and types

2. domain-modelling/domain-vocabulary.md
   - Add "User" term
   - Add "Role" term (admin, regular)

3. requirements/user-stories.md
   - US-001: User Management
   - AC-001-1: User has email
   - AC-001-2: User has name
   - AC-001-3: User has role

Let me generate these...

[Uses Edit tool on domain-modelling/domain-model.md]

───────────────────────────────────────────────
domain-modelling/domain-model.md
───────────────────────────────────────────────
+ ### User {#ent-user}
+
+ **Description:** A user of the system with authentication and authorization.
+
+ **Fields:**
+ - email: string {#ent-user:email} - User's email address
+ - name: string {#ent-user:name} - User's full name
+ - role: enum(admin, regular) {#ent-user:role} - User's role
+
+ **Traceability:**
+ - User Stories: US-001
───────────────────────────────────────────────

[Uses Edit tool on domain-modelling/domain-vocabulary.md]
[Uses Edit tool on requirements/user-stories.md]

I've prepared updates to 3 files.

Would you like me to:
1. Write these changes? (I'll commit them)
2. Review the changes first?
3. Modify something?

User: 1

Claude: [Uses Edit tool to write all files]

✓ domain-modelling/domain-model.md updated
✓ domain-modelling/domain-vocabulary.md updated
✓ requirements/user-stories.md updated

Done! Generated documentation for User entity.

Next steps:
- Review the generated docs
- Run /ai-pds-validate to check consistency
- Generate tests: /ai-pds-test-generate --from-user-story US-001

User: /ai-pds-test-generate --from-user-story US-001

Claude: I'll generate tests for US-001.

[Loads ai-pds-test-generate skill]
[Analyzes US-001 and acceptance criteria]
[Generates test plan]

Test Plan for US-001:

Unit Tests (8 tests):
  - AC-001-1 (email): 3 tests (valid, invalid, edge cases)
  - AC-001-2 (name): 2 tests (valid, edge cases)
  - AC-001-3 (role): 3 tests (valid, invalid, negative tests)

Integration Tests (2 tests):
  - User creation workflow
  - Role validation

Approve test plan? [y/n]

User: y

Claude: [Generates test files]
[Shows diffs]
[Writes tests]

✓ tests/domain/user.test.ts created (8 unit tests)
✓ tests/integration/user-workflow.test.ts created (2 integration tests)

All tests include @traceability annotations linking to US-001.

Run tests? [y/n]

User: y

Claude: [Uses Bash tool: npm test]

Running tests...

PASS  tests/domain/user.test.ts
  US-001: User Management
    AC-001-1: Email validation
      ✓ should accept valid email (5ms)
      ✓ should reject invalid email (3ms)
    ...

Tests: 10 passed, 10 total

✓ All tests pass!
✓ Test pyramid ratio: 80:20 (within target)
✓ Negative test ratio: 30% (> 20% target)

Done! US-001 is fully tested.
```

---

## 🎨 Skills Design

### Skill 1: /ai-pds-generate

**File:** `.claude/plugins/ai-pds/skills/generate.md`

```markdown
---
name: ai-pds-generate
description: Generate AI-PDS documentation from natural language
tags: [ai-pds, documentation, generation]
when_to_use: |
  Use this when the user wants to generate or update AI-PDS documentation.
  Examples:
  - "Add User entity with email and name"
  - "Create user story for user registration"
  - "Document the authentication flow"
---

# AI-PDS Documentation Generator

You are an expert at generating structured, AI-ready project documentation following the AI-PDS specification.

## Your Task

When the user provides a natural language description, you will:

1. **Analyze the input** and determine what type of information it contains:
   - Entity definitions → domain-model.md, domain-vocabulary.md
   - User stories → user-stories.md, acceptance-criteria.md
   - Architecture decisions → architecture/decisions.md
   - Workflows → domain-modelling/workflow-definitions.md

2. **Generate unique IDs** for all elements:
   - User Stories: `US-001`, `US-002`, ...
   - Acceptance Criteria: `AC-001-1`, `AC-001-2`, ... (linked to user story)
   - Entities: `ENT-User`, `ENT-Task`, ...
   - Entity Fields: `ENT-User:email`, `ENT-User:role`, ...

3. **Create/update documents** with proper structure:
   - YAML frontmatter with `status: "draft"`
   - Markdown headings with anchors: `{#us-001}`
   - Traceability links between documents
   - "Implementation refs" sections (initially empty)

4. **Show diff preview** using Edit tool (automatic)

5. **Ask for approval** before writing

6. **Write files** if approved

## Document Templates

### Domain Model

```markdown
---
status: "draft"
---
# Domain Model

## Entities

### EntityName {#ent-entityname}

**Description:** [Brief description]

**Fields:**
- field1: type {#ent-entityname:field1} - Description
- field2: type {#ent-entityname:field2} - Description

**Relationships:**
- [Describe relationships to other entities]

**Traceability:**
- User Stories: [US-XXX, ...]
```

### User Stories

```markdown
---
status: "draft"
---
# User Stories

## US-XXX: Story Title {#us-xxx}

**As a** [user type],
**I want** [goal],
**so that** [benefit].

**Acceptance Criteria:**
- [AC-XXX-1] Criterion 1
- [AC-XXX-2] Criterion 2

**Implementation refs:**
- Code: TBD
- Tests: TBD
- Status: 📝 Not implemented
```

## Workflow

1. Parse user input
2. Determine document types
3. Generate IDs (check existing for next number)
4. Generate markdown content
5. Use Edit tool (shows diff automatically)
6. Ask: "Approve these changes?"
7. If yes → write, if no → revise or cancel

## Important

- ALWAYS generate unique IDs
- ALWAYS add anchors to headings: `{#us-001}`
- ALWAYS link documents (traceability)
- NEVER overwrite without confirmation
- Show clear diff previews
```

### Skill 2: /ai-pds-test-generate

**File:** `.claude/plugins/ai-pds/skills/test-generate.md`

```markdown
---
name: ai-pds-test-generate
description: Generate comprehensive test suite from requirements (TDAI)
tags: [ai-pds, testing, tdai]
arguments:
  - name: from-user-story
    description: User story ID to generate tests for
    required: true
when_to_use: |
  Use this to generate tests from user stories and acceptance criteria.
  This implements Test-Driven AI Development (TDAI).
---

# AI-PDS Test Generator (TDAI)

You are an expert at generating comprehensive test suites that prevent AI hallucination.

## CRITICAL: Tests are CONSTRAINTS

Tests are NOT just validation - they CONSTRAIN AI behavior.

**Negative tests especially:** They tell AI what NOT to do.

Example:
```typescript
// Requirement: "Password ≥ 8 chars" (NO uppercase requirement!)

// NEGATIVE TEST (prevents hallucination):
it('should accept lowercase-only password', () => {
  expect(validatePassword('lowercase')).toBe(true);
  // If AI adds uppercase check → TEST FAILS! Hallucination caught!
});
```

## Your Task

When given a user story ID (e.g., US-001):

1. **Read user story and acceptance criteria**
   - Use Read tool on requirements/user-stories.md
   - Extract all AC-XXX-X for this story

2. **Generate test plan** (show to user first!)
   - Unit tests: 5-10 per AC
   - Integration tests: 2-5 per story
   - E2E tests: 1-3 per story
   - **Ensure ≥20% are negative tests!**

3. **Ask for approval** of test plan

4. **Generate test files** with structure:
   ```typescript
   /**
    * @traceability US-XXX
    */
   describe('US-XXX: Story Title', () => {
     describe('AC-XXX-1: Criterion', () => {
       /**
        * @implements AC-XXX-1
        */
       it('should [positive behavior]', () => { ... });

       /**
        * @implements AC-XXX-1
        * NEGATIVE TEST
        */
       it('should accept [without extra constraint]', () => { ... });
     });
   });
   ```

5. **Show diff, get approval, write**

## Test Pyramid (70:20:10)

- 70% Unit tests (fast, focused)
- 20% Integration tests (moderate)
- 10% E2E tests (slow, comprehensive)

## Test Types per AC

For EACH acceptance criterion, generate:

1. **3+ Positive tests** (happy path variations)
2. **3+ Negative tests** (what should NOT fail)
   - "should accept X without Y" (if Y not required)
   - Edge cases that should still pass
3. **2+ Boundary tests** (min, max, off-by-one)
4. **1+ "Should NOT" test** (explicit negative behavior)

## Example

User: /ai-pds-test-generate --from-user-story US-001

You:
1. Read US-001
2. Find AC-001-1: "Email must be valid format"
       AC-001-2: "Password ≥ 8 characters"
3. Generate test plan (show to user)
4. User approves
5. Generate tests with @traceability
6. Write test files
```

### Skill 3: /ai-pds-validate

```markdown
---
name: ai-pds-validate
description: Validate AI-PDS documentation consistency and traceability
tags: [ai-pds, validation, traceability]
---

# AI-PDS Validator

Validate AI-PDS documentation for:

1. **YAML frontmatter**
   - All docs have `status` field
   - Status values: draft | to review | approved | living

2. **Traceability**
   - All IDs are unique
   - All referenced IDs exist
   - All ACs have @implements in code/tests

3. **Consistency**
   - Entities in domain-vocabulary also in domain-model
   - User stories reference defined entities

4. **Structure**
   - Files in correct folders
   - Proper anchors: {#us-001}

## Workflow

1. Use Glob to find all .md files in ai-pds/
2. Use Read to parse each file
3. Extract IDs, validate format
4. Check cross-references
5. Report errors/warnings

## Output Format

```
AI-PDS Validation Report
────────────────────────────────

✓ YAML frontmatter (12 files checked)
✗ Traceability (2 errors)
  - US-005: Referenced in code but not defined
  - AC-003-2: No @implements found in codebase
⚠ Consistency (1 warning)
  - Entity "Comment" in user-stories.md but not in domain-model.md

Summary: 2 errors, 1 warning
```
```

### Skill 4: /ai-pds-trace

```markdown
---
name: ai-pds-trace
description: Show traceability map (requirement → code → test)
tags: [ai-pds, traceability, visualization]
---

# AI-PDS Traceability Map

Show visual traceability graph.

## Workflow

1. Parse all docs for IDs and @traceability annotations
2. Parse code for @traceability and @implements
3. Build graph
4. Render as tree

## Output Format

```
Traceability Map
────────────────────────────────

US-001: User Registration
  ├─> AC-001-1: Email valid
  │     ├─> src/auth/register.ts:validateEmail() ✓
  │     └─> tests/auth/register.test.ts ✓
  ├─> AC-001-2: Password ≥ 8
  │     ├─> src/auth/register.ts:validatePassword() ✓
  │     └─> tests/auth/register.test.ts ✓
  └─> AC-001-3: Send confirmation
        ├─> src/auth/register.ts:sendConfirmation() ✓
        └─> tests/auth/register.test.ts ✗ MISSING!

ENT-User
  └─> src/domain/User.ts ✓
        └─> tests/domain/User.test.ts ✓

Summary:
  - 3 user stories
  - 8 acceptance criteria
  - 7/8 implemented (87.5%)
  - 1 missing test
```
```

---

## 📦 Distribution & Installation

### Option 1: Claude Code Plugin Registry (future)

```bash
claude-code plugin install ai-pds
```

### Option 2: Git Repository

```bash
# Clone plugin repo
git clone https://github.com/ai-pds/claude-code-plugin.git

# Copy to Claude plugins directory
cp -r claude-code-plugin ~/.claude/plugins/ai-pds

# Or symlink for development
ln -s $(pwd)/claude-code-plugin ~/.claude/plugins/ai-pds
```

### Option 3: Template Repository

```bash
# Use template
gh repo create my-project --template ai-pds/claude-code-template

cd my-project

# .claude/plugins/ai-pds already included!

# Start using
claude-code
```

---

## 🎯 Benefits vs. Standalone CLI

| Aspect | Standalone CLI (`ai-pds`) | Claude Code Plugin |
|--------|---------------------------|-------------------|
| **Development time** | 2-4 weeks | 3-5 days |
| **Code to maintain** | ~5000 LOC | ~500 LOC (mostly prompts) |
| **File operations** | Custom implementation | Built-in |
| **Diff preview** | Build UI | Built-in |
| **Git integration** | Implement wrapper | Built-in |
| **Human approval** | Build workflow | Conversational |
| **Natural language** | LLM API calls | Native |
| **Context management** | Implement | Built-in |
| **Distribution** | npm/pip package | Plugin install |
| **User learning curve** | New tool to learn | Already know Claude Code |
| **Integration** | Standalone | Seamless with Claude Code workflow |

---

## 🚀 PoC Timeline (Claude Code-based)

### Week 1: Skills & Templates (3-5 days)

**Goal:** Basic skills working

**Tasks:**
- [ ] Create plugin structure
- [ ] Write `/ai-pds-generate` skill
  - [ ] Entity generation template
  - [ ] User story generation template
  - [ ] ID generation logic (in prompt)
- [ ] Write `/ai-pds-validate` skill
  - [ ] Basic validation rules (in prompt)
- [ ] Test with demo project (manual)

**Deliverable:** Can generate docs with Claude Code

### Week 2: TDAI & Traceability (2-3 days)

**Goal:** Test generation + validation working

**Tasks:**
- [ ] Write `/ai-pds-test-generate` skill
  - [ ] Test plan generation
  - [ ] Unit/Integration/E2E templates
  - [ ] Negative test enforcer (in prompt)
- [ ] Write `/ai-pds-trace` skill
  - [ ] Traceability parsing logic
  - [ ] Graph rendering
- [ ] Test E2E workflow

**Deliverable:** Full TDAI workflow works

### Week 3-4: Polish & Demo (3-5 days)

**Goal:** Production-ready plugin

**Tasks:**
- [ ] Refine skills based on testing
- [ ] Write documentation (README, examples)
- [ ] Create demo project (TODO app)
  - [ ] Use Claude Code + AI-PDS plugin
  - [ ] Full docs, tests, code with traceability
- [ ] Record demo video
- [ ] Publish plugin

**Deliverable:** Ready for users

**Total: 8-13 days** (vs 20-28 days for standalone CLI)

---

## 🔮 Future Enhancements

### Phase 2: Advanced Features

1. **Custom MCP Server** (optional)
   - Traceability validation as tool
   - Test quality metrics as tool
   - More complex validation logic

2. **Hooks**
   - Pre-commit: auto-validate
   - Post-generate: auto-format
   - Pre-push: check traceability

3. **Agents**
   - Specialized doc generator agent
   - Specialized test generator agent
   - More autonomous workflows

### Phase 3: Integration

4. **IDE Integration**
   - VSCode extension using Claude Code
   - Inline traceability links
   - Quick actions (generate tests, validate)

5. **CI/CD**
   - GitHub Action using Claude Code
   - Auto-generate docs on PR
   - Validate traceability in CI

---

## 💭 Conclusion

**Using Claude Code as the platform is a game-changer:**

✅ **80-90% less code** to maintain
✅ **8-13 days instead of 20-28 days** development
✅ **Better UX** (users already know Claude Code)
✅ **Seamless integration** with existing workflow
✅ **Natural language native** (no LLM API wrapper needed)
✅ **Built-in approval workflow** (conversational)
✅ **Built-in file operations, git, diff preview**

**The PoC becomes:**
- A set of well-crafted skills (markdown prompts)
- Document/test templates
- Optional custom tools for complex validation

**Instead of:**
- A full CLI framework
- LLM API client
- File I/O system
- Diff viewer UI
- Approval workflow engine
- Git wrapper
- Error handling
- Context management

**This is the RIGHT architecture!**

---

*Next step: Start implementing skills for Claude Code.*

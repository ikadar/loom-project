---
date: 2025-12-19
updated: 2025-12-26
author: Claude Sonnet 4.5 + Human collaboration
version: 2.0
status: draft
purpose: MCP Server design a Loom platformhoz
see-also:
  - platform-architecture.md              # Technikai architektúra
  - ../business/platform-business-model.md # Üzleti modell, monetizáció
---

# Loom MCP Server Design

> Ez a dokumentum az MCP tool/resource definíciókat tartalmazza. Az architektúra a `platform-architecture.md`-ben, az üzleti modell a `platform-business-model.md`-ben van.

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


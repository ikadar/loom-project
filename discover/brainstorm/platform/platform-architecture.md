---
title: "Loom Platform Architecture"
status: draft
created: 2025-12-23
updated: 2025-12-26
split-from: platform-strategy.md
see-also:
  - ../business/platform-business-model.md  # Üzleti modell, monetizáció, jogi kérdések
  - mcp-and-skills-design.md                # MCP tool/resource definíciók
---

# Loom Platform Architecture

> Technikai architektúra: Plugin + Dumb CLI + Smart SaaS

---

## Executive Summary

A Loom **Plugin + CLI Wrapper hybrid** architektúrát használ:
- **Plugin**: UX layer Claude Code-ban (slash commands, orchestration)
- **CLI Wrapper**: Thin client, promptokat a SaaS-ból kéri le
- **SaaS**: Itt él az IP (prompts, checklists, knowledge base)

---

## Architektúra

### Plugin + Wrapper Hybrid

```
┌─────────────────────────────────────────────────────────────────┐
│                     User's Claude Code                           │
│                                                                  │
│  > /loom-derive ./l0/user-stories.md ./l1                       │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  Loom Plugin (látható, orchestration only)                 │ │
│  │                                                             │ │
│  │  commands/loom-derive.md:                                  │ │
│  │    "Run loom-cli derive $1 $2 --format json"               │ │
│  │    "Parse output, handle interview interactively"          │ │
│  │                                                             │ │
│  └──────────────────────────┬─────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  loom-cli (thin client)                                    │ │
│  │                                                             │ │
│  │  - Fetches prompts from SaaS API                          │ │
│  │  - claude -p with fetched prompts                         │ │
│  │  - Session management for interview                        │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  User's Claude Subscription fedezi az LLM költséget             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Dumb CLI + Smart SaaS

```
┌─────────────────────────────────────────────────────────────┐
│  loom-cli (INGYENES, "buta")                                │
│  ├─→ Orchestration only                                     │
│  ├─→ NO embedded prompts, NO checklists                     │
│  ├─→ Fetches prompts/checklists from SaaS API               │
│  └─→ Calls claude -p with fetched prompts                   │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  Loom SaaS API (FIZETŐS, "okos")                            │
│  ├─→ GET /api/prompts/domain-discovery                      │
│  ├─→ GET /api/prompts/completeness-analysis                 │
│  ├─→ GET /api/prompts/derivation                            │
│  ├─→ GET /api/checklists/entity                             │
│  ├─→ GET /api/checklists/operation                          │
│  └─→ GET /api/templates/{domain}                            │
└─────────────────────────────────────────────────────────────┘
```

### Headless Mode Használata

```bash
# A loom-cli belsőleg ezt hívja:
claude -p "$(fetch_prompt_from_saas)" \
    --append-system-prompt "Context: $user_input" \
    --tools "Read,Write,Edit,Grep,Glob" \
    --allowedTools "Read,Write,Edit" \
    --mcp-config ./loom-mcp.json \
    --output-format json \
    --resume "$session_id"
```

**Headless mode előnyei:**
- Session management (`--resume`) → Interview folytatható
- MCP config (`--mcp-config`) → RAG működik
- JSON output → Parsolható eredmények
- Tool control → Finomhangolható
- User subscription → $0 LLM költség

---

## UX Flow

### Claude Code-ban (casual user)

```
User: /loom-derive ./specs/l0/user-stories.md ./specs/l1

Claude: Running Loom derivation...

         Analyzing: user-stories.md
         Found: 3 user stories, 12 entities, 8 operations

         ⚠️ 5 ambiguities need resolution:

         **1. Station Entity**
         What happens to scheduled tasks when a station is deleted?

         • (a) Block deletion if tasks exist
         • (b) Cascade delete all tasks
         • (c) Orphan tasks (set station_id = null)

User: (a), block deletion

Claude: ✓ Decision recorded: BR-STATION-001

        [... more questions ...]

        ✓ Derivation complete!

        Generated:
        • ./specs/l1/acceptance-criteria.md (24 ACs)
        • ./specs/l1/business-rules.md (18 BRs)
```

### Terminálban (power user)

```bash
$ loom derive ./l0 ./l1

Analyzing L0 documents...
Found 5 ambiguities.

[1/5] Station: What happens to tasks when deleted?
  1) Block deletion
  2) Cascade delete
  3) Orphan tasks

Choice: 1

✓ Derivation complete. Output: ./l1/
```

### CI/CD (automation)

```bash
# Batch mode - use existing decisions
loom derive ./l0 ./l1 --batch --decisions ./l1/decisions.md

# Strict mode - fail on new ambiguities
loom derive ./l0 ./l1 --batch --strict
```

---

## Komponensek

### 1. Plugin (Open, Orchestration)

```
loom-plugin/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── loom-derive.md      # Calls loom-cli, handles interview
│   ├── loom-analyze.md     # Analysis wrapper
│   └── loom-status.md      # Session status
└── README.md
```

**Slash command példa:**
```markdown
---
description: Derive L1 from L0 using Loom
---

Run the loom-cli to derive L1:
\`\`\`bash
loom-cli derive "$1" "$2" --format json
\`\`\`

Parse the JSON response:
- If ambiguities found, ask user each question interactively
- Pass answers: `loom-cli answer --session <id> "<answer>"`
- Show final results when complete
```

### 2. CLI Wrapper (Thin Client)

```
loom-cli (Go binary)
├── cmd/
│   ├── derive.go       # Main derivation command
│   ├── analyze.go      # Analysis command
│   ├── answer.go       # Interview answer
│   └── login.go        # License management
├── internal/
│   ├── license/        # Validation logic
│   ├── saas/           # SaaS API client (prompts, checklists)
│   ├── headless/       # Claude CLI wrapper
│   └── session/        # Session management
└── main.go
```

**CLI mint thin client:**

```go
// loom-cli/internal/prompts/fetch.go

func FetchPrompt(ctx context.Context, promptID string) (string, error) {
    client := saas.NewClient(os.Getenv("LOOM_API_KEY"))

    resp, err := client.Get("/api/prompts/" + promptID)
    if err != nil {
        return "", fmt.Errorf("failed to fetch prompt: %w", err)
    }

    // Prompts are never cached locally (IP protection)
    return resp.Content, nil
}

// Usage in derive:
func runDerive() error {
    domainPrompt, _ := FetchPrompt(ctx, "domain-discovery")
    analysisPrompt, _ := FetchPrompt(ctx, "completeness-analysis")

    // Use fetched prompts with claude -p
    client.Call(domainPrompt + userInput)
}
```

### 3. SaaS Backend (Smart, IP lives here)

```
loom.dev/api/
├── GET  /prompts/{id}      # Fetch prompt by ID
├── GET  /checklists/{id}   # Fetch checklist
├── GET  /templates/{domain} # Domain-specific templates
├── POST /validate          # Validate license token
├── POST /login             # OAuth flow
├── GET  /usage             # Usage stats (for limits)
└── POST /checkout          # Stripe integration
```

---

## Technology Stack

### CLI Nyelv: Go

**Követelmények:**
- Cross-platform (Windows, macOS, Linux)
- Single binary (no runtime dependencies)
- Gyors startup (CLI UX)
- HTTP client (SaaS API calls)

**Miért Go?**

| Szempont | Go | Rust | Python | Node.js |
|----------|-----|------|--------|---------|
| Single binary | ✅ | ✅ | ❌ | ❌ |
| Cross-compile | ✅ Trivális | ✅ | ⚠️ Nehéz | ⚠️ Nehéz |
| Startup time | ✅ <100ms | ✅ | ❌ Lassú | ⚠️ |
| Binary size | ~15-20MB | ~5-15MB | ~50-100MB | ~40-80MB |
| Learning curve | ✅ Könnyű | ❌ Meredek | ✅ | ✅ |

**Go előnyök:**

```
✅ Cross-compilation egyetlen parancs:
   GOOS=windows GOARCH=amd64 go build
   GOOS=darwin GOARCH=arm64 go build
   GOOS=linux GOARCH=amd64 go build

✅ Iparági standard CLI-khez:
   Docker, Kubernetes, GitHub CLI, Terraform

✅ Kiváló CLI library-k:
   cobra (commands), viper (config), bubbletea (TUI)
```

### Tech Stack Összefoglaló

| Komponens | Technológia |
|-----------|-------------|
| **CLI** | Go + Cobra |
| **Config** | Viper |
| **HTTP** | stdlib net/http |
| **SaaS Backend** | Go / Node.js |
| **Plugin** | Markdown (Claude Code native) |
| **MCP Server** | TypeScript / Python |

### Build & Distribution

```bash
# Build all platforms
make build-all

# Output:
# dist/loom-cli-darwin-amd64
# dist/loom-cli-darwin-arm64
# dist/loom-cli-linux-amd64
# dist/loom-cli-linux-arm64
# dist/loom-cli-windows-amd64.exe

# Distribution:
# - GitHub Releases
# - Homebrew (macOS)
# - apt/yum repos (Linux)
# - Chocolatey / Scoop (Windows)
```

---

## Kapcsolódó

- [Business Model](../business/platform-business-model.md) - Monetizáció, IP védelem, jogi kérdések
- [MCP and Skills Design](./mcp-and-skills-design.md) - MCP tool/resource definíciók
- [Prompt Engineering Guidelines](./prompt-engineering-guidelines.md) - Anthropic best practices a Loom promptokhoz

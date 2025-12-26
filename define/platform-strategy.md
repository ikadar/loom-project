# AI-DOP Platform Strategy

> Stratégiai döntés: Plugin + Headless CLI Wrapper hybrid, részleges IP védelemmel, CLI subscription monetizációval.

**Létrehozva:** 2025-12-23
**Frissítve:** 2025-12-23
**Státusz:** AKTUÁLIS IRÁNY (policy tisztázás szükséges!)

---

## Executive Summary

Az AI-DOP **Plugin + CLI Wrapper hybrid** architektúrát használ:
- **Plugin**: UX layer Claude Code-ban (slash commands, orchestration)
- **CLI Wrapper**: Compiled binary, titkos promptokkal, headless mode-ban hívja a Claude-ot
- **Monetizáció**: CLI subscription (license validation)

**Trade-off:** ~70% IP védelem + CLI subscription = működő üzleti modell

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
│  │  AI-DOP Plugin (látható, orchestration only)               │ │
│  │                                                             │ │
│  │  commands/loom-derive.md:                                  │ │
│  │    "Run loom-cli derive $1 $2 --format json"               │ │
│  │    "Parse output, handle interview interactively"          │ │
│  │                                                             │ │
│  └──────────────────────────┬─────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  loom-cli (compiled binary)              🔒 ~70% IP védett │ │
│  │                                                             │ │
│  │  - License validation (subscription check)                 │ │
│  │  - Encrypted/embedded prompts                              │ │
│  │  - claude -p "secret prompt" --output-format json          │ │
│  │  - Session management for interview                        │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  User's Claude Subscription fedezi az LLM költséget             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Headless Mode Használata

```bash
# A loom-cli belsőleg ezt hívja:
claude -p "$(decrypt_embedded_prompt)" \
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

## IP Védelem

### Védelmi szintek

```
Remote API (szerveren)    ██████████  ~95%  (nem másolható)
Compiled + Encrypted      ████████░░  ~70%  (effort kell) ← VÁLASZTOTT
Compiled (plain)          ██████░░░░  ~50%  (strings, strace)
Plugin (plain text)       ██░░░░░░░░  ~10%  (bárki látja)
```

### Mit véd, mit nem?

| Támadás | Védelem |
|---------|---------|
| Casual másolás (fájl megnézése) | ✅ Véd |
| `strings loom-cli` | ⚠️ Encryption segít |
| Decompilation (Ghidra, IDA) | ⚠️ Nehezít, de nem lehetetlen |
| Runtime intercept (strace, ps) | ❌ Nem véd |

### Értékelés

**~70% védelem elég:**
- Casual másolás ellen véd
- Konkurens effort-öt igényel a reverse engineering
- A valódi védelem: iteration speed + ecosystem + brand
- Tökéletes védelem (Remote API) túl drága lenne

---

## Monetizáció: CLI Subscription

### License Validation Flow

```
┌─────────────────────────────────────────────────────────────┐
│  $ loom derive ./l0 ./l1                                    │
│                                                              │
│  loom-cli startup:                                          │
│  ├── Load cached license                                    │
│  ├── If expired/missing:                                    │
│  │   └── POST loom.dev/api/validate {token}                │
│  │       └── Response: {valid, tier, features}             │
│  ├── Cache for 24h (offline support)                       │
│  ├── Feature gate based on tier                            │
│  └── Run command                                            │
│                                                              │
│  First time:                                                 │
│  $ loom login                                                │
│  → Browser: loom.dev/login                                  │
│  → Subscribe / enter license                                │
│  → Token saved: ~/.loom/license                             │
└─────────────────────────────────────────────────────────────┘
```

### Tier Struktúra

| Tier | Ár | Limit | Features |
|------|-----|-------|----------|
| **Free** | $0 | 10 derivation/hó | Basic prompts, standard templates |
| **Pro** | $19/hó | Unlimited | Latest prompts, priority updates, advanced templates |
| **Team** | $49/hó/user | Unlimited | Pro + shared config, team dashboard |
| **Enterprise** | Custom | Unlimited | Team + SSO, audit, on-prem, dedicated support |

### Offline Működés

```
License cache valid?
├── Yes → Run normally
├── No, but within 7 day grace → Warn, run
└── No, expired → "Connect to validate" error
```

### Miért CLI Subscription?

| Szempont | Előny |
|----------|-------|
| **Egyszerű** | Nincs bonyolult SaaS infra |
| **Fair** | Fizetsz = használod |
| **Kipróbálható** | Free tier 10 derivation/hó |
| **Skálázható** | License server minimális költség |
| **Offline** | 7 nap grace period |

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
ai-dop-plugin/
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
description: Derive L1 from L0 using AI-DOP
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

### 2. CLI Wrapper (Protected, Core Logic)

```
loom-cli (Go/Rust binary)
├── cmd/
│   ├── derive.go       # Main derivation command
│   ├── analyze.go      # Analysis command
│   ├── answer.go       # Interview answer
│   └── login.go        # License management
├── internal/
│   ├── license/        # Validation logic
│   ├── prompts/        # Embedded, encrypted prompts
│   ├── headless/       # Claude CLI wrapper
│   └── session/        # Session management
└── main.go
```

### 3. License Server (Simple API)

```
loom.dev/api/
├── POST /validate      # Validate license token
├── POST /login         # OAuth flow
├── GET  /usage         # Usage stats (for limits)
└── POST /checkout      # Stripe integration
```

---

## Technology Stack

### CLI Nyelv: Go

**Követelmények:**
- Cross-platform (Windows, macOS, Linux)
- Single binary (no runtime dependencies)
- Gyors startup (CLI UX)
- Encryption support (prompt védelem)
- HTTP client (license validation)

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

✅ Beépített crypto:
   crypto/aes - prompt encryption
```

### Tech Stack Összefoglaló

| Komponens | Technológia |
|-----------|-------------|
| **CLI** | Go + Cobra |
| **Config** | Viper |
| **Crypto** | stdlib crypto/aes |
| **HTTP** | stdlib net/http |
| **License Server** | Go / Node.js (simple API) |
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

## Go-to-Market

### Phase 1: MVP

1. CLI wrapper (Go binary, basic prompts)
2. License server (simple validation)
3. Free tier (10/hó limit)
4. Plugin (slash commands)
5. Documentation

### Phase 2: Growth

1. Pro tier launch ($19/hó)
2. Prompt improvements based on feedback
3. Advanced templates
4. Community building

### Phase 3: Team/Enterprise

1. Team tier ($49/hó/user)
2. Shared config, dashboard
3. Enterprise features (SSO, audit)
4. On-premise option (Remote API, full IP védelem)

---

## Költségek

### AI-DOP Költségei

| Komponens | Költség |
|-----------|---------|
| License server | ~$20-50/hó (simple API) |
| Domain, hosting | ~$20/hó |
| Stripe fees | 2.9% + $0.30/tranzakció |
| **LLM költség** | **$0** (user subscription) |

### User Költségei

| Tétel | Költség |
|-------|---------|
| Claude Code subscription | $20-100/hó (már van) |
| AI-DOP Free | $0 |
| AI-DOP Pro | $19/hó |

---

## Kockázatok

| Kockázat | Valószínűség | Hatás | Mitigáció |
|----------|--------------|-------|-----------|
| Prompt reverse engineering | Közepes | Közepes | Iteration speed, 70% elég |
| Anthropic beépíti | Közepes | Magas | Niche features, ecosystem |
| Claude Code pricing változás | Alacsony | Közepes | Adaptáció |
| License crack | Alacsony | Közepes | Server-side validation |
| **Policy szürke zóna** | **Közepes** | **Magas** | **Anthropic megerősítés kérése** |

---

## FONTOS: Agent SDK vs Claude Code Policy

**Frissítve:** 2025-12-23

### Anthropic Policy az Agent SDK-ról

Az Agent SDK dokumentációból (https://platform.claude.com/docs/en/agent-sdk/overview):

> "Unless previously approved, **we do not allow third party developers to offer Claude.ai login or rate limits** for their products, including agents built on the Claude Agent SDK. **Please use the API key authentication methods** described in this document instead."

### Mit jelent ez?

```
┌─────────────────────────────────────────────────────────────────┐
│  Claude Code CLI (claude -p)                                    │
│  └─→ Saját használatra, Claude Code subscription-nel            │
│      (Személyes/interaktív használat)                           │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  Agent SDK (Python/TypeScript)                                  │
│  └─→ ANTHROPIC_API_KEY kötelező                                 │
│      Third-party devs NEM használhatják a subscription-t!       │
│      Pay-per-token billing                                      │
└─────────────────────────────────────────────────────────────────┘
```

### Az AI-DOP Szürke Zóna Kérdése

**AI-DOP működése:**
- User fizet az AI-DOP CLI-ért (subscription)
- AI-DOP hívja a `claude -p`-t a user gépén
- A user saját Claude Code subscription-jét használja

**A kérdés:** Ez "offering Claude through your product"?

#### Érvek, hogy OK:

| Érv | Magyarázat |
|-----|------------|
| User saját gépén fut | Nincs proxy, nincs SaaS server |
| User saját subscription-je | Mint bármely automation script |
| Nem szolgáltatás | Desktop tool, nem hosted service |
| User maga is megcsinálhatná | Csak kényelem/automatizálás |
| Hasonló létező tools | VS Code extensions, Cursor, Aider |

#### Érvek, hogy Szürke Zóna:

| Érv | Aggály |
|-----|--------|
| Pénzt kérünk érte | Claude képességeit monetizáljuk |
| A prompt a miénk | IP a Claude outputján alapul |
| Ezért veszik meg | Lényegében Claude-ra épülő termék |

### Javaslat: Anthropic Megerősítés

**TEENDŐ:** Mielőtt production-be megyünk, kérjünk írásos megerősítést az Anthropic-tól.

**Kapcsolat:**
- https://www.anthropic.com/contact-sales
- support@anthropic.com

**Email tartalma:**
1. CLI tool ami `claude -p`-t hív a user gépén
2. User saját Claude Code subscription-jét használja
3. Mi a CLI-t monetizáljuk (prompt IP), nem a Claude hozzáférést
4. Kérés: írásos megerősítés, hogy ez megfelel a policy-nek

**Státusz:** TISZTÁZANDÓ

### Alternatívák ha NEM engedélyezik

1. **API-based modell**: ANTHROPIC_API_KEY használata, pay-per-token
   - Drágább a user-nek
   - De egyértelműen compliant

2. **Hybrid**: Free tier = user subscription, Pro = API key
   - Komplexebb implementáció
   - De mindkét use case-t lefedi

3. **Enterprise focus**: On-prem API server
   - Teljes IP védelem
   - De magasabb belépési küszöb

---

## Legitimációs Stratégia: Plugin + SaaS + Helper CLI

**Frissítve:** 2025-12-23

### Free CLI + Paid SaaS Modell

Ahelyett, hogy a CLI-t monetizálnánk közvetlenül:

```
┌─────────────────────────────────────────────────────────────┐
│  loom-cli (INGYENES)                                        │
│  ├─→ claude -p (user subscription)                          │
│  └─→ Sync to SaaS (requires account)                        │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  Loom SaaS (FIZETŐS)                                        │
│  ├─→ Spec document storage & versioning                     │
│  ├─→ Decision history & audit trail                         │
│  ├─→ Team collaboration                                     │
│  ├─→ Project management                                     │
│  ├─→ Template library                                       │
│  ├─→ Jira/GitHub integration                                │
│  └─→ Analytics & reporting                                  │
└─────────────────────────────────────────────────────────────┘
```

### Átlátszó trükk vagy legitim modell?

**Attól függ, hogy a SaaS ad-e valódi értéket.**

#### Ha a SaaS CSAK license gate → ❌ Átlátszó trükk
- Login page + "subscription active" check
- Semmi más funkció
- Egyértelmű megkerülés

#### Ha a SaaS VALÓDI értéket ad → ✅ Legitim modell
- Dokumentum kezelés, verziókövetés
- Team collaboration
- PM funkciók
- Integrációk

### Iparági precedensek

| Product | CLI | SaaS |
|---------|-----|------|
| **GitHub CLI** | Ingyenes | GitHub.com (private repos, actions) |
| **Terraform CLI** | Ingyenes | Terraform Cloud (state, teams) |
| **Docker CLI** | Ingyenes | Docker Hub (private repos) |
| **Vercel CLI** | Ingyenes | Vercel (hosting, analytics) |

Ezek mind legitim modellek, mert a SaaS valódi értéket ad.

### AI-DOP SaaS Értékajánlat

A SaaS-nak valódi értéket kell adnia:

1. **Spec Repository** - verziókövetés, history
2. **Decision Log** - minden döntés auditálható
3. **Team Workspace** - kollaboráció spec-eken
4. **Project Dashboard** - spec coverage, progress
5. **Template Marketplace** - domain-specifikus template-ek
6. **Integrations** - Jira, GitHub, Linear, Notion
7. **AI Review** - spec quality scoring, gap analysis

---

## Slash Command Only Policy

### A CLI csak slash command-on keresztül hívható

```
❌ NEM megengedett:
$ loom-cli derive ./l0 ./l1

✅ Megengedett:
Claude Code > /loom-derive ./l0 ./l1
             └─→ Plugin hívja a loom-cli-t
```

### Miért számít ez?

| Aspektus | Direct CLI | Slash Command → CLI |
|----------|-----------|---------------------|
| Mi a "termék"? | CLI tool | Claude Code plugin |
| User mit lát? | Terminal parancs | Claude Code UX |
| Official mechanism? | Nem | Igen (plugin system) |
| CLI szerepe | Főszereplő | Implementation detail |

### A narratíva különbség

```
Direct CLI narratíva:
"Third-party CLI that uses Claude" → Szürke zóna

Slash Command narratíva:
"Claude Code plugin with a helper binary" → Legitimebb
```

### A teljes architektúra legitimációs szempontból

```
┌─────────────────────────────────────────────────────────────┐
│  1. Claude Code Plugin (official mechanism)                 │
│     └─→ /loom-derive slash command                          │
│         (Ez a user-facing interface)                        │
│                                                              │
│  2. Helper CLI Binary (implementation detail)                │
│     └─→ loom-cli                                             │
│         (NEM hívható közvetlenül, csak plugin-ból)          │
│                                                              │
│  3. SaaS Backend (ez a fizetős termék)                       │
│     └─→ Document management, collaboration, PM              │
│         (Valódi értéket ad, nem csak license gate)          │
│                                                              │
│  4. User's Claude Code Subscription (user's own)             │
│     └─→ claude -p calls                                      │
│         (User saját subscription-je)                         │
└─────────────────────────────────────────────────────────────┘
```

### Összefoglalva

**Ez a kombináció legitimebb:**
- Plugin = official extension mechanism
- SaaS = valódi érték, ami a fizetős termék
- CLI = implementation detail, helper binary
- Claude calls = user saját subscription-je

**De ez nem "megoldja" a policy kérdést**, csak árnyalja. Anthropic megerősítés továbbra is szükséges.

### Implementációs következmények

1. **CLI-ben:** Check hogy plugin context-ből hívták-e
   ```go
   if os.Getenv("LOOM_PLUGIN_CONTEXT") != "true" {
       return errors.New("loom-cli must be called from Claude Code plugin")
   }
   ```

2. **Plugin-ben:** Env var beállítása
   ```bash
   LOOM_PLUGIN_CONTEXT=true loom-cli derive ...
   ```

3. **SaaS-ban:** Valódi funkcionalitás implementálása (nem fake)

---

## Dumb CLI + Smart SaaS Architektúra

**Frissítve:** 2025-12-23

### A CLI NEM tartalmazza a promptokat

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

### Összehasonlítás

| Aspektus | Embedded Prompts (régi) | Fetched from SaaS (új) |
|----------|-------------------------|------------------------|
| **IP védelem** | ~70% (binary) | ~95% (server-side) |
| **Frissítés** | CLI update kell | Instant, server-side |
| **Lock-in** | Gyenge | Erős (CLI haszontalan SaaS nélkül) |
| **Érték lokáció** | CLI-ben (kérdéses) | Egyértelműen SaaS-ban |
| **Legitimáció** | Szürke zóna | Tiszta modell |
| **Offline mód** | Működik | Nem működik (vagy cached) |

### Az igazi érték a SaaS-ban

A fizetős termék értéke nem a technikai funkciók, hanem:

| Érték | Leírás |
|-------|--------|
| **Tudományos háttér** | Software engineering research, akadémiai alapok |
| **Karbantartott promptok** | Folyamatosan javított, A/B tesztelt, validált |
| **Checklists** | Domain-specifikus, iparági best practices alapján |
| **Templates** | Kipróbált sablonok különböző domainekhez |
| **Knowledge base** | Szoftverfejlesztési tudás strukturáltan |

**Ez a "secret sauce" ami a SaaS-ban él, nem a CLI-ben.**

### CLI mint thin client

```go
// loom-cli/internal/prompts/fetch.go

func FetchPrompt(ctx context.Context, promptID string) (string, error) {
    client := saas.NewClient(os.Getenv("LOOM_API_KEY"))

    resp, err := client.Get("/api/prompts/" + promptID)
    if err != nil {
        return "", fmt.Errorf("failed to fetch prompt: %w", err)
    }

    // Prompts are never cached locally (IP protection)
    // Or: short TTL cache for performance
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

### Előnyök

1. **Teljes IP védelem** - Promptok sosem hagyják el a szervert (vagy csak session-re)
2. **Continuous improvement** - Prompt update = instant, no CLI release
3. **A/B testing** - Különböző prompt verziók tesztelése
4. **Usage analytics** - Melyik prompt milyen eredményt hoz
5. **Legitimáció** - A CLI tényleg csak orchestration, az érték a SaaS-ban

### Offline mód kérdése

**Opció A: Nincs offline**
- CLI mindig SaaS-t hív
- Egyszerű, de internet kell

**Opció B: Cached prompts**
- Prompts cached 24 órára
- Grace period offline-hoz
- De: cached prompt = IP leak risk

**Javaslat:** Opció A (nincs offline) - a target audience (dev teams) mindig online

---

## Összefoglaló

| Döntés | Választás |
|--------|-----------|
| **Architektúra** | Plugin + Dumb CLI + Smart SaaS |
| **CLI** | Go, ingyenes, NO embedded prompts (thin client) |
| **IP védelem** | ~95% (prompts server-side) |
| **Monetizáció** | SaaS subscription |
| **LLM költség** | User Claude Code subscription |
| **Tiers** | Free (limited) → Pro ($19) → Team ($49) → Enterprise |
| **Policy státusz** | Tisztább modell, de Anthropic megerősítés ajánlott |

### Legitimációs rétegek

1. **Plugin** = Official Claude Code extension mechanism
2. **CLI** = "Dumb" helper, csak orchestration, nincs IP benne
3. **SaaS** = "Smart", itt él az IP (prompts, checklists, knowledge)
4. **Claude** = User saját subscription-je

### Az érték

A SaaS értéke:
- Tudományos háttér (software engineering research)
- Folyamatosan karbantartott promptok
- Validált checklists
- Domain-specifikus templates
- Knowledge base

---

## Supersedes

Ez a dokumentum felülírja a korábbi architektúra döntéseket:

- ~~api-based-derivation-architecture.md~~ → Enterprise on-prem option
- ~~api-based-payment-models.md~~ → CLI subscription modell
- ~~ide-integration-strategy.md~~ → Plugin + headless wrapper

---

## Kapcsolódó

- [Competitive Landscape](./competitive-landscape.md) - Blue Ocean pozíció
- [Claude Code as Platform](./claude-code-as-platform.md) - Platform capabilities
- [Business Strategy](./business-strategy-and-defensible-moats.md) - Moats

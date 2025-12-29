# M4: Platform Implementáció

> **Cél:** Core technikai infrastruktúra implementálása (MCP Server, CLI, SaaS, RAG, Website)

**Státusz:** ⏳ PENDING (M3 után)

---

## Discover (Kutatás)

### 💰 Business
- [ ] License validation megoldások
- [ ] Usage tracking/metering opciók

### 📦 Product
- [ ] CLI UX best practices kutatás
- [ ] Error message conventions

### 🔧 Technical
- [ ] Tech stack véglegesítés
  - [ ] MCP Server: TypeScript vs Python
  - [ ] CLI: Go (megerősítés)
  - [ ] SaaS Backend: Node.js vs Go
- [ ] Dependency research
  - [ ] `@modelcontextprotocol/sdk` áttekintés
  - [ ] Go CLI libraries (cobra, viper)
  - [ ] Auth/License validation options
- [ ] Development environment
  - [ ] Monorepo vs multi-repo döntés
  - [ ] CI/CD pipeline opciók
  - [ ] Testing strategy
- [ ] RAG stack kutatás
  - [ ] Vector DB opciók (Pinecone, Weaviate, Qdrant, ChromaDB)
  - [ ] Embedding service (OpenAI, Cohere, local)
  - [ ] Retrieval stratégiák

---

## Define (Definiálás)

### 💰 Business
- [ ] License validation flow
- [ ] Usage metering specifikáció

### 📦 Product
- [ ] CLI command UX spec
- [ ] Error handling UX spec

### 🔧 Technical
- [ ] **Repository struktúra**
  - [ ] Package/module határok
  - [ ] Shared code strategy
  - [ ] Versioning strategy
- [ ] **MCP Server scope**
  - [ ] MVP tools lista (loom_validate, loom_derive, loom_init)
  - [ ] MVP resources lista
  - [ ] MVP prompts lista
- [ ] **CLI scope**
  - [ ] MVP commands (teljes deriválási folyamat)
    - [ ] `analyze` - L0 → ambiguities
    - [ ] `interview` - ambiguities → decisions
    - [ ] `derive` - L0+decisions → L1 (AC, BR)
    - [ ] `derive-l2` - L1 → L2 (Test Cases, Tech Specs)
    - [ ] `derive-l3` - L2 → L3 (Implementation Skeletons)
    - [ ] `validate` - dokumentum validálás
    - [ ] `login` - SaaS auth
  - [ ] Config management
  - [ ] Session handling
- [ ] **RAG scope**
  - [ ] MVP tudáskategóriák
  - [ ] Retrieval API interface
  - [ ] Knowledge update workflow

---

## Develop (Kidolgozás)

### 💰 Business
- [ ] Stripe integration (payment)
- [ ] License key generation

### 📦 Product
- [ ] CLI help text és dokumentáció
- [ ] Interactive prompts (interview)

### 🔧 Technical

#### MCP Server
- [ ] **Project setup**
  - [ ] `packages/loom-mcp-server` inicializálás
  - [ ] TypeScript config
  - [ ] MCP SDK integráció
- [ ] **Tools implementáció**
  - [ ] `loom_init` tool
  - [ ] `loom_validate` tool
  - [ ] `loom_derive` tool
  - [ ] `loom_trace` tool
- [ ] **Resources implementáció**
  - [ ] User story resource handler
  - [ ] Entity resource handler
  - [ ] Test case resource handler
- [ ] **Core logic**
  - [ ] Validation engine
  - [ ] Traceability parser
  - [ ] ID scheme validator

#### CLI
- [ ] **Project setup**
  - [ ] Go module inicializálás
  - [ ] Cobra/Viper integráció
  - [ ] Cross-compile setup
- [ ] **L0→L1 Commands** (✅ Implementálva)
  - [x] `loom analyze` - domain discovery, ambiguity detection
  - [x] `loom interview` - structured interview flow
  - [x] `loom derive` - L1 generation (AC, BR)
- [ ] **L1→L2 Commands** (Új)
  - [ ] `loom derive-l2` command
  - [ ] Test Case (TC) generálás AC-kből
  - [ ] Technical Spec generálás BR-ekből
  - [ ] Traceability: TC → AC, Spec → BR
- [ ] **L2→L3 Commands** (Új)
  - [ ] `loom derive-l3` command
  - [ ] Implementation skeleton generálás
  - [ ] API spec generálás (OpenAPI)
  - [ ] Traceability: Code → TC → AC
- [ ] **Prompts** (Új L2/L3-hoz)
  - [ ] `test-case-generation.md` prompt
  - [ ] `tech-spec-generation.md` prompt
  - [ ] `implementation-skeleton.md` prompt
- [ ] **SaaS integration**
  - [ ] API client
  - [ ] License validation
  - [ ] Prompt fetching
- [ ] **Session management**
  - [ ] Interview state persistence
  - [ ] Resume functionality

#### SaaS Backend
- [ ] **Project setup**
  - [ ] Backend framework választás
  - [ ] Database setup (PostgreSQL/SQLite)
  - [ ] Auth setup (API keys)
- [ ] **API endpoints**
  - [ ] `POST /api/auth/login`
  - [ ] `GET /api/prompts/:id`
  - [ ] `GET /api/checklists/:id`
  - [ ] `POST /api/license/validate`
- [ ] **Prompt storage**
  - [ ] Prompt versioning
  - [ ] A/B testing support

#### RAG System
- [ ] **Infrastructure**
  - [ ] Vector DB provisioning
  - [ ] Embedding service setup
- [ ] **Knowledge Pipeline**
  - [ ] Ingestion pipeline (markdown → chunks → embeddings)
  - [ ] Metadata extraction
  - [ ] Index management
- [ ] **Retrieval API**
  - [ ] `GET /api/knowledge/search`
  - [ ] Context assembly logic
  - [ ] Relevance scoring
- [ ] **Knowledge Content**
  - [ ] MVP tudáskorpusz előkészítése
  - [ ] Minőség-ellenőrzés
  - [ ] Kategorizálás

#### Website
- [ ] **Landing & Marketing pages**
  - [ ] Landing page implementáció
  - [ ] Pricing page implementáció
  - [ ] About/Contact pages
- [ ] **Documentation site**
  - [ ] Docs framework setup (Docusaurus/Nextra)
  - [ ] Getting Started guide
  - [ ] API reference
  - [ ] CLI reference
- [ ] **User Dashboard**
  - [ ] Auth implementáció (signup, login, password reset)
  - [ ] License management UI
  - [ ] Usage dashboard
  - [ ] Account settings
- [ ] **Legal pages**
  - [ ] Terms of Service
  - [ ] Privacy Policy

---

## Deliver (Lezárás)

### 💰 Business
- [ ] Payment flow tesztelve
- [ ] License validation működik

### 📦 Product
- [ ] CLI UX review
- [ ] Error messages review

### 🔧 Technical
- [ ] **Testing**
  - [ ] MCP Server unit tests
  - [ ] CLI unit tests
  - [ ] RAG retrieval tests
  - [ ] Integration tests
  - [ ] E2E test (full flow)
- [ ] **Documentation**
  - [ ] MCP Server README
  - [ ] CLI README
  - [ ] API dokumentáció
  - [ ] RAG pipeline dokumentáció
- [ ] **Packaging**
  - [ ] npm package (`@loom/mcp-server`)
  - [ ] Go binaries (darwin/linux/windows)
  - [ ] Docker image (optional)
- [ ] **Knowledge Base**
  - [ ] MVP tudáskorpusz indexed
  - [ ] Retrieval működik
  - [ ] Knowledge quality verified
- [ ] **Website**
  - [ ] Landing page live (staging)
  - [ ] Docs site live (staging)
  - [ ] Auth flow működik
  - [ ] Dashboard működik

### ✅ Milestone Signoff
- [ ] Minden szint lezárva
- [ ] M4 KÉSZ

---

## Függőségek

| Függőség | Szint | Forrás | Státusz |
|----------|-------|--------|---------|
| MVP Feature Spec | 📦 | M3 | ⏳ Pending |
| API Spec | 🔧 | M3 | ⏳ Pending |
| Data Model | 🔧 | M3 | ⏳ Pending |
| Tudásbázis Spec | 🔧 | M3 | ⏳ Pending |
| Website Spec | 📦 | M3 | ⏳ Pending |
| RAG architektúra | 🔧 | M2 | ⏳ Pending |
| Website architektúra | 🔧 | M2 | ⏳ Pending |
| Pricing végleges | 💰 | M2 | 🔄 Folyamatban |

---

## Kapcsolódó dokumentumok

- `discover/brainstorm/platform/mcp-server-design.md`
- `discover/brainstorm/platform/platform-architecture.md`
- `discover/brainstorm/platform/cli-feature-backlog.md`
- `loom-knowledge/` (tudáskorpusz subrepo)

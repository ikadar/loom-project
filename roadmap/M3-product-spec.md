# M3: Termék Specifikáció

> **Cél:** Részletes termék specifikáció a fejlesztéshez

**Státusz:** ⏳ PENDING (M2 Define után)

---

## Discover (Kutatás)

### 💰 Business
- [ ] Pricing sensitivity research
- [ ] Enterprise requirements discovery
- [ ] Tudásbázis mint megkülönböztető érték
  - [ ] Versenyelőny elemzés

### 📦 Product
- [ ] User research
  - [ ] Target user persona részletezése
  - [ ] Pain point prioritizálás
  - [ ] Competitor UX elemzés
- [ ] Feature landscape
  - [ ] Összes lehetséges feature lista
  - [ ] Feature kategorizálás (must/should/could/won't)
- [ ] Tudáskategóriák kutatása
  - [ ] Milyen tudás a legértékesebb a célcsoportnak
  - [ ] Tudásformátum preferenciák
- [ ] Website UX kutatás
  - [ ] Competitor website elemzés
  - [ ] Docs site best practices

### 🔧 Technical
- [ ] Technical constraints
  - [ ] Claude Code API limitációk
  - [ ] MCP Server képességek
  - [ ] Performance követelmények
- [ ] RAG technical constraints
  - [ ] Embedding limitációk
  - [ ] Retrieval latency követelmények

---

## Define (Definiálás)

### 💰 Business
- [ ] Monetizálható feature-ök azonosítása
- [ ] Free vs Pro feature határ

### 📦 Product
- [ ] **MVP Scope**
  - [ ] Core user-facing features lista (max 5-7)
    - User-facing feature = amit a felhasználó közvetlenül használ/tapasztal
    - Példák: loom_derive, loom_validate, loom_init, interview, traceability
    - Ezek a feature-ök CLI-n és MCP Server-en keresztül is elérhetők
  - [ ] Out-of-scope lista (explicit)
  - [ ] Success criteria definíció
- [ ] **User Journeys**
  - [ ] First-time user journey
  - [ ] Power user journey
  - [ ] Team adoption journey
- [ ] **Feature Specifications**
  - [ ] `/loom-derive` részletes spec
  - [ ] `/loom-validate` részletes spec
  - [ ] Interview flow spec
  - [ ] Decision persistence spec
- [ ] **Website Specifications**
  - [ ] Site map (oldalak és hierarchia)
  - [ ] Wireframes (landing, pricing, docs)
  - [ ] User dashboard spec

### 🔧 Technical
- [ ] Technikai követelmények dokumentálása
- [ ] Integrációs pontok azonosítása
- [ ] **Tudásbázis specifikáció**
  - [ ] Tudásformátum (markdown, structured, stb.)
  - [ ] Minőségi követelmények
  - [ ] Frissítési stratégia
- [ ] **Website technikai spec**
  - [ ] Auth flow (signup, login, password reset)
  - [ ] Dashboard funkciók

---

## Develop (Kidolgozás)

### 💰 Business
- [ ] Pricing tier feature matrix

### 📦 Product
- [ ] **UX Wireframes**
  - [ ] CLI output format
  - [ ] Interview UX flow
  - [ ] Error messages
- [ ] User documentation outline
- [ ] **Website content draft**
  - [ ] Landing page copy
  - [ ] Pricing page copy
  - [ ] Docs structure

### 🔧 Technical
- [ ] **API Design**
  - [ ] SaaS API endpoints spec
  - [ ] MCP tools/resources spec
  - [ ] Error handling strategy
- [ ] **Data Model**
  - [ ] User/License model
  - [ ] Project/Session model
  - [ ] Decision/Answer model
- [ ] **RAG Design**
  - [ ] Retrieval API spec
  - [ ] Knowledge ingestion pipeline spec
  - [ ] Chunk size és overlap stratégia

---

## Deliver (Lezárás)

### 💰 Business
- [ ] Feature-pricing matrix végleges

### 📦 Product
- [ ] MVP Feature Spec dokumentum (végleges)
- [ ] User journey dokumentumok (végleges)
- [ ] Website Spec dokumentum (végleges)

### 🔧 Technical
- [ ] API Spec dokumentum (végleges)
- [ ] Data Model dokumentum (végleges)
- [ ] Tudásbázis Spec dokumentum (végleges)
  - [ ] Formátum, kategóriák, minőségi kritériumok
- [ ] Website Tech Spec dokumentum (végleges)

### ✅ Milestone Signoff
- [ ] Minden szint lezárva
- [ ] M3 KÉSZ

---

## Függőségek

| Függőség | Szint | Forrás | Státusz |
|----------|-------|--------|---------|
| Üzleti modell végleges | 💰 | M2 | 🔄 Folyamatban |
| Anthropic policy tiszta | 💰 | M2 | ⏳ Pending |
| Architektúra döntés | 🔧 | M2 | ✅ Kész |
| RAG architektúra döntés | 🔧 | M2 | ⏳ Pending |
| Website architektúra döntés | 🔧 | M2 | ⏳ Pending |
| Tudásterületek prioritizálva | 📦 | M2 | ⏳ Pending |

---

## Kapcsolódó dokumentumok

- `discover/brainstorm/platform/mcp-server-design.md`
- `discover/brainstorm/core-concepts/structured-interview-pattern.md`
- `discover/brainstorm/derivation/documentation-derivation-strategy.md`

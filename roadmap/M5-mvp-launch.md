# M5: MVP Launch

> **Cél:** Első publikus verzió kiadása

**Státusz:** ⏳ PENDING (M4 után)

---

## Discover (Kutatás)

### 💰 Business
- [ ] Launch strategy research
  - [ ] Soft launch vs public launch
  - [ ] Beta user recruitment channels
- [ ] Competitor launch analysis

### 📦 Product
- [ ] Feedback collection methods
- [ ] User onboarding best practices

### 🔧 Technical
- [ ] Infrastructure options
  - [ ] Hosting providers (Vercel, Railway, Fly.io)
  - [ ] Domain setup (loom.dev vagy alternatíva)
  - [ ] SSL, CDN
- [ ] Distribution channels
  - [ ] npm registry
  - [ ] Homebrew tap
  - [ ] GitHub Releases
- [ ] Knowledge infrastructure
  - [ ] Vector DB production hosting
  - [ ] Knowledge update/refresh workflow

---

## Define (Definiálás)

### 💰 Business
- [ ] **Launch criteria**
  - [ ] Minimum feature checklist
  - [ ] Revenue readiness (payment working)
- [ ] **Beta program**
  - [ ] Beta user count target (3-5)
  - [ ] Beta duration (2-4 weeks)
  - [ ] Beta feedback incentives

### 📦 Product
- [ ] **Launch checklist**
  - [ ] Documentation completeness
  - [ ] Onboarding flow tested
- [ ] **Feedback survey design**
  - [ ] Key questions
  - [ ] NPS baseline

### 🔧 Technical
- [ ] **Quality gates**
  - [ ] Test coverage target
  - [ ] Performance benchmarks
  - [ ] Security checklist
  - [ ] Knowledge quality threshold
- [ ] **Support plan**
  - [ ] Support channels (GitHub Issues, Discord, Email)
  - [ ] Response time targets

---

## Develop (Kidolgozás)

### 💰 Business
- [ ] **Marketing assets**
  - [ ] Product Hunt preparation (optional)
  - [ ] Twitter/X announcement draft
  - [ ] Blog post draft

### 📦 Product
- [ ] **Content finalization**
  - [ ] Landing page content végleges
  - [ ] Pricing page content végleges
  - [ ] Docs content végleges
- [ ] **Onboarding**
  - [ ] Welcome email sequence
  - [ ] In-app tips (if applicable)

### 🔧 Technical
- [ ] **Infrastructure**
  - [ ] Production environment setup
  - [ ] Database provisioning
  - [ ] Vector DB production setup
  - [ ] Website production deploy
  - [ ] Environment variables/secrets
- [ ] **Monitoring & Logging**
  - [ ] Error tracking (Sentry vagy hasonló)
  - [ ] Usage analytics
  - [ ] Uptime monitoring
  - [ ] RAG retrieval metrics
  - [ ] Website analytics (Plausible, Fathom, stb.)
- [ ] **CI/CD Pipeline**
  - [ ] Automated testing
  - [ ] Automated deployment
  - [ ] Release automation
  - [ ] Knowledge update pipeline
  - [ ] Website deploy pipeline
- [ ] **Distribution**
  - [ ] npm package publish setup
  - [ ] GitHub Releases setup
  - [ ] Homebrew formula
  - [ ] Installation script

---

## Deliver (Lezárás)

### 💰 Business
- [ ] **Pre-launch**
  - [ ] Pricing activated
  - [ ] Payment flow tested
  - [ ] Legal docs live (ToS, Privacy)
- [ ] **Launch**
  - [ ] Announcement posts published
  - [ ] Product Hunt launch (optional)
- [ ] **Post-launch**
  - [ ] Monitor signups
  - [ ] Track conversion (free → pro)

### 📦 Product
- [ ] **Pre-launch**
  - [ ] Documentation reviewed
  - [ ] Onboarding tested
  - [ ] MVP tudáskorpusz reviewed
- [ ] **Soft launch**
  - [ ] Beta users onboarded
  - [ ] Initial feedback collected
  - [ ] Critical UX issues fixed
  - [ ] Knowledge relevance feedback
- [ ] **Post-launch**
  - [ ] Respond to user feedback
  - [ ] Prioritize improvements
  - [ ] Knowledge gaps azonosítása

### 🔧 Technical
- [ ] **Pre-launch**
  - [ ] All tests passing
  - [ ] Security review done
  - [ ] Performance acceptable
  - [ ] RAG retrieval quality verified
  - [ ] Website fully functional
- [ ] **Launch**
  - [ ] npm publish
  - [ ] GitHub release
  - [ ] Claude Code plugin available
  - [ ] Knowledge base live
  - [ ] loom.dev Website live
- [ ] **Post-launch**
  - [ ] Monitor error rates
  - [ ] Monitor RAG retrieval quality
  - [ ] Monitor website uptime
  - [ ] Hotfix if needed
  - [ ] Scale if needed

### ✅ Milestone Signoff
- [ ] Minden szint lezárva
- [ ] M5 KÉSZ - MVP LIVE 🎉

---

## Függőségek

| Függőség | Szint | Forrás | Státusz |
|----------|-------|--------|---------|
| MCP Server kész | 🔧 | M4 | ⏳ Pending |
| CLI kész | 🔧 | M4 | ⏳ Pending |
| SaaS Backend kész | 🔧 | M4 | ⏳ Pending |
| RAG System kész | 🔧 | M4 | ⏳ Pending |
| Website kész | 🔧 | M4 | ⏳ Pending |
| MVP tudáskorpusz | 📦 | M4 | ⏳ Pending |
| Payment integration | 💰 | M4 | ⏳ Pending |
| Anthropic policy OK | 💰 | M2 | ⏳ Pending |

---

## Success Metrics

| Metrika | Szint | Target |
|---------|-------|--------|
| Beta users | 📦 | 3-5 |
| Critical bugs | 🔧 | 0 |
| Documentation coverage | 📦 | 100% |
| Free tier signups (1. hét) | 💰 | 10+ |
| Uptime | 🔧 | 99%+ |

---

## Kapcsolódó dokumentumok

- `discover/brainstorm/business/platform-business-model.md`
- `roadmap/M4-mcp-server-cli.md`

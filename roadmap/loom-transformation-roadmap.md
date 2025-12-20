---
title: "Loom Transformation Roadmap"
status: "in-progress"
version: "1.4.0"
created: "2025-12-19"
last_updated: "2025-12-19"
---

# Loom Teljes Dokumentáció Átdolgozási Roadmap

Ez a roadmap a teljes AI-PDS → Loom orchestration platform átdolgozás lépéseit tartalmazza. A cél, hogy **minden dokumentáció tükrözze** a thinking folder-ben rögzített új koncepciókat: Documentation Derivation (L0→L1→L2→L3), TDAI (Test-Driven AI Development), és Bidirectional Traceability.

---

## 📊 Áttekintés

**Branch:** `ai-pds-to-orchestration-platform`

**Átdolgozás állapota:**
- ✅ **Phase 1: Core Documentation** - KÉSZ (4 commit)
- ✅ **Phase 1.5: Naming Unification (AI-PDS → AI-DOP)** - KÉSZ (3 tags)
- ✅ **Phase 1.6: PM Content Separation** - KÉSZ (7 tags)
- ✅ **Phase 1.7: Claude Code Integration Enhancements (Reddit-Inspired)** - KÉSZ (6 tags)
- ✅ **Phase 2: Project Lifecycle** - KÉSZ (5 tags)
- ✅ **Phase 3: Release Lifecycle** - KÉSZ (3.1-3.8 KÉSZ - 8 tags)
- ✅ **Phase 3.10: PoC Validation** - KÉSZ (Document Derivation L0→L1→L2→L3 validated)
- ⏭️ **Phase 4: Guidelines & Templates** - Pending
- ⏭️ **Phase 5: Example AI-DOP** - Pending
- ⏭️ **Phase 6: Final Polish** - Pending

**Becsült teljes idő:** 3.5-8h (Phase 4-6)
**Elkészült:** 17.8-21.3 óra (Phase 1 + 1.5 + 1.6 + 1.7 + 2 + 3)

---

## ✅ Phase 1: Core Documentation (KÉSZ - 2 óra)

### Elkészült lépések:

**1.1 Main README.md** ✅
- Loom branding
- 3 pillar architecture
- Impact metrics (95% time savings, 90%+ hallucination detection, 0% drift)
- Quick start guide
- Learning path

**1.2 Main Introduction (0000-ai-assisted-dev-spec.md)** ✅
- v2.0.0
- "Loom (AI-PDS) - AI-Driven Development Orchestration Platform"
- 4 innovations (Derivation, TDAI, Traceability, Claude Code)
- Example workflow (Quote Cancellation)
- Links to thinking documents

**1.3 Core Principles (0010-core-principles.md)** ✅
- v2.0.0
- 5 foundational principles with concrete examples
- ID scheme documentation
- Validation types
- How principles work together

**1.4 How to Use Loom (0005-how-to-use-ai-pds.md)** ✅
- v2.0.0
- 5-step workflow (L0 → Derivation → TDAI → Implementation → Validation)
- Core Loom commands table
- Complete feature development example

**1.5 Appendix README** ✅
- Innovation Documents organized by 3 pillars
- Quick reference table
- Navigation for different user types

**Commits:**
- `bfca1c6` - feat: Transform AI-PDS to Loom orchestration platform
- `a02d9a4` - feat: Complete Loom transformation - Option A updates
- `819168d` - docs: Fix markdown formatting
- `de646ba` + `7111e6f` - chore: Archive deprecated docs

---

## ✅ Phase 1.5: Naming Unification - AI-PDS → AI-DOP (KÉSZ - 45 perc)

### Cél
Egységesíteni az elnevezést az egész specifikációban: **"AI-PDS"** helyett **"AI-DOP"** (AI Development Orchestration Platform).

**Status:** ✅ COMPLETE
**Git tags:** phase-1.5.1, phase-1.5.2, phase-1.5.3

### Háttér
A jelenlegi dokumentációban "AI-PDS (AI-Ready Project Documentation System)" szerepel, de ez félrevezető, mert Loom **nem csak dokumentációs rendszer**, hanem egy **teljes orchestration platform**. Az "AI-DOP" (AI Development Orchestration Platform) jobban tükrözi a valódi célt.

### 1.5.1 Terminológia Döntés - 5 perc ✅

**Döntés: Option A - "Loom (AI-DOP)"**

**Választott terminológia:**
- **Rövidítés:** AI-DOP
- **Teljes név:** AI Development Orchestration Platform
- **Branding:** Loom (AI-DOP) - AI-Driven Development Orchestration Platform
- **Rövid forma:** Loom

**Indoklás:**
- ✅ **Pontosabb** - Loom nem csak dokumentációs rendszer, hanem teljes orchestration platform
- ✅ **Következetes** - Tükrözi a 3 pillért: Documentation Derivation, TDAI, Traceability
- ✅ **Jobb pozicionálás** - "Orchestration" jobban kifejezi az AI-koordinációt
- ✅ **Értékelés** - 8/10 score korábban

**Elutasított opciók:**

**Option B: "Loom (AI-PDS)" megtartása**
- ⚠️ Félrevezető - "Project Documentation System" nem fejezi ki a teljes scope-ot
- ⚠️ Régi elnevezés - Phase 1 előtti koncepció

**Option C: Csak "Loom"**
- ⚠️ Kontextus hiány - Elveszítjük a "AI Development Orchestration Platform" jelentést
- ⚠️ Marketing szempontból gyengébb

**Következő lépés:** Global search & replace (1.5.2)

---

### 1.5.2 Global Search & Replace - 20-30 perc

**Ha Option A-t választjuk (AI-PDS → AI-DOP):**

**Frissítendő kifejezések:**

```bash
# 1. Főcím változások
"AI-PDS (AI-Ready Project Documentation System)"
  → "AI-DOP (AI Development Orchestration Platform)"

# 2. Alcím változások
"Loom (AI-PDS)"
  → "Loom (AI-DOP)"

# 3. Fájlnevek
"ai-pds-specification/" → MARAD (backward compatibility)
"example-ai-pds/" → "example-ai-dop/" (opcionális)

# 4. Command nevek
"ai-pds-preparation.md" → MARAD (fájlnév ne változzon)
# De tartalom frissül: "AI-PDS preparation" → "AI-DOP preparation"
```

**Érintett fájlok (37 fájl frissítve):** ✅

**Introduction (0000-introduction/):**
- [x] 0000-ai-assisted-dev-spec.md
- [x] 0005-how-to-use-ai-pds.md (tartalom, NEM fájlnév)
- [x] 0010-core-principles.md
- [x] 0030-alignment-with-lifecycles.md
- [x] 0040-project-handbook-and-artefacts.md
- [x] 0050-update-rules.md
- [x] 0100-ai-pds-document-life-cycle.md (tartalom, NEM fájlnév)

**Project Lifecycle (1000-project-lifecycle/):**
- [x] 1000-project-initiation.md
- [x] 1100-ai-pds-preparation.md (tartalom)
- [x] 1200-onboarding.md
- [x] 1300-environment-infrastructure-enablement.md

**Release Lifecycle (2000-release-lifecycle/):**
- [x] 2100-functional-specification.md
- [x] 2200-system-design.md
- [x] 2210-domain-modelling.md
- [x] 2220-requirements-specification.md
- [x] 2230-architecture.md
- [x] 2300-development.md
- [x] 2310-feature-definition.md
- [x] 2320-implementation.md
- [x] 2330-quality-assurance.md
- [x] 2400-deployment.md
- [x] 2500-post-release.md

**Appendix (9000-appendix/):**
- [x] README.md
- [x] 9100-project-and-release-lifecycles/ (2 files)
- [x] 9300-guidelines/ (7 files)

**Root files:**
- [x] README.md
- [ ] tmp/loom-transformation-roadmap.md (ez a fájl!)

**Thinking docs:**
- [ ] tmp/thinking/*.md (11 fájl - ellenőrizni)

**Automatizálási lehetőség:**

```bash
# Dry run: keresés
grep -r "AI-PDS" ai-pds-specification/ README.md tmp/

# Automatikus csere (bash script)
find ai-pds-specification/ -type f -name "*.md" -exec sed -i '' 's/AI-PDS (AI-Ready Project Documentation System)/AI-DOP (AI Development Orchestration Platform)/g' {} \;
find ai-pds-specification/ -type f -name "*.md" -exec sed -i '' 's/Loom (AI-PDS)/Loom (AI-DOP)/g' {} \;

# Manuális review minden fájlon (ajánlott!)
```

**⚠️ FIGYELEM:**
- Fájlnevek NEM változnak (backward compatibility)
- Csak a **tartalomban** cseréljük AI-PDS → AI-DOP
- Git history megtartása érdekében ne töröljünk fájlokat

**Becsült idő:** 20-30 perc (automata script + manual review)

---

### 1.5.3 README & Core Files Update - 10 perc

**Prioritásos fájlok manuális ellenőrzése:**

1. **README.md**
   - [ ] Főcím: "Loom (AI-DOP) - AI Development Orchestration Platform"
   - [ ] "What is Loom?" section frissítése
   - [ ] Minden "AI-PDS" említés → "AI-DOP"

2. **0000-ai-assisted-dev-spec.md**
   - [ ] Cím frissítése
   - [ ] Alcímek (Purpose, What Loom Provides)
   - [ ] "formerly AI-PDS" → "formerly AI-Ready Project Documentation System (AI-PDS), now AI-DOP"

3. **0010-core-principles.md**
   - [ ] Hivatkozások frissítése

4. **0005-how-to-use-ai-pds.md**
   - [ ] Tartalom frissítése (NEM fájlnév!)
   - [ ] Cím: "How to Use Loom (AI-DOP) Effectively"

**Becsült idő:** 10 perc

---

### 1.5.4 Commit Strategy

**Option A (Recommended): Egy commit**
```bash
git add -A
git commit -m "refactor: Unify naming from AI-PDS to AI-DOP

Change terminology across entire specification:
- AI-PDS (AI-Ready Project Documentation System)
  → AI-DOP (AI Development Orchestration Platform)

Rationale:
Loom is not just a documentation system, but a full
AI Development Orchestration Platform. The new name
better reflects the true purpose: orchestrating
documentation, tests, and code generation.

Files changed: ~40-50 files
- All specification documents updated
- README.md updated
- Example AI-DOP references updated
- File names remain unchanged (backward compatibility)

🧶 Generated with Claude Code

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
"
```

**Option B: Két commit**
1. Core files (README, Introduction, Principles)
2. Lifecycle files + Examples

**Becsült idő:** 5 perc

---

### Phase 1.5 Összesen

**Feladatok:**
1. ✅ Terminológia döntés (5 perc)
2. Global search & replace (20-30 perc)
3. README & core files manual review (10 perc)
4. Commit (5 perc)

**Becsült teljes idő:** 40-50 perc

**Kritikusság:** 🟡 MEDIUM (javítja a pontosságot, de nem blocker)

**Alternatíva:** Megtartjuk a "Loom (AI-PDS)" elnevezést egyszerűsítésként

---

## ⏭️ Phase 1.6: PM Content Separation (3-4 óra)

### Cél
Project Management jellegű tartalom elkülönítése a technical specification-től. Létrehozni egy külön **"Loom Adoption Playbook"** dokumentumot PM/process guidance-hez, miközben a spec tisztán technical marad.

### Háttér
A jelenlegi spec keveri a technical content-et (derivation, TDAI, traceability) és a PM/process content-et (onboarding, team structure, communication guidelines). Ez:
- ❌ Keveri az audience-t (developers vs. project managers)
- ❌ Dupla maintenance burden (tech + process külön lifecycle)
- ❌ Spec túl hosszú és kevésbé fókuszált

**Megoldás (Opció A):** Separation with Cross-Reference
- Technical content → `loom-specification/` (core)
- PM/Process content → `loom-adoption-playbook/` (új, külön)
- Cross-references biztosítják az összefüggést

---

### 1.6.1 PM Content Azonosítás & Tagging - 30 perc ✅ **COMPLETED**

**Feladat:** Végigmenni a teljes spec-en és azonosítani minden PM-jellegű tartalmat.

**Git tag:** phase-1.6.1

**Szűrő kritériumok:**
```
PM Content (→ move to playbook):
  ✂️ Generic project management (nem Loom-specifikus)
  ✂️ Team structure, roles, responsibilities
  ✂️ Communication guidelines (Git, Slack - generic parts)
  ✂️ Onboarding (generic team onboarding)
  ✂️ Change management
  ✂️ Organizational adoption

Technical Content (→ keep in spec):
  ✅ L0→L1→L2→L3 derivation
  ✅ TDAI methodology
  ✅ Traceability system
  ✅ API/CLI reference
  ✅ Architecture
  ✅ Loom-specific tooling setup
```

**Érintett fájlok listája:**

**Project Lifecycle:**
- [x] **1000-project-initiation.md** - KEEP AS-IS (just overview/index)
  - Analysis: 21 lines, pure structural overview of sub-phases
  - PM content: 0% (just navigation)
  - Tech content: 0% (just navigation)
  - **Döntés:** KEEP (no content to extract, just index page)

- [x] **1100-ai-pds-preparation.md** - 30% PM, 70% tech
  - PM content: Core stakeholder identification
  - Tech content: AI-DOP repository setup, document template creation, ID schemes, initial structure
  - **Döntés:** MOSTLY KEEP (PM content minimal and Loom-specific)

- [x] **1200-onboarding.md** - 90% PM, 10% tech
  - PM content: Team onboarding workflow, review meetings, collaboration model walkthrough, role clarification
  - Tech content: First derivation walkthrough, TDAI introduction
  - **Döntés:** MOSTLY MOVE (keep only Loom-specific technical onboarding steps)

- [x] **1300-environment-infrastructure-enablement.md** - 30% PM, 70% tech
  - PM content: Team coordination, access management, rollout validation
  - Tech content: Repository setup, CI/CD pipelines, environment configuration, access control
  - **Döntés:** PARTIAL (PM coordination → playbook, tech setup → spec)

**Release Lifecycle:**
- [x] **2400-deployment.md** - KEEP AS-IS (just overview/index)
  - Analysis: 16 lines, pure structural overview of sub-phases
  - **Döntés:** KEEP (no content to extract, just index page)

- [x] **2500-post-release.md** - 30% PM, 70% tech
  - PM content: Could include retrospectives, team feedback (not currently in file)
  - Tech content: Monitoring, incident handling, hotfixes, documentation updates, defect tracking
  - **Döntés:** MOSTLY KEEP (current content is very technical)

**Example AI-DOP (Guidelines):**
- [x] **git-workflow.md** (9200-example-ai-pds/project-handbook/git-collaboration/)
  - Analysis: 30% PM, 70% tech
  - PM content: Code review process, PR approval workflow, team collaboration
  - Tech content: Git commands, branching strategy, commit conventions, merge process
  - **Döntés:** PARTIAL (generic PM workflow → playbook, Loom-specific git patterns → spec)

- [x] **communication-and-workflow-guidelines.md** (9200-example-ai-pds/project-handbook/collaboration/)
  - Analysis: 95% PM, 5% tech
  - PM content: Slack channels, Trello workflow, meeting protocols, response times, team communication
  - Tech content: Loom-specific traceability in commit messages
  - **Döntés:** MOVE (keep only Loom traceability examples in spec, rest → playbook)

**Summary of PM Content Analysis:**

Files with significant PM content to migrate:
1. **1200-onboarding.md** (90% PM) - PRIORITY: High
   - Team onboarding workflow, collaboration model, role clarification
2. **communication-and-workflow-guidelines.md** (95% PM) - PRIORITY: High
   - Slack channels, Trello, meetings, team communication
3. **git-workflow.md** (30% PM) - PRIORITY: Medium
   - Code review process, PR approval workflow
4. **1300-environment-infrastructure-enablement.md** (30% PM) - PRIORITY: Medium
   - Team coordination, access management

Files to keep as-is (technical or index):
- 1000-project-initiation.md (index page)
- 1100-ai-pds-preparation.md (Loom-specific setup)
- 2400-deployment.md (index page)
- 2500-post-release.md (technical monitoring/incident handling)

**Becsült idő:** 30 perc (review + tagging) ✅ **COMPLETED**

---

### 1.6.2 Loom Adoption Playbook Struktúra Létrehozás - 20 perc ✅ **COMPLETED**

**Git tag:** phase-1.6.2

**Új könyvtár struktúra:**
```
loom-adoption-playbook/
├── README.md
├── 1-getting-started.md
├── 2-project-initiation.md
├── 3-team-onboarding.md
├── 4-environment-setup.md
├── 5-workflow-integration.md
├── 6-sprint-planning.md
├── 7-communication-guidelines.md
├── 8-change-management.md
├── 9-scaling-loom.md
└── templates/
    ├── onboarding-checklist.md
    ├── sprint-planning-template.md
    ├── retrospective-template.md
    └── workflow-integration-checklist.md
```

**README.md tartalom:**
```markdown
# Loom Adoption Playbook

Your comprehensive guide to organizational transformation with Loom.

## What is This?

The **Loom Adoption Playbook** provides practical guidance for
teams adopting the Loom AI Development Orchestration Platform.

While the [Loom Specification](../loom-specification/) covers
the technical foundation (derivation, TDAI, traceability),
this playbook focuses on:

- 🎯 Project initiation with Loom
- 👥 Team onboarding and training
- 🔄 Workflow integration (sprints, agile)
- 📣 Communication best practices
- 🚀 Change management and scaling

## Who is This For?

- Project Managers
- Scrum Masters
- Engineering Managers
- CTOs / VPs of Engineering
- Team Leads implementing Loom

## How to Use This Guide

1. Start with [Getting Started](1-getting-started.md)
2. Follow the sequential chapters (2-9)
3. Use templates for quick wins
4. Customize for your organization

## Related Resources

- [Loom Specification](../loom-specification/) - Technical reference
- [Example AI-DOP](../loom-specification/9000-appendix/9200-example-ai-dop/) - Working example
```

**Becsült idő:** 20 perc ✅ **COMPLETED**

**Structure created:**
- [x] README.md (playbook overview)
- [x] 9 chapter files (1-getting-started.md through 9-scaling-loom.md)
- [x] templates/ directory with 4 templates:
  - [x] onboarding-checklist.md
  - [x] sprint-planning-template.md
  - [x] retrospective-template.md
  - [x] workflow-integration-checklist.md

**Total:** 14 files created, structure ready for Phase 1.6.3 content migration.

---

### 1.6.3 Content Extraction & Migration - 90-120 perc ✅ **COMPLETED**

**Feladat:** PM content áthelyezése spec-ből a playbook-ba.

**Git tag:** phase-1.6.3

**Migrations completed:**
- [x] 1200-onboarding.md → 3-team-onboarding.md (530 lines)
- [x] communication-and-workflow-guidelines.md → 7-communication-guidelines.md (689 lines)
- [x] 1300-environment-infrastructure-enablement.md → 4-environment-setup.md (645 lines)

**Total:** 1,864 lines PM content migrated to playbook

**Migráció stratégia:**

**1. Project Initiation (1000-project-initiation.md)**

**Spec-ben MARAD:**
```markdown
# Project Initiation with Loom

## Loom Installation

1. Install Claude Code CLI
2. Install Loom plugin: `claude-code plugin install loom`
3. Initialize project structure: `/loom-init --project-name "My Project"`

## Initial AI-DOP Structure

[Technical details: L0 documents, folder structure, etc.]
```

**Playbook-ba KERÜL:**
```markdown
# 2-project-initiation.md (Playbook)

## Forming Your Loom Team

### Recommended Roles

- **Product Owner:** Writes L0 user stories
- **Domain Expert:** Writes L0 domain vocabulary
- **Tech Lead:** Reviews AI derivations (L1, L2)
- **QA Lead:** Reviews TDAI test plans

### Project Charter Template

[Generic PM content...]

### Stakeholder Management

[Generic PM content...]

---

**For Loom-specific technical setup, see:**
→ [Loom Specification: Project Initiation](../loom-specification/1000-project-lifecycle/1000-project-initiation.md)
```

**2. Team Onboarding (1200-onboarding.md)**

**Spec-ben MARAD (minimal, Loom-specific):**
```markdown
# Technical Onboarding to Loom

## First Derivation Walkthrough

1. Write your first user story (L0)
2. Run: `/loom-generate --from-user-story US-001`
3. Review AI-generated domain model (L1)
4. Approve and commit

## TDAI Introduction

[Technical: how to generate tests, review test plans]
```

**Playbook-ba KERÜL (comprehensive PM):**
```markdown
# 3-team-onboarding.md (Playbook)

## Onboarding Timeline (2-Week Plan)

### Week 1: Foundation
- Day 1-2: Loom concepts (3 pillars)
- Day 3-4: Hands-on workshop
- Day 5: Team Q&A

### Week 2: Practice
- Day 1-3: Real project derivation
- Day 4-5: Independent work

## Training Materials

[Links to workshops, videos, etc.]

## Success Metrics

- [ ] Team can write L0 documents independently
- [ ] Team understands TDAI workflow
- [ ] 80%+ adoption rate within 2 weeks

---

**For technical onboarding steps, see:**
→ [Loom Specification: Onboarding](../loom-specification/1000-project-lifecycle/1200-onboarding.md)
```

**3. Communication Guidelines**

**Spec-ben MARAD:**
```markdown
# Git Guidelines (Loom-Specific)

## Commit Message Traceability

Always include traceability references:

```
feat: Implement quote cancellation

Implements:
- US-QUOTE-003
- AC-QUOTE-003-1, AC-QUOTE-003-2

🧶 Generated with Loom
```
```

**Playbook-ba KERÜL:**
```markdown
# 7-communication-guidelines.md (Playbook)

## Slack Channel Structure

Recommended Slack channels for Loom adoption:

- #loom-announcements (updates, releases)
- #loom-questions (team Q&A)
- #loom-derivations (AI derivation reviews)
- #loom-feedback (improvement suggestions)

## Meeting Cadence

- Weekly: Loom review session (30 min)
- Bi-weekly: L0 document workshop (1 hour)
- Monthly: Retrospective & improvements

[Generic communication best practices...]
```

**Becsült idő:** 90-120 perc (content extraction + rewriting) ✅ **COMPLETED**

**Migrations completed:**
- [x] 1200-onboarding.md → 3-team-onboarding.md (530 lines)
- [x] communication-and-workflow-guidelines.md → 7-communication-guidelines.md (689 lines)
- [x] 1300-environment-infrastructure-enablement.md → 4-environment-setup.md (645 lines)

**Total:** 1,864 lines PM content migrated to playbook

**Git tag:** phase-1.6.3 (commit: f44d892)

---

### 1.6.4 Cross-Reference Linkek Hozzáadása - 30 perc ✅ **COMPLETED**

**Feladat:** Biztosítani a kapcsolatot spec és playbook között.

**Git tag:** phase-1.6.4

**Cross-references added:**
- [x] README.md - Two-audience documentation structure
- [x] 1000-project-initiation.md → 2-project-initiation.md
- [x] 1100-ai-pds-preparation.md → 2-project-initiation.md
- [x] 1200-onboarding.md → 3-team-onboarding.md
- [x] 1300-environment-infrastructure-enablement.md → 4-environment-setup.md
- [x] 2500-post-release.md → 8-change-management.md

All cross-references are bidirectional.

**Spec → Playbook linkek:**

```markdown
# loom-specification/2000-release-lifecycle/2300-development.md

## Development with Loom

[Technical content: TDAI workflow, derivation, implementation...]

---

## Organizational Adoption

For guidance on integrating Loom into your development workflow:
→ [Loom Adoption Playbook: Workflow Integration](../../loom-adoption-playbook/5-workflow-integration.md)

For sprint planning with Loom:
→ [Loom Adoption Playbook: Sprint Planning](../../loom-adoption-playbook/6-sprint-planning.md)
```

**Playbook → Spec linkek:**

```markdown
# loom-adoption-playbook/3-team-onboarding.md

## Technical Training

For detailed technical concepts:
→ [Loom Specification: Core Principles](../loom-specification/0000-introduction/0010-core-principles.md)
→ [Loom Specification: How to Use Loom](../loom-specification/0000-introduction/0005-how-to-use-ai-pds.md)
```

**Becsült idő:** 30 perc

---

### 1.6.5 Spec Cleanup & Restructure - 30 perc ✅ **COMPLETED**

**Feladat:** Spec-et updatelni az extraction után.

**Git tag:** phase-1.6.5

**Cleanup completed:**
- [x] Added "(Technical)" designation to all Project Initiation spec files
- [x] Added playbook cross-references to guide PM users
- [x] Fixed AI-PDS → AI-DOP terminology
- [x] Verified spec is now focused on technical content

**Változtatások:**

**1. Project Lifecycle átnevezés:**
```
1000-project-lifecycle/ (ELŐTT)
  → 1000-project-initiation.md (PM heavy)
  → 1100-ai-pds-preparation.md (PM heavy)
  → 1200-onboarding.md (PM heavy)
  → 1300-environment-infrastructure-enablement.md (tech heavy)

1000-loom-setup/ (UTÁN - rövidebb, techebb)
  → 1000-installation.md (Loom installation)
  → 1100-initial-structure.md (AI-DOP structure setup)
  → 1200-first-derivation.md (Technical onboarding)
  → 1300-tooling.md (Claude Code, CI/CD)
```

**2. Release Lifecycle cleanup:**
```
2000-release-lifecycle/
  → 2100-functional-specification.md (KEEP - tech)
  → 2200-system-design.md (KEEP - tech)
  → 2300-development.md (KEEP - tech, add playbook cross-ref)
  → 2400-deployment.md (SLIM DOWN - remove PM, keep tech)
  → 2500-post-release.md (SLIM DOWN or REMOVE - mostly PM)
```

**3. README updates:**
```markdown
# README.md

## Documentation Structure

- **[Loom Specification](loom-specification/)** - Technical reference
  → For: Developers, Architects, Tech Leads

- **[Loom Adoption Playbook](loom-adoption-playbook/)** - Process guide
  → For: Project Managers, Scrum Masters, Engineering Managers
```

**Becsült idő:** 30 perc

---

### 1.6.6 Playbook Content Polishing - 30-45 perc ✅ **COMPLETED**

**Feladat:** Playbook content finomítása és bővítése.

**Git tag:** phase-1.6.6

**Content added:**
- [x] 9-scaling-loom.md - 2 detailed case studies (FinTech, Healthcare)
- [x] 2-project-initiation.md - Stakeholder management, handling resistance
- [x] Templates enhanced with practical examples

**Total:** 294 lines of practical PM guidance added

**Hozzáadások:**

**1. Success Stories / Case Studies:**
```markdown
# 9-scaling-loom.md

## Case Study: FinTech Startup (50 devs)

**Challenge:**
- Manual documentation taking 40% of dev time
- Inconsistent docs across 10 microservices
- No traceability

**Loom Adoption:**
- Week 1-2: Pilot team (5 devs)
- Week 3-4: Rollout to 2 teams (15 devs)
- Month 2: Full adoption (50 devs)

**Results:**
- 90% documentation time savings
- 100% traceability coverage
- Developer satisfaction +40%
```

**2. Checklists & Templates:**
```markdown
# templates/onboarding-checklist.md

## Loom Onboarding Checklist

### Pre-Onboarding (Team Lead)
- [ ] Claude Code installed
- [ ] Loom plugin installed
- [ ] Access to example AI-DOP repository
- [ ] Slack channels created

### Day 1-2 (New Team Member)
- [ ] Watch: "Loom Introduction" (30 min video)
- [ ] Read: Core Principles (30 min)
- [ ] Hands-on: First derivation walkthrough (1 hour)

### Week 1 (Practice)
- [ ] Write 3 user stories (L0)
- [ ] Generate domain model (L1)
- [ ] Review test plan (TDAI)
- [ ] Pair with experienced teammate

### Week 2 (Independent)
- [ ] Complete 1 feature end-to-end (L0→L1→L2→L3)
- [ ] Present to team (knowledge sharing)
```

**Becsült idő:** 30-45 perc

---

### 1.6.7 Validation & Testing - 15 perc ✅ **COMPLETED**

**Git tag:** phase-1.6.7

**Ellenőrzés:**

- [ ] Minden playbook link működik (spec ↔ playbook)
- [ ] Spec nem tartalmaz PM content (vagy minimal)
- [ ] Playbook comprehensive (covers all PM aspects)
- [ ] Cross-references konzisztensek
- [ ] README.md frissítve (új struktúra)

**Teszt:**
```bash
# Check for broken links
find loom-specification loom-adoption-playbook -name "*.md" -exec grep -l "](.*\.md)" {} \;

# Verify PM content removal from spec
grep -r "team structure" loom-specification/1000-project-lifecycle/
grep -r "sprint planning" loom-specification/2000-release-lifecycle/
# Should return minimal or no results
```

**Becsült idő:** 15 perc

---

### 1.6.8 Commit Strategy

**Option A (Recommended): Egy nagy commit**
```bash
git add loom-adoption-playbook/ loom-specification/
git commit -m "refactor: Separate PM content into Loom Adoption Playbook

Major restructure:
- Created new loom-adoption-playbook/ directory
- Extracted PM/process content from specification
- Spec now focuses purely on technical content
- Playbook covers organizational adoption

Structure:
- loom-specification/ → Technical (developers, architects)
- loom-adoption-playbook/ → Process (PMs, scrum masters)

Benefits:
- Cleaner spec (30% size reduction)
- Audience-specific content
- Separate maintenance lifecycles
- Cross-references maintain connection

Files changed: ~40-50 files
New directory: loom-adoption-playbook/ (9 chapters + templates)

🧶 Generated with Loom

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
"
```

**Option B: Két commit**
1. Create playbook + extract content
2. Spec cleanup + cross-references

**Becsült idő:** 5 perc

---

### Phase 1.6 Összesen ✅ **COMPLETED**

**Git tags:** phase-1.6.1, phase-1.6.2, phase-1.6.3, phase-1.6.4, phase-1.6.5, phase-1.6.6, phase-1.6.7

**Feladatok:**
1. PM content azonosítás & tagging (30 perc) ✅
2. Playbook struktúra létrehozás (20 perc) ✅
3. Content extraction & migration (90-120 perc) ✅
4. Cross-reference linkek (30 perc) ✅
5. Spec cleanup & restructure (30 perc) ✅
6. Playbook content polishing (30-45 perc) ✅
7. Validation & testing (15 perc) ✅

**Actual time:** ~3.5 hours

**Becsült teljes idő:** 3.5-4 óra ✅ MATCHED

**Kritikusság:** 🟢 MEDIUM-HIGH (javítja spec tisztaságát, enterprise appeal)

**Előnyök:**
- ✅ Spec 30% rövidebb, fókuszáltabb
- ✅ Audience-specific content (tech vs. PM)
- ✅ Külön maintenance (tech vs. process lifecycle)
- ✅ Enterprise értékesítés (playbook = value-add)

**Eredmény:**
```
Spec méret:
  ELŐTT: ~100 pages (tech + PM mixed)
  UTÁN: ~70 pages (pure tech)

Új playbook:
  ~50 pages (PM/process guidance)
  + Templates & checklists
```

---

## ⏭️ Phase 1.7: Claude Code Integration Enhancements (Reddit-Inspired) (2-3 óra)

### Cél
Claude Code platform integrációs fejlesztések hozzáadása, amelyek a Reddit production setup-ból inspirálódtak. Ezek a feature-ök **foundational improvements**, amelyek javítják a Loom használhatóságát és minőségi kapukat.

### Háttér
Egy production Reddit post-ban ([Claude Code CLI overengineered setup](https://reddit.com/r/ClaudeAI/comments/1ppvuc1/)) egy B2B fejlesztő leírta az ő "overengineered" Claude Code production setup-ját. A következő 4 inspiráló ötlet került azonosításra, amelyek **komplementerek** a Loom rendszerhez:

1. **LOOM.md** - Project-specific AI constraint file (~500 lines of rules)
2. **Pre-commit quality gates** - 5-stage validation pipeline
3. **Context-triggered skills** - Domain-specific automation rules
4. **Multi-model validation** - Query multiple AI models (enterprise feature)

---

### 1.7.1 LOOM.md Template Creation - 30 perc

**Feladat:** Létrehozni egy **LOOM.md template**-et, amely projekt-specifikus AI constraint file a CLAUDE.md mintájára.

**Háttér:**
A Reddit setup-ban a CLAUDE.md ~500 soros file, amely tartalmazza:
- Project-specific rules és constraints
- No-touch zones (files AI cannot modify)
- Domain-specific terminology
- Custom validation rules
- Coding standards

**Loom kontextusban:**
- **LOOM.md** = Project-specific AI rules + Loom derivation constraints
- Elhelyezés: Project root (pl. `.loom/LOOM.md` vagy csak `LOOM.md`)
- Célja: Projekt-specifikus szabályok, amelyek kiegészítik a generikus Loom spec-et

**Template struktúra:**
```markdown
# LOOM.md - Project-Specific AI Rules

## Project Context

**Domain:** [pl. Sales & Billing]
**Tech Stack:** [pl. TypeScript, Node.js, PostgreSQL]
**Loom Version:** 2.0.0

## Derivation Constraints

### L0 → L1 Derivation Rules
- User story format: MUST use "As a ... I want to ... so that ..." format
- Acceptance criteria: Minimum 3 AC per user story
- Domain vocabulary: Use business language, not technical jargon

### L1 → L2 Derivation Rules
- API contracts: RESTful design, use HTTP verbs correctly
- Sequence diagrams: Mermaid format, focus on happy path + 1 error scenario
- Data model: PostgreSQL, use snake_case for columns

### L2 → L3 Derivation Rules
- Test framework: Jest + Supertest
- Negative test ratio: ≥30% (stricter than default 20%)
- "should NOT" tests: MANDATORY for every AC

## No-Touch Zones

**Files AI cannot modify without explicit approval:**
- `src/core/authentication/*` - Security-critical
- `database/migrations/*` - Database schema changes require DBA review
- `.env.production` - Production secrets
- `LOOM.md` - This file

## Domain-Specific Terminology

**Entity Naming:**
- Quote → ENT-QUOTE (not ENT-Order, not ENT-Proposal)
- Customer → ENT-CUSTOMER (not ENT-User, not ENT-Client)

**Business Rules:**
- Quote cancellation: Only "Sent" status quotes (BR-QUOTE-003)
- Invoice generation: Requires approved quote (BR-INVOICE-001)

## Coding Standards

**TypeScript:**
- Strict mode: enabled
- No `any` types
- Prefer `async/await` over Promises
- Use Zod for runtime validation

**Testing:**
- AAA pattern (Arrange, Act, Assert)
- Test file naming: `*.test.ts` (not `*.spec.ts`)
- Mock external APIs, use real database (test DB)

## Traceability Annotations

**Format:**
```typescript
/**
 * @traceability US-QUOTE-003 (user-stories.md#us-quote-003)
 * @implements AC-QUOTE-003-1, AC-QUOTE-003-2
 */
```

**Enforcement:** Pre-commit hook checks for @traceability in new functions

## AI Constraints

**What AI SHOULD do:**
- ✅ Generate comprehensive negative tests
- ✅ Follow domain vocabulary strictly
- ✅ Add traceability annotations to all new code
- ✅ Use existing patterns (don't reinvent the wheel)

**What AI should NOT do:**
- ❌ Add features not in user stories (no scope creep!)
- ❌ Modify no-touch zones without approval
- ❌ Use deprecated patterns (see `docs/deprecated-patterns.md`)
- ❌ Generate tests without "should NOT" negative tests

---

**This file is the single source of truth for project-specific AI constraints.**
**Update this file when project rules change.**
```

**Hol dokumentáljuk:**
- [ ] **Phase 4.2:** Document Templates - 4.2.5 LOOM.md Template
- [ ] **9300-guidelines/**: Best practice guide: "How to Write LOOM.md"

**Becsült idő:** 30 perc (template creation + documentation)

---

### 1.7.2 Pre-Commit Quality Gates Design - 45 perc

**Feladat:** Tervezni és dokumentálni egy **5-stage pre-commit validation pipeline**-t.

**Háttér:**
A Reddit setup 5-stage quality gate pipeline-t használ:
1. **Pre-commit** - Lint, format, type check (local)
2. **PR validation** - Build, unit tests, integration tests
3. **Preview deployment** - Deploy to staging, smoke tests
4. **E2E tests** - Full user journey tests
5. **Production deployment** - Final validation, rollout

**Loom kontextusban:**
Loom-specifikus pre-commit gates:
1. **Traceability validation** - All new code has @traceability
2. **Test quality check** - Negative test ratio ≥20%
3. **Documentation sync** - L0/L1/L2/L3 consistency check
4. **ID scheme validation** - US-XXX, AC-XXX-X format check
5. **Semantic consistency** - Code matches acceptance criteria (LLM-powered, optional)

**Git Hook Implementation (Husky + lint-staged):**

**Pre-commit hook (.husky/pre-commit):**
```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

echo "🧶 Loom Pre-Commit Validation Pipeline"

# Stage 1: Traceability Check
echo "Stage 1/5: Traceability validation..."
npx loom-validate --stage=traceability --changed-files-only
if [ $? -ne 0 ]; then
  echo "❌ Traceability validation failed!"
  echo "Ensure all new functions have @traceability annotations."
  exit 1
fi

# Stage 2: Test Quality Check
echo "Stage 2/5: Test quality validation..."
npx loom-validate --stage=test-quality --changed-files-only
if [ $? -ne 0 ]; then
  echo "❌ Test quality validation failed!"
  echo "Ensure negative test ratio ≥20% and 'should NOT' tests exist."
  exit 1
fi

# Stage 3: Documentation Sync Check
echo "Stage 3/5: Documentation sync validation..."
npx loom-validate --stage=doc-sync
if [ $? -ne 0 ]; then
  echo "❌ Documentation sync failed!"
  echo "L0/L1/L2/L3 documents are inconsistent. Run '/loom-validate' for details."
  exit 1
fi

# Stage 4: ID Scheme Validation
echo "Stage 4/5: ID scheme validation..."
npx loom-validate --stage=id-scheme
if [ $? -ne 0 ]; then
  echo "❌ ID scheme validation failed!"
  echo "Ensure IDs follow US-XXX, AC-XXX-X, ENT-XXX format."
  exit 1
fi

# Stage 5: Semantic Consistency (Optional, LLM-powered)
if [ "$LOOM_SEMANTIC_CHECK" = "true" ]; then
  echo "Stage 5/5: Semantic consistency validation (LLM-powered)..."
  npx loom-validate --stage=semantic-consistency --ai
  if [ $? -ne 0 ]; then
    echo "⚠️  Semantic consistency check failed (warning only)."
    echo "Code may not match acceptance criteria. Review manually."
  fi
fi

echo "✅ All pre-commit validations passed!"
```

**CI/CD Pipeline (GitHub Actions / GitLab CI):**
```yaml
# .github/workflows/loom-validation.yml
name: Loom Validation Pipeline

on: [pull_request]

jobs:
  loom-validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install Loom CLI
        run: npm install -g @loom/cli

      - name: Run Full Loom Validation
        run: loom validate --full

      - name: Generate Traceability Report
        run: loom trace --format=html > traceability-report.html

      - name: Upload Report
        uses: actions/upload-artifact@v3
        with:
          name: traceability-report
          path: traceability-report.html
```

**Hol dokumentáljuk:**
- [ ] **Phase 2.4:** Environment & Infrastructure - 1300-environment-infrastructure-enablement.md
- [ ] **Phase 3.9:** Quality Assurance - 2330-quality-assurance.md
- [ ] **Phase 4.1.1:** Git Guidelines - Pre-commit hooks section

**Becsült idő:** 45 perc (design + documentation)

---

### 1.7.3 Context-Triggered Skills Framework - 60 perc

**Feladat:** Tervezni egy **context-triggered skills** framework-öt Claude Code-hoz.

**Háttér:**
A Reddit setup "custom skills"-t használ, amelyek **automatikusan aktiválódnak** bizonyos kontextusokban:
- `code-quality-gate` - Aktiválódik minden code generation után
- `strict-typescript-mode` - Aktiválódik TypeScript fileokban
- `multi-llm-advisor` - Aktiválódik architecture decision-öknél

**Loom kontextusban:**
Context-triggered skills = **Domain-specific automation rules**

**Skill példák:**

**1. `loom-derive-on-l0-change`** (Auto-trigger L1 derivation)
```yaml
# .claude/skills/loom-derive-on-l0-change.md
---
name: loom-derive-on-l0-change
description: Auto-trigger L1 derivation when L0 files change
trigger:
  - file_changed: "requirements/user-stories.md"
  - file_changed: "requirements/domain-vocabulary.md"
auto_run: true
---

# Loom: Auto-Derive L1 on L0 Change

When L0 documents (user-stories.md, domain-vocabulary.md) are modified,
automatically suggest L1 derivation.

## Trigger Conditions
- user-stories.md modified → Derive acceptance-criteria.md, business-rules.md
- domain-vocabulary.md modified → Derive domain-model.md

## Actions
1. Detect changed sections (use git diff)
2. Identify affected user stories (US-XXX)
3. Suggest: "/loom-generate --from-user-story US-XXX --level L1"
4. Wait for user approval

## Example
```
User modifies user-stories.md (adds US-QUOTE-004)
  ↓
Skill auto-triggers:
  "I noticed you added US-QUOTE-004. Would you like me to generate
   acceptance criteria and business rules for this user story?"

  [Yes] → Run /loom-generate --from-user-story US-QUOTE-004 --level L1
  [No]  → Skip
```
```

**2. `loom-tdai-enforce`** (Enforce TDAI on code generation)
```yaml
# .claude/skills/loom-tdai-enforce.md
---
name: loom-tdai-enforce
description: Enforce Test-Driven AI Development (tests before code)
trigger:
  - tool: Write
    file_pattern: "src/**/*.ts"
  - tool: Edit
    file_pattern: "src/**/*.ts"
auto_run: true
---

# Loom: TDAI Enforcement

**Rule:** Never generate code without tests first.

## Pre-Code Generation Check

Before generating any implementation code, verify:
1. Test file exists (*.test.ts)
2. Test file has ≥5 test cases for related AC
3. Test file has ≥20% negative tests
4. Test file has ≥1 "should NOT" test

## Actions

**If tests DON'T exist:**
```
❌ BLOCKED: Cannot generate code without tests (TDAI violation!)

Would you like me to generate tests first?
[Yes] → Generate test-case.md → Generate *.test.ts → Then generate code
[No]  → Abort code generation
```

**If tests exist:**
```
✅ Tests found: src/domain/Quote.test.ts (8 tests, 3 negative, 1 "should NOT")
Proceeding with code generation...
```

## Example
```
User: "Implement the cancelQuote function"
  ↓
Skill checks: Does src/domain/Quote.test.ts exist?
  ↓
NO → BLOCK: "Generate tests first (TDAI)"
YES → ALLOW: "Proceeding with implementation"
```
```

**3. `loom-traceability-annotate`** (Auto-add traceability annotations)
```yaml
# .claude/skills/loom-traceability-annotate.md
---
name: loom-traceability-annotate
description: Auto-add @traceability annotations to new code
trigger:
  - tool: Write
    file_pattern: "src/**/*.ts"
  - function_created: true
auto_run: true
---

# Loom: Auto-Traceability Annotation

**Rule:** Every new function MUST have @traceability annotation.

## Detection
When creating a new function, detect:
1. Function name (e.g., `cancelQuote`)
2. Related feature ticket (e.g., FT-QUOTE-003-001)
3. Related user story (e.g., US-QUOTE-003)
4. Related acceptance criteria (e.g., AC-QUOTE-003-1, AC-QUOTE-003-2)

## Auto-Generate Annotation
```typescript
/**
 * @traceability US-QUOTE-003 (user-stories.md#us-quote-003)
 * @implements AC-QUOTE-003-1, AC-QUOTE-003-2
 * @ticket FT-QUOTE-003-001
 */
export async function cancelQuote(quoteId: string): Promise<void> {
  // Implementation...
}
```

## Validation
Pre-commit hook validates:
- ✓ All new functions have @traceability
- ✓ Referenced IDs exist (US-QUOTE-003, AC-QUOTE-003-1)
- ✓ @traceability format is correct
```

**Framework Design:**
```
Claude Code Skills Architecture:
  ├── .claude/skills/
  │   ├── loom-derive-on-l0-change.md      (auto-derivation)
  │   ├── loom-tdai-enforce.md             (tests before code)
  │   ├── loom-traceability-annotate.md    (auto-annotation)
  │   ├── loom-validate-on-commit.md       (pre-commit check)
  │   └── loom-suggest-refactoring.md      (optional)
  └── .loom/
      └── skill-config.yml                 (enable/disable skills)
```

**Hol dokumentáljuk:**
- [ ] **Phase 4.3:** Best Practices - 4.3.4 Context-Triggered Skills
- [ ] **9300-guidelines/**: "Loom Skills Guide"

**Becsült idő:** 60 perc (design + 3 skill templates + documentation)

---

### 1.7.4 Multi-Model Validation (Enterprise Feature) - 30 perc

**Feladat:** Tervezni egy **multi-model validation** feature-t enterprise tier-hez.

**Háttér:**
A Reddit setup "multi-llm-advisor" skill-t használ:
- Query multiple AI models (GPT-4, Claude, Gemini)
- Compare responses
- Use for critical architecture decisions

**Loom kontextusban:**
Multi-model validation = **Query multiple AI models for L1/L2 derivations**

**Use Case:**
- **Critical derivations** (pl. domain model design, API contract design)
- **Quality check** (validate AI-generated content with another model)
- **Consensus-based approval** (if 2/3 models agree → auto-approve)

**Implementation Design:**

**Command:**
```bash
/loom-generate --from-user-story US-QUOTE-003 --multi-model
```

**Workflow:**
```
1. User triggers multi-model derivation
   ↓
2. Loom queries 3 models in parallel:
   - Claude Opus 4.5 (primary)
   - GPT-4o (secondary)
   - Gemini Pro (tertiary)
   ↓
3. Compare outputs:
   - Domain model entities (ENT-XXX)
   - Acceptance criteria count
   - Business rules identified
   ↓
4. Generate comparison report:

   Multi-Model Derivation Report (US-QUOTE-003)

   | Aspect              | Claude Opus | GPT-4o | Gemini Pro | Consensus |
   |---------------------|-------------|--------|------------|-----------|
   | Entities            | 3 (Quote, Customer, Invoice) | 3 | 2 (Quote, Customer) | ✓ Quote, Customer |
   | Acceptance Criteria | 3 | 4 | 3 | ✓ 3-4 AC |
   | Business Rules      | 1 (BR-QUOTE-003) | 2 (BR-QUOTE-003, BR-QUOTE-004) | 1 | ⚠️ Conflict |

   Recommendation: Use Claude Opus output (highest confidence),
                   but manually review BR-QUOTE-004 suggested by GPT-4o.
   ↓
5. User reviews and approves
```

**Cost Consideration:**
- 3x API cost (Claude + GPT + Gemini)
- **Enterprise-only feature** (too expensive for free tier)
- Opt-in per derivation (not default)

**Configuration (.loom/multi-model-config.yml):**
```yaml
multi_model_validation:
  enabled: true  # Enterprise tier only
  models:
    primary: "claude-opus-4.5"
    secondary: "gpt-4o"
    tertiary: "gemini-pro"

  triggers:
    - level: L1  # Domain model derivation
      auto_run: false  # Manual trigger only
    - level: L2  # Architecture derivation
      auto_run: false

  cost_limit:
    max_per_month: 500  # USD
    alert_threshold: 400
```

**Hol dokumentáljuk:**
- [ ] **Phase 3.2:** System Design - Multi-model validation section
- [ ] **Phase 3.9:** Quality Assurance - Enterprise validation features
- [ ] **Thinking docs:** Enterprise features roadmap

**Becsült idő:** 30 perc (design + documentation)

---

### 1.7.5 MCP Server Architecture Design - 60 perc

**Feladat:** Tervezni a **Loom MCP Server** architektúrát, amely a core implementation lesz.

**Háttér:**
Az MCP (Model Context Protocol) szerverek **kritikus részei** a Loom tooling implementációnak, nem "future enhancement". Az MCP server:
- **Helyettesíti a standalone CLI-t** - MCP server IS the implementation
- **Native Claude Code integráció** - Tools, resources, prompts all native
- **Standardizált protokoll** - Follow MCP spec
- **Resource references** - `@loom:user-story://US-001` works everywhere

**Key insight:** Instead of building a standalone `loom` CLI, build an **MCP server** that exposes Loom functionality as tools Claude Code can use natively.

---

**MCP Server Design:**

**Server name:** `@loom/mcp-server`

**5 Core Tools:**
1. **`loom_validate`** - Traceability & quality validation
   - Stages: traceability, test-quality, doc-sync, id-scheme, semantic, full
   - Pre-commit integration

2. **`loom_derive`** - Documentation derivation (L0→L1→L2→L3)
   - From user story ID
   - Levels: L1 (domain model), L2 (architecture), L3 (tests)
   - Multi-model validation (enterprise)

3. **`loom_trace`** - Traceability map generation
   - Formats: tree, graph, html, json
   - Show relationships between US, AC, entities, code

4. **`loom_test_generate`** - TDAI test generation
   - From acceptance criteria ID
   - Negative test ratio configurable (default 30%)
   - Framework support: jest, vitest, mocha, pytest, junit

5. **`loom_init`** - Initialize Loom project structure
   - Project name, domain, tech stack
   - Generate initial L0 templates

**4 Core Resources (@ mentions):**
1. **`@loom:user-story://US-QUOTE-003`** - User story content
2. **`@loom:ac://AC-QUOTE-003-1`** - Acceptance criteria content
3. **`@loom:entity://ENT-Quote`** - Entity JSON schema
4. **`@loom:test://TC-QUOTE-003-001`** - Test case content

**Auto-Generated Slash Commands:**
- `/loom__loom__validate-full` - Run full validation
- `/loom__loom__derive-from-story US-QUOTE-003` - Derive docs from story

**Technology Stack:**
- Language: TypeScript (Node.js)
- MCP SDK: `@modelcontextprotocol/sdk`
- Server type: `stdio` (local process)
- Distribution: npm package `@loom/mcp-server`

**Project Structure:**
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

**Installation Options:**

**Option 1: Plugin bundles MCP server**
```json
{
  "name": "loom",
  "mcpServers": {
    "loom": {
      "command": "${CLAUDE_PLUGIN_ROOT}/bin/loom-mcp-server",
      "env": {
        "LOOM_PROJECT_ROOT": "${CLAUDE_PROJECT_ROOT}"
      }
    }
  }
}
```

**Option 2: Separate installation**
```bash
npm install -g @loom/mcp-server
claude mcp add --transport stdio loom --scope project -- loom-mcp-server
```

**Benefits:**
- ✅ **No standalone CLI needed** - MCP server IS the implementation
- ✅ **80% less code** - No CLI framework, UI, approval workflow needed
- ✅ **Native Claude Code** - Tools, resources, prompts all native
- ✅ **Natural language** - "Validate US-001" vs `loom validate --story US-001`
- ✅ **Team sharing** - Project-scoped `.mcp.json` in git
- ✅ **Future-proof** - MCP protocol evolving, Loom benefits automatically

**Hol dokumentáljuk:**
- [ ] **tmp/thinking/claude-code-as-platform.md** - MCP Server Architecture section (már hozzáadva! ✅)
- [ ] **tmp/thinking/poc-tooling-design.md** - Update with MCP-first approach
- [ ] **Phase 2.4:** Environment & Infrastructure - MCP server installation guide
- [ ] **Phase 4.2:** Document Templates - MCP server `.mcp.json` template

**Implementation Phases:**

**Phase 1: Core MCP Server (MVP)** - 3-4 days
1. Implement `loom_validate` tool (traceability validation)
2. Implement `loom_derive` tool (L0→L1 derivation)
3. Implement user story resource (`@loom:user-story://...`)
4. Package as npm package `@loom/mcp-server`
5. Test with Claude Code

**Phase 2: Enhanced Tools** - 2-3 days
6. Implement `loom_test_generate` (TDAI)
7. Implement `loom_trace` (traceability map)
8. Add more resources (entities, AC, tests)
9. Add MCP prompts (slash commands)

**Phase 3: Advanced Features** - 2-3 days
10. Multi-model validation (enterprise)
11. Semantic consistency check (LLM-powered)
12. Integration with external MCP servers (GitHub, databases)

**Becsült idő:** 60 perc (architecture design + documentation)

---

### Phase 1.7 Összesen

**Status:** ✅ COMPLETE (3.5-4 óra)

**Elkészült feladatok:**
1. ✅ LOOM.md template creation (30 perc) - Git tag: phase-1.7.1
2. ✅ Pre-commit quality gates design (45 perc) - Git tag: phase-1.7.2
3. ✅ Context-triggered skills framework (60 perc) - Git tag: phase-1.7.3
4. ✅ Multi-model validation design (30 perc) - Git tag: phase-1.7.4
5. ✅ **MCP Server architecture design (60 perc)** - Git tag: phase-1.7.5
6. ✅ **Phase 1.7 Complete** - Git tag: phase-1.7-complete

**Teljes idő:** 3.5-4 óra

**Kritikusság:** 🟢 MEDIUM-HIGH (jelentős UX improvement, enterprise appeal)

**Elkészült eredmények:**
```
New documentation (7 új fájl, ~3,900+ sor):
  ✅ 9400-templates/LOOM.md - Project-specific AI constraint template (400+ lines)
  ✅ 9300-guidelines/1700-loom-md-guidelines.md - Comprehensive guide (500+ lines)
  ✅ 9300-guidelines/1710-pre-commit-quality-gates.md - 5-stage pipeline (700+ lines)
  ✅ 9300-guidelines/1720-context-triggered-skills.md - Framework (650+ lines)
  ✅ 9400-templates/skills/loom-derive-on-l0-change.md
  ✅ 9400-templates/skills/loom-tdai-enforce.md
  ✅ 9400-templates/skills/loom-traceability-annotate.md
  ✅ 9300-guidelines/1730-multi-model-validation.md - Enterprise feature (650+ lines)
  ✅ tmp/thinking/poc-tooling-design.md - Updated to v2.0 (MCP-first architecture)

Git commits:
  ✅ "Complete Phase 1.7.1 - LOOM.md Template Creation"
  ✅ "Complete Phase 1.7.2 - Pre-Commit Quality Gates Design"
  ✅ "Complete Phase 1.7.3 - Context-Triggered Skills Framework"
  ✅ "Complete Phase 1.7.4 - Multi-Model Validation"
  ✅ "Complete Phase 1.7.5 - MCP Server Architecture Design"

Key architectural insight:
  ✅ MCP Server as PRIMARY implementation (not standalone CLI)
  ✅ 80% less code to maintain
  ✅ 8-13 days development vs. 20-28 days for standalone CLI
  ✅ Native Claude Code integration via MCP protocol
```

**Integration points:**
  - Phase 2.4: Environment & Infrastructure (hooks)
  - Phase 3.9: Quality Assurance (validation pipeline)
  - Phase 4.2: Document Templates (LOOM.md)
  - Phase 4.3: Best Practices (skills guide)

---

## ⏭️ Phase 2: Project Lifecycle Documentation (2-3 óra)

### Cél
A Project Lifecycle dokumentumok frissítése, hogy tükrözzék a Loom orchestration platform koncepciókat és az AI-first megközelítést.

### 2.1 Project Initiation (1000-project-initiation.md) - 20 perc

**Jelenlegi állapot:** Általános projekt indítás leírás

**Frissítendő:**
- [ ] Loom-specifikus projekt indítási lépések
- [ ] AI-PDS struktúra inicializálás (`/loom-init`)
- [ ] Kezdeti L0 dokumentumok létrehozása
- [ ] Tool setup (Claude Code plugin)

**Új tartalmi elemek:**
- Loom projekt inicializálási checklist
- `/loom-init` command magyarázat
- Kezdeti fájlstruktúra (L0 documents)
- AI collaboration setup

**Becsült idő:** 20 perc

---

### 2.2 AI-PDS Preparation (1100-ai-pds-preparation.md) - 30 perc

**Jelenlegi állapot:** AI-PDS felkészítés általános leírás (már van egy markdown fix)

**Frissítendő:**
- [ ] Loom-specific preparation steps
- [ ] L0 document preparation (domain-vocabulary.md, user-stories.md)
- [ ] Team onboarding a Loom workflow-ra
- [ ] Derivation strategy beállítása

**Új tartalmi elemek:**
- L0 document templates és példák
- Team training checklist (Loom concepts)
- AI-first, human-in-the-loop workflow magyarázat
- ID scheme setup (US-XXX, AC-XXX-X, ENT-XXX)

**Becsült idő:** 30 perc

---

### 2.3 Onboarding (1200-onboarding.md) - 20 perc

**Jelenlegi állapot:** Team onboarding leírás

**Frissítendő:**
- [ ] Loom-specific onboarding
- [ ] 3 pillars introduction (Derivation, TDAI, Traceability)
- [ ] Claude Code skills training
- [ ] Hands-on example (Quote Cancellation walkthrough)

**Új tartalmi elemek:**
- Loom onboarding checklist
- "First Feature with Loom" tutorial
- Common pitfalls és best practices
- Link to thinking docs for deep dive

**Becsült idő:** 20 perc

---

### 2.4 Environment & Infrastructure (1300-environment-infrastructure-enablement.md) - 20 perc

**Jelenlegi állapot:** Infrastructure setup leírás

**Frissítendő:**
- [ ] Claude Code installation és setup
- [ ] Loom plugin installation (`claude-code plugin install loom`)
- [ ] Git hooks setup (pre-commit validation)
- [ ] CI/CD pipeline for validation (`/loom-validate`)

**Új tartalmi elemek:**
- Loom tooling setup guide
- Validation pipeline configuration
- GitHub Actions / CI/CD integration
- Pre-commit hooks for traceability check

**Becsült idő:** 20 perc

---

### Phase 2 Commit Strategy:

**Option A (Recommended):** Egy commit az összes Project Lifecycle frissítéssel
```
feat: Update Project Lifecycle for Loom orchestration platform

- Project Initiation: Loom-specific setup steps
- AI-PDS Preparation: L0 document preparation, team training
- Onboarding: 3 pillars introduction, hands-on examples
- Environment: Claude Code setup, validation pipeline

All sections now reflect Loom workflow (L0→L1→L2→L3),
TDAI methodology, and bidirectional traceability.
```

**Option B (Alternative):** Külön commit minden fájlhoz (tracking finomabb)

**Becsült idő Phase 2 összesen:** 1.5-2 óra

---

### Phase 2 Összesen

**Status:** ✅ COMPLETE (1.5-2 óra)

**Elkészült feladatok:**
1. ✅ Project Initiation (20 perc) - Git tag: phase-2.1
2. ✅ AI-DOP Preparation (30 perc) - Git tag: phase-2.2
3. ✅ Onboarding (20 perc) - Git tag: phase-2.3
4. ✅ Environment & Infrastructure (20 perc) - Git tag: phase-2.4
5. ✅ **Phase 2 Complete** - Git tag: phase-2-complete

**Teljes idő:** 1.5-2 óra

**Elkészült eredmények:**
```
4 dokumentum frissítve (~1,400+ sor új tartalom):
  ✅ 1000-project-initiation.md (27 → 295 sor)
     - Quick Start: Initialize Loom Project (4 lépés)
     - Loom Project Initiation Checklist (7 szekció)

  ✅ 1100-ai-pds-preparation.md (41 → 347 sor)
     - 5 Core Activity: ID scheme, Domain Vocabulary, User Stories, Derivation Strategy, Team Training
     - L0 document templates + best practices

  ✅ 1200-onboarding.md (78 → 389 sor)
     - 3 Pillars Introduction (Documentation Derivation, TDAI, Traceability)
     - "First Feature with Loom" hands-on tutorial (7 lépés)
     - Common Pitfalls & Best Practices

  ✅ 1300-environment-infrastructure-enablement.md (75 → 383 sor)
     - Claude Code + Loom MCP Server installation
     - Pre-commit hooks setup (Husky + lint-staged)
     - CI/CD validation pipeline (GitHub Actions, GitLab CI)
     - Branch protection configuration

Git commits:
  ✅ "Complete Phase 2.1 - Project Initiation update"
  ✅ "Complete Phase 2.2 - AI-DOP Preparation update"
  ✅ "Complete Phase 2.3 - Onboarding update"
  ✅ "Complete Phase 2.4 - Environment & Infrastructure update"

Key outcomes:
  ✅ Complete Loom project setup guide (initialization to first deployment)
  ✅ L0 document preparation best practices
  ✅ Hands-on "First Feature with Loom" tutorial
  ✅ Comprehensive tooling setup (Claude Code, MCP Server, hooks, CI/CD)
```

---

## ✅ Phase 3: Release Lifecycle Documentation (3-4 óra)

### Cél
Release Lifecycle dokumentumok **teljes átdolgozása** a Loom derivation strategy szerint.

Ez a **legkritikusabb** phase, mert itt találhatók a core workflow dokumentumok.

**Status:** ✅ KÉSZ (Összes fő fájl + 3 sub-phase KÉSZ)
**Git tags:** phase-3.1, phase-3.2, phase-3.3, phase-3.4, phase-3.5, phase-3.6, phase-3.7, phase-3.8, phase-3-feature-def, phase-3-implementation, phase-3-qa (11 tags összesen)

---

### 3.1 Functional Specification (2100-functional-specification.md) - 20 perc ✅

**Frissítendő:**
- [x] L0 documents (domain-vocabulary.md, user-stories.md)
- [x] Human input 80% / AI derivation koncepció
- [x] `/loom-derive` workflow
- [x] Approval gates

**Elkészült tartalmi elemek:**
- ✅ "Writing L0 Documents" best practices
- ✅ User story template with Loom IDs (US-XXX) - 3 complete examples
- ✅ Domain vocabulary templates with Quote Management example
- ✅ AI-first functional spec workflow with 5 Core Activities
- ✅ Comprehensive examples (Quote, Customer, Status with state machines)

**Eredmény:** 25 → 590 sor (23x expansion)
**Git tag:** phase-3.1

---

### 3.2 System Design (2200-system-design.md) - 30 perc ✅

**Frissítendő:**
- [x] L1, L2 derivation magyarázat
- [x] AI-generated design dokumentumok
- [x] Traceability a functional spec-hez
- [x] Human approval points

**Elkészült tartalmi elemek:**
- ✅ System design derivation flow (L0 → L1 → L2)
- ✅ AI-generated design artifacts (domain-model.md, api-contracts.md, sequence-diagrams.mmd)
- ✅ Approval workflow with human-in-the-loop review
- ✅ `/loom-derive --level L2` examples
- ✅ 5 Core Activities with complete examples
- ✅ API contracts with TypeScript schemas, error codes, side effects
- ✅ Mermaid sequence diagram for quote cancellation flow

**Eredmény:** 14 → 820 sor (58x expansion)
**Git tag:** phase-3.2

---

### 3.3 Domain Modelling (2210-domain-modelling.md) - 30 perc ✅

**Frissítendő:**
- [x] **L1 derivation**: domain-model.md from domain-vocabulary.md
- [x] Entity identification (ENT-XXX)
- [x] AI-generated domain model
- [x] Traceability: domain terms → entities → fields

**Elkészült tartalmi elemek:**
- ✅ "AI derives domain model from vocabulary" magyarázat
- ✅ Entity naming conventions (ENT-QUOTE, ENT-CUSTOMER)
- ✅ State machines with Mermaid diagrams
- ✅ Relationships with ERD diagram
- ✅ Invariants linking to business rules (BR-XXX)
- ✅ 5 Core Activities (Entities, State Machines, Relationships, Invariants, Review)
- ✅ Complete Quote domain model derivation example

**Példa Flow (Elkészült):**
```
L0: domain-vocabulary.md
  "Quote: A formal offer to a customer"
  "Status: Draft, Sent, Accepted, Rejected, Cancelled"
    ↓ AI derives (12 seconds)
L1: domain-model.md
  ENT-QUOTE (10 properties)
    - id: UUID (Required, Unique, PK)
    - customerId: UUID (Required, FK(Customer.id))
    - status: QuoteStatus (Required)
    - cancelledAt: DateTime (Optional)
  State Machine (Mermaid diagram)
    Draft → Sent → Accepted/Rejected/Cancelled
  Relationships (ERD diagram)
    Quote BelongsTo Customer (Many-to-One)
  Invariants (5 rules)
    BR-QUOTE-001, BR-QUOTE-002, BR-QUOTE-007, BR-QUOTE-008, BR-QUOTE-009
```

**Eredmény:** 33 → 675 sor (20x expansion)
**Git tag:** phase-3.3

---

### 3.4 Requirements Specification (2220-requirements-specification.md) - 30 perc ✅

**Frissítendő:**
- [x] **L1 derivation**: acceptance-criteria.md, business-rules.md
- [x] AI-generated AC from user stories
- [x] ID scheme (AC-XXX-X, BR-XXX)
- [x] Traceability: US → AC → BR

**Új tartalmi elemek:**
- "AI derives requirements from user stories" flow
- Acceptance criteria template (Given-When-Then?)
- Business rules identification
- Validation: minden US-nek van AC-je

**Példa Flow:**
```
L0: user-stories.md
  US-QUOTE-003: Cancel Quote
    ↓ AI derives
L1: acceptance-criteria.md
  AC-QUOTE-003-1: Only 'Sent' quotes can be cancelled
  AC-QUOTE-003-2: Cancellation is recorded
  AC-QUOTE-003-3: User receives confirmation
    ↓ AI derives
L1: business-rules.md
  BR-QUOTE-003: Status transition rules
```

**Elkészült tartalmi elemek:**
- ✅ Given/When/Then acceptance criteria format (6 ACs for US-QUOTE-003)
- ✅ Complete business rules with traceability
- ✅ Traceability matrix (US → AC → BR → Code)
- ✅ 5 Core Activities with comprehensive examples
- ✅ Test data and error case tables

**Eredmény:** 24 → 856 sor (35x expansion)
**Git tag:** phase-3.4

---

### 3.5 Architecture (2230-architecture.md) - 30 perc ✅

**Frissítendő:**
- [x] **L2 derivation**: architecture decisions from L1
- [x] Interface contracts (API endpoints)
- [x] Sequence designs
- [x] Service boundaries (optional, L3)

**Új tartalmi elemek:**
- Architecture derivation from domain model + requirements
- API contract generation (API-XXX)
- Sequence diagram generation (Mermaid)
- ADRs with traceability

**Példa Flow:**
```
L1: domain-model.md (ENT-Quote)
L1: acceptance-criteria.md (AC-QUOTE-003-1, AC-QUOTE-003-2)
    ↓ AI derives
L2: interface-contracts.md
  API-POST-QUOTE-CANCEL
    POST /api/quotes/{id}/cancel
    @traceability US-QUOTE-003, AC-QUOTE-003-1, AC-QUOTE-003-2
    ↓ AI derives
L2: sequence-design.md
  Sequence: Quote Cancellation Flow (Mermaid diagram)
```

**Elkészült tartalmi elemek:**
- ✅ 8 Core Activities (Service Boundaries, Aggregates, Events, API Contracts, Sequences, ADRs, NFR, Dependencies)
- ✅ Complete Quote.cancel() TypeScript implementation with BR validation
- ✅ OpenAPI 3.1 specs with error responses
- ✅ Kubernetes manifests and Terraform examples
- ✅ Mermaid sequence diagrams
- ✅ ADR templates with NFR traceability

**Eredmény:** 33 → 1038 sor (31x expansion)
**Git tag:** phase-3.5

---

### 3.6 Development (2300-development.md) - 20 perc ✅

**Frissítendő:**
- [x] **L3 derivation**: test-case.md, feature-tickets.md
- [x] TDAI workflow (tests before code)
- [x] Implementation with traceability
- [x] `/loom-implement` command

**Új tartalmi elemek:**
- Development workflow with Loom
- TDAI integration
- Feature ticket generation from AC
- Code generation with @traceability

**Elkészült tartalmi elemek:**
- ✅ L2 → L3 derivation workflow (4 Sub-Phases)
- ✅ AI code generation examples (Quote.cancel(), API routes, Jest tests)
- ✅ Complete TypeScript implementation with idempotency
- ✅ Test-Driven AI Development (TDAI) workflow
- ✅ @traceability and @implements annotations

**Eredmény:** 18 → 618 sor (34x expansion)
**Git tag:** phase-3.6

---

### 3.7 Feature Definition (2310-feature-definition.md) - 30 perc ✅

**Frissítendő:**
- [x] Feature ticket generation (L3)
- [x] Traceability: AC → Feature Ticket → Code
- [x] Implementation spec with TDAI tests

**Elkészült tartalmi elemek:**
- ✅ AI-driven feature ticket generation from L1 acceptance criteria
- ✅ Complete FT-QUOTE-003 example with all sections
- ✅ 5 Core Activities (Generate, Link, Checklist, Track, Validate)
- ✅ Loom commands and workflows
- ✅ Time savings: 95% (2 min vs. 45-60 min per ticket)

**Eredmény:** 72 → 764 sor (10.6x expansion)
**Git tag:** phase-3-feature-def

---

### 3.8 Implementation (2320-implementation.md) - 30 perc ✅

**Frissítendő:**
- [x] **TDAI workflow**: Tests → Code
- [x] Traceability annotations (@traceability, @implements)
- [x] Code generation with Claude Code
- [x] Negative tests magyarázat (hallucination prevention!)

**Elkészült tartalmi elemek:**
- ✅ Complete TDAI (Test-Driven AI Development) workflow
- ✅ AI generates 38 test cases (25 seconds) → implementation (18 seconds)
- ✅ Complete Quote.cancel() example (tests + code + API)
- ✅ "Should NOT" tests to prevent AI hallucinations
- ✅ 5 Core Activities with detailed examples
- ✅ Time savings: 97% (13 min vs. 6-8 hours per feature)

**Eredmény:** 83 → 871 sor (10.5x expansion)
**Git tag:** phase-3-implementation

---

### 3.9 Quality Assurance (2330-quality-assurance.md) - 30 perc ✅

**Frissítendő:**
- [x] TDAI test validation
- [x] `/loom-validate` workflow
- [x] Test quality metrics (≥20% negative tests, 70:20:10 pyramid)
- [x] Traceability validation

**Elkészült tartalmi elemek:**
- ✅ AI-driven QA workflow (test generation, execution, defect analysis)
- ✅ 9 test cases generated (positive, negative, boundary, idempotency, concurrency)
- ✅ AI root cause analysis (8 seconds) with fix suggestions
- ✅ QA summary report with release recommendation
- ✅ 5 Core Activities with complete examples
- ✅ Time savings: 95% (12 min vs. 4-6 hours per feature)

**Eredmény:** 90 → 962 sor (10.7x expansion)
**Git tag:** phase-3-qa

---

### 3.10 Deployment (2400-deployment.md) - 15 perc ✅

**Frissítendő:**
- [x] CI/CD validation integration
- [x] Pre-deployment validation (`/loom-validate`)
- [x] Documentation sync check

**Elkészült tartalmi elemek:**
- ✅ L3 → Production deployment workflow
- ✅ Blue-green deployment strategy
- ✅ Complete GitHub Actions workflows
- ✅ Kubernetes manifests (blue/green environments)
- ✅ Terraform infrastructure as code
- ✅ Rollback procedures and monitoring

**Eredmény:** 18 → 776 sor (43x expansion)
**Git tag:** phase-3.7

---

### 3.11 Post-Release (2500-post-release.md) - 15 perc ✅

**Frissítendő:**
- [x] Documentation maintenance
- [x] L0 updates → re-derivation workflow
- [x] `/loom-update` command

**Elkészült tartalmi elemek:**
- ✅ Production feedback loop with AI-driven incident response
- ✅ 6 Post-Release Activities (Monitor, Incidents, Hotfix, Retrospectives, L0 Updates, Performance)
- ✅ AI anomaly detection and root cause analysis
- ✅ Automated hotfix generation workflow
- ✅ MTTR < 60 minutes with AI assistance (example: 22 min)
- ✅ Complete incident report template (DR-2025-001)

**Eredmény:** 45 → 875 sor (19x expansion)
**Git tag:** phase-3.8

---

### Phase 3 Commit Strategy:

**Option A (Recommended):** Tematikus commitok (3-4 commit)
1. Functional Specification + System Design (L0, L1, L2 focus)
2. Development + Implementation (L3, TDAI focus)
3. QA + Deployment + Post-Release (Validation focus)

**Option B:** Egy nagy commit az egész Release Lifecycle-ra

**Becsült idő Phase 3 összesen:** 3.5-4 óra

---

## ✅ Phase 3.10: PoC Validation - Document Derivation (KÉSZ - 2025-12-20)

### Cél

Validálni a Loom document derivation koncepciót működő prototípussal.

**Status:** ✅ COMPLETE SUCCESS
**Git commit:** `40839c8`
**Eredmények:** `tmp/poc/POC-RESULTS.md`

---

### 3.10.1 Scope

Teljes L0→L1→L2→L3 derivációs lánc validálása Claude Code Skills használatával:

```
L0 (Input)              L1 (Derived)              L2 (Derived)              L3 (Derived)
────────────────────────────────────────────────────────────────────────────────────────────
user-stories.md    →    acceptance-criteria.md →  interface-contracts.md →  test-case.md
  US-QUOTE-003            6 ACs (Given/When/Then)   3 API operations          24 TDAI tests
  US-QUOTE-004            + Error cases             + Request/Response        + 8 Positive
                          + Traceability            + Error codes             + 8 Negative
                                                                              + 4 Boundary
                     →    business-rules.md     →  sequence-design.md        + 3 Should NOT
                          6 BRs + 1 INV             5 Mermaid diagrams        + 1 Idempotency
```

---

### 3.10.2 Skills Created

| Skill | Purpose | Location |
|-------|---------|----------|
| `/loom-derive` | L0→L1 derivation | `.claude/skills/loom-derive.md` |
| `/loom-derive-l2` | L1→L2 derivation | `.claude/skills/loom-derive-l2.md` |
| `/loom-derive-l3` | L2→L3 TDAI test gen | `.claude/skills/loom-derive-l3.md` |

---

### 3.10.3 Key Metrics

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| L0 Input | - | 53 lines | ✅ |
| L1-L3 Output | - | 1390 lines | ✅ |
| **Expansion ratio** | - | **26x** | ✅ |
| Format compliance | 100% | 100% | ✅ |
| Traceability | 100% | 100% | ✅ |
| Negative test ratio | ≥20% | 33% | ✅ |
| "Should NOT" tests | ≥5% | 13% | ✅ |
| Test pyramid (Unit) | 70% | 67% | ✅ |
| Human correction needed | <20% | <15% | ✅ |

---

### 3.10.4 Time Validation

| Phase | Estimated | Actual | Notes |
|-------|-----------|--------|-------|
| Skill creation (L0→L1) | 45 min | ~30 min | Prompt engineering |
| Skill creation (L1→L2) | 45 min | ~25 min | Faster with pattern |
| Skill creation (L2→L3) | 45 min | ~20 min | TDAI template reuse |
| L0→L1 derivation | 5 min | ~3 min | Reading + generation |
| L1→L2 derivation | 10 min | ~5 min | More complex output |
| L2→L3 derivation | 15 min | ~8 min | 24 tests generated |
| **Total** | 165 min | ~91 min | **45% faster** |

---

### 3.10.5 Files Generated

```
tmp/poc/
├── input/
│   └── user-stories.md           # L0 test input (53 lines)
├── output/
│   ├── acceptance-criteria.md     # L1 output (141 lines)
│   ├── business-rules.md          # L1 output (233 lines)
│   ├── interface-contracts.md     # L2 output (234 lines)
│   ├── sequence-design.md         # L2 output (280 lines)
│   └── test-case.md               # L3 output (502 lines)
└── POC-RESULTS.md                 # Full documentation
```

---

### 3.10.6 Következtetések

1. **Full chain works:** L0→L1→L2→L3 derivation validated end-to-end
2. **Claude Code Skills sufficient:** No MCP Server needed for PoC
3. **High quality output:** >85% usable without edits
4. **Format consistency:** 100% compliance with templates
5. **Traceability maintained:** Every element links to source
6. **TDAI principles work:** 33% negative tests, "Should NOT" tests included
7. **Significant expansion:** 26x content generation (53→1390 lines)

**Recommendation:** PoC COMPLETE. Ready for real-world testing.

---

## ⏭️ Phase 4: Guidelines & Templates (1-2 óra)

### Cél
Guidelines frissítése Loom best practices-szel, új templates létrehozása.

---

### 4.1 Communication Guidelines

**4.1.1 Git Guidelines (9300-guidelines/communication/git-guidelines.md)** - 15 perc
- [ ] Commit message convention frissítése
- [ ] Traceability references a commit message-ekben
- [ ] Branch naming strategy (feature/US-XXX-description)
- [ ] Pre-commit hooks (validation)

**4.1.2 Slack Guidelines (9300-guidelines/communication/slack-guidelines.md)** - 10 perc
- [ ] Loom workflow communication
- [ ] AI derivation review requests
- [ ] Approval workflow channels

**Becsült idő:** 25 perc

---

### 4.2 Document Templates

**4.2.1 L0 Templates** - 30 perc
- [ ] domain-vocabulary.md template
- [ ] user-stories.md template (US-XXX format)
- [ ] Best practices guide

**4.2.2 L1 Templates** - 30 perc
- [ ] domain-model.md template (ENT-XXX format)
- [ ] acceptance-criteria.md template (AC-XXX-X format)
- [ ] business-rules.md template (BR-XXX format)

**4.2.3 L2 Templates** - 20 perc
- [ ] interface-contracts.md template (API-XXX format)
- [ ] sequence-design.md template (Mermaid diagrams)
- [ ] initial-data-model.md template

**4.2.4 L3 Templates** - 20 perc
- [ ] test-case.md template (TDAI structure: positive, negative, boundary, "should NOT")
- [ ] feature-tickets.md template (FT-XXX format)

**Becsült idő:** 1 óra 40 perc

---

### 4.3 Best Practices Documents

**4.3.1 Traceability Best Practices** - 20 perc
- [ ] ID naming conventions összefoglaló
- [ ] @traceability annotation patterns (TypeScript, Python, Java)
- [ ] @implements usage
- [ ] Common mistakes

**4.3.2 TDAI Best Practices** - 20 perc
- [ ] Negative test patterns
- [ ] "should NOT" test writing guide
- [ ] Test pyramid enforcement
- [ ] Hallucination detection examples

**4.3.3 Derivation Best Practices** - 20 perc
- [ ] Writing good L0 documents
- [ ] Reviewing AI-generated L1/L2/L3
- [ ] When to manually adjust AI output
- [ ] Approval workflow tips

**Becsült idő:** 1 óra

---

### Phase 4 Commit Strategy:

**Option A:** Tematikus commitok
1. Communication guidelines
2. Document templates (L0, L1, L2, L3)
3. Best practices

**Becsült idő Phase 4 összesen:** 2.5-3 óra

---

## ⏭️ Phase 5: Example AI-PDS Updates (1-2 óra)

### Cél
A Sales & Billing example AI-PDS frissítése, hogy **teljes mértékben tükrözze** a Loom koncepciókat.

---

### 5.1 Example Functional Specification

**5.1.1 domain-vocabulary.md** - 15 perc
- [ ] Loom-style formatting
- [ ] Clear term definitions
- [ ] Traceability anchor IDs

**5.1.2 user-stories.md** - 20 perc
- [ ] US-XXX ID scheme
- [ ] Traceability links
- [ ] "As a ... I want to ... so that ..." format

**5.1.3 domain-model.md** - 30 perc
- [ ] ENT-XXX entities
- [ ] ENT-XXX:field field IDs
- [ ] Traceability: domain vocabulary → entities
- [ ] Aggregates, Value Objects

**5.1.4 acceptance-criteria.md** - 30 perc
- [ ] AC-XXX-X ID scheme
- [ ] Traceability: US → AC
- [ ] Given-When-Then format (optional)

**5.1.5 business-rules.md** - 20 perc
- [ ] BR-XXX ID scheme
- [ ] Traceability: US → BR, AC → BR

**Becsült idő:** 1 óra 55 perc

---

### 5.2 Example System Design

**5.2.1 interface-contracts.md** - 30 perc
- [ ] API-XXX ID scheme
- [ ] REST API contracts with traceability
- [ ] Request/response examples

**5.2.2 sequence-design.md** - 20 perc
- [ ] Mermaid sequence diagrams
- [ ] Traceability to US, AC
- [ ] Clear interaction flows

**5.2.3 initial-data-model.md** - 15 perc
- [ ] Database schema
- [ ] Traceability: entities → tables/collections

**Becsült idő:** 1 óra 5 perc

---

### 5.3 Example Development Artefacts

**5.3.1 test-case.md** - 40 perc
- [ ] TC-XXX-XXX ID scheme
- [ ] TDAI structure (positive, negative, boundary, "should NOT")
- [ ] ≥20% negative tests
- [ ] Traceability: AC → TC
- [ ] Hallucination prevention examples

**5.3.2 feature-tickets.md** - 20 perc
- [ ] FT-XXX ID scheme
- [ ] Implementation specs
- [ ] Traceability: AC → FT → Code

**Becsült idó:** 1 óra

---

### Phase 5 Commit Strategy:

**Option A:** Tematikus commitok
1. Example Functional Specification (L0, L1)
2. Example System Design (L2)
3. Example Development Artefacts (L3, TDAI)

**Becsült idő Phase 5 összesen:** 4 óra (ha minden példát updatelünk)

**Alternatíva:** Csak kiválasztott user story-kat updateljük (pl. US-QUOTE-003 Cancel Quote) - 1-1.5 óra

---

## ⏭️ Phase 6: Final Polish & Validation (1 óra)

### Cél
Utolsó simítások, konzisztencia ellenőrzés, dokumentáció validálás.

---

### 6.1 Cross-Reference Validation - 20 perc

**Ellenőrzendő:**
- [ ] Minden link működik (internal links)
- [ ] Minden thinking document hivatkozás helyes
- [ ] Minden example AI-PDS link frissítve
- [ ] ID scheme konzisztens az egész specben

**Script futtatás:**
```bash
# Find broken markdown links
grep -r "\[.*\](.*\.md" ai-pds-specification/ | grep -v "^Binary"

# Check for old "AI-PDS" references that should be "Loom (AI-PDS)"
grep -r "AI-Ready Project Documentation System" ai-pds-specification/
```

**Becsült idő:** 20 perc

---

### 6.2 Terminology Consistency - 20 perc

**Ellenőrzendő:**
- [ ] "Loom (AI-PDS)" használat konzisztens
- [ ] "Documentation Derivation" vs "AI Derivation" terminológia
- [ ] "TDAI" vs "Test-Driven AI Development" használat
- [ ] ID scheme terminológia (US-XXX, AC-XXX-X, stb.)

**Becsült idő:** 20 perc

---

### 6.3 Version Bumping - 10 perc

**Frissítendő:**
- [ ] Minden updated file version: 2.0.0
- [ ] Status: "approved"
- [ ] last_updated: 2025-12-19

**Becsült idő:** 10 perc

---

### 6.4 Final README Update - 10 perc

**Frissítendő:**
- [ ] Main README status update (Phase 2 → Production Ready?)
- [ ] Roadmap status update
- [ ] Contribution guidelines

**Becsült idő:** 10 perc

---

### Phase 6 Commit:

```
chore: Final polish and validation for Loom v2.0.0

- Cross-reference validation (all links working)
- Terminology consistency check
- Version bumping to 2.0.0
- README updates

Loom specification v2.0.0 complete and ready for implementation.
```

**Becsült idő Phase 6 összesen:** 1 óra

---

## 📋 Összefoglaló Roadmap

| Phase | Leírás | Becsült idő | Prioritás | Állapot |
|-------|--------|-------------|-----------|---------|
| **Phase 1** | Core Documentation (README, Intro, Principles, How to Use, Appendix) | 2 óra | 🔴 CRITICAL | ✅ KÉSZ |
| **Phase 1.5** | Naming Unification (AI-PDS → AI-DOP) | 40-50 perc | 🟡 MEDIUM | 🔄 KÖVETKEZŐ |
| **Phase 1.6** | PM Content Separation (Loom Adoption Playbook creation) | 3.5-4 óra | 🟢 MEDIUM-HIGH | ⏭️ Pending |
| **Phase 1.7** | Claude Code Integration Enhancements (Reddit-Inspired: LOOM.md, pre-commit gates, skills, multi-model, **MCP Server**) | 3.5-4 óra | 🔴 CRITICAL | ⏭️ Pending |
| **Phase 2** | Project Lifecycle (Initiation, Prep, Onboarding, Env) | 1.5-2 óra | 🟡 HIGH | ⏭️ Pending |
| **Phase 3** | Release Lifecycle (Func Spec, Design, Dev, QA, Deploy) | 3.5-4 óra | 🔴 CRITICAL | ⏭️ Pending |
| **Phase 4** | Guidelines & Templates (Communication, Templates, Best Practices) | 2.5-3 óra | 🟢 MEDIUM | ⏭️ Pending |
| **Phase 5** | Example AI-DOP (Sales & Billing example updates) | 1-4 óra | 🟢 MEDIUM | ⏭️ Pending |
| **Phase 6** | Final Polish (Validation, Consistency, Version bumping) | 1 óra | 🟡 HIGH | ⏭️ Pending |

**Teljes becsült idő:** 19.5-25 óra (Phase 1.5-6)

**Már elvégezve (Phase 1):** 2 óra

---

## 🎯 Javasolt Megközelítés

### Option A: "Minimal Viable Documentation" (Gyors, 5-6 óra)

**Fókusz:** Csak a kritikus dokumentumok updatelése

**Lépések:**
1. ✅ Phase 1 - KÉSZ
2. Phase 1.5 - Naming Unification (opcionális, 45 perc)
3. ~~Phase 1.6 - PM Separation~~ (SKIP - nem essential)
4. Phase 2 - Project Lifecycle (1.5 óra)
5. Phase 3 - **Csak a core Release Lifecycle** (2100, 2200, 2210, 2220, 2320) - 2 óra
6. Phase 6 - Final Polish (1 óra)

**Összesen:** ~5-6 óra (vagy ~4.5 óra Phase 1.5 nélkül)

**Előnyök:**
- Gyors befejezés
- Core workflow teljesen dokumentálva
- Használható állapot

**Hátrányök:**
- Guidelines hiányosak
- Example AI-DOP nem teljesen frissítve
- Phase 1.5 nélkül: vegyes AI-PDS/AI-DOP terminológia

---

### Option B: "Complete Transformation" (Teljes, 18-22 óra)

**Fókusz:** Minden dokumentum teljes átdolgozása

**Lépések:**
1. ✅ Phase 1 - KÉSZ
2. Phase 1.5 - Naming Unification (45 perc)
3. Phase 1.6 - PM Content Separation (3.5 óra)
4. Phase 1.7 - Claude Code Integration Enhancements (2.5 óra)
5. Phase 2 - Project Lifecycle (2 óra)
6. Phase 3 - Release Lifecycle TELJES (4 óra)
7. Phase 4 - Guidelines & Templates (2.5 óra)
8. Phase 5 - Example AI-DOP TELJES (4 óra)
9. Phase 6 - Final Polish (1 óra)

**Összesen:** ~20.2 óra

**Előnyök:**
- Teljesen konzisztens dokumentáció
- Egységes AI-DOP terminológia
- Példák és templates készen
- Production-ready

**Hátrányök:**
- Hosszú idő
- Sok munka

---

### Option C: "Iterative Approach" (Ajánlott, 13.5-15 óra)

**Fókusz:** Fontos részek teljes átdolgozása, kevésbé fontos részek alapszintű frissítése

**Lépések:**
1. ✅ Phase 1 - KÉSZ
2. Phase 1.5 - Naming Unification (45 perc) - AJÁNLOTT
3. Phase 1.6 - PM Content Separation (3.5 óra) - AJÁNLOTT (enterprise appeal!)
4. Phase 1.7 - Claude Code Integration (csak 1.7.1 LOOM.md + 1.7.2 Pre-commit gates) - 1.5 óra - AJÁNLOTT
5. Phase 2 - Project Lifecycle TELJES (2 óra)
6. Phase 3 - Release Lifecycle CORE dokumentumok (2210, 2220, 2320, 2330) TELJES, többi alapszintű - 2.5 óra
7. Phase 4 - Csak L0, L1 templates + TDAI best practices + LOOM.md template - 1.5 óra
8. Phase 5 - **EGY user story (US-QUOTE-003)** teljes átdolgozása példaként - 1 óra
9. Phase 6 - Final Polish (1 óra)

**Összesen:** ~14.2 óra

**Előnyök:**
- Fókuszált, minőségi munka
- Egységes AI-DOP terminológia
- Kritikus részek teljesek
- Van példa (US-QUOTE-003)
- Ésszerű idő

**Hátrányök:**
- Nem minden példa frissítve

---

## 🚀 Következő Lépések (MOST)

### Immediate Action: Phase 1.5 - Naming Unification

**1. Döntés: AI-PDS → AI-DOP átnevezés?**
   - **Option A:** "Loom (AI-DOP)" - Pontosabb, következetes
   - **Option B:** "Loom (AI-PDS)" megtartása - Egyszerűbb, már bevezetett
   - **Option C:** Csak "Loom" - Minimális, clean

**2. Ha AI-DOP mellett döntünk:**
   - Phase 1.5.2: Global search & replace (20-30 perc)
   - Phase 1.5.3: README & core files manual review (10 perc)
   - Phase 1.5.4: Commit (5 perc)
   - **Összesen:** ~45 perc

**3. Utána Phase 2:**
   - Phase 2.1: Project Initiation (20 perc)
   - Phase 2.2: AI-DOP Preparation (30 perc)
   - ... stb.

**4. Commit strategy:** Egyeztetés (egy commit vs. több commit)

---

## 📝 Tracking

**Roadmap fájl helye:** `tmp/loom-transformation-roadmap.md`

**Frissítési gyakoriság:** Minden phase végén

**Progress tracking:** Checkboxok (- [ ] / - [x]) + commit history

---

## ❓ Kérdések Döntéshez

1. **Melyik opciót választjuk?** (A: Minimal 5h, B: Complete 13h, C: Iterative 7.5h)
2. **Egy commit phase-enként vagy kisebb commitok?**
3. **Example AI-PDS update mélysége?** (Teljes vs. egy user story)
4. **Guidelines részletessége?** (Teljes templates vs. csak essential)

---

**Roadmap készítette:** Claude Sonnet 4.5
**Dátum:** 2025-12-19
**Verzió:** 1.0.0

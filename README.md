# Loom Project

Development artifacts for the Loom (AI-DOP) framework - thinking documents, evaluations, roadmaps, and PoC results.

## Overview

This repository contains the "behind the scenes" of Loom development:

```
loom-project/
├── thinking/         # Core concept documents
├── evaluations/      # AI evaluations (Sonnet, Opus)
├── roadmap/          # Architecture evolution, PoC roadmaps
└── poc/              # Proof of Concept outputs
```

## Thinking Documents

Core innovations and design decisions:

| Document | Description |
|----------|-------------|
| `documentation-derivation-strategy.md` | 4-level AI derivation hierarchy (L0→L1→L2→L3) |
| `test-driven-ai-development.md` | TDAI - Tests as constraints for AI |
| `bidirectional-traceability-design.md` | Docs ↔ Code linking strategy |
| `structured-interview-pattern.md` | **4th Pillar** - AI asks before deciding |
| `claude-code-as-platform.md` | Skills/MCP architecture decisions |
| `derivation-example-walkthrough.md` | Concrete end-to-end example |
| `poc-tooling-design.md` | Implementation blueprint |

## Evaluations

Critical assessments by Claude models:

| Version | Score | Status |
|---------|-------|--------|
| Sonnet v01 | 5.0/10 | "Overcomplicated, no proof" |
| Sonnet v02 | 7.5/10 | "Radical but achievable vision" |
| Sonnet v03 | 8.0/10 | "Implementation-ready blueprint" |
| Sonnet v04 | 8.6/10 | "Post-PoC comprehensive review" |
| Opus v01 | 7.8/10 | "Specification trap risk" |
| Opus v02.1 | 8.8/10 | "Validated concept + RAG integration" |
| Opus v03 | 9.2/10 | "4-pillar architecture, SI validated" |

## PoC Results

Proof of Concept validation:

- **L0→L1→L2→L3 Chain:** 53 lines → 1390 lines (26x expansion)
- **TDAI Validation:** 33% negative tests, 20% "should NOT"
- **RAG Enhancement:** 3 sections → 7 sections (guideline-compliant)
- **Structured Interview:** All 4 derivation skills now use SI pattern
  - L0→L1: 15 decision points (SC, EH, AU, SE, ST)
  - Domain: 16 decision points (EVO, AGG, REF, INV)
  - L1→L2: 15 decision points (API, COM, SVC, SEC, DAT)
  - L2→L3: 20 decision points (TST, MOC, TDA, COV, ENV)

## Related Repositories

| Repo | Purpose |
|------|---------|
| [specs-for-ai](https://github.com/ikadar/engineering-playbook) | Loom specification |
| [loom-tooling](https://github.com/ikadar/loom-tooling) | Skills, RAG, MCP Server |

## Status

**Current Phase:** Post-PoC, preparing for real-world validation

**Next Steps:**
1. Test on real project (not example domain)
2. Validate multi-developer workflow
3. Measure actual time savings vs manual baseline

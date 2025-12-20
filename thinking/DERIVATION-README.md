---
title: Documentation Derivation System - Overview
date: 2025-12-19
purpose: Quick reference guide for Loom documentation derivation
---

# Documentation Derivation System - Quick Reference

## 📚 Documents in this Package

### 1. **documentation-derivation-strategy.md**
**Comprehensive strategy document**

Contains:
- 4-level derivation hierarchy (L0→L1→L2→L3)
- Detailed derivation rules for every document type
- AI agent responsibility matrix
- Validation & quality gates
- Claude Code skills design
- Dependency graph
- Success metrics

**Use this for:** Understanding the complete derivation system, implementation planning

---

### 2. **derivation-example-walkthrough.md**
**Concrete end-to-end example**

Scenario: "Customer wants to cancel a quote"

Shows:
- Step-by-step derivation from L0 → L3
- Exact input/output for each AI agent
- Time estimates (5 min human input → 500+ lines of docs!)
- Complete test generation (10 tests with TDAI)
- Validation results
- 95% time savings

**Use this for:** Understanding how derivation works in practice, training examples

---

## 🎯 Quick Start

### For Implementers

1. **Read:** `documentation-derivation-strategy.md` (full specs)
2. **Study:** `derivation-example-walkthrough.md` (concrete example)
3. **Implement:** Claude Code skills for derivation
4. **Test:** Run example scenario to validate

### For Users

1. **Understand:** Read the walkthrough example first
2. **Try:** Use `/loom-generate` with natural language
3. **Review:** Approve AI-generated docs at each level
4. **Validate:** Run `/loom-validate` to check quality

---

## 🔄 Derivation Levels Summary

```
┌─────────────────────────────────────────────────────────┐
│ L0: FOUNDATIONAL (Human-Provided)                       │
│   ➜ domain-vocabulary.md, user-stories.md, NFRs        │
│   Human: 80% │ AI: 20%                                  │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│ L1: PRIMARY DERIVATION (AI from L0)                     │
│   ➜ domain-model.md, acceptance-criteria.md,           │
│     business-rules.md, bounded-context-map.md           │
│   Human: 20% (review) │ AI: 80%                         │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│ L2: SECONDARY DERIVATION (AI from L0+L1)                │
│   ➜ interface-contracts.md, sequence-design.md,        │
│     initial-data-model.md, aggregate-design.md          │
│   Human: 10% (optional) │ AI: 90%                       │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│ L3: TERTIARY DERIVATION (AI from L0+L1+L2)              │
│   ➜ test-case.md (TDAI!), feature-tickets.md,          │
│     service-boundaries.md, event-message-design.md      │
│   Human: 5% (test plan) │ AI: 95%                       │
└─────────────────────────────────────────────────────────┘
```

---

## 🤖 Key AI Agents

| Agent | Derivation | Input | Output | Approval |
|-------|------------|-------|--------|----------|
| **DomainModelAgent** | L0→L1 | vocabulary | domain-model.md | Required |
| **AcceptanceCriteriaAgent** | L0→L1 | user-stories | acceptance-criteria.md | Required |
| **BusinessRulesAgent** | L0→L1 | vocab + stories | business-rules.md | Optional |
| **InterfaceContractAgent** | L1→L2 | model + AC + rules | interface-contracts.md | Required |
| **SequenceDesignAgent** | L1→L2 | model + stories | sequence-design.md | Optional |
| **TestGeneratorAgent** | L2→L3 | AC + contracts | test-case.md | Test plan |
| **FeatureTicketAgent** | L2→L3 | stories + AC | feature-tickets.md | Optional |

---

## ✅ Validation Checklist

After derivation, validate:

- [ ] **Structural:** All sections present, YAML valid, IDs follow conventions
- [ ] **Traceability:** All links valid, bidirectional links consistent
- [ ] **Content:** No duplicate IDs, no contradictions, all references exist
- [ ] **Completeness:** All AC have tests, all US have AC, all entities covered
- [ ] **Test Quality:** ≥20% negative tests, 70:20:10 pyramid, hallucination tests

**Command:** `/loom-validate`

---

## 📊 Expected Benefits

### Time Savings

| Task | Manual | With Loom | Savings |
|------|--------|-----------|---------|
| Write acceptance criteria | 1-2 hours | 5 min (review) | 95% |
| Design API contracts | 2-3 hours | 10 min (review) | 90% |
| Create test cases | 3-4 hours | 10 min (test plan) | 95% |
| Update all related docs | 1-2 hours | 5 min | 95% |
| **Total per feature** | **7-11 hours** | **30 min** | **95%** |

### Quality Improvements

- ✅ **100% traceability** (every requirement → code → test)
- ✅ **90%+ hallucination detection** (TDAI with negative tests)
- ✅ **0% documentation drift** (AI validates consistency)
- ✅ **Comprehensive test coverage** (10+ tests per feature)

---

## 🚀 Implementation Roadmap

### Phase 1: Foundation (Week 1-2)
- [ ] Implement L0 → L1 derivations
- [ ] Basic validation (structure, traceability)
- [ ] Human approval workflow (Claude Code)
- [ ] Example project (TODO app)

### Phase 2: Advanced Derivation (Week 3-4)
- [ ] Implement L1 → L2 derivations
- [ ] Implement L2 → L3 derivations (TDAI!)
- [ ] Automated validation
- [ ] Dependency graph visualization

### Phase 3: Intelligence (Week 5-6)
- [ ] AI learns from corrections
- [ ] Confidence scoring (auto-approve)
- [ ] Incremental re-derivation
- [ ] Analytics dashboard

---

## 💡 Best Practices

### DO ✅
- Start with high-quality foundational docs (L0)
- Review and approve L1 derivations (high impact)
- Run validation after every derivation
- Use traceability to understand change impact
- Iterate on derivation rules based on feedback

### DON'T ❌
- Skip validation steps
- Auto-approve foundational derivations (L0→L1)
- Manually edit derived docs (re-derive instead!)
- Ignore derivation errors/warnings
- Break traceability links

---

## 📖 Related Documentation

### Core Concepts
- `bidirectional-traceability-design.md` - How traceability works
- `test-driven-ai-development.md` - TDAI methodology
- `claude-code-as-platform.md` - Platform architecture

### Evaluation
- `sonnett-evaluation-02.md` - Comprehensive system evaluation
- Overall AI-PDS score: **7.5/10**

---

## 🎓 Learning Path

**For new users:**
1. Read: `derivation-example-walkthrough.md` (30 min)
2. Try: Generate docs for a simple feature (30 min)
3. Study: `documentation-derivation-strategy.md` (1 hour)
4. Practice: Real project feature (2 hours)

**For implementers:**
1. Study: Full derivation strategy (2 hours)
2. Review: Example walkthrough code (1 hour)
3. Prototype: Single derivation (L0→L1) (4 hours)
4. Test: Run example scenario (2 hours)
5. Iterate: Improve based on results (ongoing)

---

## 🔗 Quick Links

- **Strategy Doc:** `documentation-derivation-strategy.md`
- **Example Walkthrough:** `derivation-example-walkthrough.md`
- **Evaluation:** `sonnett-evaluation-02.md`
- **Example Structure:** `ai-pds-specification/9000-appendix/9200-example-ai-pds/`

---

## 📞 Support

**Questions?**
- Review the example walkthrough for practical guidance
- Check the strategy doc for detailed specs
- Study the example AI-PDS structure for reference

**Issues?**
- Validation errors → Check traceability links
- Derivation quality → Review input docs (L0)
- AI hallucinations → Check test coverage (negative tests!)

---

*This derivation system is the heart of Loom (AI-PDS). It transforms simple human inputs into comprehensive, traceable, validated documentation with 95% time savings.*

*Ready to start? Begin with the example walkthrough!*

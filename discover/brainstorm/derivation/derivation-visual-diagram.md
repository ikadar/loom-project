---
title: Documentation Derivation - Visual Diagrams
date: 2025-12-19
purpose: Mermaid diagrams for derivation system visualization
status: active
note: Agent names (DomainModelAgent, etc.) are conceptual - see ADR-001 in documentation-derivation-strategy.md
---

# Documentation Derivation - Visual Diagrams

## 📊 Complete Derivation Flow

```mermaid
graph TB
    subgraph L0["LEVEL 0: FOUNDATIONAL (Human Input)"]
        DV[domain-vocabulary.md<br/>👤 Human: 2 min]
        US[user-stories.md<br/>👤 Human: 3 min]
        NFR[non-functional-requirements.md<br/>👤 Human: optional]
        PH[project-handbook/*<br/>👤 Human: one-time setup]
    end

    subgraph L1["LEVEL 1: PRIMARY DERIVATION (AI 80%)"]
        DM[domain-model.md<br/>🤖 AI → 👤 Review: 1 min]
        AC[acceptance-criteria.md<br/>🤖 AI → 👤 Review: 2 min]
        BR[business-rules.md<br/>🤖 AI → ✓ Auto-approved]
        BC[bounded-context-map.md<br/>🤖 AI → ✓ Auto-approved]
    end

    subgraph L2["LEVEL 2: SECONDARY DERIVATION (AI 90%)"]
        IC[interface-contracts.md<br/>🤖 AI → 👤 Review: optional]
        SD[sequence-design.md<br/>🤖 AI → ✓ Auto-approved]
        IDM[initial-data-model.md<br/>🤖 AI → ✓ Auto-approved]
        AD[aggregate-design.md<br/>🤖 AI → ✓ Auto-approved]
        TS[tech-specs.md<br/>🤖 AI → ✓ Auto-approved]
    end

    subgraph L3["LEVEL 3: TERTIARY DERIVATION (AI 95%)"]
        TC[test-cases.md<br/>🤖 AI → 👤 Test Plan: 3 min]
        API[openapi.json<br/>🤖 AI → ✓ Auto-approved]
        SKEL[implementation-skeletons.md<br/>🤖 AI → ✓ Auto-approved]
        FT[feature-tickets.md<br/>🤖 AI → ✓ Auto-approved]
        SB[service-boundaries.md<br/>🤖 AI → ✓ Auto-approved]
        EM[event-message-design.md<br/>🤖 AI → ✓ Auto-approved]
        DEP[dependency-graph.md<br/>🤖 AI → ✓ Auto-approved]
    end

    %% L0 → L1 Derivations
    DV -->|DomainModelAgent| DM
    DV -->|BusinessRulesAgent| BR
    US -->|AcceptanceCriteriaAgent| AC
    US -->|BusinessRulesAgent| BR
    DV -->|BoundedContextAgent| BC

    %% L1 → L2 Derivations
    DM -->|InterfaceContractAgent| IC
    AC -->|InterfaceContractAgent| IC
    BR -->|InterfaceContractAgent| IC
    DM -->|SequenceDesignAgent| SD
    US -->|SequenceDesignAgent| SD
    BR -->|SequenceDesignAgent| SD
    DM -->|DataModelAgent| IDM
    DM -->|AggregateDesignAgent| AD
    IC -->|AggregateDesignAgent| AD

    %% L2 → L3 Derivations
    AC -->|TestGeneratorAgent<br/>TDAI| TC
    IC -->|TestGeneratorAgent<br/>TDAI| TC
    BR -->|TestGeneratorAgent<br/>TDAI| TC
    US -->|FeatureTicketAgent| FT
    AC -->|FeatureTicketAgent| FT
    DM -->|ServiceBoundaryAgent| SB
    IC -->|ServiceBoundaryAgent| SB
    IC -->|EventMessageAgent| EM
    SD -->|EventMessageAgent| EM

    style L0 fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    style L1 fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style L2 fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    style L3 fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px

    style DV fill:#bbdefb
    style US fill:#bbdefb
    style TC fill:#c8e6c9
```

---

## 🔄 Derivation Pipeline (Linear View)

```mermaid
flowchart LR
    H1[👤 Human:<br/>domain-vocabulary.md<br/>2 min] --> A1[🤖 AI Agent:<br/>DomainModelAgent]
    H2[👤 Human:<br/>user-stories.md<br/>3 min] --> A2[🤖 AI Agent:<br/>AcceptanceCriteriaAgent]

    A1 --> D1[📄 domain-model.md]
    A2 --> D2[📄 acceptance-criteria.md]

    D1 --> R1[👤 Review:<br/>1 min]
    D2 --> R2[👤 Review:<br/>2 min]

    R1 --> A3[🤖 AI Agent:<br/>InterfaceContractAgent]
    R2 --> A3

    A3 --> D3[📄 interface-contracts.md]

    D3 --> A4[🤖 AI Agent:<br/>TestGeneratorAgent<br/>TDAI]
    D2 --> A4

    A4 --> D4[📄 test-case.md<br/>10 tests]

    D4 --> R3[👤 Review:<br/>Test Plan<br/>3 min]

    R3 --> F[✅ Complete!<br/>~500 lines of docs<br/>Total time: 10 min]

    style H1 fill:#bbdefb
    style H2 fill:#bbdefb
    style R1 fill:#fff9c4
    style R2 fill:#fff9c4
    style R3 fill:#fff9c4
    style F fill:#c8e6c9
```

---

## 🎯 Human Touch Points

```mermaid
gantt
    title Human Effort Distribution (10 min total)
    dateFormat  mm
    axisFormat  %M min

    section Human Input
    Write vocabulary           :a1, 00, 2m
    Write user stories         :a2, 02, 3m

    section AI Derivation
    AI generates L1 docs       :crit, a3, 05, 1m

    section Human Review
    Review acceptance criteria :a4, 06, 2m

    section AI Derivation
    AI generates L2 & L3       :crit, a5, 08, 1m

    section Human Review
    Review test plan           :a6, 09, 1m

    section Complete
    Validation                 :milestone, m1, 10, 0m
```

---

## 🧪 TDAI Test Generation Flow

```mermaid
flowchart TD
    AC["📄 Acceptance Criteria<br/>AC-QUOTE-003-1, AC-QUOTE-003-2, AC-QUOTE-003-3"]
    IC["📄 Interface Contracts<br/>POST /api/quotes/:id/cancel"]
    BR["📄 Business Rules<br/>BR-QUOTE-003: Status rules"]

    AC --> TG["🤖 TestGeneratorAgent<br/>TDAI Engine"]
    IC --> TG
    BR --> TG

    TG --> TP["📋 Test Plan<br/>10 tests total"]

    TP --> HA{"👤 Human<br/>Approve?"}

    HA -->|Yes| GEN["🤖 Generate All Tests"]
    HA -->|No| REV["🔄 Revise Plan"]

    REV --> TP

    GEN --> POS["✅ Positive Tests<br/>3 tests"]
    GEN --> NEG["❌ Negative Tests<br/>4 tests"]
    GEN --> BND["⚖️ Boundary Tests<br/>2 tests"]
    GEN --> HAL["🛡️ Hallucination Prevention<br/>1 test"]

    POS --> VAL["✓ Validation"]
    NEG --> VAL
    BND --> VAL
    HAL --> VAL

    VAL --> CHK{"Metrics OK?"}

    CHK -->|✓ Yes| DONE["✅ test-case.md<br/>10 tests generated"]
    CHK -->|✗ No| REG["🔄 Regenerate"]

    REG --> GEN

    style AC fill:#fff3e0
    style IC fill:#fff3e0
    style BR fill:#fff3e0
    style TG fill:#e1f5ff
    style TP fill:#fff9c4
    style DONE fill:#c8e6c9
    style HAL fill:#ffcdd2
```

---

## 🔗 Traceability Graph Example

```mermaid
graph LR
    subgraph Documentation
        DV["domain-vocabulary.md<br/>#quote-cancellation"]
        US["user-stories.md<br/>#us-quote-003"]
        AC1["acceptance-criteria.md<br/>#ac-quote-003-1"]
        AC2["acceptance-criteria.md<br/>#ac-quote-003-2"]
        BR["business-rules.md<br/>#br-quote-003"]
        API["interface-contracts.md<br/>#api-post-quote-cancel"]
    end

    subgraph Code
        QE["src/domain/Quote.ts<br/>ENT-Quote entity"]
        CM["src/domain/Quote.ts<br/>cancel method"]
    end

    subgraph Tests
        TC1["test-case.md<br/>#tc-quote-003-001"]
        TC2["test-case.md<br/>#tc-quote-003-002"]
        TC9["test-case.md<br/>#tc-quote-003-009"]
    end

    DV -->|defines| US
    US -->|derives| AC1
    US -->|derives| AC2
    AC2 -->|derives| BR
    AC1 -->|derives| API
    BR -->|constrains| API

    US -.->|@traceability| QE
    AC2 -.->|@implements| BR
    BR -.->|@implements| CM
    API -.->|implements| CM

    AC1 -->|generates| TC1
    AC2 -->|generates| TC2
    AC1 -->|hallucination check| TC9

    TC1 -.->|tests| CM
    TC2 -.->|tests| CM
    TC9 -.->|tests| CM

    style DV fill:#bbdefb
    style US fill:#bbdefb
    style QE fill:#c8e6c9
    style CM fill:#c8e6c9
    style TC1 fill:#fff9c4
    style TC2 fill:#fff9c4
    style TC9 fill:#ffcdd2
```

---

## 📊 Time Savings Breakdown

```mermaid
pie title Time Savings: Manual vs Loom
    "Loom (AI + Review)" : 10
    "Manual Work Eliminated" : 390
```

**Manual:** 400 minutes (6.7 hours)
**With Loom:** 10 minutes
**Savings:** 390 minutes (95%)

---

## 🏗️ Architecture: AI Agents

```mermaid
flowchart TB
    subgraph User Interface
        CC[Claude Code<br/>Natural Language Input]
    end

    subgraph Orchestrator
        DO[DerivationOrchestrator<br/>Manages workflow]
    end

    subgraph AI Agents
        DMA[DomainModelAgent]
        ACA[AcceptanceCriteriaAgent]
        BRA[BusinessRulesAgent]
        ICA[InterfaceContractAgent]
        SDA[SequenceDesignAgent]
        TGA[TestGeneratorAgent<br/>TDAI]
    end

    subgraph Validators
        SV[StructuralValidator]
        TV[TraceabilityValidator]
        CV[ConsistencyValidator]
        QV[TestQualityValidator]
    end

    subgraph Storage
        FS[File System<br/>Markdown + YAML]
    end

    CC -->|/loom-generate| DO
    DO -->|L0→L1| DMA
    DO -->|L0→L1| ACA
    DO -->|L0→L1| BRA
    DO -->|L1→L2| ICA
    DO -->|L1→L2| SDA
    DO -->|L2→L3| TGA

    DMA -->|validate| SV
    ACA -->|validate| SV
    BRA -->|validate| SV
    ICA -->|validate| SV
    SDA -->|validate| SV
    TGA -->|validate| QV

    SV -->|validate| TV
    TV -->|validate| CV

    CV -->|✓ pass| FS
    CV -->|✗ fail| DO

    style CC fill:#e3f2fd
    style DO fill:#fff3e0
    style TGA fill:#c8e6c9
    style FS fill:#f3e5f5
```

---

## 🎓 Learning Curve

```mermaid
journey
    title Loom Learning Journey
    section Day 1
      Read walkthrough example: 5: User
      Try first feature generation: 4: User
      Review AI-generated docs: 3: User
    section Week 1
      Understand derivation levels: 4: User
      Practice with real features: 5: User
      Master approval workflow: 4: User
    section Month 1
      Customize derivation rules: 5: User
      Optimize validation: 4: User
      Full productivity: 5: User
```

---

## 📈 Quality Metrics Dashboard

```mermaid
graph TB
    subgraph Metrics
        M1[Derivation Coverage<br/>✓ 100%]
        M2[Traceability Accuracy<br/>✓ 100%]
        M3[Test Quality<br/>✓ Negative tests: 50%<br/>✓ Pyramid: 70:20:10]
        M4[Hallucination Detection<br/>✓ 90%+]
        M5[Time Savings<br/>✓ 95%]
    end

    M1 --> PASS[✅ All Metrics Pass]
    M2 --> PASS
    M3 --> PASS
    M4 --> PASS
    M5 --> PASS

    style PASS fill:#c8e6c9
    style M1 fill:#e1f5ff
    style M2 fill:#e1f5ff
    style M3 fill:#e1f5ff
    style M4 fill:#e1f5ff
    style M5 fill:#e1f5ff
```

---

*These visual diagrams illustrate the Documentation Derivation System in Loom (AI-PDS). Use them for presentations, documentation, and training materials.*

*Render these diagrams in any Mermaid-compatible viewer (GitHub, Mermaid Live Editor, VS Code, etc.)*

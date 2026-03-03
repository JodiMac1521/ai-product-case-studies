# Building Trust Through Better Scoping

How a human-in-the-loop AI system scaled revenue without scaling headcount.

---

## The Constraint

Revenue growth was constrained by a 3–5 day manual scoping bottleneck tied directly to operations headcount.

At 6 new projects per week, we hit a capacity ceiling of ~$400K ARR.  
To reach $1M ARR, we needed 12+ projects per week.

We had two options:

**Option A:** Hire 3 additional operations staff  
**Option B:** Build infrastructure

We chose infrastructure.

---

## The Decision

Build a human-in-the-loop AI Scoping Agent instead of expanding the operations team.

Why:

- Preserve margin
- Avoid fixed-cost risk (bootstrap survival constraint)
- Create proprietary IP
- Improve both speed and trust
- Unlock scalable capacity

The goal was not full automation.

The goal was **controlled automation.**

---

## System Architecture: Human-in-the-Loop Model

```mermaid
flowchart LR
    A[Customer Brief Submission] --> B[AI Scope Draft Generated]
    B --> C[Ops Review & Refinement]
    C --> D[Polished Scope Delivered to Customer]
    D --> E[Project Execution]
    E --> F[Outcome & Performance Data]
    F --> G[Template Refinement + Prompt Improvement]
    G --> B

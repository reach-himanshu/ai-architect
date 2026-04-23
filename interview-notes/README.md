# 07 — Interview Notes

> **Goal:** Consolidate everything into interview-ready notes — system design frameworks,
> trade-off discussions, behavioural stories, and a rapid-recall cheatsheet.

---

## Structure

```
07-interview-notes/
├── README.md
├── 01_ai_system_design_cases.md   ← 8 full system design walk-throughs
├── 02_tradeoff_discussions.md     ← ML/architecture trade-offs with structured answers
├── 03_behavioral_prep.md          ← STAR stories, leadership principles, common questions
└── 04_key_concepts_cheatsheet.md  ← Quick-reference tables for interviews
```

---

## How to Use This Folder

### 1. Daily Review (15 min/day)

*   Week 1: System design cases (01)
*   Week 2: Trade-off discussions (02)
*   Week 3: Behavioural stories (03)
*   Week 4: Full cheatsheet review (04)

### 2. Before Each Interview

*   Read the relevant system design case + the cheatsheet.
*   Practice saying the trade-off answers out loud (not just reading them).
*   Review 2–3 STAR stories from (03).

### 3. During the Interview

*   **System design:** Use the RADIO framework (Requirements → API → Data → Implementation → Optimisation).
*   **Trade-offs:** Always present 2+ options with pros/cons before recommending one.
*   **Behavioural:** Use STAR (Situation, Task, Action, Result) with concrete numbers.

---

## Key Themes for AI Architect Interviews

| Theme | Key Questions |
|-------|-------------|
| LLMs & RAG | Design a RAG system; how do you handle hallucinations? |
| ML Platform | Feature store design; model registry; CI/CD for ML |
| System Design | Design a recommendation system; fraud detection |
| Data Engineering | Design a real-time feature pipeline |
| Reliability | How do you handle model degradation in production? |
| Trade-offs | Batch vs streaming; SQL vs NoSQL; fine-tuning vs RAG |
| Leadership | Tell me about a time you drove a technical decision |

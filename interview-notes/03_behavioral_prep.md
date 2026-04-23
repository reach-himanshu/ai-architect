# Behavioural Interview Preparation

## The STAR Framework

**S**ituation — Set the context (1–2 sentences).
**T**ask — What was your specific responsibility?
**A**ction — What did YOU do? (Use "I", not "we")
**R**esult — Quantify the outcome.

**Golden rule:** Every story should have a number in the Result.

---

## Core Behavioural Categories

### 1. Technical Leadership / Driving Technical Decisions

**Template story:**

*   **Situation:** Team was debating between X and Y for [system]. No consensus after 2 weeks.
*   **Task:** As the technical lead, I needed to drive alignment and move us forward.
*   **Action:** I created a structured comparison (latency, cost, team expertise, operational burden). I ran a 2-day spike to validate assumptions. I facilitated a 1-hour decision meeting with data.
*   **Result:** We aligned on [choice] within the week. The system launched on time and [metric improved by N%].

**Questions this fits:**

*   "Tell me about a time you had to make a difficult technical decision."
*   "Describe a situation where you influenced without authority."
*   "How do you handle technical disagreements?"

---

### 2. Handling Failure / Lessons Learned

**Template story:**

*   **Situation:** We shipped [feature/model] to production and [metric degraded / incident occurred].
*   **Task:** I was responsible for the root cause analysis and recovery.
*   **Action:** I coordinated the on-call response. I identified the root cause as [X]. I implemented [fix]. I added monitoring for [Y] to prevent recurrence. I wrote a blameless post-mortem.
*   **Result:** Service restored within [N] hours. New monitoring caught similar issues 3 times in the following month. We reduced MTTR by 40%.

**Questions this fits:**

*   "Tell me about a time something went wrong in production."
*   "What's the biggest mistake you've made? What did you learn?"
*   "Describe a time you failed to meet a deadline."

---

### 3. Working with Ambiguity

**Template story:**

*   **Situation:** The product team asked us to "make recommendations better" with no specific metric or constraint.
*   **Task:** I needed to define success and build an execution plan.
*   **Action:** I interviewed 5 PMs and 3 business stakeholders to understand the real problem. I discovered the actual issue was recommendation diversity, not relevance. I proposed 2 experiments with clear success metrics (diversity@10, CTR). I got alignment in one meeting.
*   **Result:** We shipped the right experiment (not the most technically complex one). CTR improved 8%. Saved the team 3 weeks of building the wrong thing.

**Questions this fits:**

*   "How do you handle ambiguous requirements?"
*   "Tell me about a time you proactively identified a problem."
*   "How do you prioritise competing demands?"

---

### 4. Cross-Functional Collaboration

**Template story:**

*   **Situation:** ML team wanted to retrain the model weekly; data engineering team said it would require a new pipeline that would take 3 months to build.
*   **Task:** Bridge the gap between the two teams and find a path forward.
*   **Action:** I facilitated a joint working session. We identified that incremental fine-tuning (not full retraining) would meet the ML team's accuracy goal with a 2-week pipeline instead of 3 months. I wrote the joint proposal.
*   **Result:** Shipped in 3 weeks. Model freshness improved from monthly to weekly. Both teams adopted the shared pipeline for other models.

---

### 5. Mentoring / Growing Others

**Template story:**

*   **Situation:** A junior engineer was struggling to move from notebooks to production-ready code.
*   **Task:** Help them level up without doing the work for them.
*   **Action:** I set up weekly 1:1s with a structured agenda: code review, architecture discussion, and one new concept per session. I assigned them as the primary owner of a low-risk feature.
*   **Result:** Within 2 months they independently shipped their first production model. They presented the architecture in a team review. They received a positive performance review.

---

## Common AI Architect Questions

### Technical Deep-Dives

*   "Walk me through how RAG works. What are its failure modes?"
*   "How would you design a feature store from scratch?"
*   "What's the difference between batch and online serving? When do you use each?"
*   "How do you detect and handle data drift in production?"
*   "Explain how you'd handle LLM hallucinations in a production system."

**Answering framework:**

1.  Define the concept clearly.
2.  Give a concrete example.
3.  Identify the trade-offs.
4.  Describe how you've implemented it (or would implement it).

---

### System Design

*   "Design a real-time recommendation system."
*   "Design a platform for training and deploying ML models."
*   "How would you build a search system with semantic understanding?"

**Answering framework (RADIO):**

1.  **Requirements:** Ask clarifying questions (scale, latency, accuracy trade-offs).
2.  **API:** Define the interface (inputs, outputs).
3.  **Data:** Schema, storage, pipeline.
4.  **Implementation:** Architecture diagram, component walk-through.
5.  **Optimisation:** Scale, cost, reliability improvements.

---

### Leadership / Culture

*   "How do you balance technical debt vs new features?"
*   "How do you evaluate engineers on your team?"
*   "What does good engineering culture look like to you?"

**Tips:**

*   Be specific — avoid generic answers like "I believe in communication."
*   Reference real principles (blameless post-mortems, make it reversible, working backwards).
*   Show you've thought about failure modes of your own approach.

---

## Pre-Interview Checklist

```
Research
 [ ] Read the company's engineering blog (last 6 months)
 [ ] Understand their AI/ML stack (job descriptions are a signal)
 [ ] Know their products — what ML problems do they likely have?

Preparation
 [ ] 3 strong STAR stories ready (technical decision, failure, impact)
 [ ] 3 system designs practiced out loud (not just in your head)
 [ ] Key trade-offs from 02_tradeoff_discussions.md reviewed

Day-of
 [ ] 5-min warm-up: say your intro out loud
 [ ] Have a notepad — draw diagrams during system design
 [ ] Prepare 3 thoughtful questions for the interviewer
```

---

## Questions to Ask the Interviewer

*   "What does the data stack look like today, and what are the biggest pain points?"
*   "How does the team balance research/experimentation vs production reliability?"
*   "What does the on-call rotation look like for ML systems?"
*   "How are model performance decisions made — who owns the go/no-go?"
*   "What does success look like in the first 90 days?"

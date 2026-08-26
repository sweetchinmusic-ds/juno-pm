# AI Strategy One-Pager - Juno Automated Prioritization

## 1. Problem & Workflow

The Problem: resolving P0 escalations at RocketShip is slow, evidence-scattered, and inconsistent across PMs—roadmap decisions are driven by the loudest voice rather than synthesized data.

Prevention: explicitly prevents 'arbitrary prioritization' - a PM ranking features based on exec pressure or competitor moves rather than a cross-referenced audit of Slack escalations, Jira tickets, and customer evidence.

## 2. Target Metrics

Time to Triage P0: reduction in time-to-decision from 2 hours to 15 minutes.

Leadership proof: 80% PM confidence in prioritization decisions - PMs accept Juno's ranked backlog more often than their own manual synthesis, proving the logic is evidence-backed and defensible.

## 3. Autonomy Level

Choice: Copilot. AI synthesizes evidence and drafts the ranked backlog with citations; a human PM must review and approve before publish, especially for decisions with <70% confidence.

Explicitly avoiding: Agent. We will not give AI end-to-end autonomy to publish externally (Slack, email, Intercom) or act on low-confidence decisions without a safety valve - a single hallucinated customer name in a P0 triage is a trust-erosion event hard to undo.

## 4. Data & Model Approach

Approach: Ground (RAG). Ground the model in RocketShip's corpus: Slack #escalations (P0/P1, 90 days), Jira ROCKET project (6 months), and Notion Product workspace (12 months) to verify customer signals with citations.

Explicitly avoiding: generic LLM (Buy). Using a general model without RAG grounding would lead to hallucinations of generic enterprise risks (compliance, scalability) that don't cite RocketShip's actual escalation threads or ticket patterns.

## 5. Risks & Mitigations

Risk: if the training data for past human prioritization favoured loud customers with high ARR, the AI might instinctively penalise smaller customers in any ranking - a 'one-way door' where smaller customers churn and structural bias compounds.

Mitigation: a hard eval gate where 5% of all AI-ranked backlogs are blind-reviewed by an independent PM to ensure parity between high-ARR and low-ARR customer issues. V1 runs in ARR-blind mode to avoid this failure.

## 6. V1 Scope

In: P0/P1 escalations in Slack #escalations with clear customer signals and retrievable context from Jira/Notion.

Out: (1) legal/contract decisions, (2) decisions requiring ARR data not in the corpus. Both require human PM judgment that the model cannot reliably reproduce without hallucination risk.

# AI Strategy One-Pager - Juno Automated Prioritization

## 1. Problem & Workflow

The Problem: resolving host-guest damage claims is slow, emotionally charged, and inconsistent across support agents.

Prevention: explicitly prevents 'arbitrary settlement' - a human agent issuing a refund or charge based on gut feeling rather than a cross-referenced audit of house rules and check-in photos.

## 2. Target Metrics

Resolution Cycle Time: reduction in time-to-closed-claim from 72 hours to 15 minutes.

Leadership proof: 20% reduction in appeal rates - both guests and hosts accept the AI's first decision more often than a human's, proving the logic is fairer and more defensible.

## 3. Autonomy Level

Choice: Copilot. AI reviews evidence and drafts the decision letter with a recommended payout; a human supervisor must click 'approve' for any transaction over $500.

Explicitly avoiding: Agent. We will not give AI end-to-end autonomy to move money out of user bank accounts without a safety valve - a single error in damage assessment is a trust-erosion event hard to undo.

## 4. Data & Model Approach

Approach: Ground (RAG). Ground the model in the specific house rules of that listing and the metadata from uploaded photos (timestamps/location) to verify when damage occurred.

Explicitly avoiding: generic LLM (Buy). Using a general model without RAG grounding would lead to hallucinations of standard policies that might contradict the host's specific legal house rules.

## 5. Risks & Mitigations

Risk: if the training data for past human settlements favoured Power Hosts, the AI might instinctively penalise new guests in any dispute - a 'one-way door' where guests stop using the platform and structural bias compounds.

Mitigation: a hard eval gate where 5% of all AI-mediated decisions are blind-reviewed by a third-party legal team to ensure parity between guest and host outcomes.

## 6. V1 Scope

In: claims under $1,000 involving physical property damage with photo evidence.

Out: (1) personal injury claims, (2) disputes involving local noise ordinances. Both require human judgement that the model cannot reliably reproduce.

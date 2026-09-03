# Juno — AI Associate PM for RocketShip

> Juno turns messy P0/P1 escalation threads across Slack, Jira, and Notion into the top-3 cited risk signals — cutting P0 triage from ~2 hours to 15 minutes, with every risk grounded in a source and a strategy clause.

_Devika Sadekar + August 17 cohort_

Repo: https://github.com/sweetchinmusic-ds/juno-pm

This repo is my final project for the AI Product Management Certification — **Juno**. Each module’s artefact lives in its own folder; this README is the dashboard and the pitch.

---

## Module artefacts

### M1 · Prompting
- **System prompt** — [`01-prompting/system-prompt.md`](01-prompting/system-prompt.md)
- **Prototype** — https://lovable.dev/projects/d89ac9db-9f54-484e-a9da-54818f16676c

### M2 · Strategy
- **Decision matrix** — [`02-strategy/decision-matrix.md`](02-strategy/decision-matrix.md)
- **AI Strategy one-pager** — [`02-strategy/strategy-one-pager.md`](02-strategy/strategy-one-pager.md)

### M3 · RAG / AI PRD
- **AI PRD** — [`03-rag-prd/prd.md`](03-rag-prd/prd.md)

### M4 · AI-Native UX
- **AI user flow** — [`04-ai-ux/user-flow.md`](04-ai-ux/user-flow.md)
- **Trust-gap mitigations** — [`04-ai-ux/trust-gaps.md`](04-ai-ux/trust-gaps.md)

### M5 · Agentic Workflows
- **Agent Workflow Spec (AWSpec)** — [`05-agentic-workflows/awspec.md`](05-agentic-workflows/awspec.md)
- **Agent Control Panel** — [`05-agentic-workflows/agent-control-panel.md`](05-agentic-workflows/agent-control-panel.md)

### M6 · Evals and Guardrails
- **Eval stack** — [`06-evals/eval-stack.md`](06-evals/eval-stack.md)
- **Human evaluation rubric** — [`06-evals/human-rubric.md`](06-evals/human-rubric.md)

---

## PM Execution Plan

### Where Juno is today
- M1–M6 specced and committed.
- Lovable prototype validates the M1 triage flow.
- Automated evals not yet wired (format / citation / refusal / fail-safe checks + LLM-judge harness defined; cron stub only). Human rubric drafted, graders not yet staffed.

### What ships next (next 2 sprints)
- Sprint 1: wire the eval harness (format / citation / refusal / fail-safe checks in CI + nightly cron); build the 200-thread golden set; staff 2 graders + PM tiebreak.
- Sprint 2: open beta with 3 on-call PMs; weekly Friday rubric batch; instrument user-feedback signals (thumbs, regenerate, edit-before-send, abstention-override).

### What I watch (dashboards)
- Daily: thumbs-down rate, regenerate rate (≤15%), edit-before-send / override rate (≤30%), abstention-override rate (≤10%).
- Weekly: human-rubric mean per dimension (Accuracy, Citation grounding, Safety ≥4.0/5); refusal hit-rate; median time-to-first-action (≤15 min); cost per run.
- Per release: golden-set accuracy (≥90%).

### Red lines (what blocks shipping)
- Any "1" on Safety in human eval (PII surfaced or contractual language verbatim).
- Any "1" on Citation grounding, or an automated citation-check fail (fabricated / unresolvable citation).
- Any cross-team data leakage (team_id namespace breach) — 0% tolerance.
- <90% golden-set accuracy.
- Fail-safe breach: a ranking produced on <3 retrieved chunks.
- Any PII leakage in the last 30 days (0% tolerance).
- Cost >$0.50 / run.

### Governance
- Compliance: per-team team_id namespace isolation (no cross-team retrieval leakage); draft-only output to #pm-daily / #pm-juno-review — Juno never auto-posts externally or writes to source systems.
- Safety: 0% PII leakage (hard gate); contracts / legal language → refusal + human escalation; <3-chunk fail-safe (insufficient-evidence banner, no ranking).
- Reliability: p95 ≤4s target, 30s hard cap; single 70% confidence gate (sub-70% items labelled "review before acting"); dual-citation (one strategy clause + one evidence source) on every risk.
- Reputation: ARR-blind — risks ranked on severity, never revenue; every claim cited; Juno abstains rather than guesses; weekly human rubric keeps quality honest.

---

## Build Insights

- **Friction point.** The hard part wasn't defining "good" — it was making "good" measurable. My first anchors leaned on words like "poorly" and "hedges appropriately," the kind of language everyone reads differently. Rewriting each into something you can actually point to — "cites a message that doesn't exist," "ranks a P2 above a P0" — was slow, humbling work, and it's where the rubric finally became real.
- **Key learning.** Precision is the lever: tightening one field — a numeric pass bar, a named owner, a cadence — made the downstream artifacts snap into consistency, while vague prose let contradictions hide across files.
- **Aha moment.** The eval rubric, not the PRD, is where the product actually gets decided — writing observable anchors and a hard gate forces choices vague PRD prose lets you skate past.

---

_Certification submission — AI Product Management Certification._

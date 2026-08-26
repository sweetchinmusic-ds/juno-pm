# AI PRD · Juno PM

> Module 3 · RAG / AI PRD. Juno's AI product requirements document with explicit RAG architecture.

---

## Problem & user

**Problem:** PMs at RocketShip are drowning in Signal Collapse. P0 escalations stack up in Slack #escalations, thousands of tickets sit unread in Jira, Notion product docs drift out of sync, and sales velocity stalls because roadmap decisions are driven by the loudest voice rather than synthesized evidence. The PM team is the bottleneck, and there is zero headcount budget.

**User:** Product Managers at RocketShip (B2B SaaS for Enterprise Data Teams) who need to synthesize multi-channel noise into evidence-backed decisions, draft "Version 0.1" specs faster, and flag risks they would otherwise miss.

---

## Solution overview

**Juno PM** is an AI Associate PM embedded in RocketShip's existing tools (Slack, Notion, Jira). It operates at three autonomy levels across three pillars:

1. **Synthesize Insights** (Execute autonomy): Transform raw Slack threads, Jira tickets, and Notion docs into structured summaries with citations. Low-risk, high-volume.
2. **Draft Specs** (Draft autonomy): Generate "Version 0.1" PRDs from synthesized findings. Human PM edits and approves before publish.
3. **Prioritize Risks** (Suggest autonomy): Flag unclear edge cases, technical debt, and risky assumptions. Human PM reviews before acting.

Juno is a **Copilot**, not an Agent. It drafts and suggests; humans approve high-stakes decisions.

---

## Retrieval requirements (RAG)

### Dual-Mode Architecture

Juno operates in **two modes** depending on whether a strategy document is provided:

| Mode | Trigger | Priority Basis | Use Case |
|------|---------|----------------|----------|
| **Strategy Mode** | Strategy document uploaded | Strategic alignment with company pillars (P0/P1/P2/P3) | Production use: PMs upload RocketShip strategy, Juno aligns requests to actual business priorities |
| **Quality Mode** | No strategy document | Request quality signals (0-100 score → P1/P2/P3) | Safety net: If strategy isn't loaded, Juno still works (detects anti-patterns, scores on clarity/evidence/specificity) |

**Rationale:** Strategy documents change (H1 → H2 strategy refresh, new pillars). Quality Mode ensures Juno remains useful even when strategy is out of sync or missing. It's a **graceful degradation** pattern, not a failure mode.

**Quality Signals (Quality Mode only):**
| Signal | Weight | What it measures |
|--------|--------|------------------|
| Problem Clarity | 0-25 | Clear user problem vs. solution-first thinking |
| Evidence Quality | 0-25 | Customer data/quotes vs. assumptions |
| Requirement Specificity | 0-25 | Estimable vs. vague scope |
| Anti-Pattern Check | 0-25 | Absence of red flags (competitor-driven, vanity, arbitrary deadlines) |

**Priority mapping (Quality Mode):**
- 80-100 points → **P1** (high quality request)
- 50-79 points → **P2** (medium quality)
- 20-49 points → **P3** (low quality)
- 0-19 points → **P3 + `notRecommended: true`**

**Strategy Mode behavior:**
- Priority (P0/P1/P2/P3) determined by **alignment** with strategy pillars (not quality)
- `strategicPillar` extracted from strategy document (e.g., "Signal-to-Clarity Acceleration")
- `strategicAlignment` scored 0-100 (how well request supports the pillar)
- `strategicRationale` cites strategy document sections
- Anti-patterns flagged **from the strategy doc** (company-specific, not generic)

**Warning when strategy missing:** Juno displays: *"⚠️ No strategy document loaded. Priorities reflect request quality, not strategic alignment."*

### Sources

Juno retrieves from three RocketShip data sources:

1. **Slack #escalations channel**
   - All threads tagged `P0` or `P1`
   - Preserves message timestamps, author metadata, thread structure
   - Indexed: last 90 days of history

2. **Notion: RocketShip Product workspace**
   - Past PRDs, product strategy docs, retrospectives, launch post-mortems
   - Pages tagged `#product` or in `/Product/PRDs/` folder
   - Indexed: all pages updated in last 12 months

3. **Jira: ROCKET project**
   - All tickets (open and closed)
   - Includes: summary, description, priority, status, assignee, comments
   - Indexed: last 6 months of activity

**Excluded sources:** Customer contracts, ARR data (requires PM to provide manually), legal docs, HR/internal ops.

### Chunking / indexing

| Source | Chunking strategy | Rationale |
|--------|-------------------|-----------|
| **Slack threads** | By thread (one thread = one or more chunks), max 800 tokens per chunk | Preserve conversational context; threads are semantic units |
| **Notion pages** | By section heading (H2/H3), ~1000 tokens per chunk | PRDs have clear sections (Problem, Solution, Scope); retrieval should return relevant sections, not full docs |
| **Jira tickets** | One ticket = one chunk (summary + description + top 3 comments), ~600 tokens | Tickets are atomic units; metadata (priority, status) preserved as filters |

**Embedding model:** OpenAI `text-embedding-3-large` (3072 dimensions, better retrieval accuracy than `ada-002`)

**Vector store:** pgvector (PostgreSQL extension), one namespace per RocketShip team for tenant isolation

### Grounding rule

**No answer without a cited source.**

- Every insight Juno returns must include a source citation:
  - Slack: message timestamp + thread link
  - Notion: page title + section anchor link
  - Jira: ticket ID (e.g., `ROCKET-1234`)
- If confidence score <70%, Juno must escalate to human PM with reasoning: `"Low confidence (62%). Suggest @pm-oncall review before acting."`
- If retrieval returns zero relevant chunks, Juno responds: `"No relevant context found in Slack/Notion/Jira. Suggest manual investigation or ask a clarifying question."`

### Freshness

How current the data must be, by source:

| Source | Refresh cadence | Rationale |
|--------|-----------------|-----------|
| **Slack #escalations** | Every 15 minutes | P0s require near-real-time awareness |
| **Jira tickets** | Every 1 hour | Tickets update frequently; 1hr lag acceptable for prioritization |
| **Notion product docs** | Every 4 hours | PRDs change less often; 4hr lag acceptable |

**Freshness guarantee:** Juno shows `"Data current as of [timestamp]"` with every response.

**Staleness alarm:** If Slack data is >30 minutes old, show warning: `"⚠️ Escalations data may be stale. Last synced: [timestamp]."`

### Retrieval strategy

**Hybrid search:** Vector similarity (semantic) + keyword matching (exact match on ticket IDs, customer names, feature names)

**Top-k:** Retrieve 10 chunks per query

**Reranker:** ON (Cohere `rerank-v3`), rerank top-20 → top-10 for improved precision

**Latency target:** p95 ≤4 seconds (from query to final LLM response)

**Query rewriting:** If user query is ambiguous (e.g., "What's the status?"), Juno asks clarifying questions before retrieval: `"Which area? (Escalations / Open tickets / Recent PRDs)"`

**Metadata filters (pre-retrieval):**
- Slack: filter by date range, P0/P1 tag
- Jira: filter by priority (P0/P1), status (Open/In Progress)
- Notion: filter by tags (`#product`, `#prd`)

---

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|-------------|----------|---------------------|
| 1 | Juno must retrieve context from Slack #escalations, Notion Product workspace, and Jira ROCKET project | **Must** | Retrieval returns chunks from all 3 sources when relevant; metadata preserved |
| 2 | Every insight must cite source (Slack timestamp, Notion page, Jira ticket ID) | **Must** | 100% of responses include at least one citation; no uncited claims |
| 3 | Juno must refuse to answer if confidence <70% and escalate to human PM | **Must** | Confidence threshold enforced; escalation message includes reasoning |
| 4 | Data freshness: Slack ≤15min, Jira ≤1hr, Notion ≤4hr | **Must** | Timestamps shown; staleness alarm triggers if Slack >30min old |
| 5 | Retrieval latency p95 ≤4 seconds | **Must** | Measured in eval logs (M6); 95% of queries return in ≤4s |
| 6 | Juno must ask clarifying questions if query is ambiguous | **Should** | Ambiguous queries (e.g., "What's happening?") trigger clarification prompt |
| 7 | Hybrid search: vector + keyword matching (ticket IDs, customer names) | **Should** | Keyword matches boost relevance; exact ticket IDs always retrieved |
| 8 | Per-team vector namespace for tenant isolation | **Must** | Team A cannot retrieve Team B's Slack/Jira/Notion data |
| 9 | Graceful degradation if vector DB is down | **Must** | Fallback message: "Unable to retrieve context. Escalating to human PM." No crash. |
| 10 | Reranker (Cohere rerank-v3) improves top-10 precision | **Should** | Eval shows reranker improves retrieval accuracy by ≥15% vs. no reranker (M6 baseline) |

---

## Out of scope

**V1 explicitly excludes:**

1. **Publishing externally without PM approval** — Juno drafts only. No auto-send to Slack, email, or Intercom.
2. **Acting on <70% confidence** — Low-confidence decisions must escalate, not auto-execute.
3. **Customer contracts, ARR data, legal docs** — Juno does not index these. PM must provide manually if needed.
4. **Cross-team data access** — Per-team tenancy enforced. Team A's Juno cannot see Team B's Slack/Jira/Notion.
5. **Real-time Slack streaming** — 15-minute refresh is acceptable for P0 triage; true real-time (websocket) is out of scope.
6. **Jira ticket creation/editing** — V1 is read-only. Juno suggests, PM executes in Jira.
7. **Notion page editing** — V1 is read-only. Juno drafts PRDs, PM copies into Notion and owns the doc.

---

## Failure modes + guardrails

| # | Failure mode | Impact | Guardrail |
|---|--------------|--------|-----------|
| 1 | **Hallucinated customer impact** — Juno invents ARR numbers, contract details, or customer names not in corpus | High (PM makes wrong prioritization call, damages trust with customer) | **Citation requirement** — every claim must link to Slack/Jira/Notion source. If ARR needed, Juno asks PM: "ARR data not found. Please provide ARR sheet or confirm manually." **Confidence threshold** — if <70%, escalate instead of claiming. |
| 2 | **Stale data in critical decisions** — Juno retrieves 2-hour-old Slack data; PM misses a P0 that escalated 10 minutes ago | Medium (delayed response to urgent escalation) | **Freshness SLA** — Slack refreshes every 15 minutes. **Staleness alarm** — if Slack data >30min old, show warning: "⚠️ Data may be stale. Last synced: [timestamp]." **Timestamp display** — every response shows "Data current as of [timestamp]." |
| 3 | **Privacy leak across teams** — Team A's Juno retrieves Team B's confidential escalation thread | Critical (regulatory, trust collapse) | **Per-team vector namespace** — pgvector namespaces enforce tenant isolation. **Pre-retrieval filtering** — user context (team ID) filters vector search before retrieval. **Audit log** — every retrieval logs (user, team, query, sources retrieved) for compliance review (M6). |
| 4 | **Retrieval failure (vector DB down)** — pgvector crashes, Juno cannot retrieve context | Medium (Juno offline until DB restored) | **Graceful degradation** — if retrieval fails, return: "Unable to retrieve context from Slack/Jira/Notion. Escalating to human PM." Do not hallucinate or guess. **Health check** — Juno pings vector DB every 60 seconds; if down >5 minutes, post alert to #pm-juno-ops. |
| 5 | **Ambiguous query, wrong retrieval** — PM asks "What's the status?" → Juno retrieves irrelevant Jira tickets instead of recent escalations | Low (PM wastes time, re-asks) | **Clarifying questions** — if query lacks specificity, Juno asks: "Which area? (Escalations / Open tickets / Recent PRDs)" before retrieval. **Query rewriting** — expand vague queries into structured retrieval filters. |
| 6 | **Chunk boundary cuts critical context** — A Notion PRD's "Out of Scope" section is split mid-sentence across two chunks; Juno retrieves first chunk only, misses key exclusion | Medium (PM drafts spec with wrong scope assumptions) | **Overlap chunking** — Notion chunks overlap by 100 tokens to preserve sentence context across boundaries. **Section-aware chunking** — chunk by H2/H3 headings, not arbitrary token counts, so sections stay intact. |
| 7 | **Bias amplification** — Historical Jira tickets prioritized "loud customers" with high ARR; Juno learns to rank their issues higher, ignoring smaller customers with real bugs | High (unfairness, smaller customers churn) | **Diversity sampling (future)** — retrieval should sample across customer segments, not just highest-similarity chunks. **Human review of prioritization** (M6 eval) — PMs spot-check Juno's top-3 risks weekly for bias. **ARR-blind mode** — V1 does not retrieve ARR data to avoid this failure mode. |

---

## Eval plan (stub, M6 fills this)

**Golden set:** 50 real P0 threads from RocketShip history, with human-labeled ground truth (top-3 risks, citations, confidence scores)

**Eval dimensions:**
1. **Retrieval accuracy** — Are the right chunks retrieved? (precision@10, recall@10)
2. **Citation correctness** — Do citations link to real sources? (100% pass rate required)
3. **Risk ranking quality** — Does Juno's top-3 match PM's top-3? (human eval, M6)
4. **Latency** — p95 ≤4s? (instrumented in Langflow eval-log node)
5. **Confidence calibration** — Is <70% confidence actually unreliable? (M6 analysis)

**Regression cadence:** Run golden set eval before every deploy. Block deploy if citation correctness <100% or retrieval accuracy drops >10% vs. baseline.

**See M6 for full eval stack, human rubric, and release gates.**

---

## Success metrics (30-day measurable)

| Metric | Baseline (today) | Target (30 days post-launch) | How measured |
|--------|------------------|------------------------------|--------------|
| **Time to triage P0** | 2 hours (manual PM read + synthesis) | 15 minutes (Juno synthesis + PM review) | Slack timestamp (P0 posted) → PM decision logged in Jira |
| **PM confidence in prioritization** | 60% (survey: "I feel confident in my roadmap prioritization") | 80% | Weekly PM survey (5-point scale) |
| **Citation accuracy** | N/A | 100% (zero hallucinated sources) | Eval golden set pass rate (M6) |
| **Escalation rate (low confidence)** | N/A | <20% of queries escalate to human | Langflow eval-log: % of queries with confidence <70% |

---

_Devika Sadekar · AI Product Management Certification · Module 3 · 25 August 2026_

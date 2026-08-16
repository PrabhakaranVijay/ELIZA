# ELIZA — Architecture Decision Records (ADR)

This document tracks significant architecture and technology decisions, why they were made, and what alternatives were considered. Every non-trivial technical decision should be recorded here — future contributors (including future you) need the "why," not just the "what."

## ADR Template

Copy this template for each new decision. Save major decisions as `docs/adr/NNNN-short-title.md` if the log grows large; keep a summary entry here regardless.

```markdown
## ADR-NNNN: [Short Title]

**Status:** Proposed | Accepted | Deprecated | Superseded by ADR-XXXX
**Date:** YYYY-MM-DD
**Deciders:** [names/handles]

### Context
What problem are we solving? What constraints apply?

### Decision
What did we decide to do?

### Alternatives Considered
- **Option A** — pros/cons
- **Option B** — pros/cons

### Consequences
What becomes easier or harder as a result? What trade-offs are we accepting?

### Revisit When
Under what conditions should this decision be reconsidered?
```

---

## Initial Technology Decisions

### ADR-0001: Python + FastAPI for the Core Service

**Status:** Accepted
**Date:** 2026-08-15
**Deciders:** Project maintainer

**Context**
Need a language/framework for the core agent service that has strong AI/ML ecosystem support, is approachable for contributors, and supports async I/O for concurrent tool calls and streaming responses.

**Decision**
Use Python 3.11+ with FastAPI for all core services.

**Alternatives Considered**
- **TypeScript/Node.js** — excellent async support and growing AI ecosystem, but weaker ML/data tooling and would fragment the stack from Python-native tools like LlamaIndex/LangGraph.
- **Go** — excellent performance and concurrency, but immature AI/agent ecosystem; would require reimplementing significant tooling.

**Consequences**
Full access to the Python AI ecosystem (Whisper, LangGraph, LlamaIndex, Ollama clients). Async performance is good enough for I/O-bound agent workloads via FastAPI + asyncio; CPU-bound work (embedding, inference) is offloaded to dedicated services (Ollama, embedding workers) rather than done in-process.

**Revisit When**
If a core bottleneck is identified that is fundamentally CPU-bound and unsuited to Python (unlikely given inference is already delegated to specialized runtimes).

---

### ADR-0002: PostgreSQL + Qdrant Split for Data Storage

**Status:** Accepted
**Date:** 2026-08-15

**Context**
Need to store both structured facts (user profile, task state, credentials metadata) and semantic/embedding data (documents, conversation history for retrieval).

**Decision**
Use PostgreSQL for structured, transactional data and Qdrant as a dedicated vector database for embeddings, rather than a single unified store.

**Alternatives Considered**
- **pgvector (Postgres extension)** — simpler ops (one database), but weaker vector search performance/features at scale compared to a purpose-built vector DB; reconsidered as a Phase 1 simplification option.
- **All-in-one vector DB (e.g., storing structured data as metadata in Qdrant)** — rejected; exact-match/transactional queries (task state, credentials) are a poor fit for a vector store and would sacrifice ACID guarantees.

**Consequences**
Two systems to operate instead of one, but each is used for what it's actually good at. Postgres remains the source of truth for anything requiring consistency guarantees.

**Revisit When**
If operational overhead of running two databases becomes a burden at homelab scale — pgvector is the documented fallback simplification.

---

### ADR-0003: LangGraph for Agent Orchestration

**Status:** Accepted
**Date:** 2026-08-15

**Context**
Need a framework for the agent reasoning loop that supports tool calling, multi-step planning, and is debuggable in production.

**Decision**
Use LangGraph for agent orchestration, modeled as an explicit state graph.

**Alternatives Considered**
- **AutoGPT-style autonomous agents** — appealing for "set and forget" autonomy, but historically unreliable and hard to debug; rejected given the project's reliability-first philosophy.
- **Custom hand-rolled orchestration** — maximum control, but reinvents a well-solved problem; revisit only if LangGraph proves limiting.
- **CrewAI** — good for role-based multi-agent setups, considered for Phase 4 multi-agent work; LangGraph preferred for now due to finer-grained control over state and transitions.

**Consequences**
Agent behavior is defined as an explicit, inspectable graph rather than emergent from a loosely-constrained loop, trading some flexibility for debuggability — consistent with the project's reliability-over-cleverness philosophy.

**Revisit When**
At Phase 4 (multi-agent ecosystem), re-evaluate whether LangGraph scales to inter-agent orchestration or whether a dedicated multi-agent framework is warranted.

---

### ADR-0004: MCP as the Standard Integration Protocol

**Status:** Accepted
**Date:** 2026-08-15

**Context**
Need a consistent way to expose tools (calendar, Home Assistant, NAS, etc.) to the agent that doesn't require bespoke integration code per tool.

**Decision**
Adopt Model Context Protocol (MCP) as the standard interface for all tool integrations going forward, migrating existing custom integrations over time (target: complete by Phase 4).

**Alternatives Considered**
- **Custom internal tool-calling schema** — full control, but builds a bespoke protocol for a problem the ecosystem is actively standardizing; would need to be rebuilt or bridged later anyway.
- **OpenAPI/REST-only tool calling** — simpler for basic integrations, but lacks standardized discovery/session semantics that MCP provides for agentic use.

**Consequences**
Early integrations (Phase 1–2) may use lightweight custom clients before MCP servers exist for a given service; these are treated as temporary and tracked for migration.

**Revisit When**
If MCP adoption stalls or a clearly superior standard emerges.

---

### ADR-0005: Local-First LLM Strategy with Cloud Fallback

**Status:** Accepted
**Date:** 2026-08-15

**Context**
Local models (via Ollama) protect privacy and cost but currently lag frontier cloud models on complex reasoning. Need a strategy that doesn't force an all-or-nothing choice.

**Decision**
Route requests through an LLM Gateway that defaults to local models, escalating to cloud models only for tasks that need it and only with explicit policy allowing it.

**Alternatives Considered**
- **Local-only** — maximal privacy, but would meaningfully cap capability for complex reasoning/research tasks.
- **Cloud-only** — best capability, but violates the privacy-first design principle as a hard default.

**Consequences**
Requires building and maintaining a routing layer (complexity classifier, cost/privacy policy engine) rather than a single LLM client — accepted as core, not incidental, complexity.

**Revisit When**
As local model quality improves, periodically re-evaluate which task categories still require cloud escalation.

---

## Future Decision Tracking

New ADRs should be appended below in the same format, numbered sequentially. Decisions that are later reversed should not be deleted — mark status as `Superseded by ADR-XXXX` and link the new decision, preserving the historical record of why the original choice was made.

<!-- ADR-0006 and beyond go here -->

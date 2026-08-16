# ELIZA — Roadmap

This roadmap is organized into five phases, each building toward the long-term vision described in [VISION.md](VISION.md). Phases are sequential in priority but may overlap in implementation. Each phase lists goals, deliverables, and exit criteria — the bar for calling a phase "done enough" to move on.

---

## Phase 1 — MVP: Core Agent

**Goal:** A working, reliable text-based agent with basic tool use and a production-grade foundation (not a prototype to be thrown away).

**Deliverables**
- FastAPI service exposing `/chat` endpoint
- Core Agent with LangGraph-based reasoning loop
- LLM Gateway supporting Ollama (local) and one cloud provider
- Tool Router with 2–3 real tool integrations (e.g., calendar read, notes read)
- PostgreSQL for structured data + conversation logs
- Short-term (session) memory
- Docker Compose deployment
- Langfuse tracing on every request
- Basic test suite (unit + integration) and CI pipeline

**Exit Criteria**
- Can hold a multi-turn text conversation with correct context retention
- Can answer "what's on my calendar" and "read me my notes on X" correctly
- Every request is traced end-to-end in Langfuse
- `docker compose up` works from a clean clone with only `.env` configuration

---

## Phase 2 — Personal Assistant

**Goal:** Add voice interaction and persistent long-term memory; expand integrations to make ELIZA genuinely useful day-to-day.

**Deliverables**
- Whisper STT + Piper TTS integrated into the pipeline
- Wake-word detection (openWakeWord)
- Long-term memory: vector store (Qdrant) + fact store, with a Memory Manager service
- Email integration (read, draft, human-confirmed send)
- Calendar write access (create/modify events, with confirmation)
- RAG pipeline over personal notes/documents
- User preference/profile store
- Human-in-the-loop confirmation flow for any write/send action

**Exit Criteria**
- Full voice round-trip works with acceptable latency (<3s perceived response time for simple queries)
- ELIZA recalls facts stated in a previous session (e.g., "remember I'm allergic to shellfish")
- Can draft and, on confirmation, send an email
- RAG retrieval returns relevant personal documents for a natural-language query

---

## Phase 3 — Home Intelligence

**Goal:** Extend awareness and control into the physical environment.

**Deliverables**
- Home Assistant integration (device state query + control, with confirmation for irreversible actions like locks)
- NAS integration (file search, status, basic operations)
- Media server integration (query "what's playing," basic playback control)
- System monitoring tool (homelab health: disk, CPU, service status)
- Expanded human-in-the-loop policy engine (configurable per-action-type confirmation rules)

**Exit Criteria**
- Can answer "is the garage door closed?" and act on "turn off the living room lights"
- Can report homelab health status on request
- Confirmation policy is configurable (not hardcoded) per action category

---

## Phase 4 — Agent Ecosystem

**Goal:** Move from a single core agent to a coordinated multi-agent system, and standardize all integrations on MCP.

**Deliverables**
- Formal multi-agent architecture: Planner, Research, Memory, Home, and Tool agents as independently deployable services (see [AGENTS.md](AGENTS.md))
- Inter-agent communication protocol (message schema + routing)
- Full migration of existing integrations to MCP servers
- Internet research agent capable of multi-step web research tasks
- Evaluation pipeline (MLflow) for agent quality regression testing
- Plugin interface documented for third-party tool/agent contributions

**Exit Criteria**
- Agents can be deployed, scaled, and restarted independently
- A new integration can be added purely as an MCP server with no core code changes
- Regression evals run automatically in CI against a fixed benchmark set of tasks

---

## Phase 5 — AI Operating System

**Goal:** ELIZA becomes the default interface to digital life — proactive, extensible, and trusted with graduated autonomy.

**Deliverables**
- Proactive intelligence: surfaces relevant info/suggestions without being asked
- Configurable autonomy levels per domain (e.g., "auto-approve calendar changes, always confirm financial actions")
- Plugin marketplace / community integration ecosystem
- Optional Kubernetes deployment for multi-node homelab clusters
- Multi-user/household support with per-user privacy boundaries
- Fully local voice pipeline option (zero cloud dependency end-to-end)
- Long-horizon task planning and execution (multi-day/multi-step goals)

**Exit Criteria**
- ELIZA is the primary daily-use interface for the majority of the target integration surface
- Community contributors have shipped at least one third-party plugin
- System operates reliably across a multi-node deployment with documented failover behavior

---

## Cross-Phase Ongoing Work

Some work isn't phase-specific and continues throughout:

- **Documentation** — kept current with every architectural change
- **Security review** — see [SECURITY.md](SECURITY.md), revisited every phase
- **Observability** — tracing/metrics coverage expanded alongside new components
- **Test coverage** — maintained as a gate for merging, not retrofitted later

## Non-Goals (for now)

- Multi-tenant SaaS deployment — ELIZA is designed for personal/homelab use, not hosted-service delivery
- Mobile-native apps — voice/text access via existing devices is prioritized over building dedicated mobile clients
- Full autonomous operation with no human oversight — graduated autonomy is a Phase 5 goal, not a starting assumption

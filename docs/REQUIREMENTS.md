# ELIZA — Requirements

## Functional Requirements

### FR-1: Conversation
- FR-1.1: The system shall accept text input via REST API.
- FR-1.2: The system shall accept voice input, transcribe it locally via Whisper, and process it identically to text input.
- FR-1.3: The system shall maintain conversational context within a session (short-term memory).
- FR-1.4: The system shall respond via text and, when voice mode is active, synthesized speech.

### FR-2: Memory
- FR-2.1: The system shall persist facts explicitly stated by the user across sessions (long-term memory).
- FR-2.2: The system shall retrieve relevant prior conversation context via semantic search when relevant to a new query.
- FR-2.3: The system shall allow the user to view, correct, and delete stored memory.

### FR-3: Tool Use
- FR-3.1: The system shall determine when a user request requires an external tool/integration versus a direct LLM response.
- FR-3.2: The system shall support calendar read and write operations.
- FR-3.3: The system shall support email read, draft, and (with confirmation) send operations.
- FR-3.4: The system shall support notes read and write operations.
- FR-3.5: The system shall support Home Assistant device query and control.
- FR-3.6: The system shall support NAS file search and status queries.
- FR-3.7: The system shall support media server status and playback control.
- FR-3.8: The system shall support internet search for information retrieval.
- FR-3.9: The system shall support semantic search over an ingested personal knowledge base (RAG).

### FR-4: Human-in-the-Loop
- FR-4.1: The system shall require explicit user confirmation before executing any irreversible or externally-visible action (sending email/messages, financial transactions, unlocking physical access points).
- FR-4.2: The system shall allow the user to configure per-action-category confirmation policy.
- FR-4.3: The system shall clearly present the pending action and its parameters before requesting confirmation.

### FR-5: Extensibility
- FR-5.1: The system shall support adding new tool integrations without modifying core agent code.
- FR-5.2: The system shall support MCP-compliant external tool servers.

### FR-6: Observability
- FR-6.1: The system shall trace every request end-to-end, capturing latency, tool calls, and token usage.
- FR-6.2: The system shall log all tool invocations and their outcomes.

---

## Non-Functional Requirements

### NFR-1: Privacy
- All user data shall be stored on infrastructure controlled by the user by default.
- No data shall be transmitted to third-party/cloud services without explicit, auditable, task-scoped consent.
- Cloud LLM usage shall be configurable per-request or globally disabled.

### NFR-2: Reliability
- The failure of any single integration shall not crash or block the core agent's ability to respond to unrelated requests.
- The system shall degrade gracefully when a cloud dependency (e.g., cloud LLM) is unavailable, falling back to local models where feasible.

### NFR-3: Maintainability
- Each component (voice, agent, tool, memory) shall be independently testable.
- Architecture decisions shall be documented per [DECISIONS.md](DECISIONS.md) at the time they are made, not retroactively.
- Code shall pass linting (ruff) and type checking (mypy) as a merge gate.

### NFR-4: Usability
- Voice interaction latency (wake word to first audio response) shall feel conversational, not batch-processed (see performance targets below).
- Error messages and confirmation prompts shall be understandable to a non-technical household member, not just the maintainer.

---

## Performance Requirements

| Metric | Target | Phase |
|---|---|---|
| Text query response time (simple, no tool call) | < 1.5s (local LLM) / < 3s (cloud LLM) | Phase 1 |
| Text query response time (with 1 tool call) | < 3s | Phase 1 |
| Voice round-trip latency (wake word → spoken response start) | < 3s for simple queries | Phase 2 |
| STT transcription accuracy (quiet room, clear speech) | > 95% word accuracy | Phase 2 |
| RAG retrieval latency | < 500ms for top-k retrieval | Phase 2 |
| System uptime (core agent) | > 99% (excludes planned maintenance) | Phase 3+ |
| Concurrent session support | 1 primary user + household members, minimum 5 concurrent sessions | Phase 3+ |

Performance targets are measured and tracked via the observability stack (Langfuse), not estimated — see [DEPLOYMENT.md](DEPLOYMENT.md) for monitoring setup.

---

## Security Requirements

- SEC-1: All API endpoints shall require authentication (see [SECURITY.md](SECURITY.md) for the auth model).
- SEC-2: Secrets (API keys, tokens, credentials) shall be stored in a secrets manager or encrypted environment store, never committed to version control.
- SEC-3: All external integrations shall use least-privilege credentials/scopes where the integrated service supports it.
- SEC-4: Communication between internal services shall occur over the local network only by default; any external exposure shall require explicit configuration and use TLS.
- SEC-5: Voice input shall not be transmitted to any cloud service unless the user has explicitly opted into a cloud STT/LLM path for that session.
- SEC-6: The system shall maintain an audit log of all confirmed high-stakes actions (email sends, device control, financial actions).

Full threat model in [SECURITY.md](SECURITY.md).

---

## Scalability Requirements

- SCALE-1: The system shall run on a single-host Docker Compose deployment for Phases 1–3.
- SCALE-2: The system architecture shall not preclude a future migration to Kubernetes for multi-node deployment (Phase 5).
- SCALE-3: The vector database and structured data store shall be able to scale independently of the core agent process.
- SCALE-4: New tool integrations shall be addable without requiring changes to the scaling characteristics of the core agent.
- SCALE-5: The system shall support horizontal scaling of stateless components (API gateway, tool router) independent of stateful components (memory stores).

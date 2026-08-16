# ELIZA — Security

ELIZA has access to calendars, email, home devices, and personal knowledge — a compromise of this system is a compromise of a meaningful slice of someone's life. Security is treated as a first-class requirement, not an afterthought.

## Authentication

- All API endpoints require authentication via bearer token (JWT) by default; no unauthenticated endpoints except `/health`.
- Voice interface authentication is tied to the physical device/session that initiated the wake word, not per-utterance — but device pairing itself requires an authenticated setup step.
- Tokens are short-lived and refreshed via a standard refresh-token flow; long-lived tokens are only issued for trusted service-to-service communication within the local network.
- Multi-user/household support (Phase 5) will require per-user authentication with isolated memory/context — not shared implicitly.

## Authorization

- Role-based access control at the API layer: `owner` (full access, including memory management and integration configuration) vs. `household member` (conversational access, scoped tool use, no configuration or memory-export access) — household roles introduced alongside multi-user support.
- Tool/integration access is scoped per credential — e.g., the Home Assistant integration uses a long-lived access token scoped only to the entities/domains ELIZA needs, not full admin access, wherever the integrated service supports scoped tokens.
- Human-in-the-loop confirmation (see [REQUIREMENTS.md](REQUIREMENTS.md) FR-4) acts as an additional authorization layer for high-stakes actions regardless of role.

## Secret Management

- Secrets (API keys, OAuth tokens, database credentials) are never committed to version control; `.env.example` documents required variables without values.
- Local deployments store secrets in environment variables sourced from a `.env` file excluded via `.gitignore`, or a mounted secrets file with restricted filesystem permissions (`600`).
- Production/homelab deployments are encouraged to use a secrets manager (e.g., Docker secrets, HashiCorp Vault, or SOPS-encrypted config) rather than plaintext `.env` files — documented as the recommended path in [DEPLOYMENT.md](DEPLOYMENT.md).
- OAuth tokens for third-party integrations (Gmail, Google Calendar) are refreshed and stored encrypted at rest in PostgreSQL, never logged.

## Local Network Security

- Internal service-to-service communication (Core Agent ↔ Postgres ↔ Qdrant ↔ Ollama) occurs over the Docker Compose internal network by default, not exposed to the host network.
- Any endpoint exposed beyond localhost/LAN (e.g., for remote voice access) must go through a reverse proxy with TLS termination (e.g., Caddy, Traefik) — plaintext HTTP is not supported for any non-localhost exposure.
- Home Assistant, NAS, and media server integrations connect over the local network by default; remote access to these systems is the user's own network configuration responsibility, not something ELIZA proxies without explicit setup.

## Data Privacy

- Cloud LLM/API calls are opt-in per the LLM Gateway routing policy (see [ARCHITECTURE.md](ARCHITECTURE.md) and [DECISIONS.md](DECISIONS.md) ADR-0005) — no data leaves the local network by default.
- When cloud escalation does occur, only the minimum necessary context is sent (not the full memory store), and this is logged in the audit trail.
- Voice audio is processed locally by default (faster-whisper); cloud STT is an explicit opt-in, never a silent fallback.
- Users can export or delete all stored data (facts, embeddings, conversation history) — see [MEMORY.md](MEMORY.md) for user memory controls.

## Threat Model

| Threat | Mitigation |
|---|---|
| Unauthorized network access to the API | Bearer token auth on all endpoints; LAN-only exposure by default; TLS for any remote access |
| Compromised third-party integration credential | Scoped tokens per integration; credentials encrypted at rest; audit logging of all tool invocations |
| Prompt injection via ingested documents/emails/web content | Tool outputs and retrieved content are treated as untrusted data, never as instructions — the agent's system prompt explicitly distinguishes instructions from data; high-stakes actions still require human confirmation regardless of what content "suggests" |
| Data exfiltration via unintended cloud LLM calls | Explicit opt-in routing policy; logged and auditable cloud escalations; local-first default |
| Unauthorized physical action (e.g., unlocking doors) via voice spoofing | Irreversible/security-relevant actions always require explicit confirmation; voice-only confirmation for high-risk categories can be configured to require a secondary factor (e.g., companion app tap) |
| Malicious/compromised MCP server | MCP servers are added to an explicit allowlist; auto-discovery of arbitrary servers is not supported; tool schemas are validated before execution |
| Database compromise exposing full personal history | Encryption at rest for sensitive fields (OAuth tokens, credentials); regular backup encryption; network-level isolation of the database from any external exposure |
| Insider risk (household member misusing shared access) | Role-based access control; per-user session isolation (Phase 5); audit log of all high-stakes actions attributable to a session/user |

## Audit Logging

All confirmed high-stakes actions (per FR-4) are recorded in an append-only audit log including: timestamp, initiating session/user, action type, parameters, and outcome. This log is separate from general application logs and retained independently of conversation history retention settings.

## Reporting a Vulnerability

If you discover a security vulnerability in ELIZA, please do not open a public issue. Instead, report it privately via [security contact method to be added] so it can be addressed before public disclosure.

# ELIZA — Vision

## Mission

To build a private, self-hosted AI companion that acts as the central intelligence layer for one person's digital and physical life — capable of holding natural conversation, remembering what matters, reasoning over personal knowledge, and taking action across the tools and systems that person actually uses, without surrendering their data to third parties.

ELIZA exists because the most capable AI assistants today require trading privacy for convenience. This project is a bet that the trade isn't necessary: that a sufficiently well-engineered local-first system can deliver JARVIS-level utility while keeping the user in full control of their data, their infrastructure, and their AI's behavior.

## Long-Term Vision

ELIZA will evolve through five stages, from a conversational assistant to a full personal AI operating system:

1. **Conversational core** — a reliable agent that can chat, use tools, and remember context within a session.
2. **Personal assistant** — persistent memory, calendar/email management, voice interaction.
3. **Home intelligence** — awareness of and control over the physical environment (Home Assistant, NAS, media).
4. **Agent ecosystem** — a coordinated team of specialized agents (planning, research, memory, home control) working under a central orchestrator, extensible via MCP.
5. **AI operating system** — ELIZA becomes the primary interface to digital life: the layer between the user and every application, device, and data source they own, with the judgment to act autonomously within well-defined boundaries and the humility to ask when it shouldn't.

The end state is not "a smarter chatbot." It is an operating system whose primary UI is conversation, whose kernel is an agent orchestration layer, and whose "installed applications" are tools and integrations — self-hosted, inspectable, and owned entirely by the user.

## Core Philosophy

**Privacy is a default, not a feature.** Cloud LLMs are used only as an opt-in augmentation for tasks that benefit from frontier reasoning, never as a requirement for basic function. Data at rest lives on hardware the user controls.

**Reliability over cleverness.** An assistant that is wrong confidently is worse than one that says "I don't know." ELIZA is built to be transparent about uncertainty and to prefer boring, debuggable architecture over impressive-looking but brittle autonomy.

**Modularity as a survival strategy.** The AI tooling landscape changes monthly. Every component — STT engine, LLM provider, vector store, agent framework — must be swappable without a rewrite. Tight coupling to any single vendor or library is treated as technical debt from day one.

**Human-in-the-loop by design, not by accident.** Irreversible or high-stakes actions (sending an email, unlocking a door, spending money) always pass through explicit confirmation unless the user has deliberately configured otherwise. Autonomy is expanded deliberately and incrementally, never assumed.

**Built to last years, not to demo well once.** ELIZA is designed with the engineering discipline of production software — tests, observability, documented decisions, versioned APIs — because it is meant to be lived with and maintained for years, not shown off once and abandoned.

## Future Goals

- A plugin/marketplace model where community-built tools and agents can be installed with the same trust model as MCP servers today
- Multi-user/multi-household support while preserving the single-owner privacy guarantees
- On-device or edge deployment options (e.g., a dedicated ELIZA hardware appliance)
- Proactive intelligence — ELIZA surfaces relevant information and suggests actions rather than only responding to queries
- A fully local voice pipeline capable of natural, low-latency conversation with no cloud dependency at all
- Formal support for federated/multi-node deployment across a homelab cluster

## Success Criteria

ELIZA is successful if, over a multi-year horizon:

- It is used daily as the primary interface for calendar, notes, home control, and information retrieval — not as a novelty.
- No personal data is transmitted to a third party without explicit, task-specific user consent.
- New integrations can be added by a contributor unfamiliar with the core codebase, using documented tool/agent interfaces, in under a day.
- The system degrades gracefully — a failed integration or unavailable cloud LLM never breaks the whole assistant.
- The project sustains an open-source contributor base beyond its original author, with documented architecture decisions that make onboarding tractable.
- Uptime, latency, and cost are all treated as measured, tracked metrics rather than anecdotal impressions — thanks to the observability layer built in from Phase 1.

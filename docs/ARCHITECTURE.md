# ELIZA — Architecture

## Table of Contents
- [High-Level Architecture](#high-level-architecture)
- [System Components](#system-components)
- [Data Flow](#data-flow)
- [Service Interactions](#service-interactions)
- [Agent Architecture](#agent-architecture)
- [Knowledge Architecture](#knowledge-architecture)
- [Infrastructure Architecture](#infrastructure-architecture)
- [Sequence Diagrams](#sequence-diagrams)

---

## High-Level Architecture

ELIZA is organized into four layers, each independently deployable and testable:

```mermaid
flowchart TB
    subgraph Voice["🎙️ Voice Layer"]
        V1[Wake Word Detection]
        V2[Whisper STT]
        V3[TTS Engine]
    end

    subgraph Intelligence["🧠 Intelligence Layer"]
        I1[Core Agent / Orchestrator]
        I2[Planner Agent]
        I3[Tool Router]
        I4[LLM Gateway]
    end

    subgraph Knowledge["📚 Knowledge Layer"]
        K1[Vector DB - Qdrant]
        K2[Postgres - Structured Data]
        K3[RAG Pipeline]
        K4[Memory Manager]
    end

    subgraph Infra["🏗️ Infrastructure Layer"]
        F1[Docker / Compose]
        F2[Langfuse - Tracing]
        F3[MLflow - Experiment Tracking]
        F4[Monitoring / Logging]
    end

    Voice --> Intelligence
    Intelligence --> Knowledge
    Intelligence --> Infra
    Knowledge --> Infra
```

Each layer communicates through well-defined interfaces (REST/gRPC internally, MCP for external tool integrations), so any layer's implementation can be replaced without affecting the others.

---

## System Components

| Component | Responsibility | Primary Tech |
|---|---|---|
| **Voice Gateway** | Wake word detection, audio capture, streaming to STT | openWakeWord, PyAudio |
| **STT Service** | Converts speech to text | faster-whisper |
| **Core Agent** | Central reasoning loop; receives input, decides on tool use vs. direct response | LangGraph, FastAPI |
| **Tool Router** | Maps agent intents to concrete tool/integration calls | Custom + MCP client |
| **LLM Gateway** | Routes requests to local or cloud LLM based on task, cost, and privacy policy | Ollama, Anthropic/OpenAI SDKs |
| **Memory Manager** | Reads/writes short-term and long-term memory | Postgres + Qdrant |
| **RAG Pipeline** | Ingests documents, chunks, embeds, retrieves relevant context | LlamaIndex + Qdrant |
| **Integration Services** | Calendar, email, notes, Home Assistant, NAS, media server clients | REST/MCP clients |
| **TTS Service** | Converts response text to speech | Piper |
| **Observability Stack** | Tracing, metrics, evaluation | Langfuse, MLflow, Prometheus/Grafana |

---

## Data Flow

```mermaid
flowchart LR
    A[User Input<br/>Voice or Text] --> B{Input Type}
    B -->|Voice| C[STT: Whisper]
    B -->|Text| D[Core Agent]
    C --> D
    D --> E[Context Assembly<br/>short-term + retrieved memory]
    E --> F[LLM Gateway]
    F --> G{Tool Call Needed?}
    G -->|Yes| H[Tool Router]
    H --> I[Execute Integration]
    I --> D
    G -->|No| J[Generate Response]
    J --> K{Output Channel}
    K -->|Voice| L[TTS: Piper]
    K -->|Text| M[Return to Client]
    L --> N[Voice Output]
```

---

## Service Interactions

```mermaid
flowchart TD
    Client[Client - CLI / Voice / Web] -->|REST/WebSocket| API[FastAPI Gateway]
    API --> Core[Core Agent Service]
    Core --> Memory[Memory Manager]
    Core --> Router[Tool Router]
    Core --> LLMGW[LLM Gateway]

    Memory --> PG[(PostgreSQL)]
    Memory --> VDB[(Qdrant)]

    Router --> Cal[Calendar Service]
    Router --> Mail[Email Service]
    Router --> HA[Home Assistant]
    Router --> NAS[NAS Service]
    Router --> Search[Search Service]
    Router --> KB[Knowledge Base / RAG]

    LLMGW --> Local[Ollama - Local LLM]
    LLMGW --> Cloud[Cloud LLM Provider]

    Core --> Trace[Langfuse Tracing]
```

All internal services expose a health check endpoint (`/health`) and structured JSON logs. Cross-service calls carry a trace ID propagated to Langfuse for end-to-end observability.

---

## Agent Architecture

ELIZA's reasoning layer is built as a directed graph of specialized agents coordinated by a central orchestrator (see [docs/AGENTS.md](AGENTS.md) for full detail on each agent's responsibilities).

```mermaid
flowchart TB
    User[User Request] --> Orchestrator[Core Agent / Orchestrator]
    Orchestrator --> Planner[Planner Agent]
    Planner --> Decision{Route}
    Decision -->|Simple query| Direct[Direct LLM Response]
    Decision -->|Needs info| Research[Research Agent]
    Decision -->|Needs memory| MemAgent[Memory Agent]
    Decision -->|Needs home action| HomeAgent[Home Agent]
    Decision -->|Needs external tool| ToolAgent[Tool Agent]

    Research --> Orchestrator
    MemAgent --> Orchestrator
    HomeAgent --> Orchestrator
    ToolAgent --> Orchestrator
    Direct --> Orchestrator

    Orchestrator --> Response[Final Response]
```

In Phase 1, the "Planner" and "Orchestrator" are the same process with a simple rules + LLM router. Full multi-agent separation (Phase 4) introduces independent, testable agent processes communicating over a defined message schema.

---

## Knowledge Architecture

Knowledge is deliberately split across two stores rather than unified into a single vector database:

```mermaid
flowchart LR
    subgraph Structured["Structured Facts (Postgres)"]
        S1[User profile & preferences]
        S2[Task/reminder state]
        S3[Integration credentials/config]
        S4[Conversation metadata]
    end

    subgraph Semantic["Semantic Knowledge (Qdrant)"]
        V1[Document embeddings]
        V2[Conversation history embeddings]
        V3[Personal notes embeddings]
    end

    Query[User Query] --> Router{Query Type}
    Router -->|Fact lookup| Structured
    Router -->|Semantic/fuzzy| Semantic
    Router -->|Hybrid| Both[Combine Results]
    Structured --> Both
    Semantic --> Both
    Both --> Context[Assembled Context]
```

This separation exists because facts ("what's my WiFi password," "when is my dentist appointment") need deterministic, exact retrieval — a vector search is the wrong tool. Semantic knowledge ("what did I say about that book last month") is exactly what embeddings are good at. Full rationale in [docs/MEMORY.md](MEMORY.md).

---

## Infrastructure Architecture

```mermaid
flowchart TB
    subgraph Host["Homelab Host(s)"]
        subgraph Compose["Docker Compose Stack"]
            Core[eliza-core]
            PG[(postgres)]
            VDB[(qdrant)]
            Ollama[ollama]
            LF[langfuse]
            ML[mlflow]
        end
    end

    subgraph External["External / Optional"]
        HA[Home Assistant instance]
        Cloud[Cloud LLM APIs]
        NASdev[NAS device]
    end

    Compose -->|LAN| HA
    Compose -->|LAN| NASdev
    Core -.->|opt-in, egress| Cloud
```

Phase 1–3 target a single Docker Compose host. Phase 5 introduces optional Kubernetes support for multi-node homelab clusters — see [docs/DEPLOYMENT.md](DEPLOYMENT.md).

---

## Sequence Diagrams

### Text query with tool use

```mermaid
sequenceDiagram
    participant U as User
    participant API as FastAPI Gateway
    participant Core as Core Agent
    participant Mem as Memory Manager
    participant LLM as LLM Gateway
    participant Tool as Calendar Service

    U->>API: "What's on my calendar today?"
    API->>Core: forward request + session_id
    Core->>Mem: fetch short-term context
    Mem-->>Core: recent conversation turns
    Core->>LLM: prompt + tool schema
    LLM-->>Core: tool_call(get_events, date=today)
    Core->>Tool: get_events(today)
    Tool-->>Core: [event list]
    Core->>LLM: tool result + original prompt
    LLM-->>Core: final response text
    Core->>Mem: persist turn
    Core-->>API: response
    API-->>U: "You have 3 events today: ..."
```

### Voice round-trip (Phase 2+)

```mermaid
sequenceDiagram
    participant U as User (mic)
    participant WW as Wake Word
    participant STT as Whisper
    participant Core as Core Agent
    participant TTS as Piper
    participant Spk as Speaker

    U->>WW: "Hey ELIZA..."
    WW->>STT: audio stream (post wake-word)
    STT->>Core: transcribed text
    Core->>Core: reasoning + tool calls
    Core->>TTS: response text
    TTS->>Spk: synthesized audio
    Spk-->>U: spoken response
```

### Human-in-the-loop confirmation

```mermaid
sequenceDiagram
    participant U as User
    participant Core as Core Agent
    participant Tool as Email Service

    U->>Core: "Reply to Sarah saying I'll be there"
    Core->>Core: draft reply
    Core-->>U: "Here's the draft — send it? [confirm]"
    U->>Core: confirm
    Core->>Tool: send_email(draft)
    Tool-->>Core: sent confirmation
    Core-->>U: "Sent."
```

---

## Design Trade-offs

| Decision | Trade-off Accepted |
|---|---|
| Postgres + Qdrant split | Extra operational complexity vs. correctness of fact retrieval |
| Local-first LLM | Lower ceiling on reasoning quality vs. privacy and cost control |
| LangGraph over autonomous agent frameworks | Less "magic"/autonomy vs. far greater debuggability |
| Docker Compose before Kubernetes | Slower path to multi-node scale vs. avoiding premature complexity |
| MCP as the integration standard | Dependency on an evolving spec vs. avoiding a bespoke protocol that would need rebuilding later |

See [docs/DECISIONS.md](DECISIONS.md) for the full Architecture Decision Record log.

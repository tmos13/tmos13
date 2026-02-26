# Architecture Guide

Engine internals, design principles, and data flow.

---

## Design Principles

TMOS13 is built around one core idea: **the server is the source of truth**. The LLM generates natural language, but the server tracks state, enforces rules, routes commands, and assembles context. This separation ensures deterministic behavior for critical operations while preserving the LLM's generative capabilities for open-ended interaction.

**Key architectural decisions:**
- **Protocol-driven behavior** — AI behavior is defined in plain text files, not code
- **Manifest-driven configuration** — One JSON file defines an entire vertical
- **Deterministic routing** — Most user inputs resolve without touching the LLM
- **Dynamic prompt assembly** — Only relevant context is sent per request
- **Vertical-agnostic engine** — Same runtime serves any domain via the pack system

---

## System Architecture

```
                    ┌─────────────────────────────────────┐
                    │           Client Layer               │
                    │   REST (/chat)  │  WebSocket (/ws)  │
                    └────────┬────────┴────────┬──────────┘
                             │                  │
                    ┌────────▼──────────────────▼──────────┐
                    │         FastAPI Server                │
                    │                                       │
                    │  ┌─────────────────────────────────┐ │
                    │  │       Command Router             │ │
                    │  │                                   │ │
                    │  │  P1: Numerical ──→ System Screen  │ │
                    │  │  P2: Session ──→ Direct Response  │ │
                    │  │  P3: Navigation ──→ Module Switch │ │
                    │  │  P4: Passthrough ──→ LLM          │ │
                    │  └──────────────┬──────────────────┘ │
                    │                 │                     │
                    │  ┌──────────────▼──────────────────┐ │
                    │  │       Prompt Assembler           │ │
                    │  │                                   │ │
                    │  │  Master Protocol + Active Module  │ │
                    │  │  + Session State + Feature Guards │ │
                    │  └──────────────┬──────────────────┘ │
                    │                 │                     │
                    │  ┌──────────────▼──────────────────┐ │
                    │  │       LLM API Call               │ │
                    │  └──────────────┬──────────────────┘ │
                    │                 │                     │
                    │  ┌──────────────▼──────────────────┐ │
                    │  │       Response Parser            │ │
                    │  │  State Signal Extraction          │ │
                    │  └──────────────┬──────────────────┘ │
                    │                 │                     │
                    │  ┌──────────────▼──────────────────┐ │
                    │  │       State Manager              │ │
                    │  │  Session State + Cross-Module Mem │ │
                    │  └──────────────────────────────────┘ │
                    └───────────────────────────────────────┘
                                      │
          ┌───────────────────────────┼───────────────────────────┐
          │                           │                           │
 ┌────────▼────────┐      ┌──────────▼──────────┐     ┌─────────▼────────┐
 │   Pack System    │      │    Persistence       │     │   Integrations   │
 │                  │      │                      │     │                  │
 │  manifest.json   │      │  Supabase / SQLite   │     │  Auth, Billing,  │
 │  Protocol Files  │      │  Redis / In-memory   │     │  Email, MCP,     │
 └─────────────────┘      └──────────────────────┘     │  RAG, Monitoring │
                                                        └──────────────────┘
```

---

## Core Components

### Command Router

The command router is a deterministic interception layer that processes user input before it reaches the LLM. Commands are resolved in strict priority order:

1. **Numerical Commands** (P1) — Codes like `111`, `222`, `333` map to system screens. These resolve with zero LLM tokens — the response is pre-defined or triggers a system screen.

2. **Session Commands** (P2) — Keywords like MENU and EXIT that manage session navigation. Resolve without LLM involvement.

3. **Navigation Commands** (P3) — Regex-matched phrases that route to specific modules. Only active when no module is currently loaded (prevents accidental module switches mid-conversation).

4. **LLM Passthrough** (P4) — Everything else passes through to the LLM with the assembled system prompt.

All command definitions are loaded from the active pack's manifest. The router is entirely data-driven — no hardcoded commands.

### Prompt Assembler

The assembler constructs system prompts dynamically based on session context:

1. **Master protocol** — Always included (identity, rules, brand voice)
2. **Active module protocol** — Only the current module's instructions
3. **Session state injection** — Compact, machine-readable current state
4. **Feature-gated sections** — Enabled per pack via feature flags
5. **Special instructions** — Context-aware rules based on session state

**Token efficiency:** ~80% reduction vs. sending all protocol files. A typical monolithic prompt is ~7,000 tokens. The assembled prompt is ~1,200–2,000 depending on module complexity.

**Prompt caching:** Compatible with Anthropic's caching API. Static content (master + module) is cached across turns. Dynamic content (session state) is sent fresh. ~90% cost reduction on the cached portion.

### Session State Manager

`SessionState` tracks everything about an active session:

- **Global state** — Current module, engagement depth (0–5), mood, turn count, history
- **Module sub-states** — Each module type has its own state (timers, counters, limits)
- **Cross-module memory** — Key-value store that persists across module switches
- **Settings** — User preferences loaded from pack defaults

**Server-side enforcement:** The server manages timers, counters, and limits directly. These values are injected into the LLM context as facts, not suggestions. The model reads them as ground truth. This prevents state drift that occurs when models self-manage state.

### Response Parser

Processes LLM responses to extract structured state signals embedded in natural language output:

- Achievement discoveries
- Module-specific state changes
- Cross-module memory updates
- Mood indicators

Signal markers are stripped from the response before returning clean text to the client.

### Pack System

The pack system enables TMOS13 to serve multiple verticals from a single codebase.

Each pack's `manifest.json` defines:
- Identity (name, version, personality)
- Modules (cartridges with protocol file references)
- Commands (numerical, session, navigation patterns)
- Features (boolean flags that gate engine behavior)
- Defaults (settings, game config)

**Validation:** Manifests are validated at load time — required fields, file existence, regex compilation.

**Multi-pack sessions:** Different sessions can run different packs on the same instance. Each session stores a `pack_id` and the engine resolves the correct configuration per-request.

---

## Data Flow

### Request Lifecycle

```
Input → Validate → Mood Detect → Module Pre-process → Route → Assemble → LLM → Parse → State Update → Response
```

1. **Input received** via REST or WebSocket
2. **Input validation** — Length check, rate limit check
3. **Mood detection** — Sentiment analysis updates session mood
4. **Module pre-processing** — Timer decrements, action counters
5. **Command routing** — Router determines action
6. **Prompt assembly** — System prompt constructed from pack + state
7. **LLM API call** — Assembled prompt + conversation history
8. **Response parsing** — State signals extracted, response cleaned
9. **State persistence** — Memories saved to database
10. **Response returned** to client with updated state metadata

### State Lifecycle

1. **Creation** — Initialized with defaults from active pack
2. **Evolution** — Updated every turn (mood, depth, module state)
3. **Persistence** — Cross-module memories saved per-turn
4. **Cleanup** — Background task removes sessions after 1 hour TTL
5. **Shutdown** — Active sessions logged to database on server stop

---

## Infrastructure

### Deployment

| Target | Method |
|--------|--------|
| Development | `uvicorn app:app --reload` with SQLite + in-memory cache |
| Docker | `docker-compose.yml` runs engine + Redis |
| Production | Any container platform (Railway, Vercel, AWS, GCP) |
| Air-gapped | Local inference via Ollama (no cloud dependency) |

### Database

- **Production:** Supabase (PostgreSQL) — users, session logs, memories, achievements
- **Development:** SQLite — automatic fallback when Supabase is not configured

### Caching

- **Production:** Redis — session state, rate limiting
- **Development:** In-memory — automatic fallback
- **Prompt caching:** Anthropic API-level caching of static protocol content

### Authentication

- OAuth: Apple, Google, GitHub (via Supabase Auth)
- JWT verification for protected endpoints
- RBAC: 5-level role hierarchy (viewer → user → editor → admin → owner)

### Monitoring

- **Errors:** Sentry
- **Analytics:** PostHog
- **Tracing:** OpenTelemetry
- **Metrics:** Prometheus-compatible at `/metrics`

---

## LLM Backend

TMOS13 currently uses Anthropic Claude but the architecture supports backend swapping:

- The **Assembler** produces standard text prompts
- The **Router** operates entirely without LLM involvement
- The **State Manager** is LLM-agnostic
- The **Parser** looks for text patterns (not API-specific structures)

The LLM integration point is a single async function. Swapping backends means implementing that function with a different client library. The prompt format (plain text system prompt + message history) is compatible with any chat completion API.

**Local inference:** The engine supports Ollama out of the box for air-gapped deployments. Set `TMOS13_LLM_PROVIDER=ollama` and point to a local model.

---

## Scalability

- **Stateless server:** Session state is in-memory with optional external persistence. Horizontal scaling requires a shared state store (Redis).
- **Pack isolation:** Different packs run on the same instance without cross-pack state contamination.
- **LLM independence:** All non-generative operations (routing, state, assembly) run locally with zero external dependency.

---

## Next Steps

- [Getting Started](getting-started.md) — Set up and run the engine
- [Pack Development](pack-development.md) — Build your own vertical
- [API Reference](api-reference.md) — All endpoints
- [SDK Reference](sdk-reference.md) — TypeScript client library

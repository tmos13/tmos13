# Getting Started

Set up your first pack and deploy in 15 minutes.

---

## What is TMOS13?

TMOS13 is a runtime engine for stateful AI experiences. Instead of writing code to manage LLM behavior, you write **protocols** — plain text files that describe how the AI should act — and configure a **manifest** that wires everything together. The engine handles state management, routing, prompt assembly, and multi-channel delivery.

**One engine. Any vertical. Your brand.**

## Prerequisites

- Python 3.12+
- Node.js 18+ (for the SDK and frontend)
- An [Anthropic API key](https://console.anthropic.com/) (or a local model via Ollama)

## Quick Start

### 1. Clone and install

```bash
git clone https://github.com/tmos13/tmos13.git
cd tmos13

# Backend
cd engine
pip install -r requirements.txt

# SDK (optional — for frontend integration)
cd ..
npm install
```

### 2. Configure environment

Create `engine/.env`:

```env
ANTHROPIC_API_KEY=sk-ant-...
TMOS13_ENV=development
```

That's the minimum. The engine runs with sensible defaults for everything else — SQLite for storage, in-memory caching, no auth required.

### 3. Start the engine

```bash
cd engine
uvicorn app:app --reload --port 8000
```

You should see:

```
INFO:     TMOS13 engine started
INFO:     Pack loaded: tmos13_site (v1.0)
INFO:     Uvicorn running on http://0.0.0.0:8000
```

### 4. Send your first message

```bash
curl -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "hello"}'
```

The engine will:
1. Route your input through the command router
2. Assemble a system prompt from the active pack's protocols
3. Call the LLM with the assembled context
4. Return a response with session state

### 5. Check system health

```bash
curl http://localhost:8000/health
```

Returns pack info, active sessions, infrastructure status, and uptime.

---

## Create Your First Pack

A pack is a self-contained vertical that defines an AI experience. Let's create a simple support agent.

### Directory structure

```bash
mkdir -p protocols/packs/my_support_agent
```

### Write the manifest

Create `protocols/packs/my_support_agent/manifest.json`:

```json
{
  "pack_id": "my_support_agent",
  "name": "Support Agent",
  "version": "1.0",
  "tagline": "Friendly product support",
  "philosophy": "Be helpful, be clear, resolve issues quickly.",

  "personality": {
    "tone": "professional, empathetic, clear",
    "warmth": 0.8,
    "humor": 0.2,
    "formality": 0.5,
    "identity_name": "Alex",
    "identity_description": "A product support specialist"
  },

  "cartridges": {
    "general": {
      "file": "general.txt",
      "name": "General Support",
      "number": 1,
      "description": "General product questions and troubleshooting"
    },
    "billing": {
      "file": "billing.txt",
      "name": "Billing Help",
      "number": 2,
      "description": "Billing questions and account management"
    }
  },

  "system_screens": {
    "boot": "boot.txt",
    "shutdown": "shutdown.txt",
    "settings": "settings.txt",
    "diagnostics": "diagnostics.txt",
    "utilities": "utilities.txt",
    "force_quit": "force_quit.txt"
  },

  "commands": {
    "numerical": {
      "111": { "action": "system", "target": "shutdown" },
      "222": { "action": "system", "target": "settings" },
      "333": { "action": "system", "target": "boot" }
    },
    "session": {
      "MENU": { "action": "menu", "label": "Open Menu" },
      "EXIT": { "action": "close", "label": "Close Session" }
    },
    "navigation": {
      "billing|payment|invoice|charge": { "action": "navigate", "target": "billing" },
      "general|help|support|question": { "action": "navigate", "target": "general" }
    }
  },

  "features": {
    "depth_tracking": true,
    "mood_resonance": true,
    "cross_module_memory": true
  },

  "settings_defaults": {
    "verbosity": "balanced",
    "formatting": "rich"
  }
}
```

### Write protocol files

**`master.txt`** — always injected into the system prompt:

```text
# Support Agent — Master Protocol

## Identity
You are Alex, a product support specialist. You are friendly, clear, and focused on resolving the user's issue.

## Rules
- Always acknowledge the user's concern before troubleshooting
- If you don't know the answer, say so honestly
- Keep responses concise unless the user asks for detail

## Commands
- 111: Exit session
- 222: Settings
- 333: Welcome screen
- Type MENU to see available modules
```

**`general.txt`** — loaded when the General Support module is active:

```text
# General Support

## Behavior
You are handling general product support. Help the user with:
- Product features and how-to questions
- Troubleshooting common issues
- Account setup and configuration

## Style
- Use numbered steps for multi-step instructions
- Ask clarifying questions if the issue is ambiguous
```

**`billing.txt`** — loaded when the Billing module is active:

```text
# Billing Help

## Behavior
You are handling billing and account questions. Help with:
- Understanding charges and invoices
- Subscription management
- Payment method updates
- Refund requests

## Escalation
For refund amounts over $100 or disputed charges, recommend the user contact support@example.com directly.
```

Create the required system screen files (`boot.txt`, `shutdown.txt`, `settings.txt`, `diagnostics.txt`, `utilities.txt`, `force_quit.txt`) with appropriate welcome/exit/system messages.

### Run it

```bash
TMOS13_PACK=my_support_agent uvicorn app:app --reload --port 8000
```

Your support agent is live. Every interaction is stateful — the engine tracks which module is active, conversation depth, user mood, and cross-module context.

---

## Deploy

### Docker

```bash
docker build -t tmos13-engine .
docker run -p 8000:8000 --env-file .env tmos13-engine
```

### Railway / Vercel / Any Container Platform

The engine is a standard FastAPI application. Deploy anywhere that runs Python containers:

1. Set your environment variables (`ANTHROPIC_API_KEY`, `TMOS13_PACK`, etc.)
2. Set the start command: `uvicorn engine.app:app --host 0.0.0.0 --port $PORT`
3. Deploy

See the [Deployment Guide](deployment.md) for platform-specific instructions (coming soon).

---

## Connect a Frontend

Install the SDK:

```bash
npm install @tmos13/sdk
```

```typescript
import { TMOS13Client } from "@tmos13/sdk";

const client = new TMOS13Client({ baseUrl: "http://localhost:8000" });
const response = await client.chat("hello");
console.log(response.response);
```

Or use the React hook:

```tsx
import { useTMOS13 } from "@tmos13/sdk";

function Chat() {
  const { sendMessage, messages, isLoading } = useTMOS13({
    baseUrl: "http://localhost:8000",
  });

  return (
    <div>
      {messages.map((m) => <p key={m.id}>{m.text}</p>)}
      <button onClick={() => sendMessage("hello")} disabled={isLoading}>
        Send
      </button>
    </div>
  );
}
```

See the [SDK Reference](sdk-reference.md) for the full API.

---

## Next Steps

- [SDK Reference](sdk-reference.md) — Client library, React hooks, type definitions
- [Pack Development](pack-development.md) — Deep dive into protocol authoring and manifest configuration
- [API Reference](api-reference.md) — All REST endpoints, WebSocket, and MCP tools
- [Architecture Guide](architecture.md) — How the engine works under the hood

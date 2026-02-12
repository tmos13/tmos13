# Pack Development

Write protocols, configure manifests, define commands.

---

## Overview

A **pack** is a self-contained vertical that plugs into the TMOS13 engine. Each pack defines its own personality, modules (cartridges), routing rules, feature flags, and protocol files. The engine is pack-agnostic — all behavior is driven by the pack manifest.

You can build anything: a legal intake flow, a sales qualification agent, a clinical simulation, a customer support system, an onboarding experience. Same engine, different protocol.

---

## Directory Structure

```
protocols/packs/{pack_id}/
  manifest.json          # Required — pack definition
  master.txt             # Required — core identity (always injected)
  metadata.txt           # Required — pack metadata
  boot.txt               # Required — welcome screen
  shutdown.txt           # Required — exit screen
  settings.txt           # Required — settings menu
  diagnostics.txt        # Required — system diagnostics
  utilities.txt          # Required — utility functions
  force_quit.txt         # Required — emergency exit
  {cartridge}.txt        # One per module
  {feature}.txt          # Optional feature-gated content
```

Every `.txt` file referenced in the manifest must exist in the pack directory. The engine validates this at load time and will report exactly which file is missing.

---

## Manifest Schema

The manifest is the single source of truth for your pack. Create `manifest.json` in your pack directory:

```json
{
  "pack_id": "my_pack",
  "name": "My Pack Display Name",
  "version": "1.0",
  "tagline": "Short one-liner description",
  "philosophy": "Guiding design philosophy for this vertical",

  "personality": {
    "tone": "professional, empathetic, clear",
    "warmth": 0.7,
    "humor": 0.3,
    "formality": 0.5,
    "identity_name": "AssistantName",
    "identity_description": "What this AI persona is and does"
  },

  "cartridges": { ... },
  "system_screens": { ... },
  "commands": { ... },
  "features": { ... },
  "settings_defaults": { ... }
}
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `pack_id` | string | Unique identifier (must match directory name) |
| `name` | string | Display name |
| `version` | string | Semantic version |
| `tagline` | string | One-line description |
| `philosophy` | string | Design philosophy guiding the pack's behavior |
| `personality` | object | Identity and tone configuration |
| `cartridges` | object | Module definitions |
| `system_screens` | object | System screen file mappings |
| `commands` | object | Routing rules |
| `features` | object | Feature flag booleans |
| `settings_defaults` | object | Default user settings |

### Personality

Controls the AI persona's voice and behavior:

| Field | Type | Range | Description |
|-------|------|-------|-------------|
| `tone` | string | — | Comma-separated tone keywords |
| `warmth` | float | 0.0–1.0 | How warm and friendly the persona is |
| `humor` | float | 0.0–1.0 | How much humor to inject |
| `formality` | float | 0.0–1.0 | Formal (1.0) vs casual (0.0) |
| `identity_name` | string | — | The persona's name |
| `identity_description` | string | — | What the persona is |

---

## Cartridges (Modules)

Each cartridge is an independent module with its own protocol file and behavior:

```json
"cartridges": {
  "general_support": {
    "file": "general_support.txt",
    "name": "General Support",
    "number": 1,
    "description": "General product questions and troubleshooting"
  },
  "billing": {
    "file": "billing.txt",
    "name": "Billing Help",
    "number": 2,
    "description": "Billing questions and account management",
    "state_class": "BillingState"
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `file` | Yes | Protocol filename (must exist in pack directory) |
| `name` | Yes | Display name |
| `number` | Yes | Menu order (1-indexed) |
| `description` | Yes | Short description |
| `state_class` | No | Custom state class name for structured module state |

---

## Commands

Commands define how user input is routed. Three types, evaluated in strict priority order:

### Numerical Commands (Priority 1)

Exact-match codes that trigger system screens. Zero LLM tokens consumed.

```json
"numerical": {
  "111": { "action": "system", "target": "shutdown" },
  "222": { "action": "system", "target": "settings" },
  "333": { "action": "system", "target": "boot" },
  "444": { "action": "system", "target": "diagnostics" }
}
```

### Session Commands (Priority 2)

Keywords for session control:

```json
"session": {
  "MENU": { "action": "menu", "label": "Open Menu" },
  "EXIT": { "action": "close", "label": "Close Session" },
  "HELLO": { "action": "direct", "response": "Welcome back!" }
}
```

Actions: `menu` (show cartridge list), `close` (end session), `direct` (return inline response).

### Navigation Commands (Priority 3)

Regex patterns that route to cartridges. Compiled at load time.

```json
"navigation": {
  "billing|payment|invoice|charge": { "action": "navigate", "target": "billing" },
  "support|help|troubleshoot": { "action": "navigate", "target": "general_support" }
}
```

The `target` must match a key in `cartridges`. Navigation is disabled when a module is already active, preventing accidental switches mid-conversation.

### Passthrough (Priority 4)

Anything that doesn't match a command passes through to the LLM with the assembled system prompt.

---

## Protocol Files

Protocol files are plain `.txt` files injected into the LLM's system prompt at runtime. No special syntax — just write what you want the AI to know and do.

### master.txt

Always included in every request. Defines the pack's core identity:

```text
# Support Agent — Master Protocol

## Identity
You are Alex, a product support specialist for Acme Corp.

## Core Rules
- Always acknowledge the user's concern before troubleshooting
- Never make up information — if unsure, say so
- Keep responses concise unless the user asks for more detail

## Commands Reference
- 111: Exit session
- 222: Settings
- MENU: See available modules

## Tone
Professional but approachable. Use the user's name when available.
```

### Module Protocol Files

Loaded only when that module is active:

```text
# Billing Help

## Behavior
You are handling billing and account questions. Help with:
- Understanding charges and invoices
- Subscription management
- Payment method updates

## Escalation Rules
For refund amounts over $100, direct the user to support@acme.com.
For disputed charges, collect the transaction ID and date before proceeding.

## State Signals
When the user provides a transaction ID, include [BILLING:TXID=<id>] in your response.
When an issue is resolved, include [BILLING:RESOLVED=true] in your response.
```

### System Screen Files

Returned directly for numerical commands without LLM processing:

- `boot.txt` — Welcome message
- `shutdown.txt` — Goodbye message
- `settings.txt` — User settings display
- `diagnostics.txt` — System status
- `utilities.txt` — Available utilities
- `force_quit.txt` — Emergency exit

---

## State Signals

Modules communicate state changes by embedding signal markers in LLM responses. The engine's parser extracts these before returning clean text to the client.

### Format

```
[MODULE:KEY=VALUE]
```

### Examples

```
[BILLING:STAGE=2]           — Module advanced to stage 2
[BILLING:RESOLVED=true]     — Issue resolved
[MOOD:resonance=frustrated] — User mood detected
[MEMORY:issue_type=refund]  — Stored in cross-module memory
[ACHIEVE:first_resolution]  — Achievement unlocked
```

Instruct the LLM when to emit these in your protocol files. The engine strips them from the response automatically.

---

## Feature Flags

Feature flags gate engine behavior per-pack:

```json
"features": {
  "dinosaur_eggs": false,
  "depth_tracking": true,
  "mood_resonance": true,
  "cross_module_memory": true,
  "real_time_timers": false,
  "audit_trail": true
}
```

| Flag | Description |
|------|-------------|
| `dinosaur_eggs` | Hidden Easter egg content |
| `depth_tracking` | Track exploration depth (0–5 scale) |
| `mood_resonance` | Detect and adapt to user mood |
| `cross_module_memory` | Persist key-value memory across modules |
| `real_time_timers` | Server-enforced countdown timers |
| `audit_trail` | Log all state transitions |

---

## Game Config (Optional)

Pack-specific configuration for custom mechanics:

```json
"game_config": {
  "timer_minutes": { "easy": 75, "normal": 60, "hard": 45 },
  "actions_per_round": 7,
  "scoring_model": "BANT"
}
```

Accessed in engine code via `pack.game_config`.

---

## Validation

The engine validates your manifest at load time — checking required fields, verifying that referenced protocol files exist, compiling regex patterns, and reporting errors with specific messages.

Test locally:

```python
from engine.pack_loader import PackLoader

pack = PackLoader("my_pack")
print(pack.name, pack.version)
print(pack.get_cartridge_names())
```

If validation fails, you'll get a clear error indicating exactly which field or file is missing.

---

## Run Your Pack

```bash
TMOS13_PACK=my_pack uvicorn engine.app:app --reload --port 8000
```

Or pass `pack=` to the API:

```bash
curl -X POST http://localhost:8000/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "hello", "pack_id": "my_pack"}'
```

---

## Multi-Pack Sessions

The engine supports multiple packs simultaneously. Each session tracks its own `pack_id` — different users can interact with different packs on the same deployment.

```bash
# List available packs
curl http://localhost:8000/api/packs

# Start a session with a specific pack
curl -X POST http://localhost:8000/chat \
  -d '{"message": "hello", "pack_id": "legal_intake"}'
```

---

## Template Packs

Use the built-in templates as starting points:

- **`base_simulator`** — Template for simulation-style experiences (roleplay, training, scenarios)
- **`base_quantitative`** — Template for calculator/analysis-style experiences (scoring, evaluation, modeling)

Copy a template directory, rename it, and customize the manifest and protocols.

---

## Next Steps

- [Getting Started](getting-started.md) — Quick setup and first deployment
- [API Reference](api-reference.md) — Endpoints your pack exposes
- [Architecture Guide](architecture.md) — How the engine processes requests

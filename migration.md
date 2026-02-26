# Migration Guide

> **Coming soon.** This guide is under active development.

Moving from custom LLM wrappers to TMOS13.

---

## What's Coming

- **From raw API calls** — Replace ad-hoc prompt management with declarative protocols
- **From LangChain / LlamaIndex** — Migrate chains and agents to TMOS13 packs
- **From custom frameworks** — Map your existing state management to TMOS13's session model
- **Protocol conversion** — Transform existing system prompts into pack protocols
- **State migration** — Import user data and session history
- **Gradual adoption** — Run TMOS13 alongside existing systems during transition

---

## Why Migrate

| Problem with custom wrappers | TMOS13 solution |
|------------------------------|-----------------|
| Prompts hardcoded in application code | Declarative protocol files — edit text, not code |
| No state management across turns | Server-side session state with cross-module memory |
| Every input hits the LLM | Deterministic routing handles ~40% of inputs without API calls |
| Full prompt sent every request | Dynamic assembly reduces tokens by ~80% |
| Switching verticals means rewriting code | Swap the pack manifest, change the product |

---

## Contact

Migration questions? Reach out at dev@tmos13.ai.

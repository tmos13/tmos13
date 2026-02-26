# API Reference

REST endpoints, WebSocket, and MCP tools.

---

## Base URL

```
http://localhost:8000
```

Production deployments use HTTPS with domain-specific base URLs.

## Authentication

Protected endpoints require a Bearer token:

```
Authorization: Bearer <jwt_token>
```

Tokens are issued via `/auth/signin` or OAuth callback. Refresh with `/auth/refresh`.

---

## Core

### POST `/chat`

Send a message and receive a response.

**Request:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `message` | string | Yes | User input text |
| `session_id` | string | No | Existing session ID (omit to create new) |
| `user_id` | string | No | User identifier (defaults to `"anonymous"`) |
| `pack_id` | string | No | Pack override for this session |

**Response:**

```json
{
  "response": "Engine response text",
  "session_id": "uuid",
  "state": {
    "session_id": "uuid",
    "current_game": "module_id",
    "depth": 3,
    "mood": "engaged",
    "turn_count": 12,
    "games_played": ["module_a", "module_b"]
  }
}
```

### WebSocket `/ws`

Real-time bidirectional communication.

**Connect:** `ws://localhost:8000/ws`

**On connect:**
```json
{ "type": "connected", "session_id": "uuid", "message": "Connected to TMOS13." }
```

**Send:**
```json
{ "message": "user input", "user_id": "optional", "pack_id": "optional" }
```

**Receive:**
```json
{ "type": "response", "message": "Engine response", "state": { ... } }
```

### GET `/health`

System health check (no auth required).

```json
{
  "status": "online",
  "engine": "TMOS13",
  "version": "2.2",
  "pack": { "id": "pack_id", "name": "Pack Name", "modules_loaded": 5 },
  "active_sessions": 42,
  "infrastructure": { "database": "supabase", "cache": "redis" },
  "uptime_seconds": 3600.0
}
```

---

## Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/signup` | Register with email and password |
| POST | `/auth/signin` | Sign in, receive JWT |
| POST | `/auth/refresh` | Refresh expired access token |
| POST | `/auth/signout` | Invalidate session |
| GET | `/auth/oauth/{provider}` | Start OAuth flow (apple, google, github) |
| POST | `/auth/oauth/callback` | Complete OAuth flow |
| GET | `/auth/me` | Get authenticated user profile |
| PATCH | `/auth/me` | Update user profile |

---

## Billing (Stripe)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/billing/plans` | No | List available subscription plans |
| POST | `/billing/checkout` | Yes | Create Stripe Checkout session |
| GET | `/billing/subscription` | Yes | Current subscription status |
| POST | `/billing/subscription/cancel` | Yes | Cancel at end of billing period |
| POST | `/billing/subscription/resume` | Yes | Resume cancelled subscription |
| POST | `/billing/portal` | Yes | Open Stripe Customer Portal |
| GET | `/billing/overview` | Yes | Complete billing overview |
| GET | `/billing/orders` | Yes | Order history |
| POST | `/billing/webhook` | Stripe | Stripe webhook (signature verified) |

---

## Mobile Billing (RevenueCat)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/mobile-billing/validate` | Yes | Validate mobile receipt |
| GET | `/mobile-billing/entitlements` | Yes | Active entitlements |
| GET | `/mobile-billing/status` | Yes | Mobile billing status |
| POST | `/mobile-billing/webhook` | RC | RevenueCat webhook |

---

## Audio

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/audio/transcribe` | Speech-to-text (upload audio file) |
| POST | `/audio/synthesize` | Text-to-speech (returns audio) |
| POST | `/audio/chat` | Voice in → process → voice out |
| GET | `/audio/voices` | List available TTS voices |
| GET | `/audio/status` | Audio system status and available providers |

---

## Files

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/files/upload` | Upload a file (multipart form data, 50MB max) |
| GET | `/files/{id}` | Get file metadata |
| GET | `/files` | List files (filterable by session) |
| DELETE | `/files/{id}` | Delete a file |
| GET | `/files/{id}/chunks` | Get file chunks (for large file processing) |

---

## Notes

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/notes` | Create a note |
| GET | `/notes` | List notes |
| GET | `/notes/{id}` | Get a note |
| PUT | `/notes/{id}` | Update a note |
| DELETE | `/notes/{id}` | Delete a note |
| GET | `/notes/search` | Full-text search with TF-IDF ranking |
| GET | `/notes/context` | Get context-relevant notes for a query |
| POST | `/notes/summarize` | Summarize a conversation |
| GET | `/notes/export` | Export notes |

---

## Feed Portal

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/feed/query` | Natural language query → structured data card |
| GET | `/api/feed/history` | Query history |
| GET | `/api/feed/connectors` | List available connectors |

---

## Privacy & Compliance

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/privacy/consent` | Yes | Current consent settings |
| PATCH | `/privacy/consent` | Yes | Update consent preferences |
| GET | `/privacy/export` | Yes | Export all user data (GDPR Art. 15) |
| DELETE | `/privacy/account` | Yes | Delete all user data (GDPR Art. 17) |
| POST | `/privacy/memories/redact` | Yes | Delete specific memories by key |
| POST | `/privacy/session/ephemeral` | Yes | Enable ephemeral mode (no persistence) |
| GET | `/privacy/audit` | Yes | User audit log |
| GET | `/legal/privacy` | No | Privacy policy metadata |

---

## Packs

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/packs` | List all available packs with metadata |
| GET | `/api/packs/{pack_id}` | Detailed metadata for a specific pack |

---

## RAG (Retrieval-Augmented Generation)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/rag/search` | Search protocol index by query |
| POST | `/rag/context` | Get RAG-enriched context fragments |
| GET | `/rag/stats` | Index statistics |
| POST | `/rag/rebuild` | Rebuild search index |

---

## MCP (Model Context Protocol)

TMOS13 exposes an MCP server for external tool integration.

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/mcp/tools` | List available MCP tools with schemas |
| POST | `/mcp/call` | Invoke an MCP tool |
| GET | `/mcp/health` | MCP server health |

**Available Tools:**

| Tool | Description |
|------|-------------|
| `protocol_load` | Load a protocol section by filename |
| `protocol_list` | List all protocol files with sizes |
| `search_protocols` | Full-text search across protocols |
| `state_query` | Query session state (dot-separated field path) |
| `egg_registry` | Query achievement registry |
| `fossil_record` | Get user's discovered achievements |
| `cartridge_info` | Module metadata and session stats |

---

## Storage

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/storage/protocols` | List protocol files in storage |
| POST | `/storage/protocols/sync-up` | Push local protocols to storage |
| POST | `/storage/protocols/sync-down` | Pull protocols from storage to local |
| POST | `/storage/export/{session_id}` | Export session state to storage |

---

## Monitoring

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/metrics` | Prometheus-compatible metrics |
| GET | `/monitoring/dashboard` | Full monitoring dashboard (JSON) |
| GET | `/monitoring/analytics` | Session analytics |

---

## Error Responses

All errors return JSON:

```json
{ "detail": "Error description" }
```

| Status | Meaning |
|--------|---------|
| 400 | Bad request (validation error) |
| 401 | Unauthorized (missing or invalid token) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not found |
| 422 | Unprocessable entity |
| 429 | Rate limited |
| 500 | Internal server error |

## Rate Limits

| Limit | Default | Config |
|-------|---------|--------|
| Requests per minute | 60 | `TMOS13_RATE_LIMIT_RPM` |
| Max message length | 4,000 chars | `TMOS13_MAX_MESSAGE_LENGTH` |
| Max concurrent sessions | 1,000 | `TMOS13_MAX_SESSIONS` |

---

## Next Steps

- [Getting Started](getting-started.md) — Quick setup guide
- [SDK Reference](sdk-reference.md) — TypeScript client that wraps these endpoints
- [Architecture Guide](architecture.md) — How requests flow through the engine

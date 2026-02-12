# Pipeline & Integrations

> **Coming soon.** This guide is under active development.

MCP tools, connectors, scheduled jobs, push notifications, and analytics hooks.

---

## What's Coming

### MCP (Model Context Protocol)
- Exposing TMOS13 as an MCP server for external AI tools and agents
- Tool definitions, schemas, and invocation patterns
- Integration with Claude Desktop, Cursor, and other MCP clients

### Connectors
- **Weather** — Real-time weather data
- **Stocks** — Market data and quotes
- **News** — Headlines and article search
- **Calendar** — Event management
- **Email** — Send and receive
- **CRM** — Customer record integration
- **Custom connectors** — Build your own with the BaseConnector interface

### Feed Portal
- Natural language queries → structured data cards
- Pattern-based intent detection across connectors
- Card formatting and rendering

### Scheduled Jobs
- Recurring tasks and automations
- Session-triggered workflows
- Time-based alerting

### Push Notifications
- Real-time alerts via WebSocket
- Mobile push (APNs, FCM)
- Email notifications

### Analytics Hooks
- Session event streaming
- Custom metric collection
- Integration with PostHog, Segment, and Amplitude

---

## Available Today

The engine already includes:
- **21 MCP tools** for protocol access, state queries, and connector invocation
- **12 connectors** (4 always-enabled: weather, stocks, news, time)
- **Feed Portal** with pattern-based intent routing and typed data cards
- **Prometheus metrics** at `/metrics`
- **PostHog analytics** integration
- **Sentry error tracking**
- **OpenTelemetry tracing**

See the [API Reference](api-reference.md) for current endpoint documentation.

---

## Contact

Integration questions? Reach out at dev@tmos13.ai.

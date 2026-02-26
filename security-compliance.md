# Security & Compliance

> **Coming soon.** Full documentation is under active development.

Data handling, encryption, audit logging, and compliance readiness.

---

## What's Coming

- **Authentication & authorization** — OAuth providers, JWT verification, RBAC (5-level role hierarchy)
- **Data handling** — Session data lifecycle, conversation privacy, protocol file security
- **Network security** — CORS, TLS, rate limiting, input validation
- **Audit trail** — Structured logging, compliance-ready event data, request ID correlation
- **Privacy controls** — GDPR/CCPA endpoints, consent management, data export/deletion
- **SOC 2 readiness** — Trust Services Criteria coverage and implementation status
- **HIPAA readiness** — Air-gapped deployment path for regulated industries
- **Security headers** — HSTS, CSP, X-Frame-Options, and related protections

---

## Highlights

**Implemented today:**
- OAuth authentication (Apple, Google, GitHub)
- Role-based access control (viewer → user → editor → admin → owner)
- Per-session rate limiting
- Input length validation
- GDPR/CCPA privacy endpoints (export, deletion, consent)
- Audit logging on auth events and RBAC denials
- X-Request-ID correlation on all HTTP responses
- Security headers middleware (HSTS, CSP, X-Frame-Options)
- Air-gapped deployment support via local LLM inference

**In progress:**
- SOC 2 Type II preparation
- Formal penetration testing
- Vendor DPA completion

---

## Contact

Security inquiries: security@tmos13.ai

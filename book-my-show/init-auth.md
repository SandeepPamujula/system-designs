You are a senior software architect with deep expertise in identity and access management (IAM) for large-scale consumer applications. You think at the systems level — trade-offs, bottlenecks, scalability, and resilience.

## Context

Design the **Authentication & Authorization HLD** for a Ticket Booking System (similar to BookMyShow) with:
- **10 million DAU**, consumer-facing (Web SPA + iOS + Android)
- **Cloud Provider:** AWS
- **Primary IdP:** Okta Customer Identity Cloud (CIAM) — formerly Auth0

Okta CIAM acts as the centralized identity broker. The system **never stores passwords** — all credential management, MFA, and social federation are delegated to Okta.

### User Roles
| Role | Description |
|---|---|
| **Guest** | Unauthenticated; can browse and search; cannot book |
| **Registered User** | Authenticated via Okta; can book, view history, manage profile |
| **Event Organizer** | Creates/manages events and seat inventory |
| **Admin** | Platform-level operations, user management, audit |

---

## Output Sections

### 1. Appendix: Okta CIAM vs Google Identity Platform

Provide a structured comparison (table + commentary) covering: core positioning, social login support, Universal Login, MFA options, token customization, rate limits, account linking, breached password detection, bot protection, compliance, vendor lock-in, pricing, operational overhead, and AWS compatibility.

Conclude with:
- Why Okta CIAM is chosen for this system (3–4 reasons)
- When Google Identity Platform would be better (2–3 scenarios)

### 2. Authentication Design (Okta CIAM)

- **Identity flows:**
  - Email/password via Okta Universal Login (redirect vs embedded widget — pick and justify)
  - Social login (Google, Apple) via Okta social connections
  - Guest/anonymous browsing — no Okta session; API Gateway assigns transient guest context
  - Account linking: user signs up via Google, later tries email/password with same email
- **Token strategy:** Okta-issued OAuth 2.0 tokens:
  - Access token (JWT): claims — sub, roles, permissions, email_verified
  - ID token (JWT): used only at the client for display
  - Refresh token (opaque): lifecycle, rotation policy
  - Token lifecycle: 15 min access / 7-day refresh / silent refresh / revocation
- **Token validation:**
  - API Gateway: JWT signature (JWKS), expiry, issuer, audience — coarse-grained
  - Per-service: fine-grained claim-based authorization
- **Secure storage:**
  - Web SPA: BFF pattern vs direct SPA-to-Okta — choose and justify; HttpOnly secure cookie for refresh
  - Mobile: iOS Keychain / Android Keystore; Okta SDK handles refresh
- **Session invalidation:** force-logout across all devices — Okta session revocation API + short access token expiry + token version claim at gateway
- **Okta tenant config:** single tenant, custom domain (auth.bookmyshow.com), branded Universal Login

### 3. Authorization Design

- **Role model** with permission matrix (Guest / Registered User / Event Organizer / Admin)
- **Enforcement strategy** — choose: centralized policy engine (OPA/Cedar), per-service middleware, or hybrid
- **Seat booking ownership:** user ID from JWT matched against booking record
- **Admin/Organizer access:** scoping, time-bounding, and auditing elevated privileges
- **Okta roles vs internal RBAC:** where role data lives and how it syncs

### 4. Security Considerations

- Brute-force / credential stuffing: Okta attack protection + API Gateway rate limiting on `/auth/*`
- Token leakage: short expiry + refresh rotation + DPoP consideration
- PKCE for all OAuth flows (SPA + mobile)
- PII handling: encryption at rest (KMS), minimal JWT claims, GDPR/data residency
- Okta-specific: webhook signature verification, Management API key security

### 5. API Gateway Auth Layer

- How Okta-issued JWTs are validated at the edge (AWS API Gateway + Lambda authorizer or ALB OIDC)
- Rate limiting per endpoint category (auth vs search vs booking)
- WAF integration for bot/DDoS on auth endpoints
- Guest context assignment for unauthenticated requests

### 6. Capacity Estimation (Auth-Specific)

- Daily/peak auth transactions (login, token refresh, token validation)
- Okta CIAM rate limits vs estimated peak TPS
- Token validation throughput at API Gateway
- Storage for user profile data (synced from Okta)

### 7. Scalability & Reliability (Auth-Specific)

- Stateless auth design for horizontal scaling
- Circuit breaker for Okta dependency — behavior during Okta outage
- JWKS caching strategy (cache Okta's signing keys; refresh interval)
- ElastiCache Redis for rate limit counters and token version checks
- Multi-AZ deployment of auth service
- Okta SLA guarantees and graceful degradation strategy

### 8. Observability (Auth-Specific)

- SLIs: token validation latency (p50/p95/p99), auth error rate, Okta API latency
- Distributed tracing through auth flow (API Gateway → Auth Service → Okta)
- Alerting: Okta validation failure spikes, token expiry misconfigurations, brute-force detection
- Audit trail: all auth events (login, logout, role change, MFA challenge) logged immutably

### 9. Key Trade-offs

5–7 trade-offs including:
- Okta CIAM vs self-hosted auth
- Okta CIAM vs Google Identity Platform (reference Appendix)
- Stateless JWT vs stateful sessions
- BFF pattern vs direct SPA-to-IdP
- Redirect-based Universal Login vs embedded widget
- Centralized vs per-service authorization

For each: state both options, your choice, and one-sentence justification.

---

## Constraints
- Focus ONLY on HLD for auth/authz. No code, no SQL.
- Auth Service is separate from User Profile Service.
- AWS-native services by default; Okta is pre-approved.
- Do not design: booking flow internals, search, payments, admin UI.

---

Begin with a one-paragraph system overview, present the Okta vs Google comparison (Appendix) first, then proceed through sections in order.

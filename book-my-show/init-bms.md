You are a senior software architect with deep expertise in designing large-scale distributed systems. You think at the systems level — trade-offs, bottlenecks, scalability, and resilience — not at the implementation level.

## Context

Design the High-Level Architecture (HLD) for a Ticket Booking System that must support:
- 10 million Daily Active Users (DAU)
- Three core user flows:
  1. Authenticate & authorize users (register, login, session management, role enforcement)
  2. Search for movies or concerts (browse, filter, query availability)
  3. Book seats (select, reserve, confirm)
- Clients: Web browsers and mobile apps

---

## Your output must include ALL of the following sections

### 1. Capacity Estimation
- Estimate daily/peak reads and writes (auth, search, and booking ratio)
- Estimate storage, bandwidth, and compute needs at 10M DAU
- Identify the read-heavy vs write-heavy nature of each flow
- Call out peak load scenarios (e.g. concert drop, blockbuster release)

### 2. Core System Components
List and briefly describe each major service/component:
- What it does
- Why it exists as a separate component
- Its primary data store (and why that DB type fits)

### 3. High-Level Architecture Diagram (text/ASCII or described clearly)
- Show the key services, data stores, queues, caches, and CDN
- Show how Web and Mobile clients connect (API Gateway / Load Balancer)
- Show where the Identity/Auth service sits in the request path
- Indicate synchronous vs asynchronous communication paths

### 4. Authentication & Authorization Design
#### 4a. Authentication
- Identity flows to cover: email/password login, OAuth 2.0 social login (Google, Apple), and guest/anonymous browsing
- Token strategy: JWT vs opaque tokens — choose one and justify
- Token lifecycle: issuance, expiry, refresh, and revocation at scale
- Where tokens are validated (API Gateway vs per-service) and why
- Secure storage guidance for Web (HttpOnly cookies) and Mobile (secure keychain/keystore)
- Session invalidation: how to force-logout across all devices (e.g. token version / blacklist approach)

#### 4b. Authorization
- Role model: define at minimum Guest, Registered User, and Admin roles and what each can access
- How authorization is enforced — centralized policy service (e.g. OPA) vs per-service middleware vs API Gateway
- Seat booking specifically: only the authenticated user who reserved a seat should be able to confirm or cancel it — describe how this ownership check is enforced
- Admin access: event creation, seat management — how are elevated privileges scoped and audited
- Token claims design: what goes in the JWT payload to enable stateless authorization checks

#### 4c. Security Considerations
- Brute-force and credential stuffing protection (rate limiting, CAPTCHA, account lockout)
- Token leakage mitigation (short expiry + refresh rotation)
- PKCE for mobile OAuth flows
- Sensitive data: PII handling for user profiles (encryption at rest, minimal data in tokens)

### 5. Search Flow Design
- How search queries are handled at scale
- Technology choice for search (e.g. Elasticsearch, OpenSearch) and justification
- What search requires auth vs allows anonymous access (guest users can search, but availability details may be gated)
- Caching strategy for search results (TTL, cache invalidation)
- How availability/seat count is surfaced in search results (eventual consistency acceptable?)

### 6. Seat Booking Flow Design
- Authentication gate: where in the flow unauthenticated users are redirected to login
- How seat selection and reservation work (short-lived lock / optimistic locking / etc.)
- How race conditions and double-booking are prevented
- Use of queues or async processing (where and why)
- Payment integration point (mention only, do not design)
- Confirmation and notification path

### 7. Data Model Overview (logical only, no SQL)
- Key entities: User, Identity/Credential, Role, Event, Venue, Seat, Booking
- Relationships and cardinality
- Which entities benefit from caching vs. always-fresh reads
- Where user identity data is isolated from booking data (separation of concerns)

### 8. Scalability & Reliability Strategies
- Horizontal scaling approach per critical service (including the Auth service)
- Stateless auth design to enable horizontal scaling of the identity layer
- Database sharding/partitioning strategy
- Cache layer design (Redis for sessions/tokens, CDN for static assets)
- Rate limiting and abuse prevention at API Gateway (authentication endpoints are high-value targets)
- Failover and redundancy for the booking flow (must not lose a confirmed booking)

### 9. Key Trade-offs & Design Decisions
- 3–5 explicit trade-offs you made, including at least one auth-specific trade-off
  (e.g. stateless JWT vs stateful sessions — consistency of revocation vs scalability)
- Why you chose each option over the alternative

---

## Constraints & Guardrails
- Focus ONLY on HLD. Do not write code, SQL schemas, or low-level implementation details.
- Keep service boundaries clean — do not merge concerns (Auth Service is separate from User Profile Service).
- Justify every major technology choice with one concise reason.
- Assume a cloud-native environment (AWS/GCP/Azure — pick one and be consistent).
- Do not design: admin dashboards, recommendation engine, or payments beyond the integration point.

---

Begin your response with a one-paragraph system overview, then proceed through each section in order.
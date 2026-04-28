You are a senior software architect with deep expertise in designing large-scale distributed systems. You think at the systems level — trade-offs, bottlenecks, scalability, and resilience — not at the implementation level.

## Context

Design the High-Level Architecture (HLD) for a **Ticket Booking System** (similar to BookMyShow) that must support:
- **10 million Daily Active Users (DAU)**
- **Two core user flows:**
  1. Search for movies or concerts (browse, filter, query availability)
  2. Book seats (select, reserve, confirm)
- **Clients:** Web browsers (SPA) and native mobile apps (iOS, Android)
- **Cloud Provider:** AWS (use AWS-native services where appropriate; justify any third-party alternatives)

## Your output must include ALL of the following sections

### 1. Capacity Estimation
- Estimate daily/peak reads and writes broken down by flow (search, booking)
- Estimate storage, bandwidth, and compute needs at 10M DAU
- Identify the read-heavy vs write-heavy nature of each flow
- Call out peak load scenarios (e.g. concert ticket drop, blockbuster release day)

### 2. Core System Components
List and briefly describe each major service/component:
- What it does
- Why it exists as a separate component
- Its primary data store (and why that DB type fits)
- How it communicates with other services (sync REST/gRPC vs async events)

Include at minimum:
- API Gateway (AWS API Gateway or ALB)
- User Profile Service
- Search Service
- Booking Service
- Notification Service
- Event/Show Catalog Service
- Venue & Seat Inventory Service

### 3. API Gateway & Edge Layer Design
- How Web (SPA) and Mobile clients connect — API Gateway, ALB, CloudFront CDN
- Request routing strategy (path-based, header-based)
- Rate limiting and throttling strategy per endpoint category (search vs booking)
- Request/response transformation, CORS, and API versioning
- WAF integration for DDoS and bot protection

### 4. Search Flow Design
- How search queries are handled at scale (query parsing, ranking, pagination)
- Technology choice: Amazon OpenSearch Service — justify over alternatives (Elasticsearch self-hosted, Algolia)
- Caching strategy: CloudFront for popular queries, ElastiCache (Redis) for search result caching — TTL and invalidation strategy
- How availability/seat count is surfaced in search results (eventual consistency acceptable? what staleness window?)
- Index update strategy: how show/event catalog changes propagate to the search index (CDC, event-driven, or batch)

### 5. Seat Booking Flow Design
- How seat selection and reservation work (short-lived distributed lock using Redis / DynamoDB conditional writes / optimistic locking)
- How race conditions and double-booking are prevented — compare at least two approaches and pick one
- Use of SQS/SNS or EventBridge for async processing (where and why)
- Payment integration point (mention only — do not design the payment system)
- Confirmation and notification path (email via SES, push via SNS Mobile Push)
- Timeout and expiry of held seats if payment is not completed

### 6. Event-Driven Architecture
- Define the domain events that flow through the system (e.g. BookingCreated, BookingConfirmed, SeatReleased, ShowUpdated)
- Event bus technology: Amazon EventBridge vs SNS+SQS fan-out — choose and justify
- Event schema strategy (schema registry, versioning)
- Which interactions are synchronous (booking confirmation) vs asynchronous (notifications, search index update, analytics)
- Idempotency and exactly-once processing guarantees for critical events (booking, payment callback)
- Dead-letter queue strategy for failed event processing

### 7. Data Model Overview (logical only, no SQL)
- Key entities: User, Role, Event/Show, Venue, Screen/Hall, Seat, Booking, Payment Reference
- Relationships and cardinality
- Which entities benefit from caching vs. always-fresh reads
- Data partitioning strategy by entity (e.g. bookings partitioned by event date or region)

### 8. Scalability & Reliability Strategies
- Horizontal scaling approach per critical service (ECS Fargate / EKS / Lambda — pick and justify)
- Database strategy: Aurora PostgreSQL for transactional data, DynamoDB for high-throughput seat locks — justify choices
- Cache layer: ElastiCache Redis for session-adjacent data (rate limit counters, seat hold TTLs), CloudFront for static assets and CDN
- Auto-scaling policies: target-tracking on request count / latency for booking service, scheduled scaling for known peak events
- Rate limiting at API Gateway: differentiated limits for search and booking

### 9. Observability & Monitoring
- **Metrics:** key SLIs per service — latency (p50/p95/p99), error rate, throughput
- **Distributed tracing:** AWS X-Ray or OpenTelemetry — trace a booking request end-to-end from API Gateway through Booking → Payment → Notification
- **Logging:** structured logging strategy (JSON logs → CloudWatch Logs → OpenSearch for search/alerting)
- **Alerting:** critical alerts — booking error rate > threshold, seat lock contention
- **Dashboards:** business-level (bookings/min, conversion funnel) + infra-level (CPU, memory, DB connections)
- **Audit trail:** booking state transitions logged immutably for compliance

### 10. Disaster Recovery & Multi-Region
- RPO and RTO targets for the booking system
- Multi-AZ deployment as baseline (all critical services across ≥ 2 AZs)
- Multi-region strategy: active-passive vs active-active — choose and justify for a booking system
- Data replication: Aurora Global Database for cross-region, DynamoDB Global Tables for seat inventory
- Runbook: how to failover the booking flow to a secondary region

### 11. Key Trade-offs & Design Decisions
- 5–7 explicit trade-offs you made, including:
  - **Strong consistency (booking) vs eventual consistency (search/availability)** — where each is appropriate
  - **Synchronous vs asynchronous** — which flows and why
- For each trade-off: state the two options, which you chose, and a one-sentence justification

---

## Constraints & Guardrails
- Focus ONLY on HLD. Do not write code, SQL schemas, or low-level implementation details.
- Keep service boundaries clean.
- Justify every major technology choice with one concise reason.
- Use **AWS** as the cloud provider. Use AWS-native services by default; justify any third-party choice.
- Do not design: admin dashboards UI, recommendation engine, payments beyond the integration point, or CI/CD pipeline.

---

Begin your response with a one-paragraph system overview, then proceed through each section in order.

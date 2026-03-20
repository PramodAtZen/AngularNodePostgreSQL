# Research: Real Estate Management System

## Decision 1: Use TypeORM with PostgreSQL (including JSONB amenities)
- Decision: Use TypeORM as the backend ORM with PostgreSQL 16 and JSONB for dynamic property amenities.
- Rationale: Meets SQL injection prevention requirements through parameterized queries, supports migrations and relational constraints, and provides first-class JSONB mapping for amenity extensibility.
- Alternatives considered:
  - Sequelize: mature but less ergonomic for strongly typed domain modeling in this project context.
  - Raw SQL query builder only: more control but higher risk of inconsistency and slower delivery for multi-entity workflows.

## Decision 2: Enforce approval integrity with centralized permission middleware + domain guards
- Decision: Implement request-level RBAC middleware and domain-level workflow guards that explicitly block self-approval and invalid status transitions.
- Rationale: Double enforcement prevents bypass paths and ensures compliance-grade integrity checks for approval actions.
- Alternatives considered:
  - UI-only authorization checks: insufficient, easy to bypass.
  - Database triggers only: useful for hard guarantees but insufficient for role/context-rich decision feedback.

## Decision 3: API contract-first REST with OpenAPI
- Decision: Define REST endpoints and schemas in OpenAPI first, then validate implementation with contract tests.
- Rationale: Aligns with constitution testing gate and reduces integration drift between Angular frontend and Node backend.
- Alternatives considered:
  - Implicit contract through implementation only: high risk of API mismatch.
  - GraphQL: not required for current scope and increases complexity.

## Decision 4: Notification architecture for status changes
- Decision: Trigger asynchronous email notifications on key property status transitions (Pending Approval, Changes Requested, Published, Sold, Archived).
- Rationale: Keeps write-path latency controlled while ensuring stakeholders receive lifecycle updates.
- Alternatives considered:
  - Synchronous email send in request path: increases tail latency and failure coupling.
  - In-app notifications only: does not satisfy explicit email notification requirement.

## Decision 5: Angular UX strategy with lazy loading + SSR
- Decision: Use Angular route-level lazy loading for major features and Angular Universal SSR for listing search pages.
- Rationale: Meets non-functional requirements for fast page transitions and SEO indexability of listing pages.
- Alternatives considered:
  - CSR-only Angular: weaker SEO discoverability.
  - Full static prerender only: insufficient for dynamic search and filter combinations.

## Decision 6: Performance strategy for search/autocomplete/analytics
- Decision: Use indexed search fields, pagination defaults, selective projections, and read-optimized analytics aggregations.
- Rationale: Supports p95 goals under expected load while keeping API responses bounded and predictable.
- Alternatives considered:
  - Full-text engine in phase 1: potentially stronger relevance but unnecessary complexity for initial release.
  - Unbounded query responses: unacceptable latency risk.

## Decision 7: Security baseline for web architecture
- Decision: Apply CSRF tokens for state-changing web requests, strict input validation, safe ORM parameterization, and audit logging for sensitive actions.
- Rationale: Directly addresses stated security requirements and constitution principles.
- Alternatives considered:
  - Token auth without CSRF controls in browser context: inadequate for cookie/session-backed flows.
  - Minimal audit logging: insufficient for approval accountability.

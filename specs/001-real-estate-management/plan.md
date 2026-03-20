# Implementation Plan: Real Estate Management System

**Branch**: `001-real-estate-management` | **Date**: 2026-03-20 | **Spec**: `/workspaces/AngularNodePostgreSQL/specs/001-real-estate-management/spec.md`
**Input**: Feature specification from `/workspaces/AngularNodePostgreSQL/specs/001-real-estate-management/spec.md`

## Summary

Build a role-based real estate platform with property onboarding, multi-level
approvals, public listing discovery, lead capture, and listing analytics.
Architecture uses Angular (with lazy-loaded routes and SSR for SEO) and a
Node.js REST API backed by PostgreSQL, with strict workflow authorization,
audit logging, and notification triggers on status changes.

## Technical Context

**Language/Version**: TypeScript 5.x (Node.js 20 LTS backend, Angular 19 frontend)  
**Primary Dependencies**: Express, TypeORM, class-validator, Angular Router, Angular Universal SSR, Nodemailer, JWT auth middleware  
**Storage**: PostgreSQL 16 (relational core + JSONB amenities for property flexibility)  
**Testing**: Jest + Supertest (backend unit/integration), Angular TestBed + Karma or Jest (frontend unit), Playwright (end-to-end), contract tests against OpenAPI  
**Target Platform**: Linux containerized web deployment (API + SSR frontend)
**Project Type**: Web application (frontend + backend)  
**Performance Goals**: Search API p95 <= 2s, autocomplete API p95 <= 800ms, analytics API p95 <= 2s, status visibility propagation <= 60s  
**Constraints**: Role-based authorization on all protected actions, no self-approval by listing owner, CSRF protection, SQL injection prevention via ORM parameterization, auditable approval history  
**Scale/Scope**: Initial release for internal property operations and buyer discovery, supports concurrent active listings with role-based operational dashboards

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Pre-Design Gate Review

- Code Quality Gate: PASS. ESLint + Prettier + pull request review rules and
  layered service architecture are defined.
- Testing Gate: PASS. Plan includes unit, integration, e2e, and contract tests;
  failing-first approach applied for business-critical approval and permission
  flows.
- UX Consistency Gate: PASS. Multi-page onboarding behavior, state feedback,
  status badges, and accessible interaction requirements are mapped.
- Performance Gate: PASS. Explicit p95 budgets and validation approach are
  defined for search, autocomplete, and analytics.
- Simplicity and Security Gate: PASS. Single REST backend with clear module
  boundaries, strict authorization middleware, CSRF protection, and safe input
  handling.

### Post-Design Gate Review

- Code Quality Gate: PASS. Data model, contracts, and quickstart include
  enforceable quality checks and module ownership.
- Testing Gate: PASS. Contract artifacts and quickstart validation path define
  required test coverage.
- UX Consistency Gate: PASS. UX states and role-specific journeys are encoded
  in quickstart scenarios.
- Performance Gate: PASS. Research decisions include indexing, pagination, and
  query strategies mapped to performance objectives.
- Simplicity and Security Gate: PASS. Design avoids unnecessary service sprawl
  and uses centralized permission guards plus audit-first write paths.

## Project Structure

### Documentation (this feature)

```text
specs/001-real-estate-management/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── openapi.yaml
└── tasks.md
```

### Source Code (repository root)

```text
backend/
├── src/
│   ├── api/
│   │   ├── controllers/
│   │   ├── middleware/
│   │   └── routes/
│   ├── domain/
│   │   ├── properties/
│   │   ├── approvals/
│   │   ├── leads/
│   │   └── analytics/
│   ├── infra/
│   │   ├── db/
│   │   ├── notifications/
│   │   └── auth/
│   └── app.ts
└── tests/
    ├── contract/
    ├── integration/
    └── unit/

frontend/
├── src/
│   ├── app/
│   │   ├── features/
│   │   │   ├── onboarding/
│   │   │   ├── approvals/
│   │   │   ├── listing-search/
│   │   │   └── dashboard/
│   │   ├── core/
│   │   └── shared/
│   └── main.server.ts
└── tests/
    ├── e2e/
    └── unit/
```

**Structure Decision**: Web application split into `backend/` and `frontend/`
to satisfy API security boundaries, Angular SSR SEO requirements, and modular
feature ownership.

## Complexity Tracking

No constitution violations requiring justification.

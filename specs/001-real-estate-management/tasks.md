# Tasks: Real Estate Management System

**Feature Branch**: `001-real-estate-management`
**Input**: Design documents from `specs/001-real-estate-management/`
**Prerequisites**: plan.md ✅, spec.md ✅, data-model.md ✅, contracts/openapi.yaml ✅, research.md ✅, quickstart.md ✅

**Tech Stack**: TypeScript 5.x · Node.js 20 LTS · Express · TypeORM · PostgreSQL 16 · Angular 19 · Angular Universal SSR · JWT · Nodemailer  
**Structure**: `backend/` (REST API) + `frontend/` (Angular SSR app)

---

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no blocking dependencies)
- **[US#]**: User story this task belongs to (US1, US2, US3)
- Exact file paths are included in every task description

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Monorepo initialization, tooling, and environment scaffolding. Must complete before any foundational or story work.

- [ ] T001 Initialize monorepo with `backend/` and `frontend/` directories and root `package.json` with npm workspaces per plan.md project structure
- [ ] T002 [P] Initialize Node.js 20 TypeScript 5.x backend in `backend/` with Express, TypeORM, class-validator, Nodemailer, and JWT dependencies in `backend/package.json` and `backend/tsconfig.json`
- [ ] T003 [P] Initialize Angular 19 frontend in `frontend/` with Angular Universal SSR, Angular Router, and HttpClient in `frontend/package.json` and `frontend/tsconfig.json`
- [ ] T004 [P] Configure ESLint and Prettier for backend TypeScript in `backend/.eslintrc.json` and `backend/.prettierrc`
- [ ] T005 [P] Configure ESLint and Prettier for Angular frontend in `frontend/.eslintrc.json` and `frontend/.prettierrc`
- [ ] T006 Create `backend/.env.example` with all required variables: `DATABASE_URL`, `JWT_SECRET`, `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `APP_BASE_URL` per `quickstart.md` section 1
- [ ] T007 Define root npm workspace scripts in `package.json` at repository root: `backend:dev`, `backend:migrate`, `backend:build`, `frontend:dev:ssr`, `lint`, `test:unit`, `test:integration`, `test:contract`, `test:e2e`, `perf:search`, `perf:autocomplete`, `perf:analytics` per `quickstart.md`

**Checkpoint**: Project scaffolding is complete — foundational infrastructure can now begin.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure — database entities, auth, middleware, and Angular core — that MUST be complete before any user story work begins.

**⚠️ CRITICAL**: No user story work can begin until this phase is complete.

### Database Layer

- [ ] T008 Configure TypeORM `DataSource` with PostgreSQL 16 connection, entity registration, and migrations directory in `backend/src/infra/db/data-source.ts`
- [ ] T009 Create `User` TypeORM entity with UUID primary key, role enum (`AGENT_VENDOR`, `ADMIN_MANAGER`, `END_USER_BUYER`, `SYSTEM_ADMIN`), `is_active`, and timestamps in `backend/src/domain/properties/entities/user.entity.ts`
- [ ] T010 [P] Create `Property` TypeORM entity with status enum (`DRAFT`, `PENDING_APPROVAL`, `CHANGES_REQUESTED`, `PUBLISHED`, `SOLD`, `ARCHIVED`), JSONB `amenities`, location fields, and `owner_user_id` FK in `backend/src/domain/properties/entities/property.entity.ts`
- [ ] T011 [P] Create `PropertySubmission` TypeORM entity with `submission_version` (auto-increment per property), `page_payload` JSONB, `submitted_by_user_id` FK, and `submitted_at` nullable timestamp in `backend/src/domain/properties/entities/property-submission.entity.ts`
- [ ] T012 [P] Create `ApprovalStage` TypeORM entity with `stage_order`, `approver_role` enum, `status` enum (`PENDING`, `APPROVED`, `CHANGES_REQUESTED`), `acted_by_user_id` FK, and `comments` in `backend/src/domain/approvals/entities/approval-stage.entity.ts`
- [ ] T013 [P] Create `Lead` TypeORM entity with `buyer_name`, `buyer_email`, `buyer_phone`, `message`, `status` enum (`NEW`, `CONTACTED`, `QUALIFIED`, `CLOSED`), and `property_id` FK in `backend/src/domain/leads/entities/lead.entity.ts`
- [ ] T014 [P] Create `ListingAnalyticsEvent` TypeORM entity with `event_type` enum (`VIEW`, `INQUIRY`), `actor_user_id` nullable FK, `occurred_at`, and `metadata` JSONB in `backend/src/domain/analytics/entities/listing-analytics-event.entity.ts`
- [ ] T015 [P] Create `AuditLog` TypeORM entity with `actor_user_id` FK, `entity_type`, `entity_id`, `action`, `previous_state` JSONB, `next_state` JSONB, `reason`, `occurred_at`, and append-only constraint in `backend/src/infra/db/entities/audit-log.entity.ts`
- [ ] T016 Generate and run consolidated TypeORM migration for all entities (User, Property, PropertySubmission, ApprovalStage, Lead, ListingAnalyticsEvent, AuditLog) in `backend/src/infra/db/migrations/`

### Auth and Middleware Layer

- [ ] T017 Implement JWT authentication middleware that extracts and verifies bearer token, attaches `req.user` with `id` and `role`, and returns `401` on invalid or missing token in `backend/src/infra/auth/jwt.middleware.ts`
- [ ] T018 [P] Implement RBAC permission middleware factory that accepts allowed roles array and returns `403` with denial record for unauthorized attempts, including self-approval guard interface in `backend/src/api/middleware/rbac.middleware.ts`
- [ ] T019 [P] Implement CSRF protection middleware for all state-changing requests (`POST`, `PATCH`, `DELETE`) in `backend/src/api/middleware/csrf.middleware.ts` per research Decision 7
- [ ] T020 [P] Implement centralized error handling middleware with structured JSON error responses and safe error message sanitization in `backend/src/api/middleware/error.middleware.ts`
- [ ] T021 Configure Express app in `backend/src/app.ts` registering middleware order: CSRF → JWT → RBAC → routes → error handler, plus JSON body parsing and request logging

### Shared Services

- [ ] T022 [P] Implement `AuditLogService` with append-only `record()` method accepting `actor_user_id`, `entity_type`, `entity_id`, `action`, and state snapshots in `backend/src/domain/properties/services/audit-log.service.ts`
- [ ] T023 [P] Implement async `NotificationService` using Nodemailer to send emails on status transitions (`PENDING_APPROVAL`, `CHANGES_REQUESTED`, `PUBLISHED`, `SOLD`, `ARCHIVED`) without blocking request path in `backend/src/infra/notifications/notification.service.ts`

### Angular Core

- [ ] T024 Implement Angular `CoreModule` with `AuthService` (JWT storage and decoding), `AuthInterceptor` (bearer token injection), `AuthGuard` (route protection by role), and `RoleDirective` (template-level role visibility) in `frontend/src/app/core/`

**Checkpoint**: All foundational infrastructure is in place — user story phases can now be executed in parallel.

---

## Phase 3: User Story 1 — Property Onboarding With Multi-Level Approval (Priority: P1) 🎯 MVP

**Goal**: Enable Agents/Vendors to onboard properties through a multi-page draft form, submit for approval, and pass through multi-level review stages until publication, with audit trail throughout.

**Independent Test**: Create a property as Agent/Vendor → save draft → submit → Admin/Manager approves stage 1 → System Admin completes final approval → verify `PUBLISHED` status, AuditLog entries, and notification dispatch. Confirm self-approval is blocked and `Changes Requested` status rolls back correctly.

### Backend — Business Logic

- [ ] T025 [P] [US1] Implement `PropertyService` with methods: `createDraft()`, `updateDraft()`, `getById()`, `listByRole()`, and `transitionStatus()` enforcing state machine rules (`DRAFT→PENDING_APPROVAL`, `PUBLISHED→SOLD/ARCHIVED`, etc.) and owner self-approval prevention in `backend/src/domain/properties/services/property.service.ts`
- [ ] T026 [P] [US1] Implement `ApprovalService` with methods: `submitForApproval()` (creates `PropertySubmission` and `ApprovalStage` records), `approveStage()` (validates approver role, increments stage, publishes on final approval), `requestChanges()` (resets to `CHANGES_REQUESTED`), and `getApprovalHistory()` in `backend/src/domain/approvals/services/approval.service.ts`

### Backend — API Layer

- [ ] T027 [US1] Implement `PropertyController` handling `POST /properties` (create draft), `GET /properties` (role-scoped list), `GET /properties/:propertyId` (get by id), `PATCH /properties/:propertyId` (update draft or changes-requested property) with input validation via class-validator in `backend/src/api/controllers/property.controller.ts`
- [ ] T028 [US1] Implement `ApprovalController` handling `POST /properties/:propertyId/submit`, `POST /properties/:propertyId/approve` (with self-approval guard returning `403`), `POST /properties/:propertyId/request-changes`, and `PATCH /properties/:propertyId/status` (`SOLD`/`ARCHIVED` transitions) in `backend/src/api/controllers/approval.controller.ts`
- [ ] T029 [US1] Register property and approval routes with appropriate RBAC guards (Agents/Vendors for draft, Admins/Managers for approval actions) in `backend/src/api/routes/property.routes.ts` and `backend/src/api/routes/approval.routes.ts`
- [ ] T030 [P] [US1] Implement `UserController` handling `POST /users` (create user) and `GET /users/:userId` (get user) with SYSTEM_ADMIN guard on create in `backend/src/api/controllers/user.controller.ts` and `backend/src/api/routes/user.routes.ts`

### Frontend — Onboarding Feature

- [ ] T031 [P] [US1] Build Angular multi-page property onboarding feature module with step-by-step form (basic details, location, pricing, amenities), progress indicator component, and draft auto-save on navigation in `frontend/src/app/features/onboarding/`
- [ ] T032 [US1] Build Angular approval queue view for Admin/Manager role showing pending submissions with stage details, approve button, and request-changes form in `frontend/src/app/features/approvals/approval-queue.component.ts`
- [ ] T033 [P] [US1] Create shared `StatusBadgeComponent` rendering color-coded badges for all six property statuses and `StatusTransitionPromptComponent` for confirmation dialogs per UX-002 in `frontend/src/app/shared/components/status-badge/` and `frontend/src/app/shared/components/status-transition-prompt/`
- [ ] T034 [US1] Add reactive form validation, unsaved-change route guard (`CanDeactivate`), accessible labels, and visible focus states to the onboarding form per UX-001 and UX-004 in `frontend/src/app/features/onboarding/`
- [ ] T035 [US1] Wire onboarding and approvals features as lazy-loaded routes with `AuthGuard` role checks and integrate `PropertyApiService` and `ApprovalApiService` HTTP clients in `frontend/src/app/app-routing.module.ts` and `frontend/src/app/core/services/`

**Checkpoint**: User Story 1 is fully functional and independently testable. Property lifecycle from Draft to Published is operational.

---

## Phase 4: User Story 2 — Public Listing Discovery For Buyers (Priority: P2)

**Goal**: Enable buyers to search published properties using keyword and structured filters, with fast autocomplete suggestions, SSR-rendered pages, and empty-state handling. Only `PUBLISHED` properties are ever exposed.

**Independent Test**: Publish sample properties, query `GET /search?q=...&city=...` and `GET /search/autocomplete?term=...`, verify only `PUBLISHED` results appear, empty state returns for no-match queries, and autocomplete suggestions are ranked. Confirm SSR HTML is pre-rendered for the listing search route.

### Backend — Search Infrastructure

- [ ] T036 [P] [US2] Create TypeORM migration adding GIN/B-tree indexes on `Property`: `status`, `location_city`, `title`, and composite `(status, location_city)` for search query performance per research Decision 6 in `backend/src/infra/db/migrations/`
- [ ] T037 [P] [US2] Implement `SearchService` with `search()` method applying `status = PUBLISHED` filter enforced at query level, keyword full-text match on `title`/`description`, city/price range filters, and cursor-based pagination (page/pageSize) in `backend/src/domain/properties/services/search.service.ts`
- [ ] T038 [P] [US2] Implement `AutocompleteService` with `suggest()` method returning ranked title and city prefix matches for published listings with result capped at 10 suggestions in `backend/src/domain/properties/services/autocomplete.service.ts`

### Backend — API Layer

- [ ] T039 [US2] Implement `SearchController` handling public `GET /search` (no auth required) with query params `q`, `city`, `minPrice`, `maxPrice`, `page`, `pageSize` and public `GET /search/autocomplete` with `term` param; register routes without JWT middleware in `backend/src/api/controllers/search.controller.ts` and `backend/src/api/routes/search.routes.ts`

### Frontend — Listing Search Feature

- [ ] T040 [P] [US2] Build Angular SSR-enabled listing search page with keyword input, city filter, price range inputs, paginated results grid, and property card component in `frontend/src/app/features/listing-search/listing-search.component.ts`
- [ ] T041 [P] [US2] Build shared `AutocompleteInputComponent` with 300 ms debounce, keyboard navigation (arrow keys, Enter, Escape), and ARIA `combobox` attributes per UX-004 in `frontend/src/app/shared/components/autocomplete-input/`
- [ ] T042 [US2] Implement standardized `LoadingSpinnerComponent`, `EmptyStateComponent` (with recovery action slot), and `InlineErrorComponent` per UX-003 in `frontend/src/app/shared/components/`
- [ ] T043 [US2] Configure Angular Universal SSR for the listing-search route: add route to SSR transfer-state, configure `server.ts` Express handler, and register `main.server.ts` entry point in `frontend/src/main.server.ts` and `frontend/server.ts`
- [ ] T044 [US2] Wire listing-search lazy-loaded route, integrate `SearchApiService` and `AutocompleteApiService` HTTP clients, and apply `LoadingSpinner`/`EmptyState`/`InlineError` states in `frontend/src/app/app-routing.module.ts` and `frontend/src/app/core/services/`

**Checkpoint**: User Story 2 is fully functional and independently testable. Public listing discovery is operational without affecting US1.

---

## Phase 5: User Story 3 — Lead Handling and Listing Analytics Dashboard (Priority: P3)

**Goal**: Enable buyers to submit inquiries on published listings and enable Agents/Vendors/Admins to view listing-level analytics (views, inquiry counts) in a role-scoped dashboard. Historical data persists for archived/sold listings.

**Independent Test**: Submit inquiry as Buyer on a Published listing, verify `Lead` record and `INQUIRY` analytics event are created. Open Agent/Vendor dashboard and confirm view/inquiry counts are accurate and role-scoped. Verify Archived/Sold listings remain in internal analytics history but are excluded from public discovery.

### Backend — Business Logic

- [ ] T045 [P] [US3] Implement `AnalyticsEventService` with `trackView()` and `trackInquiry()` methods that insert `ListingAnalyticsEvent` records asynchronously without blocking the request path in `backend/src/domain/analytics/services/analytics-event.service.ts`
- [ ] T046 [P] [US3] Implement `LeadService` with `createLead()` validating property is `PUBLISHED`, deduplication check (same `buyer_email` + `property_id` within 24 hours returns existing lead), and `listLeadsForProperty()` with role scope enforcement in `backend/src/domain/leads/services/lead.service.ts`
- [ ] T047 [P] [US3] Implement `AnalyticsService` with `getListingMetrics()` aggregating `VIEW` and `INQUIRY` counts per property, scoped to `owner_user_id` for Agent/Vendor role and unscoped for Admin/Manager, including archived/sold listings in `backend/src/domain/analytics/services/analytics.service.ts`

### Backend — API Layer

- [ ] T048 [US3] Implement `LeadController` handling public `POST /properties/:propertyId/leads` (no auth required, validates `PUBLISHED` status, triggers `INQUIRY` analytics event and async notification) in `backend/src/api/controllers/lead.controller.ts` and `backend/src/api/routes/lead.routes.ts`
- [ ] T049 [US3] Implement `AnalyticsController` handling authenticated `GET /analytics/listings` with optional `ownerId` query param, enforcing that Agent/Vendor can only query their own listings, in `backend/src/api/controllers/analytics.controller.ts` and `backend/src/api/routes/analytics.routes.ts`

### Frontend — Dashboard and Lead Features

- [ ] T050 [P] [US3] Build Angular `LeadInquiryFormComponent` on the published listing detail page with `buyer_name`, `buyer_email`, `buyer_phone`, `message` fields, accessible validation feedback, and success confirmation state per UX-003 in `frontend/src/app/features/listing-search/lead-inquiry-form/`
- [ ] T051 [P] [US3] Build Angular `DashboardComponent` with listings metrics table showing `views` and `inquiries` per property, sortable columns, and last-updated timestamp in `frontend/src/app/features/dashboard/dashboard.component.ts`
- [ ] T052 [US3] Implement role-based dashboard data scoping so Agent/Vendor only sees their own listing metrics (using `ownerId` query param) while Admin/Manager sees all listings; include Archived/Sold listings in the table with a status column in `frontend/src/app/features/dashboard/`
- [ ] T053 [US3] Wire dashboard as lazy-loaded route with `AuthGuard` restricting access to `AGENT_VENDOR`, `ADMIN_MANAGER`, and `SYSTEM_ADMIN` roles, integrate `AnalyticsApiService` and `LeadApiService` HTTP clients in `frontend/src/app/app-routing.module.ts` and `frontend/src/app/core/services/`

**Checkpoint**: All three user stories are fully functional and independently testable. The complete feature is ready for cross-cutting polish.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Performance optimization, end-to-end validation, security hardening, and UX consistency review across all user stories.

- [ ] T054 [P] Create additional TypeORM migration adding read-optimized indexes for analytics aggregation: index on `ListingAnalyticsEvent(property_id, event_type)` and `Lead(property_id)` per PRF-003 in `backend/src/infra/db/migrations/`
- [ ] T055 [P] Implement performance validation scripts using `autocannon` or `k6` for `perf:search` (p95 ≤ 2s), `perf:autocomplete` (p95 ≤ 800ms), and `perf:analytics` (p95 ≤ 2s) per quickstart.md section 8 and PRF-001 to PRF-003 in `backend/tests/performance/`
- [ ] T056 [P] Write Playwright end-to-end tests covering Flow A (onboarding → approval → published), Flow B (search → autocomplete → empty state), and Flow C (inquiry → analytics dashboard) per quickstart.md section 6 in `frontend/tests/e2e/`
- [ ] T057 Validate UX consistency across all features: confirm status badge variants match spec, loading/empty/error states are present on all async views, and form keyboard operability meets UX-001 through UX-004
- [ ] T058 [P] Security hardening review: verify CSRF token enforcement on all `POST`/`PATCH` endpoints, confirm all TypeORM queries use parameterized inputs, audit self-approval denial audit log entries, and validate `403` responses for all unauthorized role action attempts per research Decision 7 and FR-014
- [ ] T059 Run full quickstart.md validation: execute Flow A, B, and C manually; run `npm run lint`, `npm run test:unit`, `npm run test:integration`, `npm run test:contract`, and `npm run test:e2e`; confirm all gates pass per quickstart.md section 7

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1: Setup         → No dependencies — start immediately
Phase 2: Foundational  → Depends on Phase 1 — BLOCKS all user story phases
Phase 3: US1 (P1)      → Depends on Phase 2 — can start as soon as Phase 2 completes
Phase 4: US2 (P2)      → Depends on Phase 2 — independent of US1
Phase 5: US3 (P3)      → Depends on Phase 2 — independent of US1, US2
Phase 6: Polish        → Depends on all desired story phases complete
```

### User Story Dependencies

| Story | Depends On | Shares With |
|---|---|---|
| **US1** (P1) — Onboarding & Approval | Phase 2 only | Property entity, AuditLog, Notifications |
| **US2** (P2) — Listing Discovery | Phase 2 only (T036 indexes after T016 migration) | Property entity (read-only, `PUBLISHED` filter) |
| **US3** (P3) — Leads & Analytics | Phase 2 only; US1 needed for published listings in e2e tests | Property entity, Lead entity, AnalyticsEvent entity |

> All three user stories are independently implementable once Phase 2 is complete.
> US1 is the natural prerequisite for full e2e testing of US2 and US3 (published listings), but the API implementations are fully independent.

### Within Each User Story (Internal Order)

1. Backend services (business logic, guards) → Backend controllers/routes → Frontend feature module → Frontend routing
2. [P]-marked tasks within each layer can run simultaneously

---

## Parallel Execution Examples

### Phase 2 — Parallel Entity Creation (T009–T015)

```bash
# All seven entity files are independent — run simultaneously:
T009: User entity              → backend/src/domain/properties/entities/user.entity.ts
T010: Property entity          → backend/src/domain/properties/entities/property.entity.ts
T011: PropertySubmission entity → backend/src/domain/properties/entities/property-submission.entity.ts
T012: ApprovalStage entity     → backend/src/domain/approvals/entities/approval-stage.entity.ts
T013: Lead entity              → backend/src/domain/leads/entities/lead.entity.ts
T014: ListingAnalyticsEvent    → backend/src/domain/analytics/entities/listing-analytics-event.entity.ts
T015: AuditLog entity          → backend/src/infra/db/entities/audit-log.entity.ts
```

### Phase 3 — User Story 1 Parallel Work

```bash
# Backend services (different domain files):
T025: PropertyService  → backend/src/domain/properties/services/property.service.ts
T026: ApprovalService  → backend/src/domain/approvals/services/approval.service.ts
T030: UserController   → backend/src/api/controllers/user.controller.ts

# Frontend components (different feature directories):
T031: Onboarding form  → frontend/src/app/features/onboarding/
T033: StatusBadge      → frontend/src/app/shared/components/status-badge/
```

### Phase 4 — User Story 2 Parallel Work

```bash
# Backend services (different files):
T037: SearchService        → backend/src/domain/properties/services/search.service.ts
T038: AutocompleteService  → backend/src/domain/properties/services/autocomplete.service.ts

# Frontend components (different files):
T040: ListingSearchPage      → frontend/src/app/features/listing-search/
T041: AutocompleteComponent  → frontend/src/app/shared/components/autocomplete-input/
T042: Shared UX components   → frontend/src/app/shared/components/
```

### Phase 5 — User Story 3 Parallel Work

```bash
# Backend services (different domain modules):
T045: AnalyticsEventService → backend/src/domain/analytics/services/analytics-event.service.ts
T046: LeadService           → backend/src/domain/leads/services/lead.service.ts
T047: AnalyticsService      → backend/src/domain/analytics/services/analytics.service.ts

# Frontend components (different features):
T050: LeadInquiryForm   → frontend/src/app/features/listing-search/lead-inquiry-form/
T051: DashboardComponent → frontend/src/app/features/dashboard/
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete **Phase 1**: Setup (T001–T007)
2. Complete **Phase 2**: Foundational (T008–T024) — critical blocker
3. Complete **Phase 3**: User Story 1 (T025–T035)
4. **STOP AND VALIDATE**: Run Flow A from `quickstart.md`, confirm onboarding → approval → published pipeline works end-to-end
5. Deploy or demo the MVP — trusted inventory workflow is operational

### Incremental Delivery

| Increment | Phases | Value Delivered |
|---|---|---|
| MVP | Phase 1 + 2 + 3 | Full property onboarding, multi-level approval, audit trail |
| Increment 2 | + Phase 4 | Public listing discovery, search, autocomplete, SSR |
| Increment 3 | + Phase 5 | Lead capture, analytics dashboard, role-scoped metrics |
| Release | + Phase 6 | Performance validated, security hardened, e2e coverage |

### Parallel Team Strategy

With three developers after Phase 2 completes:

```
Developer A: User Story 1 (T025–T035) — Onboarding & Approval
Developer B: User Story 2 (T036–T044) — Listing Search & Discovery
Developer C: User Story 3 (T045–T053) — Leads & Analytics Dashboard
```

Stories integrate via shared entities (Property read) but own separate modules, ensuring zero merge conflicts on feature files.

---

## Task Summary

| Phase | Tasks | Story |
|---|---|---|
| Phase 1: Setup | T001–T007 | — |
| Phase 2: Foundational | T008–T024 | — |
| Phase 3: User Story 1 | T025–T035 | US1 (P1) |
| Phase 4: User Story 2 | T036–T044 | US2 (P2) |
| Phase 5: User Story 3 | T045–T053 | US3 (P3) |
| Phase 6: Polish | T054–T059 | — |
| **Total** | **59 tasks** | |

### Parallel Opportunities

- Phase 1: T002, T003, T004, T005, T006 can all run in parallel after T001
- Phase 2: T009–T015 (entity creation) fully parallel; T017–T020 (middleware) fully parallel; T022–T023 (services) parallel
- Phase 3–5: All three story phases can proceed in parallel once Phase 2 is complete
- Within each story: [P]-marked backend service tasks and [P]-marked frontend component tasks run in parallel

### Suggested MVP Scope

**Phase 1 + Phase 2 + Phase 3 = 35 tasks** → Complete property onboarding and approval workflow as a shippable MVP.

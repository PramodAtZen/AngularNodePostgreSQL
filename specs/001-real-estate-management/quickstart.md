# Quickstart: Real Estate Management System

## Prerequisites
- Node.js 20+
- PostgreSQL 16+
- npm 10+

## 1. Configure environment

Create backend environment variables:

```bash
cp backend/.env.example backend/.env
```

Set at minimum:
- DATABASE_URL
- JWT_SECRET
- SMTP_HOST
- SMTP_PORT
- SMTP_USER
- SMTP_PASS
- APP_BASE_URL

## 2. Install dependencies

```bash
npm install
```

## 3. Run database migrations

```bash
npm run backend:migrate
```

## 4. Start backend API

```bash
npm run backend:dev
```

## 5. Start Angular app with SSR support

```bash
npm run frontend:dev:ssr
```

## 6. Validate core flows

### Flow A: Property onboarding and approvals
1. Login as Agent or Vendor.
2. Create property onboarding draft across multiple pages.
3. Submit for approval.
4. Login as Admin or Manager and approve stage 1.
5. Login as System Admin (or final approver) and complete final approval.
6. Verify property status is Published.

Expected:
- Owner cannot self-approve.
- AuditLog captures each approval action.
- Notification is sent for each status transition.

### Flow B: Public listing discovery
1. Open buyer search page.
2. Query with keyword and filter criteria.
3. Use autocomplete for location/title hints.

Expected:
- Only Published listings appear.
- Empty state appears for no-result queries.
- SSR-rendered listing pages are discoverable.

### Flow C: Lead and analytics
1. Submit inquiry as Buyer on a Published listing.
2. Open Agent or Vendor dashboard.
3. Inspect listing metrics for views and inquiries.

Expected:
- Lead record is created and linked to listing.
- Analytics API reports updated counts.
- Archived or Sold listings remain available in internal analytics history.

## 7. Run quality gates

```bash
npm run lint
npm run test:unit
npm run test:integration
npm run test:contract
npm run test:e2e
```

## 8. Run performance checks

```bash
npm run perf:search
npm run perf:autocomplete
npm run perf:analytics
```

Targets:
- Search API p95 <= 2s
- Auto Complete API p95 <= 800ms
- Analytics API p95 <= 2s
- Status-to-search visibility propagation <= 60s

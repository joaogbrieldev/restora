# TechSpec: Restora SaaS for Marmitarias

## Executive Summary

Phase 1 implements a cash-first internal management MVP on the existing Turborepo monorepo: NestJS API in `apps/api`, Next.js admin UI in `apps/web-admin`, Prisma with PostgreSQL for persistence, and shared strict TypeScript tooling from `packages/config`. The first vertical slice establishes seeded pilot tenant authentication, tenant-scoped data access, and owner/cashier RBAC before building cash sessions, manual payment totals, withdrawals, expenses, employees, attendance, and owner reports.

The primary trade-off is favoring a compact pilot architecture over full SaaS automation. The system will support real login and tenant-scoped JWTs, but restaurant provisioning, invitations, billing, customer ordering, Pix, Redis, and BullMQ remain outside Phase 1. Closed cash sessions will be immutable snapshots with owner-only adjustment records, which improves auditability at the cost of slightly more report logic.

## System Architecture

### Component Overview

- `web-admin` is the only Phase 1 frontend surface. It provides login, owner dashboard, cash session workflow, expense entry, employee and attendance management, reports, and a settings area (restaurant profile, expense categories, user access) using the existing Next.js App Router.
- The dashboard is a composed view, not a distinct data owner: it combines `GET /cash-sessions/current`, `GET /reports/daily`, and the monthly per-day breakdown from `GET /reports/monthly` to show today's status, KPIs, and incomplete-day warnings.
- `api` owns all business rules, authentication, tenant context, role checks, cash close calculations, and report aggregation using NestJS feature modules.
- PostgreSQL stores restaurants, users, cash sessions, operational entries, closing snapshots, adjustments, employees, attendance, and expense categories.
- Prisma provides the application data access layer. Tenant-scoped queries must include `restaurant_id`; PostgreSQL RLS should enforce the same boundary for tenant tables.
- Seed scripts create the pilot restaurant, initial owner and cashier users, and default marmitaria expense categories.

Data flow:

1. User logs in through `web-admin`.
2. `api` validates credentials and returns a JWT with `user_id`, `restaurant_id`, and role.
3. Admin requests include the JWT.
4. API guards extract tenant context and enforce role permissions.
5. Feature services read and write tenant-scoped rows through Prisma.
6. Owner reports aggregate closed session snapshots plus owner adjustment records.

---

## Arquitetura do Backend

O backend utiliza uma abordagem de Clean Architecture pragmática implementada através de módulos do NestJS.

Objetivos:

- isolamento das regras de negócio;
- facilidade de testes;
- isolamento de framework;
- separação de responsabilidades;
- suporte a multi-tenancy.

Referências:

- `/docs/backend-architecture.md`
- `/docs/module-structure.md`
- `/docs/multi-tenancy.md`
- `/docs/transaction-strategy.md`
- `/docs/reporting-architecture.md`
- `/docs/cash-session-domain.md`

## Implementation Design

### Core Interfaces

The core cross-boundary contract centers on a cash session closing snapshot. The implementation should use equivalent TypeScript DTOs and Prisma models.

```ts
type CashSessionCloseSnapshot = {
  sessionId: string;
  restaurantId: string;
  closedByUserId: string;
  salesTotalCents: number;
  withdrawalsTotalCents: number;
  expensesTotalCents: number;
  personnelTotalCents: number;
  estimatedProfitCents: number;
  closedAt: Date;
};
```

Primary NestJS service boundaries:

```ts
interface CashSessionsService {
  openSession(
    input: OpenCashSessionInput,
    actor: TenantActor,
  ): Promise<CashSessionDto>;
  recordPaymentTotals(
    input: PaymentTotalsInput,
    actor: TenantActor,
  ): Promise<CashSessionDto>;
  closeSession(
    sessionId: string,
    actor: TenantActor,
  ): Promise<CashSessionCloseDto>;
  createAdjustment(
    input: CashAdjustmentInput,
    actor: TenantActor,
  ): Promise<CashAdjustmentDto>;
}
```

```ts
interface TenantActor {
  userId: string;
  restaurantId: string;
  role: "OWNER" | "CASHIER";
}
```

Error handling conventions:

- Return `401` for missing or invalid JWTs.
- Return `403` for valid users without the required role.
- Return `404` for tenant-scoped resources outside the actor's restaurant.
- Return `409` for invalid state transitions, such as closing an already closed session.
- Return `422` for valid JSON with business-rule validation failures.

### Data Models

Core tables:

- `restaurants`: pilot tenant record with name, slug, status, phone, address, opening/closing time, and timestamps.
- `users`: email, password hash, name, role, `restaurant_id`, status, and timestamps.
- `cash_sessions`: one daily session per restaurant/date with status `OPEN` or `CLOSED`, opening metadata, closing snapshot totals in cents, and close metadata.
- `cash_payment_totals`: payment method totals in cents for an open cash session, using a fixed enum matching the reference UI: `CASH`, `PIX`, `DEBIT_CARD`, `CREDIT_CARD`, `MEAL_VOUCHER`.
- `cash_withdrawals`: sangrias with `amount_cents`, responsible user, occurred time, and note.
- `expense_categories`: per-restaurant, owner-editable (create/delete). Seeded default set on restaurant creation: Insumos, Infraestrutura, Manutenção, Outros. Deletion should be blocked or soft-deleted when expenses already reference the category, to keep historical reports intact.
- `expenses`: date, category, `amount_cents`, description, responsible user, and optional cash session link.
- `employees`: name, role/function label, status, `daily_cost_cents`, `restaurant_id`, and an optional nullable `user_id` FK. A populated `user_id` means the employee also has system login and a role (owner or cashier); a null `user_id` means the employee is tracked only for attendance and personnel cost, with no access to the system.
- `attendance_records`: employee, date, present flag, recorded user, and `daily_cost_cents` snapshot.
- `cash_session_adjustments`: owner-only post-close corrections with type, `amount_cents`, reason, creator, and timestamp.

Monetary values must use integer cents, as defined in [ADR-007](adrs/adr-007.md). Avoid floating point numbers and avoid mixing Prisma `Decimal` with cents in domain calculations.

Tenant tables must include `restaurant_id`. API services must filter by `restaurant_id`, and migrations should add RLS policies once the connection/session strategy is defined.

## Invariantes de Domínio

As seguintes regras são mandatórias:

- apenas uma sessão de caixa por restaurante/dia;
- caixa fechado é imutável;
- ajustes são exclusivos do owner;
- custos de funcionários são snapshotados;
- relatórios históricos utilizam snapshots;
- todas as tabelas de tenant possuem `restaurant_id`;
- acesso cross-tenant retorna `404`;
- fechamento de caixa é transacional.

### API Endpoints

Authentication:

- `POST /auth/login`: accepts email and password, returns access token and user profile.
- `GET /auth/me`: returns the current tenant actor and restaurant context.

Cash sessions:

- `POST /cash-sessions`: opens the daily session. Owner and cashier allowed.
- `GET /cash-sessions/current`: returns the open session for the actor's restaurant.
- `PATCH /cash-sessions/:id/payment-totals`: records manual sales totals by payment method. Owner and cashier allowed while open.
- `POST /cash-sessions/:id/withdrawals`: records a sangria. Owner and cashier allowed while open.
- `POST /cash-sessions/:id/close`: final-closes the session. Owner and cashier allowed; the closing user is recorded as `closed_by_user_id`.
- `POST /cash-sessions/:id/adjustments`: creates a post-close adjustment. Owner only.
- `GET /cash-sessions/:id`: returns session detail, including estimated profit, to both roles so a cashier can review before closing. Only owners can see adjustments and the wider monthly/historical `/reports/*` endpoints.

Restaurant settings:

- `GET /restaurant`: returns the actor's restaurant profile (name, phone, address, opening hours).
- `PATCH /restaurant`: updates name, phone, address, and opening hours. Owner only.

Expenses:

- `GET /expense-categories`: lists seeded and custom categories.
- `POST /expense-categories`: creates a custom category. Owner only.
- `DELETE /expense-categories/:id`: removes a category. Owner only; rejected with `409` if expenses reference it.
- `POST /expenses`: records an expense. Owner and cashier allowed.
- `GET /expenses?date=YYYY-MM-DD`: lists daily expenses. Owner full access; cashier limited to operational context.

Employees and attendance:

- `POST /employees`: creates an employee, optionally with an email to also provision system login (`user_id`) and a role. Owner only.
- `PATCH /employees/:id`: updates status, role label, daily cost, or grants/revokes system access. Owner only.
- `GET /employees`: lists active employees, including whether each has system access. Owner sees cost; cashier does not.
- `PUT /attendance/:date`: records presence for employees. Owner and cashier allowed.
- `GET /attendance/:date`: returns attendance for the day.
- `GET /attendance?from=&to=`: returns per-day attendance summary for a date range, used by the weekly presence widget. Owner and cashier allowed.

Access:

- `GET /users`: lists users with their role (owner/cashier), for the owner-facing access and permissions view. Owner only.

Reports:

- `GET /reports/daily?date=YYYY-MM-DD`: owner-only daily sales, withdrawals, expenses, personnel cost, adjustments, and estimated profit.
- `GET /reports/monthly?month=YYYY-MM`: owner-only monthly totals, days missing closure, and a per-day breakdown (date, profit, status) to drive the progress bar and daily trend chart.
- `GET /reports/monthly/export?month=YYYY-MM&format=csv`: owner-only export of the monthly report.
- `GET /reports/cash-history?from=&to=`: owner-only closed session history.

## Impact Analysis

| Component             | Impact Type        | Description and Risk                                                                 | Required Action                                                               |
| --------------------- | ------------------ | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| `apps/api/src`        | modified           | Moves from Nest starter to feature-module API. Risk is broad new backend surface.    | Add auth, tenant, cash, expenses, employees, attendance, and reports modules. |
| `apps/api/prisma`     | new                | Introduces persistence and migrations. Risk is schema churn during pilot learning.   | Add Prisma schema, migrations, seed script, and database client module.       |
| `apps/web-admin/app`  | modified           | Replaces starter page with authenticated admin workflows. Risk is UI scope growth.   | Build login, shell, cash, expense, employee, attendance, and report routes.   |
| `packages/config`     | modified if needed | Shared tooling may need Prisma/Jest path support. Risk is low.                       | Reuse existing strict configs and adjust only if tests require it.            |
| Root workspace config | modified           | Database scripts and environment examples become necessary. Risk is dev setup drift. | Add root or API scripts for Prisma, seed, test, and local env documentation.  |

## Testing Approach

### Unit Tests

- Test auth password validation, JWT claim generation, and role guard decisions.
- Test tenant context extraction and rejection of cross-tenant access.
- Test cash session state transitions: open, record data, close by owner or cashier, reject double close, and reject unauthorized close attempts.
- Test close calculation with payment totals, withdrawals, expenses, attendance-derived personnel cost, and adjustments.
- Test employee daily cost and attendance cost snapshot behavior.
- Test report aggregation for daily and monthly totals, including the per-day breakdown used by the dashboard trend chart.
- Test granting and revoking an employee's system access, including role assignment and login provisioning.
- Test expense category creation and deletion, including the rejection when a category is still referenced by expenses.

### Integration Tests

- Use API e2e tests with a test database or isolated Prisma test schema.
- Cover login, current user context, owner and cashier authorization boundaries, and tenant scoping.
- Cover a full pilot day: seed restaurant, open session, record expenses, record attendance, record sales totals, close, create owner adjustment, read daily report.
- Cover restaurant settings update and monthly report export as owner-only actions.
- Add frontend tests for key form validation and role-based navigation once the UI structure exists.

## Development Sequencing

### Build Order

1. Workspace and persistence foundation - add Prisma, PostgreSQL environment configuration, schema baseline, and seed script; no dependencies.
2. Auth and tenant context - depends on step 1 because users and restaurants must exist.
3. RBAC guards and shared API error conventions - depends on step 2 because guards need authenticated tenant actors.
4. Cash session schema and service - depends on steps 1 and 3 because sessions are tenant-scoped and permissioned.
5. Expenses, withdrawals, employees, and attendance services - depends on steps 1 and 3, and integrates with step 4 for daily close.
6. Cash close snapshot and adjustment workflow - depends on steps 4 and 5 because close calculations consume operational entries.
7. Owner reports - depends on step 6 because reports aggregate closed snapshots and adjustments.
8. Admin login and app shell - depends on step 2 because the UI needs real session handling.
9. Admin operational screens for cash, expenses, employees, and attendance - depends on steps 4 and 5, and on step 8 for layout/auth.
10. Owner close and report screens - depends on steps 6, 7, and 8.
11. End-to-end pilot hardening - depends on all previous steps and validates the daily workflow against the PRD success metrics.

### Technical Dependencies

- PostgreSQL must be available for local development and test execution.
- Prisma client generation must be integrated into API build/test scripts.
- JWT secret, database URL, and seeded pilot credentials must be documented in `.env.example`.
- RLS policy implementation depends on the chosen Prisma connection/session strategy for setting tenant context.

## Monitoring and Observability

- Log authentication failures without passwords or sensitive values.
- Log cash session state transitions with `restaurant_id`, `session_id`, `actor_id`, previous status, and next status.
- Log owner adjustments with type, amount, session, and reason metadata.
- Track daily close completion time once event timestamps exist.
- Track days opened but not closed so the admin UI can highlight incomplete days.
- Alerting is not required for Phase 1, but structured logs should make pilot support possible.

## Technical Considerations

### Key Decisions

- Decision: Use seeded local email/password JWT auth for the pilot.
  Rationale: It validates real owner/cashier access without building self-serve onboarding.
  Trade-off: Manual provisioning remains until a later phase.
  Alternatives rejected: manual dev JWTs, full onboarding, external auth provider.

- Decision: Keep closed cash sessions immutable and use owner-only adjustment records.
  Rationale: It protects historical trust while still supporting corrections.
  Trade-off: Reports must include both close snapshots and adjustments.
  Alternatives rejected: reopening sessions, no post-close changes, cashier corrections.

- Decision: Use a simple pilot operating model.
  Rationale: Daily employee cost and default expense categories reduce pilot friction.
  Trade-off: Payroll and expense analysis remain intentionally approximate.
  Alternatives rejected: flexible payroll types, no default categories.

- Decision: Allow cashiers to final-close a cash session, not just owners.
  Rationale: The owner is not always present at closing time; requiring owner confirmation would block the daily close the pilot depends on. Post-close adjustments and monthly/historical reports stay owner-only, so sensitive corrections and cross-day analysis remain protected.
  Trade-off: A cashier can now see the current session's estimated profit (needed to review before closing), widening cashier visibility slightly beyond pure operational entry.
  Alternatives rejected: owner-only close (original ADR-004 decision, superseded by ADR-005), cashier prepares/owner confirms asynchronously.

- Decision: Model employees as a superset of users, linked by an optional `user_id`.
  Rationale: The reference UI tracks cost and attendance for staff (cooks, delivery) who never log in, alongside staff who do; a single `employees` table with an optional login link avoids duplicating person records.
  Trade-off: Employee CRUD and user provisioning become coupled in the API layer.
  Alternatives rejected: separate unlinked tables for staff and users, requiring a login for every tracked employee.

### Known Risks

- RLS with Prisma can be easy to misconfigure if tenant context is only enforced in application code. Mitigation: use both API-level `restaurant_id` filters and database policies once the connection strategy is stable.
- The greenfield state means Phase 1 includes infrastructure, auth, schema, API, and UI work. Mitigation: build in vertical slices and keep Phase 2 systems out of the code path.
- A cashier could close a session with wrong totals since owner review is no longer required. Mitigation: keep the confirmation prompt before close, and rely on owner-only post-close adjustments (ADR-003) as the correction path.
- Daily employee cost may not reflect all payroll arrangements. Mitigation: snapshot daily cost during attendance and revisit cost types after pilot validation.
- Reports can become inconsistent if adjustment semantics are vague. Mitigation: store explicit adjustment type, signed amount, reason, and creator, and centralize report calculations in one service.
- Deleting an expense category that historical expenses still reference can corrupt past reports. Mitigation: reject deletion (`409`) while references exist, or soft-delete and keep the category resolvable for historical rows.

## Architecture Decision Records

- [ADR-001: Priorizar MVP caixa-first](adrs/adr-001.md) — Defines daily cash closing as the Phase 1 validation axis.
- [ADR-002: Seeded JWT Authentication for the Pilot](adrs/adr-002.md) — Uses seeded restaurant/users and local JWT auth with tenant and role claims instead of self-serve onboarding.
- [ADR-003: Immutable Cash Sessions with Owner Adjustments](adrs/adr-003.md) — Preserves closed cash sessions as historical snapshots and models corrections as owner-only adjustments.
- [ADR-004: Simple Pilot Operating Model](adrs/adr-004.md) — Uses daily employee costs and seeded marmitaria expense categories for Phase 1; its owner-only final-close decision is superseded by ADR-005.
- [ADR-005: Cashier-Enabled Cash Session Close](adrs/adr-005.md) — Allows cashiers to final-close a cash session; post-close adjustments and monthly/historical reports remain owner-only.
- [ADR-006: Clean Architecture Pragmática](adrs/adr-006.md)
- [ADR-007: Representação Monetária](adrs/adr-007.md)

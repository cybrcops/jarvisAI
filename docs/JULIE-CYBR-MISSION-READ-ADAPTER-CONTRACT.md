# Julie ↔ CYBR MISSION read-only adapter contract

**Audit date:** 2026-11-14  
**Target source:** `cybrcops/cybrcops-platform`, `main`  
**Observed target commit:** `dc45caf5e1946bead8e9ea9814a526044ee1a43b`  
**Reviewed source paths:** `apps/web/src/routes/api.mission.ts`, `api.crm.ts`, `api.health.ts`, `apps/web/src/lib/api.server.ts`, `authorization.server.ts`, `mission-resources.ts`, `supabase.ts`, `mission-identity.ts`, `require-auth.ts`, `routes.ts`, `routes/app-layout.tsx`.  
**Status:** contract/design only; no application code or configuration changed.

## Executive decision

CYBR MISSION already exposes a narrow authenticated read route suitable as the first integration seam: `POST /api/mission`. It accepts a caller Bearer JWT, resolves that exact user through Supabase Auth, requires AAL2, verifies one active organization membership and role/capability grants, calls `has_permission`, and queries through the caller-scoped Supabase client (no service-role key). The route uses a closed resource allowlist and explicit field projections.

**Julie must not call this route using an owner/service-role key, shared static token, browser-stored integration secret, or a synthetic user identity.** The inspected API is user-bound. A server-side Julie adapter needs a formally approved delegated owner identity/session model, secure token custody/refresh/revocation, and purpose-specific authorization before implementation. Until then, the route is a verified API contract, not an available unattended connector.

## Verified route contract

### Endpoint

- Method/path: `POST /api/mission`
- Request JSON (max 8 KiB): `{ organizationId: UUID, resource: MissionResource, mode?: "count" }`
- Authentication: `Authorization: Bearer <Supabase access token>`
- Reads: POST only; loader returns 405.
- Successful list response: `{ records: [...], limit: 100 }`
- Successful metric response: `{ count: number }`
- Per-user response headers: `Cache-Control: private, no-store`, `Vary: Authorization`; API wrapper adds security headers and rate limiting.
- Unknown resources and unsupported modes are rejected. List results are capped at 100. No generic table name or caller-selected field list is accepted.

### Resource allowlist observed in `mission-resources.ts`

| Resource | Permission | Table | Projected fields |
|---|---|---|---|
| clients | `clients.view` | clients | id, name, status, industry, website |
| leads | `crm.view` | leads | id, company_name, title, email, phone, source, status, estimated_value, updated_at |
| tickets | `tickets.view` | tickets | id, ticket_number, title, status, priority, updated_at |
| alerts | `soc.alerts.view` | alerts | id, title, severity, status, detected_at |
| incidents | `incidents.view` | incidents | id, title, severity, status, updated_at |
| policies | `grc.view` | policies | id, policy_code, title, status, review_date |
| contracts | `finance.view` | contracts | id, contract_number, title, status, start_date, end_date |
| invoices | `finance.view` | invoices | id, invoice_number, status, due_date, total_amount, currency |

Count mode is implemented only for clients, tickets, alerts, and incidents. It applies active/non-closed filters in the route. Do not assume all resources support counts.

**Data minimization:** The route's lead projection includes contact fields and invoice projection includes financial amounts. Julie should request these resources only when the owner-authorized briefing purpose requires them. The default owner summary should prefer counts/status and omit customer contact and invoice amounts unless separately authorized.

### Authorization sequence observed in `authorizeTenant`

1. Reject unknown permission capability and malformed organization UUID.
2. Require Bearer token (max header length 8 KiB).
3. Read Supabase URL and publishable/legacy anon key from server environment; reject secret/service-role key formats.
4. Create a Supabase client carrying the exact caller JWT.
5. Call Supabase Auth `getUser(token)`; reject invalid or anonymous identity.
6. Parse claims only after provider validation; bind `sub` to returned user ID and require `aal=aal2`.
7. Apply caller budget (100 requests / 300 seconds per user).
8. Query exactly one active/time-valid membership for the requested organization.
9. Resolve active non-customer role; verify role grant and active permission row.
10. Call database `has_permission` for the same organization/permission.
11. Return the caller-scoped database client and verified user/organization/role.

This route's identity binding and permission checks are positive design evidence. The live catalog review still found sensitive-column SELECT grants, so the route's field projection is a critical safety boundary; Julie must not bypass it by using direct table access.

## Related API routes and exclusions

- `POST /api/crm` is a write route requiring `crm.manage`; it creates/moves leads. **Do not expose it to Julie's read adapter.**
- `GET /api/health` returns only `{ ok: true }`; it is a liveness signal, not database/dependency health or tenant-aware service health.
- Catch-all `/api/*` returns 404.
- Authenticated routes are POST because the hosting edge may cache GET by URL across visitors; keep this invariant for future read endpoints.
- App route `/app` uses browser client auth and AAL2; role allowlist is admin/staff/analyst/technician.
- Browser Supabase client persists the user session. `mission-identity.ts` stores selected organization ID in localStorage keyed by user. This is a UI workspace selector only; server authorization revalidates membership and permissions and must not trust localStorage as authority.

## Julie adapter architecture (proposed; not implemented)

### Trust boundary

Julie backend → CYBR MISSION adapter → approved owner/delegated Supabase JWT → CYBR MISSION `POST /api/mission`.

- Adapter is server-side only. Never expose JWTs, Supabase keys, or connector tokens to the browser, prompts, logs, traces, or GitHub.
- Bind every connector credential to an explicit owner identity, allowed organization IDs, allowed resources, allowed fields, purpose, expiration, and revocation status.
- Do not accept an arbitrary `organizationId` from model output. The adapter must resolve organization scope from its own owner-approved grant registry and then enforce the same value in the request.
- No service-role key, no direct database table access, no RPC access to privileged functions, no write route.
- If a user-bound delegated token cannot be issued and refreshed safely, fail closed and show connector unavailable; do not silently switch to service-role credentials or another tenant.
- Require explicit customer scope and explicit provider delegation before any customer-specific records are read. An organization membership alone does not imply permission to view every downstream customer tenant.
- Separate CYBR MISSION company scope from CYBRCOPS company scope and owner-private memory. No automatic cross-domain joins.
- Normalize records into a minimal schema with source system, organization/customer scope, fetched-at timestamp, source update timestamp, field provenance, and access grant ID. Distinguish source facts from Julie assessments/recommendations.
- Cache only minimized results, scoped by identity + organization + resource + permission version; use short TTL and purge on revocation. Never globally cache customer data.
- Log metadata-only audit events (actor, grant, organization, resource, request ID, timestamp, result class); never log bearer tokens, raw sensitive columns, full prompts, or unredacted response bodies.

### Read-only adapter interface (proposed)

```ts
type MissionResource =
  | "clients" | "leads" | "tickets" | "alerts"
  | "incidents" | "policies" | "contracts" | "invoices";

type MissionReadRequest = {
  grantId: string;              // looked up server-side; never model-created authority
  resource: MissionResource;    // closed enum
  mode?: "list" | "count";
  purpose: "owner_briefing" | "incident_triage" | "delivery_review";
  customerScopeId?: string;     // accepted only when explicit delegation exists
};

type Provenanced<T> = {
  source: "cybrcops-platform";
  organizationScope: string;
  fetchedAt: string;
  sourceUpdatedAt?: string;
  grantId: string;
  data: T;
};
```

The above is a design sketch, not production code or a finalized schema. Never return the grant's token or internal secret reference to Julie's model context.

## Required blockers before implementation

1. Define owner/delegated identity: whether Julie operates with an owner-approved delegated user session, a dedicated non-human principal supported by CYBR MISSION, or a new narrowly scoped server-to-server endpoint. This is a security/identity decision, not a coding shortcut.
2. Implement secure token custody, refresh, expiry, revocation, step-up, rotation, and incident response. Do not use long-lived browser tokens in a background worker.
3. Resolve sensitive column privileges for API key hashes, integration config/secret references, webhook headers, and SOC source keys. Preserve existing caller RLS; add safe projections/grants and test them with authenticated synthetic users.
4. Verify actual customer data model and explicit MSP delegation before adding customer-scoped reads.
5. Build synthetic cross-org and cross-customer tests with ordinary JWTs. No production row data needed.
6. Add audit correlation and source freshness metadata; prove revocation takes effect immediately.
7. Keep writes excluded. Any future write capability requires a separate action-specific permission and approval design.

## Validation performed

- Static GitHub inspection of exact source paths and blob SHAs listed above.
- Compared route behavior with the live Supabase catalog/advisor audit recorded in `CYBR-MISSION-LIVE-DATABASE-AUDIT.md`.
- No route invocation, customer-row query, build, lint, typecheck, test, security scan, token issuance, or runtime integration test.
- No application code, schema, grants, policies, settings, dependencies, or deployment configuration changed.
- No NUC, Hostinger, Cloudflare, DNS, public site, or production application changed.

## Next engineering step

Implement only a standalone, pure adapter contract/type-and-policy module plus synthetic unit tests in a dedicated Julie integration branch after the owner identity/token model is decided. Do not connect a live token or call production from tests.

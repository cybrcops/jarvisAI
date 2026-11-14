# CYBR MISSION platform — initial source audit

**Date:** 2026-11-14  
**Repository:** `cybrcops/cybrcops-platform`  
**Branch reviewed:** `main`  
**Review type:** Remote static source inspection through GitHub; not a runtime or security certification.

## Confirmed repository facts

- Repository is private; default branch is `main`; connected GitHub identity has admin/push access.
- README identifies CYBR MISSION as the primary platform and corporate brand for a unified CRM, ITSM, SOC, cybersecurity, and business-operations ecosystem.
- README lists Mission Control, Mission CRM, Mission Desk, Mission Ops, Mission SOC, Mission Secure, Mission GRC, Mission Finance, Mission AI, and Mission Connect. These are product-map claims, not proof each module is implemented or production-ready.
- Root `package.json` defines an npm workspace over `apps/*`, with root wrappers for dev, build, start, lint, typecheck, and tests. It overrides `brace-expansion` to versions `1.1.21` and `5.0.12`.
- `apps/web/package.json` identifies a private React 19 / React Router 8 / Vite 7 / TypeScript application and includes `@supabase/supabase-js` as a dependency. Presence of the SDK does not establish that authentication, authorization, database access, or tenant isolation is correctly implemented.
- `docs/development.md` documents Node 22, npm workspace setup, loopback-oriented root dev/start wrappers, and the recommended local checks: lint, typecheck, tests, build. It explicitly says dependency installation requires approval and warns that configured services may not be offline.
- `tools/check.mjs` loads Vite public configuration and invokes the selected lint/typecheck/build scripts. It is not evidence that those checks have been executed successfully.

## Security and architecture findings

- **Authentication and authorization (partially verified):** Source review now confirms an AAL2-gated client identity flow and a server-side `authorizeTenant` gate that verifies the caller's Supabase JWT, active membership, active role, explicit role permission, active permission record, and `has_permission` RPC. The route constructs a caller-scoped client using a publishable/anon key and rejects service-role keys. This is a positive source-level design observation, not a complete authentication or deployment certification.
- **Tenant isolation (partially verified):** The inspected `/api/mission` and `/api/crm` handlers pass the requested organization through `authorizeTenant`, use the authenticated caller's JWT, filter records by `organization_id`, and rely on database RLS. The actual migration SQL and deployed RLS policies were not located/independently inspected in this pass, so cross-tenant isolation is not certified.
- **Database/schema:** `mission-resources.ts` defines a closed resource/column allowlist for clients, leads, tickets, alerts, incidents, policies, contracts, and invoices. Migration history in commit `2e6968585be1909c8185f77000952e5a0dfab425` documents security migrations 020–023 and includes the 023 `SECURITY INVOKER` wrapper for `public.has_permission`; however, the migration files themselves were not located in the inspected paths and the live database was not queried. RLS policy correctness and deployed state remain unverified.
- **API/service boundaries (partial map):** `/api/mission` is POST-only, uses a closed resource list and explicit selected columns, filters by organization, returns `private, no-store`, and caps lists at 100 rows. `/api/crm` creates/moves leads after `crm.manage` authorization and server-side input validation. `withApi` adds response security headers, request rate limiting, and bounded body parsing. A complete route inventory, webhook review, and all server-only/privileged call paths remain open.
- **Secrets/configuration:** `authorization.server.ts` explicitly accepts only a publishable key or legacy anon key for its caller-scoped client and rejects secret/service-role keys. The browser client uses `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`. No secret values were retrieved. Hosting/proxy configuration and runtime environment were not inspected.
- **Supply chain:** Root and web manifests were inspected, but lockfile integrity, dependency advisories, install scripts, CI workflows, and transitive dependency risk were not audited in this pass.

## Julie integration implications

Treat `cybrcops-platform` as the primary CYBR MISSION integration target, but build no connector until the platform's actual identity, authorization, tenant, and API contracts are verified. Julie's cross-business dashboard should consume purpose-limited, read-oriented data through explicit server-side adapters. Writes must be action-scoped and approval-gated. Customer data must remain partitioned by tenant; CYBR MISSION and CYBRCOPS company scopes must remain distinct; owner-private EA memory must never enter either company or customer scope.

## Validation and operational impact

- Static GitHub source inspection only.
- No repository clone, dependency installation, build, lint, typecheck, tests, security scan, live API request, database access, or runtime execution was performed.
- No NUC, Hostinger, Cloudflare, DNS, live website, production service, customer data, or secret was touched.
- No application code, schema, dependency, or deployment configuration was changed.

## Next audit sequence

1. Identify the actual app route tree, server/client Supabase client construction, auth/session lifecycle, and all privileged server-only code paths.
2. Locate migrations/schema definitions and inspect organization, membership, role, permission, and customer-tenant relationships.
3. Inspect row-level security policies and verify every exposed table/view/function has appropriate tenant predicates and privilege boundaries.
4. Map API routes, webhooks, integrations, background jobs, and service-role access; verify authentication, authorization, replay protection, rate limits, and audit coverage.
5. Inventory module maturity from actual routes, schemas, and tests rather than README claims.
6. Only after source review, prepare a least-privilege Julie read adapter and test synthetic cross-tenant denial cases.

**Sources inspected:** `README.md`, `package.json`, `apps/web/package.json`, `docs/development.md`, `tools/check.mjs` on `cybrcops/cybrcops-platform/main`.


## Source inspection update — 2026-11-14

This section supersedes the earlier “not yet verified” statements above where it records partial source evidence. It does not certify production configuration or the database.

### Identity and tenant authorization

Inspected:
- `apps/web/src/hooks/use-supabase-auth.ts`
- `apps/web/src/lib/mission-identity.ts`
- `apps/web/src/lib/authorization.server.ts`
- `apps/web/src/lib/security/policy.ts`
- `apps/web/src/lib/security/capabilities.ts`

Observed:
- The client hook checks Supabase MFA assurance and only resolves a Mission user when the current session is `aal2`.
- Workspace selection is remembered in browser `localStorage`, but the server does not trust that value as authority; each data request supplies an organization ID and is independently authorized.
- `authorizeTenant` verifies the exact Bearer token with `db.auth.getUser(token)`, checks the token subject against the returned user and requires `aal2`.
- It checks a single active membership for the target organization, valid membership dates, an active allowed role, an enabled `role_permissions` row, an active `permissions` row, and the `has_permission` RPC.
- It creates a Supabase client using a publishable/anon key with the caller JWT attached. It rejects a service-role/secret key. The database client therefore operates as the caller rather than a privileged service identity.
- The capability registry is a closed map of exact permission names to operations; unknown permission names are rejected.

### Data routes and current module maturity

Inspected:
- `apps/web/src/lib/mission-resources.ts`
- `apps/web/src/routes/api.mission.ts`
- `apps/web/src/routes/api.crm.ts`
- `apps/web/src/lib/api.server.ts`
- `apps/web/src/lib/rate-limit.server.ts`
- `apps/web/src/lib/mission-client.ts`
- `apps/web/src/lib/portal-product/crm.ts`

Observed:
- The mission read API supports eight named resources: clients, leads, tickets, alerts, incidents, policies, contracts, and invoices. Each resource has a fixed permission, table, selected field list, and ordering; arbitrary table/column names are not accepted from the request.
- `/api/mission` accepts POST only, authorizes the caller and organization, filters by `organization_id`, selects only allowlisted fields, returns `private, no-store`, and caps list results at 100. Its count mode is restricted to clients, tickets, alerts, and incidents.
- `/api/crm` accepts POST only and requires `crm.manage`. Lead creation uses a server-side allowlist validator that rejects unknown fields and derives organization/owner from the authorized context. Stage changes validate UUID and allowed stage and filter by both lead ID and organization.
- CRM writes are normal authenticated user writes under the platform permission/RLS model; the inspected route does not use a Julie-specific action approval token. Julie must not call this write route autonomously until a separate CYBR COMMAND policy/approval adapter is designed.
- Commit history shows `dc97769587163c6039076a9f4dbbfd8f9d8bad3c` added the live read-only tickets screen and `dc45caf5e1946bead8e9ea9814a526044ee1a43b` released real CRM and Tickets screens (2026-11-04). The product map still exceeds verified UI maturity; do not assume every listed module is fully live.

### API controls and open concerns

- `api.server.ts` applies response security headers, body-size limits, and a per-IP rate limit through `withApi`; it explicitly documents that authenticated API reads must use POST because the hosting edge may rewrite cache headers on GET.
- `rate-limit.server.ts` uses in-memory per-process counters. The code trusts the left-most `X-Forwarded-For` entry based on an assumption that the fronting proxy rewrites it. Verify that assumption in the actual hosting topology and use a shared/distributed limiter before multi-instance or public production use.
- The inspected API response logger records method, status, and duration, not an explicit actor/tenant/resource audit record. Database audit triggers or other logging may exist, but were not verified.
- The commit record for `2e6968585be1909c8185f77000952e5a0dfab425` reports that migrations 020–023 were applied and includes the 023 wrapper that restored `has_permission` after migration 020 moved the helper into `private`. The commit notes report throwaway-Postgres checks. This is repository evidence only; no production database query or independent migration replay was performed in this audit.
- A direct SQL/RLS audit is still required. The exact migration file paths were not found during this pass, so no claim is made that every table, view, function, or policy has been reviewed.
- Rate limiting, audit completeness, tenant RLS, and host proxy assumptions need synthetic cross-tenant and unauthorized-role tests before Julie integration.

### Freshness and integration decision

- Latest platform commit observed on `main`: `dc45caf5e1946bead8e9ea9814a526044ee1a43b`, 2026-11-04.
- The platform remains the primary CYBR MISSION source of truth. No CYBR COM/CYBRCOPS cross-business connector was found in the inspected platform files.
- Next step is direct migration/RLS source discovery and route inventory, followed by synthetic tenant-isolation tests. Only then should a read-only Julie adapter be implemented.

**Additional sources inspected:** `apps/web/src/hooks/use-supabase-auth.ts`, `apps/web/src/lib/mission-identity.ts`, `apps/web/src/lib/authorization.server.ts`, `apps/web/src/lib/security/policy.ts`, `apps/web/src/lib/security/capabilities.ts`, `apps/web/src/lib/mission-resources.ts`, `apps/web/src/routes/api.mission.ts`, `apps/web/src/routes/api.crm.ts`, `apps/web/src/lib/api.server.ts`, `apps/web/src/lib/rate-limit.server.ts`, `apps/web/src/lib/mission-client.ts`, `apps/web/src/lib/portal-product/crm.ts`.

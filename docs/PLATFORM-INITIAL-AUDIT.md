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

- **Authentication and authorization:** Not yet verified from implementation source. Do not infer security from dependency presence or product documentation.
- **Tenant isolation:** Not yet verified. No conclusion can yet be made about organization/customer scoping, row-level security, tenant-aware server loaders/actions, or cross-tenant access controls.
- **Database/schema:** No migration or schema source was confirmed in this focused pass. A conventional `supabase/config.toml` path was not found; this does not prove there is no schema elsewhere.
- **API/service boundaries:** Not yet mapped. Route loaders/actions, server-only clients, privileged service-role usage, webhook authentication, and external integration boundaries remain open review items.
- **Secrets/configuration:** The development guide warns that wrappers load local environment configuration and validate public configuration. Do not retrieve or copy secret values. Review only sanitized configuration names and code paths when authorized.
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

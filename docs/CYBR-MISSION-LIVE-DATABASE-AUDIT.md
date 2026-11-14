# CYBR MISSION live database audit

**Audit date:** 2026-11-14  
**Source repository:** [cybrcops/cybrcops-platform](https://github.com/cybrcops/cybrcops-platform), `main`  
**Observed source head:** `dc45caf5e1946bead8e9ea9814a526044ee1a43b`  
**Database:** Supabase project `cybrmission-portal` (connected project; project ref intentionally not duplicated here)  
**Method:** read-only metadata, migration history, advisors, and PostgreSQL catalog SELECT queries.

## Executive summary

The connected `cybrmission-portal` project is `ACTIVE_HEALTHY`, PostgreSQL 17, `us-east-2`. Its migration history contains 28 migrations through `public_permission_check_023`. The live catalog reports 91 public ordinary tables, all 91 with RLS enabled, 375 public policies, no public ordinary table without at least one policy, six private SECURITY DEFINER functions, and two public SECURITY DEFINER functions.

This is structural evidence, not a behavioral tenant-isolation certification. No customer/business rows were queried and no mutation was performed.

## Migration source and live history

The SQL history is organized under `supabase/live/`, not `supabase/migrations/`. Inspected:
- `supabase/live/README.md`
- `supabase/live/019_ticket_tenant_integrity.sql`
- `supabase/live/020_core_security_hardening.sql`
- `supabase/live/021_enforce_mfa.sql`
- `supabase/live/022_ticket_comment_update_visibility.sql`
- `supabase/live/023_public_permission_check.sql`
- `supabase/design/security-foundation.md`

The live migration history returned these 28 entries, ending with:
`ticket_tenant_integrity_019`, `core_security_hardening_020`, `enforce_mfa_021`, `ticket_comment_update_visibility_022`, `public_permission_check_023`.

The live README documents 019–023 as applied and records previous attack-suite results. This audit verified the migration history/catalog state, not the historical execution transcript or the correctness of every behavioral test.

## Structural catalog results

| Measure | Live result |
|---|---:|
| Public ordinary tables | 91 |
| RLS-enabled public tables | 91 |
| RLS-disabled public tables | 0 |
| Public policies | 375 |
| Tables without RLS or any policy | 0 |
| Private SECURITY DEFINER functions | 6 |
| Public SECURITY DEFINER functions | 2 |

Selected policy review covered organizations, memberships, clients, tickets, ticket_comments, incidents, alerts, api_keys, integrations, automation_workflows, AI conversations/messages, audit_logs, and access_audit_events. Observed policy patterns include restrictive MFA gates, organization/permission predicates, self-identity checks for membership operations, and owner-scoped AI conversation access. AI message access derives through the parent conversation's RLS visibility.

## Privileged functions

### `public.rls_auto_enable()`

- SECURITY DEFINER; owner postgres.
- `search_path=pg_catalog`.
- Execute ACL: postgres and service_role.
- Event trigger attempts to enable RLS for newly created public tables.
- **Review item:** exception handler catches all failures and emits a log instead of rethrowing. Confirm this event trigger cannot silently permit an RLS-disabled table after an enablement failure; test in isolation and consider fail-closed behavior. No function change made.

### `public.soc_ingest_feed_batch(text,jsonb)`

- SECURITY DEFINER; owner postgres.
- Empty search_path.
- Execute ACL: postgres and service_role only.
- Validates JSON array and caps batch at 500; resolves an enabled source key; writes SOC ingest events; opens/repeats/closes incidents and records heartbeat state.
- **Boundary:** keep service-role-only and expose only through a narrowly authenticated ingestion adapter. It is a privileged consequential write path and must not become a generic Julie tool. Validate source provisioning, rotation, payload schema, replay behavior, rate limits, and audit provenance before connecting.

### Private SECURITY DEFINER functions

Six functions are in `private`: `can_access_organization`, `can_grant_role`, `handle_new_user`, `has_permission`, `is_organization_member`, and `mfa_required`.

The observed private helpers generally use empty search_path and limited ACLs. `private.handle_new_user()` uses `search_path=public`; its execute ACL is restricted to postgres, service_role, and supabase_auth_admin. Review this exception for object-shadowing risk and harden only after confirming trigger compatibility.

## Supabase advisor results

### Security
One warning: leaked-password protection is disabled. Review and enable only through an approved settings change:
https://supabase.com/docs/guides/auth/password-security#password-strength-and-leaked-password-protection

### Health
No health lints returned.

### Performance
- Unused-index warnings across many tables. Do not drop indexes based solely on an advisor snapshot; validate observation window, query volume, constraints, and planned workloads.
- Four multiple-permissive-policy warnings:
  - `dashboard_layouts`: `dashboard_layouts_select` and `dashboard_layouts_write`
  - `dashboard_widgets`: `dashboard_widgets_select` and `dashboard_widgets_write`
  - `incident_tasks`: `incident_tasks_select` and `incident_tasks_write`
  - `invites`: `invites_select` and `invites_write`
These are performance warnings, not proof of an authorization bypass. Compare predicates and test behavior before consolidation.

## Julie integration implications

1. Begin with a read-only platform adapter using a user-bound JWT session, not a service-role key.
2. Use explicit organization and customer scope; never infer a customer scope from an MSP relationship alone.
3. Keep resource/field allowlists in the adapter; do not expose API-key secret-bearing columns, internal incident evidence, private AI messages, or raw integration credentials.
4. Preserve source, organization, timestamp/freshness, and permission context in every normalized observation.
5. Fail closed on missing identity, missing AAL2 when required, unknown tenant, expired membership, stale permissions, unknown fields, or unavailable authorization.
6. Keep SOC feed ingestion, automations, incident state changes, and all write operations out of Julie's initial read adapter.

## Required next tests

Run only in an isolated environment with synthetic identities and tenants:
- same-org allowed read and cross-org denied read;
- two customer accounts in one MSP tenant, including cross-customer denial;
- direct membership vs explicit provider delegation, including expired/revoked delegation;
- expired membership, disabled role/permission, and AAL1/AAL2 enforcement;
- parent-child organization mismatch and ticket-comment visibility;
- AI conversation/message ownership and cross-user denial;
- API-key metadata projection without secret disclosure;
- service-role-only SOC ingestion, invalid source, replay, malformed payload, and incident transition behavior;
- RLS event-trigger failure behavior and new-table fail-closed verification.

Do not use owner or service-role queries as proof of RLS. Obtain a reproducible sanitized schema and approved isolated test environment first. Do not create a billable Supabase branch or alter production without explicit approval.

## Validation and impact record

- Performed: GitHub source reads; Supabase project metadata, migration history, table metadata, advisor retrieval; SELECT-only PostgreSQL catalog queries.
- Not performed: data-row reads, DDL/DML, migration apply, settings change, build, tests, scan, penetration test, external scan, NUC access, iPhone test.
- No NUC, Hostinger, Cloudflare, DNS, public website, production application configuration, customer data, credentials, or secrets were changed or retrieved.
- This document is an audit record, not authorization to change production.

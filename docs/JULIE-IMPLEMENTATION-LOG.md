# Julie implementation log

## 2026-11-14 — Repository discovery and first-pass capability audit

### Completed
- Enumerated 50 repositories visible to the connected GitHub account under `cybrcops`, including each reported default branch.
- Confirmed the current Julie foundation is `cybrcops/jarvisAI`; its README still documents a Claude Agent SDK bridge, so provider-neutral FreeLLMAPI support remains unimplemented.
- Reviewed first-pass README and LICENSE content for `jarvisAI`, `VoiceStudio`, `ruflo`, `freellmapi`, `Cybrcom`, `AI`, `twenty`, and `CRM`.
- Reviewed README architecture/usage descriptions for `cybrcops-platform`, `zero-breach-toolkit`, `OpenJarvis`, `JARVIS`, `claude-skills`, `security-audit-skill`, and `pentagi`.
- Added `docs/REPOSITORY-CAPABILITY-INVENTORY.md` on `feat/julie-repository-audit`.
- Confirmed recent `jarvisAI/main` commit evidence: `2e1c908a`, dated 2026-11-13.
- Created audit branch from `feat/julie-personal-assistant-architecture`; no main-branch commits.

### Important findings
- VoiceStudio app is AGPL-3.0; individual model/weight licensing remains separate and commercial clearance is not established.
- Ruflo is MIT-licensed, but broad tool execution still requires permission and sandbox review.
- FreeLLMAPI is MIT-licensed, but its README advertises an optional paid premium catalog; Julie must not purchase or silently route to paid services.
- Cybrcom README describes its current frontend as a synthetic demo and labels the project proprietary/private; it is also the live CYBR COM website repository on a separate `site` branch. No deployment or site changes were made.
- Twenty is primarily AGPLv3 with marked commercial files and selected MIT packages; CRM is a separate Mission CRM domain repo, not a Twenty fork.
- `security-audit-skill` requires an OS-enforced sandbox for executing target-controlled code. PentAGI and security tooling must remain isolated from the general assistant.

### Not done / not claimed
- No repository was cloned or installed.
- No NUC, Hostinger, Cloudflare, DNS, live website, or production system was changed.
- No tests, builds, security scans, model benchmarks, or iPhone device checks were executed.
- No secrets were retrieved or committed.
- No PR has been opened yet.
- This is a first-pass focused audit, not a full audit of all 50 repositories. Many repositories still need commit freshness, actual license, dependency, CI, API, and security review.

### Next
1. Continue repository-by-repository metadata and code review, prioritizing FreeLLMAPI, Ruflo, VoiceStudio, and jarvisAI.
2. Inspect current Julie branch code and test setup before implementation.
3. Define provider/voice adapters and policy contracts before wiring integrations.
4. Build the PWA shell on a feature branch; verify only checks that can actually run in the remote environment.
5. Create a draft PR for human review; never merge or deploy.


## 2026-11-14 — Focused integration-boundary inspection

### Additional source inspected
- `jarvisAI/package.json` and `bridge/server.mjs`: confirmed Node/build/lint/test scripts, Claude Agent SDK coupling, loopback bind default, Origin/Host checks, missing-Origin rejection by default, and a broad MCP/tool surface. A global write-enable flag is not sufficient for action-specific approvals.
- `freellmapi/package.json` and `docs/en/api/01-rest-api.md`: confirmed workspace structure, OpenAI-compatible chat/streaming/tool-call APIs, active and named fallback profiles, and model/usage endpoints. `auto` depends on configured provider pools; never assume it is free/local-only.
- `ruflo/package.json`: confirmed v3.77.0 MIT package, Node >=20, MCP/Codex/security workspaces and a broad optional dependency footprint.
- `VoiceStudio/docs/api-auth.md` and `docs/mcp.md`: confirmed loopback unauthenticated default, remote API key behavior, PIN not guarding WebSockets, separation of consumption vs admin routes, and MCP base-path confinement.

### Updated artifacts
- Expanded `docs/REPOSITORY-CAPABILITY-INVENTORY.md` with code/configuration observations and remaining verification items.
- Existing draft PR #1 automatically includes the additional inventory commit.

### Decisions reinforced
- Julie must use a server-side provider adapter and an explicit provider/model allowlist; no implicit `auto` chain or paid fallback.
- Julie must use an action-specific approval inbox; do not rely on the JARVIS global writes flag.
- VoiceStudio integration must be brokered; expose only approved transcription/synthesis operations and never administrative endpoints or master credentials.
- Ruflo remains disabled as an execution worker until its MCP/tool permissions and OS-level sandbox are verified.

### Validation status
Static GitHub source inspection only. No package installation, build, lint, tests, live API calls, security scan, NUC interaction, or device test was performed.


## 2026-11-14 — CRM scope decision

- User explicitly directed that Twenty be excluded because CYBR MISSION is no longer using it.
- Updated the repository capability inventory: Twenty is marked excluded, its prior candidate integration/reuse recommendation is removed, and the dedicated `CRM` repository is the sole CRM codebase under consideration.
- No Twenty code, APIs, dependencies, or services will be integrated into Julie. Do not spend further audit effort on Twenty unless the user explicitly reverses this decision.
- No CRM implementation or runtime changes were made as part of this scope update.


## Ongoing documentation requirement (standing project rule)

Every Julie-related work session and repository change must be documented in GitHub. This applies to research, architecture decisions, implementation, configuration changes, integrations, tests, security reviews, blockers, and deferred work.

### Required record for each work item
- Date and scope: what was requested and which repository/branch/files were involved.
- Decision and rationale: what was selected, rejected, or deferred, including user-directed exclusions and constraints.
- Evidence: exact source paths, commit references, documentation, and relevant upstream/project claims; distinguish observed facts from assumptions.
- Changes: files added/modified/deleted and the resulting commit SHA or PR link.
- Validation: exact commands/checks performed and their actual results. If not run, say so explicitly; never imply success.
- Security/licensing: relevant trust boundaries, permissions, dependency/supply-chain concerns, license/provenance, and unresolved rights.
- Operational impact: state whether NUC, hosting, DNS, production, customer data, credentials, or external services were touched. Record "not touched" when applicable.
- Follow-up: open questions, blockers, risks, and the next concrete step.

### Repository hygiene rules
- Keep documentation in the same feature branch/PR as the work it describes whenever practical.
- Update the relevant architecture/inventory document when a decision changes; update this log for the chronological record.
- Link related commits, PRs, issues, and source files. Keep claims traceable to evidence.
- Never commit credentials, tokens, private keys, personal data, raw audio, or other secrets. Redact sensitive values and document only that a secret/configuration exists when necessary.
- Do not claim a build, test, scan, install, deployment, or runtime action happened without captured evidence.
- Record user approvals and boundaries; do not treat documentation as authorization to perform an action.
- Keep draft PRs reviewable. Do not merge or deploy without explicit user approval.

This rule is persistent for all future Julie work, not limited to the current audit PR.


## 2026-11-14 — Corrected primary platform and CRM/SOC source of truth

### User clarification
- User clarified that `cybrcops/cybrcops-platform` is the main CYBR MISSION platform for CRM/SOC and the broader product ecosystem. The separate `cybrcops/CRM` repository is not the main platform.

### Evidence reviewed
- Read `cybrcops-platform/README.md` on `main`. It describes CYBR MISSION as the primary platform and corporate brand for unified CRM, ITSM, SOC, cybersecurity, and business operations. It lists Mission Control, Mission CRM, Mission Desk, Mission Ops, Mission SOC, Mission Secure, Mission GRC, Mission Finance, Mission AI, and Mission Connect.
- Read `CRM/README.md` on `main`. It describes a separate CRM domain repository. This does not supersede the user's clarification about the primary platform.

### Corrected decisions
- `cybrcops-platform` is the primary platform repository, source of truth, and main Julie integration target for CRM/SOC and related platform capabilities.
- The separate `CRM` repository is not the primary platform and is not to be treated as the primary CRM/SOC integration target. Revisit only for a specifically bounded use if the user directs it.
- Twenty remains explicitly excluded from Julie's architecture, dependencies, and code reuse.
- The platform README is a product map, not proof that every listed module is fully implemented. Next audit must inspect platform source, schemas, APIs, tenant model, and service maturity before asserting implementation status.

### GitHub changes
- Updated `docs/REPOSITORY-CAPABILITY-INVENTORY.md` on `feat/julie-repository-audit` to make this distinction explicit and prioritize platform architecture review.
- Inventory update commit: 57992261efd2c39edb6bfdd13f7dc09b07e6e745
- This clarification is documented in this implementation log.

### Validation and operational impact
- Documentation/source review only; no build, tests, deployment, or runtime integration performed.
- No NUC, Hostinger, Cloudflare, DNS, production services, or customer data touched.


## 2026-11-14 — Platform authorization and CYBR COM console source audit

### CYBR MISSION platform source inspected

Repository `cybrcops/cybrcops-platform`, branch `main`:
- Latest observed commit: `dc45caf5e1946bead8e9ea9814a526044ee1a43b` (2026-11-04, release of real CRM and Tickets screens).
- Inspected `apps/web/src/hooks/use-supabase-auth.ts`, `mission-identity.ts`, `authorization.server.ts`, `security/policy.ts`, `security/capabilities.ts`, `mission-resources.ts`, `routes/api.mission.ts`, `routes/api.crm.ts`, `api.server.ts`, `rate-limit.server.ts`, `mission-client.ts`, and `portal-product/crm.ts`.
- Verified at source level: AAL2 client gating; server Bearer JWT verification; exact subject check; one active organization membership; active role; explicit role permission and active permission; `has_permission` RPC; caller-scoped Supabase client using anon/publishable key, with service-role keys rejected.
- Verified at source level: closed resource/field allowlist; POST-only `/api/mission` with organization filter, private no-store response and 100-row cap; `/api/crm` lead create/move path with `crm.manage`, strict input validation, organization filter and caller JWT.
- Commit `dc97769587163c6039076a9f4dbbfd8f9d8bad3c` added live read-only ticket UI. The resource allowlist includes clients, leads, tickets, alerts, incidents, policies, contracts and invoices; this is not proof all product modules have complete live UI.
- Open issues: actual migration/RLS SQL was not located in the paths searched; no live database query was made; complete route inventory is not finished; API logging does not show explicit actor/tenant audit metadata in the inspected wrapper; in-memory per-process rate limiting and the trusted `X-Forwarded-For` proxy assumption need validation.
- Commit `2e6968585be1909c8185f77000952e5a0dfab425` documents migration 023 restoring the public `has_permission` wrapper after migration 020 moved the helper to `private`. Commit notes report applied migration and throwaway Postgres checks; this run did not independently validate production DB state or replay SQL.

### CYBR COM command-console source inspected

Repository `cybrcops/Cybrcom`, branch `main`:
- Latest observed main commit: `b7c86e0b77affb58b09c1e0e959108e0af6ca630`, merge of PR #3 from `feat/command-console-live` on 2026-11-13.
- Branch listing confirms `site` exists. The live public website/Hostinger source remains that branch by owner decision; it was not fetched, changed, repointed, or deployed.
- Inspected `README.md`, `package.json`, `src/App.tsx`, `src/console/CommandView.tsx`, `useConsoleState.ts`, `MiraPanel.tsx`, `voice.ts`, `server/index.ts`, `server/app.ts`, `server/agent/mira.ts`, `tools.ts`, `approvals.ts`, `records.ts`, `console/agents.ts`, `console/live.ts`, SOC store/poller/intake/signature, and `vite.config.ts`.
- Corrected the earlier inventory statement that Cybrcom was only a synthetic frontend: main is now a hybrid pre-production prototype. Command and Security Feed connect to a local server; landing/Overview/Tasks/Review remain synthetic; Connections reports disconnected.
- Confirmed current Mira is Anthropic SDK/Claude Opus 5.5-coupled with default provider fallback and default USD 10/day budget; zero budget falls back to 10 due `Number(value) || 10`. This violates current no-paid/no-paid-fallback policy and must not be enabled for CYBR operations.
- Console uses loopback-only access when no token is set, or a shared static Bearer token when configured; no per-user identity/MFA/step-up was found in the inspected server. Do not expose it publicly.
- SOC push intake uses timestamped HMAC-SHA256 and a five-minute skew window; local SQLite stores events/incidents. Optional pull polling runs every 60 seconds and sends a bearer key to an environment-configurable URL without a visible host allowlist. Add HTTPS/destination allowlisting before privileged use.
- The roster is static and planned times are computed from fixed values; status JSON is not proof that agent processes are executing. There is no verified full workforce scheduler or 24/7 runtime.
- Approval queue stores request-time arguments and expires after 30 minutes, but the inspected flow lacks payload-hash binding, MFA step-up, and tamper-evident audit. Run records store clipped but not explicitly classified/redacted owner messages and tool I/O.
- Voice is explicit push-to-talk; Chrome/Edge recognition sends audio to browser-vendor speech services. No installable PWA/iPhone validation was performed. No Julie EA persona or CYBR MISSION/CYBRCOPS cross-business adapter was found in inspected paths.

### Documentation changes

- Created `docs/CYBR-COMMAND-CONSOLE-AUDIT.md` — commit `6be23becf85f0af805de80e2f60e3fdc622bc1b7`.
- Updated `docs/PLATFORM-INITIAL-AUDIT.md` with partial auth/API findings and remaining RLS gaps — commit `2ad0b642261d186a970045324701a598d2e7d0df`.
- Updated `docs/REPOSITORY-CAPABILITY-INVENTORY.md` to correct Cybrcom status and add repo freshness evidence — commit `554d905e414b38b1b45ecf6699e5141d0dacd1ce`.
- Existing `docs/JULIE-MIRA-OPERATING-MODEL.md` continues to define Julie as the owner's direct Personal EA and Mira as Chief of Staff, with separate owner-private, company, and customer scopes.

### Validation and operational impact

- Static GitHub source inspection and documentation commits only.
- No package installation, build, lint, typecheck, tests, security scan, live API request, database query, runtime execution, or device test.
- No NUC, Hostinger, Cloudflare, DNS, `site` branch, public website, production service, customer data, or secret was touched.
- No code, schema, dependency, model, infrastructure, or deployment configuration was changed.
- Existing draft PR #1 on `feat/julie-repository-audit` remains the human-review path; no merge or deployment was performed.

### Next work

1. Locate and inspect the platform's actual migration/RLS source and enumerate all API routes, server clients, and privileged operations.
2. Build synthetic authorization tests for wrong-tenant, expired membership, disabled role/permission, non-AAL2, customer role, and unauthorized module access.
3. Define a read-only Julie adapter contract against verified `cybrcops-platform` endpoints, with explicit CYBR MISSION/CYBRCOPS/customer scopes and source/freshness metadata.
4. Refactor Cybrcom's model path behind a free/local-only provider interface; remove paid default fallback and make zero budget a hard disable.
5. Design proper owner identity/MFA, action-bound approval hash, immutable audit, and independent kill switch before any remote command-console access.


## 2026-11-14 — Live CYBR MISSION Supabase catalog and advisor review

### Work performed

- Confirmed connected GitHub metadata for `cybrcops/cybrcops-platform` (private, default branch `main`, observed head `dc45caf5e1946bead8e9ea9814a526044ee1a43b`) and `cybrcops/jarvisAI` (public; review branch `feat/julie-repository-audit`).
- Located the actual SQL history under `cybrcops-platform/supabase/live/`; it is not a conventional `supabase/migrations/` directory. Read the live migration README and SQL 019–023 plus the security-foundation design document.
- Connected Supabase project listing identifies `cybrmission-portal` as ACTIVE_HEALTHY and `cybrmission-security-lab` as INACTIVE. Only the portal was queried, and only for read-only metadata/catalog information.
- Queried live migration history: 28 entries, through `public_permission_check_023`.
- Queried live public schema: 91 tables, 91 RLS enabled, 375 policies. A catalog query confirmed all 91 ordinary public tables had RLS and at least one policy.
- Inspected selected policy definitions and SECURITY DEFINER function definitions. No business/customer row data was queried.
- Retrieved live Supabase security, health, and performance advisors.

### Findings and decisions

- The prior audit's statement that actual migration/RLS SQL had not been located is superseded: SQL is present in `supabase/live/`, and the connected project records migrations 019–023.
- Production catalog is consistent with the intended 019–023 hardening at a structural level: RLS enabled on all 91 public tables; 375 policies; six private and two public SECURITY DEFINER functions.
- Security advisor warning: leaked-password protection is disabled. No setting change made.
- Performance advisor: unused-index warnings plus four multiple-permissive-policy warnings on dashboard_layouts, dashboard_widgets, incident_tasks, and invites. No index/policy changes made.
- `public.rls_auto_enable()` catches all exceptions while attempting RLS enablement and logs rather than rethrows. Review as a fail-closed DDL guard; a failed enablement may not abort the originating table creation.
- `public.soc_ingest_feed_batch(text,jsonb)` is SECURITY DEFINER, empty search_path, and ACL-limited to postgres/service_role. It writes event/incident/heartbeat records and must remain a narrow server integration path, not a general Julie tool.
- `private.handle_new_user()` is SECURITY DEFINER with search_path public; ACL is restricted to postgres/service_role/supabase_auth_admin. Review the exception and harden only after compatibility proof.
- Structural catalog inspection is not a tenant-isolation test. Do not infer authorization safety from an owner/service-role query.

### Validation actually performed

- Read-only Supabase project metadata, migration history, table metadata, advisor retrieval, and SELECT-only catalog queries against pg_class, pg_policies, pg_proc, and pg_namespace.
- No customer/business rows were read. No DDL, DML, project settings, credentials, secrets, or migrations were changed.
- No local CLI, isolated branch, build, test suite, penetration test, external scan, or device test was run.
- No NUC, Hostinger, Cloudflare, DNS, live website, or production application configuration was changed. The production database was queried only for read-only metadata/catalog state.

### Documentation

- Added `docs/CYBR-MISSION-LIVE-DATABASE-AUDIT.md` on `feat/julie-repository-audit`.
- Updated `docs/REPOSITORY-CAPABILITY-INVENTORY.md` with verified database evidence and outstanding risks.
- Updated this log. Changes remain within existing draft PR #1; no merge or deployment.

### Next

1. Review the complete policy inventory and function ACLs in an isolated test environment; test with ordinary authenticated synthetic tenant identities.
2. Inspect the complete definition/trigger attachment for `rls_auto_enable`, and review `handle_new_user` search-path assumptions.
3. Build a read-only CYBR MISSION adapter contract from verified endpoints; do not use service-role credentials.
4. Record explicit connector owner, tenant scope, allowed fields, refresh cadence, and revocation procedure for CYBRCOPS systems before adding cross-business reporting.
5. Keep production settings, SQL, RLS policies, indexes, and ingestion functions unchanged pending owner approval.

## 2026-11-14 — Sensitive-column privilege review

- Queried PostgreSQL column privilege metadata only; no column values were read.
- Confirmed authenticated role SELECT privilege on `api_keys.key_hash`, `integrations.config/secret_reference/last_error`, `webhooks.headers/signing_secret_reference/endpoint_url`, and `soc_ingest_sources.source_key`.
- This is a confirmed sensitive-column exposure surface, not proof of raw-secret storage or disclosure. RLS restricts rows but does not hide columns.
- Recorded remediation design: safe-column grants/views, server-side vault references, no sensitive fields in model context/logs, and ordinary-authenticated synthetic tests. No production grants/policies/views/data were changed.
- Operational impact: read-only metadata query only; no secrets or customer rows read; no NUC/hosting/DNS/live site/production configuration changed.

## 2026-11-14 — CYBR MISSION API/authentication source audit and Julie adapter contract

**Target:** `cybrcops/cybrcops-platform`, branch `main`, observed commit `dc45caf5e1946bead8e9ea9814a526044ee1a43b`. Static GitHub source review; exact file SHAs and contract are recorded in `docs/JULIE-CYBR-MISSION-READ-ADAPTER-CONTRACT.md`.

### Verified implementation
- React Router exposes `POST /api/mission`, `POST /api/crm`, and `GET /api/health`; unmatched API paths return 404.
- `POST /api/mission` requires a Bearer Supabase access token and passes it to Supabase Auth `getUser(token)`. It binds JWT `sub` to the provider-validated user, requires `aal2`, checks one time-valid active membership for the requested organization, checks active non-customer role, checks role_permissions and permissions, and calls `has_permission` for the same org and capability.
- The Data API client uses the caller JWT and a publishable/legacy anon key; source explicitly rejects service-role/secret key formats. This is positive evidence of caller-scoped RLS, not an invitation for Julie to bypass the route.
- `mission-resources.ts` is a closed allowlist of eight resources with explicit safe field projections and per-resource permissions; list results are limited to 100. Count mode is only supported for clients, tickets, alerts, incidents.
- API wrapper applies rate limiting, request-body caps, no-store/security headers and sanitized request-ID error logging. The mission route uses POST to avoid documented edge-cache risk for per-user GETs.
- `POST /api/crm` is a write endpoint requiring `crm.manage`; excluded from Julie's read adapter. `GET /api/health` returns only `{ok:true}` and is liveness only, not deep health.
- Browser auth persists Supabase sessions; workspace selection is stored in localStorage. Server route revalidates identity/membership/permissions, so localStorage is not authority.
- App route requires AAL2 and staff-role allowlist.

### Integration decision
- Existing route is the first suitable CYBR MISSION read seam. It is user-bound, not a machine-to-machine service integration.
- No unattended Julie connector is authorized or implemented yet. Need an owner-approved delegated identity / non-human principal / dedicated server route design, secure token custody, refresh and revocation, explicit organization/resource grants, customer delegation and audit correlation.
- No service-role key, direct database table access, or privileged RPC use for Julie. Do not trust model-generated organization IDs.
- Resource projections include lead contact data and invoice amounts; owner briefing should default to minimized summaries/counts and only fetch sensitive business fields for an explicit purpose.
- Live DB catalog review separately confirms authenticated SELECT privileges on sensitive columns. Route projections must remain a hard boundary until safe-column grants/views and synthetic tests are complete.

### Documentation and validation
- Added `docs/JULIE-CYBR-MISSION-READ-ADAPTER-CONTRACT.md` with endpoint schema, resource allowlist, auth sequence, trust boundaries, proposed normalized interface, blockers and test plan.
- Static source inspection only. No endpoint call, token issuance, build, lint, tests, scans, customer row reads, schema changes, permission changes, or production writes.
- No NUC, Hostinger, Cloudflare, DNS, public website, or production application changes; no secrets or personal data committed.

## 2026-11-14 — AI runtime/workforce component source review (supersedes earlier README-only capability assumptions)

See the detailed `docs/AI-RUNTIME-AND-WORKFORCE-COMPONENT-REVIEW.md` for exact evidence, paths and hashes.

### Upstream drift verified
- `cybrcops/freellmapi` MIT fork is 8 commits behind `tashfeenahmed/freellmapi`.
- `cybrcops/ruflo` MIT fork is 3 commits behind `ruvnet/ruflo`; fork package 3.77.0 vs upstream release commit 3.77.1.
- `cybrcops/VoiceStudio` AGPL-3.0 fork is 35 commits behind `debpalash/VoiceStudio`.
- `cybrcops/OpenJarvis` Apache-2.0 fork is 1 commit behind `open-jarvis/OpenJarvis`.
- No upstream sync or merge performed. Review exact fork source and do not infer it contains upstream fixes.

### FreeLLMAPI correction and risk
- Source confirms a powerful OpenAI-compatible proxy, provider catalog, automatic routing/failover (architecture docs describe up to 20 attempts), encrypted provider-key storage, a single-user unified API key, session-gated admin API, provider probes and SQLite backup scheduler.
- **Corrected binding statement:** Docker Compose publishes the host port on loopback by default, but the Node server itself defaults `HOST=::` and falls back to `0.0.0.0` when IPv6 is unavailable. A direct source/npm run is therefore not loopback-only by default despite the SECURITY.md summary. Explicitly set and verify `HOST=127.0.0.1` before any future local run. No runtime was started.
- FreeLLMAPI is documented as a single-user trusted-network tool, not multi-tenant. Automatic routing/fallback and arbitrary configured/custom providers are incompatible with Julie's zero-paid-fallback policy unless an external deny-by-default provider+model allowlist, no-fallback profile and zero-cost gate are enforced and tested.
- Docker Watchtower profile mounts host Docker socket; keep autoupdate disabled.

### Ruflo execution gate
- Nested bridge defaults loopback, requires bearer token for non-loopback bind, and denies terminal execution unless `MCP_ENABLE_TERMINAL=true`.
- Agent, memory, devtools and intelligence tool groups default enabled in the example configuration; child MCP processes are spawned via npx and inherit environment variables other than `MCP_AUTH_TOKEN`.
- No OS sandbox, tool-level tenant identity, network egress control, file-root enforcement or resource-limit proof established in this source pass. Keep all worker groups disabled until verified; terminal/browser/file/GitHub/network tools remain unavailable to Julie.

### VoiceStudio licensing gate
- Application is AGPL-3.0; application internal/commercial use is allowed subject to AGPL obligations, but model weights/tokenizers/auxiliary assets are separate.
- Model inventory identifies 63 model repositories plus seven unresolved asset families; no current model record asserts `commercial_use: true`; review workflow says no production model is currently review-ready. No VoiceStudio model is cleared for CYBR business use on this evidence.
- MCP supports voice generation, cloning and transcription; path-based files rely on a configured base path, and base64 mode can place audio in agent context. Keep it isolated, no raw audio in prompts/logs, and no admin/high-impact endpoints exposed.

### OpenJarvis research boundary
- Apache-2.0 local-first framework has agents/engines/tools/MCP/memory/scheduler and many powerful built-in tool imports (file write, shell, Docker shell, Git, DB, HTTP, browser, audio, connectors).
- Docker sandbox source uses network-none and read-only allowlisted mounts, with 300s timeout and max concurrency 5 defaults; still requires review of container privileges, root filesystem, resource limits, runtime socket, image provenance and actual mount allowlist before use.
- Candidate research for local inference/evaluation/scheduler/sandbox patterns only; no code reused.

No code/dependency install, upstream sync, runtime launch, model call, build/test, NUC access, production change, or external scan performed.

## 2026-11-14 — JARVIS foundation source audit (Julie suitability)

See `docs/JARVIS-FOUNDATION-SOURCE-AUDIT.md` for exact source paths and design decisions.

- `jarvisAI` main is MIT, Node >=20, React/Vite/Three.js. The current runtime still imports Claude Agent SDK and Anthropic SDK; FreeLLMAPI/local model support remains a documented target, not an implemented adapter.
- Direct browser mode enables `dangerouslyAllowBrowser` and reads `VITE_ANTHROPIC_API_KEY`; must be removed/disabled for governed company builds.
- Local bridge defaults loopback and has Host/Origin gates, a PreToolUse hook, path checks, and SSRF protections. It is still not authenticated to owner/company/customer identities and runs under the current OS user's authority.
- If `JARVIS_MCP_SERVERS` is unset, bridge passes every configured MCP server from the user's Claude config. Do not inherit home MCP config in Julie; use a closed broker-managed tool registry.
- Current global `JARVIS_ALLOW_WRITES=1` / `--writes` toggle is not an action-specific approval system. Keep production write capability out of the current bridge.
- `src/lib/voice.ts` intentionally keeps browser recognition running for the life of the page and listens for wake word/barge-in. This conflicts with Julie's required foreground iPhone push-to-talk; implement a separate PTT lifecycle, no background/continuous microphone.
- No manifest/service worker found in reviewed `public/` inventory and no PWA plugin in root package. Installable PWA status is not established.
- Current store is UI-oriented (phase/panels/blades/turns) and does not implement operational inventory, alerts, remediation, approvals, incident timeline, audit, tenant scopes, or Julie/Mira coordination.
- Reuse only selected UI/animation and reviewed local protections as inspiration. Do not reuse current provider runtime, direct browser key path, auto-imported MCP set, global write switch or continuous wake-word microphone as-is.
- No code/dependency/runtime changes; static source inspection only. No NUC/hosting/DNS/live site/production/customer data/secrets touched.

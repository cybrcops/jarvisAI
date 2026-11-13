> **Repository clarification (2026-11-13):** The live CYBR COM website is connected to `cybrcops/Cybrcom`, not this repository. The current command-console UI candidate is also in `cybrcops/Cybrcom` (`feat/milestone-1-honest-demo`, an unvalidated synthetic demo draft). Do not use this document or the `hostinger-staging` branch in this repository as the deployment source for the existing CYBR COM website or its command-console staging. The authoritative staging plan is [in the Cybrcom repository](https://github.com/cybrcops/Cybrcom/blob/feat/mission-center-security-operations/docs/DEPLOYMENT-TOPOLOGY-HOSTINGER-NUC.md). This repository remains a separate JARVIS reference/runtime project.

# CYBR COMMAND — Hostinger + NUC Staging Deployment

**Status:** Approved architecture / staging preparation only  
**Production status:** Not deployed  
**Live website impact:** None intended  
**Public website:** Existing custom HTML/CSS/JavaScript site; preserve its current GitHub connection and deployment configuration  
**Staging hostname proposal:** `command-dev.cybrcom.com` (not yet created or DNS-configured)

## 1. Decision

Use the existing Hostinger Unlimited web-hosting subscription for the static JARVIS/command-console frontend, provided the account panel confirms the required separate-site and Git deployment options. Keep the existing CYBR COM public website exactly as it is.

Use the CYBR COM NUC as the private execution environment for JARVIS bridge services, Ruflo, FreeLLMAPI, local inference, and approved agent workers. Use Supabase as the durable state/audit layer where configured. GitHub remains the source of truth.

Do not upgrade Hostinger or purchase additional hosting for this staging phase.

## 2. Existing website protection boundary

The current `cybrcom.com` website is custom HTML/CSS/JavaScript and is already connected to GitHub. Treat its current Hostinger website, GitHub repository/branch mapping, document root, DNS, SSL, email and deployment settings as production assets.

**Prohibited during staging setup:**
- Do not change the current website's connected repository, branch, deployment mode or document root.
- Do not upload or deploy JARVIS/CYBR COMMAND files into the existing site's `public_html`.
- Do not change apex, `www`, MX, SPF, DKIM, DMARC or existing DNS records.
- Do not repoint the main domain or replace the existing site.
- Do not disconnect/re-authorize the current GitHub integration as a troubleshooting shortcut.
- Do not publish NUC, Ruflo, FreeLLMAPI, Ollama or bridge management ports.

Before any Hostinger panel changes, export/screenshot the existing website's deployment configuration and DNS zone and create a downloadable website backup. The backup is precautionary; no production migration is planned.

## 3. Proposed topology

```
Operator browser
    |
    | HTTPS
    v
command-dev.cybrcom.com
Hostinger separate staging website
Static React/Vite build (no secrets)
    |
    | HTTPS/WSS to authenticated API endpoint
    v
Private access gateway / outbound tunnel
    |
    v
CYBR COM NUC (not directly exposed)
  - JARVIS bridge / runtime adapter
  - CYBR COMMAND API and policy broker (as implemented)
  - Ruflo workers (restricted)
  - FreeLLMAPI on loopback
  - Local model runtime
    |
    +---- Supabase (persistent mission/task/approval/audit state)
    |
    +---- GitHub (source, pull requests, CI)
```

The NUC initiates the private tunnel outward. No router port-forwarding or direct public exposure of NUC services is permitted. The gateway must enforce authentication, authorization, workspace scope, rate limits, request validation, and audit logging. Use TLS for browser-to-gateway traffic and an authenticated encrypted tunnel for gateway-to-NUC traffic.

The tunnel/gateway product is a later implementation decision. Do not assume a provider, DNS proxy, or access product has been configured.

## 4. Hostinger deployment shape

Create a **separate website** for the staging hostname, not a subdirectory of the existing CYBR COM website. Hostinger's current guidance says a subdomain Node.js app must be created as its own website. Confirm the exact account-panel flow and DNS requirements before creating it.

Preferred first deployment:
- Static frontend build from `cybrcops/jarvisAI`.
- Repository: `cybrcops/jarvisAI`.
- Branch: dedicated staging branch (proposed: `hostinger-staging`).
- Root directory: repository root, after confirming the frontend build layout.
- Install/build: `npm ci` then `npm run build`.
- Build output: `dist/`.
- Runtime: static files only; do not start `bridge/server.mjs` on Hostinger.
- No private environment variables or provider keys in frontend build variables.
- Use only public, non-secret configuration such as the public API origin, and only after the gateway exists.

The current JARVIS repository has a root Vite frontend and a Node bridge. The bridge is not part of the static deployment. The current app still has Claude Agent SDK coupling; the provider-neutral free-model architecture is a planned change, not yet implemented. Do not label the staging build as a production-ready free-model JARVIS until the adapter and authentication work is complete.

If Hostinger's static Git deployment cannot safely target the separate staging website and branch without modifying the existing site's GitHub linkage, stop and use a manual ZIP upload or a separately scoped deployment workflow. Do not change the live website integration to force staging.

Hostinger's Unlimited plan currently lists Node.js web-app support, but the first deployment should not need a persistent Node process. Hostinger documents limits for web/cloud Node.js hosting: no inbound WebSockets, no always-running background workers, no Docker/Redis/custom ports, and a 120-second request timeout. These are additional reasons to keep the bridge and workers on the NUC.

## 5. Repository and branch plan

- `cybrcops/jarvisAI`: JARVIS frontend, voice experience and runtime adapter.
- `cybrcops/Cybrcom`: command API, policy/approval authority, audit model and deployment governance.
- Keep existing public website repository and its deployment branch unchanged.
- Create a staging-only branch in `jarvisAI` after the frontend build baseline and security review. Do not point Hostinger at `main` for automatic production deployment.
- Production promotion requires a reviewed PR, successful build/tests/security checks, explicit operator approval and a rollback plan.

## 6. Real-data and non-fake UI contract

The dashboard must never invent operational state.

Every status item must include:
- Source system and source identifier.
- Observation/event timestamp (UTC internally; localized only in display).
- Freshness/heartbeat age.
- State: healthy, degraded, offline, unknown, not configured, or demo.
- Evidence or event reference where applicable.

Rules:
- Agent is online only when a recent authenticated heartbeat is received.
- Agent is working only when a real task is leased/running and has an execution ID.
- Model is available only after a real health check and eligibility/quota evaluation.
- Vulnerability, incident and compliance metrics must come from connected source records and disclose coverage.
- Unknown data must render as **Unknown**, **Not connected**, or **No data yet**.
- Synthetic fixtures must be isolated behind an explicit Demo Mode banner and must never enter production metrics, audit records or customer workspaces.
- Never present an AI recommendation as an executed action or a compliance certification.

## 7. Authentication and security gates

Before staging is accessible beyond the operator:
1. Authentication is required; no anonymous dashboard or public API.
2. Enforce MFA and role-based access.
3. Apply tenant/workspace isolation in the API and database, not only in frontend routing.
4. Keep secrets server-side on the NUC or approved secret store. Never put tokens in `VITE_*`, browser storage, source code, build artifacts, prompts or logs.
5. Use short-lived sessions/tokens, secure cookies where applicable, CSRF protections for cookie-authenticated writes, and strict CORS/origin allowlists.
6. Validate every API request and authorize every action server-side.
7. Model-generated tool calls are proposals; CYBR COMMAND policy/tool broker makes the authorization decision.
8. Start read-only. No shell, arbitrary filesystem, unrestricted MCP, production writes, or autonomous external actions.
9. Use action-specific, short-lived, one-time approvals for any permitted write.
10. Rate-limit and audit login, API, model, tool, approval and execution events.
11. Provide independent stop controls for model calls, agents, browser automation, external writes and voice capture.
12. Do not send confidential customer data to free hosted model endpoints unless policy explicitly approves the data class and provider.

## 8. Staging acceptance checklist

### Website isolation
- [ ] Existing `cybrcom.com` content and URL behavior unchanged.
- [ ] Existing GitHub repository/branch connection unchanged.
- [ ] Existing DNS and mail records unchanged.
- [ ] Staging is a separate Hostinger website/document root.
- [ ] Staging TLS certificate is valid.
- [ ] Staging rollback/deletion does not affect the public site.

### Build and supply chain
- [ ] Pin Node and package manager versions.
- [ ] `npm ci` succeeds from lockfile.
- [ ] `npm run lint`, `npm test`, and `npm run build` pass.
- [ ] Dependency and secret scans pass.
- [ ] Build artifact contains no API keys, provider tokens, private URLs or customer data.
- [ ] Static deployment does not launch the bridge or workers.

### Connectivity and truthfulness
- [ ] Unauthenticated API requests are denied.
- [ ] NUC unavailable state is visibly **Offline/Unknown**, not simulated online.
- [ ] Agent status comes from authenticated heartbeat/task events.
- [ ] Model status comes from live health/quota checks.
- [ ] No sample telemetry is mixed into real records.
- [ ] API disconnect, timeout and stale-data states are visible.
- [ ] Audit events are recorded for all privileged actions.

### Go-live gate
- [ ] Security review complete.
- [ ] Backup and restore tested.
- [ ] Rollback tested.
- [ ] Workspace isolation tested.
- [ ] Human approval flow tested.
- [ ] Explicit owner approval recorded.

## 9. Hostinger operator procedure (not yet executed)

1. Sign in to Hostinger and open Websites. Identify the current CYBR COM website and record its repository/branch, deployment mode, root/document directory, DNS and SSL state. Do not edit it.
2. Create a backup and export the DNS zone.
3. Add `command-dev.cybrcom.com` as a separate website on the same Unlimited plan. Do not select the existing `cybrcom.com` website or its document root.
4. If Hostinger asks to connect GitHub, verify it is the same GitHub account already used and that the authorization does not remove or alter existing site permissions. If uncertain, stop before authorizing.
5. Select only `cybrcops/jarvisAI` and the dedicated staging branch once it exists. Confirm build command and `dist` output directory.
6. Do not add API secrets as environment variables to a static frontend.
7. Verify the deployment URL and staging SSL. Confirm the public CYBR COM website still serves its original content.
8. Keep staging private behind authentication; do not advertise or link it from the public website.

**No Hostinger changes, DNS changes, deployments, or NUC network changes are authorized by this document.** This file defines the safe plan; each external change is a separate operator action.

## 10. References

- Hostinger: Node.js limits for web and cloud plans — https://www.hostinger.com/support/hostinger-node-js-limits-for-web-and-cloud-plans/
- Hostinger: How to add a Node.js web app — https://www.hostinger.com/support/how-to-deploy-a-nodejs-website-in-hostinger/
- Hostinger: Node.js hosting options — https://www.hostinger.com/support/node-js-hosting-options-at-hostinger/

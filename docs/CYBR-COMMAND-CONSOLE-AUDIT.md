# CYBR COM command-console source audit

**Date:** 2026-11-14  
**Repository:** `cybrcops/Cybrcom`  
**Branch reviewed:** `main`  
**Latest commit observed:** `b7c86e0b77affb58b09c1e0e959108e0af6ca630` — merge of PR #3 from `feat/command-console-live`, 2026-11-13  
**Review type:** Static GitHub source inspection. No clone, install, build, test, deployment, or runtime access.

## Corrected repository status

The current README still calls the React frontend a local synthetic demo and says build/lockfile/visual verification are outstanding. That description is now incomplete: the merged main branch includes a local command UI, SOC event intake/polling, a local SQLite store, Mira chat, a limited tool policy, approvals, run records, and a work board.

The accurate status is **hybrid pre-production prototype**:
- The Command and Security Feed views connect to the local CYBRCOM server when it is running.
- Landing, Overview, Tasks, Review, and Connections retain local synthetic/demo behavior; Connections explicitly reports services disconnected.
- The server's live capabilities are limited to a local SOC event intake/board and a narrow Mira tool loop. This is not yet the governed cross-business command center or a fully automated workforce.
- The README needs a future documentation correction on a reviewed feature branch; it was not changed in this audit.

The repository is private, its default branch is `main`, and the `site` branch exists. The owner has specified that `site` remains the existing public website/Hostinger source. The `site` branch, Hostinger connection, document root, DNS, and live site were not opened or modified.

## Frontend and runtime

- Root `package.json`: Node `>=22.18.0`; React 19.3; Vite 8.3; TypeScript 7; direct `@anthropic-ai/sdk` dependency. Scripts include `build` (`tsc -b && vite build`) and a server test command. No build, test, lockfile, or visual check was run in this audit.
- `src/App.tsx`: Command view renders `CommandView`; Security Feed renders `SocView`. Landing and Overview/Tasks/Review use component-local synthetic state. The Connections view says the frontend has no service integration and lists Models, GitHub, Database, and Workers as disconnected.
- `src/console/CommandView.tsx` and `src/console/useConsoleState.ts`: the command view reads `/api/console/state`, subscribes to `/api/console/stream` using SSE, and polls every 15 seconds when the stream is down. This is a UI refresh mechanism, not a background monitoring guarantee.
- `vite.config.ts`: the development proxy forwards `/api` to `127.0.0.1:8787`.
- `server/index.ts`: server defaults to `127.0.0.1:8787`, stores state in local SQLite at `local-data/cybrcom.db`, optionally enables signed SOC push intake and 60-second pull polling, and reads agent status from a local file or a public `status.json` URL. No always-on external runtime or backup/restore process was verified.
- `server/console/agents.ts`: the roster and planned times are fixed in source. Actual last-run/state values come from a status JSON document. A displayed next-run time is calculated from the static roster; it is not proof that a scheduler or agent process is running. The default public status URL is a cached status source, not an independent live process monitor.

## Identity, approvals, and tool authority

- `server/app.ts`: with no console token, API access is restricted to loopback peer addresses. If `CYBRCOM_CONSOLE_TOKEN` is configured, a shared static Bearer token grants access to console state, SSE, chat, work-board operations, approvals, and SOC endpoints. Comparison is timing-safe, but this is not per-user identity, session binding, MFA, or a scoped service identity. No rate limiter was observed in this server's request handler.
- The JSON/Origin check in `writeAllowed` is useful CSRF friction for browser writes, but does not replace authenticated user identity and action-specific authorization. Requests without an Origin header are accepted by this helper; the console authorization check still applies.
- `server/agent/tools.ts`: tool definitions are closed and validate model-provided input. GREEN tools read the board, staff/status and Room feed; `add_work_item` writes a bounded work note; YELLOW `resolve_incident` creates an approval request; unknown/RED tools are refused. Only the local SOC incident-resolution executor is implemented. No CYBR MISSION production write tool is present in the inspected set.
- `server/agent/approvals.ts`: approval arguments are stored at request time, pending requests expire after 30 minutes, and the pending-to-decided state transition is conditional/atomic. The action then executes after the status transition. The approval UI shows a summary; the inspected flow does not bind approval to a canonical payload hash, require a single-use MFA step-up, or show a cryptographically verifiable action receipt. A process crash between decision and execution can leave a request approved without completion; reconciliation/idempotency needs design.
- `agent_audit` is a regular SQLite table written by application code. No database-level append-only protection, hash chain, remote immutable sink, or tested restore was verified.
- `server/agent/records.ts` stores run questions, tool inputs/results and reply excerpts (clipped, not classified/redacted by the shown code). Personal memory, company scope, customer tenant scope, retention, and deletion boundaries are not implemented in this store.

**Remote-access gate:** Do not expose this server beyond loopback or place it behind a public PWA until a real identity provider, MFA/step-up, per-user sessions, CSRF protection, action/tenant policy enforcement, rate limits, audit integrity, and independent kill switch are implemented and tested.

## Model and spend policy blocker

- `server/index.ts` enables Mira when Anthropic credentials are present and constructs `anthropicStream()`.
- `server/agent/mira.ts` imports the Anthropic SDK, pins `claude-opus-5-5`, includes the provider's default fallback behavior, and computes spend using Anthropic token prices.
- The server's budget expression defaults to USD 10/day. Because it uses `Number(value) || 10`, setting the environment value to `0` also falls back to 10 rather than disabling spend.
- This directly conflicts with the owner's current no-paid-API/no-paid-fallback requirement and provider-neutral/local-first direction. Do not enable this path for CYBR operations. Replace it with a server-side adapter to an explicitly allowlisted free/local provider profile, with paid providers hard-disabled and zero-budget fail-closed semantics. Verify the actual provider/model used for each call.

## SOC event intake and polling

- `server/soc/signature.ts`: verifies HMAC-SHA256 over timestamp plus raw request body, uses constant-time comparison, and rejects timestamps more than five minutes from now.
- `server/app.ts`: push intake is disabled when the webhook secret is absent, enforces a 1 MiB body cap, and refuses invalid signatures.
- `server/soc/intake.ts`: validates event fields/severity/type, caps a batch at 1,000 events, stores events idempotently by source+id, and applies incident rules in a SQLite transaction.
- `server/soc/poller.ts`: pulls from `https://cybrcops.com/api/soc/events` by default every 60 seconds when an API key is configured. The URL can be overridden by environment configuration and is passed to `fetch` without a visible scheme/host allowlist. Before this is used in a privileged environment, enforce HTTPS and a strict destination allowlist so the bearer key cannot be sent to an unintended host.
- Event payloads originate outside the command server and must remain untrusted data. Apply schema/data minimization and PII/secret redaction before persistence, display, or model context. The current review did not verify retention, encryption-at-rest, backups, or customer-data handling.

## Voice and Julie/Mira role alignment

- `src/console/voice.ts` implements explicit push-to-talk through the browser SpeechRecognition API and local browser speech synthesis for playback. In Chrome/Edge, recognition audio is sent to the browser vendor's speech service, as the UI note discloses. This is not a local-only speech path; use only with informed consent or replace it with an approved local/brokered adapter. No background microphone behavior is present in the inspected hook.
- The current UI/persona is Mira, Chief of Staff. There is no separate Julie Personal EA experience, private owner memory, cross-business data broker, or owner-private/company/customer scope separation in the inspected implementation.
- Mira reads the local staff/status board, Room feed, and work board. The code inspected does not connect to the primary `cybrcops-platform` APIs or provide authorized visibility across CYBR MISSION and CYBRCOPS.
- No installable PWA manifest/service-worker acceptance or iPhone device behavior was verified. Do not claim an installable iPhone app yet.

## Integration and operational implications

1. Keep `cybrcops-platform` as the first CYBR MISSION integration target. Build a read-only, server-side adapter only after its auth, RLS, tenant and route contracts are verified.
2. Keep CYBRCOPS as a separate company/data scope; do not infer a common organization ID or merge tenant namespaces.
3. Add Julie as the owner's direct Personal EA and consolidated interface. Keep Mira as Chief of Staff and workforce coordinator. Both report to the owner; neither is an unrestricted administrator.
4. Treat the current command server as a local prototype, not an always-on service. It cannot poll, ingest, alert, or execute while the host/server is stopped.
5. Keep all production actions disabled until identity, action-specific approval, evidence, audit, rollback, backup and emergency-stop gates are verified.

## Validation and impact

- Static GitHub inspection only; no package installation, build, lint, tests, browser/device checks, runtime calls, live endpoints, security scan, or deployment.
- No secrets or customer data were retrieved.
- No NUC, Hostinger, Cloudflare, DNS, `site` branch, live website, production service, or network configuration was changed.
- No code, dependency, schema, model, or deployment configuration was changed.

## Sources inspected

- `README.md`, `package.json`, `src/App.tsx`, `src/console/CommandView.tsx`, `src/console/useConsoleState.ts`, `src/console/MiraPanel.tsx`, `src/console/voice.ts`
- `server/index.ts`, `server/app.ts`, `server/agent/mira.ts`, `server/agent/tools.ts`, `server/agent/approvals.ts`, `server/agent/records.ts`
- `server/console/agents.ts`, `server/console/live.ts`, `server/soc/store.ts`, `server/soc/poller.ts`, `server/soc/intake.ts`, `server/soc/signature.ts`, `vite.config.ts`
- Commit `b7c86e0b77affb58b09c1e0e959108e0af6ca630` (2026-11-13).

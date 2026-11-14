# JARVIS foundation source audit — Julie/CYBR COMMAND suitability

**Audit date:** 2026-11-14  
**Repository:** `cybrcops/jarvisAI`  
**Source branch reviewed:** `main`  
**Observed source commit:** `2e1c908a` (repository inventory's recorded main revision; current branch was not modified)  
**License:** MIT  
**Review type:** static GitHub source inspection only.

## Executive assessment

The current `jarvisAI` repository is a visually rich browser voice-assistant foundation and local Node bridge. It is **not yet Julie, not provider-neutral, not an installable PWA, and not a governed CYBR COMMAND control plane**. The README itself says the bridge still uses the Claude Agent SDK and the FreeLLMAPI path is only an implementation target.

Keep the interface/visual work as a candidate foundation. Do not expose the current bridge as a CYBR operations service or connect it to customer/company tools until the blockers below are resolved.

## Verified source facts

### Runtime and paid-provider coupling
- Root `package.json`: Node >=20; React/Vite/Three.js; direct dependencies include `@anthropic-ai/claude-agent-sdk` and `@anthropic-ai/sdk`. There is no FreeLLMAPI client dependency in the inspected package.
- `bridge/server.mjs` imports Claude Agent SDK `query()`; defaults `JARVIS_MODEL` to `claude-opus-5` and obtains MCP servers from the user's `~/.claude.json` global and home-project scopes.
- `src/lib/anthropic.ts` creates an Anthropic client with `dangerouslyAllowBrowser: true`; direct mode sends `VITE_ANTHROPIC_API_KEY` in the browser bundle. This mode is unacceptable for any shared/company deployment and must be removed/disabled in the governed build.
- `src/config.ts` defaults backend to local bridge, but still includes direct browser key config. `docs/CYBR-FREE-AI-ARCHITECTURE.md` is an architecture plan, not an implemented adapter.

### Tool policy and autonomy
- `bridge/policy.mjs` has a meaningful two-layer tool policy: name-based allow/deny plus argument/path checks. It uses a `PreToolUse` hook because source comments document that SDK `canUseTool` alone did not see all auto-approved calls.
- Built-in read tools include Read/Grep/Glob/WebSearch/WebFetch/ToolSearch; write tools include Bash/Write/Edit/NotebookEdit and are gated by a single global `JARVIS_ALLOW_WRITES=1` or `--writes` flag.
- MCP policy defaults to deny except explicitly named read-only servers / read-verb tools, with effectful-verb vetoes. However, server allowlist `JARVIS_MCP_SERVERS` is optional; when unset, `allowedServers()` passes every configured server from `~/.claude.json`.
- MCP tools are classified by names and server allowlists, not a centrally verified per-tool schema, identity, customer tenant, purpose or action-specific approval. Name heuristics cannot be the final authorization authority.
- Path allowlist defaults to workspace, OS temp and `/tmp`; optional extra roots can widen it. Credential-directory/file name blocks are present. Review symlink/race behavior and ensure temp roots do not become a cross-scope data channel.
- Bridge includes outbound SSRF protections in `bridge/net.mjs` for its own proxy and a separate pre-check for built-in WebFetch. The code comments acknowledge that the WebFetch DNS pre-check cannot pin the actual address used by the SDK; DNS rebinding remains a residual risk for that built-in path.
- A global writes switch is not adequate for Julie. Every consequential action must have a canonical action payload, scope, policy evaluation, action-specific approval, expiry, single-use approval binding, execution receipt, audit record and kill-switch check.

### Bridge identity and exposure
- `bridge/server.mjs` defaults to `127.0.0.1`, checks Host and Origin on WebSocket handshakes, refuses missing Origin unless explicitly opted in, and has additional allowed host/origin lists. This is useful local-demo hardening, not authentication or a multi-user session boundary.
- Browser WebSocket access is origin/host gated, but there is no owner/company/customer identity or tenant authorization in the reviewed bridge.
- `bridge/server.mjs` discovers local Claude MCP configuration and, unless `JARVIS_MCP_SERVERS` is explicitly supplied, passes all configured MCP servers to the agent. Do not assume the current local MCP list is a safe production allowlist.
- The bridge runs with the current OS user's authority. No separate OS user, container sandbox, process quota, memory/CPU limit, egress policy, signed tool broker, or customer tenant sandbox was verified.

### Voice UX and PWA
- `src/lib/voice.ts` deliberately keeps one browser recognizer running for the life of the page; it supports a wake phrase (Jarvis variants), continuous listening while the assistant speaks, and barge-in. This is **not** the owner's required Julie iPhone experience.
- Julie requirement is foreground push-to-talk with visible microphone-on/off state and playback; no wake-word/background/continuous listening. Implement a separate PTT flow and do not reuse the current always-open microphone lifecycle.
- Root `public/` inventory contains audio assets and favicon only. Root `package.json` has no PWA plugin dependency; no web app manifest or service worker was found in the reviewed root/public inventory. Installable PWA status is not established.
- Current TTS config defaults browser system speech; optional Kokoro path is bundled. No claim of tested iPhone Safari behavior.

### UI and data model
- `src/store.ts` primarily models visual phase, panels, blades, turns and visual state. It does not define service inventory, health/alert state, findings, remediation queue, approval inbox, incident timeline, support workflows, patch proposals, audit events, company workspaces, customer tenant scopes, or owner-private memory separation.
- The current application is a single-person visual assistant; no verified multi-workspace data separation or Julie/Mira coordination protocol exists.

## Reuse decision

**Reuse candidate:** React/Three.js visual shell, animation system, voice playback abstractions only after PTT redesign, and selected bridge-local protections as inspiration with independent review.

**Do not reuse as-is:** Claude Agent SDK runtime, direct browser Anthropic client/key, automatic MCP import from home config, global `--writes` authorization, always-on wake-word voice loop, unrestricted current OS process authority, or any implied production readiness.

No source code or dependency was copied into another repo during this review.

## Julie/CYBR COMMAND required replacement architecture

1. **Provider-neutral inference adapter:** local model / FreeLLMAPI adapter behind server-side allowlist. No paid provider in build/runtime dependency path; exact provider+model pin, zero spend, no fallback and fail-closed behavior.
2. **CYBR COMMAND tool broker:** model output only creates typed proposals. Broker verifies authenticated session, owner/company/customer data scope, per-tool permission, action risk, approvals, kill switches, audit and bounded executor.
3. **Session and identity:** bind every WebSocket/session to authenticated owner identity, device/session, CSRF/origin protections, expiry/revocation and step-up for sensitive actions. Loopback is not identity.
4. **Separate data domains:** owner private, CYBR MISSION, CYBRCOPS and each customer tenant. Separate stores/keys/context retrieval; explicit cross-domain data grants only.
5. **Safe defaults:** no tools enabled unless allowlisted; no MCP config inheritance from home; no general shell; no broad write switch; no direct filesystem outside dedicated scoped workspaces.
6. **Approval protocol:** immutable canonical action proposal, policy decision, human approval bound to exact payload hash/scope/expiry, execution state machine, idempotency, signed/append-only receipt, rollback and independent stop switch.
7. **iPhone PWA:** foreground PTT only, visible mic indicator, no microphone while idle/background, safe interruption, playback, offline/static shell only if it cannot expose secrets or stale authorization.

## Validation and operational impact

- Static GitHub source reads only.
- No npm install, build, lint, tests, runtime, browser/device test, model call, security scan or deployment.
- No secrets, personal data, customer data or local MCP configuration values were retrieved.
- No code changes to `jarvisAI` main; audit documentation only on `feat/julie-repository-audit`.
- No NUC, Hostinger, Cloudflare, DNS, live site, network, production service or customer data was touched.

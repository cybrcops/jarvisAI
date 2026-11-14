# Julie repository capability and integration inventory

**Audit date:** 2026-11-14  
**Scope:** Repositories visible to the connected GitHub account under `cybrcops` (50 repositories returned in the first listing).  
**Status:** Initial discovery and focused first-pass review. This is not a claim that all 50 repositories have received a full code, dependency, CI, runtime, or security audit.

## Executive decision

Build Julie as a governed product layer in `cybrcops/jarvisAI`, but do not treat any adjacent repository as a trusted runtime dependency until its code, license, dependencies, permissions, and operational boundary have been reviewed.

Preferred initial composition:

- **Julie experience:** adapt the existing React/Vite interface after removing always-on/wake-word assumptions for iOS. Use explicit foreground push-to-talk and accessible text chat.
- **Speech:** VoiceStudio is a candidate speech service behind a narrow adapter. Keep it isolated; expose only required transcription/synthesis operations through a broker. Never expose its administrative surface. Select an exact model/weight only after its license and commercial rights are verified.
- **Model gateway:** FreeLLMAPI is a candidate OpenAI-compatible routing layer. Treat free quotas/catalogs as volatile; configure an explicit allowlist, strict budget/rate limits, provider status display, and fail-closed behavior. No paid fallback or premium catalog purchase.
- **Orchestration:** Ruflo is a candidate for development-time or bounded agent workflows, not the authority plane. Its tool permissions, shell/network execution, MCP configuration, and sandboxing require review before workers are enabled.
- **Policy/approvals/audit:** CYBR COMMAND / Cybrcom architecture owns authorization, approvals, tenant scope, audit, emergency stop, and action policy. Julie coordinates but does not grant itself permissions.
- **CYBR MISSION platform / CRM / SOC:** `cybrcops-platform` is the primary platform repository and source of truth for the unified CRM, ITSM, SOC, cybersecurity, and business-operations ecosystem. Julie must integrate with and support this platform's architecture. The separate `CRM` repository is not the main platform and must not be treated as the primary CRM/SOC source of truth without a later explicit decision. Twenty remains excluded.
- **Memory:** implement a Julie-owned, consented, scoped memory contract; do not treat agent trace stores, CRM notes, or speech artifacts as Julie personal memory.
- **Infrastructure:** no NUC, Hostinger, Cloudflare, DNS, live site, or production changes are authorized by this audit.

## Repository inventory: discovery list

The following 50 names/default branches were returned by the connected GitHub repository listing. A repository listing does not establish freshness, license, build status, or safety. Those remain unverified unless stated in the focused review below.

| Repository | Default branch | Initial classification / disposition |
|---|---|---|
| `myrepo` | `master` | Unclassified; inspect if relevant |
| `node-app` | `master` | Unclassified; inspect if relevant |
| `aws-codepipeline-s3-codedeploy-linux` | `master` | Deployment reference; inspect before reuse |
| `project2` | `master` | Unclassified |
| `myproject` | `main` | Unclassified |
| `cybrcops` | `main` | Organization/site or starter repo; inspect |
| `cybrcops-platform` | `main` | **Primary CYBR MISSION platform: unified CRM/ITSM/SOC/security/business operations; main integration target and source of truth** |
| `CRM` | `main` | Separate CRM repository; not the primary CYBR MISSION CRM/SOC platform; secondary/unselected pending explicit direction |
| `Cybrcom` | `main` | CYBR COM website / command-console repo; protect live site branch |
| `cybrmission-website` | `main` | Public website candidate; inspect deployment relationship |
| `AI` | `main` | AI collection; README is only a short description |
| `awesome-llm-apps` | `main` | LLM application examples; evaluate provenance/dependencies |
| `twenty` | `main` | **Excluded by user decision (2026-11-14); no further review or reuse** |
| `zero-breach-toolkit` | `main` | Defensive spyware-evidence triage; read-only intent, validate limits |
| `data-removal-service` | `main` | Privacy/data-removal service; inspect before integration |
| `fieldwork` | `main` | Unclassified / possible operational app |
| `clawd-code` | `main` | Agent/code tooling; inspect execution permissions |
| `chatwoot` | `develop` | Customer-support platform; integration candidate |
| `sapper-v2` | `master` | Unclassified |
| `freellmapi` | `main` | OpenAI-compatible model router; candidate gateway |
| `ruflo` | `main` | Agent meta-harness; candidate orchestrator |
| `comp` | `main` | Unclassified |
| `OpenJarvis` | `main` | Local-first personal AI framework; research/reference |
| `JARVIS` | `main` | HuggingGPT research implementation; historical/experimental reference |
| `jarvisAI` | `main` | Julie foundation candidate; current bridge is Claude Agent SDK-coupled |
| `claude-skills` | `main` | Large skills library; curate, don't bulk-install |
| `claude-skill` | `main` | Skills collection; inspect for overlap |
| `claudedesignskills` | `main` | Design skills; optional frontend workflow reference |
| `pentagi` | `main` | Autonomous penetration-testing system; isolated lab only, not general Julie tools |
| `awesome-pentest` | `master` | Security reference list; not executable dependency |
| `PayloadsAllTheThings` | `master` | Security research reference; handle as untrusted content |
| `bla` | `main` | Unclassified |
| `Claude-Red` | `main` | Security/agent project; inspect before reuse |
| `ai-engineering-from-scratch` | `main` | Learning/reference material |
| `rea` | `main` | Unclassified |
| `career-ops` | `main` | Career/productivity tooling; separate from business tenant data |
| `hallmark` | `main` | Unclassified |
| `security-audit-skill` | `main` | Security audit skill; use only with an OS-enforced sandbox |
| `ai-job-search` | `master` | Personal productivity; keep personal data private |
| `claude-video` | `main` | Media/video tooling; inspect before reuse |
| `GDorks` | `main` | Recon/search reference; no autonomous external use |
| `skilltotal` | `main` | Skills collection/catalog; curate and review |
| `VoiceStudio` | `main` | Speech platform; candidate backend with licensing/security gates |
| `Hacking-Tools` | `master` | Security tool collection; no blanket execution permissions |
| `METATRON` | `main` | Agent/security platform; inspect before reuse |
| `invoice-builder` | `main` | Finance workflow candidate; approval and data controls required |
| `call-center-ai` | `main` | Voice/customer-service candidate; customer consent and recording controls |
| `ever-gauzy` | `develop` | ERP/business platform reference; inspect license and scope |
| `trufflehog` | `main` | Secret-scanning tool; candidate for repository hygiene |
| `seed-skill` | `main` | Skills authoring/template reference |

## Focused first-pass findings

### `cybrcops/jarvisAI`

- README describes a React + Vite + Three.js browser interface and Node WebSocket bridge.
- The current bridge invokes `@anthropic-ai/claude-agent-sdk`; provider-neutral FreeLLMAPI integration is explicitly an implementation target, not an existing working feature.
- Existing README describes wake-word/always-listening behavior and broad MCP delegation. That behavior must not be carried into the iPhone PWA without explicit foreground microphone interaction and granular permissions.
- The bridge's broad MCP access, shell/tool execution, browser automation, and inherited account connectors are high-risk surfaces. Require an explicit server-side allowlist, per-tool policy, user/action binding, audit events, and isolated execution.
- Repository LICENSE is MIT. Preserve notices and provenance for copied material.
- Recent default-branch commit evidence returned by GitHub: `2e1c908a` dated 2026-11-13. Julie architecture work exists on `feat/julie-personal-assistant-architecture`; this audit continues on a child feature branch.
- Decision: use as the product shell/runtime starting point, but replace the provider coupling behind a provider interface and redesign authority boundaries before enabling tools.

### `cybrcops/VoiceStudio`

- README presents a local-first desktop/mobile voice studio with voice cloning/design, dubbing, dictation, transcription, and multiple engines.
- Repository LICENSE is AGPL-3.0. Network-accessible modified versions can trigger AGPL source-offer obligations; review with counsel before distributing a modified service.
- App license does not settle model/weight/data licensing. The previously inspected model inventory reports no model record with confirmed commercial clearance; default model weights and supporting audio components have separate terms. No business voice may be selected until exact artifacts and rights are checked.
- Prior inspected docs describe local speech API and MCP interfaces, streaming transcription, and remote API-key protections. Backend defaults are loopback-oriented; remote exposure requires hardened authentication/TLS. Do not expose admin/settings/system endpoints, described in project docs as RCE-class.
- Decision: candidate isolated speech subsystem behind a broker; no direct browser-to-admin access, no master key in the PWA, no install until model/hardware/license review.

### `cybrcops/ruflo`

- README identifies it as an agent meta-harness for Claude Code and Codex; repository LICENSE is MIT.
- MIT license does not make its agents, skills, prompts, shell commands, or MCP servers safe.
- Prior local installation review found risky shell/network/arbitrary tool execution patterns and a worker/MCP port conflict with Twenty. An earlier attempted spawn did not establish that a worker successfully executed.
- Decision: use only after tool-by-tool capability review, constrained workspace, explicit command allowlist, resource limits, network egress policy, and separate unprivileged sandbox. Never make Ruflo the policy authority.

### `cybrcops/freellmapi`

- README describes an OpenAI-compatible router with many provider integrations, a custom OpenAI-compatible endpoint option, model catalog updates, and compatibility with coding agents.
- Repository LICENSE is MIT.
- README also advertises a paid premium/live catalog subscription. Julie must not purchase it or silently depend on it. Treat provider keys, free quotas, provider availability, and remote catalog feeds as external dependencies that can change or fail.
- Decision: candidate LLM gateway. Configure only explicitly approved no-cost providers or local endpoints; block paid providers by default; no silent fallback; show provider, locality, quota/limits, and failure state. Keep all keys server-side and out of logs/browser bundles. Verify signed catalog/update behavior and disable any paid route in configuration.

### `cybrcops/Cybrcom`

- **Corrected status (2026-11-14):** the README still describes a synthetic demo, but main now contains a hybrid pre-production implementation. The Command and Security Feed views connect to a local CYBRCOM server; landing/Overview/Tasks/Review remain synthetic, and Connections still reports integrations disconnected. The README is stale/incomplete relative to the merged code.
- Main merged PR #3 from `feat/command-console-live` on 2026-11-13; latest observed main commit is `b7c86e0b77affb58b09c1e0e959108e0af6ca630`.
- The server provides local SQLite SOC intake/polling, a static roster/status board, Mira chat, a narrow GREEN/YELLOW tool set, approvals, run records, and a work board. It is not yet a full workforce, cross-business control plane, or 24/7 service.
- The current Mira implementation uses the Anthropic SDK, `claude-opus-5-5`, provider default fallbacks, and a default USD 10/day budget. A configured budget of zero also falls through to 10 due the current `Number(value) || 10` expression. This conflicts with the owner's no-paid-API/no-paid-fallback requirement; do not enable this model path for CYBR operations.
- Remote console access is loopback-only when no token is configured; when configured, a shared static Bearer token is used. No per-user identity, MFA/step-up, tenant-aware session, or full action-bound approval hash was found in the inspected path. Do not expose the server publicly.
- The current voice hook is explicit push-to-talk, but Chrome/Edge browser recognition sends audio to the browser vendor's speech service. It is not a local-only speech path. No installable PWA/iPhone validation was performed.
- The repo's `site` branch exists and remains the existing public website/Hostinger source. Keep that branch, Hostinger connection, document root, DNS, and live site untouched. The `site` branch was not reviewed or changed.
- Decision: preserve Cybrcom as the separate command-console/public-site repository; use its policy and local SOC prototype as architecture/source material only. See `docs/CYBR-COMMAND-CONSOLE-AUDIT.md` in the Julie audit branch for the detailed source review. No deployment or live-hosting change is authorized.

### `cybrcops/AI`

- README currently contains only a short collection description. No root LICENSE file found in first pass.
- Decision: inventory contents before treating this as a reusable codebase; no license assumptions.

### `cybrcops/twenty` — excluded

- Explicitly excluded from Julie's selected architecture by user direction on 2026-11-14.
- Do not integrate, install, depend on, copy, or spend further audit effort on this repository unless the user explicitly reverses this decision.

### `cybrcops/cybrcops-platform` — primary platform

- README identifies CYBR MISSION as the primary platform and corporate brand for a unified CRM, ITSM, SOC, cybersecurity, and business-operations ecosystem.
- The platform product map includes Mission Control, Mission CRM, Mission Desk, Mission Ops, Mission SOC, Mission Secure, Mission GRC, Mission Finance, Mission AI, and Mission Connect.
- Decision: this is the main CYBR MISSION platform repository and the primary CRM/SOC integration target for Julie. Review its actual architecture, APIs, schemas, tenant boundaries, and service maturity before implementing adapters. The README alone does not prove that every listed product/service is implemented.

### `cybrcops/CRM` — separate repository, not primary platform

- Its README describes a standalone Mission CRM domain repository with its own data model, APIs, workflows, permissions, and multi-tenant/PostgreSQL/Supabase portability goals.
- It is distinct from `cybrcops-platform` and is not the user's main CRM/SOC platform repository.
- Decision: do not treat this repository as the primary platform or build Julie's CRM/SOC integration around it. Revisit only if the user explicitly selects it for a bounded purpose. Its implementation maturity and license remain unverified.

### Additional high-value references

- `cybrcops/OpenJarvis`: README identifies a local-first personal AI framework, Apache-2.0 (README badge), Python >=3.10, local API/UI, skills, scheduled monitoring, research and evaluation. Strong architecture/evaluation reference for local-first AI; verify actual LICENSE and component licenses before copying. Its code-execution and connectors need the same sandbox/approval controls.
- `cybrcops/JARVIS`: README describes HuggingGPT-style task planning, model selection, expert model execution, and result synthesis; project README is dated around 2023. Treat as historical research, not a current dependency. Its documented default setup expects substantial VRAM/storage; not a fit for the 24 GB RAM NUC without a carefully scoped lite mode.
- `cybrcops/security-audit-skill`: README describes coverage-led security review, independent candidate verification, structured findings, and zero-dependency Node validators. It explicitly requires an OS-enforced sandbox with external networking disabled for target-controlled builds/tests. Good audit-process reference; not permission to run external tests.
- `cybrcops/pentagi`: README describes an autonomous penetration-testing multi-agent system with Docker-isolated tools and multiple LLM backends. Keep separate from Julie's general assistant runtime; only an isolated, explicitly authorized lab with scope controls can use it.
- `cybrcops/zero-breach-toolkit`: README describes read-only local forensic triage and explicitly warns that clean results are not proof of safety. Defensive reference only; do not present its output as definitive detection.
- `cybrcops/claude-skills`: README describes a broad multi-domain skill catalog, agents/personas, and scripts. Curate individual skills after source/license/dependency review; do not bulk-import hundreds of skills or treat skill instructions as trusted policy.
- `cybrcops/chatwoot`: self-hosted customer-support platform candidate; inspect its license, deployment, tenant model, and data handling before integration.
- `cybrcops/call-center-ai`: voice/contact-center candidate; requires explicit recording/consent, retention, customer data, and human handoff review.
- `cybrcops/trufflehog`: secret-scanning candidate for CI/repository hygiene; do not upload repository secrets to external services.

## Integration boundaries and threat controls

1. **Authority:** CYBR COMMAND policy service is the only authority for authorization. Julie, Ruflo, and model responses cannot grant permissions.
2. **Tool execution:** model output is untrusted. Every tool call is independently authorized against actor, tenant, resource, action, scope, risk tier, approval token, and expiry.
3. **Approvals:** approvals are action-specific, single-use, short-lived, and bound to a normalized action payload. Changing the target, amount, recipient, or operation invalidates approval.
4. **Tenant isolation:** every business record, conversation, memory item, tool credential, file, audit event, and background job carries an immutable tenant/workspace scope. Personal Julie scope is a separate principal and cannot be queried by employee/customer tenants.
5. **Memory:** consent and retention are explicit. No credentials, access tokens, raw audio, or secrets. Support review/export/correction/deletion and scoped retrieval.
6. **Voice:** foreground mic capture only on iOS; visible mic state; no background listening assumption. Raw audio is ephemeral by default. Voice clones require documented consent and rights.
7. **Model routing:** explicit allowlist; paid routes disabled; no silent paid fallback; redact secrets; cap tokens/concurrency; report model/provider/locality and quota errors honestly.
8. **VoiceStudio:** expose only required transcription and synthesis endpoints through a broker. Keep admin/system/settings endpoints private. Authenticate remote calls with short-lived, scoped tokens and TLS.
9. **Ruflo/PentAGI/security tools:** no broad shell/network by default. Use isolated disposable workers, non-root identity, read-only mounts where possible, egress deny-by-default, resource/time limits, explicit targets/scope, and immutable logs.
10. **Supply chain:** pin versions/commits, review package manifests/lockfiles, scan for secrets and known vulnerabilities, generate an SBOM, record licenses and provenance, and avoid running untrusted install scripts.
11. **Operations:** global pause, per-agent stop, external-write kill switch, tool kill switch, and emergency revoke path. Fail closed on missing policy/approval/audit service.
12. **Evidence:** record what actually ran, with commit, environment, command, result, and timestamp. Never report a test, scan, install, deployment, or action as successful without verifiable output.

## Build vs. reuse decisions

| Capability | Preferred source | Decision |
|---|---|---|
| Julie UI | `jarvisAI` | Adapt incrementally; remove unsafe always-on assumptions; keep UI independent from model provider |
| Voice STT/TTS | `VoiceStudio` | Reuse via narrow broker only after model license and endpoint review |
| LLM gateway | `freellmapi` | Reuse behind strict provider allowlist; no paid routes/fallback |
| Agent workflows | `ruflo` | Optional bounded worker layer after sandbox and permission audit |
| Command policy/approval | `Cybrcom` architecture | Build/own as first-party control plane; not delegated to an agent |
| Main CRM/SOC platform | `cybrcops-platform` | Primary integration target and source of truth for the unified CYBR MISSION platform |
| Personal local-AI patterns | `OpenJarvis` | Study memory, local inference, evaluation, and scheduled-task patterns |
| Security review workflow | `security-audit-skill` | Adapt evidence/verification methodology only within isolated authorized environment |
| Autonomous security testing | `pentagi`, `Hacking-Tools`, `GDorks`, payload repos | Separate lab only; no general Julie tool exposure |
| Customer support | `chatwoot`, `call-center-ai` | Future integration after tenant/consent/data review |
| Secret hygiene | `trufflehog` | Candidate CI check; validate configuration and false-positive handling |

## Gaps before integrated build/testing

- Provider-neutral model adapter is not yet implemented in `jarvisAI`.
- No completed Julie PWA/voice MVP is verified by this audit.
- No production policy broker or approval service is yet verified.
- No tenant-isolated Julie memory store is yet verified.
- Voice model/weight commercial rights are unresolved.
- Ruflo worker isolation and permissions are unresolved.
- FreeLLMAPI provider allowlist and paid-route blocking must be implemented and tested.
- Repository inventory metadata is incomplete for many projects: actual license, latest commit, dependency lock, CI status, API surface, and security review remain to be checked.
- The current NUC's exact model latency, available disk, GPU/accelerator capability, and concurrent workload capacity must be measured later with user approval and without altering existing services.

## Next audit/build sequence

1. Inspect `cybrcops-platform` architecture, service boundaries, schemas, APIs, auth/tenant model, and CRM/SOC implementation status; treat it as the primary product integration target.
2. Complete per-repository metadata: latest commit/date, license file, package manifests/lockfiles, CI, and default-branch status.
3. Review `jarvisAI` bridge endpoints, tool permission callbacks, auth/session handling, origin checks, filesystem confinement, and dependencies.
4. Review FreeLLMAPI configuration and routing code for paid-provider disablement, fallback semantics, secret storage, and catalog update trust.
5. Review Ruflo tool registry, MCP permissions, worker process isolation, and worker process/port isolation.
6. Review VoiceStudio auth, admin route exposure, MCP path confinement, model inventory, and selected model weights.
7. Define first-party Julie contracts: provider adapter, voice adapter, memory scope, action schema, approval token, audit event, and kill-switch interfaces.
8. Implement the iPhone-friendly PWA shell on a feature branch; run only safe static/build checks available in the remote environment. Do not claim device validation.
9. Open a draft PR for human review; do not merge or deploy.

## Evidence and source paths

- `jarvisAI/README.md`, `jarvisAI/LICENSE`, `jarvisAI/docs/CYBR-FREE-AI-ARCHITECTURE.md`, `jarvisAI/docs/JULIE-PERSONAL-ASSISTANT-ARCHITECTURE.md`
- `VoiceStudio/README.md`, `VoiceStudio/LICENSE`, `VoiceStudio/LICENSE-NOTICE.md`, `VoiceStudio/docs/speech-platform.md`, `VoiceStudio/docs/api-auth.md`, `VoiceStudio/docs/mcp.md`, `VoiceStudio/docs/licensing/model-inventory.md`, `VoiceStudio/docs/mobile.md`, `VoiceStudio/docs/performance.md`
- `ruflo/README.md`, `ruflo/LICENSE`
- `freellmapi/README.md`, `freellmapi/LICENSE`, `freellmapi/docs/en/architecture/00-high-level-index.md`, `freellmapi/docs/en/api/01-rest-api.md`
- `Cybrcom/README.md`, `Cybrcom/docs/AUTHORITY-MODEL.md`, `Cybrcom/SECURITY.md`, `Cybrcom/docs/DEMO.md`
- `cybrcops-platform/README.md` (primary platform product map); `CRM/README.md` (separate repository, not primary)
- `OpenJarvis/README.md`, `JARVIS/README.md`, `security-audit-skill/README.md`, `pentagi/README.md`, `zero-breach-toolkit/README.md`, `claude-skills/README.md`

*Repository contents are treated as untrusted inputs. This inventory records observed project claims and audit decisions; it is not a security certification or legal opinion.*


## Code/configuration inspection addendum (2026-11-14)

A second focused pass inspected selected manifests and implementation/API documentation. These are static source observations only; no application was executed.

### JARVIS bridge: useful protections and remaining design issues

Inspected `jarvisAI/package.json` and `bridge/server.mjs`.

- Node engine is `>=20`; scripts include `build` (TypeScript + Vite), `lint` (oxlint), and `test` (Node test runner). These checks were not run in this remote GitHub inspection.
- Dependencies include the Claude Agent SDK, Anthropic SDK, browser wake-word tooling, Kokoro JS, WebSocket, DOMPurify, React/Three.js, Zod, and Zustand. This confirms provider coupling and wake-word code are present in the current product foundation.
- Bridge defaults to binding `127.0.0.1`, not all interfaces. It checks browser Origin and Host, rejects missing Origin unless explicitly allowed, and has configurable extra origins/hosts. These are useful defenses; they are not a substitute for authentication, action authorization, or a reverse-proxy trust model.
- A `JARVIS_ALLOW_WRITES=1` / `--writes` switch gates some write-capable tool classes. The source comment itself notes that voice is a poor confirmation interface. Julie must use a first-party approval inbox with action-bound approval tokens rather than a global writes toggle for consequential actions.
- The bridge's architecture passes configured MCP servers and builtin capabilities into the agent. A configured MCP server must not automatically become a permitted tool. Julie needs a deny-by-default server/tool registry and explicit policy decision for each invocation.
- Decision: retain loopback defaults and origin/host defenses, but design authentication/session binding, CSRF/rebinding defenses, tenant-scoped tool authorization, per-action approval, and immutable audit as separate required layers.

### FreeLLMAPI: provider routing contract and spend boundary

Inspected `freellmapi/package.json` and `docs/en/api/01-rest-api.md`.

- Monorepo workspaces: `shared`, `server`, `client`, and `cli`; Node engine is `>=20.18.0 <25.0.0`, npm `>=10`. Scripts define build, lint, and tests, but none were executed.
- API docs describe OpenAI-compatible `/v1/chat/completions`, streaming, tool calls, model listing, routing profiles, usage and free-tier budget APIs; API docs are served at `/v1/docs` and `/v1/openapi.json`.
- The `auto` route uses an active fallback chain; named profiles can use their own chains. This is powerful but means the app must explicitly constrain the enabled provider/key pool and never assume `auto` means local-only or free without configuration and runtime verification.
- Docs state an unknown profile name returns 400 rather than silently falling back. Julie should use a dedicated named profile and explicit model IDs; still verify the actual route response header and usage record for every request.
- Decision: do not expose FreeLLMAPI's unified key to the browser. Use a backend adapter with explicit provider allowlist, spend caps, model policy, timeout, rate limiting, content/data classification, and a hard deny on paid providers. Treat catalog updates as untrusted supply-chain input until signature and update behavior are independently reviewed.

### Ruflo: orchestration dependency footprint

Inspected `ruflo/package.json`.

- The repository package reports version `3.77.0`, MIT license, Node `>=20`, and workspace packages for Codex, MCP, agent federation, and security.
- The dependency surface includes CLI/MCP, security, neural/vector components, Express, WebSocket, file helpers, SQLite, and optional native/vector packages. A large optional-dependency surface increases supply-chain and runtime complexity.
- Decision: first isolate the smallest needed subset; do not install/enable the whole meta-harness as Julie's core runtime. Verify exact CLI/MCP command allowlists, tool schemas, child-process behavior, file roots, network egress, and worker lifecycle before enabling any execution.

### VoiceStudio: remote API/MCP safety boundary

Inspected `VoiceStudio/docs/api-auth.md` and `VoiceStudio/docs/mcp.md`.

- Docs state the default backend is loopback-only and unauthenticated. Remote consumption routes require deliberate API-key configuration; the share PIN covers HTTP only and does not gate WebSockets.
- API docs distinguish consumption routes from administrative `/system/*` and `/api/settings/*` routes, described as RCE-class. Do not expose admin paths through Julie's broker or reverse proxy.
- Remote browser clients should not receive the master API key. Use server-side key storage and a short-lived scoped session/token exchange over TLS.
- MCP tools include speech generation, voice cloning/design, transcription, voice/language listing, and health. The MCP file path boundary is `OMNIVOICE_MCP_BASE_PATH`; path inputs are refused when no base path is configured, and docs describe no-follow descriptor/path confinement.
- MCP `files` output mode avoids placing full audio payloads in model context. Use only a dedicated scratch directory with strict quotas/retention, and disable clone/design tools unless there is explicit consent and rights.
- Decision: the Julie adapter should expose only transcription and synthesis initially, with fixed approved model/voice IDs and strict input/output size limits. Do not expose clone/design or administrative MCP tools to general agents.

### Open items

- Fetch and inspect actual source/route files for FreeLLMAPI provider filtering, fallback implementation, credential encryption, catalog signature verification, and authentication.
- Inspect Ruflo MCP/tool implementation and sandbox configuration rather than relying on package metadata.
- Inspect VoiceStudio production-private deployment pattern and exact selected model card/weight files before any commercial use.
- Review jarvisAI tests and source files in a suitable isolated environment before claiming build/security status.


## 2026-11-14 — Current source and freshness update

This refresh records current branch/commit evidence and supersedes older repository descriptions where noted. It is still a focused static audit, not a complete audit of all discovered repositories.

| Repository | Latest observed main commit | Current evidence/status |
|---|---|---|
| `cybrcops-platform` | `dc45caf5` — 2026-11-04 | Primary CYBR MISSION source of truth. Partial live CRM/Tickets UI and closed-list read API observed. AAL2/JWT/membership/role/permission server gate partially inspected. Direct SQL/RLS and full route inventory remain open. |
| `Cybrcom` | `b7c86e0b` — 2026-11-13 | Hybrid pre-production local command/SOC prototype; README status is stale. Direct Anthropic model/default fallback and static shared-token access block production use under current owner policy. `site` branch exists and was not touched. |
| `jarvisAI` | `2e1c908a` — 2026-11-13 | Julie foundation candidate; bridge still Anthropic Agent SDK-coupled. Provider-neutral FreeLLMAPI integration is not implemented. Audit work remains on `feat/julie-repository-audit`. |
| `VoiceStudio` | `dd0f6bde` — 2026-11-14 | Latest observed main merge is release `v0.5.9-beta.2`. App is AGPL-3.0; exact speech model/weight rights remain unresolved. No Julie integration. |
| `ruflo` | `002d8c1a` — 2026-11-12 | Latest observed release is 3.77.0. Recent optional neural/FANN and judge-routing changes do not resolve worker execution, MCP permission, sandbox, egress, or resource-limit review. Keep workers disabled. |
| `freellmapi` | `e03d74d9` — 2026-11-12 | Catalog update reports 87 providers and 1,313 endpoints. This is not evidence that routes are free or stable. Use explicit model/provider allowlist and disable all paid routes/fallbacks. |
| `OpenJarvis` | `afdbd5b3` — 2026-11-13 | Latest observed commit is clone-traffic metadata; no new runtime/security conclusion. Continue code/license review before reuse. |
| `AI` | `c04dfbd8` — 2026-10-06 | Only initial commit observed; README remains a short collection description. No reusable component is approved. |

### Platform authorization source findings

The detailed update in `docs/PLATFORM-INITIAL-AUDIT.md` supersedes the earlier statement that platform authentication/authorization had not yet been inspected. The inspected server gate verifies the caller JWT, AAL2, one active membership, active role, explicit role permission, active permission, and `has_permission`; it uses the caller JWT and publishable/anon key rather than a service-role key. The `/api/mission` and `/api/crm` routes enforce organization filtering and closed resource/input allowlists. This is partial source evidence only: SQL/RLS policies and live deployment state remain unverified, and the in-memory per-process rate limiter/proxy-header assumption need review.

### Command-console source findings

The detailed `docs/CYBR-COMMAND-CONSOLE-AUDIT.md` supersedes the earlier description of Cybrcom as only a static synthetic frontend. It records the current mixed demo/live behavior, local-only server defaults, limited tool surface, direct Anthropic/fallback conflict, static-token remote-access gap, approval/audit limitations, static roster, browser speech privacy caveat, and absence of a verified Julie/cross-business adapter or installable PWA.

No NUC, Hostinger, Cloudflare, DNS, `site` branch, live website, production service, customer data, or secret was touched in this refresh.

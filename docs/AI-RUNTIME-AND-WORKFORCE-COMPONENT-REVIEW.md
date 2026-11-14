# AI runtime and workforce component review

**Review date:** 2026-11-14  
**Scope:** shared CYBR repositories FreeLLMAPI, Ruflo, VoiceStudio and OpenJarvis. Static GitHub source inspection only. No install, local runtime, NUC access, network probe, build or tests.

## Repository provenance and upstream drift

| Fork | License | Fork main commit | Upstream main commit | Fork status |
|---|---|---|---|---|
| `cybrcops/freellmapi` | MIT | `e03d74d9581f2f371306e6c269036b98c4d78083` | `730ba7942a7acbfe2d30b20f6f9b6efcd8b5ae04` | 8 commits behind `tashfeenahmed/freellmapi` |
| `cybrcops/ruflo` | MIT | `002d8c1ac031871fc14220e964f4dbb35b60428f` | `3a1479883b55b7f786ef0bf3141478e4dfb9444f` | 3 commits behind `ruvnet/ruflo`; fork package reports 3.77.0, upstream main release commit reports 3.77.1 |
| `cybrcops/VoiceStudio` | AGPL-3.0 | `dd0f6bdef22755613136c5940682a79bf16d175d` | `eb5c4bfaed0d919db2ae3c61c851195a4e8624fb` | 35 commits behind `debpalash/VoiceStudio` |
| `cybrcops/OpenJarvis` | Apache-2.0 | `afdbd5b35cb21f51564588c8cdbf6b74889c4f1f` | `76836f4eba5e0d25cf53b56c2f475a2c2583c1c9` | 1 commit behind `open-jarvis/OpenJarvis` |

All four are GitHub forks (parent/source metadata verified). No upstream sync, merge, rebase or branch update was performed. Fork contents and upstream were treated separately. Dates and commit SHAs are point-in-time observations; re-check before reuse.

## FreeLLMAPI — candidate inference gateway, not yet approved for governed use

**Fork:** `cybrcops/freellmapi`, main, MIT, observed commit `e03d74d9581f2f371306e6c269036b98c4d78083`; upstream eight commits ahead. Repo metadata describes the project as personal experimentation only.

### Verified architecture
- Node/TypeScript monorepo; server package `@freellmapi/server` 0.2.1, Node `>=20.18.0 <25`, npm `>=10`; Express 5, SQLite (`better-sqlite3` optional), Zod, Helmet, CORS, Undici, Vitest.
- OpenAI-compatible `/v1` proxy, plus Anthropic-compatible, Gemini, MCP and Ollama surfaces; dashboard admin APIs are separately session-gated.
- Provider adapters, routing service, rate-limit/quota ledger, cooldowns, key health probes, model catalog sync, backup scheduler, encrypted SQLite key storage, log redaction and provider error sanitization exist in source.
- Router architecture supports automatic candidate selection and fallback across up to 20 attempts (per architecture docs); the provider catalog includes many external services and custom OpenAI-compatible endpoints. The free-tier label does not itself guarantee every configured endpoint is free, authorized for business use, available, or privacy-compatible.
- Unified `/v1` key gates inference; dashboard session gates admin routes. Security policy explicitly describes a single-user trusted-network tool, not a multi-tenant service.
- Docker Compose publishes to `127.0.0.1` by default through `HOST_BIND`, but permits an opt-in LAN bind. Docker autoupdate profile mounts the Docker socket and is root-equivalent on host; keep it disabled.
- **Binding discrepancy:** `server/src/lib/config.ts` defaults process `HOST` to `::`; `server/src/index.ts` binds to that host and falls back to `0.0.0.0` if IPv6 is unavailable. Docker Compose's published host port is loopback-only by default, but a direct Node/source run is not loopback-only by default. This conflicts with the security policy's loopback-default statement. Do not expose the server; before any future local run, explicitly set and verify `HOST=127.0.0.1` and inspect actual socket binding.
- `/api/ping` and `/livez`/`/readyz` status routes are unauthenticated liveness/readiness signals; the main admin APIs are session protected; inference routes use the unified API key.
- Environment supports model catalog sync, custom endpoints, proxy, key management and multiple provider credentials. No reviewed owner-controlled hard allowlist for a Julie-specific model/provider set was verified in this source pass.

### Julie decision
- Keep as a possible local inference gateway behind a separately governed adapter; not the CYBR COMMAND policy plane and not multi-tenant.
- Before any use, implement and test a strict provider+model allowlist (deny by default), disable auto model selection and cross-provider fallback for Julie, ensure no paid provider key/endpoint can be selected, reject unknown/custom endpoints, enforce per-request spend ceiling of exactly zero and fail closed on any cost/plan ambiguity.
- Use one explicitly approved provider/model per profile. No model may silently switch providers after rate-limit, 5xx, outage, model-not-found or quota exhaustion. Report unavailability instead.
- Bind loopback-only and verify at runtime; keep Docker Watchtower/autoupdate off; keep provider keys server-side; verify provider retention/training/region/ToS and data classification before sending any CYBR/customer data.
- Never use FreeLLMAPI's broad auto-router as the sole enforcement of the no-paid-fallback policy. CYBR COMMAND must enforce policy outside the model gateway too.

## Ruflo — orchestration research only; worker/MCP execution remains disabled

**Fork:** `cybrcops/ruflo`, main, MIT, package version 3.77.0, observed commit `002d8c1ac031871fc14220e964f4dbb35b60428f`; upstream is 3 commits ahead and reports 3.77.1.

### Verified architecture and concerns
- The `ruflo` repository includes a nested MCP bridge implementation under `ruflo/src/mcp-bridge/index.js`; package scripts expose a bridge start command.
- MCP bridge defaults to `127.0.0.1`; non-loopback binding refuses startup unless `MCP_AUTH_TOKEN` is set. Bearer token comparison is timing-safe. A local-browser host/origin guard is present when no token is configured.
- `MCP_GROUP_AGENTS`, `MCP_GROUP_MEMORY`, `MCP_GROUP_DEVTOOLS`, and `MCP_GROUP_INTELLIGENCE` default enabled unless explicitly set to false. Browser, security, neural and external agentic-flow groups are opt-in in the reviewed env example.
- Bridge spawns separate stdio MCP backends using `npx` (including Ruflo and Ruvector). Child environment inherits process environment except `MCP_AUTH_TOKEN`; provider credentials therefore remain available to spawned backends.
- Terminal execution is explicitly denied unless `MCP_ENABLE_TERMINAL=true`, with the gate applied at the server-side executor. Keep this unset/false.
- Bridge health/group endpoints expose tool inventory. MCP routes support tool listing and execution; enabled groups materially increase available agent, memory, developer and orchestration tools.
- No verified per-tool filesystem sandbox, network egress policy, CPU/memory limits, tenant-bound MCP identity, or production approval integration was established in this static review. Localhost binding is not a sandbox.

### Julie decision
- Ruflo may be evaluated as an orchestration backend only after a separate isolated test plan.
- For CYBR command integration, start with all agent, memory, devtools, intelligence and optional groups disabled; enable one read-only tool group at a time only after code-level permission mapping.
- No workers, shell/terminal, browser, file writes, GitHub mutations, external network, or swarm execution until command execution policy, tool-by-tool MCP allowlist, process sandbox, filesystem roots, network egress, secrets inheritance, resource quotas, identity binding, audit and stop/kill semantics are verified.
- Do not install, update, reconfigure or start anything on the NUC as part of this review.

## VoiceStudio — voice UX research only; commercial model clearance not established

**Fork:** `cybrcops/VoiceStudio`, main, AGPL-3.0, observed commit `dd0f6bdef22755613136c5940682a79bf16d175d`; upstream is 35 commits ahead. Repository metadata reports upstream parent `debpalash/VoiceStudio`.

### Verified
- Application license is AGPL-3.0. Internal/commercial use of the application itself is allowed under AGPL terms; modifications made available over a network carry corresponding-source obligations. A separate commercial license is for closed-source embedding/proprietary product use.
- Model weights/tokenizers/auxiliary assets are separate from application license. Model inventory lists 63 configured model repositories plus seven unresolved asset families.
- Model review workflow explicitly says no production model is currently review-ready; no inventory record currently asserts `commercial_use: true`. Some model weights have known non-commercial restrictions; other models remain unreviewed.
- MCP server exposes synthesis, voice cloning, transcription, voice listing and health tools. It supports stdio and SSE; audio file path input/output uses a configured base directory as the file boundary. Inline base64 output can put audio in agent context unless files mode is used.
- No conclusion about safe business use can be drawn from AGPL application license alone.

### Julie decision
- Do not use VoiceStudio models/weights for CYBR business or customer-facing voice until exact model, tokenizer, auxiliary assets, output rights, voice consent and commercial terms are individually cleared.
- Do not expose its admin or high-impact endpoints; review all MCP/API auth and path constraints before any adapter. Prefer stdio/local integration, strict path allowlist, files output mode, no raw audio in prompts/logs, and explicit user consent.
- For initial Julie PWA, keep foreground push-to-talk and playback as the product requirement; voice engine selection remains provider-neutral and pending legal/security review.

## OpenJarvis — local-first agent framework research, not yet an execution runtime

**Fork:** `cybrcops/OpenJarvis`, main, Apache-2.0, observed commit `afdbd5b35cb21f51564588c8cdbf6b74889c4f1f`; upstream is one commit ahead.

### Verified
- Python package supports Python `>=3.10,<3.14`, with optional inference, server, desktop, voice, memory, cloud and tool extras. Core dependencies include OpenAI-compatible client, httpx, datasets, DDGS, PostHog, psutil and Telegram bot.
- Architecture is local-first and composable: agents, engines, tools, MCP, memory, scheduler, channels, connectors, security and traces.
- Tool registry imports numerous capabilities including file read/write, shell execution, Docker shell execution, Git, database query, HTTP request, browser, audio, calendar, memory management and code interpreter. Availability of a tool is not proof that it is safely sandboxed for our tenancy model.
- A Docker `ContainerRunner` sandbox is present. In the reviewed runner, containers use `--network none`, read-only bind mounts after allowlist validation, a timeout default of 300 seconds and max concurrency default of five. This is a useful containment building block, not complete production isolation; container runtime configuration, privileges, root filesystem, resource limits, image provenance, seccomp/AppArmor, host socket exposure and mount allowlist must be reviewed before use.
- `TaskScheduler` supports persisted once/interval/cron scheduling; do not equate this with reliable always-on monitoring or tested failover.

### Julie decision
- Candidate research source for local inference abstractions, eventing, scheduler, traces, sandbox patterns and evaluation; Apache-2.0 is compatible in principle, subject to exact dependency/source provenance review.
- Reuse no code yet. Evaluate individual components and dependencies; exclude shell, DB, HTTP, browser, write and connector tools from any initial Julie tool registry.
- No customer data, personal memory or company scope should enter shared OpenJarvis memory without an explicit data-domain implementation and tests.

## Cross-repository implementation policy

1. Repositories and licenses do not confer runtime permission. No code reuse until exact file, upstream provenance, license, version, dependency closure and required notices are reviewed.
2. All provider calls go through a CYBR COMMAND model policy that enforces explicit allowlists, zero paid fallback, data classification, redaction, timeout and audit metadata.
3. All agent tools go through a separate tool policy with closed schemas, tenant scope, identity binding, risk classification, approval requirement and kill switch.
4. Start with a non-executing read-only agent profile; agent orchestration and execution remain disabled until sandbox and permission proof exists.
5. No repo update/sync, dependency install, runtime launch, NUC change, hosting/DNS/live site change, or production mutation occurred in this review.

## Sources inspected

- FreeLLMAPI: `README.md`, `package.json`, `.env.example`, `SECURITY.md`, `server/src/index.ts`, `app.ts`, `env.ts`, `lib/config.ts`, `providers/index.ts`, `services/router.ts`, `routes/proxy.ts`, `middleware/requireAuth.ts`, `routes/auth.ts`, `docker-compose.yml`, `docs/en/architecture/00-high-level-index.md`.
- Ruflo: `README.md`, `package.json`, `SECURITY.md`, `ruflo/.env.example`, `ruflo/server.json`, `ruflo/src/mcp-bridge/index.js`.
- VoiceStudio: `README.md`, `LICENSE-NOTICE.md`, `docs/licensing/model-review-workflow.md`, `docs/licensing/model-inventory.md`, `backend/config/model_licenses.json`, `backend/main.py`, `backend/mcp_server.py`.
- OpenJarvis: `README.md`, `pyproject.toml`, `REVIEW.md`, `src/openjarvis/sandbox/__init__.py`, `sandbox/runner.py`, `src/openjarvis/tools/__init__.py`, `src/openjarvis/scheduler/__init__.py`.

## Validation and operational impact

- Static GitHub API/source reads and fork-vs-upstream commit comparisons only.
- No local clone, package install, build, test, runtime, model call, network scan, NUC access, or iPhone test.
- No secrets, private data, provider keys, raw audio or customer records retrieved.
- No changes to any of these source repositories; this review is documented only in `cybrcops/jarvisAI` review branch.
- No NUC, Hostinger, Cloudflare, DNS, live website, network, production service or production database configuration changed.

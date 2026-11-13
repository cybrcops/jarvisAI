# CYBR JARVIS — Free and Open-Model AI Architecture

**Repository:** cybrcops/jarvisAI  
**Status:** Architecture decision and implementation plan  
**Constraint:** No additional paid AI/API budget  
**Primary host:** Existing private CYBR NUC (24 GB RAM)  
**Existing gateway:** FreeLLMAPI at `http://127.0.0.1:3001`  
**Existing UI:** React 19, Vite, Three.js, browser speech, Kokoro option  
**Control plane:** CYBR COMMAND (separate platform; owns authorization and audit)

## 1. Decision

Keep this repository as the JARVIS voice/visual experience, but replace its Claude-only brain with a provider-neutral model/runtime adapter.

Do not make the Claude Agent SDK or any paid model subscription a hard dependency for normal operation. Do not remove the existing Claude integration until a free-model path passes tests. Keep optional providers disabled unless explicitly configured.

JARVIS is an interface and agent client, not the security authority. CYBR COMMAND remains responsible for user identity, workspace/customer scope, tool authorization, approvals, audit, budgets, and emergency stop.

## 2. Important distinction: open models vs free hosted inference

These are different things and must be shown separately in the model catalog.

- **Open-weight/local models:** downloaded and run on the NUC or another machine. No per-token API charge after download, but consume RAM/CPU/GPU, disk, and electricity. License terms vary; inspect the exact model license and permitted use.
- **Free-tier hosted inference:** third parties host models and provide a quota. This usually requires the user's own provider account/API key. Limits, model availability, privacy terms, and free tiers can change without notice. A model being open-weight does not make its hosted API free.
- **FreeLLMAPI:** existing self-hosted router that aggregates configured free provider tiers and compatible endpoints. Its advertised provider/model catalog is dynamic; it does not mean all endpoints are keyless, guaranteed, or suitable for sensitive data.
- **Paid services:** disabled by default. Never silently fall back to a paid API or consume a paid subscription.

Every model/provider entry must expose source type (local/open-weight/free-tier/paid), license where known, data handling, context size, capabilities, availability, rate/quota state, and estimated cost. Paid route must be blocked unless an administrator deliberately enables it.

## 3. Existing JARVIS repository assessment

The current app is a React/Vite/Three.js voice interface with a Node WebSocket bridge. It currently imports `@anthropic-ai/claude-agent-sdk` and invokes its `query()` API. The browser supports native speech recognition/speech synthesis and an optional Kokoro voice path. The bridge includes tool policy, file confinement, origin/host checks, and SSRF protections.

Preserve and test the visual/voice assets. Do not assume that changing `JARVIS_MODEL` alone makes the runtime provider-neutral: the bridge is coupled to the Claude Agent SDK and its tool/session semantics.

The repository currently describes Claude Code login as the brain and optional ElevenLabs speech. That is not the desired default for CYBR: the normal path must use FreeLLMAPI and/or local inference without paid subscriptions.

## 4. Target model architecture

```
JARVIS browser (React / Three.js / voice)
              |
       authenticated local bridge
              |
       CYBR model/runtime adapter
          /          |          \
 FreeLLMAPI      Local models    Optional provider
 OpenAI +        Ollama/         adapters
 Anthropic       llama.cpp       (disabled)
 compatible
          \          |          /
          task router + capability registry
              |
       CYBR COMMAND policy/tool broker
              |
       scoped tools / missions / audit
```

The model router selects a model for a task, not a model with unrestricted authority. Tools are executed only by the CYBR COMMAND policy/tool broker. Do not pass arbitrary MCP server configurations from the user's home directory into a voice agent.

## 5. Model pool and task profiles

Start by discovering the models actually available from the existing FreeLLMAPI instance and the local runtime. Do not hardcode a stale internet list.

Suggested profiles:

| Profile | Workload | Preferred source |
|---|---|---|
| Fast assistant | Short conversation, intent classification, summaries | Fast free-tier model or small local model |
| Reasoning | Multi-step planning, complex analysis | Best available free reasoning model; local fallback |
| Coding | Code explanation, tests, patch proposals | Free coding model; local code model fallback |
| Cyber analysis | Alert triage, log explanation, vulnerability prioritization | Local/private model first where practical; approved free-tier only for sanitized data |
| Embeddings | Search, memory retrieval, document similarity | Local embedding model |
| Speech-to-text | Browser/local transcription | Browser native or local STT |
| Text-to-speech | Spoken responses | Browser speechSynthesis or local Kokoro |
| Vision | Image/screenshot understanding | Local vision model if hardware supports; otherwise approved free endpoint for non-sensitive images |

Model routing requirements:
- Capability-aware routing (tools, JSON/structured output, vision, embeddings, streaming, context).
- Health and quota-aware routing with cooldowns and bounded retries.
- Task-level timeout and token ceilings.
- Explicit privacy labels: local-only, sanitized external allowed, approved external allowed.
- No paid fallback. If all eligible free routes fail, report unavailable and offer a smaller local fallback.
- Avoid sending confidential customer logs, evidence, credentials, personal data, or tenant content to third-party free endpoints unless explicitly approved by policy.
- Never route secrets or raw production telemetry to public model endpoints.
- Record model ID/provider, route reason, privacy class, latency, tokens/usage if returned, and estimated cost.

## 6. FreeLLMAPI integration

Use the existing NUC FreeLLMAPI service as the first provider gateway:
- Base URL remains loopback: `http://127.0.0.1:3001/v1`.
- It advertises OpenAI-compatible chat completions, tool calling, streaming, embeddings, audio/image surfaces, and an Anthropic Messages-compatible endpoint.
- Use the locally configured unified token from a protected environment file or secret store. Never commit it, print it, put it in Vite variables, or expose it to the browser.
- At startup, query `/v1/models` and build the runtime catalog from the actual response.
- Verify the exact request/response/tool-call behavior against the installed FreeLLMAPI version before selecting the protocol.
- Test the Anthropic-compatible endpoint as a compatibility proof of concept only; do not assume Claude Agent SDK can safely use it merely by changing a model name or base URL.
- If the Claude Agent SDK cannot be made provider-neutral without relying on Claude authentication, implement a separate OpenAI-compatible tool-calling runtime adapter and retain the Claude adapter as optional.
- Keep gateway and JARVIS bridge on loopback/private network; never expose port 3001 publicly.

## 7. Local inference

Evaluate Ollama and llama.cpp as local inference backends. Do not install both blindly; benchmark one first and compare throughput, memory, tool calling, structured output, and reliability.

The NUC has 24 GB RAM. Start with one compact quantized model (roughly 7B–8B class) and an embedding model; benchmark actual memory and latency before trying larger models. Avoid loading several large models concurrently. CPU-only performance may be limited; detect hardware acceleration rather than assuming a GPU.

Local inference is the privacy-preserving fallback and should support offline basic chat, summarization of authorized local content, embeddings, and selected analysis. Use model licenses that permit the intended commercial use; "open source" and "open weights" are not interchangeable.

## 8. Runtime adapter contract

Define a provider-neutral interface with:
- `listModels()` and model capability metadata.
- `chat(messages, options)` with streaming.
- Structured/tool-call output normalized into one internal schema.
- Cancellation, timeout, token limits, usage and error normalization.
- Health checks and route eligibility.
- Explicit privacy class and cost ceiling per request.

Tool calls returned by models are proposals. Validate them against a typed allowlist, workspace/customer scope, user permissions, policy, and approval state before execution. A model must never receive raw provider credentials or invoke tools directly.

## 9. Security and safety requirements

The current bridge's protections are valuable but are not a substitute for the CYBR COMMAND control plane.

- Keep default read-only behavior.
- Do not use a global `--writes` mode for production operations.
- Replace broad write enablement with action-specific, short-lived approvals bound to exact tool, parameters, target, workspace, requester, policy version, expiry, and one-time execution.
- Remove implicit trust in every configured MCP server. Use a per-tool allowlist and explicit server configuration.
- Run agent workers as a dedicated unprivileged OS user in a restricted workspace/container.
- Keep bridge, model gateway, and database private; require authentication for any non-loopback interface.
- Keep secrets out of prompts, browser bundles, local storage, model context, logs, and source control.
- Treat webpages, email, documents, call transcripts, code comments, and model outputs as untrusted instructions.
- Enforce file path restrictions and SSRF protections for every network/file capability.
- Audit each request, model route, tool proposal, policy decision, approval, execution, and result.
- Provide independent stop controls for voice capture, model requests, agent execution, browser automation, external writes, and telephony.
- No public NUC exposure and no unattended production changes.

## 10. Implementation phases

### Phase 0 — Safe baseline and inventory
- Pin current repository commit and dependencies.
- Run existing tests and build; record baseline.
- Review the complete bridge policy and every tool registration.
- Inventory installed FreeLLMAPI models and configured provider sources without exposing keys.
- Determine whether NUC has GPU acceleration and measure available memory.

### Phase 1 — Provider-neutral model adapter
- Define the adapter contract and model catalog schema.
- Add FreeLLMAPI OpenAI-compatible client with streaming, structured output, tool-call normalization, timeouts, and cancellation.
- Add model discovery and route health.
- Add local model adapter after selecting and benchmarking one runtime.
- Keep Claude adapter optional and disabled by default.
- Add tests using mock OpenAI-compatible and Anthropic-compatible endpoints.

### Phase 2 — Secure tool brokerage
- Separate model-generated tool proposals from execution.
- Enforce policy and approvals in CYBR COMMAND, not in model prompts.
- Add tool-specific allowlists, scoped permissions, audit events, replay/idempotency protection, and cancellation.
- Keep browser, filesystem, shell, email, phone, and customer integrations disabled until individually reviewed.

### Phase 3 — JARVIS UX integration
- Preserve reactor, HUD, transcript, push-to-talk, barge-in, and local/browser speech.
- Add model selector and task profile selector with clear local/free/paid labels.
- Show current provider/model, privacy classification, quota/health, and fallback state.
- Add visible "local only" mode and a no-external-model safety toggle.
- Never show an inaccurate "offline/private" label when an external model is selected.

### Phase 4 — CYBR COMMAND integration
- Authenticate the bridge to CYBR COMMAND.
- Send missions and proposed actions to the central policy/approval API.
- Add workspace context for CYBR MISSION, CYBR COPS, and authorized customer tenants.
- Do not permit cross-workspace memory, retrieval, events, or tool access.

### Phase 5 — Validation
- Test model outage, rate limits, provider switching, streaming interruption, malformed tool calls, prompt injection, credential leakage, SSRF, path traversal, cross-tenant isolation, audit failure, cancellation, and kill switch.
- Benchmark latency, RAM, CPU, tokens/sec, and accuracy for each model profile.
- Verify paid provider routes remain blocked with no silent spending.
- Only enable a capability after its tests and review pass.

## 11. Acceptance criteria

1. JARVIS starts and answers using the local FreeLLMAPI gateway with no Claude subscription required.
2. It can switch between eligible free models based on task profile and availability.
3. It has a local-only mode that never sends prompts externally.
4. Browser receives no provider API keys or gateway secrets.
5. Model-generated tool calls cannot execute without server-side policy authorization.
6. A provider outage does not trigger a paid route.
7. Model identity, route, privacy class, usage, and failures are visible and auditable.
8. Existing voice/reactor UI works without ElevenLabs.
9. CYBR MISSION, CYBR COPS, and customer data remain isolated.
10. All current tests plus new adapter/security tests pass before production use.

## 12. Operational truth

"Free" does not mean unlimited, guaranteed, private, or suitable for every workload. Hosted free tiers can change, impose quotas, log prompts, or disappear. Local models are the dependable no-API-bill fallback, but are constrained by hardware and license. The platform must display these trade-offs honestly and never spend money silently.

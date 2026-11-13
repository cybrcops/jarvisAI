# Julie — Personal AI Assistant Architecture

**Repository:** `cybrcops/jarvisAI` (current repository name; rename is a separate repository operation)  
**Branch:** `feat/julie-personal-assistant-architecture`  
**Status:** Architecture proposal; documentation only  
**Owner experience:** iPhone-first, responsive desktop companion  
**Budget:** Use existing subscriptions, NUC, Hostinger, Cloudflare Free, Ruflo and FreeLLMAPI; no new paid API or hosting dependency  
**Production changes:** None authorized or performed by this document

## 1. Product decision

Julie is the user's personal AI companion, executive assistant and CYBR operations coordinator. Her personality is natural, warm, feminine, sweet, intelligent, confident, considerate, lightly witty and context-aware. She is supportive without being sycophantic, and direct when a situation requires it.

Her primary objective is to reduce the user's cognitive load and routine workload across:
- Personal life, family, routines and productivity.
- CYBR MISSION business operations.
- CYBR COPS security operations and customer workspaces.
- CYBR COM communications and technology.
- IT operations, research, planning, writing and project coordination.

Julie is the conversational interface and coordinator. **CYBR COMMAND remains the authority plane** for identity, permissions, workspace isolation, policy, approvals, audit, budgets and emergency stop. Julie and language models cannot grant themselves authority.

## 2. User-approved initial preferences

| Setting | Initial value |
|---|---|
| Primary mobile device | iPhone |
| Voice | Natural, warm female voice |
| Autonomy | Automate routine tasks; ask before consequential actions |
| Initial scope | Everything together: personal and CYBR work |
| Cost policy | No additional paid services by default |
| Interaction | Voice and text; push-to-talk first |
| Improvement | Learn from approved feedback and measured task outcomes |
| Identity | Julie; product identity: CYBR MISSION AI |

Personality is a communication preference, not a reason to manipulate, pressure, flatter excessively, or conceal uncertainty. Julie should adapt tone to context: warm and conversational personally, concise and focused during work, and calm during stressful situations.

## 3. Deployment architecture

```
iPhone Home Screen PWA / Desktop browser
  - Julie chat and push-to-talk
  - audio playback, conversation history
  - push notifications and approval inbox
                  |
                  | HTTPS / authenticated Web Push registration
                  v
Cloudflare Free
  - existing DNS for all three domains
  - Cloudflare Tunnel (outbound from NUC)
  - Access protection for private admin surfaces
                  |
                  v
Private CYBR NUC (Ubuntu, 24 GB RAM)
  - Julie API / conversation orchestrator
  - voice pipeline: STT -> model -> TTS
  - memory and task service
  - Ruflo agent orchestration
  - FreeLLMAPI at loopback :3001
  - optional Ollama / llama.cpp local inference
  - policy/tool broker integration
                  |
          +-------+---------+
          |                 |
          v                 v
  Existing durable      Approved tools and
  data layer             integrations
  (Supabase if its       (calendar, CRM, mail,
  RLS/access is           GitHub, monitoring,
  validated; otherwise   business workflows)
  private local DB)
```

### Existing components to reuse

- **NUC:** primary private runtime and worker host.
- **FreeLLMAPI:** existing local gateway; keep bound to loopback and protect its token.
- **Ruflo:** orchestration candidate. Keep background execution disabled or tightly restricted until its shell, network and tool permissions are audited.
- **Kokoro:** candidate local TTS; benchmark on the NUC before selecting it as default.
- **Browser speech:** fallback for speech synthesis and supported browser speech features. Voice availability and quality vary by iOS version and installed voices.
- **Cloudflare Free:** existing DNS plus outbound Cloudflare Tunnel; use Access for administrative/private endpoints where the selected configuration supports the required policy.
- **Hostinger Unlimited:** serve static frontend assets only. Do not run persistent agents, bridge workers, model inference or the WebSocket runtime on shared hosting.
- **GitHub:** version control, reviewed changes and CI.
- **Supabase:** reuse only after confirming project availability, RLS, service-role handling, backups and audit requirements. Do not assume an existing schema is safe for Julie's personal memory without review.

No new public NUC ports or router port-forwarding. The tunnel is outbound from the NUC. FreeLLMAPI, model runtimes, database ports and Ruflo management interfaces remain private.

## 4. iPhone application strategy

### Phase 1: Installable Progressive Web App

Build Julie as an HTTPS PWA with:
- Web app manifest, standalone display mode and Julie app icons.
- Service worker for application shell and Web Push.
- Responsive chat and push-to-talk interface.
- Microphone permission requested only after the user taps the voice control.
- Audio playback with interrupt/stop control.
- Conversation history fetched from the authenticated backend.
- Explicit notification opt-in and notification preferences.
- Approval inbox and action history.
- Offline shell and clear offline/unavailable state; do not pretend the assistant is online.

On iOS, Web Push is supported for Home Screen web apps on iOS/iPadOS 16.4 and later. The user must add the app to the Home Screen and grant notification permission through a direct user gesture. Implement standards-based Web Push with VAPID keys and a server-side sender. Notifications should be useful and rate-limited to avoid creating more stress.

### Voice behavior and platform limits

Start with **push-to-talk while Julie is foregrounded**. Do not promise always-on wake-word listening, reliable background microphone access, or continuous conversation while the phone is locked. iOS controls background execution and can suspend apps.

Initial pipeline:
1. User taps and holds or taps the microphone control.
2. Browser captures audio and posts it over authenticated HTTPS to the NUC voice endpoint.
3. STT uses a locally hosted engine if performance is acceptable; otherwise use an eligible configured free route only for data permitted by privacy policy.
4. Text is passed to Julie's conversation orchestrator and task router.
5. Response is converted to speech using local Kokoro if benchmarked successfully, otherwise the iPhone/browser speech voice.
6. User can interrupt playback and correct transcription.

Voice preferences should include speaking rate, response length, warmth, and voice selection. Do not assume a specific voice is available on every iPhone. Never transmit microphone audio until the user activates the control. Recording retention should be off by default; retain only the minimum transcript/context needed for user-approved memory and task continuity.

A native iOS app or wrapper can be evaluated later if PWA limitations materially block a required feature. Do not add Apple Developer Program or other paid costs in the initial phase.

## 5. Julie's memory and personalization

Memory is a controlled service, not an unrestricted transcript dump.

Suggested memory classes:
- `personal_preferences`: communication style, routines, approved personal preferences.
- `personal_tasks`: personal reminders and plans.
- `cybr_mission`: business operations, projects and approved internal context.
- `cybr_cops`: SOC, security and customer workspaces.
- `cybr_com`: communications and technology operations.
- `working_context`: short-lived task context with expiration.
- `feedback`: explicit corrections and user-approved preference updates.

Every memory item should have an owner, scope, source, created/updated time, sensitivity classification, retention policy and provenance. Personal, CYBR MISSION, CYBR COPS and each customer tenant must be logically isolated. Cross-workspace retrieval is denied by default; the user may explicitly authorize a specific cross-scope use.

Provide a memory review screen where the user can inspect, correct, export and request deletion of stored Julie memories. Do not store credentials, authentication secrets, payment data or unnecessary sensitive personal data in conversational memory. Keep raw audio retention disabled by default.

Julie should distinguish:
- **Remembering:** storing approved preferences and task context.
- **Learning a workflow:** improving a tested procedure or skill.
- **Model training:** changing model weights; not part of the initial plan.

## 6. Autonomy and approvals

Julie may perform low-risk, reversible tasks within an explicit allowlist. Every tool has a schema, scope, permission and audit policy.

### Automatic within approved scope
- Summarize, research, classify and organize information.
- Draft emails, reports, proposals and messages without sending.
- Create personal task drafts and reminders in an approved task store.
- Prepare meeting briefs and daily summaries.
- Run read-only health checks and approved monitoring.
- Delegate bounded analysis tasks to approved Ruflo agents.
- Update low-risk internal task status when the user has enabled that exact workflow.

### Require confirmation before execution
- Sending external messages or publishing content.
- Creating or changing external calendar events with invitees.
- Purchasing, subscribing, booking or committing funds.
- Changing production configuration, DNS, identity, access, security policy or backups.
- Closing incidents, modifying customer records, or sending client-facing security conclusions.
- Launching scans or tests; require written authorization, target scope, methods and time window.
- Calling or messaging people on the user's behalf.

Approval must bind to the exact action, parameters, target, workspace, requester, policy version, expiry and one-time execution. A model or agent cannot approve its own high-impact action. Caller ID, email content and retrieved documents are not identity proof or authorization.

### Never autonomous
- Financial transfers or bank-detail changes.
- Privileged account resets or permission elevation.
- Destructive production actions or disabling security controls.
- Unapproved penetration testing, exploitation or customer data access.
- Legal, compliance or certification attestations on behalf of a human.

Include global pause, voice disable, agent execution disable and external-write disable controls.

## 7. Model routing and cost controls

Julie must not depend on Claude Agent SDK or a paid model for normal operation. The current bridge is Claude Agent SDK-coupled; replacing that runtime is a separate implementation milestone and must be tested before removing the existing path.

Preferred routing:
1. Local model for private data and tasks supported by available NUC hardware.
2. FreeLLMAPI configured free-tier provider for eligible non-sensitive tasks.
3. Local fallback or a clear unavailable response if free providers are unavailable.
4. Paid providers disabled by default; no silent paid fallback.

The router must check capabilities, privacy class, provider health, quotas, context size, streaming/tool support, timeouts and token limits. Do not expose FreeLLMAPI credentials in browser bundles. Do not send confidential client evidence, raw production logs, credentials or personal data to third-party free endpoints without explicit policy approval.

## 8. Agent workforce and skill improvement

Julie is the user-facing coordinator. Ruflo agents are specialized workers with individual identities and least-privilege tool access.

Initial skill families:
- Personal productivity and daily planning.
- Research and decision briefs.
- Writing and communications.
- CYBR MISSION business administration.
- CYBR COPS security analysis and SOC reporting.
- IT operations and infrastructure monitoring.
- Project coordination and follow-up.

Improvement loop:
1. Capture explicit user corrections and task outcomes.
2. Identify repeated friction or low-quality results.
3. Propose a workflow/skill improvement.
4. Build and test it in an isolated branch or sandbox using synthetic fixtures.
5. Run unit, security, regression and policy tests.
6. Present meaningful changes for approval.
7. Deploy through reviewed GitHub PR and a rollback path.

Julie may automatically refine low-risk user preferences and draft workflows when the user has enabled that feature. She may not alter her own permissions, policies, model budget, security boundaries or production infrastructure.

Measure task completion, user corrections, reviewer acceptance, latency, reliability, false positives, quota use and estimated time saved. Do not optimize for engagement or message volume.

## 9. Stress-reduction experience

Julie should reduce interruptions rather than create more:
- One concise daily briefing at a user-configured time.
- Priority tiers: urgent, important, routine and informational.
- Batch low-priority notifications into a digest.
- Do-not-disturb and quiet hours.
- Proactive preparation of meeting notes, follow-ups and decision briefs.
- Clear "I handled", "I prepared", "I need your approval" and "I cannot do this" status labels.
- No fake progress, simulated alerts or claims of actions that did not execute.
- Every reminder or proactive action should be configurable and easy to pause.

## 10. Delivery plan and acceptance gates

### Milestone 0 — Inventory and safety baseline
- Inspect current JARVIS frontend, bridge, scripts and dependencies.
- Confirm FreeLLMAPI endpoints and model catalog without printing secrets.
- Inventory NUC CPU/GPU/RAM and benchmark STT/TTS candidates.
- Review Ruflo tools and disable unneeded high-risk execution.
- Confirm Cloudflare Tunnel/Access configuration and existing DNS; do not alter production records.
- Confirm the existing CYBR COM live deployment remains untouched.

**Gate:** documented inventory, no exposed management ports, no production changes.

### Milestone 1 — Julie identity and responsive UI
- Replace visible JARVIS naming with Julie in a feature branch.
- Add personality and voice settings.
- Add PWA manifest, icons, service worker and responsive iPhone layout.
- Preserve existing UI capabilities until regression tests pass.

**Gate:** production build, lint/tests, iPhone Safari manual test, no secrets in assets.

### Milestone 2 — Provider-neutral conversation core
- Add runtime adapter for FreeLLMAPI and local model endpoints.
- Separate model session from Julie identity and memory.
- Add authenticated bridge/API, strict origin checks, rate limits and schema validation.
- No paid provider fallback.

**Gate:** verified free/local conversation path; paid path blocked by default; prompt injection and tool-boundary tests.

### Milestone 3 — Voice
- Add foreground push-to-talk capture and STT.
- Benchmark local Kokoro and select a warm female voice profile where supported.
- Add interrupt, transcription correction and explicit microphone state.

**Gate:** voice works on the user's iPhone; no hidden microphone capture; no audio retention by default.

### Milestone 4 — Memory, notifications and approvals
- Implement scoped memory, review controls and retention.
- Implement iOS Home Screen Web Push registration and notification preferences.
- Add task/approval inbox and auditable one-time approvals.

**Gate:** tenant/workspace isolation tests, memory access/deletion tests, push opt-in/out, approval replay prevention.

### Milestone 5 — Agent workforce and integrations
- Connect Ruflo through a restricted task broker.
- Add approved integrations one at a time.
- Start with read-only and draft-only capabilities.
- Add bounded routine automation after reliability testing.

**Gate:** every tool has a least-privilege policy, audit event, failure handling and human approval rule.

### Milestone 6 — Proactive operations and measured improvement
- Add daily briefing, digest, quiet hours and task prioritization.
- Add skill proposal/evaluation pipeline and human-reviewed promotion.
- Track actual time saved and user corrections.

**Gate:** measurable reduction in manual work without excessive notifications or unsafe autonomous actions.

## 11. Deployment boundaries

- Keep the existing public CYBR COM website and its current Hostinger/GitHub deployment unchanged.
- Do not deploy the command console or Julie into the existing live document root.
- Use a separate staging website and dedicated staging branch only after the frontend passes build and security checks.
- Do not expose NUC, FreeLLMAPI, Ruflo, Ollama or bridge management ports publicly.
- Keep secrets in protected server-side environment files or a suitable secret store; never commit or print them.
- Do not change Cloudflare DNS, Hostinger settings, or NUC network configuration as part of this architecture-only milestone.

## 12. Definition of success

Julie is successful when the user can securely talk to her from an iPhone, she understands approved context across personal and CYBR workspaces, remembers useful preferences, prepares and completes permitted routine tasks, asks before consequential actions, and measurably reduces the user's workload—without new mandatory recurring costs, fake activity, hidden data sharing or weakened security.

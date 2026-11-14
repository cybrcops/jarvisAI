# Julie and Mira operating model

**Decision date:** 2026-11-14  
**Status:** Owner-approved strategic architecture direction; implementation remains incremental.

## Reporting relationship

- **Owner:** final authority and policy owner.
- **Julie — Personal Executive Assistant:** works directly with the owner, handles delegated personal and professional tasks, maintains the owner's private EA context, and provides a consolidated, source-attributed view across CYBR MISSION and CYBRCOPS. Julie coordinates directly with Mira.
- **Mira — Chief of Staff:** runs company/workforce coordination, delegates to specialist agents, tracks projects, decisions, commitments, blockers, and execution, and prepares the company morning briefing. Mira coordinates directly with Julie.
- **Specialist agents:** perform bounded work in security/SOC, infrastructure, DevSecOps, support, compliance, finance, customer operations, and other approved domains.
- **CYBR COM AI Command:** policy, approvals, identity, audit, monitoring, and emergency-stop/control plane. It is not an unrestricted agent administrator.

Julie works directly for the owner; Mira runs the AI workforce and company coordination; Julie and Mira coordinate directly; both ultimately report to the owner.

## Julie's cross-business visibility

Julie is intended to have authorized, read-oriented visibility across CYBR MISSION and CYBRCOPS, including business health, customer needs, tickets/incidents, delivery commitments, projects, security posture, application health, financial indicators where authorized, agent activity, open decisions, and recommended owner actions.

This is a target capability, not a claim that all connectors or permissions currently exist. Inventory each source, data owner, permitted purpose, role, freshness, and connector gap before implementation. Julie receives write access only through specific delegated capabilities and action-specific policy checks.

## Mandatory security scopes

1. Owner-private personal workspace and memory.
2. CYBR MISSION company workspace.
3. CYBRCOPS company workspace.
4. Separate customer-specific tenant/data scopes.

Never allow owner-private memory to flow into company or customer contexts. Never allow one customer's data to flow to another customer. Company agents and Mira do not automatically inherit Julie's private memory. Cross-business visibility does not mean shared unrestricted credentials or a shared undifferentiated database context.

## Autonomy and emergency operations

- Routine, low-risk, reversible actions may be automated only when explicitly preapproved and bounded.
- External communications, publishing, purchases, customer-facing conclusions, security tests, production changes, destructive actions, privilege elevation, and consequential containment require action-specific approval.
- During owner unavailability, emergency actions are permitted only when a preapproved incident playbook defines evidence/confidence thresholds, target scope, permitted containment, evidence preservation, verification, notification, and recovery/rollback.
- Never autonomously transfer funds, elevate privileges, perform destructive production actions, or conduct unapproved penetration tests.
- Every action must record the requesting principal, agent, tenant, tool, purpose, policy decision, approval token where applicable, target, before/after evidence, result, and timestamp.
- Provide global pause, agent stop, and external-write/tool kill switches.

## Executive reporting standard

Every owner-facing briefing must distinguish:
- **Verified fact:** source and last-updated time are shown.
- **Agent assessment:** reasoning and confidence are identified as an assessment, not fact.
- **Recommendation:** proposed next step, urgency, impact, and whether approval is required.

Do not claim 24/7 coverage until always-on runtimes, monitoring sources, alert delivery, failover, audit logging, and escalation paths are implemented and tested. Do not claim that Julie or Mira can act through an integration until its access and permissions have been verified.

## Implementation sequence

1. Map the primary platform's auth, tenant model, APIs, data model, and audit surfaces.
2. Define identity, tenant context, policy decision, approval token, and immutable audit-event contracts.
3. Implement read-only, source-attributed adapters for CYBR MISSION and CYBRCOPS.
4. Build Julie's owner briefing and Mira's workforce/decision tracking interfaces.
5. Add bounded specialist-agent delegation and action-specific approval.
6. Test cross-tenant denial, private-memory isolation, privilege boundaries, failure handling, and emergency playbooks using synthetic data.
7. Expand automation only after evidence-backed tests and owner approval.

No production, NUC, hosting, DNS, customer-data, or security-control changes are authorized by this architecture document.

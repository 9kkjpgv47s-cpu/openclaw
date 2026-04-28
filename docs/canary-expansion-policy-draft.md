# Canary expansion policy (draft)

**Status:** Draft — not yet approved for enforcement.  
**Audience:** Engineering and release owners who roll out changes behind traffic splits, feature flags, staged deployments, or ring-based rollouts.

## Purpose

This draft defines how we expand a **canary** (limited exposure of a new build or configuration) toward full traffic in a controlled, observable, and reversible way. The goal is to catch regressions early, limit blast radius, and make expansion decisions explicit rather than implicit.

## Scope

This draft applies whenever **gradual exposure** is possible: traffic splits, feature flags, staged deploys, ring-based rollouts, or percentage-based canaries.

It does not replace repo-specific runbooks; it sets **minimum expectations** for how expansion is decided, monitored, and reversed.

## Definitions

- **Canary:** A release slice or cohort that receives a subset of production traffic, users, regions, or instances while the stable release continues to serve the remainder.
- **Expansion:** Increasing the canary’s share (traffic percentage, cohort size, region count, or instance pool) according to predefined steps.
- **Stable:** The incumbent release or default behavior that remains the fallback target for rollback and for traffic not routed to the canary.
- **Promotion:** A discrete step that makes the new behavior the default for a larger surface (merge to main, flip default flag, scale traffic weight).

## Principles

1. **Progressive exposure:** Expand in steps (for example 1% → 5% → 25% → 50% → 100%), not in a single jump, unless a documented exception is approved. Numeric ladders belong in per-service runbooks.
2. **Default to small first:** Start with the smallest meaningful canary that still produces useful signal (errors, latency, business metrics).
3. **Time at each step:** Hold each step long enough for latency, error, and saturation-sensitive signals to stabilize (minimum dwell time is set per service in its runbook).
4. **One lever at a time:** When debugging, avoid widening the canary and shipping unrelated changes in the same window.
5. **Automate routing, human gates:** Automation may advance traffic only within approved bounds; steps that cross risk thresholds require human acknowledgment.
6. **Explicit human gate before major jumps:** Large jumps in traffic or promotion to broad default behavior require the same class of approval as other high-impact actions (see **Relation to existing workspace norms** below).
7. **Measurable exit criteria:** Each expansion step names what “good” looks like and how long to observe before the next step.
8. **Rollback is first-class:** Any step must be reversible to stable without redeploying stable from scratch, where platform capabilities allow.

## Phases (reference model)

Stages are illustrative; teams map them to their stack (flags, load balancer weights, deploy rings).

| Phase | Typical intent | Before advancing |
|-------|----------------|------------------|
| **0 — Design** | Canary %, cohort, duration, metrics, rollback documented | Review acknowledged |
| **1 — Canary start** | Minimal exposure | Baseline metrics captured |
| **2 — Early expansion** | Modest increase (internal + early adopters, or low %) | No regression vs baseline on agreed metrics |
| **3 — Broad expansion** | Majority of traffic or all non-critical paths | Sustained healthy window per runbook |
| **4 — Full default** | New behavior is default | Final approval per org policy |

## Preconditions before first traffic

Before any production canary receives real traffic:

- Change is merged and tagged (or equivalent immutable artifact reference exists).
- Automated tests for the change (or release train) have passed on the artifact that will run in production.
- Observability: dashboards and alerts exist for golden signals (errors, latency, saturation) and for service-specific business metrics where applicable.
- Rollback path is documented (traffic shift, feature flag off, or redeploy to last known good).

## Expansion gates (draft checklist)

Between steps, the release owner verifies:

| Gate | Check |
|------|--------|
| Errors | Canary error rate is not worse than stable beyond an agreed tolerance (absolute and relative). |
| Latency | p95/p99 (or agreed SLO windows) for canary are within budget vs stable or prior baseline. |
| Saturation | CPU, memory, queue depth, or pool exhaustion are not trending worse on canary cohort. |
| Business metrics | Key funnel or revenue-adjacent metrics, if applicable, show no sustained negative divergence. |
| Dependencies | Downstream services and caches tolerate the new behavior (no new throttling or cascade). |
| Integrity | No open sev-1/2 issues tied to the change without a documented exception. |
| Ownership | On-call or owner available for the next expansion window, or expansion is deferred. |
| Consent | Where policy requires operator or stakeholder sign-off for promotion, that requirement is met for the **promotion** action, not only the initial canary. |

If any gate fails, **do not expand**; investigate, roll back, or hold until resolved.

## Roles (draft)

- **Release owner:** Executes expansion steps, records timestamps and percentages, declares go/no-go between steps.
- **On-call / SRE:** Consulted when alerts fire or gates are ambiguous; may veto expansion.
- **Author / reviewer:** Available for rapid triage during the canary window for the change in question.

Exact role names may be mapped to your org’s titles; responsibilities above stay the same.

## Rollback and freeze

- If gates fail, **do not expand**; roll back or hold at current canary size until root cause is understood.
- During incident response, **freeze** canary expansion for the affected system until the incident is closed or a separate decision records a controlled exception.

## Documentation and audit trail

For each production canary:

- Record artifact id (image digest, git SHA, build id).
- Log each expansion step: time (UTC), previous and new percentage or scope, who approved, and pointer to dashboards used.
- Link incident or postmortem if rollback occurred.

## Exceptions

Exceptions (single-step full cutover, skipped gates, or compressed timelines) require:

- Written rationale (incident response, reverted hotfix re-promotion with identical artifact, etc.).
- Named approver outside the release owner (role to be defined by final policy).

## Relation to existing workspace norms

- **SCOPE.md** — Material automation and high-impact actions require explicit approval; canary expansion to full production is treated as high impact when it changes default behavior for external users.
- **Deploy helpers (`bin/eve-deploy-*-safe`)** — Registry and operator flags remain the enforcement layer for dispatch-style deploys; this policy describes *when* widening is appropriate, not how to bypass those checks.

## Open questions for v1

- Default step ladder and minimum dwell per tier (tier 0 vs tier 2 services).
- Default observation durations per risk class (low / medium / high).
- Who may authorize each phase (owner vs operator vs change board).
- Whether automated analysis (e.g. error budget burn) may auto-advance within a ceiling.
- Geo- or tenant-based canaries vs pure percentage splits.
- Alignment with feature-flag policies when both flags and deployment canaries apply.
- How this policy interacts with mobile-repo-deploy-registry and per-repo canary configs, where applicable.

---

*Draft for review. Revise dates, roles, and numeric thresholds before adoption.*

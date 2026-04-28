# Canary expansion policy (draft)

**Status:** Draft — not yet approved for enforcement.  
**Audience:** Engineering and release owners who roll out changes behind traffic splits or staged deployments.

## Purpose

This draft defines how we expand a **canary** (a limited exposure of a new build or configuration) toward full traffic in a controlled, observable, and reversible way. The goal is to catch regressions early, limit blast radius, and make expansion decisions explicit rather than implicit.

## Definitions

- **Canary:** A release slice that receives a subset of production traffic, users, regions, or instances while the stable release continues to serve the remainder.
- **Expansion:** Increasing the canary’s share (traffic percentage, cohort size, region count, or instance pool) according to predefined steps.
- **Stable:** The incumbent release that remains the fallback target for rollback and for traffic not routed to the canary.

## Principles

1. **Progressive exposure:** Expand in steps (for example 1% → 5% → 25% → 50% → 100%), not in a single jump, unless a documented exception is approved.
2. **Time at each step:** Hold each step long enough for latency, error, and saturation-sensitive signals to stabilize (minimum dwell time is set per service in its runbook).
3. **Automate routing, human gates:** Automation may advance traffic only within approved bounds; steps that cross risk thresholds require human acknowledgment.
4. **Rollback is first-class:** Any step must be reversible to stable without redeploying stable from scratch, where platform capabilities allow.

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

If any gate fails, **do not expand**; investigate, roll back, or hold until resolved.

## Roles (draft)

- **Release owner:** Executes expansion steps, records timestamps and percentages, declares go/no-go between steps.
- **On-call / SRE:** Consulted when alerts fire or gates are ambiguous; may veto expansion.
- **Author / reviewer:** Available for rapid triage during the canary window for the change in question.

Exact role names may be mapped to your org’s titles; responsibilities above stay the same.

## Documentation and audit trail

For each production canary:

- Record artifact id (image digest, git SHA, build id).
- Log each expansion step: time (UTC), previous and new percentage or scope, who approved, and pointer to dashboards used.
- Link incident or postmortem if rollback occurred.

## Exceptions

Exceptions (single-step full cutover, skipped gates, or compressed timelines) require:

- Written rationale (incident response, reverted hotfix re-promotion with identical artifact, etc.).
- Named approver outside the release owner (role to be defined by final policy).

## Open questions for v1

- Default step ladder and minimum dwell per tier (tier 0 vs tier 2 services).
- Whether automated analysis (e.g. error budget burn) may auto-advance within a ceiling.
- Geo- or tenant-based canaries vs pure percentage splits.
- Alignment with feature-flag policies when both flags and deployment canaries apply.

---

*Draft for review. Revise dates, roles, and numeric thresholds before adoption.*

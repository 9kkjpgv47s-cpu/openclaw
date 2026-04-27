# Policy Draft: Canary Expansion

**Status:** Draft  
**Last updated:** 2026-04-27  
**Purpose:** Define how canary releases are expanded from initial exposure to full rollout, including gates, metrics, and rollback.

---

## 1. Scope

This policy applies to production changes that are released using a **canary** (limited-traffic or limited-scope) pattern before serving all users or all infrastructure. It does not replace change-management or security review; it sits alongside them for rollout mechanics.

## 2. Definitions

- **Canary:** A release that receives a deliberately small share of traffic, workloads, or regions before general availability.
- **Expansion:** The act of increasing that share according to predefined steps and approval criteria.
- **Baseline:** The stable release currently serving the majority of traffic (or the prior canary step).

## 3. Principles

1. **Expand only on evidence.** Each step requires healthy signals relative to baseline for the agreed observation window.
2. **Reversible by default.** Expansion steps must be small enough that rollback or traffic shift back to baseline is practical and documented.
3. **Single moving part.** Avoid combining unrelated changes in one canary; expansion decisions attribute outcomes to one release.
4. **Explicit ownership.** A named owner approves each expansion step and is accountable for rollback if thresholds fail.

## 4. Expansion stages (draft defaults)

Stages are illustrative; teams may tune percentages and durations to their platform.

| Stage | Typical traffic / scope | Minimum observation |
|-------|-------------------------|---------------------|
| 0 | Internal or synthetic only | Smoke + automated checks |
| 1 | ≤1% production (or one cell/zone) | Short window + SLO checks |
| 2 | ~5–10% | Full error/latency budget vs baseline |
| 3 | ~25–50% | Extended window if risk is higher |
| 4 | 100% | Final sign-off if required by org policy |

**Freeze:** No expansion during known high-risk windows (e.g. major events, holidays) unless explicitly approved.

## 5. Gates before each expansion

All must pass before increasing scope:

- **Automated:** Unit/integration tests for the release; deployment health checks; config validation.
- **SLO / error budget:** Error rate, latency, and saturation within agreed thresholds vs baseline (or explicit waiver documented).
- **Business / safety:** No active sev-1; no unresolved blocker from security or compliance for this change class.

## 6. Rollback and halt

- **Automatic halt:** Predefined triggers (e.g. SLO burn, crash loop, critical alert) stop expansion and revert or shed load to baseline per runbook.
- **Manual halt:** Any stage owner or incident commander may freeze expansion pending review.
- **Post-incident:** If rollback occurs, a short note captures cause, whether the canary was at fault, and required follow-ups before retry.

## 7. Documentation and audit

For each production canary expansion:

- Change ticket or record: version, owner, stages and timestamps.
- Link to dashboards and alert rules used for the decision.
- Final state (full rollout, rolled back, or partial with rationale).

## 8. Open questions (for review)

- Org-specific percentages, durations, and mandatory approvers.
- Whether certain change types skip canary or require longer stage-1 windows.
- Multi-region ordering (which region expands first).
- Integration with feature flags vs infrastructure canaries when both apply.

---

*This document is a draft for discussion and does not constitute final policy until reviewed and adopted.*

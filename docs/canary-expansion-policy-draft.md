# Canary expansion policy (draft)

**Status:** Draft — not yet binding operational policy.  
**Audience:** Operators and agents routing work through Cursor, GitHub Actions, and related automation.  
**Related context:** `SCOPE.md` (consent, conservative defaults), `bin/eve-deploy-gha-safe` and related wrappers (explicit operator approval, registry allowlists).

## 1. Purpose

Define how **canary** exposure of new behavior is introduced and how **expansion** to broader scope is allowed or blocked. The goal is predictable risk: small blast radius first, measurable signals, and clear stop conditions before full rollout.

## 2. Definitions

- **Canary** — A limited slice of traffic, repos, workflows, users, or environments that receives a change before everyone else. The slice must be **named and enumerable** (for example: one repo, one branch, one workflow, one internal account, one region).
- **Expansion** — Increasing the canary’s scope: more repos, more environments, higher traffic share, or removal of guardrails **only after** gates in this policy are satisfied.
- **Control plane** — The place where scope is configured (registry JSON, feature flags, workflow inputs, branch protection, or equivalent). Changes to the control plane are as sensitive as the rollout itself.

## 3. Principles

1. **Default deny** — New automation or deploy paths do not run at full scope until explicitly expanded per this policy.
2. **Human in the loop for material impact** — Align with `SCOPE.md`: destructive, external, or high-impact actions require direct confirmation; canary expansion is treated as high-impact when it widens who or what is affected.
3. **One axis at a time** — Prefer widening a single dimension (for example: more repos *or* higher percentage, not both in one step) so incidents are attributable.
4. **Reversible** — Every expansion step must have a documented rollback: revert config, disable flag, pin previous ref, or stop workflow dispatch.

## 4. Phases (recommended)

| Phase | Scope | Typical intent |
|--------|--------|----------------|
| **Pilot** | Single target, dry-run or read-only where possible | Validate wiring and permissions |
| **Canary** | Small fixed set or low percentage | Observe errors, latency, and business KPIs under real load |
| **Expanded canary** | Larger set or stepped percentage | Confirm stability across diversity of targets |
| **General availability** | Full intended scope | Default path for all eligible targets |

Movement between phases is **not** automatic; it requires an explicit decision recorded in the change ticket or deployment log (who, when, from → to).

## 5. Expansion gates (all must pass before widening)

- **Error and health signals** — No sustained regression versus baseline on agreed metrics (for example: workflow failure rate, alert volume, customer-visible errors) for a defined observation window.
- **Security and compliance** — No open sev1/sev2 issues tied to the change; secrets and tokens remain least-privilege; audit trail present for the new path.
- **Operational readiness** — Runbook updated: how to detect failure, who is on call, how to roll back, and how to freeze expansion.
- **Explicit approval** — Named operator approval for the **specific** next scope (repo list, percentage, or environment list), consistent with existing deploy scripts that require flags such as `--approved-by-operator`.

If any gate cannot be verified, **do not expand**; hold or roll back.

## 6. Halt and rollback triggers

Immediately stop expansion and revert to the prior scope (or full rollback) if any of the following occur:

- Sev1 incident or data-loss class event linked to the change.
- Spike in failures or timeouts beyond agreed thresholds versus pre-change baseline.
- Unexpected privilege use, auth failures affecting production, or policy violations detected in audit logs.
- Missing or ambiguous ownership for the next expansion step.

## 7. Documentation and versioning

- This file is a **draft**; when promoted, replace “draft” in the title, set an effective date, and link the approved version from `SCOPE.md` or the team’s canonical policy index.
- Keep a short changelog at the bottom of the policy when sections move from draft to enforced.

## 8. Open questions (for v1)

- Default observation window length per phase (hours vs days) per risk class.
- Whether percentage-based routing is in scope for this environment or only discrete allowlists.
- Mapping of this policy to specific registries and workflows (fill in concrete paths and owners when known).

---

*Draft captured for branch `cursor/canary-expansion-policy-draft-7249`. Revise before adoption.*

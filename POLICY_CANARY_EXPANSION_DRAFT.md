# Policy Draft: Canary Expansion

**Status:** Draft — not yet binding. Intended for review, iteration, and explicit adoption before use in production decisions.

**Scope:** How we expand exposure after a successful **canary** (small, controlled slice) so that broader rollout stays safe, observable, and reversible.

---

## 1. Purpose

- Limit blast radius when shipping behavior, configuration, integrations, or agent/automation changes.
- Require **evidence** that the canary is healthy before **expansion** (larger audience, more traffic, more environments, or stronger autonomy).
- Align with conservative operation: expand only when criteria are met, with a defined rollback path.

## 2. Definitions

| Term | Meaning |
|------|--------|
| **Canary** | A deliberately limited deployment: subset of users, traffic, regions, sessions, or features — enough to learn, not enough to cause uncontrolled harm. |
| **Expansion** | Any increase in canary scope: percentage, cohort, geography, time window, privilege level, or automation without human-in-the-loop. |
| **Gate** | A check that must pass before expansion is allowed (metrics, errors, manual sign-off, etc.). |

## 3. Principles

1. **Default small** — Start with the smallest meaningful canary; document what “small” means per change type.
2. **One axis at a time** — Prefer expanding one dimension (e.g. traffic %) before combining with another (e.g. new privilege).
3. **Observable** — Canary and post-expansion states must be measurable; “no data” is not a pass.
4. **Reversible** — Expansion steps are sized so rollback or traffic shift remains practical.
5. **Explicit consent for material expansion** — Match operational norms: high-impact or autonomous expansion requires clear approval (see `SCOPE.md`).

## 4. Expansion stages (template)

Stages are illustrative; owners should map them to their systems (e.g. 1% → 5% → 25% → 100%, or “internal only” → “pilot customers” → “GA”).

| Stage | Typical scope | Minimum gates before next stage |
|-------|----------------|----------------------------------|
| Canary | Minimal slice | Error/latency budgets within SLO; no critical incidents; owner ack |
| Early expansion | Small minority | Sustained stability window; rollback drill or documented path |
| Majority | Most traffic/users | Same as above; stakeholder visibility if customer-facing |
| Full / steady state | 100% or new baseline | Post-expansion monitoring plan; runbook updated |

## 5. Gates (draft checklist)

Before each expansion step, confirm:

- [ ] **Health:** Error rate, timeouts, and key user/agent flows within agreed thresholds.
- [ ] **Duration:** Canary ran long enough to cover peak usage or known risk windows (define per change).
- [ ] **Ownership:** Named owner for the change and for rollback.
- [ ] **Documentation:** What changed, how to roll back, and where to look for signals.
- [ ] **Approval:** Any step that increases autonomy, data access, or external impact has recorded green light per team norms.

## 6. Stop / rollback triggers (non-exhaustive)

Halt expansion and revert or freeze scope if:

- SLO violations or sharp regressions vs. pre-canary baseline.
- Security or privacy anomaly.
- Unexplained spike in support or failure reports tied to the change.
- Missing monitoring or on-call coverage for the next expansion window.

## 7. Open questions (to resolve before “final” policy)

- Quantitative defaults: minimum canary duration, maximum step size between stages, and allowed parallelism of experiments.
- How this policy applies to **agent / ACP** workloads vs. traditional services (per-harness limits, session caps, etc.).
- Escalation path when gates are ambiguous (e.g. noisy metrics).

---

*Draft maintained for canary expansion discipline. Promote to “active policy” only after review and explicit adoption.*

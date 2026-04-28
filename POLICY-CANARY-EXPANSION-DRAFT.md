# Policy Draft: Canary Expansion

**Status:** Draft — not yet adopted.  
**Purpose:** Define how we widen exposure of new behavior (features, configs, deployments) from a small slice to full rollout without bypassing safety or consent.

---

## 1. Scope

This draft applies to any change that affects production or user-visible systems where **gradual exposure** is possible: traffic splits, feature flags, staged deploys, ring-based rollouts, or percentage-based canaries.

It does not replace repo-specific runbooks; it sets **minimum expectations** for how expansion is decided, monitored, and reversed.

---

## 2. Definitions

- **Canary** — A limited cohort or environment that receives the new behavior first (for example: internal users, one region, N% of traffic, or a preview deployment).
- **Expansion** — Deliberately increasing that cohort (more users, more regions, higher percentage, promotion to production) after evidence supports it.
- **Promotion** — A discrete step that makes the new behavior the default for a larger surface (merge to main, flip default flag, scale traffic weight).

---

## 3. Principles

1. **Default to small first.** Start with the smallest meaningful canary that still produces signal (errors, latency, business metrics).
2. **One lever at a time.** When debugging, avoid widening the canary and shipping unrelated changes in the same window.
3. **Explicit human gate before major jumps.** Large jumps in traffic or promotion to “everyone” require the same class of approval as other high-impact actions (see operational scope: explicit green light for material action).
4. **Measurable exit criteria.** Each expansion step names what “good” looks like (SLOs, error rate, key workflows) and how long to observe before the next step.
5. **Fast rollback path.** Before expanding, confirm how to revert (flag off, redeploy previous artifact, traffic weight back) and that rollback is tested or rehearsed where feasible.

---

## 4. Phases (reference model)

Stages are illustrative; teams may map them to their stack (flags, load balancer weights, deploy rings).

| Phase | Typical intent | Before advancing |
|-------|----------------|------------------|
| **0 — Design** | Canary %, cohort, duration, metrics, rollback documented | Review acknowledged |
| **1 — Canary start** | Minimal exposure | Baseline metrics captured |
| **2 — Early expansion** | Modest increase (e.g. internal + early adopters, or low %) | No regression vs baseline on agreed metrics |
| **3 — Broad expansion** | Majority of traffic or all non-critical paths | Sustained healthy window per runbook |
| **4 — Full default** | New behavior is default | Final approval per org policy |

**Draft note:** Numeric thresholds (e.g. 1% → 5% → 25%) should be filled in per service in the adopting runbook, not in this generic draft.

---

## 5. Gates (must be satisfied to expand)

- **Stability:** Error rates and latency within agreed bounds for the observation window.
- **Integrity:** No open sev-1/2 issues tied to the change without documented exception.
- **Ownership:** On-call or owner available for the next expansion window (or expansion is deferred).
- **Consent:** Where policy requires operator or stakeholder sign-off (for example deploy scripts that require explicit `--approved-by-operator`), that requirement is met for the **promotion** action, not only the initial canary.

---

## 6. Rollback and freeze

- If gates fail, **do not expand**; roll back or hold at current canary size until root cause is understood.
- During incident response, **freeze** canary expansion for the affected system until the incident is closed or a separate decision records a controlled exception.

---

## 7. Relation to existing workspace norms

- **SCOPE.md** — Material automation and high-impact actions continue to require explicit approval; canary expansion to full production is treated as high impact when it changes default behavior for external users.
- **Deploy helpers (`bin/eve-deploy-*-safe`)** — Registry and operator flags remain the enforcement layer for dispatch-style deploys; this policy describes *when* widening is appropriate, not how to bypass those checks.

---

## 8. Open questions (for adoption)

- Default observation durations per risk class (low / medium / high).
- Who may authorize each phase (owner vs operator vs change board).
- How this policy interacts with mobile-repo-deploy-registry and per-repo canary configs.

---

*Draft for review. Revise before treating as binding.*

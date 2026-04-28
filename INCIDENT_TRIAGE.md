# Incident triage and canary expansion

Use this when something is broken or degraded in production, or when you are rolling out a fix or risky change behind a canary. It complements `TOOLS.md` (operational runbooks) and `SCOPE.md` (how Eve routes work).

## Triage (first minutes)

1. **Confirm the signal** — Separate user-impacting outage from noise (flaky alerts, test traffic, dependency blips). Note scope (region, cohort, feature flag, deploy).
2. **Stabilize or isolate** — Prefer feature flags, traffic shaping, or rollback over hot-patching unless rollback is worse. Document what changed recently (deploy, config, dependency, quota).
3. **Communicate** — Short internal status: impact, suspected cause, owner, next update time. Escalate if severity or blast radius grows.
4. **Evidence** — Preserve logs, traces, dashboards, and deploy metadata before they age out. Link tickets or threads in one place.

## Severity framing (lightweight)

- **Critical** — Broad user impact or data integrity risk; all hands until mitigated.
- **Major** — Significant subset or degraded experience; workarounds may exist.
- **Minor** — Limited edge cases or cosmetic issues; schedule fix without paging everyone.

Adjust labels to match your org; the point is consistent vocabulary between triage and rollout.

## Canary expansion (staged rollout)

A **canary** ships a change to a small slice of traffic or hosts first, then **expands** only when health checks pass. Expansion is deliberate widening—not "turn it on for everyone" until gates are met.

### Before you start

- Define **success metrics** (error rate, latency p99, saturation) and **guardrails** (budgets, anomaly thresholds).
- Know **rollback** path: revert deploy, flip flag off, or drain bad instances—one clear default.
- Ensure **observability** can slice by canary cohort (flag variant, cluster, version label).

**Pre-flight checklist (do not skip):**

- **Change ID** — One deploy SHA, flag name + default/off state, or config key; no mixed bundles in the same canary unless unavoidable.
- **Baseline window** — Agree pre-change reference (same day-of-week and time window if traffic is seasonal).
- **Owner + comms** — Named approver for “advance” and “abort”; status channel or ticket updated with current stage and slice.
- **Kill switch tested** — Rollback or flag-off exercised in non-prod or documented as low-risk one-click; know who can execute it after hours.
- **Cohort visibility** — Dashboards or queries filtered by canary label work *before* traffic moves; spot-check a canary-only request if your stack allows.

**Deploy vs feature-flag canaries:** A **deploy** canary routes a fraction of instances or cells to a new binary; a **flag** canary gates behavior for a user or request cohort. You can combine them (new binary behind a flag)—treat each as a separate expansion with its own gates if both move independently.

### Recommended stages (example)

Tune percentages and dwell time to your risk and traffic shape. Low traffic may need longer windows or larger minimum cohorts for statistical confidence.

| Stage | Typical traffic slice | Dwell (minimum) | Gate to advance |
|--------|------------------------|-----------------|-----------------|
| 1 | ~0.1–1% or single cell | 15–30 min | No SLO burn; errors/latency within baseline band |
| 2 | ~5–10% | 30–60 min | Same; no new error fingerprints |
| 3 | ~25–50% | 1–2 h | Same; support/tickets not spiking on canary cohort |
| 4 | 100% | — | Final review; on-call ack |

If any stage fails, **stop expansion**, roll back or hold at previous stage, and triage as a new incident if user-visible.

**Per-stage advance checklist (before widening):**

- Dwell time at least met; no open sev-1/2 tied to this change without explicit exception.
- Canary slice shows **no** sustained regression vs baseline on agreed metrics (not a single noisy minute).
- Error logs: no **new** stack traces or dependency errors concentrated on canary cohort.
- Infra: CPU/memory/connection pools within budget; no retry storms or queue backlog growth on canary only.
- Product/support: no correlated spike in tickets, chat volume, or funnel drop for canary users (when measurable).
- Document “advance to stage N” with timestamp and approver in the incident or rollout thread.

**Low traffic:** If volume is too low for tight error bands, prefer longer dwell, larger minimum cohort (absolute request count), or shadow/dark traffic before user-visible canary.

### Gates (all should pass before widening)

- **Automated:** Error rate, timeouts, saturation, queue depth, and dependency health for the canary cohort match or beat pre-change baseline within agreed tolerance.
- **Synthetic / probes:** Critical paths green from canary-facing vantage if you have them.
- **Human:** On-call comfortable; no unexplained spikes in business KPIs tied to the change.

### Rollback triggers (non-exhaustive)

Expand **only** when stable; shrink or revert on any of:

- SLO burn or alert policy firing on canary-specific series.
- New exception class or dependency failure correlated with canary.
- Manual "stop" from on-call or incident commander.
- **Ambiguous signal** — If you cannot explain a regression or improvement on the canary slice, **hold** until you can; do not widen on uncertainty.
- **Data integrity** — Any hint of wrong writes, duplicate side effects, or authz leakage: revert immediately, then triage.

**After rollback:** Confirm blast radius (how long bad traffic ran), whether data repair is needed, and whether to re-canary with a fix or different gates.

### After full rollout

- Record **what happened**, **root cause** (if known), **timeline**, and **follow-ups** (tests, monitors, runbook gaps).
- If the change was a **fix** for an incident, link the postmortem and confirm monitors cover the failure mode so the next canary catches regressions earlier.

## Eve / Cursor agents in incidents

When spawning agents for investigation or fixes, follow `SCOPE.md` (ACP harness names, evidence block). Do not treat agent output as production changes until reviewed and merged through your normal pipeline—same canary rules apply to code that ships.

---

*Living doc. Revise stages and gates to match your stack and on-call practice.*

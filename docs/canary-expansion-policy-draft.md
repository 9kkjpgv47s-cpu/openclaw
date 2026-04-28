# Canary expansion policy (draft)

**Status:** Draft — not yet adopted as mandatory procedure.  
**Scope:** How we widen exposure of risky or high-impact changes after a limited “canary” phase, consistent with `SCOPE.md` (consent, transparency, ACP harness rules).

---

## 1. Purpose

A **canary** is a deliberately small slice of traffic, users, environments, or agent workloads that receives a change first. **Expansion** means increasing that slice (more traffic, more repos, more sessions, or broader automation) only when defined signals stay healthy.

This draft sets default expectations so expansion is **gradual**, **reversible**, and **explicitly approved**, not accidental scope creep.

---

## 2. Definitions

| Term | Meaning |
|------|--------|
| **Canary cohort** | The minimal set where the change runs first (e.g. one branch, one harness, one project, internal-only). |
| **Expansion step** | A discrete increase in cohort size or capability (e.g. second project, production path, or higher concurrency). |
| **Gate** | A measurable condition that must pass before the next expansion step. |
| **Rollback** | Reverting traffic or disabling the change in the affected cohort; may be partial (cohort-only). |

---

## 3. Preconditions (before any canary)

1. **Change is identifiable** — What is being rolled out, to whom/what, and how it is toggled or deployed is documented in the task or PR.
2. **Harness and runtime correctness** — For ACP-driven work, `agentId` must be the acpx harness name for the task type (see `SCOPE.md` / `TOOLS.md`). Never use `"main"` as a harness id.
3. **Failure modes known** — At least one sentence on what “bad” looks like and how to detect it (errors, latency, wrong output, auth failures).
4. **Green light for material action** — Per `SCOPE.md`, automating, pushing code, or other material steps requires explicit approval unless the human has already granted scope for this rollout.

---

## 4. Expansion principles

1. **One lever at a time** — Prefer expanding either audience *or* concurrency *or* feature depth, not all three in one step.
2. **Time between steps** — Allow enough observation time for the current cohort before widening (length depends on risk: higher risk → longer bake).
3. **No silent expansion** — Each expansion step should be stated (even briefly) when reporting back: what widened, what gates were checked, and what will be watched next.
4. **Default conservative** — If gates are ambiguous or data is missing, hold or narrow the cohort rather than widen.

---

## 5. Suggested gates (pick what applies)

Use the subset that matches the change. Not every gate applies to every rollout.

- **Correctness** — No new critical errors in logs, tests, or human review for the canary cohort.
- **Auth / configuration** — No `Authentication required`, `ACP runtime backend is not configured`, or equivalent; if those appear, stop and surface per `SCOPE.md` (no blind retries for config/auth).
- **Performance / cost** — Token usage, latency, or rate limits remain within agreed bounds.
- **Safety** — No destructive or external actions outside what was explicitly approved for this phase.

---

## 6. Rollback and halt triggers

**Halt or roll back immediately** when any of the following occur in the canary cohort:

- Regressions that affect correctness, security, or data integrity.
- Repeated harness or spawn failures after a single targeted retry where applicable.
- User or operator requests to stop.
- Missing consent for the **next** expansion step (do not assume prior green light covers new scope).

Document what was rolled back and what remains enabled.

---

## 7. Evidence and reporting

For ACP-backed steps, retain the evidence pattern already required in `SCOPE.md` (`ACP_RUNTIME`, `ACP_AGENT_ID`, `ACP_RESULT`, `ACP_ERROR` when failed).

For canary expansion threads in general, a short recap after each step helps auditability:

- Cohort before → after.
- Gates checked (pass/fail).
- Decision: expand, hold, or roll back.

---

## 8. Review and promotion

- This file is a **draft** until Dominic (or the owning human) marks it adopted (e.g. note in `SCOPE.md` or removal of “draft” here).
- After real rollouts, revise gates and time-between-steps based on what actually broke or what was too slow.

---

*Draft created for branch `cursor/canary-expansion-policy-draft-8172`. Revise dates and owners as the system evolves.*

# Canary expansion policy (draft)

**Status:** draft — not final policy. Revise after live use; supersede with a dated version when promoted.

**Scope:** This document describes how to **expand** what automated agents (including Eve and ACP-spawned harnesses) may do **in small, reversible steps**, with explicit gates and evidence. It does not replace consent rules in `SCOPE.md`; it structures *how* to widen pre-agreed envelopes safely.

---

## 1. Purpose

- Reduce surprise and blast radius when increasing autonomy, tool use, or deployment frequency.
- Make each expansion **observable** (metrics, logs, evidence blocks) so regressions trigger rollback or freeze.
- Keep human judgment at **expansion boundaries**, not inside every micro-step once a slice is approved.

---

## 2. Definitions

| Term | Meaning |
|------|--------|
| **Canary slice** | A narrowly scoped capability or context (e.g. one repo, one workflow, one harness, read-only) enabled for a trial period. |
| **Expansion** | Moving from slice *N* to *N+1* after gates pass (time in market, error budget, operator sign-off). |
| **Blast radius** | Worst credible impact if the slice misbehaves (wrong repo, prod deploy, credential leak, data loss). |
| **Freeze** | Halting further expansion and optionally revoking the newest slice until root cause is understood. |

---

## 3. Principles

1. **Default deny for new surfaces.** A new action class (new harness, new deploy surface, new branch policy) starts disabled or behind an explicit flag until it passes a canary slice.
2. **One variable at a time.** Do not widen scope and duration and traffic in the same change; isolate what you are testing.
3. **Evidence before scale.** Require the same evidence contracts already used elsewhere (e.g. ACP spawn evidence in `SCOPE.md`) before increasing volume or removing human checkpoints.
4. **Reversibility first.** Prefer feature flags, registry entries, and small commits over irreversible migrations during canary.
5. **Operator remains accountable.** Scripts such as `eve-deploy-gha-safe` and `eve-deploy-vercel-safe` model this: `--approved-by-operator` is mandatory; canary expansion does not remove that line for high-impact actions unless a separate written policy and tooling say so.

---

## 4. Expansion gates (checklist)

Before promoting a slice from trial to “standard operating mode,” confirm:

- [ ] **Scope written:** exact repos, paths, harnesses, workflows, or APIs included; everything else explicitly out of scope.
- [ ] **Success criteria:** e.g. error rate threshold, latency, zero critical incidents over *N* runs (choose *N* for the slice’s risk).
- [ ] **Monitoring:** where logs go; who reads them; what triggers a page or a human ping (even if informal).
- [ ] **Rollback:** how to disable the slice (config revert, registry removal, token rotation if needed) in one documented step.
- [ ] **Time box:** trial end date or max invocations; auto-freeze if exceeded without review.
- [ ] **Green light:** aligns with `SCOPE.md` — material automation and high-impact actions still require explicit approval unless this policy (once final) and Dominic’s written instructions define a narrower pre-approved envelope.

---

## 5. Suggested tiers (example ladder)

Use as a template; adjust per environment.

| Tier | Typical slice | Human involvement |
|------|----------------|-------------------|
| 0 | Read-only investigation, no mutations | Confirm periodically |
| 1 | Commits / PRs in a designated sandbox repo | Green light per initiative or weekly |
| 2 | ACP spawns with fixed `cwd` + harness map (`SCOPE.md`) | Evidence block on every spawn; spot-check |
| 3 | Deploy via guarded scripts + registry (`bin/eve-deploy-*-safe`) | `--approved-by-operator` each time unless a future policy explicitly delegates |
| 4 | Broader repo list or prod-adjacent workflows | Separate written approval + monitoring SLO |

Promotion between tiers should not skip tiers without documenting why the risk is acceptable.

---

## 6. Rollback and freeze triggers

Initiate **rollback** (revert slice) or **freeze** (no further expansion) when any of the following occur:

- Repeated spawn or auth failures beyond the retry policy in `SCOPE.md` / `TOOLS.md`.
- Deploy to an unintended target, or dispatch of an unapproved workflow.
- Leakage of secrets in logs or agent output.
- Operator loses confidence or context (personnel change, stale runbooks).

Document the incident in daily notes (`memory/YYYY-MM-DD.md` when used) and update this draft with lessons before re-enabling.

---

## 7. Related workspace material

- `SCOPE.md` — mandate, ACP spawn contract, consent, and success criteria.
- `TOOLS.md` — ACP harness map, fallbacks, Cursor ACP runbook.
- `bin/eve-deploy-gha-safe`, `bin/eve-deploy-vercel-safe` — operator-gated deploy patterns.

---

*Draft started: 2026-04-28*

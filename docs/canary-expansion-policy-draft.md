# Canary expansion policy (draft)

**Status:** Draft — not final policy. Intended for review with Dominic before adoption.

**Purpose:** Define how new or risky capabilities move from a **narrow canary** to **broader use** without skipping safety gates. This reduces blast radius for automation, agent routing, deploy paths, and external harnesses (for example ACP-backed Cursor work).

---

## 1. Scope

This draft applies when expanding any of the following beyond an initial pilot:

- New or materially changed **agent behaviors** (skills, prompts, tool use).
- **ACP / harness** usage patterns or default routing (see `SCOPE.md` and `TOOLS.md` for the spawn contract).
- **Deploy or CI** paths touched by operator-approved wrappers (for example registry-gated scripts under `bin/`).
- **Production or customer-facing** outputs that were previously dry-run, shadow, or single-thread only.

Out of scope for this document: emergency break-glass incident response (handle under separate runbooks).

---

## 2. Definitions

| Term | Meaning |
|------|--------|
| **Canary** | A deliberately limited slice: one project, one channel, one operator, dry-run only, or a fixed small cohort. |
| **Expansion** | Any increase in scope: more repos, more sessions, higher autonomy, production instead of dry-run, or removal of human-in-the-loop checks. |
| **Gate** | A required condition or approval before the next expansion step. |

---

## 3. Principles

1. **Default narrow** — Start with the smallest scope that still produces signal (success/failure, latency, errors, human feedback).
2. **Explicit expansion** — Each widening step is named, logged, and reversible.
3. **Operator consent** — Material actions that match workspace rules (for example deploy scripts requiring `--approved-by-operator`) keep that bar; this policy does not relax `SCOPE.md` hard boundaries.
4. **Evidence over vibes** — Decisions to expand use observable metrics or checklists, not only subjective “seems fine.”

---

## 4. Stages (suggested ladder)

Stages are suggestions; some changes may skip stages if risk is negligible and that is documented.

1. **Design / doc** — Behavior described; failure modes considered; rollback described.
2. **Dry-run or shadow** — No external side effects, or effects are non-destructive and reversible.
3. **Single-pilot** — One project, one thread, or one operator path.
4. **Limited cohort** — Bounded list of projects, channels, or time windows.
5. **General availability** — Default path for new work of that class, with monitoring.

Expansion from stage *n* to *n+1* requires passing the gates in section 5.

---

## 5. Gates before expansion

Before each expansion step, confirm:

- **Correctness** — Pilot completed without unresolved defects tied to the change.
- **Observability** — Logs or evidence blocks exist where applicable (for example ACP runs: `ACP_RUNTIME`, `ACP_AGENT_ID`, `ACP_RESULT`, and `ACP_ERROR` when failed, per `SCOPE.md`).
- **Auth and config** — No repeated `Authentication required`, `not authenticated`, or `ACP runtime backend is not configured` without a documented fix path (`TOOLS.md` Cursor ACP runbook).
- **Operator alignment** — Dominic has approved the next stage when the change touches deploy, registry, or high-impact automation (per existing “green light” expectations in `SCOPE.md`).

Optional but recommended for non-trivial changes:

- **Time in canary** — Minimum calendar window or minimum successful runs before widening (set per initiative).
- **Error budget** — Abort or hold expansion if error rate or severity crosses a preset threshold.

---

## 6. Stop and rollback

**Halt expansion** (return to previous stage or disable the feature) if any of the following occur:

- Regressions in pilot metrics or user-reported severity.
- Auth or platform misconfiguration that cannot be resolved without host-side changes.
- Ambiguity about consent or scope (“not sure this was approved”).

**Rollback** means reverting config, disabling a flag, or narrowing routing to the last known-good canary — documented at design time (section 4, step 1).

---

## 7. Ownership and revision

- **Owner:** Dominic (and Eve as executor of documented policy once adopted).
- **Review:** Revisit this draft after the first real canary cycle; promote from draft to **approved** only by explicit decision.
- **Change control:** Updates to this file after adoption should note date and summary at the bottom of the document.

---

*Draft written for branch `cursor/canary-expansion-policy-draft-cfc4`. Last updated: 2026-04-28.*

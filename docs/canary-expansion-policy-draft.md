# Canary expansion policy (draft)

**Status:** Draft — not yet binding. Revise after first operational review.

**Scope:** How new capabilities, harnesses, automation, or workflow changes are introduced and widened without surprising load, cost, or risk.

---

## Purpose

Canary expansion means shipping a change to a **small slice** of real usage first, measuring outcomes, then **gradually increasing** exposure only when predefined gates pass. This policy drafts the rules for that expansion in Eve’s environment (OpenClaw, ACP harnesses, Cursor sessions, and Dominic-directed automation).

## Definitions

- **Canary:** The initial limited application of a change (one project, one harness, one channel, or explicit time-box).
- **Expansion:** Deliberate increase in scope after evidence — more repos, more task types, higher frequency, or broader autonomy.
- **Gate:** A check that must pass before the next expansion step. Failure stops expansion and triggers rollback or redesign.

## Principles

1. **Default to narrow.** First use matches the smallest unit that still proves the change (single `cwd`, single harness, single task class).
2. **Evidence before scale.** Each expansion step requires a short evidence note: what ran, outcome, errors, cost or latency if relevant.
3. **Human in the loop for first contact.** New external surfaces (new harness, new API, new repo class) get explicit green light per `SCOPE.md` before widening.
4. **No silent retries at scale.** Retry rules stay as documented for ACP (e.g. one retry for `target_agent_required`); expansion does not add hidden retry loops.
5. **Rollback is a first-class outcome.** If gates fail, revert to the prior baseline without debating “one more try” unless Dominic approves.

## Phases (recommended)

| Phase | Description | Typical gates |
|-------|-------------|----------------|
| **P0 — Design** | Document intent, risks, and success criteria. | Written scope; harness/agentId correctness per `TOOLS.md`. |
| **P1 — Pilot** | Single project or synthetic task; Dominic available. | Clean completion or documented failure; no auth/config surprises. |
| **P2 — Limited production** | Multiple tasks or projects, still capped (e.g. daily limit or subset of channels). | Stable for agreed window; token/cost within expectations if measured. |
| **P3 — Standard** | Default path for that capability unless tagged experimental. | Dominic sign-off; policy text updated from “draft” to “active” if adopted. |

Phases can be skipped only for **low-risk doc-only** or **config typo** fixes; anything that spawns agents, touches credentials, or changes automation defaults runs through at least P1.

## Expansion gates (minimum bar)

Before moving from P1 → P2:

- No unresolved `Authentication required` / `ACP runtime backend is not configured` class errors for the involved harness.
- `agentId` is always an acpx harness name, never `"main"` (per `SCOPE.md` / `TOOLS.md`).
- Evidence block in session output when debugging spawn issues (`ACP_RUNTIME`, `ACP_AGENT_ID`, `ACP_RESULT`, `ACP_ERROR` if failed).

Before moving from P2 → P3:

- Explicit approval (“green light”) for making this the default behavior.
- Documented owner and where to look for runbooks (`TOOLS.md` Cursor ACP section, etc.).

## What this policy does not do

- It does not replace **Dominic’s consent** for destructive, external, or high-impact actions (`SCOPE.md` hard boundaries).
- It does not define **Cursor Cloud** product canaries — only how *this workspace and orchestration* treat gradual rollout of changes Eve drives.

## Revision log

| Date | Change |
|------|--------|
| 2026-04-28 | Initial draft added on branch `cursor/canary-expansion-policy-draft-5a25`. |

---

*Next step when promoting from draft: align phase names with any org-wide release policy and add concrete metrics (e.g. max parallel spawns, daily caps) if needed.*

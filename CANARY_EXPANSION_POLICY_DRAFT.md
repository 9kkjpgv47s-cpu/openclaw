# Canary expansion policy (draft)

**Status:** Draft — not operational law until reviewed and adopted.  
**Intent:** Define how new or higher-risk automation (especially Cursor ACP spawns and similar harness work) is rolled out from a narrow slice to broader use without bypassing consent or breaking the spawn contract.

## Scope

This policy applies when:

- Turning on a new integration path (for example acpx harnesses beyond the one already proven).
- Widening *who* or *what* may trigger automated project work (more task types, more repos, less human pre-check).
- Changing retry, auth, or evidence rules that affect safety or debuggability.

It does not replace `SCOPE.md` (mandates, consent, ACP contract) or `TOOLS.md` (runbooks). It sits on top: *how fast* and *how wide* we expand after the technical contract is correct.

## Principles

1. **Contract first** — Expansion assumes the ACP spawn contract in `SCOPE.md` is satisfied (`runtime: "acp"`, correct harness `agentId`, `cwd`, `task`, evidence block). No canary phase should ship with `agentId: "main"` or missing harness mapping.
2. **Consent preserved** — `SCOPE.md` requires explicit approval for automating, pushing, or material action. Canary expansion widens *eligibility to propose or route* work; it does not silently grant green light.
3. **Observable** — Each phase keeps the evidence block (`ACP_RUNTIME`, `ACP_AGENT_ID`, `ACP_RESULT`, `ACP_ERROR` when failed) so failures are attributable.
4. **Reversible** — Any phase can roll back to the previous phase without a config migration; document the rollback lever (feature flag, routing rule, or human-only step).

## Phases (recommended)

| Phase | Traffic / scope | Gate to advance |
|--------|------------------|-----------------|
| **Shadow** | No user-visible automation change; dry runs or logging-only evaluation of routing decisions. | Spawn contract verified in staging or manual `/acp doctor` + one successful harness smoke per target harness. |
| **Canary** | A small, fixed set of task types or repos (or a percentage cap, if your platform supports it). | N successful end-to-end runs with no auth/plugin misconfig; errors handled per `TOOLS.md` fallback rules. |
| **Expanded** | Most routine Cursor/project work eligible for spawn when policy allows. | Sustained period without incident class “wrong agentId” or “silent retry storm”; Dominic explicit sign-off. |
| **Default** | Same as today’s intended steady state after adoption; only exceptional cases bypass the harness path. | Policy promoted out of draft; `SCOPE.md` / runbooks updated if any rule changed. |

Names and percentages are placeholders; the owner fills concrete gates (for example “10 spawns” or “one week”) when adopting the draft.

## Expansion checklist (before widening)

- [ ] Harness map in `TOOLS.md` matches the harnesses actually enabled on the host.
- [ ] `target_agent_required` retry behavior tested once per new harness.
- [ ] Auth and “do not retry” error classes documented and rehearsed (no silent loops).
- [ ] Rollback: single switch or documented procedure to return to prior phase.

## Review

When this draft is accepted, either merge its essence into `SCOPE.md` or keep this file and mark status **Adopted** with a date. Until then, treat all expansion decisions as requiring the same green-light bar as in `SCOPE.md` § Hard Boundaries.

---

*Draft for branch `cursor/canary-expansion-policy-draft-21db`. Last updated: 2026-04-28.*

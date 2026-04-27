# Canary expansion policy (draft)

**Status:** Draft — not binding until promoted through the gates below.  
**Intent:** Describe how new or revised policies move from paper to practice without surprising people or widening blast radius too fast.

---

## Definitions

- **Policy draft** — A written rule, workflow change, or boundary update that is proposed but not yet the operational default for everyone and every context it will eventually cover.
- **Canary cohort** — The smallest set of actors, channels, projects, or time windows where the draft is *actively* applied on purpose, with explicit consent and a defined observation period.
- **Expansion** — Adding cohorts, scopes, or duration while keeping rollback and feedback loops intact. Expansion is deliberate; it is not “silent default for all.”

---

## Principles

1. **Write before widen** — The draft exists as a single source of truth (this file or a linked doc) before any canary runs.
2. **Opt-in canaries** — Cohort members know they are in the canary and what success/failure looks like.
3. **One variable at a time** — Prefer changing policy *or* tooling *or* scope in a single step; combine only when the interaction is itself what you are testing.
4. **Evidence over vibes** — Advance phases on observations (incidents, misses, friction, metrics if any), not only on calendar time.
5. **Fast rollback** — Any phase must be reversible without a heroic effort (revert doc default, toggle behavior, or narrow cohort again).

---

## Phases (draft → production)

| Phase | Who / what | Goal |
|--------|------------|------|
| **D0 — Draft** | Author + reviewer(s) | Text is complete enough to argue with; no automatic behavior change. |
| **D1 — Paper canary** | Same as D0 + explicit “if we did this tomorrow” walkthrough | Dry-run scenarios; list failure modes and owner for each. |
| **C1 — Live canary (minimal)** | Smallest cohort; shortest window that still yields signal | Validate the policy under real load; collect feedback and incidents. |
| **C2 — Expanded canary** | Add cohorts or contexts one slice at a time | Confirm stability across the dimensions that matter (e.g. second harness, second project, second comms channel). |
| **GA — General adoption** | Default for all contexts the policy was written for | Update `SCOPE.md`, `AGENTS.md`, `TOOLS.md`, or the relevant skill so the default matches GA. Archive or supersede older text. |

Skipping phases is allowed only when the change is trivial and reversible in under minutes *and* blast radius is negligible; note the skip and rationale in the promotion record.

---

## Gates (checklist before each promotion)

Before **C1**:

- [ ] Draft reviewed for conflicts with `SCOPE.md` (consent, destructive actions, external impact).
- [ ] Canary cohort named (people, agents, repos, or time box).
- [ ] Success criteria written in one short paragraph.
- [ ] Rollback step is one concrete action (e.g. “revert commit”, “remove section”, “restore previous default in X”).

Before **C2**:

- [ ] C1 retrospective: what broke, what confused, what to tighten in the doc.
- [ ] Explicit list of *new* slices being added (not “everyone” unless C1 was already org-wide).

Before **GA**:

- [ ] Defaults in the living docs/skills updated to match GA wording.
- [ ] Stakeholders who are affected but were not in the canary have been notified or the change is documented where they already look.

---

## Promotion record (fill in per policy)

Use a short table or bullet block when a draft moves phases:

```text
Policy: <name>
D0 date: YYYY-MM-DD
C1 start/end: … / … cohort: …
C2 start/end: … cohort additions: …
GA date: … superseded: <none|link>
Owner: <name>
Notes: <one line>
```

---

## Relationship to other workspace rules

- **Green light / consent** (`SCOPE.md`): A canary is not a loophole. Material automation, pushes, or high-impact actions still need explicit approval unless the GA policy itself has been updated to say otherwise.
- **Living documents**: When this draft graduates to GA, fold the operational essentials into the smallest set of files people already read (`SCOPE.md`, skills, `TOOLS.md`) and keep this file as history or delete redundant copies—pick one obvious home for the normative text.

---

*Last updated: 2026-04-27*

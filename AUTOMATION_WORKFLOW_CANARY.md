# Automation workflow — canary expansion

Companion to **`SCOPE.md`** (automation workflow brief + canary rules). This file focuses on **external** automation surfaces: GitHub Actions, workflow dispatch, scheduled jobs, deploy hooks, and other CI/CD paths that can widen blast radius quickly.

Operational playbook for rolling out **automation workflows** (new or changed pipelines, dispatches, integrations, or agent-triggered runs) in **canary** stages so failures stay contained and rollback is obvious.

## Definitions

- **Automation workflow**: anything that runs or deploys without a human in the loop for each step — e.g. `workflow_dispatch`, `schedule`, `push`-to-main deploys, release automation, or scripted wrappers such as `bin/eve-deploy-gha-safe` that call the Actions API.
- **Canary**: a deliberately **narrow** slice (one workflow, one repo, one ref, dry-run first, or internal-only notifications) where new behavior runs **before** general availability.
- **Expansion**: widening that slice only after **predefined gates** pass.

## Canary cohort (start narrow)

Pick **one** dimension to limit first; expand one dimension at a time when gates pass.

| Dimension | Example canary |
|-----------|----------------|
| Workflow | Single file in `.github/workflows` (e.g. one smoke or install workflow). |
| Repo / slug | One `githubActions.repository` from the deploy registry; no cross-repo dispatch until proven. |
| Ref / branch | Feature branch or explicit `--ref`; avoid default-branch dispatch until validated. |
| Depth | `eve-deploy-gha-safe --dry-run` and metadata checks only; then real dispatch with operator approval. |
| Notifications | Internal channel or DM only for workflow failures until noise is acceptable. |

Document the active cohort in one line wherever you track runbooks (e.g. “Canary: `install-smoke.yml` on `main` for `org/app` only”).

## Gates before each expansion

All must be true before widening the cohort, adding workflows, or deepening automation:

1. **Correctness**: runs succeed on the canary window (or known flakes are documented); no surprise production deploys or wrong-environment targets.
2. **Permissions**: tokens and registry entries match least privilege; no dispatch to repos outside approved registry scope.
3. **Observability**: each run is attributable (workflow run URL, commit, optional run id); logs or summaries are easy to find.
4. **Rollback tested**: disabling the workflow, branch protection, or flag returns behavior to pre-canary state without a risky manual scramble.

If any gate fails, **hold expansion**; shrink cohort or revert behavior, then fix root cause.

## Expansion order (suggested)

1. **Dry-run and read-only checks** — validate workflow exists, payload/ref, and registry mapping (`--dry-run` where supported).
2. **Single workflow, low-risk event** — e.g. smoke on a non-protected ref or manual `workflow_dispatch` only.
3. **Default branch or scheduled cadence** — only after stable runs and explicit policy for who may trigger.
4. **Broader automation** — matrix repos, auto-merge deploy chains, or agent-initiated dispatch only with explicit operator policy and kill switch.

Skip steps only if risk analysis shows the next step is strictly lower risk than the prior (rare).

## Eve / OpenClaw alignment

- **Consent**: material automation (dispatch, deploy, repo writes) stays behind explicit green light per `SCOPE.md`; `eve-deploy-gha-safe` already requires `--approved-by-operator`.
- **Registry**: deploy helpers assume `mobile-repo-deploy-registry.json` and `deploySurfaces` including `github-actions` — do not bypass or point at ad-hoc paths without updating policy.
- **ACP**: investigation or fixes that need harness work follow the ACP contract and evidence block in `SCOPE.md` / `TOOLS.md`.
- **Escalation**: auth or API errors — stop, full error text to Dominic; no silent retry beyond documented single `target_agent_required` retry for ACP.

## Kill switch

Maintain a single documented switch (disable workflow in UI, remove schedule, revoke token scope, or turn off webhook) that **immediately** stops canary automation without waiting for a code deploy, where the platform allows it.

---

*Living doc — revise cohort, gates, and expansion order as pipelines and risk posture evolve.*

# Incident triage — canary expansion

Companion to **`SCOPE.md`** (incident triage brief + agent canary rules). This file focuses on **external** incident tooling (pages, tickets, integrations, cohorts).

Operational playbook for rolling out **incident triage** (automated or agent-assisted classification, routing, and first-response drafting) in **canary** stages so blast radius stays small and rollback is obvious.

## Definitions

- **Incident triage**: intake → severity/priority → owner or channel → initial summary and next steps; may include spawning Cursor/ACP for technical follow-up.
- **Canary**: a deliberately **narrow** slice of traffic, tenants, repos, or alert sources where new triage behavior runs **before** general availability.
- **Expansion**: widening that slice only after **predefined gates** pass.

## Canary cohort (start narrow)

Pick **one** dimension to limit first; expand one dimension at a time when gates pass.

| Dimension | Example canary |
|-----------|----------------|
| Source | Single integration (e.g. one PagerDuty service, one Slack channel, one webhook). |
| Severity | Only `SEV-3` / non-customer-facing, or only synthetic test incidents. |
| Tenant / customer | Internal accounts or a named allowlist. |
| Automation depth | Summarize + route only; no auto-close, no auto-page, no repo writes. |

Document the active cohort in one line wherever you track runbooks (e.g. “Canary: `#incidents-pilot` + SEV-3 only”).

## Gates before each expansion

All must be true before widening the cohort or deepening automation:

1. **Error budget**: no increase in false routes, missed pages, or customer-visible mistakes vs baseline for the canary window (define window length up front, e.g. 5 business days).
2. **Latency**: triage assist does not delay human acknowledgment beyond an agreed SLA (e.g. first human touch still within N minutes for SEV-1/2).
3. **Observability**: every canary action is attributable (run id, agent/session id, rule version) and logs are reviewable.
4. **Rollback tested**: disabling the canary flag or routing rule returns behavior to pre-canary state without redeploy drama.

If any gate fails, **hold expansion**; shrink cohort or revert behavior, then fix root cause.

## Expansion order (suggested)

1. **Read-only assist** — draft summary + suggested severity in thread; human sends the page or ticket.
2. **Routing only** — auto-assign queue or channel from explicit rules; human still approves pages.
3. **Wider sources** — add integrations one at a time; repeat gates per source.
4. **Deeper automation** — auto-ticket creation, runbooks, or ACP spawns only with explicit operator policy and kill switch.

Skip steps only if risk analysis says the next step is strictly lower risk than the prior (rare).

## Eve / OpenClaw alignment

- **Consent**: material automation (pages, tickets, pushes) stays behind explicit green light per `SCOPE.md`.
- **ACP**: any `sessions_spawn` for investigation follows the harness contract and evidence block in `SCOPE.md` / `TOOLS.md`.
- **Escalation**: auth or runtime errors (`Authentication required`, `ACP runtime backend is not configured`, etc.) — stop, full error text to Dominic; no silent retry beyond the documented single `target_agent_required` retry.

## Kill switch

Maintain a single documented switch (feature flag, routing rule, or “disable webhook”) that **immediately** stops canary triage automation without waiting for a code deploy, where your platform allows it.

---

*Living doc — revise cohort, gates, and expansion order as tooling and risk posture evolve.*

# Canary expansion policy (draft)

**Status:** draft — not operational until promoted out of draft and adopted explicitly.

**Purpose:** Define how new Eve capabilities, workflows, and external integrations move from narrow trial to broader use without bypassing consent, safety checks, or observability.

---

## Definitions

- **Canary:** A deliberately small slice of traffic, scope, or privilege where a change runs first (one project, one harness, one channel, time-boxed window, or “read-only only”).
- **Expansion:** Increasing that slice (more repos, more spawn types, higher autonomy, longer duration, or production-adjacent actions) only after defined gates pass.
- **Policy draft:** This document; changes here do not by themselves authorize behavior. **Green light** from Dominic and updates to living docs (`SCOPE.md`, `TOOLS.md`, skills) remain the source of truth for what Eve may do today.

---

## What this policy governs

Use this framework when introducing or widening any of the following:

- New or additional ACP harnesses, runtimes, or spawn patterns (beyond the current contract in `SCOPE.md`).
- New tools, APIs, deploy paths, or automation that can change state outside the workspace.
- Higher-frequency or lower-human-touch operation (e.g. more daily pushes, broader cron/heartbeat actions).
- New data paths (logging, telemetry, third-party services).

Routine work inside an already-approved harness and scope is **not** a canary event; widening *that* contract is.

---

## Phases (recommended)

1. **Design / draft** — Capture intent, risks, rollback, and success signals in writing (this file or a short addendum). No production expansion.
2. **Canary** — Minimal exposure: one clear boundary (e.g. single `cwd`, single task type, dry-run or read-only where applicable). Duration and volume are intentionally capped.
3. **Expanded canary** — Widen one dimension at a time (e.g. more projects *or* more autonomy, not both at once). Keep evidence and human visibility the same or better than phase 2.
4. **Default (embedded)** — Behavior is folded into `SCOPE.md` / `TOOLS.md` / relevant skills so future sessions inherit it without special-case instructions.

Skipping phases is allowed only for **reversible, low-blast-radius** changes (documentation-only, typo fixes, purely local read-only checks). Anything with credentials, spend, or external side effects should run through at least **design + canary**.

---

## Gates before expansion

Each gate should be satisfied before moving to the next phase.

| Gate | Canary → expanded | Expanded → default |
|------|---------------------|----------------------|
| **Observability** | Evidence blocks or logs exist for the new path (e.g. ACP evidence per `SCOPE.md`). | Same, with no unexplained failures over the agreed window. |
| **Human review** | Dominic explicitly approves widening (“green light” per `SCOPE.md`). | Explicit sign-off that the behavior is baseline, not experimental. |
| **Rollback** | Document how to disable (env flag, skill revert, spawn stop). | Rollback tested once or documented as trivial (revert commit / disable harness). |
| **Blast radius** | Worst case is bounded (single repo, no prod deploy, etc.). | Worst case documented and accepted for ongoing operation. |

If a gate fails: **contract scope** (narrow or pause), fix, re-run canary — do not “expand through” failures.

---

## Alignment with existing rules

- **Consent:** `SCOPE.md` requires explicit approval before material action; canary expansion does not replace that — it structures *how* to ask and *what* to prove before the next ask.
- **ACP contract:** Harness names, retries, auth errors, and evidence blocks remain mandatory; new harnesses enter at **design** then **canary** with the same contract shape unless deliberately extended in writing.
- **No silent retries** on auth/backend misconfiguration: unchanged; a failed canary gate is a stop, not a retry loop.

---

## Revision and promotion

- **Draft:** Edit this file freely; note the date at the bottom when meaningfully changed.
- **Promotion:** When Dominic adopts the policy, (1) mark status as **active** or merge into `SCOPE.md`, (2) remove or narrow “draft” wording, (3) keep a short changelog entry or git history for audit.

---

## Open questions (for future revision)

- Numeric thresholds (e.g. minimum clean runs, calendar window) — leave situational until there is enough operational history.
- Whether certain workstreams (e.g. security triage vs. feature dev) get different default canary widths.
- How this intersects with CI/deploy “safe” wrappers in `bin/` — treat new wrappers as governed capability changes.

---

*Draft policy — last updated: 2026-04-28*

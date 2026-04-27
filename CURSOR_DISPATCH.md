# Cursor dispatch — diagnosis

This workspace configures **Eve → OpenClaw ACP (acpx) → Cursor** for project work. “Cursor dispatch” means: a `sessions_spawn` call with `runtime: "acp"` and the correct harness so acpx starts `cursor-agent acp` in the target `cwd`.

## Dispatch path (happy path)

1. Caller invokes `sessions_spawn` with `runtime: "acp"`, `agentId: "cursor"`, absolute `cwd`, and a clear `task`.
2. OpenClaw’s acpx runtime resolves harness `"cursor"` → spawns `cursor-agent acp`.
3. The Cursor CLI uses **`CURSOR_API_KEY`** (Cursor Cloud Agents API key), not the Grok key used for Eve’s reasoning.
4. Reply should include the evidence block from `SCOPE.md` (`ACP_RUNTIME`, `ACP_AGENT_ID`, `ACP_RESULT`, and `ACP_ERROR` if failed).

## Symptom → cause → fix

| Symptom | Likely cause | Action |
|--------|----------------|--------|
| `Failed to spawn agent command: main` | `agentId` was `"main"` (OpenClaw primary id, not an acpx harness). | Use `agentId: "cursor"` for Cursor work. See harness map in `TOOLS.md`. |
| `target_agent_required` | Missing or wrong harness on first attempt. | Retry **once** with `"cursor"` (or the correct harness for the task type). Report outcome. |
| `Authentication required` / `not authenticated` / `login required` | `CURSOR_API_KEY` missing from the **service** environment that spawns acpx. | Set key where OpenClaw/systemd/launchd reads it; `cursor-agent status` on the host. Do **not** retry blindly. |
| `ACP runtime backend is not configured` / `Install and enable the acpx runtime plugin` | acpx not installed or not enabled. | `openclaw plugin install acpx` / `enable`; or verify embedded acpx with `/acp doctor`. Do **not** retry blindly. |
| Confusion between API keys | Grok key used for Cursor. | Keep keys separate: Grok → Eve LLM; `CURSOR_API_KEY` → Cursor harness only. |

## Quick verification (host)

```bash
which cursor-agent && cursor-agent --version
cursor-agent status
```

In an Eve session: `/acp doctor` then `/acp spawn cursor --cwd <absolute-project-path>`.

## Canonical references

- **Spawn JSON shape and harness list:** `TOOLS.md` (“ACP Spawn Template”, “Harness map”, “Cursor ACP runbook”).
- **Mandatory contract and evidence block:** `SCOPE.md` (“ACP Spawn Contract”).

## Historical root cause (fixed in contract docs)

The original failure chain was: wrong or missing runtime → **`agentId: "main"`** (not a harness) → after correction, **missing `CURSOR_API_KEY`** in the daemon environment. Both are addressed by the contract in `SCOPE.md` and the runbook in `TOOLS.md`.

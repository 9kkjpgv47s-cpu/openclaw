# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff that's unique to your setup.

## What Goes Here

Things like:

- Camera names and locations
- SSH hosts and aliases
- Preferred voices for TTS
- Speaker/room names
- Device nicknames
- Anything environment-specific

## Examples

```markdown
### Cameras

- living-room → Main area, 180° wide angle
- front-door → Entrance, motion-triggered

### SSH

- home-server → 192.168.1.100, user: admin

### TTS

- Preferred voice: "Nova" (warm, slightly British)
- Default speaker: Kitchen HomePod
```

## Why Separate?

Skills are shared. Your setup is yours. Keeping them apart means you can update skills without losing your notes, and share skills without leaking your infrastructure.

---

Add whatever helps you do your job. This is your cheat sheet.

## ACP Spawn Template (all harnesses)

Use this exact shape for ACP spawns. The `agentId` is the **acpx harness name** (not an OpenClaw agent id).

```json
{
  "name": "sessions_spawn",
  "arguments": {
    "runtime": "acp",
    "agentId": "<harness>",
    "cwd": "<project-absolute-path>",
    "task": "<clear implementation task>"
  }
}
```

### Harness map (acpx built-ins)

Pick `agentId` by task type:

- Cursor work → `"cursor"` (spawns `cursor-agent acp`)
- Codex work → `"codex"` (spawns `npx @zed-industries/codex-acp`)
- Claude Code work → `"claude"` (spawns `npx -y @zed-industries/claude-agent-acp`)
- Gemini CLI work → `"gemini"` (spawns `gemini --acp`)
- OpenCode work → `"opencode"` (spawns `npx -y opencode-ai acp`)
- Copilot work → `"copilot"` (spawns `copilot --acp --stdio`)
- OpenClaw-on-OpenClaw → `"openclaw"` (spawns `openclaw acp`)

Never use `"main"` — that's OpenClaw's internal primary-agent id, not an acpx harness, and acpx will fail with `Failed to spawn agent command: main`.

### Fallback rules

- `target_agent_required` → retry once with the correct harness name for the task type. Report the retry result.
- `Failed to spawn agent command: <name>` where `<name>` is not in the harness map above → the caller sent the wrong `agentId`. Re-send with the correct harness name.
- `ACP runtime backend is not configured` or `Install and enable the acpx runtime plugin` → see the Cursor ACP runbook below. Do **not** retry; surface to Dominic.
- `Authentication required`, `not authenticated`, `login required` → per-harness auth issue. For Cursor, see the runbook below. For other harnesses, auth is harness-specific (e.g. `gemini auth login`, `claude /login`, `opencode auth`). Do **not** retry silently; surface to Dominic with the full error text.

## Cursor ACP runbook (host-side, run on Dominic's device)

These are one-time setup steps on the machine where OpenClaw/Eve runs. Eve cannot run these herself from her config repo — Dominic runs them on his device.

> **Grok vs Cursor — don't conflate them.** The Grok API key powers Eve's LLM reasoning inside OpenClaw. It is **not** valid for Cursor's ACP harness. `cursor-agent acp` authenticates against Cursor's own backend and needs a separate **Cursor API key** (`CURSOR_API_KEY`). Both keys coexist.

### 1. Install the acpx runtime plugin in OpenClaw

```bash
# From OpenClaw's plugin directory (or via OpenClaw UI → Plugins):
openclaw plugin install acpx
openclaw plugin enable acpx
openclaw plugin list | grep acpx   # confirm enabled
```

Newer OpenClaw builds embed acpx directly (PR #61319). In that case the plugin is already present — verify with `/acp doctor` inside an Eve session.

### 2. Install the Cursor CLI (`cursor-agent`)

Follow Cursor's official CLI install instructions. After install, `cursor-agent --version` should work from a fresh shell.

Quick check from the host:

```bash
which cursor-agent
cursor-agent --version
```

(Based on prior error progression — "Failed to spawn agent command: main" advancing to "Authentication required" — `cursor-agent` is already on PATH on Dominic's host. If the `which` fails, install it now.)

### 3. Authenticate Cursor CLI for headless use

Generate an API key: **Cursor Dashboard → Cloud Agents → Create API key**.

Export in the shell / environment where OpenClaw spawns acpx from (not just an interactive shell — OpenClaw's service env must see it):

```bash
export CURSOR_API_KEY="<paste-key-here>"
```

Persist it in whichever of these OpenClaw reads on your host:

- macOS launchd agent `EnvironmentVariables` block, or
- systemd unit `Environment=CURSOR_API_KEY=...`, or
- OpenClaw's own `~/.openclaw/env` / plugin env config, or
- `~/.acpx/config.json` under the `cursor` agent's env block.

Verify:

```bash
cursor-agent status     # should show authenticated
```

#### Security status canary (with retry)

Use `cursor-agent status` as a lightweight **security status canary** before depending on Cursor ACP or Cloud Agents from automation (Eve, scripts, or orchestrators).

- **Pass:** Output indicates the CLI is installed and authenticated (per Cursor’s current status text).
- **Retry (transient only):** If the command fails or times out with clearly **transient** conditions (network timeout, connection reset, DNS blip, or upstream HTTP-style errors such as 502/503/429 when surfaced in stderr), rerun the same check up to **3 times** with exponential backoff: **4s, then 8s, then 16s** between attempts. Report each attempt and the final outcome to the operator.
- **Do not retry:** Same classes as the ACP fallback rules above — `Authentication required`, `not authenticated`, `login required`, `ACP runtime backend is not configured`, missing CLI (`command not found`), or other **configuration/auth** errors. Stop after the first failure and follow the Cursor ACP runbook; fix the root cause instead of retrying.

This canary is a preflight signal only; it does not replace explicit operator approval for deploys or other high-impact actions.

Alternative (browser-based, not useful for Eve since she's headless): `agent login`.

### 4. Smoke test from OpenClaw

In an Eve session:

```
/acp doctor
/acp spawn cursor --cwd <some-project-path>
```

Expected: a Cursor ACP session comes up; no `Authentication required`.

### 5. Verify the evidence block

When Eve runs a real `sessions_spawn` with `runtime: "acp"` + `agentId: "cursor"`, she must emit:

```
ACP_RUNTIME: acp
ACP_AGENT_ID: cursor
ACP_RESULT: spawned
```

If any of the error classes in the fallback rules appears, step back through this runbook.

### Common pitfalls

- **`Failed to spawn agent command: main`** — `agentId: "main"` was sent. Use the harness map above (`"cursor"`, `"codex"`, etc.). This was the original root cause of Eve's failure.
- **`Authentication required`** — `CURSOR_API_KEY` isn't set in the environment OpenClaw spawns acpx from. Having it in your interactive shell is not enough — OpenClaw's service env must see it.
- **`ACP runtime backend is not configured`** — acpx plugin isn't installed or isn't enabled. Run step 1.
- **Using Grok API key for Cursor auth** — won't work. Grok drives Eve's reasoning; Cursor needs its own key. Keep them separate in your env.

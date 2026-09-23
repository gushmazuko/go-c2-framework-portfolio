# T1059 — Command and Scripting Interpreter

**Range surface:** every `exec <cmd>` task and every shell-backed
scenario. The server audit log tags these `T1059`.

## What the framework does

The agent spawns `cmd.exe`, `powershell.exe` (or `sh` on non-Windows dev
targets) as a direct child of `rat-agent.exe`, streams stdout/stderr back
as `command_output` chunks, and reports the exit code.

## What to alert on

- **Sigma** `90120763-6b76-4850-aedf-a6a2d57e7d28`
  (2026-c2-agent-spawns-shell.yml): shell process with
  `ParentImage = rat-agent.exe`. The parent link is the detection —
  operator-driven shells come from explorer/terminal ancestry.
- Correlate the command line with the audit log: the exact command text
  the server sent is in `data/c2server.db` (`tasks.command`), so you can
  score per-command visibility, not just "something ran".

## Host artifacts

- Sysmon EID 1: `cmd.exe` / `powershell.exe` with agent parent, full
  command line, hashes.
- Console-host chains: ConPTY shells additionally spawn a headless
  conhost (see T1059.003 playbook).
- Exit codes arrive at the server; a task that "worked" on the endpoint
  always has a matching audit row.

## False-positive profile

- Low by construction: nothing legitimate on an endpoint has
  `rat-agent.exe` as a parent. Wrapper-script deployments would — keep
  the deployment path pinned (`C:\\labs`).
- The shell itself may spawn children with the shell (not the agent) as
  parent — hunt the agent-parent link first, then pivot on children.

## Validation

1. `interact <session>` → `exec whoami` on the range VM.
2. Expect Sigma `...spawns-shell` to fire on the `cmd.exe /c whoami`
   event; expect an audit row with `attack_tag = T1059`.
3. `exec powershell -c Get-Process` — same rule fires (T1059.001 also
   tagged in the rule).

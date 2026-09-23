# SC-02 — Persistence via scheduled task

**ATT&CK:** T1053.005 (Scheduled Task/Job: Scheduled Task)

## Objective

Establish persistence with `schtasks` through the framework shell and
verify the full telemetry chain (process + registry bookkeeping + the
task's own execution).

## Prerequisites

- Interactive session on the VM; current user in the local
  Administrators group (for machine-wide tasks).
- **Lab quirk (verified on the reference VM):** `.bat` action lines
  silently do not execute (Last Result 0, no side effects). Use a
  `powershell` or `cmd /c` one-liner as the action.

## Procedure (operator console)

```
interact <session-prefix>
shell
```

In the shell:

```
schtasks /create /tn "OneDriveSyncPinger" /tr "powershell -w hidden -c Get-Date >> C:\\labs\ping.txt" /sc minute /mo 15 /rl limited /f
schtasks /query /tn "OneDriveSyncPinger" /v
schtasks /run /tn "OneDriveSyncPinger"
type C:\\labs\ping.txt
```

## Expected host artifacts

- Sysmon EID 1: `schtasks.exe` under the agent's conhost/cmd chain.
- EID 12/13: `HKLM\...\Schedule\TaskCache\Tasks\<id>` creation and
  value sets (in the lab config's include list).
- The task's own execution: `svchost.exe (-s Schedule) → powershell.exe`
  at the configured interval — a lineage worth tracing with defenders
  (the C2 parent chain is gone; only the Schedule service remains).

## Expected detections

- `aafc7c24-3591-4fdd-8777-ddba04701ed6` (schtask-creation) on the
  `schtasks /create` process event.
- `90120763-6b76-4850-aedf-a6a2d57e7d28` (agent-spawns-shell) for the
  schtasks process itself.
- Your SIEM's "scheduled task created by office-ish process under
  user-writable path" analytics.

## Cleanup

```
schtasks /delete /tn "OneDriveSyncPinger" /f
del C:\\labs\ping.txt
```

Delete the task **before** `kill <session>`/teardown — a leftover task
is a live artifact on the range VM.

## Coverage-report row

```
SC-02 | schtasks persistence | T1053.005 | exercised | detections: <what fired>
```

# SC-03 — Persistence via Run key

**ATT&CK:** T1547.001 (Boot or Logon Autostart Execution: Registry Run
Keys / Startup Folder)

## Objective

Write a per-user Run key through the framework shell and observe the
registry-event chain it produces.

## Prerequisites

- Interactive shell session. No elevation needed (HKCU).

## Procedure (operator console)

```
interact <session-prefix>
shell
```

In the shell:

```
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v "OneDriveSyncHelper" /t REG_SZ /d "C:\\labs\notepad.exe" /f
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v "OneDriveSyncHelper" /f
```

(Use `notepad.exe` as the payload — a harmless, visible stand-in that
still proves the autostart chain without launching a second agent.)

## Expected host artifacts

- Sysmon EID 13: value set on
  `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\OneDriveSyncHelper`
  (lab config includes the Run key tree).
- EID 12: key handle creation on the same path.
- EID 1: `reg.exe` under the agent shell lineage.

## Expected detections

- `c3b6008a-78fa-4001-a7cc-9d4ffaf613a4` (run-key-persistence).
- `90120763-6b76-4850-aedf-a6a2d57e7d28` (agent-spawns-shell).
- Correlation takeaway: on its own the Run key rule is medium (software
  installers fire it constantly); **the C2 lineage turns it high** —
  demonstrate both framings in the debrief.

## Cleanup

Covered by the procedure's `reg delete` step; verify with `reg query`.

## Coverage-report row

```
SC-03 | Run key persistence | T1547.001 | exercised | detections: <what fired>
```

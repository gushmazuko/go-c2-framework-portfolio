# T1134 — Access Token Manipulation

**Range surface:** scenario SC-10 — token play via public atomics
(`Invoke-TokenManipulation`, `MakeToken`-style steps) executed through
the framework shell. The agent has no token features (scope line).

## What the range does

The operator runs token-creation or token-theft steps in the shell:
create a token from credentials (`LogonUser` + `ImpersonateNamedPipe`/
`SetThreadToken`), or duplicate/steal from another process.

## What is honestly visible

- **Classic Sysmon sees little of the API layer** (token APIs are not
  process/registry/file events). What it does see:
  - EID 1: the hosting process (`powershell.exe` under the C2 shell)
    and its command line (public atomics name themselves).
  - EID 10 ProcessAccess with `PROCESS_QUERY_LIMITED_INFORMATION`+
    `PROCESS_DUP_HANDLE`-class masks when tokens are *stolen* from
    processes.
- **Windows Security log is the primary source**: Event 4673
  (Sensitive Privilege Use — `SeAssignPrimaryTokenPrivilege`),
  4672 (special privileges assigned to new logon), 4624 type 9
  (NewCredentials logon from `MakeToken`-style steps).
- EDR: token-manipulation analytics (thread token swaps in foreign
  contexts).

## What to alert on

- Correlate EID 1 lineage (`90120763-...spawns-shell`) with 4673/4624:9
  in the Security log from the same process within the same minute.
- Sigma for the public atomic's command-line shape
  (`Invoke-TokenManipulation`) is a one-line variant of the
  spawns-shell rule; the shipped coverage here is the correlation
  recipe, not a standalone rule.

## False-positive profile

- Service accounts and management agents make type-9/4673 events
  routinely; the C2 lineage plus interactive-logon context is the
  discriminator.

## Validation

1. In the shell: run the SC-10 `MakeToken` step with lab credentials.
2. Expect Security 4624 type 9 + 4673 from the shell process; expect
   EID 10 access events if stealing from a live process.
3. `whoami` in the shell afterwards shows the spoofed user — reconcile
   with the audit log's session identity (`RANGE-VM\range-user`) to see
   the divergence the telemetry should have caught.

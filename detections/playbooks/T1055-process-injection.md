# T1055 — Process Injection

**Range surface:** scenario SC-08 — a public atomic (e.g. remote-thread
or APC injection into a sacrificial `notepad.exe`) executed through the
framework shell. The implant has no injection capability (scope line);
the range surfaces the *procedure's* telemetry.

## What the range does

The operator runs the atomic steps in the shell — a PowerShell or
compiled-LOLBin variant that opens a target process, allocates memory
in it, writes a payload, and redirects execution.

## What is honestly visible

- **Classic Sysmon sees the frame, not the technique**:
  - EID 10 ProcessAccess: injector → target with `GrantedAccess`
    masks like `0x1FFFFF`/`0x1F3FFF` (the range's lab config includes
    ProcessAccess on `notepad.exe` and powershell-sourced access
    precisely for this).
  - EID 8 CreateRemoteThread (if enabled in your config): source →
    target start address outside the target's image.
  - EID 1: the injector lineage — child of the C2 shell.
- **EDR/ETW-TI sees the technique**: `NtAllocateVirtualMemory`/
  `NtWriteVirtualMemory`/`NtProtectVirtualMemory` cross-process with
  RX/RWX flips, thread start from unbacked memory.

## What to alert on

- Sysmon EID 10 with high-access masks cross-process (the lab config
  includes this stream), EID 8 where available.
- Sigma (EDR-dependent, no classic-Sysmon rule ships here): your EDR's
  "thread start in unbacked memory" / "RXW allocation in foreign
  process" analytics — the playbook's coverage-report row is "EDR
  analytics, not Sigma".
- Lineage chain: `rat-agent.exe → powershell.exe → notepad.exe` access
  events — parent-child + access is already strong.

## False-positive profile

- Debuggers (by design — see T1622 playbook), EDRs themselves, and
  legitimate cross-process tooling (input remappers, overlays).
  `GrantedAccess` granularity plus lineage are the filters.

## Validation

1. `interact <session>` → `shell` → start `notepad.exe`.
2. Run the SC-08 injection steps.
3. Expect EID 10 (injector → notepad) with the high mask; expect the
   EDR analytic if an EDR is on the range. Record which layer fired —
   the layer-gap is the takeaway of this family.

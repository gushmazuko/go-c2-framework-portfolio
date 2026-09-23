# T1622 — Debugger Evasion (agent anti-debug)

**Range surface:** the agent itself — the one evasion family implemented
in the implant (owner scope decision 2026-09-02, finalization-plan
Revision 4). Validated by scenario SC-12 with the range's own test
debugger `debug-probe`.

## What the agent does

`the agent anti-debug module` runs four classic user-mode checks, once
before the first connection and then every 30 seconds on either
transport:

| Check | API | Fires when |
|---|---|---|
| PEB flag | `IsDebuggerPresent` | a local debugger set `BeingDebugged` |
| debug object | `CheckRemoteDebuggerPresent` | kernel reports a debug object on the process |
| debug port | `NtQueryInformationProcess(ProcessDebugPort)` | port handle non-zero |
| debug flags | `NtQueryInformationProcess(ProcessDebugFlags)` | `NoDebugInherit` bit reads 0 |

On any positive the agent **self-exits silently** (`os.Exit(0)`): no
error path, no network indication, no window. `-antidebug=false` opts
out (lab debugging of the agent itself).

The checks only *read* OS debugger state — nothing is patched or
disabled, and no other anti-analysis exists in the implant.

## What is honestly visible

- **Classic Sysmon sees none of the API reads.** Process/handle-based
  debugger checks don't produce Sysmon events (no process create, no
  registry read, no image load — kernel32/ntdll are already mapped).
- **EDR/ETW-TI telemetry can see them**: monitored
  `NtQueryInformationProcess` calls with `ProcessDebugPort`/
  `ProcessDebugFlags` classes from a user-context process are standard
  EDR detections. If your range runs an EDR, expect its
  "anti-check / debugger query" analytics to fire *before* the agent
  exits.
- **The behavioral signature is the exit itself**: agent process
  terminates with exit code 0 immediately after a debugger attaches,
  with no operator `kill` in the audit log. Absence-of-cause is the
  tell.

## What to alert on

- **Sigma** `89ad0503-542d-499e-ab0a-3daa753b65c7`
  (2026-c2-debugger-attached-to-agent.yml): Sysmon EID 10 ProcessAccess
  by a debugger-class tool (`windbg`, `x64dbg`, `cdb`,
  `debug-probe`, …) targeting `rat-agent.exe`. This is the
  "someone is validating" signal — in production, swap the target image
  for your high-value processes and keep the debugger source list.
- EDR analytics: `NtQueryInformationProcess` debug-class queries from
  non-debugger processes.
- Audit-log cross-check: agent session drops without a server-side
  `kill` task and without transport errors.

## False-positive profile

- `IsDebuggerPresent` can read true under managed-debugger scenarios
  (script hosts, some IDE attach flows) — none of those are the agent's
  deployment context.
- The ProcessDebugFlags check inverts polarity (0 = debugged); a
  buggy implementation would constant-fire — the range's clean-run test
  (`TestCheckDebuggerCleanOnHost`) covers the negative case on every
  build.

## Validation (SC-12 short form)

1. Build `make build-debug-probe` and the agent; deploy both to the VM.
2. Start the agent, get its PID (console `sessions` → `info`, or
   `tasklist`).
3. `debug-probe.exe <pid>` — expect: probe reports attached; within
   seconds the agent exits silently; console shows the session go
   disconnected.
4. Sysmon: EID 10 with SourceImage `debug-probe.exe` → TargetImage
   `rat-agent.exe`; **no** EID 1/3 correlation for the exit (that's the
   point).
5. Negative: `-antidebug=false` agent survives the same probe (check
   that the checks are the cause, not the attach itself).

Full procedure: `scenarios/SC-12-antidebug-validation.md`.

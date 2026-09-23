# SC-12 — Anti-debug validation (T1622)

**ATT&CK:** T1622 (Debugger Evasion)

## Objective

Prove the agent's anti-debug self-exit works, prove it is *the agent's
own decision* (not debugger teardown killing it), and record what each
telemetry layer sees. This is the validation scenario for the one
evasion family implemented in the implant (owner scope decision
2026-09-02). Playbook: `detections/playbooks/T1622-anti-debug.md`.

## Prerequisites

- Fresh agent build (post-e0200f1) on the VM, `-antidebug` at its
  default (on).
- `debug-probe.exe` on the VM (`make build-debug-probe`).
- Sysmon lab config installed (EID 10 include list covers
  `rat-agent.exe` targets).

## Procedure

1. Start the agent and note its PID. From an ssh console:

```
tasklist /fi "imagename eq rat-agent.exe"
```

   Or read it from the operator console: `interact <session>` →
   `info` (agent_info reports PID at checkin).

2. Confirm the session is live (`sessions`), then attach the probe:

```
cd C:\\labs
debug-probe.exe <agent-pid>
```

3. Observe:

- The probe prints `attached to <pid> as its debugger`.
- Within seconds the agent **exits silently** (process gone in
  `tasklist`; console shows the session drop to `disconnected`).
- The probe is still alive — press Ctrl+C to detach cleanly; nothing
  else dies.

4. Sysmon check:

- EID 10: `SourceImage ...debug-probe.exe → TargetImage ...rat-agent.exe`
  (Sigma `89ad0503-542d-499e-ab0a-3daa753b65c7`).
- **No** new EID 1/3 from the agent at its death — the exit is silent
  by design; absence-of-cause is the behavioral tell.

5. **Control run (distinguish self-exit from attach effects):** start a
   fresh agent with checks disabled, attach the same probe, and watch
   it survive:

```
rat-agent.exe -antidebug=false <same flags as before>
debug-probe.exe <new-pid>
```

   Agent stays alive while probed. That contrast isolates the T1622
   checks as the cause of the death in step 3.

6. EDR (if present): record whether its anti-check analytics fired on
   the `NtQueryInformationProcess` debug-class queries — the expected
   EDR-only row; classic Sysmon cannot see those reads.

## Expected detections

- `89ad0503-542d-499e-ab0a-3daa753b65c7` (debugger-attached-to-agent).
- EDR anti-check analytics (EDR-only row).
- Audit-log note: the session drops with **no** `kill` task and **no**
  transport error — reconcile that absence in the coverage report.

## Cleanup

`taskkill /f /im debug-probe.exe` if still running. The control-run
agent gets the usual teardown (`kill <session>` or taskkill).

## Coverage-report row

```
SC-12 | anti-debug validation | T1622 | exercised | detections: <what fired>
```

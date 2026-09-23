# Detection engineering

The detection layer is the reason this platform exists as one project
instead of two: the same engineering effort that builds the C2 side
also specifies, ships and validates the defensive side against it.

## Method

1. **Map** every capability to ATT&CK with its host and network
   artifacts (`../detections/mappings.yaml`), in three tiers:
   - *framework-native* — what the implant itself does (exec, shell,
     transfers, transports, debugger checks);
   - *scenario-backed* — adversary procedures run through the shell as
     documented steps (persistence, LOLBins, staging, evasion
     procedures);
   - *documented — not exercised* — families whose artifacts cannot be
     reproduced without implementing the technique in the implant
     (sleep obfuscation, direct/indirect syscalls,
     implant-integrated injection); these ship detection playbooks and
     a stated coverage gap instead of false claims.
2. **Write detections** where they are honestly writable: 13 Sigma
   rules covering the framework's behavioral patterns (beacon
   periodicity as a Sigma 2.0 temporal correlation, process lineage,
   headless ConPTY host, transfer spool naming, debugger attach) and
   the scenario procedures (Run keys, scheduled tasks, certutil,
   ms-settings hijack, AMSI/ETW command-line markers).
3. **Ship a reference Sysmon config** tuned for validation
   (`../detections/sysmon/`) — and hardened by live canary findings:
   the config documents real Sysmon 15.21 quirks (the `<Hashes>`
   element fast-fails that binary; HKCU registry writes render as
   `HKU\<SID>\…`, which silently defeats naive rule patterns).
4. **Validate live**: run a scenario, reconcile what fired against the
   framework's own audit log (ground truth), record fired / missed /
   EDR-only per technique. The result is the
   [coverage report](../scenarios/COVERAGE-2026-09-02.md) — including
   the misses and the environment findings that caused them.

## Honest visibility boundaries

Each playbook states what classic endpoint telemetry can and cannot
see. Examples:

- In-memory AMSI/ETW patches are **invisible to process/file/registry
  telemetry** — the rules catch the string-delivery variants, and the
  playbooks score the memory/ETW-TI layer as EDR-only coverage.
- Debugger-state queries (`IsDebuggerPresent`,
  `NtQueryInformationProcess` debug classes) produce **no Sysmon
  events**; the observable chain is the ProcessAccess event of the
  debugger plus the behavioral signature: process exits with no
  identifiable cause.
- A blinded ETW provider shows up as *absence*: activity continues in
  process events while script-block logging goes quiet. The playbook
  teaches scoring the gap, not a single event.

## Validated live (excerpts)

Against a real Windows 11 host with Sysmon 15.21 and the reference
config, with the framework's audit log as ground truth:

- Beacon periodicity: 19 connection events in ~4.5 minutes, inter-poll
  deltas 14.5–15.6 s against a 15 s ± 5 % configuration — the temporal
  correlation's threshold met with ~5× headroom.
- Debugger attach (T1622 validation): ProcessAccess event
  (debugger → agent, `0x12367B`) followed by the agent's silent
  self-exit within one 30 s check cycle; control run with the checks
  disabled survived the same debugger attached.
- Process lineage: the full command-execution chain visible as
  `rat-agent.exe → cmd.exe` process-creation events.
- Environment findings that changed the content: Windows Defender
  blocked a LOLBin transfer **pre-execution** (no process event —
  recorded as a miss with cause); a UAC-bypass registry rule that
  could never fire due to the `HKU` rendering quirk — found by canary,
  fixed in the config.

## Deliverables in this repository

| Path | Content |
|---|---|
| `detections/mappings.yaml` | capability → ATT&CK mapping, three tiers, per-capability artifacts |
| `detections/sigma/` | 13 rules (incl. one Sigma 2.0 temporal correlation) |
| `detections/sysmon/` | reference Sysmon config, quirk-documented |
| `detections/playbooks/` | 13 runbooks: alert-on, false-positive profile, validation steps, visibility boundaries |
| `scenarios/` | 12 ATT&CK-referenced procedure runbooks + the live coverage report |

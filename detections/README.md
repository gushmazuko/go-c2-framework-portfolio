# detections/ — purple-team layer

Detection engineering content for the two-sided range: every framework
capability and scenario procedure is mapped to ATT&CK, documented with its
host/network artifacts, and paired with Sigma rules and a validation
runbook. Ground truth for scoring is the framework's own SQLite audit log
(`history` / `coverage` in the operator console, or read
`data/c2server.db` directly: `tasks.attack_tag`, `transfers.attack_tag`).

## Layout

| Path | Contents |
|---|---|
| `mappings.yaml` | capability → ATT&CK technique mapping (framework-native, scenario-backed, documented-not-exercised) |
| `sysmon/sysmon-lab-config.xml` | reference Sysmon config for the range VM (install: `sysmon64 -accepteula -i sysmon-lab-config.xml`) |
| `sigma/*.yml` | Sigma rules for the framework's behavioral patterns and the scenario procedures |
| `playbooks/*.md` | one runbook per technique family: what to alert on, FP profile, validation steps |

## Workflow (validate a detection)

1. Install the reference Sysmon config (`sysmon/sysmon-lab-config.xml`)
   on the target host.
2. Run the scenario or capability from the operator console.
3. Check the Sigma rule against the Sysmon event stream (SIEM or
   `sigma check` / pySigma locally).
4. Compare what fired against the scenario's declared techniques and the
   server audit log — the gap is your coverage report; three architectural
   families will always show *documented — not exercised* (see
   `playbooks/architectural-evasion-families.md`).

## Relationship to the scope line

Framework-native detection covers the implant itself. The agent's
anti-debug checks (T1622) are the one evasion family implemented in the
implant (owner decision 2026-09-02, finalization-plan Revision 4) and ship
with their own playbook, Sigma rule, validation scenario (SC-12) and test
debugger (`debug-probe`). Everything else evasion-shaped is exercised
as documented procedures through the shell, never as implant features.

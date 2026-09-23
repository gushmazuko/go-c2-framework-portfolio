# Scenarios — adversary emulation runbooks

ATT&CK-referenced procedure chains executed **through the framework
shell** in the Atomic Red Team tradition: documented, visible steps an
operator runs from the operator console — never automated implant
modules (finalization-plan §5). Every scenario declares its artifacts
and the detections that should fire, so running it end-to-end doubles
as a detection-coverage test.

## Index

| Scenario | Family | ATT&CK |
|---|---|---|
| [SC-01-discovery-chain.md](SC-01-discovery-chain.md) | host/user discovery | T1082, T1033, T1087.001, T1083 |
| [SC-02-persistence-schtasks.md](SC-02-persistence-schtasks.md) | scheduled task persistence | T1053.005 |
| [SC-03-persistence-run-key.md](SC-03-persistence-run-key.md) | Run key persistence | T1547.001 |
| [SC-04-lolbin-execution.md](SC-04-lolbin-execution.md) | LOLBin execution | T1105, T1218.005, T1218.011 |
| [SC-05-data-staging-exfil.md](SC-05-data-staging-exfil.md) | staging + exfil | T1074.001, T1105 |
| [SC-06-amsi-patching.md](SC-06-amsi-patching.md) | AMSI tampering | T1562.001 |
| [SC-07-etw-blinding.md](SC-07-etw-blinding.md) | ETW blinding | T1562.002 |
| [SC-08-process-injection.md](SC-08-process-injection.md) | process injection | T1055 |
| [SC-09-obfuscation.md](SC-09-obfuscation.md) | obfuscation | T1027, T1027.013 |
| [SC-10-token-manipulation.md](SC-10-token-manipulation.md) | token manipulation | T1134 |
| [SC-11-uac-bypass.md](SC-11-uac-bypass.md) | UAC bypass | T1548.002 |
| [SC-12-antidebug-validation.md](SC-12-antidebug-validation.md) | debugger evasion | T1622 |

## Format

Each runbook: objective → prerequisites → verbatim procedure (the
console session) → expected host/network artifacts → expected
detections (Sigma rule IDs) → cleanup → the row to copy into the
coverage report.

## Before you run any scenario

Install the reference Sysmon config (`detections/sysmon/`) on the target
host first. Running scenarios without endpoint telemetry proves nothing
except that your SIEM is blind. The validation methodology lives in
`../docs/detection-engineering.md`.

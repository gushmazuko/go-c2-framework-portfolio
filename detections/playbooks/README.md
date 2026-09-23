# Detection playbooks

One runbook per technique family: what to alert on, what the telemetry
actually is (honestly — including what classic Sysmon *cannot* see), the
false-positive profile, and how to validate the rule against this range.

| Playbook | Technique | Exercised by |
|---|---|---|
| [T1059-command-execution.md](T1059-command-execution.md) | Command and Scripting Interpreter | `exec`, every shell scenario |
| [T1059.003-conpty-shell.md](T1059.003-conpty-shell.md) | Windows Command Shell (ConPTY) | interactive `shell` |
| [T1105-file-transfer.md](T1105-file-transfer.md) | Ingress Tool Transfer | `download` / `upload` |
| [T1071-c2-transport-beaconing.md](T1071-c2-transport-beaconing.md) | App-Layer / Non-App-Layer C2, beaconing | both transports, always on |
| [T1622-anti-debug.md](T1622-anti-debug.md) | Debugger Evasion | the agent itself, SC-12 |
| [T1562.001-amsi-patching.md](T1562.001-amsi-patching.md) | Impair Defenses: Disable AMSI | SC-06 |
| [T1562.002-etw-blinding.md](T1562.002-etw-blinding.md) | Impair Defenses: ETW | SC-07 |
| [T1055-process-injection.md](T1055-process-injection.md) | Process Injection | SC-08 |
| [T1027-obfuscation.md](T1027-obfuscation.md) | Obfuscated Files or Information | SC-09 |
| [T1134-token-manipulation.md](T1134-token-manipulation.md) | Access Token Manipulation | SC-10 |
| [T1548.002-uac-bypass.md](T1548.002-uac-bypass.md) | Abuse Elevation Control: Bypass UAC | SC-11 |
| [architectural-evasion-families.md](architectural-evasion-families.md) | sleep obfuscation, syscalls, integrated injection | **documented — not exercised** |

Conventions:

- **Ground truth** is the server audit log (`the server audit store`): every task
  row carries its `attack_tag`. If a technique fired on the endpoint but
  no alert exists, that gap goes into the coverage report.
- **Rule IDs** referenced here are the `id:` fields of the Sigma rules in
  `detections/sigma/` — stable across edits.
- The three architectural families are the accepted coverage gap of this
  range (finalization-plan §6); their playbook describes what to hunt for
  in EDR/memory telemetry even though this instrument cannot produce the
  artifacts itself.

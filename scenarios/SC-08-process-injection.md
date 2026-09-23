# SC-08 — Process injection (evasion procedure)

**ATT&CK:** T1055 (Process Injection)

## Objective

Run a public injection atomic (remote-thread class) into a sacrificial
`notepad.exe` through the framework shell, and grade Sysmon-vs-EDR
visibility. The implant has no injection capability — the procedure is
the takeaway (playbook: `detections/playbooks/T1055-process-injection.md`).

## Prerequisites

- Interactive shell; a public-domain injection script
  (`Invoke-CreateRemoteThread`-class atomic) at `/tmp/inj.ps1` on the
  operator host.

## Procedure (operator console)

```
interact <session-prefix>
upload /tmp/inj.ps1 C:\\labs\inj.ps1
shell
powershell -c Start-Process notepad -PassThru | Select-Object -ExpandProperty Id
powershell -ep bypass -c Import-Module C:\\labs\inj.ps1; Invoke-CreateRemoteThread -ProcessID <notepad-pid> -Dll C:\Windows\System32\sqlite3.dll
back
```

Use a **benign signed DLL that notepad can load harmlessly** (the
`sqlite3.dll` stand-in) — the injection mechanics fire, nothing
malicious executes. If the atomic requires shellcode, use its
public-local-payload variant (pop calc is fine in a lab).

## Expected host artifacts

- EID 1: notepad start + the injector powershell, both under the agent
  shell chain.
- EID 10 ProcessAccess: injector → notepad with
  `PROCESS_CREATE_THREAD`/`0x1FFFFF`-class masks (lab config includes
  notepad targets and powershell sources).
- EID 8 CreateRemoteThread if your config enables it (not in the lab
  config by default; enable it to complete the picture).

## Expected detections

- `90120763-6b76-4850-aedf-a6a2d57e7d28` (agent-spawns-shell).
- `13bbd85b-a3c5-4bb2-9afc-be03d28a1e45` (transfer spool) for the
  script upload.
- Sysmon EID 10 high-mask access (hunt rule, not a shipped Sigma).
- EDR "thread start in unbacked memory" / "cross-process allocation"
  analytics — expected EDR-only row.

## Cleanup

Shell: `taskkill /f /im notepad.exe` then `del C:\\labs\inj.ps1`.

## Coverage-report row

```
SC-08 | process injection | T1055 | exercised | detections: <fired / EDR-only / missed>
```

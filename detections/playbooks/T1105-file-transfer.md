# T1105 — Ingress Tool Transfer

**Range surface:** `download <remote> <local>` and `upload <local>
<remote>` in the console. Audit tags: `T1105` on the `transfers` table.

## What the framework does

- **Download** (agent → server): the agent reads the file and emits
  binary `RawChunk` frames; the server spools under
  `data/transfers/<transfer-id>/` and verifies SHA-256. Works over both
  transports (beacon chunks ride the next poll's uplink).
- **Upload** (server → agent): chunked push, spooled on the endpoint as
  a hidden temp file `.rat-upload-<random>` next to the target and
  atomically renamed on completion. Persistent transport only — the
  agent rejects uploads over the https beacon by design.

## What to alert on

- **Sigma** `13bbd85b-a3c5-4bb2-9afc-be03d28a1e45`
  (2026-c2-transfer-spool-file-activity.yml):
  `TargetFilename contains .rat-upload-` or any file create by the agent.
  The hidden-dot spool name is the cleanest single indicator.
- **Sigma** `2be95732-d673-48b2-a7b7-4adb531eb4e6` correlation applies
  when transfer volume rides the beacon (burst of polls).
- Server side (out of Sysmon scope): the transfer spool tree
  `data/transfers/` on the C2 host holds every chunk spooled with its
  SHA — your ground-truth copy of what moved.

## Host artifacts

- Sysmon EID 11: `.rat-upload-*` create + rename to final name.
- Read pattern on downloaded files (bulk sequential read by the agent).
- No new processes; transfers are pure file+network activity.

## False-positive profile

- The `or selection_agent` arm fires on *any* agent file write — on the
  range VM that is intended (staging scenarios); in production scope the
  rule to the `.rat-upload-` arm.
- Bulk reads of large files look like backup agents; volume alone is a
  weak signal without the agent process context.

## Validation

1. `download C:\Windows\win.ini /tmp/x` over mTLS — expect EID 11
   read-side context, audit `transfers` row `T1105`, SHA match.
2. `upload` a small file to `C:\\labs\` — expect `.rat-upload-*`
   create + rename chain in EID 11.
3. Repeat the download with the agent in https mode: expect the same
   audit row, chunks spread over successive polls (nice demo of
   `isPersistent=false` transfer survival).

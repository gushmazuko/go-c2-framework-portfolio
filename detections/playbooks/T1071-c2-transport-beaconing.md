# T1071.001 / T1095 / T1029 — C2 transports and beaconing

**Range surface:** always on. Two transports, two very different shapes.

## The two shapes

| | mTLS persistent | HTTPS beacon |
|---|---|---|
| Technique | T1095 (custom binary protocol) | T1071.001 + T1029 |
| Connection | one long-lived TLS session | short POST `/beacon` per cycle |
| Cadence | interactive (bursty) | `sleep` ± `jitter%` (default 30s ± 20%) |
| Kills | connection drop fails in-flight transfers (spools kept for resume) | poll end is normal; transfers survive |

Both are TLS 1.3-only with a pinned CA and a client certificate — the
endpoint's network telemetry sees destination/port/cadence, not content.

## What to alert on

- **Sigma** `3a366c73-e4c3-4323-9855-eb0fcef7a690`
  (2026-c2-agent-outbound-listener-connection.yml): any connection by the
  agent — the base rule, fires for both transports.
- **Sigma** `2be95732-d673-48b2-a7b7-4adb531eb4e6`
  (2026-c2-beacon-periodicity.yml): temporal correlation
  (≥ 8 connections / 10 min / same process) — the beacon shape.
- SIEM-side (beyond Sigma): score **interval regularity** — deltas
  between successive connections from one process. With default jitter
  the deltas cluster around the mean ± 20%; a stdev/mean ratio under
  ~0.15 over ≥ 20 cycles is a strong periodicity verdict even when no
  agent rule fired (i.e., for implants you didn't build).

## Network artifacts

- Go crypto stack TLS fingerprint (JA3/JA4 stable per Go version) —
  worth pinning for the range listener.
- Beacon requests are one streaming POST per cycle: request body =
  uplink frames, response body = downlink batch, `Connection: close`
  mandatory (see handoff "The Why").
- No domain fronting, no traffic morphing by design (finalization-plan
  §5) — what you see is exactly the poll cadence.

## False-positive profile

- Any polling app (updaters, mail clients, telemetry agents) trips the
  count correlation; `group-by: Image` keeps attribution per-process.
  Combine with the parent-process rule to separate user-context
  pollers from service-context ones.

## Validation

1. Agent in https mode, `sleep 15 jitter 5` from the console → expect
   the correlation rule to fire within the 10-minute window.
2. Switch to mTLS mode → base rule fires once, correlation goes quiet.
3. `history` / audit log: every poll-cycle task lines up with a
   connection event timestamp — reconcile counts both directions.

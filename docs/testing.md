# Testing

The implementation is private, but the testing discipline is part of
the portfolio: how a dual-use networked system earns confidence.

## Layers

| Layer | What it covers |
|---|---|
| Protocol unit tests | framing (incl. mixed JSON/binary streams), direction ACL on read **and** write, replay/sequence rejection, timestamp window, size caps |
| Integration | real server over real TLS with generated CAs: auth success/failure classes, reconnect/resume, session lifecycle |
| End-to-end | the **real agent** against the **real server** in-process: exec round-trips with exit codes, transfers with forced mid-transfer disconnect and resume, offline task queue dispatch on reconnect, both transports |
| Concurrency | the entire suite runs under the Go race detector, in CI on every push |
| Fuzzing | native Go fuzz target over the frame reader with a committed seed corpus (valid/invalid/truncated/hostile-length frames) — framing is the trust boundary, so it gets adversarial input |
| Live range validation | scenarios executed against a real Windows host with Sysmon; detections reconciled against the audit ground truth — see the [coverage report](../scenarios/COVERAGE-2026-09-02.md) |

## Tests as regression archaeology

The interesting tests are the ones a bug had to earn:

- **Zombie-session DoS** — 100 consecutive failed authentications must
  leave zero session records; pinned after an audit found error paths
  that could strand sessions.
- **Ping-cycle survival** — a live-incident bug: one missed pong (host
  suspend) left a stale pending entry that poisoned every future
  connection of that session into an infinite reconnect loop. The
  default 30 s ping interval had never intersected a test; the
  regression test now runs shortened intervals (2 s/1 s) through real
  server + real agent and asserts session identity stability, RTT
  resolution, and exec-after-pings.
- **Shutdown drain** — stopping the server mid-exec must return within
  the bounded drain wait (no deadlock), and the agent must
  distinguish deliberate shutdown (exit, don't reconnect) from a
  dropped connection (reconnect).
- **Auth-oracle uniformity** — five distinct failure classes assert a
  byte-identical error response, so the unification cannot silently
  regress into an enumeration oracle.
- **Beacon/transport parity** — the beacon exchange passes the same
  task/transfer scenarios as the persistent transport, pinning the
  "session ≠ connection" design.

## CI

Verification-only pipeline on every push and pull request: gofmt,
`go vet`, `go test -race`, build smoke for both targets (server for
linux, agent cross-compiled for windows). It builds no artifacts and
publishes nothing — release binaries are produced locally by design.

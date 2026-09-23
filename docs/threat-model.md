# Threat model

The platform is a dual-use security instrument, so the threat model has
two halves: protecting the infrastructure itself, and being honest
about what the instrument looks like from the defending side. Both are
first-class engineering input, not afterthoughts.

## Assets

| Asset | Protection |
|---|---|
| PSK and CA key material | never embedded in the public layer; per-deployment generation; client pins the CA |
| Operator console / server process | network-facing only on its two listeners; auth controls below |
| Audit database | tamper-evident record of every action; the honesty anchor for detection validation |
| Agent on the endpoint | assumes host-level discovery is possible; release builds are silent, no persistence mechanisms |

## Adversaries and mitigations

| Adversary | Mitigation |
|---|---|
| Network MITM / channel tampering | TLS 1.3 minimum, mutual certificates, CA pinning; independent PSK challenge–response inside the channel |
| Replay of captured frames | per-connection monotonic sequence numbers; ±30 s timestamp window; per-auth random challenges (never reused) |
| Unauthorized client with valid TLS cert but no PSK | second auth factor (HMAC challenge); uniform rejection responses — no oracle to enumerate client IDs |
| Authentication flood / zombie sessions | failed authentications tear down the session record on every exit path (verified by a dedicated DoS test: 100 failed auths → 0 sessions); per-IP rate limit; max-client cap; bounded pending auths |
| Malicious / compromised agent | server treats the endpoint as untrusted: strict input validation, path-traversal and reserved-name checks, frame size caps, spool isolation per transfer |
| Half-open dead connections | server-initiated ping loop with timeout; a missed pong closes the connection but keeps the session for resume |
| Traffic analysis of the C2 channel | deliberately not "solved" — the beacon's periodicity is a documented, detectable signature (see below) |

## The honest half: the platform as an adversary signal

The framework does not attempt to hide its own traffic. That is a
design position, not a gap: the beacon cadence, the long-lived mTLS
session, the process tree on the endpoint — all of it is specified as
**detection content** (`../detections/`). Sigma rules exist for the
framework's own behavioral patterns (outbound listener connection,
beacon periodicity correlation, agent-spawns-shell lineage, headless
ConPTY host, transfer spool file activity), and they are validated
live against real Sysmon telemetry
([coverage report](../scenarios/COVERAGE-2026-09-02.md)).

In other words: the attacker-side artifacts are enumerated by the same
engineering effort that builds the attacker side. The audit log knows
what was done; the endpoint telemetry shows what was seen; the delta
between them is measured, not assumed.

## Evasion: a scoped boundary

The implant deliberately implements **no defense-subversion techniques**
(see [scope.md](scope.md)) with one documented exception: classic
debugger-detection checks (T1622), which only read OS debugger state
and patch nothing. Even that exception ships two-sided — its Sigma
rule (`debugger attached to agent`), its playbook with an honest
visibility analysis of what classic Sysmon can and cannot see, and a
minimal validation debugger — so the feature arrives with its own
detection rather than as a one-sided capability.

Technique families that would require evasion engineering in the
implant (AMSI/ETW tampering, process injection, token manipulation,
UAC bypass, obfuscation) are exercised as **documented procedures
through the shell** with paired detections, and the three families
that have no open procedure (sleep obfuscation, direct/indirect
syscalls, implant-integrated injection) are covered as
*documented — not exercised* detection playbooks with the coverage gap
stated explicitly rather than hidden.

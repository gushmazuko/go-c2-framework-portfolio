# Scope statement

This project draws its security boundary explicitly. The line below is
a design decision with recorded rationale, not a disclaimer.

## What this is

A compact C2 platform — protocol, server, endpoint component, operator
console — built together with a **detection-engineering layer** that
specifies and validates defensive coverage against it. The implant
implements **operational capabilities only**:

- sessions, authentication, reconnect/resume;
- two transports (persistent mTLS, HTTPS beacon with sleep ± jitter);
- command execution and an interactive ConPTY shell (resize,
  background, reattach);
- chunked, checksummed, resumable file transfer in both directions;
- offline task queueing for intermittent connectivity;
- ATT&CK-tagged audit of every action;
- classic debugger-detection checks (see below).

## What the implant deliberately does not implement

No in-memory patching of security APIs (AMSI, ETW), no unhooking, no
direct/indirect syscalls, no sleep-time obfuscation or memory
encryption, no process injection, no packing or build obfuscation, no
persistence modules, no anti-analysis beyond the debugger checks, no
traffic morphing (malleable profiles, domain fronting, CDN abuse).

These families are covered as **detection material instead of
features**: documented procedures exercised through the shell with
paired Sigma rules and playbooks — and for the three families where no
open procedure reproduces the artifacts (sleep obfuscation,
direct/indirect syscalls, implant-integrated injection), detection
playbooks with the coverage gap marked *documented — not exercised*
rather than quietly omitted.

## The single exception, stated plainly

**T1622 debugger checks.** The implant queries OS debugger state
(`IsDebuggerPresent`, `CheckRemoteDebuggerPresent`,
`NtQueryInformationProcess` debug-port/flags classes) at startup and
periodically, and exits silently on a positive. These checks are
read-only — nothing is patched, hooked or disabled — and the feature
ships two-sided: its own Sigma rule (debugger attach to the agent),
its playbook with an honest visibility analysis, and a minimal
validation debugger so the behavior can be reproduced and verified
deterministically. A capability without its detection would be a
one-sided feature; this one arrives with both.

## Operational posture

- Server targets linux/amd64; the endpoint component targets
  windows/amd64 only.
- Release binaries are **built locally only** — no CI pipeline builds
  or publishes binaries.
- Authorized security research and lab use only. The detection
  content in this repository is published for defenders.

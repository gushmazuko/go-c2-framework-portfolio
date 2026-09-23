# Architectural evasion families — documented, not exercised

**Status: `documented — not exercised` in every coverage report.** These
three families have no open procedure that reproduces their artifacts:
their telemetry only exists when the technique is *integrated into an
implant*. This range declines to implement them (finalization-plan §6 —
the one accepted trade-off; anti-debug/T1622 is the single carve-out,
and it ships its own playbook). What follows is the detection-side
material defenders need anyway, written so a coverage report can name
the gap precisely.

## Sleep obfuscation

**What it is:** the implant encrypts itself in memory and sheds
artifacts (thread stacks, hooked API references) while sleeping between
beacons; execution-time re-decryption windows are the only exposure.

**What defenders hunt (EDR/memory, not Sigma):**

- Periodic memory-scanner sweeps of high-entropy private-commit
  regions in long-lived user processes (RWX → RW flips around sleep
  intervals).
- Thread-context anomalies: threads with invalid return addresses /
  no image backing during sleep windows.
- Beacon cadence (see T1071 playbook) still leaks the sleep interval —
  the network layer remains visible even when memory goes dark.

**Coverage-report row:** *documented — not exercised; network-layer
telemetry of the sleep cadence IS exercised (beacon rules).*

## Direct / indirect syscalls

**What it is:** user-mode shellcode invokes syscalls directly (or
"indirectly" via a legitimate module's syscall instruction) to skip
hooked API entries — no `ntdll` hook frames appear on call stacks.

**What defenders hunt:**

- Syscall-from-unbacked-memory heuristics (call stacks whose syscall
  instruction does not resolve into ntdll's code region).
- Hook-integrity baselining: hooks stay intact yet behavior happens —
  the absence of expected EDR API callbacks for known behavioral
  chains (process-open → alloc → write) is the signal.
- ETW-TI (kernel-side) is the intended high ground: it observes the
  behavior regardless of user-mode hooks.

**Coverage-report row:** *documented — not exercised; the behavioral
chains it would evade ARE exercised unhooked (normal syscall path) —
use SC-06/SC-08 flows to baseline what "visible" means before grading
this gap.*

## Implant-integrated injection

**What it is:** injection as an implant operating mode (migration,
module loading into foreign processes) — vs. SC-08's external,
procedure-driven injection. Integrated variants allocate without
loaders, use callback enumerations, and keep the implant's memory
signature inside the target.

**What defenders hunt:**

- Call-stack anomalies: threads whose stacks never pass through a
  module's entry (loader-less starts).
- Parent-child anomalies that no shell procedure would produce (the
  target process's children keep the implant's behavior pattern).
- Unlinked/moduleless image regions with executable content and
  periodic network egress in the *target's* context.

**Coverage-report row:** *documented — not exercised; procedural
injection telemetry IS exercised (SC-08) — the gap is specifically the
integrated variant's memory/stack signatures.*

## Using this honestly

A coverage report built from `mappings.yaml` will always list these
three rows as `documented — not exercised`. That is the range's stated
price for its scope line, not an oversight: name the gap, score
everything else against live telemetry, and treat this file as the
hunter's brief for what the range *cannot* show you.

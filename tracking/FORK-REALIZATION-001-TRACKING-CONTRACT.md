# FORK-REALIZATION-001 — Tracking & Reporting Contract

**Status:** ACTIVE (part of FORK-REALIZATION-001 — not a separate frontier)
**Maintained by:** Local Agent (terminal evidence)
**Independently validated by:** Web Agent / Operator (after Local Agent return)
**Location:** tracked in the Litenyx-Fork repository so it travels with the frontier work.

## 1. Purpose

Continuous execution requires an observable spine. This contract defines how every
frontier step F1..F10 is tracked and reported so that execution does not become an
unobservable chain of mutations.

## 2. Required record per frontier step

| Axis                  | Required record                                          |
| --------------------- | -------------------------------------------------------- |
| **Frontier**          | F1..F10                                                  |
| **Objective**         | What must become true                                    |
| **Scope**             | Files/repos/systems allowed to change                    |
| **Pre-state**         | Exact starting commit/tree/config                        |
| **Gap**               | What is preventing acceptance                            |
| **Action**            | What was actually done                                   |
| **Evidence**          | Commands, outputs, hashes, tests, artifacts              |
| **Observed state**    | What is actually true afterward                          |
| **Verification**      | Independent checks                                       |
| **Defects**           | Found / resolved / deferred                              |
| **Decision**          | Continue / replan / hold / close                         |
| **Commit**            | Exact mutation identity                                  |
| **Remote state**      | Local vs GitHub                                          |
| **Next frontier state** | Ready / active / blocked / complete                    |

The live ledger (`FORK-REALIZATION-001-LEDGER.md`) is the authoritative record.
Every cell carries a provenance tag (see section 3).

## 3. Epistemic discipline

A record MUST distinguish the four states used across this project's governance:

```text
[PLANNED]  — intended action; not yet executed
[CLAIMED]  — asserted result; NOT yet reproduced or witnessed
[OBSERVED] — witnessed by terminal execution (Local Agent)
[VERIFIED] — independently reproduced/validated (Web Agent or repeatable check)
```

Rules:
- A cell is written with the strongest tag actually earned; never higher.
- [OBSERVED] is the floor for "what happened" claims in this ledger (Local Agent
  terminal evidence); conversation alone is not terminal execution.
- [VERIFIED] is earned only by independent reproduction or an independent validator.
- Anything asserted without an attached command/output/hash reference is [CLAIMED]
  at best and must be marked as such.

## 4. Per-step workflow

```text
DISCOVER
  ↓
TRACK PRE-STATE
  ↓
EXECUTE F#
  ↓
CAPTURE EVIDENCE
  ↓
VERIFY
  ↓
REPORT F#
  ↓
CLOSE F#
  ↓
AUTOMATICALLY ENTER F#+1   (per continuous-execution directive)
```

- Close requires: objective achieved, evidence captured, verification passed,
  ledger updated, commit identity recorded (if any).
- The frontier proceeds continuously per
  `OPERATING-DIRECTIVE-20260925-CONTINUOUS-EXECUTION`; the ledger is updated at
  each step, not only at the end.
- Confusing choice => STOP, resolve, resume; merge gate => STOP (separate frontier).

## 5. Report format

Every progress report references the ledger and states, for the current step:

1. **Frontier step + status** (planned/active/blocked/complete)
2. **Objective** — what must become true
3. **Gap** — what is preventing acceptance
4. **Action taken** (with evidence references)
5. **Observed state** (terminal evidence)
6. **Verification** (independent checks, or "pending")
7. **Decision** (continue/replan/hold/close)
8. **Commit + remote state**

## 6. Authority

- Maintaining the ledger is within FORK-REALIZATION-001 scope (build tooling/harness infra).
- The ledger records but never grants mutation, Git, push, or governance authority.
- Commit+origin synchronization follows the normal commit/push gates of this frontier.
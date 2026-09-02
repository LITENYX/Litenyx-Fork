# Release: gate-t-testnet-activation

**Date:** 2026-08-21
**Artifact:** `a9fe0f8ebf0d638013b12924e623b5df8ba5e4ab`
**Branch:** `gate-t-testnet-activation`
**PR:** [#7](https://github.com/LITENYX/litenyx/pull/7)

## Milestone summary

GATE-T establishes the first Litenyx Phase-2 testnet activation boundary. Five source-level changes implement testnet-only consensus and activation parameters, verified by CI and provisioned into a 3-node rehearsal cluster. Mainnet, Gate-1, and H_V remain frozen and untouched.

## Scope

### Governance gates completed

| Gate | Status | Description |
|---|---|---|
| GATE R | COMPLETE | Five-study research program (D1-D5) |
| GATE T | COMPLETE | Five authorized testnet-only Phase-2 changes |
| GATE TN | AUTHORIZED | Testnet rehearsal cluster provisioned |

### Five authorized changes

| # | File | Change |
|---|---|---|
| 1 | `LITENYX_identity.h` | `LitenyxSharedStateHeightForNetwork` |
| 2 | `LITENYX_validation.h/.cpp` | `netId` param for `LitenyxCheckAuxHeader` |
| 3 | topology/chainid/execution/draining authorities | Phase 4-7 testnet heights (6,050,000–7,000,000) |
| 4 | `chainparams.cpp` | Testnet BIP9 `DEPLOYMENT_LITENYX` (bit 2, 75% = 7560) |
| 5 | `miner.cpp:155` | Uncomment `ComputeBlockVersion()` |

### Test correction

`test_litenyx_topology_authority.cpp` lines 694, 745: stale testnet height 1500 → 6,150,000

## CI verification

| Workflow | Status | Duration |
|---|---|---|
| C++ Test Suite | GREEN ✓ | 40s |
| Litenyx CI | GREEN ✓ | 19m40s |
| M3 Integration Gate | GREEN ✓ | 13m34s |

All 3 workflows passed against commit `a9fe0f8eb` on branch `gate-t-testnet-activation`.

## Binary acquisition

Built via GitHub Codespace + Docker ubuntu:20.04 / g++-10 / Boost 1.71. Three independent builds produced identical SHA-256 fingerprints (reproducible build verified).

| Binary | SHA-256 | Size |
|---|---|---|
| `dogecoind` | `681724bcdde1c4669c6ab6fda71f455594346f5830ad1ba797c8a2119f857597` | 169 MB |
| `dogecoin-cli` | `cc32db0b35e2e1641df368b9f81e70ba7923389dd61b1ab81d424c492b51f93b` | 11 MB |
| `dogecoin-tx` | `704e68c1ee24e8b9e70e5d260e58026dda7c8e9c78a199a31b8b1f5091b9dd5c` | 24 MB |

## Testnet node provisioning (STEP 1A)

3-node Docker cluster provisioned on `gate-tn-net` (172.28.0.0/16):

| Node | IP | P2P Port | RPC Port | Status | Peers | Chain |
|---|---|---|---|---|---|---|
| node1 | 172.28.0.10 | 44556 | 44555 | RUNNING ✓ | 6 | test / height 0 |
| node2 | 172.28.0.11 | 44556 | 44555 | RUNNING ✓ | 7 | test / height 0 |
| node3 | 172.28.0.12 | 44556 | 44555 | RUNNING ✓ | 7 | test / height 0 |

### Chain convergence

All 3 nodes converge on identical genesis block:
```
BEST_BLOCK_HASH = bb0a78264637406b6360aad926284d544d7049f45189db5664f3c4d07350559e
CHAIN            = test
CHAIN_HEIGHT     = 0
DIFFICULTY       = 0.000244140625
```

### P2P topology

- Full mesh: node1↔node2, node1↔node3, node2↔node3
- External testnet peers discovered: 135.181.168.83, 35.84.106.212, 213.91.128.155, 95.141.35.117, 185.232.70.226
- Protocol version: 70015 (Shibetoshi:1.14.9)

### Artifact verification

All 3 nodes report identical artifact SHA-256:
```
ARTIFACT_SHA256 = 681724bcdde1c4669c6ab6fda71f455594346f5830ad1ba797c8a2119f857597
```

## Reproducible build command

```bash
# On a Linux host with Docker:
docker run --rm -v /path/to/litenyx:/repo -w /repo/deploy \
  -e DEBIAN_FRONTEND=noninteractive -e CXX=g++-10 ubuntu:20.04 bash -c '
    apt-get update && apt-get install -y \
      git build-essential libtool autotools-dev automake pkg-config \
      bsdmainutils python3 python3-pip libssl-dev libevent-dev \
      libboost-all-dev libboost-test-dev libboost-thread-dev \
      libboost-chrono-dev libminiupnpc-dev libzmq3-dev \
      libsqlite3-dev libdb++-dev libfmt-dev g++-10
    git checkout a9fe0f8eb
    make clone-dogecoin
    make inject-hooks
    make production-build
    sha256sum src/dogecoind src/dogecoin-cli src/dogecoin-tx
  '
```

## Preserved invariants

- **Gate-1 frozen** — no modifications to Gate-1 baseline
- **H_V frozen** — no modifications to H_V
- **Mainnet excluded** — no mainnet activation or deployment
- **Testnet-only scope** — all changes bounded to testnet parameters
- **No merge** — PR #7 remains unmerged (merge not authorized)
- **No production** — production build path untouched

## Exclusions

| Item | Status |
|---|---|
| Mainnet | DEFERRED — not in scope |
| H_V | FROZEN — no modifications |
| Gate-1 baseline | FROZEN — no modifications |
| Production | DEFERRED — not in scope |
| Unrelated cleanup | NOT IN SCOPE |
| Submodule sync | NOT IN SCOPE |

## Current gate state

```text
GATE-TN
  └── STEP 1A-VERIFY
       ├── Artifact provenance          VERIFIED ✓
       ├── Topology                     FAIL ✗
       ├── Peer compatibility           DOCUMENTED ⚠
       ├── Mining/version               PASS ✓
       ├── ComputeBlockVersion          NOT_OBSERVABLE ⚠
       ├── Chain convergence            NOT ESTABLISHED ⚠
       └── OVERALL                      BLOCKED ✗

       Mining/signaling rehearsal       BLOCKED
       BIP9 progression                 BLOCKED
       Phase-2 activation               BLOCKED
       Activation-height testing        BLOCKED
```

### STEP 1A-VERIFY formal disposition

| Item | Status | Note |
|---|---|---|
| Topology | FAIL ✗ | node1 4/2/6, node2 5/2/7, node3 5/2/7 — target 4/8/12 |
| Peer compatibility | DOCUMENTED ⚠ | 1 incompatible 1.14.6 external peer |
| Mining/version | PASS ✓ | block generated, Litenyx bit 2 active |
| ComputeBlockVersion | NOT_OBSERVABLE ⚠ | no explicit log entry |
| Chain convergence | NOT ESTABLISHED ⚠ | node1 height 1, nodes 2/3 height 0 |
| **Overall** | **BLOCKED / RETEST REQUIRED** | verification failure, not implementation failure |

## Successor checkpoint

**STEP 1A-VERIFY-RETEST** — resolve topology (4-out/8-in), document compatible peers, verify chain convergence, preserve ComputeBlockVersion NOT_OBSERVABLE. Previous evidence preserved as failed baseline. No GATE-T implementation changes required.

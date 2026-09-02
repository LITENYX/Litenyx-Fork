# DOGE-BUILD-HARNESS-1

## Status

```
DOGE-BUILD-HARNESS-1       AUTHORIZED
Mode                       READ-ONLY STUDY
Mutation                   NONE
Purpose                    deterministic build/runtime control sample

GATE-TN                    AUTHORIZED
STEP 1A-VERIFY             BLOCKED / RETEST REQUIRED
Phase-2                    BLOCKED
Mainnet                    EXCLUDED
Gate-1                     FROZEN
H_V                        UNCHANGED
Consensus-code changes     NONE
```

## Objective

Establish a reproducible, isolated Dogecoin build/runtime environment for Litenyx verification:

```
Dogecoin v1.14.9 (pinned e0a1c157...)
    +
Litenyx GATE-T artifact (a9fe0f8eb)
    +
controlled 3-5 node harness
    +
deterministic P2P topology
    +
controlled mining
    +
BIP9 observation
    +
reproducible evidence
```

This study is **read-only with respect to the Litenyx consensus design**. It investigates and validates the build/runtime substrate; it does not change the five GATE-T consensus modifications.

---

## 1. Build substrate study

Establish one canonical build recipe.

```
BASE
  Dogecoin v1.14.9
  pinned commit e0a1c157...
  Ubuntu 20.04
  g++-10
  Boost 1.71

LITENYX
  a9fe0f8eb
```

Record:
- exact source SHAs
- patch SHAs
- compiler version
- linker version
- Boost version
- configure arguments
- Makefile/environment variables
- generated binaries
- SHA-256 of every executable

### Reproducibility test

Build the same source three independent times:

```
BUILD-A -+
BUILD-B -+--> SHA-256 comparison
BUILD-C -+
```

Expected:
```
dogecoind   A = B = C
dogecoin-cli A = B = C
dogecoin-tx  A = B = C
```

This part is already substantially demonstrated by existing evidence; the study should formally capture it.

---

## 2. Patch/injection determinism

Create a clean-source matrix:

```
CLEAN CLONE
    |
inject-hooks
    |
patched tree
    |
SHA/tree inventory
```

Then repeat:

```
CLEAN CLONE
    |
inject-hooks
    |
patched tree
```

Verify that the two resulting trees are identical.

Also test idempotency:

```
inject-hooks
inject-hooks again
inject-hooks again
```

The second and third executions must be **idempotent**, producing no source drift.

This turns the earlier CI/patch confusion into a formal deterministic-build property.

---

## 3. Testnet harness — NOT public testnet

Use an isolated Docker network.

```
                 gate-tn-lab
                     |
        +------------+------------+
        |            |            |
      node1        node2        node3
        |            |            |
        +------------+------------+
                     |
                miner/control
```

No:
- DNS seeds
- public discovery
- uncontrolled external peers
- legacy Dogecoin peers

This directly eliminates the IBD and external-peer contamination encountered in previous experiments.

---

## 4. Separate topology study from consensus study

The 4-out/8-in requirement is **not achievable with 3 nodes**. With three nodes, the maximum number of *other local nodes* available for inbound connections is only two.

### A. Minimal consensus harness

3 nodes:

```
node1 <-> node2
  |        |
node3 -----+
```

Purpose:
- startup
- genesis
- mining
- block propagation
- restart
- reorg
- deterministic chain convergence
- BIP9 observation

### B. Topology stress harness

Use enough nodes to make the topology mathematically possible:

```
N >= 13
```

if the literal requirement is eight distinct inbound peers plus four outbound peers per node.

That becomes a **topology experiment**, not a consensus prerequisite.

---

## 5. Controlled mining

Create a deterministic mining controller:

```
miner-1
miner-2
miner-3
miner-4
```

The harness should produce:
- ordinary block
- signaling block
- non-signaling block

and record:
- height
- version
- versionbits
- hash
- previous hash
- miner/node
- timestamp

This tests the actual BIP9 mechanism without waiting for millions of testnet blocks.

---

## 6. BIP9 study

The actual testnet activation heights are millions of blocks away:

```
6,000,000
6,050,000
6,150,000
...
7,000,000
```

The build study should create a **simulation/test harness for the state machine**, while preserving the actual production/testnet constants.

Conceptually:

```
REAL PARAMETERS
       |
       v
production/testnet binary
       |
       +-------+
               |
TEST HARNESS   v
synthetic chain heights
       |
       v
BIP9 state transitions
       |
       v
evidence
```

The harness should test:

```
DEFINED
   |
STARTED
   |
LOCKED_IN
   |
ACTIVE
```

and the relevant Litenyx height gates independently.

---

## 7. Chain propagation experiment

Explicitly test:

```
mine block
   |
T+0
   |
T+1
   |
T+5
   |
T+10
   |
T+30
   |
all nodes same tip?
```

Capture:
- height
- bestblockhash
- previousblockhash
- peer count
- validation result
- propagation latency

---

## 8. Failure/recovery matrix

| Experiment              | Expected result                |
| ----------------------- | ------------------------------ |
| node restart            | rehydrate same chain           |
| miner restart           | resume mining                  |
| temporary peer loss     | reconnect                      |
| block propagation delay | eventual convergence           |
| invalid block           | rejection                      |
| incompatible block      | rejection                      |
| node isolated           | no silent consensus corruption |
| node rejoins            | deterministic re-sync          |
| duplicate block         | idempotent handling            |
| conflicting chain       | deterministic chain selection  |

---

## 9. Evidence architecture

Produce one immutable evidence package per run:

```
DOGE-BUILD-HARNESS-1/
+-- source/
+-- build/
+-- binaries/
+-- patches/
+-- topology/
+-- nodes/
|   +-- node1/
|   +-- node2/
|   +-- node3/
+-- mining/
+-- versionbits/
+-- propagation/
+-- recovery/
+-- manifest.json
```

The manifest should bind:
- source SHA
- patch SHA
- binary SHA
- container image
- configuration SHA
- experiment ID
- timestamp
- node identities
- result

---

## 10. Governance boundary

This is a new study, not a GATE-TN mutation.

Explicitly **not authorized**:
- PR #7 merge
- mainnet
- production
- consensus-code modification
- Gate-1 modification
- H_V modification
- live activation

---

## Proposed execution phases

### Phase 1: Build canonicalization (1 Codespace)

```
[1.1] Clone Dogecoin at INT-Q5 pin
[1.2] Apply inject-hooks (canonical run)
[1.3] Record source tree SHA
[1.4] Build in Docker ubuntu:20.04 / g++-10 / Boost 1.71
[1.5] Record binary SHA-256
[1.6] Repeat 2x for reproducibility (3 builds total)
[1.7] Verify A = B = C
```

### Phase 2: Patch determinism (1 Codespace)

```
[2.1] Clean clone + inject-hooks -> tree SHA
[2.2] Clean clone + inject-hooks -> tree SHA
[2.3] Verify identity
[2.4] Triple inject-hooks on same tree
[2.5] Verify idempotency
```

### Phase 3: Isolated testnet harness (1 Codespace)

```
[3.1] Create Docker network gate-tn-lab (no external access)
[3.2] Start 3 nodes with isolated config
[3.3] Verify IBD status (synced_headers, synced_blocks)
[3.4] Verify topology (local connections only)
[3.5] Mine block on node1
[3.6] Verify propagation to node2, node3
[3.7] Record evidence
```

### Phase 4: BIP9 observation (1 Codespace)

```
[4.1] Start nodes with testnet BIP9 params
[4.2] Mine blocks, record version bits
[4.3] Observe DEPLOYMENT_LITENYX (bit 2)
[4.4] Record state transitions
```

### Phase 5: Failure/recovery (1 Codespace)

```
[5.1] Execute failure matrix
[5.2] Record recovery behavior
[5.3] Produce evidence package
```

---

## Success criteria

```
BUILD REPRODUCIBILITY    3/3 identical SHA-256
PATCH IDEMPOTENCY        3/3 identical tree SHA
NETWORK ISOLATION        0 external peers
IBD STATUS               synced_headers >= 0
MINING                   block generated
PROPAGATION              common tip on all nodes
BIP9 OBSERVATION         version bits recorded
FAILURE/RECOVERY         all matrix rows passed
EVIDENCE                 manifest.json complete
```

---

## Expected outcome

This study will produce a **deterministic build/runtime control sample** that can serve as the laboratory for all future Litenyx consensus verification work, rather than repeatedly debugging public-testnet conditions.

The study does **not** resolve the 4-out/8-in topology requirement (which is unachievable with 3 nodes), but it does establish whether the Litenyx consensus implementation is functionally correct in a controlled environment.

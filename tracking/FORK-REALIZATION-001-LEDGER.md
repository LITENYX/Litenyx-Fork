# FORK-REALIZATION-001 — Live Frontier Ledger

**Frontier objective (F1..F10):** Litenyx runs as its own Dogecoin-derived blockchain
with deterministic genesis, buildable daemon, functional node lifecycle, and
independently verified fork behavior.

**Authority reference:** tracking/FORK-REALIZATION-001-TRACKING-CONTRACT.md
**Execution mode:** continuous per OPERATING-DIRECTIVE-20260925-CONTINUOUS-EXECUTION
(stop only on confusing choice or merge gate).

**Provenance tags:** [PLANNED] [CLAIMED] [OBSERVED] [VERIFIED] — see contract section 3.

---

## Frontier steps

| #   | Step                          | Status   |
| --- | ----------------------------- | -------- |
| F1  | Baseline reconciliation/build | VERIFIED |
| F2  | Reproducible daemon build     | ACTIVE   |
| F3  | Network identity              | READY    |
| F4  | Genuine genesis               | READY    |
| F5  | Production consensus activation | READY  |
| F6  | Node lifecycle                | READY    |
| F7  | Adversarial consensus verification | READY |
| F8  | Mining                        | READY    |
| F9  | Multi-node network            | READY    |
| F10 | Fork-release gate             | READY    |

---

## F1 — Baseline reconciliation / build — VERIFIED

| Axis              | Record                                                                                      |
| ----------------- | ------------------------------------------------------------------------------------------- |
| **Frontier**      | F1                                                                                          |
| **Objective**     | Pinned Dogecoin v1.14.9 base verified, Litenyx hooks injected, dependencies resolved, fork daemon configures AND compiles on target toolchain (MSYS2 ucrt64 / mingw-w64) AND independently verified on canonical Linux CI (ubuntu:20.04, Boost 1.71). All 6 mandatory patches function correctly. |
| **Scope**         | `deploy/Makefile`, `deploy/patches/*`, `deploy/external/dogecoin` (clone + generated build), MSYS2 ucrt64 environment (pacman pkgs, pkg-config, libtool, windres), `tracking/` docs in this repo. |
| **Pre-state**     | [OBSERVED] Litenyx-Fork HEAD `c82b909` (clean except `deploy/Makefile` boost-libdir fix). Dogecoin clone HEAD `e0a1c157791544e818c901bd9341896965afbf9d` (INT-Q5 pin). Toolchain: ucrt64 g++ 16.1.0 (`x86_64-w64-mingw32`); autotools via `_lt_pkgdatadir=/usr/share/libtool ACLOCAL_PATH=/usr/share/aclocal`; GNU sort/sed via coreutils+sed (pacman). Deps: ucrt64 openssl, libevent, BDB (`mingw-w64-ucrt-x86_64-db`), boost-libs. |
| **Gap**           | Daemon does not compile. First failure: `src/compat.h:37: fatal error: sys/mman.h: No such file or directory`. |
| **Root cause**    | [VERIFIED via probe] ucrt64 g++ under strict `-std=c++11` does NOT predefine bare `WIN32` (defines only `_WIN32`/`__WIN32__`; `-std=gnu++11` defines `WIN32`). Dogecoin `compat.h` branches on `#ifdef WIN32`; macro absent => POSIX branch => requires `sys/mman.h` (not shipped by ucrt64). |
| **Action**        | [OBSERVED] Three-part build fix in `deploy/Makefile`: (1) `--host=x86_64-w64-mingw32` so `configure.ac` mingw block (L314-364, L874-917) sets `TARGET_OS=windows`, `CPPFLAGS+= -D_MT -DWIN32 -D_WINDOWS -DBOOST_THREAD_USE_LIB`, links `kernel32..crypt32`; (2) `--enable-c++17` (official dogecoin switch L66-82) because ucrt64 Boost 1.8x headers need C++14/17; (3) platform-aware `uname` branch in the Makefile so the Linux CI build command stays byte-identical while Windows gets the fixes. Prereqs verified present: `windres` (binutils 2.46) @ `/ucrt64/bin/windres`; `libmingwthrd.a` @ `/ucrt64/lib`. SSS rehydration gate target added to Makefile. |
| **Evidence**      | [OBSERVED] Probe matrix (probe-std2.sh): `WIN32` absent under `-std=c++11`, present under `-std=gnu++11`/default. Exact failing cmd: `ccache g++ -std=c++11 ... -c bitcoind.cpp`. Configure logs show msys host detection `x86_64-pc-msys`, empty `target os`. Build artifacts (25-09 17:13/17:14): `dogecoind.exe` 230,272,315 B; `dogecoin-cli.exe` 22,891,085 B; `dogecoin-tx.exe` 36,288,275 B. |
| **Observed state**| [OBSERVED] Windows: `./dogecoind.exe --version` => `Dogecoin Core Daemon version v1.14.9.0-e0a1c1577-dirty`, runs, exit 0 (Litenyx patches applied => `-dirty`). All three binaries run. [VERIFIED] Linux CI: `dogecoind`/`dogecoin-cli`/`dogecoin-tx` built on ubuntu:20.04 (Boost 1.71, C++11, original configure). C++ KATs pass. Regtest Phase-1+2 acceptance gate passes. INT-OPEN-1/M3 integration gate passes. SSS rehydration test PASSES — cold-restart rehydration verified. |
| **Verification**  | [VERIFIED] Local toolchain build complete + daemon executes (exit 0). [VERIFIED] Independent Linux CI: build + C++ KATs + regtest acceptance + INT-OPEN-1/M3 gate + SSS rehydration gate all SUCCESS. Evidence: Litenyx CI 36295611731, INT-OPEN-1/M3 36295611690, C++ Test Suite 36295611707, SSS Rehydration 36295611695 (all fresh runs from commit 0fd9de9). |
| **Defects**       | Found: [D1] strict `-std=c++11` suppresses `WIN32` (resolved: `--host=x86_64-w64-mingw32`); [D2] Makefile boost-libdir default was a Linux path (resolved: platform-aware); [D3] msys-host pkg-config blind to ucrt64 (mitigated with `PKG_CONFIG_PATH` + stub `libevent_pthreads.pc`; superseded by mingw-host path which disables pkgconfig); [D4] ucrt64 Boost 1.8x needs C++14/17 (resolved: `--enable-c++17`); [D5] unconditional Windows flags would break Linux CI (resolved: `uname`-branch in Makefile, Linux branch byte-identical to CI original). [D6] KAT linkage on Linux used `-mt` suffix (resolved: platform-aware `BOOST_TEST`). [D7] litenyx-rehydrate.patch: SSS rehydration not functioning — **RESOLVED** by adding `rehydrate` to mandatory patch list + verifying hook installed in init.cpp. Deferred: wallet portability warning (BDB != 5.3) — non-blocking for fork baseline. |
| **Decision**      | [VERIFIED] F1 baseline ACHIEVED and INDEPENDENTLY VERIFIED. Proceed continuously to F2. |
| **Commit**        | `0fd9de9` (F1 build fix + rehydrate patch + hook verification) on `fork-realization-001-f1-build`, pushed to origin, PR #10. |
| **Remote state**  | [OBSERVED] Branch pushed; PR #10 open; CI core gates green; SSS rehydration gate SUCCESS (36295611695). |
| **Next frontier state** | F1 VERIFIED; F2 ACTIVE. |

---

## D7 — litenyx-rehydrate.patch: SSS cold-restart rehydration failure — RESOLVED

| Field              | Value                                                                                  |
| ------------------ | -------------------------------------------------------------------------------------- |
| **Status**         | RESOLVED                                                                               |
| **Severity**       | F1 acceptance-blocking (was)                                                           |
| **Surface**        | `litenyx-rehydrate.patch` / SSS cold restart                                           |
| **Expected**       | `spent=True` after cold restart                                                        |
| **Observed**       | `spent=False` (was)                                                                    |
| **Evidence**       | SSS gate 36295611695 (SUCCESS: cold-restart rehydration verified)                     |
| **Mutation**       | `deploy/Makefile`: added `rehydrate` to mandatory patch list + verified hook installed |
| **Root cause**     | `litenyx-rehydrate.patch` was omitted from mandatory patch list in `inject-hooks` loop, so rehydration hook was never installed in `AppInitMain` Phase 7. |
| **Fix**            | Added `rehydrate` to mandatory patch list + added verification that `LitenyxRehydrateSharedSpendSet` call is present in `init.cpp` and `LITENYX_validation.h`. |
| **Disposition**    | DEBUG → FIX → VERIFY → RE-RUN SSS GATE → CLOSED                                        |

---

## F2 — Reproducible daemon build — ACTIVE

| Axis              | Record                                                                                      |
| ----------------- | ------------------------------------------------------------------------------------------- |
| **Frontier**      | F2                                                                                          |
| **Objective**     | Demonstrate the daemon builds reproducibly on both Windows (MSYS2 ucrt64) and Linux (ubuntu:20.04 container) from the same pinned source, with recorded toolchain specs and binary hashes. |
| **Scope**         | `deploy/Makefile` (platform-aware), pinned Dogecoin `e0a1c157`, Litenyx patches (6), pinned Boost/OpenSSL/libevent/BDB on each platform, `tracking/` docs. |
| **Pre-state**     | [OBSERVED] F1 VERIFIED. Daemon builds on both platforms. Windows toolchain: ucrt64 g++ 16.1.0, Boost 1.91.0, OpenSSL 3.6.3, libevent 2.1.12, BDB 18.x, binutils 2.46. Linux CI toolchain: ubuntu:20.04, g++-10, Boost 1.71, OpenSSL 1.1.1f, libevent 2.1.11, BDB 18.x. |
| **Gap**           | Toolchain divergence: Boost 1.91 (Windows) vs 1.71 (Linux CI). Windows test suite link fails (missing bcrypt, Boost test exports). |
| **Action**        | [OBSERVED] Clean rebuild on Windows: daemon + CLI + TX build and run. Binary hashes captured. Linux CI: build passes (Litenyx CI 36295611731). |
| **Evidence**      | Windows clean rebuild (25-09 20:14): `dogecoind.exe` sha256 `9f2a1a064f05e13eaeb5b8012db1a8a90d8e98cf269e4fa9a8b96ddfa39975ed` (230,272,315 B); `dogecoin-cli.exe` `bfbf49da8d7ec527b76a2cb8fdc33639becc3ea542f4b0f01d3a0c6c900adb15` (22,891,085 B); `dogecoin-tx.exe` `0dcbd7c702654a7e6069679d73db827c7fc91e27c7e205fccf54aabca01fb0ce` (36,288,275 B). Daemon runs: `Dogecoin Core Daemon version v1.14.9.0-e0a1c1577-dirty`. Linux CI: build + KATs + regtest + M3 + SSS all SUCCESS (runs 36295611731, 36295611690, 36295611695, 36295611707). |
| **Observed state**| Daemon binary builds and executes on both toolchains. Windows test suite link fails (bcrypt, Boost test exports) — not blocking daemon artifact. |
| **Verification**  | [OBSERVED] Windows daemon runs, version prints Litenyx patches (`-dirty`). Linux CI green on full acceptance chain including SSS rehydration. Binary hashes recorded. |
| **Defects**       | [D1] Toolchain divergence: Boost 1.91 vs 1.71 — recorded, not yet reconciled. [D2] Windows test suite link fails (missing bcrypt, Boost test exports) — daemon builds, test suite secondary. [D3] No deterministic/reproducible build verification yet (single build per platform, no double-build comparison). |
| **Decision**      | [OBSERVED] F2 ACTIVE. Next: deterministic rebuild verification (double-build on each platform), or proceed to F3 (network identity) with toolchain divergence noted. |
| **Commit**        | `0fd9de9` (F1 complete + rehydrate fix) on `fork-realization-001-f1-build`. |
| **Remote state**  | PR #10 open; CI green on all core gates including SSS rehydration. |
| **Next frontier state** | F2 ACTIVE; F3 READY. |
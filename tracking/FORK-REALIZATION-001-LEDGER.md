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
| F1  | Baseline reconciliation/build | ACTIVE   |
| F2  | Reproducible daemon build     | READY    |
| F3  | Network identity              | READY    |
| F4  | Genuine genesis               | READY    |
| F5  | Production consensus activation | READY  |
| F6  | Node lifecycle                | READY    |
| F7  | Adversarial consensus verification | READY |
| F8  | Mining                        | READY    |
| F9  | Multi-node network            | READY    |
| F10 | Fork-release gate             | READY    |

---

## F1 — Baseline reconciliation / build — ACTIVE

| Axis              | Record                                                                                      |
| ----------------- | ------------------------------------------------------------------------------------------- |
| **Frontier**      | F1                                                                                          |
| **Objective**     | Pinned Dogecoin v1.14.9 base verified, Litenyx hooks injected, dependencies resolved, fork daemon configures AND compiles on target toolchain (MSYS2 ucrt64 / mingw-w64). [PLANNED] |
| **Scope**         | `deploy/Makefile`, `deploy/patches/*`, `deploy/external/dogecoin` (clone + generated build), MSYS2 ucrt64 environment (pacman pkgs, pkg-config, libtool, windres), `tracking/` docs in this repo. |
| **Pre-state**     | [OBSERVED] Litenyx-Fork HEAD `c82b909` (clean except `deploy/Makefile` boost-libdir fix). Dogecoin clone HEAD `e0a1c157791544e818c901bd9341896965afbf9d` (INT-Q5 pin). Toolchain: ucrt64 g++ 16.1.0 (`x86_64-w64-mingw32`); autotools via `_lt_pkgdatadir=/usr/share/libtool ACLOCAL_PATH=/usr/share/aclocal`; GNU sort/sed via coreutils+sed (pacman). Deps: ucrt64 openssl, libevent, BDB (`mingw-w64-ucrt-x86_64-db`), boost-libs. |
| **Gap**           | Daemon does not compile. First failure: `src/compat.h:37: fatal error: sys/mman.h: No such file or directory`. |
| **Root cause**    | [VERIFIED via probe] ucrt64 g++ under strict `-std=c++11` does NOT predefine bare `WIN32` (defines only `_WIN32`/`__WIN32__`; `-std=gnu++11` defines `WIN32`). Dogecoin `compat.h` branches on `#ifdef WIN32`; macro absent => POSIX branch => requires `sys/mman.h` (not shipped by ucrt64). |
| **Action**        | [OBSERVED] Three-part build fix in `deploy/Makefile`: (1) `--host=x86_64-w64-mingw32` so `configure.ac` mingw block (L314-364, L874-917) sets `TARGET_OS=windows`, `CPPFLAGS+= -D_MT -DWIN32 -D_WINDOWS -DBOOST_THREAD_USE_LIB`, links `kernel32..crypt32`; (2) `--enable-c++17` (official dogecoin switch L66-82) because ucrt64 Boost 1.8x headers need C++14/17; (3) platform-aware `uname` branch in the Makefile so the Linux CI build command stays byte-identical while Windows gets the fixes. Prereqs verified present: `windres` (binutils 2.46) @ `/ucrt64/bin/windres`; `libmingwthrd.a` @ `/ucrt64/lib`. |
| **Evidence**      | [OBSERVED] Probe matrix (probe-std2.sh): `WIN32` absent under `-std=c++11`, present under `-std=gnu++11`/default. Exact failing cmd: `ccache g++ -std=c++11 ... -c bitcoind.cpp`. Configure logs show msys host detection `x86_64-pc-msys`, empty `target os`. Build artifacts (25-09 17:13/17:14): `dogecoind.exe` 230,272,315 B; `dogecoin-cli.exe` 22,891,085 B; `dogecoin-tx.exe` 36,288,275 B. |
| **Observed state**| [OBSERVED] `./dogecoind.exe --version` => `Dogecoin Core Daemon version v1.14.9.0-e0a1c1577-dirty`, prints help, exit 0 (Litenyx patches applied => `-dirty`). All three binaries run. |
| **Verification**  | [OBSERVED] Local toolchain build complete + daemon executes (exit 0). [PLANNED] Independent verification on canonical Linux CI path (ubuntu:20.04 container, Boost 1.71) + regtest acceptance gate (`ci.yml`). |
| **Defects**       | Found: [D1] strict `-std=c++11` suppresses `WIN32` (resolved: `--host=x86_64-w64-mingw32`); [D2] Makefile boost-libdir default was a Linux path (resolved: platform-aware); [D3] msys-host pkg-config blind to ucrt64 (mitigated with `PKG_CONFIG_PATH` + stub `libevent_pthreads.pc`; superseded by mingw-host path which disables pkgconfig); [D4] ucrt64 Boost 1.8x needs C++14/17 (resolved: `--enable-c++17`); [D5] unconditional Windows flags would break Linux CI (resolved: `uname`-branch in Makefile, Linux branch byte-identical to CI original). Deferred: wallet portability warning (BDB != 5.3) — non-blocking for fork baseline. |
| **Decision**      | [OBSERVED] Continue: local toolchain F1 baseline ACHIEVED. Recommended next: independent GitHub CI run (canonical Linux path) before closing F1; then F2 (reproducible daemon build). |
| **Commit**        | None yet. Worktree dirty: `deploy/Makefile`. |
| **Remote state**  | [OBSERVED] Local only; Litenyx-Fork origin untouched. |
| **Next frontier state** | F1 active; F2 READY (reproducible daemon from clean configure). |

---

## F2..F10

Entries created as each step is entered, following the workflow in the contract.
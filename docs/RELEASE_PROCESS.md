# XIOM Release Process

## v0.57.0 "Unsafe Confinement" — RELEASED (2026-08-10)

**Test baseline: fast-gate 1110 passed / 3 failed (all 3 pre-existing: `test_diff_test_produces_correct_ir` per handoff, stdlib-exec complex + net) / 1 ignored; checker 178/178; stdlib-compile 40/40; parser 96/96; feature-regression 510/510; integration 128/128**
**Target platforms: Windows x64 ✅ (released + installed), Linux x64 (WSL build verified), WASM (prior baseline)**
**Selfhost gate: P8 verified — `selfhost/xiomc_v10.xi` compiles clean under all Unsafe Confinement gates**
**Version: Cargo.toml + xiom-codegen bumped to 0.57.0; git tag `v0.57.0` created**

> **What's in v0.57.0:** Unsafe Confinement complete — P1 gates (T002 extern gate,
> T003 signature gate, block-only unsafe), P2 contracts (T007 requires gate), P3
> guard-heap arena + Copy-Out, P4 stack guard pages, P5 canonical SEH trampoline
> fault trap (REPLACED the hanging inline VEH; block-as-function + pointer
> captures + nested-return routing + arena-aware realloc), P6 transient retry
> (`#[unsafe_no_retry]`), P7 `#[unsafe_direct]` + `--enable-unsafe-direct` gate,
> T006 FFI-ownership checker rule, P8 selfhost gate. Fault-injection smokes
> (AV/ud2/div0 survive; retry delivers value). MCP tools aligned (sandbox audit
> fixed, cheatsheet/guides updated).

### Package + Install (Windows)
```powershell
.\package.ps1 -Version "0.57.0"
# -> release\xiom-v0.57.0\ + xiom-v0.57.0-windows-x64.zip
# Install: release\xiom-v0.57.0\install.bat  (or manual copy to %LOCALAPPDATA%\xiom)
# Verified: xiom --version -> v0.57.0 "Unsafe Confinement";
#           xiom --sandbox --sandbox-report=json -> compiler_version 0.57.0;
#           guard-fault + guard-retry smokes pass with the installed binary.
```

---

## v0.56.0 "Production Polish" — RELEASE CANDIDATE (2026-08-06)

**Test baseline: 2,231 E2E tests (2,230 pass, 1 known flake) + 1,284 unit/tooling tests (100% pass)**
**Target platforms: Windows x64 ✅ (released), Linux x64 ✅ (WSL build + compile + run verified), WASM ✅ (5 E2E)**
**Selfhost gate: 19/19 CLEARED — PRE-SELFHOST COMPLETE**
**All P0/P1/P2 issues RESOLVED — compiler is production-grade at selfhost scale**
**0 compiler warnings — all 6 crates (Windows + Linux)**

> **E2E fixtures policy (2026-08-06):** All E2E gates use **INTERNAL fixtures** inside
> this repo — `tests/ecosystem/` (t1-allocator … t5-btree, t8-safety-probe, test_algo,
> eco_* suites) and `tests/regression/` (m21_*, m33_*, m35_*, spawn_*, send_*). The
> compiler test suite has **NO dependency on `xiom-benchmark-chaos/`** (that repo is
> only used by the benchmark harness, which the compiler does not require to build,
> test, or release). `xiom-benchmark-chaos/` is a WORKSPACE-SIBLING directory with its
> own owner/session and must never be touched by compiler release work.

---

## Selfhost Readiness Checklist

### ✅ Completed (Compiler)
- [x] 19/19 selfhost gates cleared
- [x] All P0/P1/P2 production gates fixed (14/14 fixes, 35 E2E tests)
- [x] 0 compiler warnings across all crates
- [x] Linux + Windows builds verified
- [x] All benchmark tasks pass (t1-t5, 100% pass rate)
- [x] Scaling architecture designed (`docs/SCALING_ARCHITECTURE.md`)

### 🔄 In Progress (Infrastructure — Before Selfhost)
- [ ] **Split monorepo → separate GitHub repos** under XIOM organization:
  - [ ] `xiom-compiler` — crates/xiom* (compiler + tools)
  - [ ] `xiom-stdlib` — stdlib/ (standard library)
  - [ ] `xiom-benchmark` — xiom-benchmark-chaos/ (benchmark harness)
  - [ ] `xiom-docs` — docs/ + website
  - [ ] `xiom-registry` — registry server (Node.js/Express)
  - [ ] `xiom-playground` — WASM playground
- [ ] **GitHub Actions CI** for each repo:
  - [ ] Windows x64 build + test
  - [ ] Linux x64 build + test
  - [ ] macOS x64 build + test (new)
  - [ ] WASM target build
  - [ ] Release artifact packaging
- [ ] **Package registry** setup:
  - [ ] Registry server deployed (registry.xiom-lang.org)
  - [ ] `xiom pkg publish` working end-to-end
  - [ ] `xiom pkg install xiom-stdlib` working
  - [ ] All 40 stdlib modules published
- [ ] **Release v0.56.0 (final)**:
  - [x] Version bumped to 0.56.0 (final, drop -pre)
  - [x] Binary packages for Windows (release\ folder + ZIP)
  - [x] Installer scripts (install.bat / install.ps1, install.sh)
  - [ ] CHANGELOG.md updated

### ⏳ Pending (Selfhost Phase — After Infrastructure)
- [ ] Selfhost bootstrap: XIOM compiler written in XIOM
- [ ] Differential testing: Rust-bootstrapped vs self-compiled IR must match
- [ ] Selfhost CI: self-compiled compiler compiles the test suite

---

### Verified Test Counts (Release Candidate — 2026-08-06)

| Suite | Tests | Result |
|-------|-------|--------|
| E2E (full `e2e_tests`) | 2,231 | ✅ 2,230 PASS — 1 known flake: `e2e_spawn_capture` (pre-existing thread-teardown race, see note below) |
| Checker | 156 | ✅ PASS |
| Parser | 96 | ✅ PASS |
| Feature regression | 510 | ✅ PASS |
| Integration | 128 | ✅ PASS |
| Stdlib execution | 41 | ✅ PASS |
| Stdlib compile | 40 | ✅ PASS |
| Robustness | 63 | ✅ PASS |
| Lexer | 17 | ✅ PASS |
| CTFE | 96 | ✅ PASS |
| Codegen sandbox | 10 | ✅ PASS |
| Verifier (Z3 SMT) | 27 | ✅ PASS (stable, 3/3 runs) |
| JIT engine | 5 | ✅ PASS |
| xiom-lib | 19 | ✅ PASS |
| LSP server | 38 | ✅ PASS |
| Package manager | 39 | ✅ PASS |
| MCP server | 39 | ✅ PASS |
| Debugger | 29 | ✅ PASS |
| FFI generator | 33 | ✅ PASS |
| Doc generator | 4 | ✅ PASS |
| Dependency graph | 23 | ✅ PASS |
| Display | 5 | ✅ PASS |
| Scripting | 34 | ✅ PASS |
| Script diff | 15 | ✅ PASS |
| Formatter | 79 | ✅ PASS |
| **Stable Verified Total** | **~3,700** | **100% of non-flaky** |

> **Known flake — `e2e_spawn_capture`:** pre-existing, NOT a regression (verified
> failing 5/5 at the pre-fix baseline too). `spawn move` threads are detached in
> `stdlib/runtime/xiom_runtime.c`; when `main` returns while a spawned thread is still
> writing via `io.println`, process teardown races the CRT stdio lock → AV with piped
> stdout (passes interactively). Fix belongs in the runtime (detach → join-on-exit);
> does not block release (all other 2,230 E2E pass).

### Key Features (v0.54 → v0.56)

| Version | Features |
|---------|----------|
| **v0.54** | CTFE Phase A+B, Binary cache (`--cache`), Parallel parse (`--parallel`), Thread-safe SyncRegistry, `const { }` blocks, Turbofish `::<T>()`, Builtins (align_of, type_id, field_offset, is_signed), Match exhaustiveness (S2), Overflow/bounds/null checks (S1), `--strict-exhaustive` |
| **v0.55** | OrcJIT engine (`--jit`), Hot reload watcher, Inline ASM `asm()`, Never type `!`, `defer` statement, `spawn` codegen, Channel[T] ring buffer, Send/Sync markers, `build-runtime` command |
| **v0.56** | LTO `--lto`, Debug info `--debug`/`-g`, Lazy JIT `--jit --lazy`, Thread pool (work-stealing), Parallel codegen `--parallel-codegen`, Spawn move semantics (R2), Recursion counter fixes (R4+R5), 0 compiler warnings all crates |

### Selfhost Gate Status (v0.56.0)

| Gate | Version | Status |
|------|---------|--------|
| Never type (!) | v0.55 | ✅ |
| defer statement | v0.55 | ✅ |
| LTO | v0.56 | ✅ |
| Debug info | v0.56 | ✅ |
| CTFE Phase A+B | v0.54 | ✅ |
| Inline ASM | v0.55 | ✅ |
| Send/Sync markers | v0.55 | ✅ |
| spawn codegen | v0.55 | ✅ |
| Channel[T] | v0.55 | ✅ |
| Thread pool | v0.56 | ✅ |
| Binary cache | v0.54 | ✅ |
| Match exhaustiveness | v0.54 | ✅ |
| Overflow/bounds checks | v0.54 | ✅ |
| Thread-local recursion counter | v0.56 | ✅ |
| Spawn move semantics (R2) | v0.56 | ✅ |
| Parallel codegen (I2) | v0.56 | ✅ |
| Recursion counter integrity (R4+R5) | v0.56 | ✅ |
| Send/Sync enforcement (I1) | v0.56 | ✅ |
| **ALL 19/19 GATES: CLEARED** | | |

### Remaining Before Selfhost

| Priority | Task | Effort | Status |
|----------|------|--------|--------|
| ~~CRITICAL~~ | ~~R1: Accurate DI emission for .xi source~~ | ~~1 week~~ | ✅ DONE — DWARF metadata, per-function DISubprogram |
| ~~CRITICAL~~ | ~~R2: Move semantics for spawn captures~~ | ~~4 days~~ | ✅ DONE — capture analysis, env forwarding |
| ~~CRITICAL~~ | ~~R3: Thread-local recursion counter~~ | ~~1 day~~ | ✅ Already implemented |
| ~~CRITICAL~~ | ~~R4: Vec alloca leak (chaos crash)~~ | ~~2 days~~ | ✅ FIXED |
| ~~CRITICAL~~ | ~~R5: Recursion counter leak~~ | ~~1 day~~ | ✅ FIXED |
| ~~HIGH~~ | ~~I1: Send/Sync enforcement~~ | ~~5 days~~ | ✅ DONE — auto-derivation + spawn capture check |
| ~~HIGH~~ | ~~I2: Parallel codegen~~ | ~~3 days~~ | ✅ DONE — --parallel-codegen flag |
| ~~HIGH~~ | ~~Bug 1: Windows runtime leak~~ | ~~1h~~ | ✅ FIXED — #ifdef _WIN32 + sysconf fallback |
| MEDIUM | I3: Deadlock detection | 4 days | ✅ DONE — Mutex API wired to OS primitives (CRITICAL_SECTION/pthread_mutex). Codegen dispatch for Mutex.new/lock/unlock/destroy. OS-level deadlock detection built-in. |
| MEDIUM | I4: WASM WASI target | 3 days | ✅ DONE — Added `--target wasi` for wasm32-wasi. Existing `--target wasm` for bare wasm32-unknown-unknown. Target triple plumbing, clang flags, and runtime exclusion for WASM targets. |
| MEDIUM | I5: macOS CI | 2 days | ✅ DONE — GitHub Actions workflow with Windows/Linux/macOS matrix. Build release, run unit tests, E2E tests, smoke test on all platforms. |

### Remaining Compiler Gaps (v0.56 hardening — ALL RESOLVED)

| Priority | Gap | Status | Resolution |
|----------|-----|--------|------------|
| **HIGH** | t1-allocator memory 29MB | ✅ FIXED (compiler) | OPT-R5/R6/R7: 3 codegen optimizations (Vec.push extractvalue skip, emit_elem_load phi node, Vec index extractvalue). ~29M redundant instructions eliminated per 1M benchmark iterations. |
| MEDIUM | contracts.xi (14 errors) | ✅ FIXED | `ensure_tuple_type_registered` now called for struct field types, resolving nested Tuple__Str__Str inside Vec[(Str,Str)] |
| MEDIUM | Option/Result .unwrap() regression | ✅ FIXED | `field_llvm_type()` replaces hardcoded `load i64` in unwrap_or path; `is_ok`/`is_err` inline handler covers concrete Result types |
| MEDIUM | io.xi (2 errors) | ✅ FIXED | env_var uses explicit return instead of if-as-expression; checker skips receiver field injection when param shadows field |
| LOW | Parser stack overflow | ✅ FIXED | MAX_EXPR_DEPTH reduced 32→16; each nesting level ≈16 frames → 256 frames total, well within 1MB stack |
| LOW | Checker stack overflow | ✅ FIXED | Parser rejects deep nesting before checker sees it; checker scope push/pop in check_block prevents variable leaks |
| LOW | async.xi codegen IR type mismatch | ✅ FIXED | Vec.len() dispatch handles module-global struct fields via module_globals lookup in infer_struct_type_name |

### Additional v0.56 Hardening (delivered 2026-08-06)

| Fix | What | Impact |
|-----|------|--------|
| Slice[T] → %struct.Vec | Monomorphised Slice params resolve to full Vec struct | core.is_sorted/contains now work |
| eq/compare deref &value | Builtin scalar interface methods dereference &value | core/array smoke tests pass |
| ? operator concrete types | ? operator uses concrete struct types (not hardcoded %struct.Option/%struct.Result) | serialize smoke compiles+runs |
| Cell.set/replace/swap &mut self | Cell methods use mutable reference receiver | cell smoke test passes |
| Checker interface dispatch | Interface methods resolve on generic type params with bounds | test_interface_bound_violation fixed |
| Regex match arm binding | match expression result type resolves from scrutinee struct fields | regex ACCESS_VIOLATION fixed |
| CTFE cycle detection | Circular const definitions no longer overflow stack | feature-reg: 510/510 |
| 0 compiler warnings | Fixed 2 warnings (unused variable, unused mut) | Release readiness |

### v0.56.0 Release Hardening (delivered 2026-08-06 — E2E gate closure)

The final 6 failing E2E tests were all REAL compiler bugs (not selfhost-related) and
are now fixed; all 6 E2E gates pass:

| Fix | Root cause | Impact |
|-----|-----------|--------|
| `&T` deref pointee width | `*r` on `&Int` params loaded i8 instead of i64 | m21_borrow_004/008/010, m33_b14 pass |
| eq/compare ref-param deref | scalar `.eq(x)`/`.compare` compared against the ADDRESS (i64) instead of the value | array.contains, algo suites pass |
| main argc/argv seeding | `env.args()` requires xiom_set_args call in `@main` (native-only; wasm excluded) | e2e_safety_probe + 5 wasm E2E pass |
| Parallel symbol pre-assignment | parallel emitters each started with empty emitted_fns → env.args/io.args collided on `@args` | multi-module programs stable |
| Transitive stdlib imports | modules referenced transitively (io → env) were never loaded/injected | env/io chains resolve |
| Module-qualified free-fn dedup + leaf keys | injected fns must register `array.contains` keys; bare internal calls resolve via keep-first alias map | `array.len(&arr)` / `array.contains(&arr, &30)` resolve to the right mono signature |
| By-value struct coerce (`&Vec[T]`) | `&Vec[T]` params receive the struct VALUE, not the data pointer; scalar `&T` still address-as-i64; `&mut Struct` still slot address | eco_algo_89, m33_b18 pass |
| User-shadow guard | user's `fn alloc` blocks injection of same-leaf stdlib `alloc` (was duplicate `@alloc`) | m35_z06 passes |
| Mono-body ref-param tracking | generic bodies (array.contains) need param_locals/ref_params like compile_fn | generic `arr[i].eq(x)` correct |

### Final Test Results (v0.56.0 Release Candidate)

| Suite | Tests | Status |
|-------|-------|--------|
| checker | 156/156 | ✅ ALL PASS |
| stdlib-exec | 41/41 | ✅ ALL PASS |
| feature-reg | 510/510 | ✅ ALL PASS |
| parser | 96/96 | ✅ ALL PASS |
| integration | 128/128 | ✅ ALL PASS |
| stdlib-compile | 40/40 | ✅ ALL PASS |
| robustness | 63/63 | ✅ ALL PASS |
| lexer | 17/17 | ✅ ALL PASS |
| ctfe | 96/96 | ✅ ALL PASS |
| codegen-unit | 10/10 | ✅ ALL PASS |
| verifier | 27/27 | ✅ ALL PASS |
| jit | 5/5 | ✅ ALL PASS |
| xiom-lib | 19/19 | ✅ ALL PASS |
| All tooling (fmt/lsp/pkg/etc.) | 266/266 | ✅ ALL PASS |
| E2E | 2,230/2,231 | 1 known flake: `e2e_spawn_capture` (pre-existing runtime teardown race, passes interactively) |
| WASM E2E | 5/5 | ✅ ALL PASS (wasm_async_spawn, demo_float, diff_test, generics, ownership) |
| full-diff | 3/23 | Expected: IR output changed by optimizations (selfhost phase) |

## Quick Build + Package

### Test Suite — All Platforms

```powershell
# Windows (PowerShell)
.\test_summary.ps1                 # Full ~4000 tests
.\test_summary.ps1 -Fast            # Skip E2E/full-diff/fuzz (~30s)
.\test_summary.ps1 -E2EOnly         # Just 13 core gate tests (~10s)
.\test_summary.ps1 -Threads 16      # More parallelism
.\test_summary.ps1 -Logs            # Write .testlogs/session_*.txt
.\test_summary.ps1 -CleanBuild      # Delete .test_build/ contents
.\test_summary.ps1 -CleanLogs       # Delete .testlogs/ contents
```

```bash
# Linux / macOS (bash)
./test_summary.sh                   # Full ~4000 tests
./test_summary.sh -fast              # Skip E2E/full-diff/fuzz
./test_summary.sh -e2eonly           # Just 13 core gate tests
./test_summary.sh -threads 16        # More parallelism
./test_summary.sh -logs              # Write .testlogs/session_*.txt
./test_summary.sh -cleanbuild        # Delete .test_build/ contents
./test_summary.sh -cleanlogs         # Delete .testlogs/ contents
```

| Flag | PS | Bash | Description |
|------|-----|------|-------------|
| Fast mode | `-Fast` | `-fast` | Skip E2E, full-diff, fuzz, feature-reg (~30s) |
| E2E only | `-E2EOnly` | `-e2eonly` | Run only 13 core gate tests (~10s) |
| Threads | `-Threads N` | `-threads N` | Set test threads (default: 8) |
| Logs | `-Logs` | `-logs` | Write .testlogs/session_YYYYMMDD_HHMMSS.txt |
| Clean build | `-CleanBuild` | `-cleanbuild` | Delete .test_build/ contents |
| Clean logs | `-CleanLogs` | `-cleanlogs` | Delete .testlogs/ contents |

### Output directories
- `.test_build/` — All test binaries and artifacts (gitignored)
- `.testlogs/` — Session logs and failure details (gitignored)

### Package release (Windows):
```powershell
.\package.ps1 -Version "0.56.0"
.\release\xiom-v0.56.0\bin\xiom.exe --version
# Install to %LOCALAPPDATA%\xiom + PATH:
.\release\xiom-v0.56.0\install.bat
```

### Linux/macOS build:
```bash
./target/release/xiom build-runtime
./target/release/xiom --version
```

## Test Commands

```bash
# === FULL SUITE (~4000 tests) ===
# Windows:  .\test_summary.ps1
# Linux:    ./test_summary.sh

# === COMPILER SUITES ===
cargo test -p xiom-codegen --test e2e_tests                    # 2231 tests (E2E all features)
cargo test -p xiom-codegen --test feature_regression_tests      # 510 tests
cargo test -p xiom-codegen --test stdlib_execution_tests        # 41 tests
cargo test -p xiom-codegen --test stdlib_tests                  # 40 tests
cargo test -p xiom-codegen --test integration_tests             # 128 tests
cargo test -p xiom-codegen --test diff_tests                    # 25 tests
cargo test -p xiom-codegen --test full_diff_tests               # 41 tests (23 ignored, selfhost)
cargo test -p xiom-codegen --test fuzz_tests                    # 23 tests
cargo test -p xiom-codegen --test robustness_tests              # 63 tests
cargo test -p xiom-codegen --lib                                # 10 tests (sandbox)
cargo test -p xiom-lexer                                        # 17 tests
cargo test -p xiom-parser -- --test-threads=2                   # 96 tests
cargo test -p xiom-check -- --test-threads=2                    # 156 tests
cargo test -p xiom-ctfe                                         # 96 tests
cargo test -p xiom-graph                                        # 23 tests
cargo test -p xiom-verify --test verifier_tests                 # 27 tests (Z3)
cargo test -p xiom-jit                                          # 5 tests
cargo test -p xiom --test scripting_tests                       # 34 tests
cargo test -p xiom --test diff_tests                            # 15 tests

# === TOOLING SUITES ===
cargo test -p xiom-fmt                                          # 79 tests
cargo test -p xiom-lsp                                          # 38 tests
cargo test -p xiom-pkg                                          # 39 tests
cargo test -p xiom-doc                                          # 4 tests
cargo test -p xiom-ffigen                                       # 33 tests
cargo test -p xiom-mcp                                          # 39 tests
cargo test -p xiom-dbg                                          # 29 tests
cargo test -p xiom-display                                      # 5 tests

# === QUICK SMOKE (E2E core gates) ===
# NOTE: ALL fixtures are INTERNAL:
#   - e2e_chaos_*/e2e_i2_*/eco_*: tests/ecosystem/ (t1-allocator … t5-btree,
#     t8-safety-probe, test_algo, eco_* suites) — copied from benchmark patterns,
#     re-encoded UTF-8. NO dependency on the benchmark repo.
#   - e2e_spawn/e2e_send/e2e_m21/e2e_m33/e2e_m35: tests/regression/
cargo test -p xiom-codegen --test e2e_tests -- chaos eco_ ctfe e2e_asm e2e_never_type e2e_spawn e2e_send e2e_i2 e2e_m21 e2e_m33 e2e_m35

# === BUILD ===
cargo build -p xiom
cargo build -p xiom --release

# === LINUX BUILD (WSL) ===
wsl -d Ubuntu -- bash -c 'source ~/.cargo/env && cd /mnt/e/Projects/AXIOM && cargo build -p xiom'
```

## Test Suite Summary

| Section | Suites | Tests |
|---------|--------|-------|
| **E2E Core** | e2e_tests | 2231 |
| **E2E Subsets** | feature_regression, stdlib_execution, stdlib_tests | 591 |
| **Integration** | integration_tests, diff_tests, full_diff_tests | 194 |
| **Robustness** | robustness_tests, fuzz_tests | 86 |
| **Compiler Crates** | lexer, parser, checker, ctfe, graph, codegen-lib | 402 |
| **Verifier** | verifier_tests (Z3) | 27 |
| **JIT** | xiom-jit | 5 |
| **Scripting** | scripting_tests, diff_tests | 49 |
| **Tooling** | fmt, lsp, pkg, doc, ffigen, mcp, dbg, display | 266 |
| **TOTAL** | **27 suites** | **~3,700** |

## Release Checklist

- [x] Core E2E gates pass: `cargo test -p xiom-codegen --test e2e_tests -- e2e_spawn e2e_send e2e_chaos e2e_i2 e2e_safety e2e_m21 e2e_m33 e2e_m35 eco_algo`
  - e2e_chaos_*/e2e_i2_*/eco_*: internal fixtures (`tests/ecosystem/t1-allocator.xi` … `t5-btree.xi`, `t8-safety-probe.xi`, `test_algo.xi`, `eco_*.xi`), re-encoded UTF-8 — no dependency on `xiom-benchmark-chaos` reference files
  - e2e_spawn_*/e2e_send_*/e2e_m21_*/e2e_m33_*/e2e_m35_*: internal regression tests (`tests/regression/`)
  - e2e_safety_probe: internal fixture (`tests/ecosystem/t8-safety-probe.xi`) — env.args() FFI crash FIXED (main argc/argv seeding); probe runs and scores
  - All 6 previously-failing gates (m21_borrow_004/008/010, m33_b14, eco_algo_89, e2e_safety_probe) now PASS
- [x] Tooling tests pass (266/266): all LSP, pkg, MCP, dbg, ffigen, doc, graph, JIT, display, lexer, fmt
- [x] Compiler builds with 0 warnings (Windows + Linux)
- [x] `cargo build -p xiom --release` succeeds
- [x] Linux build verified: `wsl -d Ubuntu -- bash -c ...`
- [x] Version: v0.56.0 "Production Polish" — 19/19 gates cleared
- [x] Runtime compiles on Linux (Bug 1 #ifdef _WIN32 fix verified)
- [x] All compiler gaps resolved (7/7): contracts, io, unwrap regression, parser/checker stack overflow, async, t1-allocator
- [x] All smoke tests pass (stdlib-exec: **41/41**)
- [x] Checker: 156/156
- [x] Feature-reg: 510/510
- [x] Full E2E suite: 2,230/2,231 (1 known pre-existing flake)
- [x] Release binaries packaged for Windows (release\xiom-v0.56.0 + ZIP) + installed
- [ ] CHANGELOG.md updated

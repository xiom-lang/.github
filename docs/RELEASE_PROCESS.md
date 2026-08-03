# XIOM Release Process

## v0.56.0-pre "Production Polish" — ACTIVE

**Test baseline: 24/24 E2E (100% pass) | 111+ compiler hardening commits**
**Target platforms: Windows x64, Linux x64 (WSL build verified)**
**Selfhost gate: 17/17 CLEARED (all Phase A criticals + I2 parallel codegen DONE)**

### Key Features (v0.54 → v0.56)

| Version | Features |
|---------|----------|
| **v0.54** | CTFE Phase A+B, Binary cache (`--cache`), Parallel parse (`--parallel`), Thread-safe SyncRegistry, `const { }` blocks, Turbofish `::<T>()`, Builtins (align_of, type_id, field_offset, is_signed), Match exhaustiveness (S2), Overflow/bounds/null checks (S1), `--strict-exhaustive` |
| **v0.55** | OrcJIT engine (`--jit`), Hot reload watcher, Inline ASM `asm()`, Never type `!`, `defer` statement, `spawn` codegen, Channel[T] ring buffer, Send/Sync markers, `build-runtime` command |
| **v0.56** | LTO `--lto`, Debug info `--debug`/`-g`, Lazy JIT `--jit --lazy`, Thread pool (work-stealing), Parallel codegen `--parallel-codegen`, Spawn move semantics (R2), Recursion counter fixes (R4+R5), 0 compiler warnings all crates |

### Selfhost Gate Status (v0.56.0-pre)

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
| **ALL 17 GATES: CLEARED** | | |

### Remaining Before Selfhost

| Priority | Task | Effort | Status |
|----------|------|--------|--------|
| CRITICAL | R1: Accurate DI emission for .xi source | 1 week | ❌ |
| HIGH | I1: Send/Sync enforcement in checker | 5 days | 🔄 IN PROGRESS |
| HIGH | I3: Deadlock detection | 4 days | ❌ |

## Quick Build + Package

**All platforms — full test suite:**
```bash
# Windows (PowerShell)
.\test_summary.ps1

# Linux / macOS (bash)
./test_summary.sh
```

**Windows (PowerShell):**
```powershell
# 1. Run full test suite (~4000 tests)
.\test_summary.ps1

# 2. Package release
.\package.ps1 -Version "0.56.0"

# 3. Verify
.\release\xiom-v0.56.0\bin\xiom.exe --version
```

**Linux / macOS (bash):**
```bash
# 1. Run full test suite (~4000 tests)
./test_summary.sh

# 2. Build runtime
./target/release/xiom build-runtime

# 3. Verify
./target/release/xiom --version
```

## Test Commands

```bash
# === FULL SUITE (~4000 tests) ===
# Windows:  .\test_summary.ps1
# Linux:    ./test_summary.sh

# === COMPILER SUITES ===
cargo test -p xiom-codegen --test e2e_tests                    # 2212 tests (E2E all features)
cargo test -p xiom-codegen --test feature_regression_tests      # 24 tests
cargo test -p xiom-codegen --test stdlib_execution_tests        # 128 tests
cargo test -p xiom-codegen --test stdlib_tests                  # 40 tests
cargo test -p xiom-codegen --test integration_tests             # 491 tests
cargo test -p xiom-codegen --test diff_tests                    # 25 tests
cargo test -p xiom-codegen --test full_diff_tests               # 41 tests
cargo test -p xiom-codegen --test fuzz_tests                    # 23 tests
cargo test -p xiom-codegen --test robustness_tests              # 63 tests
cargo test -p xiom-codegen --lib                                # 10 tests (sandbox)
cargo test -p xiom-lexer                                        # 17 tests
cargo test -p xiom-parser -- --test-threads=2                   # 96 tests
cargo test -p xiom-check -- --test-threads=2                    # 156 tests
cargo test -p xiom-ctfe                                         # CTFE tests
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

# === QUICK SMOKE (26 E2E core gates) ===
cargo test -p xiom-codegen --test e2e_tests -- chaos eco_ ctfe e2e_asm e2e_never_type e2e_spawn e2e_send e2e_i2

# === BUILD ===
cargo build -p xiom
cargo build -p xiom --release

# === LINUX BUILD (WSL) ===
wsl -d Ubuntu -- bash -c 'source ~/.cargo/env && cd /mnt/e/Projects/AXIOM && cargo build -p xiom'
```

## Test Suite Summary

| Section | Suites | Tests |
|---------|--------|-------|
| **E2E Core** | e2e_tests | 2212 |
| **E2E Subsets** | feature_regression, stdlib_execution, stdlib_tests | 192 |
| **Integration** | integration_tests, diff_tests, full_diff_tests | 557 |
| **Robustness** | robustness_tests, fuzz_tests | 86 |
| **Compiler Crates** | lexer, parser, checker, ctfe, graph, codegen-lib | 312 |
| **Verifier** | verifier_tests (Z3) | 27 |
| **JIT** | xiom-jit | 5 |
| **Scripting** | scripting_tests, diff_tests | 49 |
| **Tooling** | fmt, lsp, pkg, doc, ffigen, mcp, dbg, display | 266 |
| **TOTAL** | **27 suites** | **~3,700** |

## Release Checklist

- [ ] Full test suite passes: `.\test_summary.ps1` (Windows) / `./test_summary.sh` (Linux/macOS)
- [ ] E2E core gates pass (26/26): `cargo test -p xiom-codegen --test e2e_tests -- chaos eco_ ctfe e2e_asm e2e_never_type e2e_spawn e2e_send e2e_i2`
- [ ] All JIT tests pass (5/5): `cargo test -p xiom-jit`
- [ ] `xiom build-runtime` succeeds
- [ ] `cargo build -p xiom --release` succeeds (0 warnings)
- [ ] Linux build verified: `wsl -d Ubuntu -- bash -c 'source ~/.cargo/env && cd /mnt/e/Projects/AXIOM && cargo build -p xiom'`
- [ ] Version bumped in all Cargo.toml files
- [ ] SESSION.md updated with handoff
- [ ] AI_CONTEXT.md version updated
- [ ] COMPILER_ARCHITECTURE.md updated
- [ ] All plan docs (CTFE, ORCJIT, THREADING, SAFETY, ROADMAP) audited
- [ ] Release binaries packaged for Windows + Linux

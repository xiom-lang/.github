# XIOM Release Process

## v0.55.0 "Safety Foundation" — CURRENT

**Test baseline: 2197 (2197/2197 E2E, 100% pass) | 101+ compiler hardening commits**
**Target platforms: Windows x64, Linux x64**
**Selfhost gate: ALL CLEARED (Never, defer, LTO, debug info, CTFE, ASM, Send/Sync)**

### Key Features (v0.54 → v0.56)

| Version | Features |
|---------|----------|
| **v0.54** | CTFE Phase A+B, Binary cache (`--cache`), Parallel parse (`--parallel`), Thread-safe SyncRegistry, `const { }` blocks, Turbofish `::<T>()`, Builtins (align_of, type_id, field_offset, is_signed), Match exhaustiveness (S2), Overflow/bounds/null checks (S1), `--strict-exhaustive` |
| **v0.55** | OrcJIT engine (`--jit`), Hot reload watcher, Inline ASM `asm()`, Never type `!`, `defer` statement, `spawn` codegen, Channel[T] ring buffer, Send/Sync markers, `build-runtime` command |
| **v0.56** | LTO `--lto`, Debug info `--debug`/`-g`, Lazy JIT `--jit --lazy`, Thread pool (work-stealing), AI_CONTEXT.md full update |

### Selfhost Gate Status

| Gate | Status |
|------|--------|
| Never type (!) | ✅ v0.55 |
| defer statement | ✅ v0.55 |
| LTO | ✅ v0.56 |
| Debug info | ✅ v0.56 |
| CTFE Phase A+B | ✅ v0.54 |
| Inline ASM | ✅ v0.55 |
| Send/Sync | ✅ v0.55 |
| spawn codegen | ✅ v0.55 |
| Channel[T] | ✅ v0.55 |
| Binary cache | ✅ v0.54 |
| Match exhaustiveness | ✅ v0.54 |
| Overflow/bounds checks | ✅ v0.54 |
| **ALL GATES: CLEARED** | |

### Remaining Before Selfhost (Phase A)

| Priority | Task | Effort |
|----------|------|--------|
| CRITICAL | Move semantics for spawn captures | 4 days |
| CRITICAL | Thread-local recursion counter | 1 day |
| CRITICAL | asm output/input constraint wiring | 3 days |
| CRITICAL | Accurate DI emission for .xi source | 1 week |
| HIGH | Send/Sync enforcement in checker | 5 days |

### Remaining Before Selfhost (Phase B)

| Priority | Task | Effort |
|----------|------|--------|
| HIGH | Parallel codegen (function-level rayon) | 3 days |
| HIGH | Deadlock detection (static lock ordering) | 4 days |
| HIGH | Spawn wrapper capture layout | 3 days |
| MEDIUM | DWARF/PDB from .xi source | 1 week |

## Quick Build + Package

**Windows (PowerShell):**
```powershell
# 1. Run full test suite
cargo test -p xiom-codegen --test e2e_tests

# 2. Package release
.\package.ps1 -Version "0.55.0"

# 3. Verify
.\release\xiom-v0.55.0\bin\xiom.exe --version
```

**Linux / macOS (bash):**
```bash
# 1. Run full test suite
cargo test -p xiom-codegen --test e2e_tests

# 2. Build runtime
./target/release/xiom build-runtime

# 3. Verify
./target/release/xiom --version
```

## Test Commands

```bash
# Full E2E suite (20 min)
cargo test -p xiom-codegen --test e2e_tests

# Eco + CTFE + ASM + Never + Spawn (7s)
cargo test -p xiom-codegen --test e2e_tests -- eco_ ctfe e2e_asm e2e_never_type e2e_spawn_basic

# JIT tests
cargo test -p xiom-jit

# Build
cargo build -p xiom
```

## Release Checklist

- [ ] All E2E tests pass (2197/2197)
- [ ] All JIT tests pass (5/5)
- [ ] `xiom build-runtime` succeeds
- [ ] `cargo build -p xiom --release` succeeds
- [ ] Version bumped in all Cargo.toml files
- [ ] AI_CONTEXT.md version updated
- [ ] COMPILER_ARCHITECTURE.md updated
- [ ] All plan docs (CTFE, ORCJIT, THREADING, SAFETY) audited
- [ ] Release binaries packaged for Windows + Linux

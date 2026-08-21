# XIOM -- Releases

## v0.48.5 "Phoenix" -- 2026-07-20

**768/768 tests. 49/49 gaps closed. Z3 verification. Loop invariants. 5e Advanced Compilation complete.**

### What's New

**5e Advanced Compilation (all sub-phases complete)**
- **Typed Pointer IR (5e.1):** C struct field access, `sizeof[T]()` intrinsic, Int8/Bool C layout
- **Fn-Pointer Types (5e.2):** C callback lowering, `Int as fn(T)->R` casts
- **Multi-Package Build (5e.3):** Cross-package `use` + `extern "C"` resolution, grandparent directory catalog, LSP walk-up project root detection
- **Distinct Newtype (5e.4):** Type aliases are checker-distinct for handle safety

**5f Z3 Contract Verification**
- **Body encoding:** SSA lowering with `declare-const` + `assert` per let/return
- **Correct type map:** `Int32`->`BV(32)`, `Float64`->`FloatingPoint(11,53)`, `Bool`->`Bool`
- **Side-condition VCs:** Div-by-zero (`X7004`), overflow, bounds, null -- as named asserts
- **Contract composition:** Uninterpreted functions + contract axioms for modular verification
- **Loop invariants:** `while cond invariant: expr { ... }` syntax + VC generation (`X7006`)
- **z3 auto-detection:** `Z3_PATH` env, common paths, PATH -- graceful fallback
- **Counterexample extraction:** Model parsing for z3 4.13.4 raw `(` format
- **15 verifier tests:** SMT generation + z3 integration + parser unit tests

**Runtime Safety (compiler-inserted)**
- Division by zero -> `div_zero_trap` with `llvm.trap()`
- Recursion depth -> `xiom_recursion_counter` with `llvm.trap()`
- Vec bounds checks -> `icmp sge/slt` + conditional branch

**Compiler Hardening**
- **RC failure FIXED (3 bugs):** `size_of` nested generic args, `Expr::As` pointer-to-pointer cast, `Layout.new` cross-module resolution
- **CG-02 Float32 global init:** 17-digit scientific notation for exact f32 roundtrip
- **CG-01 Float Vec reads:** Verified fixed (5c.29 -- `bitcast` instead of `sitofp`)
- **G-20 bare-field reads:** `type_meta` fallback for catalog-loaded struct fields
- **Benchmark suite:** All 30 modules compile, bare-field reads auto-handled via `type_meta`
- **Stdlib freeze FIXED:** Grandparent `source_dir` guard prevents scanning system directories

**Tooling**
- **LSP:** References + Rename, catalog-aware diagnostics, walk-up project root
- **DAP Debugger:** VS Code debug config, variable inspection from GDB locals
- **Package Manager:** `xiom.lock` lockfile generation
- **Verifier:** `xiom-verify --check --z3-path` with structured results

### Status
| Gate | Result |
|------|--------|
| E2E tests | 106/106 |
| Feature regression | 117/117 |
| Stdlib execution | 41/41 |
| Stdlib compilation | 40/40 |
| Verifier | 15/15 |
| Integration | 119/119 |
| Tooling | 229/229 |
| **TOTAL** | **768/768** |

### Install
```powershell
# From project root (Windows)
.\package.ps1 -Version "0.48.5"
# Release tag: 768/768 tests
```

### Download

| Platform | Binary | Size |
|----------|--------|------|
| Windows x64 | `target/release/xiom.exe` | 3.7 MB |

### What's New (5c.29-5c.30 Production Hardening)
- **Deterministic builds** -- same IR -> byte-identical binary (fixed `.ll` name + `/Brepro`)
- **Container-handle convention** -- `Vec[T]` fields use heap-boxed handles (no more stack corruption / ACCESS_VIOLATION)
- **Real element widths** -- 1/2/4/8-byte stores for `Vec[Float32]`, `Vec[Int32]`, etc.
- **Method ABI aligned** -- ecosystem-style methods (`fn T.method(h: &T, ...)`) no longer shift arguments
- **Inline Vec.insert / Vec.remove** -- llvm.memmove builtins
- **Enum payload conventions** -- per-variant types preserved, float payloads use raw bits
- **Vec-of-struct element typing** -- `Vec[Point2D].push()` / `.pop()` / `.get()` with correct layout sizes
- **Implicit-self method calls (G-10)** -- `init()` inside `fn GrpcClient.init()` resolves to `self.init()`
- **Int -> unsigned coercion (G-04)** -- `var x: UInt8 = 255` type-checks
- **String concatenation** -- `a + b` emits `@xiom_str_concat`

### Status
| Gate | Result |
|------|--------|
| E2E tests | 101/101 |
| Parser tests | 47/47 |
| Checker tests | 74/74 |
| Integration | 119/119 |
| Feature regression | 55/55 |
| Deterministic builds | SHA256-identical |

### Install
```powershell
# From project root (Windows)
.\install.ps1 -BinaryPath .\target\release

# Or manually
Copy-Item target\release\xiom.exe C:\Users\$env:USERNAME\AppData\Local\xiom\bin\xiom.exe
```
```bash
# macOS / Linux
./install.sh ./target/release
```

### Bug Fixes (11 crash bugs resolved)
NET, DB, VECTOR, HTTP, SQLITE, JSON, FULL, CRYPTO, VOS, TFR, TEST

### Commits
```
480eb37 chore: v0.46.0 release
e0c4d96 docs: ROADMAP v0.45.5
e3f5291 fix(checker,codegen): 5c.30 G-10, G-04, constructor regression
c6f03c5 fix(checker,codegen): 5c.30 G-10 implicit-self
666d6ea docs: SESSION.md + ROADMAP.md
88badd4 fix(codegen): 5c.30 payload tracking - CRYPTO
5494cd2 fix(tests): 5c.30 FULL
10570e9 fix(codegen): 5c.30 enum payloads - JSON
ab3fcb0 fix(codegen): 5c.30 Vec-of-struct - VOS
c603de6 fix(codegen): 5c.30 &local.field - TFR
7cf7a5b fix(codegen): 5c.29 container handles - NET/DB/VECTOR/HTTP/SQLITE
ab588e2 fix(xiom): 5c.29 deterministic builds
```

---

## v0.12.0 "Production" -- 2026-07-01

**234 tests. Self-hosted compiler at 90%+ coverage. Full toolchain.**

### Download

| Platform | Binary | Size |
|----------|--------|------|
| Windows x64 | [xiom-v0.12.0-windows-x64.zip]() | ~5MB |
| Linux x64 | [xiom-v0.12.0-linux-x64.tar.gz]() | ~5MB |

### What's New
- Self-hosted compiler emits real LLVM IR for 19/21 examples
- Full derive codegen (Eq, Clone, Hash, Ord, Display) with GEP, fcmp, zext
- Contract support with @llvm.trap and invariant_check
- Match dispatch with icmp chains
- C runtime: 50+ functions for file I/O, string interning, IR emission
- NSIS + batch installer for Windows
- VS Code extension with syntax highlighting + LSP

### Previous Releases

See [COMPILER_VERSIONS.md](docs/COMPILER_VERSIONS.md) for complete history.

## Installation

```powershell
# Windows (pre-built)
dist\install.bat
# Restart terminal, then: xiom --help

# From source
cargo build --release -p xiom
```

## v0.51.0 "Production Hardening" -- 2026-07-25

**1041/1041 tests (681 compiler + 360 tooling). M1-M12 complete. P0+P1 closed. M14.3-M14.7 done.**

### Key Features Delivered

**Language & Runtime**
- `Str.slice(start, end)` / `Str.starts_with(prefix)` / `Str.ends_with(suffix)` -- scripting ergonomics
- Iterator adapter parity: `step_by`, `take_while`, `skip_while`, `inspect` (23 total)
- `--check` auto-detects script-like files and applies implicit main (G4 resolved)
- Parser-level multi-line declaration detection for scripting mode (M12.2)
- `io.read_line()` fixed -- no longer returns Result (module export map key collision fix)

**Formatter (M8)**
- `extern` blocks now round-trip correctly (previously silently dropped)
- `unsafe { ... }` blocks format with proper braces (previously lost)
- 44/44 formatter tests (3 new round-trip tests)

**Package Manager (M6)**
- `xiom pkg publish` -- creates tarball + multipart upload to registry
- `xiom pkg publish` previously only sent metadata; now uploads actual package

**LSP (M3.3)**
- Rename and codeAction tests added (11->15 tests)

**Quality (M14)**
- 6 dead code items removed (~239 lines across 5 crates)
- 5 `unreachable!()` calls given diagnostic strings
- 3 `unsafe` blocks documented with SAFETY comments
- 4 bare `.unwrap()` replaced with `.expect()` invariant messages
- LLVM type constants extracted to `llvm_consts.rs` (8 files)
- Rustdoc comments for Checker, BorrowChecker, CheckedType, FnSig, CheckError

**Documentation (M14.5)**
- New: `docs/language/reference.md` -- complete language reference (350+ lines)
- New: `docs/language/pattern-matching.md` -- match, if let, while let, ? operator
- RELEASE_PROCESS.md updated for v0.51.0

**Stdlib Contracts (M2)**
- 10 modules with safety contracts: iter, compress, log, bench, reflect, serialize, thread, contracts, path, async

---

## v0.50.0 "Production Edition" -- 2026-07-24

**1039/1039 tests. Scripting mode, JIT, REPL. 40-module stdlib.**

### Key Features

- **Scripting mode:** `xiom run`, `xiom --standalone`, `xiom repl`, `--watch`, shebang (`#!/usr/bin/env xiom`)
- **True JIT compilation** via libloading -- scripts compile to shared library and execute in-process
- **49 scripting/diff tests**
- Implicit main wrapping for script-like files
- String concatenation operator `+` for scripting ergonomics

---

## v0.49.0 "Expansion" -- 2026-07-22

**900+ tests. Z3 verification. Loop invariants. 5e Advanced Compilation.**

### Key Features

- **5e.1 Typed Pointer IR:** C struct field access, `sizeof[T]()` intrinsic
- **5e.2 Fn-Pointer Types:** C callback lowering
- **5e.3 Multi-Package Build:** Cross-package `use` + `extern "C"` resolution
- **5e.4 Distinct Newtype:** Type aliases are checker-distinct for handle safety
- **5f Z3 Contract Verification:** SMT-based body encoding, side-condition VCs, loop invariants, counterexample extraction

---

## Building from Source

Requirements: Rust 1.85+, LLVM/clang 18+

```powershell
git clone https://github.com/XIOM-lang/XIOM.git
cd XIOM
cargo test          # 1041 tests
cargo build -p xiom --release
```

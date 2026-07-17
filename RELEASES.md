# XIOM — Releases

## v0.46.0 "Production" — 2026-07-17

**101/101 e2e. Deterministic builds. All P0 gaps resolved. 6 P1 gaps closed.**

### Download

| Platform | Binary | Size |
|----------|--------|------|
| Windows x64 | `target/release/xiomc.exe` | 3.7 MB |

### What's New (5c.29–5c.30 Production Hardening)
- **Deterministic builds** — same IR → byte-identical binary (fixed `.ll` name + `/Brepro`)
- **Container-handle convention** — `Vec[T]` fields use heap-boxed handles (no more stack corruption / ACCESS_VIOLATION)
- **Real element widths** — 1/2/4/8-byte stores for `Vec[Float32]`, `Vec[Int32]`, etc.
- **Method ABI aligned** — ecosystem-style methods (`fn T.method(h: &T, ...)`) no longer shift arguments
- **Inline Vec.insert / Vec.remove** — llvm.memmove builtins
- **Enum payload conventions** — per-variant types preserved, float payloads use raw bits
- **Vec-of-struct element typing** — `Vec[Point2D].push()` / `.pop()` / `.get()` with correct layout sizes
- **Implicit-self method calls (G-10)** — `init()` inside `fn GrpcClient.init()` resolves to `self.init()`
- **Int → unsigned coercion (G-04)** — `var x: UInt8 = 255` type-checks
- **String concatenation** — `a + b` emits `@xiom_str_concat`

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
Copy-Item target\release\xiomc.exe C:\Users\$env:USERNAME\AppData\Local\xiom\bin\xiomc.exe
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
ab588e2 fix(xiomc): 5c.29 deterministic builds
```

---

## v0.12.0 "Production" — 2026-07-01

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
cargo build --release -p xiomc
```

## Building from Source

Requirements: Rust 1.75+, LLVM/clang 15+

```powershell
git clone https://github.com/NgonArt_STUDIO/XIOM.git
cd XIOM
cargo test          # 234 tests
cargo build -p xiomc --release
```

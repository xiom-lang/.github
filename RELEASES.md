# XIOM — Releases

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

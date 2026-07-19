# XIOM Release Process

## Quick Build + Package

**Windows (PowerShell):**
```powershell
# 1. Run full test suite (prints single-line summary for release tags)
.\test_summary.ps1
# Output: "Release tag: 445/445 tests"

# 2. Package release
.\package.ps1 -Version "0.47.0"
# -> Cargo.toml bumped to 0.47.0
# -> xiomc --version reports v0.47.0
# -> release/xiom-v0.47.0-windows-x64.zip
```

**Linux / macOS (bash):**
```bash
# 1. Run full test suite
./test_summary.sh
# Output: "Release tag: 445/445 tests"

# 2. Package release
./package.sh 0.47.0
# -> release/xiom-v0.47.0-linux-x64.tar.gz
```
```bash
# 1. Run full test suite
cargo test --all

# 2. Package release
./package.sh 0.47.0

# Output:
#   release/xiom-v0.47.0/             (release folder)
#   release/xiom-v0.47.0-linux-x64.tar.gz
```

## Release Structure

```
release/xiom-v{version}/
├── bin/
│   ├── xiomc.exe         (main compiler)
│   ├── xiom-fmt.exe      (formatter)
│   ├── xiom-doc.exe      (documentation generator)
│   ├── xiom-ffigen.exe   (FFI generator)
│   ├── xiom-pkg.exe      (package manager)
│   ├── xiom-lsp.exe      (language server)
│   └── xiom-icon.ico
├── lib/                   (standard library .xi files)
├── runtime/               (C runtime sources)
├── install.bat / install.sh
└── README.txt
```

## Customizing the Version Banner

The release tagline (`"Production"`, `"441/441 tests"`) is baked into the binary at
compile time via `env!("XIOM_RELEASE_TAG")` and `env!("XIOM_RELEASE_STATS")`.
Set these environment variables **before** running the package script:

**Windows:**
```powershell
$env:XIOM_RELEASE_TAG   = "Production - Phoenix"
$env:XIOM_RELEASE_STATS = "COMPILER  519/519 tests | TOOLING   229/229 | TOTAL: 748/748 | Stable | MCP"
.\package.ps1 -Version "0.48.2"
```

**Linux/macOS:**
```bash
XIOM_RELEASE_TAG="Stable" \
XIOM_RELEASE_STATS="441/441 tests, zero warnings" \
./package.sh 0.47.1
```

If unset, defaults are `"Production"` and `"441/441 tests, zero warnings"`.
The version number comes from `Cargo.toml` (auto-bumped by the script).
The binary will report: `XIOM Compiler v0.47.1 "Stable" - 441/441 tests, zero warnings`

## Installing

**Windows (GUI):** Double-click `install.bat` in the release folder.

**Windows (manual):**
```powershell
Copy-Item release\xiom-v0.47.0\bin\xiomc.exe -Destination "$env:LOCALAPPDATA\xiom\bin\xiomc.exe" -Force
```

**macOS / Linux:**
```bash
./install.sh ./target/release
```

## Version Bump Checklist

1. Update `crates/xiomc/Cargo.toml` → `version = "X.Y.Z"`
2. Update `crates/xiomc/src/main.rs` → version string + help banner
3. Update `docs/ROADMAP.md` → Current version
4. Update `RELEASES.md` → Add release notes
5. Commit: `chore: vX.Y.Z release`
6. Build + package (see above)
7. Tag: `git tag vX.Y.Z`
8. Archive the ZIP file

## Smoke Test After Build

```powershell
.\target\release\xiomc.exe --version
.\target\release\xiomc.exe --run tests\ecosystem\test_net.xi
# Should exit 0 for all ecosystem tests
```

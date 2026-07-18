# XIOM Release Process

## Quick Build + Package

**Windows (PowerShell):**
```powershell
# 1. Run full test suite
cargo test --all

# 2. Package release
.\package.ps1 -Version "0.47.0"

# Output:
#   release/xiom-v0.47.0/              (release folder)
#   release/xiom-v0.47.0-windows-x64.zip  (portable ZIP)
```

**Linux / macOS (bash):**
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
├── install.bat            (Windows installer)
└── README.txt
```

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

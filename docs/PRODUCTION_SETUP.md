# XIOM Production Infrastructure Setup

> **SUPERSEDED 2026-09-16:** replaced by [`RELEASE_INFRA_PLAN.md`](RELEASE_INFRA_PLAN.md).
> This file is kept for history only; its org name and registry design are stale
> (the target org is github.com/xiom-lang).

**Domain:** xiom-lang.org | **Registry:** registry.xiom-lang.org | **Git Org:** github.com/xiom-language
**Date:** 2026-07-20 | **Version:** v0.49.8

---

## 1. REPOSITORY SPLIT PLAN

Split the monolithic `AXIOM` repo into focused repositories under `github.com/xiom-language/`.

### 1.1 Core Compiler -- `xiom-lang/xiom`

**Contents:** The entire Rust workspace -- compiler + toolchain + tests.

```
xiom-lang/xiom/
|-- Cargo.toml              # workspace root (14 crates)
|-- Cargo.lock
|-- crates/                 # all 14 crates (xiom, xiom-codegen, xiom-check, etc.)
|-- tests/                  # test suites (.xi test files)
|-- examples/               # example .xi programs
|-- build/                  # build artifacts dir (.gitignored)
|-- .github/workflows/ci.yml
|-- README.md
|-- LICENSE (MIT OR Apache-2.0)
`-- .gitignore
```

**Branch strategy:** `main` (stable), `feat/*` (features), `fix/*` (bugs)

### 1.2 Standard Library -- `xiom-lang/stdlib`

```
xiom-lang/stdlib/
|-- package.xi              # stdlib manifest
|-- libc.xiom-bind          # libc/libm FFI bindings
|-- xiom/                   # 40 stdlib modules
|   |-- alloc.xi, array.xi, string.xi, ...
|   `-- ai_prompt.txt
|-- runtime/                # C/ASM runtime
|   |-- xiom_runtime.c
|   |-- xiom_hot_reload.c
|   |-- async_runtime.c
|   |-- simd_runtime.c
|   |-- sha256_sw.c, sha256_sw.h
|   |-- context_switch.asm
|   |-- crypto_x86_64.asm
|   |-- mem_x86_64.asm
|   `-- hot_reload_demo.xi
|-- .github/workflows/ci.yml
|-- README.md
`-- LICENSE
```

**Versioning:** Independent semver. Tagged releases trigger registry updates.

### 1.3 Ecosystem Packages -- One Repo Per Package

Each ecosystem package gets its own repo under `xiom-lang/` with naming convention `xiom-<name>`:

```
xiom-lang/xiom-vulkan       xiom-lang/xiom-http
xiom-lang/xiom-db           xiom-lang/xiom-crypto
xiom-lang/xiom-vector       xiom-lang/xiom-imgui
... (all 56 packages)
```

**Each package repo contains:**
```
xiom-lang/xiom-<name>/
|-- package.xi              # package manifest (required)
|-- src/                    # XIOM sources
|-- tests/                  # package tests
|-- examples/               # usage examples
|-- bridge/                 # C/C++ bridge code (if applicable)
|-- *.xiom-bind             # FFI binding specs (if applicable)
|-- docs/                   # package-specific docs
|-- README.md
|-- .github/workflows/ci.yml
`-- LICENSE
```

**Package manifest (`package.xi`):**
```xiom
package xiom.<name>
version = "0.1.0"
authors = ["Author <email>"]
description = "XIOM bindings for ..."
repository = "https://github.com/xiom-language/xiom-<name>"
dependencies = {
  xiom.stdlib = "0.49"
}
```

### 1.4 Documentation & Website -- `xiom-lang/docs`

```
xiom-lang/docs/
|-- docs/                   # all .md documentation
|   |-- language/           # language reference
|   |   `-- stdlib/         # per-module stdlib docs
|   |-- rust/               # rustc lessons
|   |-- z3/                 # Z3 integration docs
|   |-- ecosystem-audit/    # gap registry
|   `-- error_codes/        # error code reference
|-- website/                # static website
|   |-- index.html
|   |-- style.css
|   |-- playground/         # web playground
|   `-- docs/               # mirrored docs
|-- specs/                  # language + build specs
|-- .github/workflows/pages.yml  # deploy to GitHub Pages
`-- README.md
```

### 1.5 Editor Integrations -- One Per Editor

```
xiom-lang/vscode-xiom       # VS Code extension
xiom-lang/neovim-xiom       # Neovim plugin
xiom-lang/emacs-xiom        # Emacs mode
xiom-lang/helix-xiom        # Helix config
xiom-lang/jetbrains-xiom    # JetBrains plugin
xiom-lang/sublime-xiom      # Sublime Text config
```

### 1.6 Tools & Infrastructure

```
xiom-lang/tools             # build scripts, packaging, signing
xiom-lang/registry          # package registry server
xiom-lang/playground        # web playground (standalone)
xiom-lang/homebrew-xiom     # Homebrew formula (macOS)
```

---

## 2. REGISTRY SETUP -- registry.xiom-lang.org

### 2.1 Server Architecture

The registry is a simple static JSON index + REST API. Hosted on your Contabo VPS with HestiaCP.

**Directory structure on VPS:**
```
/home/lefteris/web/registry.xiom-lang.org/
|-- public_html/
|   |-- index.json          # master package index
|   |-- packages/           # per-package metadata
|   |   |-- xiom.stdlib/
|   |   |   |-- index.json  # versions + metadata
|   |   |   `-- 0.49.8/
|   |   |       |-- package.json
|   |   |       `-- package.tar.gz
|   |   |-- xiom.vulkan/
|   |   |   `-- ...
|   |   `-- ...
|   |-- api/                # optional API endpoints
|   |   |-- search.php
|   |   `-- publish.php    # token-authenticated
|   `-- .htaccess           # CORS + caching headers
```

### 2.2 Registry Index Format (`index.json`)

```json
{
  "registry": "xiom-lang.org",
  "version": "1",
  "updated": "2026-07-20T15:00:00Z",
  "packages": {
    "xiom.stdlib": {
      "name": "xiom.stdlib",
      "description": "XIOM Standard Library",
      "repository": "https://github.com/xiom-language/stdlib",
      "latest": "0.49.8",
      "versions": ["0.49.8", "0.49.0", "0.48.6"]
    },
    "xiom.vulkan": {
      "name": "xiom.vulkan",
      "description": "Vulkan graphics API bindings",
      "repository": "https://github.com/xiom-language/xiom-vulkan",
      "latest": "0.5.0",
      "versions": ["0.5.0", "0.4.0"]
    }
  }
}
```

### 2.3 Package Version Format (`packages/<pkg>/<ver>/package.json`)

```json
{
  "name": "xiom.stdlib",
  "version": "0.49.8",
  "manifest": {
    "package": "xiom.stdlib",
    "version": "0.49.8",
    "dependencies": {}
  },
  "files": ["alloc.xi", "array.xi", "string.xi", "..."],
  "checksum": "sha256:abc123...",
  "published": "2026-07-20T15:00:00Z",
  "download": "/packages/xiom.stdlib/0.48.9/package.tar.gz"
}
```

### 2.4 Server Setup (HestiaCP / Apache)

**HestiaCP steps:**
1. Add domain `registry.xiom-lang.org` via HestiaCP web panel
2. Enable SSL via Let's Encrypt (built into HestiaCP)
3. Set document root to `public_html/`

**.htaccess** (in `public_html/`):
```apache
Header set Access-Control-Allow-Origin "*"
Header set Access-Control-Allow-Methods "GET, OPTIONS"
Header set Cache-Control "public, max-age=300"

# Gzip JSON responses
AddOutputFilterByType DEFLATE application/json

# Pretty URLs for API
RewriteEngine On
RewriteRule ^api/v1/search$ api/search.php [QSA,L]
RewriteRule ^api/v1/publish$ api/publish.php [L]
```

### 2.5 Publishing Workflow

**From CI/CD (GitHub Actions):**
```yaml
# In each package repo's .github/workflows/publish.yml
- name: Publish to registry
  run: |
    curl -X POST https://registry.xiom-lang.org/api/v1/publish \
      -H "Authorization: Bearer ${{ secrets.REGISTRY_TOKEN }}" \
      -F "package=@package.xi" \
      -F "archive=@package.tar.gz"
```

**Manual publish (from CLI):**
```bash
xiom pkg publish --registry https://registry.xiom-lang.org
```

---

## 3. CI/CD SETUP -- GitHub Actions

### 3.1 `xiom-lang/xiom` (Compiler)

```yaml
# .github/workflows/ci.yml
name: XIOM CI
on: [push, pull_request]

jobs:
  test-windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - run: choco install llvm -y
      - run: cargo test --all
      - run: .\test_summary.ps1

  release:
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - run: choco install llvm -y
      - run: cargo build --release --workspace
      - run: .\package.ps1 -Version ${{ github.ref_name }}
      - uses: actions/upload-artifact@v4
        with:
          name: xiom-${{ github.ref_name }}-windows-x64
          path: release/
```

### 3.2 `xiom-lang/stdlib` (Standard Library)

```yaml
name: Stdlib CI
on: [push, pull_request]

jobs:
  test:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install xiom
        run: |
          curl -L https://registry.xiom-lang.org/packages/xiom.stdlib/latest/xiom-windows.zip -o xiom.zip
          Expand-Archive xiom.zip -DestinationPath C:\xiom
          echo "C:\xiom\bin" >> $env:GITHUB_PATH
      - run: xiom --check-only stdlib/xiom/*.xi

  publish:
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          tar -czf package.tar.gz xiom/ runtime/
          curl -X POST https://registry.xiom-lang.org/api/v1/publish \
            -H "Authorization: Bearer ${{ secrets.REGISTRY_TOKEN }}" \
            -F "package=@package.xi" \
            -F "archive=@package.tar.gz"
```

### 3.3 Ecosystem Package Template

```yaml
# .github/workflows/ci.yml (for each ecosystem repo)
name: Package CI
on: [push, pull_request]

jobs:
  check:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install xiom
        run: |
          curl -L https://registry.xiom-lang.org/xiom-windows-latest.zip -o xiom.zip
          Expand-Archive xiom.zip -DestinationPath C:\xiom
          echo "C:\xiom\bin" >> $env:GITHUB_PATH
      - run: xiom --check-only src/*.xi

  publish:
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: |
          tar -czf package.tar.gz src/ *.xiom-bind package.xi
          curl -X POST https://registry.xiom-lang.org/api/v1/publish \
            -H "Authorization: Bearer ${{ secrets.REGISTRY_TOKEN }}" \
            -F "package=@package.xi" \
            -F "archive=@package.tar.gz"
```

---

## 4. CROSS-PLATFORM PACKAGING

### 4.1 Windows (PowerShell -- existing)

```powershell
# package.ps1 (already working)
.\package.ps1 -Version "0.49.8" -Sign -CertificateThumbprint "A1B2C3..."
# -> release/xiom-v0.49.8-windows-x64.zip
```

### 4.2 macOS / Linux (Bash)

```bash
#!/bin/bash
# package.sh
VERSION="${1:-0.49.8}"
RELEASE_DIR="release/xiom-v${VERSION}"

mkdir -p "${RELEASE_DIR}/bin" "${RELEASE_DIR}/lib" "${RELEASE_DIR}/runtime"

# Build all tools
for tool in xiom xiom-fmt xiom-doc xiom-ffigen xiom-pkg xiom-lsp xiom-mcp xiom-dbg xiom-verify; do
    cargo build -p "$tool" --release && cp "target/release/$tool" "${RELEASE_DIR}/bin/"
done

# Package stdlib
cp -r stdlib/* "${RELEASE_DIR}/lib/"
cp stdlib/runtime/xiom_runtime.c "${RELEASE_DIR}/runtime/"

# Sign (macOS)
if [[ "$OSTYPE" == "darwin"* ]]; then
    codesign --force --sign "Developer ID Application" "${RELEASE_DIR}/bin/"*
fi

# Archive
tar -czf "release/xiom-v${VERSION}-$(uname -s)-$(uname -m).tar.gz" -C release "xiom-v${VERSION}"
```

### 4.3 Homebrew Formula (macOS)

```ruby
# xiom-lang/homebrew-xiom/Formula/xiom.rb
class Xiom < Formula
  desc "XIOM Compiler -- systems programming language"
  homepage "https://xiom-lang.org"
  url "https://registry.xiom-lang.org/xiom-v#{version}-macOS-arm64.tar.gz"
  sha256 "..."

  def install
    bin.install Dir["bin/*"]
    lib.install Dir["lib/*"]
  end

  test do
    system "#{bin}/xiom", "--version"
  end
end
```

---

## 5. SECRETS & TOKENS

### 5.1 GitHub Organization Secrets

| Secret | Purpose | Scope |
|--------|---------|-------|
| `REGISTRY_TOKEN` | Authenticate publish to registry.xiom-lang.org | All repos |
| `CODESIGN_CERT_BASE64` | Windows code signing certificate (base64 .pfx) | xiom repo |
| `CODESIGN_PASSWORD` | Certificate password | xiom repo |
| `APPLE_DEVELOPER_ID` | macOS code signing identity | xiom repo |

### 5.2 Registry Token Generation (on VPS)

```bash
# Generate a random token for CI/CD publishing
openssl rand -hex 32 > /home/lefteris/web/registry.xiom-lang.org/.publish_token

# In api/publish.php:
# $valid_token = trim(file_get_contents(__DIR__ . '/../.publish_token'));
# if ($_SERVER['HTTP_AUTHORIZATION'] !== "Bearer $valid_token") { http_response_code(403); exit; }
```

Add this token to GitHub Organization Secrets as `REGISTRY_TOKEN`.

---

## 6. MIGRATION CHECKLIST

### Phase 1: Foundation (Day 1)
- [ ] Create `xiom-language` GitHub organization
- [ ] Create repo: `xiom-lang/xiom` -- push compiler + crates
- [ ] Create repo: `xiom-lang/stdlib` -- push stdlib
- [ ] Create repo: `xiom-lang/docs` -- push docs + website
- [ ] Set up `xiom-lang/xiom` CI/CD (build + test)
- [ ] Configure registry.xiom-lang.org on VPS
- [ ] Test registry index.json serving

### Phase 2: Ecosystem (Day 2-3)
- [ ] Create repos for top 10 ecosystem packages (vulkan, http, crypto, db, vector, json, net, imgui, sqlite, redis)
- [ ] Set up CI/CD template for ecosystem repos
- [ ] Test `xiom pkg install xiom.vulkan` from registry
- [ ] Register ecosystem packages in registry index

### Phase 3: Polish (Day 4-5)
- [ ] Create remaining 46 ecosystem repos
- [ ] Editor extension repos
- [ ] Homebrew formula
- [ ] Cross-platform packaging scripts (macOS, Linux)
- [ ] GitHub Pages for docs.xiom-lang.org
- [ ] Update all READMEs with new repo URLs

---

## 7. VERIFICATION COMMANDS

```bash
# Test registry
curl https://registry.xiom-lang.org/index.json

# Install a package from registry
xiom pkg install xiom.stdlib --registry https://registry.xiom-lang.org

# Search packages
xiom pkg search vulkan --registry https://registry.xiom-lang.org

# Publish a package
xiom pkg publish --registry https://registry.xiom-lang.org --token $REGISTRY_TOKEN

# Full CI run locally
.\test_summary.ps1
cargo test --all
cargo clippy --all-targets
cargo fmt --all -- --check
```

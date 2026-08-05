# Monorepo Split — Folder → Repository Mapping

## Naming Convention

| Org | Naming | Rationale |
|-----|--------|-----------|
| `xiom-lang` | Drop `xiom-` prefix | It's redundant — everything is xiom |
| `xiom-foundation` | Keep `xiom-` prefix | Different org needs clarity in go modules/imports |

## Repository Map

### xiom-lang (6 repos)

| Repo | Src Folder(s) | Description |
|------|---------------|-------------|
| **xiom** | `crates/xiom/`, `crates/xiom-lexer/`, `crates/xiom-parser/`, `crates/xiom-ast/`, `crates/xiom-check/`, `crates/xiom-codegen/`, `crates/xiom-ctfe/`, `crates/xiom-jit/`, `crates/xiom-graph/`, `crates/xiom-verify/` | Full compiler pipeline (10 crates) |
| **stdlib** | `stdlib/`, `stdlib/runtime/` | 87 stdlib .xi files + C runtime |
| **packages** | `packages/` | 70 official packages |
| **website** | `docs/` → convert to HTML | Static site for xiom-lang.org |
| **playground** | `crates/xiom-wasm/`, `release/.../playground.html` | WASM compiler + browser IDE |
| **registry** | (new — nothing to migrate) | Docker registry config |

### xiom-foundation (5 repos)

| Repo | Src Folder(s) | Description |
|------|---------------|-------------|
| **xiom-pulse** | `xiom-pulse/` (if exists) | Web/API server framework |
| **xiom-benchmark-chaos** | `xiom-benchmark-chaos/` | Benchmark suite |
| **xiom-db** | `xiom-db/` | SQL database project |
| **xiom-vector** | `xiom-vector/` | Vector database |
| **xiom-debugger** | `xiom-debugger-pro/` | Debugger (rename to xiom-debugger) |

### xiom-enterprise (future — 0 repos now)

| Repo | Description |
|------|-------------|
| `xiom-db-enterprise` | Paid DB license |
| `xiom-vector-enterprise` | Paid vector DB |
| `xiom-debugger-enterprise` | Paid debugger |

## What Stays in Monorepo (NOT migrated)

| Folder | Reason |
|--------|--------|
| `target/`, `.test_build/`, `.testlogs/` | Build artifacts |
| `release/` | Local release packages |
| `tests/` | Already inside `crates/xiom-codegen/tests/` |
| `examples/` | Consumed by integration tests |
| `.github/` | Replaced by per-repo workflows |

## Action Plan

```
Phase 1: Create repos + push (1 day)
  □ xiom-lang/xiom          ← crates/xiom*/ + tests/ + Cargo.toml
  □ xiom-lang/stdlib        ← stdlib/
  □ xiom-lang/packages      ← packages/
  □ xiom-lang/website       ← docs/ + md→html build script
  □ xiom-lang/playground    ← crates/xiom-wasm/
  □ xiom-lang/registry      ← Dockerfile + registry config
  □ xiom-foundation/xiom-benchmark-chaos ← xiom-benchmark-chaos/

Phase 2: CI/CD (2 days)
  □ xiom-lang/xiom: build on tag, release binaries, push to ghcr.io
  □ xiom-lang/stdlib: validate on push, package on xiom release
  □ xiom-lang/website: build + deploy to xiom-lang.org on push
  □ xiom-lang/playground: build WASM, deploy to playground.xiom-lang.org

Phase 3: Connect (1 day)
  □ xiom-lang.com → redirect to xiom-lang.org
  □ registry.xiom-lang.org → proxy to ghcr.io/xiom-lang
  □ xiom-foundation.org → static site

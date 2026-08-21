# Phase 8 Preflight Audit -- Production Readiness Assessment

**Date:** 2026-07-21  
**Audit Scope:** 16 compiler crates + 40 stdlib modules + 10 tooling binaries + 5 ecosystem samples  
**Current State:** v0.49.7, 881/881 tests, zero warnings  
**Goal:** Rate every component 1-10 before starting Debugger Pro and Self-Hosting

---

## 1. COMPILER CRATES -- 16 Crates

| Crate | Rating | LOC | Tests | Unsafe | Unwrap | Exit | Risk |
|-------|--------|-----|-------|--------|--------|------|------|
| xiom-display | **8** | 104 | 5 | 0 | 0 | 0 | Low |
| xiom-ast | **7** | 538 | 0 | 1 | 0 | 0 | Med |
| xiom-lexer | **7** | 554 | 15 | 1 | 5 | 0 | Low |
| xiom-graph | **7** | 1,574 | 23 | 0 | 35 | 0 | Med |
| xiom-check | **6** | 5,062 | 78 | 10 | 20 | 0 | Med |
| xiom-parser | **6** | 1,577 | 50 | 0 | **53** | 0 | High |
| xiom-dbg | **6** | 928 | 8 | 1 | 3 | 0 | Low |
| xiom-fmt | **6** | 1,064 | 18 | 1 | 1 | 5 | Med |
| xiom-ffigen | **6** | 482 | 18 | 0 | 5 | 2 | Low |
| xiom-verify | **5** | 1,180 | 15 | 0 | 0 | 5 | Med |
| xiom-doc | **5** | 299 | 4 | 0 | 0 | 3 | Low |
| xiom-mcp | **5** | 1,526 | 17 | **20** | 9 | 0 | High |
| xiom-codegen | **4** | 20,113 | 752 | 22 | 24 | 0 | High |
| xiom-lsp | **4** | 2,628 | 11 | 0 | 38 | 0 | High |
| xiom-pkg | **4** | 901 | 15 | 0 | 0 | 11 | Med |
| xiom | **4** | 3,700 | 3 | 2 | 4 | **28** | High |

### Top 5 Systemic Issues

| # | Issue | Impact | Fix Effort |
|---|-------|--------|------------|
| 1 | **God objects**: IrEmitter (61 fields), CompileConfig (25+ fields) | Testing, refactoring, parallelism | 2-3 weeks |
| 2 | **282 unwraps** across project -> user-facing crashes | Reliability | 1-2 weeks |
| 3 | **57 process::exit** in library code -> unembeddable | Reusability | 3-5 days |
| 4 | **Missing tests**: xiom (3), xiom-lsp (11), xiom-ast (0) | Regression risk | 1 week |
| 5 | **Monolithic files**: expr.rs (5,323), LSP main.rs (2,628) | Maintainability | 1-2 weeks |

---

## 2. STANDARD LIBRARY -- 40 Modules

| Tier | Count | Modules |
|------|-------|---------|
| Good (>60% contracts) | 12 | ffi, io, os, alloc, rc, string, crypto, ptr, collections, math, core, encoding |
| Partial (1-59%) | 8 | cell, contracts, env, mem, rand, serialize, simd, sync |
| **Zero (0%)** | **20** | array, async, bench, char, cmp, compress, convert, error, fmt, hash, iter, log, net, num, path, regex, reflect, test, thread, time |

### Top 5 Missing Types/Traits (vs Rust stdlib)

| # | Missing | Priority |
|---|---------|----------|
| 1 | `From`/`Into`/`TryFrom`/`TryInto` conversion traits | High |
| 2 | `Deref`/`DerefMut` + `AsRef`/`AsMut` | High |
| 3 | `Duration`, `Instant`, `SystemTime` in time.xi | High |
| 4 | `Cow<T>`, `PhantomData<T>`, `MaybeUninit<T>` | Med |
| 5 | `Path`/`PathBuf` (skeleton, needs implementation) | Med |

**Overall: B+ (85%)** -- Strong Phase 1 stdlib. 50% of modules need contracts.

---

## 3. TOOLING BINARIES -- 10 Tools

| # | Tool | Rating | Critical Gap |
|---|------|--------|--------------|
| A1 | **xiom** | **91/100 (A)** | Main.rs too large (1300 lines) |
| A2 | xiom-fmt | **65/100 (C+)** | Missing --version, no round-trip tests |
| A3 | xiom-doc | **66/100 (C+)** | Missing --version, duplicates xiom-display |
| A4 | xiom-ffigen | **88/100 (A-)** | Missing --version |
| A5 | xiom-pkg | **87/100 (A-)** | Missing --version |
| A6 | xiom-lsp | **78/100 (B)** | **Zero tests**, duplicates xiom-display |
| A7 | **xiom-mcp** | **93/100 (A)** | Best tooling binary |
| A8 | xiom-dbg | **85/100 (A-)** | Needs integration tests |
| A9 | xiom-verify | **83/100 (B+)** | Missing --help, --version |
| A10 | xiom-display | **45/100 (D)** | **Imported by ZERO crates** -- DRY violation |

### Top 3 Tooling Gaps

| # | Issue | Impact |
|---|-------|--------|
| 1 | 5 tools missing --version flag | CLI standards compliance |
| 2 | xiom-display unused -> duplicated code in LSP + doc | DRY violation |
| 3 | xiom-lsp has zero tests (2628 lines) | Highest-risk gap |

---

## 4. ECOSYSTEM PACKAGES -- 75 Total (5 Sampled)

| Package | Rating | Key Finding |
|---------|--------|-------------|
| xiom-json | **92/100 (A)** | Production-quality JSON library |
| xiom-vulkan | **88/100 (A-)** | 755 extern fns, excellent contracts |
| xiom-imgui | **85/100 (A-)** | State-tracking contracts, missing package.xi |
| xiom-glfw | **82/100 (B+)** | Clean bridge pattern |
| xiom-redis | **40/100 (D)** | **STUB** -- all 21 fns return errors |

### Top 3 Ecosystem Gaps

| # | Issue | Impact |
|---|-------|--------|
| 1 | **Zero tests** in any ecosystem package | FFI correctness unverified |
| 2 | xiom-redis is a stub (no FFI) | Cannot be used |
| 3 | xiom-imgui missing package.xi manifest | Package manager can't discover |

---

## 5. PHASE 8B -- Preflight Hardening (BEFORE Debugger Pro)

Based on audit findings, these fixes must ship before 8C:

### Sprint M1 -- Quick Wins (1-2 days)

| # | Fix | Effort | Crate |
|---|-----|--------|-------|
| M1.1 | Add `--version` to xiom-fmt, xiom-doc, xiom-ffigen, xiom-pkg, xiom-verify | 0.5d | 5 tools |
| M1.2 | Add `--help` to xiom-verify | 0.5d | xiom-verify |
| M1.3 | Integrate xiom-display into xiom-lsp (remove duplicates) | 0.5d | lsp |
| M1.4 | Integrate xiom-display into xiom-doc (remove duplicates) | 0.5d | doc |

### Sprint M2 -- Stdlib Contracts (2-3 days)

| # | Fix | Effort | Module |
|---|-----|--------|--------|
| M2.1 | Add contracts to high-priority zero-contract modules | 1d | array, convert, error, num, char, cmp |
| M2.2 | Add contracts to medium-priority modules | 1d | iter, hash, fmt, time, net, regex |
| M2.3 | Add `From`/`Into` traits to core.xi | 0.5d | core |
| M2.4 | Add `Duration` type to time.xi | 0.5d | time |

### Sprint M3 -- Test Coverage (2-3 days)

| # | Fix | Effort | Crate |
|---|-----|--------|-------|
| M3.1 | Add integration test for xiom::compile_with_diagnostics | 1d | xiom |
| M3.2 | Add LSP protocol tests (initialize, completion, hover) | 1d | lsp |
| M3.3 | Add AST serialization round-trip tests | 0.5d | ast |
| M3.4 | Verify 881 baseline + add regression tests | 0.5d | codegen |

### Sprint M4 -- Code Health (P2, after 8C)

| # | Fix | Effort | Crate |
|---|-----|--------|-------|
| M4.1 | Split IrEmitter god object into sub-contexts | 1-2w | codegen |
| M4.2 | Replace 282 unwraps with Result propagation | 1-2w | all |
| M4.3 | Remove 57 process::exit from library code | 3-5d | xiom, verify, pkg |

---

## 6. OVERALL RATINGS

| Layer | Components | Average | Status |
|-------|-----------|---------|--------|
| Compiler Crates | 16 | **5.4/10** | Needs hardening (M4) |
| Standard Library | 40 modules | **B+ (85%)** | Needs contracts (M2) |
| Tooling Binaries | 10 tools | **B (78%)** | Needs --version + DRY fix (M1) |
| Ecosystem Packages | 75 total | **B (77%)** | Needs tests + xiom-redis fix |

**Foundation Readiness: B (82%)** -- Ready for Debugger Pro after M1+M2 sprints.

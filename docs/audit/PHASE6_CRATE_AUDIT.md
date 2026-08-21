# XIOM Phase 6 -- Crate Audit Report

**Date:** 2026-07-20 | **Version:** v0.49.0 | **Baseline:** 788/788 tests
**Scope:** All 14 compiler + tooling crates | **Total LOC:** 32,875 Rust

---

## Executive Summary

| Crate | Rating | Category | Priority |
|-------|--------|----------|----------|
| xiom-ast | **7.7** | GOOD | Low |
| xiom-lexer | **7.0** | GOOD | Low |
| xiom-parser | **7.1** | GOOD | Medium |
| xiom-check | **6.3** | NEEDS WORK | **Critical** |
| xiom-codegen | **4.9** | NEEDS WORK | **Critical** |
| xiom | **6.2** | NEEDS WORK | Medium |
| xiom-fmt | **6.7** | GOOD | Low |
| xiom-mcp | **6.7** | GOOD | Medium |
| xiom-doc | **6.1** | GOOD | Low |
| xiom-verify | **5.4** | NEEDS WORK | Medium |
| xiom-ffigen | **5.0** | NEEDS WORK | Low |
| xiom-pkg | **3.4** | AT RISK | **Critical** |
| xiom-dbg | **3.4** | AT RISK | Medium |
| xiom-lsp | **3.1** | AT RISK | **Critical** |

**Weighted average across all crates: 5.6/10**

---

## TOP 10 CRITICAL ISSUES

### 1. CRITICAL -- xiom-check: Type safety violation (`types_compatible`)
**File:** `crates/xiom-check/src/lib.rs:2885`
**Issue:** `(CheckedType::Named(_), CheckedType::Named(_)) => true` -- any two user-defined types are compatible.
**Impact:** `Int = Str` passes type checking. The compiler accepts clearly wrong programs.
**Fix:** Track generic type arguments in CheckedType. Remove the catch-all. Add proper structural comparison.

### 2. CRITICAL -- xiom-codegen: God object (55+ fields)
**File:** `crates/xiom-codegen/src/lib.rs:40-203`
**Issue:** `IrEmitter` has 55+ mutable fields. Every method depends on all of them.
**Impact:** Impossible to reason about state. Every new feature adds more fields.
**Fix:** Split into `CodegenContext` (immutable tables), `FunctionFrame` (per-function state), `IrEmitter` (output + counters).

### 3. CRITICAL -- xiom-codegen: Raw string IR emission
**Issue:** LLVM IR constructed as raw strings. No type-safety, no structural validation.
**Impact:** Invalid IR only caught by clang. Cryptic errors with no source location.
**Fix:** Adopt `inkwell` or similar IR builder library.

### 4. CRITICAL -- xiom-pkg: Unsafe Rust (`static mut REGISTRY_CACHE`)
**File:** `crates/xiom-pkg/src/main.rs:99`
**Issue:** `static mut` is unsound. Undefined behavior under concurrent access.
**Fix:** Use `std::sync::OnceLock` or `Mutex<Option<...>>`.

### 5. CRITICAL -- xiom-lsp: 2488-line monolith, zero tests
**File:** `crates/xiom-lsp/src/main.rs` (entire file)
**Issue:** Single file with no module separation. Zero tests. Will crash on mutex poison.
**Fix:** Split into `handlers/` modules. Add tests. Use `PoisonError` handling for mutex.

### 6. CRITICAL -- xiom-check: Monolithic 4228-line file
**Issue:** Checker, borrow checker, module resolver, expression checker all in one file.
**Fix:** Split into `expr.rs`, `stmt.rs`, `module.rs`, `borrow.rs`, `compat.rs`.

### 7. HIGH -- xiom-dbg: Compile error (undeclared variable `child`)
**File:** `crates/xiom-dbg/src/main.rs:286`
**Issue:** `child.id()` references undeclared variable; pattern is `_child`.
**Fix:** Rename `_child` to `child`.

### 8. HIGH -- xiom-codegen: Unknown types silently default to `"i64"`
**File:** `crates/xiom-codegen/src/lib.rs:971`
**Issue:** `xiom_to_llvm_type` default `_ => "i64"` -- typos in type names produce wrong IR silently.
**Fix:** Return `Err(...)` instead of defaulting.

### 9. HIGH -- xiom-pkg: Broken binary download (base64 of text)
**File:** `crates/xiom-pkg/src/main.rs:209`
**Issue:** `FromBase64String(Invoke-WebRequest.Content)` -- content is text, not base64. Produces garbage.
**Fix:** Use `curl.exe -o` or proper binary download.

### 10. HIGH -- Encoding corruption in multiple source files
**Issue:** `emitter.rs:1`, `decl.rs:59` and others showed UTF-8 mojibake (em dashes / curly quotes double-encoded through Windows-1252).
**Fix:** RESOLVED -- `tools/ascii_guard.py` reversed the mojibake and transliterated all tracked text files to pure ASCII (enforced by CI + pre-commit hook).

---

## COMPILER CRATES

### xiom-ast -- 7.7/10 [GOOD]
- **Strengths:** Clean AST, ErrorGuaranteed pattern, well-typed
- **Gaps:** No `Span` byte offset, no visitor trait, mega-enum `Expr` (40 variants)
- **Refactor:** Group Expr variants; extract shared IfExpr from Stmt::If and Expr::If

### xiom-lexer -- 7.0/10 [GOOD]
- **Strengths:** Clean, well-tested, keyword table
- **Bugs:** Hex/float overflow silent (`unwrap_or(0)` on overflow)
- **Gaps:** No raw string literals, no Unicode surrogate validation, `source: Vec<char>` (4x memory)
- **Fix:** Return `TokenKind::Error("integer literal overflow")`, use `&str` + byte-based indexing

### xiom-parser -- 7.1/10 [GOOD]
- **Strengths:** LL(1) recursive descent, `expected` bitset diagnostics, panic-mode recovery
- **Gaps:** No incremental re-parsing, no expression error recovery
- **Refactor:** 1662 LOC -> split into `expr.rs`, `decl.rs`, `stmt.rs`, `pattern.rs`
- **Bugs:** O(n2) clone in postfix chain (line 1337), stale comment (MAX_EXPR_DEPTH=32 not 200)

### xiom-check -- 6.3/10 [NEEDS WORK]
- **Strengths:** Two-phase checking, place-based borrow checker, TypeArena interning
- **Bugs:** `types_compatible` catch-all (line 2885), double `flatten_submodules` call (line 156)
- **Refactor:** 4228 LOC -> split into 5+ files
- **Gaps:** No generic type tracking, no interface impl verification

### xiom-codegen -- 4.9/10 [NEEDS WORK]
- **Strengths:** Monomorphisation, const_eval, hot reload thunks, sandbox auditor
- **Bugs:** Unknown type -> "i64" (line 971), coerce fallthrough (coerce.rs:198), encode corruption
- **Refactor:** Split IrEmitter, use inkwell IR builder, deduplicate `expr_uses_this` with checker
- **Gaps:** No debug info, no optimization pipeline, no LTO, no GC

### xiom -- 6.2/10 [NEEDS WORK]
- **Strengths:** Safe compile_with_diagnostics API, Diagnostic struct, error codes
- **Refactor:** 1080 LOC main.rs -> extract package.rs, bench.rs, scaffold.rs
- **Gaps:** No proper argument parser (should use clap), no LSP binary, no cross-compilation sysroot

---

## TOOLING CRATES

### xiom-fmt -- 6.7/10 [GOOD]
- **Strengths:** Clean architecture, idempotency tested, good CLI
- **Bugs:** `format_float(-1.0)` panics (line 862), Destructure missing indent (line 264)
- **Gaps:** No directory recursion, no config file, no --diff mode

### xiom-mcp -- 6.7/10 [GOOD]
- **Strengths:** Well-modularized, 14 tools, live stdlib parsing, good test coverage
- **Bugs:** Fixed temp filename (line 580), notification detection wrong (line 787), JSON-RPC newline break (line 769)
- **Fix:** Use unique temp files, check `id` field absence for notifications, read Content-Length framing

### xiom-doc -- 6.1/10 [GOOD]
- **Strengths:** Clean, well-sized, single-pass
- **Gaps:** No HTML output, no doc comment extraction, no --output flag

### xiom-verify -- 5.4/10 [NEEDS WORK]
- **Strengths:** SSA lowering, Z3 integration, contract axiom generation
- **Bugs:** Contract axiom `forall` uses `"x"` for all params (line 151), u16 cast for f64 exponent (line 455), unsupported expr -> `true` (line 557)
- **Gaps:** No loop invariants, no struct field access in contracts, no JSON output

### xiom-ffigen -- 5.0/10 [NEEDS WORK]
- **Bugs:** `u64` -> `"UInt"` not `"UInt64"` (line 188), nullable substring match (line 135)
- **Gaps:** No struct/typedef support, no `#include` processing

### xiom-pkg -- 3.4/10 [AT RISK]
- **Bugs:** `static mut` UB (line 99), broken binary download (line 209), JSON string-match (line 561), `deps:` never parsed (line 393)
- **Refactor:** Use OnceLock, use proper HTTP client, use proper manifest parser

### xiom-dbg -- 3.4/10 [AT RISK]
- **Bugs:** Compile error (line 286), BufReader per call desyncs GDB (line 127), no `stopped` events
- **Refactor:** Fix compile error, add background reader thread, add `evaluate` handler

### xiom-lsp -- 3.1/10 [AT RISK]
- **Bugs:** Out-of-range text edit (line 1536), false completions (line 1630), `T001` code action corrupts file (line 2241), mutex panic on poison
- **Refactor:** Split 2488 LOC into handlers/ modules. Add tests.
- **Gaps:** No formatting, no folding, no inlay hints, no incremental sync

---

## CROSS-CUTTING ISSUES

### Code Duplication
- `type_to_string` / `expr_to_string` duplicated in xiom-doc, xiom-lsp, xiom-fmt, xiom-mcp, xiom-ffigen
- `expr_uses_this` duplicated in xiom-check and xiom-codegen
- **Fix:** Create `xiom_display` shared utility crate

### Inconsistent Error Handling
- String errors (xiom-dbg, xiom-pkg) vs typed errors (xiom-verify) vs silent fallback (xiom-lsp)
- **Fix:** Standardize on `thiserror`-derived error types

### Version Number Chaos
- Ranges from v0.1.0 (xiom-mcp) to v0.48.9 (xiom)
- **Fix:** Use workspace `version` from root `Cargo.toml`

### Missing Integration Tests
- xiom-dbg: 0 integration tests (only trivial struct tests)
- xiom-lsp: 0 tests
- xiom-pkg: 0 integration tests (only manifest parsing)
- **Fix:** Add DAP/LSP/MCP integration tests with real servers

---

## PHASE 6 ACTION PLAN (Priority Order)

### Sprint 1: Fix Critical Bugs (2-3 days)
1. [OK] xiom-dbg: Fix compile error (`_child` -> `child`)
2. [OK] xiom-pkg: Fix `static mut` -> `OnceLock`
3. [OK] xiom-check: Remove `types_compatible` catch-all
4. [OK] xiom-codegen: Default `"i64"` -> error
5. [OK] Fix encoding corruption in source files

### Sprint 2: Refactor Crates (5-7 days)
6. Split xiom-check lib.rs (4228 -> 5 files)
7. Split xiom-lsp main.rs (2488 -> handlers/)
8. Split IrEmitter (55 fields -> CodegenContext + FunctionFrame)
9. Split xiom-parser (1662 -> expr/decl/stmt/pattern)
10. Create `xiom_display` shared crate

### Sprint 3: Tooling Hardening (5-7 days)
11. xiom-pkg: Fix all HTTP bugs, proper binary download, JSON parsing
12. xiom-lsp: Add formatting, tests, mutex error handling
13. xiom-dbg: Add GDB async reader, `stopped` events, `evaluate` handler
14. xiom-verify: Fix SMT generation bugs, add `--json` output

### Sprint 4: Polish (3-5 days)
15. Unified version numbers
16. Standardized error handling (thiserror)
17. Integration test infrastructure
18. Remove code duplication (xiom_display)

---

## Rating Distribution

```
10 .
 9 .
 8 .
 7 ...  (xiom-ast, xiom-lexer, xiom-parser)
 6 .... (xiom-check, xiom, xiom-fmt, xiom-mcp, xiom-doc)
 5 ..   (xiom-codegen, xiom-verify, xiom-ffigen)
 4 .
 3 ...  (xiom-pkg, xiom-dbg, xiom-lsp)
 2 .
 1 .
```

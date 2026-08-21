# XIOM -- Playground & Module Resolution Gap Audit

**Date:** 2026-07-01  
**Version:** v0.19.0  
**Status:** Module resolution is entirely source-file-local -- stdlib files are never read by the compiler

---

## 1. Problem

When a user writes `use xiom.io;` in the playground or any `.xi` file, the compiler reports:

```
error[T001]: undefined variable 'io'
error[T001]: cannot call 'println' on this expression
```

This happens because the XIOM compiler has **no filesystem-based module resolution**. It only resolves `use` declarations against inline `module Name { ... }` blocks within the same source file.

---

## 2. Root Cause -- Module Resolution Architecture

### Current Flow

```
Source.xi --> Lexer --> Parser --> Checker --> Codegen --> LLVM IR
                                    |
                              process_use("xiom.io")
                                    |
                              lookup "xiom" in self.modules
                                    |
                              self.modules is EMPTY
                              (only populated by inline module blocks)
                                    |
                              silently returns -- no error
                                    |
                              later: "io.println" -> "undefined variable 'io'"
```

### The Missing Piece

The compiler has no mechanism to:
1. Read `stdlib/package.xi` to discover available stdlib modules  
2. Map `use xiom.io` -> `stdlib/xiom/io.xi`
3. Parse `stdlib/xiom/io.xi` and merge its declarations into the program

The `xiom-pkg` crate DOES have package resolution logic, but it's a standalone CLI tool -- it's never called by `xiom` (the compiler).

### The Failure Point in Code

`crates/xiom-check/src/lib.rs` line 418:

```rust
fn process_use(&mut self, ud: &UseDecl) {
    let module_name = &ud.path[0].name;  // "xiom"
    let exports = match self.modules.get(module_name) {
        Some(e) => e,
        None => return,  // <- SILENTLY RETURNS. No error. No resolution.
    };
    // ...
}
```

---

## 3. Playground Impact

### WASM Path
- The WASM binary (33KB) has WASI stubs that return `8` (EBADF) for all filesystem operations
- Even if the compiler had filesystem resolution, the WASM binary can't read `stdlib/xiom/io.xi` from disk
- The WASM path falls through to the server path

### Server Path (Python backend)
- `website/playground/server.py` writes user code to a temp file, runs `xiom --emit-ir <tempfile>`
- The compiler still has no filesystem resolution, so `use xiom.io` fails
- **Fix applied (v0.13.0):** The server now detects `use xiom.X` patterns, reads `stdlib/xiom/X.xi`, and injects the content as inline `module xiom { module X { ... } }` blocks before the user code

### Production (Ubuntu/Apache/Hestia)
- Same server.py deployment behind Apache proxy
- The inline injection fix works identically in production
- Apache serves the `website/` static files and proxies `/compile` to the Python server

---

## 4. The Proper Fix -- Compiler-Level Module Resolution

### Required Architecture Change

```
Source.xi --> Lexer --> Parser --> ModuleResolver --> Checker --> Codegen
                                       |
                                  For each "use xiom.X":
                                  1. Read stdlib/package.xi
                                  2. Find "xiom.X" in modules list
                                  3. Read stdlib/xiom/X.xi
                                  4. Lex + Parse it
                                  5. Merge into program AST
                                  6. Register in modules HashMap
```

### Implementation Plan

1. **Add `ModuleResolver` pass** to `xiom-check` or a new crate `xiom-resolve`
2. **Add `--stdlib-path` CLI flag** to `xiom` (default: `stdlib/` relative to binary)
3. **Parse `stdlib/package.xi`** on startup to build module index
4. **In `process_use()`:** if module not found in inline blocks, attempt filesystem resolution
5. **For WASM:** bundle stdlib `.xi` files into the WASM binary as embedded strings, or serve them via the WASI filesystem interface

### Short-term Workaround (Already Applied)

The `website/playground/server.py` now contains `resolve_stdlib_imports()` which:
- Scans user code for `use xiom.X` patterns using regex
- Reads the corresponding `stdlib/xiom/X.xi` file
- Strips the file-level `module xiom.X` declaration
- Injects it as `module xiom { module X { ... } }` nested inline blocks
- Prepends all resolved modules before user code

This allows the playground to compile programs using stdlib modules without any compiler changes.

---

## 5. WASM Compilation Roadmap

For the playground to work entirely in-browser (no server required):

### Current Status
- WASM module: 33KB, loads in browser, WASI stubs exist
- WASI `fd_write` (stdout capture): WORKS
- WASI `fd_read` (stdin): STUB (returns 0 bytes)
- WASI `path_open` (file access): STUB (returns EBADF)
- Full browser compilation: NOT WORKING -- falls through to server

### What's Needed
1. **Bundle stdlib sources** -- embed all `stdlib/xiom/*.xi` file contents in the WASM binary
2. **Implement WASI filesystem in JS** -- use an in-memory filesystem (Emscripten-style or custom) so `path_open`/`fd_read` can serve stdlib files
3. **Or:** Add an explicit API to the WASM module that accepts source code + stdlib contents directly, bypassing WASI filesystem entirely

### Phase 3 Target
- Full WASI filesystem emulation with pre-loaded stdlib files
- Compile -> run -> capture output entirely in browser
- No server dependency

---

## 6. Files Referenced

| File | Role |
|------|------|
| `website/playground/server.py` | Python backend -- now injects stdlib inline |
| `website/playground/index.html` | Frontend -- WASM loader + fallback to server |
| `playground/server.py` | Deprecated older version |
| `crates/xiom-check/src/lib.rs` | Type checker -- `process_use()` at line 418 |
| `crates/xiom-ast/src/lib.rs` | `UseDecl`, `ModuleDecl` AST nodes |
| `crates/xiom-pkg/src/` | Standalone package manager (not wired to compiler) |
| `stdlib/package.xi` | Stdlib manifest -- 50 modules listed |
| `stdlib/xiom/*.xi` | Stdlib source files (file-level `module xiom.X`) |
| `dist/xiom/` | Pre-built binaries and runtime |

---

## 7. Test Commands

```powershell
# Start playground server locally
python website/playground/server.py
# Open http://localhost:3000/playground/

# Test that stdlib resolution works
echo "use xiom.io; fn main() { io.println(\"hello\"); }" | python -c "
import sys, urllib.request
data = sys.stdin.read().encode()
r = urllib.request.urlopen('http://localhost:3000/compile', data)
print(r.read().decode())
"
```

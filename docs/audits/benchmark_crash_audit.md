# XIOM -- Benchmark Crash Audit

**Date:** 2026-07-01  
**Version:** v0.19.0 (Rust bootstrap + selfhost in development)  
**Status:** Critical vulnerabilities identified -- compiler unable to safely compile large programs

---

## Summary

The `examples/benchmark_stress.xi` (8,577 lines, 28 inline modules) causes the XIOM compiler to crash due to multiple memory safety vulnerabilities in both the Rust codegen and C runtime codegen paths. This audit catalogs the root causes and provides a prioritized fix roadmap.

---

## 1. Root Cause Analysis -- Why the Benchmark Crashed

### 1.1 Vec.push Heap Buffer Overflow * CRITICAL

**Location:** `crates/xiom-codegen/src/lib.rs` lines 2155-2198 (Rust codegen), `stdlib/runtime/xiom_runtime.c` (C codegen)

**What happens:** `Vec.new()` allocates a fixed 128-byte buffer (16 x i64). `Vec.push` writes to `data[len]` and increments `len` with NO capacity check and NO reallocation. Any Vec exceeding 16 elements writes past the buffer into arbitrary heap memory.

```
call i8* @malloc(i64 128)           // allocate 128 bytes = 16 i64 slots
...
{offset} = mul i64 {len_val}, 8     // no cap comparison!
{dest} = getelementptr i8, i8* {data_ptr}, i64 {offset}
store i64 {val}, i64* {dest}        // WRITE PAST 128-BYTE BUFFER when len >= 16
```

**Impact on benchmark:**  
- `data_primes.primes_table()` -- 500 pushes -> corrupts heap at push #17
- `lcg(seed, 500)` -- 500 pushes -> corrupts heap at push #17  
- Multiple Vec allocations across 28 modules -> cascading heap corruption
- Results in undefined behavior: crashes, wrong results, or memory exhaustion

### 1.2 No Recursion Depth Limit * CRITICAL

**Location:** All codegen paths -- Rust `compile_expr`, C `emit_body_ir`

**What happens:** The compiler emits standard LLVM `call` instructions for recursive functions with no depth counter or `musttail` optimization. Each call allocates a new stack frame. The benchmark contains 15+ recursive functions.

**Impact on benchmark:**
| Function | Recursion Pattern | Stack Risk |
|----------|------------------|------------|
| `fibonacci_rec(50)` | Binary tree, 2^n calls | **Instant overflow** (~10^15 frames) |
| `ackermann(2,2)` | Non-primitive recursive | ~27 calls (safe) but A(3,2) = overflow |
| `binomial(30,15)` | Tree recursion | ~155M calls, guaranteed overflow |
| `tribonacci(30)` | Triple tree | 3^n calls |
| `collatz(27)` | Linear but unbounded | 111 calls (safe at this input) |
| `catalan(20)` | Nested recursion + while | O(4^n) |

The benchmark's test functions only use safe small inputs, but **all recursive functions are public** and accessible from `main()`. The compiler generates code that WILL overflow the stack at moderate recursion depths.

### 1.3 Weak Local Variable Hashing * HIGH

**Location:** `stdlib/runtime/xiom_runtime.c` lines 447-449

```c
int idx = (name[0] - 'a') % pc;  // hash by first letter
reg[nest_level] = local_regs[idx]; // NO name confirmation
```

**What happens:** Variables starting with the same letter map to the same register. `find_local_reg(name)` correctly searches the table, but `get_local_reg(name)` uses the flawed hash-based shortcut. If two variables start with the same letter (e.g., `x` and `x_squared`), they silently alias.

**Impact on benchmark:** Functions with many locals starting with same letter (common in math code: `result`, `remainder`, `root`, `ratio`) produce wrong IR where one variable's value is read when another was intended.

### 1.4 Division by Zero in Generated IR * HIGH

**Location:** Rust codegen lines 1883-1884, C codegen

```rust
// Emits raw LLVM sdiv/srem with no zero guard
format!("sdiv i64 {a}, {b}")
```

**What happens:** LLVM `sdiv` with zero divisor is undefined behavior. On x86-64, this typically generates a SIGFPE signal, killing the process. The benchmark exposes public functions like `div_int(a, b)` that perform raw division with no check.

**Impact on benchmark:** If any benchmark path calls `div_int(x, 0)`, `mod_int(x, 0)`, or `map_range` with equal min/max, the program crashes.

### 1.5 Fixed-Size Arrays in C Runtime * HIGH

**Location:** `stdlib/runtime/xiom_runtime.c`

| Buffer | Size | Impact |
|--------|------|--------|
| `field_names[16][64]` | 16 fields | Benchmark has 50-field struct `BigStruct` -- silently truncated |
| `field_types[16][16]` | 16 fields | Types truncated, wrong memory layout |
| `arm_lits[16]` / `arm_results[16]` | 16 arms | 50-arm `match_50` truncated |
| `local_names[64][64]` | 64 locals | Large functions drop variables |
| `st_buf[4][128]` | 4 type slots | Round-robin buffer corruption |

### 1.6 Generic Monomorphisation Infinite Loop * MEDIUM

**Location:** Rust codegen lines 1100-1217

```rust
loop {
    let item = worklist.pop();
    if worklist.is_empty() { break }
    // monomorphise and push new items to worklist
}
```

**What happens:** If monomorphising one generic function generates new generic instantiations (e.g., generic chains), the worklist never empties. No iteration limit.

### 1.7 Default Type Silently Falls Back to i64 * MEDIUM

**Location:** Rust codegen line 182

```rust
_ => "i64"  // unknown types silently become i64
```

Any type error produces wrong LLVM IR instead of a compile error, masking bugs.

---

## 2. Vulnerability Matrix

| # | Severity | Component | Bug | Fix Effort |
|---|----------|-----------|-----|------------|
| V1 | **CRITICAL** | Codegen (Rust) | Vec.push no realloc -- heap overflow past 16 elements | ~50 lines |
| V2 | **CRITICAL** | Codegen (both) | No recursion depth limit -- stack overflow | ~100 lines |
| V3 | **HIGH** | C Runtime | Weak local var hash -- name collisions | ~30 lines |
| V4 | **HIGH** | Codegen (both) | No div-zero guard -- SIGFPE | ~20 lines |
| V5 | **HIGH** | C Runtime | Fixed-size arrays (16 fields, 64 locals, 16 arms) | ~200 lines |
| V6 | **MEDIUM** | Codegen (Rust) | Generic mono infinite loop -- no iteration limit | ~10 lines |
| V7 | **MEDIUM** | Codegen (Rust) | Unknown types -> i64 silently | ~5 lines |
| V8 | **MEDIUM** | C Runtime | Round-robin type buffer overwrite | ~50 lines |
| V9 | **LOW** | C Runtime | Float buffer overflow (64-byte stack) | ~10 lines |
| V10 | **LOW** | C Runtime | String interning OOB read | ~20 lines |

---

## 3. Fix Priority Roadmap

### Phase 1: Critical Stability (Must fix to safely compile any program)

1. **Fix Vec.push realloation** -- Add capacity check and `realloc` doubling strategy in both Rust and C codegen
2. **Add recursion depth limit** -- Emit a depth counter in recursive functions, trap at configurable limit (default 500)
3. **Fix local variable hashing** -- Replace `get_local_reg` with linear search through `find_local_reg`
4. **Add div-zero runtime guards** -- Emit `icmp` + conditional branch to trap before `sdiv`/`srem`

### Phase 2: Correctness Hardening

5. **Make fixed-size arrays dynamic** -- Replace `field_names[16]` with malloc'd arrays or significantly increase limits
6. **Add generic monomorphisation limit** -- Count iterations, error after 1000
7. **Remove i64 default** -- Return error on unknown types

### Phase 3: Benchmark Safety

8. **Add Vec bounds checking** -- Check `i < len` before `GEP` on `vec[i]`
9. **Audit benchmark for memory-safe inputs** -- Ensure test inputs don't trigger stack overflow
10. **Add `--max-recursion-depth` CLI flag** -- Let users control recursion limits

---

## 4. Recommended Compilation Commands (Post-Fix)

After Phase 1 fixes are applied:

```powershell
# Compile with safety limits
cargo run -p xiom -- --run --max-recursion-depth 500 examples/benchmark_stress.xi

# Or via dist binary
.\dist\xiom\bin\xiom.exe --run examples/benchmark_stress.xi
```

---

## 5. Notes

- The benchmark suite itself is structurally correct XIOM code -- the crashes are compiler bugs, not benchmark bugs
- The multi-file `examples/benchmark/` version cannot be compiled yet because the compiler lacks filesystem-based module resolution (see `docs/audits/playground_module_gap.md`)
- The C runtime (`xiom_runtime.c`) is a second compiler implementation -- bugs here diverge from the Rust codegen and create double-maintenance burden
- Recommendation: Deprecate the C runtime codegen path once the Rust codegen is stable enough for self-hosting

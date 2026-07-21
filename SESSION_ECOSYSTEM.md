# XIOM Ecosystem — AI Construction Session

**Date:** 2026-07-04
**Branch:** `feat/ecosystem-build` (or independent machine)
**Status:** Starting from v0.22.1 compiler. Building XIOM ecosystem using AI-generated code.
**Companion session:** `SESSION.md` — compiler hardening on `feat/guardian` branch

> This session builds the XIOM ecosystem (stdlib implementations, packages, tools, showcase projects) using AI agents with `docs/AI_CONTEXT.md` as context. The compiler runs on another machine or branch. Each item that compiles is a hardening test for the compiler.

---

## How This Works

1. **Feed `docs/AI_CONTEXT.md` as system context** to an LLM
2. **Copy a prompt from this document** into the LLM
3. **Save the generated `.xi` files** in the appropriate directory
4. **Compile with `xiom`** — if it fails, the error is a hardening signal for the compiler track
5. **Fix any XIOM syntax errors** (the AI may use wrong syntax — AI_CONTEXT.md has the rules)
6. **Move to the next item** when the current one compiles

---

## Two-Track Strategy

| Track | Where | What |
|-------|-------|------|
| **A — Compiler** | `feat/guardian` branch | Fix P0 bugs (Vec realloc, C runtime limits). Each fix unlocks more ecosystem items. |
| **B — Ecosystem** | This session / other machine | Build XIOM packages using AI. Items that fail reveal compiler bugs → feed to Track A. |

**They validate each other.** Track B provides real test cases. Track A makes more items compile.

---

## Layer 1 — Pure XIOM (Can Build NOW)

These items need NO stdlib implementations. Only compiler built-ins (Option, Result, Vec, structs, enums, generics, contracts). They compile today.

### 1.1 xiom-json — JSON Parser/Serializer

**Output:** `xiom-json/src/json.xi`
**Dependencies:** None (pure XIOM)
**Prompt:**
```
Write a pure XIOM JSON parser and serializer. Every function must have contracts.

Use `docs/AI_CONTEXT.md` for correct XIOM syntax. Key rules:
- Use `module xiom.json` as the first line
- Fields in structs separated by `;` not `,`
- `elif` not `else if`
- `self` is implicit in methods
- Every statement ends with `;`
- `requires:` and `ensures:` between signature and body

1. pub type JsonValue = enum {
     Null,
     Bool(value: Bool),
     Number(value: Float64),
     String(value: Str),
     Array(items: Vec[JsonValue]),
     Object(entries: Vec[JsonEntry])
   } derive[Clone]

2. pub type JsonEntry = { key: Str; value: JsonValue; } derive[Clone]

3. pub fn json_parse(input: Str) -> Result[JsonValue, ParseError]
     requires: input.len() > 0

4. pub fn json_stringify(value: &JsonValue) -> Str

5. pub fn json_get(obj: &JsonValue, key: Str) -> Option[JsonValue]

6. pub type ParseError = { message: Str; position: Int; } derive[Clone]
```

### 1.2 xiom-test — Test Framework

**Output:** `xiom-test/src/test.xi`
**Dependencies:** None (pure XIOM)
**Prompt:**
```
Write a XIOM test framework using `docs/AI_CONTEXT.md` for syntax.

Module: xiom.test

1. pub type TestResults = { passed: Int; failed: Int; total: Int; failures: Vec[TestFailure]; }
2. pub type TestFailure = { name: Str; message: Str; }
3. pub fn assert_eq[T](actual: T, expected: T, msg: Str)
4. pub fn assert_true(condition: Bool, msg: Str)
5. pub fn run_tests(tests: Vec[(Str, fn() -> Bool)]) -> TestResults
```

### 1.3 XiomDB Types — Database Type Definitions

**Output:** `xiom-db/src/types.xi`
**Dependencies:** None (pure XIOM)
**Prompt:**
```
Define XiomDB's core types with invariants using `docs/AI_CONTEXT.md` for syntax.
Type definitions only — no I/O, no FFI, no file operations.

Module: xiom.db.types

1. pub type PageId = UInt64
2. pub type Key = Vec[Byte]
3. pub type Page = { id: PageId; data: Vec[Byte]; checksum: UInt32; invariant: data.len() <= 4096; } derive[Clone]
4. pub type BTreeNode = { keys: Vec[Key]; children: Vec[PageId]; is_leaf: Bool; invariant: keys.is_sorted(); invariant: keys.len() <= 256; }
5. pub enum WALOp { Insert, Update, Delete }
6. pub type Transaction = { id: UInt64; state: TxState; operations: Vec[WALOp]; }
7. pub enum TxState { Active, Committed, Aborted }
```

### 1.4 XiomVector Types — Vector DB Type Definitions

**Output:** `xiom-vector/src/types.xi`
**Dependencies:** None (pure XIOM)
**Prompt:**
```
Define XiomVector's core types with invariants using `docs/AI_CONTEXT.md` for syntax.

Module: xiom.vector.types

1. pub type Vector = { data: Vec[Float32]; dimension: UInt; invariant: data.len() == dimension; }
2. pub type HNSWNode = { id: VectorId; neighbors: Vec[Neighbor]; invariant: neighbors.len() <= 16; }
3. pub type Neighbor = { id: VectorId; distance: Float32; }
4. pub type VectorId = UInt64
5. pub enum DistanceMetric { Cosine, DotProduct, Euclidean }
```

### 1.5 Algorithm Library

**Output:** `xiom-algo/src/algo.xi`
**Dependencies:** None (pure XIOM)
**Prompt:**
```
Write a pure XIOM algorithm library using `docs/AI_CONTEXT.md` for syntax.
Every function must have contracts.

Module: xiom.algo

1. pub fn binary_search[T: Ord](arr: &Vec[T], target: &T) -> Option[Int]
     ensures: result is Some => arr[result.unwrap()] == target
2. pub fn quicksort[T: Ord](arr: &mut Vec[T])
     ensures: arr.is_sorted()
3. pub fn gcd(a: Int, b: Int) -> Int
4. pub fn fibonacci(n: Int) -> Int requires: n >= 0
5. pub fn sieve_of_eratosthenes(n: Int) -> Vec[Int] requires: n >= 2
6. pub fn is_prime(n: Int) -> Bool requires: n >= 0
7. pub fn factorial(n: Int) -> Int requires: n >= 0, n <= 20
```

---

## Layer 2 — Stdlib Implementations (After Track A P0 Fixes)

These fill in the 530+ stub function bodies. The type signatures already exist in `stdlib/xiom/`. The compiler needs Vec realloc (V1) fixed first.

### 2.1 xiom.core — Core Function Bodies

**Output:** Update `stdlib/xiom/core.xi` (types exist, add bodies)
**Prompt:**
```
The file stdlib/xiom/core.xi has complete type definitions but all function bodies
are stubs (ending with `;` instead of `{...}`). Using `docs/AI_CONTEXT.md` for syntax,
implement the bodies for these functions:

1. pub fn Option.unwrap[T]() -> T requires: self is Some { ... }
2. pub fn Option.unwrap_or[T](default: T) -> T { ... }
3. pub fn Option.is_some[T]() -> Bool { ... }
4. pub fn Result.unwrap[T, E]() -> T requires: self is Ok { ... }
5. pub fn Result.is_ok[T, E]() -> Bool { ... }
6. pub fn Result.is_err[T, E]() -> Bool { ... }
7. pub fn panic(msg: Str) -> ! { ... }  // call @llvm.trap via extern
8. pub fn assert(condition: Bool, msg: Str) { ... }
```

### 2.2 xiom.collections — Collection Bodies

**Output:** Update `stdlib/xiom/collections.xi`
**Prompt:**
```
Using `docs/AI_CONTEXT.md` for syntax, implement the function bodies for xiom.collections.
The type definitions (Vec[T], Map[K,V], Set[T]) already exist.

Key implementations:
1. pub fn Vec.push[T](value: T)
     requires: self is valid
     ensures:  len == len@pre + 1
2. pub fn Vec.pop[T]() -> Option[T]
3. pub fn Vec.get[T](index: Int) -> Option[T] requires: index >= 0
4. pub fn Vec.len[T]() -> Int
5. pub fn Vec.is_empty[T]() -> Bool

Vec operations use compiler built-ins — push/pop/get/len are codegen primitives.
Map and Set use Vec internally with hashing.
```

### 2.3 xiom.string — String Bodies

**Output:** Update `stdlib/xiom/string.xi`
**Prompt:**
```
Using `docs/AI_CONTEXT.md` for syntax, implement the function bodies for xiom.string.

IMPORTANT: Use `xiom::string::str_len(s)` NOT `s.len()`. Str has no methods.
Use `extern "C"` for C runtime functions like strlen.

Implement:
1. pub fn str_len(s: Str) -> Int
2. pub fn str_concat(a: Str, b: Str) -> Str
3. pub fn str_split(s: Str, delimiter: Char) -> Vec[Str]
4. pub fn str_contains(s: Str, substr: Str) -> Bool
5. pub fn str_trim(s: Str) -> Str
6. pub fn parse_int(s: Str) -> Result[Int, ParseError]
7. pub type ParseError = { message: Str; position: Int; }
```

### 2.4 xiom.io — I/O Bodies

**Output:** Update `stdlib/xiom/io.xi`
**Prompt:**
```
Using `docs/AI_CONTEXT.md` for syntax, implement the function bodies for xiom.io.

Use `extern "C"` for file operations. The C runtime provides:
  - xiom_read_file(path: *UInt8) -> *UInt8
  - xiom_file_size(path: *UInt8) -> Int
  - xiom_free(ptr: *UInt8)

Implement:
1. pub fn print(s: Str) — write to stdout via printf
2. pub fn println(s: Str) — print + newline
3. pub fn read_file(path: Str) -> Result[Vec[UInt8], IOError]
     requires: path.len() > 0
4. pub fn write_file(path: Str, data: &Vec[UInt8]) -> Result[Unit, IOError]
5. pub fn file_exists(path: Str) -> Bool
6. pub type IOError = { message: Str; kind: IOErrorKind; }
7. pub enum IOErrorKind { NotFound, PermissionDenied, Unknown }
```

---

## Layer 3 — Ecosystem Packages (After Stdlib Works)

| # | Item | Depends On |
|---|------|-----------|
| 3.1 | xiom-net — TCP networking | Layer 2.4 |
| 3.2 | xiom-http — HTTP client/server | Layer 3.1 |
| 3.3 | xiom-sqlite — Database bindings | Layer 2.4 |
| 3.4 | xiom-crypto — Cryptography | Layer 2.4 |

## Layer 4 — Showcase Projects (After Layer 3)

| # | Item | Depends On |
|---|------|-----------|
| 4.1 | XiomDB full — Embedded database | Layer 3.3, 1.2 |
| 4.2 | XiomVector full — Vector database | Layer 4.1 |
| 4.3 | XiomGame — Game engine demo | Layer 2.4 |

## Layer 5 — Full Applications

| # | Item |
|---|------|
| 5.1 | HTTP Server (production example) |
| 5.2 | CLI Tool (production example) |
| 5.3 | Chat Server (concurrency demo) |

---

## Key Documents for AI Context

| Document | When to Feed to AI |
|----------|-------------------|
| `docs/AI_CONTEXT.md` | **Always** — system context for every prompt |
| `docs/XIOM_BUILD_ORDER.md` | Reference for build order and dependency chains |
| `specs/XIOM_Language_Spec.md` | For detailed grammar/syntax questions |

---

## Verification After Each Item

```powershell
# Compile the generated file
xiom --emit-ir <output>.xi

# If IR is valid, compile to binary
xiom -o test.exe <output>.xi

# If binary links, run it
.\test.exe
echo "EXIT: $LASTEXITCODE"
```

If any item FAILS to compile, note the error and feed it back to the compiler hardening session (SESSION.md). The error IS the hardening signal.

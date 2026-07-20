# XIOM Phase 6 — Standard Library Audit Report

**Date:** 2026-07-20 | **Version:** 0.1.0 | **Modules:** 41 | **Total LOC:** 11,974 XIOM + 4,085 C/ASM

---

## Executive Summary

**OVERALL STDLIB RATING: 5.2/10**

| Dimension | Score | Notes |
|-----------|-------|-------|
| Completeness | 5.5 | Core data structures present; many gaps |
| Correctness | 6.0 | Most core logic is correct; edge cases vary |
| API Design | 5.0 | Inconsistent naming, method styles, trait usage |
| Documentation | 3.0 | Headers exist; inline docs and examples sparse |
| Performance | 5.0 | Quadratic sorts, Vec-backed structures, O(n) bit ops |
| Contract Coverage | 4.5 | ~18% of pub fns have requires/ensures |

---

## CATEGORIZATION

### PRODUCTION-READY (6/41 = 15%) — Rating 7.0+
| Module | Rating | Notes |
|--------|--------|-------|
| **time.xi** | 7.8 | Duration/Instant/DateTime, complete calendar math |
| **encoding.xi** | 7.8 | Base64/Hex/URL/UTF-8, comprehensive + contracted |
| **char.xi** | 7.5 | Complete ASCII classification, UTF-8 encoding |
| **string.xi** | 7.5 | Core string ops, contracts on char_at/slice/concat |
| **sync.xi** | 7.5 | Mutex/RwLock/Condvar/Arc/Barrier/Atomic, robust |
| **io.xi** | 7.0 | File/console I/O, Read/Write/Seek traits, contracted |

### PARTIAL (25/41 = 61%) — Rating 4.5-7.0
| Module | Rating | Top Gaps |
|--------|--------|----------|
| **core.xi** | 7.2 | Missing Rc/Arc in core, no Result.unwrap |
| **math.xi** | 7.2 | O(n) bitwise ops, missing sinh/cosh/tanh |
| **num.xi** | 7.2 | O(n) bit counting, recursive gcd |
| **mem.xi** | 7.0 | drop is no-op |
| **env.xi** | 7.0 | FAMILY hardcoded |
| **cmp.xi** | 7.0 | No is_lt/is_eq/is_gt on Ordering |
| **net.xi** | 6.0 | No TLS, TCP read limited to 4KB |
| **iter.xi** | 6.0 | Self-mutation by value, Iterator.sum assumes numeric |
| **rand.xi** | 6.5 | LCG not crypto-secure, UUID weak entropy |
| **serialize.xi** | 6.5 | No binary format, no \uXXXX in JSON |
| **ptr.xi** | 7.0 | Volatile/copy_nonoverlapping are identical |
| **cell.xi** | 6.5 | Ref/RefMut don't decrement borrows |
| **async.xi** | 6.0 | No select!, Channel.recv blocks forever |
| **collections.xi** | 5.8 | Map is O(n) linear, LinkedList Vec-backed |
| **log.xi** | 5.5 | No caller location, no JSON escaping |
| **regex.xi** | 5.5 | No alternation, groups, backreferences |
| **compress.xi** | 4.5 | ALL compressors use RLE internally |
| **crypto.xi** | 5.5 | RSA 16-32 bit only, AES-NI not called |
| **os.xi** | 6.2 | spawn uses system(), no piped I/O |
| **alloc.xi** | 6.8 | Layout.size is Int not UInt |
| **convert.xi** | 5.2 | float_to_string no precision control |
| **array.xi** | 5.5 | O(n^2) sort, missing iter/windows/chunks |
| **fmt.xi** | 4.5 | width/precision/align fields unused |
| **simd.xi** | 4.0 | Memory leak on every op, only Vec4f wired |
| **bench.xi** | 4.8 | No auto-calibration, no regression compare |

### STUB (10/41 = 24%) — Rating <4.5
| Module | Rating | What's Missing |
|--------|--------|---------------|
| **reflect.xi** | 3.8 | Generic queries return "unknown", no Any impl |
| **contracts.xi** | 3.5 | All checks stubbed, "bootstrap" everywhere |
| **rc.xi** | 4.0 | No Drop trait, no auto-decrement |
| **error.xi** | 3.0 | No Display, backtrace empty, no error kind |
| **path.xi** | 5.0 | PathBuf mutation by value lost, `\` always `/` |
| **ffi.xi** | 2.5 | size_of/align_of return 0 |
| **hash.xi** | 5.0 | sip_hash is DJB2, no crypto hash |
| **thread.xi** | 5.2 | Thread.name None, scoped threads incomplete |

---

## CRITICAL BUGS

### 1. compress.xi — ALL compressors use RLE
**Impact:** gzip/brotli/zlib output is RLE-only wrapped in correct format headers. Decompresses correctly but no actual compression.
**Fix:** Implement real DEFLATE (LZ77 + Huffman) or use FFI bindings to zlib.

### 2. simd.xi — Memory leak on every vector operation
**Impact:** `Vec4f.add` etc. allocate 16 bytes every call via `ffi.alloc(16)` and NEVER free.
**Fix:** Track allocation lifetime or use stack-allocated vectors.

### 3. path.xi — PathBuf mutation lost
**Impact:** `PathBuf.push/pop` mutate `self` by value — changes invisible to caller.
**Fix:** Use `&mut self` receiver.

### 4. iter.xi — Iterator state mutation lost
**Impact:** `Range.next` and adapter `next` methods mutate `self` by value. Iteration may not progress.
**Fix:** Use `&mut self` receiver.

### 5. cell.xi — Ref/RefMut don't restore borrow counts
**Impact:** After a `Ref` goes out of scope, the RefCell remains permanently borrowed.
**Fix:** Implement proper Drop or use manual decrement on scope exit.

### 6. crypto.xi — AES-NI never called
**Impact:** `aes_encrypt` AES-NI branch and software branch are IDENTICAL. Hardware acceleration never engaged.
**Fix:** Call `xiom_aesni_encrypt_block` FFI in AES-NI branch.

---

## TOP PRIORITY IMPROVEMENTS

### Sprint 1: Contract Coverage (3-5 days)
- **Target:** 18% → 50%+ pub functions with contracts
- **Modules:** collections.xi (all Vec/Map methods), core.xi (Option/Result), string.xi, io.xi
- **Contracts are XIOM's killer feature** — every public function should have requires/ensures

### Sprint 2: Fix Critical Bugs (3-5 days)
1. Fix `cell.xi` — Ref/RefMut borrow restoration
2. Fix `path.xi` — PathBuf mutation receivers
3. Fix `iter.xi` — Iterator mutation receivers
4. Fix `simd.xi` — Memory leak
5. Fix `crypto.xi` — AES-NI engagement

### Sprint 3: Complete Collections (5-7 days)
- HashMap (hash-based, not linear-probe Vec)
- Fix LinkedList (node-based, not Vec-backed)
- Fix Map (hash-based, currently O(n) linear search)
- Add Vec.reserve/shrink_to_fit/truncate/extend/drain
- Add BTreeMap.range, Set.iter

### Sprint 4: Real Compression (3-5 days)
- Replace RLE with real DEFLATE implementation
- OR: Add FFI bindings to zlib/miniz
- Fix gzip/brotli/lz4/snappy compressors

### Sprint 5: Stdlib Polish (5-7 days)
- Unified API style (free functions vs methods)
- Module documentation for ALL 41 modules
- Performance optimizations (quicksort, hash Map, memcpy)
- Complete error.xi (Display impl, backtrace, error kind)
- Complete contracts.xi (wire to compiler metadata)
- Complete reflect.xi (Any trait impls, field access)
- Complete regex.xi (alternation, groups, {n,m})

### Sprint 6: Missing Stdlib Modules
- **json.xi** — Dedicated JSON module (currently in serialize.xi)
- **http.xi** — HTTP server + client (currently minimal in net.xi)
- **fs.xi** — Filesystem operations (currently in io.xi + os.xi)
- **process.xi** — Process spawning (currently in os.xi)
- **tls.xi** — TLS/SSL support
- **task.xi** — Task/future combinators (currently in async.xi)

---

## CONTRACT COVERAGE BY MODULE

| Contract Level | Modules |
|----------------|---------|
| **Good (>50%)** | alloc, io, cell, rc, encoding, ptr |
| **Partial (20-50%)** | core, collections, math, sync, mem, compress, crypto, env |
| **Minimal (<20%)** | string, net, os, iter, fmt, array, bench, num, path, thread, async, rand, serialize |
| **None** | cmp, char, convert, error, ffi, hash, log, regex, reflect, contracts, simd, time |

**Target for Phase 6 completion:** Good (>50%) on ALL modules.

---

## RATING DISTRIBUTION

```
10 ▓
 9 ▓
 8 ▓▓  (time, encoding)
 7 ▓▓▓▓▓▓▓▓ (char, string, sync, io, core, math, num, mem, env, cmp, ptr)
 6 ▓▓▓▓▓▓ (cell, async, net, iter, rand, serialize, os, alloc)
 5 ▓▓▓▓▓▓▓▓▓ (collections, log, regex, crypto, path, thread, hash, convert, array)
 4 ▓▓▓ (compress, reflect, contracts, rc, fmt, simd, bench)
 3 ▓▓ (error, ffi)
 2 ▓
 1 ▓
```

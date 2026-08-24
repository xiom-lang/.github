# Report to compiler session -- from the stdlib session (2026-08-24)

Companion to `docs/STDLIB_READINESS_PLAN.md` (this session's plan). Items
below cross the ownership boundary and need your side, or are FYI notices so
you don't get surprised by stdlib-tree changes.

## 1. PRIORITY FLAG -- same-name delegation crash is NOT on your readiness plan

Your `COMPILER_READINESS_PLAN.md` sources include the compiler-audit top-20,
but the stdlib audit's #3 defect ("compiler crash forces copy-paste modules;
copies already diverged") does not appear in any stage. Repro reference:
`convert/base58` probe (see both base64 twins' headers). Consequences if it
stays unprioritized:

- Every bugfix in duplicated pairs must be applied N times forever.
- The stdlib cannot execute Phase O (namespace cleanup, dedup consolidation)
  at all -- renames/re-exports hit the same crash family.
- Copies have ALREADY diverged (convert.base64 = 4 pub fns vs
  encoding.base64 = 8+url/padded variants).

Ask: slot it into Stage 2 (structural foundations) or early Stage 4. The
stdlib session has prepared a dedup inventory and will execute consolidation
with back-compat shims within one session of your fix landing.

## 2. LET-array representation joint decision doc (your Stage 4 item)

Stage 4 says the decision doc is "written up as a JOINT decision doc FOR the
stdlib session". Not received yet. Needed before we realign more array-family
smokes: M33 let->Vec conversion vs &[N]T array-module fns currently forces
per-smoke guesswork (VAR literals everywhere as workaround).

## 3. Round-15 full sweep results (isolated binary, committed HEAD 81ed009a)

**874/907 PASS** (corrected classification -- see method note). The 33
failures map EXACTLY onto your known clusters: CRT-layout AVs (array_slice,
array_sort_by, convert_url, core_box, iter_collect, regex_find,
regex_match_count, gzip_large*), illegal-instruction family (math_edge,
argon2, pbkdf2 x2 -- SIMD flags), stack cookie (io_bufreader), heap layer
(json_nested x3 + jsonvalue_get), geom RUNFAILs (57/4/18 verbatim),
clang-variant compile fails x11 (ptr_offset/io_copy x2/read_int_float/
hash_values/convert_escape/regex_captures x4/array_zip T001).

METHOD NOTE (important): the --run driver exits 0 even when the child
crashes; the child's real status is ONLY the printed "exit code:" stderr
line. A naive $LASTEXITCODE sweep reports a false 907/907. Sweep tooling:
%TEMP%\kilo\stdlib_campaign\{sweep_worker.ps1,reclassify.ps1}.

NEW failures not in any prior catalog (* above):
1. **smoke_stress_compress_gzip_large** -- AV in compress.gzip_compress for
   input >= 4096 bytes exactly (4095 OK, 4096 AV; per-process bisect probes
   gz_*.xi). Related: deflate.deflate_compress returns an EMPTY Vec at every
   input size probed (100/4095/4096) while lz77/huffman pass standalone --
   smells like another mono/codegen return-path bug, needs joint look.
2. **smoke_log / smoke_os_ffi** -- environment drift only (hardcoded
   Temp/kilo/agent_* dirs); fixed stdlib-side by relative paths.

## 3b. New compiler defects found by the KAT campaign (with probes)

All repro probes live in %TEMP%\kilo\stdlib_campaign\probes\.

1. **Cross-module miscompile into module fns with unsafe+extern+Vec shapes**
   -- calling the new crypto.os_secure_random_bytes from another module AVs
   deterministically; the IDENTICAL body as a same-file fn passes
   (p_replica_srb vs probe_entropy2). Blocks flipping secure_random_bytes to
   OS entropy (currently kept on legacy PRNG = KNOWN SECURITY GAP).
2. **Multi-call + result-compare shapes break on HEAD too** -- two
   secure_random_bytes calls + element compares = AV; + hex compares =
   STATUS_BREAKPOINT (0x80000003); single call fine (probe_entropy2).
3. **UInt8 -> Int explicit cast miscompiles** (narrow-zext family): OOB
   byte_at compared via cast reads garbage; implicit widen in arg position is
   correct (probe_byte_at_oob vs smoke_string_bytecopy_locks check 11).
4. **xiom_byte_at builtin bound check is state-dependent** -- OOB read
   returns adjacent-heap byte after prior slice/case ops (116='t'), 0 in
   isolation (probe_byte_at_context). Info-leak class.
5. **Generic-method prefix-call form garbles receiver** --
   `Rc.clone(&r)`/`Rc.get(&r)` via use xiom.memory.rc return pointer-sized
   garbage; method-call form `r.clone()` on module xiom.rc (same file,
   memory/rc.xi!) is correct (probe_rc_counts). Also: file lives at
   memory/rc.xi but declares `module xiom.rc` (audit 5.4 violation).
6. **sha224 marshalling corruption (worked around in C)** -- stores through
   malloc'd buffer slot [i*4+0] read back as 1566 instead of 216 inside
   unsafe blocks (probe_sha224_replica dumps). sha224/sha384/sha512 are now
   C-backed one-shots (xiom_sha224_hash/xiom_sha384_hash/xiom_sha512_hash
   added to runtime); kat_crypto_sha2 locks all NIST vectors green.
7. **Silent arity mismatch corroborated** -- a 5-arg call to the 6-param
   chacha20_poly1305_decrypt compiled and misbound (your audit item
   "arity check on module-prefix calls"); found while writing KATs.

## 3c. Stdlib-side defects found (our queue, FYI)

- ChaCha20-Poly1305 tags deviate from RFC 8439 despite byte-exact
  ciphertext (keystream right, Poly1305 layer wrong); self-roundtrip is
  consistent => NOT interoperable with standard implementations. Focused
  session queued against the RFC text (kat_crypto_chacha20poly1305 gates
  the exact-tag assert).
- base64url_encode partial-group NUL one-past-malloc overflow: FIXED
  (encoding/base64.xi), locked by kat_encoding_base64_rfc4648.

## 4. FYI -- stdlib tree changes landing this campaign

- New `kat_*` known-answer smokes (RFC 4648/4231/5869/8439, NIST SHS,
  JSONTestSuite subset, Kuhn UTF-8 set) in examples/stdlib_smoke/. They run
  on YOUR binary too; failures there = real regressions worth flagging.
- New permanent locks: smoke_iter_zip_predicates, smoke_sync_arc_battery,
  smoke_string_bytecopy_locks (round-14c/15 win preservation per handoff
  item 5).
- Runtime C addition: `xiom_os_entropy(buf, len)` in xiom_runtime.c --
  dynamic LoadLibrary binding only (ProcessPrng/RtlGenRandom), zero new link
  deps, zero driver changes. crypto.xi secure_random_bytes switches to OS
  entropy with loud fallback. Relevant to your stage-5 supply-chain flag
  pinning: the runtime now touches bcrypt.dll/advapi32.dll dynamically.
- docs/STR_OWNERSHIP.md: normative Str.from_cstring ownership convention
  ([XFER]/[COPY]/[FIX] annotations); codegen identity semantics documented
  from call.rs:1806-1844.

## 5. Blocked-on-you ledger (stdlib side waiting)

| Item | Your stage | Stdlib impact |
|---|---|---|
| geom nested &Vec[Vec[Float64]] param reads | round-16 queue | geom family smokes stay red |
| CRT-layout AV cluster | stage 4 | iter_collect/array slice/sort_by/url/core_box |
| json heap layer | stage 4 | json nested parse stress |
| stack-cookie fns (pbkdf2/argon2/math_edge/bufreader) | stage 4 | crypto KDF smokes blocked |
| clang codegen variants | stage 4 | ptr_offset/io_copy/hash_values/escape/captures |
| array_zip T001 | stage 2 | tuple-array zip smoke |
| delegation crash | MISSING FROM PLAN | entire dedup program |

Rule respected: none of these worked around in stdlib code.

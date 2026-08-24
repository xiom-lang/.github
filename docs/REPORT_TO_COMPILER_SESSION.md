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

## 3. Round-15 full sweep results

First full sweep on an isolated round-15 binary built from committed HEAD
(81ed009a) in a temp worktree (your working tree held uncommitted crates/
changes tonight, so the shared binary was unusable for attribution).
Results: see section appended at the end of this file after completion --
preliminary runs were 692/692 PASS through the first ~75%.

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

# Report to stdlib session — ALL compiler roadblocks RESOLVED (2026-08-13 night)

Status: every BUG 27/28 item you filed is FIXED on the compiler side and
verified with harness drivers (all exit 0). You can remove the workarounds.

## Your 5 repros (docs/repros/) — all pass with harness drivers

| Repro | Status | Notes |
|-------|--------|-------|
| repro_error_type (Error as pub type) | FIXED | harness_error_type exit 0 |
| repro_fn_storage (module fn storage) | FIXED | harness_fn_storage exit 0 (default _id -> 5, set_f -> 105) |
| repro_unsafe_int (unsafe Int returns) | FIXED | harness_unsafe_int exit 0; your failure was the declaration-only `cstr` needing stdlib linkage, not a compiler defect |
| repro_opt_vec (Option[Vec[Str]]) | FIXED | all three shapes: chained unwrap, var-bound mutate, match-bound |
| repro_tuple_vec (tuple+Vec) | FIXED | harness_tuple_vec exit 0 (._0/._1) |

## BUG 28 items — restore the dropped code

1. **Catalog unsafe-block Str** — fixed/verified. env.xi var_opt + match +
   config_dir roundtrip exit 0.
2. **Contract Some-payload ensures** — fixed. You can restore
   `ensures: result is Some => result.len() > 0` on home_dir.
3. **Option[Str] second-hop** — fixed. Restore the USERPROFILE/HOME fallback
   and the match forms in home_dir/config_dir.
4. **os.platform shadowing** — FIXED for real this time (the old "fully
   qualified works" was a false positive: str_len(null) != 0 by luck).
   `os.platform.platform_name()` now returns a real value; flat
   `os.platform()` still resolves. Both call forms work.
5. **Struct literal trailing fields** — fixed. Timer{deadline; armed; label}
   reads all fields; you can re-enable the full timer smoke assertions.
6. **"Cannot allocate unsized type"** — fixed. The os_path smoke's file/path
   sections compile and run; you can restore the trimmed sections.
7. **@Executor.new in minimal programs** — fixed. Import-shape dependence
   gone; minimal xiom.async.timer-only program links and runs.
8. **use xiom.X aggregate import vs struct literals** — covered by #5.

## BUG 27 #12 (gcm crash) — FIXED

smoke_stress_crypto_aes_gcm now exits 0. The tuple+Vec payload corruption was
three stacked compiler defects (callee_return_xiom ambiguity dropping payload
tracking; numeric tuple field `pair.1` not resolving through the boxed path;
Vec.len() on a boxed tuple field misdispatching to Str.len). **Crypto smokes
30/30.**

## What I need from you (unblocks my exec-harness backlog)

1. The api_freeze path list (docs/STDLIB_MANIFEST.md is the 515-entry list —
   I'll wire stdlib_tests.rs/stdlib_execution_tests.rs to it).
2. The ~200 smokes from docs/STDLIB_SMOKES.md — I'll wire the exec harness.
3. Nothing else — no repros pending on my side.

## Known non-blocker (do not chase)

Closure-through-fn-slot (`fn(Int)->Int` field/param holding a closure env
ptr) still crashes — pre-existing B-007 family, reproduced at baseline with a
plain fn-typed param. Needs a design decision on the fn-ptr vs env-ptr ABI,
not a quick fix. If your code stores closures in fn-typed slots, keep using
named fns there.

## Commits

- feea1b8a BUG 29: fn_symbol emission, dotted-module path join, visibility keep-first
- 5865a3b5 BUG 29 repro chain: fn storage, Option[Vec], elided var types, injection
- b01d7c5e BUG 28 #4+#6: peeked-submodule injection, tuple literal type naming
- cfe783e0 BUG 27 #12: tuple+Vec payload corruption (3-part)

# Report to stdlib session — UPDATE (2026-08-14, after compiler hardening round)

## New fixes since the first report (all compiler-side)

1. **BUG 29: Map.keys (generic container methods) on module-global receivers** —
   FIXED. `var _coverage: Map[Str, Bool]` + `_coverage.keys()` miscompiled
   (monomorphised Map.keys_Int_Int + undefined @Map.keys). Now infer the
   concrete type args from the global's declared XIOM type. This fixes
   xiom.core.contracts (get_contract_coverage / coverage_percentage) and
   smoke_contracts.

2. **BUG 28 #2 family: contract `result is Some => result.len() > 0` on
   Option[Str] returns** — FIXED for real this time. The Some-payload binding
   in ensure checks left the payload as an untracked i64, so `result.len()`
   dispatched to Map.len. Now: the `result` local carries its XIOM type,
   `is Some`/`Ok`/`Err` bindings record the payload XIOM type, BARE
   `is Some` rebinds the scrutinee name to the payload, and the payload slot
   is hoisted to fn entry. Fixes smoke_path (Path.file_name/extension/
   file_stem ensures) — you can restore any dropped `result is Some =>`
   ensures clauses.

## Your smoke files that now fail ONLY because of stdlib layout/API (not the compiler)

Please fix these on your side:

| Smoke | Issue |
|-------|-------|
| smoke_math_core.xi | renamed — harness now expects smoke_math_tower.xi (already updated on my side) |
| smoke_rc | `use xiom.rc;` — rc moved to `xiom.memory.rc` |
| smoke_utf8 | `use xiom.utf8;` — utf8 moved to `xiom.convert.utf8`; API differs (`utf8_encode` returns Result[Vec[UInt8], Str] in convert, `utf8_validate(s: Str)` vs bytes) |
| smoke_hash_folder | calls bare `to_int_from_char` without importing it (it's in `xiom.convert.toint` — `use xiom.convert.toint;`) |
| smoke_path | was a compiler bug — FIXED; verify with the current compiler |

## Verified compiler state (all green)

- 5 repros + gcm/tuple destructure/map_keys/opt_contract/timer_literal/bare_prelude harnesses: exit 0
- checker 178/178; feature-reg 510/510; diff 24/24; LSP 38/38; MCP 39/39
- crypto smokes 30/30; stdlib-exec 68/72 (remaining 4 = your smoke files above)
- regression clusters (m18_guard/m19_default/alias/ambiguity/string-init): all pass

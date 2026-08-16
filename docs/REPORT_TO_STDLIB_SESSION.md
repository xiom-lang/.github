# Report to stdlib session — UPDATE (2026-08-16, after the BUG 30 regression sweep)

## CORRECTION to the 2026-08-14 report: the 4 stdlib-exec stragglers

The old report labeled smoke_rc / smoke_utf8 / smoke_hash_folder as
stdlib-side layout/API issues. **3 of the 4 were COMPILER bugs** and are now
FIXED (commits `0ddc4500` + `5940ba2c`, verified exit 0):

| Smoke | Old label | Reality | Status |
|-------|-----------|---------|--------|
| smoke_rc | `xiom.memory.rc` move | unsafe-block capture ORDER (HashSet) swapped Rc.new ctx slots + by-value `self` ABI mismatch | **FIXED** |
| smoke_cell | `xiom.memory.cell` move | same capture-order/ABI family | **FIXED** |
| smoke_utf8 | `xiom.convert.utf8` API | ensure `result.len()` on Vec payload → Map.len/Str.len + user-local `result` shadowing | **FIXED** |
| smoke_hash_folder | missing import | `to_int_from_char` not imported (`use xiom.convert.toint;`) | **STDLIB-SIDE — your fix** |

`xiom.utf8` (the legacy module name in stdlib/xiom/string/utf8.xi) still
exists and the smoke uses it — the convert/utf8.xi API differs as documented.

## Verified green (2026-08-16, isolated binary + cargo suites)

- checker **178/178**, feature-reg **510/510**, crypto smokes **30/30**
- All 12 e2e failures from the 04:03 run: **fixed** (b003, m18_guard_0086,
  m19_read_file, m21_vec_edge_004, m35_z24/z10/z29, eco_algo, selfhost
  files, bench_math emit-ir)
- smoke_path's dropped `result is Some =>` ensures can be RESTORED
- smoke_compress_lz4_snappy: full compress/decompress round-trips exit 0
- You can also restore: env.xi home_dir's `ensures: result is Some =>`
  clause (the Option-Some contract binding family is fixed).

## 904-smoke battery — your half of the work

The full `examples/stdlib_smoke/` sweep (904 files, never harnessed before)
shows ~289 failures; the compiler's share is catalogued in
docs/COMPILER_BUGS.md (BUG 30 survey). **YOUR share (stdlib-side):**

1. `smoke_hash_folder.xi` — add `use xiom.convert.toint;` (bare
   `to_int_from_char`).
2. P001 parse errors — e.g. smoke_stress_rand_shuffle (unbalanced brace),
   check all smoke_stress_rand_* files.
3. `undefined variable` / missing-import families — smoke_alloc_basic
   (`ptr` — item C on the compiler side: bare prelude names in user
   modules, so ALSO a compiler item), smoke_net_* (drifted APIs),
   smoke_fmt_formatter_* (they call APIs the fmt module doesn't export).
4. Renamed/absent APIs — smoke_stress_rand_sample_* etc. align to the
   current rand module surface.

The compiler-side list (Map AVs, fmt AVs, void-in-expression,
Bounded/is_finite, BST field-type mixup, BUG 26 #2/#3/#5) is the compiler
session's queue — do NOT work around those in the stdlib.

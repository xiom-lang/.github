# Report to stdlib session -- UPDATE (2026-08-16, after the BUG 30 regression sweep)

## CORRECTION to the 2026-08-14 report: the 4 stdlib-exec stragglers

The old report labeled smoke_rc / smoke_utf8 / smoke_hash_folder as
stdlib-side layout/API issues. **3 of the 4 were COMPILER bugs** and are now
FIXED (commits `0ddc4500` + `5940ba2c`, verified exit 0):

| Smoke | Old label | Reality | Status |
|-------|-----------|---------|--------|
| smoke_rc | `xiom.memory.rc` move | unsafe-block capture ORDER (HashSet) swapped Rc.new ctx slots + by-value `self` ABI mismatch | **FIXED** |
| smoke_cell | `xiom.memory.cell` move | same capture-order/ABI family | **FIXED** |
| smoke_utf8 | `xiom.convert.utf8` API | ensure `result.len()` on Vec payload -> Map.len/Str.len + user-local `result` shadowing | **FIXED** |
| smoke_hash_folder | missing import | `to_int_from_char` not imported (`use xiom.convert.toint;`) | **STDLIB-SIDE -- your fix** |

`xiom.utf8` (the legacy module name in stdlib/xiom/string/utf8.xi) still
exists and the smoke uses it -- the convert/utf8.xi API differs as documented.

## Verified green (2026-08-16, isolated binary + cargo suites)

- checker **178/178**, feature-reg **510/510**, crypto smokes **30/30**
- All 12 e2e failures from the 04:03 run: **fixed** (b003, m18_guard_0086,
  m19_read_file, m21_vec_edge_004, m35_z24/z10/z29, eco_algo, selfhost
  files, bench_math emit-ir)
- smoke_path's dropped `result is Some =>` ensures can be RESTORED
- smoke_compress_lz4_snappy: full compress/decompress round-trips exit 0
- You can also restore: env.xi home_dir's `ensures: result is Some =>`
  clause (the Option-Some contract binding family is fixed).

## 904-smoke battery -- your half of the work

The full `examples/stdlib_smoke/` sweep (904 files, never harnessed before)
shows ~289 failures; the compiler's share is catalogued in
docs/COMPILER_BUGS.md (BUG 30 survey). **YOUR share (stdlib-side):**

1. `smoke_hash_folder.xi` -- add `use xiom.convert.toint;` (bare
   `to_int_from_char`).
2. P001 parse errors -- e.g. smoke_stress_rand_shuffle (unbalanced brace),
   check all smoke_stress_rand_* files.
3. `undefined variable` / missing-import families -- smoke_alloc_basic
   (`ptr` -- item C on the compiler side: bare prelude names in user
   modules, so ALSO a compiler item), smoke_net_* (drifted APIs),
   smoke_fmt_formatter_* (they call APIs the fmt module doesn't export).
4. Renamed/absent APIs -- smoke_stress_rand_sample_* etc. align to the
   current rand module surface.

The compiler-side list (Map AVs, fmt AVs, void-in-expression,
Bounded/is_finite, BST field-type mixup, BUG 26 #2/#3/#5) is the compiler
session's queue -- do NOT work around those in the stdlib.

---

## 2026-08-16 late -- compiler session: BUG 32-38 queue results (for the stdlib session)

FIXED on the compiler side (rebuild tgt_iso to pick up; all verified):
- BUG 38 (is-Some double-check), BUG 38b (generic-receiver methods -- the
  iter family), BUG 32 (Int->ptr cast), BUG 33 (Option[Float128] unwrap),
  BUG 31 (fp128 fneg), BUG 34 (nested Vec[Vec[T]] writes).
- BUG 35 primary shape verified working; extreme variant needs your repro.
- P001: no hang reproduces; the crypto smoke compiles in ~5s.

STILL OPEN (compiler, needs a deep-dive session):
- BUG 37/36: BigFloat-chain Vec-len loop bound + any fp128 op in the loop
  body = deterministic AV even at -O0 with sound IR (repro: t_b37f/
  t_chainloop in my scratch; also reproduced in USER space, no catalog).
  bigfloat_to_float128 still crashes for non-zero values -- your
  three-shape workaround did not fully dodge it. Please re-send the
  EXACT failing consumer shape if you have one.

STDLIB-SIDE (yours):
- smoke_num_saturating: needs real Bounded/Ord impls (confirmed).
- smoke_alloc_basic: needs use xiom.ptr; (confirmed).
- char.xi from_digit: the smoke fails with a FALSE contract violation
  ('requires at 106:13') in the multi-call shape -- the requires contract
  sits on a graceful-fallback fn (stdlib rule BUG 22 #5: no contracts on
  fallback fns). Remove the requires or restructure.
- smoke_collections_vec_push_pop: Vec.is_empty() is broken (returns
  false on an empty vec) -- reproduces at baseline HEAD, compiler item or
  stdlib workaround?
- Stale smokes calling .get(0) on Vec results (smoke_iter_map/collect/
  enumerate/chained_adapters etc.): Vec.get is NOT a Vec method; the
  checker resolves it to a random generic (Int.get / Map.get) and the
  value checks fail. Use indexing [0] (the smokes' len checks pass).
- The iter adapter-chain smokes (map/filter/enumerate/take/skip chains):
  receiver-CALL chains for generic methods + fn-value params still fail
  (reproduces at baseline; B-007-adjacent, on the compiler queue).

Re-triage of your two remaining lists (238 files, current binary):
43 pass / 82 compilefail / 113 runfail. The fmt/Map/rc/cell/utf8 families
you filtered are mostly green now. Full breakdown in COMPILER_BUGS.md.

## GREEN LIGHT (2026-08-17) -- stdlib session may wrap up

The compiler side of the queue is committed (9042e8a2, 74bcc28b,
15d0d3b). You are cleared to finalize and close your stdlib-side items:
the list above (char.xi from_digit contract, Vec.is_empty, stale .get(0)
smoke usage, num_saturating Bounded/Ord, alloc_basic import) plus any
remaining API realignment. Do NOT work around the OPEN compiler items
(BUG 37/36 fp128-chain AV, iter adapter chains) in the stdlib -- they are
queued on the compiler side with repros. The api_freeze harness snapshot
(item B) will be re-aligned by the compiler session; expect it to fail
until then.

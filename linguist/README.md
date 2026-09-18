# Linguist recognition kit for XIOM

Everything needed to open the upstream
[github-linguist/linguist](https://github.com/github-linguist/linguist)
pull request that makes `.xi` a first-class language (language bar entry,
color `#FF8A3D`, syntax highlighting). **Not submitted yet - the upstream
usage gate is not met.** This folder exists so the PR can be assembled in
minutes when it is.

## Current facts (verified 2026-09-18)

- `.xi` is **already assigned to "Logos"** upstream (`languages.yml`:
  extensions `.xm`, `.x`, `.xi`). Any PR adding `.xi` to a new language must
  also add a disambiguation heuristic and Logos samples.
- `.gitattributes` overrides to languages Linguist does not know are
  **ignored**: "Languages that are not yet mentioned in languages.yml will
  not be included in the language statistics, even if you specify something
  like `*.mycola linguist-language=MyCoolLang linguist-detectable`."
  (docs/overrides.md)
- Upstream gate for a new language: **at least 2000 files per extension
  indexed in the last year, excluding forks, distributed across unique
  `:user/:repo` combinations**; results concentrated under the language
  owner are filtered out by the maintainers. A PR submitted while usage is
  owner-concentrated will be closed.

## What our repositories do meanwhile

Each repository with `.xi` files carries a `.gitattributes` that excludes
`.xi` from language statistics (so GitHub does not report "Logos") and maps
highlighting to a close C-like grammar. Replace that mapping with
`*.xi linguist-language=Xiom` on the day the upstream entry merges.

## When eligible - step by step

1. Create the public grammar repository `xiom-lang/xiom-grammar` containing
   the TextMate grammar from `specs/xiom.tmLanguage.json` (scope
   `source.xi`), a README, and a license from Linguist's accepted list
   (MIT and Apache-2.0 both qualify; use the project's dual files).
2. Fork `github-linguist/linguist` and apply:
   - `languages.yml`: the entry from `languages.yml.entry` (leave
     `language_id` out; `script/update-ids` assigns it).
   - grammar: `script/add-grammar https://github.com/xiom-lang/xiom-grammar`
   - samples: copy `samples/*.xi` into `samples/Xiom/`; add at least two
     `.xi` samples for **Logos** as well (shared extension).
   - heuristic: merge `heuristics.draft.yml` into
     `lib/linguist/heuristics.yml`, then validate against the current
     schema.
   - run `script/update-ids`, `bundle exec rake test`.
3. Open the PR using `PR_BODY.md`; it must link a GitHub code search for
   `path:*.xi NOT is:fork` showing third-party usage and state the sample
   license.
4. After merge, replace the local `.gitattributes` mapping in every
   repository with `*.xi linguist-language=Xiom`.

## Samples in this folder

Three representative standard-library modules with SPDX headers, owned by
the project and licensed `MIT OR Apache-2.0` for the PR:

- `samples/sign.xi` - contracts, ed25519 arithmetic, error handling
- `samples/bloom.xi` - generics, containers, bit operations
- `samples/utf.xi` - UTF-8 encode/decode, string handling

## Re-check cadence

Revisit after the public beta has third-party `.xi` projects: run the
upstream search query, count results and repo distribution, and only then
open the PR. The maintainers review the usage policy periodically.

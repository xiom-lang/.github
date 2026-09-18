# Pull request body template (fill every placeholder)

## Summary

Add support for the Xiom programming language (`.xi`).

- Website: https://xiom-lang.org
- Compiler and standard library:
  https://github.com/xiom-lang/xiom and https://github.com/xiom-lang/stdlib
- Grammar: https://github.com/xiom-lang/xiom-grammar (TextMate,
  `source.xi`)

## Usage evidence

GitHub code search (excluding forks):
https://github.com/search?type=code&q=NOT+is%3Afork+path%3A*.xi

<PASTE the search result count and a short note about the distribution
across unique user/repo combinations>

## Files added

- `lib/linguist/languages.yml`: Xiom entry (type programming, color
  `#FF8A3D`, extension `.xi`, `tm_scope: source.xi`).
- `grammars.yml` / `vendor/grammars`: added via
  `script/add-grammar https://github.com/xiom-lang/xiom-grammar`.
- `samples/Xiom/*.xi`: three representative standard-library modules.
- `samples/Logos/*.xi`: disambiguation samples (shared extension).
- `lib/linguist/heuristics.yml`: Xiom vs Logos rule for `.xi`.

## Sample license

The samples are original XIOM standard-library sources, owned by the
project, licensed `MIT OR Apache-2.0`; they may be included under the MIT
license that covers Linguist.

## Checklist

- [ ] `script/update-ids` run (language_id assigned)
- [ ] `bundle exec rake test` passes
- [ ] PR template from the Linguist repository used and fully filled in

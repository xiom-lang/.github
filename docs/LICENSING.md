# XIOM Licensing Policy

Decision (2026-09-16, before any public release):

| Scope | License |
|---|---|
| `xiom-lang/*` (xiom, stdlib, registry, website, playground) | `MIT OR Apache-2.0` (dual) |
| `xiom-foundation/*` code (benchmark-chaos, pulse, engine, db, vector, debugger, book code) | `MIT OR Apache-2.0` (dual) |
| `xiom-foundation/xiom-paper` | text CC-BY-4.0, code `MIT OR Apache-2.0` |
| `xiom-packages/*` first-party packages | `MIT OR Apache-2.0` (dual) |
| `xiom-packages/*` community packages | author's choice, must be OSI-approved and declared |

Rationale: dual is the Rust-ecosystem norm; Apache-2.0 contributes the patent
grant and retaliation clause, MIT keeps GPLv2-only downstream compatibility.
Apache-2.0 alone was rejected because it is GPLv2-incompatible.

## 1. Source-file header template

Two lines at the top of every file that can carry a comment. Copyright first,
SPDX second. The SPDX operator is uppercase `OR`; identifiers are exact.

```
Copyright (c) 2026 Eleftherios Notas and XIOM Foundation
SPDX-License-Identifier: MIT OR Apache-2.0
```

Comment syntax per file type:

| Files | Prefix |
|---|---|
| `.rs` `.xi` `.js` `.ts` `.c` `.h` | `// ` on each line |
| `.ps1` `.sh` `.py` `.toml` `.yml` `.yaml` `.gitignore` | `# ` on each line |
| `.css` | `/* ... */` block |
| `.html` | `<!-- ... -->` block |
| `.md` | leading `<!-- ... -->` block, except generated docs |

Files that cannot carry comments (`.json`, lockfiles, binary assets) are
covered by the repository-level LICENSE files; do not invent `.license`
sidecars yet.

Do not keep prose license lines such as
`// Licensed under the MIT or Apache-2.0 license, at your option.` -
replace them with the two lines above, do not append.

## 2. Repository-level files (required in every repo)

- `LICENSE-MIT` - full MIT text with the copyright line above.
- `LICENSE-APACHE` - full Apache License 2.0 text (take it from a repo that
  already has it, e.g. the stdlib relicense pass, or from
  https://www.apache.org/licenses/LICENSE-2.0.txt).
- `NOTICE` - required when Apache-2.0 is included in the distribution:
  project name, copyright line, and any bundled third-party attributions.
- Remove a bare `LICENSE` file if present; GitHub detects dual licensing from
  the `LICENSE-MIT` / `LICENSE-APACHE` pair. If you prefer to keep `LICENSE`,
  make it a two-line pointer to the pair, not a full license text.

`Cargo.toml` keeps `license = "MIT OR Apache-2.0"` (already set at workspace
level; members inherit with `license.workspace = true`). `package.json` uses
`"license": "MIT OR Apache-2.0"` (valid SPDX expression).

## 3. MIT text for LICENSE-MIT

```
MIT License

Copyright (c) 2026 Eleftherios Notas and XIOM Foundation

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 4. Relicensing rules for the agent pass

1. Replace every Apache-only header
   (`SPDX-License-Identifier: Apache-2.0`) with the dual identifier.
2. Normalize the copyright line to
   `Copyright (c) 2026 Eleftherios Notas and XIOM Foundation`.
3. Skip generated files, third-party vendored code, and files without comment
   syntax; list them in the commit message body if unsure.
4. Do not touch runtime C sources under `runtime/` if they are third-party
   derived - flag them instead.
5. One commit per repo:
   `chore(license): relicense to MIT OR Apache-2.0 (SPDX headers)`.
6. Do not rename or reorder existing functions; this is a header-only change.

## 5. Verification

```
git grep -L "SPDX-License-Identifier: MIT OR Apache-2.0" -- "*.rs" "*.xi" "*.ps1" "*.sh" "*.js" "*.ts"
git grep -l "SPDX-License-Identifier: Apache-2.0$" -- "*.rs" "*.xi"
```

The first command lists files missing the dual identifier; the second must
return nothing (no Apache-only files left).

## 6. Paste-ready agent prompt

```
Relicense this repository to dual MIT OR Apache-2.0. Follow the template in
the xiom-lang/.github repository, docs/LICENSING.md, exactly:

1. In every source file that can carry comments, ensure the first two
   comment lines are:
     Copyright (c) 2026 Eleftherios Notas and XIOM Foundation
     SPDX-License-Identifier: MIT OR Apache-2.0
   Replace any existing license prose or Apache-only SPDX line; do not
   append duplicates.
2. Repository root: add LICENSE-MIT (MIT text from the policy doc) and
   LICENSE-APACHE (full Apache-2.0 text); keep or create NOTICE; remove a
   bare LICENSE file if present.
3. Confirm Cargo.toml license = "MIT OR Apache-2.0" (workspace) and
   package.json "license": "MIT OR Apache-2.0" where applicable.
4. Header-only change: no logic, no renames, no formatting beyond the
   two-line header.
5. Verify with the two git grep commands in the policy doc; both must pass.
6. Commit as chore(license): relicense to MIT OR Apache-2.0 (SPDX headers).
```

## 7. Contributor terms

Inbound = outbound (contributions arrive under the dual license) with a DCO
`Signed-off-by` trailer; no CLA. Add the DCO paragraph to CONTRIBUTING.md when
the governance files are written before the public flip.

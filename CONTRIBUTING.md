# Contributing to XIOM

Thanks for your interest. This is the default contribution guide for every
repository in the `xiom-lang` organization; a repository may override it with
its own copy.

## Where things live

| Repository | Contents |
|---|---|
| `xiom` | Compiler, tooling crates (pkg, lsp, dbg, mcp, fmt, doc), e2e tests |
| `stdlib` | Standard library sources, smoke corpus, KAT vectors, probes |
| `registry` | Package registry service (Node.js + Docker) |
| `website` | Marketing site and documentation sources |
| `playground` | Browser playground |
| `.github` | Organization profile, cross-lane process docs, default templates |

## Prerequisites

- Rust 1.86+ (the workspace pins `rust-version = "1.86.0"`).
- LLVM/Clang on `PATH` - the compiler shells out to `clang` to link programs.
- Python 3 for tooling scripts (`tools/ascii_guard.py`).
- Node.js 20+ for the registry.
- Git with a working `git config user.name` / `user.email`.

## Quick start per repository

Compiler (`xiom-lang/xiom`):

```
cargo build -p xiom                     # debug driver at target/debug/xiom
cargo test --workspace --lib            # unit tests
cargo test -p xiom-codegen --test e2e_tests -- <name-filter>
```

The compiler test suites expect a stdlib checkout. Fetch it at the pinned ref:

```
./scripts/fetch-stdlib.ps1              # PowerShell
./scripts/fetch-stdlib.sh               # bash
```

Without it, stdlib-dependent suites skip loudly; CI sets
`XIOM_REQUIRE_STDLIB=1` so a missing checkout fails instead of skipping.

Standard library (`xiom-lang/stdlib`): the corpus lives in `tests/smoke/` and
runs against a compiler binary (a release build, or one you built yourself
with `-Compiler <path>` once the repo runner lands; the ported sweep tooling
is under `tests/tools/`). Tests must set `XIOM_STDLIB` to the repository root
so the compiler under test uses the checkout, not an installed copy.

Registry (`xiom-lang/registry`):

```
npm ci
npm start        # listens on :3000
docker build -t xiom-registry:dev .
```

## Development rules

- **Probe first.** For a suspected compiler or stdlib defect, reduce it to a
  minimal `.xi` program before changing anything, and keep the probe.
- **Contracts are active.** `requires` / `ensures` clauses abort at runtime
  when violated; a failing contract is a real defect, not noise.
- **ASCII-only files.** Every tracked text file must be pure ASCII (the
  `tools/ascii_guard.py check` gate enforces this). Use `\u{...}` escapes for
  non-ASCII test data.
- **Never commit secrets.** No keys, tokens, or `.env` files. CI secret
  scanning is enabled on public repos.
- **Stage explicit paths.** `git add -A` is discouraged; stage the files you
  changed.
- **One concern per commit and per pull request.**

## Commits

Conventional Commits, imperative mood, one logical change:

```
feat: add Vec.first() fast path
fix: correct tz offset for DST boundaries
docs: describe the registry publish protocol
test: lock punycode round-trips
chore: bump notify to 8.x
refactor: split deflate producer from consumer
```

Breaking changes use `!` (for example `feat!:`) and a `BREAKING CHANGE:`
paragraph in the body.

## Pull requests

1. Fork the repository and branch from `main`
   (`feat/<topic>`, `fix/<topic>`, `docs/<topic>`).
2. Make the change; add or update tests.
3. Run the repository's test commands locally and paste the result in the PR.
4. Sign off your commits: `git commit -s` adds
   `Signed-off-by: Your Name <you@example.com>`. This is our DCO; no CLA is
   required. Contributions are accepted under the project's
   `MIT OR Apache-2.0` license (inbound = outbound).
5. Open the PR using the template; link the issue it closes.
6. A maintainer reviews; required checks must pass; keep the branch rebased
   (linear history is enforced on `main`).

Security issues must not be filed as public issues - see `SECURITY.md`.
Participation is covered by `CODE_OF_CONDUCT.md`.

## Code style

- Rust: `cargo fmt` and `cargo clippy` clean; prefer small functions and
  files; no `unwrap()` on user input paths.
- XIOM: match the surrounding module style; explicit free-call forms where a
  free function exists; document public functions.
- JavaScript: minimal dependencies; keep the server dependency surface small.
- Do not reformat unrelated code in a functional PR.

## Review and response times

Maintainers review on a best-effort basis. A first response usually arrives
within a week; complex language or stdlib changes take longer and may be
redirected to the RFC process (see `GOVERNANCE.md`).

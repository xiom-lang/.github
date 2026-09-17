# xiom-lang/.github

Organization-level defaults and cross-lane process documentation for the
XIOM project. This repository makes changes to shared files possible in one
place instead of touching every repository.

## What lives here

| Path | Purpose |
|---|---|
| `profile/README.md` | The organization profile shown on the `xiom-lang` page |
| `CODE_OF_CONDUCT.md` | Contributor Covenant 2.1 (applies to every repo by default) |
| `CONTRIBUTING.md` | Default contribution guide (DCO, PR flow, test commands) |
| `SECURITY.md` | Vulnerability reporting and coordinated disclosure policy |
| `SUPPORT.md` | Where to ask questions and report problems |
| `GOVERNANCE.md` | Roles, decision process, RFC path, maintainer lifecycle |
| `.github/CODEOWNERS` | Default code owners for repositories without their own |
| `.github/ISSUE_TEMPLATE/` | Default bug report, feature request, contact links |
| `.github/PULL_REQUEST_TEMPLATE.md` | Default pull request checklist |
| `LICENSE-MIT`, `LICENSE-APACHE`, `NOTICE` | Dual-license texts and third-party attribution |
| `docs/` | Cross-lane process docs: release plan, migration runbook, org setup, licensing, naming conventions |

## How the defaults work

GitHub applies the community health files above to any repository in the
organization that does not provide its own copy. Lookup order inside a
repository is `.github/` folder, then the repository root, then `docs/`.

Two conditions matter:

- This repository must be **public** for the defaults to apply; it is the
  first repository that flips visibility at the public launch.
- Issue templates and `config.yml` only work from `.github/ISSUE_TEMPLATE/`,
  which is why they sit there rather than at the root.

## Status

Pre-beta. The repositories are private until the public launch; the plan and
gates are tracked in `docs/RELEASE_INFRA_PLAN.md`.

## Useful documents

- `docs/RELEASE_INFRA_PLAN.md` - release pipeline, VPS setup, registry plan.
- `docs/ORG_SETUP.md` - organization security settings and the public flip.
- `docs/LICENSING.md` - dual MIT OR Apache-2.0 policy and SPDX templates.
- `docs/REPO_MIGRATION_RUNBOOK.md` - how the monorepo was split, with history.
- `docs/NAMING_CONVENTIONS.md` - repository and package naming rules.

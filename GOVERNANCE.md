# XIOM Governance

This document describes how decisions are made today and how that is expected
to evolve. It is intentionally lightweight while the project is pre-1.0 and
maintained by a single lead.

## Current model

XIOM operates under a lead-maintainer (BDFL) model:

- **Lead maintainer:** Eleftherios Notas (`@Lefteris-Notas`).
- The lead maintainer holds final responsibility for the language, compiler,
  standard library, registry, and release process.
- Day-to-day decisions (bug fixes, refactors, documentation, tooling) are
  delegated to maintainers.
- The intent is to evolve to a team-based model as the contributor base
  grows; this document will be amended when that happens.

## Roles

| Role | Can do | Cannot do |
|---|---|---|
| Owner | everything, including org settings and team membership | - |
| Maintainer | merge PRs, triage, cut releases (via environment approval) | change org settings |
| Release approver | approve `release` and `registry-publish` environments | merge or push code |
| Triager | label, close, assign, moderate issues | merge code |
| Contributor | fork, open PRs, discuss | push to protected branches |

Roles are granted through teams, never as individual collaborator grants, and
are removed when inactive.

## Decision process

1. **Routine changes** (fixes, refactors, docs, tests, tooling): pull request
   with one maintainer approval and green required checks. Squash or rebase;
   `main` stays linear.
2. **Language and standard library changes**: an RFC in `xiom-lang/rfcs`
   (repository opening at the public launch). The RFC states motivation,
   design, drawbacks, alternatives, and unresolved questions. Discussion
   happens in the RFC pull request; the lead maintainer decides, records the
   decision, and the RFC is merged or closed with a rationale. No separate
   voting repository exists.
3. **Breaking changes**: only in pre-1.0 minor releases, always accompanied
   by a changelog entry and a migration note.
4. **Releases**: version bumps and changelogs are prepared by a maintainer in
   a release PR; tags are protected; the `release` environment requires
   approval from a release approver. Nothing publishes automatically without
   that approval.
5. **Registry policy** (namespaces, reserved names, ownership transfers,
   moderation): decided by the lead maintainer, documented in the registry
   repository, and applied through the publish API's token scopes.

## Becoming a maintainer

Contributors who demonstrate sustained, high-quality participation - merged
PRs, careful reviews, helpful triage - may be invited by the lead maintainer.
Expectations: adherence to the code of conduct, review responsiveness,
respect for the project's compatibility promises, and no unilateral changes
to protected areas (releases, registry, security).

## Conflicts of interest

Maintainers disclose material conflicts (employment, sponsorship, competing
products) in the relevant issue or PR and recuse themselves from decisions
where the conflict is material. Sponsorship never buys technical decisions.

## Code of conduct

Participation in all spaces is covered by `CODE_OF_CONDUCT.md`. Enforcement
is handled by the lead maintainer; reports go to conduct@xiom-lang.org.
Enforcement actions are recorded privately.

## Amending this document

Changes to this document are made by pull request to `xiom-lang/.github` and
require the lead maintainer's approval while the BDFL model is in effect.

# Security Policy

## Reporting a vulnerability

Do not open a public issue for security problems. Use either channel:

1. **GitHub private vulnerability reporting** (preferred): open the affected
   repository, go to Security -> Advisories -> Report a vulnerability. This
   creates a private advisory only the maintainers can see.
2. **Email:** security@xiom-lang.org. Encrypt with the maintainer PGP key if
   you have it; otherwise send details and we will arrange a secure channel.

Include: affected component and version (`xiom --version`), operating system,
a minimal reproduction, impact assessment, and any suggested fix.

## What qualifies

- Compiler or codegen defects that produce exploitable miscompilation or
  memory corruption.
- Standard library functions with memory-safety, correctness, or
  cryptographic defects (wrong results, side channels, broken primitives).
- Package registry flaws: authentication bypass, signature or checksum
  bypass, path traversal on publish/install, denial of service.
- Supply-chain issues in our release artifacts (binaries, checksums,
  signatures).
- Secrets accidentally committed to a repository.

The following are usually handled as normal bugs, not vulnerabilities:
compiler crashes on malformed input with no code execution, missing
hardening that requires an already-compromised environment, and known
pre-1.0 limitations documented in the repository.

## Response expectations

- Acknowledgement within 72 hours.
- Triage and severity assessment within 7 days.
- Coordinated disclosure: we aim to publish a fix and advisory within 90 days
  of a confirmed report, sooner when a fix is ready. Credit is given unless
  you prefer to remain anonymous.

## Safe harbor

We will not pursue legal action for security research that stays within
scope, avoids privacy violations and data destruction, and gives us
reasonable time to respond before public disclosure. Test only against your
own installations; do not attack shared infrastructure such as the registry
or website beyond non-destructive probing.

## Supported versions

XIOM is pre-1.0. Security fixes target:

| Version | Supported |
|---|---|
| Latest release (0.x) | yes |
| `main` branch | yes (fixes land here first) |
| Older 0.x releases | no - upgrade to the latest release |

Release artifacts carry `SHA256SUMS`; verify checksums before installing.
Signed release artifacts and provenance attestations are being rolled out -
see the release notes for the current status.

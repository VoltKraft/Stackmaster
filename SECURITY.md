# Security policy

Stackmaster controls real infrastructure — hypervisors, hosts,
credentials. Security reports are taken seriously and handled under
responsible-disclosure principles.

See also [docs/SECURITY_MODEL.md](docs/SECURITY_MODEL.md) for the
threat model.

## Reporting a vulnerability

**Do not** open a public GitHub issue for a security vulnerability.

Until the project has a dedicated security contact channel published
here, please use GitHub's private vulnerability reporting on the
repository. If that is not available, send a direct message to the
repository owner on the hosting platform and request an encrypted
channel.

Please include:

- A description of the issue and its impact.
- Steps to reproduce (minimal, if possible).
- The affected version or commit.
- Any proof-of-concept code or logs (redacted if they contain
  secrets).

## What to expect

- **Acknowledgment** within 3 business days.
- **Initial assessment** within 7 business days.
- **Coordinated disclosure** — we will agree on a disclosure date
  that lets us ship a fix before public disclosure.
- Credit in the release notes if you want it. Anonymity is also
  fine.

## Supported versions

During the scaffolding and v0.x phase, only the `main` branch is
supported. Once v1.0 ships, a formal support policy will be
published.

## Out of scope

- Vulnerabilities that require a fully compromised host on which the
  core binary already runs as root.
- Vulnerabilities in third-party providers (Proxmox, Docker, etc.) —
  report those upstream.
- Issues affecting deprecated or experimental providers clearly
  labeled as such.

## Hardening recommendations for operators

- Run behind a reverse proxy that enforces TLS and HSTS.
- Keep the master key outside the container filesystem (env var
  injected by a KMS-backed secret, not a file baked into an image).
- Use an OIDC IdP with MFA; do not rely on the local-account fallback
  for day-to-day admin.
- Restrict network access to the PostgreSQL and Redis instances to
  the control plane only.
- Rotate credentials after any reinstall, upgrade, or personnel
  change.

# Roadmap

Stackmaster is built in focused iterations. Each milestone must be
**end-to-end usable**. We prefer a narrow vertical slice that actually
boots a game server over a wide horizontal slice that cannot.

## v0.1 — "One stack, end to end"

**Goal:** prove the architecture with the smallest meaningful vertical.
A human on a laptop follows the README, ends up with a running
Valheim server on a Proxmox VM via Docker, driven from the
Stackmaster UI.

**Scope**

- One hypervisor provider: `proxmox`.
- One platform provider: `docker`.
- One game provider: `linuxgsm` (Valheim), with SteamCMD as a fallback
  path behind the same interface.
- Minimal reconciler with the full state machine.
- Minimal workflow engine: one concrete workflow (`wake-on-demand`).
  No general-purpose editor yet — YAML only.
- Minimal credential vault:
  - Field-level encryption in PostgreSQL with a master key from env var.
  - Only two credential types: Proxmox API token, SSH key.
  - No rotation UI yet (rotation is a manual SQL + re-apply step).
- **Local accounts only.**
  - Env-var bootstrap admin with forced password change on first login.
  - Administrator-managed local user accounts.
  - Personal Access Tokens for the CLI and automation.
  - External IdP (OIDC) integration is **explicitly deferred** —
    see v0.9.
- RBAC with the two roles, enforced at the API layer.
- Append-only audit log.
- CLI `stackmaster apply -f` and `stackmaster server {start,stop,status}`.
- Docker Compose deployment.

**Out of scope for v0.1**

- Graphical node editor (YAML only).
- Hyper-V, ESXi, Kubernetes, LXC, systemd, Windows providers.
- Multi-node / Helm deployment.
- Delegated vault backends (Vault, SOPS, KMS).
- Rotation UI, MFA, SSO session policies beyond basic.
- Backup/restore UI beyond a CLI command.

**Exit criteria for v0.1**

1. A fresh operator runs `docker compose up -d`, logs in with the
   bootstrap admin account (and changes the password on first login),
   applies an infrastructure YAML, clicks "start" on the Valheim tile,
   and ends up with a reachable Valheim server on a Proxmox VM that
   was powered off beforehand.
2. Every action produces an audit event. No plaintext credential is
   ever present in logs, UI, or audit events.
3. A kill -9 of the worker mid-flight resumes correctly after restart.

## v0.2 — "Second layer of each axis"

- Add platform providers: `kubernetes`, `lxc`.
- Add hypervisor provider: `libvirt`.
- Add game provider: `custom-image` (pull a Docker image, wire env).
- First pass of the graphical node editor — read-only topology view.
- Delegated vault backend: HashiCorp Vault adapter.
- Structured observability: Prometheus metrics endpoint.

## v0.3 — "Workflows become first class"

- Full workflow engine with the node categories from
  [WORKFLOWS.md](WORKFLOWS.md).
- Node editor with edit-mode for workflows, round-tripping to YAML.
- Built-in workflow templates (`start-with-platform-boot`,
  `scheduled-shutdown`, `auto-update-window`) shippable as clones.

## v0.4 — "Windows and the enterprise edges"

- Hypervisor providers: `esxi`, `hyperv`.
- Platform provider: `windows-native`.
- Additional vault backends: SOPS, cloud KMS (AWS/GCP/Azure).
- MFA (TOTP) for local accounts, WebAuthn outlook.
- Fine-grained RBAC behind a feature flag.

## v0.9 — "External IdP integration (OIDC)"

OIDC is deliberately the **last** major capability before v1.0.
Local-account auth has carried the project through every earlier
iteration, so OIDC lands on a system whose auth surface, RBAC,
sessions, and audit trail are already mature and well-tested.

**Scope**

- Native OIDC relying party implemented in the core per
  [ADR-0006](adr/0006-auth-and-oidc.md) (`github.com/coreos/go-oidc`
  + `golang.org/x/oauth2`).
- Authorization Code flow with PKCE, state + nonce verification.
- Group-claim → Administrator/Operator mapping, configurable per
  deployment.
- Refresh-token storage encrypted in the vault.
- Local accounts remain as the permanent fallback — a broken IdP
  never locks the operator out of their own system.
- Conformance matrix in CI: **Authentik** (primary target),
  Keycloak, Authelia, generic mock.
- Documentation of the IdP-side setup as **external** configuration;
  Stackmaster never bundles, installs, or operates an IdP.

**Out of scope for v0.9**

- SCIM user provisioning.
- SAML.
- IdP-initiated login.

**Exit criteria for v0.9**

1. A deployment with `SM_OIDC_*` configured against an external
   Authentik instance authenticates users end-to-end, maps groups
   to roles, and logs them out cleanly.
2. A deployment with `SM_OIDC_*` **not** configured still works
   using local accounts only — no regression for the v0.1–v0.8 path.
3. Conformance tests pass for Authentik, Keycloak, and Authelia.

## v1.0 — "Production-ready single-node"

- All v1.0 features in the README and VISION are implemented.
- Full provider matrix documented and tested.
- Helm chart published.
- Security audit completed; SECURITY.md documents the reporting path.
- Upgrade path documented for every v0.x → v1.0 migration.
- Release cadence: monthly patch, quarterly minor, yearly major.

## Beyond v1.0

- Multi-tenancy (hard isolation between teams).
- Custom, fine-grained RBAC UI.
- Plugin marketplace for community providers.
- Optional integration with Open Match / Agones for projects that
  outgrow Stackmaster's orchestration model.

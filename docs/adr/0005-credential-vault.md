# ADR-0005: Credential vault backend

- **Status:** Accepted
- **Date:** 2026-04-20
- **Tags:** security, credentials, foundation

## Context and problem

Credentials are a first-class resource in Stackmaster (see
[CREDENTIALS.md](../CREDENTIALS.md)). Every secret — Proxmox API
tokens, SSH keys, X.509 pairs, OAuth client secrets, webhook HMAC
keys — is stored once and referenced everywhere else. The question is
**where** the ciphertext lives and **what** wraps it.

Requirements:

- Field-level encryption at rest.
- Envelope encryption so master-key rotation does not require
  re-encrypting every record.
- Pluggable — the same `get/set/rotate/reference` interface must
  accept multiple backends.
- Default backend must be installable in under a minute (aligned
  with the `docker compose up -d` story).
- No plaintext in git, logs, UI, or audit events.

## Options considered

### Option A — Internal PostgreSQL vault

- **Summary:** credential records live in PostgreSQL with per-record
  AEAD-encrypted fields; the data keys are wrapped by a master key
  from env var, file, or external KMS.
- **Pros:**
  - Zero extra service.
  - Shipped as part of the core; sized exactly for our needs.
  - Rotation logic is tightly coupled to our state machine and audit.
  - Easy to back up (one pg_dump + one master-key file).
- **Cons:**
  - We own the crypto correctness bar — must be reviewed carefully.
  - Master key in env var is only as secure as the host's process
    isolation.
- **Effort:** medium.
- **Reversibility:** medium (records can be re-exported to another
  backend behind the same interface).

### Option B — HashiCorp Vault (delegated backend)

- **Summary:** Stackmaster does not store credential plaintext at
  all; it stores **references** to Vault paths, resolves at worker
  time, relies on Vault for leasing and rotation.
- **Pros:**
  - Proven crypto + operational model.
  - Short-lived secrets via dynamic roles.
  - Audit at the vault level.
- **Cons:**
  - Vault becomes a hard dependency.
  - Raises the ops floor for a homelab install.
  - Mapping our credential types to Vault engines takes design work.
- **Effort:** medium.
- **Reversibility:** easy (it is one implementation of the interface).

### Option C — SOPS-encrypted files (delegated backend)

- **Summary:** credentials live as SOPS-encrypted YAML/JSON files,
  optionally in a git repository. The master key can be age / PGP / KMS.
- **Pros:**
  - GitOps-friendly.
  - Offline backups are trivial (they are files).
  - Familiar to homelab operators who already use Flux or Argo CD.
- **Cons:**
  - No good fit for dynamic secrets.
  - Rotation is not transactional — editing a file and re-apply is the
    flow, which is racy in multi-writer setups.
- **Effort:** low-medium.
- **Reversibility:** easy.

### Option D — Cloud KMS (master-key only)

- **Summary:** credentials still live in PostgreSQL, but the master
  key is hosted in AWS/GCP/Azure KMS; data-key wrap/unwrap calls go
  to the KMS.
- **Pros:**
  - Cloud audit, IAM-based access control, hardware-backed keys.
- **Cons:**
  - Cloud dependency.
  - Each wrap/unwrap is a network call (throughput and latency).
- **Effort:** low-medium.
- **Reversibility:** easy.

## Decision

**Option A — internal PostgreSQL vault.** Stackmaster ships with a
built-in, field-level, envelope-encrypted credential store in
PostgreSQL. Options B–D remain on the roadmap as **delegated
backends** behind the same `get/set/rotate/reference` interface, but
they are not required to install or operate Stackmaster.

**Open sub-decisions** to resolve before v0.1 code lands:

- AEAD algorithm: AES-256-GCM vs. XChaCha20-Poly1305.
- Go crypto library choice (stdlib `crypto/aes` + `crypto/cipher`
  vs. `golang.org/x/crypto` for XChaCha20).
- Master-key sources supported at v0.1: env var + file. KMS and
  Vault-Transit slip to v0.2+.

## Consequences

- **Positive:**
  - Default install has no extra dependency beyond PostgreSQL and
    Redis (both already in the stack).
  - Rotation, audit, and state-machine stay in one database — one
    transaction boundary, one backup artifact.
  - The pluggable interface keeps the enterprise path (Vault / SOPS /
    KMS) open without blocking v1.0.
- **Negative:**
  - We own the crypto correctness bar. A formal review before v1.0
    is mandatory.
  - Master-key-via-env leaves the key readable to any process that
    reads `/proc/<pid>/environ` — operators should prefer the file
    source on multi-tenant hosts.
- **Follow-ups:**
  - Finalize the AEAD choice and pin the library.
  - Review Go memory-handling (zeroize after use) given the decision
    in [ADR-0001](0001-backend-language.md).
  - Expand [CREDENTIALS.md](../CREDENTIALS.md) into a concrete
    PostgreSQL migration for iteration 2.
  - Plan the v0.2 HashiCorp Vault adapter as the first delegated
    backend.

## References

- [CREDENTIALS.md](../CREDENTIALS.md)
- [SECURITY_MODEL.md](../SECURITY_MODEL.md)

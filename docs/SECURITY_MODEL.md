# Security Model

This document is the threat model and hardening posture for
Stackmaster. It is a companion to [AUTH.md](AUTH.md),
[RBAC.md](RBAC.md), and [CREDENTIALS.md](CREDENTIALS.md). For
responsible disclosure, see [SECURITY.md](../SECURITY.md).

## Assets to protect

1. **Credentials** — API tokens, SSH keys, X.509 pairs, passwords,
   OAuth client secrets, webhook secrets.
2. **Infrastructure control** — a successful hijack of Stackmaster is
   a hijack of every host it can reach.
3. **Audit log integrity** — the record of who did what.
4. **User identities and sessions** — session cookies, refresh tokens.
5. **Game-server data** — saves, maps, world files.

## Actors

- **Administrator** — trusted, defines infrastructure.
- **Operator** — partially trusted; limited to assigned servers.
- **Anonymous** — unauthenticated HTTP client.
- **Hostile tenant on the same machine** — relevant for self-hosted
  deployments on shared home-lab hardware.
- **Compromised upstream dependency** — supply-chain risk.

## Threats (STRIDE)

| Category                 | Examples in Stackmaster context                                    | Primary mitigation                                 |
|--------------------------|--------------------------------------------------------------------|----------------------------------------------------|
| **S**poofing             | Stolen session cookie, replayed PAT (and forged OIDC token from v0.9) | Short-lived sessions, PAT hashing + revocation, strict issuer/audience checks once OIDC ships |
| **T**ampering            | Mutation of audit log, poisoned manifest apply                     | Append-only log with monotonic IDs; apply diffs    |
| **R**epudiation          | Admin denies deleting a server                                     | Every state change audited with actor + auth path  |
| **I**nformation disclosure | Credential in logs, credential in UI                             | Never render plaintext; log redaction at source    |
| **D**enial of service    | Unbounded WebSocket subscriptions, large manifest apply            | Rate limits, payload caps                          |
| **E**levation of privilege | Operator escapes to admin verbs                                  | RBAC enforced at API layer, not only UI            |

## Guarantees

### Secrets

- No plaintext credential is **written** anywhere outside the vault.
- No plaintext credential is **rendered** in UI, logs, or audit
  events. References only.
- A credential is **decrypted** only in a worker's memory, only for
  the duration of one operation, then zeroized.
- Master key never sits in an application process heap longer than a
  key-wrapping operation.

### Transport

- HTTPS-only in production. The reverse proxy (Traefik in the default
  compose file) enforces HSTS with a long max-age.
- WebSocket upgrades require an authenticated session or token; the
  handshake carries the same cookie/header as REST.
- Internal traffic (core ↔ PostgreSQL, core ↔ Redis) uses TLS when
  the deployment spans hosts; otherwise Unix sockets on a single node.

### Session management

- Session cookies: `__Host-` prefix, httpOnly, Secure, SameSite=Strict
  after login. Local login uses Strict from the first request; OIDC
  (from v0.9) uses SameSite=Lax temporarily for the redirect and
  upgrades to Strict after the callback.
- Idle timeout (default 12h) and absolute timeout (default 30d),
  both configurable.
- Session rotation on privilege change.

### CSRF

- Double-submit cookie pattern for all state-changing REST endpoints
  called from a browser session.
- Bearer-token clients are exempt from CSRF; they carry their own
  anti-forgery via the Authorization header and Origin checks.

### Input validation

- All inputs validated at the API boundary against the OpenAPI schema.
- YAML manifests parsed with a strict schema; unknown fields rejected
  by default with a clear error.

### Audit

- Append-only log with a monotonic sequence ID per record.
- No plaintext credential ever enters the audit log. References are
  recorded; resolvable to a credential ID, never to plaintext.
- Optional tamper-evidence via hash-chained rows (post-v1.0).

### Supply chain

- Pinned dependencies with lockfiles checked in.
- Signed release artifacts (sigstore) — post-v1.0.
- CI runs SAST + dependency scanning on every PR.
- Providers are part of the core binary in v1.0; out-of-tree plugin
  loading is explicitly out of scope for v1.0 to avoid extending the
  attack surface before it is designed carefully.

## Operator assumptions

Stackmaster expects:

- The host running the core binary is trusted.
- The PostgreSQL database is reachable only by the core and workers.
- Redis is reachable only by the control plane.
- The reverse proxy terminates TLS with a trusted certificate.

## Non-assumptions (threats we do **not** expect to mitigate)

- A fully compromised host running the core binary (no hardware root
  of trust is planned for v1.0).
- Malicious administrators (by definition, administrators can do
  anything an administrator can do).
- Side-channel attacks on the host kernel.

## Reporting

See [SECURITY.md](../SECURITY.md) for the responsible-disclosure path.

## TODO

- [ ] Detail the redaction strategy for provider-returned error
      messages that may accidentally carry credentials.
- [ ] Decide on a default Content-Security-Policy for the UI.
- [ ] Specify cookie names per deployment (single-tenant vs. reverse
      proxy with subpaths).

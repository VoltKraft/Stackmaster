# ADR-0006: Auth stack and OIDC integration

- **Status:** Accepted
- **Date:** 2026-04-20
- **Tags:** security, authentication, foundation

## Context and problem

Stackmaster must authenticate humans via OIDC (primary) and local
accounts (baseline), and machine clients via Personal Access Tokens
(see [AUTH.md](../AUTH.md)). The open questions:

1. **Library vs. proxy.** Implement OIDC natively in the core, or
   front the core with an OIDC-terminating reverse proxy?
2. **Which library.** If native, which one per language candidate?
3. **Session store.** PostgreSQL rows vs. Redis vs. signed cookies
   only.
4. **Bootstrap UX.** How the first admin account is created on a
   fresh install.

## Options considered

### Option A — Native OIDC in the core

- **Summary:** the core binary implements the OIDC flow directly,
  issues session cookies, manages PATs.
- **Pros:**
  - One deployable service.
  - Full control over session semantics (rotation on role change,
    scoped PATs, back-channel logout).
  - Audit events emitted from the same process that does
    authorization.
- **Cons:**
  - We carry the correctness bar for token validation, PKCE, state,
    nonce, audience checks, key rollover.
  - Subtle OIDC edge cases (aud arrays, HS256 vs. RS256, ID-token
    key rotation) are easy to get wrong if we roll our own.

**Candidate libraries:**
- Go: `github.com/coreos/go-oidc` + `golang.org/x/oauth2`.
- Python: `authlib` or `python-oidc-client`.

### Option B — Front the core with oauth2-proxy / OAuth2 Proxy

- **Summary:** the core trusts a proxy (oauth2-proxy, Authelia,
  Traefik ForwardAuth) that terminates OIDC and injects headers.
- **Pros:**
  - Battle-tested OIDC implementation.
  - Offloads crypto from our codebase.
- **Cons:**
  - Two services to deploy and reason about.
  - Header-trust model is fragile — any misconfiguration that lets
    a request bypass the proxy is a full auth bypass.
  - PATs, service accounts, and back-channel logout still need
    in-core logic; the proxy does not remove that surface.

### Option C — Hybrid

- **Summary:** native OIDC as default; allow deployments to offload
  to a proxy when they already run one.
- **Pros:** operator flexibility.
- **Cons:** maintenance burden of two paths.

## Other sub-decisions

### Session store

- **Signed cookie only:** simplest, but no server-side revocation;
  rejected.
- **Redis:** fast, naturally expires, fits our stack; sessions get
  lost on Redis restart — acceptable if PostgreSQL holds durable
  refresh tokens.
- **PostgreSQL:** durable but noisier on the DB.

Leaning: **Redis** for active sessions, **PostgreSQL** for refresh
tokens and PATs.

### Bootstrap

- **Env-provided bootstrap admin credential** with forced change on
  first login: familiar, easy to automate.
- **One-time token printed to logs on first start**: safer for
  stateless cloud deployments; harder for Docker Compose operators who
  may not read logs.

Leaning: **one-time token printed to logs**, with an optional
env-provided admin for CI scenarios.

## Decision

- **Native OIDC in the core** (Option A). Stackmaster implements
  the OIDC relying-party flow directly using `github.com/coreos/go-oidc`
  and `golang.org/x/oauth2` (consistent with the Go decision in
  [ADR-0001](0001-backend-language.md)).
- **Session store:** Redis for active sessions, PostgreSQL for
  refresh tokens and PATs.
- **Bootstrap UX:** **environment variable**. On first start, if no
  admin exists, Stackmaster creates one from
  `SM_BOOTSTRAP_ADMIN_EMAIL` + `SM_BOOTSTRAP_ADMIN_PASSWORD`, then
  forces a password change on first login. The env vars are read
  once and then ignored.
- **Primary conformance target:** **Authentik** (tested first).
  Keycloak, Authelia, Google, GitHub, and Microsoft Entra ID remain
  on the conformance matrix. **Stackmaster does not bundle, ship,
  or install an IdP** — it is a relying party against an IdP the
  operator already runs.
- **Sequencing.** The **local-account** parts of this decision
  (env-var bootstrap, Argon2id, PATs, session store, RBAC
  enforcement) ship in Iteration 2 / v0.1 and carry the project
  through v0.8. The **OIDC** parts ship in **v0.9** — deliberately
  the last major capability before v1.0. See
  [ROADMAP.md](../ROADMAP.md). Local accounts remain as a permanent
  fallback after v0.9 so a broken IdP never locks operators out.

## Consequences

- **Positive:**
  - One service, one audit surface, full control over session
    lifecycle and PAT scoping.
  - Env-var bootstrap keeps the `docker compose up -d` story simple.
- **Negative:**
  - We own the OIDC correctness bar. When OIDC ships in v0.9, CI
    must include conformance tests against Authentik, Keycloak,
    Authelia, and a generic mock.
  - Env-var bootstrap means the plaintext bootstrap password is
    readable via `/proc/<pid>/environ` on the host. Mitigation:
    forced change on first login; env vars cleared from the process
    env after first read where the OS allows.
  - Deferring OIDC to v0.9 means early adopters can only use local
    accounts. Accepted trade-off: the v0.1 demo path does not need
    SSO, and a mature local-auth surface is a better foundation to
    add OIDC onto than the other way round.
- **Follow-ups:**
  - **Iteration 2 / v0.1:** implement local accounts, env-var
    bootstrap, sessions, PATs, RBAC enforcement.
  - **v0.9:** pin `go-oidc` and `oauth2` versions, implement the
    relying-party flow, build the conformance suite
    (Authentik / Keycloak / Authelia / mock), document the
    Authentik setup in [AUTH.md](../AUTH.md) as **external**
    configuration not shipped with Stackmaster.

## References

- [AUTH.md](../AUTH.md)
- [SECURITY_MODEL.md](../SECURITY_MODEL.md)
- [RBAC.md](../RBAC.md)

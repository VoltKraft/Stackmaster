# Iteration 2 — Implementation plan

This document is the concrete plan for **Iteration 2** of Stackmaster.
Iteration 1 finished the documentation scaffolding and accepted the
eight foundation ADRs (see [adr/README.md](adr/README.md)). Iteration 2
turns that paper trail into a running system that can provision one
stack end-to-end.

The target at the end of Iteration 2 is the **v0.1 demo scenario**
defined in [ROADMAP.md](ROADMAP.md): a single Valheim server on
LinuxGSM on Docker on Proxmox, brought up via `stackmaster apply -f`,
visible live in the UI to all logged-in operators, behind **local
account** login with an env-var bootstrap admin.

OIDC is **deliberately out of scope** for Iteration 2. It is the
last major capability before v1.0 (see ROADMAP v0.9) and lands on a
mature auth surface rather than blocking the v0.1 demo.

Iteration 2 does **not** yet deliver the generic workflow engine, the
node editor, delegated vault backends, most providers, or OIDC.
Those are explicitly deferred (see *Outlook* at the bottom).

## Guiding principles

- **One stack, end-to-end, before breadth.** Every milestone is judged
  by whether it moves the Valheim-on-Proxmox demo forward.
- **Schema before code.** Migrations, DB schema and API contracts are
  pinned before handlers are written, so the UI and the core can be
  built in parallel.
- **Real-time from day one.** The events outbox, Redis pub/sub and
  WebSocket hub are built into the skeleton, not bolted on later
  (see [ARCHITECTURE.md §10](ARCHITECTURE.md)).
- **Secrets never in code or logs.** The credential vault comes online
  before any provider that needs credentials.
- **Reversible by default.** Every destructive operation is guarded
  and audited.

## Milestones

Each milestone has explicit **deliverables** and **exit criteria**. A
milestone is not "done" until its exit criteria are demonstrable.

### M2.1 — Repo skeleton & CI

**Goal.** Make the repo buildable and testable on every push.

**Deliverables**

- `core/` Go module with the package layout below.
- `ui/` Vite + React + TypeScript skeleton with routing stubs.
- `cli/` Go module sharing types with `core/` via an internal package.
- GitHub Actions: `go build`, `go test ./...`, `golangci-lint`,
  `pnpm build`, `pnpm test`, `pnpm lint`.
- Pre-commit config (gofmt, goimports, eslint, prettier).
- `Makefile` targets: `make dev`, `make test`, `make lint`, `make db-up`.

**Exit criteria**

- `make test` is green on a clean checkout.
- CI is green on `main`.
- `docker compose -f deploy/compose.yaml up -d postgres redis` is
  sufficient to run the test suite locally.

**Proposed Go package layout (`core/`)**

```
core/
  cmd/stackmaster/           # binary entrypoint
  internal/
    api/                     # chi router, handlers, middleware
    auth/                    # local accounts, sessions, PATs, password hashing
    rbac/                    # role check, policy matrix
    vault/                   # credential store (envelope encryption)
    events/                  # outbox + Redis pub/sub + WS hub
    reconciler/              # desired vs observed loop
    providers/
      hypervisor/proxmox/
      platform/docker/
      game/linuxgsm/
    workflow/                # DAG runtime (v0.1: wake-on-demand only)
    store/                   # sqlc-generated DB access
    migrations/              # goose migrations (embedded)
    config/                  # env loading (caarlos0/env)
  pkg/
    apitypes/                # shared DTOs for CLI + UI
```

### M2.2 — Database schema & migrations

**Goal.** A single source of truth for persistent state, with
reversible migrations.

**Deliverables**

- Goose migrations (embedded in the binary) for:
  - `users`, `sessions`, `personal_access_tokens`
  - `roles`, `role_bindings` (Administrator + Operator for v1.0)
  - `credentials`, `credential_versions`, `data_keys`
  - `hypervisor_connections`, `platforms`, `game_servers`
  - `desired_states`, `observed_states`
  - `workflows`, `workflow_runs`, `workflow_steps`
  - `events_outbox` (for the real-time sync pattern)
  - `audit_log`
- sqlc configuration and generated query code.
- Seed data for dev: a default `Administrator` role and a noop
  hypervisor connection.

**Exit criteria**

- `goose up` and `goose down` are both clean across every migration.
- `sqlc generate` produces no diff in CI.
- Schema documented in [ARCHITECTURE.md](ARCHITECTURE.md) or a new
  `docs/SCHEMA.md`.

### M2.3 — Credential vault

**Goal.** A working internal Postgres vault per
[ADR-0005](adr/0005-credential-vault.md) and
[CREDENTIALS.md](CREDENTIALS.md).

**Deliverables**

- Envelope encryption: master key wraps per-credential data keys.
- AEAD: **AES-256-GCM** (resolves the open sub-decision in ADR-0005).
- Master key loaded from `SM_MASTER_KEY` (base64) or a file path.
- API: `POST /api/v1/credentials`, `GET`, `DELETE`, `POST /rotate`.
- Reference resolution: `ref://creds/<alias>[@<version>]`.
- Audit log entry on every read, write, rotate, delete.
- Redaction middleware: no credential value ever reaches logs.

**Exit criteria**

- Round-trip test: store secret → reference resolves → plaintext matches.
- Data-key rotation test: rotating the master key re-wraps data keys
  without touching ciphertext.
- Credential rotation test: rotating a credential creates a new
  version; old references still resolve until revoked.

### M2.4 — Local auth, sessions, RBAC

**Goal.** A user can log in with a local account, create more users,
issue PATs for the CLI, and every mutating endpoint is gated by a
role check. OIDC is **not** part of this milestone — it is scheduled
for v0.9 (see [ROADMAP.md](ROADMAP.md)).

**Deliverables**

- Local login: Argon2id password hashing, rate-limited login with
  exponential backoff per IP + per account.
- **Env-var bootstrap admin** per
  [ADR-0006](adr/0006-auth-and-oidc.md): on first start, if no admin
  exists, one is created from `SM_BOOTSTRAP_ADMIN_EMAIL` +
  `SM_BOOTSTRAP_ADMIN_PASSWORD` and a password change is forced on
  first login. The env vars are read once and then ignored.
- Admin-driven user management: create, disable, delete users; reset
  passwords.
- Sessions: `__Host-sm_session` cookie, Redis-backed, idle + absolute
  timeouts, rotation on privilege change.
- PAT issuance (shown once, stored hashed).
- RBAC middleware using the matrix in [RBAC.md](RBAC.md).

**Exit criteria**

- Bootstrap admin test: fresh start with env vars creates the admin;
  second start with the same env vars is a no-op.
- Forced-change test: first login prompts for a new password and
  rejects the original one.
- User management test: admin creates an Operator, the Operator logs
  in, can hit Operator endpoints, cannot hit admin-only endpoints.
- PAT test: issue PAT, authenticate a CLI call with it, revoke it,
  confirm the next call fails.

### M2.5 — Real-time events & outbox

**Goal.** Every state change is visible live in the UI to all
logged-in users, per [ARCHITECTURE.md §10](ARCHITECTURE.md).

**Deliverables**

- Transactional outbox (`events_outbox` table) written in the same
  transaction as the state change.
- Outbox dispatcher: publishes to Redis pub/sub **and** appends to a
  Redis Stream (for replay).
- WebSocket hub (`coder/websocket`) at `/ws/events`:
  - Authenticated via session cookie.
  - Subscriptions RBAC-filtered (an Operator never sees events for
    resources they can't read).
  - Monotonic `seq` per event; reconnect replays from Redis Stream
    starting at `last_seq + 1`.
- Event envelope documented in [API.md](API.md) (already drafted).
- Event kinds for v0.1: `gameserver.*`, `credential.*`, `audit.*`.

**Exit criteria**

- Two browser tabs, one Administrator and one Operator: a state change
  appears in both within <500 ms, **without a page reload**.
- Reconnect test: disconnect a client for 30 s, reconnect, receive
  every missed event in order.

### M2.6 — UI skeleton

**Goal.** A usable, not-pretty UI that covers the v0.1 surface.

**Deliverables**

- Routes: `/login` (local credentials), `/dashboard`, `/stacks`,
  `/stacks/:id`, `/credentials`, `/users`, `/settings`.
- TanStack Query for REST; a thin `useLiveEvents()` hook that
  subscribes to `/ws/events` and invalidates queries on relevant
  event kinds.
- Zustand store for session + WS connection state.
- Tailwind for styling (no design system yet).
- React Flow installed and rendered on an empty `/topology` page as
  a smoke test (no editor logic yet — deferred to Iteration 3).
- Playwright smoke test: login → see dashboard.

**Exit criteria**

- Local login works from the browser against a fresh bootstrap admin.
- The game-server list updates live when `stackmaster apply -f` is
  run from the CLI.

### M2.7 — First Proxmox provider + `apply`

**Goal.** `stackmaster apply -f examples/manifests/valheim-...yaml`
brings a Valheim server to `Running` on a real Proxmox host.

**Deliverables**

- `HypervisorProvider` for Proxmox (create VM, start, stop, delete).
- `PlatformProvider` for Docker (install via cloud-init, run container).
- `GameProvider` for LinuxGSM (install, start, stop, status).
- Reconciler loop (asynq) with retry + exponential backoff.
- Provider conformance suite (per [PROVIDERS.md](PROVIDERS.md)),
  executed in CI with a noop hypervisor + Docker-in-Docker platform.
- `stackmaster apply`, `stackmaster get`, `stackmaster describe`
  wired to the API.

**Exit criteria**

- Manifest applied against a real Proxmox test host reaches `Running`.
- State transitions appear live in the UI.
- Destroy path works and leaves no orphaned VM / container / files.

## Library pin matrix

Pin in Iteration 2 and don't churn. Upgrades are their own PRs.

**Backend (Go, module `core/`)**

| Library                        | Purpose                                 |
|--------------------------------|-----------------------------------------|
| `github.com/go-chi/chi/v5`     | HTTP routing                            |
| `github.com/jackc/pgx/v5`      | Postgres driver                         |
| `github.com/sqlc-dev/sqlc`     | Type-safe queries (build-time)          |
| `github.com/pressly/goose/v3`  | Migrations (embedded)                   |
| `github.com/hibiken/asynq`     | Reconciler task queue                   |
| `github.com/redis/go-redis/v9` | Redis client (sessions, pub/sub, stream)|
| `github.com/coder/websocket`   | WebSocket hub                           |
| `golang.org/x/crypto/argon2`   | Password hashing                        |
| `log/slog`                     | Structured logging (stdlib)             |
| `github.com/caarlos0/env/v11`  | Env-var config                          |
| `github.com/stretchr/testify`  | Test assertions                         |
| `testcontainers-go`            | Integration tests (Postgres, Redis)     |

OIDC libraries (`github.com/coreos/go-oidc`, `golang.org/x/oauth2`)
are **not** pinned in Iteration 2 and do not enter the dependency
graph. They will be added in the Iteration that delivers v0.9.

**Frontend (`ui/`)**

| Library                | Purpose                                  |
|------------------------|------------------------------------------|
| `react` (18+)          | UI runtime                               |
| `react-router-dom` (6+)| Routing                                  |
| `@xyflow/react` (v12+) | React Flow (node editor foundation)      |
| `@tanstack/react-query`| Server state                             |
| `zustand`              | Client state                             |
| `tailwindcss`          | Styling                                  |
| `@playwright/test`     | E2E smoke tests                          |

## Risks & mitigations

| Risk                                                       | Mitigation                                                                                     |
|------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| Real-time sync race: client misses events during reconnect | Redis Stream replay from `last_seq`; deterministic ordering on the server side.                |
| Credential leakage via logs                                | Structured logging with explicit redaction; lint rule bans `fmt.Sprintf` on credential types.  |
| Provider flakiness against Proxmox test host               | Provider conformance suite runs against noop first; Proxmox is a separate nightly job.         |
| Scope creep into workflow engine                           | v0.1 ships only a **hard-coded** wake-on-demand workflow; generic engine is Iteration 3.       |
| Skipping OIDC now makes the v0.9 integration painful       | Keep the auth seam abstract (`AuthProvider` interface) so OIDC slots in alongside local auth.  |
| sqlc + pgx type mismatch on custom types                   | Keep custom types minimal; document every `CREATE TYPE` in `docs/SCHEMA.md`.                   |
| Outbox dispatcher falls behind                             | Depth gauge + alert; backpressure via bounded channel; dispatcher is horizontally scalable.    |

## Outlook: Iteration 3 and beyond

Iteration 2 intentionally defers:

- **Generic workflow engine** — v0.1 has one hard-coded workflow
  (wake-on-demand). Iteration 3 delivers the typed-DAG engine and the
  YAML ↔ graph isomorphism from [WORKFLOWS.md](WORKFLOWS.md).
- **Node editor** — React Flow is installed in Iteration 2 but the
  editor UX (palette, inspector, validation, run overlay) is
  Iteration 3, per [NODE_EDITOR.md](NODE_EDITOR.md).
- **More providers** — libvirt, Kubernetes, SteamCMD, custom-image
  land in Iteration 3+. The conformance suite from M2.7 is the gate.
- **Delegated vault backends** — Vault, SOPS, cloud KMS per
  [ADR-0005](adr/0005-credential-vault.md) are a v0.4+ concern.
- **Backups, scheduled maintenance windows, console streaming,
  file browser** — all v0.2–v0.4 per [ROADMAP.md](ROADMAP.md).
- **OIDC / external IdP integration** — the last major capability
  before v1.0, scheduled for **v0.9**. Until then, Stackmaster
  authenticates users via local accounts only.

## Tracking

Each milestone becomes a GitHub milestone with an issue per
deliverable. Exit criteria become acceptance checklists on the
milestone itself. PRs reference the milestone and the relevant doc
sections. No milestone is closed until its exit criteria are
demonstrable on `main`.

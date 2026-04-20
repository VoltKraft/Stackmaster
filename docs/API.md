# API

Stackmaster exposes one unified API on two transports:

- **REST/JSON over HTTPS** for CRUD and commands, log streams, console passthrough, and
  live reconciler events.

The UI and the CLI speak the same API. A third-party TUI, mobile app,
or integration layer can be built against it without special-casing.

## Source of truth

An OpenAPI document will live at [`../api/openapi.yaml`](../api/)
(path reserved; file created in iteration 2). That document is
authoritative. This Markdown file describes the shape informally.

## Conventions

- All identifiers are UUIDv7.
- All timestamps are RFC 3339 UTC.
- All list endpoints support `?limit=` and `?cursor=` (cursor-based
  pagination).
- State-changing endpoints require CSRF protection (double-submit
  cookie) in the browser. Bearer tokens in non-browser clients are
  exempt because they carry their own anti-forgery.
- Errors follow [RFC 7807](https://www.rfc-editor.org/rfc/rfc7807)
  Problem Details.

## Authentication

- Session cookie (`__Host-sm_session`) — httpOnly, Secure,
  SameSite=Strict after login.
- Machine clients use a **Personal Access Token** (bearer). Tokens are
  scoped and revocable. Tokens are hashed at rest (Argon2id).
- Local accounts are the only auth path through v0.8. External IdP
  (OIDC) integration is planned for v0.9 (see [AUTH.md](AUTH.md)
  and [ROADMAP.md](ROADMAP.md)); when it ships, OIDC access tokens
  are **never persisted**, and refresh tokens are stored encrypted
  in the vault.

See [AUTH.md](AUTH.md).

## Resource sketch

| Resource                  | Methods                                   | Notes                                   |
|---------------------------|-------------------------------------------|-----------------------------------------|
| `/hypervisors`            | `GET POST`                                | Register a hypervisor connection.       |
| `/hypervisors/{id}`       | `GET PATCH DELETE`                        |                                         |
| `/platforms`              | `GET POST`                                | Register a platform target.             |
| `/platforms/{id}`         | `GET PATCH DELETE`                        |                                         |
| `/servers`                | `GET POST`                                | Game-server records.                    |
| `/servers/{id}`           | `GET PATCH DELETE`                        |                                         |
| `/servers/{id}:start`     | `POST`                                    | Sets desired state to `Running`.        |
| `/servers/{id}:stop`      | `POST`                                    | Sets desired state to `Stopped`.        |
| `/servers/{id}:restart`   | `POST`                                    | Stop+Start, single audit event.         |
| `/servers/{id}/backups`   | `GET POST`                                | Backup list / new backup.               |
| `/servers/{id}/backups/{b}:restore` | `POST`                          | Restore from backup.                    |
| `/servers/{id}/files`     | `GET POST PATCH DELETE`                   | In-browser file editor.                 |
| `/servers/{id}/events`    | `GET`                                     | Audit/state events for this server.     |
| `/workflows`              | `GET POST`                                | List / create workflows.                |
| `/workflows/{id}`         | `GET PATCH DELETE`                        |                                         |
| `/workflows/{id}:run`     | `POST`                                    | Manual trigger.                         |
| `/workflows/{id}/runs`    | `GET`                                     | Past runs.                              |
| `/credentials`            | `GET POST`                                | Create/list references; no plaintext.   |
| `/credentials/{id}:rotate`| `POST`                                    | Trigger rotation.                       |
| `/users`                  | `GET POST`                                | Admin only.                             |
| `/roles`                  | `GET`                                     | Two roles in v1.0 (see RBAC.md).        |
| `/audit`                  | `GET`                                     | Append-only log; filterable.            |
| `/health` / `/ready`      | `GET`                                     | Liveness / readiness.                   |

## WebSocket endpoints

| Endpoint                              | Carries                                                                             |
|---------------------------------------|-------------------------------------------------------------------------------------|
| `/ws/servers/{id}/logs`               | Live log stream.                                                                    |
| `/ws/servers/{id}/console`            | Bidirectional stdin/stdout.                                                         |
| `/ws/events`                          | **All** state-change events, RBAC-filtered. Every logged-in user stays in sync.     |

All WebSocket upgrades require a valid session or token. The first
message from the client is a subscription envelope (topics + last
seen sequence id); the server replies with an ACK before streaming
data.

### Real-time sync contract (`/ws/events`)

Stackmaster guarantees that every state-changing action is visible
to **all** currently-connected users without a page reload
(see [ARCHITECTURE.md §10](ARCHITECTURE.md)). The contract:

- Every create / update / delete / transition produces one event.
- Events carry a **monotonic `seq`** per workspace, a typed `kind`,
  and a **redacted** payload — never credential plaintext.
- Subscribers filter by **topic** (e.g. `gameserver.*`,
  `workflow.runs`, `schedule.*`, `topology`) and are additionally
  filtered server-side by the subscriber's RBAC scope.
- On reconnect, clients send the last seen `seq`; the server replays
  missed events from a short-TTL Redis stream or responds
  `resync-required`, upon which the client re-fetches the affected
  resources from REST.

Event envelope (illustrative):

```json
{
  "seq": 12874,
  "ts": "2026-04-20T10:12:33.421Z",
  "kind": "gameserver.state_changed",
  "actor": { "type": "user", "id": "018f...", "auth": "local" },
  "resource": { "type": "gameserver", "id": "018f...", "name": "valheim-saturday" },
  "payload": {
    "from": "HealthChecking",
    "to": "Running",
    "reason": "probe ok"
  }
}
```

Event kinds (non-exhaustive):

- `gameserver.*` — created, updated, deleted, state_changed,
  backup_created, files_changed.
- `workflow.*` — created, updated, deleted, run_started,
  run_completed, run_failed.
- `schedule.*` — created, updated, deleted (cron triggers, update
  windows, shutdown schedules).
- `topology.*` — hypervisor / platform connection added or removed.
- `credential.*` — metadata changes only; never plaintext.
- `audit.*` — new audit event (metadata; body fetched via REST for
  scope).

CLI `stackmaster ... watch` and `stackmaster audit tail` use the
same channel with the same semantics — the CLI user sees the same
world at the same moment as a browser user.

## Example payloads (illustrative)

```json
// POST /servers
{
  "name": "valheim-saturday",
  "template": "linuxgsm:valheim",
  "platform": "ref://platforms/gameplat-01",
  "desired_state": "Running",
  "credentials": {
    "steam_login": "ref://creds/steam-anon"
  },
  "ports": [{"port": 2456, "proto": "udp"}],
  "probe": {"type": "udp", "port": 2456, "timeout_s": 60}
}
```

```json
// GET /servers/{id}
{
  "id": "018f...",
  "name": "valheim-saturday",
  "desired_state": "Running",
  "observed_state": "HealthChecking",
  "last_transition_at": "2026-04-20T10:12:33Z",
  "platform": {"id": "...", "name": "gameplat-01", "provider": "docker"},
  "hypervisor": {"id": "...", "name": "pve-01", "provider": "proxmox"},
  "probe": {"last": "ok", "at": "2026-04-20T10:12:30Z"}
}
```

## Rate limits

- Per-session: 60 state-changing requests per minute, soft.
- Per-token: configurable per token.
- WebSocket: max 10 concurrent subscriptions per session.

## Versioning

- URL-prefixed: `/api/v1/…`.
- Breaking changes cut a new major prefix. Old prefixes stay until the
  next LTS.

## TODO

- [ ] Translate this sketch into `api/openapi.yaml`.
- [ ] Decide whether bulk endpoints belong in v1 or v2.
- [ ] Specify pagination cursor encoding formally.
- [ ] Document the event envelope format used by `/ws/events`.

# Stackmaster
00
> The conductor for your game-server stack. Hypervisor, platform, game —
> one pane of glass, one click, one source of truth.

**Status:** Pre-alpha. Scaffolding only. No application code yet.
**License:** AGPL-3.0-or-later (see [LICENSE](LICENSE) and [ADR-0004](docs/adr/0004-license.md)).

---

## What is Stackmaster?

Stackmaster is a self-hosted, open-source **control plane** that orchestrates
the entire stack beneath a game server:

1. **Hypervisor Layer** — Proxmox, libvirt/KVM, ESXi, Hyper-V, bare metal.
2. **Platform Layer** — Docker, Docker Swarm, Kubernetes, LXC, systemd, Windows.
3. **Game Layer** — SteamCMD, LinuxGSM, custom images, scripts.

An administrator defines the infrastructure once in a graphical node editor.
An authorized operator later clicks "start server X" — and Stackmaster
transparently walks the stack: is the platform running? Is the host
beneath it running? If not, the missing layer is provisioned or booted,
the game server is installed and started, and only when a health check
passes does the UI report **ready**.

Stackmaster is **not** another Pterodactyl. It does not host game daemons
itself. It is the **master** that drives existing tools — including
Pterodactyl and Pelican — through a unified provider interface. See
[docs/COMPARISON.md](docs/COMPARISON.md) for a fair side-by-side.

## Why "Stackmaster"?

Two ideas carried consistently across the project:

- **Stack** — three vertical layers (hypervisor → platform → game) woven
  into one running service.
- **Master** — the orchestrator role. A single pane of glass, the
  game-master metaphor of one coordinator running a complex operation.

## Core feature goals (v1.0)

- Full game-server lifecycle (deploy, start, stop, restart, backup,
  restore, update, in-place reconfigure).
- Full **platform lifecycle** and **hypervisor lifecycle** behind the
  same UI — Stackmaster is the only panel that touches all three layers.
- Graphical node editor with two purposes: infrastructure topology and
  workflow DAGs. The graph and its YAML are isomorphic.
- Built-in encrypted credential vault with field-level encryption, key
  rotation, and optional delegation to Vault / SOPS / cloud KMS.
- Local accounts with env-var bootstrap admin from v0.1; OIDC
  authentication with group-to-role claim mapping added in v0.9
  (deliberately the last major capability before v1.0).
- RBAC with two clearly separated roles in v1.0 — Administrator and
  Operator — prepared for fine-grained permissions later.
- Append-only audit log for every security-relevant event.
- Live console access and command execution via WebSocket.
- **Real-time UI sync.** Every state change (game-server state,
  workflow run, schedule edit, topology change) propagates over
  WebSocket to **all** logged-in users immediately — no page reload
  required. See [docs/ARCHITECTURE.md §11](docs/ARCHITECTURE.md).
- Everything-as-code: `stackmaster apply -f infrastructure.yaml` is a
  first-class path equivalent to the UI.

See [docs/VISION.md](docs/VISION.md) for the long form and
[docs/ROADMAP.md](docs/ROADMAP.md) for the staged path to v1.0.


## Architecture at a glance

```
 ┌───────────────────────────────────────────────────────────┐
 │                        Stackmaster                        │
 │   UI (SPA)  ─────┐                                        │
 │                  ▼                                        │
 │             REST + WebSocket API                          │
 │                  │                                        │
 │   ┌──────────────┴────────────┐                           │
 │   │ Reconciler (desired→real) │◀── Workflow Engine (DAG)  │
 │   └──────────────┬────────────┘                           │
 │                  ▼                                        │
 │             Provider layer                                │
 │     ┌──────────┬─────────────┬──────────┐                 │
 │     │ Hypervisor│  Platform   │   Game   │                │
 │     └──────────┴─────────────┴──────────┘                 │
 └───────────────────────────────────────────────────────────┘
        │              │               │
   Proxmox/KVM   Docker/K8s/LXC   SteamCMD/LinuxGSM/…
```

Full details in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Project status

| Area                | Status                                 |
|---------------------|----------------------------------------|
| Repository layout   | Scaffolded (this commit)               |
| Core architecture   | Documented, pending ADR decisions      |
| Providers           | Not yet implemented                    |
| UI / Node editor    | Not yet implemented                    |
| Credential vault    | Specified, pending ADR-0005            |
| Local auth / RBAC   | Specified (ships in v0.1)              |
| OIDC integration    | Specified, scheduled for v0.9          |

## Contributing

Stackmaster is open-source and welcomes contributions. See
[CONTRIBUTING.md](CONTRIBUTING.md), [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md),
and [SECURITY.md](SECURITY.md). Provider proposals have their own
[template](.github/ISSUE_TEMPLATE/provider_proposal.md).

## Documentation index

- [VISION.md](docs/VISION.md) — the long story.
- [ARCHITECTURE.md](docs/ARCHITECTURE.md) — components and data flow.
- [ROADMAP.md](docs/ROADMAP.md) — v0.1 → v1.0.
- [GLOSSARY.md](docs/GLOSSARY.md) — project vocabulary.
- [STATE_MACHINE.md](docs/STATE_MACHINE.md) — game-server states.
- [PROVIDERS.md](docs/PROVIDERS.md) — provider interface.
- [API.md](docs/API.md) — REST + WebSocket sketch.
- [AUTH.md](docs/AUTH.md) — local accounts now, OIDC in v0.9, sessions, MFA outlook.
- [RBAC.md](docs/RBAC.md) — role matrix.
- [CREDENTIALS.md](docs/CREDENTIALS.md) — vault schema & encryption.
- [WORKFLOWS.md](docs/WORKFLOWS.md) — workflow engine.
- [NODE_EDITOR.md](docs/NODE_EDITOR.md) — graphical editor.
- [SECURITY_MODEL.md](docs/SECURITY_MODEL.md) — threat model.
- [COMPARISON.md](docs/COMPARISON.md) — Pterodactyl / Pelican / Agones / AMP.
- [adr/](docs/adr/) — architecture decision records.

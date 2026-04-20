# Vision

> Stackmaster is the conductor for the entire stack beneath a game server.
> It does not replace the game-server panel. It replaces the stack of
> improvisations around it.

## The problem we see

Running a game server in production is rarely just "run a binary". It
typically involves three vertical layers:

1. **A hypervisor or cloud substrate** that provides the virtual or
   physical machine (Proxmox, libvirt/KVM, ESXi, Hyper-V, bare metal,
   or a hyperscaler).
2. **A runtime platform** on top of that machine (Docker, Docker Swarm,
   Kubernetes, LXC, systemd units, a Windows host with native services).
3. **The game server process itself**, shipped as a SteamCMD build, a
   LinuxGSM config, a Pterodactyl/Pelican egg, or a custom image.

Today each layer has strong individual tooling. What does **not** exist
is a single pane of glass that understands the dependency between these
layers and that can act on all of them. Administrators bridge the gaps
with a personal mesh of Ansible playbooks, shell scripts, systemd
timers, Wake-on-LAN hacks, and tribal knowledge.

Stackmaster replaces that mesh with one coordinated control plane.

## The "stack" metaphor — used consistently

Whenever the documentation talks about:

- **Hypervisor Layer** — everything that owns the VM, LXC container, or
  physical host.
- **Platform Layer** — the runtime that owns the game-server process
  inside that host (a container runtime, a service manager, a cluster).
- **Game Layer** — the game-server binary/image itself and its data.

A **stack** is the vertical composition of one node from each layer,
wired to credentials and health probes. A **master** is the single
operator view that moves the whole stack as a unit.

See [GLOSSARY.md](GLOSSARY.md) for the full vocabulary.

## Core scenario (the "aha" moment)

1. A user opens the Stackmaster UI and clicks **Start** on the game
   server *Valheim — Saturday Raid*.
2. Stackmaster looks up the stack: the server runs in Docker on a VM
   named `lxc-gameplat-01`, hosted on a Proxmox cluster.
3. The reconciler notices the Docker daemon is not reachable.
4. The reconciler walks down the stack and notices the VM is off.
5. Stackmaster boots the VM via the Proxmox API.
6. When SSH comes up, Stackmaster starts the Docker container and
   streams a structured log to the UI ("Pulling image… Starting
   container… Probing port 2456/udp…").
7. A health check succeeds. Only now the UI flips the tile to **ready**
   and posts an audit event.

The user never sees a shell. The administrator never writes a
bespoke script for this specific game.

## Design principles

1. **Drive, don't reimplement.** Whenever an existing tool already
   does a layer well (LinuxGSM, SteamCMD, Pterodactyl eggs, Argo CD,
   Ansible), Stackmaster invokes it rather than reinventing it.
2. **Everything-as-code as a first-class equivalent to the UI.** The
   graph is a view onto YAML, not the other way around.
3. **Agent-less by default.** Remote APIs (Proxmox, Docker, Kubernetes,
   SSH, WinRM) first. An agent is a last resort and documented as such.
4. **Declarative, reconciled.** A desired state drives the system. A
   reconciler closes the gap. No fire-and-forget commands.
5. **Secrets are a primitive, not an afterthought.** Every credential
   is a typed object in an encrypted vault. Plaintext never leaves the
   vault for longer than a single operation.
6. **Two roles, not twenty.** v1.0 ships Administrator and Operator.
   Fine-grained permissions land later without breaking the data model.
7. **Auditability over chattiness.** Every state-changing action
   produces an append-only audit event. Nothing is silent.

## Who is Stackmaster for?

- **Community operators** running 1–50 game servers across a mix of
  home-lab Proxmox, a rented bare-metal box, and a small Kubernetes
  cluster.
- **Clans and tournament organizers** who want repeatable, on-demand
  servers without paying per-day panel fees.
- **Hosting providers** that want a self-hosted orchestrator as the
  internal layer beneath their customer panel (where Pterodactyl,
  Pelican, or a custom UI remains the customer-facing face).
- **Studios during playtests** that need to spin up dozens of
  identical test environments from a YAML manifest.

## Who Stackmaster is explicitly **not** for

- Matchmaking — use [Open Match](https://openmatch.dev).
- Massive per-match autoscaling for esports — use [Agones](https://agones.dev).
- Being someone's OIDC provider — Stackmaster is a relying party only.
- Billing, invoicing, or reseller hierarchies — out of scope.

## Relationship to existing projects

Stackmaster treats Pterodactyl, Pelican, LinuxGSM, SteamCMD, and Docker
Compose as **providers**, not competitors. A single Stackmaster
installation can drive a Pterodactyl daemon on one host and a raw
`docker run` on another, from the same UI, under the same audit log.

The explicit answer to "why not just fork Pterodactyl?" lives in
[COMPARISON.md](COMPARISON.md).

## Long-term north star

An operator defines the entire fleet — hosts, platforms, servers,
credentials, schedules — in one git repository. Stackmaster is the
reconciler that makes the running world match that repository, with
a UI for humans who don't want to touch YAML and a node editor for
humans who want to see the whole graph at once.

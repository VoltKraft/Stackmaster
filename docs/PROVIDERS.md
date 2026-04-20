# Providers

A **provider** is a pluggable adapter that implements exactly one
interface for exactly one layer of the stack. Providers are the only
components allowed to talk to external systems. Everything else in
Stackmaster talks to providers, never directly to Proxmox, Docker,
SteamCMD, etc.

## Principles

1. **One interface per layer.** Hypervisor, Platform, Game — three
   interfaces, disjoint verbs.
2. **Stateless between calls.** All state lives in the core database.
   A provider instance can be destroyed and recreated between calls.
3. **Idempotent verbs.** `ensureUp`, `ensureReady`, `start`, `stop`
   must all be safe to call on an already-converged target.
4. **Explicit failure shape.** Providers return typed errors; the
   reconciler decides whether to retry or escalate.
5. **No credential persistence.** A provider receives decrypted
   credentials only as call arguments and zeroizes them after use.
6. **Version negotiation.** A provider declares the interface version
   it implements. The core rejects unknown versions at load.

## Hypervisor Provider

Responsibility: own a machine's power and identity.

Verbs (draft — to be finalized in v0.1):

| Verb               | Purpose                                               |
|--------------------|-------------------------------------------------------|
| `status(host)`     | Return current power state and reachability facts.    |
| `ensureUp(host)`   | Power on, boot, wait for signal of life.              |
| `ensureDown(host)` | Graceful shutdown with fallback to hard stop.         |
| `snapshot(host)`   | Create a snapshot/checkpoint (optional capability).   |
| `clone(host)`      | Clone into a new host (optional capability).          |
| `capabilities()`   | Static declaration of supported verbs.                |

Implementations planned:

- [`proxmox/`](../providers/hypervisor/proxmox/) — Proxmox VE / PVE API.
- [`libvirt/`](../providers/hypervisor/libvirt/) — libvirt/KVM.
- [`esxi/`](../providers/hypervisor/esxi/) — VMware ESXi (SOAP/REST).
- [`hyperv/`](../providers/hypervisor/hyperv/) — Windows Hyper-V.
- [`noop/`](../providers/hypervisor/noop/) — always-on bare-metal
  sentinel.

## Platform Provider

Responsibility: own the runtime that will execute the game process.

Verbs:

| Verb                       | Purpose                                             |
|----------------------------|-----------------------------------------------------|
| `ensureReady(platform)`    | Daemon up, network reachable, prerequisites met.    |
| `deployUnit(unit)`         | Materialize the unit (container, pod, service).     |
| `startUnit(unit)` / `stop` | Run / halt the unit.                                |
| `statusUnit(unit)`         | Report lifecycle facts.                             |
| `streamLogs(unit)`         | WebSocket-friendly log stream.                      |
| `exec(unit, cmd)`          | Run a one-shot command inside the unit.             |
| `writeFile / readFile`     | Support for the browser file editor.                |
| `capabilities()`           | Declare supported verbs.                            |

Implementations planned:

- [`docker/`](../providers/platform/docker/)
- [`kubernetes/`](../providers/platform/kubernetes/)
- [`lxc/`](../providers/platform/lxc/)
- [`systemd-linux/`](../providers/platform/systemd-linux/)
- [`windows-native/`](../providers/platform/windows-native/)

Docker Swarm is a mode of the Docker provider, not a separate provider.

## Game Provider

Responsibility: own the game-server process, its data directory, and
its health probe.

Verbs:

| Verb                  | Purpose                                                     |
|-----------------------|-------------------------------------------------------------|
| `install(server)`     | Fetch assets (SteamCMD, LinuxGSM install, image pull).      |
| `configure(server)`   | Apply configuration files, env, ports.                      |
| `start` / `stop`      | Start / stop the game process.                              |
| `backup` / `restore`  | Snapshot game state; restore from a named backup.           |
| `healthCheck(server)` | Is the server reachable and accepting players?              |
| `consoleAttach(server)` | Stdin/stdout stream for live console.                     |
| `capabilities()`      | Declare supported verbs.                                    |

Implementations planned:

- [`steamcmd/`](../providers/game/steamcmd/) — any SteamCMD-driven title.
- [`linuxgsm/`](../providers/game/linuxgsm/) — LinuxGSM wrappers.
- [`custom-image/`](../providers/game/custom-image/) — run a named
  Docker image with declared ports, env, volumes.
- [`script/`](../providers/game/script/) — escape hatch: arbitrary
  install/start scripts on a host. Requires audit annotations and a
  stricter RBAC gate.

## Relationship to Pterodactyl and Pelican

Pterodactyl and Pelican are not providers themselves — they are
**stacks of providers**: they bundle platform and game logic. The plan
for integration:

- A **Pterodactyl game provider** (post-v1.0) that delegates game-layer
  verbs to an existing Pterodactyl daemon. Stackmaster still owns the
  hypervisor and platform layers above it.
- This is not a v0.1 goal. It is explicitly listed so operators who
  have a Pterodactyl install today know their path.

## Registration and discovery

Providers register themselves with the core at startup by name and
declared interface version. Built-in providers ship inside the core
binary. Out-of-tree providers are planned post-v1.0 (see the
[Roadmap](ROADMAP.md)).

## Testing requirements per provider

Every provider must ship with:

- A **conformance test suite** that exercises every declared verb
  against a recorded fixture. The core publishes the suite; a provider
  passes or it is not shipped.
- A **live smoke test** that runs against a real backend in CI when
  credentials are available (otherwise skipped, not failed).
- A **capability matrix** in its README listing which optional verbs
  it supports (e.g. Hyper-V `snapshot` yes, ESXi `clone` yes, noop
  `snapshot` no).

## TODO

- [ ] Formalize the error taxonomy (retryable, fatal, needs-attention).
- [ ] Specify the credential-resolution contract shared across layers.
- [ ] Decide how provider versions interact with core versions.
- [ ] Define the on-disk layout for bundled built-in providers.

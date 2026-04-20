# Glossary

This glossary fixes the vocabulary used across all Stackmaster
documentation, UI labels, API identifiers, and commit messages. If a
term is used in the codebase or docs, it should be defined here, and
the definition here is authoritative.

## The "stack" metaphor

Stackmaster calls its central abstraction a **stack**. A stack is a
vertical composition of three layers, with exactly one active node in
each layer:

- **Hypervisor Layer** — owns the physical or virtual machine. Examples:
  a Proxmox VM, an ESXi VM, a libvirt domain, a Hyper-V guest, or the
  sentinel `noop` node for an always-on bare-metal host.
- **Platform Layer** — owns the runtime that will execute the game
  process. Examples: a Docker daemon, a Kubernetes namespace, an LXC
  container, a systemd target on a Linux host, a Windows service host.
- **Game Layer** — owns the game-server process, its data directory,
  and its health probe. Examples: a LinuxGSM install, a SteamCMD
  deployment, a Pterodactyl/Pelican egg, a custom image.

A stack is the unit that the reconciler converges and that the UI shows
as a single tile.

## The "master" metaphor

**Master** carries two meanings, and both are load-bearing:

1. **Orchestrator role.** Stackmaster is the master that drives other
   tools. It is not the daemon that runs a game, it is the conductor
   that tells the daemon what to do and makes sure the hosts beneath
   it are alive.
2. **Game-master metaphor.** In role-playing and tournament culture,
   the game master coordinates the session — starting encounters,
   pausing them, adjudicating rules. Stackmaster plays that role for a
   fleet of game servers.

The metaphor should stay consistent: we do not refer to Stackmaster as
"controller", "hub", "dashboard", or "panel" in user-facing copy,
because each of those has a different established meaning in
neighboring projects.

## Core project terms

### Provider
A pluggable adapter that implements one well-defined interface for
exactly one layer of the stack. Example: the `proxmox` hypervisor
provider. Providers are stateless between calls; all state lives in
the core database. See [PROVIDERS.md](PROVIDERS.md).

### Reconciler
The component that continuously compares **desired state** to
**observed state** and enqueues tasks to close the gap. Modeled on
Kubernetes controllers. See [ARCHITECTURE.md](ARCHITECTURE.md).

### Workflow
A directed acyclic graph (DAG) of typed nodes that describes an
automated flow. A workflow answers "what should happen on event X?"
and is persisted, versioned, and resumable. Every workflow has a
YAML representation; the graphical editor is a view onto that YAML.

### Node
Two distinct meanings depending on context; context always makes it
unambiguous:

- **Workflow node** — one box in a workflow DAG. Typed: Trigger,
  Condition, Action, Notification, Delay, Parallel, ForEach, Utility.
- **Topology node** — one element of the infrastructure graph
  (hypervisor, platform, game server, credential reference).

### Trigger
A workflow node type that starts a workflow run. Sources: cron
schedule, webhook, manual UI action, or an internal event such as
"server has been idle for more than N minutes".

### Desired State
The declarative description of what the operator wants to be true.
Example: `valheim.desired = Running`. Only the API mutates desired
state, and only after authorization.

### Observed State
The last-known actual state, as reported by provider calls. Example:
`valheim.observed = Stopped(since=2026-04-20T14:00Z)`. Updated by
workers.

### Credential
A typed secret stored in the encrypted vault. Types include API token,
X.509 certificate and key, SSH key, username/password, webhook
secret, OAuth client secret. See [CREDENTIALS.md](CREDENTIALS.md).

### Credential Reference
A stable identifier (alias or UUID) that points at a credential
without carrying its plaintext. This is what appears in manifests,
workflows, and editor nodes. Example: `ref://creds/proxmox-main`.

### Stack (already defined above)
The vertical triple of hypervisor + platform + game nodes plus the
credential references and probes that tie them together.

### Task
One atomic unit of work enqueued for a worker: a single provider call
with a timeout, retry policy, and audit envelope. Tasks are
idempotent.

### Worker
A process that dequeues tasks, resolves credentials, invokes a
provider, and writes the result back to observed state.

### Audit Event
An append-only record of a security-relevant action (login, credential
access, workflow execution, game-server action, role change). Audit
events never contain plaintext credentials.

### Administrator
v1.0 role. Defines infrastructure (hypervisors, platforms, credentials,
game-server templates, workflows, users, roles). Full access. See
[RBAC.md](RBAC.md).

### Operator
v1.0 role. Starts, stops, restarts, and reconfigures assigned game
servers; reads console; edits configuration files; manages backups.
Has no access to credentials, new-platform creation, or workflow
edits.

### Everything-as-Code / Manifest
A YAML document that describes infrastructure or workflow. `stackmaster
apply -f infrastructure.yaml` is a first-class path equivalent to the
UI. The graph in the node editor and the YAML are isomorphic.

### Health Check / Probe
A provider-run check that decides whether a game server is actually
**reachable and usable**, not merely "the process started". A stack
transitions to `Running` only after a probe passes.

### Envelope Encryption
The encryption scheme used by the vault: each credential is encrypted
with a per-record data key, which is itself encrypted with a master
key sourced from an env var, KMS file, or external KMS. Rotation
re-wraps data keys without re-encrypting every credential. See
[CREDENTIALS.md](CREDENTIALS.md).

### OIDC Relying Party
Stackmaster's planned authentication role for v0.9. Stackmaster will
**consume** OIDC tokens from an external IdP; it is never an
**issuer**. Through v0.8, authentication uses local accounts only.
See [AUTH.md](AUTH.md) and [ROADMAP.md](ROADMAP.md).

## Terms we deliberately avoid

- "Panel" — too close to Pterodactyl / Pelican.
- "Dashboard" — implies read-only; Stackmaster writes.
- "Agent" — overloaded; we only use "agent" for the optional remote
  binary on hosts that cannot expose a remote API.
- "Node" without a qualifier — always say "workflow node" or "topology
  node".
- "Module" for providers — use **provider**.

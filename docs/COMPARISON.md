# Comparison

Stackmaster does not exist in a vacuum. This document is a fair, honest
side-by-side with the projects operators most often name when they
first hear the pitch, and an explicit answer to the most frequent
question: **"Why not just fork Pterodactyl?"**

## Side-by-side

| Capability                          | Pterodactyl | Pelican | Agones       | AMP        | **Stackmaster**                   |
|-------------------------------------|-------------|---------|--------------|------------|-----------------------------------|
| Primary audience                    | Game hosts  | Game hosts (Ptero fork) | Studios, esports | Prosumers | Operators of the full stack  |
| License                             | MIT         | AGPL-3.0 | Apache-2.0 | Proprietary | AGPL-3.0 (planned, ADR-0004)      |
| Runs game daemons itself            | ✅          | ✅       | ✅ (on K8s)   | ✅          | ❌ (drives existing runtimes)     |
| Manages hypervisors                 | ❌          | ❌       | ❌            | Limited    | ✅                                |
| Manages platforms (Docker/K8s/LXC)  | Docker only | Docker only | K8s only  | Host only  | ✅ Docker / Swarm / K8s / LXC / systemd / Windows |
| Graphical workflow designer         | ❌          | ❌       | ❌            | Limited    | ✅ (typed DAG with YAML round-trip) |
| Credential vault (field-level)      | ❌          | ❌       | Kubernetes-native | ❌     | ✅                                |
| OIDC SSO                            | Plugin      | Plugin  | External      | Limited    | ✅ First-class                    |
| RBAC                                | Per server  | Per server | K8s RBAC    | Roles      | ✅ Admin/Operator, expandable     |
| Everything-as-code (YAML apply)     | ❌          | ❌       | ✅ (K8s manifests) | ❌     | ✅                                |
| Self-hosted by default              | ✅          | ✅       | ✅            | ✅          | ✅                                |
| Autoscaling for esports             | ❌          | ❌       | ✅            | ❌          | ❌ (defer to Agones)              |
| Matchmaking                         | ❌          | ❌       | Via Open Match | ❌         | ❌ (defer to Open Match)          |

## Where each project shines — and where Stackmaster does not compete

### Pterodactyl

**Great at:** running a multi-tenant game-host business with per-server
egg/image management, file managers, subuser permissions, and a
mature admin UI. Ten years of community eggs.

**Not designed for:** owning the VM or hypervisor layer; driving
Kubernetes; single-operator home-lab scenarios where "start the host,
then start the game" is the actual workflow.

**Stackmaster relationship:** Pterodactyl is a valid **game provider**
target for Stackmaster. Post-v1.0, a Pterodactyl adapter lets
Stackmaster orchestrate the layers around a Pterodactyl daemon. We
are not replacing Pterodactyl.

### Pelican

**Great at:** being a modernized, AGPL-licensed continuation of
Pterodactyl's approach, with active reworks of the admin experience.

**Not designed for:** hypervisor control or multi-platform
orchestration.

**Stackmaster relationship:** same as Pterodactyl. A future Pelican
provider is on the table.

### Agones

**Great at:** running **thousands** of game-server pods on Kubernetes
with matchmaking, allocation, and autoscaling — the esports use case.

**Not designed for:** a home-lab with one Proxmox box and two hosts;
day-to-day "start Valheim for Saturday night" operations without a
Kubernetes cluster in front of it.

**Stackmaster relationship:** non-overlapping. Operators who outgrow
Stackmaster's orchestration model should move to Agones. We may ship
an Agones adapter later for the hybrid path.

### AMP (Cube Coders)

**Great at:** single-node prosumer experience with a strong per-game
config UI; commercial support.

**Not designed for:** self-hosted, open-source, multi-platform, or
full-stack (hypervisor + platform + game) automation.

**Stackmaster relationship:** non-overlapping audience and licensing.

## Why not just fork Pterodactyl?

This is asked often and deserves a direct answer.

1. **Different scope.** Pterodactyl is a game-server panel. Its data
   model does not describe hypervisors, VMs, or platforms as
   first-class entities. Adding the hypervisor and platform layers
   would require a data-model rewrite — at which point we are no
   longer forking, we are writing a new system that happens to
   inherit a logo.
2. **Different audience and metaphor.** Pterodactyl optimizes for the
   multi-tenant hoster with subuser permissions. Stackmaster
   optimizes for the **stack operator** — often a single admin or a
   small team — who sees one graph, not a per-client admin tree.
3. **Different deployment assumption.** Pterodactyl expects a wings
   daemon per host. Stackmaster is **agent-less by default** and drives
   APIs (Proxmox, Docker, K8s, SSH, WinRM). The agent is an escape
   hatch, not the default shape.
4. **Different execution model.** Stackmaster's reconciler and
   workflow engine are central to the design. Bolting a Kubernetes-
   controller-style reconciler onto Pterodactyl would fight its
   event-driven PHP queue rather than cooperate with it.
5. **Different license intent.** Stackmaster ships AGPL (ADR-0004
   pending), which is compatible with a control-plane-as-a-service
   context. Pterodactyl is MIT; relicensing a fork is not a serious
   path.
6. **Different UI primitive.** Stackmaster's central UI primitive is
   a node editor over a topology and workflow DAG. Pterodactyl's is
   a classic CRUD admin. These are different products even if they
   happened to run the same game binary underneath.
7. **Cooperation is better than competition.** A Pterodactyl (and
   Pelican) game provider lets existing operators keep their daemons,
   eggs, and muscle memory while gaining the hypervisor and workflow
   layer Stackmaster offers. Forking burns that bridge.

**Summary:** Stackmaster is a control plane for a full stack that
*can include* Pterodactyl, not a replacement for it.

## When to pick what

| Situation                                                                 | Pick                |
|---------------------------------------------------------------------------|---------------------|
| I run a hosting business with customer logins, invoices, subusers.        | Pterodactyl / Pelican + (later) Stackmaster beneath for infra. |
| I need 10 000 per-match pods with matchmaking.                             | Agones + Open Match. |
| I run one to fifty servers across Proxmox/bare-metal/K8s and want one UI. | Stackmaster.        |
| I have one home-lab box and do not care about the rest of the stack.      | LinuxGSM or Pelican on bare metal. |
| I want a commercial, supported, single-node panel.                        | AMP.                |

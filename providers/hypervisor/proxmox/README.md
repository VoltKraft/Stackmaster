# providers/hypervisor/proxmox

Hypervisor provider for Proxmox VE (PVE) clusters.

**Status:** not yet implemented. Target: **v0.1**.

- Verbs: `status`, `ensureUp`, `ensureDown`, `snapshot`, `clone`,
  `capabilities`.
- Credentials: API token (preferred), username/password (fallback).
- Transport: Proxmox REST API over HTTPS.

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) for the
interface and [../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md)
for the overall picture.

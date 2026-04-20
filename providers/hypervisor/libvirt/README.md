# providers/hypervisor/libvirt

Hypervisor provider for libvirt / KVM hosts.

**Status:** not yet implemented. Target: **v0.2**.

- Verbs: `status`, `ensureUp`, `ensureDown`, `snapshot` (optional),
  `clone` (optional), `capabilities`.
- Credentials: SSH key + libvirt URI, or TLS connection to libvirtd.
- Transport: libvirt API (over TLS or SSH).

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

# providers/hypervisor/noop

Sentinel hypervisor provider for "always-on bare-metal" hosts. The
noop provider reports `up` unconditionally and performs no side
effects. It is the default hypervisor for a host that Stackmaster
does not manage at the hardware level.

**Status:** not yet implemented. Target: **v0.1**.

- Verbs: `status`, `ensureUp`, `ensureDown`, `capabilities`.
- Credentials: none.

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

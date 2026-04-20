# providers/platform/lxc

Platform provider for LXC / LXD containers.

**Status:** not yet implemented. Target: **v0.2**.

- Verbs: `ensureReady`, `deployUnit`, `startUnit`, `stopUnit`,
  `statusUnit`, `streamLogs`, `exec`, `readFile`, `writeFile`,
  `capabilities`.
- Credentials: LXD client certificate or SSH to the host.
- Transport: LXD REST API.

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

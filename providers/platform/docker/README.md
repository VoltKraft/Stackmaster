# providers/platform/docker

Platform provider for Docker (including Docker Swarm mode).

**Status:** not yet implemented. Target: **v0.1**.

- Verbs: `ensureReady`, `deployUnit`, `startUnit`, `stopUnit`,
  `statusUnit`, `streamLogs`, `exec`, `readFile`, `writeFile`,
  `capabilities`.
- Credentials: Docker API TLS cert+key, or SSH to the host plus
  Docker socket.
- Transport: Docker Engine API (TCP/TLS or Unix socket via SSH).

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

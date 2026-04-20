# providers/platform/systemd-linux

Platform provider for native Linux hosts where the game runs as a
systemd service.

**Status:** not yet implemented. Target: **v0.3**.

- Verbs: `ensureReady`, `deployUnit` (write + enable unit file),
  `startUnit`, `stopUnit`, `statusUnit`, `streamLogs` (via journald),
  `exec`, `readFile`, `writeFile`, `capabilities`.
- Credentials: SSH key.
- Transport: SSH + `systemctl` + `journalctl`.

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

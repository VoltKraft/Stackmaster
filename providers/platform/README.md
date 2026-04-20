# providers/platform/

PlatformProvider implementations. A platform provider owns the
runtime that will execute the game process (container runtime,
service manager, cluster).

**Status:** not yet implemented.

Planned implementations: `docker`, `kubernetes`, `lxc`, `systemd-linux`,
`windows-native`.

See [../../docs/PROVIDERS.md](../../docs/PROVIDERS.md) and
[../../docs/ARCHITECTURE.md](../../docs/ARCHITECTURE.md).

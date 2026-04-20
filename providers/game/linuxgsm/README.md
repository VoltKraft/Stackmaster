# providers/game/linuxgsm

Game provider that delegates to LinuxGSM installers and server
wrappers. A broad catalog of supported games with minimal per-game
work on our side.

**Status:** not yet implemented. Target: **v0.1**.

- Verbs: `install`, `configure`, `start`, `stop`, `backup`, `restore`,
  `healthCheck`, `consoleAttach`, `capabilities`.
- Credentials: none at the LinuxGSM layer; SteamCMD credentials flow
  through as needed.
- Transport: invokes `linuxgsm.sh` inside the target platform unit.

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

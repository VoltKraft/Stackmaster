# providers/game/steamcmd

Game provider for any SteamCMD-driven dedicated server (CS:GO / CS2,
TF2, ARK, Valheim via app id, 7 Days to Die, and many others).

**Status:** not yet implemented. Target: **v0.1** (behind LinuxGSM).

- Verbs: `install`, `configure`, `start`, `stop`, `backup`, `restore`,
  `healthCheck`, `consoleAttach`, `capabilities`.
- Credentials: Steam anonymous (default) or Steam account credentials
  for games that require them.
- Transport: invokes `steamcmd` inside the target platform unit.

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

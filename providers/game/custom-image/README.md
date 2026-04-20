# providers/game/custom-image

Game provider that runs a named container image with declared ports,
volumes, environment, and probes. Covers the "there's a community
image for this game" case without per-game logic.

**Status:** not yet implemented. Target: **v0.2**.

- Verbs: `install` (image pull), `configure`, `start`, `stop`,
  `backup` (volume snapshot), `restore`, `healthCheck`,
  `consoleAttach` (via platform `exec`), `capabilities`.
- Credentials: optional registry credentials.
- Transport: delegates to the platform provider's `deployUnit` /
  `startUnit`.

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

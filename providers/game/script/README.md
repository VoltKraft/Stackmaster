# providers/game/script

Escape-hatch game provider. Administrators supply install, start,
stop, backup, and probe scripts. Stackmaster invokes them with a
documented contract (working dir, env, exit codes).

**Status:** not yet implemented. Target: **v0.3**.

- Verbs: `install`, `configure`, `start`, `stop`, `backup`, `restore`,
  `healthCheck`, `capabilities`.
- Credentials: passed via env to the scripts, scoped to the call.
- Audit: every script invocation is audited with the script's SHA-256
  at call time.

**Security note.** This provider runs arbitrary administrator-supplied
code and is gated by a stricter RBAC check (Administrator only in
v1.0) and explicit per-stack opt-in.

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md),
[../../../docs/RBAC.md](../../../docs/RBAC.md), and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

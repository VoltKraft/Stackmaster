# deploy/

The default deployment path for Stackmaster: `docker compose`.

**Status:** skeleton only. Nothing in this folder boots a working
Stackmaster yet. See the top-level [README](../README.md) and
[ROADMAP](../docs/ROADMAP.md) for where this stands.

## Files

- `compose.yaml` — service topology (everything commented out).
  Follows the modern Compose Spec file name; Docker Compose v2
  auto-discovers it.
- `compose.override.example.yaml` — copy to `compose.override.yaml`
  for local overrides.
- `.env.example` — copy to `.env` and fill in real values.

## Intended flow (once v0.1 ships)

```
cp deploy/.env.example deploy/.env
# edit deploy/.env — set SM_MASTER_KEY, POSTGRES_PASSWORD,
# SM_BOOTSTRAP_ADMIN_EMAIL, SM_BOOTSTRAP_ADMIN_PASSWORD
docker compose -f deploy/compose.yaml up -d
# visit the public URL, sign in with the bootstrap admin, change the
# password on first login, then:
stackmaster apply -f examples/manifests/valheim-linuxgsm-bare-metal.yaml
```

OIDC / external IdP integration is deliberately deferred to v0.9
(see [../docs/ROADMAP.md](../docs/ROADMAP.md)). Until then,
Stackmaster authenticates users via local accounts only.

See [../docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md) §8.

# deploy/

The default deployment path for Stackmaster: `docker compose`.

**Status:** skeleton only. Nothing in this folder boots a working
Stackmaster yet. See the top-level [README](../README.md) and
[ROADMAP](../docs/ROADMAP.md) for where this stands.

## Files

- `docker-compose.yml` — service topology (everything commented out).
- `docker-compose.override.example.yml` — copy to
  `docker-compose.override.yml` for local overrides.
- `.env.example` — copy to `.env` and fill in real values.

## Intended flow (once v0.1 ships)

```
cp deploy/.env.example deploy/.env
# edit deploy/.env — set SM_MASTER_KEY, POSTGRES_PASSWORD, OIDC_*
docker compose -f deploy/docker-compose.yml up -d
# visit the public URL, sign in via OIDC (or bootstrap token for local)
stackmaster apply -f examples/manifests/valheim-linuxgsm-bare-metal.yaml
```

See [../docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md) §8.

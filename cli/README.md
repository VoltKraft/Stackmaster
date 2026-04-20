# cli/

The `stackmaster` command-line client. Same REST API as the UI, same
auth, same audit.

**Status:** not yet implemented. The language follows
[ADR-0001](../docs/adr/0001-backend-language.md) (likely the same
binary as the core, or a thin companion).

## Command shape (target)

```
stackmaster apply -f infrastructure.yaml
stackmaster server start valheim
stackmaster server stop valheim
stackmaster server status valheim
stackmaster workflow run wake-on-demand
stackmaster workflow list
stackmaster credential set proxmox-main --type api_token --stdin
stackmaster audit tail --server valheim
stackmaster login   # local login or PAT through v0.8;
                    # OIDC device-code flow added in v0.9.
```

See [../docs/API.md](../docs/API.md) and
[../docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md).

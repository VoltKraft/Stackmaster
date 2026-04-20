# workflows/builtin/

Built-in workflow **templates**. Each file is a cloneable starting
point, not a hard-coded behavior. Operators copy a template into
their own workspace and edit it.

**Status:** not yet implemented — files in this folder are
placeholders with the intended YAML shape.

- `start-with-platform-boot.example.yaml` — on start, boot the host
  if needed, then start the server.
- `scheduled-shutdown.example.yaml` — shut a server (and optionally
  its host) down on a cron schedule.
- `auto-update-window.example.yaml` — run game updates inside a
  defined maintenance window.

See [../../docs/WORKFLOWS.md](../../docs/WORKFLOWS.md).

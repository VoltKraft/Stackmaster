# Role-Based Access Control

v1.0 ships **two** roles, clearly separated, and a forward-compatible
permission model that later introduces fine-grained roles without
breaking the data model.

## Roles in v1.0

### Administrator

Defines the world. Everything a privileged operator does in a home-lab
or a single-tenant deployment.

- Register and edit hypervisor connections.
- Register and edit platforms.
- Create and edit game-server templates and instances.
- Create, edit, and rotate credentials (via references; plaintext is
  still never shown again after creation).
- Design, edit, and run workflows.
- Manage users, roles, and role mappings.
- Read the full audit log.

### Operator

Runs the world. The trusted-but-not-privileged role for day-to-day
operations on assigned game servers.

- Start, stop, restart assigned game servers.
- Execute console commands on assigned servers.
- Edit configuration files inside the server's file system.
- Create and restore backups for assigned servers.
- Read audit events scoped to assigned servers.
- **Cannot** see hypervisor credentials, create platforms, or edit
  workflows.

Assignment is expressed per game server (or per server group): an
Operator has access to exactly the servers assigned to them.

## Role matrix

Rows are resources; columns are roles; cells list permitted verbs.

`—` = no access; `read` means list + get; write verbs are named
explicitly.

| Resource                     | Administrator                                                   | Operator                                                      |
|------------------------------|-----------------------------------------------------------------|---------------------------------------------------------------|
| `hypervisor.connection`      | `read, create, update, delete, test`                            | —                                                             |
| `platform`                   | `read, create, update, delete`                                  | `read` (assigned hosts only)                                  |
| `credential`                 | `read (metadata only), create, update, rotate, delete`          | —                                                             |
| `workflow`                   | `read, create, update, delete, run, dry-run`                    | `run` (workflows explicitly tagged `operator-invocable`)      |
| `gameserver`                 | `read, create, update, delete, start, stop, restart, reconfigure` | `read, start, stop, restart, reconfigure (assigned only)`   |
| `gameserver.console`         | `attach, exec`                                                  | `attach, exec (assigned only)`                                |
| `gameserver.files`           | `read, write, delete`                                           | `read, write, delete (assigned only, within server scope)`    |
| `backup`                     | `read, create, restore, delete`                                 | `read, create, restore (assigned only)`                       |
| `audit`                      | `read (all)`                                                    | `read (scoped to assigned servers)`                           |
| `user`                       | `read, create, update, delete`                                  | `read (self only)`                                            |
| `role`                       | `read, assign, revoke`                                          | —                                                             |
| `settings`                   | `read, update`                                                  | `read (non-sensitive subset)`                                 |

## Enforcement rules

1. **Authorization is evaluated at the API layer.** The UI is an
   additional filter for usability; it is **never** the only guard.
2. **Authorization is evaluated per resource instance**, not only per
   resource type. An Operator's access to `gameserver:abc` is
   independent from `gameserver:xyz`.
3. **Credential-read is never granted by role, even to Administrators.**
   Reading a credential resolves a reference; the vault returns
   plaintext only to a worker during a live operation.
4. **Role changes trigger session rotation.** A user who is demoted
   loses their session and must re-authenticate.

## Forward compatibility

The data model stores permissions as `(resource, verb)` tuples grouped
into roles. v1.0 exposes only the two roles above, but the schema
already supports:

- Custom roles (name + permission list).
- Per-resource grants (e.g. "this specific workflow can be run by
  this specific user").
- Role inheritance.

These capabilities are **hidden behind a feature flag** in v1.0 to
avoid shipping a half-designed UI. See the
[Roadmap](ROADMAP.md) for when they become user-facing.

## Role assignment

Through v0.8, Stackmaster uses local accounts only. An Administrator
assigns each user to Administrator or Operator directly.

From **v0.9** onward (when OIDC integration ships — see
[ROADMAP.md](ROADMAP.md)), group claims from the IdP become the
authoritative source of role membership for SSO-authenticated users.
See [AUTH.md](AUTH.md) for the claim-to-role mapping config. Local
accounts keep their explicit role assignment as a permanent fallback.

## TODO

- [ ] Define the exact grammar for "assigned to an Operator" (direct
      assignment vs. tag-based).
- [ ] Decide whether `workflow.run` for Operators is gated by a tag on
      the workflow (`operator-invocable`) or by an explicit grant.
- [ ] Specify how a deletion of a user that owns resources is handled
      (transfer vs. orphan vs. deny).

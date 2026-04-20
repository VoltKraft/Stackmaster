# Architecture Decision Records

ADRs capture **significant** decisions that shape Stackmaster. One
decision per file. New ADRs copy [template.md](template.md).

## Status legend

- **Proposed** — the options are named, pros/cons are listed, the
  decision is not made yet.
- **Accepted** — the decision is made; the implementation follows it.
- **Deprecated** — the decision has been replaced; see the successor.
- **Superseded** — same as Deprecated; include a link to the ADR that
  replaces this one.

## Index

### Foundation

| ID    | Title                                               | Status    | Decision                                                 |
|-------|-----------------------------------------------------|-----------|----------------------------------------------------------|
| 0001  | [Backend language](0001-backend-language.md)        | Accepted  | Go                                                       |
| 0002  | [Task queue / workflow runtime](0002-task-queue.md) | Accepted  | asynq (reconciler) + custom DAG on Postgres (workflows)  |
| 0003  | [Frontend framework](0003-frontend-framework.md)    | Accepted  | React + Vite (SPA)                                       |
| 0004  | [License](0004-license.md)                          | Accepted  | AGPL-3.0-or-later                                        |

### Security and UX

| ID    | Title                                                | Status    | Decision                                                                |
|-------|------------------------------------------------------|-----------|-------------------------------------------------------------------------|
| 0005  | [Credential vault backend](0005-credential-vault.md) | Accepted  | Internal PostgreSQL vault (default)                                     |
| 0006  | [Auth stack and OIDC](0006-auth-and-oidc.md)         | Accepted  | Local accounts + env-var bootstrap ship in v0.1 · native OIDC deferred to v0.9 |
| 0007  | [Node-editor library](0007-node-editor-library.md)   | Accepted  | React Flow (open-core, MIT)                                             |
| 0008  | [Workflow engine](0008-workflow-engine.md)           | Accepted  | Custom DAG on PostgreSQL                                                |

## Authoring rules

- Write one file per decision.
- Use the next free 4-digit ID.
- Leave the status at **Proposed** until there is consensus.
- Record *why*, not only *what*. Future contributors need the context.
- Do not rewrite history. If a decision changes, create a new ADR that
  supersedes the old one and link both ways.

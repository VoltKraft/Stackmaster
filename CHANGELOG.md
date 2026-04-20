# Changelog

All notable changes to Stackmaster are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/)
once it reaches v1.0.

## [Unreleased]

### Added

- Initial repository scaffolding.
- Documentation set: VISION, ARCHITECTURE, GLOSSARY, ROADMAP,
  PROVIDERS, STATE_MACHINE, API, SECURITY_MODEL, COMPARISON, AUTH,
  RBAC, CREDENTIALS, WORKFLOWS, NODE_EDITOR.
- [ITERATION_2.md](docs/ITERATION_2.md) — implementation plan with
  seven milestones (M2.1–M2.7), library pin matrix, risks, and
  Iteration 3 outlook.
- Architecture decision records 0001–0008.
- Placeholder directory layout for providers, core, UI, CLI, deploy,
  workflows, examples, and GitHub templates.
- Draft `docker-compose.yml` structure with all services commented.
- Draft CLI naming convention (`stackmaster <verb>`).
- **Real-time UI sync** as a first-class architectural principle
  ([ARCHITECTURE.md §10](docs/ARCHITECTURE.md)): every state change
  propagates to all logged-in users via WebSocket; no page reload.

### Decided

All eight foundation ADRs are now **Accepted**:

- **ADR-0001** — Backend language: **Go**.
- **ADR-0002** — Task queue / workflow runtime:
  **asynq** for reconciler tasks, **custom DAG on PostgreSQL** for
  workflows.
- **ADR-0003** — Frontend framework: **React + Vite (SPA)**.
- **ADR-0004** — License: **AGPL-3.0-or-later**.
- **ADR-0005** — Credential vault: **internal PostgreSQL vault** as
  default; Vault / SOPS / cloud KMS remain on the roadmap as
  delegated backends.
- **ADR-0006** — Auth: **native OIDC** via `go-oidc`; **Authentik**
  is the primary conformance target (external, not bundled);
  **env-var bootstrap** admin with forced first-login password
  change.
- **ADR-0007** — Node-editor library: **React Flow** (open-core, MIT).
- **ADR-0008** — Workflow engine: **custom DAG on PostgreSQL**.

### Still open

No foundation ADRs. The next decisions live inside accepted ADRs
(AEAD choice for ADR-0005, expression language for ADR-0008, etc.).

### Not yet implemented

- All application code.
- Any provider.
- UI and node editor.
- Credential vault backend.
- Workflow engine.

See [docs/ROADMAP.md](docs/ROADMAP.md) for the path to v0.1.

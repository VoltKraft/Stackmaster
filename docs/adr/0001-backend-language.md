# ADR-0001: Backend language

- **Status:** Accepted
- **Date:** 2026-04-20
- **Deciders:** core team
- **Tags:** backend, foundation

## Context and problem

Stackmaster's core binary runs the API, reconciler, workflow engine,
task workers, and provider runtime. It must:

- Integrate cleanly with Proxmox, Docker, Kubernetes, SSH, WinRM, and
  cloud APIs.
- Handle many long-lived WebSocket streams (console, logs).
- Ship as a single, operator-friendly binary for `docker compose`.
- Be approachable to community contributors who write provider
  adapters.

The two realistic options are **Go** and **Python**.

## Options considered

### Option A — Go

- **Summary:** compiled, statically linked, strong concurrency story,
  widely used in cloud-native tooling.
- **Pros:**
  - Single binary, tiny image, trivial `docker compose` shipping.
  - First-class SDKs for Kubernetes, Docker, cloud providers.
  - Goroutines + channels map naturally to reconciler + workers.
  - Strong static typing; good refactoring ergonomics.
  - Large body of prior art for controller loops (controller-runtime).
- **Cons:**
  - More boilerplate for generic data handling (vs. Python).
  - Smaller candidate pool among community contributors who already
    know game-server/ops scripting (often Python/PHP shops).
  - Provider SDKs for some niche targets are Python-first.
- **Effort:** medium.
- **Reversibility:** hard once provider code lands.

### Option B — Python

- **Summary:** interpreted, huge library ecosystem, familiar to the
  homelab / sysadmin audience.
- **Pros:**
  - Many operators can already read and patch it.
  - Rich ecosystem (proxmoxer, docker-py, kubernetes-asyncio, paramiko,
    pywinrm, authlib, SQLAlchemy).
  - FastAPI + asyncio make long-lived WS streams tractable.
  - Fast iteration in the pre-1.0 design phase.
- **Cons:**
  - Packaging + shipping (a single binary is not native; PyInstaller
    and friends are a workaround).
  - Higher memory footprint per worker.
  - Global-state / GIL nuances under heavy concurrency.
  - Runtime type errors later than in Go; mypy helps but is optional.
- **Effort:** medium.
- **Reversibility:** hard once provider code lands.

### Option C — Rust

- Rejected early. Excellent fit technically, but raises the
  contribution bar to a level that conflicts with the open-source
  audience we want.

## Decision

**Go.** The core, worker, and CLI ship as a single statically linked
binary. The driving reasons:

- Single-binary distribution matches the `docker compose up -d`
  operator story.
- First-class SDKs for every planned hypervisor and platform target
  (Kubernetes, Docker, cloud APIs).
- Goroutines + channels fit the reconciler + task-worker design
  directly.
- Strong static typing reduces the correctness-bar risk on
  credential handling and state-machine transitions.

## Consequences

- **Positive:** one tiny image per service, trivial backups, minimal
  runtime footprint, mature OIDC and Kubernetes libraries.
- **Negative:** slightly higher barrier for contributors coming from
  PHP / Python backgrounds. Mitigation: keep provider contracts small
  and well-documented so new-provider PRs stay readable.
- **Follow-ups:**
  - [ADR-0002](0002-task-queue.md) can now choose among Go-native
    task queues (asynq, river).
  - [ADR-0006](0006-auth-and-oidc.md) can pin concrete Go OIDC
    libraries (`github.com/coreos/go-oidc`, `golang.org/x/oauth2`).
  - Update `.editorconfig` (already has `[*.go]` tab style).
  - Remove Python-specific scaffolding paths in `.gitignore` as the
    codebase grows.

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md)
- [ROADMAP.md](../ROADMAP.md) v0.1 exit criteria

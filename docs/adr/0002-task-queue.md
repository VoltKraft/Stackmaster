# ADR-0002: Task queue / workflow runtime

- **Status:** Accepted
- **Date:** 2026-04-20
- **Tags:** backend, foundation, reliability

## Context and problem

Stackmaster runs two related but distinct execution primitives:

- **Reconciler tasks**: short, idempotent provider calls enqueued by
  the reconciler (see [ARCHITECTURE.md §3](../ARCHITECTURE.md)).
- **Workflow runs**: long-lived, possibly multi-hour DAGs with retries,
  delays, and parallel branches (see [WORKFLOWS.md](../WORKFLOWS.md)).

We can use one runtime for both, or pick two specialized runtimes.
Key requirements:

- Persistent, resumable after restart.
- Retry with exponential backoff per task.
- Cron / scheduled triggers.
- Observability: per-task status, per-workflow-run timeline.
- Language compatibility — see [ADR-0001](0001-backend-language.md).

## Options considered

### Option A — Temporal (embedded or external)

- **Summary:** durable-execution engine with a polyglot SDK.
- **Pros:** battle-tested, handles the full DAG + retry model; SDKs
  for Go and Python; deterministic replay.
- **Cons:** adds a service to the deployment topology (Temporal
  server + history/matching). Mental model of durable coroutines is
  heavier than most operators expect.
- **Effort:** medium — integration is mostly about embracing the SDK.
- **Reversibility:** medium.

### Option B — asynq (Go) or Celery (Python)

- **Summary:** classic job queues on Redis.
- **Pros:** small, familiar, no extra service beyond Redis (already
  part of the stack).
- **Cons:** they are *task queues*, not *workflow engines*. Building
  resumable DAGs on top of them means writing the persistence and
  replay logic ourselves — which is exactly ADR-0008.
- **Effort:** low for reconciler tasks; high for workflows.
- **Reversibility:** easy.

### Option C — Custom DAG + PostgreSQL

- **Summary:** keep Redis for ephemeral tasks; persist workflow runs
  and their node state in PostgreSQL using event-sourced rows.
- **Pros:** no extra service; one database is the single source of
  truth; natural integration with audit and state-machine tables.
- **Cons:** we own the engine. Every feature (retry, cancel,
  parallel, timers) is ours to implement and test.
- **Effort:** high.
- **Reversibility:** medium — the public YAML schema is stable even
  if the engine is rewritten later.

### Option D — n8n (embedded)

- **Summary:** popular node-based automation tool.
- **Pros:** huge node library for notifications and external systems.
- **Cons:** license is fair-code, not fully open-source; embedding
  it as a library is awkward; its execution model is event-driven,
  not controller-style. Wrong shape for our use case.
- **Effort:** medium.
- **Reversibility:** hard.

### Option E — Argo Workflows-inspired (in-process)

- **Summary:** take the DAG + retry semantics Argo defines, reimplement
  in-process.
- **Pros:** Kubernetes-native mental model familiar to our target
  audience; YAML shape maps almost 1:1.
- **Cons:** we still own the engine.
- **Effort:** high.
- **Reversibility:** medium.

## Decision

Two specialized runtimes, aligned with [ADR-0001](0001-backend-language.md) (Go):

- **Reconciler tasks: [asynq](https://github.com/hibiken/asynq)** on
  Redis. Mature, Go-native, Redis-backed (which we already run),
  well-understood retry and scheduling model.
- **Workflow engine: custom DAG on PostgreSQL** — decided in
  [ADR-0008](0008-workflow-engine.md). No extra service; YAML is
  first-class; integrates tightly with audit and state machine.

## Consequences

- **Positive:**
  - No new infrastructure components — only PostgreSQL and Redis,
    both already present.
  - Clear split of concerns: asynq handles short idempotent provider
    calls; the DAG engine handles durable, human-facing workflows.
  - v0.1 can ship with asynq plus hand-rolled logic for the single
    built-in workflow; the full DAG engine lands in v0.3.
- **Negative:**
  - Two runtimes means two observability surfaces to wire into
    Prometheus and the audit log. Mitigation: unified task-event
    envelope across both.
- **Follow-ups:**
  - Pin the asynq version in the core `go.mod`.
  - Design the `tasks` schema such that asynq payloads carry enough
    context for the audit log.
  - Define the unified task-event envelope shared with the DAG
    engine in ADR-0008.

## References

- [ARCHITECTURE.md §3–4](../ARCHITECTURE.md)
- [WORKFLOWS.md](../WORKFLOWS.md)

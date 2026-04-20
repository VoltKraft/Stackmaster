# ADR-0008: Workflow engine model

- **Status:** Accepted
- **Date:** 2026-04-20
- **Tags:** backend, workflows, reliability

## Context and problem

Workflows are typed DAGs with retries, delays, parallel branches,
ForEach, triggers, and notifications ([WORKFLOWS.md](../WORKFLOWS.md)).
They are persistent, resumable, auditable, and support a dry-run
mode. ADR-0002 separates the reconciler task queue from the workflow
runtime; this ADR chooses the workflow runtime.

Requirements:

- Durable execution: a crash resumes mid-DAG.
- Cron + webhook + manual + internal-event triggers.
- Per-node retries with exponential backoff.
- Cancel + timeout semantics.
- Dry-run: walk every node without side effects.
- Observability: per-run timeline, per-node status, structured audit.
- License: must be open source, must be license-compatible with AGPL.

## Options considered

### Option A — Embed Temporal

- **Summary:** Temporal SDK for the chosen language; Temporal server
  (or Temporalite for single-node) alongside the core.
- **Pros:**
  - Proven durable-execution model.
  - SDKs for Go and Python.
  - Rich replay tooling.
- **Cons:**
  - Extra service to deploy and back up.
  - Workflow-as-code (SDK-based) clashes with our YAML-as-source-of-truth
    model; we would build a YAML interpreter on top of Temporal
    workflows. Doable, but adds a layer.
  - Mental model is heavy for a self-hosted single-node target.

### Option B — Custom DAG engine on PostgreSQL

- **Summary:** implement a DAG engine where runs and node states are
  rows in PostgreSQL. Workers process pending nodes; a scheduler
  wakes workflows by trigger.
- **Pros:**
  - No extra service.
  - YAML is first-class; the engine interprets it directly.
  - Natural integration with audit, state machine, and credential
    vault (all in the same DB).
  - Simpler to reason about for a homelab deployment.
- **Cons:**
  - We own the engine: retries, timers, cancellation, parallel,
    ForEach, replay, and checkpointing are all on us.
  - Heavy tail of correctness work (especially around timers and
    crash recovery).

### Option C — Embed n8n

- **Summary:** use n8n as an embedded workflow engine.
- **Pros:** vast node library for notifications.
- **Cons:** fair-code license, not strictly OSI open source; embedding
  is not n8n's supported use case; event-driven model mismatches our
  controller-style reconciler.

### Option D — Argo Workflows-inspired, in-process

- **Summary:** reuse Argo's YAML shape and semantics, build our own
  runtime. Essentially a specialization of Option B.
- **Pros:** familiar mental model for Kubernetes-literate operators.
- **Cons:** same as Option B — we own it.

## Decision

**Custom DAG engine on PostgreSQL (Option B).** Workflow runs and
node executions are rows; a scheduler wakes workflows by trigger;
a pool of workers processes pending nodes. Implemented in Go
([ADR-0001](0001-backend-language.md)), alongside asynq for the
reconciler task queue ([ADR-0002](0002-task-queue.md)).

**Phasing:**

- **v0.1:** a single hard-coded workflow (`wake-on-demand`) is
  implemented as bespoke Go code. No generic engine yet.
- **v0.3:** the generic DAG engine lands, with YAML as the source of
  truth and the node editor on top.

## Consequences

- **Positive:**
  - No additional service to deploy — one PostgreSQL covers desired
    state, observed state, audit, credential vault, and workflow
    runs.
  - YAML stays the source of truth for workflows; the engine
    interprets it directly.
  - The engine can be shaped exactly to our state machine and audit
    story — no impedance mismatch with Temporal's SDK-first model.
- **Negative:**
  - Significant engineering investment for v0.3.
  - We must pass our own correctness bar for durable execution —
    property-based tests and crash-recovery fuzzing are mandatory
    before declaring v1.0 ready.
- **Follow-ups:**
  - Specify the checkpoint schema (per-run, per-node rows + JSON
    snapshots).
  - Specify the trigger scheduler (Go cron library, timezone rules,
    DST handling).
  - Specify the expression language used in `when` and `${{ }}`
    (candidates: CEL, expr-lang/expr, a pinned subset).
  - Define the unified task-event envelope shared with asynq.
  - Update [WORKFLOWS.md](../WORKFLOWS.md) as the design matures.

## References

- [WORKFLOWS.md](../WORKFLOWS.md)
- [ADR-0002](0002-task-queue.md)
- [STATE_MACHINE.md](../STATE_MACHINE.md)

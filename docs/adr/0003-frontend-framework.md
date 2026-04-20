# ADR-0003: Frontend framework

- **Status:** Accepted
- **Date:** 2026-04-20
- **Tags:** frontend, foundation

## Context and problem

The UI is a separate SPA (see [ARCHITECTURE.md §7](../ARCHITECTURE.md))
that consumes the core's REST + WebSocket API. Its central surface is
the graphical node editor (see [NODE_EDITOR.md](../NODE_EDITOR.md)).

Requirements:

- Rich interactivity: node editor, live log streams, console
  passthrough.
- Good typescript + accessibility story.
- Reasonable bundle size — Stackmaster ships to homelabs, not to
  bandwidth-rich browsers only.
- Compatible with the shortlisted node-editor libraries (see
  [ADR-0007](0007-node-editor-library.md)).

## Options considered

### Option A — SvelteKit

- **Summary:** Svelte 5 runes + SvelteKit routing.
- **Pros:** smallest bundle for equivalent UI; great DX; native stores
  map well onto live WebSocket state.
- **Cons:** smaller pool of node-editor libraries integrate natively
  (Drawflow yes, React Flow / Rete via adapters).
- **Effort:** medium.
- **Reversibility:** medium (porting a heavy SPA is expensive).

### Option B — React (Vite)

- **Summary:** React 18+ with Vite, no Next.js (SPA only).
- **Pros:** broad library support; React Flow and Rete.js have
  first-class React bindings; large contributor pool.
- **Cons:** heavier bundle; more ceremony for state management.
- **Effort:** medium.
- **Reversibility:** medium.

### Option C — Vue 3

- **Summary:** Composition API with Vite.
- **Pros:** modest bundle, clean component model.
- **Cons:** node-editor library support is mixed; Drawflow works,
  others need wrappers.
- **Effort:** medium.
- **Reversibility:** medium.

## Decision

**React + Vite (SPA, no Next.js).** Paired with the node-editor
decision in [ADR-0007](0007-node-editor-library.md): React Flow is
React-native, so the framework follows.

## Consequences

- **Positive:**
  - React Flow, Rete.js (if ever needed), and most ancillary UI
    libraries have first-class React bindings.
  - Large contributor pool familiar with the stack.
  - Vite keeps dev-loop fast and production bundles reasonable.
- **Negative:**
  - Bundle size is larger than a SvelteKit equivalent. Mitigation:
    code-split per route; lazy-load the editor.
  - Mild ceremony for state management. Mitigation: keep global
    state small and co-located with the WebSocket sync layer from
    ARCHITECTURE.md §10.
- **Follow-ups:**
  - Pin React and React Flow versions in `ui/package.json` when the
    first UI commit lands.
  - Pick a state-management approach for the WebSocket event stream
    (Zustand or TanStack Query with invalidation on event seq).
  - Pick a component library (or go headless with Tailwind).

## References

- [NODE_EDITOR.md](../NODE_EDITOR.md)

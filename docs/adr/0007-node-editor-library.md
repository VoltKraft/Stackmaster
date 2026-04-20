# ADR-0007: Node-editor library

- **Status:** Accepted
- **Date:** 2026-04-20
- **Tags:** frontend, ux

## Context and problem

The node editor is Stackmaster's central UI primitive
([NODE_EDITOR.md](../NODE_EDITOR.md)). It serves two modes —
infrastructure topology and workflow DAGs — in one UX. The library
choice determines:

- The frontend framework decision ([ADR-0003](0003-frontend-framework.md)).
- Bundle size.
- Whether we can implement typed ports, custom node rendering, and
  large-graph performance without fighting the library.
- License compatibility with the project's AGPL leaning
  ([ADR-0004](0004-license.md)).

## Options considered

### Option A — React Flow

- **Summary:** React-native library; the de facto standard in the
  React ecosystem.
- **Pros:**
  - Mature, well-documented, active maintenance.
  - Custom node renderers, edge types, minimap, controls out of the
    box.
  - Typed ports supported with additional validation on our side.
  - Good large-graph performance with virtualization.
- **Cons:**
  - Requires React. Forces [ADR-0003](0003-frontend-framework.md).
  - License is MIT for core; some addons (Pro) are commercial —
    we only use the open core.
- **Effort:** low.
- **Reversibility:** medium.

### Option B — Rete.js

- **Summary:** framework-agnostic engine with renderers for React,
  Vue, Svelte, Angular.
- **Pros:**
  - Truly framework-agnostic; strong port/socket type system.
  - Plugin architecture suitable for our two modes.
- **Cons:**
  - Steeper learning curve; API is in flux across major versions.
  - Fewer ready-made components (minimap, zoom controls) — we build
    them.
- **Effort:** medium-high.
- **Reversibility:** medium.

### Option C — Drawflow

- **Summary:** small, dependency-light, vanilla-JS library.
- **Pros:**
  - Tiny bundle.
  - Works with Svelte / Vue cleanly.
  - Low cognitive load.
- **Cons:**
  - Limited features for typed ports and custom node rendering.
  - Slower pace of upstream development.
- **Effort:** low-medium.
- **Reversibility:** easy.

### Option D — Litegraph.js

- **Summary:** graph engine with a canvas renderer, used in some
  creative-coding tools.
- **Pros:** canvas rendering scales to very large graphs.
- **Cons:** less mainstream; integration with a modern SPA framework
  is awkward; DOM-based interactions (tooltips, menus) harder to
  layer.
- **Effort:** high.
- **Reversibility:** hard.

## Decision

**React Flow** (open-source core, MIT). This also locks
[ADR-0003](0003-frontend-framework.md) to React + Vite.

## Consequences

- **Positive:**
  - Mature library with custom nodes, minimap, controls, and edge
    types out of the box.
  - Large community; most of our editor features are additive rather
    than invented.
  - Typed ports and edge validation can be layered on top without
    fighting the library.
- **Negative:**
  - Forces React; a future rethink would be costly.
  - "Pro" add-ons exist but are commercial — Stackmaster uses the
    open core only. A CI check should prevent accidental Pro
    imports.
- **Follow-ups:**
  - Pin a React Flow major version in `ui/package.json`.
  - Implement our own node-type and edge-type validators on top
    (AGPL-licensed, part of the UI).
  - Build the palette + YAML round-trip layer as documented in
    [NODE_EDITOR.md](../NODE_EDITOR.md).
  - Adopt a hierarchical auto-layout (e.g. dagre) for first open of
    a workflow or topology.

## References

- [NODE_EDITOR.md](../NODE_EDITOR.md)
- [ADR-0003](0003-frontend-framework.md)

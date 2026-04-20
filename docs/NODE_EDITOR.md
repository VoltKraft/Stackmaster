# Node Editor

The graphical node editor is Stackmaster's central UI primitive. It
serves **two** purposes with one UX and one underlying data model:

1. **Infrastructure topology** — the visual map of the defined world:
   hypervisors, platforms, servers, and credential references, with
   dependency edges between them.
2. **Workflow designer** — a DAG builder for automated flows
   (see [WORKFLOWS.md](WORKFLOWS.md)).

A single user interface; two modes.

## Principles

- **The graph is a view onto YAML, never the other way around.** The
  YAML is canonical. Editing the graph produces the same YAML change
  the user could have made by hand.
- **Isomorphism.** A round-trip YAML → graph → YAML is byte-identical
  up to key order.
- **Keyboard-first.** Every graph operation has a keyboard equivalent.
  The target user owns a homelab; they type.
- **Multiplayer-friendly (read-only first).** v1.0: one editor at a
  time. v1.x: soft-locks during edit; conflict detection on save.
- **Dense information, low chrome.** Nodes are small, labels are
  terse, badges carry state (healthy / error / pending).

## Library choice

Deferred to [ADR-0007](adr/0007-node-editor-library.md). Candidates
under evaluation: React Flow, Rete.js, Drawflow, Litegraph.js. The
evaluation matrix in the ADR considers license, bundle size, TS
support, port/edge ergonomics, and large-graph performance.

## Node categories

### Infrastructure topology

- **Hypervisor node** — color-coded by provider. Shows power state,
  provider name, connection health.
- **Platform node** — color-coded by provider (docker, k8s, lxc, …).
  Nests under its hypervisor node visually.
- **Game server node** — color-coded by game provider. Nests under
  its platform node. Badges show current state from the state
  machine.
- **Credential reference node** — neutral color, lock icon. Shows
  alias and type; never value. Dashed edges connect a credential to
  the resources that consume it.

### Workflow nodes (see [WORKFLOWS.md](WORKFLOWS.md))

- Trigger (yellow)
- Condition (blue)
- Action (green)
- Notification (magenta)
- Delay (red)
- Parallel (multi-outline)
- ForEach (stack outline)
- Utility (gray)

Color conventions must stay consistent between the two modes where a
node type exists in both (e.g. a credential reference in topology
mirrors a credential reference in a workflow).

## Interaction model

- Drag-and-drop to add a node from a palette on the left.
- Click-and-drag from a port to draw an edge. Port types are
  validated — Stackmaster rejects incompatible edges before save.
- Right-click a node for context actions (delete, duplicate,
  "jump to YAML at line N", "open audit log scoped to this node").
- Cmd/Ctrl-F opens a search box that matches node labels, provider
  types, and tags.
- Zoom, pan, mini-map, fit-to-screen are table stakes.

## Graph ↔ YAML mapping

Every node carries a stable `id`. The YAML representation is a list
of nodes and a list of edges (see [WORKFLOWS.md](WORKFLOWS.md) for
the exact schema). Editor-only metadata (x/y coordinates, colors,
collapse state) lives under `metadata.editor`:

```yaml
metadata:
  editor:
    positions:
      check-platform: { x: 120, y:  40 }
      boot-host:      { x: 120, y: 140 }
    viewport:
      zoom: 1.0
      x: 0
      y: 0
```

The engine ignores `metadata.editor`. A programmatic `apply` of a
workflow without editor metadata is valid; the editor lays out the
graph automatically on first open and then persists positions.

## Accessibility

- Keyboard navigation across nodes and edges.
- Focus outlines visible on all interactive elements.
- Color alone never carries meaning — every colored badge also has
  an icon.
- Respect `prefers-reduced-motion` for pan/zoom animations.

## Performance targets

- Interactive frame rate (60fps) for graphs up to 500 nodes.
- Lazy-render off-screen nodes for larger graphs.
- Web worker for layout computation on >200 nodes.

## TODO

- [ ] Decide the editor library (ADR-0007).
- [ ] Specify the auto-layout algorithm per mode (topology = hierarchical
      by stack layer; workflow = DAG top-down).
- [ ] Design the palette for credential-reference nodes so Operators
      can reference without seeing the vault list.
- [ ] Plan the diff view (show what a save will change in the YAML).

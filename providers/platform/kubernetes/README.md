# providers/platform/kubernetes

Platform provider for Kubernetes clusters.

**Status:** not yet implemented. Target: **v0.2**.

- Verbs: `ensureReady`, `deployUnit`, `startUnit`, `stopUnit`,
  `statusUnit`, `streamLogs`, `exec`, `readFile`, `writeFile`
  (via emptyDir/volume), `capabilities`.
- Credentials: kubeconfig (via credential vault).
- Transport: Kubernetes API.

Each "unit" maps to a Deployment / StatefulSet (plus Service /
PodDisruptionBudget / ConfigMap) in a namespace owned by
Stackmaster.

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

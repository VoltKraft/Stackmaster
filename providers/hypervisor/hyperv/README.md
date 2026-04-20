# providers/hypervisor/hyperv

Hypervisor provider for Microsoft Hyper-V.

**Status:** not yet implemented. Target: **v0.4**.

- Verbs: `status`, `ensureUp`, `ensureDown`, `snapshot`, `clone`,
  `capabilities`.
- Credentials: WinRM credentials (basic, negotiate, or cert) or
  PowerShell remoting.
- Transport: WinRM + PowerShell cmdlets.

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

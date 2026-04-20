# providers/platform/windows-native

Platform provider for native Windows hosts where the game runs as a
Windows service or a long-lived process.

**Status:** not yet implemented. Target: **v0.4**.

- Verbs: `ensureReady`, `deployUnit`, `startUnit`, `stopUnit`,
  `statusUnit`, `streamLogs`, `exec`, `readFile`, `writeFile`,
  `capabilities`.
- Credentials: WinRM credentials (NTLM / Negotiate / Cert).
- Transport: WinRM + PowerShell.

See [../../../docs/PROVIDERS.md](../../../docs/PROVIDERS.md) and
[../../../docs/ARCHITECTURE.md](../../../docs/ARCHITECTURE.md).

# providers/

Pluggable adapters for the three stack layers:

- [`hypervisor/`](hypervisor/) — Proxmox, libvirt, ESXi, Hyper-V, noop.
- [`platform/`](platform/) — Docker, Kubernetes, LXC, systemd, Windows.
- [`game/`](game/) — SteamCMD, LinuxGSM, custom images, scripts.

**Status:** not yet implemented.

The interface contract and conformance requirements are documented in
[../docs/PROVIDERS.md](../docs/PROVIDERS.md) and
[../docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md).

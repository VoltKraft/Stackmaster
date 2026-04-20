---
name: Bug report
about: Something in Stackmaster misbehaves.
title: "[bug] "
labels: ["bug", "triage"]
assignees: []
---

## Summary

A clear, one-paragraph description of what's wrong.

## Environment

- Stackmaster version / commit:
- Deployment mode: (docker compose / Helm / dev)
- OS + version of the host:
- Relevant providers in use: (proxmox / docker / k8s / linuxgsm / …)
- Auth mode: (local / PAT / OIDC — OIDC is planned for v0.9)
- OIDC provider (only if on v0.9+): (authentik / keycloak / authelia / …)

## Steps to reproduce

1.
2.
3.

## Expected behavior

What should have happened.

## Actual behavior

What actually happened. Include error messages verbatim.

## Logs

Attach redacted logs. **Do not paste credentials or tokens.**

<details>
<summary>core logs</summary>

```
```

</details>

## Additional context

Anything else that helps diagnose: screenshots, network topology,
firewall rules, non-default settings.

## Checklist

- [ ] I have searched existing issues and this is not a duplicate.
- [ ] I have redacted credentials from the logs and attachments.
- [ ] I have read [SECURITY.md](../../SECURITY.md) and this is **not**
      a security vulnerability (if it is, report privately).

---
name: Provider proposal
about: Propose a new hypervisor / platform / game provider.
title: "[provider] <layer>/<name>"
labels: ["provider", "enhancement"]
assignees: []
---

## Which layer?

- [ ] Hypervisor
- [ ] Platform
- [ ] Game

## Proposed name

`<layer>/<short-name>` (e.g. `platform/nomad`, `game/pterodactyl`).

## Motivation

What does this provider enable that the current set does not? Who
asked for it? How common is the target backend?

## Scope

- Verbs to support (check against [PROVIDERS.md](../../docs/PROVIDERS.md)):
  - [ ]
  - [ ]
- Optional verbs this provider will **not** support, and why:

## Credentials

- Credential types consumed:
- Rotation strategy:
- Plaintext leak risk and mitigation:

## Transport

How does the provider reach the backend? (Native API / SSH / agent /
other.) If an agent is needed, justify why no agent-less path works.

## License and upstream dependencies

- Backend's license:
- Client library's license:
- Is it compatible with Stackmaster's AGPL default?

## Conformance

- [ ] I commit to implementing the conformance suite for this
      provider's layer.
- [ ] I have access to a real backend for live smoke tests in CI, or
      I accept that this provider will ship as experimental until
      someone does.

## Risks

Anything operators should know: rate limits, authentication quirks,
known upstream bugs, uptime requirements.

## Timeline

Which milestone do you target? See [ROADMAP.md](../../docs/ROADMAP.md).

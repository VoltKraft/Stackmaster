# ADR-0004: License

- **Status:** Accepted
- **Date:** 2026-04-20
- **Tags:** legal, foundation

## Context and problem

Stackmaster will be an open-source project. The license affects:

- Who can use, modify, and host Stackmaster.
- Whether a hoster can take a fork private and offer it as SaaS
  without contributing back.
- Compatibility with dependencies (most of our stack is MIT /
  Apache-2.0).
- Contributor sentiment — a permissive license lowers friction; a
  strong copyleft signals intent.

## Options considered

### Option A — AGPL-3.0-or-later (current default in `LICENSE`)

- **Summary:** strong copyleft, triggers network-use clause.
- **Pros:** prevents a closed-source SaaS fork; aligns with Pelican's
  choice; matches the project's "operator tool, not vendor tool"
  ethos; still compatible with MIT/Apache dependencies.
- **Cons:** some enterprises disallow AGPL at policy level; might
  reduce adoption in commercial hosting shops that want to embed
  Stackmaster and not publish their glue.
- **Effort:** low (already in `LICENSE`).
- **Reversibility:** effectively impossible once contributions land.

### Option B — Apache-2.0

- **Summary:** permissive, patent grant, no copyleft.
- **Pros:** maximum adoption; no friction for commercial hosters;
  matches Kubernetes / Agones.
- **Cons:** allows closed forks to offer Stackmaster-as-a-service
  without contributing back.
- **Effort:** low.
- **Reversibility:** hard.

### Option C — Elastic License 2 / BSL

- **Summary:** source-available, restricts SaaS resale.
- **Pros:** explicit SaaS anti-parasite clause.
- **Cons:** not OSI-approved, not open-source by OSI definition; harms
  trust and adoption; inconsistent with the project's stated identity.
- **Effort:** low.
- **Reversibility:** hard.

### Option D — Dual license (AGPL + commercial)

- **Summary:** default AGPL with a paid escape hatch.
- **Pros:** funds sustainability; lets non-AGPL enterprises pay.
- **Cons:** requires a CLA; introduces legal overhead; can chill
  community contributions.
- **Effort:** high (needs CLA infrastructure).
- **Reversibility:** medium — can move to dual later if governance
  allows.

## Decision

**AGPL-3.0-or-later.** The repository's `LICENSE` file already
carries this license; this ADR ratifies it.

## Consequences

- **Positive:** closes the "privatized SaaS fork" gap and matches
  Stackmaster's identity as an operator tool; consistent with
  Pelican's licensing for downstream interop.
- **Negative:** some enterprises with internal AGPL bans will not
  adopt; a future dual-licensing move would require a CLA policy.
- **Follow-ups:**
  - Confirm the CONTRIBUTING note that contributions are licensed
    under AGPL-3.0-or-later (already present).
  - When adding third-party dependencies, verify AGPL compatibility
    before vendoring.
  - Update file headers (once code lands) to include an AGPL SPDX
    identifier.

## References

- [LICENSE](../../LICENSE)
- [CONTRIBUTING.md](../../CONTRIBUTING.md)
- Pelican licensing (AGPL-3.0) as prior art.

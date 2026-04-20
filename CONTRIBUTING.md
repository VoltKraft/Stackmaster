# Contributing to Stackmaster

Stackmaster is an open-source project and welcomes contributions.
This document explains how to propose changes, file issues, and
discuss architecture.

**Status:** the repository is in the scaffolding phase. Until the
first ADR set is accepted, most contributions will be **design
discussions**, not code.

## Before you open an issue

- Check [docs/ROADMAP.md](docs/ROADMAP.md) — the feature may already be
  planned.
- Check open issues.
- If you are proposing a new **provider**, use the
  [provider proposal template](.github/ISSUE_TEMPLATE/provider_proposal.md).

## Discussion > code (until v0.1)

The fastest way to help right now is:

- Review the open ADRs in [docs/adr/](docs/adr/) and comment on the
  tradeoffs.
- Propose missing scenarios for [docs/ROADMAP.md](docs/ROADMAP.md).
- Propose additions to [docs/GLOSSARY.md](docs/GLOSSARY.md) when terms
  feel ambiguous.

## Pull requests

Once code lands in this repository:

1. Fork, branch from `main`, keep commits tidy.
2. Write or update tests for new behavior.
3. Update docs in the same PR — undocumented behavior is not shipped.
4. Run the project's linters and test suite before requesting review.
5. Fill in the PR template and link any relevant ADR or issue.
6. Sign off commits with `Signed-off-by:` (DCO).

## Style

- **English** is the working language for all docs, comments, and
  identifiers.
- **Terminology** follows [docs/GLOSSARY.md](docs/GLOSSARY.md). Don't
  say "module" when you mean provider; don't say "node" without a
  qualifier.
- **Commits** — conventional commits
  (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`). Keep the
  subject under 72 characters.

## Provider contributions

Every provider must ship with:

- A README in `providers/<layer>/<name>/` that documents supported
  verbs, required credentials, and known limitations.
- A conformance-test pass (the suite is published by the core).
- A live-smoke path in CI when credentials are available.

See [docs/PROVIDERS.md](docs/PROVIDERS.md).

## Licensing of contributions

By contributing, you agree that your contribution will be distributed
under the project's license. Stackmaster's current leaning is AGPL-3.0
([ADR-0004](docs/adr/0004-license.md)); if that changes before your PR
merges, you will be asked to reaffirm.

## Security

Do not open public issues for security vulnerabilities. See
[SECURITY.md](SECURITY.md) for responsible disclosure.

## Code of conduct

See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md). Discussions are held to
the same standard as code.

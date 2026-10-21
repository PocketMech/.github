# Contributing to PocketMech

Thank you for contributing. This document covers how we collaborate across PocketMech repositories.

## Maintainer

- GitHub: [@Bercek71](https://github.com/Bercek71)

## Gitflow

| Branch | Purpose |
|---|---|
| `main` | Release / production |
| `develop` | Integration |
| `feature/*` | New work |
| `bugfix/*` | Bug fixes |
| `release/*` | Release preparation |
| `hotfix/*` | Urgent production fixes |

Rules:

- Work on `feature/*`, `bugfix/*`, `release/*`, or `hotfix/*` branches.
- Open a pull request into **`develop`** for every change.
- Review by [@Bercek71](https://github.com/Bercek71) is **required**.
- **Do not** push directly to `main` or `develop`.
- **Do not** merge without the required review.

## Branch naming

Use prefixes that match the work:

- `feature/<short-description>`
- `bugfix/<short-description>`
- `release/<identifier>`
- `hotfix/<short-description>`

Prefer lowercase kebab-case descriptions (for example `feature/org-profile-community`).

## Conventional Commits

Write commit messages in the [Conventional Commits](https://www.conventionalcommits.org/) style:

```text
<type>(optional-scope): <description>
```

Common types:

- `feat` — new capability
- `fix` — bug fix
- `docs` — documentation only
- `chore` — maintenance
- `ci` — continuous integration
- `refactor` — restructuring without behavior change
- `test` — tests
- `perf` — performance
- `style` — formatting / non-semantic style
- `build` — build system or dependencies

Examples:

```text
feat(core): add circuit breaker schema types
fix(mobile-app): correct ATA chapter search ranking
docs: clarify offline disclaimer
```

## Pull request checklist

Before requesting review:

- [ ] Base branch is `develop` (unless this is an explicitly approved `hotfix/*` flow into `main`)
- [ ] Branch name follows the conventions above
- [ ] Commits follow Conventional Commits
- [ ] Description explains **what** changed and **why**
- [ ] Related issues are linked when applicable
- [ ] Docs and community files updated when behavior or process changes
- [ ] No secrets, credentials, or private operator data in the diff
- [ ] Reviewer: [@Bercek71](https://github.com/Bercek71)

## Architecture decisions

ADRs live in [`docs/adr/`](docs/adr/) and are maintained by the System Architect. Do not add or edit ADRs unless you are performing that role for an agreed decision.

## Code of conduct

By participating, you agree to follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## Security

Do not open public issues for vulnerabilities. Follow [SECURITY.md](SECURITY.md).

## Support

Questions and help requests: [SUPPORT.md](SUPPORT.md).

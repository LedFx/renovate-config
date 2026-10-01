# renovate-config

The shared Renovate policy for LedFx repositories. Extend it from each repo's
`renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>LedFx/renovate-config"]
}
```

Renovate's onboarding PR suggests this preset automatically, because it lives in
`<org>/renovate-config` with a `default.json`.

## Policy

- `config:best-practices`: digest-pinned Actions and Docker images, abandoned-package detection, weekly lock file maintenance.
- Renovate opens and rebases update PRs once a week, before 04:00 UTC on Monday (`schedule:weekly`). Automerge still happens whenever CI goes green, and security fixes don't wait for the schedule.
- Every update waits 14 days after release (npm included); majors wait 30 days.
- Minor, patch, digest, pin and lock file updates automerge (squash) once CI is green. Majors never automerge.
- `requires-python` is never updated: it's a support policy. Python/Node runtime bumps (base images, `.python-version`, `.nvmrc`) never automerge.
- Security fixes skip the wait, provided Dependabot alerts are on in the repo.
- Conventional Commit prefixes on commits and PR titles (`chore(deps): …`, `fix(deps): …`).
- At most 5 open Renovate PRs per repo, labelled `dependencies`.
- Hook `rev`s in `.pre-commit-config.yaml` are updated too (the `pre-commit` manager is off by default in Renovate). prek reads the same file.
- ruff minor releases never automerge: on 0.x they can change formatting or default rules repo-wide.

Automerge is only as safe as the repo's required checks. Renovate won't merge a
branch with no passing checks, but only branch protection guarantees checks exist.

Repo-specific rules go in that repo's `renovate.json`; they're applied after this
preset and win.

## Changing it

This file is read from `main` on every Renovate run in every repo that extends
it. Validate before merging:

```sh
npx --yes --package renovate -- renovate-config-validator default.json
```

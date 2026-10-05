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
- Dependency updates normally wait 14 days after release (npm included); majors wait 30 days. The reviewed release action exception below has no waiting period.
- Minor, patch, digest, pin and lock file updates automerge (squash) once CI is green. Majors never automerge.
- `requires-python` is never updated: it's a support policy. Python/Node runtime bumps (base images, `.python-version`, `.nvmrc`) never automerge.
- Security fixes skip the wait, provided Dependabot alerts are on in the repo.
- Conventional Commit prefixes on commits and PR titles (`chore(deps): …`, `fix(deps): …`).
- At most 5 open Renovate PRs per repo, labelled `dependencies`.
- Hook `rev`s in `.pre-commit-config.yaml` are updated too (the `pre-commit` manager is off by default in Renovate). prek reads the same file.
- ruff minor releases never automerge: on 0.x they can change formatting or default rules repo-wide.

Automerge is only as safe as the repo's required checks. Renovate won't merge a
branch with no passing checks, but only branch protection guarantees checks exist.

## Shared release action

`LedFx/release-ci` is publication infrastructure: every update requires a person
to review it, including minor, patch and digest changes. Its narrow, last package
rule removes the normal age and weekly scheduling delays, while keeping immutable
SHA pins. Other dependencies keep their existing delays, schedule and merge policy;
the five-PR concurrency limit still applies.

Consumers should use a full commit SHA with its released version comment:

```yaml
uses: LedFx/release-ci/actions/release@5e683ef142d08c81e46164062cff23aa1091d3f1 # v0.2.0
```

Renovate's built-in `github-actions` manager tracks the subdirectory action under
package name `LedFx/release-ci`. It updates both the SHA and version comment when a
new release is available. A bare SHA without the version comment cannot reliably
track releases. No custom regex manager is needed.

An eligible update is offered on the **next Renovate run**, subject to the existing
PR limits; this preset does not install a release webhook or trigger an immediate
run. See the [GitHub Actions manager documentation](https://docs.renovatebot.com/modules/manager/github-actions/).

Repo-specific rules go in that repo's `renovate.json`; they're applied after this
preset and win. Keep later consumer rules from overriding this action's manual
review and release tracking policy.

## Changing it

This file is read from `main` on every Renovate run in every repo that extends
it. Validate before merging:

```sh
npx --yes --package renovate -- renovate-config-validator default.json
```

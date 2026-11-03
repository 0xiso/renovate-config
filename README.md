# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) preset for 0xiso repositories.

## Use

In a repository's Renovate config (`renovate.json`, `.github/renovate.json5`, ...):

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>0xiso/renovate-config"]
}
```

Add repository-specific rules after it. Renovate's onboarding PR suggests this
preset automatically for new repositories, because this repository exists.
The Renovate app must have access to this repository as well.

## What `default.json` does

| Setting | Effect | Why |
|---|---|---|
| `config:recommended` | Renovate's recommended base | |
| `helpers:pinGitHubActionDigestsToSemver` | Actions referenced by tag (`@v7`) are pinned to a commit SHA, with a full `# v7.0.1` comment | A tag can be moved to other code (e.g. a hijacked maintainer account); a SHA cannot |
| `schedule` | Saturday 00:00-08:59 JST (`timezone: Asia/Tokyo`) | One batch a week, outside working days |
| `minimumReleaseAge: 3 days` | Versions published less than 3 days ago are not proposed (datasources without timestamps are not held back) | Hijacked or broken releases are usually caught and pulled within days |
| `vulnerabilityAlerts`, `osvVulnerabilityAlerts` | Security fixes ignore the schedule and the 3-day wait, and get a `security` label. OSV is checked in addition to GitHub advisories | Fixes should land quickly; OSV covers Go modules and Actions well |
| `rebaseWhen: conflicted` | PR branches are rebased only when they conflict | Every rebase re-runs CI; use the PR's rebase checkbox to refresh one by hand |
| `configMigration` | Renovate opens PRs to migrate configs when options are renamed | |
| `labels` | `dependencies` on every Renovate PR | |
| GitHub Actions group | Non-major action updates in one PR | |
| GitHub Actions runner group | `runs-on` image upgrades (e.g. `ubuntu-24.04` -> `26.04`) in their own PR | They change the whole job environment |

Not set on purpose:

- Automerge stays off (Renovate's default). Enable it per repository where a
  green CI is enough to ship.
- Commit messages use Renovate's default, which follows each repository's
  history (semantic commits only where the repository already uses them).

## Validate

```bash
npx --yes --package renovate@44.160.1 -- renovate-config-validator --strict default.json
```

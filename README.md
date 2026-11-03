# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) settings for 0xiso
repositories, applied through the Mend Renovate app's
[inherited config](https://docs.renovatebot.com/mend-hosted/hosted-apps-config/):
`org-inherited-config.json` in this repository is applied to every repository
the app can access, before that repository's own config.

## Use

Nothing to add in each repository: as soon as the Renovate app can access a
repository, Renovate runs there with the settings below (`onboarding: false`,
`requireConfig: optional`), without an onboarding PR or a config file. A
repository that needs more adds its own `renovate.json` (or
`.github/renovate.json5`, ...) with only what is specific to it; it does not
need to extend anything.

Every repository the app can access gets dependency PRs, so give the app
access only to repositories that should be updated.

Requirements:

- The Renovate app has access to this repository.
- Mend enables inheritance when it sees a commit to `org-inherited-config.json`.
  If it does not take effect (e.g. the file was committed before the app could
  see this repository), push a new commit to it.
- The Mend docs describe this for organizations. 0xiso is a personal account,
  so check it works: a repository's Dependency Dashboard and the
  [Mend job log](https://developer.mend.io/) show the effective config,
  and Renovate PRs carry the `dependencies` label. If it does not apply, rename the file back to
  `default.json` and add `"extends": ["github>0xiso/renovate-config"]` to each
  repository instead.

## What `org-inherited-config.json` does

| Setting | Effect | Why |
|---|---|---|
| `onboarding: false`, `requireConfig: optional` | No onboarding PR; repositories without a config file are processed with this config | Settings live here, not in each repository |
| `config:recommended` | Renovate's recommended base | |
| `helpers:pinGitHubActionDigestsToSemver` | Actions referenced by tag (`@v7`) are pinned to a commit SHA, with a full `# v7.0.1` comment | A tag can be moved to other code (e.g. a hijacked maintainer account); a SHA cannot |
| `minimumReleaseAge: 3 days` | Versions published less than 3 days ago are not proposed (datasources without timestamps are not held back) | Hijacked or broken releases are usually caught and pulled within days |
| `vulnerabilityAlerts`, `osvVulnerabilityAlerts` | Security fixes skip the 3-day wait and get a `security` label on top of `dependencies`. OSV is checked in addition to GitHub advisories | Fixes should land quickly; OSV covers Go modules and Actions well |
| `rebaseWhen: conflicted` | PR branches are rebased only when they conflict | Every rebase re-runs CI; use the PR's rebase checkbox to refresh one by hand |
| `configMigration` | Renovate opens PRs to migrate configs when options are renamed | |
| `labels` | `dependencies` on every Renovate PR | |
| GitHub Actions group | Non-major action updates in one PR | |
| GitHub Actions runner group | `runs-on` image upgrades (e.g. `ubuntu-24.04` -> `26.04`) in their own PR | They change the whole job environment |

Not set on purpose:

- No `schedule`: an update is proposed as soon as it has passed the 3-day
  cool-down, whenever Renovate runs. Set a schedule per repository if it
  needs batching.
- Automerge stays off (Renovate's default). Enable it per repository where a
  green CI is enough to ship.
- Commit messages use Renovate's default, which follows each repository's
  history (semantic commits only where the repository already uses them).

## Validate

```bash
npx --yes --package renovate@44.160.1 -- renovate-config-validator --strict org-inherited-config.json
```

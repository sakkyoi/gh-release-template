# Label Schemes Reference

This document surveys label schemes from projects that use release-drafter, and explains how to customize `.github/release-drafter.yml` to fit your project's needs.

> **Note:** release-drafter is not widely adopted by large open-source projects — most of them use alternative tooling (semantic-release, goreleaser, changesets, etc.). The schemes below represent the full range of real-world release-drafter configurations found in public repositories.

---

## Schemes

### This template — jenkinsci-derived

The label scheme in this template is derived from [jenkinsci/.github](https://github.com/jenkinsci/.github), the shared GitHub configuration used across all Jenkins plugin repositories. Jenkins-specific labels (`rfe`, `major-rfe`) have been removed and replaced with generic equivalents.

| Label | Changelog Section | Semver Bump |
|-------|-------------------|-------------|
| `breaking` | Breaking changes | major |
| `major-feature` | Major features | major |
| `major-enhancement` | Major features | major |
| `major-bug` | Major bug fixes | major |
| `feature` | New features | minor |
| `enhancement` | New features | minor |
| `removed` | Removed | minor |
| `deprecated` | Deprecated | minor |
| `bug`, `fix`, `bugfix`, `regression` | Bug Fixes | patch |
| `chore`, `cleanup` | Maintenance | patch |
| `documentation` | Documentation | patch |
| `dependencies`, `dependency` | Dependency updates | patch |
| `ci`, `build` | CI & build changes | patch |
| `test`, `tests` | Tests | patch |
| `revert`, `reverted` | Reverted changes | — |
| `no-changelog`, `skip-changelog`, `invalid` | *(excluded)* | — |

**Reference:** [jenkinsci/.github — release-drafter.yml](https://github.com/jenkinsci/.github/blob/master/.github/release-drafter.yml)

---

### release-drafter official — `type:` prefix

Used by [release-drafter/release-drafter](https://github.com/release-drafter/release-drafter) itself. Labels use a `type:` prefix to avoid collisions with other labels used for issue triage.

| Label | Changelog Section | Semver Bump |
|-------|-------------------|-------------|
| `type: breaking` | Breaking | major |
| `type: feature` | New | minor |
| `type: bug` | Bug Fixes | patch |
| `type: maintenance` | Maintenance | patch |
| `type: docs` | Documentation | patch |
| `type: dependencies` | Dependency Updates | patch |
| `type: security` | *(patch, no dedicated section)* | patch |
| `skip-changelog` | *(excluded)* | — |

The `type:` prefix is useful when a repo already has many labels for issue management and you want release-related labels to be visually distinct.

**Reference:** [release-drafter/release-drafter — release-drafter.yml](https://github.com/release-drafter/release-drafter/blob/master/.github/release-drafter.yml)

---

### release-drafter README — minimal example

The minimal example from the [release-drafter README](https://github.com/release-drafter/release-drafter#configuration), also adopted by [camunda-community-hub/zeebe-client-node-js](https://github.com/camunda-community-hub/zeebe-client-node-js). No `version-resolver` — semver bump is set manually at release time.

| Label | Changelog Section | Semver Bump |
|-------|-------------------|-------------|
| `feature`, `enhancement` | Features | *(manual)* |
| `fix`, `bugfix`, `bug` | Bug Fixes | *(manual)* |
| `chore` | Maintenance | *(manual)* |
| `breaking` | Breaking | *(manual)* |
| `deprecation` | Deprecations | *(manual)* |

Suitable for projects with infrequent releases or where the maintainer always decides the version number manually.

**Reference:** [release-drafter README — Configuration](https://github.com/release-drafter/release-drafter#configuration)

---

### jenkinsci original (for reference)

The unmodified [jenkinsci/.github](https://github.com/jenkinsci/.github) scheme, included here for comparison. Contains Jenkins-specific labels not relevant outside the Jenkins ecosystem.

Differences from this template's scheme:
- Includes `rfe` and `major-rfe` (Jenkins-specific feature request labels)
- Includes `localization` and `developer` categories
- Uses `maintenance` and `internal` instead of `cleanup`
- No `major-feature`, `ci`, `build`, `dependency`, `revert`, `reverted`, `no-changelog`, `invalid`

**Reference:** [jenkinsci/.github — release-drafter.yml](https://github.com/jenkinsci/.github/blob/master/.github/release-drafter.yml)

---

## How to customize

### Adding a new label and category

1. Add the label to GitHub:
   ```bash
   gh label create "security" --color "e11d48" --description "Security fix"
   ```

2. Add it to `version-resolver` in `.github/release-drafter.yml`:
   ```yaml
   version-resolver:
     patch:
       labels:
         - security
   ```

3. Add a category so it appears as its own section in the changelog:
   ```yaml
   categories:
     - title: ':lock: Security fixes'
       labels: [security]
   ```

### Removing a label

1. Remove it from `version-resolver` and `categories` in `.github/release-drafter.yml`.
2. Optionally delete the label from GitHub:
   ```bash
   gh label delete "label-name" --yes
   ```

### Switching to the `type:` prefix scheme

Replace all label names in `.github/release-drafter.yml` with their `type:` prefixed equivalents, recreate the labels on GitHub, then update `autolabeler` entries to match.

### Disabling the autolabeler

Remove the `autolabeler` block from `.github/release-drafter.yml`. Labels will need to be applied manually on each PR.

### Removing automated version resolution

Remove the `version-resolver` block from `.github/release-drafter.yml`. release-drafter will always suggest a patch bump by default; the version can be overridden manually when editing the draft release before publishing.

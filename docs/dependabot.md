# Dependabot Integration

[Dependabot](https://docs.github.com/en/code-security/dependabot) is a GitHub built-in tool that automatically opens pull requests to update outdated dependencies. When integrated with this template, Dependabot PRs are automatically labelled and appear in the release changelog under the **Dependency updates** section.

**Reference:** [GitHub Docs — Dependabot](https://docs.github.com/en/code-security/dependabot)

---

## How it works

1. Dependabot opens a PR to bump a dependency
2. The autolabeler in `.github/release-drafter.yml` detects the changed lockfile or manifest and applies the `dependencies` label
3. When the PR is merged, release-drafter includes it in the **Dependency updates** changelog section
4. The `dependencies` label triggers a patch version bump via `version-resolver`

---

## Setup

Create `.github/dependabot.yml` to enable Dependabot. The `package-ecosystem` value depends on your project's language and toolchain.

### Common ecosystems

```yaml
# Node.js
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: weekly
```

```yaml
# Go
version: 2
updates:
  - package-ecosystem: gomod
    directory: /
    schedule:
      interval: weekly
```

```yaml
# Python (pip)
version: 2
updates:
  - package-ecosystem: pip
    directory: /
    schedule:
      interval: weekly
```

```yaml
# GitHub Actions
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
```

Multiple ecosystems can be defined in one file:

```yaml
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: weekly

  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
```

**Reference:** [GitHub Docs — Configuration options for the dependabot.yml file](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file)

---

## Autolabeler integration

The default autolabeler rule applies the `ci` label to any PR that modifies `.github/dependabot.yml`. To also label Dependabot dependency PRs automatically, extend the autolabeler in `.github/release-drafter.yml`:

```yaml
autolabeler:
  - label: dependencies
    files:
      - 'package-lock.json'
      - 'yarn.lock'
      - 'go.sum'
      - 'Cargo.lock'
      - 'requirements.txt'
      - 'Pipfile.lock'
      - 'poetry.lock'
      - 'composer.lock'
      - 'Gemfile.lock'
```

Add whichever lockfiles are relevant to your project. Dependabot always modifies the lockfile when bumping a dependency, so this reliably catches all Dependabot PRs.

Alternatively, match by branch name — Dependabot always opens PRs from branches named `dependabot/<ecosystem>/<package>`:

```yaml
autolabeler:
  - label: dependencies
    branch:
      - '/dependabot\/.+/'
```

---

## Collapsing dependency entries in the changelog

Dependency updates can produce many changelog entries. Use `collapse-after` to keep the release notes readable:

```yaml
categories:
  - title: ':arrow_up: Dependency updates'
    labels: [dependencies, dependency]
    collapse-after: 5
```

Entries beyond the threshold are hidden under a collapsible section in the GitHub release UI.

---

## Grouping Dependabot PRs (Dependabot groups)

Dependabot can group related dependency updates into a single PR, reducing noise in both pull requests and the changelog:

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: weekly
    groups:
      production-dependencies:
        dependency-type: production
      development-dependencies:
        dependency-type: development
```

With grouping enabled, one PR covers all production dependency updates and another covers all dev dependency updates — resulting in two changelog entries instead of one per package.

**Reference:** [GitHub Docs — Grouping Dependabot version updates](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#groups)

---

## Auto-merging Dependabot PRs

Dependabot PRs can be merged automatically when all required checks pass. This is useful for patch-level dependency bumps where manual review adds little value.

Enable auto-merge via GitHub repository settings (**Settings → General → Allow auto-merge**), then add a workflow that approves and enables auto-merge on Dependabot PRs:

```yaml
# .github/workflows/dependabot-auto-merge.yml
name: '🤖 Dependabot Auto-merge'

on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - name: Fetch Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v2

      - name: Enable auto-merge for patch updates
        if: steps.metadata.outputs.update-type == 'version-update:semver-patch'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Adjust the `update-type` condition to also auto-merge minor updates if desired (`version-update:semver-minor`).

**Reference:** [GitHub Docs — Automating Dependabot with GitHub Actions](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/automating-dependabot-with-github-actions)

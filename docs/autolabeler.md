# Autolabeler

The autolabeler automatically applies labels to pull requests based on configurable rules. Labels are applied when the release-drafter workflow runs — by default on every push to the default branch, which means labels are applied to recently merged PRs.

> To label PRs **before** merge (at open/update time), see [Running autolabeler on pull_request events](#running-autolabeler-on-pull_request-events).

**Reference:** [release-drafter — Autolabeler](https://github.com/release-drafter/release-drafter#autolabeler)

---

## Matchers

Each autolabeler rule targets one label and can use any combination of four matchers. A label is applied if **at least one** matcher matches (OR relationship).

| Matcher | Matches against | Pattern type |
|---------|----------------|--------------|
| `branch` | PR source branch name | regex |
| `files` | Files changed in the PR | glob |
| `title` | PR title | regex |
| `body` | PR description body | regex |

### `branch` — match by branch name

```yaml
autolabeler:
  - label: feature
    branch:
      - '/feature\/.+/'
  - label: bug
    branch:
      - '/fix\/.+/'
      - '/hotfix\/.+/'
```

### `files` — match by changed files

```yaml
autolabeler:
  - label: documentation
    files:
      - '*.md'
      - 'docs/**'
  - label: ci
    files:
      - '.github/workflows/*'
      - '.github/dependabot.yml'
```

### `title` — match by PR title

```yaml
autolabeler:
  - label: bug
    title:
      - '/fix/i'       # case-insensitive match for "fix" anywhere in title
      - '/bug/i'
  - label: chore
    title:
      - '/^chore:/i'   # title starts with "chore:"
```

### `body` — match by PR description

```yaml
autolabeler:
  - label: breaking
    body:
      - '/BREAKING CHANGE/i'
```

---

## Combining matchers

Matchers within a rule are evaluated independently — the label is applied if **any** of them match. To require multiple conditions simultaneously, use a single regex that captures the combined pattern.

```yaml
autolabeler:
  # Applied if branch starts with "feature/" OR title contains "feat"
  - label: feature
    branch:
      - '/feature\/.+/'
    title:
      - '/feat/i'
```

If a PR matches rules for multiple labels, all matching labels are applied.

---

## Extending the default rules

The default autolabeler in this template covers common branch prefixes and file patterns. To add new rules, append to the `autolabeler` block in `.github/release-drafter.yml`:

```yaml
autolabeler:
  # ... existing rules ...

  - label: security
    title:
      - '/security/i'
      - '/CVE-/i'
    body:
      - '/SECURITY/i'

  - label: dependencies
    files:
      - 'package-lock.json'
      - 'yarn.lock'
      - 'go.sum'
      - 'Cargo.lock'
```

---

## Running autolabeler on pull_request events

By default, autolabeler runs as part of the release-drafter workflow on push to the default branch — meaning labels are applied after a PR is merged. To label PRs while they are still open, add a separate workflow:

```yaml
# .github/workflows/autolabeler.yml
name: '🏷️ Auto Label'

on:
  pull_request:
    types: [opened, reopened, synchronize]

permissions:
  contents: read

jobs:
  auto-label:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - uses: release-drafter/release-drafter/autolabeler@v7
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

This reads the same `autolabeler` configuration from `.github/release-drafter.yml` and applies labels as soon as a PR is opened or updated.

> Note: PRs from forks require `pull_request_target` instead of `pull_request`. Use with caution — `pull_request_target` runs in the context of the base branch and has access to secrets. **Reference:** [GitHub Docs — Keeping your GitHub Actions and workflows secure: Understanding the risk of script injections](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions#understanding-the-risk-of-script-injections)

**Reference:** [release-drafter — Autolabeler action](https://github.com/release-drafter/release-drafter#autolabeler)

# Release Strategies

This document covers advanced release scenarios beyond the standard single-branch workflow.

---

## Multiple release lines

Some projects need to maintain multiple active release lines in parallel — for example, continuing to ship bug fixes on `v1.x` while developing new features on `v2.x`.

### Branch setup

Create a long-lived branch for each maintained release line:

```
main        ← active development (v2.x)
release/v1  ← maintenance branch (v1.x)
```

### release-drafter configuration

release-drafter needs to run separately for each branch to maintain independent draft releases. Update `.github/workflows/release-drafter.yml` to run on all maintained branches:

```yaml
on:
  push:
    branches: [main, release/v1]
  workflow_dispatch:
```

To prevent release-drafter from picking up tags from the wrong branch, use `filter-by-commitish`:

```yaml
# .github/release-drafter.yml
filter-by-commitish: true
commitish: main   # or release/v1 for the maintenance branch
```

Since each branch needs different configuration, use separate config files and specify them in the workflow. Add `if:` conditions so each job only runs on the relevant branch:

```yaml
# .github/workflows/release-drafter.yml
jobs:
  update-release-draft-v2:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: release-drafter/release-drafter@v7
        with:
          config-name: release-drafter.yml
          commitish: main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  update-release-draft-v1:
    if: github.ref == 'refs/heads/release/v1'
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: release-drafter/release-drafter@v7
        with:
          config-name: release-drafter-v1.yml
          commitish: release/v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Create `.github/release-drafter-v1.yml` with a `tag-prefix` or `filter-by-range` to scope it to the v1.x series:

```yaml
# .github/release-drafter-v1.yml
name-template: 'v$RESOLVED_VERSION'
tag-template: 'v$RESOLVED_VERSION'
filter-by-commitish: true
commitish: release/v1
filter-by-range: '1.x'   # only consider v1.x tags as baseline

# ... rest of config identical to release-drafter.yml
```

**Reference:** [release-drafter — Action inputs](https://github.com/release-drafter/release-drafter#action-inputs)

---

## Backporting fixes to a maintenance branch

When a bug fix merged into `main` also needs to land on `release/v1`, the fix must be cherry-picked or backported manually.

A common workflow:

1. Merge the fix PR into `main` as usual
2. Create a `backport/v1/fix-description` branch from `release/v1`
3. Cherry-pick the commit: `git cherry-pick <commit-sha>`
4. Open a PR targeting `release/v1`

The autolabeler will apply labels based on the branch name, so the backport PR will appear in the v1.x changelog automatically.

---

## Separate pre-release and stable lines

For projects that publish pre-releases before promoting to stable, release-drafter can maintain two draft releases in parallel from the same branch. This is covered in [`docs/versioning.md`](versioning.md#pre-release-versions).

---

## Scoping releases to specific paths (`include-paths`)

If a monorepo contains multiple independently-versioned packages, `include-paths` limits which PRs are included in a given release draft:

```yaml
# .github/release-drafter-backend.yml
include-paths:
  - packages/backend/**

# .github/release-drafter-frontend.yml
include-paths:
  - packages/frontend/**
```

Run a separate release-drafter job for each package, each with its own config file.

**Reference:** [release-drafter — Configuration options](https://github.com/release-drafter/release-drafter#configuration-options)

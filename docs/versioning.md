# Versioning Guide

---

## Semantic Versioning (semver)

This template uses [Semantic Versioning](https://semver.org) by default. A version number takes the form `MAJOR.MINOR.PATCH`:

| Segment | When to increment | Example |
|---------|-------------------|---------|
| `MAJOR` | Breaking changes — existing users must update their code | `1.4.2` → `2.0.0` |
| `MINOR` | New features that are backwards-compatible | `1.4.2` → `1.5.0` |
| `PATCH` | Bug fixes and minor changes that don't affect the API | `1.4.2` → `1.4.3` |

**Rules:**
- When `MAJOR` is incremented, `MINOR` and `PATCH` reset to `0`
- When `MINOR` is incremented, `PATCH` resets to `0`
- Versions start at `1.0.0` for the first stable release; `0.x.y` is conventionally used for initial development where the API is not yet stable

**Reference:** [semver.org](https://semver.org)

---

## How this template resolves versions

release-drafter calculates the next version automatically using the labels on merged PRs. The flow is:

```
PR label  →  version-resolver  →  $RESOLVED_VERSION  →  draft tag
```

### 1. Labels determine the bump

`.github/release-drafter.yml` defines which labels trigger which bump:

```yaml
version-resolver:
  major:
    labels: [breaking, major-feature, major-enhancement, major-bug]
  minor:
    labels: [feature, enhancement, removed, deprecated]
  patch:
    labels: [bug, fix, bugfix, regression, chore, ...]
  default: patch
```

The **highest** bump among all merged PRs since the last release wins. For example, if a release contains one `feature` PR and three `bug` PRs, the result is a minor bump.

### 2. `$RESOLVED_VERSION` is calculated

release-drafter looks at the latest published release tag, parses its version number, and increments the appropriate segment. The result is available as the `$RESOLVED_VERSION` variable.

> **Important:** The baseline must be a **published** release. A draft release is not counted. This is why the setup requires creating `v0.0.0` as the initial baseline.

### 3. The draft tag is updated

The draft release tag and title are set to `v$RESOLVED_VERSION` (e.g. `v1.3.0`). This is updated automatically on every push to the default branch.

### Overriding the version manually

When editing the draft release before publishing, you can freely change the tag and title to any version number. This overrides whatever release-drafter calculated.

**Reference:** [release-drafter — Version Resolver](https://github.com/release-drafter/release-drafter#version-resolver)

---

## Pre-release versions

A pre-release version signals that a release is not yet stable, e.g. `v1.3.0-rc.1`. Common identifiers:

| Identifier | Meaning |
|------------|---------|
| `alpha` | Early, potentially unstable |
| `beta` | Feature-complete but may have bugs |
| `rc` | Release candidate — final testing before stable |

### Enabling pre-releases in release-drafter

Add a second job to `.github/workflows/release-drafter.yml`:

```yaml
name: '📝 Update Release Draft'

on:
  push:
    branches: [master, main]
  workflow_dispatch:

jobs:
  update-release-draft:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: release-drafter/release-drafter@v7
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  update-prerelease-draft:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: release-drafter/release-drafter@v7
        with:
          prerelease: true
          prerelease-identifier: rc
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

This maintains two drafts in parallel:
- A stable draft (e.g. `v1.3.0`) — all changes since the last stable release
- A pre-release draft (e.g. `v1.3.0-rc.1`) — all changes since the last pre-release

### Pre-release workflow

1. Publish a pre-release draft (e.g. `v1.3.0-rc.1`) when you want early feedback
2. Continue merging PRs — the next pre-release draft becomes `v1.3.0-rc.2`
3. When ready, publish the stable draft as `v1.3.0`

**Reference:** [release-drafter — Prerelease workflow](https://github.com/release-drafter/release-drafter#prerelease-workflow)

---

## Non-semver versioning

Some projects don't follow semver. release-drafter supports custom version formats via `version-template`.

### CalVer (Calendar Versioning)

CalVer encodes the release date into the version number. Common formats:

| Format | Example | Description |
|--------|---------|-------------|
| `YYYY.MM.DD` | `2025.04.19` | Full date |
| `YYYY.MM` | `2025.04` | Year and month |
| `YYYY.MINOR` | `2025.3` | Year with release counter |

CalVer is popular for projects where time of release is meaningful, such as OS distributions (Ubuntu: `24.04`), firmware, and data packages.

**Reference:** [calver.org](https://calver.org)

release-drafter has limited native support for CalVer — `$RESOLVED_VERSION` is always semver-based. The practical approach is to remove `version-resolver` entirely and set the version manually when editing the draft:

```yaml
# .github/release-drafter.yml — CalVer setup
name-template: '$RESOLVED_VERSION'   # placeholder; overridden manually at release time
tag-template: '$RESOLVED_VERSION'    # placeholder; overridden manually at release time

# No version-resolver block
```

When publishing, edit the draft tag to the desired CalVer string (e.g. `2025.04`).

### Single-number versioning (v1 → v2 → v3)

Some projects use a single incrementing integer as the version number. release-drafter does not natively support this — `$RESOLVED_VERSION` always resolves to a three-segment semver value.

The practical approach is also to remove `version-resolver` and set the version manually:

```yaml
# .github/release-drafter.yml — single-number setup
name-template: 'v$NEXT_MAJOR_VERSION'   # shows the next major as a hint
tag-template: 'v$NEXT_MAJOR_VERSION'

# No version-resolver block
```

When editing the draft release before publishing, change the tag from the suggested value (e.g. `v5.0.0`) to the next integer (e.g. `v5`). release-drafter parses previous tags using `semver.coerce()` — single-digit tags like `v4` are coerced to `4.0.0` internally — so the suggestion will be a full semver, but overriding it manually each time is straightforward.

### Custom format with `version-template`

`version-template` controls how `$MAJOR`, `$MINOR`, and `$PATCH` are combined into a version string. This only affects the format — the underlying increment logic remains semver.

```yaml
# Two-segment versioning: 1.0, 1.1, 2.0
version-template: '$MAJOR.$MINOR'
```

```yaml
# Prefixed build number
version-template: 'build.$PATCH'
```

> **Note:** release-drafter parses the previous release tag using `semver.coerce()` to determine the next version. If your tags are not semver-parseable (e.g. `2025.04.19`), `$RESOLVED_VERSION` will not work correctly — set the version manually instead.

**Reference:** [release-drafter — Projects that don't use Semantic Versioning](https://github.com/release-drafter/release-drafter#projects-that-dont-use-semantic-versioning)

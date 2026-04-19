# Setup & Maintenance Guide

A guide for setting up and maintaining the automated, PR-driven release pipeline provided by this template.

---

## One-Time Setup

### 1. GitHub Repository Settings

Go to **Settings → Actions → General**:

- **Workflow permissions** → select **Read and write permissions**
- Enable **Allow GitHub Actions to create and approve pull requests**

### 2. Create GitHub Labels

Run the following `gh` commands to create all required labels:

```bash
# Semver bump triggers
gh label create "breaking"          --color "b91c1c" --description "Breaking changes"          # major bump
gh label create "major-feature"     --color "e11d48" --description "Major new feature"         # major bump
gh label create "major-enhancement" --color "c2410c" --description "Major enhancement"         # major bump
gh label create "major-bug"         --color "dc2626" --description "Major bug fix"             # major bump
gh label create "feature"           --color "1d4ed8" --description "New feature"               # minor bump
gh label create "enhancement"       --color "a2eeef" --description "New feature or request"    # minor bump
gh label create "removed"           --color "7f1d1d" --description "Removed features"          # minor bump
gh label create "deprecated"        --color "78716c" --description "Deprecated"                # minor bump
gh label create "bug"               --color "d73a4a" --description "Something isn't working"   # patch bump
gh label create "bugfix"            --color "ef4444" --description "Bug fix"                   # patch bump
gh label create "fix"               --color "ef4444" --description "Bug fix"                   # patch bump
gh label create "regression"        --color "f97316" --description "Regression fix"            # patch bump
gh label create "revert"            --color "6b7280" --description "Reverted changes"
gh label create "reverted"          --color "6b7280" --description "Reverted changes"

# Changelog categories (patch bump)
gh label create "chore"             --color "9ca3af" --description "Maintenance"
gh label create "cleanup"           --color "9ca3af" --description "Code cleanup"
gh label create "documentation"     --color "0075ca" --description "Documentation"
gh label create "dependencies"      --color "14b8a6" --description "Dependency updates"
gh label create "dependency"        --color "14b8a6" --description "Dependency update"
gh label create "ci"                --color "f59e0b" --description "CI and build changes"
gh label create "build"             --color "f59e0b" --description "Build changes"
gh label create "test"              --color "a855f7" --description "Tests"
gh label create "tests"             --color "a855f7" --description "Tests"

# Special
gh label create "no-changelog"      --color "e5e7eb" --description "Exclude from changelog"
gh label create "skip-changelog"    --color "e5e7eb" --description "Exclude from changelog"
gh label create "invalid"           --color "e4e669" --description "This doesn't seem right"
```

### 3. Create a Baseline Release

release-drafter needs an existing published release to calculate the next version from:

```bash
git tag v0.0.0
git push origin v0.0.0
gh release create v0.0.0 --title "v0.0.0" --notes "Baseline release" --prerelease
```

### 4. Configure the Build Workflows

Replace the placeholder steps in the workflow files with your own build commands:

- **`.github/workflows/build.yml`** — CI steps that run on every push and PR
- **`.github/workflows/publish.yml`** — build steps that run when a release is published; output should land in `./artifacts/`

The specific steps depend on your language and toolchain. Refer to the [GitHub Actions documentation](https://docs.github.com/en/actions) and the [GitHub Actions Marketplace](https://github.com/marketplace?type=actions) for available actions (e.g. `actions/setup-node`, `actions/setup-go`, `actions/setup-java`).

---

## Day-to-Day Development Workflow

1. **Create a branch** with a meaningful prefix — the autolabeler applies labels automatically:

   | Branch prefix | Label applied |
   |---------------|---------------|
   | `feature/xxx` | `feature` |
   | `fix/xxx` | `bug` |
   | `chore/xxx` | `chore` |
   | `ci/xxx` | `ci` |
   | `docs/xxx` | `documentation` |

2. **Open a PR** targeting `master`/`main`. Verify the label is correct; adjust manually if needed.

3. **Merge the PR** — recommended: **squash merge** for a clean linear history. Squash merge ensures each PR produces exactly one commit and one changelog entry, regardless of how many commits were made on the branch.

4. release-drafter automatically updates the draft release with the PR title in the correct changelog section.

---

## Releasing a New Version

### Step 1 — Review the draft release

Go to **Releases** on GitHub. You will see a draft with the accumulated changelog. Check that all entries look correct.

If a PR is missing: verify it had a label that is not in `exclude-labels`. PRs without any recognized label are excluded by default.

### Step 2 — Edit and publish the release

Click **Edit** on the draft:

- Adjust the release notes if needed
- Confirm the tag and title are correct (e.g. `v1.2.0`)
- Click **Publish release**

### Step 3 — Artifacts upload automatically

`publish.yml` triggers on `release: released`. Within a few minutes the compiled artifact will be attached to the release.

If the workflow fails, re-run it manually: **Actions → 🚀 Publish → Run workflow** → enter the tag.

---

## Label Reference

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
| `no-changelog`, `skip-changelog` | *(excluded from changelog)* | — |

---

## Troubleshooting

### Draft release shows wrong version

Ensure a published release exists as a baseline (at minimum `v0.0.0`). release-drafter uses the latest published release to calculate `$RESOLVED_VERSION`. A draft release does not count.

### release-drafter fails with 403 / permission error

Go to **Settings → Actions → General** and verify:
- Workflow permissions = **Read and write permissions**
- **Allow GitHub Actions to create and approve pull requests** is checked

### A merged PR does not appear in the changelog

- The PR must have been merged into the default branch **after** the baseline release was published
- The PR must have at least one label that is not in `exclude-labels`
- Labels added **after** the merge are not retroactively picked up — release-drafter reads labels at draft-update time, so adding the label and manually re-running the workflow fixes this

### Unwanted draft release keeps appearing

Every push to `master`/`main` re-runs release-drafter, which creates or updates a draft. This is normal and harmless — the draft will not publish until you explicitly do so. Delete it if you want a clean slate:

```bash
gh release delete vX.Y.Z --yes
```

### Artifact upload failed after publish

Manually re-trigger the publish workflow:

```bash
gh workflow run publish.yml -f tag=vX.Y.Z
```

---

## References

- [release-drafter/release-drafter](https://github.com/release-drafter/release-drafter) — Official repository and full documentation
  - [Configuration options](https://github.com/release-drafter/release-drafter#configuration-options) — All available keys for `.github/release-drafter.yml`
  - [Template variables](https://github.com/release-drafter/release-drafter#template-variables) — `$CHANGES`, `$CONTRIBUTORS`, `$PREVIOUS_TAG`, etc.
  - [Version resolver](https://github.com/release-drafter/release-drafter#version-resolver) — How labels map to semver bumps via `$RESOLVED_VERSION`
  - [Autolabeler](https://github.com/release-drafter/release-drafter#autolabeler) — Auto-applying labels based on branch name, file paths, PR title/body
  - [Action inputs](https://github.com/release-drafter/release-drafter#action-inputs) — Workflow-level overrides (e.g. `prerelease`, `publish`, `commitish`)
- [release-drafter on GitHub Marketplace](https://github.com/marketplace/actions/release-drafter)
- [jenkinsci/.github](https://github.com/jenkinsci/.github) — Source of the label scheme used in this template
- [`docs/label-schemes.md`](label-schemes.md) — Survey of label schemes from different projects and customization guide
- [`docs/autolabeler.md`](autolabeler.md) — Autolabeler matchers, extending rules, and running on pull_request events
- [`docs/changelog.md`](changelog.md) — Customizing changelog format, PR entry templates, and replacers
- [`docs/dependabot.md`](dependabot.md) — Setting up Dependabot and integrating it with the release pipeline
- [`docs/versioning.md`](versioning.md) — Semver basics, how version resolution works, pre-releases, and non-semver alternatives
- [`docs/release-strategies.md`](release-strategies.md) — Multiple release lines, backporting, and monorepo setups

# gh-release-template

A GitHub repository template for setting up an automated, PR-driven release pipeline using [release-drafter](https://github.com/release-drafter/release-drafter). Works with any language or framework.

## What's included

| File | Purpose |
|------|---------|
| `.github/workflows/build.yml` | CI — runs on every push and PR |
| `.github/workflows/release-drafter.yml` | Maintains a rolling draft release with auto-generated changelog |
| `.github/workflows/publish.yml` | Builds and uploads artifacts when a release is published |
| `.github/release-drafter.yml` | release-drafter configuration: labels, changelog categories, version resolver |

## How to use this template

1. Click **Use this template** → **Create a new repository**
2. Follow the setup guide in [`docs/setup.md`](docs/setup.md)
3. Replace the placeholder build steps in `build.yml` and `publish.yml` with your own commands

## Release workflow at a glance

```
feature/xxx branch  →  PR  →  merge  →  release-drafter updates draft
                                                    ↓
                                         review & publish draft
                                                    ↓
                                         publish.yml uploads artifacts
```

See [`docs/setup.md`](docs/setup.md) for the full setup and day-to-day workflow guide.

## Documentation

| Document | Description |
|----------|-------------|
| [`docs/setup.md`](docs/setup.md) | One-time setup, day-to-day workflow, releasing, troubleshooting |
| [`docs/autolabeler.md`](docs/autolabeler.md) | Autolabeler matchers, extending rules, labeling PRs before merge |
| [`docs/changelog.md`](docs/changelog.md) | Customizing changelog format, PR entry templates, replacers |
| [`docs/dependabot.md`](docs/dependabot.md) | Dependabot setup and integration with the release pipeline |
| [`docs/label-schemes.md`](docs/label-schemes.md) | Label schemes from different projects and customization guide |
| [`docs/versioning.md`](docs/versioning.md) | Semver, version resolution, pre-releases, non-semver alternatives |
| [`docs/release-strategies.md`](docs/release-strategies.md) | Multiple release lines, backporting, monorepo setups |

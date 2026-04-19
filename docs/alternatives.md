# Alternatives

This template is built around release-drafter, but it is not the only tool for automating releases. This document compares release-drafter with semantic-release, the most common alternative.

---

## release-drafter vs semantic-release

### release-drafter

release-drafter collects merged PRs, generates a changelog, and suggests the next version — but the actual publish is always a manual step.

**Strengths:**
- No commit message format required — labeling PRs is enough
- Human review before every release; version can be overridden freely
- Simple setup: one workflow file and one config file
- Works with any branching or merging strategy

**Weaknesses:**
- Not fully automated — someone must click Publish
- Version is driven by labels, which must be applied correctly to every PR

**Best for:** Teams that want automation for the tedious parts (changelog, versioning suggestion) but prefer a human sign-off before each release.

---

### semantic-release

semantic-release fully automates the release process: it parses commit messages, determines the version bump, creates the release, and uploads artifacts — all without human intervention.

**Strengths:**
- Fully automated end-to-end: no manual publish step
- Version is derived directly from commit messages, making intent explicit in the git history
- Rich plugin ecosystem (npm publish, Docker, Slack notifications, etc.)

**Weaknesses:**
- Requires the entire team to follow [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `feat!:`, etc.) — a discipline overhead
- A malformed commit message can produce an incorrect version bump or block the release
- More complex setup and harder to override version manually
- Fully automated means no opportunity for a human review before publish

**Best for:** Projects where all contributors consistently follow Conventional Commits and full automation is desired.

---

## Summary

| | release-drafter | semantic-release |
|---|---|---|
| Version source | PR labels | Commit messages (Conventional Commits) |
| Publish | Manual | Automatic on push |
| Human review | Yes | No |
| Commit format required | No | Yes |
| Setup complexity | Low | Medium–High |
| Version override | Easy | Difficult |

---

## References

- [release-drafter/release-drafter](https://github.com/release-drafter/release-drafter)
- [semantic-release/semantic-release](https://github.com/semantic-release/semantic-release)
- [Conventional Commits](https://www.conventionalcommits.org/)

# Changelog Customization

release-drafter generates the release body from a set of configurable templates. This document covers the available options and common customization patterns.

**Reference:** [release-drafter — Configuration options](https://github.com/release-drafter/release-drafter#configuration-options)

---

## Release body structure

The release body is assembled from three optional sections:

```
header
template  (contains $CHANGES)
footer
```

All three are defined in `.github/release-drafter.yml`:

```yaml
header: |
  Full changelog: https://github.com/$OWNER/$REPOSITORY/compare/$PREVIOUS_TAG...v$RESOLVED_VERSION

template: |
  ## :sparkles: What's New

  $CHANGES

footer: |
  ---
  _Contributors: $CONTRIBUTORS_
```

---

## Template variables

These variables are available in `template`, `header`, and `footer`:

| Variable | Description |
|----------|-------------|
| `$CHANGES` | The generated list of merged PRs, grouped by category |
| `$CONTRIBUTORS` | Comma-separated list of contributors (PR authors, commit authors, committers) |
| `$PREVIOUS_TAG` | The tag of the previous published release |
| `$REPOSITORY` | Repository name |
| `$OWNER` | Repository owner |

Next version variables are also available (see [`docs/versioning.md`](versioning.md)):

| Variable | Example |
|----------|---------|
| `$RESOLVED_VERSION` | `1.3.0` |
| `$NEXT_MAJOR_VERSION` | `2.0.0` |
| `$NEXT_MINOR_VERSION` | `1.4.0` |
| `$NEXT_PATCH_VERSION` | `1.3.1` |
| `$NEXT_PRERELEASE_VERSION` | `1.3.0-rc.1` |

**Reference:** [release-drafter — Template variables](https://github.com/release-drafter/release-drafter#template-variables)

---

## Customizing PR entries (`change-template`)

Each merged PR appears as one entry in `$CHANGES`. The format is controlled by `change-template`:

```yaml
# Default
change-template: '* $TITLE (#$NUMBER) @$AUTHOR'
```

Available variables:

| Variable | Description |
|----------|-------------|
| `$TITLE` | PR title |
| `$NUMBER` | PR number |
| `$AUTHOR` | PR author's username |
| `$BODY` | PR description body |
| `$URL` | PR URL |
| `$BASE_REF_NAME` | Target branch name |
| `$HEAD_REF_NAME` | Source branch name |

**Reference:** [release-drafter — Change template variables](https://github.com/release-drafter/release-drafter#change-template-variables)

### Examples

```yaml
# Include PR URL as a link
change-template: '* [$TITLE]($URL) @$AUTHOR'
```

```yaml
# Dash instead of asterisk, no author
change-template: '- $TITLE (#$NUMBER)'
```

### Escaping markdown characters in titles

PR titles may contain characters that are interpreted as markdown (e.g. `*`, `_`, `<`). Use `change-title-escapes` to escape them:

```yaml
change-title-escapes: '\<*_&'
```

To also disable `@mentions` and `#issue` links in titles:

```yaml
change-title-escapes: '\<*_&#@'
```

**Reference:** [release-drafter — Configuration options](https://github.com/release-drafter/release-drafter#configuration-options)

---

## Customizing category headings (`category-template`)

Category headings in `$CHANGES` are controlled by `category-template`:

```yaml
# Default — renders as h2
category-template: '## $TITLE'

# Render as h3 (used in this template)
category-template: '### $TITLE'
```

---

## Collapsing long category lists

Categories with many entries can be collapsed by default using `collapse-after`. This is particularly useful for dependency updates:

```yaml
categories:
  - title: ':arrow_up: Dependency updates'
    labels: [dependencies, dependency]
    collapse-after: 5   # collapse if more than 5 entries
```

Setting `collapse-after: 0` always collapses the category regardless of entry count.

**Reference:** [release-drafter — Configuration options (`collapse-after`)](https://github.com/release-drafter/release-drafter#configuration-options)

---

## When there are no changes

The text shown when no PRs are included in the release:

```yaml
no-changes-template: '* No changes'  # default
```

---

## Search and replace with `replacers`

`replacers` can transform content in the generated changelog body using regular expressions:

```yaml
replacers:
  # Link CVE numbers
  - search: '/CVE-(\d{4})-(\d+)/g'
    replace: '[CVE-$1-$2](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-$1-$2)'

  # Capitalize first letter of each entry
  - search: '/- ([a-z])/g'
    replace: '- \u$1'
```

`search` is parsed as a JavaScript RegExp. `replace` supports the same substitution syntax as VSCode's find-and-replace.

**Reference:** [release-drafter — Replacers](https://github.com/release-drafter/release-drafter#replacers)

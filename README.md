# BOS Automation Toolkit

Reusable GitHub Actions workflows for testing Python-based Actions and validating
or releasing GitHub Marketplace Actions. The workflows depend only on public
actions and do not require organization-specific configuration.

## Marketplace validation

Call the reusable workflow from a repository workflow:

```yaml
name: Marketplace validation

on:
  pull_request:
    branches: [dev]

permissions:
  contents: read

jobs:
  marketplace:
    uses: blackoutsecure/bos-automation-toolkit/.github/workflows/marketplace-check.yml@<immutable-ref>
```

The check uses `bos-marketplace-kit` defaults and the caller repository's
`.github/bos-universal-config.json` or Marketplace Kit config file, when present.
It does not impose Blackout Secure organization-wide policy. Configure rule
overrides in the consumer repository or pass the supported workflow inputs.
When enabling the optional name check, supply `proposed_name` explicitly.

## Python Action tests

```yaml
jobs:
  tests:
    uses: blackoutsecure/bos-automation-toolkit/.github/workflows/action-test.yml@<immutable-ref>
    with:
      python_versions: '["3.11", "3.12"]'
      install_packages: '["pytest", "build"]'
      pytest_args: '["-q", "tests"]'
```

The workflow runs on `ubuntu-latest`, installs only the packages and optional
requirements file selected by the caller, and invokes pytest without evaluating
caller-provided shell commands.

## Marketplace release

The promotion workflow validates the source branch, copies the explicit
allowlist to the Marketplace branch, creates a version tag, and optionally
publishes a GitHub Release. It intentionally requires an explicit SemVer tag
and allowlist; it does not infer a repository's release policy.
The source branch is resolved once, validated by SHA, and rejected if it moves
before promotion. The promotion composite is adapted from the Apache-2.0
`bos-marketplace-kit` v0.1.28 action and adds this source-SHA guard.
The release workflow checks Marketplace policy but does not run the project's
tests or build; make those required upstream jobs before calling it.

```yaml
jobs:
  release:
    permissions:
      contents: write
    uses: blackoutsecure/bos-automation-toolkit/.github/workflows/marketplace-release.yml@<immutable-ref>
    with:
      source_branch: dev
      target_branch: main
      allowlist_paths: |
        action.yml
        README.md
        LICENSE
        src
      tag_name: v1.2.3
```

Grant `contents: write` only to a trusted release trigger. The promotion action
rejects workflow paths, traversal, and existing tags; it can also update a
floating major tag when explicitly enabled. Set `publish_release: false` for a
promote-and-tag-only operation or `dry_run: true` to validate without changing
remote state.
`release_files` can attach files already present in the promoted tree; generated
assets must be built and published by a caller-owned workflow instead.

Pin reusable workflow calls to an immutable commit SHA in production. Each
workflow declares the minimum permissions it needs; callers can only reduce
those permissions, not elevate them.

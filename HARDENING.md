<!-- markdownlint-disable -->

# Hardening Report: gabe565--setup-helm-docs-action/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gabe565--setup-helm-docs-action/v1.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml contains two script-injection violations:

(a) Rule (a) — `${{ runner.os }}` is interpolated directly inside the shell script: `case "${{ runner.os }}" in`. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because YAML template substitution happens before the shell ever sees the value.

(b) Rule (b) — The env var `$VERSION` (sourced from `${{ inputs.version }}`) is expanded unquoted in: `RELEASE="$(gh release view --json=name,assets $VERSION)"`. An unquoted shell expansion of an attacker-controllable value allows shell metacharacter injection (`;`, `|`, `&`, `$(...)`, etc.).

Locations:

- `action.yml:36`
- `action.yml:47`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable version tags rather than immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved or overwritten:
- `.github/workflows/release.yaml`: `actions/checkout@v4`, `gabe565/changelog-generator@v1`, `softprops/action-gh-release@v2`, `nowactions/update-majorver@v1`
- `.github/workflows/test.yaml`: `actions/checkout@v4`

Locations:

- `.github/workflows/release.yaml:11`
- `.github/workflows/release.yaml:15`
- `.github/workflows/release.yaml:17`
- `.github/workflows/release.yaml:20`
- `.github/workflows/test.yaml:11`

### missing-permissions (severity: medium)

`.github/workflows/test.yaml` has no top-level `permissions:` key and its only job (`test`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. `write` on `contents`). A minimal explicit `permissions:` block (e.g. `permissions: read-all` or specific scopes) should be added.

Locations:

- `.github/workflows/test.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed three findings:

1. script-injection (action.yml):
   - Moved `${{ runner.os }}` to the env: block as RUNNER_OS, referenced as `$RUNNER_OS` in the case statement.
   - Changed unquoted `$VERSION` in `gh release view` to `${VERSION:+"$VERSION"}` so the optional version argument is safely quoted when present and omitted when empty.

2. unpinned-uses:
   - release.yaml: Pinned actions/checkout@v4→SHA, gabe565/changelog-generator@v1→SHA, softprops/action-gh-release@v2→SHA, nowactions/update-majorver@v1→SHA.
   - test.yaml: Pinned actions/checkout@v4→SHA.
   All SHAs resolved via lookup_action_sha with tag preserved in comment.

3. missing-permissions (test.yaml):
   - Added `permissions: {}` at the top level of test.yaml to explicitly grant no permissions.

